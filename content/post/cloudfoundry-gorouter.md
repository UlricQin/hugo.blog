+++
date = "2013-12-03T17:19:04+08:00"
menu = "main"
tags = ["cloudfoundry", "paas", "router"]
title = "CloudFoundry中gorouter深入解读"

+++

# 导读
首先，所谓的深入解读比较标题党了哈：）只是给大家分享一下我的理解，希望共同进步

我们以一个问题开篇，主要介绍代码结构、接口交互、主要逻辑，最后稍微总结一下，希望能把你讲明白：）
 
# 背景问题
思考这么一个问题：paas是多租户的，每个app都希望有自己的域名，比如miui.com、miliao.com、xiaomi.com，为了提高可用性，每个app一般都会有多个instance。OK，想象一下，一个http request过来了，请求的是miui.com（至于怎么过来的CloudFoundry是不管的，自己可以做个cname之类的），CloudFoundry应该如何处理呢？
 
- 针对这个域名的请求最终应该打到某个instance上，so，应该有个地方来保存路由信息，可不能让miui.com的请求打到miliao.com的instance上去
- 既然有个路由表，那某个instance挂掉的时候或者扩容新增instance的时候，希望能够动态更新这个路由表
- http协议本身是无状态的，如果一会请求instance a，一会请求instance b，就会存在session一致性问题，我们可以用一个简单的策略解决，达到这样的效果：上次这个用户请求打到哪个instance我们记录下来，下次仍使其打到这个instance上；第一次请求就随便喽，不过要有负载均衡的效果
- app不光是http协议提供服务的，还有websocket的、tcp的呢，怎么整？

OK，gorouter就是来解决以上问题的，他保存了一张路由表在内存中，他来做转发，所有的请求都从router节点走，所以性能是个需要注意的问题，cf1.0中使用nginx+lua+ruby来搞效果不是很好，当然，还有一个更普遍的问题，可用性问题：） so，一般来讲，在CloudFoundry平台中，gorouter是部署多个的，前面加一层LVS
 
# 代码结构
gorouter的代码地址：https://github.com/cloudfoundry/gorouter ，分了不少目录，我们简述一下各个目录的功能，可以认为是代码模块的划分了
 
- common 通用代码，比如获取本机IP：LocalIP，生成UUID：GenerateUUID，通用组件struct定义：VcapComponent（gorouter设计成一个Component），/healthz接口返回的内容，http权限校验逻辑等等
- config 故名思议，就是处理配置的，通过这个package可以读取gorouter的配置文件
- example_config 里面只是一个配置文件，样例，yaml格式的
log 这个package是用来打印log的
- proxy 最重要的package，gorouter作为路由节点主要就是干proxy这个事情的，里边定义了如何处理http请求，如何处理websocket请求，如何处理tcp请求
- register 各个app来更新路由表的时候通过这个这个package中的Register和UnRegister方法处理
- route 这个package里是一些数据结构，存储路由表相关信息
- router main函数所在的位置
- stats 保存了一些状态信息，貌似是存储了访问量比较大的app列表，还定义了一个数据结构heap，没仔细看
- test目录没看，谁让他起这么个名字呢
- util 工具性package，只有一个方法，写pid文件
- varz 统计状态并存放到相应数据结构里，比如多少BadRequest，多少BadGatway之类的，/varz接口 会返回这里保存的信息

# 接口交互
gorouter提供了三个http接口，可以这么调用查看：
 
	# 查看路由表信息的  
	curl http://router:routerPass@routerIp:routerPort/routes  
	# 健康检查接口  
	curl http://router:routerPass@routerIp:routerPort/healthz  
	# 状态信息查看接口  
	curl http://router:routerPass@routerIp:routerPort/varz  
 
 
上面的router:routerPass分别是在配置文件中配置的user和pass，routerPort就是你配置的port，默认是8082，这个端口提供的接口都是管理性质的接口

gorouter还会同时启动一个http server来监听80端口，处理终端用户的访问请求，最新版本的gorouter直接使用了golang内置的net/http，之前版本还自己定义了蛮多东西
 
