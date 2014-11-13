+++
date = "2013-10-20T20:46:46+08:00"
menu = "main"
tags = ["gobyexample", "golang"]
title = "gobyexample - 20 - Interfaces"

+++

Go语言中的接口无非就是一坨方法的集合罢了。

下面的示例中有一个接口类型：geometry，它有两个方法：area和perim。square和circle这两个struct实现了area和perim这两个方法，我们就说square和circle实现了geometry接口，此时就可以把square和circle当做geometry来用。根本不需要像其他语言中那样，实现个接口还要使用implements关键字。Go给你展示了什么叫真正的无侵入式设计，碉堡了：）

	package main

	import "fmt"
	import "math"

	type geometry interface {
		area() float64
		perim() float64
	}

	type square struct {
		width, height float64
	}
	type circle struct {
		radius float64
	}

	func (s square) area() float64 {
		return s.width * s.height
	}
	func (s square) perim() float64 {
		return 2*s.width + 2*s.height
	}

	func (c circle) area() float64 {
		return math.Pi * c.radius * c.radius
	}
	func (c circle) perim() float64 {
		return 2 * math.Pi * c.radius
	}

	func measure(g geometry) {
		fmt.Println(g)
		fmt.Println(g.area())
		fmt.Println(g.perim())
	}
	func main() {
		s := square{width: 3, height: 4}
		c := circle{radius: 5}
		// 因为square和circle都实现了geometry接口，此处就可以直接把
		// square和circle的实例当做参数传给measure
		measure(s)
		measure(c)
	}

以上例程的输出：

	$ go run interfaces.go
	{3 4}
	12
	14
	{5}
	78.53981633974483
	31.41592653589793

如果想了解更多Go语言的接口设计思路，可以看这篇文章：[如何在Go语言中使用接口](http://jordanorelli.tumblr.com/post/32665860244/how-to-use-interfaces-in-go)

