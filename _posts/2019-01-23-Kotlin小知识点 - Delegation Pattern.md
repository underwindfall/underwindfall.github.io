---
layout:     post
title:      "Kotlin小知识点 - Delegation Pattern"
subtitle:   ""
date:       2019-01-23 12:00:00
author:     "Undervoid"
header-img: "img/kotlin.png"
catalog: true
tags:
    - 磨砻淬砺集
    - Kotlin
---


# Kotlin小知识集合

> 总结一些 `Kotlin` 方面的调用，会不断的持续更新这个文章

## 设计模式

### 单例模式 
(https://juejin.im/post/5acf49a06fb9a028d700ff4d)
(https://medium.com/@programmerr47/singletons-in-android-63ddf972a7e7)
(https://medium.com/@BladeCoder/kotlin-singletons-with-argument-194ef06edd9e)

> 单例模式实现几种方式

1.饿汉式实现



### 代理模式

简介：在委托模式中，有两个对象参与处理同一个请求，接受请求的对象将请求委托给另一个对象来处理。委托模式是一项基本技巧. 注意这里的委托模式其实是一种Composing

```kotlin
interface I {
     void f();
     void g();
 }
 
 class A implements I {
     public void f() { System.out.println("A: doing f()"); }
     public void g() { System.out.println("A: doing g()"); }
 }
 
 class B implements I {
     public void f() { System.out.println("B: doing f()"); }
     public void g() { System.out.println("B: doing g()"); }
 }
 
 class C implements I {
     // delegation
     I i = new A();
 
     public void f() { i.f(); }
     public void g() { i.g(); }
 
     // normal attributes
     public void toA() { i = new A(); }
     public void toB() { i = new B(); }
 }
 
 
 public class Main {
     public static void main(String[] args) {
         C c = new C();
         c.f();     // output: A: doing f()
         c.g();     // output: A: doing g()
         c.toB();
         c.f();     // output: B: doing f()
         c.g();     // output: B: doing g()
     }
 }
```

kotlin 的委托模式

#### 类代理

```kotlin
interface Base {
    val message: String
    fun print()
}

class BaseImpl(val x: Int) : Base {
    override val message = "BaseImpl: x = $x"
    override fun print() { println(message) }
}

class Derived(b: Base) : Base by b {
    // This property is not accessed from b's implementation of `print`
    override val message = "Message of Derived"
}

fun main() {
    val b = BaseImpl(10)
    val derived = Derived(b)
    derived.print()
    println(derived.message)
}
```

输出结果，可以看出他其实是两种class的结合

```
BaseImpl: x = 10
Message of Derived
```

#### 属性代理
 先来谈谈kotlin自带的属性代理吧

 - Lazy
 
```kotlin
     val lazyData: Double by lazy {
    println("Initializing lazyData")
    2.0
}
fun main(args: Array<String>) {
    println(lazyData)
    println(lazyData)
}
```
```
结果是 
Initializing lazyData
2.0
2.0
```
这里其实牵扯到一个线程问题，该值只在一个线程中计算，并且所有线程会看到相同的值。如果初始化委托的同步锁不是必需的，这样多个线程可以同时执行，那么将 LazyThreadSafetyMode.PUBLICATION 作为参数传递给 lazy() 函数。 而如果你确定初始化将总是发生在单个线程，那么你可以使用 LazyThreadSafetyMode.NONE 模式， 它不会有任何线程安全的保证和相关的开销。


 - Observable

 ```kotlin
 var observableData: String by Delegates.observable("Initial value") {
    property, oldValue, newValue ->
    println("${property.name}: $oldValue -> $newValue")
}

fun main(args: Array<String>) {
    observableData = "New value"
    observableData = "Another value"
}
 ```
 ```
结果是 
observableData: Initial value -> New value
observableData: New value -> Another value
```
可以想到一个在android程序中使用的实例,就是通知一个`adapter`是否变化数据来更新`UI``

example:

``` kotlin
var items: List by Delegates.observable(emptyList()) {
    _, _, _ -> notifyDataSetChanged()
}
```

 - Vetoable
 和 `Observable`很相近但是会选择一个condition来判断新改变的`value` 是否可以改变

 ```kotlin
var vetoableData: Int by Delegates.vetoable(0) {
    property, oldValue, newValue ->
    println("${property.name}: $oldValue -> $newValue")
    newValue >= 0
}

fun main(args: Array<String>) {
    vetoableData = -1
    println("vetoableData = $vetoableData")
    vetoableData = 1
    println("vetoableData = $vetoableData")
}
```
结果是

```
vetoableData: 0 -> -1
vetoableData = 0
vetoableData: 0 -> 1
vetoableData = 1
```


 - Storing Map

```kotlin
//这里也可以被MutableMap 代替 var 变量
class User(val map: Map<String, Any?>) {
    val name: String by map
    val age: Int     by map
}

val user = User(mapOf(
    "name" to "John Doe",
    "age"  to 25
))

```
```
结果是
println(user.name) // Prints "John Doe"
println(user.age)  // Prints 25
```

 自定义 Delegation 类
 
 其实最常见的就是 `bindView`
 
 ```kotlin

class MainActivity : AppCompatActivity() ,MainContract.View {

    val imageView : ImageView by bindView(R.id.imageView)

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        imageView.setImageDrawable(null)
    }
}

fun <T: View> bindView( id : Int): FindView<T> = FindView(id)

class FindView<T : View >(val id:Int) : ReadOnlyProperty<Any?,T>{

    override fun getValue(thisRef: Any?, property: KProperty<*>): T {

        if(this.value == null) {
            this.value = (thisRef as Activity).findViewById<T>(id)
        }
        return this.value?:throw RuntimeException("can not find this view")
    }

    var value : T? = null
}
```