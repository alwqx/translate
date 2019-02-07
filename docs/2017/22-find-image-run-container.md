# docker基础：查找镜像和运行容器
	
## 说明
- [原文链接](https://rskupnik.github.io/docker-series-1-image-and-container)
- [翻译：@adolphlwq](https://github.com/adolphlwq)
- [项目地址](https://github.com/adolphlwq/translate)
- [tt](https://github.com/adolphlwq/tt)：自动生成翻译模板
- 用时:
- <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/80x15.png" /></a>

## 介绍
也许你已经听说过docker，这里我不打算深入介绍docker是什么以及它的工作原理。如果你从未听说过docker，[这篇文章](https://www.docker.com/what-docker)会帮助你熟悉docker。如果你了解docker的话我们从一些基本的功能说起：**镜像**和**容器**以及如何使用它们解决真实场景中的问题，比如想要学习一门很棒的语言又不希望花太多时间在安装语言和环境搭建上。

继续之前，需要说明三点：
- 这个系列是关于基本操作的，目的在于介绍概念，为深入学习做铺垫
- 我并不是这个领域的专家，甚至也不高级。实际上我是初学者并且也没有在任何重要项目中使用过docker。我想看到后一句话很多人已经不打算看了。
- 我正在使用Windows系统，所以我的操作是在Windows上进行的。这并不意味着你不能在Mac或Linux上参考本文，实际上只有安装部分不同，其余的应该是一样的

## 安装
参考[官方安装指导](https://docs.docker.com/engine/installation/)，如果是windows，会有两个选项：直接安装Docker或者Docker Toolbox（docker工具箱）。需要注意新版本Docker需要64-bit Windows 10 Pro并且支持Microsoft Hyper-V。如果你的windows版本比较老，安装Toolbox，它兼容Oracle Virtual Box。

当使用新版本docker时，你会得到提示：

![](https://rskupnik.github.io/public/images/docker_tray.png)

如果你使用Docker Toolbox，会有很多工具，其一是**Docker Quickstart Terminal**。

如果安装原生的Docker，你可以在最爱的终端中运行docker命令，如果是Toolbox，在Quickstart Terminal中输入命令。

另一件需要注意的点在于，原生安装docker时通过localhost访问容器中的webapps，如果是Toolbox，你需要注意启动Quickstart Terminal时的输出信息，它会给你容器的IP，在我的电脑上是192.168.99.100。

快速总结下，在windows上你有两个选择：
- 原生Docker，你需要64-bit Windows 10 Pro并且支持Microsoft Hyper-V，你会有docker的系统图标，能在任何终端中输入docker命令
- Docker Toolbox，你的windows版本低时的选择。需要安装Oracle Virtual Box，启动Quickstart Terminal后输入docker命令

## 使用
假设一个简单但是贴近现实的例子：学习基于JVM的语言Scala。在成为专家前，你总是需要学习很多新的知识，你要参考很多基本的入门指南，需要一个REPL执行操作，但是你不希望在自己的电脑上下载安装Scala。那么如何使用Docker解决这个问题？很简单，你只需要运行一个包含Scala和相关依赖的容器即可。

如果你是开发者，可以很容易理解镜像和容器的区别，容器相对于镜像就像对象相对于类。

镜像是从网络上下载的不可变文件，它描述了如何构建特定的容器。它们可能很大，所以要分模块构建以便在网络上传输。容器是镜像的实例，Docker启动容器后执行镜像中指定的指令。

在我们的例子中，我们需要搜索Scala镜像并启动容器。

### 搜索镜像
两个选项：直接在Google搜索`Scala docker image`或者运行命令`docker search scala`

![Does that single out-of-place ‘6’ bother you, too? Probably didn’t, until I mentioned it. You’re welcome.](https://rskupnik.github.io/public/images/docker_search.png)

我们以上图中第一个镜像为例，可以登录docker hub（https://hub.docker.com/r/hseeberger/scala-sbt）了解它的详细信息。

### 如何启动容器
启动容器的命令是：
```
docker run -it hseeberger/scala-sbt
```

运行`docker run --help`可以查看更详细的信息：
- i表示交互式，STDIN会被开启，即使我们没有附着到容器上
- t表示tty，我们会得到一个伪tty与容器交互

-it经常结合在一起使用。run命令运行结束后你会看到如下内容：
```
root@3d5b83c7ea03:~#
```

做了这么多麻烦事就得到一个命令行提示符？显然不，这里你得到的新容器的shell，时刻准备接收命令。如果执行ls，会看到一个scala-2.12.2目录，我们进入这个目录然后执行scala，然后得到scala REPL，在这里就可以实战学习scala了。`Ctrl+C`退出REPL，exit退出容器。

### 如何做得更好
我们可以使用[第二个镜像](https://hub.docker.com/r/williamyeh/scala/)，然后直接运行`docker run -it williamyeh/scala`命令就可以得到scala REPL了。为何运行两个镜像得到不同的结果？我会在下一篇文章中介绍。

总结一下，运行容器只需要简单运行命令`docker run -it your/image`，运行的容器会做什么取决于你的镜像了。

## 有用的命令
- docker ps：列出所有正在运行的容器
- docker image ls：列出所有的镜像
- docker stop <name>：暂停容器
- docker rm <name>：删除容器

## 译者说
本文是一篇学习笔记，记录了镜像和容器的基本操作，非常基础，直接去看[Docker官方文档](https://docs.docker.com/)要更好一些。https://docs.docker.com/