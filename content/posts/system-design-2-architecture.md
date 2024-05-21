+++
title = '如何設計一個撮合系統'
date = 2024-03-26T00:00:00+08:00
tags = ['go', 'system-design']
+++

![https://raw.githubusercontent.com/superj80820/system-design/master/doc/exchange-arch.png](https://raw.githubusercontent.com/superj80820/system-design/master/doc/exchange-arch.png)

撮合系統主要由Sequence(定序模組)、Asset(資產模組)、Order(訂單模組)、Matching(撮合模組)、Clearing(清算模組)組成。

以Create Order Event(創建訂單)來舉例:

1. [Sequence 定序模組](Sequence%20%E5%AE%9A%E5%BA%8F%E6%A8%A1%E7%B5%84%20db000a8db63c401db03d9858c7ebb926.md)
    1. 大量併發的訂單請求進入服務
    2. 定序模組會將訂單以有序的方式儲存
2. [Asset 資產模組](Asset%20%E8%B3%87%E7%94%A2%E6%A8%A1%E7%B5%84%20637b5517f13c4b85bdebca20e82a5340.md)
    1. 資產模組凍結訂單所需資產
3. [Order 訂單模組](Order%20%E8%A8%82%E5%96%AE%E6%A8%A1%E7%B5%84%20239433f497b346eb9ce31b0ed99bfba0.md)
    1. 訂單模組產生訂單
4. [Matching 撮合模組](Matching%20%E6%92%AE%E5%90%88%E6%A8%A1%E7%B5%84%20dfcf9bcba7e44c1d88d611bbc8003875.md)
    1. 撮合模組獲取訂單進行撮合，更新order book後產生match result
5. [Clearing 清算模組](Clearing%20%E6%B8%85%E7%AE%97%E6%A8%A1%E7%B5%84%205503c74dc3594ba391e9d9c8a33d8058.md)
    1. 清算模組依照match result來transfer、unfreeze資產
6. [整合交易系統](%E6%95%B4%E5%90%88%E6%92%AE%E5%90%88%E7%B3%BB%E7%B5%B1%2017370932a5bc40eeafcb67bbb3aea0d5.md)
    1. 各模組產生的result組成trading result，produce給下游服務，下游服務透過這些資料cache與persistent data來達到eventual consistency

由於需要快速計算撮合內容，計算都會直接在memory完成，過程中不會persistent data，但如果撮合系統崩潰，memory資料都會遺失，所以才需定序模組將訂單event都儲存好，再進入撮合系統，如此一來，如果系統崩潰也可以靠已儲存的event來recover撮合系統。

下游服務都須考慮idempotency冪等性，可透過sequence id來判斷event是否重複或超前，如果重複就拋棄，超前就必須讀取先前的event。

### 參考

感謝以下參考資源，讓我完成此系統，如有錯誤，敬請賜教，謝謝。

- [gitbitex](https://github.com/gitbitex)
- [廖雪峰-设计交易引擎](https://www.liaoxuefeng.com/wiki/1252599548343744/1491662232616993)
- [Linux 核心的紅黑樹](https://hackmd.io/@sysprog/linux-rbtree)
- [How to Build a Fast Limit Order Book](https://web.archive.org/web/20110219163448/http://howtohft.wordpress.com/2011/02/15/how-to-build-a-fast-limit-order-book/)
- [用 Go 語言實現固定大小的 Ring Buffer 資料結構](https://blog.wu-boy.com/2023/01/ring-buffer-queue-with-fixed-size-in-golang/)