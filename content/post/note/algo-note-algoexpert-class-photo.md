+++
title = '隨筆: algoexpert-class-photos'
date = 2024-01-05T00:00:00+08:00
tags = ['algo']
+++

- 題目: [https://www.algoexpert.io/questions/class-photos](https://www.algoexpert.io/questions/class-photos)
- 問題: 班級要排隊拍照，要符合以下條件:
    - 要分兩排來拍照，有一半學生穿紅色衣服，一半學生穿藍色衣服，同排必須是相同顏色
    - 前排的學生必須比較矮
    - 每列的學生須由高到矮
- 解決思路:
    - 每列的學生須由高到矮，所以先對兩排學生進行排序
    - 由第一列的學生決定是紅色為第一排，還是藍色為第一排
    - 如果紅色為第一排，那後續的列如果有紅色比藍色高的情形，則無法滿足問題，反之
- 演算法:
    - 程式
        
        ```go
        package main
        
        import "sort"
        
        func ClassPhotos(redShirtHeights []int, blueShirtHeights []int) bool {
            sort.Slice(redShirtHeights, func(i, j int) bool {
                return redShirtHeights[i] > redShirtHeights[j]
            })
            sort.Slice(blueShirtHeights, func(i, j int) bool {
                return blueShirtHeights[i] > blueShirtHeights[j]
            })
        
            var firstRow string
            if redShirtHeights[0] < blueShirtHeights[0] {
                firstRow = "RED"
            } else {
                firstRow = "BLUE"
            }
        
            for idx := range redShirtHeights {
                if firstRow == "RED" {
                    if redShirtHeights[idx] >= blueShirtHeights[idx] {
                        return false
                    }
                } else if firstRow == "BLUE" {
                    if blueShirtHeights[idx] >= redShirtHeights[idx] {
                        return false
                    }
                }
            }
            
            return true
        }
        ```
        
    - 時間複雜度: O(nlogn)
        - 最快的排序法為O(nlogn) (e.g. Merge Sort, Heap Sort)
        - 下方有一個for迴圈，速度為O(n)
        - 以上兩者相加，可忽略O(n)，所以為O(nlogn)
    - 空間複雜度: O(1)
        - 不需要額外的空間來儲存，所以即為O(1)