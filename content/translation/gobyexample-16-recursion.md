+++
date = "2013-10-16T20:42:20+08:00"
menu = "main"
tags = ["gobyexample", "golang"]
title = "gobyexample - 16 - Recursion"

+++

Go语言支持递归函数，这里有一个典型的例子：计算阶乘。代码如下：

	package main

	import "fmt"

	// 此处fact函数会一直调用自身，直到fact(0)
	func fact(n int) int {
		if n == 0 {
			return 1
		}
		return n * fact(n-1)
	}
	func main() {
		fmt.Println(fact(7))
	}

上面程序运行结果：

	$ go run recursion.go 
	5040

下个例子我们讲述指针
