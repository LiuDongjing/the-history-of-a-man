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

## 默认参数
默认参数指在module加载的时候计算一次，所有不要使用list和dict等可变的数据作为默认值，否则在程序运行过程中，如果某段代码改变了默认值(比如在list末尾添加元素)，那么后续的代码使用的就是改变之后的默认值。
```python
No:  def foo(a, b=[]):
         ...
No:  def foo(a, b=time.time()):  # The time the module was loaded???
         ...
No:  def foo(a, b=FLAGS.my_thing):  # sys.argv has not yet been parsed...
```

## 属性
在设计类的时候，使用属性来保存、获取、修改数据，而不要accessor和setter方法。只读属性用@property装饰器。

## True/False判断
尽量使用隐式的False，在判断布尔值的时候，0、None、[]、{}和''都会判为False。有下面几条规则需要注意：
- 不要用==和!=来比较对象，应当使用is和is not
- 当你想表达if x is not None时，不要使用if x，因为x可能是[]或者{}，此时if x都不会通过
- 不要用==比较布尔变量和False，应当用if not x
- 当需要把False和None分开的时候，请用if not x and x is not None(只有False能通过这个判断)
- 判断string、dict、list、tuple等，记住数据为空是False，所以推荐用if not seq和if seq来代替if len(seq)和if not len(seq)
- 处理整数类型的数据时，应避免使用隐式的False，参考下面给的例子。
    ```python
    Yes: if not users:
         print 'no users'

     if foo == 0:
         self.handle_zero()

     if i % 10 == 0:
         self.handle_multiple_of_ten()

     No:  if len(users) == 0:
         print 'no users'

     if foo is not None and not foo:
         self.handle_zero()

     if not i % 10:
         self.handle_multiple_of_ten()
    ```

[0]: https://www.pylint.org/
[1]: https://www.python.org/dev/peps/pep-0008/
[2]: https://google.github.io/styleguide/pyguide.html