+++
title = 'Matching System Part6: Clearing 清算模組'
date = 2024-03-30T00:00:00+08:00
tags = ['go', 'system-design']
+++

Tags: Golang, Maching System, System Design
Date: March 31, 2024

![https://raw.githubusercontent.com/superj80820/system-design/master/doc/clearing.jpg](https://raw.githubusercontent.com/superj80820/system-design/master/doc/clearing.jpg)

source code:
* [usecase](https://github.com/superj80820/system-design/tree/master/exchange/usecase/clearing)

---

撮合模組撮合後，雙方資產還沒有實際交換，訂單也還沒更新，必須再透過清算模組進行處理，故注入資產模組與訂單模組

```go
type clearingUseCase struct {
	userAssetUseCase domain.UserAssetUseCase
	orderUseCase     domain.OrderUseCase

	// another code...
}

```

清算模組需實作的method不多，只需實作`ClearMatchResult()`，將`MatchResult`帶入，資產的轉換結果生成`TransferResult`提供給下游系統使用

```go
type ClearingUseCase interface {
	ClearMatchResult(ctx context.Context, matchResult *MatchResult) (*TransferResult, error)

	// another code...
}

```

將taker訂單更新，並依照taker的方向進行清算，以下分別介紹

```go
func (c *clearingUseCase) ClearMatchResult(ctx context.Context, matchResult *domain.MatchResult) (*domain.TransferResult, error) {
	// 新增`transferResult`紀錄資產轉換結果
	transferResult := new(domain.TransferResult)

	// 更新taker在訂單模組的狀態
	taker := matchResult.TakerOrder
	if err := c.orderUseCase.UpdateOrder(ctx, taker.ID, taker.UnfilledQuantity, taker.Status, taker.UpdatedAt); err != nil {
		return nil, errors.Wrap(err, "update order failed")
	}
	switch matchResult.TakerOrder.Direction {
	// taker是賣單的情形
	case domain.DirectionSell:
		// another code...
	// taker是買單的情形
	case domain.DirectionBuy:
		// another code...
	default:
		return nil, errors.New("unknown direction")
	}

	return transferResult, nil
}

```

如果taker是賣單，依照`MatchResult`的`MatchDetails`逐一將maker訂單更新:

- 將taker賣單凍結的數量轉換給maker，數量是`matched`
- 將maker買單凍結的資產轉換給taker，金額是`(maker price)*matched`

並把taker跟maker最後的資產紀錄在`TransferResult`

```go
	// taker是賣單的情形
	case domain.DirectionSell:
		// 依照`MatchDetails`逐一拿出撮合maker的細節來處理
		for _, matchDetail := range matchResult.MatchDetails {
			// 更新maker在訂單模組的狀態
			maker := matchDetail.MakerOrder
			if err := c.orderUseCase.UpdateOrder(ctx, maker.ID, maker.UnfilledQuantity, maker.Status, maker.UpdatedAt); err != nil {
				return nil, errors.Wrap(err, "update order failed")
			}
			matched := matchDetail.Quantity

			// 將taker賣單凍結的數量轉換給maker，數量是matched
			transferResultOne, err := c.userAssetUseCase.TransferFrozenToAvailable(ctx, taker.UserID, maker.UserID, c.baseCurrencyID, matched)
			if err != nil {
				return nil, errors.Wrap(err, "transfer failed")
			}
			// 將轉換結果紀錄
			transferResult.UserAssets = append(transferResult.UserAssets, transferResultOne.UserAssets...)
			// 將maker買單凍結的資產轉換給taker，金額是(maker price)*matched
			transferResultTwo, err := c.userAssetUseCase.TransferFrozenToAvailable(ctx, maker.UserID, taker.UserID, c.quoteCurrencyID, maker.Price.Mul(matched))
			if err != nil {
				return nil, errors.Wrap(err, "transfer failed")
			}
			// 將轉換結果紀錄
			transferResult.UserAssets = append(transferResult.UserAssets, transferResultTwo.UserAssets...)
			// 如果maker買單數量減至0，則移除訂單
			if maker.UnfilledQuantity.IsZero() {
				if err := c.orderUseCase.RemoveOrder(ctx, maker.ID); err != nil {
					return nil, errors.Wrap(err, "remove failed")
				}
			}
		}
		// 如果taker買單數量減至0，則移除訂單
		if taker.UnfilledQuantity.IsZero() {
			if err := c.orderUseCase.RemoveOrder(ctx, taker.ID); err != nil {
				return nil, errors.Wrap(err, "remove failed")
			}
		}

```

如果taker是買單，則相反:

- 將taker買單凍結的資產轉換給maker，金額是`(maker price)*matched`
- 將maker賣單凍結的資產轉換給taker，數量是`matched`

需注意的是，如果taker是買單，凍結的資產金額是(taker price)*matched，須退還`(taker price-maker price)*matched`金額給taker。

例如10usdt price數量3btc的買單，需凍結30usdt，如果撮合到1usdt price數量2btc的賣單，撮合到2btc，taker理應用`1*2=2usdt`會獲得2btc，須退還`(10-1)*2=18usdt`給taker

```go
	// taker是買單的情形
	case domain.DirectionBuy:
		// 依照`MatchDetails`逐一拿出撮合maker的細節來處理
		for _, matchDetail := range matchResult.MatchDetails {
			// 更新maker在訂單模組的狀態
			maker := matchDetail.MakerOrder
			if err := c.orderUseCase.UpdateOrder(ctx, maker.ID, maker.UnfilledQuantity, maker.Status, maker.UpdatedAt); err != nil {
				return nil, errors.Wrap(err, "update order failed")
			}
			matched := matchDetail.Quantity

			// taker買單的價格如果高於maker賣單，則多出來的價格須退還給taker，金額是(taker price-maker price)*matched
			// 退還給taker不需紀錄在`transferResult`，因為後續taker還會從maker拿到資產，taker資產的結果只需紀錄最後一個就可以了
			if taker.Price.Cmp(maker.Price) > 0 {
				unfreezeQuote := taker.Price.Sub(maker.Price).Mul(matched)
				_, err := c.userAssetUseCase.Unfreeze(ctx, taker.UserID, c.quoteCurrencyID, unfreezeQuote)
				if err != nil {
					return nil, errors.Wrap(err, "unfreeze taker failed")
				}
			}
			// 將taker買單凍結的資產轉換給maker，金額是(maker price)*matched
			transferResultOne, err := c.userAssetUseCase.TransferFrozenToAvailable(ctx, taker.UserID, maker.UserID, c.quoteCurrencyID, maker.Price.Mul(matched))
			if err != nil {
				return nil, errors.Wrap(err, "transfer failed")
			}
			// 將轉換結果紀錄
			transferResult.UserAssets = append(transferResult.UserAssets, transferResultOne.UserAssets...)
			// 將maker賣單凍結的資產轉換給taker，數量是matched
			transferResultTwo, err := c.userAssetUseCase.TransferFrozenToAvailable(ctx, maker.UserID, taker.UserID, c.baseCurrencyID, matched)
			if err != nil {
				return nil, errors.Wrap(err, "transfer failed")
			}
			// 將轉換結果紀錄
			transferResult.UserAssets = append(transferResult.UserAssets, transferResultTwo.UserAssets...)
			// 如果maker買單數量減至0，則移除訂單
			if maker.UnfilledQuantity.IsZero() {
				if err := c.orderUseCase.RemoveOrder(ctx, maker.ID); err != nil {
					return nil, errors.Wrap(err, "remove maker order failed, maker order id: "+strconv.Itoa(maker.ID))
				}
			}
		}
		// 如果taker買單數量減至0，則移除訂單
		if taker.UnfilledQuantity.IsZero() {
			if err := c.orderUseCase.RemoveOrder(ctx, taker.ID); err != nil {
				return nil, errors.Wrap(err, "remove taker order failed")
			}
		}

```

最後的`TransferResult`我們將傳送給下游模組進行處理。