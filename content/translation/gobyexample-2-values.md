+++
date = "2013-10-02T20:18:49+08:00"
menu = "main"
tags = ["gobyexample", "golang"]
title = "gobyexample - 2 - Values"

+++

Go语言支持多种数据类型，比如：字符串、整型、浮点型、bool型等等。下面是一个简单的例子：

	package main

	import (
		"fmt"
	)

	func main() {
		// 字符串可以用 + 拼接
		fmt.Println("go" + "lang")

		// 整型、浮点型例子
		fmt.Println("1+1=", 1+1)
		fmt.Println("7.0/3.0=", 7.0/3.0)

		// bool型，支持bool运算
		fmt.Println(true && false)
		fmt.Println(true || false)
		fmt.Println(!true)
	}

运行结果如下：

	$ go run values.go
	golang
	1+1 = 2
	7.0/3.0 = 2.3333333333333335
	false
	true
	false
