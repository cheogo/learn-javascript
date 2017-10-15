# Event Loop

本文以 Node.js 为例，讲解 Event Loop 在 Node.js 的实现，[原文](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)，JavaScript 中的实现大同小异。

#### 什么是 Event Loop ？

单线程的 Node.js 能够实现`无阻塞IO`的原因就是事件循环（Event Loop）。

现在大多数系统内核是多线程的，所以它们可以在后台执行多个操作，当这些操作完成时，内核就会通知 Node.js，而这些操作的回调函数被添加到事件轮询列表（poll queue），并且 Node.js 会在适当的时机执行回调函数。

#### 概览 Event Loop

当 Node.js 开始执行时，便初始化 Event Loop，执行过程中会存在许多异步操作，如：REPL、定时器（timers）、调用异步 API（请求，事件监听），`在主进程代码执行完后，便开始运行 Event Loop`。

下图描述了 Event Loop 中的各个阶段

```
   ┌───────────────────────┐
┌─>│        timers         │ 这个阶段执行 `setTimeout()` 和 `setInterval()` 中的回调函数
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     I/O callbacks     │ 这个阶段执行除了 `close` 回调函数以外的几乎所有的 I/0 回调函数
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     idle, prepare     │ 这个阶段仅仅 Node.js 内部使用
│  └──────────┬────────────┘      ┌───────────────┐
│  ┌──────────┴────────────┐      │   incoming:   │
│  │         poll          │<─────┤  connections, │ 执行队列中的回调函数、检索新的回调函数
│  └──────────┬────────────┘      │   data, etc.  │
│  ┌──────────┴────────────┐      └───────────────┘
│  │        check          │ `setImmediate()` 将在这里被调用
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
└──┤    close callbacks    │ `close` 回调函数被调用如：socket.on('close', ...)
   └───────────────────────┘
```

#### 详解 Event Loop 的各个阶段

###### timers

setTimeout() 和 setInterval() 都要指定一个运行时间，这个运行时间其实不是确切的运行时间，而是一个期望时间，Event Loop 会在 timers 阶段执行超过期望时间的定时器回调函数，但由于你不确定在其他阶段甚至主进程中的事件执行时间，所以定时器不一定会按时执行。

``` javascript
var asyncApi = function (callback) {
  setTimeout(callback, 90)
}

const timeoutScheduled = Date.now();
setTimeout(() => {
  const delay = Date.now() - timeoutScheduled;
  console.log(`${delay}ms setTimeout 被执行`); // 140ms 之后被执行
}, 100);

asyncApi(() => {
  const startCallback = Date.now();
  while (Date.now() - startCallback < 50) {
    // do nothing
  }
})
```

###### I/O callbacks 

这个阶段主要执行一些系统操作带来的回调函数，如 TCP 错误，如果 TCP 尝试链接时出现 `ECONNREFUSED` 错误 ，一些 *nix 会把这个错误报告给 Node.js。而这个错误报告会先进入队列中，然后在 I/O callbacks 阶段执行。

###### poll

poll 阶段有两个主要功能：

1. 也会执行时间定时器到达期望时间的回调函数
2. 执行事件循环列表（poll queue）里的函数

当 Event Loop 进入 poll 阶段并且没有其余的定时器，那么：
- 如果事件循环列表`不为空`，则迭代同步的执行队列中的函数。
- 如果事件循环列表`为空`，则判断是否有 `setImmediate()` 函数待执行。如果有结束 `poll` 阶段，直接到
`check` 阶段。如果没有，则等待回调函数进入队列并立即执行。

###### check

在 poll 阶段结束之后，执行 `setImmediate()`。

###### close

突然结束的事件的回调函数会在这里触发，如果 `socket.destroy()`，那么 `close` 会被触发在这个阶段，也有可能通过 `process.nextTick()` 来触发。

#### setImmediate()、setTimeout()、process.nextTick()

这里要说明一下 `process.nextTick()` 是在下次事件循环之前运行，如果把 `process.nextTick()` 和 `setImmediate()` 写在一起，那么是 `process.nextTick()` 先执行。`next` 比 `immediate` 快，官方也说这个函数命名有问题，但是因为历史存留没办法解决。

```
process.nextTick(() => {
  console.log('nextTick');
});
setImmediate(() => {
  console.log('setImmediate');
});
setTimeout(() => {
  console.log('setTimeout'); 
}, 0)

// 执行结果，nextTick, setTimeout, setImmediate
// 查看 Node.js 源码，setTimeout(fun, 0) 会转化成 setTimeout(fun, 1)，所以在这种简单的情况下，对于不同设备，setImmediate 有可能早于 setTimeout 执行。
```

#### 总结

理解事件循环，会知道 JavaScript 如何无阻塞运行的，以及它简洁的开发思路和事件驱动风格。

------

作者：肖沐宸，[github](https://github.com/cheogo/learn-javascript)。