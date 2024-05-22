+++
title = 'Golang 隨筆'
date = 2024-02-02T00:00:00+08:00
tags = ['go']
+++

## 錯誤處理

- [Golang 錯誤處理](Golang%20%E9%9A%A8%E7%AD%86%209fabdb9d476348f59038e4c2dc8adbf5/Golang%20%E9%8C%AF%E8%AA%A4%E8%99%95%E7%90%86%20dd86c225fd3444f49063883087ca9bc1.md)
- Golang 最佳錯誤處理實踐: [https://medium.com/@dche423/golang-error-handling-best-practice-cn-42982bd72672](https://medium.com/@dche423/golang-error-handling-best-practice-cn-42982bd72672)

## 語法基礎

- Slice
    - [Golang for 迴圈怎麼會都列出最後一個值](Golang%20%E9%9A%A8%E7%AD%86%209fabdb9d476348f59038e4c2dc8adbf5/Golang%20for%20%E8%BF%B4%E5%9C%88%E6%80%8E%E9%BA%BC%E6%9C%83%E9%83%BD%E5%88%97%E5%87%BA%E6%9C%80%E5%BE%8C%E4%B8%80%E5%80%8B%E5%80%BC%20e850a5e26f9041229e3289bd1d6e021a.md)
    - [Golang Slice 有無設定 cap 的差異](Golang%20%E9%9A%A8%E7%AD%86%209fabdb9d476348f59038e4c2dc8adbf5/Golang%20Slice%20%E6%9C%89%E7%84%A1%E8%A8%AD%E5%AE%9A%20cap%20%E7%9A%84%E5%B7%AE%E7%95%B0%20580a0e3f56324f0397e52cb60033669a.md)
- Map
    - [https://stackoverflow.com/questions/49176090/map-as-a-method-receiver](https://stackoverflow.com/questions/49176090/map-as-a-method-receiver)

## 測試

- Mock: [https://www.myhatchpad.com/insight/mocking-techniques-for-go/](https://www.myhatchpad.com/insight/mocking-techniques-for-go/)

## 並發

- Worker Pool
- 解決 deadlock 的方式
    - 順序鎖
    - 分段鎖
- Graceful Shutdown: [https://ithelp.ithome.com.tw/articles/10272236](https://ithelp.ithome.com.tw/articles/10272236)
- Golang Atomic: [https://gfw.go101.org/article/concurrent-atomic-operation.html](https://gfw.go101.org/article/concurrent-atomic-operation.html)
- Golang select中的break、continue和return: [https://tomjamescn.github.io/post/2019-07-11-golang-basic-1-break-continue-return-in-select/](https://tomjamescn.github.io/post/2019-07-11-golang-basic-1-break-continue-return-in-select/)

## 底層

- MPG模型

## 其他

- Golang 為什麼不能用slice當作map的key
    - [https://stackoverflow.com/questions/20297503/slice-as-a-key-in-map](https://stackoverflow.com/questions/20297503/slice-as-a-key-in-map)

## 注意事項

- [Golang for 迴圈怎麼會都列出最後一個值](Golang%20%E9%9A%A8%E7%AD%86%209fabdb9d476348f59038e4c2dc8adbf5/Golang%20for%20%E8%BF%B4%E5%9C%88%E6%80%8E%E9%BA%BC%E6%9C%83%E9%83%BD%E5%88%97%E5%87%BA%E6%9C%80%E5%BE%8C%E4%B8%80%E5%80%8B%E5%80%BC%20e850a5e26f9041229e3289bd1d6e021a.md)

## 套件與資源

- Golang 基本功練習
    - [https://github.com/GoesToEleven/GolangTraining/tree/master](https://github.com/GoesToEleven/GolangTraining/tree/master)
- worker pool: [https://github.com/panjf2000/ants](https://github.com/panjf2000/ants)
- dependency injection: [https://github.com/google/wire](https://github.com/google/wire)

---

- 子頁面
    
    [Golang Slice 有無設定 cap 的差異](Golang%20%E9%9A%A8%E7%AD%86%209fabdb9d476348f59038e4c2dc8adbf5/Golang%20Slice%20%E6%9C%89%E7%84%A1%E8%A8%AD%E5%AE%9A%20cap%20%E7%9A%84%E5%B7%AE%E7%95%B0%20580a0e3f56324f0397e52cb60033669a.md)
    
    [Golang for 迴圈怎麼會都列出最後一個值](Golang%20%E9%9A%A8%E7%AD%86%209fabdb9d476348f59038e4c2dc8adbf5/Golang%20for%20%E8%BF%B4%E5%9C%88%E6%80%8E%E9%BA%BC%E6%9C%83%E9%83%BD%E5%88%97%E5%87%BA%E6%9C%80%E5%BE%8C%E4%B8%80%E5%80%8B%E5%80%BC%20e850a5e26f9041229e3289bd1d6e021a.md)
    
    [Golang 錯誤處理](Golang%20%E9%9A%A8%E7%AD%86%209fabdb9d476348f59038e4c2dc8adbf5/Golang%20%E9%8C%AF%E8%AA%A4%E8%99%95%E7%90%86%20dd86c225fd3444f49063883087ca9bc1.md)