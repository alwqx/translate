#Docker化你的应用
使用`docker run`命令在容器中运行应用

>如果你使用的是远程Docker 进程（daemon），使用`sudo docker run`

##Hello world
```shell
sudo docker run ubuntu:14.04 /bin/echo 'Hello world'
```
执行这条命令Docker首先会在本地的Docker主机上找image：ubuntu：14.04，如果没找到，Docker会到Docker Hub上下载这个镜像
Docker容器仅仅在你指定的命令激活时才运行，在上面的命令中，当输出`hello world`后，容器就停止。

##An Interactive Container（交互式容器）
```shell
$ sudo docker run -t -i ubuntu:14.04 /bin/bash
root@af8bae53bdd3:/#
```
`docker run`启动镜像ubuntu14.04，`-t`在启动的容器中使用`终端`，`-i`表示允许我们建立交互式的连接，通过获取容器的标准输入[stdin]
`exit`或者ctrl+D退出终端

##A Daemonized Hello world（将命令守护进程化）
```language
$ sudo docker run -d ubuntu:14.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
eb643329659cb6b6830b70b87ef9576e0da1913682d4972d8ab904fb709072b7
```
`-d`表示后台运行容器
`/bin/sh -c "while true; do echo hello world; sleep 1; done"`无限输出`hello world`
返回一个`a bit long`表示`container ID`

>Note: The container ID is a bit long and unwieldy and a bit later on we'll see a shorter ID and some ways to name our containers to make working with them easier.

```language
adolph@geek:~$ docker ps
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS               NAMES
eb643329659c        ubuntu:14.04        "/bin/sh -c 'while t   2 minutes ago       Up 2 minutes                            clever_hypatia      
```
>docker会自动命名我们启动的container，当然你也可以自己重新命名

查看容器日志并且返回它的输出
```language
docker logs container_name[clever_hypatia]
```

停止容器
```language
adolph@geek:~$ sudo docker stop clever_hypatia 
clever_hypatia
adolph@geek:~$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

##总结
* docker ps
* sudo docker run [image name] [command]
* sudo docker run ubuntu:14.04 /bin/echo 'Hello world'
* sudo docker -t -i run [image name] [command]...交互式操作（有自己的命令行）
* sudo docker run -t -i ubuntu:14.04 /bin/bash
* sudo docker run -d [iamge name] [command]
* sudo docker run -d ubuntu:14.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
* sudo docker logs [container name]...查看容器日志和输出信息
* docker logs clever_hypatia 
* docker stop [container name] ...停止容器

[原文链接](https://docs.docker.com/userguide/dockerizing/)