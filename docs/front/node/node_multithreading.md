# javaScript多线程实现

我们都知道，JavaScript是单线程语言。在单线程里，js通过“时间循环”，“线程池”来现实多线程操作。可是你知道，js其实也提供了多线程api吗？

Node.js从版本12开始引入worker_threads模块，允许创建Worker线程。这里每个 Worker 线程都是一个独立的 Node.js 进程。worker的设计是用于处理 CPU 密集型任务，拥有独立V8实例，不参与主线程的事件循环。

Worker 线程拥有自己的事件循环，可以处理自己的 I/O 操作和定时器。通过 postMessage 方法在 Worker 线程和主线程之间传递消息，也可以使用 worker_data 和 parentPort 进行通信。

Worker 线程之间不共享内存，需要通过消息传递进行通信。

这里有个 Worker 线程的使用示例：

主进程代码：

```
const { Worker } = require('worker_threads');

const worker = new Worker('./fibonacci.js', { workerData: 10 });

worker.on('message', (result) => {
  console.log(`Fibonacci of 10 is ${result}`);
});

worker.on('error', (error) => {
  console.error(error);
});

worker.on('exit', (code) => {
  console.log(`Worker stopped with exit code ${code}`);
});
```

Worker线程代码（fibonacci.js）
```
const { parentPort } = require('worker_threads');

function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

parentPort.on('message', (n) => {
  const result = fibonacci(n);
  parentPort.postMessage(result);
});

```


### 浏览器环境实现多线程

说完了在node.js中如何实现并发的，其实我们使用js语言大部分是做前端开发的，那么在浏览器环境中，有没有类似的api能帮助我们实现多线程并行操作呢？

浏览器中，也提供了个一些API用来实现多线程操作。

首先是 Web Workers ，它是一种在后台线程中运行 JavaScript 代码的方法，不会阻塞主 UI 线程。也是用来执行计算密集型或耗时的任务。主线程和 Worker 线程通过消息传递进行通信。

```
// 主线程
const worker = new Worker('worker.js');
worker.postMessage('Start processing');
worker.onmessage = function(event) {
  console.log('Result from worker:', event.data);
};

// Worker 线程 (worker.js)
self.onmessage = function(event) {
  const result = doSomeHeavyProcessing(event.data);
  self.postMessage(result);
};

```

除了 Web Worker，还有一种 Service Workers ，这是一种特殊的 Worker，主要用于拦截和处理网络请求，实现离线功能、请求拦截和响应修改等。


## JavaScript 和其他多线程的异同

最后，我们来讲讲在 JavaScript 中的多线程和其他语言的多线程的异同。

首先，node.js明明是单线程，这里的 Worker 线程能够和主线程并行执行（注意是并行而不是并发），就是实现了真正的多线程的能力，那么为什么还说node.js是单线程呢？

1. 尽管有 Worker 线程的存在，Node.js 的核心仍然是单线程的。主线程负责处理所有的 I/O 操作和事件循环。
2. Worker 线程主要用于处理计算密集型任务，它们不参与事件循环，也不直接处理 I/O 操作。它们的存在是为了补充主线程的能力，而不是改变 Node.js 的单线程本质。
3. Worker 线程拥有独立的线程上下文，不共享主线程的内存空间，Worker 线程与主线程之间的通信通过消息传递机制实现，不会出现同时操作同一块内存的情况。