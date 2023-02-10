## 前言

`大家好，我是梁木由，是一个有想头的前端。这几天再回顾基础知识时，对Promise有了较为深入的理解，那今天就来分享下怎么由浅入深的掌握Promise并且学会手写Promise`

## 概念

Promise 是异步编程的一种解决方案，比传统的解决方案——回调函数和事件——更合理和更强大。它由社区最早提出和实现，ES6 将其写进了语言标准，统一了用法，原生提供了`Promise`对象。

所谓`Promise`，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。从语法上说，Promise 是一个对象，从它可以获取异步操作的消息。Promise 提供统一的 API，各种异步操作都可以用同样的方法进行处理。

## Promise 拉出来单练

那我们先把Promise拉出来练练，看看是什么玩意，在控制台中打印看下
![image-20230112101134170.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1a06f55452624b2fa459b200848c1774~tplv-k3u1fbpfcp-watermark.image?)
在上图可以看出什么信息呢，那我们罗列下

-   首先我们看出在`new Promise`时，需要传入一个回调函数
-   它是一个类，并且会返回一个Promise对象
-   那还可以看出它有`constructor`构造函数，还有`catch`、`finally`、`then`三个方法

那我们根据上述分析出的信息，简单实现一下

```javascript
class CustomPromise {
  constructor(callBack) {
  }
  catch() {
  }
  then() {
  }
  finally() {
  }
}
const customPromise = new CustomPromise()
console.log(customPromis)
```

看下我们自己简单实现的输出结果
![image-20230112103322764.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/05d708c68c984663a738efca30a0eaf1~tplv-k3u1fbpfcp-watermark.image?)
那我们再写一个Promise的常规用法

```javascript
const promise = new Promise((resolve, reject) => {
  console.log("hellow Promise");
});
console.log(promise);
```

![image-20230112104207892.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3aa98f90a22c48169d3188452127841f~tplv-k3u1fbpfcp-watermark.image?)

那我们来看看打印结果，能分析出什么结果

-   hellow promise 在控制台被输出了，那是不是说我们传入的回调函数被立即执行了，那说明传入的是一个执行器

那再改进一下我们的CustomPromise

```javascript
class CustomPromise {
  constructor(executor) {
      executor()
  }
  catch() { }
  then() { }
  finally() { }
}
const customPromise = new CustomPromise((resolve, reject) => {
  console.log('hellow Promise')
})
console.log(customPromise)
```

## Promise基本原理与基本特征

**那我们来看看我们所熟知的`Promise`的基本原理**

-   首先我们在调用Promise时，会返回一个Promise对象。
-   构建Promise对象时，需要传入一个executor函数，Promise的主要业务流程都在executor函数中执行。
-   如果运行在excutor函数中的业务执行成功了，会调用resolve函数；如果执行失败了，则调用reject函数。
-   Promise的状态不可逆，同时调用resolve函数和reject函数，默认会采取第一次调用的结果。

**结合Promise/A+规范，我们还可以分析出哪些基本特征**

