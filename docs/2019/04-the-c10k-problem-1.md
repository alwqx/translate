# C10K问题一：预备知识

## 说明
- [原文链接](http://www.kegel.com/c10k.html)
- [翻译：@alwqx](https://github.com/alwqx)
- [项目地址](https://github.com/alwqx/translate)
- [tt](https://github.com/alwqx/tt)：自动生成翻译模板
- 用时:
- <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/80x15.png" /></a>

现在web服务器可以同时处理一万个客户端了，你不觉得吗？毕竟，web现在是一个很大的舞台。

电脑也很棒，花费大约1200美元可以购买一台2GB RAM、每秒1000Mbit以太网卡和CPU频率1000MHz的机器。让我们估算下-如果每个客户端需要50KHz、100Kbytes内存和网速50Kbits每秒的资源，能够服务20000个客户端。它不用于其它功能，每秒仅从磁盘中取出4千字节并将它们发送到网络一次，服务2万个客户端。（顺便提一下，每个客户的费用为0.08美元。而一些操作系统每客户端收费100美元价格有点沉重！）因此硬件不再是瓶颈。

1999年，最繁忙的ftp站点之一`cdrom.com`实际上通过千兆以太网管道同时处理了10000个客户端请求。截至2001年，一些[ISP才提供相同的并发量](http://www.senteco.com/telecom/ethernet.htm)，它们希望这项服务越来越受大型企业客户欢迎。

随着服务器开始为互联网数千个客户提供服务，瘦客户端计算模型似乎又流行起来。

考虑到这一点，这里有一些关于如何配置操作系统和编写代码以支持数千客户端的说明。讨论围绕类Unix操作系统，因为这是我个人感兴趣的领域，但Windows也有一点涉及。


## 相关站点
请参阅Nick Black的优秀页面[《Fast UNIX Servers》][1]，了解大约2009年的情况。

2003年10月，Felix von Leitner整理了一个关于网络可扩展性的优秀[web网页](http://bulk.fefe.de/scalability/)和[演示文稿](http://bulk.fefe.de/scalable-networking.pdf)，其中包括比较各种网络系统调用和操作系统的基准测试。他的一个观察结果是Linux2.6内核确实击败了2.4内核，同时有很多很棒的图表，会让操作系统开发人员在一段时间内深思熟虑。（另外请参阅[Slashdot][2]的评论;看看是否有人会根据Felix的结果改进后续基准，这将会很有趣。）

## 首先需要读一本书
如果你还没读过，请快速浏览W. Richard Stevens的[《Unix Network Programming : Networking Apis: Sockets and Xti (Volume 1)》][2]。它描述了许多编写高性能服务器相关的I/O策略和陷阱。它甚至谈到了[thundering herd][3]问题。当你在这部分时，请阅读[Jeff Darcy关于高性能服务器设计的说明](http://pl.atyp.us/content/tech/servers.html)。

（对于那些使用而不是编写Web服务器的人来说，另一本可能更有用的书是Cal Henderson的[《构建可扩展的网站》][4]。）

## I/O框架
框架提供了预打包的库，它们抽象了下面介绍的一些技术，使代码与操作系统隔离，更具可移植性。

- [ACE](http://www.cs.wustl.edu/~schmidt/ACE.html)是一个重量级的C++ I/O框架，包含一些面向对象的I/O策略实现以及许多其它有用的东西。特别是，他的Reactor是执行非阻塞I/O的面向对象方式，而Proactor是执行异步I/O的面向对象方式。
- [ASIO](http://asio.sf.net/)是一个C++ I/O框架，它正在成为Boost库的一部分。就像是STL时代更新版的ACE。
- [libevent](http://monkey.org/~provos/libevent)是Niels Provos编写的轻量级C I/O框架。它支持kqueue和select，很快就会支持poll和epoll。我认为，这只是水平触发，它有好坏两面。Niels有一个[很好的图表](http://monkey.org/~provos/libevent/libevent-benchmark.jpg)，表示函数处理一个事件的时间和连接数的关系。它显示kqueue和sys_epoll是明显的赢家。
- 我自己关于轻量级框架的尝试（很遗憾，没有及时更新）：
    - [Poller][5]是一个轻量级的C ++ I/O框架，它基于你想要的底层自带API（poll，select，/ dev / poll，kqueue或sigio）实现一个级别触发的API。它对于比较[各种API性能的基准测试非常有用][5]。本文档链接到下面的Poller子类，以说明如何使用每个API。
    - [rn][6]是一个轻量级的C I/O框架，这是我在Poller之后的第二次尝试。它是lgpl（因此它在商业应用程序中更易使用）和C（易在非C++应用程序中使用）。它被用于一些商业产品中。
- Matt Welsh在2000年4月写了一篇关于如何在构建可伸缩服务器时平衡工作线程和事件驱动技术的使用的[论文][7]。该论文描述了他的Sandstorm I/O框架部分内容。
- Cory Nelson's Scale! library，Windows的异步套接字、文件和管道I/O库。

## I/O策略
网络软件的设计者有很多选择。比如以下：
- 是否以及如何从单线程发出多个I/O调用
    - 不要这么做;使用阻塞/同步调用，并可能使用多个线程或进程来实现并发
    - 使用非阻塞调用（例如，设置作用在套接字上的write函数参数O_NONBLOCK）来启动I/O，以及准备就绪通知（例如poll（）或/dev/poll）以了解何时可以启动该通道上的下一个I/O 。通常只能用于网络I/O，而不能用于磁盘I/O。
    - 使用异步调用（例如aio_write（））来启动I/O，以及完成通知（例如信号或完成端口）以了解I/O何时完成。适用于网络和磁盘I/O。
- 如何控制为每个客户端服务的代码
    - 为每个客户端运行一个进程（经典的Unix方法，自1980年以来都在使用）
    - 一个OS级别的线程处理许多客户端;每个客户由以下人员控制：
        - 用户级线程（例如GNU状态线程，经典的Java绿色线程）
        - 状态机（有点深奥，但在某些圈子很受欢迎;我最喜欢）
        - 延续（有点深奥，但在某些圈子中很受欢迎）
    - 每个客户端运行一个操作系统级线程（例如，经典的Java原生线程）
    - 每个活跃的客户端运行一个操作系统级线程（例如，带有apache前端的Tomcat、NT完成端口、线程池）
- 要么使用标准O/S服务，或者将一些代码放入内核（例如，在自定义驱动程序，内核模块或VxD中）

以下五种组合似乎很受欢迎：
- [为每个线程服务多个客户端，并使用非阻塞I/O和级别触发的就绪通知](http://www.kegel.com/c10k.html#nb)。
- [为每个线程服务多个客户端，并使用非阻塞I/O和准备就绪更改通知](http://www.kegel.com/c10k.html#nb.edge)。
- [为每个服务器线程提服务多个客户端，并使用异步I/O](http://www.kegel.com/c10k.html#aio)。
- [为每个服务器线程服务一个客户端，并使用阻塞I/O](http://www.kegel.com/c10k.html#threaded)。
- [将服务器代码构建到内核中](http://www.kegel.com/c10k.html#kio)。

[1]: <http://dank.qemfd.net/dankwiki/index.php/Network_servers>
[2]: <http://developers.slashdot.org/developers/03/10/19/0130256.shtml?tid=106&tid=130&tid=185&tid=190>
[3]: <http://www.citi.umich.edu/projects/linux-scalability/reports/accept.html>
[4]: <https://www.amazon.com/gp/product/0596102356>
[5]: <http://www.kegel.com/dkftpbench/Poller_bench.html>
[6]: <http://www.kegel.com/rn/>
[7]: <http://www.cs.berkeley.edu/~mdw/papers/events.pdf>
[8]: <http://svn.sourceforge.net/viewcvs.cgi/*checkout*/int64/scale/readme.txt>