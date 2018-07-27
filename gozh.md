---
title: golang字符串与时间戳转换
categories: go
tags: go
---
.
<!-- more -->

Go语言中，获取时间戳用time.Now().Unix()，格式化时间用t.Format，解析时间用time.Parse。
看实例代码：
```
package main

 

import (

"fmt"

"time"

)

 

func main() {

//获取时间戳

timestamp := time.Now().Unix()

fmt.Println(timestamp)

 

//格式化为字符串,tm为Time类型

tm := time.Unix(timestamp, 0)

fmt.Println(tm.Format("2006-01-02 03:04:05 PM"))

fmt.Println(tm.Format("02/01/2006 15:04:05 PM"))

 

 

//从字符串转为时间戳，第一个参数是格式，第二个是要转换的时间字符串

tm2, _ := time.Parse("01/02/2006", "02/08/2015")

fmt.Println(tm2.Unix())

}

```
//增加时间年月日
time.AddDate(3, 0, 0)        增加三年
//时间戳转string
buytimeT.AddDate(3, 0, 0).Format("2006-01-02")


 
输出结果：
1423361979
2015-02-08 10:19:39 AM
08/02/2015 10:19:39 AM
1423353600
 
看了上面的代码，你可能会好奇，为什么格式字符串的时候，用的是2006-01-02这种格式。其实在Go语言里，这些数字都是有特殊函义的，不是随便指定的数字，见下面列表：
月份 1,01,Jan,January
日　 2,02,_2
时　 3,03,15,PM,pm,AM,am
分　 4,04
秒　 5,05
年　 06,2006
周几 Mon,Monday
时区时差表示 -07,-0700,Z0700,Z07:00,-07:00,MST
时区字母缩写 MST
```
1. #string到int  
2. int,err:=strconv.Atoi(string)  
3. #string到int64  
4. int64, err := strconv.ParseInt(string, 10, 64)  
5. #int到string  
6. string:=strconv.Itoa(int)  
7. #int64到string  
8. string:=strconv.FormatInt(int64,10)
```
```
判断数据类型
package main 
import ( 
"fmt" “reflect"
) 
func main() { 
var x int32 = 20 
fmt.Println("type:", reflect.TypeOf(x)) 
}
字符串截取
   rs := []rune(str)
    fmt.Println(string(rs[起始位置:截取长度]))
```