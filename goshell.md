---
title: golang调用os命令
categories: go
tags: go
---
.
<!-- more -->
```
package main

import (
    "bytes"
    "fmt"
    "log"
    "os/exec"
)

func shell_exec(s string) {

    cmd := exec.Command("/bin/bash", "-c", s)
    var out bytes.Buffer

    cmd.Stdout = &out
    err := cmd.Run()
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("%s", out.String())
}
func main() {

    shell_exec("cd /Users/zengguang/Downloads/pi&&rm -rf ./ceshi.sh")
}
```
