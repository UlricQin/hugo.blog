+++
date = "2013-10-05T20:24:00+08:00"
menu = "main"
tags = ["gobyexample", "golang"]
title = "gobyexample - 5 - For"

+++

for循环是Go语言中唯一的循环结构，下面是for循环的三种类型：

	package main

	import "fmt"

	func main() {
		// 最基本的结构类型，只有一个条件，当while使
		i := 1
		for i <= 3 {
			fmt.Println(i)
			i = i + 1
		}
		// for经典结构 initial/condition/after
		for j := 7; j <= 9; j++ {
			fmt.Println(j)
		}
		// 如果for关键字后面没有条件语句，那就是死循环
		// 此时可以使用break跳出循环，或者使用return返回当前函数
		for {
			fmt.Println("loop")
			break
		}
	}


上面程序的输出结果：

	$ go run for.go
	1
	2
	3
	7
	8
	9
	loop

接下来我们将看到使用for的其他形式，比如range语句，channels，或者其他数据结构
