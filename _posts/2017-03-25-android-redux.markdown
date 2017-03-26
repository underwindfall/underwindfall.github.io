---
layout:     post
title:      "Redux Android初体验"
subtitle:   " \"Unidirectional data flow in Android App\""
date:       2017-03-25 12:00:00
author:     "Undervoid"
header-img: "img/post-bg-android.jpg"
catalog: true
tags:
    - 磨砻淬砺集
    - Android开发
    - Redux
---

# Redux 初探

>对我来说，生活不是为了去换取成功，不是为了去超越别人。是一种想去体验一个更大的世界的欲望。以此勉励自己。

## 概念

「Unidirectional data flow - Redux」是一种数据结构形式，是一系列的状态组合。Redux的核心思想就是要提供*可预测*的状态管理，这对日益膨胀的大型应用来说尤其重要。他其实与[React](https://facebook.github.io/react/)有着不解之缘，但自己本身不面向前端发展，就不具体介绍 「React」了，感兴趣的朋友可以去[React官网](https://facebook.github.io/react/)仔细阅读。自己写下这篇文章算是对最近使用[Redux结构](http://redux.js.org/)在 Android 应用的一个总结吧。

## Redux基本知识

Redux 中其实可以分为四个基础部分：State/Store/Reducer/Action 但官网上解释为只有 Store/Reducer/Action 三个部分，但是自己觉得是应该加入 State 的。我理解的详情如下：

> Action 

官方文档解释为：「**Actions** are payloads of information that send data from your application to your store. They are the only source of information for the store. You send them to the store using store.dispatch().」

自己关于 Action 的理解是： Action 是把数据从应用送到 Store 一种负载形式。它可以通过 store.dispatch() 的方法来把 action 传递给 store。在我自己看来它可以来分辩 Action 是一种 Action 还是一种 AsyncAction，用户可以通过 UI 操作来激活 Action 来进行具体操作。

> Reducer

官方文档解释为： 「Actions describe the fact that something happened, but don’t specify how the application’s state changes in response. This is the job of a **reducer**.」

自己关于Reducer的理解是：Reducer 是 Action 只是描述了**有事情发生了**这一事实，并没有指明应用如何更新 state。而这正是 reducer 要做的事情。它是一个回调函数，它一般都会有两个参数 State 和 Action 会根据 Action 的 type 来对旧的 state 进行操作返回新的 state。

> Store

官方文档解释为：「The **Store** is the object that brings them together. The store has the following responsibilities:

- Holds application state;
- Allows access to state via [`getState()`](http://redux.js.org/docs/api/Store.html#getState);
- Allows state to be updated via [`dispatch(action)`](http://redux.js.org/docs/api/Store.html#dispatch);
- Registers listeners via [`subscribe(listener)`](http://redux.js.org/docs/api/Store.html#subscribe);」

自己关于Reducer的理解是：在 Redux 中，整个程序的状态都是通过一个单一的 **Store** 来管理的。重点是 **Store** 是唯一的在 Redux 应用当中。当需要拆分数据处理逻辑时，你应该使用而不是创建多个 store。对于 Reducer 和 Action 是如何联系在一起的，我没有研究但感觉上 Store 已经解决了这个担忧。

> State

官方文档解释为： 「**State** (also called the *state tree*) is a broad term, but in the Redux API it usually refers to the single state value that is managed by the store and returned by [`getState()`](http://redux.js.org/docs/api/Store.html#getState). It represents the entire state of a Redux application, which is often a deeply nested object.」

自己关于Reducer的理解是：在 Redux 应用中，应用中都是以 **State** 的形式存储程序状态，然后每次更新 UI 的部分，都要通过 State 获取。

## Redux Android 结构

> **Redux in Android Architecture :**

![](https://s3.eu-central-1.amazonaws.com/undervoidfall/blog/redux_android.jpg)

通过上图我们可以发现，在「Redux-Android」应用中，用户通过 UI 操作来触发 Action 的方法， 由它来进行判断和执行出发 Reducer操作更新 Appstate。 Appstore 更像是一辆载满了 Appstate 的公交车，Android端来通过 Appstate.getStore() 的方式获取最新的应用状态从而达到更新界面的效果。

## Redux Data Lifecycle

**严格的单向数据流**是 Redux 架构的设计核心。

这意味着应用中所有的数据都遵循相同的生命周期，这样可以让应用变得更加可预测且容易理解。同时也鼓励做数据范式化，这样可以避免使用多个且独立的无法相互引用的重复数据。

![](https://s3.eu-central-1.amazonaws.com/undervoidfall/blog/redux_data_lifecycle_simple.png)





## Redux Android 中间件 (Middleware) 

> It provides a third-party extension point between dispatching an action, and the moment it reaches the reducer.

Middleware 是一个插件，他被应用于 Action 被唤醒去 dispatch.store() 和 store即将使用 reducer 使用。本质的目的是提供第三方插件的模式，自定义拦截 **action -> reducer ** 的过程。变为 **action -> middlewares -> reducer** 。这种机制可以让我们改变数据流，实现如异步 action ，action 过滤，日志输出，异常报告等功能。

![](https://s3.eu-central-1.amazonaws.com/undervoidfall/blog/redux-middleware.png)

 

也就是说现在的 **Redux 数据流声明周期**变成了

![](https://s3.eu-central-1.amazonaws.com/undervoidfall/blog/redux_data_lifecycle_complex.png)



## Android Redux Code Demo

自己尝试用 [Kotlin](!https://kotlinlang.org/) 和 [redux-Kotlin](!https://github.com/pardom/redux-kotlin)搭建了一个简单的关于Redux在Android的应用，它能够通过输入 Github 用户名获取其相关的仓库。算是给大家一个小常识吧。

[Demo地址](!https://github.com/underwindfall/kotlin_redux_interview)

## 参考

[Redux Middleware: Behind the Scene]: http://briantroncone.com/?p=529
[Understanding Redux Middleware]: https://medium.com/@meagle/understanding-87566abcfb7a#.r7x2ierw5
[How Redux works]: http://arqex.com/1087/using-redux-devtools-without-redux





