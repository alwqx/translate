#Dockerfile实战：构建基础的ubuntu14.04镜像
>我们可以从Docker Hub上下载官方仓库中的镜像，我自己就下载了ubuntu镜像，只有`188M`左右，很小巧了。但是看了下，里面的软件源还是官方的，而且没有安装`vim`，所以就打算自己写一个`Dockerfile`，用它来构建适合自己的ubuntu基础镜像。

```language
deb http://archive.ubuntu.com/ubuntu/ trusty main restricted
deb-src http://archive.ubuntu.com/ubuntu/ trusty main restricted
...................
```

##构建上下文
`build context`，一个自定义的文件夹，里面放置Dockerfile和一些需要的文件。比如我的：

* Dockerfile...这个是必须的
* sources.list...自己在官方社区找的[ubuntu14.04](http://wiki.ubuntu.org.cn/Template:14.04source)的源
* vimrc...安装好vim后用到的配置文件。我事先配置好的，都是些基础的配置。
```language
.
├── baseimage
│   ├── Dockerfile
│   ├── README.md
│   ├── sources.list
│   └── vimrc
```

##Dokerfile
制作image有两种方法：
* 从现有容器通过`commit`命令创建
	* dockerfile中不方便的操作可以在容器中操作然后提交
	* 没有批量启动容器的需要
	* 自己学、习练习，不需要移植
* 利用Dockerfile构建
	* 方便，灵活，可移植
	* 适合部署大量的镜像和容器

###Dockerfile基础
* '#'表示注释，一般Dockerfile第一行注释容器的基本信息和版本。
* Dockerfile以`命令：参数`为基本构建语句，命令全部大写，后面的参数视命令而定
* FROM，必须是第一个命令项，表示我的镜像是以哪个镜像为基础构建的
```language
FROM ubuntu
```

* MAINTAINER，后面接构建这的姓名和邮箱，方便联系
```language
MAINTAINER adolphlwq <kenan3015@gmail.com>
```

* LABEL，用键值对的方式来指定image的元数据
```language
LABEL Description="it is used as a basic image for DuoHuoStudio and my study.I will update and install vim." Vendor="Basic image"
```

* ADD，在构建时向Docker daemon传递文件
```language
ADD sources.list /etc/apt/
```

* RUN，接操作和命令`sudo apt-get install -y vim`等
```language
ADD sources.list /etc/apt/ 
```

* CMD，构建成功的镜像第一次启动时默认启动的命令
	* CMD只有1条，一般默认在Dockerfile的最后
	* 如果有多个CMD，只有最后一个起作用
	* CMD会被`docker run ..`后面的命令覆盖
```language
CMD ["/bin/bash"]
```

* ENV，设置环境变量
```language
ENV REFRESHED_AT 2015-05-18
```

###构建命令
```language
cd baseimage(构建上下文文件夹)
docker build -t="duohuosrudio/ubuntu:14.04_64_base_image" .
```
`docker build`中`-t`表示容器的名字
`duohuosrudio/ubuntu`中`duohuostudio`表示仓库名（不允许大写），`ubuntu`表示镜像名。
`ubuntu:14.04_64_base_image`后的`14.04_64_base_image`是标签，如果没有指定，默认的是`latest`

构建过程：

![](http://7vihfm.com1.z0.glb.clouddn.com/2015-05-19-base-image.gif)

###实践中遇到的错误
* `apt-get upgrade`和`apt-get install vim`都要加上** -y**选项，不然会报错
* ADD后面必须接两个参数，`ADD <src>... <dest>`<src>表示要添加的文件，<dest>表示文件添加到哪里。
* ADD添加的文件必须以`构建上下文`为根目录来找，不能超出构建上下文的范围。

如果除错停止构建了也不要担心，Docker会把构建过程中的文件都缓存起来，再次构建时会从缓存的地方开始，节省时间。
除错停止后`docker images`会出现一个只有`IMAGE ID`的镜像，这个就是构建失败后留下的缓存，`我们可以通过image id来运行这个镜像，然后执行除错的命令来检查为什么出错！`（下图的最后1行）
```language
adolph@geek:~/programs/DockerWorkspace/dockerfile/baseimage$ docker images
REPOSITORY               TAG                   IMAGE ID            CREATED             VIRTUAL SIZE
test/ubuntu              14.04_64_base_image   e9390454465c        14 hours ago        269.1 MB
test2/ubuntu             14.04_64_base_image   e9390454465c        14 hours ago        269.1 MB
duohuostudio/ubuntu      14.04_64_base_image   e9390454465c        14 hours ago        269.1 MB
<none>                   <none>                f6efc4dac25a        16 hours ago        269.1 MB

```
##总结
```language
docker build -t="duohuostudio/ubuntu:14.04_64_base_image" .
```
这条命令的**最后一个参数是用来指定Dockerfile的路径**，千万不要忘记。

dockerfile已经上传到github，[地址](https://github.com/adolphlwq/DockerfileHub)