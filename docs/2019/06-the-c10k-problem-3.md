# C10K问题三：IO策略之多客户端、非阻塞IO和准备就绪更改通知
	
## 说明
- [原文链接](http://www.kegel.com/c10k.html#nb.edge)
- [翻译：@AdolphLWQ](https://github.com/adolphlwq)
- [项目地址](https://github.com/adolphlwq/translate)
- [tt](https://github.com/adolphlwq/tt)：自动生成翻译模板
- 用时: 2h（人机）
- <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/80x15.png" /></a>

## 单线程服务多个客户端，并使用非阻塞I/O和准备就绪更改通知

准备改变通知（或边缘触发准备通知）意味着你为内核提供文件描述符，稍后，当该描述符从未准备好转换为准备好时，内核会以某种方式通知你。然后它假定你知道文件描述符已准备好，并且不会再发送该类型描述符的准备就绪通知，直到你执行某些操作，导致文件描述符非就绪状态为止（例如，你在recv、send或accept中收到EWOULDBLOCK错误为止，或者recv与send传输的数据小于请求的字节数）。

当你使用准备就绪更改通知时，你必须为虚假事件做好准备，因为一个常见的实现是在收到任何数据包时发出就绪信号，无论文件描述符是否已经准备好。

这与“[水平触发](http://www.kegel.com/c10k.html#nb)”准备就绪通知相反。它对编程错误的宽容度要低一些，因为如果你只错过一个事件，那么事件的连接就会永远停滞不前。尽管如此，我发现边缘触发的就绪通知使得使用OpenSSL开发非阻塞客户端变得更容易，因此值得尝试。

[Banga，Mogul，Drusha'99](http://www.cs.rice.edu/~druschel/usenix99event.ps.gz)在1999年描述了这种方案。

有几个API可以让应用程序检索“文件描述符就绪”通知：

### kqueue()

这是FreeBSD（很快就是NetBSD）推荐的边缘触发poll替换。

FreeBSD 4.3及更高版本、以及截至2002年10月的NetBSD-current均支持poll()的通用替代方案，称为kqueue()/kevent();它支持边沿触发和水平触发。（另请参阅[Jonathan Lemon的页面](http://people.freebsd.org/~jlemon/)和他[关于kqueue的BSDCon 2000论文](http://people.freebsd.org/~jlemon/papers/kqueue.pdf)。）

与/dev/poll一样，你可以分配一个监听对象，不是打开文件/dev/poll，而是调用kqueue()来分配。要更改你正在侦听的事件，或者获取当前事件列表，请在kqueue()返回的描述符上调用kevent()。它不仅可以监听套接字就绪，还可以监听纯文件就绪、信号、甚至是I/O完成。

注意：截至2000年10月，FreeBSD上的线程库与kqueue()的交互不好;当kqueue()阻塞时，整个进程阻塞，而不仅仅是调用线程。

有关如何与许多其它就绪通知方案互换使用kqueue()的示例，请参阅[Poller_kqueue](http://www.kegel.com/dkftpbench/doc/Poller_kqueue.html)。

使用kqueue()的例子和库有：
- [PyKQueue](http://people.freebsd.org/~dwhite/PyKQueue/) kqueue的python实现
- [Ronald F. Guilmette的echo server例子](http://www.monkeys.com/kqueue/echo.c)

### epoll

这是Linux 2.6内核推荐的边缘触发轮询替换。

2001年7月11日，Davide Libenzi提出了实时信号的替代方案;他的补丁提供了他现在所称的[/dev/epoll](http://www.xmailserver.org/linux-patches/nio-improve.html) 。这就像实时信号就绪通知一样，但它可以合并冗余事件，并且具有更有效的批量事件检索方案。

Epoll方案合并到Linux2.5内核树上，其API从调用/dev/epoll文件转到系统调用sys_epall后合并到2.5.46中。2.4版本内核可以使用旧版epoll的补丁。

- Polyakov的kevent（Linux 2.6+）新闻报道：2006年2月9日和2006年7月9日，Evgeniy Polyakov发布了补丁，似乎统一了epoll和aio;他的目标是支持网络AIO。详情见：
    - [the LWN article about kevent](http://lwn.net/Articles/172844/)
    - [his July announcement](http://lkml.org/lkml/2006/7/9/82)
    - [his kevent page](http://tservice.net.ru/~s0mbre/old/?section=projects&item=kevent)
    - [his naio page](http://tservice.net.ru/~s0mbre/old/?section=projects&item=naio)
    - [some recent discussion](http://thread.gmane.org/gmane.linux.network/37595/focus=37673)
- Drepper的新网络接口（Linux 2.6+提案）
在OLS 2006上，Ulrich Drepper提出了一种新的高速异步网络API，详情：
    - 他的论文[《The Need for Asynchronous, Zero-Copy Network I/O》](http://people.redhat.com/drepper/newni.pdf)
    - [his slides](http://people.redhat.com/drepper/newni-slides.pdf)
    - [LWN article from July 22](http://lwn.net/Articles/192410/)

### 实时信号
这是2.4 Linux内核推荐的边缘触发轮询替换。

2.4 linux内核可以通过特定的实时信号提供套接字就绪事件。以下是如何打开此行为：
```c
/* Mask off SIGIO and the signal you want to use. */
sigemptyset(&sigset);
sigaddset(&sigset, signum);
sigaddset(&sigset, SIGIO);
sigprocmask(SIG_BLOCK, &m_sigset, NULL);
/* For each file descriptor, invoke F_SETOWN, F_SETSIG, and set O_ASYNC. */
fcntl(fd, F_SETOWN, (int) getpid());
fcntl(fd, F_SETSIG, signum);
flags = fcntl(fd, F_GETFL);
flags |= O_NONBLOCK|O_ASYNC;
fcntl(fd, F_SETFL, flags);
```

当read()或write()等普通I/O函数完成时，它会发送该信号。要使用它，在外部循环编写一个普通的poll()，在循环内部，在你接收到poll()处理完所有的fd后，调用sigwaitinfo()。

有关如何将rtsignals与许多其它就绪通知方案互换使用的示例，请参阅[Poller_sigio](http://www.kegel.com/dkftpbench/doc/Poller_sigio.html)。

有关直接使用此功能的示例代码，请参阅[Zach Brown的phhttpd](http://www.kegel.com/c10k.html#phhttpd)。

### Signal-per-fd
Chandra和Mosberger提出了实时信号方法的修改,称为“signal-per-fd”。它通过合并冗余事件来减少或消除实时信号队列溢出。但它并没有超越epoll。他们的[论文](www.hpl.hp.com/techreports/2000/HPL-2000-174.html)将此方案的性能与select()和/dev/poll进行了比较。

Vitaly Luban于2001年5月18日宣布实施该计划的补丁; 他的补丁在网址www.luban.org/GPL/gpl.html上。（注意：截至2001年9月，这个补丁在重负载下可能仍然存在稳定性问题。大约4500个用户的[dkftpbench](http://www.kegel.com/dkftpbench)可能会触发oops。）

有关如何将signal-per-fd与许多其他就绪通知方案互换使用的示例，请参阅[Poller_sigfd](http://www.kegel.com/dkftpbench/doc/Poller_sigfd.html)。