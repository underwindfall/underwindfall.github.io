---
layout:     post
title:      "Javascript 柯里化"
subtitle:   " \"基础知识\""
date:       2018-09-13 12:00:00
author:     "Undervoid"
header-img: "img/pg-javascript.gif"
catalog: true
tags:
    - 磨砻淬砺集
    - Javascript
    - 工作总结
---


# Javascript Curry 前言
`Javascript` 本身是个非常动态化的语言，对于现在前端而言的话，其实是非常是非常必要的，可以说是`JS`争抢了前端，移动端乃至后端的半壁江山。自己本身不太习惯`JS`这门语言，更偏向与`JAVA`之类的，但因为工作需要还是做些总结。函数的柯里化，听上去高大上的不行，仿佛计算机领域很多专业术语都强调了专业性而缺乏本身的可读和理解性，比如说之前提到的`RN`里面的`高阶组件(Higher Order Component)`，听上去高大上实质上就是个套了个壳子的`Container`。 同样的 `Curring`简单来说就是只传递一部分参数来调用它，让他去返回一个函数去处理剩下的参数。

## 准备工作

### Javascript 隐式转换
`Javascript` 作为一种弱类型语言，它的隐式转换是非常灵活有趣的。当我们没有深入了解隐式转换的时候可能会对一些运算的结果会感动困惑，比如`4 + true = 5`，`!!undefined = false`等。当然，如果你对隐式转换了解足够深刻，肯定是能够很大程度上提高你对js的使用能力。只是这里我没有打算将所有的隐式转换规则分享给大家，这里暂时只分享一下，函数在隐式转换中的一些规则。

### 实例

```javascript
function fn() {
    return 20;
}

console.log(fn + 10); 

//结果是
function fn() {
    return 20;
}10
```

```javascript
function fn() {
    return 20;
}

fn.toString = function() {
    return 10;
}

console.log(fn + 10);

//结果是
20
```

```javascript
function fn() {
    return 20;
}

fn.toString = function() {
    return 10;
}

fn.valueOf = function() {
    return 5;
}

console.log(fn + 10);

//结果是
15
```
首先我们要知道，当使用console.log，或者进行运算时，隐式转换就会发生。从上面三个相似的例子中我们可以得出一些结论。

>当我们没有重新定义toString与valueOf时，函数的隐式转换会调用默认的toString方法，它会将函数的定义内容作为字符串返回。而当我们主动定义了toString/vauleOf方法时，那么隐式转换的返回结果则由我们自己控制了。其中valueOf的优先级会toString高一点。

因此上面例子的结论就很容易理解了。建议大家动手尝试一下。

## Javascript Curry 介绍

## 实例

```javascript
const add = x=> y=> x+y;
const increment = add(1);
const addTen = add(10);

increment(2);
// 3

addTen(2);
// 12
```
我们定义了一个 `add` 函数，它接受一个参数并返回一个新的函数。调用 add 之后，返回的函数就通过闭包的方式记住了 `add` 的第一个参数。一次性地调用它实在是有点繁琐，好在我们可以使用一个特殊的 `curry` 帮助函数（`helper function`）使这类函数的定义和调用更加容易。掌握好了`curry`其实就是掌握了高阶函数




