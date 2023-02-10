- 同步异步任务执行过程

- 微任务与宏任务队列

- 事件循环机制

  

## 前言

`大家好，我是梁木由，一个有想头的前端。JavaScript知识点真是无穷无尽，今天来理解下JavaScript运行机制。`

## JavaScript单线程机制

**JavaScript是一门单线程语言**，就注定它同一时间内就只能做一件事，只能自上而下执行，那么如果上一行解析时间很长，那下面代码就会被阻塞，那对用户而言体验感是非常不友好的。于是**JavaScript**出现了**同步任务**和**异步任务**。

## JavaScript事件循环

### 同步

程序的执行顺序与任务的排列顺序是一致的、同步的。比如要烧水煮饭，需等水开了，再去煮

### 异步

在做这件事的同时，你还可以去处理其他事情。比在烧水的同时，可以去切菜

看看同步任务与异步任务的执行过程

![image-20230113102810251.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0260e9e2f0fa4b22816c20cad5cf3d60~tplv-k3u1fbpfcp-watermark.image?)
我们先来看一个问题

```
setTimeout(() => {
  console.log('setTimeout')
})

new Promise((resolve) => {
  console.log('promise')
  resolve()
}).then(() => {
  console.log('then')
})

console.log('console')
```

根据上述，那么执行结果应该是

```
promise
console
setTimeout
then
```

哎不对呀，与实际输出结果不相符

```
promise
console
then
setTimeout
```

### 宏任务与微任务

我们知道，虽然都是异步任务，但是`promise`和`setTimeout`却是不同的异步任务，异步任务有两种

-   宏任务（`script`，`setTimeout`，`setInterval`，`Ajax`，UI渲染，`I/O`，`postMessage`等）
-   微任务（`promise`，`process.nextTick`）

不同的任务也会进入不同的队列中，所以在执行异步任务时也会进入不同的事件队列

![image-20230113105003985.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/26e9e96139364cfa824c3ce7bb5d9138~tplv-k3u1fbpfcp-watermark.image?)

这样的话，就理解为什么输出结果是

```
promise
console
then
setTimeout
```

### 事件循环

那给组合一下看看整体的事件循环机制

![image-20230113112603431.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ff86e09d1e634a4a9c6dfdbe54e36b02~tplv-k3u1fbpfcp-watermark.image?)

### 总结

-   异步任务分类：宏任务，微任务
-   同步任务和异步任务分别进入不同的执行"场所"
-   先执行主线程执行栈中的宏任务
-   执行过程中如果遇到微任务，进入Event Table并注册函数，完成后移入到微任务的任务队列中
-   宏任务执行完毕后，立即执行当前微任务队列中的所有微任务（依次执行）
-   主线程会不断获取任务队列中的任务、执行任务、再获取、再执行任务也就是常说的Event Loop(事件循环)。
