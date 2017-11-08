# JavaScript 异步流程控制

本文主要讲解 JavaScript 在异步流程控制中的一些实践、容错以及复杂异步环境下我们该如何去处理。

#### 发展历史

简要的提及一下，异步流程控制的发展历史大概是 `callback hell` => `Promise` => `Generator` => `async/await`

ES6 中 `Promise` 是通过 `.then().then().catch()` 的方式来解决 `callback` 多层嵌套的问题。但 `Promise` 依然是异步执行的，这时候 TJ 的 [co](https://github.com/tj/co)，通过 `Generator` 实现了异步代码的同步化。这个模式和 ES7 中的 `async/await` 类似。

``` javascript
function A() {
  // async get dataA
  function B(dataA) {
    // async get dataB
    function C(dataB) {
    }
  }
}

Promise(A).then(B).then(C).catch(err => console.log(err))

co(function *() {
  var dataA = yield A()
  var dataB = yield B(dataA)
  var dataC = yield C(dataB)
})

async () => {
  const dataA = await A()
  const dataB = await B(dataA)
  const dataC = await C(dataB)
}
```

#### 使用

首先是语法糖支持情况，你可以使用下面命令行查看当前 node 版本对于 ES6/ES7 的支持。目前大多浏览器是不支持新语法的，如果你当前环境不支持新语法，你可以使用 [bable](https://github.com/babel/babel)、 [co](https://github.com/tj/co)、 [Promise](https://github.com/then/promise)、 [bluebird](https://github.com/petkaantonov/bluebird) 等开源项目来扩展功能。

`$ node --v8-options | grep harmony`

对了如果你还对这些新语法的使用方式和 API 陌生的话，建议看看 [《ECMAScript 6 入门》](http://es6.ruanyifeng.com/#README) 这本书，下面的内容，假定你对基本的使用已经有所了解，我们开始正篇。


#### Promise 实践和容错

之前当面试官的时候，如果面试对象经常使用 ES6，我会喜欢问一个问题：`假设你的移动端页面上有头部、中部、底部三部分数据需要并发的去请求 api 拿到返回数据，你会怎么处理？用 Promise 如何实现？如果其中一个 API 出了错误怎么容错？`

1.第一个问题很简单，依次执行三个异步请求函数，在获取到数据后执行渲染函数填充到页面上

2.第二个问题，其实也没多绕，你可以同时执行三个 Promise 函数，也可以打包成 Promise.all() 一并执行，显然对于这种并发执行的异步函数 Promise.all() 更符合程序设计。

``` javascript
const render = log = console.log
const asyncApi = (num) => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (typeof num !== 'number') {
        reject('param error')
      }
      num += 10
      resolve(num)
    }, 100);
  })
}

asyncApi(0).then(render).catch(log)  // 10
asyncApi(5).then(render).catch(log)  // 15
asyncApi(10).then(render).catch(log) // 20

Promise.all([asyncApi(0), asyncApi(5), asyncApi(10)]).then(render).catch(log) // [ 10, 15, 20 ]
```

3.无论怎样，我会把面试者引导到 Promise.all() 上，这时候我会抛出问题 `如果其中一个 API 出了错误怎么容错？`

``` javascript
asyncApi(0).then(render).catch(log)       // 10
asyncApi(false).then(render).catch(log)   // param error
asyncApi(10).then(render).catch(log)      // 20

Promise.all([asyncApi(0), asyncApi(false), asyncApi(10)]).then(render).catch(log) // param error
```

对比发现，Promise 之间互不影响。但由于 Promise.all() 其实是将传入的多个 Promise 打包成一个，任何一个地方出错了都会直接抛出异常，导致不执行 `then` 直接跳到了 `catch`，丢失了成功的数据。

4.解决方式是使用 `resolve` 传递错误，then 环节去处理

```javascript
const render = log = console.log
const asyncApi = (num) => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (typeof num !== 'number') {
        resolve({ err: 'param error' }) // 修改前：reject('param error')
      }
      num += 10
      resolve({ data: num }) // 修改前：resolve(num)
    }, 100);
  })
}

Promise.all([asyncApi(0), asyncApi(false), asyncApi(10)]).then(render).catch(log) 
// [ { data: 10 }, { err: 'param error' }, { data: 20 } ]，这时候就可以区分处理了
```


#### 复杂环境

我们假设一个如下的复杂场景，异步请求之间相互依赖。仅仅用 `Promise` 来实现，会不停的调用 `then`、 `return` 并且创建匿名函数。

``` javascript
// 流程示意图
//          data            data1
// asyncApi -----> asyncApi -----> render/error
//     10 + data            data2           data3
//          -----> asyncApi -----> asyncApi -----> render/error

asyncApi(0).then(data => {
  return Promise.all([asyncApi(data.data), asyncApi(10 + data.data)])
}).then(([data1, data2]) => {
  render(data1)
  return asyncApi(data2.data)
}).then(render).catch(log)
```

而如果加上 `async/await` 来改写它，就可以完全按同步的写法来获取异步数据，并且语义清晰。

``` javascript
const run = async () => {
  let data = await asyncApi(0)
  let [data1, data2] = await Promise.all([asyncApi(data.data), asyncApi(10 + data.data)])
  render(data1)
  let data3 = await asyncApi(data2.data)
  render(data3)
}
run().catch(log)
```

或许你觉得差不了太多，那我再改一下，现在我们看到 `data3` 是需要 `data2` 作为函数参数才能获取，假如：获取 `data3` 需要 `data` 和 `data2` 呢？

你会发现 `Promise` 的写法隔离了环境，如果需要 `data` 这个值，那就要想办法传递下去或保存到其他地方，而 `async/await` 的写法就不需要考虑这个问题。

#### 总结

在本文的前半部分简单介绍了流程控制的发展历史和如何使用这些新的语法糖，后半部分我们聊到了 `Promise` 和 `async/await` 如何去实现复杂的异步流程环境，并满足容错和可读性。

做一个有追求的程序员，在实际项目中多去思考`容错`和`可读性`，相信代码质量会有不错的提升。

------

作者：肖沐宸，[github](https://github.com/cheogo/learn-javascript)。