+++
date = "2013-10-03T20:20:32+08:00"
menu = "main"
tags = ["gobyexample", "golang"]
title = "gobyexample - 3 - Variables"

+++

在Go语言中，变量要求显式声明，编译器据此判断函数调用的正确性。这是编译型强类型语言的典型特点啦：）

下面直接上一段代码，通过代码注释的方式为大家解释：

	package main

	import "fmt"

	func main() {
		// var 声明一个或多个变量
		var a string = "initial"
		fmt.Println(a)
		// 可以一次性声明多个变量
		var b, c int = 1, 2
		fmt.Println(b, c)
		// 如果一个变量赋有初始值，go可以据此推断出变量类型
		var d = true
		fmt.Println(d)
		// 没有初始值的变量声明，go会自动为其设置零值
		// 比如此处，int的零值就是0（string的零值是空字符串""，bool的零值是false）
		var e int
		fmt.Println(e)
		// := 语法是声明并初始化一个变量的简写方式，
		// 此处实际是表示：var f string = "short"
		f := "short"
		fmt.Println(f)
	}

程序运行结果：

	$ go run variables.go
	initial
	1 2
	true
	0
	short
