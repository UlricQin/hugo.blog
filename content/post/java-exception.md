+++
date = "2011-03-09T20:02:52+08:00"
menu = "main"
tags = ["java", "exception", "interview"]
title = "说说异常处理机制和最佳实践"

+++

这个问题仁者见仁智者见智，每个人心中的最佳实践不见得一致，但是你要有想法，这个很关键，如果连思考都没有思考过，那就不太好了。

很多高级语言都提供了异常处理，比如Java、Python、Ruby，比较底层的语言，比如C，没有提供异常机制，最近时兴的Golang，也没有提供通常的try-catch异常机制。

异常机制是必须的么？显然不是，因为我们通常可以用多个返回值来解决，如果语言本身不支持多返回值，那异常机制就是必须的，否则这个语言写起来真的会很痛苦，你想想是不是这样？：）

异常，故名思议，就是不正常的程序执行流程，我们拿Java中的一个类继承关系举例：
![java_exception](http://beego-blog.qiniudn.com/static/uploads/editor/1411818663.png)

异常情况大体可以分成两种：

- 1、没办法恢复的，上图中的Error红色部分，这种异常情况出现，程序只能崩溃。比如OutOfMemoryError，显然是系统内存不够用了，程序显然跑不下去了，必须崩溃
- 2、可以尝试恢复的，上图中的Exception绿色部分，这种情况通常不会导致程序崩溃，但是我说的是通常，如果处理不当，程序照崩不误

对于第一种情况，我们没法处理，属于不可控因素，通常的做法就是提前监控，比如发现系统内存剩余量少于10%立马报警，运维人员肯定对此深有感触：）

第二种情况我们一定要竭尽所能去处理，因为我们的服务要追求尽可能高的稳定性，五个9、六个9的SLA，所以程序一抛一个exception就立马crash，实在是不合适

下面我们重点聊聊绿色部分Exception，大部分语言的异常处理机制类似，比如C#、Python、Ruby，出问题了抛异常，上层代码可以捕获，也可以不捕获，语言设计者认为这样就OK了，Java分得更细，分成Checked Exception和Unchecked Exception

- a）Checked Exception是要在方法定义的时候显式声明，上层代码也要显式捕获，这点会在编译阶段强制检查
- b）Unchecked Exception，也叫Runtime Exception，不需要在方法定义的时候显式声明，也不强制上层代码显式捕获，一旦抛异常，JVM会沿着方法调用栈层层往上寻找catch块，如果找遍了都找不到，程序crash

那这两种异常的使用场景是什么呢？下面是笔者的个人观点，仅供参考：

1、Runtime Exception只用来处理坏的编码，这些都是程序员应该提前处理但是忘记处理的，比如NullPointerException，程序员应该在上层代码中判断变量是否为空，如果忘记判断了，那下层代码只能用NullPointerException这个RuntimeException来还以颜色。再比如：ArrayIndexOutOfBoundsException、IllegalArgumentException，咋样，有没有点感觉了？

2、RuntimeException虽说是程序员犯下的错误，但是为了程序不崩溃，还是要处理，所以我们通常会在方法调用栈的最顶层catch所有的异常，然后打印log，发报警邮件给工程师，便于以后做bugfix之类的

3、Checked Exception通常是需要程序员去尝试恢复的，比如：FileNotFoundException、EOFException。Checked Exception在向上层代码传递一些信息，让上层代码对症下药

4、举个例子，比如我们跟数据库打交道有的时候会遇到connection reset这种异常，这个应该封装为哪种异常返回呢？显然这不是坏的编码导致的，所以要用Checked Exception，上层代码要显式catch，如果发现是ConnectionResetException，要尝试重连数据库。

5、用户登录的时候，有人封装了UserNotFoundException、PasswordErrorException，你觉得是合理的么？个人认为可以这么用，比如写了一个Login方法，用户登录失败可能会有多种原因，要把这多种信息传达给上层代码，在一个没有多返回值的语言中，只能使用多种异常

6、为了传达文件不存在这个信息JDK提供了一个FileNotFoundException的异常，你觉得这是合理的么？笔者的观点是这样的：要分情况看待，如果要写一个专门判断文件是否存在的方法FileExists，此时无需定义异常，直接返回一个bool类型的值就行，如果在读写文件的方法中，发现某个文件不存在，可以抛出异常，这其实还是因为不能有多个返回值导致的，因为文件读写操作的方法参数和返回值都是与字节数目啊，字节数组啊之类的，没法往上传递文件不存在这个信息，所以，只能用异常了

7、如果能用if-else分支结构来描述的逻辑，不要用异常，虽说try-catch看起来可以做分支结构，但是try-catch效率低，不是正常的编码，百害而无一利

说得比较散乱，看来文笔还是不行啊，看官多多包含，希望能帮到你：）
本文来自微信公众账号：it_mianshiti，一个人每天整理面试题真是比较势单力薄，求投稿：）
