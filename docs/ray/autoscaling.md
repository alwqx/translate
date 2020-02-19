# 自动化集群搭建

- [原文链接](https://ray.readthedocs.io/en/latest/autoscaling.html)
- [翻译：@adolphlwq](https://github.com/adolphlwq)
- <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/80x15.png" /></a>


>该部分介绍了使用自带的autoscaler自动搭建集群的过程。Ray支持根据需要自动伸缩节点，一提高节点利用率，降低成本。

Ray内建了autoscaler，让部署Ray集群很容易。只需要在本地机器运行`ray up`就可以启动或者更新位于云端或者内部的集群。一旦Ray集群运行起来，你就可以手动SSH到集群，或者通过提供的命令`ray attach`、`ray rsync-up`和`ray-exec`来访问集群和运行Ray程序。

## 搭建
这部分提供指令用于配置autoscaler在AWS、GCP、Kubernetes或私有集群上启动Ray集群。

一旦你完成配置autoscaler创建Ray集群，查看下面的快速开始指南获取在集群上运行Ray程序的详细信息。

### AWS
首先，安装boto(`pip install boto3`)然后按照[boto文档(http://boto3.readthedocs.io/en/latest/guide/configuration.html)描述的在`~/.aws/credentials`里配置你的AWS凭据。

一旦配置好boto来管理你AWS账号上的资源，你就应该准备好运行autoscaler了。Ray仓库中提供的集群配置文件`ray/python/ray/autoscaler/aws/example-full.yaml`会创建一个小集群：一个m5.large的head节点(你可以按照需求改变)，它会自动创建两个m5.large的spot worker。

你可以从本地机器运行下面的代码来测试集群：
```shell
# Create or update the cluster. When the command finishes, it will print
# out the command that can be used to SSH into the cluster head node.
$ ray up ray/python/ray/autoscaler/aws/example-full.yaml

# Get a remote screen on the head node.
$ ray attach ray/python/ray/autoscaler/aws/example-full.yaml
$ source activate tensorflow_p36
$ # Try running a Ray program with 'ray.init(address="auto")'.

# Tear down the cluster.
$ ray down ray/python/ray/autoscaler/aws/example-full.yaml
```

?>建议：对于AWS的节点配置，你可以设置`"ImageId: latest_dlami"`自动在你的区使用最新的[Deep Learning AMI](https://aws.amazon.com/machine-learning/amis/)，比如：`head_node: {InstanceType: c5.xlarge, ImageId: latest_dlami}`。

?>注意：你可能会看到类似这样的消息：`bash: cannot set terminal process group (-1): Inappropriate ioctl for device bash: no job control in this shell`。这是一个无害的错误，如果集群启动失败，很可能是其它因素。

### GCP
首先，安装Google API client `pip install google-api-python-client`，设置你的GCP凭据，创建一个新的GCP项目。

一旦API clint配置好管你GCP账号的资源，你就应该准备好运行autoscaler了。Ray仓库中提供的集群配置文件`ray/python/ray/autoscaler/gcp/example-full.yaml`会创建一个小集群：一个n1-standard-2 head节点(可以按需调整)，它会自动创建两个n1-standard-2 [preemptible workers](https://cloud.google.com/preemptible-vms/)。注意，你需要在模板中填入你的GCP项目ID。

你可以从本地机器运行下面的代码来测试集群：
```shell
# Create or update the cluster. When the command finishes, it will print
# out the command that can be used to SSH into the cluster head node.
$ ray up ray/python/ray/autoscaler/gcp/example-full.yaml

# Get a remote screen on the head node.
$ ray attach ray/python/ray/autoscaler/gcp/example-full.yaml
$ source activate tensorflow_p36
$ # Try running a Ray program with 'ray.init(address="auto")'.

# Tear down the cluster.
$ ray down ray/python/ray/autoscaler/gcp/example-full.yaml
```

### Kubernetes
autoscaler还可以用来在Kubernetes集群上运行Ray集群。首先，安装Kubernetes API client(`pip install kubernetes`)。然后，确保Kubernetes凭据被正确配置，可以访问Kubernetes集群(如果`kubectl get pods`命令执行成功，则你的配置是好的)。

一旦`kubectl`本地配置正确可以访问远程集群，你就应该准备好运行autoscaler了。Ray仓库中提供的集群配置文件`ray/python/ray/autoscaler/kubernetes/example-full.yaml`会创建一个小集群：一个pod用于head节点，它被配置自动启动两个pod用于worker node，所有pod需要1CPU和0.5GiB内存。

你可以从本地机器运行下面的代码来测试集群：
```shell
# Create or update the cluster. When the command finishes, it will print
# out the command that can be used to get a remote shell into the head node.
$ ray up ray/python/ray/autoscaler/kubernetes/example-full.yaml

# List the pods running in the cluster. You shoud only see one head node
# until you start running an application, at which point worker nodes
# should be started. Don't forget to include the Ray namespace in your
# 'kubectl' commands ('ray' by default).
$ kubectl -n ray get pods

# Get a remote screen on the head node.
$ ray attach ray/python/ray/autoscaler/kubernetes/example-full.yaml
$ # Try running a Ray program with 'ray.init(address="auto")'.

# Tear down the cluster
$ ray down ray/python/ray/autoscaler/kubernetes/example-full.yaml
```

### 私有集群
autoscaler还可以用来在私有主机集群上部署Ray集群，用来链接的机器通过IP地址指定。你可以通过填充文件`ray/python/ray/autoscaler/local/example-full.yaml`相关字段开始。确保填写正确的`head_ip`、`worker_ips`列表和`ssh_user`这三个字段。

你可以从本地机器运行下面的代码来测试集群：
```shell
# Create or update the cluster. When the command finishes, it will print
# out the command that can be used to get a remote shell into the head node.
$ ray up ray/python/ray/autoscaler/local/example-full.yaml

# Get a remote screen on the head node.
$ ray attach ray/python/ray/autoscaler/local/example-full.yaml
$ # Try running a Ray program with 'ray.init(address="auto")'.

# Tear down the cluster
$ ray down ray/python/ray/autoscaler/local/example-full.yaml
```

### 外部节点提供者
Ray还支持外部节点提供者(在[node_provider.py](https://github.com/ray-project/ray/tree/master/python/ray/autoscaler/node_provider.py)中实现)。你可以通过yaml格式的配置文件来指定：
```
provider:
    type: external
    module: mypackage.myclass
```

module字段要以`package.provider_class`或者`package.sub_package.provider_class`的形式。

### 其它云供应商
若要在其它云供应商或集群管理系统上使用Ray的自动伸缩功能，你要自己实现`NodeProvider`接口(大约100行代码)并且注册到[node_provider.py](https://github.com/ray-project/ray/tree/master/python/ray/autoscaler/node_provider.py)中。欢迎贡献代码！

## 快速开始
### 启动和更新集群
This includes any changes to synced files specified in the file_mounts section of the config

当你对现有集群运行`ray up`时，这条命令会检查本地配置和应用到集群中的配置是否不同。这包括在配置项`file_mounts`部分指定的对同步文件的任何更改。如果改变，新的文件和配置项会被更新到集群中。之后，Ray服务会被重启。

如果集群可能处于坏的状态，你还可以使用`ray up`重启集群(这会导致Ray重启所有的服务，即使配置文件没有任何改变)。

如果你不希望更新过程重启服务(比如，变更不需要重启)，可以给重启调用传递`--no-restart`选项：
```shell
# Replace '<your_backend>' with one of: 'aws', 'gcp', 'kubernetes', or 'local'.
$ BACKEND=<your_backend>

# Create or update the cluster.
$ ray up ray/python/ray/autoscaler/$BACKEND/example-full.yaml

# Reconfigure autoscaling behavior without interrupting running jobs.
$ ray up ray/python/ray/autoscaler/$BACKEND/example-full.yaml \
    --max-workers=N --no-restart

# Tear down the cluster.
$ ray down ray/python/ray/autoscaler/$BACKEND/example-full.yaml
```

### 在已有的新集群运行命令
你可以使用`ray exec`命令方便地(`conveniently`)在集群上运行命令，注意你运行的脚本需要通过`ray.init(address="auto")`连接到Ray：
```shell
# Run a command on the cluster
$ ray exec cluster.yaml 'echo "hello world"'

# Run a command on the cluster, starting it if needed
$ ray exec cluster.yaml 'echo "hello world"' --start

# Run a command on the cluster, stopping the cluster after it finishes
$ ray exec cluster.yaml 'echo "hello world"' --stop

# Run a command on a new cluster called 'experiment-1', stopping it after
$ ray exec cluster.yaml 'echo "hello world"' \
    --start --stop --cluster-name experiment-1

# Run a command in a detached tmux session
$ ray exec cluster.yaml 'echo "hello world"' --tmux

# Run a command in a screen (experimental)
$ ray exec cluster.yaml 'echo "hello world"' --screen
```

你还可以通过`ray submit`在集群执行python脚本。这条命令会把指定的脚本`rsync`到集群上，然后根据参数执行。
```shell
# Run a Python script in a detached tmux session
$ ray submit cluster.yaml --tmux --start --stop tune_experiment.py
```

### attach到运行的集群上
使用`ray attach`命令attach到集群的交互式屏幕会话。
```shell
# Open a screen on the cluster
$ ray attach cluster.yaml

# Open a screen on a new cluster called 'session-1'
$ ray attach cluster.yaml --start --cluster-name=session-1

# Attach to tmux session on cluster (creates a new one if none available)
$ ray attach cluster.yaml --tmux
```

### 应用端口转发
如果你想在集群上运行web应用(想Jupyter Notebook能通过浏览器访问的应用)，需要在`ray exec`添加`--port-forward`选项。默认情况下本地端口和远程端口一致。

注意：对于Kubernetes集群，执行命令时`port-forward`不会被使用。你需要分别执行两次`ray exec`命令：
```shell
$ ray exec cluster.yaml --port-forward=8899 'source ~/anaconda3/bin/activate tensorflow_p36 && jupyter notebook --port=8899'
```

### 手动同步文件
通过`ray rsync_down`和`ray rsync_up`命令向集群的head节点下载或上传文件：
```shell
$ ray rsync_down cluster.yaml '/path/on/cluster' '/local/path'
$ ray rsync_up cluster.yaml '/local/path' '/path/on/cluster'
```

### 安全
对于云提供商场景，节点默认被启动在他们自己的安全组，流量只允许在同一个组节点间传输。会创建新的SSH key存储在本地机器上，用于访问集群。

### 自动伸缩
Ray集群使用基于负载的autoscaler。当集群资源使用情况超过阈值(默认80%)时，会在指定的`max_workers`限制内启动新节点。当空闲节点超过一个超时时间时(`When nodes are idle for more than a timeout`)，在`min_workers`限制内该节点被移除。head节点永不删除。

空闲超时默认5分钟，`This is to prevent excessive node churn which could impact performance and increase costs`这是为了预防可能影响性能和增加成本的过渡节点波动(在AWS/GCP中，每个实例的最低计费时间为1分钟，之后按秒计费)。

### 监控集群状态
Ray还提供了dashboard，它在head节点上通过HTTP访问(默认监听在`localhost:8265`)。如果要本地访问，你需要转发到本地端口，或者你可以使用内置命令`ray dashboard`自动化操作。

你还可以通过查看`/tmp/ray/session_*/logs/monitor*`里的自动伸缩日志监控集群的日志和伸缩状态。

Ray autoscaler还会以实例标签的形式报告每个节点的状态。在你的云提供商控制台，你可以点击节点，进入标签面板，然后添加标签`ray-node-status`作为列。这会帮你看到每个节点的状态。

![](https://ray.readthedocs.io/en/latest/_images/autoscaler-status.png)

### 定制化启动集群
我们鼓励你拷贝示例YAML文件并根据需要进行修改。这可能包括添加额外的命令来安装库或者同步本地数据文件。

?>注意：如果你启动了自己定制化的节点，把该节点创建成新的机器镜像(或者docker容器)并在配置文件中使用是个好主意。这会worker设置时间，改善自动伸缩的效率。

你的设置命令理想情况下应该是`幂等`的，即可以多次运行结果相同。这让Ray在节点创建后能够更新节点。通常你需要很小的修改使命令幂等，比如`git clone foo`可以改写为`test -e foo || git clone foo`，它首先会检查仓库是否已经克隆过了。

大多数示例YAML文件都可以修改，这里有[最小YAML配置文件参考](https://github.com/ray-project/ray/tree/master/python/ray/autoscaler/aws/example-minimal.yaml)，你可以在[完全YAML配置文件](https://github.com/ray-project/ray/tree/master/python/ray/autoscaler/aws/example-full.yaml)找到可选字段的默认值。

### 同步git分支
一个常用的使用案例是同步本地git逻辑分支到集群所有worker上。但是，如果你只在设置命令中放置命令`git checkout <branch>`，autoscaler将不知道何时重新运行这条命令拉取更新。一个好的解决方案(`workaround`)是包含git SHA在输入中(分支更新后，文件的哈希值会变)：
```shell
file_mounts: {
    "/tmp/current_branch_sha": "/path/to/local/repo/.git/refs/heads/<YOUR_BRANCH_NAME>",
}

setup_commands:
    - test -e <REPO_NAME> || git clone https://github.com/<REPO_ORG>/<REPO_NAME>.git
    - cd <REPO_NAME> && git fetch && git checkout `cat /tmp/current_branch_sha`
```

这个设置会告诉`ray up`从你个人电脑同步当前git分支的哈希值到集群的临时文件上(假设你已经把git branch head更新到仓库)。然后设置命令会读取文件并计算应该检出哪个哈希值到节点。**注意每个命令运行在自己的会话中**。最后，集群更新的工作流如下：
1. git分支产生变更
2. 使用`git commit`和`git push`提交变更
3. 使用`ray up`更新Ray集群中的文件

### 使用Amazon EFS
要使用Amazon EFS，需要安装一些工具库并在`setup_commands`中挂载EFS。**注意这些指令只有在你使用AWS Autoscaler才生效**。

?>注意：使用配置前你需要把`{{FileSystemId}}`替换成自己的EFS ID。你可能还要为实例设置正确的`SecurityGroupIds`。

```shell
setup_commands:
    - sudo kill -9 `sudo lsof /var/lib/dpkg/lock-frontend | awk '{print $2}' | tail -n 1`;
        sudo pkill -9 apt-get;
        sudo pkill -9 dpkg;
        sudo dpkg --configure -a;
        sudo apt-get -y install binutils;
        cd $HOME;
        git clone https://github.com/aws/efs-utils;
        cd $HOME/efs-utils;
        ./build-deb.sh;
        sudo apt-get -y install ./build/amazon-efs-utils*deb;
        cd $HOME;
        mkdir efs;
        sudo mount -t efs {{FileSystemId}}:/ efs;
        sudo chmod 777 efs;
```

### 常见集群配置
`example-full.yaml`中的配置足够使用Ray了，但是对于更加计算密集型负载你可能想改变实例类型使用更多的GPU或更大的计算实例。这里是一些常见集群配置：

**GPU single node**：在单个更大的GPU实例使用Ray。
```shell
max_workers: 0
head_node:
    InstanceType: p2.8xlarge
```

**Docker**：指定docker image。这样所有的命令会在所有节点的docker容器里执行，还会打开所有必要端口支持Ray集群。如果节点没有安装docker，还会自动安装。目前这还不支持GPU。
```shell
docker:
    image: tensorflow/tensorflow:1.5.0-py3
    container_name: ray_docker
```

**Mixed GPU and CPU nodes**：`for RL applications that require proportionally more CPU than GPU resources`RL应用需要比GPU更高比例的CPU。你可以使用额外的CPU worker和GPU head节点。
```shell
max_workers: 10
head_node:
    InstanceType: p2.8xlarge
worker_nodes:
    InstanceType: m4.16xlarge
```

**Autoscaling CPU cluster**：使用小的head节点并让Ray根据需要自动伸缩worker节点。对于负载大的集群，这个配置高效经济。对于额外的开销，你还可以现场请求worker。
```shell
min_workers: 0
max_workers: 10
head_node:
    InstanceType: m4.large
worker_nodes:
    InstanceMarketOptions:
        MarketType: spot
    InstanceType: m4.16xlarge
```

**Autoscaling GPU cluster**：和上面类似，但是使用了GPU worker。
```shell
min_workers: 0  # NOTE: older Ray versions may need 1+ GPU workers (#2106)
max_workers: 10
head_node:
    InstanceType: m4.large
worker_nodes:
    InstanceMarketOptions:
        MarketType: spot
    InstanceType: p2.xlarge
```