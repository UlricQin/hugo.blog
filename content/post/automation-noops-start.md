+++
date = "2015-03-04T12:53:29+08:00"
menu = "main"
tags = ["sre"]
title = "自动化运维之路-混沌之初"

+++

首先，这一系列文章只是笔者个人不成熟的一些思路，甚至后面的文章论点会推翻前面的文章论点，因为见识多了，思路就不同了。我只是希望用文字的方式记录下来自己思路的变迁，梳理一些零零散散。过程中如果恰好可以碰撞出你思维的火花，我想那也是极好的。

#### 自动化运维从批量执行开始
ssh、pssh、fabric、ansible……这些常见工具都可以让你批量去一堆机器上执行某些命令或脚本，依赖机器上的sshd

此时，中控机+信任关系是小规模机器运维的利器，几十台机器运维起来问题不大

慢慢的，机器多了，我们可能要对某一批机器修改配置，对某一批机器安装nginx，对某一批机器重启xx服务，检查某一批机器的端口监听，检查某一批机器的系统时间……此时一个新功能提上日程：

#### 机器要有分组
我们可以写个很简单的web系统，姑且叫做HostManager，维护机器的信息和分组信息，包装一个批量执行工具，姑且叫做sshx，传入的参数不再是机器列表了，直接传入机器分组名称，sshx去HostManager获取该分组下的机器，然后去批量ssh执行shell命令

此处要插一句，HostManager维护机器信息，那到底维护哪些机器信息呢？至少应该有个ip，有个ID，HostManager后台应该有个数据库，每录入一台机器就可以自增得到一个ID，还应该有个hostname，嗯

#### 机器应该有个hostname表述自己的用途
公司大了之后可能会有多个机房，会有多个部门、产品线，会有很多Service，这些信息都可以组织到机器hostname中，比如：qd-sa-falcon-redis01.hd，这里的qd表示青岛机房，sa表示运维部，falcon表示监控系统这个产品，redis表示falcon用到的redis机器，01表示索引，多个redis机器就是02，03这样，最后的hd表示华东地区。如此一来，根据机器名基本就可以知道机器的归属和用途了

而且要做到去某个机器执行命令的时候无需非得用ip，也可以用机器名，比如：

```
ssh qd-sa-falcon-redis01.hd uptime
```

要做到这样，需要内网dns的支持，给系统和网络组提需求吧

#### 机器分组命名规范

机器要分组，那分组如何命名呢？笔者举个schema例子：运维组-Project-Module-Other，比如：lbs-search-index-master表示lbs这个运维组，search这个项目，index这个模块，master机器（index还有另一部分slave机器）；也可以不用非得这么长，比如可以建立lbs-search组，放置所有search项目的机器。

这个分组需要运维人员坐在一起讨论出这么个规范

嗯，到此为止，我们增加了机器分组功能，可以更方便的进行批量执行啦~

今天先到这里，下次针对机器分组问题详细聊一聊，冥冥中觉得只是这么命名分组还不够……