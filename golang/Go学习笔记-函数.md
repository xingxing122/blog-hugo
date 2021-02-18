---
title: "Go学习笔记-函数"
date: 2020-04-06T16:20:19+08:00
draft: false  
categories: [golang]
tags: [golang]

---

### 函数

<!--more-->

#### 1. 函数的定义

函数用于对代码块的逻辑封装，提供代码复用的最基本方式

语法

```go
func  函数名称(参数) [返回值类型]{
        函数体
}
```

#### 2. 函数调用

```go
package main

import "fmt"

// 定义hello，无参，无返回值
func  sysHello(){
	fmt.Println("hello")
}

//有参数，无返回值
func  sayHi(name string){
	fmt.Println("Hi",name)
}

//有返回值 add 
func  add(a int,b int)int{
	return  a  + b  //return  关键字用来向函数调用返回结果
}

func  main(){
	// 调用函数,函数名称(参数[实参])
	sysHello()
	sayHi("xingxing")
	c := add(4,3 )
    fmt.Println(c)
}

package main

import (
      "fmt"
)

func  sayHello(){
      fmt.Println("hello")
}

func add(a,b int) int{
      return a + b
}

//返回多个值
func op(a,b int)(int,int,int,int){
      return  a + b, a - b , a * b, a/b
}

// 命名返回值
func opv2(a,b int)(sum int, sub int,mul int, div int){
      sum  = a + b
      sub  = a - b
      mul  = a * b
      div  = a /b
      return
}


func main(){
      sayHello()
      add(1,2)
      c := add(3,4)
      fmt.Println(c)
      fmt.Println(op(4,2))
      a,b,c,d :=op(8,2)
      fmt.Println("a=",a,"b=",b,"c=",c,"d=",d)

      fmt.Println(opv2(3,2))

}
```

