+++
date = "2013-10-14T20:39:08+08:00"
menu = "main"
tags = ["gobyexample", "golang"]
title = "gobyexample - 14 - Variadic Function"

+++

可变参数函数：即函数调用的时候可以传入一个参数，可以传入两个参数，可以传入三个参数，或者更多……fmt.Println就是一个典型的可变参数函数

	package main

	import "fmt"

	// 下面这个sum函数可以接收任意数目的数字作为参数
	func sum(nums ...int) {
		fmt.Print(nums, " ")
		total := 0
		fmt.Printf("len(nums)=%d\n", len(nums))
		for _, num := range nums {
			total += num
		}
		fmt.Println(total)
	}
	func main() {
		// 可变参数函数可以使用通常的方法调用，传入多个参数值即可
		sum(1, 2)
		sum(1, 2, 3)

		// 如果你已经有一个包含有多个参数的slice，可以直接使用这个slice作为
		// 可变参数函数的参数，语法：func(slice...) 三个点...可以理解为把slice打散
		nums := []int{1, 2, 3, 4}
		sum(nums...)
	}

以上代码输出结果：

	$ go run a.go
	[1 2] len(nums)=2
	3
	[1 2 3] len(nums)=3
	6
	[1 2 3 4] len(nums)=4
	10

在Go语言中，函数本身可以作为参数传递进入另一个函数，所以可以制作闭包，下节介绍：）

