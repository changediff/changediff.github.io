---
title: 理解用户空间与内核空间
tags:
  - linux
  - user-space
  - kernel-space
  - 八股文
date: 2022-06-21 23:13:31
---

最近面试被问到了用户空间和内核空间的问题，发现脑子以为懂了和嘴巴说出来之间又一段差距，就用写作来填平吧。

## 什么是用户空间和内核空间？
在Linux中，系统内存可以分为用户空间(user space)和内核空间(kernel space)，用户空间是运行用户进程的内存空间，内核空间是运行内核进程的内存空间。

## 为什么要区分用户空间和内核空间
系统内核位于用户进程和硬件资源之间，将硬件资源虚拟化，提供了接口，可以让用户进程访问硬件资源，并且负责管理硬件资源的调度。
具体来说，内核主要干这四件事情：
1. 内存管理：跟踪所有进程用了哪些内存、用了多少、存在了哪个物理地址等等
2. 进程管理：管理进程对CPU时间分片的占用
3. 设备驱动：作为中间人或者解释器，替进程访问硬件设备（Unix系统中一切皆文件的思想，就是对硬件设备的绝妙抽象）
4. 系统调用system calls与安全：接收从用户空间传来的系统调用，做安全检查并执行（什么是系统调用后面细说）

以上可以看出为什么要区分用户空间和内核空间，一方面这样的分层设计可以让系统内核的维护更加灵活，另一方面可以让系统更加安全，敏感的操作通过统一的syscalls接口切换到内核空间中完成。
![syscalls](user-space-vs-kernel-space-basic-system-calls.png)

Linux中的系统调用流程如下图，syscalls是通过汇编语言指令的方式提供的，我们最经常接触的进程实际上是调用的`glibc/解释器/编译器`等库提供的功能，这也是为什么Linux发行版中单独升级glibc版本是一个风险很大的操作（glibc的版本变更可能导致很多系统基础组件的syscalls不兼容）
![syscalls调用](user-space-vs-kernel-space-system-calls-gears.png)

## system calls

系统调用如此重要，那么它具体提供了哪些功能呢？syscalls的详细内容可以参考[man syscalls](https://man7.org/linux/man-pages/man2/syscalls.2.html)，主要包含进程控制(fork,exit,wait)、文件管理(open,read,write,close)、设备管理(ioctl,read,write)、进程信息维护(getpid,alarm,sleep)、进程间通信(pipe,shmget,mmap)等等。

下面简单介绍下常见的syscall的作用：
`open()`：打开文件，返回文件描述符，详细可参考[链接](https://www.cnblogs.com/kelamoyujuzhen/p/9638710.html)

`read()`：读取文件，读取的文件需要有文件描述符标识，读取前需要先执行`open()`，read()系统调用需要传入三个参数（文件描述符fd、文件读缓存的地址、读取的字节数）；这里涉及到了读写缓存的概念，这个缓存是由内核维护在内存中的page cache，具体可参考[链接](https://www.oreilly.com/library/view/understanding-the-linux/0596005652/ch15s01.html#:~:text=The%20page%20cache%20is%20the,User%20Mode%20processes's%20read%20requests.)

`write()`：写入文件，这是程序输入的常见方式之一，write()系统调用需要传入三个参数（文件描述符、写缓存的地址、写入的字节数）

`close()`：关闭文件，释放文件描述符

`fork()`：创建子进程，详细参考[链接](https://www.csl.mtu.edu/cs4411.ck/www/NOTES/process/fork/create.html)

`wait()`：等待子进程结束，返回子进程的返回值。如果子进程执行结束了，父进程没有执行wait，就会产生僵尸进程。运维层面解决僵尸进程的方法是kill父进程，让僵尸进程成为孤儿进程，又init进程来执行wait，让僵尸进程死亡。程序代码实现层面有两种方式避免僵尸进程，1)调用wait(), 2) fork()两次让init进程来管理子进程。**需要注意的是**，kill父进程和fork()两次在容器里面不管用，因为容器里面通常没有init进程存在，这种情况需要重启容器来解决。
对于僵尸进程的详细讲解可以参考[链接](https://www.cnblogs.com/anker/p/3271773.html)

`exit()`：退出进程，返回值为0，在多线程场景中则表示该线程执行结束。操作系统会回收exit()进程的资源

`kill()`：可以参考常用的kill命令理解，发送不同信号量例如SIGINT、SIGKILL等等。


## 中断
pass

## 用户空间和内核空间在Linux、Windows和MacOS中的区别？
上面关于用户空间和内核空间的讨论都是基于Linux的，这些功能在Windows和macOS上的实现是又差别的，具体又涉及到了宏内核和微内核设计的差异，我的知识还不足以讲清楚这些。**但不管那种实现都提供了类似的用户空间和内核空间的抽象。**
下面仅提供文章和图片用于参考~

[关于微内核的对话](https://www.yinwang.org/blog-cn/2019/08/19/microkernel)
[聊了聊宏内核和微内核，并吹了一波 Linux](https://segmentfault.com/a/1190000040898100)
![宏内核和微内核](macro-micro-kernel.webp)
![windows-linux](type-of-syscalls.png)

## 参考资料
1. https://www.redhat.com/en/topics/linux/what-is-the-linux-kernel
1. https://www.redhat.com/en/blog/architecting-containers-part-1-why-understanding-user-space-vs-kernel-space-matters
2. https://www.redhat.com/en/blog/architecting-containers-part-2-why-user-space-matters
3. https://www.redhat.com/en/blog/architecting-containers-part-3-how-user-space-affects-your-application
4. [内核空间定义](http://www.linfo.org/kernel_space.html)
5. [用户空间定义](http://www.linfo.org/user_space.html)
5. https://www.tutorialspoint.com/what-is-the-purpose-of-system-calls#
6. [孤儿进程与僵尸进程](https://www.cnblogs.com/anker/p/3271773.html)
7. [聊了聊宏内核和微内核，并吹了一波 Linux](https://segmentfault.com/a/1190000040898100)
