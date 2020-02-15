# 安装Ray
- [原文链接](https://ray.readthedocs.io/en/latest/installation.html)
- [翻译：@adolphlwq](https://github.com/adolphlwq)
- <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/80x15.png" /></a>

Ray目前支持MacOS和Linux，计划将来支持Windows。

## 最新稳定版
你可以通过下面的命令安装Ray最新稳定版：
```shell
pip install -U ray  # also recommended: ray[debug]
```

## 最新快照(Nightlies)
下面是最新wheels包(由master分支的commit id构建)的链接。运行下面命令安装这些wheels包：
```shell
pip install -U [link to wheel]
```

| Linux | MacOS |
| :--: | :--: |
| [Linux Python 3.7][1] | [MacOS Python 3.7][2] |
| [Linux Python 3.6][3] | [MacOS Python 3.6][4] |
| [Linux Python 3.5][5] | [MacOS Python 3.5][6] |

[1]:https://s3-us-west-2.amazonaws.com/ray-wheels/latest/ray-0.9.0.dev0-cp37-cp37m-manylinux1_x86_64.whl
[2]:https://s3-us-west-2.amazonaws.com/ray-wheels/latest/ray-0.9.0.dev0-cp37-cp37m-macosx_10_13_intel.whl
[3]:https://s3-us-west-2.amazonaws.com/ray-wheels/latest/ray-0.9.0.dev0-cp36-cp36m-manylinux1_x86_64.whl
[4]:https://s3-us-west-2.amazonaws.com/ray-wheels/latest/ray-0.9.0.dev0-cp36-cp36m-macosx_10_13_intel.whl
[5]:https://s3-us-west-2.amazonaws.com/ray-wheels/latest/ray-0.9.0.dev0-cp35-cp35m-manylinux1_x86_64.whl
[6]:https://s3-us-west-2.amazonaws.com/ray-wheels/latest/ray-0.9.0.dev0-cp35-cp35m-macosx_10_13_intel.whl

## 从指定提交安装
通过下面的模板，你可以安装Ray的`master`分支上任意指定提交的wheels包，你需要指定提交的哈希值、Ray版本号、操作系统类型和Python版本：
```shell
pip install https://ray-wheels.s3-us-west-2.amazonaws.com/master/{COMMIT_HASH}/ray-{RAY_VERSION}-{PYTHON_VERSION}-{PYTHON_VERSION}m-{OS_VERSION}_intel.whl
```

例如，提交哈希为`a0ba4499ac645c9d3e82e68f3a281e48ad57f873`、Ray版本号0.9.0.dev0、操作系统为MacOS、Python3.5的wheels包安装代码为：
```shell
pip install https://ray-wheels.s3-us-west-2.amazonaws.com/master/a0ba4499ac645c9d3e82e68f3a281e48ad57f873/ray-0.9.0.dev0-cp35-cp35m-macosx_10_13_intel.whl
```

## 使用Anaconda安装
如果你使用[Anaconda](https://www.anaconda.com/)并且想在一个定义好的虚拟环境(比如`ray`)使用Ray，命令如下：
```shell
conda create --name ray
conda activate ray
conda install --name ray pip
pip install ray
```
通过命令`pip list`确认`ray`已经安装。

## 从源代码编译Ray
使用`pip`安装已经足够满足大多数Ray用户。

但是，如果你想从源代码编译安装，下面的命令适用于MacOS和Linux。

### 依赖
编译Ray需要安装下面的依赖，我们推荐使用[Anaconda](https://www.anaconda.com/)。

Ubuntu运行下面的命令：
```shell
sudo apt-get update
sudo apt-get install -y build-essential curl unzip psmisc

# If you are not using Anaconda, you need the following.
sudo apt-get install python-dev  # For Python 2.
sudo apt-get install python3-dev  # For Python 3.

pip install cython==0.29.0
```

MacOS运行下面的命令：
```shell
brew update
brew install wget

pip install cython==0.29.0
```
如果你使用Anaconda，你还需要运行：
```shell
conda install libgcc
```

### 编译Ray
下面是通过Ray仓库编译：
```shell
git clone https://github.com/ray-project/ray.git

# Install Bazel.
ray/ci/travis/install-bazel.sh

# Optionally build the dashboard (requires Node.js, see below for more information).
pushd ray/python/ray/dashboard/client
npm ci
npm run build
popd

# Install Ray.
cd ray/python
pip install -e . --verbose  # Add --user if you see a permission denied error.
```

### [可选]安装仪表盘
如果你要使用Ray的仪表盘，你需要安装[Node.js](https://nodejs.org/)**并且在安装Ray之前编译安装仪表盘**。编译安装Ray的相关步骤见上文。

仪表盘需要一些额外的Python包，可以通过pip安装。
```python
pip install ray[dashboard]
```

如果你使用Anaconda，并且安装`psutil`或`setproctitle`遇到问题，尝试执行：
```shell
conda install psutil setproctitle
```
执行`ray.init()``ray start --head`命令会打印仪表盘的地址，比如：
```shell
>>> import ray
>>> ray.init()
======================================================================
View the dashboard at http://127.0.0.1:8265.
Note: If Ray is running on a remote node, you will need to set up an
SSH tunnel with local port forwarding in order to access the dashboard
in your browser, e.g. by running 'ssh -L 8265:127.0.0.1:8265
<username>@<host>'. Alternatively, you can set webui_host="0.0.0.0" in
the call to ray.init() to allow direct access from external machines.
======================================================================
```

## Docker镜像
执行脚本构建Docker镜像：
```shell
cd ray
./build-docker.sh
```
上面的脚本会创建多个Docker镜像：
- `ray-project/deploy`镜像包含源代码和二进制文件，适用于终端用户。
- `ray-project/examples`镜像添加了额外的样例程序。
- `ray-project/base-deps`镜像从`Ubuntu Xenial`构建，安装Anaconda和其它基础依赖包，可以作为起点服务开发人员。

下面的命令可以查看构建的镜像：
```shell
docker images
```
输出类似下面：
```shell
REPOSITORY                          TAG                 IMAGE ID            CREATED             SIZE
ray-project/examples                latest              7584bde65894        4 days ago          3.257 GB
ray-project/deploy                  latest              970966166c71        4 days ago          2.899 GB
ray-project/base-deps               latest              f45d66963151        4 days ago          2.649 GB
ubuntu                              xenial              f49eec89601e        3 weeks ago         129.5 MB
```

### 启动Docker中的Ray
启动deployment容器：
```
docker run --shm-size=<shm-size> -t -i ray-project/deploy
```
根据你系统情况把`<shm-size>`替换成合适的大小，比如`512M`或者`2G`。`-t`和`-i`选项表示以交互的方式使用容器。

**注意：**Ray需要大量的共享内存，因为每一个对象存储都把所有的对象存储在共享内存中，所以共享内存的数量将限制对象存储的大小。

执行完会看到类似下面的输出：
```shell
root@ebc78f68d100:/ray#
```

### 测试安装是否成功
通过运行一些测试样例来确认安装是否成功。这步假设你已经克隆了git仓库：
```shell
python -m pytest -v python/ray/tests/test_mini.py
```

## 解决安装Arrow时的问题
下面是一些潜在的问题。

### 安装了不同版本的Flatbuffers
Arrow会拉取和编译自己的Flatbuffers拷贝，但是如果你已经安装了Flatbuffers，Arrow可能会拉取错误的版本。如果输出有类似`/usr/local/include/flatbuffers`的目录，可能就是问题所在。要解决这个问题，请删除旧版本的flatbuffer。

### Boost有问题
如果安装Arrow时出现了类似`If a message like Unable to find the requested Boost libraries`的消息，表明Boost可能有问题。这种情况可能是使用MacPorts安装Boost引起的，这有时可以使用brew安装Boost解决。