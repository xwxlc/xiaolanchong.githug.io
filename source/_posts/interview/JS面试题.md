---
title: JS面试题
categories:
  - 面试题
tags:
  - JS
date: 2023-04-09 16:23:28
updated:
---

# New 操作符做了什么

1. 创建一个空对象
2. 改变隐式原型的指向
3. 改变构造函数 this 的指向
4. 判断构造函数有无返回的对象

```javascript
function Child(name){
    this.name= name
}

function _new(f,…arg){
	const o = {}; //1
	o.__proto__  = f.prototype;//2
	const r = f.appy(o, arg);//3
	const isTrue = r && typeof r === ‘function’ || typeof r === ‘object’;//4
	return isTrue ? r : o;
}
```

# Js 检查数据类型

## 1. typeof

> 缺点：只能检查基本类型 null 类型的时候是 object

```javascript
typeof undefined; //undefined
typeof null; // object
typeof true; // boolean
typeof 1; // number
typeof 1n; // bigInt
typeof 'str'; // string
typeof Symbol(); // symbol
typeof new Function(); // function
typeof NaN; // number
typeof {}; // object
```

### Q：为什么 typeof 检查 null 类型的时候是 object

> A：JavaScript 存储是以 32 位字节进行存储的，而标志存储的时候遵循如下特定

```
000：对象，数据是对象的应用。
001：整型，数据是 31 位带符号整数。
010：双精度类型，数据是双精度数字。
100：字符串，数据是字符串。
110：布尔类型，数据是布尔值。
null 作为机器代码 0 号地址，也就是全部都是空，所以检测：typeof null => object
```

## 2. instanceof

> 用于检测构造函数的 prototype 属性是否出现在某个实例对象的原型链上

> 缺点：不能检测基本类型

```
object instanceof constructor
object：某个实例对象
constructor：某个构造函数
```

### Q: instanceof 检测原理

```
拿到左边对象的原型链;
拿到右边构造函数的显示原型 prototype;
如果原型链中存在显示原型 返回 true
```

## 3. Object.prototype.toString.call()

```javascript
Object.prototype.toString.call(1); // [object Number]
Object.prototype.toString.call(1n); // [object BigInt]
Object.prototype.toString.call('str'); // [object String]
Object.prototype.toString.call(true); // [object Boolean]
Object.prototype.toString.call(undefined); // [object Undefined]
Object.prototype.toString.call(null); // [object Null]
Object.prototype.toString.call([]); // [object Array]
Object.prototype.toString.call({}); // [object Object]
Object.prototype.toString.call(new Function()); // [object Function]
Object.prototype.toString.call(Symbol()); // [object Symbol]
Object.prototype.toString.call(new Map()); // [object Map]
```

# for in 与 for of 的区别

- for in

> 对<font color="#ffff00">普通对象遍历其可枚举属性</font>获取键名

普通对象>可枚举属性

```javascript
// enumerable == true
Object.defineProperty(obj, 'a', { enumerable: false });
```

- for of

> 对<font color="#ffff00">可迭代对象</font>循环迭代获取其键值

可迭代对象

```javascript
// Array Map Set String
```
