+++
date = "2013-12-01T19:22:02+08:00"
menu = "main"
tags = ["cloudfoundry", "golang", "buildpack", "paas", "docker"]
title = "CloudFoundry中各个组件的作用"

+++

CloudFoundry是一个标杆性的项目，架构设计上有很多值得借鉴之处。从CloudFoundry官网摘了一张图，我们以此剖析各个组件的作用。
![CloudFoundry](http://beego-blog.qiniudn.com/static/uploads/editor/1415094048.png)

# Router

Router是整个平台的流量入口，负责分发所有的请求到对应的组件，包括来自外部用户对app的请求和平台内部的管理请求。

Router是PaaS平台中至关重要的一个组件，它在内存中维护了一张路由表，记录了域名与实例的对应关系，所谓的实例自动迁移，靠得就是这张路由表，某实例宕掉了，就从路由表中剔除，新实例创建了，就加入路由表。

CloudFoundry1.0中的router是用nginx+lua嵌入脚本实现的，2.0用golang重写，更名为gorouter，性能有所提升，并声称试图解决websocket请求和tcp请求（虽然这在笔者看来是没用的），它的代码在https://github.com/cloudfoundry/gorouter ，大家可以研究一下。

# Authentication

这块包含两个组件，一个是Login Server，负责登录，一个是OAuth2 Server(UAA)，UAA是个Java的项目，如果想找一个OAuth2开源方案，可以尝试一下UAA

# Cloud Controller

Cloud Controller负责管理app的整个生命周期。用户通过命令行工具cf与CloudFoundry Server打交道，实际主要就是和Cloud Controller交互。

用户把app push给Cloud Controller，Cloud Controller将其存放在Blob Store，在数据库中为该app创建一条记录，存放其meta信息，并且指定一个DEA节点来完成打包动作，产出一个droplet（是一个包含Runtime的包，在任何dea节点都可以通过warden run起来），完成打包之后，droplet回传给Cloud Controller，仍然存放在Blob Store，然后Cloud Controller根据用户要求的实例数目，调度相应的DEA节点部署运行该droplet。另外，Cloud Controller还维护了用户组织关系org、space，以及服务、服务实例等等。

# Health Manager

Health Manager最初是用Ruby写的，后来用golang写了一版，称为HM9000，HM9000主要有四个核心功能：

   * 监控app的实际运行状态（比如：running, stopped, crashed等等），版本，实例数目等信息。DEA会持续发送心跳包，汇报它所管辖的实例信息，如果某个实例挂了，会立马发送“droplet.exited”消息，HM9000据此更新app的实际运行数据
   * HM9000通过dump Cloud Controller数据库的方式，获取app的期望状态、版本、实例数目
   * HM9000持续比对app的实际运行状态和期望状态，如果发现app正在运行的实例数目少于要求的实例数目，就发命令给Cloud Controller，要求启动相应数目的实例。HM9000本身，不会要求DEA做些什么。它只是收集数据，比对，再收集数据，再比对
   * 用户通过cf命令行工具是可以控制app各个实例的启停状态的，如果app的状态发生变化，HM9000就会命令Cloud Controller做出相应调整

说到底，HM9000就是保证app可用性的一个基础组件，app运行时超过了分配的quota，或者异常退出，或者DEA节点整个宕机，HM9000都会检测到，然后命令Cloud Controller做实例迁移。HM9000的代码在这里：https://github.com/cloudfoundry/hm9000 ，有兴趣的同学可以研究一下

# Application Execution(DEA)

DEA，即Droplet Execution Agent，部署在所有物理节点上，管理app实例，将状态信息广播出去。比如我们创建一个app，实例的创建命令最终会下发到DEA，DEA调用warden的接口创建container，如果用户要删除某个app，实例的销毁命令最终也会下发到DEA，DEA调用warden的接口销毁对应的container。

当CloudFoundry刚刚推出的时候，Droplet包含了应用的启动、停止等简单命令。用户应用可以随意访问文件系统，也可以在内网畅通无阻，跑满CPU，占尽内存，写满磁盘。你一切可以想到的破坏性操作都可以做到，太可怕了。CloudFoundry显然不会放任这样的情况太久，现在他们开发出了Warden，一个程序运行容器。这个容器提供了一个孤立的环境，Droplet只可以获得受限的CPU，内存，磁盘访问权限，网络权限，再没有办法搞破坏了。

Warden在Linux上的实现是将Linux内核的资源分成若干个namespace加以区分，底层的机制是CGROUP。这样的设计比虚拟机性能好，启动快，也能够获得足够的安全性。在网络方面，每一个Warden实例有一个虚拟网络接口，每个接口有一个IP，而DEA内有一个子网，这些网络接口就连在这个子网上。安全可以通过iptables来保证。在磁盘方面，每个warden实例有一个自己的filesystem。这些filesystem使用aufs实现的。Aufs可以共享warden之间的只读内容，区分只写的内容，提高了磁盘空间的利用率。因为aufs只能在固定大小的文件上读写，所以磁盘也没有出现写满的可能性。

LXC是另一个Linux Container。那为什么不使用它，而开发了Warden呢。因为LXC的实现是和Linux绑死的，CloudFoundry希望warden能运转在各个不同的平台，而不只是Linux。另外Warden提供了一个Daemon和若干Api来操作，LXC提供的是系统工具。还有最重要的一点是LXC过于庞大，Warden只需要其中的一点点功能就可以了，更少的代码便于调试。

# Service Brokers

app在运行的时候通常需要依赖外部的一些服务，比如数据库服务、缓存服务、短信邮件服务等等。Service Broker就是app接入服务的一种方式。比如我们要接入MySQL服务，只要实现CloudFoundry要求的Service Broker API即可。但实际情况是在我们使用CloudFoundry之前，MySQL服务已经由DBA做了服务化、产品化，用起来已经很方便了。有必要实现其Service Broker API，按照CloudFoundry这套规则出牌么？笔者认为没有这个必要。app仍然按照之前访问MySQL服务的方式去做即可，没有任何问题。

# Message Bus

CloudFoundry使用NATS作为内部组件之间通信的媒介，NATS是一个轻量级的基于pub-sub机制的分布式消息队列系统，是整个系统可以松散耦合的基石。

我们以向router注册路由为例来说明NATS的作用。不管是外部用户对平台上的应用发起的请求，还是对内部组件（比如Cloud Controller、UAA）发起的请求，都是经由router做的转发，要能让router转发则首先需要向router注册路由。大体逻辑实现如下：

   * router启动时，会订阅router.register这个channel，同时也会定时的向router.start这个channel发送数据
   * 其他需要向router注册的组件，启动时会订阅router.start这个channel。一旦接收到消息，会立刻收集需要注册的信息（如ip、port等），然后向router.register这个channel发送消息。
   * router接收到router.register消息后立即更新路由信息
   * 以上过程不停循环，使router的状态时刻保持最新

# Logging and Statistics

Metrics Collector会从各个模块收集监控数据，运维工程师可以据此来监控CloudFoundry，出了问题及时发现并处理。物理机的硬件监控则可以采用传统的一些监控系统来做，比如zabbix之类的。

Log这块是个大话题，CloudFoundry提供了Log Aggregator来收集app的log。我们也可以通过其他手段直接把log通过网络打出来，比如syslog、scribe之类的。

# 参考资料

- 《CloudFoundry社区文档》 http://docs.cloudfoundry.org/
- 《limengyun’s blog》 http://limengyun.com/
- 《新版CloudFoundry揭秘》 http://qing.blog.sina.com.cn/2294942122/88ca09aa33001753.html

