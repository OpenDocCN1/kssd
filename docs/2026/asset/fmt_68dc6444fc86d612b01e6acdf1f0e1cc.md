
# [原创]Java序列化反序列化源码---Jackson反序列化漏洞源码分析

## 源码分析Jackson反序列化漏洞

### 前言

本次分析从Java序列化和反序列化源码开始分析，进一步分析Jackson源码，找出造成漏洞的原因，最后以Jackson2.9.2版本，JDK1.80_171，resin4.0.52，CVE-2020-10673为例复现漏洞。

## 一．JAVA反序列化原理

### 1.1 Class对象

每一个类都有一个Class对象，Class对象包含每一个类的运行时信息，每一个类都有一个Class对象，每编译一个类就产生一个Class对象，Class类没有公共的构造方法，Class对象是在类加载的时候由JVM以及通过调用类加载器中的`DefineClass()`方法自动构造的，因此不能显式地声明一个Class对象。在类加载阶段，类加载器首先检查这个类的Class对象是否已经被加载。如果尚未加载，默认的类加载器就会根据类的全限定名查找.class文件。一旦某个类的Class对象被载入内存，我们就可以它来创建这个类的所有对象以及获得这个类的运行时信息。

获得Class对象的方法：

1). `Class.forName("类的全名")`;//com.xx.xx.xx
2). 实例对象.`getClass()`
3). 类名.class

### 1.2 反射

JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

实现Java反射机制的类都位于`java.lang.reflect`包中：

1). `Class`类：代表一个类
2). `Field`类：代表类的成员变量（类的属性）
3). `Method`类：代表类的方法
4). `Constructor`类：代表类的构造方法
5). `Array`类：提供了动态创建数组，以及访问数组的元素的静态方法

简单反射调用代码
```
Class clz=this.getClass();
Object obj= clz.getMethod("方法名",Class对象序列).invoke(this,Object参数序列);
```

### 1.3 反序列化

Java 序列化是指把 Java
对象转换为字节序列的过程便于保存在内存、文件、数据库中，`ObjectOutputStream`类的
`writeObject()` 方法可以实现序列化。

Java 反序列化是指把字节序列恢复为 Java 对象的过程，`ObjectInputStream` 类的
`readObject()` 方法用于反序列化。

RMI：是 Java 的一组拥护开发分布式应用程序的
API，实现了不同操作系统之间程序的方法调用。值得注意的是，RMI 的传输 100%
基于反序列化，Java RMI 的默认端口是 1099 端口。

JMX：JMX 是一套标准的代理和服务，用户可以在任何 Java
应用程序中使用这些代理和服务实现管理,中间件软件 WebLogic 的管理页面就是基于 JMX
开发的，而 JBoss 则整个系统都基于 JMX 构架。

只有实现了`Serializable`接口的类的对象才可以被序列化，`Serializable`
接口是启用其序列化功能的接口，实现 `java.io.Serializable`
接口的类才是可序列化的，没有实现此接口的类将不能使它们的任一状态被序列化或逆序列化。

`readObject()`
方法的作用正是从一个源输入流中读取字节序列，再把它们反序列化为一个对象，并将其返回，`readObject()`
是可以重写的，可以定制反序列化的一些行为。

`readObject()`主要做的事情，其实就是读取正常应该被序列化的字段信息后，再构造出一个map，再通过对象流，将原有通过对象流写进文件里面的map信息（容量，每个item信息等）全部读取出来，然后重新构造一个map，这样就使得我们保存在set里面的信息，在经历过对象流的序列化和反序列化后，都没有丢失。

### 1.4 readObject()分析

1). 进入`readObject()`源码，首先判断是否调用`readObjectOverride()`,其中`enableOverride`的值由`ObjectInputStream`类的构造函数决定，带参构造为false,无参构造为true。但`readObjectOverride()`函数为空方法。因此一般需要带参构造。

2). 调用`readObject0()`,进入函数中，发现有一大部分的switch判断，该部分判断读入的流是什么类型。其中`readOrdinaryObject()`值得注意，因为读入的大部分都是要反序列化的对象数据流。

### 1.5 readOrdinaryObject()函数中readClassDesc()分析

1. 进入`readOrdinaryObject()`函数，发现一开始，调用`readClassDesc()`，进而调用`checkDeserialize()`。

2. `readClassDesc()`函数中，switch判断顶端字节特征，选择执行的函数

