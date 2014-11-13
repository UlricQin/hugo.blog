+++
date = "2010-06-13T11:09:16+08:00"
menu = "main"
tags = ["java", "interview"]
title = "对比Java中的String、StringBuffer、StringBuilder"

+++

今天来个简单的题目，轻松一下：）

相信很多人对这个问题都不陌生，只要是个Java程序员，肯定就用过这几个类：
1、String是个不可变对象，这就意味着每次字符串拼接都是创建了新的实例
2、StringBuilder和StringBuffer都是专门用来做字符串拼接的
3、StringBuffer是线程安全的，StringBuilder是线程不安全的
4、线程安全是要付出代价的，所以StringBuffer比StringBuilder要慢一点点

OK，上面四条是不是倒背如流了？那问个具体问题：

1、以下虚构出来的三种写法哪个速度最快？哪个最差？

	String str = "I" + "love" + "Java" + "Python" + ... + "Golang"; 
	String str = new StringBuilder("I").append("love").append("Java").append("Python").append(...).append("Golang").toString(); 
	String str = new StringBuffer("I").append("love").append("Java").append("Python").append(...).append("Golang").toString();

解答：因为都是字符串字面量，第一种写法速度最快，在JVM看来就相当于是 String str = "IloveJavaPython...Golang” ，当然了，这么写纯属蛋疼，为了考察知识点而已，诸君一笑置之：）第三种用了StringBuffer的最慢，呵呵

如果是这么写呢？

	String a = "I"; 
	String b = "love"; 
	String c = "Java"; 
	String d = "Python"; 
	... 
	String e = "Golang"; 
	String str = a + b + c + d + ... + e;

变量之间用+连接，不再是字符串字面量，这种写法将会是最慢滴

2、再来看一个，问：下面这个方法可以用于多线程环境么？

	public static String build(String... args) { 
	    StringBuilder buf = new StringBuilder(); 
	    for (int i = 0; i < args.length; i++) { 
	        buf.append(args[i]); 
	    } 
	    return buf.toString(); 
	}

解答：此处的StringBuilder是个局部变量，虽说StringBuilder本身是线程不安全的，但是用在此处没有任何问题哈：）
