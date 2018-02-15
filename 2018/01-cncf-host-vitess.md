# CNCF项目通过Kubernetes扩展MySQL

## 说明
- [原文链接](http://thenewstack.io/cncf-host-vitess)
- [翻译：@adolphlwq](https://github.com/adolphlwq)
- [项目地址](https://github.com/adolphlwq/translate)
- <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/80x15.png" /></a>

一项由YouTube开发的指在跨多个服务器实现MySQL分片的技术，Vitess，成为[Cloud Native Computing Foundation](http://bit.ly/2DCTwne)第16个项目。

Vitess创建的目的是为了“热爱MySQL功能，却顾忌它扩展性的人们”，[Sugu Sougoumarane](https://twitter.com/ssougou)在The New Stack采访中说道。他是Vitess创始人之一，现在是[PlanetScale Data](http://www.planetscaledata.com/)，一家以Vitess为中心低调创业公司，的联合创始人和CTO。

他说：“人们使用MySQL时会顾虑它不是很好的扩展性，但现在有了Vitess，这个问题被解决了。”

Vitess用户分成两个阵营，Sougoumarane解释道。一个阵营是重度MySQL用户，他们知道如何在多个服务器间分片MySQL来提高性能，但是他们依然选择Vitess而不是自己维护软件。第二个阵营往往是较小的组织，单个MySQL实例满足不了他们的需求。

CNCF技术监督委员会（TOC）在本周投票赞成这项新技术，Vitess被CNCF采用表明CNCF TOC在认真考虑云原生运行面临的持久化存储的挑战。上周TOC也投票通过了[Rook云原生存储](https://rook.io/)项目。

## Vitess架构
Vitess是一个数据库集群系统，用来水平扩展[MySQL](https://www.mysql.com/)，一个由Oracle主导的开源关系数据库管理系统。Vitess将数据库在不同的服务器间分片，内部封装分片路由。应用程序使用Vitess API驱动程序而不是Oracle的MySQL驱动程序，这时MySQL实际上成为云原生数据库。 用户可以根据需要分割和合并分片。在此基础上，Vitess依靠CNCF开源[Kubernetes](https://www.thenewstack.io/tag/Kubernetes)容器编排引擎，通过使用容器集群加速自动扩展。

Vitess还支持其他的云原生功能，比如支持故障自动恢复、副本和滚动升级。Vitess支持多个分片方案，用户也可以创建自己的方案，它还附带master管理和性能管理工具。

Vitess在保持了MySQL基于SQL的一致性模型的同时还带来了NoSQL数据库的大规模可扩展性。在过去十年中，许多用户放弃了MySQL和其它兼容SQL的数据库，他们提供ACID数据一致性保证，转而选择更宽松、更容易扩展的NoSQL数据存储。

这些用户认为他们不需要实务属性，但是“当你遇到越来越多的用例，你希望从数据中获得更好的保证”Sougoumarane说。结果很多人最终在应用程序本身实现了所需的事务属性，“这些应用程序最终比他们期望的要复杂得多”他说。

当然，并非所有用户都使用NoSQL。许多组织（如Slack或Facebook）都倾向于使用MySQL，虽然它们需要更多的可扩展性，而不是数据库系统提供的自行管理能力。有一些方法可以扩展标准的关系数据库，但是会导致维护工作不断增加。最初，可以通过复制副本来扩展数据库，以便可以从多个副本中读取相同的数据。最终，写入性能成为棘手问题，因此下一步就是将不同的表分配到不同的服务器，并且最终必须将一个表分割到多个服务器上，这是一个令人头疼的问题，随着时间的推移越来越糟糕。

Vitess旨在隐藏所有这些复杂性。去年，AWS工程化自己的类MySQL的数据库服务Aurora，它具有多主数据库读写功能，高效地为服务提供了几乎无限的横向扩展能力。 像Vitess这样的技术可以提供相同的功能，但它不会将最终用户限制到特定的云服务厂商。

## 经过生产测试，Kubernetes已就绪
Vitess开始于2010年，作为内部YouTube项目构建一个代理服务器（现在称为VTGate），设计目的是池化链接（MySQL中的弱点），并缓冲可能导致MySQL实例宕机的查询。

最初Vitess运行在裸机上。但是，当YouTube在2013-14年迁移到Google Cloud时，开发人员需要调整该软件以便在容器化环境中运行，而容器管理软件非常类似于Kubernetes。 因此开发人员抽象、构建API到Vitess中，以便它可以在这样的环境中工作。

因此，“我们在Kubernetes发布后几乎已经准备好了在上面运行”Sougoumarane说。在Kubernetes帮助下，Vitess可以扩展多达数万个节点。 “如果没有Kubernetes，你将不得不编写大量的自定义启动脚本来部署。 “如果服务器出现故障，Kubernetes可以将故障节点调度到其它地方，并在新配置中重新平衡应用程序。

后端组件是用Go编写的，该软件支持MySQL5.6版本，以及MySQL派生的MariaDB10版本。支持网络文件服务器或BLOB存储实现备份。

Vitess是继Kubernetes，Prometheus，OpenTracing，Fluentd，Linkerd，gRPC，CoreDNS，containerd，rkt，CNI，Envoy，Jaeger之后CNCF的第16个项目。根据[CNCF毕业标准v1.0](https://github.com/cncf/toc/blob/master/process/graduation_criteria.adoc)，Vitess已被接受为孵化水平项目。

任何CNCF孵化项目的要求之一是，至少被三名独立终端用户在生产环境中成功使用。现在Vitess已经满足了：BetterCloud，Flipkart，Quiz of Kings，Slack，Square Cash和Stitch Labs都使用该软件，除了YouTube本身。使用范围涵盖移动应用程序，游戏公司，金融机构，零售和其他企业。

例如，Stitch Labs在Kubernetes上运行数百个分片，每秒处理数千个查询（QPS），Sougoumarane说。Slack是部署Vitess的另一个组织。该公司希望继续使用MySQL，尽管其快速增长的服务需要每天处理数十亿次MySQL事务。“我们需要一种能够提供熟悉的、完全满足SQL接口的解决方案，并且希望继续使用MySQL作为后端存储以保持我们的操作知识和舒适度，”Slack高级工程师Michael Demmer在一份声明中表示。“基于这个目的，Vitess是自然选择，迄今为止我们的表现非常好。”

Booking.com，GitHub，HubSpot，Slack和Square都积极参与该项目。