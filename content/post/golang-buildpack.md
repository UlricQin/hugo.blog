+++
date = "2013-12-09T19:13:00+08:00"
menu = "main"
tags = ["cloudfoundry", "golang", "buildpack", "paas", "docker"]
title = "史上最简单的buildpack，支持golang"

+++

[Beego-Blog](https://github.com/UlricQin/beego-blog) 是用golang写的，之前部署在CloudFoundry上，所以需要一个支持golang的buildpack。因为golang的项目是静态编译的，所以，我们可以先在自己的机器（要和部署有dea的机器的架构相同，比如都是64位Linux）上编译好，然后把编译之后的产物直接扔给CloudFoundry，这样看来，golang的buildpack什么事情都不用做呀：）

但是buildpack还是要写滴，所以： [cloudfoundry-buildpack-compiled](https://github.com/UlricQin/cloudfoundry-buildpack-compiled) 就应运而生了，遵循buildpack的编写规范，bin下面三个**可执行**脚本，detect和compile啥都不用做，release返回一个空的yaml即可（我准备在cf push的时候指定具体的start脚本）

Beego-Blog是采用beego框架，所以可以使用bee pack来打包，然后把beego-blog.tar.gz解包并且cd进去，执行：

	cf push --buildpack https://github.com/UlricQin/cloudfoundry-buildpack-compiled.git --command ./start_in_paas

PS：显然，如果你的项目也是类似golang这样的，比如nginx，也就同样可以使用这个buildpack来部署