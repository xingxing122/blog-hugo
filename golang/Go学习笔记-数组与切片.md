---
title: "Go学习笔记 数组与切片"
date: 2020-04-28T23:19:36+08:00
draft: false  
categories: [golang]
tags: [golang]

---

  go 数组与切片 

<!--more-->

1. 数组

   数组就是指一系列同一类型的数据集合.数组中包含的每个数据被称为数组元素，一个数组包含的元素被称为数组的长度

   数组长度必须是常量，且是类型的组成部分

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

   数组初始化

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
   
   冒泡排序
   
   ```go 
   package main
   
   import (
   	"fmt"
   	"math/rand"
   	"time"
   )
   
   func main() {
   
   	rand.Seed(time.Now().UnixNano())
   	var a [10]int
   	n := len(a)
   	
   	for i := 0; i < n; i++ {
   		a[i] = rand.Intn(100) //100以内的随机数
   		fmt.Printf("%d", a[i])
   	
   	}
   	
   	fmt.Printf("\n")
   	
   	// 冒泡排序的业务逻辑
   	for i := 0; i < n-1; i++ {
   		for j := 0; j < n-1-i; j++ {
   			if a[j] > a[j+1] {
   				a[j], a[j+1] = a[j+1], a[j]
   			}
   		}
   	
   	}
   	for i := 0; i < n; i++ {
   		fmt.Printf("%d", a[i])
   		fmt.Printf("\n")
   	}
   
   }
   ```
   
   切片
   
   slice 概述
   
   数组的长度在定义之后无法修改，数组是值类型，每次传递都将产生一份副本，显然这种数据结构无法满足开发者的真实需求。Go语言提供了数组切片(Slice)来弥补数组的不足。
   
   切片并不是数组或指针，他通过内部指针和相关的属性引用数组片段，以实现比如:
   
   arr := [...]int{0,1,2,3,4,5}
   
   s  := arr[2:6]
   
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
   
   

