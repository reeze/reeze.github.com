---
layout: post
title: 多线程下的fork及进程的写时复制导致的性能问题
date: 2014-08-23 14:00
comments: true
categories: 
  - Linux
  - Fork
  - Multi threads
  - Performance issue
  - 性能问题
  - 写时复制
  - HHVM
  - PHP
---

## 名词解释

1. PHP vs HHVM: PHP指的是[php.net(Zend)](http://php.net)实现的PHP，而HHVM指的是[Facebook开源的PHP实现](http://github.com/facebook/hhvm)。
2. PHP-FPM: (PHP Fastcgi Process Manager) 一个PHP Sapi实现，目前的主流的Web应用使用的方式。基于多进程的模型
3. HHVM: AdminServer 这是HHVM中为了更好的运维和定位问题实现的一个HTTP操作接口，可以实时的获取和操作HHVM内部状态，
   这对于我们是一个非常便利的接口，比如可以打印出内部的队列长度（fpm中也有类似接口，不过灵活性差很多）

## 多线程下fork()/exec()出现的性能问题

贴吧目前使用的[HHVM](http://github.com/facebook/hhvm)来运行PHP程序，HHVM采用的是多线程模型，
以前我们使用的是PHP-FPM，PHP-FPM采用的是多进程的模型。
我们通过一个我们上线遇到的问题来看看Linux的写时复制和多线程相关的问题。

上周我们迁移一个服务HHVM运行环境时发现上线后CPU占用飚的非常高，经过分析发现程序时间消耗的最多时间的地方是：`fork()`，
这个有点奇怪，fork的时候怎么会花那么长时间呢？经过分析发现我们有个基础库的实现中使用了PHP的`exec()`函数来启动shell程序做字符处理，exec的实现和bash类似，先fork()一个进程然后通过exec系统调用执行相应的命令，
这没有什么不对，对不对。但是为什么在HHVM下会导致CPU利用率过高，而PHP中没事呢？ 先看一些基本的背景：


## 关于写时复制

在Linux中要启动一个新进程的方式通常是：先调用`fork()`函数fork出一个新的进程，然后在
新的进程中调用exec()函数来启动新的程序从而达到启动新程序的目的，比如采用下面的代码实现。


```C
int start_prog(char *prog, char* args[])
{
  pid_t pid = fork(); // 创建子进程
  if (pid < 0) return -1;
  if (pid == 0) { // 子进程
    if (execv(prog, args) < 0) return -1; // 加载新的程序
  } else {
    return 1;
  }
}

int main()
{
  char* args[] = {NULL};
  return start_prog("/bin/ls", args);
}

```

我们知道进程间的内存地址空间是隔离的，fork()系统调用的结果是生成一个新的子进程，为了保证隔离性，
早期的UNIX采用在`fork()`将父进程的地址空间完整的复制一份。这个操作非常的耗时。
为了提高效率现代的Unix及Linux采用了一种称为`写时复制`的技术，其实也就是一种延迟操作的做法，
子进程和父进程在`fork()`时并不马上复制，而是暂时共享内存空间，随后只要父进程或者子进程试图写共享的内存就会产生一个异常，
这时内核才把内存空间进程复制，比如我们在Shell中启动一个程序时随后就会启动新的程序，启动后的程序将会覆盖旧的内存空间，
如果提前就复制了，那么这个复制操作其实是白做了，为此系统将这个操作优化为写时复制。

![写时复制，发生写时才复制内存地址空间](/images/fork.png)

![如果马上进行exec加载新的程序，那么复制地址就没必要了](/images/fork_exec.png)


## 解决方案

从前面的介绍我们知道，启动新程序的时候利用了写实复制的技术避免不必要的消耗。对于HHVM来说，
我们使用的是多线程，每个线下都在并行的执行，也就是说，在某个线程在执行fork()的是时候
还会有其他线程在处理任务，由于线程间是共享进程空间的，那时不可避免的可能会写内存。这就触发了
操作系统的写实复制，导致大量的内存复制操作（其实也是没必要的），这会导致资源占用急剧上涨。

说到这里你可能会说：你看，多线程模式不太好吧。HHVM的实现是不是有问题？*其实对于PHP也会有类似的问题的*，
如果你使用的是PHP的多线程模型(现在应该很少的人使用)。

这个问题，HHVM使用了一个比较巧妙的方式来解决。

HHVM的思路是这样的，既然多线程下写实复制容易出俩捣乱，那么就让fork发生在非多线程的进程中，让他不可能发生空间复制。

1. HHVM启动时先预先启动N个(可配置)代理进程，在父进程和这些代理进程之前预先开启管道方便通信
2. 有需要启动子进程的时候，通过管道选择一个没有正在fork()进程的代理进程，将执行信息通过管道发给代理进程
3. 代理进程根据要执行的程序fork()一个新的子进程并执行相应的命令，然后将执行完成的信息通过管道写回主进程。

从上面可以知道，因为代理进程每个进程都只有一个线程不会存在多线程写的问题。 HHVM中将它称为[轻进程](https://github.com/facebook/hhvm/blob/master/hphp/util/light-process.cpp)。

![解决方案](/images/fork_thread.png)

使用了HHVM的轻进程后，CPU直接就降了下来，我们虽然可是使用这个方案解决这个问题，不过我们还是将
`exec()`调用改成了PHP原生文件读取操作，一来exec()函数的成本相对较高，二来使用shell不利于可移植性，
虽然我们不太可能使用Windows，不过这样的耦合是没必要的。

## 总结

1. 多线程vs多进程。其实这两个模式没有绝对的好坏，就要需要什么，多进程的好处是进程隔离，程序出现问题
  也能保证服务不整体crash掉，但是多进程带来的问题是进程间通信的成本，多线程也有多好处，比如HHVM中的
  AdminServer，队列的管理等等，如果不是多线程，JIT，实例管理，Debug都会非常的复杂。
2. 对于高并发的程序建议**少用不用`exec()/system()`等函数**，除了内存占用，进程数也可能会变成资源瓶颈，
  其实和其他思路类似，尽量在用户态把事情做了。
3. 多线程下`fork()`还有其他的问题，我们的场景是fork() 然后马上执行新的程序，如果你是真的要fork，
  这可能会遇到更多的问题，比如锁等等，请参考<http://rachelbythebay.com/w/2014/08/16/forkenv/>

## 参考资料

1. 《Understanding Linux Kernel 3rd》
2. HHVM AdminServer: <https://github.com/facebook/hhvm/blob/master/hphp/doc/command.admin_server>
3. PHP-FPM status page: <https://www.centos.bz/2013/09/enable-php-fpm-status-page/>
4. HHVM: <http://hhvm.com/>
