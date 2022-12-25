---
title: Promise
date: 2020-08-01
---

## Promise 是什么

Promise 出现之前，处理异步，可以通过回调来解决，比如典型的 ajax：

```
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

```
window.addListener('click', callback)
```

可以看到，其实回调函数在不同的使用中传入方式不一。
Promise 最开始由社区提出，它避免了回调地狱，用法也更强大，es6 正式将 Promise 写进了规范。

```
ajaxPromise({ url: 'a' })
.then(res => {
  return ajaxPromise({ url: 'b'})
}).then(res => {
  return ajaxPromise({ url: 'c'})
})
```

## Promise 的用法

Promise 是一个构造函数，它的实例创建如下

```
var p = new Promise(() => {})
var p1 = new Promise(resolve => resolve(12))
var p2 = new Promise((r, reject) => reject(12))
```

promise 实例总是处于下列三种状态之一，最开始 promise 是在 pending 状态，一旦它变成‘完成’或‘拒绝’，则不会再变：

- 待定: pending
- 完成: fulfilled
- 拒绝: rejected

还支持以下几种用法：

```
Promise.resolve() 
Promise.reject()
Promise.all([])  // 参数中所有 promise 被完成，则返回完成的结果，否则返回拒绝
Promise.allSettled([]) // 参数中所有 promise 被返回，则返回它们的结果
Promise.race([])   // 参数中任意一个 promise 被返回，则立即返回它的结果
Promise.any([])  // 还在实验中，参数中任意一个 promise 被完成，则立即返回它的结果，否则返回拒绝
Promise.prototype.then 
Promise.prototype.catch
Promise.prototype.finally
```