# 主要逻辑
CloudFoundry内部组件之间通信主要通过NATS来完成，NATS是一个轻量级的基于pub-sub机制的分布式消息队列系统，它负责衔接各组件。
gorouter启动的时候，会往router.start这个channel发送一条消息，之后就是周期性发送了，周期时间由配置项 publish_start_message_interval 指定，发送的消息内容是json格式，定义如下：
 
	type RouterStart struct {  
	     Id                               string   `json:"id"`  
	     Hosts                            []string `json:"hosts"`  
	     MinimumRegisterIntervalInSeconds int      `json:"minimumRegisterIntervalInSeconds"`  
	}  
 
 
表示说：hi，各位同仁，我是gorouter，我启动了，我的地址是xx，你们有人要注册路由么？如果有需要的话每隔一段时间要给我发一条消息哈
 
gorouter会订阅router.register，这是各个组件注册路由信息的channel
还会订阅router.unregister，如果有组件想unregister，就发消息到这个channel；另外，如果一个组件长时间（droplet_stale_threshold）不发消息给router.register，router也会把对应的instance从路由表摘掉，router会经常性来做测试，测试周期是：prune_stale_droplets_interval
 
gorouter已经启动运行一段时间了，一个新的组件加入了，比如dea，他不知道gorouter的地址，怎么办？dea可以给router.greet这个channel发消息，gorouter会订阅这个channel，收到消息之后router会回复一个消息，消息内容跟发往router.start的内容一样，这样dea就获取到gorouter的地址了
 
OK，通过以上通信，router填充好了本机的路由表信息，开工，接收外部请求：）
router会监控80端口，http协议，一个request过来，router的主要处理逻辑：

- 1、找到对应的instance，之前提到的session一致性问题就是在这里解决的，cookie中如果有之前的instanceId，就用对应的这个instance，否则就从app对应的Endpoint（保存了instance相关信息）池中随机选取一个
- 2、判断是否是TcpUpgrade，判断依据：http header中Connection字段的value是upgreade，Upgrade字段的value是tcp，如果是，就用tcp的方式来handle，否则继续
- 3、判断是否是WebSocketUpgrade，判断依据：http header中Connection字段的value是upgreade，Upgrade字段的value是websocket，如果是，就用websocket的方式来handle，否则继续
- 4、走到这，说明是个纯粹的http请求，handle it即可
 
总结一下，起一个web server来监听80端口，拿到一个http request，判断是否是upgrade，如果是就升级为websocket连接或tcp连接，否则就是纯粹的http连接。如果读者了解websocket，对这种处理方式应该感觉很亲切，但是这个所谓的tcp方式有用么？后端rpc接口根本没法用这种方式暴露……反正目前我是没看到有啥用处哈，纯粹就是为了扩展，防止一个类似websocket这种协议出现吧
 
# 总结
- 1、用NATS这种通信方式来链接各个组件，非常松耦合，因为路由表信息没必要严格做到数据一致性，这样一来，router很轻易的部署多个实例，大家都监听相同的channel就都可以得到路由信息了，很方便
- 2、新版本扩展性更好，所有依赖于http协议的通信方式都可以解决，比如websocket
- 3、后端rpc模块如何提供服务目前没法解决，感觉可以仅仅把router里的路由信息作为name service来用，业务代码从此处获取原始接口提供方，即多个rpc instance，然后程序内部自己failover，自己去call。没有想到好方法，如果你有的话欢迎交流哦：）
 
gorouter作为CloudFoundry平台的入口，承载了所有的流量，虽然是个很重要的模块，不过逻辑比较简单，先写这么多，有问题可以留言讨论~：）
 
# 参考资料
- [Cloud Foundry中gorouter源码分析](http://blog.csdn.net/shlazww/article/details/11974411)
- [gorouter源码地址](https://github.com/cloudfoundry/gorouter)
- [NATS](http://limengyun.com/cloudfoundry/nats.html)
