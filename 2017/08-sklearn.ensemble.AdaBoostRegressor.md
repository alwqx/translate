# Adaboost回归器

<!-- TOC -->

- [Adaboost回归器](#adaboost回归器)
    - [说明](#说明)
    - [参数](#参数)
    - [属性](#属性)

<!-- /TOC -->

## 说明
- [原文链接](http://scikit-learn.org/stable/modules/generated/sklearn.ensemble.AdaBoostRegressor.html#sklearn.ensemble.AdaBoostRegressor)
- [翻译：@muzhenxu](https://github.com/muzhenxu)
- [项目地址](https://github.com/muzhenxu/translate)
- <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/80x15.png" /></a>

Adaboost回归器是一个超估计。它通过在原始数据集上拟合一个回归器开始，然后在相同的数据集上根据当前的预测误差调整误分类样本的权重，并用其再次拟合相同回归器，因此接下来的回归器会更关注于难以正确分类的样本。

这个类使用了被称为Adaboost的算法。

## 参数

**基估计器：可选（默认为决策树回归器）**
    
    它是boosting（提升）集成算法的基估计器。它需要支持带权重样本。

**估计器数量：整型，可选（默认50）**
    
    使得boosting终止的估计器最大数量。为了完美拟合，学习过程是stopping-early（达到合适条件就停止继续学习）的。

**学习率：浮点型，可选（默认1.0）**
    
    通过学习率去抑制每个分类器的贡献。再学习率和估计器数量之间有一个权衡。

**损失：{‘线性’，‘平方’，‘指数’}，可选（默认为‘线性’）**
    
    当每次迭代之后更新权重时，损失函数被使用。

**随机状态：整型，随机状态实例或者None，可选（默认为None）**
    
    如果是int，它就是用于随机数生成器的随机种子;如果是随机状态实例，它是一个随机数生成器;如果是None，随机数生成器通过np.random使用随机状态实例。

## 属性

**估计器：估计器列表**
    
    拟合的子估计器集合。

**估计器权重：浮点数组**
    
    集成器中每个估计器的权重。

**估计误差:浮点数组**
    
    集成器中每个估计器的回归误差。

**特征重要性：形状为[n_features]的数组**
    
    如果基估计器支持，则输出特征重要性
