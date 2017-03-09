# RESTful API设计系列一：简介

<!-- TOC -->

- [RESTful API设计系列一：简介](#restful-api设计系列一简介)
    - [说明](#说明)
    - [简介](#简介)
    - [译者说](#译者说)

<!-- /TOC -->

## 说明
- [原文链接](http://restful-api-design.readthedocs.io/en/latest/intro.html)
- [翻译：@adolphlwq](https://github.com/adolphlwq)
- [项目地址](https://github.com/adolphlwq/translate)
- <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/80x15.png" /></a>

## 简介
这篇文章里，我尝试写下我心中的、真正优美的[RESTful API](http://bitbucket.org/geertj/rhevm-api/wiki/Home)设计原则。这些经验来自于我之前参与的项目[twice](http://fedorahosted.org/rhevm-api/)，它是红帽企业版中的虚拟化产品。在API设计阶段我们必须解决真实场景中的很多问题，同时我们不希望添加那些容易实现的**非RESTful**或者**类RPC**接口到我们的API中。

在我的理解中，真实的RESTful API提供了问题的答案，你不需要再去看介绍文字。但是做到如此不可避免要遇到很多问题：
- 是否要正规描述资源？
- 如何创建有帮助、自动化的命令行接口？
- 如何做轮询、异步以及一些非标准的请求类型？
- 如何处理没有做好RESTful映射的操作？

另一方面，优美的RESTful API不会轻易偏离RESTful架构设计原则。（A beautiful RESTful API on the other hand is one that does not deviate from the principles of RESTful architecture style too easily.）比如，一个很重要但人们不经常提到的设计因素是，用户完全自动发现API，人们可以通过web浏览器访问API，不需要外部的文档指导。这个问题我会在[Forms](http://restful-api-design.readthedocs.io/en/latest/forms.html)详细描述。

我不会在文中介绍RESTful的基础知识，因为很多文章都介绍过了。如果你希望学习基础，我推荐阅读[the Atom Publishing Protocol](http://tools.ietf.org/html/rfc5023)和Roy Fielding的[博文](http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven)。

文中的观点都属个人，它们都是基于我在[rhevm-api mailing list](https://fedorahosted.org/mailman/listinfo/rhevm-api)的讨论。我要感谢那些贡献的朋友：Mark McLoughlin, Michael Pasternak, Ori Liel, Sander Hoentjen, Ewoud Kohl van Wijngaarden, Tomas V.V. Cox还有一些我忘记的伙计。

## 译者说
以下几个问题需要重视：
- 是否要正规描述资源？
- 如何创建有帮助、自动化的命令行接口？
- 如何做轮询、异步以及一些非标准的请求类型？
- 如何处理没有做好RESTful映射的操作？