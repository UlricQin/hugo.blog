+++
date = "2013-10-11T20:34:34+08:00"
menu = "main"
tags = ["gobyexample", "golang"]
title = "gobyexample - 11 - Range"

+++

Range可以对很多数据结构中的元素进行遍历，下面让我们看看如何使用range遍历我们之前学过的数据结构：）

	package main

	import "fmt"

	func main() {
		// 此处我们使用range来对slice中的数字做求和
		// 数组和slice是同样的工作方式
		nums := []int{2, 3, 4}
		sum := 0
		for _, num := range nums {
			sum += num
		}
		fmt.Println("sum:", sum)

		// range对array或slice做循环的时候，每次返回两个值：index，value
		// 上面的例子我们不需要index，所以使用下划线 _ 给忽略掉了
		// 不过有的时候我们确实需要index，比如下面的例子（牵强了，纯为举例）
		for i, num := range nums {
			if num == 3 {
				fmt.Println("index:", i)
			}
		}

		// 对一个map做range，会返回KV对
		kvs := map[string]string{"a": "apple", "b": "banana"}
		for k, v := range kvs {
			fmt.Printf("%s -> %s\n", k, v)
		}

		// 对字符串的range操作是基于Unicode码点。所以中文也不怕：）
		// 但是要注意index的值，此时的字符串可以看做是[]rune
		// index就是各个rune的第一个byte的索引，很绕是么，看两个例子
		fmt.Println("range string1>>>")
		for i, c := range "golang" {
			fmt.Println(i, c)
		}

		// “中”这个rune占了3个byte（第二、第三、第四个），所以索引是2
		// “国”这个rune占了3个byte（第五、第六、第七个），所以索引是5
		fmt.Println("range string2>>>")
		for i, c := range "go中国go" {
			fmt.Println(i, c)
		}
	}

上面程序的输出结果：

	$ go run a.go
	sum: 9
	index: 1
	a -> apple
	b -> banana
	range string1>>>
	0 103
	1 111
	2 108
	3 97
	4 110
	5 103
	range string2>>>
	0 103
	1 111
	2 20013
	5 22269
	8 103
	9 111

另外对map做循环的时候我们可以只要key，不要value，甚至都无需对第二个返回值用下划线代替：

	package main

	import "fmt"

	func main() {
		kvs := map[string]string{"a": "apple", "b": "banana"}
		for k := range kvs {
			fmt.Printf("%s\n", k)
		}
	}

