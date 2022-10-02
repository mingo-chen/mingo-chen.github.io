---
title       : "通过Channel控制并发数"
author      : "mingo"
date        : "2020-06-13 22:44:29"

layout      : post
category    : golang
tag         : ["golang","original"]
blog        : true
star        : true
---

## 概述

golang开启一个协程是很方便的, 但是如果不加限制的开启协程, 可能会导致系统资源被占尽, 进而导致服务不可用

本文总结了常见的通过channel来控制并发数量的实现思路

## 方式一

```golang
var concurrency = 16 // 希望的控制并发数为16

var limit = make(chan int, concurrency) 

for _, work := range works {
    limit <- 1

    go func(w) {
        defer func() {  // 防止process发生错误导致 limit 没有释放
            <- limit
        }()

        defer func() { // 防止子协程panic导致主协程panic
            if e := recover(); e != nil {
                log(e)
            }
        }

        process(w)  // 真正业务处理

    }(work) // 防止闭包引用错误
}
```

## 方法二

待补充

## 参考


