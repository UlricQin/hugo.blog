+++
date = "2013-11-20T16:57:07+08:00"
menu = "main"
tags = ["cloudfoundry", "paas"]
title = "cf push之后到底做了什么？ - CloudFoundry发布app过程"

+++

![](/img/cf-push-process.png)

上面的图片是从CloudFoundry官方文档中拿到的，整个过程如下：
 
- 1、用户在命令行下进入自己的app所在的目录，运行cf push，这表示说：我要上传应用了
- 2、cf命令行工具发现用户给的指令是push，于是发请求给CCNG，说：我要创建一个新应用
- 3、CCNG管辖了两个存储，一个是CCDB（是一个RDBMS，可以用mysql），另一个是BlobStore，存储一些大的二进制文件，比如用户要push的app和最后打包成的droplet。CCNG会先保存当前app的meta信息，比如名称，subdomain，实例数，内存限制等等
- 4、cf命令行工具把用户app所在目录下的所有文件（除去.cfignore中配置的文件）上传给CCNG
- 5、CCNG把用户的app文件打包存放在BlobStore中
- 6、cf命令行工具发起start指令
- 7、CCNG接收到cf命令行给的start指令，但是此时用户的app文件仅仅包含了应用逻辑代码，没有运行时环境的支持，没有启停脚本等，是没法直接run的。所以CCNG现在开始做一件很重要的事情：stage the application，所谓的stage，就是把用户的app文件和运行时依赖（javaweb的项目的话，比如jdk和tomcat）以及启停脚本一起打包的过程，stage的产物，即打包之后的那个包，称为droplet。打包这个动作本身是在DEA中完成的，所以此时CCNG选取一个合适的DEA，命令它来干活。
- 8、某个悲惨的DEA接收到stage 请求，于是开始苦逼哈哈的干活了，过程中可能会出问题，所以，stage的时候的输出需要同步显示给终端用户，方便排错
- 9、打包完成之后DEA需要把droplet上传到CCNG的BlobStore中存放
- 10、报告给CCNG说我完成stage了
- 11、CCNG读取meta信息看用户想部署几个实例以及内存要求等，然后选取相应的DEA去部署droplet
- 12、DEA是否部署和启动成功，需要汇报给CCNG，并最终反映到终端用户控制台上
 
整个过程在文档中有详尽描述，我只是翻译了一下，地址在 [这里](http://docs.cloudfoundry.org/docs/running/architecture/how-applications-are-staged.html)

