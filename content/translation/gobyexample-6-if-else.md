+++
date = "2013-10-06T20:25:23+08:00"
menu = "main"
tags = ["gobyexample", "golang"]
title = "gobyexample - 6 - If/Else"

+++

这次我们来看一下GO语言中的if-else分支语句

	package main

	import "fmt"

	func main() {
		// 这是一个基本例子
		// 通常包裹着条件语句的括号（）在GO中可以干掉啦，
		// 但是花括号{}还是不能省滴
		if 7%2 == 0 {
			fmt.Println("7 is even")
		} else {
			fmt.Println("7 is odd")
		}
		// 可以只要if没有else
		if 8%4 == 0 {
			fmt.Println("8 is divisible by 4")
		}
		// 可以在条件语句之前放置一个语句，比如此处的num := 9
		// 这个语句中声明的变量（此处是指num）在所有分支中都可见
		if num := 9; num < 0 {
			fmt.Println(num, "is negative")
		} else if num < 10 {
			fmt.Println(num, "has 1 digit")
		} else {
			fmt.Println(num, "has multiple digits")
		}
	}

上面程序的输出结果：

	$ go run if-else.go 
	7 is odd
	8 is divisible by 4
	9 has 1 digit

注意：Go语言不支持三元运算符（? :），就是说，这样的写法是不支持滴：

	isLogin := isLogin() ? true : false

