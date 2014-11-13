+++
date = "2014-03-15T18:12:40+08:00"
menu = "main"
tags = ["git"]
title = "Git中设置多个服务器地址"

+++

问题场景是这样的：公司有个私有git服务（使用公司邮箱登陆），但是我还需要操作github（使用自己的普通邮箱登陆）

显然，我们要为这两个邮箱账号分别生成公钥，通常，公私钥都是放到~/.ssh下面的，so：

	mkdir -p ~/.ssh
	cd ~/.ssh
	ssh-keygen -t rsa -C "qinxiaohui@mi.com" # 把这个文件命名为id_rsa_mi，然后一路回车
	ssh-keygen -t rsa -C "cnperl@163.com" # 把这个文件命名为id_rsa_github，然后一路回车

此时在~/.ssh下面生成了两对公私钥，把id_rsa_mi.pub的内容贴到公司git服务的ssh keys中，把id_rsa_github.pub的内容贴到github的ssh keys中。然后touch一个配置文件：


	touch ~/.ssh/config
	chmod 600 ~/.ssh/*

最后在~/.ssh/config中添加如下内容即可：


	host git.mi.com
	user git
	hostname git.mi.com
	port 22
	identityfile ~/.ssh/id_rsa_mi

	host github.com
	user git
	hostname github.com
	port 22
	identityfile ~/.ssh/id_rsa_github

