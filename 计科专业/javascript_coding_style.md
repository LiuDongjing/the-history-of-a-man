# JavaScript Coding Style
参考[Google的JavaScript编码规范](https://google.github.io/styleguide/jsguide.html)
## 关于源文件
- 文件名必须都是小写字母，可以包含下划线"\_"和短横线"-"，但不能包含其他的标点符号。后缀名必须是".js"。
- 源码文件都使用UTF-8编码
- 使用空格缩进，除了换行符和回车符以外，源码中能出现的空白字符只有空格(0x20)，字符串中其他的空白字符请使用转义。
- 有专门转义表示的，比如\\'，\\"，\\\\，\\b，\\f，\\n， \\r，\\t，\\v，就用这种方式，不要用对应的数字表示，比如\\x0a。
- 对于其他的非ASCII字符，可以直接用字符表示(比如∞)，也可以使用对应的Unicode代码(比如\\u221e)，那种方便阅读理解就使用那种。对于可以显示的字符，使用前一种方式，对于不可以显示的字符，用后一种方式，紧接着后面添加注释即可。
## 源文件结构
源文件从上到下包含以下几个部分：
1. License 或者copyright信息(可选)
2. @fileoverview JSDoc(可选)
3. module 语句
4. require语句
5. 代码实现

module语句的示例
```javascript
goog.module('search.urlHistory.UrlHistoryService');
```

不要使用ES6的export和import语句，因为它们的语义还没有正式化。

require示例：
```javascript
const MyClass = goog.require('some.package.MyClass');
const NsMyClass = goog.require('other.ns.MyClass');
const googAsserts = goog.require('goog.asserts');
const testingAsserts = goog.require('goog.testing.asserts');
const than80columns = goog.require('pretend.this.is.longer.than80columns');
const {clear, forEach, map} = goog.require('goog.array');
/** @suppress {extraRequire} Initializes MyFramework. */
goog.require('my.framework.initialization');
```

代码实现和require语句至少空一行。

## 大括号
所有的控制语句(if、while等)的主体都要用大括号包起来，即使主体只有一行。唯一的例外是if语句很简单，可占一行，并且没有else，如下
```javascript
if (shortCondition()) return;
```

大括号的使用示例：
```javascript
class InnerClass {
  constructor() {}

  /** @param {number} foo */
  method(foo) {
    if (condition(foo)) {
      try {
        // Note: this might fail.
        something();
      } catch (err) {
        recover();
      }
    }
  }
}
```

如果"}"后面跟的是else，catch，while，逗号，分号和")"，那么就不需要在"}"后面换行。

如果"{}"内没有语句，比如空的构造函数，那么"{"之后紧跟着"}"，不要添加任何字符；例外情况是这样的空块属于某个语句块的一部分，比如if else，try catch finally，那么就要按前面提到的格式写。

正确的写法：
```javascript
function doNothing() {}
```

错误的写法
```javascript
if (condition) {
  // …
} else if (otherCondition) {} else {
  // …
}

try {
  // …
} catch (e) {}
```

## 缩进
使用两个空格缩进，注释和代码块使用同样的缩进。

创建数组时的缩进方式

```javascript
const a = [
  0,
  1,
  2,
];

const b =
    [0, 1, 2];
const c = [0, 1, 2];

someMethod(foo, [
  0, 1, 2,
], bar);
```

对象数据的缩进方式
```javascript
const a = {
  a: 0,
  b: 1,
};

const b =
    {a: 0, b: 1};
const c = {a: 0, b: 1};

someMethod(foo, {
  a: 0, b: 1,
}, bar);
```

类的缩进

```javascript
class Foo {
  constructor() {
    /** @type {number} */
    this.x = 42;
  }

  /** @return {number} */
  method() {
    return this.x;
  }
}
Foo.Empty = class {};

/** @extends {Foo<string>} */
foo.Bar = class extends Foo {
  /** @override */
  method() {
    return super.method() / 2;
  }
};

/** @interface */
class Frobnicator {
  /** @param {string} message */
  frobnicate(message) {}
}
```

函数的缩进，当匿名函数作为参数传递给另一个函数时，匿名函数的主体相对于上一层缩进两个空格，如下：
```javascript
prefix.something.reallyLongFunctionName('whatever', (a1, a2) => {
  // Indent the function body +2 relative to indentation depth
  // of the 'prefix' statement one line above.
  if (a1.equals(a2)) {
    someOtherLongFunctionName(a1);
  } else {
    andNowForSomethingCompletelyDifferent(a2.parrot);
  }
});

some.reallyLongFunctionCall(arg1, arg2, arg3)
    .thatsWrapped()
    .then((result) => {
      // Indent the function body +2 relative to the indentation depth
      // of the '.then()' call.
      if (result) {
        result.use();
      }
    });
```

switch语句的缩进
```javascript
switch (animal) {
  case Animal.BANDERSNATCH:
    handleBandersnatch();
    break;

  case Animal.JABBERWOCK:
    handleJabberwock();
    break;

  default:
    throw new Error('Unknown animal');
}
```

## 语句
- 一条语句占一行
- 分号结束每一条语句
- 一行最多80个字符，例外的是URL等确实无法分割的情况，以及module、require语句。超过80个字符的要把一条语句拆分成多行。

拆分的时候偏向于从higher syntactic level拆分，如下
```javascript
currentEstimate =
    calc(currentEstimate + x * currentEstimate) /
        2.0f;
```

- 如果在运算符处拆分，拆分发生在运算符后面，但"."除外
- method和constructor与"("连在一起
- 逗号和它前面的字符连在一起
- 后面的行相对于第一行至少缩进4个空格

## 空白字符
- 定义类和对象的时候，相邻的方法间空一行。但在定义对象时，如果两个属性间没有其他代码，可不空行。在这种情况下使用空行是为了将属性分成不同的组。
- 函数体内部，使用空行将语句分成不同的逻辑组。函数体前后句不许有空行。
- 行尾不应有空白字符
- 保留关键字(比如if, for, catch)和"("之间需要一个空格
- 保留关键字(else, catch)和“}”之间又一个空格
- “{”前面有一个空格，但有两种例外：
    - template expansion, 比如abc${1 + 2}def
    - object作为函数的第一个参数或者数组中的第一个元素，比如foo({a: [{c: d}]})
- 二元或三元运算符的两边都需要空格
- 逗号和分号后面需要一个空格，但两者的前面不需要空格
- 在定义object时，分号的后面需要一个空格
- 行尾注释“//”的两边都需要空格
- JSDoc注释"/\*\*"的后面需要一个空格，“\*/”的两边都需要一个空格，比如：
    ```javascript
    this.foo = /** @type {number} */ (bar); or function(/** string */ foo) {}
    ```
- 垂直方向不需要刻意对齐，比如
    ```javascript
    {
      tiny: 42, // this is great
      longer: 435, // this too
    };

    {
      tiny:   42,  // permitted, but future edits
      longer: 435, // may leave it unaligned
    };
    ```

传递函数参数时的格式
```javascript
// Arguments start on a new line, indented four spaces. Preferred when the
// arguments don't fit on the same line with the function name (or the keyword
// "function") but fit entirely on the second line. Works with very long
// function names, survives renaming without reindenting, low on space.
doSomething(
    descriptiveArgumentOne, descriptiveArgumentTwo, descriptiveArgumentThree) {
  // …
}

// If the argument list is longer, wrap at 80. Uses less vertical space,
// but violates the rectangle rule and is thus not recommended.
doSomething(veryDescriptiveArgumentNumberOne, veryDescriptiveArgumentTwo,
    tableModelEventHandlerProxy, artichokeDescriptorAdapterIterator) {
  // …
}

// Four-space, one argument per line.  Works with long function names,
// survives renaming, and emphasizes each argument.
doSomething(
    veryDescriptiveArgumentNumberOne,
    veryDescriptiveArgumentTwo,
    tableModelEventHandlerProxy,
    artichokeDescriptorAdapterIterator) {
  // …
}
```

## 小括号
推荐使用小括号将语句的不同部分分组，但下面的语句后面不需要多余的小括号，delete, typeof, void, return, throw, case, in, of, yield。类型转换时需要小括号，比如/\*\* @type {!Foo} \*/ (foo)

## Implementation comments
区块注释和代码在同一个indent level上，可以使用/\*\*/也可以使用//。使用多行注释/\*\*/时，后面的行开头也要有"\*"。当参数的意义不够明显时，可在后面添加注释，参考下面的示例。
```javascript
/*
 * This is
 * okay.
 */

// And so
// is this.

/* This is fine, too. */

someFunction(obviousParam, true /* shouldRender */, 'hello' /* name */);
```

不要和JSDoc的注释(/\*\*...\*/)混淆了。

## 语言特性
### 声明局部变量
- 所有的局部变量用const或let来声明，除非需要重新赋值，默认使用const声明变量(const 变量不可以重新用等号赋值，也不可重新声明，如果指向的是对象，对象的属性是可以更改的)。不要使用var声明变量。
- 每次只声明一个局部变量，下面的这种方式是不允许的：let a = 1, b = 2;
- 不要在局部块开始声明局部变量，在局部变量首次使用的地方声明局部变量。
- 必要的话，使用JSDoc type annotations指明变量的类型，比如：
    ```javascript
    const /** !Array<number> */ data = [];

    /** @type {!Array<number>} */
    const data = [];
    ```

### 数组
- 在逗号处换行，如下：
    ```javascript
    const values = [
      'first value',
      'second value',
    ];
    ```
- 避免使用Array构造函数，除非是声明指定长度的数组。
- 可以unpack一个array或者iterable来给多个变量赋值，不需要的可以空着，...rest将剩余的都存入rest变量里，如下：
    ```javascript
    const [a, b, c, ...rest] = generateResults();
    let [, b,, d] = someArray;
    ```
    给函数传参数时也可以使用这种方法，如果参数时可选的话，左边提供参数的默认值，后边默认情况下是[]，如下：
    ```javascript
    /** @param {!Array<number>=} param1 */
    function optionalDestructuring([a = 4, b = 2] = []) { … };
    ```
- 推荐使用spread operator(...)，注意后面没有空格，可以将可迭代的对象分割成数组，也可以将多个可迭代对象合并成一个数组，如下：
    ```javascript
    [...foo]   // preferred over Array.prototype.slice.call(foo)
    [...foo, ...bar]   // preferred over foo.concat(bar)
    ```

### 对象
- 最后一个属性后面始终跟一个逗号
- 避免使用Object构造函数，用{}代替
- 在定义一个对象时，不要将struct style key 和dict style key混在一起，如下:
    ```javascript
    {
      a: 42, // struct-style unquoted key
      'b': 43, // dict-style quoted key
    }
    ```
- 对象中也能定义函数，有下面两种方法：
    - 这里的this就是指函数所在的对象
        ```javascript
        return {
          stuff: 'candy',
          method() {
            return this.stuff;  // Returns 'candy'
          },
        };
        ```
    - 这里的this指的是对象外的scope
        ```javascript
        class {
          getObjectLiteral() {
            this.stuff = 'fruit';
            return {
              stuff: 'candy',
              method: () => this.stuff,  // Returns 'fruit'
            };
          }
        }
        ```
- unpack, 和数组unpack的用法一样，也可以用来给函数传参，比如：
    ```javascript
    /**
     * @param {string} ordinary
     * @param {{num: (number|undefined), str: (string|undefined)}=} param1
     *     num: The number of times to do something.
     *     str: A string to do stuff to.
     */
    function destructured(ordinary, {num, str = 'some default'} = {})
    ```
- 枚举变量，使用@enum annotation来标识，必须是const的，并且里面的元素也必须是只读的(比如数字和字符串)。定义的时候不能包含非enum的元素，示例如下：
    ```javascript
    /**
     * Supported temperature scales.
     * @enum {string}
     */
    const TemperatureScale = {
      CELSIUS: 'celsius',
      FAHRENHEIT: 'fahrenheit',
    };

    /**
     * An enum with two options.
     * @enum {number}
     */
    const Option = {
      /** The option used shall have been the first. */
      FIRST_OPTION: 1,
      /** The second among two options. */
      SECOND_OPTION: 2,
    };
    ```

### 类
- 一般来说，constructor是可选的，子类在设置任何域之前要调用super函数。interface不应定义constructor。
- 所有的域都在构造函数里初始化，必要的话，使用@const和@private来标识域，private域名称后面跟一个下滑想。不要使用prototype来设置域。如下：
    ```javascript
    class Foo {
      constructor() {
        /** @private @const {!Bar} */
        this.bar_ = computeBar();
      }
    }
    ```
- 构造函数结束之后，不应再添加和删除域，这样不利于VM做优化。未初始化的域应当显式置为undefined。
- 可迭代的类要定义[Symbol.iterator]方法
- 静态方法，不影响可读性的话，推荐使用module-local的函数，而不是类的静态函数。使用关键字static来定义，不要通过对象来调用static方法，也不要通过子类来调用。
- 不要使用getter和setter属性。
- 可以覆盖toString方法，但不能有可见的副作用。
- interface使用@interface或者@record定义，非静态方法必须是空的，域紧随主体之后通过prototype定义，如下：
    ```javascript
    /**
     * Something that can frobnicate.
     * @record
     */
    class Frobnicator {
      /**
       * Performs the frobnication according to the given strategy.
       * @param {!FrobnicationStrategy} strategy
       */
      frobnicate(strategy) {}
    }

    /** @type {number} The number of attempts before giving up. */
    Frobnicator.prototype.attempts;
    ```

### 函数
- 需要export的函数直接放在exports对象里，如下：
    ```javascript
    /** @return {number} */
    function helperFunction() {
      return 42;
    }
    /** @return {number} */
    function exportedFunction() {
      return helperFunction() * 2;
    }
    /**
     * @param {string} arg
     * @return {number}
     */
    function anotherExportedFunction(arg) {
      return helperFunction() / arg.length;
    }
    /** @const */
    exports = {exportedFunction, anotherExportedFunction};
    ```
- 嵌套函数推荐使用arrow function而不是function关键字，因为它能解决在函数内使用this的问题。在callback中也推荐使用arrow function。推荐使用arrow function而不是f.bind(this)。
- Generator，定义generator时在关键字function后面添加"\*"，如果yield后面也是一个generator，应该在yield后面添加"\*"，如下：
    ```javascript
    /** @return {!Iterator<number>} */
    function* gen1() {
      yield 42;
    }

    /** @return {!Iterator<number>} */
    const gen2 = function*() {
      yield* gen1();
    }

    class SomeClass {
      /** @return {!Iterator<number>} */
      * gen() {
        yield 42;
      }
    }
    ```
- 参数，除非使用@override，函数的参数都要使用JSDoc annotation来标注。可以在参数名前面指明参数的类型，例如，(/\*\* number \*/ foo, /\*\* string \*/ bar) => foo + bar，但这种方式不要和使用@param的annotation方式混在一起。
- 默认参数使用=来指定(等号两边都有空格)，@param后面也有等号，和python不同的是，默认值可以是可变对象，应为每次调用都会重新计算默认值。当默认参数多的时候，推荐使用上面的array或object的unpack方法，这样就没有顺序限制。如下：
    ```javascript
    /**
     * @param {string} required This parameter is always needed.
     * @param {string=} optional This parameter can be omitted.
     * @param {!Node=} node Another optional parameter.
     */
    function maybeDoSomething(required, optional = '', node = undefined) {}
    ```
- rest参数，应当是参数列表中的最后一个参数，使用方法如下：
    ```javascript
    /**
     * @param {!Array<string>} array This is an ordinary parameter.
     * @param {...number} numbers The remainder of arguments are all numbers.
     */
    function variadic(array, ...numbers) {}
    ```
- 定义通用函数时应使用@template。
- Spread operator，使用方法如下：
    ```javascript
    function myFunction(...elements) {}
    myFunction(...array, ...iterable, ...generator());
    ```
### 字符串
- 使用单引号
- 连接过个字符串或者字符串中含有单引号或者字符串占多行，应当使用template string(用反引号'\`'包含起来)，占据多行的时候，可以不考虑缩进，如下：
    ```javascript
    function arithmetic(a, b) {
      return `Here is a table of arithmetic operations:
    ${a} + ${b} = ${a + b}
    ${a} - ${b} = ${a - b}
    ${a} * ${b} = ${a * b}
    ${a} / ${b} = ${a / b}`;
    }
    ```
- 不要使用续行符，用字符串的加法运算代替，如下：
    ```javascript
    const longString = 'This is a very long string that far exceeds the 80 ' +
        'column limit. It does not contains long stretches of spaces since ' +
        'the concatenated strings are cleaner.';
    ```
### 数字
使用二进制时，前缀0x, 0o, 0b应小写。

### 控制结构
- 推荐使用for-of而不是for-in，使用方法如下：
    ```javascript
    for (let value of iterable) {
      console.log(value);
    }
    for (let [key, value] of iterable) {
      console.log(value);
    }
    ```
- 出现异常时，throw Error对象，而不是字符串等对象，实例化Error对象时用new。如果catch里面没有语句，应当注明原因。
- switch，没有break，return，throw的语句需要注明，即使没有代码，default也要包含，如下：
    ```javascript
    switch (input) {
      case 1:
      case 2:
        prepareOneOrTwo();
      // fall through
      case 3:
        handleOneTwoOrThree();
        break;
      default:
        handleLargeNumber(input);
    }
    ```
### this
只在类的方法和构造函数里使用this关键字，或者在类方法和构造函数里定义arrow function时使用。
### 禁止使用的特性
with, 动态执行代码(eval)，primitive object wrapper(Boolean, Number, String, Symbol)

## 命名
- 只能使用ASCII的字符和数字，下划线和美元符号用得比较少。
- package都是lowerCamelCase，比如my.exampleCode.deepSpace
- class, interface, record, typedef names 都是UpperCamelCase。
- 方法名都是lowerCamelCase，private方法的名称后面跟一个下划线。
- enum是UpperCamelCase， 里面元素是CONSTANT_CASE。
- 常量用CONSTANT_CASE，应当是@const static属性或者module-local const，不过并不是const定义的就一定是常量，必须是内容不可更改的常量用这种命名方式，下面有个对比：
    ```javascript
    const NUMBER = 5;
    const /** Set<String> */ mutableCollection = new Set();
    ```
- 非constant的域是lowerCamelCase，私有的后面要加下划线
- 参数名是lowerCamelCase
- 局部变量是lowerCamelCase，除了module-local的常量外(用上面常量的命名方法)。注意，函数内的常量仍然是lowerCamelCase。
- template的参数应使用一个单词或字符，并且都是大写字母

## JSDoc
一般的形式如下：
```javascript
/**
 * Multiple lines of JSDoc text are written here,
 * wrapped normally.
 * @param {number} arg A number to do something to.
 */
function doSomething(arg) { … }

/** @const @private {!Foo} A short bit of JSDoc. */
this.foo_ = foo;
```

JSDoc使用markdown编写，所以可以包含HTML标签。比如：
```javascript
/**
 * Computes weight based on three factors:
 *  - items sent
 *  - items received
 *  - last timestamp
 */
```

一般来说，一个JSDoc tag占一行，但一些不需要额外信息的tag(比如@private, @const, @final, @export等)可以和其他tag结合占一行。

字符串占多行时，后面的行缩进四个空格(使用@fileoverview时不要缩进)，如下：
```javascript
/**
 * Illustrates line wrapping for long param/return descriptions.
 * @param {string} foo This is a param with a description too long to fit in
 *     one line.
 * @return {number} This returns something that has a description too long to
 *     fit in one line.
 */
exports.method = function(foo) {
  return 5;
};
```

file-level注释，使用@fileoverview，描述一下文件的内容和依赖信息。换行后不需要缩进。

类的注释
```javascript
/**
 * A fancier event target that does cool things.
 * @implements {Iterable<string>}
 */
class MyFancyTarget extends EventTarget {
  /**
   * @param {string} arg1 An argument that makes this more interesting.
   * @param {!Array<number>} arg2 List of numbers to be processed.
   */
  constructor(arg1, arg2) {
    // ...
  }
};

/**
 * Records are also helpful.
 * @extends {Iterator<TYPE>}
 * @record
 * @template TYPE
 */
class Listable {
  /** @return {TYPE} The next item in line to be returned. */
  next() {}
}
```

enum和typedef注释
```javascript
/**
 * A useful type union, which is reused often.
 * @typedef {!Bandersnatch|!BandersnatchType}
 */
let CoolUnionType;


/**
 * Types of bandersnatches.
 * @enum {string}
 */
const BandersnatchType = {
  /** This kind is really frumious. */
  FRUMIOUS: 'frumious',
  /** The less-frumious kind. */
  MANXOME: 'manxome',
};
```

函数和方法注释，参数和返回值的类型都要注明(足够简单的话，可以省略)。重载的方法应使用@override，省略参数和返回值的描述。
```javascript
/** This is a class. */
class SomeClass extends SomeBaseClass {
  /**
   * Operates on an instance of MyClass and returns something.
   * @param {!MyClass} obj An object that for some reason needs detailed
   *     explanation that spans multiple lines.
   * @param {!OtherClass} obviousOtherClass
   * @return {boolean} Whether something occurred.
   */
  someMethod(obj, obviousOtherClass) { ... }

  /** @override */
  overriddenMethod(param) { ... }
}

/**
 * Demonstrates how top-level functions follow the same rules.  This one
 * makes an array.
 * @param {TYPE} arg
 * @return {!Array<TYPE>}
 * @template TYPE
 */
function makeArray(arg) { ... }
```

匿名函数可以添加行内注释。
```javascript
promise.then(
    (/** !Array<number|string> */ items) => {
      doSomethingWith(items);
      return /** @type {string} */ (items[0]);
    });
```

属性注释，private属性如果容易理解，可不添加注释。Public exporter常量和属性一样需要注释。如果很容易就可以知道常量的类型，就不需要给它添加类型的annotation。
```javascript
/** My class. */
class MyClass {
  /** @param {string=} someString */
  constructor(someString = 'default string') {
    /** @private @const */
    this.someString_ = someString;

    /** @private @const {!OtherType} */
    this.someOtherThing_ = functionThatReturnsAThing();

    /**
     * Maximum number of things per pane.
     * @type {number}
     */
    this.someProperty = 4;
  }
}

/**
 * The number of times we'll try before giving up.
 * @const
 */
MyClass.RETRY_COUNT = 33;
```
### Type annotation
Type annotation，对@param, @return, @this, @type来说是必须的，对@const和@export是可选的。Type annotation放在大括号里面。

"!"和"?"分别表示不为null和可以是null。
Primitive types(undefined, string, number, boolean, symbol, function(...):...)和record({foo:string, bar:number})默认是不能为null的，不需要再这些类型前面添加"!"。对象类型(Array, Element, MyClass等)默认是可以为null的。除了primitive和record以外，其他类型都要显式表明"!"和"?"。

模板的参数类型应当标清楚，如下：
```javascript
const /** !Object<string, !User> */ users = {};
const /** !Array<string> */ books = [];
const /** !Promise<!Response> */ response = ...;

const /** !Promise<undefined> */ thisPromiseReturnsNothingButParameterIsStillUseful = ...;
const /** !Object<string, *> */ mapOfEverything = {};
```

可见性的标签，@private, @package, @protected，局部变量不应使用这些标签。

其他的一些标签，@author(不推荐使用), @see Link, 更多参考[JSDoc tags](https://google.github.io/styleguide/jsguide.html#appendices-documentation-annotations)。
