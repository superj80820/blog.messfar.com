+++
title = '測試框架或套件的選擇'
date = 2024-01-15T00:00:00+08:00
tags = ['testing']
+++

- behave: python BDD的發揚光大者
    - 好處
        - BDD語法
        - 底層使用python撰寫，一定程度可以使用python
    - 壞處
        - 套件生態系不是很強大
        - 併發測試需安裝叫繁瑣的框架 [https://github.com/hugeinc/behave-parallel](https://github.com/hugeinc/behave-parallel)
- pytest: 擁有廣大的生態系套件
    - 好處
        - 套件生態系強大
        - 可使用 pytest-bdd 套件，讓pytest支援BDD語法
        - 可是用 pytest-xdist 套件，讓pytest支援併發測試
        - 完全由python衍伸的測試方案，對python套件的支持度最高
    - 壞處
        - 較無
- robot framework: 常見的經典測試框架
    - 優點
        - 擁有自身語法，兼具可讀性與撰寫的好處
            - 如果有需求，也可以使用BDD的寫法(e.g. [https://github.com/yazidisme/robotframework-bdd-example/blob/master/tests/login_tests.robot](https://github.com/yazidisme/robotframework-bdd-example/blob/master/tests/login_tests.robot))
            - 除了使用自身語法，也可引用python撰寫底層
        - 社區更新與維護積極
        - 自身提供精美的報表
    - 缺點
        - 使用原生python來撰寫底層，始終多了一層語言的轉換，一些python原生語法要另外包裝會稍微麻煩，例如: lambda, pattern matching, ... 等等
        - 沒有大家共同使用的標準化lint工具，大多都是自己實作(e.g. [https://github.com/boakley/robotframework-lint](https://github.com/boakley/robotframework-lint) )，規範團隊coding style不易

## 結論

我會建議選擇pytest，原因如下:

- 有多樣的選擇，你可以用很精簡的方式撰寫測項，也可以通過pytest-bdd套件設計有架構的專案
- 擁有大量的套件，可以快速建構測項
- 原生的python語法，測試者不需重新熟悉語言

## 感謝

- 感謝 Robin([https://ithelp.ithome.com.tw/users/20110242/profile](https://ithelp.ithome.com.tw/users/20110242/profile))提供建議

## 參考

- [https://blog.testproject.io/2019/05/16/python-testing-framework-pros-cons/](https://blog.testproject.io/2019/05/16/python-testing-framework-pros-cons/)