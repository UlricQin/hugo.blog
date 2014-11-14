+++
date = "2013-12-13T17:05:32+08:00"
menu = "main"
tags = ["cloudfoundry", "buildpack", "paas"]
title = "CloudFoundry中buildpack介绍与自定义实践"

+++

# 背景
用一个问题开篇：一个服务能够run起来，到底需要些什么？

做过部署系统的同学会对这个问题认识比较深，总结一下，我们可以归为如下几类：

- 1、程序本身的代码文件，嗯，这个不用解释
- 2、需要的配置，比如测试环境下有一套配置，开发环境、线上环境各有一套配置，还有甚者，一个idc一套配置
- 3、环境依赖，比如语言环境：Python2.7、JDK6，一些操作系统特性等
- 4、运行时依赖，比如我需要上游某个模块提供的rpc接口的支持，需要用到MQ等等

看起来，要部署个程序还是比较麻烦的嘞，那怎么做才会相对容易一些呢？如果程序最后能把所有依赖的东西打成一个包（比如统一要求是tar.gz格式），扔到任何一个地方去，我们只需要解压，然后使用解压出来的start.sh即可启动执行，那就太棒了！
 
buildpack就是解决这个问题的……
 
# 相关术语
在CloudFoundry中，最后打成的那个tar.gz的包，称为droplet，任何一个DEA拿到这个droplet，解压然后start即可。那具体应该如何打包？打包过程应该是运行一系列的脚本吧，这个脚本在哪里？一般来讲，脚本文件为了备份和版本化需求我们一般会放到git或svn上，嗯，现在可以解释什么叫buildpack了：它是一坨脚本，一般是放到git上作为一个project的形态存在，这坨脚本的作用是把用户写好的程序（CloudFoundry中一般称为app）、及其依赖的环境、配置、启动脚本等打包成droplet，这个过程称之为stage。
 
