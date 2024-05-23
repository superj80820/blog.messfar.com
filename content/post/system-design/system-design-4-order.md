+++
title = 'Matching System Part4: Order 訂單模組'
date = 2024-03-28T00:00:00+08:00
tags = ['go', 'system-design']
+++

![https://raw.githubusercontent.com/superj80820/system-design/master/doc/order.jpg](https://raw.githubusercontent.com/superj80820/system-design/master/doc/order.jpg)

source code:
* [usecase](https://github.com/superj80820/system-design/tree/master/exchange/usecase/order)
* [repository](https://github.com/superj80820/system-design/tree/master/exchange/repository/order)

---

活動訂單需由訂單模組管理，可以用hash table以`orderID: order`儲存所有活動訂單，在需要以用戶ID取得相關活動訂單時，可用兩層hash table以`userID: orderID: order`取得訂單。

由於在撮合時也有機會透過API讀取memory的活動訂單，所以須用read-write-lock保護。

```go
type orderUseCase struct {
	lock          *sync.RWMutex
	activeOrders  map[int]*domain.OrderEntity
	userOrdersMap map[int]map[int]*domain.OrderEntity

	// another code...
}

```

活動訂單的欄位介紹如下:

```go
type OrderEntity struct {
	ID         int // 訂單ID
	SequenceID int // 訂單由定序模組所定的ID
	UserID     int // 用戶ID

	Price     decimal.Decimal // 價格
	Direction DirectionEnum   // 買單還是賣單

	// 狀態:
	// 完全成交(Fully Filled)、
	// 部分成交(Partial Filled)、
	// 等待成交(Pending)、
	// 完全取消(Fully Canceled)、
	// 部分取消(Partial Canceled)
	Status OrderStatusEnum

	Quantity         decimal.Decimal // 數量
	UnfilledQuantity decimal.Decimal // 未成交數量

	CreatedAt time.Time // 創建時間
	UpdatedAt time.Time // 更新時間
}

```

訂單模組主要的use case有創建訂單`CreateOrder()`、刪除訂單`RemoveOrder()`、取得訂單`GetUserOrders()`，由於在創建時需要凍結資產、刪除時需要解凍資產，所以須注入資產模組`UserAssetUseCase`。

```go
type OrderUseCase interface {
	CreateOrder(ctx context.Context, sequenceID int, orderID, userID int, direction DirectionEnum, price, quantity decimal.Decimal, ts time.Time) (*OrderEntity, *TransferResult, error)
	RemoveOrder(ctx context.Context, orderID int) error
	GetUserOrders(userID int) (map[int]*OrderEntity, error)

	// another code...
}

```

資產模組在此處的命名為`o.assetUseCase`。

在創建訂單時呼叫`CreateOrder()`，需先判斷買賣單凍結用戶資產`o.assetUseCase.Freeze()`，之後創建`order`存入`o.activeOrders`與`o.userOrdersMap`，由於過程中有資產的變化，所以除了`order`以外，也需將`transferResult`回應給下游服務。

```go
func (o *orderUseCase) CreateOrder(ctx context.Context, sequenceID int, orderID int, userID int, direction domain.DirectionEnum, price decimal.Decimal, quantity decimal.Decimal, ts time.Time) (*domain.OrderEntity, *domain.TransferResult, error) {
	var err error
	transferResult := new(domain.TransferResult)

	switch direction {
	case domain.DirectionSell:
		transferResult, err = o.assetUseCase.Freeze(ctx, userID, o.baseCurrencyID, quantity)
		if err != nil {
			return nil, nil, errors.Wrap(err, "freeze base currency failed")
		}
	case domain.DirectionBuy:
		transferResult, err = o.assetUseCase.Freeze(ctx, userID, o.quoteCurrencyID, price.Mul(quantity))
		if err != nil {
			return nil, nil, errors.Wrap(err, "freeze base currency failed")
		}
	default:
		return nil, nil, errors.New("unknown direction")
	}

	order := domain.OrderEntity{
		ID:               orderID,
		SequenceID:       sequenceID,
		UserID:           userID,
		Direction:        direction,
		Price:            price,
		Quantity:         quantity,
		UnfilledQuantity: quantity,
		Status:           domain.OrderStatusPending,
		CreatedAt:        ts,
		UpdatedAt:        ts,
	}

	o.lock.Lock()
	o.activeOrders[orderID] = &order
	if _, ok := o.userOrdersMap[userID]; !ok {
		o.userOrdersMap[userID] = make(map[int]*domain.OrderEntity)
	}
	o.userOrdersMap[userID][orderID] = &order
	o.lock.Unlock()

	return &order, transferResult, nil
}

```

在撮合訂單完全成交、取消訂單時會呼叫`RemoveOrder()`，`o.activeOrders`刪除訂單，而`o.userOrdersMap`透過訂單的userID查找也刪除訂單。

```go
func (o *orderUseCase) RemoveOrder(ctx context.Context, orderID int) error {
	o.lock.Lock()
	defer o.lock.Unlock()

	removedOrder, ok := o.activeOrders[orderID]
	if !ok {
		return errors.New("order not found in active orders")
	}
	delete(o.activeOrders, orderID)

	userOrders, ok := o.userOrdersMap[removedOrder.UserID]
	if !ok {
		return errors.New("user orders not found")
	}
	_, ok = userOrders[orderID]
	if !ok {
		return errors.New("order not found in user orders")
	}
	delete(userOrders, orderID)

	return nil
}

```

API取得訂單會呼叫`GetUserOrders()`，為了避免race-condition，查找到訂單後需將訂單clone再回傳。

```go
func (o *orderUseCase) GetUserOrders(userId int) (map[int]*domain.OrderEntity, error) {
	o.lock.RLock()
	defer o.lock.RUnlock()

	userOrders, ok := o.userOrdersMap[userId]
	if !ok {
		return nil, errors.New("get user orders failed")
	}
	userOrdersClone := make(map[int]*domain.OrderEntity, len(userOrders))
	for orderID, order := range userOrders {
		userOrdersClone[orderID] = order.Clone()
	}
	return userOrdersClone, nil
}

```