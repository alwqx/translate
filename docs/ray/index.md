# Ray

![](https://github.com/ray-project/ray/raw/master/doc/source/images/ray_header_logo.png)

- [原文链接](https://ray.readthedocs.io/en/latest/)
- [翻译：@adolphlwq](https://github.com/adolphlwq)
- <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/80x15.png" /></a>

**Ray是一个快速、简单的框架，用于构建和运行分布式应用。**

Ray打包了下面的库，用来加速机器学习计算任务。
- [Tune](https://ray.readthedocs.io/en/latest/tune.html): 可扩展超参调优库
- [RLlib](https://ray.readthedocs.io/en/latest/rllib.html): 可扩展强化学习库
- [RaySGD](https://ray.readthedocs.io/en/latest/raysgd/raysgd.html): 分布式训练库

欢迎在[GitHub](https://github.com/ray-project/ray)上star该项目。你还可以访问[教程](https://github.com/ray-project/tutorial)入门。参考[安装页](https://ray.readthedocs.io/en/latest/installation.html)获取最新版本。

?>重要通知：欢迎加入[Slack社区](https://forms.gle/9TSdDYUgxYs8SA9e8)讨论Ray。

## 快速开始
首先，通过`pip install ray`安装Ray
```python
# Execute Python functions in parallel.

import ray
ray.init()

@ray.remote
def f(x):
    return x * x

futures = [f.remote(i) for i in range(4)]
print(ray.get(futures))
```

使用Ray的actor模型：
```python
import ray
ray.init()

@ray.remote
class Counter(object):
    def __init__(self):
        self.n = 0

    def increment(self):
        self.n += 1

    def read(self):
        return self.n

counters = [Counter.remote() for i in range(4)]
[c.increment.remote() for c in counters]
futures = [c.read.remote() for c in counters]
print(ray.get(futures))
```

访问[细节页面](https://ray.readthedocs.io/en/latest/walkthrough.html)获取对Ray特性更全面的描述。

Ray程序可以运行在单个机器上，也可以无缝扩展运行在大的集群中。上面的代码要运行在云上，只需下载[配置文件](https://github.com/ray-project/ray/blob/master/python/ray/autoscaler/aws/example-full.yaml)并执行`ray submit [CLUSTER.YAML] example.py --start`。阅读[启动集群](https://ray.readthedocs.io/en/latest/autoscaling.html)获取更多信息。

## Tune快速开始
Tune是用于任何规模的超参调优库。使用Tune，你可以用少于10行代码启动多节点、分布式超参扫描任务。Tune支持当前所有深度学习框架，包括PyTorch、TensorFlow、和Keras。

?>注意：为了运行下面的例子，你需要先安装下面的库：
```python
$ pip install ray torch torchvision filelock
```

下面的代码是用PyTorch和Tune运行网格搜索算法来训练CNN：
```python
import torch.optim as optim
from ray import tune
from ray.tune.examples.mnist_pytorch import get_data_loaders, ConvNet, train, test


def train_mnist(config):
    train_loader, test_loader = get_data_loaders()
    model = ConvNet()
    optimizer = optim.SGD(model.parameters(), lr=config["lr"])
    for i in range(10):
        train(model, optimizer, train_loader)
        acc = test(model, test_loader)
        tune.track.log(mean_accuracy=acc)


analysis = tune.run(
    train_mnist, config={"lr": tune.grid_search([0.001, 0.01, 0.1])})

print("Best config: ", analysis.get_best_config(metric="mean_accuracy"))

# Get a dataframe for analyzing trial results.
df = analysis.dataframe()
```

如果安装了TensorBoard可以自动可视化所有的训练结果：
```python
tensorboard --logdir ~/ray_results
```

## RLlib快速开始
RLlib是构建在Ray之上的用于强化学习的开源库，它为不同的应用提供高可用性和统一的API。
```python
pip install tensorflow  # or tensorflow-gpu
pip install ray[rllib]  # also recommended: ray[debug]
```

```python
import gym
from gym.spaces import Discrete, Box
from ray import tune

class SimpleCorridor(gym.Env):
    def __init__(self, config):
        self.end_pos = config["corridor_length"]
        self.cur_pos = 0
        self.action_space = Discrete(2)
        self.observation_space = Box(0.0, self.end_pos, shape=(1, ))

    def reset(self):
        self.cur_pos = 0
        return [self.cur_pos]

    def step(self, action):
        if action == 0 and self.cur_pos > 0:
            self.cur_pos -= 1
        elif action == 1:
            self.cur_pos += 1
        done = self.cur_pos >= self.end_pos
        return [self.cur_pos], 1 if done else 0, done, {}

tune.run(
    "PPO",
    config={
        "env": SimpleCorridor,
        "num_workers": 4,
        "env_config": {"corridor_length": 5}})
```

## 更多信息
下面是关于Ray和库的相关演讲、论文和新闻报道，如果有的网址失效请发起Issue。

### 博客和报道
- [Modern Parallel and Distributed Python: A Quick Tutorial on Ray](https://towardsdatascience.com/modern-parallel-and-distributed-python-a-quick-tutorial-on-ray-99f8d70369b8)
- [Why Every Python Developer Will Love Ray](https://www.datanami.com/2019/11/05/why-every-python-developer-will-love-ray/)
- [Ray: A Distributed System for AI (BAIR)](http://bair.berkeley.edu/blog/2018/01/09/ray/)
- [10x Faster Parallel Python Without Python Multiprocessing](https://towardsdatascience.com/10x-faster-parallel-python-without-python-multiprocessing-e5017c93cce1)
- [Implementing A Parameter Server in 15 Lines of Python with Ray](https://ray-project.github.io/2018/07/15/parameter-server-in-fifteen-lines.html)
- [Ray Distributed AI Framework Curriculum](https://rise.cs.berkeley.edu/blog/ray-intel-curriculum/)
- [RayOnSpark: Running Emerging AI Applications on Big Data Clusters with Ray and Analytics Zoo](https://medium.com/riselab/rayonspark-running-emerging-ai-applications-on-big-data-clusters-with-ray-and-analytics-zoo-923e0136ed6a)
- [First user tips for Ray](https://rise.cs.berkeley.edu/blog/ray-tips-for-first-time-users/)
- [Tune] [Tune: a Python library for fast hyperparameter tuning at any scale](https://towardsdatascience.com/fast-hyperparameter-tuning-at-scale-d428223b081c)
- [Tune] [Cutting edge hyperparameter tuning with Ray Tune](https://medium.com/riselab/cutting-edge-hyperparameter-tuning-with-ray-tune-be6c0447afdf)
- [RLlib] [New Library Targets High Speed Reinforcement Learning](https://www.datanami.com/2018/02/01/rays-new-library-targets-high-speed-reinforcement-learning/)
- [RLlib] [Scaling Multi Agent Reinforcement Learning](http://bair.berkeley.edu/blog/2018/12/12/rllib/)
- [RLlib] [Functional RL with Keras and Tensorflow Eager](https://bair.berkeley.edu/blog/2019/10/14/functional-rl/)
- [Modin] [How to Speed up Pandas by 4x with one line of code](https://www.kdnuggets.com/2019/11/speed-up-pandas-4x.html)
- [Modin] [Quick Tip – Speed up Pandas using Modin](https://pythondata.com/quick-tip-speed-up-pandas-using-modin/)
- [Ray Blog](https://ray-project.github.io/)

### 演讲（视频）
- [Programming at any Scale with Ray | SF Python Meetup Sept 2019](https://www.youtube.com/watch?v=LfpHyIXBhlE)
- [Ray for Reinforcement Learning | Data Council 2019](https://www.youtube.com/watch?v=Ayc0ca150HI)
- [Scaling Interactive Pandas Workflows with Modin](https://www.youtube.com/watch?v=-HjLd_3ahCw)
- [Ray: A Distributed Execution Framework for AI | SciPy 2018](https://www.youtube.com/watch?v=D_oz7E4v-U0)
- [Ray: A Cluster Computing Engine for Reinforcement Learning Applications | Spark Summit](https://www.youtube.com/watch?v=xadZRRB_TeI)
- [RLlib: Ray Reinforcement Learning Library | RISECamp 2018](https://www.youtube.com/watch?v=eeRGORQthaQ)
- [Enabling Composition in Distributed Reinforcement Learning | Spark Summit 2018](https://www.youtube.com/watch?v=jAEPqjkjth4)
- [Tune: Distributed Hyperparameter Search | RISECamp 2018](https://www.youtube.com/watch?v=38Yd_dXW51Q)  

### 幻灯片
- [Talk given at UC Berkeley DS100](https://docs.google.com/presentation/d/1sF5T_ePR9R6fAi2R6uxehHzXuieme63O2n_5i9m7mVE/edit?usp=sharing)
- [Talk given in October 2019](https://docs.google.com/presentation/d/13K0JsogYQX3gUCGhmQ1PQ8HILwEDFysnq0cI2b88XbU/edit?usp=sharing)
- [Tune] [Talk given at RISECamp 2019](https://docs.google.com/presentation/d/1v3IldXWrFNMK-vuONlSdEuM82fuGTrNUDuwtfx4axsQ/edit?usp=sharing)

### 学术论文
- [Ray paper](https://arxiv.org/abs/1712.05889)
- [Ray HotOS paper](https://arxiv.org/abs/1703.03924)
- [RLlib paper](https://arxiv.org/abs/1712.09381)
- [Tune paper](https://arxiv.org/abs/1807.05118)

### 参与
- [ray-dev@googlegroups.com](https://groups.google.com/forum/#!forum/ray-dev): For discussions about development or any general questions.
- [StackOverflow](https://stackoverflow.com/questions/tagged/ray): For questions about how to use Ray.
- [GitHub Issues](https://github.com/ray-project/ray/issues): For reporting bugs and feature requests.
- [Pull Requests](https://github.com/ray-project/ray/pulls): For submitting code contributions.