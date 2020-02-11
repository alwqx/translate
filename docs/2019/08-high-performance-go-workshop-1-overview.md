# 高性能Go研讨会（一）总览
[原文链接](https://dave.cheney.net/high-performance-go-workshop/dotgo-paris.html)

## 内容总览
本研讨会的目的是介绍一些在剖析Go应用中性能问题并修复它们的工具。

今天我们将从小处着手-学习如何编写基准测试，然后剖析一小段代码。接着讨论运行追踪器（execution tracer）、垃圾回收器（garbage collector）和追踪运行的应用。剩下的部分留给大家提问和实验代码。

### 许可和材料
这个研讨会由[David Cheney](https://twitter.com/davecheney)和[Francesc Campoy](https://twitter.com/francesc)合作完成。

本演示文稿受[Creative Commons Attribution-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-sa/4.0/)许可保护。

### 预备知识
你需要安装一些软件。

1. 研讨会代码仓库
从https://github.com/davecheney/high-performance-go-workshop下载本文所需代码示例。
2. 笔记本、电源线、[下载Go1.12](https://golang.org/dl/)
3. Graphviz，pprof部分需要安装graphviz工具集
    - Linux:`[sudo] apt-get install graphviz`
    - OSX
        - MacPorts:`sudo port install graphviz`
        - Homebrew:brew install graphviz
    - [Windows](https://graphviz.gitlab.io/download/#Windows):未测试
4. Google Chrome，运行追踪器部分需要Chrome，相关材料不兼容Safari, Edge, Firefox, or IE 4.01
5. 你自己的代码来剖析和优化

### One more thing
这并不是授课，而是交流。我们有很多休息时间来提问。

如果你有不理解的部分，或者认为你听的是不对的，欢迎提问。

## 1. 微处理器性能的过去、现在和未来
这个研讨会是关于如何写高性能代码的。其它研讨会上我讨论过解耦设计和可维护性，但我们今天讨论性能。

我将从一个简短的讲稿开始，介绍我眼里的计算机发展史以及**为什么我认为写高性能软件很重要**。

事实上软件运行在硬件上，所以在讨论写高性能代码时，我们先要讨论运行代码的硬件。

### 1.1 Mechanical Sympathy
![](https://dave.cheney.net/high-performance-go-workshop/images/image-20180818145606919.png)

目前你会听到一个流行的术语，你会听到像Martin Thompson或Bill Kennedy等讨论"Mechanical Sympathy"。

"Mechanical Sympathy"一词来自于伟大的赛车手Jackie Stewart，3次获得世界一级方程式赛车冠军。他认为最好的赛车手要对汽车如何工作有足够的理解以便他们可以协调工作。

要成为伟大的赛车手，并不需要成为伟大的机修工。你需要足够而不是草率地理解汽车如何工作。

我相信这个道理对于软件工程师是一样的。我认为这个房间的任何人都不会是专业的CPU设计师，但这并不意味着我们要忽视CPU设计者面对的问题。

### 1.2 六个数量级
有一个常见的互联网模式，就像这样：
![](https://dave.cheney.net/high-performance-go-workshop/images/jalopnik.png)

当然这是荒谬的，但它强调了计算行业的变化。

**作为软件作者这间屋子的所有人都从摩尔定律收益，40年来每18个月芯片上可用晶体管数量增加一倍**。没有其它行业在一生的空间中经历了[六个数量级](https://dave.cheney.net/high-performance-go-workshop/dotgo-paris.html#_footnotedef_1)的工具改进。

但这一切都在改变

### 1.3 计算机依然在变得更快吗？
因此基本问题是，面对上图中的统计数据，我们应该问的是“计算机是否仍然变得更快？”

如果计算机依然变得更快，那么或许我们不需要关心代码的性能问题了，我们只需要等一会，硬件厂商会帮我们解决性能问题。

#### 1.3.1 我们先看看数据
下图是一张经典的图，你可以从类似*Computer Architecture, A Quantitative Approach by John L. Hennessy and David A. Patterson*的书中找到，这张图来自第五版。

![](https://community.cadence.com/cfs-file/__key/communityserver-blogs-components-weblogfiles/00-00-00-01-06/2313.processorperf.jpg)

第五版中Hennessey和Patterson讨论了三个时代的计算性能问题：
1. 第一个是1970年代和80年代初，这是形成时期。我们今天所知的微处理器那时并不存在，计算机是用离散的晶体管或小规模集成电路构建的。成本、规模和对材料科学理解上的限制是限制因素。
2. 从80年代中期到2004年，趋势线很明显。计算机整数性能平均每年提高52％。计算机功率每两年翻一番，因此人们将摩尔定律与计算机性能合并，即模具上晶体管数量翻倍、计算机性能翻倍。
3. 然后我们来到计算机性能的第三个时代。事情变慢了，总变化率为每年22％。

上图只上升到2012年，幸运的是2012年[Jeff Preshing](http://preshing.com/20120208/a-look-back-at-single-threaded-cpu-performance/)编写了自己的工具，[从Spec网站抓取数据构建自己的图表](https://github.com/preshing/analyze-spec-benchmarks)。

![](https://dave.cheney.net/high-performance-go-workshop/images/int_graph.png)

这是使用Spec1995-2017年数据生成的图。

对我来说，是说单核心性能接近极限，而不是我们在2012年的数据中看到的变化。对于浮点数而言，这些数字略好一些，但对于会议室中进行业务应用开发的我们而言，这可能并不相关。

### 1.3.2 Yes, computer are still getting faster, slowly
>The first thing to remember about the ending of Moore’s law is something Gordon Moore told me. He said "All exponentials come to an end". — [John Hennessy](https://www.youtube.com/watch?v=Azt8Nc-mtKM)

这句话引用自Hennessy在Google Next 18和他的图灵奖演讲。他的论点是肯定的：CPU性能仍在提高。但是，单线程整数性能每年提高2-3％左右。按此速度，它将需要20年的复合增长才能达到整数性能提高。相比之下，90年代的表现每两年增加一倍。

为什么会这样？

### 1.4 时钟速度
![](https://dave.cheney.net/high-performance-go-workshop/images/stuttering.png)

这张2015年的图可以很好地表达这点。顶行显示了芯片上的晶体管数量。自1970年代以来，这一趋势大致线性增长。由于这是log/lin图，因此该线性系列代表指数增长。

但是如果我们看中间这条线，我们发现时钟速度10年内没有增长，我们看到CPU速度2004年停滞不前。

底部那条线表示散热功率，即电能变成热量，遵循相同的模式：时钟速度和CPU散热是相关的。

### 1.5 Heat
为什么CPU会产生热？它是一个固态设备，没有移动组件，所以摩擦等效果在这里并没有（直接）相关。

下图来自[TI生成的优秀数据表](http://www.ti.com/lit/an/scaa035b/scaa035b.pdf)，在该模型中，N型器件开关被吸引到正电压，P型器件被正电压排斥。

![](https://dave.cheney.net/high-performance-go-workshop/images/cmos-inverter.png)

CMOS设备的电源消耗，来自于这个房间中、你们桌子上、口袋中的每个晶体管

CMOS器件的功耗，就是这个房间里的每个晶体管，桌面和口袋里的三个因素的组合。
1. 静态功耗。当晶体管静止时，即不改变其状态，有少量电流通过晶体管泄漏到地面。晶体管越小，泄漏越多。泄漏随温度升高而增加。当你拥有数十亿个晶体管时，即使是少量的泄漏也会增加！
2. 动态功耗。当一个晶体管从一个状态转换到另一个状态时，它必须对链接的栅极充电或者放电，每个晶体管的动态功率是电容的平方乘以电容和变化的频率，降低电压可以降低晶体管消耗的功率，但会导致晶体管切换较慢。
3. Crowbar, or short circuit current。我们喜欢将晶体管当作数字设备占据一个或另一个状态，原子地关闭或打开。实际上，晶体管是模拟器件。作为开关，晶体管大部分从关闭状态开始切换到开启的状态。这种转换或切换时间非常快，在现代处理器中它的速度为皮秒，但仍然代表从Vcc到地的低电阻路径的一段时间。晶体管开关越快，其频率越高，散热量就越大。

### 1.6 Dennard缩放定律终结
为了理解接下来的内容我们需要阅读一篇写于1974年由[Robert H. Dennard](https://en.wikipedia.org/wiki/Robert_H._Dennard)合作的论文。Dennard缩放定律大致指出随着晶体管越来越小，它们的[功率密度](https://en.wikipedia.org/wiki/Power_density)保持不变。较小的晶体管可以在较低的电压下运行，具有较低的栅极电容，并且开关速度更快，这有助于减少动态功率。

这里的原理是什么呢？

![](http://semiengineering.com/wp-content/uploads/2014/04/Screen-Shot-2014-04-14-at-8.49.48-AM.png)

结果并不那么好。随着晶体管的栅极长度接近几个硅原子的宽度，晶体管尺寸、电压和泄漏之间的关系破坏。

[1999年Micro-32会议](https://pdfs.semanticscholar.org/6a82/1a3329a60def23235c75b152055c36d40437.pdf)上假设，如果我们遵循时钟速度增加和晶体管尺寸缩小的趋势线，那么在处理器制作中，晶体管连接处将接近核反应堆核心的温度。显然这是不可能的。奔腾4标志着单核、高频、消费级CPU的终结。

回到这个图表，我们看到时钟速度停滞的原因是因为CPU温度超出了我们冷却它的能力。**到2006年，减小晶体管的尺寸不再改善其功率效率**。

我们现在知道降低CPU特征尺寸主要是为了降低功耗。降低能耗并不仅仅意味着“绿色”，就像回收利用不仅拯救地球一样。主要目标是将功耗和热耗散保持在[低于损坏CPU的水平](https://en.wikipedia.org/wiki/Electromigration#Practical_implications_of_electromigration)。

![](https://dave.cheney.net/high-performance-go-workshop/images/stuttering.png)

但是，图表的一部分仍在增长，即芯片上的晶体管数量。随着CPU特征尺寸不断发展，相同的区域中放置更多的晶体管，是一把双刃剑。

此外，正如您在插页中看到的那样，每个晶体管的成本持续下降，直到大约5年前，然后每个晶体管的成本开始回升。

![](https://whatsthebigdata.files.wordpress.com/2016/08/moores-law.png)

创建更小的晶体管不仅成本越来越高，而且越来越难。2016年的这份报告显示了2013年芯片制造商对将来的预测; 两年后，他们没能实现所有的预测，虽然我没有这份报告的更新版本，但没有迹象表明他们能够扭转这种趋势。

英特尔、台积电、AMD和三星花费数十亿美元，因为他们必须建立新的晶圆厂，购买所有新的工艺工具。因此，虽然每个芯片的晶体管数量持续增加，但其单位成本已开始增加。

?>甚至术`栅极长度（以纳米为单位）`也变得模棱两可。各种制造商以不同的方式测量晶体管的尺寸，使其能够展示比竞争对手更小的数量，而无需提供基准。这是CPU制造商的非GAAP收益报告模型。

## 译者说

## 翻译学习
1. `I don’t think any of us in this room will be a professional CPU designer`。don't和any是双重否定，在翻译的时候可以改成肯定更顺些：我认为这个房间的任何人都不会是专业的CPU设计师。
2. preposterous
3. underscores