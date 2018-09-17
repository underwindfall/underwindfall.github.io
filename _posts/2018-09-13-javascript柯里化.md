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




