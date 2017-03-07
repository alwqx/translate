# RESTful API模式系列二：API设计者的职责

<!-- TOC -->

- [RESTful API模式系列二：API设计者的职责](#restful-api模式系列二api设计者的职责)
    - [说明](#说明)
    - [应用](#应用)
    - [API代码](#api代码)
    - [客户端](#客户端)
    - [译者说](#译者说)

<!-- /TOC -->

## 说明
- [原文链接](http://restful-api-design.readthedocs.io/en/latest/scope.html#the-application)
- [翻译：@adolphlwq](https://github.com/adolphlwq)
- [项目地址](https://github.com/adolphlwq/translate)
- 翻译/校对：1.5h
- <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/80x15.png" /></a>

在完全深入RESTful API设计之前，详细了解RESTful API设计者的工作细节很重要。

APIs之间不是孤立的，对于API中已经存在的应用或者服务的API，新的API要和它们在功能上独立开来。在我看来，API设计者的职责是双重的：
1. 足够理解新建API在应用中的重要细节，这样你就能决定哪些功能需要暴露、如何暴露，以及哪些功能可以排除。
2. 对API中的功能建模，要能解决出现的所有使用场景，同时尽可能遵守RESTful原则。

RESTful API设计中涉及到3个不同的组件：应用、API代码和客户端。下图描述了组件间的相互关系：

![](http://restful-api-design.readthedocs.io/en/latest/_images/scope.png)

## 应用
应用和为它提供的API之间要相互独立。也许应用是GUI程序，你需要为它提供可编程接口。也许应用只能通过你设计的API访问。

和其它任何应用一样，需要设计API的应用**也有它自己的状态**。“状态”是动态的，执行很多操作后状态会改变。**状态和状态上的操作，应该被建模并暴露到API中**。

理解应用状态的最简单方法是把它描述成应用数据模型（application data model），可以表示成实体-关系图（ER图）。实体-关系图能列出应用状态中实体的细节，以及它们间的关系。

一些情景中，很容易创建实体-关系图。假设一个web应用把所有状态存在数据库中，我们很容易从数据的schema中得到关系图。其它一些没有严格定义的情景下，API设计者的工作会难一些。这时，为应用创建ER图就真的很有用。对你来说这是难得的锻炼机会，它帮助你更好地理解应用。更重要的是，它会帮你设计出更好的RESTful API。我们一会详细讨论这个。以后的例子中，我都假设我们已经有了实体-关系图（ER diagram）。

除了理解应用的状态和状态上的操作外，你还需要应用程序的入口（`entry point`），它让你能获取和更改应用状态。这个“入口”完全由应用决定，可以有多种形式。我们把这个入口称为应用程序接口（ application interface），它的正式称呼是**API**。不同的是接口不用于外部消费甚至完全没有文档记录（译者注：`正规软件开发中代码规范也是要求写API文档的`）。为了不产生疑惑，我们不会把接口称为API，API这个术语保留给我们将要设计的RESTful API。

## API代码
API代码的任务是通过应用接口获取应用的状态，同时提供状态上的操作，把应用接口暴露成RESTful API。在应用程序接口和RESTful API之间有一个转换步骤：适配应用数据模型，并且符合RESTful风格。

**转换的结果是形成RESTful风格的资源、资源上的操作以及资源之间的关系**。（译者注：API中只有`状态`的概念，RESTful后形成`资源`的概念）我们把这些称为RESTful资源模型（RESTful resource model）。

**资源**是任何RESTful API的基础，我们会在[resources](http://restful-api-design.readthedocs.io/en/latest/resources.html)中详细介绍。现在，我们只需要把资源理解成ER图中的实体（这也是为什么应用中没有实体时我建议你画ER图）。

资源间的关系通过超链接表示。这一点也是设计RESTful API的基本原则（[fundamental principles](http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven)）。资源通常响应有限的操作（通常4个），这也是RESTful架构的第二个风格。

当把应用模型对象转换成RESTful里的资源时，下面两个工具函数很有用：
- `to_resource()`：从应用模型中获取一个对象，然后转换成资源。
- `from_resource()`：把资源转换成应用模型中的对象。

后面不会再讨论这两个函数，当应用数据模型和资源模型相似时，函数会很简单；不相似的话会很复杂。

## 客户端
客户端通过标准HTTP协议消费RESTful API。理论上，RESTful API也可以在其它协议之上提供。但是，由于HTTP协议非常广泛，把RESTful API映射到另一种协议在真实场景中意义不大。因此，本文仅限于用HTTP协议相关术语描述RESTful协议。

客户端通常使用HTTP库来访问RESTful API。HTTP已经成为一个相对复杂的协议，许多目标平台/语言都有优秀的库。因此使用这些库很合理。

在某些情况下，可能有必要在HTTP库之上使用通用的REST库。但是，由于RESTful API中有一些不一致的约定，因此通用的REST库适用于特定情况下的API。

## 译者说
本篇介绍了RESTful中的三大组件：应用、API代码和客户端。读者要理清**状态**和**资源**之间的对应关系。RESTful API可以有很多实现方式，因为**HTTP**应用广泛，有非常多的库，所以RESTful API通常用HTTP协议实现。
