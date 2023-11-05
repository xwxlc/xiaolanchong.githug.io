---
title: CSS3特性（1）
categories:
  - currentColor 
  - isolation
  - scroll-padding-top
  - font-variant
  - caret-color
  - ::selection
  - user-select
  - ::first-letter
  - -webkit-box-reflect
tags:
date: 2023-11-04 13:18:53
updated:
---

# currentColor
> currentColor 经常被叫第一个属性，从字面意思理解是“当前颜色”。currentColor的值相当于当前color的值

- 使用技巧 1
``` css
.box-style {
  color: #fff;
  border-color: currentColor;
  background-color: currentColor;
  ...
}
.box-style.red {
  color: red
}
.box-style.blue {
  color: blue
}
```

- 使用技巧 2
> 我们可以利用该特性 巧妙的控制svg的颜色

原先对svg的颜色进修改需要 icon>path>fill 深乘次的选择器去控制
``` html 
<svg class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" width="50" height="50">
  <path fill="red" d="M533.504 268.288q33.792-41.984 71.68-75.776 32.768-27.648 74.24-50.176t86.528-19.456q63.488 5.12 105.984 30.208t67.584 63.488 34.304 87.04 6.144 99.84-17.92 97.792-36.864 87.04-48.64 74.752-53.248 61.952q-40.96 41.984-85.504 78.336t-84.992 62.464-73.728 41.472-51.712 15.36q-20.48 1.024-52.224-14.336t-69.632-41.472-79.872-61.952-82.944-75.776q-26.624-25.6-57.344-59.392t-57.856-74.24-46.592-87.552-21.504-100.352 11.264-99.84 39.936-83.456 65.536-61.952 88.064-35.328q24.576-5.12 49.152-1.536t48.128 12.288 45.056 22.016 40.96 27.648q45.056 33.792 86.016 80.896z"></path>
</svg>
<style>
  .icon path {
    fill: blue
  }
</style>
```
如今
``` html css
<svg class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" width="50" height="50">
  <path fill="currentColor" d="M533.504 268.288q33.792-41.984 71.68-75.776 32.768-27.648 74.24-50.176t86.528-19.456q63.488 5.12 105.984 30.208t67.584 63.488 34.304 87.04 6.144 99.84-17.92 97.792-36.864 87.04-48.64 74.752-53.248 61.952q-40.96 41.984-85.504 78.336t-84.992 62.464-73.728 41.472-51.712 15.36q-20.48 1.024-52.224-14.336t-69.632-41.472-79.872-61.952-82.944-75.776q-26.624-25.6-57.344-59.392t-57.856-74.24-46.592-87.552-21.504-100.352 11.264-99.84 39.936-83.456 65.536-61.952 88.064-35.328q24.576-5.12 49.152-1.536t48.128 12.288 45.056 22.016 40.96 27.648q45.056 33.792 86.016 80.896z"></path>
</svg>
<style>
  .icon {
    color: blue
  }
</style>
```

# isolation
> isolation的意思是脱离，在实际开发中通常会遇到z-index设置导致组件间出现了不合理的覆盖问题，通常在类似于tooltip的组件在设计时，通常会对z-index进行累加达到不被覆盖的目的

```css
.box-1 {
  isolation: isolate;
}
.box-2 {
  isolation: isolate;
}
```

# scroll-padding-top
> 当遇到顶部有header固定的页面布局时，如果页面中加入锚点链接的跳转动作，不做处理的情况下点击锚点链接跳转的位置会被header覆盖一部分区域。

> 这个时候scroll-padding-top属性就派上用场了。scroll-padding-top属性用来定义滚动窗口的最佳查看区域对于顶部的偏移量。我们可以通过给html设置scroll-padding-top的值为一个固定的高度来解决这个问题。

``` css
html {
  scroll-padding-top: 100px;
}
```

# font-variant

https://developer.mozilla.org/zh-CN/docs/Web/CSS/font-variant

# caret-color
> 自定义光标颜色

``` css
input {
   caret-color: red;
}
```

# ::selection
> 伪类选择器 表示文档中被用户高亮的部分

``` css
div::selection {
  background: rgb(91,60,146);
  color: #fff;
}
```

# user-select
> 禁止用户选择 & 可以整段选择

``` css
.none {
  user-select:none;
}
.text {
  user-select: text;
}
.all {
  user-select: all;
}
```

# ::first-letter
> 伪类选择器 表示某块级元素第一行的第一个字母（块级元素）
``` css
div::first-letter {
  font-size: 40px;
  font-style:italic;
  color: aquamarine;
}
```

# -webkit-box-reflect
> 这个属性可以在不同方向反射元素的内容。有了这个属性，我们就可以实现一些神奇的效果，比如镜像、倒影等。

``` css
// 左右镜像
.left {
  -webkit-box-reflect:right
}
// 倒影
.below {
  -webkit-box-reflect:below
}
```