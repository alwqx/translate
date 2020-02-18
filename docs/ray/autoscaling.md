# 自动化集群搭建
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