3. 无参构造创建`ObjectStreamClass()`对象，调用`initNonProxy()`.

4. 过程中调用了`lookup(Class,boolean)`方法

5. 进入lookup方法，生成一个带参构造的`ObjectStreamClass`对象，这个对象就是下面要提到的`readOrdinaryObject()`函数中的`readSerialData()`的第一个参数。

### 1.6 readOrdinaryObject()函数中readSerialData()分析

1. 进入`readSerialData()`，注意`obj`参数

2. 进入`invokeReadObject()`，发现`obj`参数(该参数上文提到)直接经过反射调用了里面的方法

## 二．Jackson序列化与反序列化关键函数

### 1.1 readValue(Object obj)序列化函数分析

1. 单步调试`ObjectMapper`初始化过程，发现在`com.fasterxml.jackson.databind.deser.BeanDeserializerFactory`类下，存在一个`HashSet<>`集合,这里是定义了一个黑名单`DEFAULT_NO_DESER_CLASS_NAMES`（默认不反序列化的类）

并且将该黑名单设置为`unmodifiableSet`（不可修改，若修改则抛出异常）。

2. 继续跟进，回到主程序，进入`enableDefaultTyping()`，这是一个Jackson在序列化和反序列化时配置的项目，当`ObjectMapper`对象调用该函数时，那么
`OBJECT_AND_NON_CONCRETE`就会生效，导致需要json字符串包含类名来反序列化。且接收封装的数组形式：**["...",{"...":"..."}]**

3. `writeValueAsString(Object value)`

跟踪`value`参数，在`_configAndWriteValue()`函数中会判断该`value`对象是否是可关闭资源，如果是，则进入`_configAndWriteCloseable()`，对`value`强制转换，防止意外异常发生。如果不是，则直接进入`serializeValue()`

4. 进入`serializeValue()`，通过反射获取`value`的`Class`对象

5. 跟踪`cls`参数流向，进入`typedValueSerializer()`

到这里就已经很明显了，获取对象的hash值，通过该hash值从`HashMap`内获取`JsonSerializer<Object>`对象，并返回，且此处加了`synchronized`（重量级锁，与本文无关，不做赘述），进行线程同步。因为此时`HashMap`里面为空，因此回到`findTypedValueSerializer()`继续判断`ser`。这里也就是Jackson通过维护一个`HashMap`用对象hash值来获取序列化对象。该序列化对象是Json串内指定的类，该序列化对象可以遍历原类的所有方法。

7. 继续跟进，进入`_addFields()`, `implName`是代表类属性名，从`findImplicitPropertyName()`内获取

该函数接收的参数为
`[field com.caucho.config.types.ResourceRef#_location]`

这个参数是从`AnnotatedMember`类的`getFullName()`获取

代码含义前文已有描述。

8. 进入`findImplicitPropertyName()`函数，再次进入`_findConstructorName()`，并携带`[field com.caucho.config.types.ResourceRef#_location]`

发现`_findConstructorName()`函数下有一个类型判断，该类型判断是判断进入的参数是否是`AnnotatedParameter`对象实例，通过观察代码，发现`AnnotatedParameter`继承自`AnnotatedMember`，而进入的参数是`AnnotatedMember`类型，因此参数是其父类对象实例，根据JAVA多态规则，该参数显然不能是子类对象实例，而之所以参数可以传进来，是因为`Annotated`是`AnnotatedMember`的父类，也就是向下转型合理，向上转型是不可以的。该函数执行结果为`null`。

9. 回到`_addFields()`（该函数主要目的是为了遍历Json串对象下的所有字段属性名以及对该属性名的一些判断）函数，继续往下跟进，发现`implName`通过`getName()`获取到字段名`_location`

继续向下，进入`hasIgnoreMarker()`这里是判断字段是否被标记为可忽略，可以忽略的话就不会严格按照json串来映射对象，通俗的说就是Json串里面可以有对象所不包含的字段。

继续判断该字段是否被声明为`transient`（JAVA中添加该属性，则字段不会被序列化）。

10. 继续跟进，断点到`build()`函数，经过几次遍历上文提到的`HashMap`，此时`field`字段值如下，这是由`FieldBuilder()`创建的，这是类内字段属性的完整表示。

`private java.lang.String com.caucho.config.types.ResourceGroupConfig._lookupName`

11. 所序列化的类中有多少字段就会遍历多少次，且所遍历到的字段都会存入一个`POJOPropertyBuilder`中。
