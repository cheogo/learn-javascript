# JavaScript 内存管理

JavaScript 具有垃圾自动回收机制（Garbage Collection）简称 GC。垃圾回收机制会中断整个代码执行，释放不可能再被使用的变量，释放内存，这个工作机制是周期性的，我们会在下文详细探讨。

#### 可释放对象

``` javascript
function fn1() {
  var obj1 = { name: 'xiaomuchen', age: '20' }
}
function fn2() {
  var obj2 = { name: 'xiaomuchen', age: '20' }
  return obj2
}
var a = fn1()
var b = fn2()
console.log(a, b) // undefined, {name: "xiaomuchen", age: "20"}
```

我们对比上面两个函数，fn1 在函数内声明变量 obj1 并且赋值，在函数执行后这个变量`便不可再访问了`，fn2 在最后把函数内的变量 obj2 返回到全局变量 b，所以 `{ name: 'xiaomuchen', age: '20' }` 这个对象（或者说 obj2）`依然可被访问`。

JavaScript 回收机制通过判断变量是否可被访问，来决定回收哪些变量。


#### 标记清除和引用计数

那么 JavaScript 是如何判断变量是否可被访问？这就要提到标记清除和引用计数。

标记清除：标记清除是目前大部分 JavaScript 引擎使用的判断方式，通过标记变量的状态来确定是否可被回收。当变量在环境中被声明时标记`进入环境`，理论上永远不要释放进入环境的变量，因为它可以在环境中的任何位置、任何时刻被访问。当环境被销毁（如函数执行完），则变量被标记`离开环境`等待回收。

``` javascript
function fn(){
  var a = { count: 10 } // 被标记，进入环境 
  var b = { count: 20 } // 被标记，进入环境
}
fn(); // 执行完毕之后 b 被标记，离开环境
```

引用计数：JavaScript 引擎维护一张`引用表`，保存内存中所有的资源的引用次数。资源被引用一次则引用 +1，资源被去掉引用或者退出变量的函数作用域时，则引用 -1，当资源的引用次数为`0`时，说明无法访问这个值，则等待回收。
（注：引用计数从 1 到 0 这个过程可能不执行，而是直接标记`可被回收`，不再进行加减运算节约开销）

``` javascript
function fn(){
  var a = { count: 10 } // 资源 { count: 10 } 被引用次数为 1
  a = { count: 20 } // 资源 { count: 20 } 被引用次数为 1，资源 { count: 10 } 被引用次数为 0，等待回收
  // do someThing
}
fn(); // 资源 { count: 20 } 被释放
```

但是引用计数存在一种`循环引用`的情况，如下例子，两个对象之间相互引用，在离开环境后对象不可访问，但由于对象的引用次数为 1，则导致不会被回收。这个例子来自《JavaScript 高级程序设计》，但我思考良久，如果引用计数把 a.param 也作为一个变量来计数，那么就没有这个问题了，引用计数实现的方式不同，产生的结果也不一样。

``` javascript
function fn(){
  var a = { count: 10 }
  var b = { count: 20 }
  a.param = b // b 的引用次数为 2
  b.param = a // a 的引用次数为 2
}
fn(); // a、b 的引用次数为 1
```

#### GC 的缺陷、分代回收和增量 GC

和其他语言一样 GC 会中断代码执行，停止其他操作。因为要遍历所有对象，回收所有不可访问对象，这个操作的耗时可能有 100ms 以上。在 V8 引擎新版本中引入了两种优化方法：1. 分代回收（Generation GC），2. 增量 GC（increment GC）

分代回收：目的是通过对象的使用频率、存在时长区分新生代与老生代对象。多回收新生代区（young generation），少回收老生代区（tenured generation），减少每次需遍历的对象，从而减少每次GC的耗时

增量 GC：把需要长耗时的遍历、回收操作`拆分运行`，减少中断时间，但是会增大上下文切换开销

#### Node.js 中的 GC 表现

当我们用 Node.js 搭建一个稳定的服务时，就需要考虑服务器内存的开销，下面一个 Node.js 内存回收执行的例子：

执行代码`node --trace_gc --trace_gc_verbose test.js`跟踪一个网络服务的 GC。

``` console.log
[41204:0x102001c00] Memory reducer: call rate 0.056, low alloc, foreground
[41204:0x102001c00] Memory reducer: started GC #1
[41204:0x102001c00] Heap growing factor 1.1 based on mu=0.970, speed_ratio=42956 (gc=675253, mutator=16)
[41204:0x102001c00] Grow: old size: 21382 KB, new limit: 33604 KB (1.1)
[41204:0x102001c00] Memory reducer: finished GC #1 (will do more)
[41204:0x102001c00]   156410 ms: Mark-sweep 27.7 (50.0) -> 21.0 (30.0) MB, 12.4 / 0.0 ms (+ 20.4 ms in 7 steps since start of marking, biggest step 4.8 ms) [Incremental marking task: finalize incremental marking] [GC in old space requested].
[41204:0x102001c00] Memory allocator,   used:  30756 KB, available: 1435612 KB
[41204:0x102001c00] New space,          used:    169 KB, available:    838 KB, committed:   1024 KB
[41204:0x102001c00] Old space,          used:  16662 KB, available:   2417 KB, committed:  19412 KB
[41204:0x102001c00] Code space,         used:   4078 KB, available:    178 KB, committed:   5120 KB
[41204:0x102001c00] Map space,          used:    642 KB, available:      0 KB, committed:   2128 KB
[41204:0x102001c00] Large object space, used:      0 KB, available: 1434571 KB, committed:      0 KB
[41204:0x102001c00] All spaces,         used:  21552 KB, available: 1438005 KB, committed:  27684 KB
[41204:0x102001c00] External memory reported:   1026 KB
[41204:0x102001c00] Total time spent in GC  : 158.6 ms
[41204:0x102001c00] Memory reducer: call rate 0.003, low alloc, foreground
```

首先我们可以看到 Node.js 区分 `New space`、`Old space` 等来划分检索空间。而提示`(+ 20.4 ms in 7 steps since start of marking, biggest step 4.8 ms)` 告诉我们这个标记的步骤分 7 步进行，耗时最长的一次时 4.8ms。这使 JavaScript 可以很好的支持开发高实时应用。

#### 总结

因为篇幅有限，留下一些小问题供大家思考：

1. 闭包一定会导致内存不可被回收？
2. 如何监控一个 Node.js 服务的内存开销，如何处理不可预知的内存泄漏？

------

作者：肖沐宸，[github](https://github.com/cheogo/learn-javascript)。