# 工作流介绍
app开发人员在命令行调用cf push之后，流程是如何走的，经过哪些步骤，在什么时候开始stage，完事之后又经历了什么，可以查看我之前的这篇文章：《[cf push之后到底做了什么？ - CloudFoundry发布app过程](http://ulricqin.com/post/cloudfoundry-push-process/)》
 
# buildpack目录结构
buildpack是工作在CloudFoundry这个大框架下的，我们知道，但凡工作在框架下的东西就要尊从一些规范，这是框架和你交互协调的根基。buildpack也不例外，CloudFoundry要求，buildpack至少含有一个bin目录，bin目录下有三个文件，文件名固定，分别是：

- detect # 这个文件的作用是侦测你的项目，比如是个Java项目 or php项目，用的什么Runtime和Framework之类的
- compile # 这是buildpack的核心文件，一般作用就是去拉取相应的Runtime（e.g. python2.7/ruby1.9.3）下来，做一下配置放到指定位置，拉取相应的Framework（e.g. Flask/Django）下来，做一下配置，放到指定位置
- release # 这个文件最终要求输出一个yaml，来描述如何启动app之类的
三个脚本由Cloudfoundry顺次执行
 
如果编写buildpack过程需要创建其他文件夹或文件，其位置怎么安排都可以，CloudFoundry框架方面没有什么限制
 
# 自定义buildpack实例
下面我们自定义一个buildpack来支持java web项目，使用JDK7和tomcat7来跑，jdk和tomcat都要从公司内网来下载，版本可以稍微放宽，支持jdk6、7、8，tomcat也可以支持多个版本
detect内容大致如下：

	#!/usr/bin/env bash  
	# cf 会给detect脚本传入一个参数，即build dir，里边就是那一坨app散文件  
	BUILD_DIR=$1  
	# 既然是java web的项目，而且跑在tomcat上，我们没别的要求，只要web.xml位于规范位置即可  
	if [ -d "${BUILD_DIR}/WEB-INF" ] && [ -f "${BUILD_DIR}/WEB-INF/web.xml" ]; then  
	# 按照惯例，一般会把所侦测到的项目类型echo出来，  
	# 这个echo出来的字符串和compile、release脚本没有半毛钱关系  
	echo "JavaWeb"  
	# 返回0表示detect成功执行，cf会继续执行compile，返回非0值cf就不继续往下走了  
	exit 0  
	fi  
	echo "no"  
	exit 1  
 
compile文件做的事情稍微复杂一些，如果觉得bash不能胜任，随便用什么脚本语言来搞都OK，cf为compile脚本传入了两个参数：build dir和cache dir，build dir与传给detect文件的内容一致，cache dir是一个临时目录，比如我们在compile过程中下载jdk和tomcat之类的，就可以用cache dir做个中转。

	#! /usr/bin/env ruby  
	# detect、compile、release这几个文件都没有后缀，用你喜欢的语言就好，这里是用的ruby  
	$stdout.sync = true  
	# 自己创建了一个lib目录，塞入环境变量，  
	# 如之前所述，除了bin目录必须固定之外，其他文件（夹）随你喜欢来安排  
	$:.unshift File.expand_path('../../lib', __FILE__)  
	require 'global'  
	require 'main_pack'  
	require 'fileutils'  
	  
	build_path = ARGV[0]  
	cache_path = ARGV[1]  
	FileUtils.mkdir_p(build_path)  
	FileUtils.mkdir_p(cache_path)  
	# 主要逻辑都放在MainPack里了，无非就是下载、配置  
	pack = MainPack.new(Global.new(build_path, cache_path))  
	pack.compile  
 

注意：

- 1、为了提高下载速度，免受网络困扰，同时增加掌控力度，我们一般会把依赖的tar包放到内网某个位置去自己管理
- 2、build_dir/.profile.d/*.sh文件都会在运行app之前由CloudFoundry帮我们提前运行，那在此导出一些环境变量之类的就比较方便了
 
release文件的内容大体为：

	#!/usr/bin/env ruby  
	  
	require 'yaml'  
	  
	yml = {  
	'addons' => [],  
	'config_vars' => {},  
	'default_process_types' => {  
	'web' => './bin/catalina.sh run'  
	}  
	}.to_yaml  
	  
	puts yml  
 
 
default_process_types字段下面的web字段的值就是告诉CloudFoundry如何启动这个app，当然，我们可以在push应用的时候覆盖这个配置，稍候介绍~
 
# 如何使用自定义buildpack
这个很easy，进入app所在的目录，push即可：

	cd /path/to/your/app  
	cf push --buildpack git://git.xiaomi.com/cf/cf-bp-javaweb-tomcat --command ./start.sh  
 
 --buildpack就是用来指定自定义buildpack地址的，--command是用来指定自定义启动命令的，如果你的启动脚本是在app根目录下的start.sh，那就可以用上面的这条命令来push应用
 
# 坑
- 1、bin目录下的detect、compile、release三个文件要求本身有可执行权限，cf不会给你chmod +x，所以我们需要在放到git中的时候就要求有可执行权限
- 2、stage的时候，cf把我们的app文件放到了这里：/tmp/staged/app，运行时放到了这里：/home/vcap/app，是的，你没看错，二者目录不同，这就需要注意了，特别是安装一些lib组件的时候，当时为了支持python的buildpack就搞的比较恶心，需要安装setuptools、pip和gunicorn，最后是放在了程序启动前来做的，而不是compile阶段
- 3、在支持嵌入式jetty程序的时候，如果你用了WebAppContext可能也会有问题，没有任何提示信息，就是死活起不来，后来追查发现jetty会为WebAppContext自动创建一个临时目录，猜测可能是没有创建成功，至于为啥没成功一直没有追出来，最后的方案是在程序启动之前提前创建好这个临时目录：mkdir -p /home/vcap/app/work，你如果搞出来了来留言分享一下吧 :)
- 4、如果你的CloudFoundry跟我们一样没有采用bosh管理，很可能rootfs里缺少一些unzip之类的命令，这个也会使stage失败，提前把一些常用的工具命令装到rootfs里是个明智的选择
 
还有一篇文章也非常推荐看一下：《[史上最简单的buildpack，支持golang](http://ulricqin.com/post/golang-buildpack/)》
 
# 结语
总体来看，buildpack帮助paas平台完成了一个app在部署层面的抽象，主要搞定app依赖的runtime和framework，相当于搞定了静态依赖。操作系统特性通过统一定制rootfs来搞，不在buildpack问题域，配置文件差异问题，buildpack认为一个app一套配置，上层怎么处理，开发人员自己说了算，可以搭建多套CloudFoundry或者创建不同的app：dev-app/prod-app之类；运行时依赖也需要开发人员自己搞定，提前确认好相应的依赖是否已经在run，版本是否OK之类。
 
有任何问题欢迎留言讨论，谢谢支持 :)
