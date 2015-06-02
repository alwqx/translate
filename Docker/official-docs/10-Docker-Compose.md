#Docker Compose
##综述
Compose是使用Docker定义和运行复杂应用的工具。有了Compose，你可以在单个文件中定义多容器应用，你只需要一个能让所有事情运转起来的命令就可以让你的应用运转起来了。

Compose在开发环境中使用是非常棒的：临时服务器、持续集成。我们并不推荐在生产环境中使用Compose。

只需要基本的3步就可以使用Compose啦：

首先，你要在`Dockerfile`中定义app的环境以便它（app）能在任何地方再生产
```language
FROM python:2.7
WORKDIR /code
ADD requirements.txt /code/
RUN pip install -r requirements.txt
ADD . /code
CMD python app.py
```

然后，你要在`docker-compose.yml`中定义组成你应用的服务，以便他们可以在孤立的环境中一起运行。
```language
web:
  build: .
  links:
   - db
  ports:
   - "8000:8000"
db:
  image: postgres
```

最后，运行`docker-compose up`后Compose会启动和运行你的整个应用。

Compose有以下命令来管理你应用的整个生命周期：
* Start,stop和rebuild服务
* 查看运行状态服务
* 流式输出运行服务的日志
* 在单个服务中运行一次性命令

##Compose documentation
* [Installing Compose](http://docs.docker.com/compose/install/)
* [Command line reference](http://docs.docker.com/compose/cli/)
* [Yaml file reference](http://docs.docker.com/compose/yml/)
* [Compose environment variables](http://docs.docker.com/compose/env/)
* [Compose command line completion](http://docs.docker.com/compose/completion/)

##Quick start
我们从在Compose上运行的一个简单的python web app为例子作为开始。只需要一点python知识，这里的概念只需要了解，不需要你非常熟悉。

####Installation and set-up
首先[安装Docker和Compose](http://docs.docker.com/compose/install/)
然后新建项目的目录
```language
mkdir composetest
$ cd composetest
```
在目录中创建`app.py`文件。


[原文连接](http://docs.docker.com/compose/)
