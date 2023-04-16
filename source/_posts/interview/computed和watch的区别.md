---
title: computed和watch的区别
categories:
  - 面试题
tags:
  - Vue
date: 2023-04-08 16:44:18
updated:
---

# watch

其实它只是 this.$watch 这个 API 的一种封装：

```javascript
export default {
	watch: {
    name(newName) {...}
	}
}

---------------------------------------------------------

export default {
	created() {
		this.$watch('name', newName => {...})
	}
}
```

## 监听属性初始化

为什么这么说，我们首先来看下初始化时 watch 属性都做了什么：

```javascript
function initState(vm) {  // 初始化所有状态时
  vm._watchers = []  // 当前实例watcher集合
  const opts = vm.$options  // 合并后的属性

  ... // 其他状态初始化

  if(opts.watch) {  // 如果有定义watch属性
    initWatch(vm, opts.watch)  // 执行初始化方法
  }
}

---------------------------------------------------------

function initWatch (vm, watch) {  // 初始化方法
  for (const key in watch) {  // 遍历watch内多个监听属性
    const handler = watch[key]  // 每一个监听属性的值
    if (Array.isArray(handler)) {  // 如果该项的值为数组
      for (let i = 0; i < handler.length; i++) {
        createWatcher(vm, key, handler[i])  // 将每一项使用watcher包装
      }
    } else {
      createWatcher(vm, key, handler) // 不是数组直接使用watcher
    }
  }
}

---------------------------------------------------------

function createWatcher (vm, expOrFn, handler, options) {
  if (isPlainObject(handler)) { // 如果是对象，参数移位
    options = handler
    handler = handler.handler
  }
  if (typeof handler === 'string') {  // 如果是字符串，表示为方法名
    handler = vm[handler]  // 获取methods内的方法
  }
  return vm.$watch(expOrFn, handler, options)  // 封装
}
```

> 首先初始化时，会判断并遍历 vm.$options.watch 属性，最终调用vm.$watch

## 监听属性实现原理

所以我们来看下$watch 的内部实现：

```javascript
Vue.prototype.$watch = function(expOrFn, cb, options = {}) {
  const vm = this
  if (isPlainObject(cb)) {  // 如果cb是对象，当手动创建监听属性时
    return createWatcher(vm, expOrFn, cb, options)
  }

  options.user = true  // user-watcher的标志位，传入Watcher类中
  const watcher = new Watcher(vm, expOrFn, cb, options)  // 实例化user-watcher

  if (options.immediate) {  // 立即执行
    cb.call(vm, watcher.value)  // 以当前值立即执行一次回调函数
  }  // watcher.value为实例化后返回的值

  return function unwatchFn () {  // 返回一个函数，执行取消监听
    watcher.teardown()
  }
}

---------------------------------------------------------------

export default {
  data() {
    return {
      name: 'cc'
    }
  },
  created() {
    this.unwatch = this.$watch('name', newName => {...})
    this.unwatch()  // 取消监听
  }
}
```

> 虽然 watch 内部是使用 this.$watch，当我们还是可以通过 this.$watch 来创建监听的，所以第二个参数 cb 会出现对象的情况。

- 设置标记 options.user=true,表明这是一个 user-watcher。
- 再给 watch 设置了 immediate 属性后，会将实例化后得到的值传入回调，并立即执行一次回调函数，这也是 immediate 的实现原理。
- 最后的返回值是一个方法，执行后可以取消对该监听属性的监听。接下来我们看看 user-watcher 是如何定义的：

```javascript
class Watcher {
  constructor(vm, expOrFn, cb, options) {
    this.vm = vm
    vm._watchers.push(this)  // 添加到当前实例的watchers内

    if(options) {
      this.deep = !!options.deep  // 是否深度监听
      this.user = !!options.user  // 是否是user-wathcer
      this.sync = !!options.sync  // 是否同步更新
    }

    this.active = true  // // 派发更新的标志位
    this.cb = cb  // 回调函数

    if (typeof expOrFn === 'function') {  // 如果expOrFn是函数
      this.getter = expOrFn
    } else {
      this.getter = parsePath(expOrFn)  // 如果是字符串对象路径形式，返回闭包函数
    }

    ...

  }
}
```

