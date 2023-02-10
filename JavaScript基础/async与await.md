- async 具体做了什么事情为什么会返回一个promise对象
- await 等待，到底在等什么内容
- async/await 解决了什么问题，比promise好在哪
- 掌握如何实现async/await



## 前言

上篇文章[5K字 由浅入深聊聊Promise实现原理](https://juejin.cn/post/7194257890893365308)，中讲述了Promise内部的实现原理。今天来聊聊`async`与`await`，那么async与await到底是什么呢。都说是`语法糖`，就来深入理解下async/await吧

> 来看下MDN的概念
>
> async 函数是使用`async`关键字声明的函数。async 函数是 [`AsyncFunction`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/AsyncFunction) 构造函数的实例，并且其中允许使用 `await` 关键字
>
> `await` 操作符用于等待一个 [`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise) 兑现并获取它兑现之后的值。它只能在[异步函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/async_function)或者[模块](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Modules)顶层中使用。
>
> `async` 和 `await` 关键字让我们可以用一种更简洁的方式写出基于 [`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise) 的异步行为，而无需刻意地链式调用 `promise`。

## async

`async`在字面上的意思呢，是`异步`的概念，根据MDN的概念呢，说明async声明的是一个异步构造函数，来看如下示例

```
const fn1 = async function fn(){
  return 1
}
console.log(fn1())
// Promise {<fulfilled>: 1}
```

根据上述示例内容，表述async声明了一个异步构造函数，并且调用了该函数，返回结果是一个`Promise`对象。

那问题来了，在看上述代码异步函数中return的是1，结果却是一个`Promise`对象，不急，答案来了。

如果在函数中return的不是一个promise,那么将等同于使用Promise.resolve(x)给包装起来

```
function fn() {
  return Promise.resolve(1);
}
```

> 将常规函数转换成Promise，返回值也是一个Promise对象

那这么看async与Promise有什么区别呢，看着是没什么区别，先不着急，再接着看下`await`

## await

`await`字面意思呢等待等候的意思，那到底是在等什么呢，等promise对象吗？

```
const fn1 = function fn() {
  return Promise.resolve(1);
};
async function result() {
  const r1 = await fn1();
  console.log(r1); // 1
}
result();
```

还可以等其它值吗？

```
const fn1 = function fn() {
  return Promise.resolve(1);
};
const fn2 = function test() {
  return "test";
};
async function result() {
  const r1 = await fn1();
  const r2 = await fn2();
  console.log(r1, r2); // 1 'test'
}
result();
```

结果呢，不是promise对象的值也等到了。

> await 不仅仅用于等 Promise 对象，它可以等任意表达式的结果。并且返回处理结果
>
> 只能在async函数内部使用

## async/await作用

根据上述内容呢，了解到async与await如何使用以及返回结果，那它们的作用体现到哪了呢？

先看下列有一个业务，业务内容呢需要重复请求操作，但是接口参数呢都需要在上一个请求结果中获取，先看下例子

```
function getFetch(type) {
  setTimeout(() => {
    let result = type + 1;
    return result;
  }, 1000);
}
getFetch(a).then((b) => {
  getFetch(b).then((c) => {
    getFetch(c).then((data) => {
      return data;
    });
  });
});
```

多重的异步操这是不是传说中的回调地狱呢，那怎么解决呢

用promise方法来解决

```
function getFetch(type) {
  return new Promise((resolve, reject) => {
    let result = type + 1;
    resolve(result);
  });
}

getFetch(0)
  .then((res) => {
    console.log(res);
    return getFetch(res);
  })
  .then((res) => {
    console.log(res);
    return getFetch(res);
  })
  .then((res) => {
    console.log(res);
  });
```

来用async/await 来解决

```
function getFetch(type) {
  let result = type + 1;
  return result;
}

async function getResult(a) {
  const n1 = await getFetch(a);
  const n2 = await getFetch(n1);
  const n3 = await getFetch(n2);
  console.log(n1, n2, n3);
}
getResult(0);
```

输出结果呢与`Promise`解决方案是一致的，但是代码看起来简洁明了

> **用同步方式，执行异步操作**达到排队效果，解决回调地狱

## Generator

async/await为什么说是语法糖呢，是谁的语法糖呢？

在阮一峰的ES6入门教程中有说到：

async 函数是什么？一句话，它就是 Generator 函数的语法糖。

###

> Generator 函数就是一个封装的异步任务，或者说是异步任务的容器。
>
> 异步操作需要暂停的地方，都用 yield 语句注明
>
> 调用 Generator 函数，返回的是指针对象(这是它和普通函数的不同之处),。调用指针对象的 next 方法，会移动内部指针。
>
> next 方法的作用是分阶段执行 Generator 函数。每次调用 next 方法，会返回一个对象，表示当前阶段的信息（ value 属性和 done 属性）。value 属性是 yield 语句后面表达式的值，表示当前阶段的值；done 属性是一个布尔值，表示 Generator 函数是否执行完毕，即是否还有下一个阶段。

了解generator用法

```
function* Generator() {
  yield "1";
  yield Promise.resolve(2);
  return "3";
}
var gen = Generator();
console.log(gen); //返回一个指针对象 Generator {<suspended>}
```

调用next方法

```
let res1 = gen.next();
console.log(res1); // 返回当前阶段的值 { value: '1', done: false }

let res2 = gen.next();
console.log(res2); // 返回当前阶段的值 { value: Promise { 2 }, done: false }

res2.value.then((res) => {
  console.log(res); // 2
});

let res3 = gen.next();
console.log(res3); // { value: '3', done: true }

let res4 = gen.next();
console.log(res4); // { value: undefined, done: true }
```

## 实现async/await

**async/await的理解**

-   **async 函数执行结果返回的是一个Promise**
-   **async 函数就是将 Generator 函数的星号（*）替换成 async，将 yield 替换成await**
-   **async/await 就是 Generator 的语法糖，其核心逻辑是迭代执行next函数**

先来初步实现一个执行结果返回Promise的函数

```
function muYouAsync(){
    // 返回一个函数
    return function(){
        // 返回一个promise
        return new Promise((resolve, reject) => {

        })
    }
}
```

并且呢muYouAsync接受一个Generator函数作为参数的，那我们再来完善一下

```
function* gen() {

}
//接受一个Generator函数作为参数
function muYouAsync(gen){
    // 返回一个函数
    return function(){
        // 返回一个promise
        return new Promise((resolve, reject) => {

        })
    }
}
```

我们来测试下看下执行结果是否返回Promise

```
const asyncGen = muYouAsync(gen)
console.log(asyncGen()) //Promise {<pending>}
```

看输出结果的话与执行结果返回Promise是一致的

至此呢第一部分函数执行返回结果已完成，那我们来完善一下Generator函数

```
const getFetch = (nums) =>
  new Promise((resolve) => {
    setTimeout(() => {
      resolve(nums + 1);
    }, 1000);
  });

function* gen() {
  let res1 = yield getFetch(1);
  let res2 = yield getFetch(res1);
  let res3 = yield getFetch(res2);
  return res3;
}
```

Generator函数也编写完成了，在下一步我们要想，怎么让它执行起来了呢，那就需要用到Generator核心逻辑迭代执行next函数，**注意点是需要将next迭代执行**

```
//接受一个Generator函数作为参数
function muYouAsync(gen) {
  // 返回一个函数
  return function () {
    // 返回一个promise
    return new Promise((resolve, reject) => {
      // 执行Generator函数
      let g = gen();
      const next = (context) => {
        const { done, value } = g.next(context);
        if (done) {
          // 这时候说明已经是完成了，需要返回结果
          resolve(value);
        } else {
          // 继续执行next函数,传入执行结果
          return Promise.resolve(value).then(val => next(val))
        }
      };
      next();
    });
  };
}
```

整体的逻辑已经全都补充好了，那我们还需要在完善下最后一步，async返回的是promise，所以我们可以用try catch 来捕获

## 完整代码

```
//接受一个Generator函数作为参数
function muYouAsync(gen) {
  // 返回一个函数
  return function () {
    // 返回一个promise
    return new Promise((resolve, reject) => {
      // 执行Generator函数
      let g = gen();
      const next = (context) => {
        let res
        try {
            res = g.next(context);
        } catch (error) {
            reject(error)
        }
        if (res.done) {
          // 这时候说明已经是完成了，需要返回结果
          resolve(res.value);
        } else {
          // 继续执行next函数,传入执行结果
          return Promise.resolve(res.value).then(val => next(val), err => next(err))
        }
      };
      next();
    });
  };
}
```

那我们最后来测试一下

```
const getFetch = (nums) =>
  new Promise((resolve) => {
    setTimeout(() => {
      resolve(nums + 1);
    }, 1000);
  });

function* gen() {
  let res1 = yield getFetch(1);
  let res2 = yield getFetch(res1);
  let res3 = yield getFetch(res2);
  return res3;
}

const asyncGen = muYouAsync(gen);
asyncGen().then(res => {console.log(res)}); // 4
```

## 结语

今年想法呢，是输出一些文章，总不能干了几年活了，还是碌碌无为，躺平过日子。年前那几天也刷了不少文章，看到了很多平台的前端打工人，都很卷的样子，那我今年也就卷一下子吧，腰断了就不卷了。
