---
title: 手写 Promise
date: 2020-08-01
tags: Javascript
---

## Promise 是什么

Promise 出现之前，处理异步，可以通过回调来解决，比如典型的 ajax：

```javascript
ajax({
  url: 'a',
  success: () => {
    ajax({
      url: 'b',
      success: () => {
        ajax({
          url: 'c',
          success: () => {}
          fail: () => {}
        })
      }
      fail: () => {}
    })
  },
  error: () => {}
})
```

还有事件监听：

```javascript
window.addListener('click', callback)
```

可以看到，其实回调函数在不同的使用中传入方式不一。
Promise 最开始由社区提出，它避免了回调地狱，用法也更强大，es6 正式将 Promise 写进了规范。

```javascript
ajaxPromise({ url: 'a' })
.then(res => {
  return ajaxPromise({ url: 'b'})
}).then(res => {
  return ajaxPromise({ url: 'c'})
})
```

## Promise 的用法

Promise 是一个构造函数，它的实例创建如下

```javascript
var p = new Promise(() => {})
var p1 = new Promise(resolve => resolve(12))
var p2 = new Promise((r, reject) => reject(12))
```

promise 实例总是处于下列三种状态之一，最开始 promise 是在 pending 状态，一旦它变成‘完成’或‘拒绝’，则不会再变：

- 待定: pending
- 完成: fulfilled
- 拒绝: rejected

支持以下几种实例用法和静态：

```javascript
Promise.prototype.then 
Promise.prototype.catch
Promise.prototype.finally

Promise.resolve(any) 
Promise.reject(any)  
Promise.all([])  
Promise.allSettled([])
Promise.any([])  
Promise.race([])
```

## Promise 实例方法的自定义实现
我们先来实现几个实例方法，过去我写过但没有成功，反思一下还是对一些细节的理解不到位导致。我们看下下边这个测试脚本：
```javascript
// 测试用例 1
var a = new Promise((resolve, reject) => {
  resolve('start');
})

var b = a.then(() => { console.log('then 执行'); return 'thenReturnResult' });
var c = a.catch(() => { console.log('catch 执行'); return 'catchReturnResult' });
var d = a.finally(() => { console.log('finally 执行'); return 'finallyReturnResult' });

console.log(a === b, b === c, c === d); 
// false false false
```
发现有一个非常重要的细节是：
> 一个 promise 实例只能一个 then/catch/finally 函数处理，处理完后返回的都是新的 promise。

写代码之前，我们先来解读一下这段代码的执行顺序，以便我们写的更加的顺滑：
1. 立即执行 new Promise 时传入的函数，Promise 给此函数传入resolve/reject参数，并记录 then、catch、finally 的挂载情况。
2. 使用者调用 resolve/reject，a 实例的状态被改变，触发 a.then/a.catch/a.finally 执行。
3. a.then 挂载函数被执行，b 实例的状态被改变。
4. a.catch 执行默认逻辑（不执行挂载函数），c 实例的状态被改变。
5. a.finally 挂载函数被执行，d 实例的状态被改变。

```javascript
class MyPromise {
  status = "pending"; // 'pending' | 'fulfilled' | 'rejected'
  value = undefined;
  // 一个实例只能被一个 then/catch/finally 函数处理，所以无需用列表存储挂载函数
  sucFn = v => v; // 成功之后执行
  errFn = v => v; // 失败之后执行

  // 立即执行，并传入 resolve/reject 函数供使用者调用
  constructor(fn) {
    fn(this.#resolve, this.#reject);
  }

  // 成功之后，触发 .then/.catch/.finally 执行
  #resolve = value => {
    if (this.status !== "pending") return;
    this.status = "fulfilled";
    this.value = value;
    queueMicrotask(() => {
      this.sucFn && this.sucFn();
    });
  }
  #reject = reason => {
    if (this.status !== "pending") return;
    this.status = "rejected";
    this.value = reason;
    queueMicrotask(() => {
      this.errFn && this.errFn();
    });
  }

  // a.then 返回的是一个新的 Promise, 挂载的 onFulfilled|onRejected 需要在 a 的状态变化后执行
  then = (onFulfilled, onRejected) => {
    return new MyPromise((resolve, reject) => {
      this.sucFn = () => {
        // 如果没传，就透传当前promise状态和值；否则执行 onFulfilled
        if (!onFulfilled) {
          return this.status === "fulfilled"
            ? resolve(this.value)
            : reject(this.value);
        }
        try {
          resolve(onFulfilled(this.value));
        } catch (e) {
          reject(e);
        }
      };
      this.errFn = () => {};
    });
  }
  catch = (onRejected) => {}
  finally = (callback) => {}
}
```
这里实现了一部分 then 的逻辑，catch、finally 的逻辑是类似的，读者可以自己尝试写一下。

