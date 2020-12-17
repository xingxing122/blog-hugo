---
title: "Go学习笔记 变量"
date: 2020-01-15T23:14:42+08:00
draft: false  
categories: [golang]
tags: [golang]

---

一、变量

<!--more-->

变量的含义:   程序允许过程中的数据都是保存在内存中，我们想要在代码中操作某个数据时就需要去内存找到这个变量，但是如果我们直接在代码中通过内存地址去操作变量的话，代码的可读性会非常差还容易出错，所以我们就利用变量将这个数据的内存地址保存起来，以后直接通过这个变量就能找到内存上对应的数据了。

变量的类型:  变量的功能是存储数据。不同变量保存的数据类型会不一样。常见变量的数据类型有: 整型、浮点型、布尔型等。Go语言中每一个变量都有自己的类型，并且变量必须经过声明才开始使用。

变量声明: Go 语言中的变量需要声明后才能使用，同一作用域内不支持重复声明。并且Go语言的变量声明后必须使用，否则会报。 

单个变量的声明与赋值 

```go
变量的声明格式:  var  <变量名称>   <变量类型>    函数外的变量声明必须都是以var 开头进行声明
变量的赋值格式:  var  <变量名称>   <表达式>
声明的同时赋值:  var  <变量名称>   [变量类型] =<表达式>
```



```go
#单个变量的声明与赋值
package main

import "fmt"

func variableZeroValue() {
	var a int
	var b string

	//变量赋值
	a = 10
	fmt.Printf("%d %q", a, b)
}
func main() {
	variableZeroValue()
	fmt.Println("hello, world")
}
```

多个变量的声明与赋值 

   全局变量的声明可使用var()方式进行简写 

   全局变量的声明不可以省略var，单可以使用并行方式

   所有变量都可以使用类型推断

  局部变量是不可以使用var的方式简写，只能使用并行方式 

```go
package main
import "fmt"

var (
	aa = 22
	bb = true
)

var abc, bbb, ccc, ddd = 1, 2, 3, 4

func main() {
	variableZeroValue()
	fmt.Println("hello, world")
	varInt()
	fmt.Println(aa, bb, abc, bbb, ccc, ddd)
}
```

二、常量

常量是一个简单值的标识符，在程序运行时，不会被修改的量。 

常量中的数据类型只有布尔型、数字型(整数型、浮点型和复数)和字符串型

























