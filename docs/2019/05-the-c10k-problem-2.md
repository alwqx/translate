# C10K问题二：IO策略之多客户端、非阻塞IO

## 说明
- [原文链接](http://www.kegel.com/c10k.html#nb)
- [翻译：@alwqx](https://github.com/alwqx)
- [项目地址](https://github.com/alwqx/translate)
- [tt](https://github.com/alwqx/tt)：自动生成翻译模板
- 用时: 2h（人机结合）
- <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/80x15.png" /></a>

## IO策略
### 为每个线程服务多个客户端，并使用非阻塞I/O和级别触发的就绪通知
为所有网络句柄设置非阻塞模式，并使用select()或poll()来告知哪个网络句柄有数据等待。这是传统方案的最爱。使用这种方案，内核会告诉你文件描述符是否准备就绪，不管上次内核会告诉后你是否已经使用该文件描述符完成了任何操作。 （'触发级别'的名称来自计算机硬件设计，与'[边缘触发](http://www.kegel.com/c10k.html#nb.edge)'相反。Jonathon Lemon在他的[BSDCON 2000论文](http://people.freebsd.org/~jlemon/papers/kqueue.pdf)中介绍了关于kqueue()的条款。）

注意：记住内核的就绪通知只是一个提示，这一点尤其重要; 当你尝试从中读取文件描述符时，它可能还没有准备好。这就为什么准备就绪通知条件下使用非阻塞模式很重要。

此方法有一个重要瓶颈，如果页面当前不在核心中，则从磁盘块中执行read()或sendfile()方法;内存映射磁盘文件的方法同样无效。内存映射磁盘文件也是如此。服务器第一次需要磁盘I/O时，其进程块阻塞，所有客户端必须等待，并且浪费原始非线程的性能。这就是异步I/O的意图，但在缺少AIO的系统上，执行磁盘I/O的工作线程或进程也可以解决这个瓶颈。一种方法是使用内存映射文件，如果mincore()指示需要硬盘I/O，请求工作线程执行I/O，并继续处理网络流量。Jef Poskanzer提到Pai，Druschel和Zwaenepoel在1999年的Flash网络服务器中使用了这个技巧;他们在[Usenix 99](http://www.usenix.org/events/usenix99/technical.html)上发表了演讲。看起来mincore()在BSD派生的Unix中可用，如FreeBSD和Solaris，但它不是[Single Unix Specification](http://www.unix-systems.org/)的一部分。[感谢Chuck Lever](http://www.citi.umich.edu/projects/citi-netscape/status/mar-apr2000.html)，它在2.3.51版本中成为Linux的一部分。

但是在[2003年11月的freebsd-hackers排行榜上](http://marc.theaimsgroup.com/?l=freebsd-hackers&m=106718343317930&w=2)，Vivek Pei等人报告了非常好的结果，即对Flash网络服务器进行系统范围的分析来解决瓶颈。他们发现的一个瓶颈是mincore（猜测的），另一个是sendfile会阻塞磁盘访问（事实）; 他们通过引入一个修改过的sendfile()来提高性能，当它获取的磁盘页面不再核心中时，它会返回类似EWOULDBLOCK的内容。（不知道你如何告诉用户页面现在是常驻的...在我看来这里真正需要的是aio_sendfile()。）他们的优化在1GHZ/1GB FreeBSD机器上用SpecWeb99基准测试得分约为800， 这比spec.org上的任何文件都要好。

对单线程来讲有很多中方法表明哪些非阻塞套接字准备好I/O了：
- **传统的select()**:不幸的是，select()仅限于FD_SETSIZE句柄。此限制将编译到标准库和用户程序中。（某些版本的C库允许你在用户应用程序编译时提高此限制）。有关select()如何与其它就绪通知方案互换使用的示例，请参阅[Poller_select](http://www.kegel.com/dkftpbench/doc/Poller_select.html)。
- **传统的poll()**:对于poll()可以处理的文件描述符数量没有硬编码限制，但处理数千个文件描述符时它确实变慢了，因为大多数文件描述符在任何时候都是空闲的，并且扫描数千个文件描述符需要时间。一些操作系统（例如Solaris 8）通过使用轮询提示等技术来加速poll()，该技术在1999年由[Niels Provos为Linux实施和基准测试](http://www.humanfactor.com/cgi-bin/cgi-delegate/apache-ML/nh/1999/May/0415.html)。有关poll()如何与其它就绪通知方案互换使用的示例，请参阅[Poller_poll](http://www.kegel.com/dkftpbench/doc/Poller_poll.html)
- **/dev/poll**:这是Solaris推荐的轮询替代品。/dev/poll背后的想法是利用通常的事实，即使用相同参数多次调用poll()。使用/dev/poll，你可以获得/dev/poll的开放句柄，并通过写入该句柄迅速告知操作系统你感兴趣的文件;从那时起，你只需从该句柄中读取当前就绪的文件描述符集。它在Solaris 7中悄然出现（参见[补丁106541](http://sunsolve.sun.com/pub-cgi/retrieve.pl?patchid=106541&collection=fpatches)），但其第一次公开亮相是在[Solaris 8](http://docs.sun.com/ab2/coll.40.6/REFMAN7/@Ab2PageView/55123?Ab2Lang=C&Ab2Enc=iso-8859-1)中; 据Sun称，在750个客户端，会占用poll()10％的开销。人们在Linux上尝试了/dev/poll的各种实现，但它们都没有达到像epoll一样的性能，并且从未真正完成。**建议不要在Linux上使用/dev/poll**。有关如何将/dev/poll与许多其他就绪通知方案互换使用的示例，请参阅[Poller_devpoll](http://www.kegel.com/dkftpbench/doc/Poller_devpoll.html)。（注意，该示例适用于Linux /dev/poll，可能无法在Solaris上正常工作。）
- kqueue()：这是FreeBSD（很快就是NetBSD）推荐的轮询替代品。kqueue()可以指定边沿触发或水平触发。