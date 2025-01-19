---
title: "Go 语言基础知识"
date: 2024-03-17T15:53:16+08:00
draft: true
categories:
  - learn
tags:
  - go
summary: "Go 语言基础知识"
description: "Go 语言基础知识"
author: ["XiangLin"]
cover:
  image: ""
  caption: ""
  alt: ""
  relative: false
---

## Go 语言基础

### Go 语言特点

原生的并发编程支持

通过管道实现goroutine之间的通信，使用简单且安全

defer机制

独特的异常处理方式保障代码的健壮性

提供了丰富便捷的网络编程接口

Go语言将编程规范集成到了语言本身

### Go 语言包管理的方式

#### Go 包管理的发展历史

GOPATH(< Go 1.5)、Go Vendor(>= Go 1.5)、Go Modules(>= Go 1.11)

GOROOT是Golang的安装目录

GOPATH是Go语言指定的工作空间，默认目录是：`/home/<username>/go`

#### Go Modules

go mod 命令

```bash
download    download modules to local cache
edit        edit go.mod from tools or scripts
graph       print module requirement graph
init        initialize new module in current directory
tidy        add missing and remove unused modules
vendor      make vendored copy of dependencies
verify      verify dependencies have expected content
why         explain why packages or modules are needed
```

`go.mod` 文件

```makefile
module example.com/m

go 1.22.1

require (
	gorm.io/driver/mysql v1.5.2
	gorm.io/gorm v1.25.5
)

require (
	github.com/go-sql-driver/mysql v1.7.1 // indirect
	github.com/jinzhu/inflection v1.0.0 // indirect
	github.com/jinzhu/now v1.1.5 // indirect
)

```

`go.sum` 文件防下载的依赖被恶意篡改，主要用于安全校验。

```makefile
github.com/go-sql-driver/mysql v1.7.0/go.mod h1:OXbVy3sEdcQ2Doequ6Z5BW6fXNQTmx+9S1MCJN5yJMI=
github.com/jinzhu/inflection v1.0.0/go.mod h1:h+uFLlag+Qp1Va5pdKtLDYj+kHp5pxUVkryuEj+Srlc=
github.com/jinzhu/now v1.1.5/go.mod h1:d3SSVoowX0Lcu0IBviAWJpolVfI5UJVZZ7cO71lE/z8=
gorm.io/driver/mysql v1.5.2/go.mod h1:pQLhh1Ut/WUAySdTHwBpBv6+JKcj+ua4ZFx1QQTBzb8=
gorm.io/gorm v1.25.2-0.20230530020048-26663ab9bf55/go.mod h1:L4uxeKpfBml98NYqVqwAdmV1a2nBtAec/cf3fpucW/k=
gorm.io/gorm v1.25.5/go.mod h1:hbnx/Oo0ChWMn1BIhpy1oYozzpM15i4YPuHDmfYtwg8=
```

#### Go 命令行工具

```bash
build       compile packages and dependencies
get         add dependencies to current module and install them
install     compile and install packages and dependencies
run         compile and run Go program
test        test packages
fmt         gofmt (reformat) package sources
mod         module maintenance
```

### 函数和变量的可见性

小写字母开头的函数、变量、结构体只能在本包内访问

大写字母开头的函数、变量、结构体可以在其它包访问

### `init()` 函数

用于程序执行前包的初始化，比如HTTPServer、RedisClient的初始化。

#### `init()`函数的执行顺序

在同一个Go文件中的多个init方法，按照代码顺序依次执行。

在同一个package中，按照文件名的顺序执行。

不同package且不相互依赖，按照import顺序执行。

不同package且相互依赖，按照依赖关系，被依赖的包中的init先执行。

#### go文件的初始化顺序

1. 引入的包
2. 当前包中的常量变量
3. 当前包的init方法
4. main函数

一个包被引用多次，这个包的init函数只会执行一次。

所有init函数都在同一个goroutine内执行。

### Go 语言获取项目根路径

```go
package main

import (
	"fmt"
	"os"
	"path/filepath"
)

func main() {
	// 执行 go run 的目录
	abs, _ := filepath.Abs("./")
	fmt.Println("abs", abs)

	getwd, _ := os.Getwd()
	fmt.Println("getwd", getwd)

	fmt.Println("os.Arg", os.Args[0])

	executable, _ := os.Executable()
	fmt.Println("executable", executable)
}

```

### Go 语言格式字符串

```ma
%d          十进制整数
%x, %o, %b  十六进制，八进制，二进制整数。
%f, %g, %e  浮点数： 3.141593 3.141592653589793 3.141593e+00
%t          布尔：true或false
%c          字符（rune） (Unicode码点)
%s          字符串
%q          带双引号的字符串"abc"或带单引号的字符'c'
%v          变量的自然形式（natural format）
%T          变量的类型
%%          字面上的百分号标志（无操作数）
```

### Go 语言中 `new` 和 `make`

make 返回类型初始化的值，new 返回类型的零值。

make 只适合 slice、map、channel 的数据，new 没有限制。

make 返回原始类型 `Type`，new 返回类型的指针 `*Type`。

```go
// The make built-in function allocates and initializes an object of type
// slice, map, or chan (only). Like new, the first argument is a type, not a
// value. Unlike new, make's return type is the same as the type of its
// argument, not a pointer to it. The specification of the result depends on
// the type:
//
//	Slice: The size specifies the length. The capacity of the slice is
//	equal to its length. A second integer argument may be provided to
//	specify a different capacity; it must be no smaller than the
//	length. For example, make([]int, 0, 10) allocates an underlying array
//	of size 10 and returns a slice of length 0 and capacity 10 that is
//	backed by this underlying array.
//	Map: An empty map is allocated with enough space to hold the
//	specified number of elements. The size may be omitted, in which case
//	a small starting size is allocated.
//	Channel: The channel's buffer is initialized with the specified
//	buffer capacity. If zero, or the size is omitted, the channel is
//	unbuffered.
func make(t Type, size ...IntegerType) Type

// The new built-in function allocates memory. The first argument is a type,
// not a value, and the value returned is a pointer to a newly
// allocated zero value of that type.
func new(Type) *Type
```

### 数组和切片

数组和切片都要求全部元素的类型必须相同，且元素存放在连续的内存空间。

数组的零值是每个元素类型的零值，切片的零值是 nil。

相同长度的数组是为同一类型可以比较，不同长度的数组不能比较，数组不能和 nil 比较。

切片之间不能比较，切片可以和 nil 比较。

#### 切片

一个 slice 由三个部分构成：指针、长度和容量。指针指向第一个 slice 元素对应的底层数组元素的地址，要注意的是 slice 的第一个元素并不一定就是数组的第一个元素。

![img](https://gopl-zh.github.io/images/ch4-01.png)
