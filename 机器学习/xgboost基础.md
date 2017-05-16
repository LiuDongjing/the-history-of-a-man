# XGBoost基础
## 简介
[XGBoost][0](e**X**treme **G**radient **Boost**ing)是一个分布式
的gradient boosting库，当初的设计目标就是**高效**、**灵活**和**可移植**。
该库使用并行的tree boosting(也即GBDT和GBM)来快速、精确地解决许多数据科学
方面的问题。同样的代码可无需改变就能在主要的分布式环境(Hadoop、SGE和MPI)
下运行，并且可处理数十亿级别的数据量。

## 整体流程与特点
- 是一种迭代式的算法
- 每次迭代都会添加一棵CART，使得目标函数相比之前更优
- 在求解目标函数的时候，并没有给出解析解，而是用泰勒二次展开近似目标函数，简化了后续的计算
- 整体分为两个部分，求解目标函数和构建CART
- 整个算法并没有涉及多少高深的数学知识，但给出的解决方法确实非常巧妙和优雅

## CART(Classfication And Regression Tree)
使用CART做预测的基本形式如下

![\hat{y}_i = \phi(x_i) = \sum_{k=1}^Kf_k(x_i), \qquad f_k \in \mathcal{F}][1]

其中

![\mathcal{F} = \{f(x) = w_{q(x)}(q:\qquad \mathbb{R}^m \rightarrow T,\qquad w \in \mathbb{R}^T)\}][2]

这里x_i是一个m维的样本，f(x)将它映射成一个实数，具体映射方法是使用决策树将x_i映射成w(实数向量，是决策树的叶子节点)，
然后再使用q(x)函数选择w中的某一维(其中一个叶子节点)作为f(x)的预测结果

----
[0]: https://github.com/dmlc/xgboost
[1]: https://chart.googleapis.com/chart?cht=tx&chl=%5Chat%7By%7D_i%20%3D%20%5Cphi(x_i)%20%3D%20%5Csum_%7Bk%3D1%7D%5EKf_k(x_i)%2C%20%5Cqquad%20f_k%20%5Cin%20%5Cmathcal%7BF%7D
[2]: https://chart.googleapis.com/chart?cht=tx&chl=%5Cmathcal%7BF%7D%20%3D%20%5C%7Bf(x)%20%3D%20w_%7Bq(x)%7D(q%3A%5Cqquad%20%5Cmathbb%7BR%7D%5Em%20%5Crightarrow%20T%2C%5Cqquad%20w%20%5Cin%20%5Cmathbb%7BR%7D%5ET)%5C%7D
