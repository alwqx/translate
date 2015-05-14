#6-Linking Containers Together
>Docker container中services/applications与主机或者其它containers之间通信的两种方式

* port mapping(端口映射)
* container linking(容器连接)

##Connect using Network port mapping
>使用端口映射来连接

常用的命令(以`training/webapp`为例)：
```language
docker run -d -P traning/webapp ...
docker run -d -p traning/webapp ...
```
###`-P`flag
* 当container被创建并且运行时，-P标记立刻生效
* 它container内部的任意端口映射到docker host的port
	* Docker host的port是随机的
	* 它的生命力是短暂的（容器停止后端口映射就会失效）

e.g.
```language
adolph@geek:~$ docker run -d -P training/webapp python app.py
3f30e81a01cdf9895a70828beebea32910f848ac00f92303e6af77faeee1db0a
adolph@geek:~$ docker ps
CONTAINER ID        IMAGE                    COMMAND             CREATED             STATUS              PORTS                     NAMES
3f30e81a01cd        training/webapp:latest   "python app.py"     8 seconds ago       Up 8 seconds        0.0.0.0:32769->5000/tcp   agitated_hawking
``` 
内部的5000端口映射到外部的主机的`32769`端口

###`-p`flag
可以指定container内外的端口
内外的5000端口映射
```language
docker run -d -p 5000:5000 training/webapp python app.py
adolph@geek:~$ docker ps
CONTAINER ID        IMAGE                    COMMAND             CREATED             STATUS              PORTS                    NAMES
cf6bd021034d        training/webapp:latest   "python app.py"     4 seconds ago       Up 2 seconds        0.0.0.0:5000->5000/tcp   compassionate_lalande   
```
>这样做的不好的地方在于你只把congtainer内外的5000端口映射在一起，container内的其它端口被抛弃了

映射到主机的××端口
```language
docker run -d -p 127.0.0.1:5000:5000 training/webapp python app.py
```

映射到主机的某个随机端口
```language
docker run -d -p 127.0.0.1::5000 training/webapp python app.py
```
注意`127.0.0.1:`有个冒号
 
-p参数使用的次数很多，主要用来配置多个端口

##Connect with the linking system
>Docker有自带的`linking system`用来连接多个container，并且允许从一个container发送信息到另一个。发送信息的称为`source container`，接收信息的称为`recipient container`，recipient container智能看到一些经过筛选的关于source container的某些信息

###The importance of naming
>Docker依赖于容器的名字来建立连接，Docker启动容器时会自动给它起个名字，当然你也可以自己命名

命名有两个非常棒的好处：
* 告知container的作用或者属于哪种类型，如`traning/webapp`可以看出是webapp的container
* 方便Docker通过name指定container

在运行container时通过`--name`标记来命名新的container
```language
adolph@geek:~$ docker run -d -P --name web training/webapp python app.py
1be7fc1ca8f9b683a8c309a1f6315c65819db15e8105ddd1b198e50c4082842f
adolph@geek:~$ docker ps
CONTAINER ID        IMAGE                    COMMAND             CREATED             STATUS              PORTS                     NAMES
1be7fc1ca8f9        training/webapp:latest   "python app.py"     4 seconds ago       Up 3 seconds        0.0.0.0:32770->5000/tcp   web
adolph@geek:~$ docker inspect -f "{{.Name}}" 1be7f
/web
```

<span style="color:red;">**注意：**</span>container的name必须是唯一的，比如说刚才的`web`，如果你想给另一个容器起名为`web`，那只能把原来的`web`container删除（docker rm [-f]）。另外，`docker run --rm --name..`会在容器停止运行后立即删除

###Communication across links
>通过连接通信

links允许容器发现对方并且建立安全的信息传输通道，link创建好后容器间通信的通道就建立好了。

注意事项:
1. 需要运行的两个容器不能重名
2. 不能和其它已经存在的（不管有没有运行）容器重名
3. `docker ps -a`查看所有容器的信息
4. `docker rm [container name]`删除容器

link代码
```language
--link <name or id>:alias
```
`name or id`指我们要连接的container的名字

shell1的代码（我在一个shell中运行这些命令总会有1个容器在启动后就`EXIT（0）`）
```language
adolph@geek:~$ docker run -i -t --name db adolph/ubuntu:14.04 
root@a1bb409128b9:/#
```

