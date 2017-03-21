> 2017-03-21 杭州 阴

# python机器学习
使用python来做机器学习相关的项目或者比赛，主要是学习两个包，scikit-learn和pandas，前者集成了各种常用的机器学习模型和算法，并提供了统一的使用接口；后者提供了处理体量较大的数据，使用类似于SQL的语法来查询更改数据。下面分别对两者做些简单的介绍。

## scikit-learn
以使用线性回归算法为例

```python
    from sklearn.linear_model import LinearRegression
    X = [[0], [1], [2], [4]]
    y = [0, 1, 2, 4]
    reg = LinearRegression()
    # 训练
    reg.fit(X, y)
    # 预测
    print(reg.predict([[5], [7]]))
    # 预测结果是[ 5.  7.]
```

sklearn中的模型基本上都遵循这个模式，先创建一个模型对象，然后调用fit方法训练模型，
最后使用predict方法做预测。更多模型的细节可参考官方的[API文档](http://scikit-learn.org/stable/modules/classes.html)。

### 保存、加载训练好的模型
使用sklearn中的joblib这个包，使用方法比较简单。
```python
    from sklearn.externals import joblib
    #将训练好的模型对象reg保存到文件linear_regression.pkl中_
    joblib.dump(reg, 'linear_regression.pkl')
    #...
    #从磁盘中加载模型
    reg = joblib.load('linear_regression.pkl')
```

值得提出的一点是joblib主要的功能是序列化python对象，所以除了保存和加载模型外，
还可以保存、加载其他的python对象，比如DataFrame对象。官方推荐使用joblib来代替python自带的
pickle包，因为前者在处理体量更大的数据时更加高效。

### 评价模型
使用sklearn.metrics包，用法比较简单，具体的可参考官方文档。下面
罗列了一些常见的metrics。

- Mean squared error, [metrics.mean_squared_error](http://scikit-learn.org/stable/modules/generated/sklearn.metrics.mean_squared_error.html#sklearn.metrics.mean_squared_error)
- accuracy, [metrics.accuracy_score](http://scikit-learn.org/stable/modules/generated/sklearn.metrics.accuracy_score.html#sklearn.metrics.accuracy_score)
- f1, [metrics.f1_score](http://scikit-learn.org/stable/modules/generated/sklearn.metrics.f1_score.html#sklearn.metrics.f1_score)
- logistic loss(也称cross-entropy loss), [metrics.log_loss](http://scikit-learn.org/stable/modules/generated/sklearn.metrics.log_loss.html#sklearn.metrics.log_loss)

### 交叉验证
使用sklearn.model_selection包，使用方法也比较简单。

```python
    def official_loss(estimator, X, y):
        '''官方定义的loss函数'''
        #注意重置index，不然会出现意想不到的问题
        y_ = y.reset_index(drop=True)
        y_p = estimator.predict(X)
        adds = (y_p + y_).abs()
        subs = (y_p - y_).abs()
        divs = subs / adds
        N = divs.shape[0] * divs.shape[1]
        return divs.sum().sum() / N

    #注意n_jobs使用多CPU时，不可以调试，否则会抛出异常
    scores = cross_val_score(model, feature, label, 
                                cv=KFold(n_splits=3, shuffle=False), 
                                #n_jobs=-1, 
                                scoring=official_loss
                                )
```

上面的代码把数据划分为3份，总共做三次验证操作，每次用一份数据作为验证集，得到3个对应的score并返回。

model就是一个模型对象，可以是LinearRegression，也可以是LogisticRegression。
feature和label分别是用于交叉验证的样本和标签，cv的[KFold](http://scikit-learn.org/stable/modules/generated/sklearn.model_selection.KFold.html#sklearn.model_selection.KFold.split)
对象就是用于将feature和label划分成均等的n_splits份，shuffle为True表示在划分数据前先打乱数据的顺序，默认
False。scoring参数使用哪种指标来评价模型，可以传入sklearn内置的指标(字符串)，也可以传入一个自定义的函数，
自定义函数的格式参考上面的代码。常见的scoring函数有下面几个，和上面的metrics也是对应的。

- 'accuracy'
- 'f1'
- 'neg_log_loss'
- 'neg_mean_squared_error'

**注意** 如果model属于自定义的模型类，该类要继承BaseEstimator并重写get_params方法，在这方法里返回相关的
模型参数(包括自定义的参数)。因为在cross_val_score函数内部，会根据model对象生成多个同类对象用于交叉验证过
程中训练数据，而这种根据对象生成另一个对象的过程会调用对象的get_params方法。

### 其他问题
#### 选择模型的经验规则

![经验规则](images/sklearn_choose.png)
<center>sklearn官方给的经验规则</center>

#### class imbalance problem
当训练数据集中正样本太少时，可采取一些数据处理方法。python中可使用[SMOTE](http://contrib.scikit-learn.org/imbalanced-learn/generated/imblearn.over_sampling.SMOTE.html)
模块。安装SMOTE可用_conda install -c glemaitre imbalanced-learn_
或者 _pip install -U imbalanced-learn_。使用方法参考下面的代码。

```python
    from imblearn.over_sampling import SMOTE
    sm = SMOTE()
    #转换原始数据，使得正负样本数量均衡
    X_res, y_res = sm.fit_sample(X, y)
```

#### 一些细节问题
1. 在sklearn中，除非特别说明，传给模型的训练数据都会默认转换为_float64_类型。
Regression的targets默认转换为_float64_类型，而classification的targets的对象类型
默认保持不变。
2. 每次调用模型的fit函数，会自动清空之前训练好的参数。

## pandas
处理数据的核心类是DataFrame，可以把它简单地看成一张二维数据表，有index(行索引)和columns
(列索引)，大部分操作都是通过index和columns进行的。需要注意的一点是这里的index和columns
除了数字以外，还可以是其他数据类型，和数据库中的表格一样，一般会给columns中的每一列取一个
有意义的名字，便于后续的操作。

**注意** 在有些操作中，如果两个表的除了index不一样，其他条件都满足，操作的结果可能和直观的
预期不一样，比如讲某一列数据insert另一个DataFrame中，如果两者的index不一样，最终的DataFrame
的index是只有后者的index，那么前者没有的index值对应的数据就会出现NaN。类似的操作还有矩阵的
加减乘除操作。下面的代码说明了这种情况。

```python
    x = pd.DataFrame({'val':[1,2,3,4]})
    y = pd.DataFrame({'new':[1,2,3,4]},index=[2,3,4,5])
    print(x)
    print(y)
    x.insert(0,'col',y)
    print(x)
```

最后的输出结果如下

```
   val
0    1
1    2
2    3
3    4

   new
2    1
3    2
4    3
5    4

   col  val
0  NaN    1
1  NaN    2
2  1.0    3
3  2.0    4
```

### 基本操作
以代码为例说明基本用法，更详细的用法可以参考[官方文档](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.html#pandas.DataFrame)。

**创建DataFrame对象**
```python
    import pandas as pd
    #创建时用一个dict指定每一列的数据，列名即是键值。每一列的数据可以
    #是list，也可是另一个DataFrame。
    df = pd.DataFrame({'id':[1,2,3,4], 'val':[0.1,0.2,0.3,0.4]})
```
**从csv文件读入或保存到csv**
```python
    #header，指明是否用第一行数据作为列名。如果没有提供names参数， 默认用
    #   第一行数据作为列名，否则使用names指定的。设为None，表示使用
    #   names作为列名。
    #names，指定每一列的名称
    #dtype，指定每一列的数据类型，如果每一列的类型不一样，可传入一个dict
    #   对象，key是列名，value是数据类型
    #usecols，是个int list，指定只读入哪些列数据，第一列的索引是0,
    #   依次类推
    user_pay = pd.read_csv('user_pay.txt', header = None,
                            names = ['uid', 'sid', 'stamp'],
                            dtype = np.str)
    #...

    #index, 置为False表示不将DataFrame的index保存到csv文件中
    #header，是否将列名保存到csv中，默认是True。
    user_pay.to_csv('user_pay_op.csv', index=False, encoding='utf-8')
```

### 查改增删
