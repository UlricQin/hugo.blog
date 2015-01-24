+++
date = "2015-01-24T12:09:04+08:00"
menu = "main"
tags = ["golang"]
title = "小心golang中的循环变量"

+++

众所周知，一个好的监控系统是产品快速迭代、稳定运行的基石。我们公司内部用的监控系统叫Falcon，为了提高监控的性能、稳定性、易运维性，最近对Falcon的部分组件做了优化，过程中遇到golang语言层面的一些该注意的点与大家分享

**今天先介绍一下golang中的循环变量**

我们coding过程中经常会遇到对一批对象做遍历处理，筛选出我们需要的那部分，比如下面的例子，我们要找出users中的所有Manager，假设你是Managers方法的编写者，InitUsers方法是之前别人写的，为了代码少改动，姑且接收了一个[]User类型的参数，为了性能上的考虑，返回的是[]*User，这样之后的代码拿到Manager列表就是个指针slice了，效率应该会高一些。

```
package main

import (
	"fmt"
)

type User struct {
	Id   int
	Name string
	Role int
}

func (this *User) String() string {
	return fmt.Sprintf("<Id:%d, Name:%s, Role:%d>", this.Id, this.Name, this.Role)
}

func (this *User) IsManager() bool {
	return this.Role > 5
}

func InitUsers() []User {
	users := make([]User, 0, 5)
	users = append(users, User{Id: 1, Name: "Ulric", Role: 10})
	users = append(users, User{Id: 2, Name: "Rain", Role: 8})
	users = append(users, User{Id: 3, Name: "Flame", Role: 5})
	users = append(users, User{Id: 4, Name: "Sun", Role: 3})
	users = append(users, User{Id: 5, Name: "Moon", Role: 1})
	return users
}

func Managers(users []User) []*User {
	managers := make([]*User, 0, len(users))
	for _, user := range users {
		if user.IsManager() {
			managers = append(managers, &user)
		}
	}
	return managers
}

func main() {
	users := InitUsers()
	managers := Managers(users)
	for _, manager := range managers {
		fmt.Println(manager)
	}
}
```

OK，上面代码有问题么？如果你没有立刻发现问题，那以后就要注意了，当然了，这个例子略牵强，一般人可能都不会这么用。我们看输出：

```
<Id:5, Name:Moon, Role:1>
<Id:5, Name:Moon, Role:1>
```

跟预期严重不符，我们有两个Manager，数目是对的，但是内容都变成了最后一个User，这个叫Moon的家伙。大家都是有经验的工程师对不对，所以你冥冥中觉得跟指针有关系。对users循环的时候如果我们加一行打印user地址的语句：

```
for _, user := range users {
	fmt.Printf("%p\n", &user)
	if user.IsManager() {
		managers = append(managers, &user)
	}
}
```

你会发现打印出的5个user的指针都是一样的！啊，那看来遍历过程中所有user都是放在一块内存中的，即临时变量user的内存假设为：`0x2081b8000` 这块内存第一次存放的是Ulric，第二次存放的是Rain ... 最后一次存放的是Moon。append到managers中的是两个一样的地址，都是 `0x2081b8000` 所以打印出如此结果。

据此，大家应该了解了golang中循环变量的机制了，不用谢：）
