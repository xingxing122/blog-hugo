---
title: "Go学习笔记-fmt包介绍"
date: 2020-04-06T16:20:19+08:00
draft: false  
categories: [golang]
tags: [golang]

---

fmt 包介绍 

fmt 实现了格式化I/O 函数，类似于C的printf 和scanf 格式

<!--more-->

一般:

```go
%v 相应值的默认格式。在打印结构体时，“加号” 标记(%+v)会添加字段名
%#v 相应值的Go语法表示
%T  相应值的类型的Go 语法表示
%%  字面上的百分号，并非值的占位符
```

布尔:

```go
%t  单词 true 或者 false
```

整数:

```
%b 二进制表达
%c 相应的unicode码点所表示的字符
%d 十进制表示
%o 八进制表示
%q 单引号围绕的字符字面值
%x 十六进制表示，字符形式小写a-f 
%X 十六进制表示，字母形式的大写A-F 
%U Unicode格式: U+1234,等于"U+%04X"
```

浮点数及其复合构成

```
%b 无小数部分的，指数为二的幂的科学计数法
%e 科学计数法
%E 科学计数法
%f 有小数点而无指数
```

字符串与字节切片

```
%s  字符串或切片的无解译字节
%q  双引号围绕的字符串，由Go 语法安全地转移
%x  十六进制，小写字母
%X  十六进制，大写字母
```

指针

```
%p   十六进制表示前缀 0x 
```



实例：

```
package main

import "fmt"

func main() {
	var a int = 50
	var b bool
	c := 'a'

	fmt.Printf("%v", a)
	fmt.Printf("%t", b)
	fmt.Printf("%v", c)

}
```

