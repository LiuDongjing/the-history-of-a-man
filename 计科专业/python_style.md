# Python coding style
可用[Pylint][0]来检查自己的代码是否符合规范，规范的细节可在配置文件里定义，一般参考Python官方的编码规范[PEP 8][1]。不过目前来看，自己还不适应规范的所有细节，需要一个过程。所以参考了[Google官方给的Python编码规范][2]，选取了常用的部分在以后的代码中执行。

## import

```python
import x #导入packages或者modules

from x import y # x是package，y是module

from x import y as z # 当有两个同名的module时或者y的名字比较长
```
**注意** 使用package的时候要把名称写全。

## 避免使用全局变量
这里的全局变量是指modlue level的变量。除了下面四种情况，请避免使用全局变量。
- 脚本默认就有的变量，比如\_\_name\_\_
- module-level的常量，比如PI。定义常量请使用大写字母和下划线。
- 偶尔使用全局变量缓存一些数据很方便
- 如果确实需要全局变量，请将它设置成module的内部变量，然后使用module的pubilc函数获取

## 函数和类的嵌套定义
class可以在method、function和class内部定义，function可以在method和function内部定义。这样的写法都是可以的。

## 使用默认的迭代器和运算符
推荐用法
```python
for key in adict: ...
if key not in adict: ...
if obj in alist: ...
for line in afile: ...
for k, v in dict.iteritems(): ...
```

## lambda函数
代码量不超过60个字符，可以使用lambda函数。否则，使用一般(嵌套)的函数。

## 条件表达式
一行可以写完的话，就可以用这种方式，否则就老老实实用if语句。比如：

```python
x = 1 if cond else 2
```

[0]: https://www.pylint.org/
[1]: https://www.python.org/dev/peps/pep-0008/
[2]: https://google.github.io/styleguide/pyguide.html