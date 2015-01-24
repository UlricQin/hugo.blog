+++
date = "2015-01-24T13:14:31+08:00"
menu = "main"
tags = ["golang"]
title = "小心for循环中的goroutine"

+++

这也是工作中写挫的代码，需求是这样的：内存中缓存了N多对象，我们要把这些对象慢慢持久化到磁盘上，内存中对象数目过千万，搞了几个worker线程去慢慢做，代码类似下面这个样子。

```
package main

import (
	"fmt"
	"time"
)

var MemoryData = map[string]string{
	"key1": "val1",
	"key2": "val2",
	"key3": "val3",
	"key4": "val4",
	"key5": "val5",
	// ...
}

func Keys() []string {
	keys := make([]string, 0, len(MemoryData))
	for key := range MemoryData {
		keys = append(keys, key)
	}
	return keys
}

var workerChan = make(chan int, 3)

func main() {
	for {
		syncDisk()
		time.Sleep(10 * time.Second)
	}
}

func syncDisk() {
	keys := Keys()
	for _, key := range keys {
		workerChan <- 1
		go func() {
			write(key)
			<-workerChan
		}()
	}
}

func write(key string) {
	fmt.Println("write", key)
}

```

运行之后你会发现每次都不是write的上面5个数据，而是类似下面这个样子：

```
write key5
write key5
write key5
write key1
write key1
```

数目是5个没错，但是key5写了三次，key1写了两次，剩下的三个key，都没轮到……这也太不公平了，原因出在哪里呢？

goroutine中用到了外面的key这个变量，创建goroutine速度很快，goroutine中的代码还没来得急调度执行，外面就循环了多次了，于是key就指向了最后一个循环变量，然后，然后就影响了goroutine的内部代码……

那我们是不是不引用外部的key就可以了呢？

```
go func(key string) {
	write(key)
	<-workerChan
}(key)
```

如此之后输出如下：

```
write key3
write key4
write key5
write key1
write key2
```

成了~多线程中的共享内存真是个容易出错的点啊，以后要注意喽：）
