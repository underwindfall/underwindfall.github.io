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


## 区别总结

- HashSet底层采用的是HashMap进行实现的，但是没有key-value，只有HashMap的key set的视图，HashSet不容许重复的对象
- Hashtable是基于Dictionary类的，而HashMap是基于Map接口的一个实现
- Hashtable里默认的方法是同步的，而HashMap则是非同步的，因此Hashtable是多线程安全的
- HashMap可以将空值作为一个表的条目的key或者value,HashMap中由于键不能重复，因此只有一条记录的Key可以是空值，而value可以有多个为空，但HashTable不允许null值(键与值均不行)
- 内存初始大小不同，HashTable初始大小是11，而HashMap初始大小是16
- 内存扩容时采取的方式也不同，Hashtable采用的是2*old+1,而HashMap是2*old。
- 哈希值的计算方法不同，Hashtable直接使用的是对象的hashCode,而HashMap则是在对象的hashCode的基础上还进行了一些变化

## 论证

> 区别1
```java
//HashSet类的部份源代码  
public class HashSet<E>  
    extends AbstractSet<E>  
    implements Set<E>, Cloneable, java.io.Serializable  
{   //用于类的序列化，可以不用管它  
    static final long serialVersionUID = -5024744406713321676L;  
    //从这里可以看出HashSet类里面真的是采用HashMap来实现的  
    private transient HashMap<E,Object> map;  
  
    // Dummy value to associate with an Object in the backing Map  
    //这里是生成一个对象，生成这个对象的作用是将每一个键的值均关联于此对象，以满足HashMap的键值对  
    private static final Object PRESENT = new Object();  
  
    /** 
     * Constructs a new, empty set; the backing <tt>HashMap</tt> instance has 
     * default initial capacity (16) and load factor (0.75). 
     */  
    //这里是一个构造函数，开构生成一个HashMap对象，用来存放数据  
    public HashSet() {  
    map = new HashMap<E,Object>();  
    }  
```
从上面的代码中得出的结论是HashSet的确是采用HashMap来实现的，而且每一个键都关键同一个Object类的对象，因此键所关联的值没有意义，真正有意义的是键。而HashMap里的键是不允许重复的，因此1也就很容易明白了。

> 区别2

```java
//从这里可以看得出Hashtable是继承于Dictionary,实现了Map接口  
public class Hashtable<K,V>  
    extends Dictionary<K,V>  
    implements Map<K,V>, Cloneable, java.io.Serializable {  
```

```java
//这里可以看出的是HashMap是继承于AbstractMap类，实现了Map接口  
//因此与Hashtable继承的父类不同  
public class HashMap<K,V>  
    extends AbstractMap<K,V>  
    implements Map<K,V>, Cloneable, Serializable  
```

> 区别3,4

