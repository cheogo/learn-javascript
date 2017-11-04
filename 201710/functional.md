# JavaScript 函数式编程

函数式编程是一种编程范式，记得在刚学编程时从`面向过程编程` 转换到 `面向对象编程` 时的触动，了解 `函数式编程` 或许会给你一个最初的惊喜。函数式编程是一个很大的命题，在本文中将介绍几个基本概念：`纯函数`、`柯里化（curry）`、`组合（compose）`、`容器（container）`、`函子（functor）`，希望能激起你对它的兴趣。

#### 如何实现链式调用

先让我们忘掉上面那些奇怪的概念，让我们看一个贯彻全文的实例，如何实现一个链式调用。

``` javaScript
var Container = function(x) {
  this.__value = x;
}

Container.of = function(x) { return new Container(x); };

Container.prototype.map = function(f){
  return Container.of(f(this.__value))
}
```

上述代码实现了一个简单的链式调用，让我们看看如何使用它

``` javaScript
Container.of(3); // Container {__value: 3}
Container.of(4); // Container {__value: 4}

var add1 = function (num) { return num + 1 };
var add2 = function (num) { return num + 2 };

Container.of(3).map(add1).map(add2) // Container {__value: 6}
Container.of(4).map(add2).map(add2).map(add2) // Container {__value: 10}
```

在这个实例中出现的 `Container` 是一个容器，通过 `Container.of` 来实例化保存值到 `this.__value` 。`add1`、`add2` 都是 `纯函数`，我们通过 `map` 函数来操作容器内的值，我们把 `Container` 看作数据结构，这种数据结构可以通过 `map` 操作，那么它就叫 `functor`。


#### 纯函数

> 什么是纯函数：纯函数是这样一种函数，即相同的输入，永远会得到相同的输出，而且没有任何可观察的副作用。

比如 `slice` 和 `splice`，这两个函数的作用并别无二致。但是我们说 `slice` 符合纯函数的定义是因为对相同的输入它保证能返回相同的输出。而 `splice` 的调用却会产生可观察到的副作用，这个数组被永久地改变了。

``` javaScript 
var xs = [1,2,3,4,5];

// 纯的
xs.slice(0,3); // => [1,2,3]
xs.slice(0,3); // => [1,2,3]

// 不纯的
xs.splice(0,3); // => [1,2,3]
xs.splice(0,3); // => [4,5]
```

在函数式编程中，我们尽量杜绝 `splice` 这种会改变数据的函数。我们追求的是 `slice` 那种可靠的，每次都能返回同样结果的函数。

再看另一个例子

``` javaScript 
// 不纯的
var num_1 = 1
var add1 = function (num) { return num + num_1 };

// 纯的
var add1 = function (num) { return num + 1 };
```

在不纯的版本中，`add1` 的结果将取决于 `num_1` 这个可变变量的值。换句话说，它取决于`系统状态`（system state）。因为它引入了外部的环境，从而增加了`认知负荷`（cognitive load）。这种依赖状态是影响系统复杂度的罪魁祸首，不仅让它变得不纯，而且导致每次我们思考整个软件的时候都痛苦不堪。

为什么要使用纯函数呢？举例容易看到的好处：1. 可缓存性，因为纯函数对于相同的输入有相同的输出，所以纯函数是可以缓存运算结果的；2. 可移植性，因为不会受`环境变量`等外部状态的影响，可以方便移植；3. 可测试性，无需配置外部变量，一个输入一个输出，直接断言；等等。

有哪些不纯的情况呢？1. IO 操作，你不知道你读取的内容会是怎样；2. 接口请求，你确定返回的内容是什么；3. dom 操作，引起了副作用；4. 甚至连 console.log 都是不纯的，因为它有副作用；等等。对于不纯的函数我们尽量把它控制在可控范围内发生，这个会在文章后面提到。

#### 函数柯里化

> 什么是柯里化（curry）？curry 的概念很简单，只传递给函数一部分参数来调用它，让它返回一个函数去处理剩下的参数

简单的实例：
``` javascript
var add = function (x, y) {  return x + y; }
add(1, 2)   // 3
add(10, 1)  // 11
add(10, 2)  // 12
add(10, 3)  // 13

// curry
var add = function(x) {
  return function(y) {
    return x + y;
  };
};

var increment = add(1);
var addTen = add(10);

increment(2); // 3
addTen(1); // 11
addTen(2); // 12
addTen(3); // 13
```

我们把 `add` 函数通过柯里化变成了接受部分参数并返回一个处理剩余函数且返回结果的函数。在实际环境中我们可能用到 `ramda` 这样的库来帮助我们实现柯里化。

``` javascript
var R = require('ramda');
var add = function (x, y) { return x + y; }
var addTen = R.curry(add)(10)

addTen(1); // 11
addTen(2); // 12
```

柯里化是函数式编程的工具，他能实现预加载函数、分步取值、避免重复传参、锁定函数运行环境等等功能。

#### 函数组合

这就是组合（compose）

``` javascript
// 简单实现，复杂实现可以传递多个函数用于组合
var compose = function(f,g) {
  return function(x) {
    return f(g(x));
  };
};
```

组合多个函数生成一个新的函数，并且函数从右往左运行。

``` javascript
var double = function (num) { return num * 2 }
var add =  R.curry(function (x, y) { return x + y; })

var price = compose(double, add(10)) // 通过成本获取商品价格

price(10) // 40
price(20) // 60
```

通过函数组合我们可以，一次性的合并多个处理函数，并且可以方便的改变函数的执行顺序。

#### 容器和函子（functor）

让我们回顾开头的例子

``` javascript
var Container = function(x) {
  this.__value = x;
}

Container.of = function(x) { return new Container(x); };

Container.prototype.map = function(f){
  return Container.of(f(this.__value))
}
```

现在我们转换角度，把调用 `Container.of` 返回的对象看作一种数据结构 `Container {__value: 3}` ，这种数据结构只能使用 `map` 方法进行操作，类似这样的数据结构被称为 `functor`。

这样做的好处是什么呢？我们能在不离开 `容器（Container）` 的情况下操作容器里面的值，操作完成之后又放回容器。我们可以不断的进行这一操作，就像 `组合函数` 一样。这是一种抽象，我们让容器保存值，并且请求容器通过 `map` 里的函数去操作值。


#### 总结

在文章中，提到了 `纯函数`、`柯里化（curry）`、`组合（compose）`、`容器（container）`、`函子（functor）`，不要看它们很遥远其实已经或多或少出现在我们身边。举个例子：尖头函数。

```javascript
const curryAdd = x => y => x + y
const compose = (f, g) => x => f(g(x))
const double = num => num * 2
const price = y => compose(double, curryAdd(10))(y)

console.log(price(0)) // 20
```

仅仅几行代码就可以体验 `curry` 和 `compose` 工具，如果在阅读本文之后对这种编程范式感兴趣的的话，或者对 `不纯操作的处理`、`处理错误和流程` 等延伸内容好奇的话，可以阅读这篇文章[《JS 函数式编程指南》](https://www.gitbook.com/book/llh911001/mostly-adequate-guide-chinese/details)。

------

作者：肖沐宸，[github](https://github.com/cheogo/learn-javascript)。
