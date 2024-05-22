+++
title = 'Asset 資產模組'
date = 2024-03-27T00:00:00+08:00
tags = ['go', 'system-design']
+++

![https://raw.githubusercontent.com/superj80820/system-design/master/doc/asset.jpg](https://raw.githubusercontent.com/superj80820/system-design/master/doc/asset.jpg)

用戶資產除了基本的UserID、AssetID、Available(可用資產)，還需Frozen(凍結資產)欄位，以實現用戶下單時把下單資金凍結。

```go
type UserAsset struct {
	UserID    int
	AssetID   int
	Available decimal.Decimal
	Frozen    decimal.Decimal
}

```

一個交易對會有多個用戶，並且每個用戶都會有多個資產，如果資產都需存在memory當中，可以用兩個hash table來實作asset repository。

由於在撮合時也有機會透過API讀取memory的用戶資產，所以須用read-write-lock保護。

```go
type assetRepo struct {
	usersAssetsMap map[int]map[int]*domain.UserAsset
	lock           *sync.RWMutex

	// another code...
}

```

用戶資產的use case主要有轉帳`Transfer()`、凍結`Freeze()`、解凍`Unfreeze()`，另外我們還需liability user(負債帳戶)，他代表整個系統實際的資產，也可用來對帳，負債帳戶透過`LiabilityUserTransfer()`將資產從系統轉給用戶。

```go
type UserAssetUseCase interface {
	LiabilityUserTransfer(ctx context.Context, toUserID, assetID int, amount decimal.Decimal) (*TransferResult, error)

	TransferFrozenToAvailable(ctx context.Context, fromUserID, toUserID int, assetID int, amount decimal.Decimal) (*TransferResult, error)
	TransferAvailableToAvailable(ctx context.Context, fromUserID, toUserID int, assetID int, amount decimal.Decimal) (*TransferResult, error)
	Freeze(ctx context.Context, userID, assetID int, amount decimal.Decimal) (*TransferResult, error)
	Unfreeze(ctx context.Context, userID, assetID int, amount decimal.Decimal) (*TransferResult, error)

	// ..another functions
}

```

如果liability user id為1、用戶A user id為100、btc id為1、usdt id為2，其他用戶在系統已存入10btc與100000usdt。

此時用戶A對系統deposit儲值100000 usdt，並下了1顆btc價格為62000usdt的買單，此時需凍結62000usdt，將資產直接展開成一個二維表顯示如下:

| user id | asset id | available | frozen |
| --- | --- | --- | --- |
| 1 | 1 | -10 | 0 |
| 1 | 2 | -200000 | 0 |
| 100 | 1 | 0 | 0 |
| 100 | 2 | 38000 | 62000 |
| 其他用戶... |  |  |  |

在deposit時會呼叫`LiabilityUserTransfer()`，`assetRepo`呼叫`GetAssetWithInit()`，如果用戶資產存在則返回，不存在則初始化創建，資產模組在需要使用用戶資產時才創建他，不需先預載用戶資料表，也因為如此，在進入撮合系統前的auth非常重要，必須是認證過的用戶才能進入資產模組。

liability user呼叫`SubAssetAvailable()`減少資產，用戶呼叫`AddAssetAvailable()`獲取資產，資產的變動會呼叫`transferResult.addUserAsset()`添加至`transferResult`，最後回應給下游服務。

```go
func (u *userAsset) LiabilityUserTransfer(ctx context.Context, toUserID, assetID int, amount decimal.Decimal) (*domain.TransferResult, error) {
	if amount.LessThanOrEqual(decimal.Zero) {
		return nil, errors.New("can not operate less or equal zero amount")
	}

	transferResult := createTransferResult()

	liabilityUserAsset, err := u.assetRepo.GetAssetWithInit(domain.LiabilityUserID, assetID)
	if err != nil {
		return nil, errors.Wrap(err, "get asset with init failed")
	}
	toUserAsset, err := u.assetRepo.GetAssetWithInit(toUserID, assetID)
	if err != nil {
		return nil, errors.Wrap(err, "get asset with init failed")
	}

	u.assetRepo.SubAssetAvailable(liabilityUserAsset, amount)
	u.assetRepo.AddAssetAvailable(toUserAsset, amount)

	transferResult.addUserAsset(liabilityUserAsset)
	transferResult.addUserAsset(toUserAsset)

	return transferResult.TransferResult, nil
}

```

其他的use case都與`LiabilityUserTransfer()`類似，只是資產的轉移有差異，最後都會透過`transferResult`將變動的資產回應給下游服務。

在創建訂單時呼叫`Freeze()`，先檢查用戶是否有足夠資產`userAsset.Available.Cmp(amount) < 0`，如果足夠則呼叫`SubAssetAvailable()`減少資產，並呼叫`AddAssetFrozen()`增加凍結資產。

```go
func (u *userAsset) Freeze(ctx context.Context, userID, assetID int, amount decimal.Decimal) (*domain.TransferResult, error) {
	// another code...

	if userAsset.Available.Cmp(amount) < 0 {
		return nil, errors.Wrap(domain.LessAmountErr, "less amount err")
	}
	u.assetRepo.SubAssetAvailable(userAsset, amount)
	u.assetRepo.AddAssetFrozen(userAsset, amount)

	transferResult.addUserAsset(userAsset)

	return transferResult.TransferResult, nil
}

```

在撮合訂單時呼叫`TransferFrozenToAvailable()`，先檢查用戶(fromUser)是否有足夠的凍結資產後呼叫`SubAssetFrozen()`減少凍結用戶(fromUser)資產，並呼叫`AddAssetAvailable()`增加用戶(toUser)資產。

```go
func (u *userAsset) TransferFrozenToAvailable(ctx context.Context, fromUserID, toUserID, assetID int, amount decimal.Decimal) (*domain.TransferResult, error) {
	// another code...

	if fromUserAsset.Frozen.Cmp(amount) < 0 {
		return nil, errors.Wrap(domain.LessAmountErr, "less amount err")
	}
	u.assetRepo.SubAssetFrozen(fromUserAsset, amount)
	u.assetRepo.AddAssetAvailable(toUserAsset, amount)

	transferResult.addUserAsset(fromUserAsset)
	transferResult.addUserAsset(toUserAsset)

	return transferResult.TransferResult, nil
}

```

在用戶轉帳時呼叫`TransferAvailableToAvailable()`，檢查用戶(fromUser)是否有足夠資產後呼叫`SubAssetAvailable()`減少資產並呼叫`AddAssetAvailable()`增加用戶(toUser)資產。

```go
func (u *userAsset) TransferAvailableToAvailable(ctx context.Context, fromUserID, toUserID, assetID int, amount decimal.Decimal) (*domain.TransferResult, error) {
	// another code...

	if fromUserAsset.Available.Cmp(amount) < 0 {
		return nil, errors.Wrap(domain.LessAmountErr, "less amount err")
	}
	u.assetRepo.SubAssetAvailable(fromUserAsset, amount)
	u.assetRepo.AddAssetAvailable(toUserAsset, amount)

	transferResult.addUserAsset(fromUserAsset)
	transferResult.addUserAsset(toUserAsset)

	return transferResult.TransferResult, nil
}

```