### Promise 静态方法实现全代码
```javascript
class MyPromise {
  status = "pending"; // 'pending' | 'fulfilled' | 'rejected'
  value = undefined;

  constructor(fn) {
    try {
      fn(this.#resolve, this.#reject);
    } catch (e) {
      this.#reject(e);
    }
  }

  #resolve = (value) => {
    if (this.status !== "pending") return;
    this.status = "fulfilled";
    this.value = value;
    queueMicrotask(() => {
      this.sucFn && this.sucFn();
    });
  };
  #reject = (reason) => {
    if (this.status !== "pending") return;
    this.status = "rejected";
    this.value = reason;
    queueMicrotask(() => {
      this.errFn && this.errFn();
    });
  };

  #callbackCatch = (fn, resolve, reject) => () => {
    if (!fn) {
      // 如果不传入，就透传当前promise状态
      return this.status === "fulfilled"
        ? resolve(this.value)
        : reject(this.value);
    }
    try {
      resolve(fn(this.value));
    } catch (e) {
      reject(e);
    }
  };

  then = (onFulfilled, onRejected) => {
    // then 返回的是一个新的 Promise
    return new MyPromise((resolve, reject) => {
      this.sucFn = this.#callbackCatch(onFulfilled, resolve, reject);
      this.errFn = this.#callbackCatch(onRejected, resolve, reject);
    });
  };

  catch = (onRejected) => {
    return new MyPromise((resolve, reject) => {
      // 如果上一个 promise 是成功的，被 catch 时依然要把成功的结果透传
      this.sucFn = this.#callbackCatch((v) => v, resolve, reject);
      this.errFn = this.#callbackCatch(onRejected, resolve, reject);
    });
  };

  finally = (callback) => {
    return new MyPromise((resolve, reject) => {
      // 正常情况下 finally 要把上一个 promise 的状态和结果透传
      // 只有一种特殊情况：如果finally内部抛错，状态仍需要变更为 rejected
      this.sucFn = this.#callbackCatch(
        () => callback(),
        () => resolve(this.value),
        reject
      );
      this.errFn = this.#callbackCatch(
        () => callback(),
        () => resolve(this.value),
        reject
      );
    });
  };
}
```

再用以下脚本来测试一下结果，看看与Promise是否一致：
```javascript
var a = new MyPromise((resolve, reject) => {
  console.log(2);  // 2
  setTimeout(() => {
    resolve('start');
  }, 0);
})
  .then((res) => {
    console.log('then1', res);  // then1 start
    return 'then1Res';
  })
  .catch((e) => {
    console.log('catch1', e);
    throw 'catch1Throw';
  })
  .then(
    (res) => {
        console.log('then2', res);   // then2 then1Res
        throw 'then2Throw';
    },
    (err) => {
      console.log('then2Catch', err);
      throw 'then2CatchThrow';
    }
  )
  .catch((e) => {
    console.log('catch2', e);  // catch2 then2Throw
    return 'catch2Res';
  })
  .then((res) => {
    console.log('then3', res); // then3 catch2Res
    return 'then3Res';
  })
  .finally((res) => {
    console.log('finally11111', res);  // finally11111 undefined
  })
  .finally((res) => {
    console.log('finally22222', res);  // finally22222 undefined
    return 'finally22222';
  })
  .then(
    (res) => {
        console.log('then4', res);   // then4 then3Res
        return 'then4Res';
    },
    (err) => {
      console.log('thenCatch', err);
      return 'thenCatchThrow';
    }
  )
  .finally(() => {
    console.log('finally33333');  // finally33333
    throw 'finally3Throw';
  })
  .then((res) => {
    console.log('then5', res);
    throw 'then5Throw';
  })
  .catch((e) => {
    console.log('catch4', e);  // catch4 finally3Throw
    return 'catch4Res';
  });
```
完全一致（图就不贴了，读者可以自行执行以下），perfect。

## Promise 静态方法的自定义实现

当实例方法实现好之后，静态方法就显得比较轻松了，大家只要理解了这几个函数的执行细节，就会好些很多。
先来一段测试脚本，先来看看 resolve、reject 静态方法的表现

```javascript
var sucFn = () => new Promise((resolve, reject) => resolve('111'));
var errFn = () => new Promise((resolve, reject) => reject('222'));
var pendingFn = () => new Promise((resolve, reject) => setTimeout(() => resolve('333'), 1000));

var a = errFn();
var b = Promise.resolve(a);
console.log(a === b); // true

var a = pendingFn();
var b = Promise.resolve(a); 
var c = Promise.resolve(Promise.resolve(a)); 
console.log(a === b); // true

var a = sucFn();
var b = Promise.resolve(a);
console.log(a === b); // true

var a = sucFn();
var b = Promise.reject(a);  // b.status => 'fulfilled', b.value => '222'
console.log(a === b); // false

var a = errFn();
var b = Promise.reject(a);  // b.status => 'rejected', b.value => Promise<rejected, '222'>
console.log(a === b); // false

var a = pendingFn();
var b = Promise.reject(a);  // b.status => 'rejected', b.value => Promise<pending, '222'>
console.log(a === b); // false

```
从执行结果我们来解读一下：
1. Promise.resolve 传入的参数如果是promise实例，则透传该实例的状态及结果；否则 fulfilled 并传出参数本身
2. Promise.reject 传入的参数如果是promise实例且被 fulfilled，则 rejected 并传出该实例的结果；否则 reject 并传出参数本身

