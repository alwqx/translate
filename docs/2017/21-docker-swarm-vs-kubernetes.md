# Docker Swarm vs Kubernetes

## 说明
- [原文链接](https://blog.cloudboost.io/docker-swarm-vs-kubernetes-c796e630ca87)
- [翻译：@alwqx](https://github.com/alwqx)
- [项目地址](https://github.com/alwqx/translate)
- [tt](https://github.com/alwqx/tt)：自动生成翻译模板
- 用时:
- <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/80x15.png" /></a>

>介绍它们间的不同，以及优缺点。

![](https://cdn-images-1.medium.com/max/800/1*qHvFfsb8YriqUsGe1Q9e9A.png)

容器化已经改变我们部署软件和微服务开发的方式。如果你刚听说容器，[这篇博客](https://blog.cloudboost.io/get-started-with-docker-by-watching-these-5-videos-80d25d71c1a5)帮你入门。

## 什么是容器编排
容器能够把服务打包成基本单元，你可以把它部署到任何地方：本地机器、测试环境或者生产系统。但是在生产环境中，你却不能把所有容器都运行在一台机器上，因为会用光系统的资源。你需要多个机器（或者节点）以集群（不同机器通过网络通信）的方式运行，然后把容器部署到集群中。现在问题变成，如果我有多个机器/节点组成的集群，我该如何决定容器运行在哪台机器上呢？有了编排软件，你只需要告诉它我要部署容器，剩下的事情交给编排软件即可。

编排软件负责以下几点：
- 选择最适合部署容器的机器。最适合指的是拥有最多空闲资源，或者说如果容器能获得更多的运行内存，比如Redis。
- 发生机器故障，能自动把故障机器上的容器部署到其它节点。
- 如果集群添加了新的机器，重新平衡容器的分配情况。
- 如果容器故障了，重启它。
- ...

现在你已经理解为什么需要容器编排了，下面我们一起看下当下最流行的两个选择以及它们间的对比。

## Docker Swarm
Swarm是为Docker开发原生的集群管理引擎。任何适配Docker container的工具、服务或软件都可以很好地兼容Swarm。下面是一些Docker Swarm的优缺点：

**优点：**
- 容易上手，“开箱即用”的用户体验。
- 零“单点故障”（single-point-of-failure）架构。
- 自动生成证书，默认提供安全机制。
- 向后兼容组件。
- 开源

**缺点：**
- 处于项目启动/早期开发阶段。我们不推荐商业应用上使用。随着时间推移，它会更加成熟。
- 功能简单有限。

## Kubernetes
Kubernetes是一个Google主导的生产就绪、企业级、成熟的编排平台。它的利弊有：

**优点：**
- 生产就绪、企业级。它被很多公司用于规模生产环境。
- 相比Docker Swarm，它更成熟。
- 可用于公有云、私有云、混合云等多种云环境。
- 模块化。
- 自愈能力：自动布局、自动重启、自动备份、自动伸缩。
- 开源。
- 因为模块化设计，可被用于部署任何架构。

**缺点：**
- 难于部署。如果不使用云服务商Azure,Google或者Amazon，在你的集群中搭建Kubernetes环境非常困难。大部分云服务商都提供了一键安装功能。
- 比Docker Swarm更陡峭的学习曲线。它不使用Docker CLI。

CloudBoost生产环境使用了60节点的Kubunertes集群，我们是无服务+BaaS（backend as a service）架构，开发者不用关心重复的任务像：认证、通知、邮件服务、管理和扩展数据库、文件、缓存等，这减少了一半的开发时间。我们使用MongoDB和Redis集群做数据存储，NodeJS支撑我们大部分微服务。CloudBoost完全开源，遵循Apache 2 License开源协议，所以你可以随意修改代码，在你的服务器上免费安装。GitHub地址[点这里](https://github.com/cloudboost/cloudboost)。

我们还提供了一个Docker Cloud/Compose [file](http://github.com/cloudboost/docker)，你可以直接使用它安装CloudBoost。

如果你希望深入了解我们如何在生产环境使用Kubernetes。可以阅读[这篇博客](https://blog.cloudboost.io/how-cloudboost-uses-docker-kubernetes-and-azure-to-scale-60-000-apps-d54d7eaf02c9)。

如果你希望快速启动集群，不运行关键应用，或者快速入门编排工具，我推荐Docker Swarm。如果你的场景接近商业环境，你应该考虑Kubernetes。