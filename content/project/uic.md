+++
date = "2014-11-16T20:59:21+08:00"
menu = "main"
tags = ["paas", "falcon", "monitor", "dinp"]
title = "PaaS与监控的公共组件UIC简介"

+++

今天给大家介绍一个小组件：User Information Center，下文以UIC代称

# 项目简介

这个组件本来是给我们的监控系统用的，后来稍微扩展了一下，也成了我们PaaS平台的一个组件。那她到底是干啥的呢？主要有两个功能：

- 维护了User与Team的对应关系，即对人做了分组
- 提供了一个简单的sso（即单点登录）功能

简单解释一下：比如对于一个监控系统，主要功能就是收集数据，然后分析处理、报警，报警的通常形式就是发送报警短信、报警邮件了，所以监控系统需要通过某种途径拿到用户的手机号和邮箱地址。

有人说，那直接让用户把手机号、邮箱配置到监控系统里好了，我们说这不是个好方法。因为监控系统中通常会配置很多策略模板，手机号、邮箱可能会出现在不同的模板中，如果一旦修改，要改好多地方。那这个简单，我们不让用户直接配置手机号、邮箱，而是配置用户名，然后系统再根据用户名拿到联系方式，没错，的确应该这么处理。

但是，一个策略模板如果触发阈值发出报警，通常不是发给一个人，而是发给一批人，比如UIC挂了，UIC的研发工程师和运维工程师都应该收到报警，所以通常来讲，策略模板配置的报警接收人应该是一个团队名称。故而我们抽象出了一个Team的概念，其实就是一批人的集合。

冥冥中，我们感觉这个User和Team的对应关系可能不光监控系统需要，所以我们就单独做为一个模块开发了，后来发现，PaaS也需要，因为一个app通常也是由多个人来开发的，这多个人都可以对这个app发起上线、回滚、扩容等操作，所以app的管理者不应该是一个人，而应该是一个Team。

这样一来，我们只需要在一个公共的地方：UIC，维护User与Team的对应关系即可。

再来说说sso功能，UIC之所以提供sso功能，是因为监控系统、PaaS平台通常都是由多个组件协调工作的，那种需要跟用户通过页面交互的组件需要有登陆认证，这显然是sso的应用场景，所以UIC提供了一个很简单的sso机制

# 项目安装

这是一个Java的项目，[GitHub地址](https://github.com/UlricQin/uic) ，安装起来比较简单，搞过Java的人肯定一看就懂，步骤如下：

1、安装JDK，配置JAVA_HOME环境变量

从oracle官网下载或者直接安装openjdk都可以，我们用的JDK1.7，也可以用1.6，代码中没有使用1.7的特性，然后导出JAVA_HOME环境变量，比如

	export JAVA_HOME=/path/to/your/jdk
	export PATH=$JAVA_HOME/bin:$PATH

2、安装Apache Ant

嗯，这个项目没有用maven，小项目我还是喜欢ant，把$ANT_HOME/bin配置到PATH中
ant官网下载地址：http://ant.apache.org/bindownload.cgi

下载解压缩，然后跟JDK类似，配置一下环境变量

	export JAVA_HOME=/path/to/your/jdk
	export ANT_HOME=/path/to/your/ant
	export PATH=$JAVA_HOME/bin:$ANT_HOME/bin:$PATH

3、编译UIC代码

	git clone https://github.com/UlricQin/uic.git
	cd uic
	ant

build.xml是ant用到的编译配置文件，里边用的JDK1.7，如果你是JDK1.6，需要把build.xml中的1.7改成1.6

4、安装web容器

我用的tomcat，你也可以用Jetty、Resin之类的，tomcat7的下载地址：http://tomcat.apache.org/download-70.cgi

java的软件其实蛮好的，下载了解压缩就可以用，tomcat也不例外，比较方便。把刚才编译好的UIC/web目录配置进去即可，contextPath要设置为/

我的做法是：干掉webapps目录的所有内容，修改conf/server.xml

把URIEncoding配置到Connector上，防止乱码，端口也可以根据系统占用情况灵活分配，此处我用的8081

    <Connector port="8081" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" URIEncoding="UTF-8" />

然后去配置文件最下面找到`<Host>`标签，在里边增加`<Context>`标签

	<Context path="" docBase="/path/to/uic/web" />

5、修改UIC配置文件

配置文件的位置是：`/path/to/uic/web/WEB-INF/config.txt`

我们简单解释一下各个配置项的作用：

- jdbcUrl、user、password是数据库的连接配置
- 假如你的DB也打算叫uic，那么`create database uic;`，然后使用`/path/to/uic/scripts/schema.sql`初始化数据库
- memcacheAddrs是用逗号分隔的多个memcached地址
- canRegister取值true或者false，以此控制是否开放注册功能，如果配置成false，不开放注册，需要使用schema.sql中初始化的root账号登陆，然后创建普通用户账号
- token 因为UIC保存了很多用户信息，有些接口不是谁都可以访问的，用token做了一个简单限制，这是一个随机字符串，我们之后介绍的PaaS Builder和Dashboard也都需要配置相同的token才可以

6、成了，启动tomcat

# 二次开发

笔者才疏学浅，不免可能会有一些bug之类的，读者可以帮忙做一些bugfix或者根据自己公司的需求开发一些新feature，我简单介绍一下开发的注意事项

- 项目使用国人开发的一个叫做[JFinal](http://www.jfinal.com/)的框架开发完成，所以，读者要先去了解一下这个框架
- 项目最好使用eclipse for jee版本来开发，即使你习惯用idea，使用JFinal开发项目最好先使用eclipse
- src、conf、frame这三个目录都是source folder，不要仅仅把src作为source folder使用
- 开发阶段lib下面的jetty-server-8.1.8.jar和web/WEB-INF/lib下的所有jar包引入classpath
- 找到`com.ulricqin.uic.config.Config.java`，里边有main方法，启动即可测试
- 使用jetty这种方式修改了代码，JFinal会自动感知并且自动编译加载修改的类，比较方便
- 使用jetty这种方式可能会提示slf4j的某个类发生了多次重复加载，这个问题我一直没弄明白，不过不影响项目测试，线上使用tomcat没有这个问题

O了，UIC就给大家介绍这么多，之后会陆续介绍PaaS的其他组件。

