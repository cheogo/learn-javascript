# this、apply、call、bind

#### 隐式绑定的 this

`this` 实际上是在函数被调用时绑定的，它指向什么完全取决于函数在哪里被调用。

``` javascript
var obj = {
  a: 1,
  foo: function () {
    console.log(this.a)
  }
}

var foo2 = obj.foo
obj.foo() // 1
foo2() // undefined
```

如果没有显式的指定函数的运行对象，默认的 `this` 会绑定到运行环境的全局对象上。

``` javascript
function foo() {
  console.log(this.a)
}
a = 'global'
foo() // global
```

看一个常见的例子，如果是回调函数，即使你传入了 `obj.foo` 还是会丢失 `this`。

``` javascript
function foo() {
  console.log(this.a)
}
function doFoo(fn) {
  fn()
}
var obj = { a: 1, foo: foo }
a = "global"
doFoo(obj.foo) // global
```

如果是链式调用呢，则是最后哪个对象调用了这个方法，`this` 指向谁。

``` javascript
function foo() {
  console.log(this.a);
}
var obj2 = { a: 11, foo: foo }
var obj1 = { a: 1, obj2: obj2 }
obj1.obj2.foo(); // 11
```

其实 this 的指向都有规律可言，在隐式绑定的 this 中，`this 绑定的是调用它的对像`。我们回过头解释上面例子。

在第一、第二个例子中 `foo2()` 和 `foo()` 如果理解成 `global.foo2()` 或者 `global.foo()` 在浏览器中是 `window` 那么就合情合理了。第三个例子涉及 JS 执行的原理，`doFoo` 函数在执行时会先执行 `fn = obj.foo` 然后执行 `window.fn()` 是不是又回到了前面。


#### 显式绑定的 this

隐式绑定的 `this` 能为我们带来很多灵活性，但是有时我们需要指定函数运行的 `this` ，这就需要 `apply、call、bind` 这三个绑定在 `Function.prototype` 上的函数了（关于 prototype 我们会在后面的章节提到）。

``` javascript
fun.apply(thisArg, [argsArray])
fun.call(thisArg, arg1, arg2, ...)
fun.bind(thisArg[, arg1[, arg2[, ...]]])
```

他们的语法十分相近，第一个参赛指定函数的 this 环境，后面的参赛指定函数需要的参数，最大的区别是 `bind` 不是执行 `fun` 而是返回一个函数，我们看看如何来用它们显式绑定 this。

``` javascript
var obj1 = { a: 1 }
var obj2 = { a: 11 }
var foo = function () {
  console.log(this.a)
}
a = 'global'
foo.apply(obj1)     // 1
foo.call(obj2)      // 11
foo.bind(global)()  // global
```

实际情况 `bind` 更适合用来固定 `this` 环境。

```javascript
var obj1 = { a: 1 }
var obj2 = { a: 11 }
var foo = function () {
  console.log(this.a)
}.bind(obj1)
foo.apply(obj2)     // 1
foo.call(obj2)      // 1
```

#### 补充

对于 this 的影响还有 `new` 和 `=>` 尖头函数。

`new` 关键字把 this 指向实例，这个过程发生了什么，我们会在[后面章节](http://localhost:4000/201710/function.html)讨论。

```javascript
var foo = function (a) {
  this.a = a
}
foo.prototype.sayA = function () {
  console.log(this.a)
}
var bar = new foo(2)
foo.sayA() // 2
```

尖头函数十分特别，你可以把它理解为 bind 函数的语法糖，它的 this 同外层函数的 this。

``` javascript
var obj = {
  a: 1,
  foo: function () {
    console.log(this) 
    setTimeout(() => {
      console.log(this) 
    })
  }
}
obj.foo() // obj { a: 1, foo: ƒ }, obj { a: 1, foo: ƒ }
```

#### 总结

this 是 JavaScript 的一大难点，多年经验的前端程序员都可能对这方面模糊。this 在大量的函数、类库中都有使用，理清显式绑定和隐式绑定有助于理解或书写这类函数。

------

作者：肖沐宸，[github](https://github.com/cheogo/learn-javascript)。