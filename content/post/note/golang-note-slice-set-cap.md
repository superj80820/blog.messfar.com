+++
title = 'Golang Slice 有無設定 cap 的差異'
date = 2024-02-02T00:00:00+08:00
tags = ['go']
+++

```go
package main

import (
	"fmt"
)

func printSlice(s []int) {
	fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}

func main() {
	s := make([]int, 5)
	printSlice(s)

	s2 := make([]int, 0, 5)
	printSlice(s2)
	for i := 0; i < 5; i++ {
		s2 = append(s2, 1)
	}
	printSlice(s2)
	s2 = append(s2, 1)
	printSlice(s2)
}

// print

// len=5 cap=5 [0 0 0 0 0]

// len=0 cap=5 []
// len=5 cap=5 [1 1 1 1 1]
// len=6 cap=10 [1 1 1 1 1 1]
```

- 設定cap會讓內部元素達到cap的量才重新調整len長度，並且是依照cap設定的值一次加上，例如cap為5就是等到元素達到6個時一次讓cap變為10
- 依照情境設計cap，可以減少調整slice的cap頻繁改變的問題，增進效能