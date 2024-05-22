+++
title = 'Golang 隨筆'
date = 2024-02-02T00:00:00+08:00
tags = ['go']
+++

## 錯誤處理

- [Golang 錯誤處理](https://superj80820.github.io/blog.messfar.com/post/note/golang-note-error-handle)
- Golang 最佳錯誤處理實踐: [https://medium.com/@dche423/golang-error-handling-best-practice-cn-42982bd72672](https://medium.com/@dche423/golang-error-handling-best-practice-cn-42982bd72672)

## 語法基礎

- Slice
    - [Golang for 迴圈怎麼會都列出最後一個值](https://superj80820.github.io/blog.messfar.com/post/note/golang-note-slice-for-loop-notice)
    - [Golang Slice 有無設定 cap 的差異](https://superj80820.github.io/blog.messfar.com/post/note/golang-note-slice-set-cap)
- Map
    - [https://stackoverflow.com/questions/49176090/map-as-a-method-receiver](https://stackoverflow.com/questions/49176090/map-as-a-method-receiver)

## 測試

- Mock: [https://www.myhatchpad.com/insight/mocking-techniques-for-go/](https://www.myhatchpad.com/insight/mocking-techniques-for-go/)

## 並發

- Worker Pool
- 解決 deadlock 的方式
    - 順序鎖
    - 分段鎖
- [Graceful Shutdown](https://superj80820.github.io/blog.messfar.com/post/design-pattern/go-design-pattern-concurrency-patterns)
- Golang Atomic: [https://gfw.go101.org/article/concurrent-atomic-operation.html](https://gfw.go101.org/article/concurrent-atomic-operation.html)
- Golang select中的break、continue和return: [https://tomjamescn.github.io/post/2019-07-11-golang-basic-1-break-continue-return-in-select/](https://tomjamescn.github.io/post/2019-07-11-golang-basic-1-break-continue-return-in-select/)

## 底層

- MPG模型

## 其他

- Golang 為什麼不能用slice當作map的key
    - [https://stackoverflow.com/questions/20297503/slice-as-a-key-in-map](https://stackoverflow.com/questions/20297503/slice-as-a-key-in-map)

## 注意事項

- [Golang for 迴圈怎麼會都列出最後一個值](https://superj80820.github.io/blog.messfar.com/post/note/golang-note-slice-for-loop-notice)

## 套件與資源

- Golang 基本功練習
    - [https://github.com/GoesToEleven/GolangTraining/tree/master](https://github.com/GoesToEleven/GolangTraining/tree/master)
- worker pool: [https://github.com/panjf2000/ants](https://github.com/panjf2000/ants)
- dependency injection: [https://github.com/google/wire](https://github.com/google/wire)
