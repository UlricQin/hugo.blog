+++
date = "2015-05-07T10:09:12+08:00"
menu = "main"
tags = ["ops-updater", "golang"]
title = "管理升级机器上各种agent的一个小工具"

+++

# 背景

我们的监控系统[open-falcon](https://github.com/open-falcon)开源在即，需要在所有机器上部署falcon-agent采集机器硬件信息。我们内部使用一个自研工具frigga来管理各种agent，包括监控的agent、部署的agent、naming的agent等等，这个工具是ruby的，部署起来不是很方便，笔者最近花了一点时间用Go写了一个简化版的frigga，静态编译部署起来方便，专门用于管理升级监控agent，随着open-falcon的发布一并发布，大家可以尝试一下，超简单。

# 组件

工具由两部分组成

- ops-updater: 部署在所有机器上，一般在装机的时候安装，功能简单，从不用升级，Go写的，静态编译，无任何lib依赖
- ops-meta: 服务端，ops-updater定期与ops-meta通信，询问本机应该部署哪几个agent，版本分别是多少，然后做相应动作，完成ops-meta的指示

# 理念

- 对于一个公司而言，agent并不多，也就有个监控agent、部署agent、naming agent，所以ops-meta直接采用配置文件而不是数据库之类的大型存储来存放agent信息
- 公司级别agent升级慢一点没关系，比如一晚上升级完问题都不大，所以ops-updater与ops-meta的通信周期默认是5min，比较长。如果做成长连接，周期调小，是否就可以不光用来部署agent，也可以部署一些业务程序？不要这么做！部署其他业务组件是部署agent的责任，ops-updater做的事情少才不容易出错。ops-updater推荐在装机的时候直接安装好，功能少基本不升级。
- 配置文件中针对各个agent有个default配置，有个others配置，这个others配置是为了解决小流量问题，对于某些前缀的机器可以采用与default不同的配置，也就间接解决了小流量测试问题
- ops-updater会汇报自己管理的各个agent的状态、版本号，这个信息直接存放在ops-meta模块的内存中，因为数据量真没多少，即使有100w机器，3个agent也没多少数据
- ops-updater启动之后应该随机sleep一段时间再与ops-meta周期性通信，避免大量ops-updater同时与ops-meta通信造成压力

# 代码

这个小组件没有放到github上，而是在国内的gitcafe，感觉用着还不错，赞一个

- ops-common: https://gitcafe.com/ops/common
- ops-updater: https://gitcafe.com/ops/updater
- ops-meta: https://gitcafe.com/ops/meta

# 编译

这是个Go语言写的项目，故而我们要先搭建Go的编译环境

```
cd ~
wget http://dinp.qiniudn.com/go1.4.1.linux-amd64.tar.gz
tar zxvf go1.4.1.linux-amd64.tar.gz
mkdir -p workspace/src
echo "" >> .bashrc
echo 'export GOROOT=$HOME/go' >> .bashrc
echo 'export GOPATH=$HOME/workspace' >> .bashrc
echo 'export PATH=$GOROOT/bin:$GOPATH/bin:$PATH' >> .bashrc
echo "" >> .bashrc
source .bashrc
```

接下来clone代码编译：

```
cd $GOPATH
mkdir -p src/gitcafe.com/ops
cd src/gitcafe.com/ops
git clone git://gitcafe.com/ops/common.git
git clone git://gitcafe.com/ops/updater.git
git clone git://gitcafe.com/ops/meta.git
cd updater
go get ./...
./control pack
cd ../meta
go get ./...
./control pack
```

上面做完之后，updater目录和meta目录分别产出一个tarball，拿着这货去部署即可。注意：上面用到的Go语言环境是linux-amd64，我也只在64位Linux下测试使用过，其他平台需要您自己搞定哈

# 部署ops-meta

我们姑且就拿编译机来部署ops-meta，作为咱们的server

```
cd ~
mkdir -p ops
tar zxvf workspace/src/gitcafe.com/ops/meta/ops-meta-0.0.2.tar.gz -C ops
cd ops
mv cfg.example.json cfg.json
# modify cfg.json
./control start
```

这样ops-meta就启动了，启动之后这个组件监听在2000端口，`./control status`可以看到进程状态，或者通过`ss -tln|grep 2000`查看端口是否在监听

# 配置ops-meta

写这个工具的本意是为了批量部署管理我们的falcon-agent，我提前准备好了一个http://7xiumq.com1.z0.glb.clouddn.com/open-falcon-agent-demo-3.1.3.tar.gz ，咱们配置一下ops-meta，来把这个demo版本的falcon-agent管理起来

修改后的cfg.json如下：

```
{
    "debug": true,
    "tarballDir": "./tarball",
    "http": {
        "enabled": true,
        "listen": "0.0.0.0:2000"
    },
    "agents": [
        {
            "default": {
                "name": "falcon-agent",
                "version": "3.1.3",
                "tarball": "http://127.0.0.1:2000/falcon",
                "md5": "",
                "cmd": "start"
            }
        }
    ]
}
```

这里我们没有使用others配置，无需小流量，ops-meta同时充当了http download server的角色，把falcon-agent放到指定位置

```
cd ~/ops
mkdir -p tarball/falcon
cd tarball/falcon
wget http://7xiumq.com1.z0.glb.clouddn.com/open-falcon-agent-demo-3.1.3.tar.gz -O falcon-agent-3.1.3.tar.gz
md5sum falcon-agent-3.1.3.tar.gz > falcon-agent-3.1.3.tar.gz.md5
```

OK，做完以上工作之后ops-meta就准备好了，她托管了一个falcon-agent，静待ops-updater来下载

# 部署ops-updater

ops-updater理应部署在公司所有机器上，这里做测试，也在当前这个机器部署，咱们把它部署在`~/agents`目录，拉取下来的falcon-agent也会出现在这个目录，具体目录组织结构可以看一下我们的README

```
cd ~
mkdir -p agents
tar zxvf workspace/src/gitcafe.com/ops/updater/ops-updater-0.0.1.tar.gz -C agents
cd agents
mv cfg.example.json cfg.json
# modify cfg.json
./control start
```

这两个组件的部署都比较简单，再次赞一下Go的静态编译，检查方式与ops-meta一样，`./control status`或者`ss -tln|grep 2001`，ops-updater监听在2001端口，updater为啥要监听一个http端口呢？纯粹为了调试，为了健康检查，如果担心安全问题可以在配置文件中配置为enabled: false

# 配置ops-updater

配置基本可以维持默认，不过测试阶段，可以把interval调得小一点，比如：3，这个是ops-updater每隔多久请求一次ops-meta，单位是秒，先上如果机器量特别大，这个间隔就要调大一点了。

稍等一会，你就可以在`~/agents`目录中看到falcon-agent目录了，访问一下http://127.0.0.1:1988/ 看到本机Dashboard了么？看到了就说明falcon-agent已经被ops-updater部署成功了：）

# 不传之秘

*如何查看各个机器上的agent部署情况呢？*

ops-updater会把管理的agent的版本上报给ops-meta，ops-meta提供了一个http接口查看，比如我们要查看falcon-agent的部署情况

```
# 使用文本方式返回
curl http://127.0.0.1:2000/status/text/falcon-agent
# 使用json返回
curl http://127.0.0.1:2000/status/json/falcon-agent
```

就这么多了，有问题可以加入我们的QQ群：373249123讨论，我会给解答滴：）

# 补充

这个项目只有两个模块，均是用Go语言编写的，我录制了一个视频放在极客学院讲解这个项目是如何一步一步做出来的，推荐Go语言学习者看一下

- http://www.jikexueyuan.com/course/1336.html
- http://www.jikexueyuan.com/course/1357.html
- http://www.jikexueyuan.com/course/1462.html
- http://www.jikexueyuan.com/course/1490.html

如果从中学到了知识，小小打赏一下吧：）



