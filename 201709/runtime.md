# JavaScript 语法解析、AST、V8、JIT

#### JavaScript 是如何执行的

对于常见编译型语言（例如：Java）来说，编译步骤分为：词法分析->语法分析->语义检查->代码优化和字节生成。

对于解释型语言（例如 JavaScript）来说，通过词法分析 -> 语法分析 -> 语法树，就可以开始解释执行了。

![](../assets/2017_09_01.png)

具体过程是这样的：  

1.词法分析是将字符流(char stream)转换为记号流(token stream)

```
NAME "AST"  
EQUALS  
NAME "is Tree"  
SEMICOLON 
```

2.语法分析成 AST (Abstract Syntax Tree)，你可以在这里试试 [http://esprima.org/](http://esprima.org/demo/parse.html#)  

3.预编译，当JavaScript引擎解析脚本时，它会在预编译期对所有声明的变量和函数进行处理！并且是先预声明变量，再预定义函数！

4.解释执行，在执行过程中，JavaScript 引擎是严格按着作用域机制（scope）来执行的，并且 JavaScript 的变量和函数作用域是在定义时决定的，而不是执行时决定的。JavaScript 中的变量作用域在函数体内有效，无块作用域；

``` javascript
function func(){
    for(var i = 0; i < array.length; i++){  
      //do something here.  
    }
    //此时 i 仍然有值，及 i == array.length  
    console.log(i); // 但在 java 语言中，则无效
}
```
JavaScript 引擎通过作用域链（scope chain）把多个嵌套的作用域串连在一起，并借助这个链条帮助 JavaScript 解释器检索变量的值。这个作用域链相当于一个索引表，并通过编号来存储它们的嵌套关系。当 JavaScript 解释器检索变量的值，会按着这个索引编号进行快速查找，直到找到全局对象（global object）为止，如果没有找到值，则传递一个特殊的 undefined 值。
``` javascript
var scope = "global";
scopeTest();
function scopeTest(){  
    console.log(scope);  
    var scope = "local";  
    console.log(scope); 
}
打印结果：undefined，local；
```

#### V8、JIT

我们常说的 V8 是 Google 发布的开源 JavaScript 引擎，采用 C++ 编写。SpiderMonkey（Mozilla，基于 C）、Rhino（Mozilla，基于 Java），而 Nodejs 依赖于 V8 引擎开发，接下来的内容是 JavaScript 在 V8 引擎中的运行状态，而类似的 JavaScript 现代引擎对于这些实现大同小异。

在本文的开头提到了编译型语言，解释型语言。JavaScript 是解释型语言，在生成 AST 之后，就开始一边解释，一边执行，但是有个弊端，当某段代码被多次执行时，它就有了可优化的空间（后面提到），而不用一次次的去重复之前的解释执行。
编译型语言如 JAVA，可以在执行前就进行优化编译，但是这会耗费大量的时间，显然不适用于 Web 交互。

于是就有了，JIT（Just-in-time），JIT 是两种模式的混合。

![](../assets/2017_09_02.jpg)

它是如何工作的呢：

1.在 JavaScript 引擎中增加一个监视器（也叫分析器）。监视器监控着代码的运行情况，记录代码一共运行了多少次、如何运行的等信息，如果同一行代码运行了几次，这个代码段就被标记成了 “warm”，如果运行了很多次，则被标记成 “hot”。

2.（基线编译器）如果一段代码变成了 “warm”，那么 JIT 就把它送到编译器去编译，并且把编译结果存储起来。比如，监视器监视到了，某行、某个变量执行同样的代码、使用了同样的变量类型，那么就会把编译后的版本，替换这一行代码的执行，并且存储。

3.（优化编译器）如果一个代码段变得 “hot”，监视器会把它发送到优化编译器中。生成一个更快速和高效的代码版本出来，并且存储。例如：循环加一个对象属性时，假设它是 INT 类型，优先做 INT 类型的判断

4.（去优化）可是对于 JavaScript 从来就没有确定这么一说，前 99 个对象属性保持着 INT 类型，可能第 100 个就没有这个属性了，那么这时候 JIT 会认为做了一个错误的假设，并且把优化代码丢掉，执行过程将会回到解释器或者基线编译器，这一过程叫做去优化。

###### 优化代码图例：

’hot‘ 代码

![](../assets/2017_09_03.jpg)

优化前

![](../assets/2017_09_04.jpg)

优化后

![](../assets/2017_09_05.jpg)


#### 总结

明白一些基本原理能拓展出更多的东西，比如：

1.var a = 10; var b = 20; ==> var a=10, b=20; 这些改代码的好处是什么，如何从原理解释？

2.JavaScript 的函数和变量是在什么时候声明的，函数声明和函数表达式的区别？

3.如何通过编译器的优化原理，如何提高 JavaScript 的执行效率？  

------

作者：肖沐宸，[github](https://github.com/cheogo)。

参考文章-02