> 当是 user-watcher 时，Watcher 内部是以上方式实例化的，通常情况下我们是使用字符串的形式创建监听属性，所以首先来看下 parsePath 方法是干什么的：

```javascript
const bailRE = /[^\w.$]/; // 得是对象路径形式，如info.name

function parsePath(path) {
  if (bailRE.test(path)) return; // 不匹配对象路径形式，再见

  const segments = path.split('.'); // 按照点分割为数组

  return function (obj) {
    // 闭包返回一个函数
    for (let i = 0; i < segments.length; i++) {
      if (!obj) return;
      obj = obj[segments[i]]; // 依次读取到实例下对象末端的值
    }
    return obj;
  };
}
```

> parsePath 方法最终返回一个闭包方法，此时 Watcher 类中的 this.getter 就是一个函数了，再执行 this.get()方法时会将 this.vm 传入到闭包内，补全 Watcher 其他的逻辑：

```javascript
class Watcher {
  constructor(vm, expOrFn, cb, options) {

    ...
    this.getter = parsePath(expOrFn)  // 返回的方法

    this.value = this.get()  // 执行get
  }

  get() {
    pushTarget(this)  // 将当前user-watcher实例赋值给Dep.target，读取时收集它

    let value = this.getter.call(this.vm, this.vm)  // 将vm实例传给闭包，进行读取操作

    if (this.deep) {  // 如果有定义deep属性
      traverse(value)  // 进行深度监听
    }

    popTarget()
    return value  // 返回闭包读取到的值，参数immediate使用的就是这里的值
  }

  ...

}
```

> 因为之前初始化已经将状态已经全部都代理到了 this 下，所以读取 this 下的属性即可，比如：

```javascript
export default {
  data() {  // data的初始化先与watch
    return {
      info: {
        name: 'cc'
      }
    }
  },
  created() {
    this.$watch('info.name', newName => {...})  // 何况手动创建
  }
}
```

> 首先读取 this 下的 info 属性，然后读取 info 下的 name 属性。大家注意，这里我们使用了读取这个动词，所以会执行之前包装 data 响应式数据的 get 方法进行依赖收集，将依赖收集到读取到的属性的 dep 里，不过收集的是 user-watcher，get 方法最后返回闭包读取到的值。

> 之后就是当 info.name 属性被重新赋值时，走派发更新的流程，我们这里把和 render-watcher 不同之处做单独的说明，派发更新会执行 Watcher 内的 update 方法内：

```javascript
class Watcher {
  constructor(vm, expOrFn, cb, options) {
    ...
  }

  update() {  // 执行派发更新
    if(this.sync) {  // 如果有设置sync为true
      this.run()  // 不走nextTick队列，直接执行
    } else {
      queueWatcher(this)  // 否则加入队列，异步执行run()
    }
  }

  run() {
    if (this.active) {
      this.getAndInvoke(this.cb)  // 传入回调函数
    }
  }

  getAndInvoke(cb) {
    const value = this.get()  // 重新求值

    if(value !== this.value || isObject(value) || this.deep) {
      const oldValue = this.value  // 缓存之前的值
      this.value = value  // 新值
      if(this.user) {  // 如果是user-watcher
        cb.call(this.vm, value, oldValue)  // 在回调内传入新值和旧值
      }
    }
  }
}
```

> 其实这里的 sync 属性已经没在官网做说明了，不过我们看到源码中还是保留了相关代码。接下来我们看到为什么 watch 的回调内可以得到新值和旧值的原理，因为 cb.call(this.vm, value, oldValue)这句代码的原因，内部将新值和旧值传给了回调函数。

