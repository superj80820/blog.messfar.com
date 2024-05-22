+++
title = 'rust trait(特徵)跟java interface(介面)的差異'
date = 2024-01-09T00:00:00+08:00
tags = ['rust', 'java']
+++

rust trait設計精神是`資料`、`行為定義`、`行為實作`，而java interface是`資料 + 行為實作`、`行為定義`，使得產生以下差異:

1. rust trait將method與資料分離，但java interface沒有分離，這會導致行為實作與資料耦合
    - java interface得修改其他人的lib
        
        ```java
        
        // 此資料在其他人的lib
        class TwoInterfaces {
        }
        
        // -------
        
        // 如果想定義一個interface規範TwoInterfaces，一定得去改其他人的lib
        interface Bar {
          int someMethod();
        }
        ```
        
    - rust trait可直接改自己的code
        
        ```rust
        // 此資料在其他lib
        struct Struct;
        
        // -------
        
        // 如果想定義一個trait規範Struct，直接在自己的code裡面規範即可
        trait Trait<T> {
            fn method(&self) -> T;
        }
        
        impl Trait<u8> for Struct {
            fn method(&self) -> u8 {
                16
            }
        }
        
        impl Trait<String> for Struct {
            fn method(&self) -> String {
                "hello".to_string()
            }
        }
        
        fn main() {
            let s = Struct;
            let a: u8 = s.method();
            let b: String = s.method();
            println!("a={}, b={}", a, b);
        }
        ```
        
2. trait可以實作相同名稱但行為不同的method
    - java interface不能這樣做
        
        ```java
        // 行為綁定在資料上
        class TwoGenerics implements Foo<Integer>, Foo<String> {
          public void someMethod(??? value) {}  // Can't implement, must be one method
        }
        interface Foo<T> {
          void someMethod(T value);
        }
        ```
        
    - rust trait可以這樣做
        
        ```rust
        struct Struct;
        
        trait Trait<T> {
            fn method(&self) -> T;
        }
        
        impl Trait<u8> for Struct {
            fn method(&self) -> u8 {
                16
            }
        }
        
        impl Trait<String> for Struct {
            fn method(&self) -> String {
                "hello".to_string()
            }
        }
        
        fn main() {
            let s = Struct;
            // 因為行為沒有綁定在資料上，而是在呼叫時說明需要哪個行為
            let a: u8 = s.method();
            let b: String = s.method();
            println!("a={}, b={}", a, b);
        }
        ```
        

## 參考

- [https://www.quora.com/How-are-Rusts-traits-different-from-Javas-interface-concept](https://www.quora.com/How-are-Rusts-traits-different-from-Javas-interface-concept)
- [https://stackoverflow.com/questions/53085270/how-do-i-implement-a-trait-with-a-generic-method](https://stackoverflow.com/questions/53085270/how-do-i-implement-a-trait-with-a-generic-method)
- [https://stackoverflow.com/questions/69477460/is-rust-trait-the-same-as-java-interface](https://stackoverflow.com/questions/69477460/is-rust-trait-the-same-as-java-interface)