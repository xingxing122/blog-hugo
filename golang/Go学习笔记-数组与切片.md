---
title: "Go学习笔记 数组与切片"
date: 2020-04-28T23:19:36+08:00
draft: false  
categories: [golang]
tags: [golang]

---

  go 数组与切片 

<!--more-->

### 1. 数组

数组就是指一系列同一类型的数据集合.数组中包含的每个数据被称为数组元素，一个数组包含的元素被称为数组的长度

数组长度必须是常量，且是类型的组成部分。

数组元素使用操作符[]来索引，索引从0开始。因此一个数组的首元素是array[0],其最后一个元素是array[len(array)-1]  

```
package main

import (
	"fmt"
)

func main() {
	// 定义一个数组,[10]int 和[5]int是不同类型
	//[数字]这个数字作为数组元素的个数

	var a [10]int
	var b [5]int
	
	fmt.Printf("len(a)=%d,len(b)=%d\n", len(a), len(b))

}
```

```
package main

import (
	"fmt"
)

func main() {
	// 定义一个数组,[10]int 和[5]int是不同类型
	//[数字]这个数字作为数组元素的个数

	var a [10]int
	var b [5]int
	
	fmt.Printf("len(a)=%d,len(b)=%d\n", len(a), len(b))
	
	//赋值
	for i := 0; i < len(a); i++ {
		a[i] = i + 1
	}
	
	//打印
	//第一个返回下标，第二个返回元素
	for i, data := range a {
		fmt.Printf("a[%d]=%d\n", i, data)
	
	}

}
```

#### 1.1 数组初始化

```
1. 声明定义同时赋值,叫初始化 

   1. 全局初始化 

      var  a[5]int =[5] int {1,2,3,4,5}

   2. 简洁写法

       b := [5]int{1,2,3,4,5}
   3. 部分初始化，没有初始化的元素，int自动复制为0 
       c :=[5]int{1,2,3}
   4. 指定某个元素初始化
        d :=[5]int{2:10,4:20}
```

实例

```go
package main

import "fmt"

func main() {

	var a [5]int = [5]int{1, 2, 3, 4, 5}
	fmt.Println("a=", a)
	//1. 全局省略写法
	b := [5]int{1, 2, 3, 4, 5}
	fmt.Println("b=", b)
	c := [5]int{1, 2, 3}
	
	fmt.Println("c=", c)

}
```

二维数组

比较

```go
   package main
   import "fmt"

   func main() {
   	// 支持比较，比较2个数组类型要一致，只支持==和 !=
   	a := [5]int{1, 2, 3, 4, 5}
   	b := [5]int{1, 2, 3, 4, 5}
   	c := [5]int{1, 2, 3}
   
   	fmt.Println("a==b", a == b)
   	fmt.Println("a==c", a == c)
   	
   	//同类型的数组可以赋值
   	var d [5]int
   	d = a
   	fmt.Println("d=", d)

   }
```

### 2. 切片

切片(slice)概述

数组的长度在定义之后无法修改，数组是值类型，每次传递都将产生一份副本，显然这种数据结构无法满足开发者的真实需求。Go语言提供了数组切片(Slice)来弥补数组的不足。

#### 2.1 切片的创建语法

```go
make([]type,length,capacity)
make([]type,length)
[]type{}
[]type{value1,value2...,valueN}
```

| 语法       | 含义/结果                                    |
| ---------- | -------------------------------------------- |
| s[n]       | 切片s中索引位置为n 的项                      |
| s[n:m]     | 从切片s的索引位置n到m-1处所获得的切片        |
| s[n:]      | 从切片s的索引位置n到len(s)-1处所获得的切片   |
| s[:m]      | 从切片s到索引位置0到m-1处获得的切片          |
| s[:]       | 从切片s 的索引位置0到len(s)-1 处所获得的切片 |
| cap(s)     | 从切片s的容量，总是len>=len(s)               |
| Len(s)     | 切皮s中所包含项的个数，总是<= cap(s)         |
| s[:cap(s)] | 增加切片s 的长度到其容量，如果两者不同的话   |

切片并不是数组或指针，他通过内部指针和相关的属性引用数组片段，以实现比如:

```go
arr := [...]int{0,1,2,3,4,5}
s  := arr[2:6]
```

```go 
package main

import "fmt"

func main() {

	array := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
	s1 := array[:]
	fmt.Println("s1=", s1)
	fmt.Printf("len=%d,cap=%d\n", len(s1), cap(s1))

}
```

切片与隐藏数组的关系 

```go
package main

import "fmt"

func main(){
   s := []string {"A","B","C","D","E","F","G"}
   t := s[:5]  //A,B,C,D,E 
   u := s[3:len(s)-1] //D,E,F 
   fmt.Println(s,t,u)


   u[1] = "X"
   fmt.Println(s,t,u)

}
结果
[A B C D E F G] [A B C D E] [D E F]
[A B C D X F G] [A B C D X] [D X F] 

分析
s[0],s[1],s[2],s[3],s[4],s[5],s[6]="A","B","C","D","E","F","G"
由于切片s、t、u 都是同一底层数组的引用，其中一个改变会影响到其他所有指向该相同数组的任何其他引用 
```

<img src="https://xing-blog.oss-cn-beijing.aliyuncs.com/2021-01-29-072902.png" alt="image-20210129152901933" style="zoom:50%;" />

#### 2.2 索引与分割切片

```go
package main

import "fmt"

func main(){
   s := []string {"A","B","C","D","E","F","G"}
   t := s[2:6]
   fmt.Println(t,s,"=",s[:4],"+",s[4:])
   s[3] = "x"
   t[len(t)-1] = "y"
   fmt.Println(t,s,"=",s[:4],"+",s[4:])
   
}
#对于一个切片s 和一个索引值i(0<=i<=len(s)),s等于s[:1]与s[i:]的连接。 
s ==s[:i] + s[i:] //s 是一个字符串，i 是整数，0<=i <= len(s)
```

我们可以改变数据，无论是通过原始切片s 还是通过切片s 的切片t，它们底层数据都改变了，因此两个切片都受影响。 

#### 2.3 遍历切片

遍历一个切片中的元素。如果我们想要取得切片中的某个元素而不想修改它，可以使用for ......range 循环  

```go
package main

import "fmt"

func main(){
   s := []string {"A","B","C","D","E","F","G"}
   for _, v := range  s {
      fmt.Println(v)
   }
}
# for ....range 循环首先初始化一个从0开始的循环计数器 
```

#### 2.4 修改切片



