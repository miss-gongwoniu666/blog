## 前言

现在 Node.js 对于前端越来越重要，笔者现在也感受到了学好 Node.js 可以极大的提升前端竞争力。之前也或多或少的接触过，但是没有一个系统的学习过程，所以笔者接下来打算系统的去学习 Node.js，同时希望建立一个从零到一学习 Node 的一个博客。

希望我的学习经验可以帮助到大家。

## Node.js 是什么

都 2020 年了，大家对 Node 肯定都有了初步的了解，但是我还是想先介绍一下什么是 Node

Node.js是一个基于 Chrome V8 引擎的JavaScript运行环境(runtime)。

Node不是一门语言是让js运行在后端的运行时,并且不包括javascript全集,因为在服务端中不包含DOM和BOM。Node也提供了一些新的模块例如http,fs模块等。

Node.js 使用了事件驱动、非阻塞式 I/O 的模型，使其轻量又高效。并且 Node.js 的包管理器 npm，是全球最大的开源库生态系统。

## 为什么使用要学习 Node.js

（1）首先 Node 在处理高并发，IO密集场景有明显优势。

高并发是指同一时间并发访问服务器

IO即输入输出，是指文件读写，网络操作，数据库操作等。相对的是 CPU 密集型，意思是逻辑处理运输频繁，一般指解密、加密、压缩、解压等操作。

（2）其次，现在前端开发越来越离不开 Node。日常工作中的 webpack、cli 脚手架工具等都是由 Node 编写。前端学会 Node 会极大的增加自己的话语权，提高自己的竞争力。

## 异步非阻塞 IO

传统语言例如大部分都是采用同步阻塞的方式，即当我读取某个文件的内容时，我必须等待所有文件内容读取完毕才能进行下面的内容，因此整个线程都将停止执行，造成了阻塞，极大程度的降低了执行效率。

由于 Node.js 采用异步非阻塞的模型，因此当我对读取某个文件时，将立即执行下面的代码，当文件读取完毕，将处理结果放入我们的回调函数当中，增加了执行效率。

阻塞模式下，一个线程只能处理一个 IO，如果想并发处理多个任务只能开启多个线程。而非阻塞模式下一个线程就可以处理多个 IO，CPU 的利用率可以达到最大。

## 单线程

Node.js 采用单线程模式，这也良好的适用了单线程非阻塞的模式。当 Node.js 需要处理 IO 时，会将任务交给内部的 libuv，libuv 处理完成后，通过 IO 事件驱动机制，将结果返回给 Node.js 的线程。

## 事件驱动

每一次 IO 操作完成都会触发相应的事件，由于 Node.js 是单线程，所以同一时刻只能处理一个任务，那多余的任务只能放到队列中等待被调用。而为了处理这一过程 Node.js 采用了事件循环的策略，类似于浏览器的事件循环，但有很多出入。

## 事件循环

当 Node.js 启动后，它会初始化事件循环，处理已提供的输入脚本，它可能会调用一些异步的 API、调度定时器，或者调用 process.nextTick()，然后开始处理事件循环。

下面是一张来自官网的事件循环的描述图。


