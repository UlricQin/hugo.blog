+++
date = "2014-04-16T18:06:36+08:00"
menu = "main"
tags = ["golang", "falcon"]
title = "Linux单机监控工具 - Falcon-eye"

+++

# 简介

* 这是我们团队正在写的监控系统的一部分，之后说不定会把整个系统开源出来
* 这是一个用golang写的小工具，没有任何部署依赖
* 这只是一个采集linux基础数据并做简单展示的agent，不会报警的哦

# 可以采集哪些数据？

* 机器基本数据，比如kernel version，uptime，hostname等等
* cpu使用情况：比如idle、user、nice、system、iowait、irq、softirq、steal、guest的当前占比
* memory使用情况，used了多少，free是多少，total是多少
* 当前loadavg是多少
* 磁盘占用情况，各个分区、设备的使用情况；以及磁盘io的情况，类似iostat的数据，比如await/svctm/%util等等
* 网络使用情况，比如各个网卡当前带宽情况、每秒丢包多少

# 长什么样？
![](http://beego-blog.qiniudn.com/static/uploads/editor/1397659646.png)
![](http://beego-blog.qiniudn.com/static/uploads/editor/1397659659.png)

# 怎么部署？

项目本身分三部分：

* goutil：是一个go的工具箱，都是些常用的方法类，没啥可说的
* falcon：一些列采集函数
* falcon-eye：利用falcon中的采集函数采集数据做展示

看看代码中import就知道喽，很简单的
so，只要找个机器下载一个golang的语言包，编译一下就行了，在项目（ https://github.com/UlricQin/falcon-eye ）的readme中有相关命令

#### 可以用它干什么？
* 可以部署到各个单机，每次报警了之后打开这个页面看看各项指标
* 可以改造它让它支持更多数据采集函数，展示你关心的数据
* 可以写一个后端server，给falcon-eye加一个push功能，每隔几秒钟采集数据push给后台server，在server做报警和图表展示
