---
title: golang gin框架应用
categories: golang
tags: golang
---
.
<!-- more -->

## main.go
```
package main

import (
    "net/http"

    "github.com/gin-gonic/gin"
)

func main() {
    gin.SetMode(gin.ReleaseMode)
    router := gin.New()
    router.LoadHTMLGlob("templates/*")
    // 注册static下静态文件目录，“.”表示全部文件
    router.StaticFS("/static", http.Dir("."))
    router.Use(gin.Logger())
    router.GET("/index", func(c *gin.Context) {
        c.HTML(http.StatusOK, "index.html", gin.H{
            "title1": "gaga",
        })
    })
    router.Run(":8080")
}
```
## 模板
```
<html>
    <h1>
        {{ .title1 }}
    </h1>
</html>
```