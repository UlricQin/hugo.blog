+++
date = "2013-10-17T20:43:23+08:00"
menu = "main"
tags = ["gobyexample", "golang"]
title = "gobyexample - 17 - Pointers"

+++

Go语言支持指针，想必大家都有C语言的基础，就不做过多介绍了，不过**Go不支持指针运算**

下面的例子通过对比zeroval和zeroptr来演示指针是如何工作的：

	package main

	import "fmt"

	// zeroval的参数是int类型的，参数通过值传递的方式传入函数
	// zeroval将会得到ival的一个拷贝，所以函数内部对ival再怎么改变赋值，都不会影响外层的值
	func zeroval(ival int) {
		ival = 0
	}

	// 相比而言,zeroptr有一个*int类型的参数，这是int类型的指针
	// 与C语言一样*iptr可以存取该指针对应的值
	func zeroptr(iptr *int) {
		*iptr = 0
	}

	func main() {
		i := 1
		fmt.Println("initial:", i)
		zeroval(i)
		fmt.Println("zeroval:", i)
		// 与C语言一样，&i可以获得i的内存地址，即i的指针
		zeroptr(&i)
		fmt.Println("zeroptr:", i)
		// 指针可以被直接打印输出
		fmt.Println("pointer:", &i)
	}

以上程序的运行结果：

	$ go run pointers.go
	initial: 1
	zeroval: 1
	zeroptr: 0
	pointer: 0x42131100

通过输出的结果可以看出，main函数中调用zeroval并没有改变变量 i 的值，但zeroptr却给改变了，因为zeroptr的参数是对变量 i 的内存地址的引用：）

下节我们学习结构体：Structs

