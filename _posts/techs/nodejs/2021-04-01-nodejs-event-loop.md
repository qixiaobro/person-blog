---
title: Node.js事件循环官方文档翻译 
date: 2021-04-01 13:26:49
permalink: /docs/techs/nodejs-event-loop
key: nodejsloop
sharing: true
show_author_profile: true
show_subscribe: true
articles:
  excerpt_type: html
sidebar:
  nav: tech-nav
categories:
  - 前端
  - 技术文章
tags:
  - Node.js
  - Event loop
---

[官方原文](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)
## 什么是事件循环？
事件循环使Node.js能够执行非阻塞的I/O操作--尽管事实上JavaScript是单线程的--通过尽可能地将操作卸载到系统内核。<br /><!--more-->
<br />由于大多数现代内核是多线程的，它们可以处理在后台执行的多个操作。当这些操作之一完成后，内核会告诉Node.js，以便适当的回调可以被添加到轮询队列中，最终被执行。我们将在本专题的后面进一步详细解释这一点。<br />

## 事件循环的解释
当Node.js启动时，它初始化事件循环，处理所提供的输入脚本（或掉入REPL，这在本文中没有涉及），这可能会进行异步API调用，安排定时器，或调用`process.nextTick()`，然后开始处理事件循环。<br />
<br />下图显示了事件循环的操作顺序的简化概述。<br />
<br />   ┌───────────────────────────┐<br />┌──>│timer 计时器 │<br />│ └─────────────┬─────────────┘<br />│ ┌─────────────┴─────────────┐<br />│ │ pending callbacks 待定回调<br />│ └─────────────┬─────────────┘<br />│ ┌─────────────┴─────────────┐<br />│ idle, prepare闲置，准备 │<br />│ └─────────────┬─────────────┘ ┌───────────────┐<br />│┌─────────────┴─────────────┐ │传入<br />│ │poll 轮询 │<─────┤连接， │<br />│ └─────────────┬─────────────┘ │ 数据，等 │。<br />│ ┌─────────────┴─────────────┐ └───────────────┘<br />│ │check 检查 │<br />│ └─────────────┬─────────────┘<br />│ ┌─────────────┴─────────────┐<br />└──┤┤close callbacks 关闭回调 │<br />   └───────────────────────────┘<br />

> 每个盒子将被称为事件循环的一个 "阶段"。


<br />每个阶段都有一个FIFO的回调队列来执行。虽然每个阶段都有自己的特殊性，但一般来说，当事件循环进入一个给定的阶段时，它将执行该阶段特有的任何操作，然后执行该阶段队列中的回调，直到队列被耗尽或回调的最大数量被执行。当队列用尽或达到回调限制时，事件循环将进入下一阶段，如此反复。<br />
<br />由于这些操作中的任何一个都可以安排更多的操作，而且在轮询阶段处理的新事件由内核排队，所以在轮询事件被处理的同时可以排队。因此，长期运行的回调可以使轮询阶段的运行时间远远超过定时器的阈值。更多细节请见计时器和投票部分。<br />
<br />在Windows和Unix/Linux的实现之间有一点差异，但这对这个演示并不重要。最重要的部分在这里。实际上有七八个步骤，但我们关心的--Node.js实际使用的--是上面这些。<br />

## 阶段概述
**timers**：该阶段执行由 `setTimeout()` 和 `setInterval()` 安排的回调。<br />**pending callbacks**：执行I/O回调，推迟到下一个循环迭代。<br />**idle, prepare**：只在内部使用。<br />**poll**：检索新的I/O事件；执行I/O相关的回调（几乎所有的回调，除了关闭回调、由定时器安排的回调和`setImmediate()`）；节点会在适当的时候在这里阻塞。<br />**check**：`setImmediate()`回调在这里被调用。<br />**close callbacks**：一些关闭回调，例如`socket.on('close', ...)`。<br />
<br />在事件循环的每次运行之间，Node.js检查它是否在等待任何异步I/O或计时器，如果没有，就干净地关闭。<br />

