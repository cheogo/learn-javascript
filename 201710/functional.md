# JavaScript 函数式编程

#### 纯函数

> 纯函数是这样一种函数，即相同的输入，永远会得到相同的输出，而且没有任何可观察的副作用。

比如 `slice` 和 `splice`，这两个函数的作用并别无二致。但是我们说 `slice` 符合纯函数的定义是因为对相同的输入它保证能返回相同的输出。而 `splice` 的调用却会产生可观察到的副作用，这个数组被永久地改变了。

``` javaScript 
var xs = [1,2,3,4,5];

// 纯的
xs.slice(0,3); // => [1,2,3]
xs.slice(0,3); // => [1,2,3]
xs.slice(0,3); // => [1,2,3]

// 不纯的
xs.splice(0,3); // => [1,2,3]
xs.splice(0,3); // => [4,5]
xs.splice(0,3); // => []
```

在函数式编程中，我们尽量杜绝 `splice` 这种会改变数据的函数。我们追求的是 `slice` 那种可靠的，每次都能返回同样结果的函数。

再看另一个例子

``` javaScript 
// 不纯的
var minimum = 20;
var checkAge = function(age) {
  return age >= minimum;
};

// 纯的
var checkAge = function(age) {
  const minimum = 20;
  return age >= minimum;
};
```

在不纯的版本中，`checkAge` 的结果将取决于 `minimum` 这个可变变量的值。换句话说，它取决于`系统状态`（system state）。这一点令人沮丧，因为它引入了外部的环境，从而增加了`认知负荷`（cognitive load）。这种依赖状态是影响系统复杂度的罪魁祸首，不仅让它变得不纯，而且导致每次我们思考整个软件的时候都痛苦不堪。

#### 函数柯里化

什么是柯里化（currying）？柯里化是把接受多个参数变换成接受一个单一参数并返回接受余下参数且返回结果的新函数的技术。

// TODO

------

作者：肖沐宸，[github](https://github.com/cheogo/learn-javascript)。
