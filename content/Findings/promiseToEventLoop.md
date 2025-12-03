---
title: "Promise 与 Event Loop 的联系与理解"
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

* **Promise** —— 代表一个“未来会结束”的异步操作，提供了状态管理、链式调用、错误传播等语义，是现代异步编程的基石
* **Event Loop（事件循环）** —— JavaScript 运行时真正的调度中心，它决定了同步代码、Promise 回调（微任务）、setTimeout/IO 等（宏任务）到底在什么时机被推入调用栈执行

这两者紧密协作：
- Promise 的 `.then` / `.catch` / `await` 后续全部属于**微任务（microtask）**
- 微任务与宏任务（macrotask）在 Event Loop 中有严格的优先级：**每执行完一个宏任务，必须先清空全部微任务，才会进行渲染并进入下一个宏任务**

掌握 Promise 的状态机细节 + Event Loop 的微/宏任务执行顺序，就等于拿到了 JavaScript 异步世界的完整“时序地图”

---

执行顺序表

| 执行顺序 | 类型         | 典型代表                                   | 触发时机 |
|---------|--------------|--------------------------------------------|----------|
|         | 同步代码(包含Promise executor)     | `console.log`、变量赋值、函数调用          | 立即执行 |
|         | 微任务（microtask） | `.then/.catch`、`await` 后续、`queueMicrotask`、`MutationObserver` | 当前宏任务结束后、渲染前全部清空 |
|         | 渲染（浏览器） | 页面重绘、布局                             | 微任务清空后（可能跳过） |
|         | 宏任务（macrotask） | `setTimeout`、`setInterval`、`I/O`、`UI 事件` | 每轮 Event Loop 执行一个 |

---

Promise 核心机制

Promise 的三个状态
```js
pending → fulfilled   (成功，返回 value)
      → rejected     (失败，返回 reason)
```

构造器 executor 是同步执行的

```js
new Promise((resolve, reject) => {
  console.log('executor start');   // 立刻打印
  setTimeout(() => resolve(100), 3000);
  console.log('executor end');     // 立刻打印
});
```

**为什么必须同步？**  
为了让你可以在创建 Promise 时立刻做初始化、参数校验、抛错等操作

内部槽位

| 槽位名                     | 内容说明                                 |
|----------------------------|------------------------------------------|
| `[[PromiseState]]`         | pending / fulfilled / rejected           |
| `[[PromiseResult]]`        | 成功值或失败原因                         |
| `[[PromiseFulfillReactions]]` | 存放所有 onFulfilled 回调              |
| `[[PromiseRejectReactions]]`  | 存放所有 onRejected 回调               |

`resolve/reject` 第一次调用成功后，后续全部忽略

executor 抛错 = 自动 reject

```js
new Promise(() => { throw new Error('boom'); })
  .catch(e => console.log(e.message)); // boom
```

thenable 与扁平化

```js
Promise.resolve({
  then(onFulfilled) {
    setTimeout(() => onFulfilled(42), 1000);
  }
}).then(console.log); // 1秒后打印 42
```

规范会：
1. 取出 `x.then`
2. 调用 `then.call(x, resolvePromise, rejectPromise)`
3. `resolvePromise/rejectPromise` 是包装过的“只允许调用一次”的函数

这就是链式调用不会无限嵌套的根本原因

---

then / catch / finally 与链式调用

```js
Promise.resolve(1)
  .then(x => x + 1)                    // 返回普通值 → 新 Promise fulfilled(2)
  .then(x => new Promise(r => setTimeout(() => r(x + 1), 100))) // 返回 Promise → 等待并采用
  .then(console.log)                   // 打印最终结果
  .catch(console.error)
  .finally(() => console.log('done'));
```

`.catch` = `.then(undefined, onRejected)`  
`.finally` 不传值，异常会继续向下抛

---

async / await 本质

```js
async function foo() {
  console.log('1');
  const a = await 111;
  console.log('2');
  const b = await 222;
  console.log('3');
  return a + b;
}
```

V8 编译后等价于：

```js
function foo() {
  console.log('1');
  return Promise.resolve(111).then(a => {
    console.log('2');
    return Promise.resolve(222).then(b => {
      console.log('3');
      return a + b;
    });
  });
}
```

**结论**：
- `async function` 永远返回 Promise
- 每一个 `await` = 把后面的代码切出去，包装成 `.then()` 回调
- `await` 右侧不是 Promise 也会被 `Promise.resolve()` 包装

---

Promise 静态方法全家桶

| 方法                | 行为                                         | 最佳实践场景                         |
|---------------------|----------------------------------------------|--------------------------------------|
| `Promise.all`       | 全部成功才成功，任一失败立即失败             | 并发请求多个必须成功的接口           |
| `Promise.allSettled`| 永远成功，返回每个结果的状态和值             | 批量操作、统计上报、不怕部分失败     |
| `Promise.any`       | 第一个成功就成功，全失败才失败               | 多 CDN 加载资源，谁快用谁            |
| `Promise.race`      | 第一个有状态（成功或失败）就返回             | 超时控制、防超时                     |

```js
// 并发 + 结构赋值
const [user, posts] = await Promise.all([
  fetch('/user'), 
  fetch('/posts')
]);

// 请求超时模板
const withTimeout = (p, ms) => Promise.race([
  p,
  new Promise((_ , reject) => setTimeout(() => reject('超时'), ms))
]);
```

---

Event Loop 完整流程（浏览器环境）

1. 执行一个宏任务（script 整体、setTimeout 回调、点击事件等）
2. 执行完当前宏任务 → 调用栈清空
3. 清空当前所有微任务（`.then`、`await` 后续、`queueMicrotask` 等）
4. （浏览器）可能进行渲染更新
5. 从宏任务队列取下一个宏任务 
6. 重复以上步骤

**Node.js 小区别**
- `process.nextTick` 比微任务还优先
- `setImmediate` 是 Node 特有的宏任务

---

经典综合例题

```js
console.log('start');

setTimeout(() => console.log('timeout'), 0);

Promise.resolve()
  .then(() => console.log('p1'))
  .then(() => console.log('p2'));

queueMicrotask(() => console.log('micro'));

(async () => {
  console.log('async start');
  await 1;
  console.log('await after');
})();

console.log('end');
```

输出顺序：
```
start
async start
end
p1
micro
await after
p2
timeout
```

---

常见陷阱 & 最佳实践

| 陷阱                               | 正确做法                                 |
|------------------------------------|------------------------------------------|
| 误以为 `await` 阻塞主线程          | 不阻塞，只是暂停当前 async 函数         |
| 多个独立请求写成串行 `await`       | 并行发起 → `Promise.all` 聚合           |
| 微任务无限自我调度                 | 避免 `then(() => then(...))` 无限循环   |
| 没处理 rejected Promise            | 全局监听 `unhandledrejection`           |
| 返回 thenable 导致意外扁平化       | 明确知道自己在干什么                     |


