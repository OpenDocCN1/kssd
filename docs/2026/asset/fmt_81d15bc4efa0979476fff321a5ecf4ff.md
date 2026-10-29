
# 源码分析Jackson反序列化漏洞

前言：本次分析从Java序列化和反序列化源码开始分析，进一步分析Jackson源码，找出造成漏洞的原因，最后以Jackson2.9.2版本，JDK1.80\_171，resin4.0.52，CVE-2020-10673为例复现漏洞。

## 一、JAVA反序列化原理

### 1.1 Class对象

每一个类都有一个`Class`对象，`Class`对象包含每一个类的运行时信息，每一个类都有一个`Class`对象，每编译一个类就产生一个`Class`对象，`Class`类没有公共的构造方法，`Class`对象是在类加载的时候由JVM以及通过调用类加载器中的`DefineClass()`方法自动构造的，因此不能显式地声明一个`Class`对象。在类加载阶段，类加载器首先检查这个类的`Class`对象是否已经被加载。如果尚未加载，默认的类加载器就会根据类的全限定名查找`.class`文件。一旦某个类的`Class`对象被载入内存，我们就可以它来创建这个类的所有对象以及获得这个类的运行时信息。

获得`Class`对象的方法：

1).`Class.forName("类的全名")`;//`com.xx.xx.xx`

2).实例对象`getClass()`

3).`类名.class`

### 1.2 反射

JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

实现Java反射机制的类都位于`java.lang.reflect`包中：

1).`Class`类：代表一个类

2).`Field`类：代表类的成员变量（类的属性）

3).`Method`类：代表类的方法

4).`Constructor`类：代表类的构造方法

5).`Array`类：提供了动态创建数组，以及访问数组的元素的静态方法

简单反射调用代码

## 二、Jackson反序列化漏洞分析

`props`中保存了字段的许多属性。且这些字段，会先从父类开始遍历，再遍历子类，以下是`ResourceRef`和`ResourceGroupConfig`的所有字段

![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-YzN53mPo-1598531061444)(media/0c563788aced6aa7a65b8b93ce9ffef7.png)\]](upload/attach/202009/899781\_UTCAWT53Z6F25TZ.jpg)

12.属性字段收集完成之后，回到`collectAll()`函数，再次进行收集方法的过程

![在这里插入图片描述](upload/attach/202009/899781\_NUR7PGM79VCHC59.jpg)

13.进入`_addMethods()`函数，首先`memberMethods`会遍历所有的成员方法。构造迭代器对象，对成员方法进行迭代遍历，将每一个方法在`getParameter()`中判断所传入的方法参数数量。

![在这里插入图片描述](upload/attach/202009/899781\_M9SGWC4KJ6RSTTD.jpg)

14.进入`_addSetterMethod()`中，一直跟踪，进入`okNameForSetter()`中，判断方法名是否以`set`开头。

![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-QAY4qXvn-1598531061445)(media/00c759c5d9135a9e0d02bff79ccf95f4.png)\]](upload/attach/202009/899781\_RVK9V8QQR4HUJS4.jpg)

进入`legacyManglePropertyName()`对方法名去除`set`并变成小写，即将`set`方法对应的字段取出（此处涉及到JAVA开发的内容，是由于封闭性以及安全性，要想使用类内的私有成员必须要对外开放一个接口，而这样的接口一般写法是将变量名首字母大写，前面加上`get/set`，`get`有返回值无参数，`set`无返回值，一般有一个参数，因此前文判断方法的类别是依靠参数以及开头三个字母判别，这样的形式大多用在实体类内，也是数据交互传输的中转类）

![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-lXIfWTiJ-1598531061446)(media/4c0d2566f5c29858b6c372c08c0f77f5.png)\]](upload/attach/202009/899781\_KUQXJNBBET99SAY.jpg)

在`_property`内，寻找是否存在上一步的对应字段名称

![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-ggGH2vBU-1598531061446)(media/9a005a1269f403bf91633b58327eb4ed.png)\]](upload/attach/202009/899781\_AQ9YS6ZWSPPG32M.jpg)

此处，如果不存在，则去缓存中寻找，在牵扯到缓存的时候，就必须要考虑多线程安全问题，这里是通过`ConcurrentHashMap`（通过`volatile`语义以及CAS操作实现，详细内容不做赘述）来存储，取出缓存后返回。

最后回到`_property()`，一通判断之后，程序认为该字段值不存在相对应的`set`方法，就将该字段存入`props`中

![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-ap7AIYHC-1598531061447)(media/22ff90f60820b70e957f703a971cd792.png)\]](upload/attach/202009/899781\_YVKGRVWFW8W7XRV.jpg)

