+++
date = "2013-10-08T20:28:29+08:00"
menu = "main"
tags = ["gobyexample", "golang"]
title = "gobyexample - 8 - Array"

+++

几乎所有语言都有数组这个数据结构，无非就是一个定长的可以用数字索引的元素集合，不做过多解释：） 

	package main

	import "fmt"

	func main() {
		// 下面声明一个数组，包含5个元素，每个元素是int类型
		// Go中数组的元素需要是同类型的，ruby/python码农请注意
		// 数组的类型由两部分组成：1、数组长度 2、元素类型
		// a这个变量的类型是[5]int，[6]int、[5]int、[5]int64是三个不同的类型
		// 数组一旦声明，默认会被赋予零值，此处的a默认值是：[0,0,0,0,0]
		var a [5]int
		fmt.Println("emp:", a)

		// 赋值和取值采用通用的语法，不做过多解释
		a[4] = 100
		fmt.Println("set:", a)
		fmt.Println("get:", a[4])

		// 数组长度采用内置的len来获取
		fmt.Println("len:", len(a))

		// 我们可以在一行代码中声明并初始化一个数组
		b := [5]int{1, 2, 3, 4, 5}
		fmt.Println("dcl:", b)

		// 数组类型是一维的，不过凭你的聪明才智，
		// 肯定可以基于此搞出多维的数据结构，不是么：）
		var twoD [2][3]int
		for i := 0; i < 2; i++ {
			for j := 0; j < 3; j++ {
				twoD[i][j] = i + j
			}
		}
		fmt.Println("2d: ", twoD)
	}

上面程序的运行结果：

	$ go run arrays.go
	emp: [0 0 0 0 0]
	set: [0 0 0 0 100]
	get: 100
	len: 5
	dcl: [1 2 3 4 5]
	2d:  [[0 1 2] [1 2 3]]

下节我们会学习比Array更常用的slice：）
