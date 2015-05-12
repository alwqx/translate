#Working with Containers(和容器一起工作)
##回顾之前讲过的一些命令
```shell
docker run -i...交互式运行
docker run -d...background运行`daemon`守护进程
docker ps...Lists containers.(容器列表)
docker logs...Shows us the standard output of a container.（显示容器的标准输出）
docker stop...Stops running containers.
```

docker命令格式：
```shell
[sudo] docker [command] [flags] [arguments]...
```
##Seeing what the Docker client can do
>docker client能干什么

`[sudo] docker`...显示docker后能执行的命令

##Seeing Docker command usage
>docker 命令使用

```language
docker command --help...查看特定命令的使用方式
```

##Running a Web Application in Docker
>docker中运行一个web应用

```language
 sudo docker run -d -P training/webapp python app.py
```

`-P`参数表示将容器内部要用到的网络端口映射到主机

```language
docker ps -l -a
```
`-l`显示容器的详细信息，`-a`表示显示所有的容器信息（包含以前运行的）

```language
docker run -d -P training/webapp python app.py
docker run -d -p 5000:5000 training/webapp python app.py
```
`-P`表示将`image`镜像的任何端口映射到我们自己的主机
`-p`自己指定`image`的网络端口和主机的端口

**实战：**
命令行输入
```language
docker run -d -P training/webapp python app.py
docker ps -l -a
out:
adolph@geek:~$ docker ps -a -l
CONTAINER ID        IMAGE                    COMMAND             CREATED             STATUS              PORTS                     NAMES
1179c34ac0e0        training/webapp:latest   "python app.py"     2 minutes ago       Up 2 minutes        0.0.0.0:32769->5000/tcp   elegant_curie
```
<span style="color:red;">这里重要的是port下面的值：0.0.0.0:32769->5000/tcp</span>，我的理解是：这条命令把`0.0.0.0:32769`这个自己主机的端口映射到容器里的`5000`端口，所以当你在自己的浏览器输入`0.0.0.0:32769`是它会映射到images的`5000`端口从而访问app.py的网页

##A Network Port Shortcut
>网站端口`Shortcut`

```language
docker port [container id|container name]
```
这条命令表示输出容器的端口和映射端口，
```language
5000/tcp -> 0.0.0.0:32769
adolph@geek:~$ docker port 1179c 5000
0.0.0.0:32769
```

##Viewing the Web Application's Logs
>查看web应用的logs

```language
adolph@geek:~$ docker logs -f elegant_curie 
 * Running on http://0.0.0.0:5000/
172.17.42.1 - - [12/May/2015 17:49:25] "GET / HTTP/1.1" 200 -
172.17.42.1 - - [12/May/2015 17:49:26] "GET /favicon.ico HTTP/1.1" 404 -
```
`-f`功能类似于`tail -f`而且我们可以看到标准输出的信息

##Looking at our Web Application Container's processes
>查看web应用容器的进程

使用`docker top`命令

```language
adolph@geek:~$ docker top elegant_curie 
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                32280               2150                0                   01:48               ?                   00:00:00            python app.py

```

##Inspecting our Web Application Container
>检查web应用容器

```language
docker inspect comtainer name
```
以`Json`格式输出信息

##Stopping&Start&Remove our Web Application Container
>停止和启动我们的web容器

```language
adolph@geek:~$ docker ps -l
CONTAINER ID        IMAGE                    COMMAND             CREATED             STATUS              PORTS                     NAMES
1179c34ac0e0        training/webapp:latest   "python app.py"     25 minutes ago      Up 25 minutes       0.0.0.0:32769->5000/tcp   elegant_curie       
adolph@geek:~$ docker stop elegant_curie 
elegant_curie
adolph@geek:~$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES                                                 
adolph@geek:~$ docker start elegant_curie 
elegant_curie
adolph@geek:~$ docker ps -l
CONTAINER ID        IMAGE                    COMMAND             CREATED             STATUS              PORTS                     NAMES
1179c34ac0e0        training/webapp:latest   "python app.py"     26 minutes ago      Up 7 seconds        0.0.0.0:32770->5000/tcp   elegant_curie       

adolph@geek:~$ docker stop elegant_curie 
elegant_curie
adolph@geek:~$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
adolph@geek:~$ docker rm elegant_curie 
elegant_curie
```
##命令总结
```shell
docker...显示docker的命令
docker command --help...显示某个命令的帮助
docker ps -a -l...显示所有的容器信息
docker run -d -P ...-P映射容器的5000端口到主机的任意端口[32768-61000]
docker run -d -p ...-p自己指定映射端口
docker port [comtainer id|container name]...查看容器的端口情况
docker logs -f [container name|id] ...输出容器标准输出
docker inspect [container name]...输出json格式的容器的详细信息
docker top [container name...查看容器的进程
docker stop|start|rm [container name]...停止|开始|删除容器

```

[原文链接](https://docs.docker.com/userguide/usingdocker/)





