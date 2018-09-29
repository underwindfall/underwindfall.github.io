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


## Javascript call && apply函数
两者的函数作用其实基本一模一样，只是传参的形式有区别而已。 感觉都是用来感觉context的作用
> apply()
apply 方法传入两个参数：一个是作为函数上下文的对象，另外一个是作为函数参数所组成的数组

```javascript
var obj = {
    name : 'linxin'
}

function func(firstName, lastName){
    console.log(firstName + ' ' + this.name + ' ' + lastName);
}

func.apply(obj, ['A', 'B']);    // A linxin B
```
可以看到，obj 是作为函数上下文的对象，函数 func 中 this 指向了 obj 这个对象。参数 A 和 B 是放在数组中传入 func 函数，分别对应 func 参数的列表元素

> call()
call 方法第一个参数也是作为函数上下文的对象，但是后面传入的是一个参数列表，而不是单个数组。

```javascript
var obj = {
    name: 'linxin'
}

function func(firstName, lastName) {
    console.log(firstName + ' ' + this.name + ' ' + lastName);
}

func.call(obj, 'C', 'D');       // C linxin D
```
对比 apply 我们可以看到区别，C 和 D 是作为单独的参数传给 func 函数，而不是放到数组中。

对于什么时候该用什么方法，其实不用纠结。如果你的参数本来就存在一个数组中，那自然就用 apply，如果参数比较散乱相互之间没什么关联，就用 call。

### apply 和 call 的用法
- 改变 this 指向

```javascript
var obj = {
    name: 'linxin'
}

function func() {
    console.log(this.name);
}

func.call(obj);       // linxin
```
我们知道，call 方法的第一个参数是作为函数上下文的对象，这里把 obj 作为参数传给了 func，此时函数里的 this 便指向了 obj 对象。此处 func 函数里其实相当于

```javascript

function func() {
    console.log(obj.name);
}
```


- 借用别的对象的方法

```javascript
var Person1  = function () {
    this.name = 'linxin';
}
var Person2 = function () {
    this.getname = function () {
        console.log(this.name);
    }
    Person1.call(this);
}
var person = new Person2();
person.getname();       // linxin
```

从上面我们看到，Person2 实例化出来的对象 person 通过 getname 方法拿到了 Person1 中的 name。因为在 Person2 中，Person1.call(this) 的作用就是使用 Person1 对象代替 this 对象，那么 Person2 就有了 Person1 中的所有属性和方法了，相当于 Person2 继承了 Person1 的属性和方法。

- 调用函数
`apply`、`call` 方法都会使函数立即执行，因此它们也可以用来调用函数。

```javascript
function func() {
    console.log('linxin');
}
func.call();            // linxin
```

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

之前听老大也讲了一道面试题, 写一个 `add`函数使其能够满足这个需求

```javascript
add(1)(2)(3) = 6
add(1, 2, 3)(4) = 10
add(1)(2)(3)(4)(5) = 15

```

>`curry`(柯里化)为部分求值，是把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回一个新的函数的技术，新函数接受余下参数并返回运算结果。


```javascript
//答案

const add=()=> {
    // 第一次执行时，定义一个数组专门用来存储所有的参数
    const _args = [].slice.call(arguments);

    // 在内部声明一个函数，利用闭包的特性保存_args并收集所有的参数值
    const adder =  () =>{
        const _adder = ()=> {
            [].push.apply(null, [].slice.call(arguments));
            return _adder;
        };

        // 利用隐式转换的特性，当最后执行时隐式转换，并计算最终的值返回
        _adder.toString =  ()=> {
            return _args.reduce(function (a, b) {
                return a + b;
            });
        }

        return _adder;
    }
    return adder.apply(null, _args);
}
```