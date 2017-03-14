---
title: the Abstract of sklearn.ensemble
date: 2017-03-12 12:40:00
categories: machine learning
tags: 
  - machine learning
  - sklearn
  - ensemble
---

# sklearn.ensemble.AdaBoostClassifier/sklearn.ensemble.AdaBoostRegressor

Adaboost分类器是一个超估计。它通过在原始数据集上拟合一个分类器开始，然后在相同的数据集上调整误分类样本的权重，并用其再次拟合相同分类器，因此接下来的分类器会更关注于难以正确分类的样本。

# sklearn.ensemble.BaggingClassifier/sklearn.ensemble.BaggingRegressor

一个bagging分类器。

bagging分类器是一种集成meta-估计器。
它用原始数据集的随机子集去拟合基分类器，并聚合各自的预测结果（投票或者平均）去形成一个最终结果。
这样一个meta-估计器典型的被用于**降低**黑箱估计器（例如决策树）的**方差**，它通过在结构化过程中引入随机性并对结果进行集成来实现。

这个算法包括了过往的一些研究。当数据集中的随机子集被抽取出来作为样本的随机子集，这个算法就是Pasting。
如果样本的抽取是有放回的，这个方法就是Bagging。当特征被随机抽取，这个方法就是Random Subspaces。
最后，如果基分类器是建立在随机的样本和特征之上，这个方法被叫做Random Patches。

# sklearn.ensemble.ExtraTreesClassifier/sklearn.ensemble.ExtraTreesRegressor

这个算法通过拟合许多基于各种各样数据集子样本的随机决策树并平均结果来改善预测精度和控制过拟合。

该方法只对样本而不对特征进行抽样。但是，它通过max_features这样一个参数去控制每次分裂。
每次分类，只随机选择max_features个特征，从中选出最优分裂。但如果随机出的特征都不支持有效分裂，
算法会从剩下的特征中继续搜索出一个合宜的分裂。分裂的终止有其他相关树特征决定。
在sklearn中，**ExtraTreesClassifier可以设置class_weight参数**，这非常好！

我查阅资料并结合参数，猜测sklearn中ET树原理： 
对于样本并不进行随机抽样（并没有max_samples参数），或者说用的是全样本。  
每次分类时，随机选择max_features特征。对于这些特征，随机选择分割点。（正常二叉树情况下，需要计算特征各种可能分裂方式并寻找出最优。但这里仅仅是随机选择出特征分裂方式）
然后比较各特征分裂的信息增益，选择最优者进行树的生长。因此在这种情况下所有样本也都是oob（out-of-bag）样本。  
bootstrap参数只能理解为是针对特征了。

# sklearn.ensemble.GradientBoostingClassifier/sklearn.ensemble.GradientBoostingRegressor

GB建立了一个前向加法模型，它允许优化任意不同的损失函数。  
对于分类情况，在每一步，n_classes_（类数）个回归树被建立去拟合logloss。二分类是一个特殊情况，只使用一颗回归树。  
对于回归情况，使用一棵回归树去拟合任意损失函数。

# sklearn.ensemble.IsolationForest

使用IsolationForest算法对每个样本回归一个异常得分。

IsolationForest通过随机选择一个特征并在该特征最大最小值之间随机选择一个分裂值去孤立观察值。

由于树结构可以表示递归分裂，因此孤立一个样本所需要的分裂次数等价于从根节点到终端节点的路径长度。

路径长度，通过对森林中随机树长度进行平均得到，作为一种正常性度量。

随机分裂过程对于异常值有显著更短的路径。因此，当随机树组成的森林对于特殊样本产生了更短路径，它们更可能是异常。

# sklearn.ensemble.RandomForestClassifier/sklearn.ensemble.RandomForestRegressor

这个算法通过拟合许多基于各种各样数据集子样本的随机决策树并平均结果来改善预测精度和控制过拟合。  
子样本大小总是和原始输入样本尺寸相同，但是当bootstrap=True时样本有放回抽取。

# sklearn.ensemble.RandomForestEmbedding

一种无监督变换，将数据集变换到高维稀疏空间。数据点根据每颗树叶子进行编码。使用one-hot编码方式，这将导致二元编码。

# sklearn.ensemble.VotingClassifier

用于对未集成的估计器列表进行投票集成。
