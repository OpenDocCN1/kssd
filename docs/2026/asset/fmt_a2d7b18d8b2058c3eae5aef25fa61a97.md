
# Android APP漏洞之战（4）——Content Provider漏洞详解

[TOC]

## 一、前言

今天总结Android APP四大组件中Content Provider挖掘的知识，主要分为两个部分，一部分是对Android Content Provider内容提供器的原理总结，另一部分便是对Android provider机制常见的一些漏洞总结，包括一些已知的漏洞方法，和一部分案例实践。

## 二、Content Provider初步介绍

### 1.Content Provider的基本原理

#### （1）Content Provider简介

![image-20210922094010216](upload/attach/202109/905443\_THZZM38BMRHHD7P.png)

```java
Android中的数据存储方式：Shared Preferences、网络存储、文件存储、外部存储、SQLite,这些存储方式一般在单独的应用程序中实现数据共享，对于不同应用之间共享数据，就要借助Content Provider。
ContentProvider为存储和读取数据提供了统一的接口，使用表的形式来对数据进行封装，使用ContentProvider可以在不同的应用程序之间共享数据，统一数据的访问方式，保证数据的安全性。
```

#### （2）Content Provider作用

![image-20210922094010216](upload/attach/202109/905443\_BFM67Z7VPAXZPGA.png)

![image-20210922094010216](upload/attach/202109/905443\_P4Y9X7NCMCPHJ76.png)

`Content Provider`可以使得不同APP进程之间进行数据交互和共享，即跨进程通信

#### （3）URI详解

我们创建一个`Content Provider`，其他的应用可以通过使用`ContentResolver`来访问`ContentProvider`提供的数据，而`ContentResolver`通过`uri`来定位自己要访问的数据，所以我们要先了解`URI`

##### URI

**URI的介绍：**

```java
（1）定义：Uniform Resource Identifier，即统一资源标识符
（2）作用：唯一标识ContentProvider &其中的数据
（3）外界进程通过URL找到对应的ContentProvider &其中数据，再进行数据操作
```

![image-20210922094010216](upload/attach/202109/905443\_92EXMEKYH5VR3J2.png)

```java
（1）标准前缀:content:// ,用来说明一个Content Provider控制这些数据
（2）URL的标识：com.carson.provider, 用于唯一标识这个ContentProvider，外部调用者可以根据这个标识来找到它。对于第三方程序，为了保证URL标识的一致性，必须是一个完整的、小写的类名，这个标识在元素的authorities属性中说明，一般是定义该ContentProvider的包.类的名称
（3）路径：User,要操作的数据库中表的名字，或者可以自己定义，记得在使用的时候保持一致
（4）记录ID:id, 如果URL中包含表示需要获取的记录ID,则返回该id对应的数据，如果没有ID,就表示返回全部
```

**构建URI的路径：**

```java
（1）操作User表中id为11的记录，构建数据：/User/11
（2）操作User表中id为11的记录的name字段：User/11/name
（3）操作User表中的所有记录：/User
（4）操作来自文件、xml或网络其他存储方式的数据，如要操作xml文件中User节点下的name字段：/User/name
（5）若要将一个字符串转换成URI,可以使用Uri类中的parse()方法：
Uri uri = Uri.parse("content://com.carson.provider/User")；
```

**URI各部分的获取：**

我们给出一个URI的样例：

```
http://www.baidu.com:8080/wenku/jiatiao.html?id=123456&name=jack
```

我们介意使用一些方法来获取URI的各个部分：

```java
getScheme()：获取 Uri 中的 scheme 字符串部分，在这里是 http
getHost()：获取 Authority 中的 Host 字符串，即 www.baidu.com
getPost()：获取 Authority 中的 Port 字符串，即 8080
getPath()：获取 Uri 中 path 部分，即 wenku/jiatiao.html
getQuery()：获取 Uri 中的 query 部分，即 id=15&name=jack
```

##### MIME

