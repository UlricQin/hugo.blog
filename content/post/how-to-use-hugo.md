+++
date = "2014-11-11T17:15:00+08:00"
draft = false
title = "使用Hugo搭建免费个人Blog"
tags = ["Hugo", "Golang"]
+++

# 标题一
## 标题2
### 标题3
#### 标题4
##### 标题5
###### 标题6

目前在写一个agent，接收用户指定的脚本然后去执行。考虑到用户写的脚本可能出各种问题导致程序hang住，需要为脚本执行设置一个超时时间。Golang中使用exec.Cmd来抽象用户要执行的外部命令，每个命令在内部是一个Process，那么我们只要在超时之后kill掉这个Process即可，我的实现代码如下：

    func CmdRunWithTimeout(cmd *exec.Cmd, timeout time.Duration) (error, bool) {
        done := make(chan error)
        go func() {
            done <- cmd.Wait()
        }()

        var err error
        select {
        case <-time.After(timeout):
            //timeout
            if err = cmd.Process.Kill(); err != nil {
                log.Error("failed to kill: %s, error: %s", cmd.Path, err)
            }
            go func() {
                <-done // allow goroutine to exit
            }()
            log.Info("process:%s killed", cmd.Path)
            return err, true
        case err = <-done:
            return err, false
        }
    }

这种利用select和channel来实现超时的写法在Golang中比较常见，简单解释一下

程序都是顺序执行的，如果cmd.Wait()放到主线程，那么一旦执行到cmd.Wait()我们就会阻塞在这里，根本没办法停掉它。所以要把这句话扔到一个goroutine中：

    go func() {
        done <- cmd.Wait()
    }()

OK，一旦接收到一个cmd，我们就把它扔到一个goroutine中去执行了，接下来开始判断是否超时，通过select和channel结合的方式，如果时间到了，会执行：case <-time.After(timeout)，如果cmd.Wait()在超时之前就跑完了，就会走到：case err = <-done。大体逻辑没问题，关键是超时之后怎么办？
我的项目中，需要kill掉这个超时的Process，cmd.Process有个kill方法，这个也可以做到，关键是下面这句话可能不好理解：

    go func() {
        <-done // allow goroutine to exit
    }()

首先，<-done肯定要被执行，否则我一kill完成立马return，done中的数据没有任何一个goroutine来取值，那么cmd.Wait()就一直没法写数据到done中，cmd.Wait()所在的goroutine就会一直存在，久而久之，程序必然崩溃……

第二点，为啥要把<-done放到一个goroutine中呢？因为cmd.Process.kill()之后cmd.Wait()不一定立马返回，所以<-done可能会很耗时，而实际我都已经打算kill进程了，根本就不关心<-done中的数据了，就把<-done扔到一个goroutine中，主线程可以不必阻塞在这，等<-done结束之后，所在的goroutine也会自动消亡

看起来说的头头是道，其实我也不是很确定这么做是不是OK的，测试的时候基本没啥问题，等程序上线跑一段时间之后我再回来更新这篇文章

最后，怎么调用上面这个方法呢？

    cmd := exec.Command(realRun)
    var stdout bytes.Buffer
    cmd.Stdout = &stdout
    var stderr bytes.Buffer
    cmd.Stderr = &stderr
    cmd.Start() // attention!

    var isTimeout bool
    err, isTimeout = systool.CmdRunWithTimeout(cmd, time.Duration(t)*time.Second)

    // handle stdout/stderr/err/isTimeout

上面的内容只是测试
