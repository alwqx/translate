# Kubernetes持续部署Docker Apps

<!-- TOC -->

- [Kubernetes持续部署Docker Apps](#kubernetes持续部署docker-apps)
    - [说明](#说明)
    - [集成Codeship到Kubernetes](#集成codeship到kubernetes)
    - [Push到Google Container Registry](#push到google-container-registry)

<!-- /TOC -->

## 说明
- [原文链接](https://blog.codeship.com/continuous-deployment-of-docker-apps-to-kubernetes/)
- [翻译：@adolphlwq](https://github.com/adolphlwq)
- [项目地址](https://github.com/adolphlwq/translate)
- <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/80x15.png" />

>这一系列文章中，我们介绍过了[using Kubernetes for deployments](https://blog.codeship.com/using-kubernetes-for-deployments/)。本篇我们开始集成Codeship到工作流中。

假设已经有了Kubernetes的Deployment（注意我们上篇文章中已经讨论了“Deployment”和Kubernetes中“Deployment”这两个概念的区别），现在我们该如何集成到我们自己的Codeship工作流中呢？最终答案取决于Kubernetes的部署模式，因为Kubernetes官方文档使用Google Cloud作为案例，我也会使用这个部署模式。

## 集成Codeship到Kubernetes
Codeship开发了相关[Google Cloud](https://documentation.codeship.com/docker/continuous-deployment/google-cloud/)功能集成到他们的CI平台，方便用户认证与部署新的image到Google Cloud。

但是，在开始前，我们要使用Codeship CLI工具创建加密的环境文件，帮助我们更方便地认证到Google Cloud。[Codeship已经有一篇教程](https://documentation.codeship.com/docker/getting-started/encryption/)关于怎么创建环境文件，所以这里不会详细介绍。但是牢记要设置以下环境变量：
- a Google Cloud Key - `GOOGLE_AUTH_JSON`
- a Google Authentication Email – `GOOGLE_AUTH_EMAIL`
- a Google Project ID – `GOOGLE_PROJECT_ID`

一旦加密环境文件准备好（并且保存环境变量到`gc.env.encrypted`），下一步我们要在`codeship-services.yml`中定义Google Cloud服务。

![](https://1npo9l3lml0zvr6w62acc3t1-wpengine.netdna-ssl.com/wp-content/uploads/2016/12/Screen-Shot-2016-12-21-at-7.05.03-PM-768x272.png)

注意需要定义两个服务，而不是一个。这是因为一个服务负责与Google Cloud交互（`google_cloud_deployment`），另一个服务负责
push Docker image到Google Cloud Registry（`gcr_dockercfg`）。我们已经[为你提供了一个模板](https://github.com/codeship-library/gcr-dockercfg-generator)。

到这里我们已经解决了一半的难题。虽然它创建了必要的服务与Google Cloud交互，它却不能自动地部署新构建的image或者更新一个Kubernetes Deployment。

## Push到Google Container Registry
感谢Codeship[内建的push步骤](https://documentation.codeship.com/docker/getting-started/steps/#push-steps)，使得部署Docker image到远程的registry非常的平滑。使用`gcr_dockercfg`服务实现上面的功能，我们需要做的就是在`codeshipsteps.yml`添加Google Container Registry的URL作为目的地址。

下面非常重要，因为我们要部署应用镜像了，注意把下面的服务名替换成你自己的。

![](https://1npo9l3lml0zvr6w62acc3t1-wpengine.netdna-ssl.com/wp-content/uploads/2016/12/Screen-Shot-2016-12-21-at-7.09.56-PM.png)

上面的参数名很好地说明它们的含义，它的基本思想是app image会被push到Google Container Registry，使用之前定义好的`gcr_dockercfg`服务实现认证。

虽然这里更新的image被push到registry，但是会有一个问题。我们没有定义image的tag，Codeship默认tag为`latest`。本身而言这还不是件糟糕的事，但是为了触发Kubernetes Deployment自动更新，我们要为每次push的image设置不同的tag。

Codeship提供了`image_tag`参数声明image的tag，[Codeship有一个变量列表](https://documentation.codeship.com/docker/getting-started/docker-push/#pushing-to-tags)来帮助我们声明image的参数；但是，为了简化，我们使用当下的Unix时间戳，因为它是独一无二的，无法替代。

有了`image_tag`，之前的配置就变成下面这样了：

![](https://1npo9l3lml0zvr6w62acc3t1-wpengine.netdna-ssl.com/wp-content/uploads/2016/12/Screen-Shot-2016-12-21-at-7.13.14-PM.png)

现在，当我们push app image到Google Container Registry时，image的tag会被标记为当下的Unix时间戳。

本篇文章到这里告一段落，记得检查新的教程。

>这是关于Kubernetes、Docker和Codeship系列文章的第二篇。如果你已经等不及了，请下载免费电子书：[Continuous Deployment for Docker Apps to Kubernetes](https://resources.codeship.com/ebooks/deploy-docker-kubernetes-codeship?utm_source=CodeshipBlog&utm_campaign=cd-docker-kubernetes)。