MIME是指定某个扩展名的文件用一种应用程序打开，就像用浏览器查看PDF格式的文件，浏览器会选择合适的应用打开。`ContentProvider` 会根据 URI 来返回 MIME 类型，`ContentProvider` 会返回一个包含两部分的字符串。MIME 类型一般包含两部分，如：

```java
text/html
text/css
text/xml
application/pdf
```

分为类型和子类型，Android 遵循类似的约定来定义MIME类型，每个内容类型的 Android MIME 类型有两种形式：多条记录（集合）和单条记录。

- 集合记录（dir）:

```
vnd.android.cursor.dir/自定义
```

- 单条记录（item）:

```
vnd.android.cursor.item/自定义
```

vnd 表示这些类型和子类型具有非标准的、供应商特定的形式。Android中类型已经固定好了，不能更改，只能区别是集合还是单条具体记录，子类型可以按照格式自己填写，在使用 Intent 时，会用到 MIME，根据 Mimetype 打开符合条件的活动。

##### URI解析

这里URI代表要操作的数据，我们在对数据进行获取时需要解析URI，Android提供了两个操作URI的工具类：`UriMatcher` 和 `ContentUris`

**UriMatcher：**

`UriMatcher`类用于匹配Uri，使用步骤如下：

- 将需要匹配的Uri路径进行注册：

```java
//常量UriMatcher.NO_MATCH表示不匹配任何路径的返回码
UriMatcher sMatcher = new UriMatcher(UriMatcher.NO_MATCH);
//如果match()方法匹配“content://com.wang.provider.myprovider/tablename”路径，返回匹配码为1
sMatcher.addURI("content://com.wang.provider.myprovider", " tablename ", 1);
//如果match()方法匹配content://com.wang.provider.myprovider/tablename/11路径，返回匹配码为2
sMatcher.addURI("com.wang.provider.myprovider", "tablename/#", 2);
```

此处采用 `addURI` 注册了两个需要用到的 URI；注意，添加第二个 URI 时，路径后面的 id 采用了通配符形式 “#”，表示只要前面三个部分都匹配上了就 OK

补充：

```java
*：表示匹配任意长度的任意字符
#：表示匹配任意长度的数字
匹配任意表的内容URI格式：
content：//com.example.app.provider/*
匹配table表中1任意一行数据的内容URI格式：
content：//com.example.app.procider/table/#
```

- 注册完需要匹配的 Uri 后，可以使用 `sMatcher.match(Uri)` 方法对输入的 Uri 进行匹配，如果匹配就返回对应的匹配码，匹配码为调用 `addURI()` 方法时传入的第三个参数

```java
switch (sMatcher.match(Uri.parse("content://com.zhang.provider.yourprovider/tablename/100"))) {
case 1:
//match 1, todo something
break;
case 2
//match 2, todo something
break;
default:
//match nothing, todo something
break;
}
```

**ContentUris：**

`ContentUris`类用于操作Uri路径后面的ID部分，有两个比较实用的方法：`withAppendedId(Uri uri, long id)`和`parseId(Uri uri)`

- `withAppendedId(Uri uri, long id)`用于为路径加上ID部分：

```java
Uri uri = Uri.parse("content://com.wang.provider.myprovider/tablename");
//生成的Uri为：content://com.wang.provider.myprovider/tablename/10
Uri resultUri = ContentUris.withAppendedId(uri, 10);
```

- `parseId(Uri uri)`则从路径中获取ID部分:

```java
Uri uri = Uri.parse("content://com.zhang.provider.myprovider/tablename/10")
//获取的结果为：7
long personid = ContentUris.parseId(uri);
```

#### （4）Content Provider数据共享

`ContentProvider`是一个抽象类，我们需要开发自己的内容提供者就需要继承这个类并复写其方法：

