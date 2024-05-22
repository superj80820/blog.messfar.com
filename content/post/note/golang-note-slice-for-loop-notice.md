+++
title = 'Golang for 迴圈怎麼會都列出最後一個值'
date = 2024-02-02T00:00:00+08:00
tags = ['go']
+++

- for range
    - 實際上是語法糖([原code在此](https://github.com/golang/gofrontend/blob/e387439bfd24d5e142874b8e68e7039f74c744d7/go/statements.cc#L5501))，會將code轉化成以下
        
        ```go
        // The loop we generate:
        //   len_temp := len(range)
        //   range_temp := range
        //   for index_temp = 0; index_temp < len_temp; index_temp++ {
        //           value_temp = range_temp[index_temp]
        //           index = index_temp
        //           value = value_temp
        //           original body
        //   }
        ```
        
    - 這導致以下幾點需注意:
        - 在for range裡以`v`進行指標的賦值，會全部指向最後一個元素，因為`v`實際上是一個每次迭代都會被修改的變數
            
            ```go
            items := []string{"a", "b", "c"}
            var newItems []*string
            for _, v := range items {
            	newItems = append(newItems, &v)
            }
            fmt.Println("new items: ", *newItems[0], *newItems[1], *newItems[2])
            
            // print: new items:  c c c
            ```
            
            - 解決方式:
                1. 不用`v`賦值，而是直接指向`items`的元素
                    
                    ```go
                    items := []string{"a", "b", "c"}
                    var newItems []*string
                    for i := range items {
                    	newItems = append(newItems, &items[i])
                    }
                    fmt.Println("new items: ", *newItems[0], *newItems[1], *newItems[2])
                    
                    // print: new items:  a b c
                    ```
                    
                2. 複製`v`後再進行賦值
                    
                    ```go
                    items := []string{"a", "b", "c"}
                    var newItems []*string
                    for _, v := range items {
                    	v := v
                    	newItems = append(newItems, &v)
                    }
                    fmt.Println("new items: ", *newItems[0], *newItems[1], *newItems[2])
                    
                    // print: new items:  a b c
                    ```
                    
        - 在for range運行修改`items`會修改原本的`items`，不會影響for range列的`v`元素，因為for range 的 items 已經被複製
            
            ```go
            items := []string{"a", "b", "c"}
            for _, v := range items {
            	items = []string{}
            	fmt.Println("value: ", v)
            	fmt.Println("items: ", items)
            }
            fmt.Println("final items: ", items)
            
            // print:
            // value:  a
            // items:  []
            // value:  b
            // items:  []
            // value:  c
            // items:  []
            // final items:  []
            ```
            
    - 還有其他細節，但比較少遇到，詳細可參考: [https://zhuanlan.zhihu.com/p/105435646](https://zhuanlan.zhihu.com/p/105435646)