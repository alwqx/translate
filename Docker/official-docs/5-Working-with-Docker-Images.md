#Working with Docker Images
##目标
*  本地主机管理镜像
*  创建自己的镜像
*  上传镜像到`Docker Hub registry`

##Listing images on the host
>列出主机的镜像

```shell
adolph@geek:~$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu              14.04               07f8e8c5e660        12 days ago         188.3 MB
hello-world         latest              91c95931e552        3 weeks ago         910 B
training/webapp     latest              31fa814ba25a        11 months ago       278.8 MB
```
镜像来自的仓库，标签和Id这些信息都很重要，来自相同的仓库我们通过`标签`来指定要运行的镜像
```language
docker run -i -t ubuntu:14.04 /bin/bash
docker run -i -t ubuntu:latest /bin/bash     (不加标签默认是`latest`)
```
>个人觉得标签的好处是对于来自相同的仓库通过标签来告诉docker我要运行的是哪个镜像

##Finding images
>寻找镜像

```language
docker search [image name]
docker pull [image name]...找到合适的后直接拉下来
```

##Creating our own images
>创建和定制自己的镜像

* 更改现有镜像并提交（保存）
* 使用`Dockerfile`来制作镜像

###Updating and committing an image
```language
docker run -i -t ubuntu:14.04 /bin/bash
root@5bc673e32e0a:/#apt-get install git
root@5bc673e32e0a:/#exit
sudo docker commit -m "install git" -a "adolph" 0b2616b0e5a8 adolph/ubuntu:14.04

output:4f177bd27a9ff0f6dc2a830403925b5360bfe0b93d476f7fc3231110e7f71b1c
```

###Building an image from a Dockerfile
```language
mkdir ubuntu
cd ubuntu
touch Dockerfile
vim Dockerfile
content:

#this is my fiest image created by dockerfile
FROM ubuntu:14.04
MAINTAINER adolph <test@126.com>
RUN apt-get update
#RUN apt-get install git
```
结果：
```language
adolph@geek:~/ubuntu$ sudo docker build -t adolph-dockerfile/ubuntu:14.04 .
Sending build context to Docker daemon 2.048 kB
Sending build context to Docker daemon 
Step 0 : FROM ubuntu:14.04
 ---> 07f8e8c5e660
Step 1 : MAINTAINER adolph <nalan3015@126.com>
 ---> Running in 241ad7c398c3
 ---> 10d866905c1a
Removing intermediate container 241ad7c398c3
Step 2 : RUN apt-get update
 ---> Running in 826db7ff28e6
Ign http://archive.ubuntu.com trusty InRelease
Ign http://archive.ubuntu.com trusty-updates InRelease
。。。。。。。。。
```

sudo docker build -t adolph-dockerfile/ubuntu:14.04 **<span style="color:red;">.</span>** 

**注意上述命令有个`.`用来指定Dockerfile文件的位置**

##Setting tags on an image
>给镜像添加标签

```language
docker tag [image id] [username]/[repository]:[tag name]
```

##Push an image to Docker Hub
>需要登录Docker Hub账号

`docker push [image name]`

##Remove an image from the host
`docker rmi [image name]`

##总结
```language
docker images...查看docker镜像
docker search [images name]...查找镜像
```

`sudo docker commit -m "install git" -a "adolph" 0b2616b0e5a8（old image id） adolph/ubuntu:14.04(new image name with tag 14.04)`
这条命令很重要和难记，`-m`和`git commit -m`的作用类似，`-a`指定作者,id指`更改过的image的id`，`adolph/ubuntu`是新的image的名字

```language
docker build -t adolph-dockerfile/ubuntu:14.04 .
后面要指定dockerfile的地址
docker tag [image id] [username]/[repository]:[tag name]
docker push [image name]
docker rmi [image name]
```



[原文链接](https://docs.docker.com/userguide/dockerimages/)