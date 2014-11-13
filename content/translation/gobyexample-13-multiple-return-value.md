+++
date = "2013-10-13T20:36:53+08:00"
menu = "main"
tags = ["gobyexample", "golang"]
title = "gobyexample - 13 - Multiple Return Values"

+++

Go语言对多返回值有语言层面的内置支持。在写Go程序的时候这个特性经常被用到，比如一个函数既返回结果又返回error（如果函数正常运行，error返回nil，如果内部出错，返回对应的error）

	package main

	import "fmt"

	// 函数签名中的 （int，int） 表示这个函数返回两个int值
	func vals() (int, int) {
		return 3, 7
	}
	func main() {
		// 此处我们调用这个多返回值的函数，使用两个变量来接收返回值
		a, b := vals()
		fmt.Println(a)
		fmt.Println(b)

		// 如果你只是需要多返回值的一个子集，使用下划线 _ 忽略不需要的值即可
		_, c := vals()
		fmt.Println(c)
	}

以上程序运行结果：

	$ go run multiple-return-values.go
	3
	7
	7

接收可变数目的参数是Go函数的另一个非常赞的特性，我们将在下节介绍：）

