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

- Key和Value都允许Null
- HashMap不保留加入数组时的顺序，顺序是由 `Hash Function`控制的
- 他不是异步操作的
- `NOT Thread safe`,但可以 `Collections.synchronizedMap(new HashMap<K,V>())`

> HashSet

- 加入Set时应该使用的`add` 而非`put`.
- `Contain`是可接受的，可用来保障一个unique list.

> HashTable

- Key和Value都`不`允许Null，会抛出 `NullPointerException`
- synchronized
- `Thread safe` 
- 使用 `Enumerator` 遍历


### 区别总结

- HashSet底层采用的是HashMap进行实现的，但是没有key-value，只有HashMap的key set的视图，HashSet不容许重复的对象
- Hashtable是基于Dictionary类的，而HashMap是基于Map接口的一个实现
- Hashtable里默认的方法是同步的，而HashMap则是非同步的，因此Hashtable是多线程安全的
- HashMap可以将空值作为一个表的条目的key或者value,HashMap中由于键不能重复，因此只有一条记录的Key可以是空值，而value可以有多个为空，但HashTable不允许null值(键与值均不行)
- 内存初始大小不同，HashTable初始大小是11，而HashMap初始大小是16
- 内存扩容时采取的方式也不同，Hashtable采用的是2*old+1,而HashMap是2*old。
- 哈希值的计算方法不同，Hashtable直接使用的是对象的hashCode,而HashMap则是在对象的hashCode的基础上还进行了一些变化

## 参考

[Kotlin](https://kotlinlang.org/)

[Kotlin开发](http://cdevlab.top/article/cc5968a9/)