15.先进入`_addGetterMethod()`中，一直跟踪，到`okNameForRegularGetter()`，在此判断方法名是否以`get`开头，同时还会判断该方法是否是回调函数，是否是`MetaClass`或者`Groovy`的内容。

![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-u8CEFvR4-1598531061448)(media/361fee38b002c372f6ece2822638bbbd.png)\]](upload/attach/202009/899781\_9H9Z9SG5NSXYJUV.jpg)

进入`legacyManglePropertyName()`对方法名进行去除`get`以及首字母小写处理。处理完返回字段名。进入`prop`查找是否存在该字段

![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-CAh3vvsR-1598531061449)(media/72966c7dd36d486ebef2d06097985756.png)\]](upload/attach/202009/899781\_PREFYVJ4UQPJ76A.jpg)

由于先前的判断，已经添加过了，因此，可以查找到。可以查找到的话就在相应的字段上添加所收集到的对应的方法

注：如果方法开头不是以`get/set`开头，则不进行任何处理。

总结：Jackson是依靠收集到的`get`和`set`方法名，对方法名进行处理，得到字段值，与原收集到的字段值对比，可以对比到，就将该`set/get`方法添加到该字段下，对应不到，就将处理好的字段名添加，并将该方法添加到新的属性字段下，即原类所拥有的属性字段就会丢失`get/set`方法。

以下小案例证明：

![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-bxxQXLio-1598531061449)(media/85e5f6d3fc1e7385b09b5c4612f77ab3.png)\]](upload/attach/202009/899781\_TJKUMZSS2QFZDZB.jpg)

![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-ErmbpRYw-1598531061450)(media/ff02e0c3d274e0a5b08994462932d635.png)\]](upload/attach/202009/899781\_JU4J6B55WX96C6N.jpg)

17.接下来就是对没有注解的字段以及没有`get`和`set`方法的字段进行删除。

往下就没什么可以说的了，观察调用栈，最后会调用到`readValue()`，至此反序列化完成

![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-o07j6gJR-1598531061451)(media/a0ba17c43b21b06654eea33b2d7d2f4e.png)\]](upload/attach/202009/899781\_C9FVGARP8MXWP3U.jpg)

17.当进行序列化时，先调用`set`方法对该字段赋值，再通过前面生成的`get`方法取出需要序列化字段的值。

![在这里插入图片描述](upload/attach/202009/899781\_Z7RXDQ6PERN6H6J.jpg)

![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-7LizhbkC-1598531061452)(media/b8203489b5c003ee88bc21d9eb9d574c.png)\]](upload/attach/202009/899781\_X5C4W9K9VF6CN28.jpg)

由此处看以看出，`lookupName`属性并不是原类里面的值，而是Jackson自己生成的。

### 2.2 writeValueAsString()

1.在`serializeFields()`函数处下断点，此处是序列化时关键函数

![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-hOO1RN7w-1598531061452)(media/43320bbb6c2c7dc0c81c219c07a4a62b.png)\]](upload/attach/202009/899781\_RVWDAS7G36B82BT.jpg)

直接将值传入`serializeAsField()`，其中`bean`存储了两个实体字段的值

![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-4XzsHNNh-1598531061453)(media/e9ecfa217b30689efe553b18138f6078.png)\]](upload/attach/202009/899781\_NEXPQ6QWV2WPU5P.jpg)

2.此处就出现了反射调用，直接调用对应属性字段的`get/set`方法，就会触发命令执行。

![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-WromRdDE-1598531061453)(media/48cbfdfa265981cd47be1259891dd90d.png)\]](upload/attach/202009/899781\_Q5A2CCMBRW4R8WV.jpg)

### 2.3 案例说明原理

1.第一种情况

`JacksonMain.java`

![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-SAV6a0VE-1598531061454)(media/cfb7b4f2e66e896bb9140fae67bd6ae5.png)\]](upload/attach/202009/899781\_BH5UZJQVJA4HAUM.jpg)

`User.java`

![在这里插入图片描述](upload/attach/202009/899781\_JKB7H4CKJX6ECCH.jpg)

正常运行后会执行命令。

2.第二种情况

`JacksonMain.java`

![在这里插入图片描述](upload/attach/202009/899781\_AU68NZUGPTBZHY9.jpg)

`User.java`

![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-6dH1GGBg-1598531061456)(media/0e4010e98de6211f6191a924ac4cb111.png)\]](upload/attach/202009/899781\_27VZBNJ6DUJF83N.jpg)

运行后报错。找不到`name`字段。根据以上两种情况即可说明，Jackson在序列化时出现了很大的问题，仅仅只是根据`set/get`方法名进行判别。要想触发该漏洞，只需要利用`set/get`方法后面的属性字段值即可产生反射调用。

