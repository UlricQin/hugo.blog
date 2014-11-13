+++
date = "2013-10-15T20:40:57+08:00"
menu = "main"
tags = ["gobyexample", "golang"]
title = "gobyexample - 15 - Closures"

+++

Go语言支持匿名函数，这个特性可以构建闭包。当你需要内联函数时匿名函数比较有用。闭包这个概念不是很好理解，我们先看下面的例子：

	package main

	import "fmt"

	// 函数intSeq返回另一个函数，它是在intSeq的函数体内定义的匿名函数
	// 返回的匿名函数用到了变量i，这形成了闭包
	func intSeq() func() int {
		i := 0
		return func() int {
			i += 1
			return i
		}
	}

	func main() {
		// 我们调用intSeq，把返回的结果（一个函数）赋值给nextInt
		// 这个函数保持自己的变量i，变量i在每次调用nextInt的时候都会被更新
		nextInt := intSeq()

		// 我们调用几次nextInt，看看闭包的效果
		fmt.Println(nextInt())
		fmt.Println(nextInt())
		fmt.Println(nextInt())

		// 为了确认不同的闭包有不同的状态，我们重新创建并且测试一个新的返回函数。
		newInts := intSeq()
		fmt.Println(newInts())
	}

上面程序的输出：

	$ go run closures.go
	1
	2
	3
	1

对于闭包我们可以这么理解：intSeq函数中的i变量，本来是一个局部变量，intSeq函数执行完了之后i的生命周期结束，但是在返回的匿名函数中用到了i，Go一看你把返回的匿名函数赋值给了nextInt，它就不敢干掉i了，因为i一旦被干掉，nextInt就没法调用了，所以经过一次调用之后i从0变成了1，此时nextInt还是有可能被调用，所以Go仍然不能干掉i，直到出了nextInt的作用域了，即main函数执行完了，Go知道nextInt肯定不会再次被用到了，这个时候才会干掉i（纯粹个人瞎掰，帮助理解闭包）