```javascript
// watch监听属性示例：
export default {  // App组件
  data() {
    return {
      name: 'cc'
    }
  },
  watch: {
    name(newName, oldName) {...}  // 派发新值和旧值给回调
  },
  mounted() {
    setTimeout(() => {
      this.name = 'ww'  // 触发name的set
    }, 1000)
  }
}
```

## 监听属性的 deep 深度监听原理

之前的 get 方法内有说明，如果有 deep 属性，则执行 traverse 方法：

```javascript
const seenObjects = new Set(); // 不重复添加

function traverse(val) {
  _traverse(val, seenObjects);
  seenObjects.clear();
}

function _traverse(val, seen) {
  let i, keys;
  const isA = Array.isArray(val); // val是否是数组

  if (
    (!isA && !isObject(val)) || // 如果不是array和object
    Object.isFrozen(val) || // 或者是已经冻结对象
    val instanceof VNode
  ) {
    // 或者是VNode实例
    return; // 再见
  }

  if (val.__ob__) {
    // 只有object和array才有__ob__属性
    const depId = val.__ob__.dep.id; // 手动依赖收集器的id
    if (seen.has(depId)) {
      // 已经有收集过
      return; // 再见
    }
    seen.add(depId); // 没有被收集，添加
  }

  if (isA) {
    // 是array
    i = val.length;
    while (i--) {
      _traverse(val[i], seen); // 递归触发每一项的get进行依赖收集
    }
  } else {
    // 是object
    keys = Object.keys(val);
    i = keys.length;
    while (i--) {
      _traverse(val[keys[i]], seen); // 递归触发子属性的get进行依赖收集
    }
  }
}
```

> 看着还挺复杂，简单来说 deep 的实现原理就是递归的触发数组或对象的 get 进行依赖收集，因为只有数组和对象才有**ob**属性，也就是我们第七章说明的手动依赖管理器，将它们的依赖收集到 Observer 类里的 dep 内，完成 deep 深度监听。

## 总结

> 这里说明了为什么 watch 和 this.$watch 的实现是一致的，以及简单解释它的原理就是为需要观察的数据创建并收集 user-watcher，当数据改变时通知到 user-watcher 将新值和旧值传递给用户自己定义的回调函数。最后分析了定义 watch 时会被使用到的三个参数：sync、immediate、deep 它们的实现原理。简单说明它们的实现原理就是：sync 是不将 watcher 加入到 nextTick 队列而同步的更新、immediate 是立即以得到的值执行一次回调函数、deep 是递归的对它的子值进行依赖收集。

# computed

计算属性不是 API，但它是 Watcher 类,也是最复杂的一种实例化的使用。

分析计算属性为何可以做到当依赖发生变化时才会重新计算，否则数据被缓存。

计算属性的值也可以是对象，这个对象需要传入 get 和 set 方法

```javascript
export default {
  computed: {
    newName: {  // 不分析这种了~
      get() {...},  // 内部会采用get属性为计算属性的值
      set() {...}
    }
  }
}
```

## 计算属性初始化

```javascript
function initState(vm) {  // 初始化所有状态时
  vm._watchers = []  // 当前实例watcher集合
  const opts = vm.$options  // 合并后的属性

  ... // 其他状态初始化

  if(opts.computed) {  // 如果有定义计算属性
    initComputed(vm, opts.computed)  // 进行初始化
  }
  ...
}

---------------------------------------------------------------------------

function initComputed(vm, computed) {
  const watchers = vm._computedWatchers = Object.create(null) // 创建一个纯净对象

  for(const key in computed) {
    const getter = computed[key]  // computed每项对应的回调函数

    watchers[key] = new Watcher(vm, getter, noop, {lazy: true})  // 实例化computed-watcher

    ...

  }
}
```

## 计算属性实现原理

> 这里还是按照惯例，将定义的 computed 属性的每一项使用 Watcher 类进行实例化，不过这里是按照 computed-watcher 的形式，来看下如何实例化的：

