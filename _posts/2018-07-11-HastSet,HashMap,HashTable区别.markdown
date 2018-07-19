---
layout:     post
title:      "HastSet,HashMap,HashTable区别"
subtitle:   " \"Java基础知识\""
date:       2018-07-11 12:00:00
author:     "Undervoid"
header-img: "img/java.png"
catalog: true
tags:
    - 答疑解惑集
    - JAVA
    - 面试总结
---


## Introduction

下面将介绍下 **Kotlin** 几个基本的语法糖：

> HastMap

官方文档解释为：「**Fold** Accumulates value starting with initial value and applying operation from left to right to current accumulator value and each element. 」

**Fold** 方法就是给定初值和迭代方法， 然后通过迭代对集合中的每一个元素执行指定的操作并将操作的结果相累加。这个操作函数的两个参数就是累加结果和集合的元素。

```Kotlin
    public inline fun <T, R> Iterable<T>.fold(initial: R, operation: (R, T) -> R): R {
        var accumulator = initial
        for (element in this) accumulator = operation(accumulator, element)
        return accumulator
    }

```

> HashSet

官方文档解释为： 「**Map** Returns a list containing the results of applying the given transform function to each entry in the original map.」

**Map** 就是将指定的转换函数运用到原始集合的每一个元素，并返回到一个转换后的集合。
```Kotlin
    inline fun <K, V, R> Map<out K, V>.map(
        transform: (Entry<K, V>) -> R
    ): List<R> (source)
```

> HashTable

官方文档解释为：「**Filter** Returns a list containing only elements matching the given predicate).」

Filter filter方法返回一个包含所有满足指定条件元素的列表。与之对应的还有filterNot，顾名思义就是返回一个包含所有不满足指定条件的元素列表。还有一个filterNotNull，返回所有不为null的元素列表。

```Kotlin
    inline fun <K, V> Map<out K, V>.filter(
        predicate: (Entry<K, V>) -> Boolean
    ): Map<K, V> (source)
```


## 编译器常数值

如果属性值在编译期间就能确定, 则可以使用 const 修饰符, 将属性标记为 编译期常数值(compile time constants). 这类属性必须满足以下所有条件:

- 必须是顶级属性, 或者是一个 object 的成员
- 值被初始化为 String 类型, 或基本类型(primitive type)
- 不存在自定义的取值方法

## lateinit延迟初始化属性

lateinit 关键字表示当前 变量 不会在声明时进行初始化操作，初始化操作会在后面进行，像是一种协议机制，告知编译器我会在后面使用该变量之前的恰当时机初始化该变量，不要进行警告⚠️。在一个 lateinit 属性被初始化之前访问它， 会抛出一个特别的异常，这个异常将会指明被访问的属性，以及它没有被初始化这一错误。

需要注意的是使用 lateinit 关键字有很多限制:

- 必须是变量，即使用 var 关键字进行声明。
- 不能修饰可为 null 的类型，比如 lateinit var str:String? 是编译不通过的。
- 不能修饰基本数据类型。例如 Int,Float等



## by lazy代理-属性延迟初始化

by lazy 是属性代理的基本运用，是经过简化后的属性代理，他为属性提供初始化方法。这里不扩展属性代理的相关问题。因为他只提供取值方法所以仅可以用在常量的初始化中，使用 by lazy 当属性被第一次访问时，就会触发初始化流程。

- 只有常量，也就是使用 val才能使用 by lazy 延迟初始化

```Kotlin
    val stuByLazy:Student by lazy {
        Student("name",11)
    }
    val mMyMsgTv:TextView by lazy {
        findViewById(R.id.mTestTv) as TextView
    }
    // 当常量被使用时才进行初始化
    logError(stuByLazy.myCls)
    mMyMsgTv.text="文本"
```

### Kotlin lateinit vs. by lazy

- lazy{} 只能用在val类型, lateinit 只能用在var类型
- lateinit不能用在可空的属性上和java的基本类型上 然而lazy{}常被用在常用java 基本类型上
- lateinit可以在任何位置初始化并且可以初始化多次。而lazy在第一次被调用时就被初始化，想要被改变只能重新定义
- lateinit 有支持（反向）域（Backing Fields）

## 参考

[Kotlin](https://kotlinlang.org/)

[Kotlin开发](http://cdevlab.top/article/cc5968a9/)