比较简单，就直接放代码了。
```javascript
MyPromise.resolve = (value) => {
  // 如果 value 是 promise 实例，透传实例即可；否则返回成功态
  if (value instanceof MyPromise) return value;
  return new MyPromise((resolve) => resolve(value));
};
MyPromise.reject = (value) => {
  return new MyPromise((resolve, reject) => {
    // 如果 value 是 promise 实例且是成功的，reject 该实例的值；否则reject value本身
    MyPromise.resolve(value)
      .then((v) => reject(v))
      .catch(() => reject(value));
  });
};
```

至于 all/allSettled/race/any，我们再来几个测试案例：

``` javascript
var a = Promise.all([1,2,3].map(v => {
  return v === 3 ? new Promise((x) => { setTimeout(() => { console.log(123); x(v) }, 1000)}) : Promise.resolve(v);
}))
setTimeout(() => console.log(a), 0); // Promise<pending>  
setTimeout(() => console.log(a), 1000) // Promise<fulfilled, [1,2,3]>

var b = Promise.allSettled([1,2,3].map(v => {
  return v === 3 ? new Promise((resolve,reject) => { setTimeout(() => { console.log(123); reject(v) }, 1000)}) : Promise.resolve(v);
}))
setTimeout(() => console.log(b), 0); // Promise<pending>  
setTimeout(() => console.log(b), 1000) // Promise<fulfilled, [{status: 'fulfilled', value: 1},{status: 'fulfilled', value: 2},{status: 'rejected', reason: 3}]>

var c = Promise.any([1,2,3].map(v => {
  return v === 3 ? Promise.reject(v) : new Promise((resolve, reject) => { setTimeout(() => { console.log(123); resolve(v) }, 1000)});
}));
setTimeout(() => console.log(c), 0); // Promise<pending>  
setTimeout(() => console.log(c), 1000) // Promise<fulfilled, 1>

var d = Promise.race([1,2,3].map(v => {
  return v === 3 ? Promise.reject(v) : new Promise((resolve, reject) => { setTimeout(() => { console.log(123); resolve(v) }, 1000)});
}));
setTimeout(() => console.log(d), 0); // Promise<rejected, 3>  
setTimeout(() => console.log(d), 1000) // Promise<rejected, 3>

```
解读一下:
- Promise.all([]) ，所有元素都被 fulfilled 才成功；任一 rejected 就 rejected；否则 pending
- Promise.allSettled([]) ，所有元素都被 fulfilled 或 rejected 才成功；否则 pending
- Promise.any([]) ，任一元素被 fulfilled 才成功; 所有 rejected 才 rejected；否则 pending
- Promise.race([]) ，任一元素被 fulfilled 或 rejected 立即返回它的结果；否则 pending

搞清楚他们的状态流转之后，代码逻辑就比较清晰了，这里直接看代码：

``` javascript
MyPromise.all = (list) => {
  return new MyPromise((resolve, reject) => {
    let result = [];
    let count = 0;
    list.forEach((item, index) => {
      MyPromise.resolve(item)
        .then((res) => {
          result[index] = res;
          count += 1;
        })
        .catch((err) => reject(err))
        .finally(() => {
          // 如果存在失败或 pending 中的实例，什么都不做
          if (count !== list.length) return;
          resolve(result);
        });
    });
  });
};
MyPromise.allSettled = (list) => {
  return new MyPromise((resolve, reject) => {
    let result = [];
    let count = 0;
    list.forEach((item, index) => {
      MyPromise.resolve(item)
        .then((res) => {
          result[index] = { status: "fulfilled", value: res };
          count += 1;
        })
        .catch((err) => {
          result[index] = { status: "rejected", reason: err };
          count += 1; // 一定要放在 result[index] 赋值后边，避免count自增后finally在赋值之前执行
        })
        .finally(() => {
          // 如果存在 pending 中的实例，什么都不做
          if (count !== list.length) return;
          resolve(result);
        });
    });
  });
};
MyPromise.any = (list) => {
  return new MyPromise((resolve) => {
    let result = [];
    let count = 0;
    list.forEach((item) => {
      MyPromise.resolve(item)
        .then((res) => {
          resolve(res);
        })
        .catch((err) => {
          result[index] = err;
          count += 0;
        })
        .finally(() => {
          if (count !== list.length) return;
          throw new Error({
            stack: "AggregateError: All promises were rejected",
            message: "All promises were rejected",
            errors: result,
          });
        });
    });
  });
};
MyPromise.race = (list) => {
  return new MyPromise((resolve, reject) => {
    list.forEach((item) => {
      MyPromise.resolve(item)
        .then((res) => resolve(res))
        .catch((err) => reject(err));
    });
  });
};
```

至此，MyPromise 就写完了，感谢各位读者耐心观看，如有帮助不胜荣幸。