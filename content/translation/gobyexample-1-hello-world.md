+++
date = "2013-10-01T20:13:25+08:00"
menu = "main"
tags = ["gobyexample", "golang"]
title = "gobyexample - 1 - Hello world"

+++

学习golang过程发现一个很好的站点： https://gobyexample.com/ ，采用example的方式来讲授golang，不过是英文的，为了golang在我大天朝的发展，我决定把它翻译成中文，过程中会加入我自己的理解，不会和原文完全一致，往好处说叫：**意译**，哈哈：）

开始之前请先安装golang的语言环境，并且设置GOROOT和GOPATH环境变量，这个步骤请自行Google解决：）
编辑器或IDE呢？嗯，看个人喜好了，我现在用sublime text，环境安装方法请参看： http://my.oschina.net/Obahua/blog/110767 

第一个例子，自然是经典的golang版本hello world，下面是代码：

	package main
	import "fmt"
	func main() {
		fmt.Println("hello world")
	}

要运行这个程序，我们可以把上面的源码保存为：hello-world.go，然后使用go run命令来运行：

	$ go run hello-world.go
	hello world

有的时候我们想把程序编译为二进制文件，go build可以帮你达成目的：

	$ go build hello-world.go
	$ ls
	hello-world	hello-world.go

编译完成之后，就可以直接执行这个二进制文件了:

	$ ./hello-world
	hello world

O了，基本的编译执行过程就是这样的啦

# 简单解读

像C和Java一样，一切从main函数开始，并且main函数所在的package（很多语言都有package概念，这是组织代码的一种典型方式，不赘述）也需要叫做main。import是导入其他的package，fmt是golang自带的一个可以用于输出的基础package，`fmt.Println("hello world")`是调用fmt包的Println方法，注意，此处方法的第一个字符是大写的，这是golang的规定，所有可以被外部包调用的方法都必须用大写字母开头，算是很多语言中public/protected/private的另一种实现方式：）