# CoreOS的Tectonic新发行版支持Kubernetes自我管理

## 说明
- [原文链接](http://thenewstack.io/coreos-offers-self-hosting-kubernetes-new-tectonic-release/)
- [翻译：@alwqx](https://github.com/alwqx)
- [项目地址](https://github.com/alwqx/translate)
- <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/80x15.png" /></a>

![](https://cdn.thenewstack.io/media/2016/12/2544ad27-philips.jpg)

为了充分利用Kubernetes原生管理容器化应用的能力，CoreOS更新了自家的Kubernetes商业发行版Tectonic，增加了无停机更新的功能。

CoreOS的CTO [Brandon Philips](https://twitter.com/BrandonPhilips)在本周纽约举办的Tectonic Summit的keynote中提到:“我们现在已经做到使用完全相同的APIs和函数监控Kubernetes和applications。我们把所有的功能集成到Tectonic控制台钩子函数中，你只需要点击一下按钮就可以完成部署。”

Philis还提到，“目前为止，Tectonic和Kubernetes的安装过程繁琐到令人抓狂。本质上是因为，人们不得不手动去更新整个分布式系统。”

“人们ssh登录到每个节点上人工修改文件，或者至少写个脚本来执行这些任务。和管理Kubernetes应用相比这些操作需要一系列技能。”

一篇[CoreOS博文](https://coreos.com/blog/rkt-and-kubernetes.html)在谈到自我管理能力时指出，“事实上，掌握**kubectl**和相关工具来管理Kubernetes应该转换为，将如何安装Kubernetes并保证它运行放在第一位。”

“这就是为什么我们非常努力地投入到上游代码，实现了Kubernetes自我管理的功能。”Philips讲到。

Philips把自我管理的能力类比为Linus Torvalds使用Linux来编译新版本的Linux。Linus Torvalds使用minix平台编译第一个Linux版本。但是Linux稳定之后，Linus就把编译器移植到稳定的Linux上，来编译新的Linux。

Kubernetes自己可以保证，某个pod故障后，它会运行一个新的pod来替代挂掉的。在这次新的Tectonic版本中，被启动的不再是新的pod，而是新版本的Kubernetes，它被打包到一组pods中。这里Tectonic利用了Kubernetes新的安装工具:kubeadm。

Kubernetes的典型升级中，与工作节点相比，所有的控制节点是优先升级的理想节点。Tectonic升级时，会在控制节点为新版本保留空间。一旦新版本运行起来，Jobs会从每个旧组件过渡到对应的新组件，直到更新完成。下面的视频介绍了Kubernetes自我更新的过程（需自备梯子）。

[视频](https://www.youtube.com/watch?v=tXyV3IQ8-0k)

这个方法和CoreOS更新自身的Linux发行版类似（[最近更名为Container Linux](http://thenewstack.io/self-driving-infrastructure-makes-internet-secure/)）。由于Tectonic是分布式应用，所以组件的更新顺序是指定的，通常以api server，scheduler，proxy，kubeket的顺序更新。

CoreOS自身通过组件[CoreUpdate](https://coreos.com/products/coreupdate/)、以容器的方式更新，这些操作在管理控制台里执行。

在Tectonic更新发布之前，CoreOS为企业测试提供了获取alpha和beta版本的渠道。

如果更新后出现问题，可以通过机制回退到以前的版本。Kubernetes的数据存储、[etcd](http://www.thenewstack.io/tag/etcd)都会备份上一个版本的信息。我们也提供了手册指导用户从不同的故障中恢复，比如scheduler故障。

Philips还讲到，大部分企业部署案例中，自动更新相比于人工更新表现更加出色。

CoreOS不止于仅仅自我管理Kubernetes，这项技术会应用在未来的软件中。毋庸置疑，其它发行版也会使用这项技术。

CoreOS还发布了[Dex 2.0](https://github.com/coreos/dex)，基于[openID connect](http://openid.net/connect/)的认证服务。openID connect是一个广泛应用的认证协议，它可以通过加密令牌管理Kubernetes上的用户、与企业用户的轻量目录访问协议（LDAP）连接。版本2允许Kubernetes不依赖外部数据库运行Dex。Dex使用Kubernetes的APIs来持久化认证数据。但是旧版本需要数据库。

“我个人认为自我驱动技术的想法有很好的前景，那会是我们的最终方案。”DigitalOcean技术经理[Joonas Bergius](https://twitter.com/joonas)谈到新版本Tectonic时如是说。

[Tectonic](https://tectonic.com/?_ga=1.162053372.1955225110.1481644180)现在免费支持10的节点。
