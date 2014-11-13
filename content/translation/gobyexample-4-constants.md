+++
date = "2013-10-04T20:22:28+08:00"
menu = "main"
tags = ["gobyexample", "golang"]
title = "gobyexample - 4 - Constants"

+++

Go语言支持字符、字符串、布尔和数字类型类型的常量，老规矩，代码说话，解释看注释：

	package main

	import "fmt"
	import "math"

	// 使用 const 关键字声明一个常量
	const s string = "constant"

	func main() {
		fmt.Println(s)
		// 一个常量声明语句可以出现在一个变量声明语句可以出现的任意位置
		const n = 500000000
		// 常量表达式可以执行任意精度的数学运算
		const d = 3e20 / n
		fmt.Println(d)
		// 一个数字常量没有类型，除非你手工指定，比如通过显式类型强转
		fmt.Println(int64(d))
		// 除了通过类型强转之外，还有一些其他方式让数字常量变得具备类型。
		// 看你使用的上下文，比如math.Sin(n)，这个Sin函数要求传入一个float64类型的数字，所以，传入这个函数的常量就自动变成了float64类型
		fmt.Println(math.Sin(n))
	}

程序运行结果：

	$ go run constant.go 
	constant
	6e+11
	600000000000
	-0.28470407323754404
