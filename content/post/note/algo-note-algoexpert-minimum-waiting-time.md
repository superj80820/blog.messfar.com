+++
title = 'algoexpert-minimum-waiting-time'
date = 2024-01-02T00:00:00+08:00
tags = ['algo']
+++

- 題目: [https://www.algoexpert.io/questions/minimum-waiting-time](https://www.algoexpert.io/questions/minimum-waiting-time)
- 問題: 一個可重新排列的array`[3, 2, 1, 2, 6]`，找出排列後累積相加最小的array，累積相加的方式為每往後一個數字，之前的數字就會累加。以此array來說，重新排列成`[1, 2, 2, 3, 6]`會有最小的累加結果，即:
    - `[1, 2, 2, 3, 6] = 0 + (1) + (1 + 2) + (1 + 2 + 2) + (1 + 2 + 2 + 3) = 17`
- 解決思路:
    - 直接排序，因為:
        - 第一個數字會累加最多次，所以一定要放最小的
        - 最後一個數字不會累加到，所以一定要放最大的
    - 累加方式為「array長度 - (index + 1) 」，因為:
        - 第一個數字的重複累加次數為: array長度 - 1
        - 第二個數字的重複累加次數為: array長度 - 2
        - 由此類推即可發現
- 演算法:
    - 程式
        
        ```go
        package main
        
        import "sort"
        
        func MinimumWaitingTime(queries []int) int {
        	sort.Slice(queries, func(i, j int) bool {
                return queries[i] < queries[j]
            })
        
            totalWatingTime := 0
            for idx, duration := range queries {
                queriesLeft := len(queries) - (idx + 1)
                totalWatingTime += duration * queriesLeft
            }
        
        	return totalWatingTime
        }
        ```
        
    - 時間複雜度: O(nlogn)
        - 最快的排序法為O(nlogn) (e.g. Merge Sort, Heap Sort)
        - 下方有一個for迴圈，速度為O(n)
        - 以上兩者相加，可忽略O(n)，所以為O(nlogn)
    - 空間複雜度: O(1)
        - 不需要額外的空間來儲存，所以即為O(1)