---
title: golang study
categories: go
tags: go
---


## **golang交换数组位置**
<!-- more -->
```
package main

import (
    "fmt"
)

func main() {
    var a = []int{11, 22}
    a[0], a[1] = a[1], a[0]
    fmt.Println(a[0], a[1])
}

💬 [🔸 6 🔸 zengguang@localhost 111]$ 👉 go run main.go
22 11
```

## **golang生成随机数**
```
func makeArr(length int) []int {
    arr := make([]int, 0)
    rand.Seed(time.Now().Unix())
    for i := 0; i < length; i++ {
        arr = append(arr, rand.Intn(100))
    }
    return arr
}
```
## **golang函数调用**
```
package main
import (
    "fmt"
    _ "github.com/go-sql-driver/mysql"
)
func main() {
    c := 5
    a := 6
    d := test(c, a)
    f := test2(c)
    fmt.Println(d)
    fmt.Println(f)
}
func test(t int, t1 int) int {
    b := t * t1
    return b
}
func test2(t1 int) int {
    b := t1 - test(1, 2)
    return b
}
```