```java
//Hashtable里的向集体增加键值对的方法，从这里可以明显看到的是  
//采用了synchronized关键字，这个关键字的作用就是用于线程的同步操作  
//因此下面这个方法对于多线程来说是安全的，但这会影响效率     
public synchronized V put(K key, V value) {  
    // Make sure the value is not null  
    //如果值为空的，则会抛出异常  
    if (value == null) {  
        throw new NullPointerException();  
    }  
  
    // Makes sure the key is not already in the hashtable.  
    Entry tab[] = table;  
    //获得键值的hashCode,从这里也可以看得出key!=null,否则的话会抛出异常的呦  
    int hash = key.hashCode();  
    //获取键据所在的哈希表的位置  
    int index = (hash & 0x7FFFFFFF) % tab.length;  
    //从下面这个循环中可以看出的是，内部实现采用了链表，即桶状结构  
    for (Entry<K,V> e = tab[index] ; e != null ; e = e.next) {  
        //如果向Hashtable中增加同一个元素时，则会重新更新元素的值   
        if ((e.hash == hash) && e.key.equals(key)) {  
                V old = e.value;  
                e.value = value;  
                return old;  
        }  
    }  
    //后面的暂时不用管它，大概的意思就是内存的个数少于某个阀值时，进行重新分配内存  
    modCount++;  
    if (count >= threshold) {  
        // Rehash the table if the threshold is exceeded  
        rehash();  
  
            tab = table;  
            index = (hash & 0x7FFFFFFF) % tab.length;  
    }  
```
```java
//HashMap中的实现则相对来说要简单的很多了，如下代码  
//这里的代码中没有synchronize关键字，即可以看出，这个关键函数不是线程安全的  
    public V put(K key, V value) {  
    //对于键是空时，将向Map中放值一个null-value构成的键值对  
    //对值却没有进行判空处理，意味着可以有多个具有键，键所对应的值却为空的元素。  
        if (key == null)  
            return putForNullKey(value);  
    //算出键所在的哈希表的位置  
        int hash = hash(key.hashCode());  
        int i = indexFor(hash, table.length);  
    //同样从这里可以看得出来的是采用的是链表结构，采用的是桶状  
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {  
            Object k;  
            //对于向集体中增加具有相同键的情况时，这里可以看出，并不增加进去，而是进行更新操作  
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {  
                V oldValue = e.value;  
                e.value = value;  
                e.recordAccess(this);  
                return oldValue;  
            }  
        }  
        //开始增加元素  
        modCount++;  
        addEntry(hash, key, value, i);  
        return null;  
    }  
```

> 区别5

```java
public Hashtable() {  
   //从这里可以看出，默认的初始化大小11，这里的11并不是11个字节，而是11个Entry,这个Entry是  
   //实现链表的关键结构  
   //这里的0.75代表的是装载因子  
this(11, 0.75f);  
 }  

 //这里均是一些定义  
 public HashMap() {  
 //这个默认的装载因子也是0.75  
     this.loadFactor = DEFAULT_LOAD_FACTOR;  
 //默认的痤为0.75*16  
     threshold = (int)(DEFAULT_INITIAL_CAPACITY * DEFAULT_LOAD_FACTOR);  
 //这里开始是默认的初始化大小，这里大小是16  
     table = new Entry[DEFAULT_INITIAL_CAPACITY];  
     init();  
 }  
```
可以看出的是两者的默认大小是不同的，一个是11，一个是16

> 区别6

```java
//Hashtable中调整内存的函数，这个函数没有synchronize关键字，但是protected呦  
protected void rehash() {  
    //获取原来的表大小  
    int oldCapacity = table.length;  
    Entry[] oldMap = table;  
  //设置新的大小为2*oldCapacity+1  
    int newCapacity = oldCapacity * 2 + 1;  
    //开设空间  
    Entry[] newMap = new Entry[newCapacity];  
  //以下就不用管了。。。  
    modCount++;  
    threshold = (int)(newCapacity * loadFactor);  
    table = newMap;  
  
    for (int i = oldCapacity ; i-- > 0 ;) {  
        for (Entry<K,V> old = oldMap[i] ; old != null ; ) {  
        Entry<K,V> e = old;  
        old = old.next;  
  
        int index = (e.hash & 0x7FFFFFFF) % newCapacity;  
        e.next = newMap[index];  
        newMap[index] = e;  
        }  
    }  
    }

//HashMap中要简单的多了，看看就知道了  
void addEntry(int hash, K key, V value, int bucketIndex) {  
Entry<K,V> e = table[bucketIndex];  
       table[bucketIndex] = new Entry<K,V>(hash, key, value, e);  
       //如果超过了阀值  
       if (size++ >= threshold)  
       //就将大小设置为原来的2倍  
           resize(2 * table.length);  
   }  
```

> 区别7

```java
//Hashtable中可以看出的是直接采用关键字的hashcode作为哈希值  
int hash = key.hashCode();  
//然后进行模运算，求出所在哗然表的位置   
int index = (hash & 0x7FFFFFFF) % tab.length;  


//HashMap中的实现  
//这两行代码的意思是先计算hashcode,然后再求其在哈希表的相应位置        
int hash = hash(key.hashCode());  
int i = indexFor(hash, table.length);  
```