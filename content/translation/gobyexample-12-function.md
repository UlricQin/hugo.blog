+++
date = "2013-10-12T20:35:44+08:00"
menu = "main"
tags = ["gobyexample", "golang"]
title = "gobyexample - 12 - Function"

+++

函数是Go语言中的重点啦，我们将通过几个例子来逐步学习

	package main

	import "fmt"

	// 下面是一个简单的例子，接收两个int参数，返回一个int结果
	func plus(a int, b int) int {
		// Go语言需要显式写return语句，不会自动得返回函数中的
		// 最后一个表达式的值，请Ruby码农注意
		return a + b
	}
	func main() {
		// 如你所想，函数调用语法为：func_name(args)
		res := plus(1, 2)
		fmt.Println("1+2 =", res)
	}

以上程序的运行结果：

	$ go run functions.go 
	1+2 = 3

Go中的函数还有一些其他特性，比如多返回值，接下来马上介绍：）
