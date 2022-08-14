# 2017年预测：云原生应用将要到来的5件事

## 说明
- [原文链接](http://www.pcquest.com/2017-predictions-five-things-to-come-for-cloud-native-applications/)
- [翻译：@alwqx](https://github.com/alwqx)
- [项目地址](https://github.com/alwqx/translate)
- <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/80x15.png" /></a>

毋庸置疑，2016年属于容器(container)。随着上游玩家（VMware启动vSphere，集成Container）入局，相继开发了很多开源项目。
作为容器化背后的动力，云原生基础设施的地位不可否认。

我们期待2017会有哪些事情呢？

## Kubenetes将打破容器调度三足鼎立现状
2016年，容器调度领域出现了Docker Swarm、Kubernetes和Mesos争霸的局面。我们大胆预测2018年Kubernetes
会成为领袖。我们已经看到，来自用户、厂商和开源社区对Kubernetes的兴趣不断增加。明年，我们将会看到Kubernetes
吸引更多的用户、增加更多产品部署和特性，凭借自身的魅力吸引更多旁观者。

## 容器会使用更多的虚拟化技术
目前容器所依赖的技术内置于Linux Kernel中，包括control groups和namespace，它们将主机上的容器相互隔离开来。
但是许多公司已经在实验使用更加轻量的操作系统，以及烧录到现代CPUs中的新功能来为容器供更加轻量的虚拟机。
这项措施很可能增加容器的隔离性和安全性，免去了额外的付出。我们预测即将到来的一年你会听到很多关于这的声音。

## 容器持久化技术走向成熟并走向生产
迄今，大部分容器还是“无状态”的，换句话说，容器关闭删除后，容器中的数据会被删除，所以任何应用状态数据要放在外部数据库
或者其它形式的存储服务。主要是因为目前市场上没有成熟的容器持久化技术。但是，随着新功能的出现，比如Kubernetes
的PetSet，以及将会出现的像来自PortWorx新技术，还有vSphere支持Docker volume的驱动。不久就会出现
更加成熟的容器持久化技术，有状态的服务也会应用到生产中。

## 容器安全方案爆发式增长
大部分容器用户最关心的是安全，很多问卷中安全都排在前列。本应这样，因为涉及容器安全性的问题非常广泛。
Linux容器和主机共享内核，因此暴露了易被渗透的安全边界。容器网络安全也处在婴儿期。但是隧道尽到是光明，因为越来越多的产品
开始使用容器，为了确保应用和数据不易被攻击，公司对于安全方案的需求会不断增加。

## Pivotal Cloud Foundry会得到应有的荣誉
过去几年容器技术几乎吸引所有的聚光灯。同时，Pivotal Cloud Foundry (PCF)的云原生应用平台
基于忠诚的云原生开发者和操作者默默构建了大量客户。这家公司今年营业额超多2亿美元大关。这表明Pivotal正健壮
持久增长。Pivotal的Spring Boot框架一直以天文数字般的速度增长，每个月超多2.5亿下载量，这也增加了客户
对PCF运行时的兴趣。明年将会是PCF闪耀的年份。