shell2
```language
adolph@geek:~$ docker run -d -P --name web --link db:db training/webapp python app.py
09fed08b63e709e61d17f698ccec55a6d04ddb6e33c1aea3879f78d1970451ce
adolph@geek:~$ docker inspect -f "{{.HostConfig.Links}}" web
[/db:/web/db]
```
>we can see that the `web` container is now linked to the `db` container `web/db`. Which allows it to access information about the db container.

| recipient container| source container|
|--------|--------|
|      web  |     db   |

为了做到容器间的通信，Docker没有使用端口，而是自己建立了`tunnel`(隧道),使用`link`连接容器的好处是我们不需要将`source container`的端口暴露给网络，Docker的tunnel使用两种方式实现连接：
* Environment variables(环境变量)
* Updating the /etc/hosts file（更新`/etc/hosts`文件）

####环境变量
>当我们连接容器是Docker会创建很多环境变量，并且它会在目标容器自动基于`--link`后面的参数创建环境变量。Docker会公开来自`source container`的所有环境变量，这些变量包括：

* the ENV commands in the source container's Dockerfile(源容器Dockerfile中的`ENV`命令)
* the `-e`, `--env` and `--env-file` options on the docker run command when the source container is started（容器运行时`run`后面的`-e`, `--env` 和 `--env-file`参数）

这些环境变量允许我们通过编程从目标容器发现来自源容器的信息

>警告：理解docker link连接容器的机制很重要，link允许所有目标容器获得源容器的指定数据和信息，所以从安全性的角度，不建议在源容器中存储敏感的数据

#####一些环境变量
`<alias>_NAME`
```language
docker run -d -P --name web --link db:db training/webapp python app.py
```
这个变量是为目标容器建立的，如上，`--link db:db`后`web`容器被链接到`db`容器，这是Docker会在`web`容器中创建DB_NAME=/web/db

`<name>_PORT_<port>_<protocol>`
Docker为每一个源容器暴露的每一个端口。
* <name>指--link中指定的目标容器的别名
* <port>源容器暴露的端口号
* <protocol> TCP或UDP

Docker使用不同的前缀格式来规定3种不同的环境变量：
* `prefix_ADDR`来自URL的ip地址。例如：WEBDB_PORT_8080_TCP_ADDR=172.17.0.82.
* `prefix_PORT`来自URL的端口。例如：WEBDB_PORT_8080_TCP_PORT=8080.
* `prefix_PROTO`来自URL的协议。例如：WEBDB_PORT_8080_TCP_PROTO=tcp.

每一组环境变量对应一个端口，如果容器公开多个端口（比如3个），Docker就会创建9个环境变量，每个端口3个。
此外，Docker还会为`源容器`第一个公开的端口创建`<alias>_PORT`，这里的`第一个`指的是具有`lowest port`的端口。例如：`WEBDB_PORT=tcp://172.17.0.82:8080`，如果它吗满足tcp和udp，那指的是tcp

最后，Docker还会创建这样一个变量：`<alias>_ENV_<name>`，用来连接源容器和目标容器的桥梁。Docker使用这个值来启动源容器。

e.g:
```shell
adolph@geek:~$ sudo docker run --rm --name web2 --link db:db training/webapp env
[sudo] password for adolph: 
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=20fb0166d9c6
DB_NAME=/web2/db
HOME=/
```
我的电脑上并没有看到环境变量！可能因为我的db容器和官网例子上的容器不一直吧，而且我的db容器中并没有安装数据库。
Docker创建的这些变量有助于我们用来连接和配置源容器里的工具，比如连接数据库...

#####Updating the /etc/hosts file
>更新`/etc/hosts`文件

这部分自己有不懂的地方，以后慢慢看吧。

##总结
###连接容器的两种方法
####`-p`，`-P`标记
####link system
* 容器的命名是唯一的，不能重复。可以更改容器的名字
	* `docker rename oldname newname`
* --link <name or id>:alias,name指要连接的容器，alais指目标容器/简称
* 能够清除代码中那个是目标容器，哪个是源容器，资源开放共享的方向

```language
docker run -d -P --name web --link db:webdb training/webapp python app.py
```
这条命令中`db`是要连接的容器，是源容器，web是目标容器，webdb是web的别称
```language
adolph@geek:~$ docker inspect -f "{{.HostConfig.Links}}" web
[/db:/web/webdb]
```

[原文链接](https://docs.docker.com/userguide/dockerlinks/#important-notes-on-docker-environment-variables)

都是个人理解，可能有错误，还请指正。
