# Docker基础：连接容器

## 说明
- [原文连接](https://rskupnik.github.io/docker_series_2_connecting_containers)
- [翻译：@alwqx](https://github.com/alwqx)
- [项目地址](https://github.com/alwqx/translate)
- [tt](https://github.com/alwqx/tt)：自动生成翻译模板
- 用时: 1.5h
- <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/80x15.png" /></a>

## 简介
这篇文章是*Docker基础*系列的第二篇。[上篇文章中](https://rskupnik.github.io/docker-series-1-image-and-container)，我们讨论了镜像和容器的区别以及几个简单的例子。

这次假设我们有连个容器，我们如何让它俩相互通信。我首先想到的场景是**application-database**间的关系。我将创建下面两个容器：

- Mysql RDBMS
- 一个简单的Python脚本，从Mysql容器fetch数据并且打印出来

## 一个绑定容器的网络
过去连接容器使用[--link](https://docs.docker.com/engine/userguide/networking/default_network/dockerlinks/)标记，但是它被弃用了。新的方法是使用[network特性](https://docs.docker.com/engine/userguide/networking/)。当我们运行`docker network ls`时会看到：
```
NETWORK ID      NAME    DRIVER  SCOPE
9872c9881f6e    bridge  bridge  local
6fc119c0ceda    host    host    local
c3fdf8d5c56e    none    null    local
```

它们的含义是：

- **bridge**: 默认网络，所有容器默认连接到它
- **none**: 没有网络接口
- **host**：连接到主机的网络栈，主机和容器间的网络没有隔离

如果想深入了解某个网络，使用`docker network inspect <name>`命令。

控制容器间通信的推荐做法是使用[用户定义网络](https://docs.docker.com/engine/userguide/networking/#user-defined-networks)，通过它我们可以方便地创建自己的网络。比如创建一个网络`docker network create my-network`然后查看`docker network ls`：

```
NETWORK ID      NAME        DRIVER  SCOPE
9872c9881f6e    bridge      bridge  local
6fc119c0ceda    host        host    local
c3fdf8d5c56e    none        null    local
19671b2b8b20    my-network  bridge  local
```

这样我们自己定义的网络就创建好并且可以使用了。

## Mysql容器
先运行一个`docker run -d --name mysql-server --network my-network -e MYSQL_ROOT_PASSWORD=secret mysql`不同参数的含义分别是：
- **-d**，容器与当前进程分离，后台运行
- **--name--，指定容器名
- **--network**,指定容器连接网络
- **-e**，设置环境变量

非常简单，我们的容器就运行起来了。现在我们要连接到我们的数据库容器中，创建数据库、表，然后添加些简单的数据。我将演示创建的网络把两个容器连接起来，我需要再运行一个不同的容器来连接到数据库服务器上：
```
docker run -it --rm --network my-network mysql sh -c 'exec mysql -h"mysql-server" -P"3306" -uroot -p"secret"'
```

如果一切正常，我们会看到：
```
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.18 MySQL Community Server (GPL)

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

刚才的命令意思是：
- **-it**，运行在交互模式
- **--rm**,容器退出后自动删除
- **--network**，指定需要连接的网络
- 然后`exec`mysql程序和必要的参数连接到数据库服务器。**注意，我们用mysql容器名作为host**

下面我们创建一个例子：python脚本连接数据库。首先运行下面命令创建数据库等：
```
CREATE DATABASE mydb;
USE mydb;
CREATE TABLE person (fname VARCHAR(20), lname VARCHAR(20));
INSERT INTO person(fname, lname) VALUES ('Mick', 'Jagger');
```

## 查询脚本
这部分我们会创建自己的镜像，window用户需要注意：由于docker在window有些限制，我们的目录和脚本需要放到`C:\Users\<someuser>`目录下。

首先创建目录`C:\Users\myuser\my-script`，然后目录下创建`Dockerfile`（是的，不需要扩展名）：
```
FROM python:2

WORKDIR /usr/src/app
RUN pip install MySQL-python
COPY . .

CMD [ "python", "./script.py" ]
```

简单解释下，这几行分别表示`python:2`基础镜像，设置工作目录，下载依赖，拷贝文件，指定容器执行命令。

然后创建`script.py`文件：
```
#!/usr/bin/python

import MySQLdb

db = MySQLdb.connect("mysql-server", "root", "secret", "mydb")
cursor = db.cursor()
cursor.execute("SELECT * FROM person")
data = cursor.fetchone()
fname = data[0]
lname = data[1]
print "fname=%s, lname=%s" % (fname, lname)
db.close()
```

这个脚本会连接到Mysql数据库，然后查询数据。最后我们构建镜像：
```
docker build -t my-script .
```

接着运行镜像：
```
docker run -it --rm --network my-network my-script
```

会看到输出：
```
fname=Mick, lname=Jagger
```

证明我们的脚本正确运行。

## 总结
- **--link**参数会被弃用
- 使用新的**--network**参数
- 会有默认的network，但是推荐使用自定义网络
- **--name**很重要，它指定主机在另一个容器里的可见地址
- 常用命令：
    - docker network ls
    - docker network create <name>
    - docker network inspect <name>