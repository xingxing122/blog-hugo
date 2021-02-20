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
函数名: 满足标识符规范
形参:  无形参
      有形参, 名字、类型，多个形参使用逗号分隔 
返回值： 
     无返回值
        返回值省略
     有返回值: 
        return关键字
     必须指定返回值类型 
           只有一个返回值，返回值只需要写类型，可以省略小括号
           如果有多个返回值，需要用小括号包含所有返回值类型 
           return 返回值数量必须和函数定义返回值类型数量一致
      命名返回值 
           返回值定义值为每个返回值指定了变量名称及类型,用括号包含
           返回是只用指定return
调用:  
    接收返回值  = 函数名(实参)
非可变参数:  
    实参数量必须与形参一致 => 实参  按照顺序传递给形参 
可变参数: 
    可变参数定义之前的变量  必须指定实参传递
    可变参数部分可以传递任意多个值(可使用切片解包)
返回值: 
   无返回值不能接收 
   有返回值 必须用相同数量的变量接收返回值(按照返回值的顺序赋值给可接收的变量)


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

#### 3 汉诺塔算法(递归)

```go
package main

import "fmt"

func  tower(a,b,c string,layer int){
   if  layer ==1 {
      fmt.Println(a,"->",c)
      return
   }
   tower(a,c,b,layer-1)
   fmt.Println(a,"->",c)
   tower(b,a,c,layer-1)
}

func main() {
  tower("A","B","C",3)

}
```

#### 4 冒泡

```go
package main

import "fmt"

func bubble(nums []int){
      for j := 0;j<len(nums)-1;j++{
            for i :=0 ; i<len(nums)-1-j;i++{
                  if nums[i] > nums[i+1] {
                        nums[i],nums[i+1]=nums[i+1],nums[i]
                  }
            }
      }
      fmt.Println(nums)

}

func main() {
      nums :=[]int{5,4,3,2}
      fmt.Println(nums)
      nums =[]int{100,90,88,70,1000,100001}
      bubble(nums)

}
```











