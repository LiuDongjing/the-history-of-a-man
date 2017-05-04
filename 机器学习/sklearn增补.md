# sklearn增补
## 分离测试集和训练集
使用[train_test_split][0]。用法如下
```python
from sklearn.model_selection import train_test_split
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.5, random_state=0)
```

## Grid Search
也就是把所有可能的参数组合都试一遍，并没有特别高深的地方。[GridSearchCV][1]。
关键参数：
- estimator: 实现了sklearn的estimator接口，需要提供score函数
- param_grid: 可能的参数组合，是个dict，键值是参数的名字，值是可能的取值列表
- cv：交叉验证

调用fit方法后，可使用GridSearchCV对象的best\_params\_方法返回最好的参数。estimator的
score函数形式如下：
```
score(self, X, y)
```

[0]: http://scikit-learn.org/stable/modules/generated/sklearn.model_selection.train_test_split.html#sklearn.model_selection.train_test_split
[1]: http://scikit-learn.org/stable/modules/generated/sklearn.model_selection.GridSearchCV.html
