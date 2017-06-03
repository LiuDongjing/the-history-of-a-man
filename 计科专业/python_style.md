# Python coding style
可用[Pylint][0]来检查自己的代码是否符合规范，规范的细节可在配置文件里定义，一般参考Python官方的编码规范[PEP 8][1]。不过目前来看，自己还不适应规范的所有细节，需要一个过程。所以参考了[Google官方给的Python编码规范][2]，选取了常用的部分在以后的代码中执行。

## Style Rules
### 分号
一行的结尾不需要分号，也不要使用分号将两条语句分开放在一行

### 一行的长度
除了比较长的import语句和注释中的URL以外，一行最长80个字符，不要使用反斜杠续行，应当使用python隐式地续行方式。在小括号、中括号、大括号里的语句可以分散至多行，并且可以添加注释，允许空行，每行的缩进不重要。如下：

```python
month_names = ['Januari', 'Februari', 'Maart',      # These are the
               'April',   'Mei',      'Juni',       # Dutch names
               'Juli',    'Augustus', 'September',  # for the months
               'Oktober', 'November', 'December']   # of the year
```

一个字符串太长的话，可以分多行来写，如下：

```python
x = ('This will build a very long long '
     'long long long long long long string')
```

注释中有URL，URL应单独占一行。

### 缩进
使用4个空格缩进，不要用TAB。参数垂直方向对齐，或者第一行没有参数的时候，后续的缩进4个空格也行。参考下面的代码。

```python
# Aligned with opening delimiter
foo = long_function_name(var_one, var_two,
			  var_three, var_four)

# Aligned with opening delimiter in a dictionary
foo = {
   long_dictionary_key: value1 +
			 value2,
   ...
}

# 4-space hanging indent; nothing on first line
foo = long_function_name(
   var_one, var_two, var_three,
   var_four)

# 4-space hanging indent in a dictionary
foo = {
   long_dictionary_key:
	   long_dictionary_value,
   ...
}
```

### 空行
顶层的definition之间空两行，比如function和class。class line和第一个method之间空一行。在function和method内部可根据自己的需要在合适的地方空一行。

### 空白符
小括号、中括号和大括号里不要使用空白符。

```python
Yes: spam(ham[1], {eggs: 2}, [])
```

逗号、分号、冒号前无空白符，之后有空白符(除了行尾)
```python
Yes: if x == 4:
         print x, y
     x, y = y, x
```

小括号和中括号前无空白符。
```python
Yes: spam(1)
Yes: dict['key'] = list[index]
```

二元操作符两边各有一个空白符，比如=, ==, \<, \>, in, not, and等等。
```python
Yes: x == 1
```

当给keyword参数赋值或定义默认参数时，等号两边不需要空格。
```python
Yes: def complex(real, imag=0.0): return magic(r=real, i=imag)
```