## 各个阶段的详细情况
### timers
计时器指定了所提供的回调可能被执行的阈值，而不是一个人希望它被执行的确切时间。定时器回调将在指定的时间过后**尽早**运行，然而，**操作系统的调度或其他回调的运行**可能会延迟它们。<br />

> 从技术上讲，轮询阶段控制着定时器的执行时间。


<br />例如，假设你安排一个超时器在100毫秒的阈值后执行，然后你的脚本开始异步读取一个文件，这需要95毫秒。
```javascript
const fs = require('fs');

function someAsyncOperation(callback) {
  // 假设这需要95ms来完成
  fs.readFile('/path/to/file', callback)。
}

const timeoutScheduled = Date.now();

setTimeout(() => {
  const delay = Date.now() - timeoutScheduled;

  console.log(`${delay}ms已过，因为我被安排了`)。
}, 100);

//进行someAsyncOperation，需要95毫秒才能完成
someAsyncOperation(() => {
  const startCallback = Date.now();

  //做一些需要10毫秒的事情......
  while (Date.now() - startCallback < 10) {
    //什么都不做
  }
});
```

<br />当事件循环进入轮询阶段时，它有一个空队列（`fs.readFile()`还没有完成），所以它将等待剩余的ms数，直到达到最快定时器的阈值。当它在等待95毫秒通过时，`fs.readFile()`完成了对文件的读取，其需要10毫秒完成的回调被添加到轮询队列中并执行。当回调完成后，队列中没有更多的回调，所以事件循环将看到最早的定时器的阈值已经达到，然后包回定时器阶段来执行定时器的回调。在这个例子中，你会看到在定时器被安排和它的回调被执行之间的总延迟将是105ms。<br />

> 为了防止轮询阶段饿死事件循环，libuv（实现Node.js事件循环和平台所有异步行为的C库）在停止轮询更多事件之前也有一个硬性上限（取决于系统）。



### pending callbacks（待定回调）
这个阶段执行一些系统操作的回调，如TCP错误的类型。例如，如果一个TCP套接字在试图连接时收到`ECONNREFUSED`，一些*nix系统希望等待报告这个错误。这将在**pending callbacks**阶段被排队执行。<br />

### poll（轮询）
轮询阶段有两个主要功能。<br />

1. 计算它应该为I/O阻塞和轮询多长时间，然后处理轮询队列中的事件。
1. 当事件循环进入轮询阶段，并且没有安排定时器时，将发生两种情况之一。


<br />如果轮询队列不是空的，事件循环将遍历其回调队列，同步执行这些回调，直到队列被耗尽，或者达到系统相关的硬限制。<br />
<br />如果轮询队列是空的，就会发生以下两种情况之一。<br />
<br />如果脚本已经被`setImmediate()`安排好了，事件循环将结束轮询阶段，继续进入检查阶段，执行那些安排好的脚本。<br />
<br />如果脚本没有被`setImmediate()`安排，事件循环将等待回调被添加到队列中，然后立即执行它们。<br />
<br />一旦轮询队列为空，事件循环将检查时间阈值已经达到的计时器。如果一个或多个定时器准备好了，事件循环将绕回到定时器阶段，执行这些定时器的回调。<br />

### check （检查）
这个阶段允许人在轮询阶段完成后立即执行回调。如果轮询阶段变得空闲，并且脚本已经用`setImmediate()`排队，事件循环可以继续到检查阶段而不是等待。<br />
<br />`setImmediate()`实际上是一个特殊的定时器，在事件循环的一个单独阶段运行。它使用libuv的API，**在轮询阶段完成后安排回调执行**。<br />
<br />一般来说，随着代码的执行，事件循环最终会进入轮询阶段，在那里它将等待一个进入的连接、请求等。然而，如果一个回调已经用`setImmediate()`安排好了，并且轮询阶段变得空闲，它将结束并继续到检查阶段，而不是等待轮询事件。<br />

### close callback（关闭回调）
如果一个套接字或句柄被突然关闭（比如`socket.destroy()`），"`close`"事件将在这个阶段被发出来。否则，它将通过`process.nextTick()`发出。<br />

