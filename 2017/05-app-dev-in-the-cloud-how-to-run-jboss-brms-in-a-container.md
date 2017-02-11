# 云端App开发：如何在容器中运行JBoss BRMS
>本文中，我们将介绍如何在云端容器中运行JBoss BRMS，云环境可以是私有云或者其它云解决方案。

我有一系列文章来跟你解释[为什么应用开发者不能再忽视技术栈](http://www.schabell.org/2016/02/appdev-cloud-stack-cant-ignore-stack-anymore.html)，这里的技术栈指的是
开发者日常工作中用到的基于云计算的基础设施。于是我探索了在本地搭建基于云计算基础设施的可能性，来代替已经成熟的[红帽云套件](https://www.redhat.com/en/technologies/cloud-computing/cloud-suite)

这么做有利的地方在于，本地私有云的方式和你在工作中使用数据中心来管理组织的方式，在某些方面具有类似的开发体验。

首先我举个[例子](https://github.com/redhatdemocentral/cdk-install-demo)，它通过[Red Hat Container Development Kit (CDK)](http://www.schabell.org/2016/09/installing-redhat-cdk-2-2-release.html)安装Red Hat OpenShift Enterprise (OSE)镜像。

然后我以在[OSE上安装JBoss BRMS](http://www.schabell.org/2016/03/real-appdev-in-cloud-jboss-brms-install-demo.html)为例。JBoss BRMS提供了完美的工作方式。
但是最终目的是使用Red Hat提供的最新产品。基于这一点，我找到一种像Demo一样简单的方式为您提供Red Hat OpenShift容器平台(OCP)，[结果发布在之前这篇文章中](http://www.schabell.org/2016/11/3-steps-to-cloud-happiness-with-ocp.html)，当然，到这里还没结束。

## 容器化JBoss Business Rules Management System (BRMS)
当你在机器上安装好OpenShift后，不管安装的是Red Hat CDK还是OCP，下一步要做的是去探索Red Hat JBoss中间件产品提供的应用开发选项。

本节将介绍另一个简单的安装示例项目，它向您展示完全可操作的、开箱即用的JBoss BRMS安装方式。不仅如此，它是安装在你的OpenShift上的容器化应用。
1. 首先，你可以安装容器化的OpenShift，可参考下面两个链接：
    - [OCP Install Demo](https://github.com/redhatdemocentral/ocp-install-demo)
    - [CDK Install Demo](https://github.com/redhatdemocentral/cdk-install-demo)
2. 如果你已经安装了OpenShift就不必多此一举。
![](https://3.bp.blogspot.com/-k1xy-YD4PuU/WDakkKC8FpI/AAAAAAAAoXg/XD3MpadKUgInnBZE8q91A6ba3S1gObCFQCEw/s320/rhcs-brms-build-ocp.png)
3. 将产品添加到安装目录
4. 运行`init.sh`或者`init.bat`。`init.bat`必须运行在管理员权限下。
```
# The installation needs to be pointed to a running version
# of OpenShift, so pass an IP address such as:
#
$ ./init.sh 192.168.99.100  # example for OCP.
$ ./init.sh 10.1.2.2        # example for CDK.
```

现在登录到JBoss BRMS，开始设置容器化项目的开发规则（登录地址由init.sh脚本自动生成）。
- OCP的样例地址：http://rhcs-brms-install-demo.192.168.99.100.xip.io/business-central ( u:erics / p:jbossbrms1! )
- CDK的样例地址：http://rhcs-brms-install-demo.10.1.2.2.xip.io/business-central ( u:erics / p:jbossbrms1! )

![](https://3.bp.blogspot.com/-irHjE1DBn0I/WDakkgLb7_I/AAAAAAAAoXo/9TSTnfS82KkDJufbDZQPZBjjqW4ixtkAACEw/s320/rhcs-brms-pod-ocp.png)

上图中的pod就是OpenShift容器平台上你刚创建的JBoss BRMS。

容器启动需要一定的时间，你要给它足够的时间来启动JBoss EAP个BRMS。你可以在OpenShift控制台检查部署好的pod，你还可以在log tab下面查看log。

以上就完成了，现在你可以在闲暇时开发企业逻辑或任务了。

你也可以继续微调，或者参考[Red Hat Demo Central](https://github.com/redhatdemocentral)。
