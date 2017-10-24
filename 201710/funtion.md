# 递归、闭包、原型、继承

本文为高阶函数一章做铺垫，主要讲解：递归、闭包是什么、闭包使用场景、什么是原型和原型链、如何实现继承、继承的原理，如果对于这些函数的基本概念已经掌握，可以跳过。

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

javascript 函数通过原型和原型链实现继承，原型是 javaScript 函数上 prototype 属性的值，原型是一个对象，包含原型方法（属性）、构造函数和原型链（\_\_proto\_\_）。

原型链指向构造函数的原型，每个对象（注意是对象！JavaScript 的函数是第一公民所以函数也是对象）都有\_\_proto\_\_ 指向其构造函数的原型。如果访问一个对象的属性，会先尝试在对象上搜索，如果没有再通过原型链向上查找。


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
0. 继承 subA.\_\_proto\_\_ => superA.prototype
1. 创建 var a1 = {}
2. 使用 `new` 关键字的时候，把原型链指向原型 a1.\_\_proto\_\_ => subA.prototype
3. 使用构造函数 subA 初始化 a1，subA.call(a1, 'xiaomuchen') => superA.call(a1, 'xiaomuchen')，a1 获得属性 name = 'xiaomuchen'
4. a1 访问 getName 方法时，因为本身没有，通过 a1.\_\_proto\_\_ 查找到 subA.prototype，subA 没有，再通过 subA.\_\_proto\_\_ 找到原型 superA.prototype 上


#### 总结

本文概括了递归、闭包、原型、继承，理清这些基本的概念，有助于你容易接纳更多的东西。

------

作者：肖沐宸，[github](https://github.com/cheogo/learn-javascript)。