#Apply custom metadata
>你可以用过`LABEL`把元数据应用到你的镜像，容器或者是守护进程中。元数据可以服务于广泛的用途。使用标签可以给镜像添加注释或者是许可信息，还可以用来标志你的主机

标签是`<key>` / `<value>`键值对，Docker以字符串的方式存储标签。你可以指定多个标签但是每一个`<key>` / `<value>`必须不同防止对已经存在的键值对覆盖。如果你给同一个`key`指定了多个不同的值，新的值会把之前的值覆盖掉。记住，对于相同的`key`，Docker只会应用你提供的最后一个值。

>**注意**:Docker1.4.1之后的版本才支持daemon-labels，对标签和容器的标签支持是1.6.0中的新特性。

##Label keys (namespaces)
>标签的`键`，（也就是命名空间）

Docker对你创建的标签中的键没有什么硬性的限制，但是简单的键也有可能冲突。例如，你通过`architecture`标签来给你的镜像分类：
```language
LABEL architecture="amd64"
LABEL architecture="ARMv7"
```
而且用户也可以通过不同风格的标签来给镜像打标签：
```language
LABEL architecture="Art Nouveau"
```
为了防止命名冲突，Docker的命名空间标签键使用`反向域名`表示。参考下面来命名你的键：

* 所有的（第三方）工具都用反向域名前缀+标签的方式来命名，这个反向域名要和工具作者提供的域名一致，如`com.example.some-label`，`com.example.some-auther="root"`
* `com.docker.*`, `io.docker.*`和`com.dockerproject.*`保留给Docker内部使用
* 键只能是小写字母，数字，点和`-`表示，及`[a-z0-9-.]`
* 键名的开始和结束只能是字母和数字
* 不能包含连续的`-`和点
* 没有名称空间的标签保留给`CLI`，这就允许最终用户给容器和镜像添加元数据而不必在终端输入繁琐的命令

上面列举的都是准则且Docker严格遵守执行。如果你没有遵守这些准则有可能导致标签名的冲突。如果恰巧你也在使用标签构建工具的话，赶快为你的标签和键使用名称空间吧

##Store structured data in labels
>在标签中存储结构化数据

标签中的值可以包含任何能被存储为字符串的值，例如下面的JSON格式的数据：
```shell
{
    "Description": "A containerized foobar",
    "Usage": "docker run --rm example/foobar [args]",
    "License": "GPL",
    "Version": "0.0.1-beta",
    "aBoolean": true,
    "aNumber" : 0.01234,
    "aNestedArray": ["a", "b", "c"]
}
```
要想把这个结构存储在标签中，首先你要把它序列化为字符串：
```language
LABEL com.example.image-specs="{\"Description\":\"A containerized foobar\",\"Usage\":\"docker run --rm example\\/foobar [args]\",\"License\":\"GPL\",\"Version\":\"0.0.1-beta\",\"aBoolean\":true,\"aNumber\":0.01234,\"aNestedArray\":[\"a\",\"b\",\"c\"]}"
```
虽然可以在标签中存储结构化的数据，但是Docker把它（结构化的数据）看作是普通的字符串。这意味着Docker本身并不提供基于嵌套属性的查询（过滤器）。如果你的工具需要通过嵌套属性来过滤，那么你的工具本身要实现这个功能，二不要让Docker去做。

##Add labels to images; the LABEL instruction
>使用`LABEL`指令给镜像添加标签

```language
LABEL [<namespace>.]<key>[=<value>] ...
```
LABEL指令用来给镜像添加标签，可选择设置它的值。对于使用`空格`的标签，要用`双引号`或者`反斜杠`。
e.g.
```language
LABEL vendor=ACME\ Incorporated
LABEL com.example.version.is-beta
LABEL com.example.version="0.0.1-beta"
LABEL com.example.release-date="2015-02-12"
```
**注意：**上面的第二行中只有键，没有值。
LABEL指令支持在一个`LABEL`下设置多个
e.g.
```language
LABEL com.example.version="0.0.1-beta" com.example.release-date="2015-02-12"
```

Docker允许使用反斜杠`\`,将1行指令分割为多个行
```language
LABEL vendor=ACME\ Incorporated \
      com.example.is-beta \
      com.example.version="0.0.1-beta" \
      com.example.release-date="2015-02-12"
```
**Docker更推荐你在一个LABEL指令中设置多个标签**，每个标签都用指令单独设置的话会让你的镜像很`低效`，这是因为每一个Dockerfile中的LABEL指令都会产生1个镜像层（怒了，这个解释直达本质啊）！！！

我们可以使用`docker inspect`来查看镜像或者容器的标签

##Query labels
>查询标签

标签除了可以用来存储元数据，还可以用来过滤镜像和容器。下面的命令将会列出所有包含`com.example.is-beta`标签并且运行这的容器：
```language
docker ps --filter "label=com.example.is-beta"
```

`color`标签且值为`blue`的运行中的容器
```language
docker ps --filter "label=color=blue"
```

包含`vendor`和`ACME`的镜像
```language
docker images --filter "label=vendor=ACME"
```

##Daemon labels
>守护标签

`docker info`这条命令的解释是：`Display system-wide information（显示全部信息）`
下面是我电脑上的结果：
```language
adolph@geek:~$ docker info
Containers: 7
Images: 44
Storage Driver: aufs
 Root Dir: /var/lib/docker/aufs
 Backing Filesystem: extfs
 Dirs: 58
 Dirperm1 Supported: false
Execution Driver: native-0.2
Kernel Version: 3.13.0-52-generic
Operating System: Ubuntu 14.04.2 LTS
CPUs: 4
Total Memory: 7.687 GiB
Name: geek
ID: HGR7:UGWW:VQVV:WYMF:CSEE:KJ4C:QS4U:IRIU:LREB:M4YC:GDJY:YPI5
Username: adolphlwq
Registry: [https://index.docker.io/v1/]
WARNING: No swap limit support
```
可以看出里面主要是关于Docker daemon的信息，这里并没有关于它的标签信息。但是我们可以通过``docker -d label=value`的方式给Docker daemon本身添加标签：
```language
docker -d \
  --dns 8.8.8.8 \
  --dns 8.8.4.4 \
  -H unix:///var/run/docker.sock \
  --label com.example.environment="production" \
  --label com.example.storage="ssd"
```