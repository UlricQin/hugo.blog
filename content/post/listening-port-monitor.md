+++
date = "2012-10-21T18:19:22+08:00"
menu = "main"
tags = ["linux", "monitor", "falcon"]
title = "获取当前机器正在Listen的端口"

+++

# 需求来源
我们正在编写一个监控系统，在每个机器上会部署一个监控的agent，这个agent需要获取当前机器所有正在listen的端口汇报给server，每30s汇报一次

# 常见做法
你的第一时间反应可能是netstat命令，但是在连接数比较多（比如达到几十万）的机器上，运行netstat根本跑不出结果。
那么使用nmap？注意我的需求，采集当前机器上面所有正在Listen的端口，那用nmap的话可能需要从1~65535这么扫描一遍，发syn包？这样会造成很多半开连接，直接创建tcp连接并最终关闭？也会耗费太多资源，而且我还需要每30s汇报一次！ - 在机器本身负载很重的情况下：（

# 我认知中的最好方案
读取/proc/net/{tcp,tcp6,udp,udp6}这四个文件，其实zabbix就是这么干的，通过匹配字符串的方式可以直接拿到所有Listening的端口

# /proc/net/tcp简介
cat /proc/net/tcp | less 可以看到有个st字段，即状态字段，它的值和对应的解释：

	00  "ERROR_STATUS",
	01  "TCP_ESTABLISHED",
	02  "TCP_SYN_SENT",
	03  "TCP_SYN_RECV",
	04  "TCP_FIN_WAIT1",
	05  "TCP_FIN_WAIT2",
	06  "TCP_TIME_WAIT",
	07  "TCP_CLOSE",
	08  "TCP_CLOSE_WAIT",
	09  "TCP_LAST_ACK",
	0A  "TCP_LISTEN",
	0B  "TCP_CLOSING",

嗯，看到0A的解释了吧，TCP_LISTEN，找的就是它！
通常的做法就是逐行匹配： `00000000:0000 0A` ，而local_address一栏冒号后面的部分就是端口号，采用16进制表示

# 注意
了解到以上信息，那我们是不是就可以逐行匹配获取了呢？理论上这样是可以的，但是……这个文件非常大，逐行匹配会非常耗费cpu资源，我们当时的情况，是差点吃掉一个核，影响了线上业务！怎么办呢？再告诉你一个秘密： **/proc/net/tcp这个文件会把Listen状态的连接放在文件开头部分** （一般人我不告诉他）。so：我们顺次读取匹配，第一行标题过滤掉，匹配一行获取一个port，一旦匹配不上了，也就直接break出for循环即可

# 参考代码
https://github.com/UlricQin/falcon/blob/master/collector/portstat.go

# 更新
上面所述版本上线之后发现，有的机器/proc/net/tcp文件中的内容并不是0A状态的在文件最上面，中间可能夹杂着03和01，这就非常麻烦了。后来我们放弃了这种读取/proc/net/tcp的方式，直接使用ss -n -l命令搞定来的：）



