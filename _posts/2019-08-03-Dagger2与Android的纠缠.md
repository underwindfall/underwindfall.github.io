---
layout:     post
title:      "Dagger2与Android的纠缠"
subtitle:   "依赖注入"
date:       2019-08-03 12:00:00
author:     "Undervoid"
header-img: "img/dagger2.jpg"
catalog: true
tags:
    - 磨砻淬砺集
    - Dagger
    - Android
---


# Android Dagger
> 简单的分析一下**dagger**是怎么实现依赖注入的过程的

## Dagger

1. Component 是连接 **@Inject** 所需要的变量和变量所需要的构造器之间的链接桥梁
![Component](https://raw.githubusercontent.com/underwindfall/blogAssets/master/blog/dagger/component.png)
- @BindsInstance
这个是目前dagger中比较推荐使用的一种绑定**Component**的方法，他主要的作用是在定义了生成的**DaggerComponent**的*builder*
方法，这样的所定义的**Component**就会持有你所提供的**parameter**参数，这样在你所需要的module中就不再需要了。

>Talk is cheap show me the code.
假设`AppComponent` 像是如此

```kotlin
@Singleton
@Component(
    modules = [
        TestAModule::class
        TestBModule::class
    ]
)
interface AppComponent {
    fun inject(app: App)

    @Component.Builder
    interface Builder {

        @BindsInstance
        fun application(application: Application): Builder

        fun build(): AppComponent
    }
}
```
这里就意味着`AppComponent`所*generate*的就会提供一个application作为成员变量，下面是所生成的代码

```java
  private static final class Builder implements AppComponent.Builder {
    private Application application;

    @Override
    public Builder application(Application application) {
      this.application = Preconditions.checkNotNull(application);
      return this;
    }

    @Override
    public AppComponent build() {
      Preconditions.checkBuilderRequirement(application, Application.class);
      return new DaggerAppComponent(application);
    }
  }
```
原来的`TestAModule.kt`中构造函数需要application作为提供变量或者 **@Provides** 所wrap的方法中也含有变量就会被这个Builder所支持原因很简单，因为
builder中的成员变量是可以被生成的`DaggerComponent`所接触到并保存起来。
这时候好奇的读者会说了，那我们只提供一个变量还行，那要是多个变量怎么办。还好，在builder中允许使用多个`@BindsInstance`

```java
public class GameMode {
    @Inject
    public GameMode(@GameModeName String gameName , @PlayerName String playerName){

    }
}

@Component
public interface GameComponent {
    GameMode getGameMode();

    @Component.Builder
    interface Builder{
        @BindsInstance Builder gameModeName(@GameModeName String str);
        @BindsInstance Builder playerName(@PlayerName String str);
        GameComponent build();
    }
}
```

- @BindsOptionalOf
 使用dagger2的时候，会有个问题，加入我们在需要被注入的类中请求注入对象A和对象B,如果注入的时候Component没有提供对象B,那么就会编译不通过

```java
@Module
public abstract class TestModule1 {
    @BindsOptionalOf
    abstract TestClass1 optionalTestClass1();
}

@Inject
Optional<TestClass1> mTestClass1;
```

2. Module
![Module](https://raw.githubusercontent.com/underwindfall/blogAssets/master/blog/kotlin/Module.png)

可惜的是*@Inject*并非万能

- 接口没有构造函数

- 第三方库的类不能被标注

- 构造函数中的参数必须配置

这几种情况就不能被使用，所以就有了 **@Provides @Module**的诞生，用来补充Module的不足。

3. SubComponent vs Component dependencies
两者从功能行上来说都是想简化*Component*的使用，从而达到不用复写已有的注入来分享新的注入功能。
```
Component dependencies - Use this when you want to keep two components independent.

SubComponents - Use this when you want to keep two components coupled.
```

和针筒抽取module很像，使用**dependencies**相当于使用针筒来抽取component。使用dependencies相当于把两个component进行组合，但是有一个指导性的原则：

**被dependencies的component需要对外告知它能够提供的内容**

4. Scope

- @Qualifier @Named

就是简单区分同样的Qualifier（限定符）的作用相当于起了个区分的别名

- Provides vs Lazy

Lazy:只有在调用 Lazy<T> 的 get() 方法时才会初始化依赖实例注入依赖
```java
public class Man {
    @Inject
    Lazy<Car> lazyCar;

    public void goWork() {
        lazyCar.get().go(); // lazyCar.get() 返回 Car 实例
    }
}
```
Provides和lazy差不多但是会根据Module提供的方法所进行注入Provider<T>，每次调用它的 get() 方法都会调用到 @Inject 构造函数创建新实例或者 Module 的 provide 方法返回实例。

- @Binds

@Provides和@Binds这两个不能共存于一个module,
最初介绍dagger的几个变种中，有一种情况是不提供provides方法，而是在构造函数上添加@Inject的方法。其实这种方法有利于书写，方便快捷的优势，但是当遇上如上这种情况，我们需要注入的是一个接口的时候，我们无法使用@Inject注解构造函数的方法来提供一个接口（我们可以把@Inject添加到实现类中，但是在注入时，dagger并不知道你想要注入的是哪个实现。）
所以@Binds就这样诞生了，可以想象的是，@Binds的作用其实就是告诉dagger，当请求注入一个接口时，我们使用某个实现类何其绑定，用实现类来进行注入。

- @IntoSet

Set注入对象 没啥好说的就是简单的注入一个HashSet，也可以同时注入多个**@ElementsIntoSet**

- @IntoMap

## Dagger Android

![Dagger分析](https://raw.githubusercontent.com/underwindfall/blogAssets/master/blog/dagger/Dagger.png)
![Dagger Android分析](https://raw.githubusercontent.com/underwindfall/blogAssets/master/blog/dagger/Dagger%20Android.png)

## DI Pattern
依赖注入，就是根据依赖关系解耦。本来的依赖关系被第三方Factory解除。
听上去不是白话，我们来举个例子

```kotlin
Class A{
    var b:B
}

Class B{}
```
转化成被第三方容器提供对象,这里dagger会通过寻找**@Inject**对象，然后发现起构造函数，如果也含有**@Inject**就会生成
相应Factory被注入
```kotlin
Class A{
    @Inject
    var b:B
}
```



