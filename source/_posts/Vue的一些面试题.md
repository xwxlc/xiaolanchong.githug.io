---
title: Vue的一些面试题
date: 2023-04-08 16:03:29
categories: [面试题]
tags: [Vue]
---

# Vue 的生命周期

> 老常规第一问

- Vue2：beforeCreate => created => beforeMount => mounted => beforeUpdate => updated => beforeDestroy => destroyed
- Vue3：setup => onBeforeMount => onMount => onBeforeUpdate => onUpdated => onBeforeUnmount => onUnmounted

> keep-alive

- Vue2：activated 、deactivated
- Vue3：onActivated 、onDeactivated

> 捕获一个来自子孙组件的异常

- vue2：errorCaptured
- vue3：onErrorCaptured

# 父子生命周期的顺序

子组件先挂载，然后父组件，更新也类似

# v-model 原理

v-model 是 vue 的语法糖

input[text/number/password]、textarea 元素使用的 value 属性和 input 事件

input[checkbox/radio]元素使用的 checked 属性和 change 事件

select 元素将 value 作为 prop 并将 change 作为事件

# 为什么 data 是个函数

因为组件会被复用需要保证唯一性

# Keep-alive 的了解

keep-alive 是 vue 内部的组件 可以使被包含的组件保留状态，避免重新渲染

1.  一般结合路由和动态组件一起使用，用于缓存组件
2.  提供的 include 和 exclude 属性
    - include => 只有匹配的会缓存
    - exclude => 匹配的不会缓存 优先级比 include 高
3.  对象两个特殊钩子函数 activated 和 deactivated

# computed 和 watch 的区别和运用场景

- computed:计算属性,有缓存,当我们需要进行数值计算，并且依赖于其他数据时使用
- watch:监听,当我们需要异步或开销较大的操作时使用,可以根据需要限制我们操作的频次

{% post_link computed和watch的区别 %}

# 聊聊 nextTick

1. 把回调函数放入 callbacks 等待执行
2. 将执行函数放入微任务或宏任务中
   > 为什么是微任务或宏任务？
   > 对当前环境进行了降级处理 Promise => MutationObserver => setImmediate => setTimeout (1、2 微任务 3、4 宏任务)
3. 事件循环到微任务或宏任务，执行函数依次执行 callbacks 中的回调

# render

> 为什么使用 render 函数？

VUE 推荐在绝大多数情况下使用 template 来创建我们的 HTML。然而在一些场景中，我们真的需要 JavaScript 的完全编程的能力，这就是 render 函数，它比 template 更接近编译器。（这是官方的话）
简单来说，我们为什么要使用 render 函数呢？？ 便是因为我们最经常使用的一个引入。

> render 函数的解析

render 函数即渲染函数，它是个函数，它的参数 createElement 也是个函数。

createElement 有三个参数

1. 一个 HTML 标签字符串 必选
2. 一个包含模板相关属性的数据对象 可选
3. 子虚拟节点 可选

[查看原文](https://blog.csdn.net/m0_59722204/article/details/126907040)

# v-show 与 v-if 的区别

- v-if 是条件渲染 确保在切换过程中条件块内的事件监听器和子组件适当地被销毁重建
- v-show 元素总会被渲染，且只是简单地基于 display 属性进行切换

# Class 与 Style 如何动态绑定

- 对象语法
- 数组语法

# vue-router 路由模式有几种

1. hash location.hash 向服务端发出请求是 hash 部分不会被发送
2. history HTML5 提供了 History API 来实现 URL 的变化
   - history.pushState()
   - history.repalceState()
3. abstract 支持所有 JavaScript 运行环境，如 Node.js 服务器端。如果发现没有浏览器的 API，路由会自动强制进入这个模式.

# SPA 单页的理解 优缺点

- SPA 仅在页面初始化时加载相应的 html、js、css 等
- SPA 不会因为用户的操作而进行页面的重新加载和跳转；取而代之的是利用路由机制实现页面内容的变化

> 优点：

1. 用户体验好、快、内容的改变不需重新加载整个页面，避免不必要的跳转和重新渲染
2. SPA 相对服务器压力小；
3. 前后端分离，架构清晰，前端负责交互逻辑，后端提供相应的数据接口；

> 缺点：

1. 首屏加载等待问题；
2. SEO 难度大

> SPA

1. CSR 客户端渲染=>当前主流模式
2. SSR 服务端渲染=>能较好的解决 SEO 的问题
3. Prerender 预渲染=>打包阶段预先生成页面
4. 同构 两者的结合=> 参考框架 Nuxt.js

{% note warning %}
注：SSR 需要路由需要使用 history 模式
{% endnote %}
