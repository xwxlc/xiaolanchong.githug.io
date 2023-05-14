---
title: 手写一个Promise
categories:
  - 前端
tags:
date: 2023-05-14 09:50:36
updated:
---

# 手写一个 Promise

## 首先看看我们 Promise 的通常用法

```javascript
const p = new Promise((resolve, reject) => {
  resolve(1);
});
p.then((res) => {
  console.log('返回成功结果', res);
});
```

1. 需要一个 Promise 构造函数
2. 需要传入一个执行方法，并且有 resolve、reject 两个方法
3. 有 then 方法
4. 当天需要满足 promise A+ 的规范 [知乎答案](https://zhuanlan.zhihu.com/p/404659910)

> 接下来我们开始自己手写一个 MpPromise

## 1. 创建一个构造函数

```javascript
class MyPromise {
  result = undefined;
  constructor(executor) {
    const resolve = (data) => {
      this.result = data;
    };
    const reject = (reason) => {
      this.result = reason;
    };
    executor(resolve, reject);
  }
  then(onFulfilled, onRejected) {}
}
```

> 一个 Promise 基本的结果就完成了

> 接下来我们给 Promise 添加状态 pending、fulfilled、rejected

## pending、fulfilled、rejected

```javascript
class MyPromise {
  state = 'pending';
  result = undefined;
  constructor(executor) {
    const resolve = (data) => {
      if (this.state != 'pending') return;
      this.state = 'fulfilled';
      this.result = data;
    };
    const reject = (reason) => {
      if (this.state != 'pending') return;
      this.state = 'rejected';
      this.result = reason;
    };
    executor(resolve, reject);
  }
  ...
}
```

> 看着好像没啥问题了，但 state,result 好像不怎么安全，所以我们需要把他们写成私有属性

> pending、fulfilled、rejected 字符串的直接使用，给后期的维护也留下了隐患

> resolve、reject 方法内的代码有点相似，那我们可以考虑抽离代码

```javascript
class MyPromise {
  // #开头表示私有属性
  #PENDING = 'pending';
  #FULFILLED = 'fulfilled';
  #REJECTED = 'rejected';
  #state = this.PENDING;
  #result = undefined;
  constructor(executor) {
    const resolve = (data) => {
      this.#changeState(this.#FULFILLED, data);
    };
    const reject = (reason) => {
      this.#changeState(this.#REJECTED, reason);
    };
    executor(resolve, reject);
  }
  #changeState(state, reject) {
    if (this.state != this.#PENDING) return;
    this.state = state;
    this.result = result;
  }
  ...
}
```

## 接下来我们来完善 then 方法

> then 是可以链式调用的所以返回也是个 Promise

```javascript
class MyPromise {
  ...
  then(onFulfilled, onRejected) {
    return new MyPromise((resolve, reject) => {
      if (this.#state === this.#FULFILLED) {
        const data = onFulfilled(this.#result)
        resolve(data);
      } else {
         const reason = onRejected(this.#result)
        reject(reason);
      }
    });
  }
}
const p = new MyPromise((resolve,reject)=>{
  setTimeout(() =>{
    resolve(1)
  },1000)
})
p.then(res=>{
  console.log(res)
})
```

> 如果是异步的我们会发现 then 拿不到结果，那么何时才能拿到结果呢

> 次数我们可以回想到之前的 #changeState 方法

> 那么如何才能让 #changeState 来支持 then 里的回调呢？

```javascript
class MyPromise {
  #handlers = []; //为什么是数组呢 因为可能存在多个then
  ...
  #changeState(state, reject) {
    if (this.state != this.#PENDING) return;
    this.state = state;
    this.result = result;
    this.#run();
  }
  #runOne(callblck, resolve, reject) {
    const data = callblck(this.#result);
    resolve(data);
  }
  #run() {
    if (this.#state === this.#PENDING) return;
    while (this.#handlers.length) {
      const { onFulfilled, onRejected, resolve, reject } = this.#handlers.shift();
      if (this.#state === this.#FULFILLED) {
        this.#runOne(onFulfilled, resolve, reject);
      } else {
        this.#runOne(onRejected, resolve, reject);
      }
    }
  }
  then(onFulfilled, onRejected) {
    return new MyPromise((resolve, reject) => {
      this.#handlers.push({
        onFulfilled,
        onRejected,
        resolve,
        reject
      });
      this.#run();
    });
  }
}
```

> 接下来我们需要分析 #runOne 了, callback 存在几种情况呢

1. 不是个函数
2. 放回的结果是 promise

```javascript
class MyPromise {
  // ...
  #isPromiseLike(value) {
    // 注意不是单纯的判断构造函数 满足promise A+规范的都属于promise
    if ((value !== null && typeof value === 'object') || typeof value === 'function') {
      return typeof value.then === 'function';
    }
    return false;
  }
  #runOne(callblck, resolve, reject) {
    if (typeof callback !== 'function') {
      const settled = this.#state == this.#FULFILLED ? resolve : reject;
      settled(this.#result);
      return;
    }
    const data = callback(this.#result);
    if (this.#isPromiseLike(data)) {
      data.then(resolve, reject);
    } else {
      resolve(data);
    }
  }
}
```

> 写到这里 myPromise 基本可用了,但还有一些细节要处理

## 完整代码

```javascript
class MyPromise {
  // #开头表示私有属性

  // 状态常量
  #PENDING = 'pending';
  #FULFILLED = 'fulfilled';
  #REJECTED = 'rejected';

  // 状态
  #state = this.PENDING;
  // 结果
  #result = undefined;
  // then 堆栈
  #handlers = [];

  constructor(executor) {
    const resolve = (data) => {
      this.#changeState(this.#FULFILLED, data);
    };
    const reject = (reason) => {
      this.#changeState(this.#REJECTED, reason);
    };
    // 捕获错误 但无法捕获异步错误
    try {
      executor(resolve, reject);
    } catch (err) {
      reject(err);
    }
  }
  // 只会有一个最终状态
  #changeState(state, result) {
    if (this.#state !== this.#PENDING) return;
    this.#state = state;
    this.#result = result;
    this.#run();
  }
  /**
   * 判断是不是promise
   * 满足promise A+规范
   */
  #isPromiseLike(value) {
    if ((value !== null && typeof value === 'object') || typeof value === 'function') {
      return typeof value.then === 'function';
    }
    return false;
  }
  // 微队列
  #runMicroTask(func) {
    if (typeof process === 'object' && typeof process.nextTick === 'function') {
      //   node 环境
      process.nextTick(func);
    } else if (typeof MutationObserver === 'function') {
      // 浏览器环境 监听
      const ob = new MutationObserver(func);
      const textNode = document.createTextNode('1');
      ob.observe(textNode, {
        characterData: true
      });
      textNode.data = '2';
    } else {
      setTimeout(func, 0);
    }
  }
  #runOne(callback, resolve, reject) {
    this.#runMicroTask(() => {
      // 不是个函数时直接透传结果
      if (typeof callback !== 'function') {
        const settled = this.#state == this.#FULFILLED ? resolve : reject;
        settled(this.#result);
        return;
      }
      try {
        const data = callback(this.#result);
        if (this.#isPromiseLike(data)) {
          data.then(resolve, reject);
        } else {
          resolve(data);
        }
      } catch (err) {
        reject(err);
      }
    });
  }
  #run() {
    if (this.#state === this.#PENDING) return;
    while (this.#handlers.length) {
      const { onFulfilled, onRejected, resolve, reject } = this.#handlers.shift();
      if (this.#state === this.#FULFILLED) {
        this.#runOne(onFulfilled, resolve, reject);
      } else {
        this.#runOne(onRejected, resolve, reject);
      }
    }
  }
  then(onFulfilled, onRejected) {
    return new MyPromise((resolve, reject) => {
      this.#handlers.push({
        onFulfilled,
        onRejected,
        resolve,
        reject
      });
      this.#run();
    });
  }
}
```

## #runMicroTask 方法

> 熟悉事件循环的朋友应该不陌生 {% post_link interview/事件循环 %}

```javascript
class MyPromise {
  ...
  #runMicroTask(func) {
    if (typeof process === 'object' && typeof process.nextTick === 'function') {
      //   node 环境
      process.nextTick(func);
    } else if (typeof MutationObserver === 'function') {
      // 浏览器环境 监听
      const ob = new MutationObserver(func);
      const textNode = document.createTextNode('1');
      ob.observe(textNode, {
        characterData: true
      });
      textNode.data = '2';
    } else {
      setTimeout(func, 0);
    }
  }
  ...
}
```
