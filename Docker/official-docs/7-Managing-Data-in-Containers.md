#Managing Data in Containers
>前面已经介绍了许多基础的知识，现在我们来一起学习如何管理Docker容器里面以及容器之间的数据

先介绍两种原始的方法来管理Docker中的应用

* ata volumes
* Data volume container

##Data volumes
>`data volumns`是专门设计的工具，它绕过了UFS直接工作与一个或多个容器。它为数据持久和分享提供了许多功能：

* 容器被创建时，卷（volumn）被初始化。如果基础镜像在指定的挂载点包含数据，这些数据也会被复制到新容器的卷中。
* 数据卷可以在多个容器间分享和复用
* 可以直接更改卷里面的数据
* 更新镜像时对容器数据卷的更改将不会被包含到新的镜像中
* 即使容器被删除，数据卷依然存在

数据卷的设计被用来持久化数据，让数据能够独立于容器的生命周期。因此当删除容器时Docker也不会自动删除数据卷。
###Adding a data volume
>增加数据卷

`docker create -v`和`docker run -v`中的**-v**标记来给容器添加数据卷，我们可以在一条命令中多次使用`-v`标记来添加多个数据卷，下面的例子挂载了一个数据卷在我们的web应用容器中。
```language
docker run -d -P --name web -v /webapp training/webapp python app.py
```
这条命令执行后会在容器中创建一个新的卷`webapp`

###Mount a Host Directory as a Data Volume
>为数据卷挂在主机目录

除了使用-v标记来创建卷之外，你还可以挂载Docker守护进程主机的目录到容器中。

><span style="color:red;">注意：</span>如果你使用Boot2Docker，那么你的Docker守护进程只能被限制访问OSX/windows特定的文件目录。Boot2Docker会努力自动分享OSX中的`/users`目录和windows中的`C:users`目录。因此你可以通过`docker run -v /Users/<path>:/<container path>` ... (OSX)或者`docker run -v /c/Users/<path>:/<container path ... `(Windows).来挂在文件或目录。所有的其它路径（不是/users和C：users）都来自Boot2Docker虚拟机中的文件系统。

```language
docker run -d -P --name web -v /src/webapp:/opt/webapp training/webapp python app.py
```
上述命令会把主机的`/src/webapp`目录挂在到容器中的`/opt/webapp`下
><span style="color:red;">注意：</span>如果`/opt/webapp`目录已经存在与容器的镜像中，那么`/opt/webapp`中的内容会被主机上的`/src/webapp`中的数据替换，这个和mount命令是一致的。

数据卷挂在数据对测试非常有用，比如我们可以把源代码挂在到容器中，然后修改代码看看应用会发生什么。主机上的目录必须是绝对路径，如果这个目录不存在Docker会自动去创建1个。
><span style="color:red;">注意：</span>不能在`Dockerfile`中来配置挂载目录，因为`Dockerfile`的目的是更方便的来一直和分享镜像，而主机目录依赖于主机，（对于一个目录，在不同的主机上可能绝对路径不一致）所以Dockerfile中目录挂载不会适用于所有的主机

挂载的数据卷默认是可读写的，当然我们可以通过命令标记来让它`只读`
```language
docker run -d -P --name web -v /src/webapp:/opt/webapp:ro training/webapp python app.py
```
上述命令中我们通过`ro`选项来让数据卷只读
###Mount a Host File as a Data Volume
>挂载主机文件作为数据卷

`-v`标记还可以用来挂在来自主机的文件，而不仅仅是目录
```language
docker run --rm -it -v ~/.bash_history:/.bash_history ubuntu /bin/bash
```
上述命令会带你到一个新容器的shell界面，你会有来自主机的bash历史。因为容器和主机共享了一个`.bash_history`文件，所以你在容器中的命令历史和主机中的历史都会记录到`.bash_history`中，这样当你退出容器中时，你在容器中的命令历史被保存下来了，在主机的shell历史记录中仍然能够看到容器中的历史。

![](http://7vihfm.com1.z0.glb.clouddn.com/2015-05-18-volumns.png)

><span style="color:red;">注意：</span>人们会使用很多工具来编辑文件，`vi`，`sed --in-place`，这些都会导致文件的索引节点改变。Docker 1.1.0之前，文件修改会报如`sed: cannot rename ./sedKdJ9Dy: Device or resource busy`这样的错误。但是在Docker 1.1.0之后，挂载文件让文件修改变得非常简单而不需要再去挂在包含这个文件的父目录了。

##Creating and mounting a Data Volume Container
>创建一个专门防数据的数据卷容器

如果你有一些持久化的数据需要在容器之间共享，或者想从非持久化容器使用持久化数据。最好的办法是创建名为`Data`的卷容器，把数据都挂在到Data容器里
我们创建一个能分享数据的命名容器，他不运行任何应用，它重复使用`training/postgres`镜像以便所有的容器使用同一个层，这样可以节省磁盘空间。

```language
docker create -v /dbdata --name dbdata training/postgres /bin/true
```
我们使用`--volumes-from`标记来绑定`/dbdata`卷到另一个容器
```language
docker run -d --volumes-from dbdata --name db1 training/postgres
```
或者
```language
docker run -d --volumes-from dbdata --name db2 training/postgres
```
在是上述的例子中，我们在容器中挂在了`/dbdata`卷，如果恰巧镜像`training/postgres`中也有`/dbdata`这个目录，那么容器会隐藏**镜像的目录，而让容器中的/dbdata目录可见**，新建多个数据容器同样是隐藏镜像的文件而显示容器中的文件，这种机制实现了数据卷的数据共享。
你可以在一条命令中使用多个`--volumes-from`标记参数把多个容器的数据卷绑定在一起。
上述的代码中db1和db2是挂载dbdata这个容器来扩展的，你也可以挂载db1或者db2来扩展你的数据卷。
```language
 docker run -d --name db3 --volumes-from db1 training/postgres
```
如果你想删除包含挂载数据卷的容器，甚至是初始化的容器`dbdata`，或者是由`dbdata`扩展的db1和db2，容器会删除，但是数据卷会留下。使用`docker rm -v`来删除容器的数据卷。

><span style="color:red;">注意：</span>当你删除容器没有使用`-v`标记的时候，Docker不会提示警告。没有使用`-v`标记删除容器，会让残留的`volumns`变得“无家可归”（就是没有容器再引用这个数据卷）。这样的卷很难删除而且会占用很多空间，我们正在努力改善数据卷的管理，你可以通过` pull request #8484`来跟进我们的进程。

##Backup, restore, or migrate data volumes
>我们可以利用数据卷来有效的备份、恢复和迁移数据

```language
docker run --volumes-from dbdata -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /dbdata
```
命令中我们启动了一个新的容器，它共享了来自`dbdata`容器的数据卷。然后我们挂在了一个本地主机的目录`/backup`。最后我们使用`tar`命令把`/dbdata`中的数据压缩成`dbdata.jar`放到`/backup`中。执行结束我们就完成了数据卷的数据备份工作。

数据恢复
```language
docker run -v /dbdata --name dbdata2 ubuntu /bin/bash
docker run --volumes-from dbdata2 -v $(pwd):/backup busybox tar xvf /backup/backup.tar
```
创建一个新的容器`dbdata2`，解压文件到新的容器的数据卷。

[原文连接](http://docs.docker.com/userguide/dockervolumes/)

