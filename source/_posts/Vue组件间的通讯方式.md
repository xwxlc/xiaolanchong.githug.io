---
title: Vue组件间的通讯方式
date: 2023-04-08 12:44:03
categories: [面试题]
tags: [Vue]
---

# Vue 组件间的通讯方式

{% note warning %}
Vue 是单项数据流
{% endnote %}

## 父组件向子组件传递 props

父组件通过 props 向子组件传递数据，子组件通过 $emit 和父组件通信

> 父组件

```html
<script setup>
  import { ref } from 'vue';
  import Child from './components/Child.vue';

  const msg = ref('msg');
  const changeMsg = (str) => {
    msg.value = str;
  };
</script>

<template>
  <Child :msg="msg" :fn="changeMsg" />
</template>

<style scoped></style>
```

> 子组件

```html
<script setup>
  defineProps({
    msg: String,
    fn: Function
  });
</script>

<template>
  <div>{{ msg }}</div>
  <button @click="fn('change msg')">change msg</button>
</template>

<style scoped></style>
```

## 子组件向父组件传递 $emit

$emit 的特点

- $emit 绑定一个自定义事件，当事件被执行的时候会将参数传递给父组件，而父组件通过 v-on 监听并接受参数

> 父组件

```html
<script setup>
  import { ref } from 'vue';
  import Child from './components/Child.vue';

  const msg = ref('msg');
  const changeMsg = (str) => {
    msg.value = str;
  };
</script>

<template>
  <Child :msg="msg" @fn="changeMsg" />
</template>

<style scoped></style>
```

> 子组件

```html
<script setup>
  defineProps({
    msg: String
  });
  defineEmits(['fn']);
</script>

<template>
  <div>{{ msg }}</div>
  <button @click="$emit('fn', 'change msg')">change msg</button>
</template>

<style scoped></style>
```

## 兄弟组件传递

1. 兄弟组件间通信可以借助父组件交互

## ref / $refs

该方法也是实现父子组件通信的

ref: 该属性用在子组件上，他指向子组件的实例，可以通过实例来访问组件的数据和方法

> 子组件

```html
<script setup>
  import { ref } from 'vue';
  const msg = ref('msg');
  const changeMsg = (str) => {
    msg.value = str;
  };
  // vue3需要主动暴露
  defineExpose({
    msg,
    changeMsg
  });
</script>

<template>
  <div>{{ msg }}</div>
</template>

<style scoped></style>
```

> 父组件

```html
<script setup>
  import { ref } from 'vue';
  import Child from './components/Child.vue';
  const refChild = ref(null);
  const changeMsg = (str) => {
    // onMounted后才能获取到
    refChild.value?.changeMsg(str);
  };
</script>

<template>
  <Child ref="refChild" />
  <button @click="changeMsg('change msg')">change msg</button>
</template>

<style scoped></style>
```

# 依赖注入 provide / inject

该方法就是 vue 中的依赖主人，该方法用于父子/祖孙组件间的通信

provide/inject 是 vue 提供的两个钩子

- provide 用来发送数据或方法
- inject 用来接受数据或方法

> 父组件

```html
<script setup>
  import { ref, provide } from 'vue';
  import Child from './components/Child.vue';
  const msg = ref('msg');
  const changeMsg = (str) => {
    msg.value = str;
  };
  provide('msg', msg);
  provide('changeMsg', changeMsg);
</script>

<template>
  <div>{{ msg }}</div>
  <Child />
</template>

<style scoped></style>
```

> 子组件

```html
<script setup>
  import { ref, inject } from 'vue';
  const msg = inject('msg');
  const changeMsg = inject('changeMsg');
</script>

<template>
  <div>{{ msg }}</div>
  <button @click="changeMsg('change msg')">change msg</button>
</template>

<style scoped></style>
```

## $parent / $children

> $parent

> $children（vue3 已移除）

```javascript
this.$children[0].message = 'JavaScript';
```

## vuex

## eventBus

## mitt

## $attrs $listeners
