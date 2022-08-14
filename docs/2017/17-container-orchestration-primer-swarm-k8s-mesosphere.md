# 容器编排初探：探索Docker swarm mode、Kubernetes和Mesosphere

## 说明
- [原文链接](https://insights.hpe.com/content/hpe-nxt/en/articles/2017/02/the-basics-explaining-kubernetes-mesosphere-and-docker-swarm.html)
- [翻译：@alwqx](https://github.com/alwqx)
- [项目地址](https://github.com/alwqx/translate)
- [tt](https://github.com/alwqx/tt)：自动生成翻译模板
- 用时: 5h
- <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/80x15.png" /></a>

容器，提供轻量化打包应用的方式，是任何DevOps的重要组成部分。但是你准备如何管理这些容器？现有的容器编排程序--[Kubernetes](https://kubernetes.io/)、[Mesosphere Marathon](https://mesosphere.github.io/marathon/)和[Docker Swarm mode](https://www.docker.com/products/docker-swarm)，很好地帮助你管理容器，避免你焦头烂额。

在谈论它们前，我们先来回顾下基础。根据[451Research调查](https://451research.com/report-long?icid=4095)，容器是增长最快的cloud-enabling技术，主要是因为[容器比虚拟机使用更少的资源](http://www.networkworld.com/article/3068392/cloud-storage/containers-vs-virtual-machines-how-to-tell-which-is-the-right-choice-for-your-enterprise.html)。毕竟，VM不仅仅运行操作系统，而且还运行操作系统所有硬件的虚拟副本。相反，容器只需要足够的操作系统和系统资源即可运行应用程序实例。

你的CFO（首席财务官）的理解是：相同的计算机硬件，你运行的容器实例是虚拟机的4-10倍。这意味着相同的数据中心负载下，你可以运行更多的应用程序。何乐不为呢？

另外，系统管理员也喜欢容器，他们可以非常容易地使用容器部署应用。“容器让你的应用瞬间有了可移植性”，Linux kernel领导开发者James Bottomley讲到。

容器技术2000年随着FreeBSD的jails就出现了，直到2013年Docker技术出现人们才关注它。然后，每个开发者、每位CTO都想要部署容器。根据451 Research，在2016年，容器技术市场的收入为7.62亿美元。到2020年，预计年度容器领域收入将达到27亿美元，复合年增长率为40％。

现在还有2个问题：如何保证容器安全、如何部署和管理容器。

## 容器需要管理
和云基础设施的任何其他组件一样，容器需要监控和控制。否则，你将根本不知道你的服务器上运行的是什么。

容器技术比如Docker可以和DevOps工具一起使用，Ansible、Chef、Puppet，但是这些工具并没有专为容器进行优化。正如[DataDog](https://www.datadoghq.com/)，一家云监控公司，在其Docker真实使用报告中指出，“容器的短生命周期和增加的部署密度使得基础设施监控愈加重要”。需要被单独监控的事物以指数的数量级增加。

大部分监控方案以主机为中心，而不是角色为中心，它们偏离市场需求。

一般有两类监控工具。一类是编排，它是新的术语，指容器的集群化和调度。少部分开发者涉及容器编排。另一类是容器管理，负责管理容器化应用和组件任务。

下面进入Docker swarm mode、Kuberbetes和Mesosphere DCOS的介绍。这些开源工具互相不可替换，它们不直接竞争。一定程度上，它们都提供以下特性：
- **Provisioning**：这些工具在容器集群中提供或者调度容器，还可以启动容器。理想情况下，它们会根据你的需求，例如资源和部署位置，在最佳VM中启动容器。
- **Configuration scripting**：脚本保证你把指定的配置加载到容器中，和[Juju Charms](https://jujucharms.com/)、[Puppet Manifests](https://www.digitalocean.com/community/tutorials/configuration-management-101-writing-puppet-manifests)或[Chef recipes](https://docs.chef.io/recipes.html)的配置方式一样。通常，这些配置用YAML或JSON编写。
- **Monitoring**：容器管理工具跟踪和监控容器的健康，将容器维持在集群中。正常工作情况下，监视工具会在容器崩溃时启动一个新实例。如果服务器故障，工具会在另一台服务器上重启容器。这些工具还会运行系统健康检查、报告容器不规律行为以及VM或服务器的不正常情况。
- **Rolling upgrades and rollback**：当你需要部署新版本的容器或者升级容器中应用时，容器管理工具自动在集群中更新你的容器或应用。如果出现问题，它们允许你回滚到正确配置的版本。
- **Service discovery**：在旧式应用程序中，您需要明确指出软件运行所需的每项服务的位置。容器使用服务发现来找到它们的资源。以上听起来是不是很熟悉？分析师Dan Kusnetzky指出，容器模式很像service-oriented architecture（SOA），SOA在2000年代得到极大的关注。对于那些错过了这种技术的人，SOA是指将应用程序分解成单独的独立服务。SOA的技术障碍是：SOA使用网络通信，这比进程间通信慢一个数量级。容器运行远比SOA快，因为容器倾向于使用同一台机器上的资源。这些工具帮助前端应用，比如WordPress，通过DNS或者代理动态发现依赖的MySQL实例。
- **Container policy management**：你希望容器运行在哪里？你希望每个容器分配多少CPU？所有这些需求都可以通过设置正确的容器部署策略实现。
- **Interoperability**：当然，容器要能够和已有的IT管理工具兼容。

最后，这三款容器管理工具都和很多云平台进行集成，包括OpenStack Magnum和Azure Container Services。

你也可以构建自己的容器管理工具，但是何必要重造轮子呢？相反，这三款工具都是建立在开源基础之上的。你可以添加任何你需要的特新。从头开始没有意义。

介绍完一般性，我们介绍具体细节。

## Docker swarm mode
如果你是容器新用户，你可从Docker开始，它是第一个吸引大量用户的容器程序。如果是使用Docker，那么Docker swarm是很自然的选择，它是Docker开发人员设计开发的。

Docker 1.12版本中，Docker的目标是内置容器编排功能，称为docker swarm mode。Docker Swarm，Docker软件栈中独立的编排器，已经影响这个内置的编排器。Swarm mode让用户控制容器整个生命周期，不仅仅是容器集群化管理和调度。

Docker Swarm和Swarm mode之间区别在哪里？Docker 1.12中，[Swarm mode](https://docs.docker.com/engine/swarm/)已经成为Docker Engine的一部分。伸缩、容器发现和安全都包含在最小的设置中。[Docker Swarm](https://docs.docker.com/swarm/overview/)是一种较旧的独立产品，曾经用于管理Docker集群。Swarm mode是Docker内置的集群管理器。

Swarm mode使用单节点概念，并且可以扩展成Swarm集群。通过`docker swarm init`命令切换到swarm mode，通过`docker swarm join`添加更多的节点。

另外，Docker 1.12和更高版本和swarm mode都支持滚动更新、节点间传输层安全加密、负载均衡和简单的服务抽象。

简言之，Docker swarm mode可以在多个主机之间传播容器负载，它允许你在多个主机平台上设置swarm（即群集）。这还需要你在主机平台上简单配置，包括集成（这让样容器可以在多个主机间通信了）和隔离（隔离和保护不同的容器工作负载）。你可能还需要虚拟网络来满足你的需求。

## Kubernetes
Kubernetes最初由有谷歌开发的开源容器管理工具。自Kubernetes推出以来，它已被移植到Azure、DC/OS以及几乎所有你叫的上名字的云平台。唯一的例外是Amazon Web Services（AWS），尽管CoreOS已经帮助用户能够在AWS上部署Kubernetes集群。

现在Kubernetes有Linux基金会下的[Cloud Native Computing Foundation](https://www.cncf.io/)管理。另外，很多公司都发布了Kubernetes发行版，包括[Red Hat OpenShift](https://www.openshift.com/)、[Canonical Distribution of Kubernetes](https://www.ubuntu.com/cloud/kubernetes)、[CoreOS Tectonic]()和[Intel Mirantis](https://www.mirantis.com/company/press-center/company-news/openstack-kubernetes-mirantis-collaborates-intel-google/)。

Kubernetes提供高度的互操作性，以及[自我修复](https://kubernetes.io/docs/user-guide/replication-controller/#what-is-a-replication-controller)、[自动升级回滚](https://kubernetes.io/docs/user-guide/deployments/#what-is-a-deployment)以及[存储编排](https://kubernetes.io/docs/user-guide/persistent-volumes/)。但是，负载均衡还很困难。我相信最终Kubernetes会很容易实现在集群内部运行外部负载均衡器，但这项工作目前还在进展中。

Kubernetes擅长自动修复问题，这方面Kubernetes做得很好，以至于你都没发现容器崩溃过。为了发现容器崩溃，你需要添加中心日志系统。

## Mesosphere Marathon
Marathon是为Mesosphere DC/OS和Apache Mesos设计的容器编排平台。DC/OS是基于Mesos分布式系统内核开发的分布式操作系统。Mesos是一款开源的集群管理系统。Marathon提供有状态应用程序和基于容器的无状态应用程序之间的管理集成。

虽然Marathon有一个用户界面，使你把它当作一个应用程序，把它当作一个Mesos上管理容器的框架可以更容易理解Marathon。Marathon设计的是DevOps中开发者部分，因为容器通过RESTful API和Marathon协同工作。

Marathon有很多特性，包括高可用、服务发现、负载均衡。如果你在DC/OS上运行Marathon，你还可以获得虚拟IP路由特性。

但是，Marathon只能运行在Mesos软件栈上。此外，某些功能（例如身份验证）仅适用于DC/OS上的Marathon。这在你的堆栈上增加一个抽象层。

## 哪一款适合你
最终，这取决于你的需求。Mesos和Kubernetes主要关于运行集群应用程序。Mesos专注于通用调度，以插件的方式提供多个不同的调度器。Google最初设计Kubernetes作为从容器构建分布式应用程序的环境。

Docker swarm mode扩展了现有的Docker API，使得一个集群的机器更容易与单个Docker API一起使用。如果你的公司有专业的Docker员工，你可能已经在运行swarm mode了。如果swarm mode为你工作良好，为什么还要切换到另一个系统？ Marathon在处理你的容器和你的旧应用程序有独特的优势。

幸运的是，您可以混合匹配这些工具，根据公司的需要来组合它们。这三个同居可以很好地彼此协作。这并不容易，但这么做是可行的，也许这是一个很好的方法来探索不同选择。

## 容器管理：领导需要注意的事情
- 要想最大化利用容器，你需要优秀的容器管理工具。三个选择是Kubernetes、Mesosphere和Docker Swarm。虽然它们有不同的特性，但它们都支持容器监控、管理。
- 除了支持容器管理，Mesosphere还支持管理数据中心。
- Docker swarm mode目的在于通过提供容器曾调度来简化集群管理。例如，它可以约束容器运行在哪个节点，与Docker Remote API交互，帮助你决定新容器应该调度到集群的哪个节点上。
- Kubernetes拥有广泛的行业合作伙伴，包括英特尔，微软，红帽和Mirantis。

## 译者说
本文首先介绍了容器技术的基础知识，说明了容器技术的前景和市场份额。容器技术的重点之一是容器的管理编排。作者介绍了三种编排工具的共同特点和各自的特性。表明企业应该根据自身需求来选择使用那一款工具或者混合使用。
