---
layout:     post
title:      "android-ormLite总结"
subtitle:   " \"基础用法\""
date:       2018-07-18 12:00:00
author:     "Undervoid"
header-img: "img/post-bg-android.jpg"
catalog: true
tags:
    - 磨砻淬砺集
    - Android开发
    - OrmLite
---

# OrmLite 初探

>最近工作之余接手了个老项目用到了这个平时不太注意的OrmLite框架，自己总结分享下。

## 概念

「OrmLite」它的英文全称是Object Relational Mapping，意思是对象关系映射；如果接触过Java EE开发的，一定知道Java Web开发就有一个类似的数据库映射框架——Hibernate。简单来说，就是我们定义一个实体类，利用这个框架，它可以帮我们吧这个实体映射到我们的数据库中，在Android中是SQLite，数据中的字段就是我们定义实体的成员变量。是一种数据结构形式，是一系列的状态组合。Redux的核心思想就是要提供*可预测*的状态管理，这对日益膨胀的大型应用来说尤其重要。[Android OrmLite](http://ormlite.com/)有着不解之缘，本身其封装完备，的确是Lite的轻量级，文档也较为轻量，可是在 `2018` 的今天，说实话有着足够多的框架可以帮助android去解决数据持久化的问题 (Google Room, Realm, ObjectBox etc.).

## Android OrmLite 基本知识

Android OrmLite 数据库特别的不多我们可以从最简单的例子开始：

> 创建表格

比如我们有个最简单的类:

```java
@DatabaseTable(tableName = "accounts")
public class Account {
    @DatabaseField(id = true)
    private String name;
    @DatabaseField
    private String password;
    public Account() {
        // ORMLite needs a no-arg constructor
    }
    public Account(String name, String password) {
        this.name = name;
        this.password = password;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public String getPassword() {
        return password;
    }
    public void setPassword(String password) {
        this.password = password;
    }
}
```
只需要在 「**POJO**」 类中加入 `@DatabaseTable` 和 ` @DatabaseField` 分别用来确认其数据库中数据表的名字和每个存入数据的名字即可， 当然其数据表头的名字可以自行定制改名，比如 `DatabaseField` ：表示定义了数据中的一个字段，id表示数据中的一个主键，如果指定为generatedId，表示自动增长id，我们不需要给它赋值。其他字段，可以使用columnName来指定字段名，canBeNull表示是否为空，这些赋值可以按照以下来指定
- (columnName = "name")
- (id = true, canBeNull = false）

[官方文档](http://ormlite.com/docs/ormlite.pdf) 还介绍了一系列的 `Annotations` 非常详尽，意外的觉得非常全面。
我在这里翻译了一部分内容，可供大家参考
|名称|数据类型|默认值|说明|
|:----:|:---: |:----: |:------------: |
|columnName|String||字段名，缺省为变量名|
|dataType|String||设置DataType即字段类型，通常不用指定，对应到SQL type|
|defaultValue|String|空|默认值，默认为空|
|width|Integer||通常用在字符串域。默认是0，意思是默认长度。字符串默认255|
|canBeNull|Boolean|true|是否可空|
|id|Boolean|false|指定该域是否为id，一个类只能有一个域设置此|
|generatedId|Boolean|false|该域是否为自动生成id域，一个类一个类只能有一个域设置此|
|generatedIdSequence|String||指定序列。从1开始，每次自增1|
|foreign|Boolean|false|指定该域是否对应另一个类，另一个类必须有id域|
|useGetSet|Boolean|false|是否需要通过get和set方法访问，通过java反射访问|
|unknownEnumName|String||如果该域为java枚举类型，可以指定一个枚举值，当数据库行的值没有在枚举类型中时，则使用此值|
|throwIfNull|Boolean|false|当字段为空进行插入时，ORMLite会抛出异常|
|persisted|Boolean|true|设置为false时，不存入数据库|
|format||||
|unique|Boolean|false|列唯一性|
|uniqueCombo|Boolean|false|联合列唯一性|
|index|Boolean|false|建立索引，索引名默认为列名加后缀_idx|
|uniqueIndex|Boolean|false|建立唯一索引，在此索引中的所有值都是唯一的|
|indexName|String|none|指定索引名，则不需要另外指定index布尔值。为了在一个索引中索引多个域，每个域需要有相同的indexName|
|uniqueIndexName|String|none|指定唯一索引名称，在此索引中的所有值都是唯一的|
|foreignAutoRefresh|Boolean|false|外键自动刷新|
|maxForeignAutoRefreshLevel||||
|allowGeneratedIdInsert|Boolean|false|如果设置为true，当插入包含id域的对象时，不会自动生成id域覆盖|
|columnDefinition||||
|foreignAutoCreate||||
|version|Boolean|false||
|foreignColumnName|String||指定外键对象关联域|

> 增删改查

首先在进行数据库基本操作之前，我们需要做一件事情就是获取想去的DAO类。你可以选择想上文那样有个Bean的class自动创建，也可以选择通过 `TableUtils.createTable` 创建相应的数据表，但我这里就有疑问了，如果我想创造JAVA 最基本的`DAO`类怎么办呢？
相应的它也提供了 `DaoManager`来处理这样的情况

```java
Dao<Account, String> accountDao =
DaoManager.createDao(connectionSource, Account.class);
Dao<Order, Integer> orderDao =
DaoManager.createDao(connectionSource, Order.class);
```
如果在Table创建好了以后想要获取它，更简单只需要调用 `OrmLiteSqliteOpenHelper`其中 **getDao** 这个方法即可。
我们在获取相应的`DAO`后，就可以进行 `增删改查`的操作了。

---
> Talk is cheap so let's show some code.

```java
public class ThemeDao {
    
    private Dao<Theme, Integer> themeDao;
    
    /**
     * 构造方法
     * 获得数据库帮助类实例，通过传入Class对象得到相应的Dao
     * @param context
     */
    public ThemeDao(Context context) {
        try {
            themeDao = dbHelper.getDao(Theme.class);
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    
    /**
     * 添加一条记录
     * @param theme
     */
    public void add(Theme theme) {
        try {
            themeDao.create(theme);
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    
    /**
     * 删除一条记录
     * @param theme
     */
    public void delete(Theme theme) {
        try {
            themeDao.delete(theme);
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    
    
    /**
     * 更新一条记录
     * @param theme
     */
    public void update(Theme theme) {
        try {
            themeDao.update(theme);
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    
    /**
     * 查询一条记录
     * @param id
     * @return
     */
    public Theme queryForId(int id) {
        Theme theme = null;
        try {
            theme = themeDao.queryForId(id);
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return theme;
    }
    
    
    /**
     * 查询所有记录
     * @return
     */
    public List<Theme> queryForAll() {
        List<Theme> themes = new ArrayList<Theme>();
        try {
            themes = themeDao.queryForAll();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return themes;
    }

}
```

>数据库版本升级

我通过文档了解到使用Ormlite在android的环境下的话需要创建一个`DatabaseHelper`去继承`OrmLiteSqliteOpenHelper`,其中`OrmLiteSqliteOpenHelper`这个类中有着很多使用的方法比如:
-   onCreate :  创建相应的数据表数据库
-   onUpgrade : 升级数据库
-   ...

下面我能公布个我觉得靠谱的`DatabaseHelper`写法，此写法来自[鸿洋大神](https://blog.csdn.net/lmj623565791/article/details/39122981)

```java
public  class DatabaseHelper extends OrmLiteSqliteOpenHelper
{
	private static final String TABLE_NAME = "sqlite-test.db";
 
	private Map<String, Dao> daos = new HashMap<String, Dao>();
 
	private DatabaseHelper(Context context)
	{
		super(context, TABLE_NAME, null, 4);
	}
 
	@Override
	public void onCreate(SQLiteDatabase database,
			ConnectionSource connectionSource)
	{
		try
		{
			TableUtils.createTable(connectionSource, User.class);
			TableUtils.createTable(connectionSource, Article.class);
			TableUtils.createTable(connectionSource, Student.class);
		} catch (SQLException e)
		{
			e.printStackTrace();
		}
	}
 
	@Override
	public void onUpgrade(SQLiteDatabase database,
			ConnectionSource connectionSource, int oldVersion, int newVersion)
	{
		try
		{
			TableUtils.dropTable(connectionSource, User.class, true);
			TableUtils.dropTable(connectionSource, Article.class, true);
			TableUtils.dropTable(connectionSource, Student.class, true);
			onCreate(database, connectionSource);
		} catch (SQLException e)
		{
			e.printStackTrace();
		}
	}
 
	private static DatabaseHelper instance;
 
	/**
	 * 单例获取该Helper
	 * 
	 * @param context
	 * @return
	 */
	public static synchronized DatabaseHelper getHelper(Context context)
	{
		context = context.getApplicationContext();
		if (instance == null)
		{
			synchronized (DatabaseHelper.class)
			{
				if (instance == null)
					instance = new DatabaseHelper(context);
			}
		}
 
		return instance;
	}
 
	public synchronized Dao getDao(Class clazz) throws SQLException
	{
		Dao dao = null;
		String className = clazz.getSimpleName();
 
		if (daos.containsKey(className))
		{
			dao = daos.get(className);
		}
		if (dao == null)
		{
			dao = super.getDao(clazz);
			daos.put(className, dao);
		}
		return dao;
	}
 
	/**
	 * 释放资源
	 */
	@Override
	public void close()
	{
		super.close();
 
		for (String key : daos.keySet())
		{
			Dao dao = daos.get(key);
			dao = null;
		}
	}
 
}
```


## 个性化Statement Builder

> **基本Query builder类型**

- QueryBuilder
```java
// get our query builder from the DAO
QueryBuilder<Account, String> queryBuilder =
accountDao.queryBuilder();
// the ’password’ field must be equal to "qwerty"
queryBuilder.where().eq(Account.PASSWORD_FIELD_NAME, "qwerty");
// prepare our sql statement
PreparedQuery<Account> preparedQuery = queryBuilder.prepare();
// query for all accounts that have "qwerty" as a password
List<Account> accountList = accountDao.query(preparedQuery);
```
- DeleteBuilder
```java
DeleteBuilder<Account, String> deleteBuilder =
accountDao.deleteBuilder();
// only delete the rows where password is null
deleteBuilder.where().isNull(Account.PASSWORD_FIELD_NAME);
deleteBuilder.delete();
```
- UpdateBuilder

```java
UpdateBuilder<Account, String> updateBuilder =
accountDao.updateBuilder();
// update the password to be "none"
updateBuilder.updateColumnValue("password", "none");
// only update the rows where password is null
updateBuilder.where().isNull(Account.PASSWORD_FIELD_NAME);
updateBuilder.update();
```
 

## Ormlite的外键约束（一对一、一对多、多对多关系）
使用这个框架需要比较注意的一点就是外键约束，这里笔者只讨论一对一、一对多的情况。
上一节我们定义了PackageInfo这个实体，里面有这样的定义

```java
// 一个套餐可以对应多个主题
    @ForeignCollectionField(eager = true) // 必须
    public ForeignCollection<Theme> themes;
    
    
    // 外部对象，一个套餐只对应一个摄影师，一个摄影师可以对应多个套餐
    @DatabaseField(foreign = true)
    public Photographer photographer;
```

这里就用到了外键的约束，我们来分析一下：
一个套餐对应多个主题：1:n的关系
一个套餐对应一个摄影师：1:1的关系

在n的一方，我们可以使用`@ForeignCollectionField`这样的注解，`eager = true`表示可以进行懒加载。

如果是一对一，我们还是用`@DatabaseField`注解，但要指定`(foreign = true)`表示是一个外键。
现在我们看一下多的一方是怎么定义的：

```java
// 外部对象字段
    @DatabaseField(foreign = true, foreignAutoRefresh = true)
    public PackageInfo mPackage;
```

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

自己尝试用 [Kotlin](!https://kotlinlang.org/) 和 [redux-Kotlin](!https://github.com/pardom/redux-kotlin)搭建了一个简单的关于Redux在Android的应用，它能够模拟面试我也加入了 Realm 开源库 。算是给大家一个小尝试吧。

[Demo地址](!https://github.com/underwindfall/kotlin_redux_interview)

* Demo 截图：

  ![](https://s3.eu-central-1.amazonaws.com/undervoidfall/blog/demo2.png)

  ​



## 参考

[How Redux works]: http://arqex.com/1087/using-redux-devtools-without-redux	"How Redux works"
[Redux Middleware: Behind the Scene](http://briantroncone.com/?p=529)
[Understanding Redux Middleware](https://medium.com/@meagle/understanding-87566abcfb7a#.r7x2ierw5)

