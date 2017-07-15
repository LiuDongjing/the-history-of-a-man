# C++ coding style
参考[Google的编码规范](https://google.github.io/styleguide/cppguide.html)。C++的东西比较多，只挑了常用的几点。

## 头文件
一般来说，除了unittest的.cc文件和包含main()函数的.cc文件外，每个.cc文件都应该有一个对应的.h文件。

模板和inline函数在.h文件里声明和定义；如果他们是一个模块私有的，可以在.cc里面声明和定义。

\#define guard。在头文件中使用，避免重复包含。格式
```
<PROJECT>_<PATH>_<FILE>_H_
```
示例如下：
```cpp
#ifndef FOO_BAR_BAZ_H_
#define FOO_BAR_BAZ_H_

...

#endif  // FOO_BAR_BAZ_H_
```

inline函数，10行以内的代码可以使用inline函数。一般来说，析构函数不要inline(因为会隐式调用成员变量的析构函数)。inline函数里也不应包括循环语句和switch(为什么)语句。虽然虚函数和递归函数可以加inline关键字，但实际上编译器不会讲他们处理成inline的形式。

头文件的顺序：Related header(也就是这个.cc文件对应的.h文件), C library, C++ library, other libraries' .h, your project's .h。需要根据条件来包含的头文件放在最后。

## 作用域
### Namespaces
除了极个别情况外，都应该将代码放在一个namespace里。不要使用*using*指令(比如using namespace foo)。使用namespace的一个例子：
```cpp
// In the .h file
namespace mynamespace {

// All declarations are within the namespace scope.
// Notice the lack of indentation.
class MyClass {
 public:
  ...
  void Foo();
};

}  // namespace mynamespace

// In the .cc file
namespace mynamespace {

// Definition of functions is within scope of the namespace.
void MyClass::Foo() {
  ...
}

}  // namespace mynamespace
```

只在.cc文件里使用的代码应该放在unnamed namespace里面，这样外部就无法访问了(代替static的用法)。示例：
```cpp
namespace {
...
}  // namespace
```

如无必要，应将nonmember function和static function放在namespace里，而不是单独创造一个类。

### 局部变量
在哪使用就在哪声明，并且声明的时候就要初始化。

只在if, while, for内部使用的变量，应当在它们作用域内部声明，比如：
```cpp
while (const char* p = strchr(str, '/')) str = p + 1;
```
但如果局部变量是对象，就不推荐这么做，因为每个循环都会调用构造函数和析构函数，应采用下面的做法：
```cpp
Foo f;  // My ctor and dtor get called once each.
for (int i = 0; i < 1000000; ++i) {
  f.DoSomething(i);
}
```

## 命名
文件名
```
my_useful_class.cc
```

类型名
```cpp
// classes and structs
class UrlTable { ...
class UrlTableTester { ...
struct UrlTableProperties { ...

// typedefs
typedef hash_map<UrlTableProperties *, string> PropertiesMap;

// using aliases
using PropertiesMap = hash_map<UrlTableProperties *, string>;

// enums
enum UrlTableErrors { ...
```

一般的变量名
```cpp
string table_name;  // OK - uses underscore.
string tablename;   // OK - all lowercase.
```

类的成员变量名
```cpp
class TableInfo {
  ...
 private:
  string table_name_;  // OK - underscore at end.
  string tablename_;   // OK.
  static Pool<TableInfo>* pool_;  // OK.
};
```

常量名，前面加一个'k'
```cpp
const int kDaysInAWeek = 7;
```

函数名
```cpp
AddTableEntry()
DeleteUrl()
OpenFileOrDie()
```

namespace命名，都是小写字母，顶层的namespace根据项目名称来。

enum命名
```cpp
enum UrlTableErrors {
  kOK = 0,
  kErrorOutOfMemory,
  kErrorMalformedInput,
};
```

## 注释
函数注释
```cpp
// Returns an iterator for this table.  It is the client's
// responsibility to delete the iterator when it is done with it,
// and it must not use the iterator once the GargantuanTable object
// on which the iterator was created has been deleted.
//
// The iterator is initially positioned at the beginning of the table.
//
// This method is equivalent to:
//    Iterator* iter = table->NewIterator();
//    iter->Seek("");
//    return iter;
// If you are going to immediately seek to another place in the
// returned iterator, it will be faster to use NewIterator()
// and avoid the extra seek.
Iterator* GetIterator() const;
```

变量注释
```
private:
 // Used to bounds-check table accesses. -1 means
 // that we don't yet know how many entries the table has.
 int num_total_entries_;
```

## 其他
- 一行最多80个字符
- 不推荐使用异常
- 一般不使用struct关键字。
- 所有的继承都应该是public的。
- 如果类有virtual function，那么它的析构函数也应该是virtual的。
- 定义类的时候，按照public、protected和private的顺序来。