![](https://gitee.com/wangyueee/tuchuang/raw/master/2020-10-11/1602424192675-image.png)


事件循环官网介绍的非常清楚，我这里把官网的内容搬运过来。

*注意：每个框被称为事件循环机制的一个阶段。*

每个阶段都有一个 FIFO 队列来执行回调。虽然每个阶段都是特殊的，但通常情况下，当事件循环进入给定的阶段时，它将执行特定于该阶段的任何操作，然后执行该阶段队列中的回调，直到队列用尽或最大回调数已执行。当该队列已用尽或达到回调限制，事件循环将移动到下一阶段，等等。

由于这些操作中的任何一个都可能调度 更多的 操作和由内核排列在**轮询**阶段被处理的新事件， 且在处理轮询中的事件时，轮询事件可以排队。因此，长时间运行的回调可以允许轮询阶段运行长于计时器的阈值时间。

### 阶段概述
- 定时器：本阶段执行已经被 setTimeout() 和 setInterval() 的调度回调函数。
- 待定回调（pending callbacks）：执行延迟到下一个循环迭代的 I/O 回调。
- idle, prepare：仅系统内部使用。
- 轮询（poll）：检索新的 I/O 事件;执行与 I/O 相关的回调（几乎所有情况下，除了关闭的回调函数，那些由计时器和 setImmediate() 调度的之外），其余情况 node 将在适当的时候在此阻塞。
- 检测（check）：setImmediate() 回调函数在这里执行。
- 关闭的回调函数（close callbacks）：一些关闭的回调函数，如：socket.on('close', ...)。

在每次运行的事件循环之间，Node.js 检查它是否在等待任何异步 I/O 或计时器，如果没有的话，则完全关闭。

### 阶段的详细概述

#### 定时器
计时器指定 可以执行所提供回调 的 阈值，而不是用户希望其执行的确切时间。在指定的一段时间间隔后， 计时器回调将被尽可能早地运行。但是，操作系统调度或其它正在运行的回调可能会延迟它们。

*注意*：**轮询** 阶段 控制何时定时器执行。

例如，假设您调度了一个在 100 毫秒后超时的定时器，然后您的脚本开始异步读取会耗费 95 毫秒的文件:

```javaScript
const fs = require('fs');

function someAsyncOperation(callback) {
  // Assume this takes 95ms to complete
  fs.readFile('/path/to/file', callback);
}

const timeoutScheduled = Date.now();

setTimeout(() => {
  const delay = Date.now() - timeoutScheduled;

  console.log(`${delay}ms have passed since I was scheduled`);
}, 100);

// do someAsyncOperation which takes 95 ms to complete
someAsyncOperation(() => {
  const startCallback = Date.now();

  // do something that will take 10ms...
  while (Date.now() - startCallback < 10) {
    // do nothing
  }
});
```

当事件循环进入 **轮询** 阶段时，它有一个空队列（此时 fs.readFile() 尚未完成），因此它将等待剩下的毫秒数，直到达到最快的一个计时器阈值为止。当它等待 95 毫秒过后时，fs.readFile() 完成读取文件，它的那个需要 10 毫秒才能完成的回调，将被添加到 **轮询** 队列中并执行。当回调完成时，队列中不再有回调，因此事件循环机制将查看最快到达阈值的计时器，然后将回到 **计时器** 阶段，以执行定时器的回调。在本示例中，您将看到调度计时器到它的回调被执行之间的总延迟将为 105 毫秒。

注意：为了防止 **轮询** 阶段饿死事件循环，libuv（实现 Node.js 事件循环和平台的所有异步行为的 C 函数库），在停止轮询以获得更多事件之前，还有一个硬性最大值（依赖于系统）。

#### 挂起的回调函数（pending callbacks）

此阶段对某些系统操作（如 TCP 错误类型）执行回调。例如，如果 TCP 套接字在尝试连接时接收到 ECONNREFUSED，则某些 *nix 的系统希望等待报告错误。这将被排队以在 **挂起的回调** 阶段执行。

#### 轮询

**轮询** 阶段有两个重要的功能：

1. 计算应该阻塞和轮询 I/O 的时间。
2. 然后，处理 **轮询** 队列里的事件。

当事件循环进入 **轮询** 阶段且 没有被调度的计时器时 ，将发生以下两种情况之一：

- 如果 **轮询** 队列 *不是空的* ，事件循环将循环访问回调队列并同步执行它们，直到队列已用尽，或者达到了与系统相关的硬性限制。

- 如果 **轮询** 队列 *是空的* ，还有两件事发生：

- - 如果脚本被 setImmediate() 调度，则事件循环将结束 **轮询** 阶段，并继续 **检查** 阶段以执行那些被调度的脚本。

- - 如果脚本 **未被** setImmediate()调度，则事件循环将等待回调被添加到队列中，然后立即执行。

一旦 **轮询** 队列为空，事件循环将检查 _已达到时间阈值的计时器_。如果一个或多个计时器已准备就绪，则事件循环将绕回计时器阶段以执行这些计时器的回调

#### 检查阶段

此阶段允许人员在轮询阶段完成后立即执行回调。如果轮询阶段变为空闲状态，并且脚本使用 setImmediate() 后被排列在队列中，则事件循环可能继续到 检查 阶段而不是等待。

setImmediate() 实际上是一个在事件循环的单独阶段运行的特殊计时器。它使用一个 libuv API 来安排回调在 轮询 阶段完成后执行。

通常，在执行代码时，事件循环最终会命中轮询阶段，在那等待传入连接、请求等。但是，如果回调已使用 setImmediate()调度过，并且轮询阶段变为空闲状态，则它将结束此阶段，并继续到检查阶段而不是继续等待轮询事件

> 以上事件循环过程的内容来自 [Node.js 官网](https://nodejs.org/zh-cn/docs/guides/event-loop-timers-and-nexttick/#timers) 

### 微任务

熟悉浏览器事件循环的都知道，每次宏任务执行完毕都会执行微任务队列里的任务。同样 Node.js 也有微任务，只不过它有两个微任务队列，分别是 process.nextTick 与 Promise。

每个阶段执行完毕都会去执行 process.nextTick 队列内的任务与 promise 队列内的任务。process.nextTick 的优先级高于 promise。

距离说明

```javaScript
Promise.resolve().then(() => console.log('promise'))

process.nextTick(() => console.log('nextTick'))

// 输出结果

// nextTick
// promise
```

以上就是 Node.js 的一些基础概念。