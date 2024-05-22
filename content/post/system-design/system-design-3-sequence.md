+++
title = 'Sequence 定序模組'
date = 2024-03-27T00:00:00+08:00
tags = ['go', 'system-design']
+++

![https://raw.githubusercontent.com/superj80820/system-design/master/doc/sequencer.jpg](https://raw.githubusercontent.com/superj80820/system-design/master/doc/sequencer.jpg)

```go
type SequenceTradingUseCase interface {
	ProduceCreateOrderTradingEvent(ctx context.Context, userID int, direction DirectionEnum, price, quantity decimal.Decimal) (*TradingEvent, error)
	ConsumeSequenceMessages(context.Context)

	SequenceAndSaveWithFilter(events []*TradingEvent, commitFn func() error) ([]*TradingEvent, error)

	// another code...
}

```

如何快速儲存event是影響系統寫入速度的關鍵，kafka是可考慮的選項之一。

kafka為append-only logs，不需像RDBMS在需查找與更新索引會增加磁碟I/O操作，並且使用zero-copy快速寫入磁碟來persistent。

create order API只需將snowflake的`orderID`、`referenceID`(全局參考ID)等metadata帶入event，並傳送給kafka sequence topic，即完成了創建訂單的事項，可回傳`200 OK`給客戶端。

```go
func (t *tradingSequencerUseCase) ProduceCreateOrderTradingEvent(ctx context.Context, userID int, direction domain.DirectionEnum, price, quantity decimal.Decimal) (*domain.TradingEvent, error) {
	referenceID, err := utilKit.SafeInt64ToInt(utilKit.GetSnowflakeIDInt64())
	if err != nil {
		return nil, errors.Wrap(err, "safe int64 to int failed")
	}
	orderID, err := utilKit.SafeInt64ToInt(utilKit.GetSnowflakeIDInt64())
	if err != nil {
		return nil, errors.Wrap(err, "safe int64 to int failed")
	}
	if price.LessThanOrEqual(decimal.Zero) {
		return nil, errors.Wrap(err, "amount is less then or equal zero failed")
	}
	if quantity.LessThanOrEqual(decimal.Zero) {
		return nil, errors.Wrap(err, "quantity is less then or equal zero failed")
	}

	tradingEvent := &domain.TradingEvent{
		ReferenceID: referenceID,
		EventType:   domain.TradingEventCreateOrderType,
		OrderRequestEvent: &domain.OrderRequestEvent{
			UserID:    userID,
			OrderID:   orderID,
			Direction: direction,
			Price:     price,
			Quantity:  quantity,
		},
		CreatedAt: time.Now(),
	}

	if err := t.produceSequenceMessages(ctx, tradingEvent); err != nil {
		return nil, errors.Wrap(err, "send trade sequence messages failed")
	}

	return tradingEvent, nil
}

```

kafka sequence topic的consume到event後，需為event定序，將一批events的透過`SequenceAndSaveWithFilter()`定序與儲存。

`SequenceAndSaveWithFilter()`過程中有可能會有失敗，如果失敗就不對kafka進行commit，下次consume會消費到同批events重試，直到成功再commit，此方式是為explicit commit。

但如果save成功commit卻失敗呢？這時可能導致消息重複`ErrDuplicate`，需透過`sequencerRepo.FilterEvents()`來filter掉已save的events，只儲存新的events，再用新的event呼叫`SequenceAndSaveWithFilter()`，如果消息完全filter掉了則回傳`ErrNoop`錯誤，代表此批消息完全重複，不處理。

最後將以定序的events透過`tradingRepo.ProduceTradingEvents`送至`trading event MQ`。

```go
func (t *tradingSequencerUseCase) ConsumeSequenceMessages(ctx context.Context) {
	t.sequencerRepo.ConsumeSequenceMessages(func(sequenceEvents []*domain.SequencerEvent, commitFn func() error) {
		tradingEvents, err := t.sequenceEventsConvertToTradingEvents(sequenceEvents)
		if err != nil {
			panic(errors.Wrap(err, "convert sequence event failed"))
		}

		events, err := t.SequenceAndSaveWithFilter(tradingEvents, commitFn)
		if errors.Is(err, domain.ErrDuplicate) {
			sequenceEvents, err = t.sequencerRepo.FilterEvents(sequenceEvents)
			if errors.Is(err, domain.ErrNoop) {
				return
			} else if err != nil {
				panic(errors.Wrap(err, "filter events failed"))
			}

			tradingEvents, err := t.sequenceEventsConvertToTradingEvents(sequenceEvents)
			if err != nil {
				panic(errors.Wrap(err, "convert sequence event failed"))
			}

			_, err = t.SequenceAndSaveWithFilter(tradingEvents, commitFn)
			if err != nil {
				panic(errors.Wrap(err, fmt.Sprintf("save with filter events failed, events length: %d", len(events))))
			}
		} else if err != nil {
			panic(errors.Wrap(err, fmt.Sprintf("save with filter events failed, events length: %d", len(events))))
		}

		if err := t.tradingRepo.ProduceTradingEvents(ctx, events); err != nil {
			panic(errors.Wrap(err, "produce trading event failed"))
		}
	})
}

```

`sequencerRepo.FilterEvents()`的方式有許多，此處是透過全局唯一`ReferenceID`來達到，如果db已儲存此`ReferenceID`，則忽略此event。

```go
func (s *sequencerRepo) FilterEvents(sequenceEvents []*domain.SequencerEvent) ([]*domain.SequencerEvent, error) {
	referenceIDFilter, err := s.GetReferenceIDFilterMap(sequenceEvents)
	if err != nil {
		return nil, errors.Wrap(err, "get filter events map failed")
	}
	for _, val := range sequenceEvents {
		if referenceIDFilter[val.ReferenceID] {
			continue
		}
		filterSequenceEvents = append(filterSequenceEvents, val)
	}

	if len(filterSequenceEvents) == 0 {
		return nil, errors.Wrap(domain.ErrNoop, "no message need to save")
	}

	return filterSequenceEvents, nil
}

```