+++
title = '隨筆: 學習地圖'
date = 2024-01-02T00:00:00+08:00
+++

## 程式語言

- JavaScript & Node.js
- Rust
    - [rust trait(特徵)跟java interface(介面)的差異](Rust%20%E9%9A%A8%E7%AD%86%20e69468ae63bd4546a6b76a5f908a65b5/rust%20trait(%E7%89%B9%E5%BE%B5)%E8%B7%9Fjava%20interface(%E4%BB%8B%E9%9D%A2)%E7%9A%84%E5%B7%AE%E7%95%B0%2031ce661b33554ee18c978eaac795e6eb.md)

## 設計模式

### Hey! Go Design Patterns

- DAY 1：Hey! Go Design Patterns: [https://ithelp.ithome.com.tw/articles/10260929](https://ithelp.ithome.com.tw/articles/10260929)

## 數據結構與演算法

- SortSet
- PriorityQueue
- BuildHeap
- [minimum-waiting-time](Algo%20%E9%9A%A8%E7%AD%86%2086f57ecd943b4daf81a556fa59191f0e/algoexpert-minimum-waiting-time%2033a9bc6a5925436ab92d22beba5b2bbf.md)
- [class-photos](Algo%20%E9%9A%A8%E7%AD%86%2086f57ecd943b4daf81a556fa59191f0e/algoexpert-class-photos%202c7196856ba04789a9b82a4af2f87c55.md)
- Tower of Hanoi
- one's complement, two's complement
- [https://noob.tw/complements/](https://noob.tw/complements/)
- one's complement: 正與負範圍的絕對值相同，會產生-0的現象
- two's complement: 正與負範圍的絕對值不同，正從0開始，負從-1開始，這也使得負數與正數最大的絕對值不同，負數會比正數大1 (e.g.  -128 ~ -1 ~ +0 ~ +127)

### 刷題

- grind169
- neetcode
- algoexpert
- Golang Leetcode 刷題筆記: [https://mp.weixin.qq.com/s/EaDLaLy3YjrNiSoNofwMMA](https://mp.weixin.qq.com/s/EaDLaLy3YjrNiSoNofwMMA)
- [283. Move Zeros](Algo%20%E9%9A%A8%E7%AD%86%2086f57ecd943b4daf81a556fa59191f0e/0283-move%20zeros%207d51da3409874d968872f8b3cf8b139c.md)

### 資料結構

- Array與List的差異: [http://sharecoder.blogspot.com/2012/10/arraylist.html](http://sharecoder.blogspot.com/2012/10/arraylist.html)
- B+樹跟紅黑樹哪個搜尋速度更快？:
    
    ```go
    Take this with a pinch of salt:
    
    // 如果在硬碟這種慢速IO的載體上，建議使用B樹
    B-tree when you're managing more than thousands of items and you're paging them from a disk or some slow storage medium.
    // 如果插入、刪除、檢索相當頻繁，建議使用紅黑樹
    RB tree when you're doing fairly frequent inserts, deletes and retrievals on the tree.
    // 如果插入、刪除相對於檢索不頻繁時，建議使用AVL樹
    AVL tree when your inserts and deletes are infrequent relative to your retrievals.
    ```
    
    - [https://stackoverflow.com/questions/1589556/when-to-choose-rb-tree-b-tree-or-avl-tree](https://stackoverflow.com/questions/1589556/when-to-choose-rb-tree-b-tree-or-avl-tree)
    - [https://stackoverflow.com/questions/647537/b-tree-faster-than-avl-or-redblack-tree](https://stackoverflow.com/questions/647537/b-tree-faster-than-avl-or-redblack-tree)
    - [https://luffy997.github.io/2021/07/29/二叉搜索树、平衡二叉树、红黑树、B-树性能对比/](https://luffy997.github.io/2021/07/29/%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91%E3%80%81%E5%B9%B3%E8%A1%A1%E4%BA%8C%E5%8F%89%E6%A0%91%E3%80%81%E7%BA%A2%E9%BB%91%E6%A0%91%E3%80%81B-%E6%A0%91%E6%80%A7%E8%83%BD%E5%AF%B9%E6%AF%94/)
- 用漫畫圖片理解紅黑樹: [https://mp.weixin.qq.com/s/0RKuO0Pk7R09wGzgyA43mw](https://mp.weixin.qq.com/s/0RKuO0Pk7R09wGzgyA43mw)
- 線上動畫解釋tree的新增、查找、刪除: [https://www.cs.usfca.edu/~galles/visualization/Algorithms.html](https://www.cs.usfca.edu/~galles/visualization/Algorithms.html)

## 資料儲存

### 資料庫儲存密碼

1. base64: 完全安全，可以直接decode
2. encrypt: 不安全，如果取得私鑰可解密
3. hash: 不安全，可能會被彩虹表攻擊
4. salt+hash: 大致安全，可能會被超級電腦攻擊，但攻擊成本高
5. bcrypt: 安全，可增加超級電腦攻擊的成本，使攻擊成本過高導致無法攻擊

### MySQL 資料破碎

- 長度可變資料欄位(varchar 或 nvarchar)導致資料表破碎嚴重: [https://dotblogs.azurewebsites.net/rockchang/2017/05/08/110423](https://dotblogs.azurewebsites.net/rockchang/2017/05/08/110423)

### MySQL Lock

- MySQL Deadlock案例: [https://github.com/aneasystone/mysql-deadlocks](https://github.com/aneasystone/mysql-deadlocks)
- Single Query Select & Update: [https://khiav223577.github.io/blog/2019/02/07/不使用-lock-又要避免-race-condition，可能嗎？/](https://khiav223577.github.io/blog/2019/02/07/%E4%B8%8D%E4%BD%BF%E7%94%A8-lock-%E5%8F%88%E8%A6%81%E9%81%BF%E5%85%8D-race-condition%EF%BC%8C%E5%8F%AF%E8%83%BD%E5%97%8E%EF%BC%9F/)

### MySQL 慢查詢

- 經典案例: [https://heapdump.cn/article/5040142](https://heapdump.cn/article/5040142)

### MySQL SQL 指令

- JOIN & UNION 圖解: [https://sam.webspace.tw/2019/07/11/SQL-JOIN/](https://sam.webspace.tw/2019/07/11/SQL-JOIN/)

### MySQL 索引

- 用對這些場景下的數據庫索引，領導說我有點東西: [https://mp.weixin.qq.com/s/4K3borSZXt-yc5t5UJnJpQ](https://mp.weixin.qq.com/s/4K3borSZXt-yc5t5UJnJpQ)
- 如何優化mysql的範圍查詢: [https://cloud.tencent.com/developer/article/1767984](https://cloud.tencent.com/developer/article/1767984)
- 了解MySQL索引的BST、Balanced BST、B樹、B+樹底層: [https://mark-lin.com/posts/20190911/](https://mark-lin.com/posts/20190911/)
- 為什麼MySQL的Innodb使用B+樹而不是紅黑樹作為索引: [https://mp.weixin.qq.com/s?__biz=MzIwNDAyOTI2Nw==&mid=2247483893&idx=1&sn=6f894a5cc2676073e50bacbdee5e6478&chksm=96c72dc9a1b0a4dfa95600ebf5ad7db9d804afbad6c7396078ab7f85d10c7516d602715172db&token=2074428159&lang=zh_CN#rd](https://mp.weixin.qq.com/s?__biz=MzIwNDAyOTI2Nw==&mid=2247483893&idx=1&sn=6f894a5cc2676073e50bacbdee5e6478&chksm=96c72dc9a1b0a4dfa95600ebf5ad7db9d804afbad6c7396078ab7f85d10c7516d602715172db&token=2074428159&lang=zh_CN#rd)

### SQL 理論
  * https://www.brentozar.com/training/think-like-sql-server-engine/
  * Dcard資深工程師寫的，很細膩:
    * https://blog.kennycoder.io/2021/08/08/Postgres-%E6%B7%B1%E5%85%A5%E6%8E%A2%E8%A8%8Eindex-engine-%E7%9A%84%E8%A1%8C%E7%82%BA/
    * https://blog.kennycoder.io/2023/11/18/%E8%AB%87%E8%AB%87-Postgres-%E8%88%87-MySQL-%E7%9A%84%E5%B7%AE%E7%95%B0/
    * https://blog.kennycoder.io/2021/07/24/Postgres-zero-downtime-migration-%E8%A9%B2%E6%B3%A8%E6%84%8F%E7%9A%84%E7%B4%B0%E7%AF%80/
### SQL 練習
   * 基本語法:
     * https://sqlbolt.com/
     * https://www.w3resource.com/mysql-exercises/
   * 情境練習:
     * https://leetcode.com/studyplan/top-sql-50/
     * https://platform.stratascratch.com/coding
     * https://datalemur.com/
     * https://www.hackerrank.com/domains/sql
     * https://sqlpad.io/
     * https://www.interviewquery.com/

### Redis

- 一文講透 Redis 事務 （事務模式 VS Lua 腳本）: [https://www.cnblogs.com/makemylife/p/17299566.html](https://www.cnblogs.com/makemylife/p/17299566.html)
- 先更新數據庫還是先更新緩存？: [https://mp.weixin.qq.com/s/SPgtpfgv6bz2AfPa1CYYeQ](https://mp.weixin.qq.com/s/SPgtpfgv6bz2AfPa1CYYeQ)
- Lua中有寫操作不能使用帶隨機性質的讀操作，如TIME，需用redis.replicate_commands()解決: [https://www.cnblogs.com/zhaoyongjie-z/p/14313046.html](https://www.cnblogs.com/zhaoyongjie-z/p/14313046.html)
- Redis的五種數據結構: [https://pdai.tech/md/db/nosql-redis/db-redis-data-types.html](https://pdai.tech/md/db/nosql-redis/db-redis-data-types.html)
- 為啥 redis 使用跳表(skiplist)而不是使用 red-black？: [https://www.zhihu.com/question/20202931](https://www.zhihu.com/question/20202931)

### OLAP

- ClickHouse
  - 介紹: [ClickHouse:時序資料庫建置與運行](https://ithelp.ithome.com.tw/users/20103975/ironman/5207)

### TSDB

- influxdb

---

## 分布式服務

### OutBox Pattern

- [https://microservices.io/patterns/data/transactional-outbox.html](https://microservices.io/patterns/data/transactional-outbox.html)
- [https://medium.com/skyler-record/微服務架的資料一致性-2-outbox-pattern-891512620453](https://medium.com/skyler-record/%E5%BE%AE%E6%9C%8D%E5%8B%99%E6%9E%B6%E7%9A%84%E8%B3%87%E6%96%99%E4%B8%80%E8%87%B4%E6%80%A7-2-outbox-pattern-891512620453)

### Docker

- Docker從入門到干活，看這一篇足矣: [https://mp.weixin.qq.com/s/YlcvlUQ-xkz25PuYkeEQqw](https://mp.weixin.qq.com/s/YlcvlUQ-xkz25PuYkeEQqw)

### 分佈式事務

- 什麼是分佈式事務: [https://mp.weixin.qq.com/s/_56jq_p_nDUiBwaI2MTlmA](https://mp.weixin.qq.com/s/_56jq_p_nDUiBwaI2MTlmA)

### 驗證

- JWT:
    - 基於 JWT + Refresh Token 的用戶認證實踐: [https://zhuanlan.zhihu.com/p/52300092](https://zhuanlan.zhihu.com/p/52300092)
    - Why Does OAuth v2 Have Both Access and Refresh Tokens?: [https://stackoverflow.com/questions/3487991/why-does-oauth-v2-have-both-access-and-refresh-tokens](https://stackoverflow.com/questions/3487991/why-does-oauth-v2-have-both-access-and-refresh-tokens)
    - OAuth 2.0 Token Exchange: [https://datatracker.ietf.org/doc/html/rfc8693](https://datatracker.ietf.org/doc/html/rfc8693)
- 安全性:
    - 零基礎資安系列（一）-認識 CSRF（Cross Site Request Forgery）: [https://tech-blog.cymetrics.io/posts/jo/zerobased-cross-site-request-forgery/](https://tech-blog.cymetrics.io/posts/jo/zerobased-cross-site-request-forgery/)
    - 零基礎資安系列（二）-認識 XSS（Cross-Site Scripting）: [https://tech-blog.cymetrics.io/posts/jo/zerobased-cross-site-scripting/](https://tech-blog.cymetrics.io/posts/jo/zerobased-cross-site-scripting/)
    - 零基礎資安系列（三）-網站安全三本柱（Secure & SameSite & HttpOnly）: [https://tech-blog.cymetrics.io/posts/jo/zerobased-secure-samesite-httponly/](https://tech-blog.cymetrics.io/posts/jo/zerobased-secure-samesite-httponly/)
    - 當瀏覽器全面停用三方 Cookie: [https://zhuanlan.zhihu.com/p/131256002](https://zhuanlan.zhihu.com/p/131256002)

### 限流

- 限流種類:
    - 令牌桶(token bucket)
    - 漏桶法(leaky bucket)
        - 恆定速路拿取token的方式對於redis來說不好實作，可用GCRA的方式實作
            - 原理:
                - [https://smarketshq.com/implementing-gcra-in-python-5df1f11aaa96](https://smarketshq.com/implementing-gcra-in-python-5df1f11aaa96)
                - [https://devpress.csdn.net/redis/62f4b93e7e66823466188869.html](https://devpress.csdn.net/redis/62f4b93e7e66823466188869.html)
            - redis實作: [https://github.com/brandur/redis-cell](https://github.com/brandur/redis-cell)
    - 固定窗口法(fixed window)
    - 滑動日制法(sliding log)
    - 滑動窗口法(sliding window)
- 限流概念: [https://guanhonly.github.io/2020/05/30/distributed-rate-limiter/](https://guanhonly.github.io/2020/05/30/distributed-rate-limiter/)
- redis 限流實作: [https://yuanchieh.page/posts/2020/2020-10-18-使用-redis-當作-api-rate-limit-的三種方法/](https://yuanchieh.page/posts/2020/2020-10-18-%E4%BD%BF%E7%94%A8-redis-%E7%95%B6%E4%BD%9C-api-rate-limit-%E7%9A%84%E4%B8%89%E7%A8%AE%E6%96%B9%E6%B3%95/)
- redis 5種限流實作: [https://medium.com/@SaiRahulAkarapu/rate-limiting-algorithms-using-redis-eb4427b47e33](https://medium.com/@SaiRahulAkarapu/rate-limiting-algorithms-using-redis-eb4427b47e33)
    - token bucket實作有點問題，導致實作結果與fixed window是一樣的
- Golang token bucket實作: [https://himanshu-007.medium.com/simple-rate-limiter-in-golang-using-token-bucket-algorithm-388d0596d1e4](https://himanshu-007.medium.com/simple-rate-limiter-in-golang-using-token-bucket-algorithm-388d0596d1e4)

### 熔斷

- 億級流量網關設計思路，學到了: [https://mp.weixin.qq.com/s/J0aSVry1-Ss1OTA-jQAX3w](https://mp.weixin.qq.com/s/J0aSVry1-Ss1OTA-jQAX3w)
- 簡單理解微服務限流、降級、熔斷: [https://juejin.cn/post/7150278137044008990](https://juejin.cn/post/7150278137044008990)

### 中間件

- Kafka:
    - 從面試角度一文學完 Kafka: [https://mp.weixin.qq.com/s/o-rqnOH4FHeHaz0VqoHnFg](https://mp.weixin.qq.com/s/o-rqnOH4FHeHaz0VqoHnFg)
    - Kafka詳解系列文:
        - [https://mp.weixin.qq.com/s?__biz=MzU2MTM4NDAwMw==&mid=2247488812&idx=1&sn=1e23afce50441bcf594c001f0965306b&chksm=fc78ca00cb0f4316e4c8583b84556c62574b50adaa8511d932459396944e9babeee9d141086b&scene=178&cur_album_id=1763234202604388353#rd](https://mp.weixin.qq.com/s?__biz=MzU2MTM4NDAwMw==&mid=2247488812&idx=1&sn=1e23afce50441bcf594c001f0965306b&chksm=fc78ca00cb0f4316e4c8583b84556c62574b50adaa8511d932459396944e9babeee9d141086b&scene=178&cur_album_id=1763234202604388353#rd)
        - [https://mp.weixin.qq.com/s?__biz=MzU2MTM4NDAwMw==&mid=2247490102&idx=1&sn=68d55b3c5ac74038c76d6837b862a11c&chksm=fc78c51acb0f4c0cd5a1d6ceedb9948f82d48791ab789e9edfd6e83e34fbad1ace5749bee203&scene=178&cur_album_id=1763234202604388353#rd](https://mp.weixin.qq.com/s?__biz=MzU2MTM4NDAwMw==&mid=2247490102&idx=1&sn=68d55b3c5ac74038c76d6837b862a11c&chksm=fc78c51acb0f4c0cd5a1d6ceedb9948f82d48791ab789e9edfd6e83e34fbad1ace5749bee203&scene=178&cur_album_id=1763234202604388353#rd)
        - [https://mp.weixin.qq.com/s?__biz=MzU2MTM4NDAwMw==&mid=2247491055&idx=1&sn=14bc485f91ec2629cc9e8bf7a36ad8f4&chksm=fc78c2c3cb0f4bd566d5ca2534805839420ad3dc67210bc8f2b7ef05283785b02b8ddef640a8&scene=178&cur_album_id=1763234202604388353#rd](https://mp.weixin.qq.com/s?__biz=MzU2MTM4NDAwMw==&mid=2247491055&idx=1&sn=14bc485f91ec2629cc9e8bf7a36ad8f4&chksm=fc78c2c3cb0f4bd566d5ca2534805839420ad3dc67210bc8f2b7ef05283785b02b8ddef640a8&scene=178&cur_album_id=1763234202604388353#rd)
        - [https://mp.weixin.qq.com/s?__biz=MzU2MTM4NDAwMw==&mid=2247491168&idx=1&sn=bd37f96692b3f7cecdaf3172abdb7a8c&chksm=fc78c14ccb0f485a451f70c7ffbf5b05d0f500dfef6321703e7cdebdc0de902d9d77a547d469&scene=178&cur_album_id=1763234202604388353#rd](https://mp.weixin.qq.com/s?__biz=MzU2MTM4NDAwMw==&mid=2247491168&idx=1&sn=bd37f96692b3f7cecdaf3172abdb7a8c&chksm=fc78c14ccb0f485a451f70c7ffbf5b05d0f500dfef6321703e7cdebdc0de902d9d77a547d469&scene=178&cur_album_id=1763234202604388353#rd)
        - [https://mp.weixin.qq.com/s?__biz=MzU2MTM4NDAwMw==&mid=2247491507&idx=1&sn=f1bec356c94cd0101809dc11dcf27ba2&chksm=fc78c09fcb0f49898f6cc9b80499aeb871a80f95ab4fbe12c32567cab6a3521a6c33b61dd807&scene=178&cur_album_id=1763234202604388353#rd](https://mp.weixin.qq.com/s?__biz=MzU2MTM4NDAwMw==&mid=2247491507&idx=1&sn=f1bec356c94cd0101809dc11dcf27ba2&chksm=fc78c09fcb0f49898f6cc9b80499aeb871a80f95ab4fbe12c32567cab6a3521a6c33b61dd807&scene=178&cur_album_id=1763234202604388353#rd)
    - 我是如何將一個老系統的kafka消費者服務的性能提升近百倍的: [https://www.cnblogs.com/softwarearch/p/16443128.html](https://www.cnblogs.com/softwarearch/p/16443128.html)

## 測試

### [測試框架或套件的選擇](Test%20%E9%9A%A8%E7%AD%86%20ae63acd045944c3a8c066c250be40b33/%E6%B8%AC%E8%A9%A6%E6%A1%86%E6%9E%B6%E6%88%96%E5%A5%97%E4%BB%B6%E7%9A%84%E9%81%B8%E6%93%87%2062a527aee9e04c9e814583a902bfd953.md)

## 數據工程管道

- Airflow
  - [一段 Airflow 與資料工程的故事：談如何用 Python 追漫畫連載](https://leemeng.tw/a-story-about-airflow-and-data-engineering-using-how-to-use-python-to-catch-up-with-latest-comics-as-an-example.html)
- Flyte

## 前端

## 運維

---

## 交易系統

- 撮合引擎:
    - [https://himanshu-007.medium.com/simple-rate-limiter-in-golang-using-token-bucket-algorithm-388d0596d1e4](https://himanshu-007.medium.com/simple-rate-limiter-in-golang-using-token-bucket-algorithm-388d0596d1e4)
    - [https://github.com/yzimhao/trading_engine](https://github.com/yzimhao/trading_engine)
        - 相關介紹: [https://yzimhao.github.io/trading_engine/](https://yzimhao.github.io/trading_engine/)
    - [https://github.com/stingbo/gome](https://github.com/stingbo/gome)
        - 相關介紹: [https://www.cnblogs.com/stingbo/p/14259539.html](https://www.cnblogs.com/stingbo/p/14259539.html)
    - [https://github.com/gitbitex/gitbitex-spot](https://github.com/gitbitex/gitbitex-spot)
        - 衍伸引擎(基於Java實作): [https://github.com/gitbitex/gitbitex-new](https://github.com/gitbitex/gitbitex-new)
    - [https://github.com/xiiiew/lightning-engine](https://github.com/xiiiew/lightning-engine)
- 撮合引擎緩存與MQ概念: [https://zhuanlan.zhihu.com/p/95033793](https://zhuanlan.zhihu.com/p/95033793)
- 撮合引擎核心概念: [https://zhuanlan.zhihu.com/p/92274215](https://zhuanlan.zhihu.com/p/92274215)
- 證券交易系統撮合引擎的設計: [https://www.liaoxuefeng.com/article/1452011784503329](https://www.liaoxuefeng.com/article/1452011784503329)
- 證券交易系統設計與開發: [https://www.liaoxuefeng.com/article/1185272483766752](https://www.liaoxuefeng.com/article/1185272483766752)
- 交易系統架構演進之路（一）：1.0版: [https://juejin.cn/post/6900444689628037133](https://juejin.cn/post/6900444689628037133)
    - 可以看看他系列文演進的思路

## 票卷系統

- [https://segmentfault.com/a/1190000022566350?fbclid=IwAR1TbX7lbN0AKpvIXpWIo2a2iE_dlyUUM37oJP4wahPnBe-5S_jTvvKS5yc](https://segmentfault.com/a/1190000022566350?fbclid=IwAR1TbX7lbN0AKpvIXpWIo2a2iE_dlyUUM37oJP4wahPnBe-5S_jTvvKS5yc)

---

## 實用工具

- 圖形工具
    - [mermaid](https://mermaid.js.org/)
        - Week28 - 讓你心裡的邏輯具現化的念能力工具 Mermaid: [https://ithelp.ithome.com.tw/articles/10234553](https://ithelp.ithome.com.tw/articles/10234553)
    - [miro](https://miro.com/)
    - [figma](https://www.figma.com/)