## 三、利用方式

1.Resin

非常流行的支持servlet和jsp的引擎，速度非常快。Resin本身包含了一个支持HTTP/1.1的WEB服务器。虽然它可以显示动态内容，但是它显示静态内容的能力也非常强，速度直逼APACHE SERVER。许多站点都是使用该WEB服务器构建的。

Resin也可以和许多其他的WEB服务器一起工作，比如Apache server和IIS等。Resin支持Servlets 2.3标准和JSP 1.2标准。熟悉ASP和PHP的用户可以发现用Resin来进行JSP编程是件很容易的事情。

2.JNDI

1).JNDI是命名与目录接口，是一组应用程序接口，它为开发人员查找和访问各种资源提供了统一的通用接口，可以用来定位用户、网络、机器、对象和服务等各种资源。

2).J2EE 规范要求所有 J2EE 容器都要提供 JNDI 规范的实现。JNDI 在 J2EE 中的角色就是“交换机” —— J2EE 组件在运行时间接地查找其他组件、资源或服务的通用机制。在多数情况下，提供 JNDI 供应者的容器可以充当有限的数据存储，这样管理员就可以设置应用程序的执行属性，并让其他应用程序引用这些属性（Java 管理扩展（Java Management Extensions，JMX）也可以用作这个目的）。JNDI 在 J2EE 应用程序中的主要角色就是提供间接层，这样组件就可以发现所需要的资源，而不用了解这些间接性。

3).举例说明

配置JNDI访问数据库

![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-YUEdDfoU-1598531061457)(media/c9e771e7325bdb5b2d77f92b31c446f3.png)\]](upload/attach/202009/899781\_PY9G28684QNKZYD.jpg)

同样的JNDI也可以访问LDAP服务器。

3).LDAP

LDAP是一种轻型目录访问协议，首先LDAP是一种通讯协议，LDAP支持TCP/IP。协议就是标准，并且是抽象的。可以把LDAP理解成存储数据的数据库。像是其他数据库一样，LDAP也是有client端和server端。server端是用来存放资源，client端用来操作增删改查等操作。而我们通常说的LDAP是指运行这个数据库的服务器。

3.根据上文提到的黑名单列表，寻找一个不存在于黑名单的类，该类具有远程访问能力，所以使用Resin容器下的`ResourceRef`类下的`getValue()`函数，配置ldap服务让服务器连接自己的ldap服务器，获取本地恶意文件。

4.进入`ResourceRef`源码，他是继承自`ResourceGroupConfig`，进入该类，发现以下函数。跟踪`getLookupName()`

![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-VKawS7Qx-1598531061457)(media/0d7495634004040fd4794ce3a249ef74.png)\]](upload/attach/202009/899781\_R88S5KDGRFGG69X.jpg)

在`ResourceRef`下面存在`getValue()`，里面有`Jndi`调用，这里就可以访问远程服务。借助Jackson反序列化以及序列化的特性（可以主动调用字段值的`get`方法，即主动调用`getValue()`方法）就可以实现命令执行。

![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-YYeqJ49G-1598531061459)(media/bc1b956ff5aa371b769ab7ffd4e24360.png)\]](upload/attach/202009/899781\_2WRE3CUM8U7VDRG.jpg)

5.攻击图解

![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-Q6EL0HgA-1598531061460)(media/82d84e239a27019894ea855096c92196.png)\]](upload/attach/202009/899781\_GVU4EMQEBJU45DK.jpg)

## 四、真实环境测试

1. 开启LDAP服务，配置目录指向本地恶意文件，监听本地Tomcat端口

2. 构造Json数据，指向本地IP地址，指向本地文件`Poc`

![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-Z3nZpaow-1598531061462)(media/6e4d0651abab23602512f34abf5f4fac.png)\]](upload/attach/202009/899781\_RNDAXXQWCETAE97.jpg)

将`Poc`编译成class文件，因为Jackson序列化时会反射调用这个`Poc`执行。

![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-6FYFP1Ah-1598531061463)(media/bce0e32e868c294aa618fb32e309d05d.png)\]](upload/attach/202009/899781\_3USRHE7ZT44JPA5.jpg)

## 五、挖洞思路

实体解析时，所使用的`get/set`方法是自身生成还是利用反射获取`get/set`方法，如果是自身生成，则单步调试，跟进源码查看是什么样的生成规则，如果是类似Jackson的生成方式，则可能存在类似漏洞。目前很多的实体类映射的组件都是与Jackson底层实现相同。

## 六、防御措施

1. 将`ResourceRef`类加入黑名单

2. 对Jackson数据做校验，禁止数据包含项目未知类

3. 升级Jackson
