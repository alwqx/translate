#4-使用Docker镜像和仓库
##image
* 镜像由文件系统叠加而成（不同的层layer叠加）
* 最底端是引导文件系统`bootfs`
* Docker用户几乎不会有和bootfs交互的机会
* ubuntu中，每个镜像存在于`/var/lib/docker`
###镜像加载
容器启动后会被移到内存，此时引导文件系统会被卸载`unmount`，释放更多的内存。
Docker利用联合加载技术（union mount，一次同时加载多个文件系统，对外只能看放到一个文件系统）加载文件系统,将各层文件系统叠加到一起，这样最终的文件系统会`所有底层的文件和目录`，docker将这样的文件系统称为`镜像`
![](http://7vihfm.com1.z0.glb.clouddn.com/2015-05-16-image-layer.png)

镜像之间存在父子关系，这样会形成一个`镜像栈`，镜像栈最底部称为`基础镜像`

![](http://7vihfm.com1.z0.glb.clouddn.com/2015-05-16-parent-image.png)

** 写时复制**（copy on wirte）
docker第一次启动容器时，`初始的读写层`是空的.当文件系统发生变化时，比如修改了一个文件，这个文件首先会从该读写层下面的只读层复制到这个读写层，然后修改。该文件的只读版本依然存在，只是读写层的副本隐藏了它。

标签用来区分不同的镜像，镜像名要简介，可以把镜像的相关信息用标签来表示。

##创建docker image
>一般说的‘创建’是指基于已有的镜像创建

###注册Docker Hub
命令行登录（ubuntu，已注册）
```shell
docker login
username:your username
password:
email:
login succeeded
````
这时Docker已经登录到Docker Hub并认证信息保存在`～/.dockercfg`供以后使用

###docker commit创建image
```language
docker commit -m="" -a="" [base image id] [new image name]:[tag]
一些标记可以通过docker commit --help查看
```

**<span style="color:red;">读者注：</span>**
我在实践时发现：** --auther=""会报错，直接写-a=“”就没问题**

###用Dockerfile构建镜像
>Dockerfile构建镜像更加灵活，Docker官方推荐使用`Dockerfile`而不是`docker commit`

`构建环境（build environment）`：用来存放Dockerfile和文件的文件夹。Docker称此环境`context（上下文）`或者`build context`。<span style="color:red;">Docker会在构建镜像时将构建上下文中的文件和目录以及Docker的构建上下文一起上传到Docker的守护进程中。这样的好处是Docker守护进程就可以访问你在构建上下文中存储的代码，文件或者数据了。</span>

* Dockerfile中的文件由`指令` `参数`构成，指令必须大写，后面必须有参数。
* DOckerfile中的指令按顺序由上到下执行，所以要合理安排指令的顺序
* 每条指令都会创建新的层并对镜像进行提交

Docker执行Dockerfile的流程
* Docker从基础镜像运行一个容器
* 执行1条指令，对容器做修改
* 执行类似`docker commit`的操作，提交一个新的镜像层
* Docker基于上步新提交的镜像运行新的容器
* 执行下一条指令，知道指令执行完毕

这样做的好处：
* 指令中途失败，依然会得到指令失败前的构建的镜像，有助于调试
* 基于中途的镜像运行容器，执行失败的Dockerfile命令进行调试

**注意：**RUN指令后面有两种执行命令的方式：
* `/bin/sh -c`命令包装器如
```language
RUN apt-get install -y nginx
```

* `exec`格式的命令,适用于不支持shell或不希望在shell中运行
```language
RUN ["apt-get", "install", "-y", "nginx"]
```

P68