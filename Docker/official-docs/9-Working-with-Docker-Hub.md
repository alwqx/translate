#Working with Docker Hub
到目前为止我们已经学习了如何使用命令行在主机上运行Docker。你已经学习了如何下载镜像，如何从已经存在的镜像运行容器，以及如何创建你自己的镜像。

下一步，你将学习如何使用Docker Hub来简化和加强你的Docker工作流。

Docker Hub是由Docker公司维护的公共注册仓库。你可以利用它：
* 下载超过15000的镜像来构建容器
* 身份验证、工作组织结构以及像`webhooks`和`trigger`这样的工作流工具
* 一些私人工具，比如私人仓库用来存放你不想和他人分享的镜像

###Docker commands and Docker Hub
Docker本身提供了一些命令用于获取Docker Hub服务：
* docker login
* docker search
* docker pull
* docker push

###Account creation and login
要想使用Docker Hub的服务，首先要有Docker Hub的账号并且登录。你可以在Docker Hub上注册或者通过命令：
```language
docker login
```
这条命令后会提示输入用户名，**会成为你公共仓库的共有命名空间**，如果已经有了用户名，Docker会提示你输入密码和邮箱，然后自动登录。登录成功后你就可以向Docker Hub上自己的仓库中推送自己的镜像了。

>注意：你的身份验证信息会被存在用户目录的`.dockercfg`认证文件中

###Searching for images
我们可以通过Docker自己的search接口或者是命令行中的接口来查找Docker Hub中的镜像。关键字可以是镜像名，用户名甚至是镜像的描述信息。
```language
$ sudo docker search centos
NAME           DESCRIPTION                                     STARS     OFFICIAL   TRUSTED
centos         Official CentOS 6 Image as of 12 April 2014     88
tianon/centos  CentOS 5 and 6, created using rinse instea...   21
...
```
其中有两个结果：`centos`，`tianon/centos`。第二个`tianon/centos`表示它来自于一位叫`tianon`的用户的仓库。第一个结果没有显示列出仓库则意味着它是受信任的官方顶级名称空间存储库。`/`将仓库名和镜像名分割开。

找到镜像后`pull`下载镜像
```language
docker pull [imagename]
```

###Contributing to Docker Hub
任何人都可以从Docker Hub下载镜像，但是如果你想向Docker Hub推送镜像，首先要注册

###Pushing a repository to Docker Hub
为了将仓库推送到`register`中，你需要已经命名的镜像或者把你的容器保存为命名的镜像，详情见[这里](http://docs.docker.com/userguide/dockerimages/)
```language
 docker push yourname/newimage
```

###Features of Docker Hub
现在我们就来看看Docker Hub有哪些特性，更多信息见[这里](http://docs.docker.com/docker-hub/)
* 私人仓库
* 组织和团队
* 自动构建
* webhooks

#####Private Repositories
如果你有镜像不想公开或和他人分享，Docker允许你拥有自己的私人仓库

#####Organizations and teams
私人仓库的一个好处是你可以把里里面的镜像分享给组织或团队里的人。Docker Hub允许你创建自己的组织，在组织里你可以和同伴一起工作，以及管理自己的仓库。详情见[这里](https://registry.hub.docker.com/account/organizations/)

#####Automated Builds
自动构建和更新github和bitbuckets中的镜像，这些工作直接在Docker Hub中进行（不是本地）。它的工作方式是这样的：在你选中的github或bitbucket中添加`hook`，当你更新仓库时会触发构建和更新操作。

自动构建的步骤：
* 创建账号并登录
* 连接github或bitbucket的账号
* 配置自动构建的选项
* 选中github或bitbucket中带有`Dockerfile`的项目
* 选择分支
* 命名
* 分配可选的Docker标签
* 指定Dockerfile文件的位置，默认是`/`目录

在[ Automated Builds page](https://registry.hub.docker.com/builds/)查看自己自动构建的项目	

不能对自动构建的仓库使用`docker push`命令。只能通过向github或bitbucket更新代码来管理自己的镜像。

你也可用为同一个项目的不同分支创建多个自动构建的项目。

#####Webhooks
webhooks附着到你的仓库并且在你更新镜像或者push操作时触发事件。通过webhook，你可以指定一个目标URL或者JSON负载均衡当push镜像时。