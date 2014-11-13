+++
date = "2013-10-19T20:45:39+08:00"
menu = "main"
tags = ["gobyexample", "golang"]
title = "gobyexample - 19 - Methods"

+++

Go语言支持定义于结构体类型之上的方法

	package main

	import "fmt"

	type rect struct {
		width, height int
	}

	// func关键字和方法名之间的部分，我们称为方法的接收者（receiver），此处的接收者是*rect类型的
	func (r *rect) area() int {
		return r.width * r.height
	}

	// 方法的receiver既可以是值类型，也可以是指针类型，此处是值类型的receiver
	func (r rect) perim() int {
		return 2*r.width + 2*r.height
	}

	func main() {
		r := rect{width: 10, height: 5}

		// r是值类型，可以调用perim无可厚非，但是它也可以调用area
		fmt.Println("area: ", r.area())
		fmt.Println("perim:", r.perim())

		// rp是指针类型，可以调用area无可厚非，但是它也可以调用perim
		rp := &r
		fmt.Println("area: ", rp.area())
		fmt.Println("perim:", rp.perim())

		// Go在你执行方法调用的时候，自动对receiver做了转换：）
	}

以上例程的输出：

	$ go run methods.go 
	area:  50
	perim: 30
	area:  50
	perim: 30

那现在的问题是，我在定义方法的时候到底是把receiver定义为值类型还是指针类型呢？
通常来说，指针类型效率更高，因为值类型有一个拷贝的过程；另外对于指针类型的receiver，我们可以改变其字段，并且对调用者产生影响，而值类型的receiver因为有一个拷贝的过程，所以如果在方法内部改变字段的值，并不会对调用者造成影响，因为我们实际是改变的调用者的一个拷贝。

