# Ray核心预览
- [原文链接](https://ray.readthedocs.io/en/latest/walkthrough.html)
- [翻译：@alwqx](https://github.com/alwqx)
- <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/80x15.png" /></a>

预览部分将概述Ray的核心概念：
- 启动Ray
- 使用远程函数(tasks)[`ray.remote`]
- 抓取结果(Object IDs)[`ray.put`,`ray.get`,`ray.wait`]
- 使用远程类(actors)[`ray.remote`]

使用Ray，你的代码不仅可以单机工作，也很容易扩展到大集群中。

## 安装
为了运行预览部分的代码，你需要通过命令`pip install -U ray`安装Ray。若要安装最新版wheels包（即`master`分支的快照），你可以使用[最新版快照(Nightlies)](https://ray.readthedocs.io/en/latest/installation.html#install-nightlies)中的指令。

## 运行Ray
将下面的代码加入你的Python脚本中即可在单机中启动Ray：
```python
import ray

# Start Ray. If you're connecting to an existing cluster, you would use
# ray.init(address=<cluster-address>) instead.
ray.init()

...
```
Ray将会使用你机器的所有CPU核心，你可以在[配置Ray](https://ray.readthedocs.io/en/latest/configure.html#configuring-ray)中找到如何配置Ray使用的CPU核心数。

如何启动多节点Ray集群请浏览[集群设置页面](https://ray.readthedocs.io/en/latest/using-ray-on-a-cluster.html)。

## 远程函数(Tasks)
Ray允许任意Python函数异步(`asynchronous`)执行。这些异步Ray函数被称为"远程函数"。把Python函数转换成远程函数的标准方法是使用`@ray.remote`装饰器。下面是样例：
```python
# A regular Python function.
def regular_function():
    return 1

# A Ray remote function.
@ray.remote
def remote_function():
    return 1
```

上面的代码有如下改变：
1. 函数调用：正常版本会调用`regular_function()`，远程版本会调用`regular_function.remote()`
2. 返回值：`regular_function`会立刻执行并返回1，然而`remote_function`会返回Object ID(后面介绍)然后创建一个将被worker进程执行的task。结果可以通过`ray.get`获取。

```python
assert regular_function() == 1

object_id = remote_function.remote()

# The value of the original `regular_function`
assert ray.get(object_id) == 1
```
3. 并发：`regular_function`函数顺序调用，例如：
```python
# These happen serially.
for _ in range(4):
    regular_function()
```
然而`remote_function`并发调用，即：
```python
# These happen in parallel.
for _ in range(4):
    remote_function.remote()
```

访问[ray.remote package reference](https://ray.readthedocs.io/en/latest/package-ref.html)查阅`ray.remote`细节文档。

**Object IDs**也可以传递给远程函数。当函数实际执行时，该参数将作为常规Python对象检索(When the function actually gets executed, the argument will be a retrieved as a regular Python object.参考有道翻译)。比如，执行下面的函数：
```python
@ray.remote
def remote_chain_function(value):
    return value + 1


y1_id = remote_function.remote()
assert ray.get(y1_id) == 1

chained_id = remote_chain_function.remote(y1_id)
assert ray.get(chained_id) == 2
```

注意下面的行为：
- 直到第一个task执行结束第二个task才会执行，因为第二个task依赖第一个task的结果。
- 如果这两个task被调度在不同的机器上，那么第一个task的结果(`y1_id`对应的值)会通过网络传输到第二个task调度的机器上。

有时，你会需要指定task需要的资源(例如一个task需要一个GPU)。`ray.init()`会自动检测机器上可用的CPUs和GPUs。但是，你可以通过指定具体的资源来覆盖默认行为，比如`ray.init(num_cpus=8, num_gpus=4, resources={'Custom': 2})`。

如要指定task的CPU和GPU，把参数`num_cpus`和`num_gpus`传递给remote装饰器即可。这样task只会运行在满足CPU和GPU资源的机器上。Ray可以处理任意自定义资源。

?>注意：
- 如果你不在装饰器`@ray.remote`指定任何资源，默认只有1核CPU，没有其它资源。
- 如果指定CPU，Ray就不会强制隔离(比如，你的task要能满足它的要求)**啥意思？没懂...**
- 如果指定GPU，Ray则会一显示形式提供隔离(设置环境变量`CUDA_VISIBLE_DEVICES`)，但是实际使用资源是Task的职责(比如通过TensorFlow或PyTorch深度学习框架来使用资源)。

```python
@ray.remote(num_cpus=4, num_gpus=2)
def f():
    return 1
```

?>The resource requirements of a task have implications for the Ray’s scheduling concurrency. In particular, the sum of the resource requirements of all of the concurrently executing tasks on a given node cannot exceed the node’s total resources.

task的资源需求会影响Ray的并发调度。特别地，给定节点上并发执行的task资源需求总和不能超过该节点的总资源。

?>Below are more examples of resource specifications:
下面是更多资源规范示例：
```python
# Ray also supports fractional resource requirements
@ray.remote(num_gpus=0.5)
def h():
    return 1

# Ray support custom resources too.
@ray.remote(resources={'Custom': 1})
def f():
    return 1
```

将来，远程函数会返回更多Object IDs：
```python
@ray.remote(num_return_vals=3)
def return_multiple():
    return 1, 2, 3

a_id, b_id, c_id = return_multiple.remote()
```

## Ray中的对象
Ray里面，我们可以创建和计算对象。我们把这些对象称为远程对象，并通过对象ID引用它们。远程对象存储在[共享内存](https://en.wikipedia.org/wiki/Shared_memory)对象存储中，集群中的每个节点都有一个对象存储。在集群设置中，我们实际上可能不知道每个对象在哪台机器上。

对象ID本质上是用来标志远程对象的唯一ID。如果你熟悉futures库(译者注：futures指Python中执行并行任务的[futures](https://docs.python.org/3/library/concurrent.futures.html)库)，它们在概念上类似。

有多种方式创建对象ID：
1. 由远程函数调用返回。
2. 通过`ray.put`返回。

```python
y = 1
object_id = ray.put(y)
```

```doc
ray.put(value, weakref=False)[source]
    Store an object in the object store.

    The object may not be evicted while a reference to the returned ID exists.

    Parameters:
    value – The Python object to be stored.
    weakref – If set, allows the object to be evicted while a reference to the returned ID exists. You might want to set this if putting a lot of objects that you might not need in the future.
    Returns:
    The object ID assigned to this value.
```

This allows remote objects to be replicated in multiple object stores without needing to synchronize the copies.

?>重要：远程对象是不可变的。这意味着，它们的值创建后不能改变。这一机制允许在多个对象存储中复制远程对象，而不需要同步副本。

## 抓取结果
`ray.get(x_id, timeout=None)`函数会接收对象ID然后从对应的远程对象创建Python对象。首先，如果当前节点的对象存储不包含这个对象，则下载它。然后，如果对象是[numpy array]()或者numpy array的集合，`get`调用是零拷贝的，返回由共享对象存储内存支持的数组。如果存在，我们把对象数据反序列化成Python对象。
```python
y = 1
obj_id = ray.put(y)
assert ray.get(obj_id) == 1
```

You can also set a timeout to return early from a get that’s blocking for too long.

你还可以设置超时，以便从阻塞时间过长的get调用中提早返回。
```python
from ray.exceptions import RayTimeoutError

@ray.remote
def long_running_function()
    time.sleep(8)

obj_id = long_running_function.remote()
try:
    ray.get(obj_id, timeout=4)
except RayTimeoutError:
    print("`get` timed out.")
```

```doc
ray.get(object_ids, timeout=None)[source]
    Get a remote object or a list of remote objects from the object store.

    This method blocks until the object corresponding to the object ID is available in the local object store. If this object is not in the local object store, it will be shipped from an object store that has it (once the object has been created). If object_ids is a list, then the objects corresponding to each object in the list will be returned.

    This method will error will error if it’s running inside async context, you can use await object_id instead of ray.get(object_id). For a list of object ids, you can use await asyncio.gather(*object_ids).

    Parameters:
    object_ids – Object ID of the object to get or a list of object IDs to get.
    timeout (Optional[float]) – The maximum amount of time in seconds to wait before returning.
    Returns:
    A Python object or a list of Python objects.

    Raises:
    RayTimeoutError – A RayTimeoutError is raised if a timeout is set and the get takes longer than timeout to return.
    Exception – An exception is raised if the task that created the object or that created one of the objects raised an exception.
```

启动一些task后，你也许想知道哪些task已经完成执行。这可以通过`ray.wait`做到。函数使用方法如下：
```python
ready_ids, remaining_ids = ray.wait(object_ids, num_returns=1, timeout=None)
```

```doc
ray.wait(object_ids, num_returns=1, timeout=None)[source]
    Return a list of IDs that are ready and a list of IDs that are not.

    If timeout is set, the function returns either when the requested number of IDs are ready or when the timeout is reached, whichever occurs first. If it is not set, the function simply waits until that number of objects is ready and returns that exact number of object IDs.

    This method returns two lists. The first list consists of object IDs that correspond to objects that are available in the object store. The second list corresponds to the rest of the object IDs (which may or may not be ready).

    Ordering of the input list of object IDs is preserved. That is, if A precedes B in the input list, and both are in the ready list, then A will precede B in the ready list. This also holds true if A and B are both in the remaining list.

    This method will error if it’s running inside an async context. Instead of ray.wait(object_ids), you can use await asyncio.wait(object_ids).

    Parameters:
    object_ids (List[ObjectID]) – List of object IDs for objects that may or may not be ready. Note that these IDs must be unique.
    num_returns (int) – The number of object IDs that should be returned.
    timeout (float) – The maximum amount of time in seconds to wait before returning.
    Returns:
    A list of object IDs that are ready and a list of the remaining object IDs.
```

## 对象回收
当对象存储装满时，会进行对象回收为新对象腾出空间。这根据LRU算法(最近最少使用)顺序回收。为了避免对象被回收，你可以调用`ray.get`然后重新存储值。当numpy array对象映射到任意python进程时，不会被回收。你还可以配置[内存限制](https://ray.readthedocs.io/en/latest/memory-management.html)控制对象存储的使用。

?>Note:Objects created with ray.put are pinned in memory while a Python reference to the object ID returned by the put exists. This only applies to the specific ID returned by put, not IDs in general or copies of that IDs.
**上面这个不太好理解**

?>注意：用过`ray.put`创建的对象会被固定在内存中，并且对put返回的Object ID对应的Python引用存在。**这只对put返回的ID适用**，不适用通常的ID或该ID的副本。

## 远程类(Actors)
Actors把Ray API从函数(task)扩展到类。下面的代码中，`ray.remote`装饰器表明`Counter`类的实例是actor的。一个actor本质上是一个有状态的worker。每个actor运行在各自的python进程中：
```python
@ray.remote
class Counter(object):
    def __init__(self):
        self.value = 0

    def increment(self):
        self.value += 1
        return self.value
```

我们可以通过下面的代码创建一对actor：
```python
a1 = Counter.remote()
a2 = Counter.remote()
```

**当一个actor实例化时，会发生以下事件**：
1. 一个python worker进程在集群的节点上启动。
2. 一个`Counter`类在worker进程中实例化。

你也可以在Actor中指定资源需求(更多细节查看[Actor](https://ray.readthedocs.io/en/latest/actors.html)部分)：
```python
@ray.remote(num_cpus=2, num_gpus=0.5)
class Actor(object):
    pass
```

我们可以通过调用actor的`.remote`操作acotr交互。然后对Object ID调用`ray.get`获得实际值：
```python
obj_id = a1.increment.remote()
ray.get(obj_id) == 1
```

不同actor上的方法调用可以并行执行，同一个actor的多个方法调用按调用顺序执行。同一个actor的方法会共享状态：
```python
# Create ten Counter actors.
counters = [Counter.remote() for _ in range(10)]

# Increment each Counter once and get the results. These tasks all happen in
# parallel.
results = ray.get([c.increment.remote() for c in counters])
print(results)  # prints [1, 1, 1, 1, 1, 1, 1, 1, 1, 1]

# Increment the first Counter five times. These tasks are executed serially
# and share state.
results = ray.get([counters[0].increment.remote() for _ in range(5)])
print(results)  # prints [2, 3, 4, 5, 6]
```