```java
ContentProvider 类主要方法的介绍：
public boolean onCreate()，在ContentProvider创建后就会被调用，而ContentProvider是在其它应用第一次访问它时被创建；
public Uri insert(Uri uri, ContentValues values)，供外部应用向ContentProvider添加数据；
public int delete(Uri uri, String selection, String[] selectionArgs)，供外部应用从ContentProvider删除数据；
public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs)，供外部应用更新ContentProvider中的数据；
public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder)，供外部应用从ContentProvider中获取数据；
public String getType(Uri uri)，返回当前Uri所代表数据的MIME类型；
```

如果操作的数据属于集合类型，那么 MIME 类型字符串应该以 `vnd.android.cursor.dir/` 开头：

```java
要得到所有 tablename 记录： Uri 为 content://com.wang.provider.myprovider/tablename，那么返回的MIME类型字符串应该为vnd.android.cursor.dir/table
```

如果要操作的数据属于非集合类型数据，那么 MIME 类型字符串应该以 `vnd.android.cursor.item/` 开头：

```java
要得到 id 为 10 的 tablename 记录，Uri 为 content://com.wang.provider.myprovider/tablename/10，那么返回的 MIME 类型字符串为：vnd.android.cursor.item/tablename
```

#### （5）Content Resolver操作数据

当外部应用需要对`ContentProvider`中的数据进行添加、删除、修改及查询操作时，可以使用`ContentResolver`类来完成，要获取`ContentResolver`对象，可以使用`Activity`提供`getContentResolver()`

`ContentResolver`类提供了与`ContentProvider`类相同签名的四个方法：

![image-20210922094010216](upload/attach/202109/905443\_FGR9PDYBR99BRGU.png)

```java
public Uri insert(Uri uri, ContentValues values)，往ContentProvider添加数据；
public int delete(Uri uri, String selection, String[] selectionArgs)，从ContentProvider删除数据；
public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs)，更新ContentProvider中的数据；
public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder)，从ContentProvider中获取数据；
```

这些方法的第一个参数为`Uri`，代表要操作的`ContentProvider`和对其中的什么数据进行操作，其实和`ContentProvider`里面的方法是一样的，最终会被传到我们之前程序里面定义的`ContentProvider`方法。

```java
假定给定的是：Uri.parse("content://com.wang.provider.myprovider/tablename/10")，
那么将会对主机名为com.wang.provider.myprovider的ContentProvider进行操作，操作的数据为tablename表中id为10的记录
```

使用`ContentResolver`对`ContentProvider`中的数据进行操作：

```java
ContentResolver resolver = getContentResolver();
Uri uri = Uri.parse("content://com.wang.provider.myprovider/tablename");
//添加一条记录
ContentValues values = new ContentValues();
values.put("name", "wang1");
values.put("age", 28);
resolver.insert(uri, values);
//获取tablename表中所有记录
Cursor cursor = resolver.query(uri, null, null, null, "tablename data");
while(cursor.moveToNext()){
Log.i("ContentTest", "tablename_id="+ cursor.getInt(0)+ ", name="+ cursor.getString(1));
}
//把id为1的记录的name字段值更改新为zhang1
ContentValues updateValues = new ContentValues();
updateValues.put("name", "zhang1");
Uri updateIdUri = ContentUris.withAppendedId(uri, 2);
resolver.update(updateIdUri, updateValues, null, null);
//删除id为2的记录，即字段age
Uri deleteIdUri = ContentUris.withAppendedId(uri, 2);
resolver.delete(deleteIdUri, null, null);
```

**监听数据变化：**

如果`ContentProvider`的访问者需要知道数据发生的变化，可以在`ContentProvider`发生数据变化时调用`getContentResolver().notifyChange(uri, null)`来通知注册在此URI上的访问者。只给出类中监听部分的代码：

```java
public class MyProvider extends ContentProvider {
public Uri insert(Uri uri, ContentValues values) {
db.insert("tablename", "tablenameid", values);
getContext().getContentResolver().notifyChange(uri, null);
}
}
```

而访问者必须使用`ContentObserver`对数据（数据采用uri描述）进行监听，当监听到数据变化通知时，系统就会调用`ContentObserver`的`onChange()`方法：
