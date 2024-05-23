---
title: 'System Design 系列'
date: 2024-03-24T00:00:00+08:00
tags:
    - 'go'
    - 'system-design'
menu:
    main:
        name: System Design 系列
        weight: 10
        params:
            icon: archives
image: 'cover.png'
---

[source code](https://github.com/superj80820/system-design)

---

## Clean Architecture

![https://i.imgur.com/c3we5K6.png](https://i.imgur.com/c3we5K6.png)

軟體世界雖然沒有silver bullet，但我們還是期待有高擴展性、測試性、維護性的系統，我將系統分為repository、usecase、delivery三層，並依照domain interface進行彼此的相依，以滿足大多情境的需求

### 教學

* [你的 Backend 可以更有彈性一點 - Clean Architecture 概念篇](https://blog.messfar.com/post/k8s-note/k8s-note-clean-architecture-part1)
* [奔放的 Golang，卻隱藏著有紀律的架構！ - Clean Architecture 實作篇](https://blog.messfar.com/post/k8s-note/k8s-note-clean-architecture-part2)
* [讓你的 Backend 萬物皆虛，萬事皆可測 - Clean Architecture 測試篇](https://blog.messfar.com/post/k8s-note/k8s-note-clean-architecture-part3)

## Matching System

![https://raw.githubusercontent.com/superj80820/system-design/master/doc/exchange-arch.png](https://raw.githubusercontent.com/superj80820/system-design/master/doc/exchange-arch.png)

source code:
* [backend](https://github.com/superj80820/system-design/tree/master/exchange)
* [frontend-gitbitex-web(open source)](https://github.com/gitbitex/gitbitex-web)

---

* 預覽網頁(❗僅用最低效能運行預覽，不是 production 運作規格): [https://preview.exchange.messfar.com](https://preview.exchange.messfar.com/)
* 可達到 100,000PRS。撮合引擎以記憶體計算
* 可回放事件。以 event sourcing 的方式實現，撮合引擎為讀取 event 的有限狀態機，可 warm backup 多台 server 聽取 event，來達到 high availability
* 可分散式。不同的domain可部署至不同機器

### 壓測

![https://i.imgur.com/V7KFvvC.png](https://i.imgur.com/V7KFvvC.png)

單機啟動 server、mysql、kafka、redis、mongodb，並進行買賣單搓合，並以k6壓測:
* exchange 機器: EC2 c5.18xlarge
* k6 機器: EC2 m5.8xlarge
* RPS (max): 102,988.52

如果將 mysql 或 kafka 等服務獨立出來，理論上可用更便宜的機器

### 教學

1. [如何設計一個撮合系統](https://blog.messfar.com/post/system-design/system-design-1-architecture)
2. [Sequence 定序模組](https://blog.messfar.com/post/system-design/system-design-2-sequence)
3. [Asset 資產模組](https://blog.messfar.com/post/system-design/system-design-3-asset)
4. [Order 訂單模組](https://blog.messfar.com/post/system-design/system-design-4-order)
5. [Matching 撮合模組](https://blog.messfar.com/post/system-design/system-design-5-matching)
6. [Clearing 清算模組](https://blog.messfar.com/post/system-design/system-design-6-clearing)
7. [整合撮合系統](https://blog.messfar.com/post/system-design/system-design-7-integration)