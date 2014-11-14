+++
date = "2010-08-14T12:08:34+08:00"
menu = "main"
tags = ["c-cpp", "interview"]
title = "C语言的#include指令不是非得引入.h头文件"

+++

搞过C语言的同学肯定都知道它的#include指令，无非就是引入头文件的。而实际上～它可以引入任何文件，比如～
我首先有个文件：inc.core，其内容为：
 
	printf("This is inc.core\n");  
 
还有个文件：inc.ext，其内容为：
 
	printf("This is inc.ext\n");  
	#include "inc.core"  
 
最后再来个C文件：hello.c，其内容为：
 
	#include <stdio.h>  
	  
	int main(int argc, char *argv[]) {  
	    printf("This is hello.c\n");  
	    #include "inc.ext"  
	    return 0;  
	}  
 
亲们，gcc -c hello.c 没有任何问题哦，gcc hello.o -o hello && ./hello 也可以成功打印哦……
这么看来，#include实际就相当于个模板语法，可以嵌套引用，可以引用任何文件进来，好吧，可能你知道这个特性，我今天刚知道的，囧，记录一下
