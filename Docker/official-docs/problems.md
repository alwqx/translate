#疑问
##docker中images和container的区别
>images

docker镜像（images）有中间层，它增加了重用性，减少硬盘的使用以及通过允许缓存每个步骤来加速构建。这些中间层默认不显示

`VIRTUAL SIZE`指的是镜像和它所有的父镜像所占据的空间，这个大小也是你执行`docker save`命令生成镜像的`tar`文件占据的磁盘空间大小

一个镜像可以被列举多次如果它有多个仓库名或者标签。但是它们用的`VIRTUAL SIZE`只被列举1次