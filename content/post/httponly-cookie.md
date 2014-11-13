+++
date = "2014-10-10T19:50:27+08:00"
menu = "main"
tags = ["xss", "web", "cookie", "interview"]
title = "说说HttpOnly Cookie的作用"

+++

做WEB开发，经常要跟cookie打交道，经常会遭受黑客的XSS，说一下HttpOnly Cookie的作用

要理解HttpOnly的作用，要先弄懂XSS攻击，即跨站脚本攻击，大伙可以Google一下看看XSS到底是什么，来自wikipedia的解释：

跨网站脚本（Cross-site scripting，通常简称为XSS或跨站脚本或跨站脚本攻击）是一种网站应用程序的安全漏洞攻击，是代码注入的一种。它允许恶意使用者将代码注入到网页上，其他使用者在观看网页时就会受到影响。这类攻击通常包含了HTML以及使用者端脚本语言。

举个简单栗子：某网站提供了一个留言页面，留言内容是个富文本编辑器（也就意味着可以提交html代码给server），小黑同学深情款款得提交了一个振聋发聩的意见，但是内容中包含了一段恶意javascript脚本：

	<script>evil_script()</script>

而服务端没有做任何处理。由于这个留言页面可以看到别人提交的留言内容，于是后来的留言者就可以看到小黑的留言内容，就会运行小黑的那段javascript脚本，假如这个脚本的功能是窃取用户的cookie发给小黑自己的服务器，而这个无良的网站，竟然把用户的登录名和密码都保存在了cookie中，于是一起很严重的安全事件诞生！

要解决这个问题，可以从两方面着手

- 1）服务端对提交上来的留言内容做过滤，把恶意代码过滤掉
- 2）让恶意javascript代码读取不到我们种的cookie

HttpOnly的cookie就是从第二点解决方案着手的，cookie是通过http response header种到浏览器的，我们来看看设置cookie的语法：

	Set-Cookie: <name>=<value>[; <Max-Age>=<age>][; expires=<date>][; domain=<domain_name>][; path=<some_path>][; secure][; HttpOnly]

是一个name=value的KV，然后是一些属性，比如失效时间，作用的domain和path，最后还有两个标志位，可以设置为secure和HttpOnly，所以，我们只要加一个;HttpOnly的标志即可，比如：

	Set-Cookie: USER=Ulric-UlricPass; expires=Wednesday, 09-Nov-99 23:12:40 GMT; HttpOnly 

这样一来，javascript就读取不到这个cookie信息了，不过在与服务端交互的时候，Http Request包中仍然会带上这个cookie信息，即我们的正常交互不受影响。

done:)

最近意识到一点：面试官如果问到了一个你很懂的问题，你可以条理清晰的去大谈特谈，这样可以让面试官发现你的亮点，同时可以减少面试官问其他问题的时间，因为通常一次面试也就1个小时，这个问题占用的时间长了，就得缩减其他问题的时间。但切忌没有逻辑的啰啰嗦嗦。

