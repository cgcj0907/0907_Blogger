---
title: "Promise 与 Event Loop 的对比与理解"
date: 2025-11-29
tags: ["JavaScript", "异步", "事件循环"]
draft: false
cover:
  image: "/covers/promiseToEventLoop.webp"  
  relative: false
ShowToc: true
----------------------------------

# Promise 与 Event Loop：分步深入指南

# 简介

JavaScript 是单线程语言，却要处理大量异步操作（如网络请求、定时器、用户交互等）。为了既不阻塞主线程、又能保证回调按预期顺序执行，语言和运行时共同提供了两套核心机制：

1. **Promise** —— 代表一个“未来会结束”的异步操作，提供了状态管理、链式调用、错误传播等语义，是现代异步编程的基石
2. **Event Loop（事件循环）** —— JavaScript 运行时真正的调度中心，它决定了同步代码、Promise 回调（微任务）、setTimeout/IO 等（宏任务）到底在什么时机被推入调用栈执行

这两者紧密协作：
- Promise 的 `.then` / `.catch` / `await` 后续全部属于**微任务（microtask）**
- 微任务与宏任务（macrotask）在 Event Loop 中有严格的优先级：**每执行完一个宏任务，必须先清空全部微任务，才会进行渲染并进入下一个宏任务**

掌握 Promise 的状态机细节 + Event Loop 的微/宏任务执行顺序，就等于拿到了 JavaScript 异步世界的完整“时序地图”

---


## 1 Promise

* 表示**未来某个时刻**可用的值或失败原因
* 三个状态：`pending`（挂起） → `fulfilled`（成功） 或 `rejected`（失败）
* 构造器形式：`new Promise(executor)`，返回立即可用的 Promise 对象

简短代码：

```js
const p = new Promise((resolve, reject) => {
  // executor 同步运行
  if (ok) resolve(value);
  else reject(new Error('err'));
});
```

---

## 2 逐步引入：从同步到微任务
1. **同步执行**：普通函数、直接 `console.log`，在调用栈中执行完才会继续
2. **创建 Promise**：构造器里的 `executor` 在创建时**同步执行**；但 `resolve/reject` 会**把后续回调放到微任务队列**
3. **注册回调（then/catch）**：回调不会立刻执行，而是进入 **microtask**（微任务）队列
4. **宏任务（setTimeout 等）**：在微任务全部清空后才会执行下一个宏任务
5. **await**：本质上是 `Promise.then` 的语法糖，`await` 后续继续执行排入 microtask

---

## 3 executor

### 3.1 executor 是同步执行的

```js
const p = new Promise((resolve, reject) => {
  console.log('executor start');
  // 同步代码
  resolve(1);
  console.log('executor end');
});
```

输出会有 `executor start`、`executor end` 在 Promise 构造时立即打印

**原因**：语言规范要求 `executor` 在 `new Promise` 时被立即调用（同步）。这使得创建 Promise 时可以马上执行需要的初始化操作（例如立刻触发 I/O 请求、立刻检查参数并拒绝等）

### 3.2 内部槽位

Promise 内部维护两个重要槽位：

* `[[PromiseState]]`：`pending` | `fulfilled` | `rejected`
* `[[PromiseResult]]`：成功值或拒绝原因

另外有一个回调列表，用来存储 `.then/.catch/.finally` 注册的处理器，当状态从 `pending` 变为 `fulfilled|rejected` 时，将这些处理器排入微任务并按注册顺序执行

### 3.3 resolve/reject 的行为要点

1. **一次性**：一旦 `resolve/reject` 被调用并把状态从 `pending` 锁定到 `fulfilled` 或 `rejected`，后续任何对 `resolve/reject` 的调用都被忽略
2. **可能异步调用**：你可以在 executor 中同步或异步调用 `resolve`，都没问题。关键是：`resolve` 改变状态后，**注册的回调会作为微任务被调度**（不是立即同步调用）

示例：

```js
new Promise((resolve) => {
  resolve(42); // 同步调用 resolve
  console.log('after resolve');
}).then(v => console.log('then', v));

// 输出顺序： after resolve, then 42
```

原因：`then` 回调被放入 microtask，仍会在当前同步栈清空后执行

### 3.4 thenable

规范有一个**Promise Resolve Procedure**：当 `resolve(x)` 中的 `x` 是一个对象并且有 `then` 方法（thenable），Promise 将尝试“采用”该 thenable 的状态

