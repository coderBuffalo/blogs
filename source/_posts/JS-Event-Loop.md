---
title: JS-Event-Loop
date: 2019-05-26 11:40:03
tags: js
---

事件循环的顺序，决定了 JavaScript 代码的执行顺序。它从 script (整体代码) 开始第一次循环。之后全局上下文进入函数调用栈。直到调用栈清空(只剩全局)，然后执行所有的 Micro Task。当所有可执行的 Micro Task 执行完毕之后。循环再次从 Macro Task 开始，找到其中一个任务队列执行完毕，然后再执行所有的 Micro Task，在执行 Micro Task、Macro Task 的时候同样遵循 Event Loop 原则，就这样一直循环下去。

在讨论 Micro Task、Macro Task 之前我们先来看一个例子，分析以下代码并思考 log 打的印顺序
> 示例代码

``` js
console.log('script start');

setTimeout(() => {
  console.log('setTimeout 1');
});

Promise.resolve()
.then(() => {
  console.log('promise 1');
})
.then(() => {
  console.log('promise 2');
});

console.log('script end');
```
> 运行结果为：

```
script start
script end
promise 1
promise 2
setTimeout 1
```

先明白几个概念

## Event Loop
Event Loop：事件循环

我们知道 JavaScript 是单线程语言，一心不能二用，也就是说它只能把一件事干完才能去干另一件事，但前端的一些任务是非常耗时的，比如网络请求，定时器和事件监听，如果让它们和别的任务一样，都老老实实的排队等待执行的话，执行效率会非常的低，甚至导致页面的假死。所以为了保证使用流畅，浏览器采用了 Event Loop 事件循环系统来管理，把那些耗时的任务放入 task queues 栈中，等主程序执行完再执行 task queues 中的任务，通常使用回调函数监听 task queues 中的任务状态。

在其它语言中可以迟延执行，比如 C 语言的 sleep(3) 可以迟延 3 秒执行后边的逻辑，但是在 JavaScript 中是没有这样的操作（除了 alert、confrim、prompt 和异步 xhr，官方说这些是历史遗留的错误设计😂），只能使用异步方式去模拟，模拟的实际也不是真正意义上的 sleep 效果。

## Micro Task、Macro Task
想要搞明白这个问题，首先要了解 Event Loop 下的 Micro Task 和 Macro Task，在运行主程序时会把异步逻辑根据情况推到 Micro Task 或者 Macro Task 栈中，具体规则如下

以下情况会推到 Micro Task 栈中
- cess.nextTick
- mise
- ect.observe
- ationObserver

下情况会推到 Macro Task 栈中
- setTimeout
- setInterval
- setImmediate
- I/O
- UI render

然后再来看实例代码的实际执行逻辑
1. 执行 console.log('script start') 直接输出
2. 执行 setTimeout 把回调函数放入 Macro Task 栈中
3. 执行 Promise 把两个 then 的回调函数放入 Micro Task 栈中
4. 执行 console.log('script end') 直接输出
5. 执行 Micro Task 中所有任务
    1.   i. 执行 console.log('promise 1') 直接输出
    2.   ii. 执行 console.log('promise 2') 直接输出
6. 执行 Macro Task 中所有任务
    1.   执行 console.log('setTimeout 1') 直接输出
