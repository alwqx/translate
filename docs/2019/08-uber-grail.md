# 使用Grail进行大规模基础设施管理
![](https://1fykyq3mdn5r21tpna3wkdyi-wpengine.netdna-ssl.com/wp-content/uploads/2018/03/Header.jpg)

- [原文链接](https://eng.uber.com/grail/)
- [翻译：@AdolphLWQ](https://github.com/adolphlwq)
- [项目地址](https://github.com/adolphlwq/translate)
- 用时: 10h
- <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/80x15.png" /></a>

易于获取当前系统状态对于规模化构建、维护基础设施至关重要。由于Uber的商业持续扩张，我们的基础设施在规模和复杂性上不断增加，使得我们必要时获取所需信息变得很困难。

为了解决这个问题，我们开发了Grail，一个聚合状态信息并在一个全局视图中展示、横跨多个[数据中心和区域](https://cloudacademy.com/blog/aws-global-infrastructure/)的平台。有了Grail，我们可以更容易地开发快速、健壮的运维工具。

继续阅读以了解Grail如何通过图模型，根本性地改变Uber工程部门操作存储的方式，使团队更容易缝合不同源头的数据。

## 设计简单的管理方式
2016年末，为了支撑不断增加的负载，我们把所有数据库主机从旋转式硬盘更新到固态硬盘。有一步很重要，就是依然能够鉴别和追踪使用旧硬件的成千上万数据库。

那时候我们没有容易的方式获取设备的当前状态，并且还要追踪大量脚本和任务。这驱使我们寻找不同的方法来开发大规模运维工具，需求如下：
1. 持续收集整个基础设施的状态。
2. 唯一的全局视图。
3. 低延迟从所有数据源获取所有数据。
4. 关联所有数据源的数据。
5. 简单添加和删除数据源。

## Grail简介
不像[Metricbeat](https://www.elastic.co/downloads/beats/metricbeat)和[osquery](https://osquery.io/)等类似信息收集系统，Grail不收集特定领域的信息，它的角色是一个平台，以高可用和响应式的方式聚合、链接和查询来自不同数据源的数据，例如主机、数据库、部署和所有权等信息。它高效隐藏了实现细节。

![](https://1fykyq3mdn5r21tpna3wkdyi-wpengine.netdna-ssl.com/wp-content/uploads/2018/03/image6-768x387.png)

此外，你可以接近实时的方式，快速获取下面问题的答案：
1. 哪些主机当前空闲空间超过4TB？
2. 某个团队的数据库使用多少磁盘空间？
3. 哪些数据库运行在旋转磁盘上？

如果你的服务和主机很少，这些问题就不重要。你只需要写一个脚本，在需要的时候直接收集信息就行了。但是以Uber的规模，当你有一堆服务和数十万主机时，这种方法就失效了。节点太多响应就会慢，查询完后数据关联会出错，结果也不能反映真实情况了。大规模场景下很难及时收集状态。

一个关键结论是“不存在唯一的真理来源”。数据中心和系统的信息总是分布在多个地方，只有把它们关联起来才能做决策。更复杂的是这些状态一直在变：主机的空闲磁盘空间在变、供应新的存储集群、并行发生的其它事件。整个系统的状态不可能实时获取，只能接近它。

## 规模化维护
Uber的存储平台团队开发维护的存储系统支撑了拍字节的关键任务数据，我们的运维工具有一套标准的自我修复范式，有三个简单步骤：首先我们收集系统状态，然后和正常状态比较，最后处理异常数据。

![我们构建的所有操作工具都经过这个循环，直到系统状态收敛于正常状态](https://1fykyq3mdn5r21tpna3wkdyi-wpengine.netdna-ssl.com/wp-content/uploads/2018/03/image1.png)

如前文所述，在大规模场景下，不使用Grail这样的聚合平台是很难收集状态。举个例子，当我们想获取所有运行主机当前状态时，比如trips数据。首先我们要先找出哪些主机包含这个数据。接下来我们要连接到这些主机并收集当前状态。最后转换并展示结果。

有了Grail，我们只需要运行一条查询语句，就可以获得需要的信息：
```
TRAVERSE datastore:trips (
  SCAN cluster (
    SCAN db (
      SCAN host (
        FIELD HostInfo
      )
    )
  )
)
```

结果以json文档的形式返回，与查询结构非常相似，对代码友好。下面的代码片段展示了运行上面查询语句的精简版结果：
```json
{
	"__id": "datastore:trips",
	"cluster": [{
		"__id": "cluster-trips-us1-44",
		"db": [{
			"__id": "cluster-trips-us1-44-db26",
			"host": [{
				"__id": "host:database862-sic1",
				"HostInfo": {
					"cpuCount": 24,
					"puppetRole": "database",
					"memory": {
						"freeBytes": 1323212425,
						"totalBytes": 137438953472
					},
					"disk": {
						"freeBytes": 48289601723,
						"totalBytes": 1598689906787
					}
				}
			}]
		}]
	}]
}
```

## 拼接数据
Grail围绕对Uber基础设施的两项观察进行设计。第一，基础设施中节点和节点间的联系可以很自然地建模为图。

![有了Grail，基础设施被表示成相互连接的节点图](https://1fykyq3mdn5r21tpna3wkdyi-wpengine.netdna-ssl.com/wp-content/uploads/2018/03/image2-768x325.png)

模型图中的节点通过唯一的键进行标识，键由类型和名字以`type:name`的形式构成。数据源使用节点键将包括属性和连接的数据附加到节点上，因此数据就好像被节点键标识一样。节点的键空间是全局的，而属性和连接的键空间相对于节点是局部的。

Grail的对象模型是这样的，建模图中的节点由数据源生成的属性和连接隐式定义，这意味着下面条件至少满足一条节点A存在：
1. 数据源产生的数据有A的属性。
2. 数据源将节点A与至少一个其它节点关联。
3. 数据源至少将其它一个节点与A关联。

第二点是单个基础设施概念，比如主机或数据库的信息是去中心化的。这意味着获取完整数据视图需要结合不同系统的信息。

Grail的方法是让每个数据源提供自己所属子图来解决去中心化问题。这些子图可能会有重叠，因为数据源可能把属性和连接附加到同一个节点。

![图4：每个数据源提供它们自己的子图，并且将数据附加到节点键上。查询时将这些子图结合起来，提供整个基础设施的视图](https://1fykyq3mdn5r21tpna3wkdyi-wpengine.netdna-ssl.com/wp-content/uploads/2018/03/image5-768x361.png)

上图最上面展示了三个子图。实线和颜色表示子图由哪些数据源提供，虚线表示整个图。下面的图表示从Grail用户的角度看到的视图。

通过方法，我们可以自动更新数据源的所有数据。对不同数据源，我们能够以不同的速度并行更新数据。

![](https://1fykyq3mdn5r21tpna3wkdyi-wpengine.netdna-ssl.com/wp-content/uploads/2018/03/image3-768x433.png)

上图中，数据源1在键`HostInfo`下附加属性，数据源2键`ServiceInfo`下附加属性，并将此节点和关联类型`Service`下的一系列服务建立联系。

## 数据导航
随着设计的实施，我们需要一种简单的方法，能够在图中执行特定遍历。我们调研的技术中没有能很好符合需求的。比如，[GraphQL](http://graphql.org/)需要定义模式，且不支持映射和节点间命名关联。[Gremlin](https://en.wikipedia.org/wiki/Gremlin_(programming_language))似乎可以，但实现并单独使用它非常复杂。所以我们开发了自己的方案。

我们的查询语言YQL，用户只需要指定一个开始节点集，然后通过后面的条件遍历图，同时与沿图属性中的字段交互。举个例子，下面的查询语句列出了所有满足条件的主机：空闲内存大于40G、剩余磁盘空间大于100G且是SSD：
```
TRAVERSE host:* (
  FIELD HostInfo
  WHERE HostInfo.disk.media = “SSD“
  WHERE HostInfo.disk.free > (100*1024^3)
  WHERE HostInfo.memory.free > (40*1024^3)
)
```

## 迁移到内存
从发布起Grail的架构经历多次迭代。起初，它是我们之前数据库运维工具的内部组件。第一版迭代受[TAO](https://www.facebook.com/notes/facebook-engineering/tao-the-power-of-the-graph/10151525983993920/)启发，基于Python开发，使用redis存储图。当它变得低效时，我们决定把它作为一个单独服务用Go重写，使用共享的ElasticSearch集群存储。但是随着时间的推移，我们发现这个方案在快速、有效摄取和查询所需信息时，缺少伸缩性和低延迟。

我们重新思考它的架构，把之前存到共享ElasticSearch集群中的数据迁移，改为直接存储到每个查询节点上定制的内存数据库里。

当前Grail的高层架构包含三个组件：
1. *Ingesters*，从配置的数据源收集数据。
2. *Coordination*，确保严格的数据更新顺序。
3. *Query Nodes*，为数据获取提供水平扩展能力

![](https://1fykyq3mdn5r21tpna3wkdyi-wpengine.netdna-ssl.com/wp-content/uploads/2018/03/image4-1-768x405.png)

Ingesters周期性从预配置的数据源中收集数据，然后通过Coordination集群传输，最后存储到每一个查询节点上的datastore中。Coordination由定制的内存[Raft集群](https://raft.github.io/)实现，基于[etcd Raft库](https://github.com/coreos/etcd/tree/master/raft)开发。Raft协议确保数据更新和被存储到datastore的顺序，同时确保重启后数据一致。`Coordination Nodes`和`Query Nodes`都包含了存储在datastore中的每个数据源最新数据更新。当Raft-logs被截断时，Coordination Nodes只使用datastore中的数据来创建当前数据的快照。

datastore是一个简单的键/值抽象，数据源的名字作为键，不同的键下面存储每个数据源最新的数据更新。所有数据源的数据分开存储，只有在执行查询时才聚合起来。

Grail通过在每个区域运行各自的实例，为我们提供基础设施的全局视图。每个实例负责从本地主机和服务收集数据。查询节点根据配置追踪本地和远程区域上的raft-log。当执行查询时，查询引擎把本地和远程信息结合起来。

为了扩展Grail，我们可以部署多个coordination集群、扩展查询引擎来支撑分布式查询，以便将来可以增加数据吞吐量和大小。

## 处理精确问题
在与分布式系统交互时，考虑到信息不准确非常重要。不管数据如何提供，来自聚合平台或直接来自源头，在系统变化时不可避免会有延迟。分布式系统不是事务的，你不能用一致的快照获取它。不管基础设施规模如何，这些条件都是对的。

我们的运维工具使用Grail的信息做决策。当这些决策需要改变系统时，在应用改变之前，我们总是确保双重检查源头的信息。举个例子，当主机端的代理程序被分配任务时，代理程序在执行任务前会先检查先决条件是否满足，比如判断主机是否有足够的磁盘空间。

## 关键点
正如前面所讨论的，高效基础设施管理需要深刻洞察系统状态。当规模很小时这很简单，你只需要按需查询数据即可。但是这个方法不适用大规模系统，这时你要将信息聚合到一处。正如我们在实践中学到的，当有数十万主机和许多系统时，快速获取合理且最新的系统状态很重要。

最后Grail的优势可以总结为三点：
1. 所有数据都被聚合到支持通用查询API的单一共享模型。
2. 低延迟查询所有地区当前状态。
3. 团队可以附加自己特定领域的概念，并将它们与来自其它领域的相关概念关联起来。

目前，Grail服务我们存储方案的大多运维工具，并且对基础设施的各个方面都有几乎无数的使用案例。事实上随着信息范围不断增加，会有更多的使用案例。

## 学习总结
>Grail has made it easy for us to build tooling that quickly and robustly performs complex operational tasks.

这句话很容易意会，但用准确、精炼的语言表达出来还很难。我是这么翻译的`有了Grail，我们可以更容易地开发快速、健壮的运维工具。`

>In figure 5, Data Source 1 attaches a property under the HostInfo key, while Data Source 2 attaches a property under the ServiceInfo key and associates the node with a range of other service nodes in the graph under the association type Service.

这句话后一句不好理解、不好翻译

>lets users specify a starting collection of nodes and then traverse the graph by following associations while interacting with the fields stored in the properties along the way

不好翻译，借鉴工具后翻译如下`让用户指定一个节点集，然后通过后面的关联遍历图，同时与沿图属性中的字段交互`。

>Over time, however, this solution also lacked the scalability and latency we needed for fast, efficient ingestion and querying.

`但是随着时间的推移，我们发现这个方案在快速、有效摄取和查询所需信息时，缺少伸缩性和低延迟。`

>Distributed systems are not transactional—you cannot capture it with a consistent snapshot.

## 译者说
本文介绍Uber存储平台团队是如何管理自己的存储基础设施的。它们需要收集状态，然后根据这些信息执行运维任务、制定决策。

他们开发了Grail来管理，不断迭代来提升性能。将基础设施抽象成图模型，并且自己开发了工具来快速遍历图。他们面临多个挑战：分布式系统的数据延迟、不准确问题，查询性能低等。最后都很好的解决了，这些方法对我们有借鉴意义。