## `setImmediate()` vs `setTimeout()`
`setImmediate()`和`setTimeout()`是相似的，但根据它们被调用的时间，表现出不同的方式。<br />

- `setImmediate()`被设计为在当前轮询阶段完成后执行一个脚本。
- `setTimeout()`安排在最小阈值（ms）过后运行一个脚本。


<br />计时器的执行顺序将取决于它们被调用的环境。如果两者都是从主模块中调用的，那么计时将受到进程性能的约束（可能受到机器上运行的其他应用程序的影响）。<br />
<br />例如，如果我们运行下面这个不在**I/O周期内**的脚本（即主模块），两个定时器的执行顺序是不确定的，因为它受到进程性能的约束。<br />

```javascript
// timeout_vs_immediate.js
setTimeout(() => {
  console.log('timeout');
}, 0);

setImmediate(() => {
  console.log('immediate');
});
```
```shell
$ node timeout_vs_immediate.js
timeout
immediate

$ node timeout_vs_immediate.js
immediate
timeout
```

<br />**然而，如果你在一个I/O周期内移动这两个调用，**`**setImmediate**`**总是先执行。因为I/O回调是在轮询阶段执行，当回调执行完毕之后队列为空，发现存在**`**setImmediate**`**的回调就会进入check阶段，执行完毕后，再进入timer阶段。**<br />

```shell
// timeout_vs_immediate.js
const fs = require('fs');

fs.readFile(__filename, () => {
  setTimeout(() => {
    console.log('timeout');
  }, 0);
  setImmediate(() => {
    console.log('immediate');
  });
});
```
```shell
$ node timeout_vs_immediate.js
immediate
timeout

$ node timeout_vs_immediate.js
immediate
timeout
```

<br />与`setTimeout()`相比，使用`setImmediate()`的主要优点是，如果在一个I/O周期内安排，setImmediate()将总是在任何定时器之前执行，与有多少个定时器有关。<br />

## `process.nextTick()`
### 
### 了解` process.nextTick()`

<br />你可能已经注意到，尽管`process.nextTick()`是异步API的一部分，但在图中没有显示。这是因为 `process.nextTick()` 在技术上不是事件循环的一部分。相反，`nextTickQueue`将在当前操作完成后被处理，与事件循环的当前阶段无关。在这里，一个操作被定义为从底层的C/C++处理程序，以及处理需要执行的JavaScript的一个过渡。<br />
<br />回顾我们的图表，**任何时候你在某个阶段调用**`**process.nextTick()**`**，所有传递给**`**process.nextTick()**`**的回调都会在事件循环继续之前解决。**这可能会造成一些不好的情况，因为它允许你通过递归调用`process.nextTick()`来 "饿死 "你的I/O，从而阻止事件循环到达轮询阶段。<br />

### 为什么会允许这样做？

<br />为什么会在Node.js中包含这样的东西？部分原因是一种设计理念，即API应该始终是异步的，即使在没有必要的地方。以这个代码片断为例。<br />

```javascript
function apiCall(arg, callback) {
  if (typeof arg !== 'string')
    return process.nextTick(callback,new TypeError('argument should be string'));
}
```
这个片段做了一个参数检查，如果不正确，它将把错误传递给回调。API最近更新了，允许向`process.nextTick()`传递参数，允许它将回调后传递的任何参数作为回调的参数来传播，所以你不必嵌套函数。<br />
<br />我们正在做的是把错误传回给用户，但只是在我们允许用户的其他代码执行之后。通过使用`process.nextTick()`，我们保证apiCall()总是在用户的其他代码之后，在事件循环被允许进行之前运行其回调。为了实现这一点，JS调用堆栈被允许解开，然后立即执行所提供的回调，这允许人们对`process.nextTick()`进行递归调用而不会达到RangeError。超过了v8的最大调用堆栈大小。<br />
<br />这种理念会导致一些潜在的问题情况。以这个片段为例。
```javascript
let bar;

// this has an asynchronous signature, but calls callback synchronously
function someAsyncApiCall(callback) { callback(); }

// the callback is called before `someAsyncApiCall` completes.
someAsyncApiCall(() => {
  // since someAsyncApiCall hasn't completed, bar hasn't been assigned any value
  console.log('bar', bar); // undefined
});

bar = 1;
```
用户将`someAsyncApiCall()`定义为有一个异步签名，但它实际上是同步操作的。当它被调用时，提供给`someAsyncApiCall()`的回调在事件循环的同一阶段被调用，因为`someAsyncApiCall()`实际上并没有异步地做什么。结果，回调试图引用bar，尽管它可能还没有这个变量的作用域，因为脚本还没有能够运行完成。<br />
<br />通过将回调放在`process.nextTick()`中，脚本仍然有能力运行到结束，允许所有的变量、函数等在回调被调用之前被初始化。这也有一个好处，就是不允许事件循环继续。在允许事件循环继续之前，提醒用户注意错误可能是很有用的。下面是使用`process.nextTick()`的前一个例子。<br />

