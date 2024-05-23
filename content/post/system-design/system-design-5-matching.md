+++
title = 'Matching 撮合模組'
date = 2024-03-29T00:00:00+08:00
tags = ['go', 'system-design']
+++

![https://raw.githubusercontent.com/superj80820/system-design/master/doc/matching.jpg](https://raw.githubusercontent.com/superj80820/system-design/master/doc/matching.jpg)

source code:
* [usecase](https://github.com/superj80820/system-design/tree/master/exchange/usecase/matching)
* [repository](https://github.com/superj80820/system-design/tree/master/exchange/repository/matching)

---

撮合模組是整個交易系統最核心的部分。

主要核心是維護一個買單book一個賣單book

訂單簿必須是「價格優先、序列號次要」，直觀上會把這兩個book當成兩個list來看

![https://raw.githubusercontent.com/superj80820/system-design/master/doc/order-book.jpg](https://raw.githubusercontent.com/superj80820/system-design/master/doc/order-book.jpg)

- 賣單簿: 4.45, 4.52, 4.63, 4.64, 4.65
- 買單簿: 4.15, 4.08, 4.06, 4.05, 4.01

但如果就把order-book用2個double-link-list來設計，這樣效能並不好，以查詢、插入、刪除來看:

- 查詢: O(n)
- 插入: O(n)，雖然插入是O(1)，但要先找到適合的價格插入，也要耗費O(n)
- 刪除: O(n)，雖然刪除是O(1)，但要先查詢到價格再刪除，也要耗費O(n)

如果搭配hash-table，查詢跟刪除是可以達到O(1)，但插入還是O(n)，有沒有更適合的資料結構呢？我們可用平衡二元搜尋樹，可考慮red-black-tree(以下簡稱rbtree)或AVL-tree，rb-tree沒有AVL-tree來的平衡，但插入與刪除時為了平衡的旋轉較少，兩者可簡化判斷要用在哪些場景:

- rbtree: 插入與刪除較多
- AVL-tree: 查詢較多

order-book會有大量訂單插入與撮合刪除場景，會較適合選擇rbtree，這樣插入速度就變快了，如下:

- 查詢: O(logn)
- 插入: O(logn)
- 刪除: O(logn)

但這樣其他操作的速度就變慢了，有沒有方法可以更加優化呢？

我們可以rbtree+double-link-list(以下簡稱list)+hash-table，rbtree的node儲存的不是一個order而是存有相同價格的order list，而hash-table有兩個，一個儲存list在tree上的位置，一個儲存order在list上的位置。
如此一來在第1次插入此價格或從訂單簿移除整個價格時速度為O(logn)，但在建立node後，後續的插入與刪除實際上是對list的操作，由於整個撮合模組是依照序列號有序輸入order的，所以只需直接對list的尾端insert order即可，速度為O(1)。

所以統整速度可以得到以下:

- 查詢: O(1)
- 第1次插入價格: O(logn)
- 移除整個訂單簿價格: O(logn)
- 插入: O(1)，透過hash-table插入list上的order
- 刪除: O(1)，透過hash-table刪除list上的order

所以我們的order-book一側定義為`rbtree<priceLevelEntity.price, priceLevelEntity>`的結構體，`price`為compare的key，`priceLevelEntity`為實際的value。

```go
type bookEntity struct {
	direction   domain.DirectionEnum
	orderLevels *rbtree.Tree // <priceLevelEntity.price, priceLevelEntity>
	bestPrice   *rbtree.Node
}

func createBook(direction domain.DirectionEnum) *bookEntity {
	return &bookEntity{
		direction:   direction,
		orderLevels: rbtree.NewWith(directionEnum(direction).compare),
	}
}
```

`priceLevelEntity`存放此price所有的order list，可以再設一個`totalUnfilledQuantity`，在存放order時也紀錄數量。

```go
type priceLevelEntity struct {
	price                 decimal.Decimal
	totalUnfilledQuantity decimal.Decimal
	orders                *list.List
}
```

買單簿與賣單簿差異只有rbtree排序的不同，依照不同的方向`directionEnum`決定compare的方式，買單簿`domain.DirectionBuy`以`高價在前`，賣單簿`domain.DirectionSell`以`低價在前`。

```go
type directionEnum domain.DirectionEnum

func (d directionEnum) compare(a, b interface{}) int {
	aPrice := a.(decimal.Decimal)
	bPrice := b.(decimal.Decimal)

	switch domain.DirectionEnum(d) {
	case domain.DirectionSell:
		// low price is first
		return aPrice.Cmp(bPrice)
	case domain.DirectionBuy:
		// hight price is first
		return bPrice.Cmp(aPrice)
	case domain.DirectionUnknown:
		panic("unknown direction")
	default:
		panic("unknown direction")
	}
}
```

將不同方向的`bookEntity`定義為`sellBook`、`buyBook`，`orderMap`紀錄order在list的位置以便刪除可以用O(1)的速度，`priceLevelMap`紀錄`priceLevelEntity` Node在樹的位置以便取用可以用O(1)的速度。這樣就有一個order-book了

```go
type orderBookRepo struct {
	sequenceID    int
	sellBook      *bookEntity
	buyBook       *bookEntity
	orderMap      map[int]*list.Element
	priceLevelMap map[string]*rbtree.Node

	// another code...
}

func CreateOrderBookRepo(cache *redisKit.Cache, orderBookMQTopic, l1OrderBookMQTopic, l2OrderBookMQTopic, l3OrderBookMQTopic mq.MQTopic) domain.MatchingOrderBookRepo {
	return &orderBookRepo{
		sellBook:      createBook(domain.DirectionSell),
		buyBook:       createBook(domain.DirectionBuy),
		orderMap:      make(map[int]*list.Element),
		priceLevelMap: make(map[string]*rbtree.Node),

		// another code...
	}
}
```

撮合過程是對order-book的一系列操作，有四個重要的methods，以下操作會以括號說明time complexity:

- `GetOrderBookFirst`: 取得特定方向的最佳價格，是為rbtree的最小值，order list的第一個
- `AddOrderBookOrder`: 加入特定方向order，如果rbtree不存在此價格則插入至price level(O(logn))，如果已存在則用`priceLevelMap`直接插入至price level(O(1))
- `RemoveOrderBookOrder`: 刪除特定方向order，用`orderMap`直接刪除order list裡的order(O(1))，用`priceLevelMap`檢查order list是否為0，如果為0則刪除rbtree這價格的Node(O(logn))
- `MatchOrder`: 用`orderMap`更新order的數量與狀態(O(1))

在不須新增或刪除整個價格時，不會對rbtree操作，只操作hash-table與list，這讓速度為O(1)，是一個需注意的細節。

```go
type MatchingOrderBookRepo interface {
	GetOrderBookFirst(direction DirectionEnum) (*OrderEntity, error)
	AddOrderBookOrder(direction DirectionEnum, order *OrderEntity) error
	RemoveOrderBookOrder(direction DirectionEnum, order *OrderEntity) error
	MatchOrder(orderID int, matchedQuantity decimal.Decimal, orderStatus OrderStatusEnum, updatedAt time.Time) error

	// another code...
}
```

簡單來說，限價單撮合過程就是taker不斷與對手盤的最佳價格撮合。我直接貼上`NewOrder`的code，透過註解解釋:

```go
func (m *matchingUseCase) NewOrder(ctx context.Context, takerOrder *domain.OrderEntity) (*domain.MatchResult, error) {
	// 如果taker是賣單，maker對手盤的為買單簿
	// 如果taker是買單，maker對手盤的為賣單簿
	var makerDirection, takerDirection domain.DirectionEnum
	switch takerOrder.Direction {
	case domain.DirectionSell:
		makerDirection = domain.DirectionBuy
		takerDirection = domain.DirectionSell
	case domain.DirectionBuy:
		makerDirection = domain.DirectionSell
		takerDirection = domain.DirectionBuy
	default:
		return nil, errors.New("not define direction")
	}

	// 設置此次撮合的sequence id
	m.matchingOrderBookRepo.SetSequenceID(takerOrder.SequenceID)
	// 紀錄此次撮合的成交的result
	matchResult := createMatchResult(takerOrder)

	// taker不斷與對手盤的最佳價格撮合直到無法撮合
	for {
		// 取得對手盤的最佳價格
		makerOrder, err := m.matchingOrderBookRepo.GetOrderBookFirst(makerDirection)
		// 如果沒有最佳價格則退出撮合
		if errors.Is(err, domain.ErrEmptyOrderBook) {
			break
		} else if err != nil {
			return nil, errors.Wrap(err, "get first order book order failed")
		}

		// 如果賣單低於對手盤最佳價格則退出撮合
		if takerOrder.Direction == domain.DirectionSell && takerOrder.Price.Cmp(makerOrder.Price) > 0 {
			break
			// 如果買單高於對手盤最佳價格則退出撮合
		} else if takerOrder.Direction == domain.DirectionBuy && takerOrder.Price.Cmp(makerOrder.Price) < 0 {
			break
		}

		// 撮合成功，設置撮合價格為對手盤的最佳價格
		m.matchingOrderBookRepo.SetMarketPrice(makerOrder.Price)
		// 撮合數量為taker或maker兩者的最小數量
		matchedQuantity := min(takerOrder.UnfilledQuantity, makerOrder.UnfilledQuantity)
		// 新增撮合紀錄
		addForMatchResult(matchResult, makerOrder.Price, matchedQuantity, makerOrder)
		// taker的數量減去撮合數量
		takerOrder.UnfilledQuantity = takerOrder.UnfilledQuantity.Sub(matchedQuantity)
		// maker的數量減去撮合數量
		makerOrder.UnfilledQuantity = makerOrder.UnfilledQuantity.Sub(matchedQuantity)
		// 如果maker數量減至0，代表已完全成交(Fully Filled)，更新maker order並從order-book移除
		if makerOrder.UnfilledQuantity.Equal(decimal.Zero) {
			// 更新maker order
			makerOrder.Status = domain.OrderStatusFullyFilled
			if err := m.matchingOrderBookRepo.MatchOrder(makerOrder.ID, matchedQuantity, domain.OrderStatusFullyFilled, takerOrder.CreatedAt); err != nil {
				return nil, errors.Wrap(err, "update order failed")
			}
			// 從order-book移除
			if err := m.matchingOrderBookRepo.RemoveOrderBookOrder(makerDirection, makerOrder); err != nil {
				return nil, errors.Wrap(err, "remove order book order failed")
			}
			// 如果maker數量不為零，代表已部分成交(Partial Filled)，更新maker order
		} else {
			makerOrder.Status = domain.OrderStatusPartialFilled
			if err := m.matchingOrderBookRepo.MatchOrder(makerOrder.ID, matchedQuantity, domain.OrderStatusPartialFilled, takerOrder.CreatedAt); err != nil {
				return nil, errors.Wrap(err, "update order failed")
			}
		}
		// 如果taker數量減至0，則完全成交，退出撮合
		if takerOrder.UnfilledQuantity.Equal(decimal.Zero) {
			takerOrder.Status = domain.OrderStatusFullyFilled
			break
		}
	}
	// 如果taker數量不為0，檢查原始數量與剩餘數量來設置status
	if takerOrder.UnfilledQuantity.GreaterThan(decimal.Zero) {
		// 預設status為等待成交(Pending)
		status := domain.OrderStatusPending
		// 如果原始數量與剩餘數量不等，status為部分成交(Partial Filled)
		if takerOrder.UnfilledQuantity.Cmp(takerOrder.Quantity) != 0 {
			status = domain.OrderStatusPartialFilled
		}
		takerOrder.Status = status
		// 新增order至taker方向的order book
		m.matchingOrderBookRepo.AddOrderBookOrder(takerDirection, takerOrder)
	}

	// 設置order-book為已改變
	m.isOrderBookChanged.Store(true)

	return matchResult, nil
}
```

最後的`MatchResult`我們將傳送給下游模組進行處理。