+++
title = '0283-move zeros'
date = 2024-01-02T00:00:00+08:00
tags = ['algo']
+++

- 題目網址: [https://leetcode.com/problems/move-zeroes](https://leetcode.com/problems/move-zeroes)
- 解法:
    
    ```go
    // 氣泡排序
    // time complexity O(n^2)
    // space complexity O(1)
    func moveZeroes(nums []int)  {
        for i := 0; i < len(nums); i++ {
            for j := i+1; j < len(nums); j++ {
                if nums[i] < nums[j] && (nums[i] == 0 || nums[j] == 0) {
                    nums[i], nums[j] = nums[j], nums[i]
                }
            }
        }
    }
    
    // 快慢指標
    // time complexity O(n)
    // space complexity O(1)
    func moveZeroes(nums []int)  {
        for i, j := 0, 1; j < len(nums); j++ {
            if nums[i] != 0 {
                i++
                continue
            }
            if nums[j] != 0 {
                nums[i], nums[j] = nums[j], nums[i]
                i++
            }
        }
    }
    ```