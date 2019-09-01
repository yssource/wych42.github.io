+++
title = "golang []byte vs string"
author = ["chi"]
date = 2019-09-01T14:19:00+08:00
lastmod = 2019-09-01T14:26:00+08:00
tags = ["golang"]
categories = ["program"]
draft = false
toc = true
+++

## 区别 {#区别}

-   `[]byte` 是 slice，可以修改
-   `string` 是 read-only `[]byte`

> Strings are immutable: once created, it is impossible to change the contents of a string.&nbsp;[^fn:1]

-   `[]byte` 底层有 data, size, capacity, `string` 没有 capacity

string immutable 的含义指的是：

-   一个 string 所在的内存区域内容是不改变的，比如 "hello"
-   多个 string 可以共享（slice）同一个内存地址，比如 "el"
-   原 string 变量（内存地址）被重新赋值之后，共享的变量们的值不会改变

{{< figure src="images/golang-byte-slice.png" >}}

```go
package main

import "fmt"

func main() {
    a := "hello"
    b := a[0:2]
    fmt.Printf("a is at %x and has value %q\n", &a, a)
    fmt.Printf("b is at %x and has value %q\n", &b, b)

    a = "world"
    fmt.Printf("a is at %x and has value %q\n", &a, a)
    fmt.Printf("b is at %x and has value %q\n", &b, b)
}

/*
a is at 40c128 and has value "hello"
b is at 40c130 and has value "he"
a is at 40c128 and has value "world"
b is at 40c130 and has value "he"
*/
```


## 互转 {#互转}

-   `[]byte` string 互转需要拷贝。[^fn:2]


## 参考 {#参考}

-   [<https://blog.golang.org/strings>](<https://blog.golang.org/strings>)
-   [<https://golang.org/ref/spec#Conversions%5Fto%5Fand%5Ffrom%5Fa%5Fstring%5Ftype>](<https://golang.org/ref/spec#Conversions%5Fto%5Fand%5Ffrom%5Fa%5Fstring%5Ftype>)
-   [<https://golang.org/ref/spec#Conversions>](<https://golang.org/ref/spec#Conversions>)
-   [<https://github.com/golang/go/issues/25484>](<https://github.com/golang/go/issues/25484>)

[^fn:1]: <https://golang.org/ref/spec#String%5Ftypes>
[^fn:2]: <https://golang.org/src/strings/builder.go#L45>
