# 递归、闭包、原型、继承

本文主要讲解、理清一些函数常用的知识点：递归、闭包是什么、闭包使用场景、什么是原型和原型链、如何实现继承、继承的原理。

#### 递归

函数的递归就是在函数中调用自身

举一个实例，著名斐波那契数列如何求得，问题是这样的：

1. 第一个月初有一对刚诞生的兔子
2. 第二个月之后（第三个月初）它们可以生育
3. 每月每对可生育的兔子会诞生下一对新兔子
4. 兔子永不死去

定义出来的数列是

![](../assets/2017_10_01.jpg)

我们需要求得 n 月有多少对兔子，通过递归算法可以求得

``` javascript
function fn(n) {
  return n < 2 ? 1 : fn(n - 1) + fn(n - 2)
}
var count = fn(30);
console.log(count);
```

#### 闭包

什么是闭包？闭包就是函数，它可以继承并访问自身所被声明的函数作用域内的变量。

``` javaScript
function fn1 () {
  var a = 'hello'
  function fn2 () {
    console.log(a)
  }
}
fn1() // 其中 fn2 就是闭包函数
```

#### 闭包的使用场景

闭包有很多使用场景，以下举例：

###### 1. 私有变量

``` javascript
function Person(){    
  var name = "default";       
  return {    
    getName : function(){    
      return name;    
    },    
  	setName : function(newName){    
      name = newName;    
    }    
  }    
};
var person = Person()
console.log(person.getName()) // default
person.setName('xiaomuchen')
console.log(person.getName()) // xiaomuchen
var person2 = Person()
console.log(person2.getName()) // default
```

上述函数，使用闭包创建私有变量 name，变量不可被外部直接操作、获取，只能通过返回的接口控制。


###### 2. 匿名自执行函数

比如在开发页面时，需要在页面初始化时，你需要立即做一些操作，那么可以在页面中使用`匿名自执行函数`，它会在 JS 引擎读取到这部分代码时就立即执行。

``` javascript
// 在 title 上添加页面打开时间
(function(){
  var openTime = new Date()
  document.title = document.title + openTime
})();  
```

#### 原型、原型链、继承

###### \_\_proto\_\_ 和 prototype 的区别

1.JavaScript 中每一个对象都拥有原型链（\_\_proto\_\_）指向其构造函数的原型（prototype）

```javascript
var a = {}
a.__proto__ === Object.prototype // true

function Person () {}
var p = new Person()
p.__proto__ === Person.prototype // true
```

2.JavaScript 中每一个函数都拥有原型（prototype），原型也是一个对象，这个对象包括：原型链、原型方法（属性）、函数构造，同理它的原型链指向其构造函数的原型

```javascript
function Person () {}
Person.prototype.getName = function () {}
Object.getOwnPropertyNames(Person.prototype) // ["constructor", "getName"]
Person.prototype.__proto__ === Object.prototype // true
```

3.当访问一个对象上的属性时，先尝试访问自身上的属性，如果没有，则访问自身原型上的属性。如果没有，则通过原型上的原型链，访问其构造函数的原型上的属性，如果还是没有则继续向上查找直到 Object.prototype，而 Object.prototype 是一个没有 \_\_proto\_\_ 原型链的对象，则查询到此为止。

###### 继承

javascript 函数通过原型和原型链实现继承

``` javascript
function superA (name) {
  this.name = name
}
superA.prototype.getName = function () {
  console.log(this.name)
}
function subA (name) {
  superA.call(this, name) // 继承属性
}
subA.prototype = new superA() // 继承方法

var a1 = new subA('xiaomuchen')
a1.getName() // xiaomuchen
```

上述代码，描述了一个函数的经典继承，其工作原理是这样的：

1. 声明父类 superA、子类 subA
2. 重写子类 subA 的原型，指向 superA 的实例
3. 当实例化 a1 时，a1.\_\_proto\_\_ => subA.prototype => new superA()，a1 的构造函数是 superA
4. 同时，运行 subA 也就是 superA.call(this, 'xiaomuchen')，其中 this 指向 a1 所以 a1 继承了 name 属性
5. 这样子类 subA 的实例 a1 就继承了 superA 的方法和属性

#### 总结

本文概括了递归、闭包、原型、继承，理清这些基本的概念，有助于你接纳更多的东西，我们会在下一个章节对函数进行更深入的讨论。

------

作者：肖沐宸，[github](https://github.com/cheogo/learn-javascript)。