1. 取 `then = x.then`（取属性时可能抛错）
2. 如果 `then` 是函数，调用 `then.call(x, resolvePromise, rejectPromise)`（这里的 `resolvePromise/rejectPromise` 是包装过的、可只调用一次的函数）
3. 若该 thenable 最终 fulfilled/rejected，则外层 Promise 相应地 fulfilled/rejected（这就是扁平化）

这就解释了为什么 `Promise.resolve(promise2)` 会“扁平化”并等待 `promise2`

**注意**：实现必须保证 `resolvePromise/rejectPromise` 只会被调用一次（规范称 `called` 标志），避免 thenable 的多次调用或意外行为破坏状态。实现细节会处理取 then 时的异常、then 调用时的异常等

### 3.5 executor 中抛错 -> Promise 被拒绝

如果 `executor` 同步抛出异常，构造出来的 Promise 会被自动 `reject(err)`：

```js
const p = new Promise((resolve, reject) => {
  throw new Error('boom');
});

p.catch(e => console.log('caught', e.message)); // 'caught boom'
```

这与同步 try/catch 行为一致：构造 Promise 时若发生同步错误，Promise 进入 `rejected`。

### 3.6 多次调用 resolve/reject

```js
new Promise((resolve, reject) => {
  resolve(1);
  resolve(2); // 被忽略
  reject(new Error('no')); // 也被忽略
}).then(v => console.log(v)); // 1
```

规范与主流实现都保证只取第一次

---

## 4 then / catch / finally 与链（扁平化再回顾）

* `.then(onFulfilled, onRejected)` 返回一个全新的 Promise：

  * 如果 `onFulfilled` 返回普通值 `v`，新 Promise `resolve(v)`
  * 如果返回一个 Promise（或 thenable），新 Promise 会 **等待并采用** 那个 Promise 的状态（扁平化）
  * 如果回调抛错，则新 Promise 被 `reject(err)`

示例：

```js
Promise.resolve(1)
  .then(v => v + 1)        // 返回 2 -> 下一个 then 收到 2
  .then(v => Promise.resolve(v + 1)) // 返回 Promise -> 扁平化
  .then(v => console.log(v)); // 3
```

`catch` 等价于 `.then(undefined, onRejected)`

`finally`：注册一个无论成功或失败都执行的回调。它不会接收 value/reason（除非 finally 内抛错会影响链）。通常用于清理

---

## 5 async / await（把 Promise 转为更易读的形式）

`async function` 总是返回 Promise。`await expr` 等同于 `Promise.resolve(expr).then(...)` 的行为：

* `await` 右侧如果是普通值，会被 `Promise.resolve` 包装并立即微任务化（继续执行放入微任务）
* 如果是被拒绝的 Promise，`await` 会抛出异常，需要 `try/catch` 捕获

示例：

```js
async function foo() {
  const v = await Promise.resolve(1);
  return v + 1;
}

foo().then(console.log); // 2
```

实现细节：`await` 暂停函数的后续执行，并将继续的逻辑放入 microtask（同 `then`）。因此 `await` 不会阻塞主线程，只暂停当前 async 的执行流程

---

## 6 Event Loop 与微任务/宏任务的配合

### 6.1 简化版 Event Loop 步骤

1. 取出并执行 **一个宏任务（macrotask）**，比如：主脚本、`setTimeout` 回调、UI 事件回调等
2. 当前宏任务执行完（调用栈清空）之后，**清空所有 microtasks**（执行到队列为空）。microtasks 包括：`Promise.then` 回调、`queueMicrotask`、`MutationObserver` 回调
3. 浏览器可能进行渲染（paint / layout）
4. 开始下一轮循环（回到第 1 步）

### 6.2 为什么 `Promise.then` 比 `setTimeout(..., 0)` 先执行？

因为 `.then` 入 microtask 队列，而 `setTimeout` 回调入宏任务队列。每个宏任务结束后都会先清空所有微任务，再才去执行下一个宏任务（如 `setTimeout`）

示例：

```js
console.log('script start');
setTimeout(() => console.log('timeout'), 0);
Promise.resolve().then(() => console.log('promise'));
console.log('script end');

// 输出： script start, script end, promise, timeout
```

### 6.3 microtask“饿死”宏任务

