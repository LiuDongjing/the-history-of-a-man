# XGBoost基础

## 简介

[XGBoost](https://github.com/dmlc/xgboost)\(e**X**treme **G**radient **Boost**ing\)是一个分布式  
的gradient boosting库，当初的设计目标就是**高效**、**灵活**和**可移植**。  
该库使用并行的tree boosting\(也即GBDT和GBM\)来快速、精确地解决许多数据科学  
方面的问题。同样的代码可无需改变就能在主要的分布式环境\(Hadoop、SGE和MPI\)  
下运行，并且可处理数十亿级别的数据量。

## 整体流程与特点

* 是一种迭代式的算法
* 每次迭代都会添加一棵CART，使得目标函数相比之前更优
* 在求解目标函数的时候，并没有给出解析解，而是用泰勒二次展开近似目标函数，简化了后续的计算
* 整体分为两个部分，求解目标函数和构建CART
* 整个算法并没有涉及多少高深的数学知识，但给出的解决方法确实非常巧妙和优雅

## CART\(Classfication And Regression Tree\)

使用CART做预测的基本形式如下

$$\hat{y}_i = \phi(x_i) = \sum_{k=1}^Kf_k(x_i), \quad f_k \in \mathcal{F}$$

其中

$$\mathcal{F} = \{f(x) = w_{q(x)}(q: \mathbb{R}^m \rightarrow T, w \in \mathbb{R}^T)\}$$

这里$$x_i$$是一个m维的样本，f\(x\)将它映射成一个实数，具体映射方法是使用决策树将$$x_i$$映射成w\(实数向量，是决策树的叶子节点\)，然后再选择w中的某一维\(其中一个叶子节点\)作为f\(x\)的预测结果。q\(x\)函数就代表了决策树的结构。最终的预测结果是K个这样的决策树越策结果的累加和。

**注意 **

---



