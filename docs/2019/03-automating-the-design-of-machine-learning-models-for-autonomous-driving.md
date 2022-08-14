# AutoML：自动设计自动驾驶机器学习模型

## 说明
- [原文链接](https://medium.com/waymo/automl-automating-the-design-of-machine-learning-models-for-autonomous-driving-141a5583ec2a)
- [翻译：@alwqx](https://github.com/alwqx)
- [项目地址](https://github.com/alwqx/translate)
- [tt](https://github.com/alwqx/tt)：自动生成翻译模板
- 用时: 2.5h(人机混合)
- 2019翻译任务：3/52
- <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/80x15.png" /></a>

## 译者说
本人介绍了谷歌自动驾驶子公司Waymo在AutoML领域的研究成果。自动驾驶对神经网络模型的准确性和延迟要求，这要求工程师手动调优不同的神经网络架构，这不仅花费了大量的时间，而且能够调优的数量是有限的。因此，Waymo和Google AI的研究员合作，自动生成神经网络架构并训练评估，即AutoML，以便节约时间并寻找最佳模型。本文介绍了他们的初步研究成果，从取得的进展来看，未来可期。

## 正文
By: Shuyang Cheng and Gabriel Bender

在Waymo，机器学习几乎在我们自动驾驶系统的每个模块都起着关键作用。它可以帮助我们的汽车看清周围的环境、感知世界、预测其他人的行为，并决定自己下一步最佳移动。

采取感知：我们的系统组合多个神经网络，使车辆能够解释传感器数据以识别物体，并随着时间的推移跟踪它们，以便车辆能够深入理解周围的世界。创建这些神经网络通常是一项耗时的任务; 我们要优化神经网络架构，以使网络的质量和速度满足在自动驾驶汽车上运行，这是一个复杂的微调过程，一项新任务的调优通常花费工程师数月时间。

现在，我们与Brain团队的Google AI研究员合作，将前沿研究付诸实践来自动生成神经网络。更重要的是，这些自动生成的神经网络比工程师手动调优的网络具有更高的质量和更快的速度。

为了把我们的自动驾驶技术带到不同的城市和环境，我们需要以极快的速度优化我们的模型以适应不同的场景。AutoML让我们做到这一点，它提供了大量的有效且持续的ML解决方案。

## 迁移学习：使用现有的AutoML架构
我们的合作始于一个简单的问题：**AutoML能否为汽车生成高质量和低延迟的神经网络**？

质量衡量神经网络的准确性。延迟衡量神经网络的速度，它也称为推理时间。由于驾驶是一项需要车辆实时反馈、系统足够安全的关键活动，因此我们的神经网络需要低延迟运行。我们大部分直接运行在车辆上的神经网络提供结果的延迟小于10ms，这比在数据中心数千台服务器上运行的许多神经网络快。

在他们的[初版AutoML论文](https://arxiv.org/abs/1611.01578)中，我们的Google AI同事能够自动探索超过12000种架构来解决经典的CIFAR-10图像识别任务：将小图像识别为十个类别中的一个，例如汽车、飞机、狗等。在一篇[后续文章](https://arxiv.org/pdf/1707.07012.pdf)中，他们发现了一组神经网络构建模块，称之为NAS单元，对于CIFAR-10和类似的任务，NAS单元自动构建的神经网路比手工调优的要好。通过这次合作，我们的研究人员决定使用这些单元自动构建针对自动驾驶任务的新模型，从而将在CIFAR-10上学到的知识迁移到自动驾驶领域。我们的第一个实验是语义分割任务：将LiDAR点云中的每个点标识为汽车、行人、树等。

![](https://cdn-images-1.medium.com/max/960/1*CT9kBawjAO2oRN2OGc7MuQ.png)
>One example of a NAS cell. This cell processes inputs from the two previous layers in a neural net.

为此，我们的研究员编写了一个自动搜索算法，在卷积网络架构（CNN）中探索数百种不同的NAS单元组合，训练和评估我们的LiDAR分割任务模型。当我们的工程师手工微调这些神经网络时，他们只能探索有限数量的架构，但通过这种方法，我们自动探索了数百个架构。我们发现新的模型在以下两方面优于以前手工调优的模型：
- 质量相似，但延迟显着降低。
- 延迟相似，但质量更高。

鉴于初步尝试取得的成功，我们将相同的搜索算法应用于两个与交通车道的检测和定位相关的附加任务。迁移学习技术也适用于这些任务，并且我们能够在汽车上部署三个新训练和改进的神经网络。

## 端到端搜索：从头开始的新搜索架构
我们受到这些初步结果的鼓舞，因此决定寻找可以提供更好结果和更广泛应用的全新架构。通过无限制组合已发现的NAS单元，我们可以更直接地寻找满足严格的延迟要求的架构。

进行端到端搜索通常需要手动探索数千种架构，会带来大量的计算成本。探索单一架构需要在具有多个GPU的数据中心计算机上进行数天的训练，这意味着单个任务找到理想的架构需要数千天的计算。因此我们设计了一个代理任务：缩小的LiDAR分割任务，可以在几小时内完成训练。

团队必须克服的一个挑战是找到一个类似于我们原始分割任务的代理任务。在我们能够确定新任务架构的质量与原始任务中架构的质量之间的关联之前，我们尝试设计了几个代理任务。然后，我们启动了类似于[初版AutoML论文](https://arxiv.org/abs/1611.01578)中的搜索算法，但现在用在代理任务上进行搜索：代理端到端搜索。这是该概念首次应用于LiDAR数据。

![](https://cdn-images-1.medium.com/max/960/1*JCPSzb1GEvUJkgrRLXqXfw.png)
>Proxy end-to-end search: Explore thousands of architecture on a scaled-down proxy task, apply the 100 best ones to the original task, validate and deploy the best of the best architectures on the car.

我们使用了几种搜索算法来优化质量和延迟，因为这对车辆至关重要。我们查看不同类型的CNN架构并使用不同的搜索策略，例如随机搜索和强化学习，我们为代理任务探索10000多种不同的架构。通过使用代理任务，原来在Google TPU集群上需要一年多计算时间的任务现在只需要两周时间。我们刚开始迁移NAS单元时就发现了比以前更好的神经网络：
- 延迟降低20-30％，质量相同。
- 质量更高，错误率降低8-10％，与之前的架构具有相同的延迟。

![](https://cdn-images-1.medium.com/max/720/1*pzaDWldooweo5ToaWnxILQ.png) ![](https://cdn-images-1.medium.com/max/720/1*yPcHE6Ib3lKBQBEQQxtg4Q.png)

>1) The first graph shows about 4,000 architectures discovered with a random search on a simple set of architectures. Each point is an architecture that was trained and evaluated. The solid line marks the best architectures at different inference time constraints. The red dot shows the latency and performance of the net built with transfer learning. In this random search, the nets were not as good as the one from transfer learning.
>2) In the second graph, the yellow and blue points show the results of two other search algorithms. The yellow one was a random search on a refined set of architectures. The blue one used reinforcement learning as in [1] and explored more than 6,000 architectures. It yielded the best results. These two additional searches found nets that were significantly better than the net from transfer learning.

搜索中发现的一些网络架构显示了卷积、池化和反卷积操作的创造性组合，如下图所示。这些架构最终适用于我们最初的LiDAR分割任务，并将部署在Waymo的自动驾驶车辆上。

![](https://cdn-images-1.medium.com/max/960/1*kXgRb7KIpuotmg1YZQu3BQ.png)
>One of the neural net architectures discovered by the proxy end-to-end search.

## 下一步呢？
我们的AutoML实验只是一个开始。对于我们的LiDAR分割任务，迁移学习和代理端到端搜索都提供了比手工调优更好的神经网络。我们现在有机会将这些机制应用于新类型的任务，这可以改善许多神经网络。

这一发展为我们未来的ML工作开辟了新的令人兴奋的途径，并将改善我们自动驾驶技术的性能和能力。我们期待与Google AI进一步工作，敬请期待！

## 致谢
Waymo和Google之间的合作由Waymo的Matthieu Devin和Google的Quoc Le发起和赞助。这项工作由Waymo的Shuyang Cheng和Google的Gabriel Bender以及Pieter-jan Kindermans执行。特别感谢Vishy Tirumalashetty的支持。

![](https://cdn-images-1.medium.com/max/960/1*kVljrdIhXvdoVRXr_n3l-g.jpeg)
>Members of the Waymo and Google teams (from left): Gabriel Bender, Shuyang Cheng, Matthieu Devin, and Quoc Le

## 参考文献
-  Barret Zoph and Quoc V. Le. Neural architecture search with reinforcement learning. ICLR, 2017.
- Barret Zoph, Vijay Vasudevan, Jonathon Shlens, Quoc V. Le, Learning Transferable Architectures for Scalable Image Recognition. CVPR, 2018.