```javascript
let bar;

function someAsyncApiCall(callback) {
  process.nextTick(callback);
}

someAsyncApiCall(() => {
  console.log('bar', bar); // 1
});

bar = 1;
```

<br />下面是另一个现实世界的例子。<br />

```javascript
const server = net.createServer(() => {}).listen(8080);

server.on('listening', () => {});
```
当只传递一个端口时，该端口立即被绑定。所以，`'listening'`回调可以立即被调用。问题是`.on('listening')`回调到那时还没有被设置。<br />
<br />为了解决这个问题，`'listening'`事件被排在nextTick()中，以允许脚本运行到完成。这使得用户可以设置任何他们想要的事件处理程序。<br />

### `process.nextTick()` vs `setImmediate()`
就用户而言，我们有两个调用是相似的，但它们的名字却令人困惑。<br />

- `process.nextTick()`在同一阶段立即触发
- `setImmediate()`在事件循环的下一次迭代或 "tick "时触发。

`process.nextTick()`比`setImmediate()`更快启动，但这是过去的产物，不太可能改变。进行这种转换会破坏npm上很大一部分的软件包。每天都有更多的新模块被添加进来，这意味着我们每天都在等待，更多的潜在破坏发生。虽然它们令人困惑，但名称本身不会改变。<br />

> 我们建议开发者在所有情况下使用`setImmediate()`，因为它更容易推理。



### 为什么使用 `process.nextTick()`？
主要有两个原因。<br />

1. 允许用户处理错误，清理任何当时不需要的资源，或者在事件循环继续之前再次尝试请求。
1. 有时有必要让回调在调用栈解开后但在事件循环继续前运行。


<br />一个例子是为了配合用户的期望。简单的例子。
```javascript
const server = net.createServer();
server.on('connection', (conn) => { });

server.listen(8080);
server.on('listening', () => { });
```
假设`listen()`在事件循环的开头运行，但监听回调被放置在`setImmediate()`中。除非传递一个主机名，否则与端口的绑定将立即发生。为了使事件循环继续进行，它必须进入轮询阶段，这意味着有一个非零的机会，即可能已经收到一个连接，允许连接事件在监听事件之前被触发。<br />另一个例子是运行一个函数构造器，比如说，继承自`EventEmitter`，它想在构造器中调用一个事件。
```javascript
const EventEmitter = require('events');
const util = require('util');

function MyEmitter() {
  EventEmitter.call(this);
  this.emit('event');
}
util.inherits(MyEmitter, EventEmitter);

const myEmitter = new MyEmitter();
myEmitter.on('event', () => {
  console.log('an event occurred!');
});
```
你不能立即从构造函数中发出一个事件，因为脚本还没有处理到用户为该事件指定一个回调的程度。因此，在构造函数本身中，你可以使用process.nextTick()来设置一个回调，在构造函数完成后发出事件，这样就能得到预期的结果。
```javascript
const EventEmitter = require('events');
const util = require('util');

function MyEmitter() {
  EventEmitter.call(this);

  // use nextTick to emit the event once a handler is assigned
  process.nextTick(() => {
    this.emit('event');
  });
}
util.inherits(MyEmitter, EventEmitter);

const myEmitter = new MyEmitter();
myEmitter.on('event', () => {
  console.log('an event occurred!');
});
```


