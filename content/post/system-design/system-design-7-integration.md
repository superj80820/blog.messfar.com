+++
title = '整合撮合系統'
date = 2024-03-31T00:00:00+08:00
tags = ['go', 'system-design']
+++

![https://raw.githubusercontent.com/superj80820/system-design/master/doc/exchange-arch.png](https://raw.githubusercontent.com/superj80820/system-design/master/doc/exchange-arch.png)

source code:
* [usecase](https://github.com/superj80820/system-design/tree/master/exchange/usecase/trading)
* [repository](https://github.com/superj80820/system-design/tree/master/exchange/repository/trading)

交易系統分為兩大部分，一個是非同步的`TradingUseCase()`、一個是同步的`syncTradingUseCase()`，`TradingUseCase()`將無序的`TradingEvent`蒐集後，有序的逐筆送入`syncTradingUseCase()`，我們會希望各種IO在`TradingUseCase()`完成，撮合這些需高速計算的部分在純memory的`syncTradingUseCase()`完成並得到trading result，再交給`TradingUseCase()`進行persistent，最後達到eventual consistency。

我們先介紹`tradingUseCase`，再介紹實作相對簡單的`syncTradingUseCase`。

`tradingUseCase`需注入相關IO

```go
type tradingUseCase struct {
	userAssetRepo domain.UserAssetRepo
	tradingRepo   domain.TradingRepo
	candleRepo    domain.CandleRepo
	quotationRepo domain.QuotationRepo
	matchingRepo  domain.MatchingRepo
	orderRepo     domain.OrderRepo

	sequenceTradingUseCase domain.SequenceTradingUseCase
	syncTradingUseCase     domain.SyncTradingUseCase

	lastSequenceID int

	// another code...
}

```

`TradingUseCase`主要實現以下幾個methods

```go
type TradingUseCase interface {
	ConsumeTradingEvents(ctx context.Context, key string)

	ConsumeTradingResult(ctx context.Context, key string)

	ProcessTradingEvents(ctx context.Context, tradingEvents []*TradingEvent) error

	// another code...
}

```

`ConsumeTradingEvents()`: consume已經過定序模組定序過的trading events，並傳入`ProcessTradingEvents()`，如果成功則進行explicit commit

```go
func (t *tradingUseCase) ConsumeTradingEvents(ctx context.Context, key string) {
	// another code...

	t.tradingRepo.ConsumeTradingEvents(ctx, key, func(events []*domain.TradingEvent, commitFn func() error) {
		if err := t.ProcessTradingEvents(ctx, events); err != nil {
			// error handle code...
		}
		if err := commitFn(); err != nil {
			// error handle code...
		}
	})
}

```

`ProcessTradingEvents()`處理trading events，需先透過sequence id是確認idempotency冪等性:

- 如果event丟失: 透過`t.sequenceTradingUseCase.RecoverEvents()`從db讀取event
- 如果event重複: 忽略event

確認訊息順序正確後，才`processTradingEvent()`進行處理

```go
func (t *tradingUseCase) ProcessTradingEvents(ctx context.Context, tes []*domain.TradingEvent) error {
	err := t.sequenceTradingUseCase.CheckEventSequence(tes[0].SequenceID, t.lastSequenceID)
	if errors.Is(err, domain.ErrMissEvent) {
		t.logger.Warn("miss events. first event id", loggerKit.Int("first-event-id", tes[0].SequenceID), loggerKit.Int("last-sequence-id", t.lastSequenceID))
		t.sequenceTradingUseCase.RecoverEvents(t.lastSequenceID, func(tradingEvents []*domain.TradingEvent) error {
			for _, te := range tradingEvents {
				if err := t.processTradingEvent(ctx, te); err != nil {
					return errors.Wrap(err, "process trading event failed")
				}
			}
			return nil
		})
		return nil
	}
	for _, te := range tes {
		err := t.sequenceTradingUseCase.CheckEventSequence(te.SequenceID, t.lastSequenceID)
		if errors.Is(err, domain.ErrGetDuplicateEvent) {
			t.logger.Warn("get duplicate events. first event id", loggerKit.Int("first-event-id", tes[0].SequenceID), loggerKit.Int("last-sequence-id", t.lastSequenceID))
			continue
		}
		if err := t.processTradingEvent(ctx, te); err != nil {
			return errors.Wrap(err, "process trading event failed")
		}
	}
	return nil
}

```

sequence id更新至`t.lastSequenceID`，並依照event type呼叫對應的method給`syncTradingUseCase`獲取對應result，以新增訂單來說，`syncTradingUseCase.CreateOrder()`會獲得`MatchResult`、`TransferResult`，需將他們包裝成`TradingResult`，透過`tradingRepo.ProduceTradingResult()`傳至`trading result MQ`。

這裡的`trading result MQ`，是用memory的MQ，實際上是當做一個`ring buffer`，把trading result蒐集起來，並以batch的方式傳入下游系統MQ，以增加throughput，不然一筆一筆傳入下游，這裡會成為bottleneck。

```go
func (t *tradingUseCase) processTradingEvent(ctx context.Context, te *domain.TradingEvent) error {
	var tradingResult domain.TradingResult

	t.lastSequenceID = te.SequenceID

	switch te.EventType {
	case domain.TradingEventCreateOrderType:
		matchResult, transferResult, err := t.syncTradingUseCase.CreateOrder(ctx, te)
		if errors.Is(err, domain.LessAmountErr) || errors.Is(err, domain.InvalidAmountErr) {
			t.logger.Info(fmt.Sprintf("%+v", err))
			return nil
		} else if err != nil {
			return errors.Wrap(err, "process message get failed")
		}

		tradingResult = domain.TradingResult{
			SequenceID:          te.SequenceID,
			TradingResultStatus: domain.TradingResultStatusCreate,
			TradingEvent:        te,
			MatchResult:         matchResult,
			TransferResult:      transferResult,
		}
	case domain.TradingEventCancelOrderType:
		// another code...
	case domain.TradingEventTransferType:
		// another code...
	case domain.TradingEventDepositType:
		// another code...
	default:
		return errors.New("unknown event type")
	}

	if err := t.tradingRepo.ProduceTradingResult(ctx, &tradingResult); err != nil {
		panic(errors.Wrap(err, "produce trading result failed"))
	}

	return nil
}

```

`ConsumeTradingResult()`batch consume `trading result MQ`，批次將trading results傳入下游`candle MQ`、`asset MQ`等下游系統。

```go
func (t *tradingUseCase) ConsumeTradingResult(ctx context.Context, key string) {
	t.tradingRepo.ConsumeTradingResult(ctx, key, func(tradingResults []*domain.TradingResult) error {
		eg, ctx := errgroup.WithContext(ctx)

		eg.Go(func() error {
			if err := t.userAssetRepo.ProduceUserAssetByTradingResults(ctx, tradingResults); err != nil {
				return errors.Wrap(err, "produce order failed")
			}
			return nil
		})

		eg.Go(func() error {
			if err := t.candleRepo.ProduceCandleMQByTradingResults(ctx, tradingResults); err != nil {
				return errors.Wrap(err, "produce candle failed")
			}
			return nil
		})

  	// another code...

		if err := eg.Wait(); err != nil {
			panic(errors.Wrap(err, "produce failed"))
		}

		return nil
	})
}

```

`syncTradingUseCase`相對簡單，需注入資產模組、訂單模組、撮合模組、清算模組來實作`CreateOrder()`等等methods

```go
type syncTradingUseCase struct {
	userAssetUseCase domain.UserAssetUseCase
	orderUseCase     domain.OrderUseCase
	matchingUseCase  domain.MatchingUseCase
	clearingUseCase  domain.ClearingUseCase
}

type SyncTradingUseCase interface {
	CreateOrder(ctx context.Context, tradingEvent *TradingEvent) (*MatchResult, *TransferResult, error)

	// another code...
}

```

`CreateOrder()`即帶入`TradingEvent`後，依序呼叫資產模組、訂單模組、撮合模組、清算模組，並獲得各個results後回傳。

```go
func (t *syncTradingUseCase) CreateOrder(ctx context.Context, tradingEvent *domain.TradingEvent) (*domain.MatchResult, *domain.TransferResult, error) {
	order, transferResult, err := t.orderUseCase.CreateOrder(
		ctx,
		tradingEvent.SequenceID,
		tradingEvent.OrderRequestEvent.OrderID,
		tradingEvent.OrderRequestEvent.UserID,
		tradingEvent.OrderRequestEvent.Direction,
		tradingEvent.OrderRequestEvent.Price,
		tradingEvent.OrderRequestEvent.Quantity,
		tradingEvent.CreatedAt,
	)
	if err != nil {
		return nil, nil, errors.Wrap(err, "create order failed")
	}
	matchResult, err := t.matchingUseCase.NewOrder(ctx, order)
	if err != nil {
		return nil, nil, errors.Wrap(err, "matching order failed")
	}
	clearTransferResult, err := t.clearingUseCase.ClearMatchResult(ctx, matchResult)
	if err != nil {
		return nil, nil, errors.Wrap(err, "clear match order failed")
	}

	transferResult.UserAssets = append(transferResult.UserAssets, clearTransferResult.UserAssets...)

	return matchResult, transferResult, nil
}

```

這樣就完成了一個交易系統，需要注意的是:

- 下游系統是可能收到重複的消息的，需忽略這些消息，可以將last sequence id存至redis，也可將db table設有sequence id欄位來檢查
- 交易系統是有可能崩潰的，雖然我們可以靠儲存在db的sequence trading events來recover，但每次都從頭讀取events會消耗大量時間，我們可以將交易系統memory的狀態每隔一段時間snapshot備份起來，在下次recover的時候先從snapshot讀取，再從db讀取
- 此實現為單一交易對。如需多個交易對可以架設多個`app/exchange-gitbitex`