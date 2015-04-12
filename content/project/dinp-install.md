+++
date = "2015-03-29T10:10:16+08:00"
menu = "main"
tags = ["dinp", "golang", "paas", "docker"]
title = "DINP安装概述"

+++

挨个组件详细讲述step by step的安装方法比较费劲，之前已经写过UIC和Builder的详细安装方法，其他组件以后补充，先把整个安装过程概述讲解一下，动手能力强的同学应该可以搞得定了

本教程要求您对Docker、Docker Registry、Dockerfile、DINP的理念有一定理解

#### 0. MySQL

UIC和Dashboard依赖MySQL

#### 1. UIC

可以参看之前的文章 [UIC安装](http://ulricqin.com/project/uic/)

#### 2. Docker

需要在所有计算节点，即运行app的机器上面安装Docker，还要在运行Docker registry和DINP Builder的机器上面安装Docker，我们线上跑的是1.3版本

#### 3. Docker registry

这是Docker的一个SCM，存放Image用的，安装方式有多种，大家Google一下吧，最简单的安装方法是：

```
docker run -p 5000:5000 registry
```

当然了，拉取国外的image可能比较慢，大家可以找个国内的镜像用一用

#### 4. DINP Builder

可以参看之前的文章 [DINP Builder安装](http://ulricqin.com/project/dinp-builder/)

#### 5. DINP Dashboard

这是web portal，用户操作入口，也是基于JFinal的Java项目，可以参考UIC的安装方式，代码在github上，oschina、gitcafe也分别有一份代码：

- https://github.com/dinp
- https://gitcafe.com/dinp
- http://www.oschina.net/p/dinp

找找里边dash的那个链接就是了

#### 6. Redis

Router和Server都有对Redis的读写，要提前安装，`yum install -y redis` 或者 `apt-get install -y redis`就可以了，需要root权限。

把/etc/redis.cnf的配置稍微修改一下，`bind 127.0.0.1` 改成 `bind 0.0.0.0` 主要是担心redis与相关组件不在一台机器上

#### 7. DINP Server、Agent、HM、Router

server、agent都是golang的项目，安装方法可以参考Builder，server和agent依赖common这个repo，所以要先clone下来

gorouter也是golang的项目，hm可以先不安装，项目也可以跑

#### 8. 相关配置和启动顺序

大部分配置部分的解释都在各个项目的readme中，启动顺序可以这么安排：

MySQL=>Redis=>UIC=>Docker=>Registry=>Builder=>Dashboard=>Server=>Agent=>Router=>HM

#### 9. 泛域名配置

比如我们得泛域名是*.apps.com，把dns配置到Router的机器上即可，当然了，最好是在Router前面架设LVS，dns配置到LVS

大家还有不明白的地方留言吧