不必使用空格来对齐一些字符(比如:、#和=等)，因为这样维护起来比较麻烦。
```python
Yes:
  foo = 1000  # comment
  long_name = 2  # comment that should not be aligned

  dictionary = {
      'foo': 1,
      'long_name': 2,
  }
```

### Shebang Line
大部分python文件都不需要**#!/usr/bin/env python**，只需要程序入口的py文件才需要。

### 注释
#### Doc strings
每个package、module、class和function的第一个字符串，可以使用pydoc生成文档。使用*"""string"""*来写doc string。doc string的组织格式如下：第一行是总结性的语句，用句号、问号或感叹号结尾。接下来空一行，剩下的是细节描述(和第一行对齐)。

#### Modules
每个文件都要包含一个license声明，比如Apache 2.0, BSD, LGPL, GPL。

#### functions和methods
除了下面三种情况，函数必须有一个docstring。
- 外部不可见
- 很短
- 作用显而易见

docstring应当描述函数的调用语法和对应的功能，不应涉及实现。对于比较难理解的代码，代码旁加注释要比docstring更合适。

docstirng由多个小节组成，每个小节有个heading line，后跟一个冒号，后面的内容要缩进两个空格。
- Args: 将输入参数按名字罗列出来，名字和说明用冒号和空格分开。当一行写不下时，剩余的行缩进两个或四个空格。说明应应包含参数的类型及意义。如果函数包含变长参数或keyword参数，应用*args或**kwargs表示。
- Returns: 对generator来说是Yields。描述返回参数的类型和语义，如果函数没有返回值，该小节可省略。
- Raises: 列出所有相关的异常。

下面这个是示例：
```python
def fetch_bigtable_rows(big_table, keys, other_silly_variable=None):
    """Fetches rows from a Bigtable.

    Retrieves rows pertaining to the given keys from the Table instance
    represented by big_table.  Silly things may happen if
    other_silly_variable is not None.

    Args:
        big_table: An open Bigtable Table instance.
        keys: A sequence of strings representing the key of each table row
            to fetch.
        other_silly_variable: Another optional variable, that has a much
            longer name than the other args, and which does nothing.

    Returns:
        A dict mapping keys to the corresponding table row data
        fetched. Each row is represented as a tuple of strings. For
        example:

        {'Serak': ('Rigel VII', 'Preparer'),
         'Zim': ('Irk', 'Invader'),
         'Lrrr': ('Omicron Persei 8', 'Emperor')}

        If a key from the keys argument is missing from the dictionary,
        then that row was not found in the table.

    Raises:
        IOError: An error occurred accessing the bigtable.Table object.
    """
    pass
```
#### Classes
class应当有docstring，如果该class有public属性，应当在对应的小节里描述出来。下面是个示例：
```python
class SampleClass(object):
    """Summary of class here.

    Longer class information....
    Longer class information....

    Attributes:
        likes_spam: A boolean indicating if we like SPAM or not.
        eggs: An integer count of the eggs we have laid.
    """

    def __init__(self, likes_spam=False):
        """Inits SampleClass with blah."""
        self.likes_spam = likes_spam
        self.eggs = 0

    def public_method(self):
        """Performs operation blah."""
```

#### 区块注释和行内注释
最后需要添加注释的地方就是一些一些比难懂的代码，在代码前面添加多行注释，个别需要特别解释的可以添加行内注释。为了方便阅读，注释与代码之间至少要间隔两个空格。参考下面的例子：
```python
# We use a weighted dictionary search to find out where i is in
# the array.  We extrapolate position based on the largest num
# in the array and the array size and then do binary search to
# get the exact number.

if i & (i-1) == 0:        # true iff i is a power of 2
```

### 类
如果自定义的类没有父类，应当继承object，这样就可以使用一些默认的方法，比如 \_\_new\_\_, \_\_init\_\_, \_\_delattr\_\_, \_\_getattribute\_\_, \_\_setattr\_\_, \_\_hash\_\_, \_\_repr\_\_和 \_\_str\_\_。

### Strings
格式化字符串时，除了两个字符串变量相邻用'+'外，其他情况应用'%'和format方法。参考下面的对比：
```python
Yes: x = a + b
     x = '%s, %s!' % (imperative, expletive)
     x = '{}, {}!'.format(imperative, expletive)
     x = 'name: %s; score: %d' % (name, n)
     x = 'name: {}; score: {}'.format(name, n)

No: x = '%s%s' % (a, b)  # use + in this case
    x = '{}{}'.format(a, b)  # use + in this case
    x = imperative + ', ' + expletive + '!'
    x = 'name: ' + name + '; score: ' + str(n)
```

避免在循环内使用'+'或者'+='来累加字符串(因为string是immutable的，连续累加会产生很多不必要的临时对象)，应当使用list来添加子字符串，最后在调用join方法。参考下面的做法：
```python
Yes: items = ['<table>']
     for last_name, first_name in employee_list:
         items.append('<tr><td>%s, %s</td></tr>' % (last_name, first_name))
     items.append('</table>')
     employee_table = ''.join(items)
```

在代码中使用单引号和双引号都可以，但要保持一致(除了避免特殊字符转义之外)。
```python
Yes:
  Python('Why are you hiding your eyes?')
  Gollum("I'm scared of lint errors.")
  Narrator('"Good!" thought a happy Python reviewer.')
```

多行字符串更倾向于使用"""，而不是'''。
```python
Yes:
  print ("This is much nicer.\n"
         "Do it this way.\n")
```

### Files和Sockets
打开的files和sockets应当显式关闭。一般使用with语句，格式如下：
```python
with open("hello.txt") as hello_file:
    for line in hello_file:
        print line
```

如果有file-like object不支持with，可使用contxtlib.closing()。
```python
import contextlib

with contextlib.closing(urllib.urlopen("https://www.python.org/")) as front_page:
    for line in front_page:
        print line
```

### TODO
使用TODO来说明一段代码是临时的解决方案。应当包含联系人(对这段代码有解释权的人)的姓名或者联系方式和进一步如何做的简短说明，两者之间用冒号或者空格分开(前后要保持一致)。格式如下：
```python
# TODO(kl@gmail.com): Use a "*" here for string repetition.
# TODO(Zeke) Change this to use relations.
```

### import格式
每个import语句占一行，在module的注释和docstring之后，module全局变量和常量之前。分三部分按顺序组织：
1. standard library imports
2. third-party imports
3. application-specific imports

每部分内部按字母顺序排列(不分大小写)，例如：
```python
import foo
from foo import bar
from foo.bar import baz
from foo.bar import Quux
from Foob import ar
```

### Statements
通常情况下一条语句占一行。但没有else的if语句除外，参考下面的对比：
```python
Yes:

  if foo: bar(foo)

No:

  if foo: bar(foo)
  else:   baz(foo)

  try:               bar(foo)
  except ValueError: baz(foo)

  try:
      bar(foo)
  except ValueError: baz(foo)
```

### 命名
避免使用的名字
- 除了计数器和迭代器之外的单字母名称
- 含有短横线(-)的package/module名称
- 前后都有两个下划线的名称(Python保留)

命名的一些约定
- 'Internal'指module的内部对象或者class的protect和private对象
- 前面有一个下划线的名称是module的protect变量和函数(无法用import * from导出)。前面有两个下划线的名称是class的private变量或方法
- 把相关的类或者top level函数放在同一个文件里

关于命名的建议

|Type|Public|Internal|
|---|---|---|
|Packages|lower_with_under||
|Modules|lower_with_under|_lower_with_under|
|Classes|CapWords|_CapWords|
|Exceptions|CapWords||
|Functions|lower_with_under()|_lower_with_under()|
|Global/Class Constants|CAPS_WITH_UNDER|_CAPS_WITH_UNDER|
|Global/Class Variables|lower_with_under|_lower_with_under|
|Instance Variables|lower_with_under|_lower_with_under (protected) or __lower_with_under (private)|
|Method Names|lower_with_under()|_lower_with_under() (protected) or __lower_with_under() (private)|
|Function/Method Parameters|lower_with_under||
|Local Variables|lower_with_under||

### Main
module内的top level代码在import的时候会被执行，因此不要在这里调用函数、创建对象。main代码放在main函数里，并且使用**_\_name\_\_ == '\_\_main\_\_'**做检查，如下：
```python
def main():
      ...

if __name__ == '__main__':
    main()
```

## Language Rules
### import

```python
import x #导入packages或者modules

from x import y # x是package，y是module

from x import y as z # 当有两个同名的module时或者y的名字比较长
```
**注意** 使用package的时候要把名称写全。

### 避免使用全局变量
这里的全局变量是指modlue level的变量。除了下面四种情况，请避免使用全局变量。
- 脚本默认就有的变量，比如\_\_name\_\_
- module-level的常量，比如PI。定义常量请使用大写字母和下划线。
- 偶尔使用全局变量缓存一些数据很方便
- 如果确实需要全局变量，请将它设置成module的内部变量，然后使用module的pubilc函数获取

### 函数和类的嵌套定义
class可以在method、function和class内部定义，function可以在method和function内部定义。这样的写法都是可以的。

### 使用默认的迭代器和运算符
推荐用法
```python
for key in adict: ...
if key not in adict: ...
if obj in alist: ...
for line in afile: ...
for k, v in dict.iteritems(): ...
```

### lambda函数
代码量不超过60个字符，可以使用lambda函数。否则，使用一般(嵌套)的函数。

### 条件表达式
一行可以写完的话，就可以用这种方式，否则就老老实实用if语句。比如：

```python
x = 1 if cond else 2
```

### 默认参数
默认参数指在module加载的时候计算一次，所有不要使用list和dict等可变的数据作为默认值，否则在程序运行过程中，如果某段代码改变了默认值(比如在list末尾添加元素)，那么后续的代码使用的就是改变之后的默认值。
```python
No:  def foo(a, b=[]):
         ...
No:  def foo(a, b=time.time()):  # The time the module was loaded???
         ...
No:  def foo(a, b=FLAGS.my_thing):  # sys.argv has not yet been parsed...
```

### 属性
在设计类的时候，使用属性来保存、获取、修改数据，而不要accessor和setter方法。只读属性用@property装饰器。

### True/False判断
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

### 多线程
不要依赖python内置类型的原子操作，比如dict，因为在某些特殊情况下，这些类型的原子操作并不具有原子性。推荐使用Queue模块和threading模块，可以的话，应当使用threading.Condition而不是低层的lock。

### 特殊功能
非必须的情况下，不要使用这些特殊的功能。比如metaclass、获取字节码、on-the-fly 编译、动态继承、object reparenting、import hacks、reflection、modification of system internals等。

## 后记
最重要的一点就是前后保持一致。定义编码风格规范的目的是方便交流，使得其他人能将注意力放在你要表达什么上，而不是你是怎么表达的。基于这一点，在修改他人代码时，也应当遵守本地的编码风格，保持一致。

[0]: https://www.pylint.org/
[1]: https://www.python.org/dev/peps/pep-0008/
[2]: https://google.github.io/styleguide/pyguide.html