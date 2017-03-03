# 一个Adaboost分类器。

<!-- TOC -->

- [一个Adaboost分类器。](#一个adaboost分类器)
    - [说明](#说明)
    - [参数](#参数)
    - [属性](#属性)

<!-- /TOC -->

## 说明
- [原文链接](http://scikit-learn.org/stable/modules/generated/sklearn.ensemble.AdaBoostClassifier.html#sklearn.ensemble.AdaBoostClassifier)
- [翻译：@muzhenxu](https://github.com/muzhenxu)
- [项目地址](https://github.com/muzhenxu/translate)
- <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/80x15.png" /></a>

Adaboost分类器是一个超估计。它通过在原始数据集上拟合一个分类器开始，然后在相同的数据集上调整误分类样本的权重，并用其再次拟合相同分类器，因此接下来的分类器会更关注于难以正确分类的样本。

这个类使用了被称为Adaboost-SAMME的算法。

## 参数

基分类器：可选（默认为决策树分类器）
    它是boosting（提升）集成算法的基分类器。它需要支持带权重样本，合适的class_和n_classes_属性。

分类器数量：整型，可选（默认50）
    使得boosting终止的分类器最大数量。为了完美拟合，学习过程是stopping-early（达到合适条件就停止继续学习）的。

学习率：浮点型，可选（默认1.0）
    通过学习率去抑制每个分类器的贡献。再学习率和分类器数量之间有一个权衡。

算法：{‘SAMME’，‘SAMME.R’}，可选（默认为‘SAMME.R’）
    如果是‘SAMME.R’则使用‘SAMME.R’boosting算法。基分类器必须支持类概率的计算。如果是‘SAMME’则使用‘SAMME’boosting算法。‘SAMME.R’一般比‘SAMME’更快收敛，并能用更少的迭代次数获得更低的测试误差。

随机状态：整型，随机状态实例或者None，可选（默认为None）
    如果是int，它就是用于随机数生成器的随机种子;如果是随机状态实例，它是一个随机数生成器;如果是None，随机数生成器通过np.random使用随机状态实例。

## 属性

分类器：分类器列表
    拟合的子分类器集合。

classes_：形状为[n_classes]的数组 
    类标签

n_classes_:整型
    类数量

分类器权重：浮点数组
    boosting中每个分类器的权重

分类器误差：浮点数组
    boosting中每个分类器的分类误差

特征重要性：形状为[n_features]的数组
    如果基分类器支持，则输出特征重要性