Promise/A+的规范比较多，在这列出一下核心的规范。[Promise/A+规范](https://link.juejin.cn/?target=https%3A%2F%2Fpromisesaplus.com%2F)

-   promise有三个状态：pending，fulfilled，rejected，默认状态是pending。
-   promise有一个value保存成功状态的值，有一个reason保存失败状态的值，可以是undefined/thenable/promise。
-   promise只能从pending到rejected, 或者从pending到fulfilled，状态一旦确认，就不会再改变。
-   promise 必须有一个then方法，then接收两个参数，分别是promise成功的回调onFulfilled, 和promise失败的回调onRejected。
-   如果then中抛出了异常，那么就会把这个异常作为参数，传递给下一个then的失败的回调onRejected。

那`CustomPromise`，还实现不了基本原理的3，4两条，那我们来根据基本原理与Promise/A+分析下，还缺少什么

-   promise有三个状态：pending，fulfilled，rejected。
-   executor执行器调用reject与resolve两个方法
-   还需要有保存成功或失败两个值的变量
-   then接收两个参数，分别是成功的回调onFulfilled,失败的回调onRejected

那再来改进下`CustomPromise`

```javascript
// 定义三个常量表示状态
const PENDING = 'pending';
const FULFILLED = 'fulfilled';
const REJECTED = 'rejected';
​
class CustomPromise {
  constructor(executor) {
    executor(this.resolve, this.reject);
  }
  // resolve和reject为什么要用箭头函数？
  // 如果直接调用的话，普通函数this指向的是window或者undefined
  // 用箭头函数就可以让this指向当前实例对象
  resolve = (value) => {
    this.value = value;
  };
  reject = (value) => {
    this.reason = value;
  };
  // 成功之后的值
  value = undefined;
  // 失败之后的值
  reason = undefined;
​
  then(onFulfilled,onRejected) {
  }
  catch() {
  }
  finally() {}
}
```

那我们根据Promise基本原理看看它原生Promise的效果

```javascript
new Promise(function (resolve, reject) {
  resolve("成功");
  reject("失败");
}).then(
  (value) => {
    console.log(value); // 结果为‘成功’
  },
  (err) => {
    console.log(err);
  }
);
​
new Promise(function (resolve, reject) {
  reject("失败");
  resolve("成功");
}).then(
  (value) => {
    console.log(value);
  },
  (err) => {
    console.log(err); // 结果为‘失败’
  }
);
```

可以看出与基本原理一样的效果，那我们分析下如何实现这种效果

-   需要控制promise状态
-   在then方法里要调用成功或失败的回调函数

```javascript
const PENDING = "pending";
const FULFILLED = "fulfilled";
const REJECTED = "rejected";
​
class CustomPromise {
  constructor(executor) {
    executor(this.resolve, this.reject);
  }
  // resolve和reject为什么要用箭头函数？
  // 如果直接调用的话，普通函数this指向的是window或者undefined
  // 用箭头函数就可以让this指向当前实例对象
  resolve = (value) => {
    // promise只能从pending到rejected, 或者从pending到fulfilled
    if (this.status == PENDING) {
      this.status = FULFILLED;
      this.value = value;
    }
  };
  reject = (err) => {
    // promise只能从pending到rejected, 或者从pending到fulfilled
    if (this.status == PENDING) {
      this.status = REJECTED;
      this.reason = err;
    }
  };
  status = PENDING;
  // 成功之后的值
  value = undefined;
  // 失败之后的值
  reason = undefined;
​
  then(onFulfilled, onRejected) {
    // 需要判断状态，根据状态选择处理回调函数
    if (this.status == FULFILLED) {
      onFulfilled(this.value);
    } else if (this.status == REJECTED) {
      onRejected(this.reason);
    }
  }
  catch() {
  }
  finally() {}
}
```

来测试下`CustomPromise`

```javascript
new CustomPromise(function (resolve, reject) {
  resolve("成功");
  reject("失败");
}).then(
  (value) => {
    console.log(value);// 结果为‘成功’
  },
  (err) => {
    console.log(err);
  }
);
​
new CustomPromise(function (resolve, reject) {
  reject("失败");
  resolve("成功");
}).then(
  (value) => {
    console.log(value);
  },
  (err) => {
    console.log(err);// 结果为‘失败’
  }
);
​
```

## Promise.then链式调用

我们都知到Primose.then是可以链式调用的，那我们先看看原生效果

```javascript
const promise = new Promise((resolve, reject) => {
  resolve("start");
});
promise
  .then((res) => {
    console.log(res);
    return new Promise((resolve, reject) => {
     setTimeout(() => {
        resolve("hellow");
     },3000)
    });
  })
  .then((res) => {
    console.log(res);
    return "promise";
  })
  .then((res) => {
    console.log(res);
  });
```

输出结果

```
start
hellow
promise
```

那我们来分析实现一下

-   首先.then 是需要返回一个Promise
-   下一个.then 需要拿到上一个.then的返回值
-   有异步操作的话，后一个回调函数，会等待该`Promise`对象的状态发生变化，在被调用
-   有异步操作的话，那就是说有任务队列，需要有收集回调函数的队列

```javascript
const PENDING = "pending";
const FULFILLED = "fulfilled";
const REJECTED = "rejected";
​
class CustomPromise {
  constructor(executor) {
    executor(this.resolve, this.reject);
  }
  // resolve和reject为什么要用箭头函数？
  // 如果直接调用的话，普通函数this指向的是window或者undefined
  // 用箭头函数就可以让this指向当前实例对象
  resolve = (value) => {
    // promise只能从pending到rejected, 或者从pending到fulfilled
    if (this.status == PENDING) {
      this.status = FULFILLED;
      this.value = value;
​
      // resolve里面将所有成功的回调拿出来执行
      if (this.onResolvedCallbacks.length) {
        this.onResolvedCallbacks.forEach((fn) => fn());
      }
    }
  };
  reject = (err) => {
    // promise只能从pending到rejected, 或者从pending到fulfilled
    if (this.status == PENDING) {
      this.status = REJECTED;
      this.reason = err;
      // reject里面将所有失败的回调拿出来执行
      if (this.onFulfilledCallbacks.length) {
        this.onFulfilledCallbacks.forEach((fn) => fn());
      }
    }
  };
  // 存储成功回调函数
  onResolvedCallbacks = [];
  // 存储失败回调函数
  onFulfilledCallbacks = [];
​
  status = PENDING;
  // 成功之后的值
  value = undefined;
  // 失败之后的值
  reason = undefined;
​
  then(onFulfilled, onRejected) {
    // 如果不传，就使用默认函数,确保是函数类型
    onFulfilled =
      typeof onFulfilled === "function" ? onFulfilled : (value) => value;
    onRejected =
      typeof onRejected === "function"
        ? onRejected
        : (reason) => {
            throw reason;
          };
​
    const thenCustomPromise = new CustomPromise((resolve, reject) => {
      const resolveCustomPromise = (callBack, value) => {
        try {
          const x = callBack(value);
          // 如果相等了，说明return的是自己，抛出类型错误并返回
          if (resolveCustomPromise === x) {
            return reject(new TypeError("类型错误"));
          }
          // 判断x是不是 CustomPromise 实例对象
          if (x instanceof CustomPromise) {
            // 执行 x，调用 then 方法，目的是将其状态变为 fulfilled 或者 rejected
            // x.then(value => resolve(value), error => reject(reason))
            // 简化之后
            x.then(resolve, reject);
          } else {
            // 普通值
            resolve(x);
          }
        } catch (error) {
          reject(error);
        }
      };
      // 需要判断状态，根据状态选择处理回调函数
      if (this.status == FULFILLED) {
        resolveCustomPromise(onFulfilled, this.value);
      } else if (this.status == REJECTED) {
        resolveCustomPromise(onRejected, this.reason);
      } else if (this.status == PENDING) {
        // 当状态为pending时,把then回调push进resolve/reject执行队列,等待执行
        this.onResolvedCallbacks.push(() =>
          resolveCustomPromise(onFulfilled, this.value)
        );
        this.onFulfilledCallbacks.push(() =>
          resolveCustomPromise(onRejected, this.reason)
        );
      }
    });
    return thenCustomPromise;
  }
  catch() {}
  finally() {}
}
​
```

来验证下.then的链式调用

```javascript
const promise = new CustomPromise((resolve, reject) => {
  resolve("start");
});
promise
  .then((res) => {
    console.log(res);
    return new CustomPromise((resolve, reject) => {
      setTimeout(() => {
        resolve("hellow");
      }, 1000);
    });
  })
  .then((res) => {
    console.log(res);
    return "promise";
  })
  .then((res) => {
    console.log(res);
  });
// 输出结果 start->hellow->promise
```

## Promise.prototype.catch()

**是 .then(null, rejection) 或是 .then(undefined, rejection)的别名，用于指定发生错误时的回调函数**

看下原生promise效果

```javascript
const promise = new Promise((resolve, reject) => {
  resolve("start");
});
promise
  .then((res) => {
    console.log(res);
    return new Promise((resolve, reject) => {
      reject("hellow");
    });
  })
  .catch(err => {console.log(err); return 'promise'})
  .then(res => console.log(res))
```

输出结果

```
start
hellow
promise
```

根据上述原生catch我们来分析下结果

-   执行.then的onRejected回调函数
-   并且可以继续链式调用

```javascript
catch(onFulfilled) {
   return this.then(null, onFulfilled)
}
```

那我们来验证下

```javascript
const promise = new CustomPromise((resolve, reject) => {
  resolve("start");
});
promise
  .then((res) => {
    console.log(res);
    return new CustomPromise((resolve, reject) => {
      reject("hellow");
    });
  })
  .catch((err) => {
    console.log(err);
    return "promise";
  })
  .then((res) => console.log(res));
​
// 输出结果
start
hellow
promise
```

## Promise.resolve()

**Promise.resolve(value)返回一个解析过的Promise对象，用法有一个value参数**

-   如果参数是 Promise 实例，那么`Promise.resolve`将不做任何修改、原封不动地返回这个实例。

    -   ```javascript
        const promise = new Promise((resolve, reject) => {
          resolve("start");
        });
        ​
        const resolvePromise = Promise.resolve(promise);
        resolvePromise.then((res) => console.log(res)); 
        // start
        ```

-   如果参数是 具有`then`方法的对象`Promise.resolve()`方法会将这个对象转为 Promise 对象，然后就立即执行`thenable`对象的`then()`方法

    -   ```javascript
        let thenable = {
          then: function(resolve, reject) {
            resolve('promise');
          }
        };
        ​
        let p1 = Promise.resolve(thenable);
        p1.then(function (res) {
          console.log(res);  // promise
        });
        ```

-   如果参数是一个原始值，或者是一个不具有`then()`方法的对象，则`Promise.resolve()`方法返回一个新的 Promise 对象，状态为`resolved`

    -   ```javascript
        const p1 = Promise.resolve('promise');
        ​
        p1.then(function (res) {
          console.log(res)
        });
        // promise
        ```

-   `Promise.resolve()`方法允许调用时不带参数，直接返回一个`resolved`状态的 Promise 对象

    -   ```javascript
        const p = Promise.resolve();
        ​
        p.then(function () {
          // ...
        });
        ```

> 参考资料：[ECMAScript 入门](<https://es6.ruanyifeng.com/#docs/promise#Promise-resolve>

来`CustomPromise`添加静态resolve方法

```javascript
//静态的resolve方法
  static resolve(value) {
    if (value instanceof CustomPromise) return value;
    return new CustomPromise((resolve) => resolve(value));
  }
```

## Promise.reject()

**Promise.reject(reason)** 返回一个Promise实例，并且携带reason

```javascript
const promise = Promise.reject("rejected message")
// 相当于
// const promise2 = new Promsie((resolve, reject) => {
//   reject("rejected message")
// })
```

根据上述例子我们来分析下结果

-   可以看出无论reason传入什么内容，都是经过reject()方法，那是不是可以理解为就是捕获错误信息

来`CustomPromise`添加静态reject方法

```javascript
//静态的reject方法
static reject(reason) {
   return new CustomPromise((resolve, reject) => reject(reason));
}
```

## Promise.prototype.finally()

**finally()不接收参数，并且在.then或.catch回调函数执行完以后，再执行finally中的方法**

看下原生promise效果

```javascript
const promise = new Promise((resolve, reject) => {
  resolve("start");
});
promise
  .then((res) => {
    console.log(res);
    return new Promise((resolve, reject) => {
      resolve("hellow");
    });
  })
  .then((res) => {
    console.log(res);
    return new Promise((resolve, reject) => {
      resolve("promise");
    });
  })
  .finally(() => {
    console.log("finally");
    return "is finally";
  })
  .then((res) => console.log(res));
​
```

输出结果

```
start
hellow
finally
promise
```

根据上述原生finally我们来分析下结果

-   finally方法和then以及catch一样，都可以返回一个新的Promise
-   finally并不会影响之前返回的Promise对象
-   可以继续链式调用并且获取之前Promise的值

来`CustomPromise`添加静态finally方法

```javascript
finally(callback) {
    return this.then(
      (value) => CustomPromise.resolve(callback()).then(() => value),
      (reason) => CustomPromise.resolve(callback()).then(() => reason)
    );
}
```

那我们来验证下

```javascript
const promise = new CustomPromise((resolve, reject) => {
  resolve("start");
});
promise
  .then((res) => {
    console.log(res);
    return new CustomPromise((resolve, reject) => {
      resolve("hellow");
    });
  })
  .then((res) => {
    console.log(res);
    return new CustomPromise((resolve, reject) => {
      resolve("promise");
    });
  })
  .finally(() => {
    console.log("finally");
    return "is finally";
  })
  .then((res) => console.log(res));
​
// 输出结果
start
hellow
finally
promise
```

## Promise.all()

**Promise.all() 方法接收一个 promise 的 iterable 类型(Array，Map，Set 都属于 ES6 的 iterable 类型)并返回一个新的Promise实例**

看下原生`Promise.all()`效果

```javascript
let p1 = new Promise((resolve, reject) => {
  resolve("start");
}).then((res) => res);
let p2 = new Promise((resolve, reject) => {
  resolve("hellow");
}).then((res) => res);
let p3 = new Promise((resolve, reject) => {
  resolve("promise");
}).then((res) => res);
​
Promise.all([p1, p2, p3])
  .then((res) => console.log("success:", res))
  .catch((err) => console.log("error:", err));
// 输出结果 success:['start', 'hellow', 'promise']
```

```javascript
let p1 = new Promise((resolve, reject) => {
  resolve("start");
}).then((res) => res);
let p2 = new Promise((resolve, reject) => {
  reject("报错了");
}).then((res) => res);
let p3 = new Promise((resolve, reject) => {
  reject("报错了2");
}).then((res) => res);
​
Promise.all([p1, p2, p3])
  .then((res) => console.log("success:", res))
  .catch((err) => console.log("error:", err));
//输出结果 error:报错了
```

那我们来根据输出结果分析下

-   成功的时候返回的是一个结果数组
-   失败的时候则返回最先被reject失败状态的值

来`CustomPromise`添加静态all()方法

```javascript
  //静态的all方法
  static all(values){
    let result = [];
    let index = 0;
    return new CustomPromise((resolve,reject) => {
      function addPromise(key, value) {
        result[key] = value
        index++
        if (index === values.length) {
          resolve(result)
        }
      }
      
      for(let i = 0; i < values.length; i++){
        let item = values[i];
        if(item instanceof CustomPromise){
          // 参数为Promise
          item.then(value => addPromise(i,value), error => reject(reason))
        }else{
          // 参数为普通值
          addPromise(i,item)
        }
      }
    })
  }
```

那我们来验证下

```javascript
let p1 = new CustomPromise((resolve, reject) => {
  resolve("start");
});
let p2 = new CustomPromise((resolve, reject) => {
  resolve("hellow");
});
let p3 = new CustomPromise((resolve, reject) => {
  resolve("promise");
});
​
CustomPromise.all([p1, p2, p3])
  .then((res) => console.log("success:", res))
  .catch((err) => console.log("error:", err));
// 输出结果 success:['start', 'hellow', 'promise']
```

```javascript
let p1 = new CustomPromise((resolve, reject) => {
  resolve("start");
}).then((res) => res);
let p2 = new CustomPromise((resolve, reject) => {
  reject("报错了");
}).then((res) => res);
let p3 = new CustomPromise((resolve, reject) => {
  reject("报错了2");
}).then((res) => res);
​
CustomPromise.all([p1, p2, p3])
  .then((res) => console.log("success:", res))
  .catch((err) => console.log("error:", err));
//输出结果 error:报错了
```

## Promise.race()

**`Promise.race()`方法返回一个 promise，一旦迭代器中的某个 promise 解决或拒绝，返回的 promise 就会解决或拒绝。**

```javascript
var p1 = new Promise(function(resolve, reject) {
    setTimeout(resolve, 300, "start");
});
var p2 = new Promise(function(resolve, reject) {
    setTimeout(resolve, 100, "hellow");
});
​
const p = Promise.race([p1, p2]).then(function(value) {
  console.log(value); // "hellow"
});
```

来分析下那就是有一个实例先改变状态，p的状态就跟着改变

来`CustomPromise`添加静态race()方法

```javascript
//静态race方法
  static race(values) {
    return new CustomPromise((resolve, reject) => {
      for (const p of values) {
        p.then(resolve, reject);
      }
    });
  }
```

那我们来验证下

```javascript
var p1 = new CustomPromise(function(resolve, reject) {
    setTimeout(resolve, 300, "start");
});
var p2 = new CustomPromise(function(resolve, reject) {
    setTimeout(resolve, 100, "hellow");
});
​
const p = CustomPromise.race([p1, p2]).then(function(value) {
  console.log(value); // "hellow"
});
```

## Promise.allSettled()

**`Promise.allSettled()` 方法**不依赖于彼此成功完成的异步任务，不管每一个操作是成功还是失败，再进行下一步操作。

```javascript
Promise.allSettled([
  Promise.resolve('start'),
  Promise.reject(new Error("error")),
  new Promise((resolve) => setTimeout(() => resolve('hellow'), 0)),
  'promise',
]).then((values) => console.log(values));
​
// [
//   { status: 'fulfilled', value: start },
//   { status: 'rejected', reason: Error: error },
//   { status: 'fulfilled', value: hellow },
//   { status: 'fulfilled', value: promise }
// ]
```

来分析下结果

-   status一个字符串，要么是 `"fulfilled"`，要么是 `"rejected"`，表示 promise 的最终状态。
-   value当 `status` 为 `"fulfilled"`，在 promise 解决时才有 value
-   reason当 `status` 为 `"rejected"`，在 promsie 拒绝时才有 reason

来`CustomPromise`添加静态allSettled()方法

```javascript
//静态allSettled方法
  static allSettled(values) {
    return new Promise((resolve, reject) => {
      let resolveDataList = [],
        resolveCount = 0;
      const addPromise = (status, value, i) => {
        resolveDataList[i] = {
          status,
          value,
        };
        resolveCount++;
        if (resolveCount === values.length) {
          resolve(resolveDataList);
        }
      };
      values.forEach((value, i) => {
        if (value instanceof CustomPromise) {
          value.then(
            (res) => {
              addPromise("fulfilled", res, i);
            },
            (err) => {
              addPromise("rejected", err, i);
            }
          );
        } else {
          addPromise("fulfilled", value, i);
        }
      });
    });
  }
```

来验证下

```javascript
CustomPromise.allSettled([
  CustomPromise.resolve('start'),
  CustomPromise.reject(new Error("error")),
  new CustomPromise((resolve) => setTimeout(() => resolve('hellow'), 0)),
  'promise',
]).then((values) => console.log(values));
​
// [
//   { status: 'fulfilled', value: start },
//   { status: 'rejected', reason: Error: error },
//   { status: 'fulfilled', value: hellow },
//   { status: 'fulfilled', value: promise }
// ]
```

## Promise.any()

**Promise.any()`接收一个由`Promise`所组成的可迭代对象，返回一个新的`promise**

```javascript
const p1 = new Promise((resolve, reject) => {
  setTimeout(resolve, 100, "start");
});
​
const p2 = new Promise((resolve, reject) => {
  reject("报错了");
});
​
const p3 = new Promise((resolve, reject) => {
  setTimeout(resolve, 500, "promise");
});
​
Promise.any([p1, p2, p3]).then((value) => {
  console.log(value);
  // start
})
```

```javascript
const p1 = new Promise((resolve, reject) => {
  setTimeout(reject, 100, "start");
});
​
const p2 = new Promise((resolve, reject) => {
  reject("报错了");
});
​
const p3 = new Promise((resolve, reject) => {
  setTimeout(reject, 500, "promise");
});
​
Promise.any([p1, p2, p3])
  .then((value) => {
    console.log("value:", value);
  })
  .catch((err) => {
    console.log("err:", err); //err: AggregateError: All promises were rejected
  });
​
```

那我们来分析下

-   与all()不同，只要有一个 `promise` 成功，会返回首个成功的 `promise` 的值，方法提前结束
-   如果所有Promise都失败，则报错All promises were

来`CustomPromise`添加静态any()方法

```javascript
//静态any方法
  static any(values) {
    return new CustomPromise((resolve, reject) => {
      let rejectCount = 0;
      values.forEach((value) => {
        value.then(
          (val) => resolve(val),
          (err) => {
            rejectCount++;
            if (rejectCount === value.length) {
              reject("All promises were rejected");
            }
          }
        );
      });
    });
  }
```

我们来验证下

```javascript
const p1 = new CustomPromise((resolve, reject) => {
  setTimeout(resolve, 100, "start");
});
​
const p2 = new CustomPromise((resolve, reject) => {
  reject("报错了");
});
​
const p3 = new CustomPromise((resolve, reject) => {
  setTimeout(resolve, 500, "promise");
});
​
CustomPromise.any([p1, p2, p3])
  .then((value) => {
    console.log("value:", value); //value: start
  })
  .catch((err) => {
    console.log("err:", err);
  });
```

```javascript
const p1 = new CustomPromise((resolve, reject) => {
  setTimeout(reject, 100, "start");
});
​
const p2 = new CustomPromise((resolve, reject) => {
  reject("报错了");
});
​
const p3 = new CustomPromise((resolve, reject) => {
  setTimeout(reject, 500, "promise");
});
​
CustomPromise.any([p1, p2, p3])
  .then((value) => {
    console.log("value:", value);
  })
  .catch((err) => {
    console.log("err:", err); //err: All promises were rejected
  });
```

## 完整代码

```javascript
const PENDING = "pending";
const FULFILLED = "fulfilled";
const REJECTED = "rejected";
​
class CustomPromise {
  constructor(executor) {
    try {
      executor(this.resolve, this.reject);
    } catch (error) {
      // 如果有错误，就直接执行 reject
      this.reject(error);
    }
  }
  // resolve和reject为什么要用箭头函数？
  // 如果直接调用的话，普通函数this指向的是window或者undefined
  // 用箭头函数就可以让this指向当前实例对象
  resolve = (value) => {
    // promise只能从pending到rejected, 或者从pending到fulfilled
    if (this.status == PENDING) {
      this.status = FULFILLED;
      this.value = value;
​
      // resolve里面将所有成功的回调拿出来执行
      if (this.onResolvedCallbacks.length) {
        this.onResolvedCallbacks.forEach((fn) => fn());
      }
    }
  };
  reject = (err) => {
    // promise只能从pending到rejected, 或者从pending到fulfilled
    if (this.status == PENDING) {
      this.status = REJECTED;
      this.reason = err;
      // reject里面将所有失败的回调拿出来执行
      if (this.onFulfilledCallbacks.length) {
        this.onFulfilledCallbacks.forEach((fn) => fn());
      }
    }
  };
  // 存储成功回调函数
  onResolvedCallbacks = [];
  // 存储失败回调函数
  onFulfilledCallbacks = [];
​
  status = PENDING;
  // 成功之后的值
  value = undefined;
  // 失败之后的值
  reason = undefined;
​
  then(onFulfilled, onRejected) {
    // 如果不传，就使用默认函数,确保是函数类型
    onFulfilled =
      typeof onFulfilled === "function" ? onFulfilled : (value) => value;
    onRejected =
      typeof onRejected === "function"
        ? onRejected
        : (reason) => {
            throw reason;
          };
​
    const thenCustomPromise = new CustomPromise((resolve, reject) => {
      const resolveCustomPromise = (callBack, value) => {
        try {
          const x = callBack(value);
          // 如果相等了，说明return的是自己，抛出类型错误并返回
          if (resolveCustomPromise === x) {
            return reject(new TypeError("类型错误"));
          }
          // 判断x是不是 CustomPromise 实例对象
          if (x instanceof CustomPromise) {
            // 执行 x，调用 then 方法，目的是将其状态变为 fulfilled 或者 rejected
            // x.then(value => resolve(value), error => reject(reason))
            // 简化之后
            x.then(resolve, reject);
          } else {
            // 普通值
            resolve(x);
          }
        } catch (error) {
          reject(error);
        }
      };
      // 需要判断状态，根据状态选择处理回调函数
      if (this.status == FULFILLED) {
        resolveCustomPromise(onFulfilled, this.value);
      } else if (this.status == REJECTED) {
        resolveCustomPromise(onRejected, this.reason);
      } else if (this.status == PENDING) {
        // 当状态为pending时,把then回调push进resolve/reject执行队列,等待执行
        this.onResolvedCallbacks.push(() =>
          resolveCustomPromise(onFulfilled, this.value)
        );
        this.onFulfilledCallbacks.push(() =>
          resolveCustomPromise(onRejected, this.reason)
        );
      }
    });
    return thenCustomPromise;
  }
  catch(onFulfilled) {
    return this.then(null, onFulfilled);
  }
  finally(callback) {
    return this.then(
      (value) => CustomPromise.resolve(callback()).then(() => value),
      (reason) => CustomPromise.resolve(callback()).then(() => reason)
    );
  }
  //静态的resolve方法
  static resolve(value) {
    if (value instanceof CustomPromise) return value;
    return new CustomPromise((resolve) => resolve(value));
  }
  //静态的reject方法
  static reject(reason) {
    return new CustomPromise((resolve, reject) => reject(reason));
  }
  //静态的all方法
  static all(values) {
    // 用来记录Promise成功的次数
    let resolveCount = 0,
      // 用来保存Promise成功的结果
      resolveDataList = [];
    return new CustomPromise((resolve, reject) => {
      function addPromise(key, value) {
        resolveDataList[key] = value;
        resolveCount++;
        if (resolveCount === values.length) {
          resolve(resolveDataList);
        }
      }
​
      for (let i = 0; i < values.length; i++) {
        let item = values[i];
        if (item instanceof CustomPromise) {
          // 参数为Promise
          item.then(
            (value) => addPromise(i, value),
            (error) => reject(error)
          );
        } else {
          // 参数为普通值
          addPromise(i, item);
        }
      }
    });
  }
  //静态race方法
  static race(values) {
    return new CustomPromise((resolve, reject) => {
      for (const p of values) {
        p.then(resolve, reject);
      }
    });
  }
  //静态allSettled方法
  static allSettled(values) {
    return new Promise((resolve, reject) => {
      let resolveDataList = [],
        resolveCount = 0;
      const addPromise = (status, value, i) => {
        resolveDataList[i] = {
          status,
          value,
        };
        resolveCount++;
        if (resolveCount === values.length) {
          resolve(resolveDataList);
        }
      };
      values.forEach((value, i) => {
        if (value instanceof CustomPromise) {
          value.then(
            (res) => {
              addPromise("fulfilled", res, i);
            },
            (err) => {
              addPromise("rejected", err, i);
            }
          );
        } else {
          addPromise("fulfilled", value, i);
        }
      });
    });
  }
  //静态any方法
  static any(values) {
    return new CustomPromise((resolve, reject) => {
      let rejectCount = 0;
      values.forEach((value) => {
        value.then(
          (val) => resolve(val),
          (err) => {
            rejectCount++;
            if (rejectCount === value.length) {
              reject("All promises were rejected");
            }
          }
        );
      });
    });
  }
}
```

## 结语

关于Promise的实现就到这里了，希望能跟大家一起进步⛽️⛽️⛽️

如果写的有问题，欢迎大家指出问题，一起讨论，让我再打磨打磨

最后呢，**希望大家支持一下**，**长文不易**，**记得给点个赞**👍👍👍

**搞了一个前端学习（摸鱼）交流群，想加入的去[沸点](https://juejin.cn/user/1583757900135464/pins)**

![pexels-king-siberia-2277978.jpg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/91363bb09e744d41b4bc164bc529db38~tplv-k3u1fbpfcp-watermark.image?)
