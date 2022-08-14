# 为什么我不使用Kubernetes Ingress

## 说明
- [原文链接](https://medium.com/@gabriel_ac/kubernetes-ingress-why-i-dont-use-it-c6a000321769)
- [翻译：@alwqx](https://github.com/alwqx)
- [项目地址](https://github.com/alwqx/translate)
- [tt](https://github.com/alwqx/tt)：自动生成翻译模板
- 用时: 1h
- <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/80x15.png" /></a>

不幸的是，目前的Kubernetes文档并不完善，这也是为什么很多事情非常混乱，其一就是Kubernetes Ingress。

## ingress是什么
非常简单，ingress是一层代理，负责根据hostname和path将流量转发到不同的服务上。使得一个负载均衡器用于多个微服务。

没有ingress的基础设施看起来是这样

![](https://cdn-images-1.medium.com/max/1600/1*6CC9E_ffUyNQD_9mrfZuFw.jpeg)

带有ingress的微服务架构是这样的

![](https://cdn-images-1.medium.com/max/1600/1*nvpauB85N951vqQGwNZoTw.jpeg)

所以使用ingress你不用担心有不同的负载均衡器，你可以在ingress的yaml文件中指定paths，hosts和目标服务。详情见[文档](https://kubernetes.io/docs/concepts/services-networking/ingress/)

**这很酷不是吗？**
不一定，我接下来会告诉你为什么。

> 每当我在思考生产环境的基础设施时，我总会优先想到高可用性。很明显，高可用意味着能够监控和阻止系统每一部分的宕机，以及部署新的微服务时不影响其它微服务。这意味着每次部署微服务时，部署之间完全隔离，互不影响。

根据我的经验，使用kubernetes和微服务方法后，生产环境中微服务的部署数量，从每天不到一个增长到每天20到30个。这时使用单个ingress就出现缺点了：
- 如果你需要更新配置或者添加一个服务的路径，又或者是更新ingress的hostname，这意味着（如果是持续集成环境）你不得不在你的流水线中添加一个步骤来检查每次部署的ingress配置，（即你在所有的微服务之上共享了一个单独的配置层）。或者你需要一个单独的流水线来更新ingress（即ingress没有更新前你的微服务无法部署）
- 新增一个抽象层后，会增加基础设施的复杂性，或者是存在单点故障的风险。（如果ingress无法工作，所有的微服务都无法获取）
- 我总是喜欢设计带有监控的生产环境。指标要能够反应基础设施的弹性以及避免宕机。所以使用不同的负载均衡器（ELBs）可以帮助你构建高效的监控策略，因为使用多个负载均衡器的情况下，不存在单个共享的堆栈。

这些就是我现在不使用Kubernetes Ingress的原因。另一个原因是，事实上微服务间的大部分流量发生在集群内部，只有少部分服务暴露到外部网络中。

很显然每一个应用的架构都是不同的，也不存在一个完美通用的解决方案。所以是否使用Ingress取决于你自己，但我希望可以给你一些参考。

## 译者说
Kubernetes发展迅猛，由于Google等巨头的支持是的Kubernetes正被越来越多的企业、厂商使用。但是问题也很多，显著的一个是Kubernetes文档不完善，结构不清晰，是得用户和社区存在很多疑惑，这一点kubernetes需要向Docker学习。