#Dockerfile Optimizing
##合理安排命令的顺序
* `apt-get update`类似的操作要放在上面。这样合理利用镜像缓存，不用重新操作
* apt-get update应该与apt-get install组合。此外，你应该采取\的优势使用多行来进行安装
* 像`WORKDIR`、`CMD`、`ENV`这些命令应该在底部
* 任何`ADD`（或其它缓存失效的命令）命令应该尽可能地在Dockerfile底部，在那里你有可能做出很多改变，然后后续命令缓存失效
* 使用CMD和ENTRYPOINT时，请务必使用数组语法
	* Docker会在你的命令前面加上/bin/sh -c，使用数组命令可以避免。
* FROM debian:jessie而不仅仅是FROM debian