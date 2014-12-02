+++
date = "2014-12-02T13:01:04+08:00"
menu = "main"
tags = ["nux", "golang", "linux"]
title = "使用golang解析/proc/pid/cmdline"

+++

# 背景起因

最近在重构我们监控系统的Agent，使用golang写的。我们要做进程监控，所以要收集机器上的进程信息，用户在我们监控的portal上面配置的时候通常通过两个tag来指定具体的进程，一个是name，一个是cmdline

- name是从/proc/pid/status文件中获取的，第一行就是，这个没啥好说的
- cmdline是从/proc/pid/cmdline文件中获取的，cat一下，你会发现其中的内容并非完全与你的启动命令一致，空格没了！

# /proc/pid/cmdline

比如你要启动某个进程，使用这样一个命令：

	/path/to/program start 8080

然后通过ps拿到其进程id

	$ cat /proc/pid/cmdline
	/path/to/programstart8080

你会发现start两侧的空格没了

其实原因是这样的，操作系统会把这个空格替换成`\0`，即ASCII码为0的一个字节，不信？

	cat /proc/pid/cmdline > abc
	vim abc

你会看到类似这样的内容

	/path/to/program^@start^@8080^@

vim中显示的`^@`实际就是`\0`，

# 使用golang解析

考虑到用户配置监控的时候通常就是直接`cat /proc/pid/cmdline`，然后截取一段内容，截取的内容中没有空格，那我在解析的时候把`\0`干掉即可。用的方法可能比较土，直接挨个byte遍历，把ASCII为0的byte剔除即可（你有更好的方法？留个言分享一下呗），代码在这里：

《[使用golang解析cmdline](https://github.com/toolkits/nux/blob/master/proc.go#L65)》

enjoy it

