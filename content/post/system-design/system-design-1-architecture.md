+++
title = 'Matching System Part1: 如何設計一個撮合系統'
date = 2024-03-25T00:00:00+08:00
tags = ['go', 'system-design']
+++

![https://raw.githubusercontent.com/superj80820/system-design/master/doc/exchange-arch.png](https://raw.githubusercontent.com/superj80820/system-design/master/doc/exchange-arch.png)

撮合系統主要由Sequence(定序模組)、Asset(資產模組)、Order(訂單模組)、Matching(撮合模組)、Clearing(清算模組)組成。

以Create Order Event(創建訂單)來舉例:

1. [Sequence 定序模組](https://blog.messfar.com/post/system-design/system-design-2-sequence)
    1. 大量併發的訂單請求進入服務
    2. 定序模組會將訂單以有序的方式儲存
2. [Asset 資產模組](https://blog.messfar.com/post/system-design/system-design-3-asset)
    1. 資產模組凍結訂單所需資產
3. [Order 訂單模組](https://blog.messfar.com/post/system-design/system-design-4-order)
    1. 訂單模組產生訂單
4. [Matching 撮合模組](https://blog.messfar.com/post/system-design/system-design-5-matching)
    1. 撮合模組獲取訂單進行撮合，更新order book後產生match result
5. [Clearing 清算模組](https://blog.messfar.com/post/system-design/system-design-6-clearing)
    1. 清算模組依照match result來transfer、unfreeze資產
6. [整合交易系統](https://blog.messfar.com/post/system-design/system-design-7-integration)
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