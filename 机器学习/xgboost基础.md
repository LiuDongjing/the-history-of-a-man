# XGBoost基础

## 简介

[XGBoost](https://github.com/dmlc/xgboost)(e**X**treme **G**radient **Boost**ing)是一个分布式的gradient boosting库，当初的设计目标就是**高效**、**灵活**和**可移植**。该库使用并行的tree boosting(也即GBDT和GBM)来快速、精确地解决许多数据科学方面的问题。同样的代码可无需改变就能在主要的分布式环境(Hadoop、SGE和MPI)下运行，并且可处理数十亿级别的数据量。

## 整体流程与特点

* 是一种迭代式的算法
* 每次迭代都会添加一棵CART，使得目标函数相比之前更优
* 在求解目标函数的时候，并没有给出解析解，而是用泰勒二次展开近似目标函数，简化了后续的计算
* 整体分为两个部分，求解目标函数和构建CART
* 整个算法并没有涉及多少高深的数学知识，但给出的解决方法确实非常巧妙和优雅

## CART(Classfication And Regression Tree)

使用CART做预测的基本形式如下

$$\hat{y}_i = \phi(x_i) = \sum\limits_{k=1}^Kf_k(x_i), \quad f_k \in \mathcal{F}$$

其中

$$\mathcal{F} = \{f(x) = w_{q(x)}(q: \mathbb{R}^m \rightarrow T, w \in \mathbb{R}^T)\}$$

这里$$x_i$$是一个m维的样本，f(x)将它映射成一个实数，具体映射方法是使用决策树将$$x_i$$映射成w(实数向量，是决策树的叶子节点)，然后再选择w中的某一维(其中一个叶子节点)作为f(x)的预测结果。q(x)函数就代表了决策树的结构。最终的预测结果是K个这样的决策树越策结果的累加和。

**注意** 这里还存在两个未解决的问题，每颗树的w是如何取值的？一棵树是如何将样本映射到叶子节点的？

## 定义目标函数

$$\mathcal{L}(\phi) = \sum\limits_i l(\hat{y_i}, y_i) + \sum\limits_k \Omega(f_k)$$  
其中  
$$\Omega(f) = \gamma T + \frac{1}{2} \lambda \|w\|^2$$

该目标函数分两部分，前一部分是通常意义上的损失函数，后一部分是正则化项，为了防止过拟合。该值越小越好。

l是有二阶导数的损失函数，并且二阶导数不能为0.正则化项的定义也很直观，控制叶子数量和w在一个合理的范围内。如果只追求l最小，那么模型就容易过拟合，对新数据没有预测能力。通常情况下，l变小会导致正则化项变大，两者取个平衡，模型才能有较好的泛化能力。

## 求解模型

上面已经把模型和目标函数都定好了，现在来求解这个模型使得目标函数最小。

### 基本思路

Gradient Tree Boosting在训练的时候是迭代进行的，每次只添加一棵树。只要添加的这棵树使目标函数的增量最小即可。那么就有下面的公式：

$$\mathcal{L}^{(t)} = \sum\limits_{i=1}^n l(y_i, \hat{y_i}^{(t-1)} + f_t(x_i)) + \Omega(f_t)$$

t-1次迭代的预测结果$$\hat{y_i}^{(t-1)}$$加上本次添加树的预测结果$$f_t(x_i)$$就是t次迭代的预测结果。注意这里的$$\mathcal{L^{(t)}}$$和上面定义的目标函数$$\mathcal{L}(\phi)$$有细微的差别。在添加正则化项的时候，没有考虑前t-1次的$$\Omega$$。因为对我们的目标来讲，这些值都是常量，不受$$f_t$$选择的影响，所以当选择$$f_t$$使得$$\mathcal{L}^{(t)}$$最小时，对应的$$\mathcal{L}(\phi)$$也是最小的。

### 近似

简化后的目标函数仍然不好求解。考虑到泰勒(Taylor)公式：

$$f(x + \Delta x) \simeq f(x) + f^{'}(x)\Delta x + \frac{1}{2}f^{''}(x)\Delta x^2$$

将$$\mathcal{L}^{(t)}$$重写成：

$$\mathcal{L}^{(t)} \simeq \sum\limits_{i=1}^n \left[ l(y_i, \hat y^{(t-1)}) + g_i f_t(x_i) + \frac{1}{2} h_i f_t^2(x_i)\right] + \Omega(f_t)$$

其中

$$g_i = \partial_{\hat y^{(t-1)}} l(y_i, \hat y^{(t-1)})\\
h_i = \partial^2_{\hat y^{(t-1)}} l(y_i, \hat y^{(t-1)})$$

目标函数是关于预测值$$\hat y$$的函数，对比泰勒公式，上面的近似变换还是很容易理解的。

移除常数项，可将目标函数进一步简化：

$$\tilde{\mathcal{L}}^{(t)} = \sum\limits_{i=1}^n \left[ g_i f_t(x_i) + \frac{1}{2} h_i f_t^2(x_i)\right] + \Omega(f_t)$$

### 求解

定义$$I_j = \left\{ i | q(x_i) = j\right\}$$，即所有映射到j号叶子节点的样本集合。引入$$I_j$$将上面的目标函数重写为如下形式：


$$
\begin{aligned}
\tilde{\mathcal{L}}^{(t)} &= \sum\limits_{i=1}^n \left[ g_i f_t(x_i) + \frac{1}{2} h_i f^2_t(x_i)\right] + \gamma T + \frac{1}{2}\lambda\sum\limits_{j=1}^T w^2_j\\
&= \sum\limits_{j=1}^T \left[ (\sum\limits_{i \in I_j} g_i)w_j +\frac{1}{2}(\sum\limits_{i \in I_j}h_i + \lambda)w_j^2\right] + \gamma T
\end{aligned}
$$


其实也就是转换了看问题的角度，开始是从样本的角度来看目标函数，现在转换成从叶子节点的角度来看。就像求二维矩阵所有元素的和一样，先求每一行的和再把结果相加与先求列的和再相加是一样的。

当q(x)固定，也就是树的结构固定，影响损失函数的只有叶子的权重w，此时损失函数是关于w的一元二次函数，有最优解：

$$w_j^* = -\frac{\sum_{i \in I_j} g_i}{\sum_{i \in I_j} h_i + \lambda}$$

对应损失函数的最优解如下：  
$$\tilde{\mathcal{L}}^{(t)}(q) = -\frac{1}{2} \sum\limits_{j=1}^T \frac{(\sum_{i \in I_j} g_i)^2}{\sum_{i \in I_j} h_i + \lambda} + \gamma T$$

也就是说给定一棵树，总能找到最优的w，那么问题就归结为寻找一棵树q使得$$\tilde{\mathcal{L}}^{(t)}(q)$$最小。至此，w的取值问题就解决了。

**注意** 每得到一棵树，树的结构和w值都需要保存下来。

## 寻找最优的树结构

### 基本思路

通常情况下枚举所有可能的树结构是不现实的，一般采用贪心算法。最开始只有一个叶子节点，所有的样本都落在这个节点里，然后迭代地向里面添加分支。设I为原叶子节点里的样本集合，$$I_L$$和$$I_R$$分别是对I拆分后的样本集合，即$$I = I_L \cup I_R$$，那么可得到拆分前后损失函数的变化量(拆分前减去拆分后)：

$$\mathcal{L}_{split} = \frac{1}{2} \left[ \frac{(\sum_{i\in I_L} g_i)^2}{\sum_{i \in I_L} h_i + \lambda} + \frac{(\sum_{i\in I_R} g_i)^2}{\sum_{i \in I_R} h_i + \lambda} - \frac{(\sum_{i\in I} g_i)^2}{\sum_{i \in I} h_i + \lambda}\right] - \gamma$$

这里就将损失函数和拆分操作联系起来了，当$$\mathcal{L}_{split}$$最大时，对该叶节点按选择的方式一分为二。

### 算法1.0(暴力枚举)

**输入** I，当前节点的所有样本集合；D，特征的维度数
  
$$G \leftarrow \sum_{i \in I}g_i, H \leftarrow \sum_{i \in I}h_i$$  

for k=1 to D do 

&nbsp;&nbsp;&nbsp;&nbsp;$$G_L \leftarrow 0, H_L \leftarrow 0$$ 
 
&nbsp;&nbsp;&nbsp;&nbsp;for j in sorted(I, by $$x_{k}$$) do
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$$G_L \leftarrow G_L + g_j, H_L \leftarrow H_L + h_j\\
G_R \leftarrow G - G_L, H_R \leftarrow H - H_L\\
score \leftarrow max(score, \frac{G_L^2}{H_L + \lambda} + \frac{G_R^2}{H_R + \lambda} - \frac{G^2}{H + \lambda})$$

&nbsp;&nbsp;&nbsp;&nbsp;end

end

**输出** 按照最大的score分割输出节点

**Note**: 第一层循环遍历所有特征，第二层循环在该特征下找最佳分割点。重点解释一下第二层循环，首先将样本集I按第k个特征$$x_k$$(论文中是$$x_{jk}$$，疑有误)排序，然后按照这个顺序依次计算是否是最佳分割点。当处理到第j个样本时，前j(包括j)个样本分在左边的节点，之后的样本分在右边的节点。将算法中的公式和上面的$$\mathcal{L}_{split}$$的公式对应起来，就好理解了。

### 算法2.0(近似分割)

for k = 1 to D do

&nbsp;&nbsp;&nbsp;&nbsp;针对特征k按照百分位数给出若干个分割点$$S_k = \left\{s_{k1}, s_{k2}, \cdots, s_{kl} \right\}
$$

&nbsp;&nbsp;&nbsp;&nbsp;这种预先计算分割点的方式可以是针对每棵树的(global)，也可以针对每个节点(local)。

end

for k = 1 to D do

&nbsp;&nbsp;&nbsp;&nbsp;$$G_{kv} \leftarrow = \sum\nolimits_{j \in \{j | s_{k,v} \ge x_{jk} \gt s_{k, v-1}\}} g_j\\
H_{kv} \leftarrow = \sum\nolimits_{j \in \{j | s_{k,v} \ge x_{jk} \gt s_{k, v-1}\}} h_j$$

end

后续操作和算法1.0一样，不过只在预计算的分割点处分割。

**Note**: 思路也是很简单的，在建树之前(global)之前会提前计算好分割点$$S_k$$，后面剖分节点时，不用一个一个尝试，只需要在$$S_k$$指定的地方分割就好。这样就简化了计算。local方式是指在分割某个节点前计算一个$$S_k$$，针对该节点下的样本集，而不是所有的样本集，这样比global方式又准确了一点，所以一般使用local方式。

### 算法3.0(Sparsity-aware Split)

**输入** I, 当前节点的所有样本；D，特征的维度  

在使用算法2.0的设置时，只将非空的样本收入buckets。
  
$$G \leftarrow \sum_{i \in I}g_i, H \leftarrow \sum_{i \in I}h_i$$  

for k = 1 to D do  

&nbsp;&nbsp;&nbsp;&nbsp;$$I_k = \left\{ i \in I | x_{ik} \ne missing\right\}$$ 
 
&nbsp;&nbsp;&nbsp;&nbsp;//将有数据缺失的样本归入右边的节点
  
&nbsp;&nbsp;&nbsp;&nbsp;$$G_L \leftarrow 0, H_L \leftarrow 0$$

&nbsp;&nbsp;&nbsp;&nbsp;for j in sorted($$I_k$$, **ascent** order by $$x_k$$) do

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$$G_L \leftarrow  G_L + g_i, H_L \leftarrow H_L + h_i\\
G_R \leftarrow G - G_L, H_R \leftarrow H - H_L\\
score \leftarrow max(score, \frac{G_L^2}{H_L + \lambda} + \frac{G^2_R}{H_R + \lambda} - \frac{G^2}{H - \lambda})$$

&nbsp;&nbsp;&nbsp;&nbsp;end

&nbsp;&nbsp;&nbsp;&nbsp;//将有数据缺失的样本归入左边的节点

&nbsp;&nbsp;&nbsp;&nbsp;$$G_R \leftarrow 0, H_R \leftarrow 0$$
 
&nbsp;&nbsp;&nbsp;&nbsp;for j in sorted($$I_k$$, **descent** order by $$x_k$$) do

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$$G_R \leftarrow  G_R + g_i, H_R \leftarrow H_R + h_i\\
G_L \leftarrow G - G_R, H_L \leftarrow H - H_R\\
score \leftarrow max(score, \frac{G_L^2}{H_L + \lambda} + \frac{G^2_R}{H_R + \lambda} - \frac{G^2}{H - \lambda})$$

&nbsp;&nbsp;&nbsp;&nbsp;end

end

**输出** score最大时对应的分割

**Note**: 算法3.0主要是为了解决在实际应用中经常出现的数据缺失的问题，为此xgboost提出了**default direction**的概念，也就是当在某一特征上分割样本而一些样本缺失该特征的数据时，则会被归入一个默认的子节点(见下图)。默认方向是在训练过程中得到的。算法的整体思路也很直观，针对每个特征，都会测试将不完整样本归入左边和右边这两种情况。注意这里的$$I_k$$是这对第k个特征来讲的，也就是说只要样本的第k个特征不为空，那么就会包含到$$I_k$$里，而不管其他特征是否为空。另外这个算法可以和算法2.0整合，只需要注意预先计算的分割点也是在$$I_k$$上进行的。

![default direction](images/default_direction.JPG)

## 其他细节

### Shrinkage和Column subsampling

这两种方法都是用来防止过拟合的。Shrinkage的意思就是每次新加一颗树时，先将它的w乘上$$\eta$$，然后再将它的预测结果累加到最终的结果上。这里的$$\eta$$就相当于学习速率。

Column subsampling也即是在每次只选取特征集合的一个子集来做分割，借鉴了随机森林里的做法。

### 处理离散特征

用一个向量表示类别特征，向量的长度就是类别的个数，使用one-hot编码将一个类别表示成一个向量。如果类别多的话，数据就比较稀疏了。然而xgboost也正好可以处理这种情况。

## 实现上的一些细节

### Column block

将数据按列组织(如下图)，训练之前会把每列按序排好。因为训练的时候只需要每个样本的g和h值，所以每一个元素会有指向g和h的指针。

![column block](images/column_block.JPG)

### cache-aware

获取g和h的时候，频繁使用间接索引，效率不高，在训练的时候，使用batch的方式一次获取后续的多个g和h值，分配一块缓冲区，计算累加结果存入缓冲区，仅接下来的若干次计算就可以直接从缓冲区中获得累加和。

## 调参

### 关键参数说明

参考[官方的API文档](http://xgboost.readthedocs.io/en/latest/python/python_api.html#module-xgboost.sklearn)，着重讲讲下面的几个参数。

* max\_depth，树的最大深度。太小了难以捕捉到数据中的模式，太大了容易过拟合。通常情况下是4-6.
* learning\_rate，也就是上面的$$\eta$$
* n\_estimators，拟合数据所用的树的个数，这个是如何确定的还没搞明白
* gamma，也就是上面的$$\mathcal{L}_{split}$$公式里的$$\gamma$$，只有当$$\mathcal{L}$$为正的时候才会在该节点处进一步分割
* subsample，按这个比例收取样本来训练决策树
* colsample\_bytree，也就是上面提到的column subsampling，按这个比例选取特征训练决策树
* colsample\_bylevel，和上面的不同之处是每次分割前都会按这个比率抽取特征。通常情况下可以不用。
* lambda, L2 regulation term on weights.用得也不多。
* reg\_alpha, L1 regulation term on weights.
* min\_child\_weight(int)，Minimun sum of instance weight(hessain) needed in a child。没搞懂为什么这样定义，不过资料上说可以防止过拟合，控制了每个叶子节点样本的数量。这个参数受总样本数量的影响。通常max\_depth越大，这个参数也越大。在线性回归模型中，就对应着样本的数量。
* max\_delta\_step(int), maximum delta step we allow each tree's weight estimation to be. Usually this parameter is not needed, but it might help in logistic regression when class is extremely imbalanced. 参考[官方参数说明文档](https://github.com/dmlc/xgboost/blob/master/doc/parameter.md)。

### 基本思路

1. Choose a relatively high learning rate. Generally a learning rate of 0.1 works but somewhere between 0.05 to 0.3 should work for different problems. Determine the optimum number of trees for this learning rate. XGBoost has a very useful function called as “cv” which performs cross-validation at each boosting iteration and thus returns the optimum number of trees required.
2. Tune tree-specific parameters ( max\_depth, min\_child\_weight, gamma, subsample, colsample\_bytree) for decided learning rate and number of trees. Note that we can choose different parameters to define a tree and I’ll take up an example here.
3. Tune regularization parameters (lambda, alpha) for xgboost which can help reduce model complexity and enhance performance.
4. Lower the learning rate and decide the optimal parameters .

参考[Complete Guide to Parameter Tuning in XGBoost (with codes in Python)](https://www.analyticsvidhya.com/blog/2016/03/complete-guide-parameter-tuning-xgboost-with-codes-python/)

## 参考文献

1. [XGBoost: A Scalable Tree Boosting System(官方论文)](https://arxiv.org/abs/1603.02754)
2. [Introduction to Boosted Trees](https://homes.cs.washington.edu/~tqchen/pdf/BoostedTree.pdf)
3. [xgboost python api的说明文档](http://xgboost.readthedocs.io/en/latest/python/python_api.html#module-xgboost.sklearn)
4. [官方的参数说明文档](https://github.com/dmlc/xgboost/blob/master/doc/parameter.md)
5. [Complete Guide to Parameter Tuning in XGBoost (with codes in Python)](https://www.analyticsvidhya.com/blog/2016/03/complete-guide-parameter-tuning-xgboost-with-codes-python/)
6. [Overtuning hyper parameters (especially re xgboost)](https://www.kaggle.com/c/santander-customer-satisfaction/discussion/20662)

---