如果微任务队列不断被微任务补充，例如：

```js
(function loop() {
  Promise.resolve().then(loop);
})();
```

那么宏任务可能长时间无法运行（页面渲染或 `setTimeout` 迟迟不执行），这是一种不良模式

### 6.4 在 Node 中的差异要点

* `process.nextTick` 在 Node 中优先于 microtasks，它会在当前操作之后、真正的 microtasks 之前执行
* `setImmediate` 是 Node 的宏任务类型（与 `setTimeout` 不同，但仍是宏任务范畴）
---

## 7 综合示例（步骤化追踪）

代码：

```js
console.log('start');

setTimeout(() => console.log('timeout'), 0);

Promise.resolve().then(() => {
  console.log('promise1');
  return Promise.resolve().then(()=> console.log('promise1.1'));
}).then(() => console.log('promise2'));

queueMicrotask(() => console.log('queueMicrotask'));

(async function(){
  console.log('async start');
  await null;
  console.log('async after await');
})();

console.log('end');
```

**解释（分步）**：

1. 同步执行：输出 `start`
2. `setTimeout` 注册宏任务（排队）
3. `Promise.resolve().then(...)` 注册一个微任务（promise1 回调）
4. `queueMicrotask` 注册微任务
5. `async` 函数同步部分执行，输出 `async start`，`await null` 相当于 `await Promise.resolve(null)`，暂停并把后续放进微任务
6. 输出 `end`（主脚本同步完成）
7. 主脚本（当前宏任务）结束 —— 现在清空微任务：

   * 执行 `promise1`（打印 `promise1`），它内部又返回一个 `Promise.resolve().then(...)`，该 then 注册的回调 `promise1.1` 进入微任务队列的末尾；
   * 执行 `queueMicrotask`（打印 `queueMicrotask`）；
   * 执行 `async after await`（因为 await 已排入微任务，打印 `async after await`）；
   * 执行 `promise1.1`（打印 `promise1.1`）；
   * 执行 `promise2`（打印 `promise2`）
8. 当微任务清空后，执行下一个宏任务（即 `setTimeout` 的回调，打印 `timeout`）

最终常见输出序列：

```
start
async start
end
promise1
queueMicrotask
async after await
promise1.1
promise2
timeout
```

（具体顺序可能因细微实现差异而不同，但微任务优先于宏任务的大原则不变）

---

## 8 常见陷阱与调试技巧

* **误以为 `await` 阻塞线程**：实际上 `await` 把后续逻辑放到微任务，不阻塞主线程
* **串行 `await` 导致性能问题**：多个互不依赖的请求应并行启动再 `await Promise.all([...])`
* **滥用微任务会阻塞渲染**：避免在微任务里执行大量同步计算或不断自我调度微任务
* **thenable 陷阱**：外部对象可能实现 `then` 且行为异常（例如重复调用回调）——规范处理时会用“只调用一次”的保护，但编写库时要小心输入对象的 `then`
* **忘记 `catch`/全局 unhandledrejection**：若 Promise 被拒绝且无人处理，会触发全局错误，需要注意链式错误处理

---

## 9 最佳实践与速览

* **立即执行初始化放在 executor**，但尽量不要在 executor 做大量同步计算
* **尽量把异步副作用放在外层**，让 executor 只做“绑定/触发”工作：例如发起请求并在回调中 `resolve`
* **并行化独立请求**：使用 `Promise.all` 或并行创建 Promise，然后 `await` 聚合结果
* **不要在微任务里做大量工作**，以免阻塞渲染

### 速览表

| 概念  |                                         代表 | 队列/时机                 |
| --- | -----------------------------------------: | :-------------------- |
| 同步  |                       直接函数调用、`console.log` | 调用栈（立即）               |
| 微任务 | `Promise.then`、`queueMicrotask`、`await` 后续 | microtask（每个宏任务结束后清空） |
| 宏任务 |                     `setTimeout`、I/O、UI 事件 | task queue（FIFO）      |

---

## 10 结论（记忆要点）

1. executor **同步执行**，但 `resolve/reject` 导致的回调是 **微任务**
2. `await` 是语法糖，本质仍是微任务机制
3. 微任务清空优先于下一个宏任务；慎用会“饿死”宏任务
4. thenable 与 Promise Resolve Procedure 是扁平化与可组合性的关键——实现时要防护重复调用与异常