```javascript
class Watcher {
  constructor(vm, expOrFn, cb, options) {
    this.vm = vm;
    this._watchers.push(this);

    if (options) {
      this.lazy = !!options.lazy; // 表示是computed
    }

    this.dirty = this.lazy; // dirty为标记位，表示是否对computed计算

    this.getter = expOrFn; // computed的回调函数

    this.value = undefined;
  }
}
```

> 这里的 App 组件在执行 extend 创建子组件的构造函数时，已经将 key 挂载到 vm 的原型中了，不过之前也是执行的 defineComputed 方法，所以不妨碍我们看它做了什么：

```javascript
function defineComputed(target, key) {
  ...
  Object.defineProperty(target, key, {
    enumerable: true,
    configurable: true,
    get: createComputedGetter(key),
    set: noop
  })
}
```

> 这个方法的作用就是让 computed 成为一个响应式数据，并定义它的 get 属性，也就是说当页面执行渲染访问到 computed 时，才会触发 get 然后执行 createComputedGetter 方法，所以之前的点到为止再这里会续上，看下 get 方法是怎么定义的：

```javascript
function createComputedGetter (key) { // 高阶函数
  return function () {  // 返回函数
    const watcher = this._computedWatchers && this._computedWatchers[key]
    // 原来this还可以这样用，得到key对应的computed-watcher

    if (watcher) {
      if (watcher.dirty) {  // 在实例化watcher时为true，表示需要计算
        watcher.evaluate()  // 进行计算属性的求值
      }
      if (Dep.target) {  // 当前的watcher，这里是页面渲染触发的这个方法，所以为render-watcher
        watcher.depend()  // 收集当前watcher
      }
      return watcher.value  // 返回求到的值或之前缓存的值
    }
  }
}

------------------------------------------------------------------------------------

class Watcher {
  ...

  evaluate () {
    this.value = this.get()  //  计算属性求值
    this.dirty = false  // 表示计算属性已经计算，不需要再计算
  }

  depend () {
    let i = this.deps.length  // deps内是计算属性内能访问到的响应式数据的dep的数组集合
    while (i--) {
      this.deps[i].depend()  // 让每个dep收集当前的render-watcher
    }
  }
}
```

> 这里的变量 watcher 就是之前 computed 对应的 computed-watcher 实例，接下来会执行 Watcher 类专门为计算属性定义的两个方法，在执行 evaluate 方法进行求值的过程中又会触发 computed 内可以访问到的响应式数据的 get，它们会将当前的 computed-watcher 作为依赖收集到自己的 dep 里，计算完毕之后将 dirty 置为 false，表示已经计算过了。

> 然后执行 depend 让计算属性内的响应式数据订阅当前的 render-watcher，所以 computed 内的响应式数据会收集 computed-watcher 和 render-watcher 两个 watcher，当 computed 内的状态发生变更触发 set 后，首先通知 computed 需要进行重新计算，然后通知到视图执行渲染，再渲染中会访问到 computed 计算后的值，最后渲染到页面。

> 当 computed 内的响应式数据变更后触发的通知：

```javascript
class Watcher {
  ...
  update() {  // 当computed内的响应式数据触发set后
    if(this.lazy) {
      this.diray = true  // 通知computed需要重新计算了
    }
    ...
  }
}
```

> 最后还是以一个示例结合流程图来帮大家理清楚这里的逻辑：

```javascript
export default {
  data() {
    return {
      manName: 'cc',
      womanName: 'ww'
    };
  },
  computed: {
    newName() {
      return this.manName + ':' + this.womanName;
    }
  },
  methods: {
    changeName() {
      this.manName = 'ss';
    }
  }
};
```

## 总结

> 为什么计算属性有缓存功能？因为当计算属性经过计算后，内部的标志位会表明已经计算过了，再次访问时会直接读取计算后的值；为什么计算属性内的响应式数据发生变更后，计算属性会重新计算？因为内部的响应式数据会收集 computed-watcher，变更后通知计算属性要进行计算，也会通知页面重新渲染，渲染时会读取到重新计算后的值。

[查看原文](https://juejin.cn/post/6844903926819454983#heading-0)
