---
layout:     post
title:      "ReactNative 工作总结"
subtitle:   " \"基础知识\""
date:       2018-07-28 12:00:00
author:     "Undervoid"
header-img: "img/post-bg-react-native.png"
catalog: true
tags:
    - 磨砻淬砺集
    - React Native开发
---

# React Native 前言

最近工作开始接触React Native（以下简称RN），这应该是最近近两年来最火的框架之一。它的推出也伴随着一个名词“全栈式前端”，这是个什么概念呢？大概意思就是一个人可以同时写Android,iOS,Web程序。虽然个人本身仍然是个Android程序员，但是接触一点RN我也相信对自己的职业生涯会有好处，目前我使用的仍然是Javascript进行RN的撰写，虽然本人我对 `javascript` 这门语言真的是没什么好感，感觉写代码就像光着身子在路上走路一样，当然也可以尝试使用 `Typescript` 去替代可毕竟自己不是RN的很感冒所以目前也只是尝试而已分享下自己的见解，如有不足敬请谅解。


## React Native 基本知识

关于React Native网上有一系列的文章和教程，我就不一一阐述了，我主要在以下几点来解析下RN的特性：

### 高阶组件 　HOC(Higher Order Components)
> 直接返回一个stateless component，如：
> 在新组件的render函数中返回一个新的class component，
> 继承（extends）原组件后返回一个新的class component


### Function as Child Components

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

### Get,Set方法

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

### Redux-Saga


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








