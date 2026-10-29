
# [原创] apk破解之“异常”破解

这次拿到一个新的apk，作者在上一次破解之后已经重写的代码，破解的关键也换了位置了，这次作者主要用到了异常，当在服务器上验证无该用户的数据时就会返回0，然后在调用`parseInt`方法时就会发生`NumberFormatException`异常，进而调用处理异常的代码，也就是提示“不成功”

注意：该文章用于技术交流，请不要用于非法破解，本人概不负责,错误疏漏之处还望海涵

apk下载地址：

55fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8S2L8W2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3M7$3S2S2M7X3g2Q4x3V1k6D9K9h3&6C8i4K6y4r3M7$3S2S2M7X3g2A6k6q4)9K6c8o6f1I4x3U0t1J5x3g2)9J5y4Y4g2C8i4K6y4p5x3U0l9I4y4K6x3^5z5e0V1^5

破解工具：这个是一个IDE 集成了反编译，回编，adb等等工具，非常不错 可以玩玩

标 题: 【下载】APK可视化修改工具：APK改之理（APK IDE）

作 者: 青椒

时 间: 2013-04-10,11:03:12

链 接: http://bbs.pediy.com/showthread.php?t=168001

视频地址：

198K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8S2L8W2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3M7$3S2S2M7X3g2Q4x3V1k6D9K9h3&6C8i4K6y4r3M7$3S2S2M7X3g2A6k6q4)9K6c8o6f1I4y4o6f1K6y4q4)9J5y4Y4g2C8i4K6y4p5x3U0l9I4y4K6x3^5z5e0V1^5

apk描述：apk正常安装后，需要注册，点击注册软件后，会有两个按钮，点击找回注册，正常情况下会显示“不成功”

如图：![](https://bbs.kanxue.com/upload/attach/201305/367996_x7nexlw56r6ygct.jpg)

## apk破解过程:

### (1)
该apk的文字提示将成为我们的利用对象,注册成功会提示"成功",当然你也可以找不成功,因为代码就在附近

### (2)
在`apk改之理`右上角的"替换和搜索"窗口中输入字符串"成功", 然后在"替换和搜索"窗口的下拉菜单中选择"转换为Uncode"
点击“搜索全部”按钮

### (3)
点击搜索之后，在搜索结果窗口中会出现全部出现“成功”的文件，如图，其中选中的文件就是注册模块，也就是 `BqimenDateInputActivity.smali`。其中还有一个文件是`BqimenDateInputActivityW.smali`，这个文件是繁体版的注册模块，我们无需理会（如果你问 为什么这个就是注册的文件，你可以观察该文件中的字符串，以及文件名就可以了解，他就是注册模块)

![](https://bbs.kanxue.com/upload/attach/201305/367996_4vcw2zvxnkb12x1.jpg)

### (4)
双击`BqimenDateInputActivity.smali`将会自动打开该文件，并且定位到该字符出现的位置，定位到该方法的开始位置，详细请看代码的注释

```
@可以看到注册的方法名称为s，参数为Lcom/forace/Bqimen/BqimenActivity/BqimenDa
@teInputActivity，背后的V为无返回值
@另外值得注意的是字段synthetic ，该字段是编译器合成的，非作者添加的，也就是如果
@你使用java DeCompiler进程查看
@Java级别的源码时，这些代码是看不到的，这也就是之前我搜索“成功”之类的字符串搜索不到的原因
.method static synthetic s(Lcom/forace/Bqimen/BqimenActivity/BqimenDateInputActivity;)V
```

### (5)
既然找到了源头，我们就顺藤摸瓜往下看看 作者到底做了什么，代码如下，详情请看注释

第一段代码：

```
.method static synthetic s(Lcom/forace/Bqimen/BqimenActivity/BqimenDateInputActivity;)V
    .locals 4

    :try_start_0  @添加异常代码
    new-instance v0, Ljava/lang/StringBuilder;  @添加字符串实例

    const-string v1, "QM"  @将QM初始化为QM

    invoke-direct {v0, v1}, Ljava/lang/StringBuilder;-><init>(Ljava/lang/String;)V    @用QV初始化字符串
    @获取getIMEI号
    invoke-virtual {p0}, Lcom/forace/Bqimen/BqimenActivity/BqimenDateInputActivity;->getIMEI()Ljava/lang/String;

    move-result-object v1

    invoke-virtual {v0, v1}, Ljava/lang/StringBuilder;->append(Ljava/lang/String;)Ljava/lang/StringBuilder;

    move-result-object v0

    invoke-virtual {v0}, Ljava/lang/StringBuilder;->toString()Ljava/lang/String;

    move-result-object v0
  @得到注册表
    invoke-virtual {p0, v0}, Lcom/forace/Bqimen/BqimenActivity/BqimenDateInputActivity;->regetkey(Ljava/lang/String;)Ljava/lang/String;

    move-result-object v1
@程序的关键

    const-string v0, ""

    new-instance v2, Lorg/apache/http/client/methods/HttpGet;

    invoke-direct {v2, v1}, Lorg/apache/http/client/methods/HttpGet;-><init>(Ljava/lang/String;)V

    new-instance v1, Lorg/apache/http/impl/client/DefaultHttpClient;

    invoke-direct {v1}, Lorg/apache/http/impl/client/DefaultHttpClient;-><init>()V

    invoke-virtual {v1, v2}, Lorg/apache/http/impl/client/DefaultHttpClient;->execute(Lorg/apache/http/client/methods/HttpUriRequest;)Lorg/apache/http/HttpResponse;

    move-result-object v1

    invoke-interface {v1}, Lorg/apache/http/HttpResponse;->getStatusLine()Lorg/apache/http/StatusLine;

    move-result-object v2

    invoke-interface {v2}, Lorg/apache/http/StatusLine;->getStatusCode()I

    move-result v2   @将返回的结果赋给v2
@这段是初始化链接，链接作者服务器的地方，如果找到用户的信息则返回非0的字符串，如果找不到则返回0    

   const/16 v3, 0xc8
   @将v2和0xc8相比，如果不相等则调转到cond_0，如果相等，则不跳转那么下执行提示“网络异常”    

if-ne v2, v3, :cond_0  
    invoke-interface {v1}, Lorg/apache/http/HttpResponse;->getEntity()Lorg/apache/http/HttpEntity;

    move-result-object v0

    const-string v1, "UTF-8"

    invoke-static {v0, v1}, Lorg/apache/http/util/EntityUtils;->toString(Lorg/apache/http/HttpEntity;Ljava/lang/String;)Ljava/lang/String;
    :try_end_0
    @双层异常，嵌套的异常，如果执行到这里说明网络不通畅，捕获ClientProtocolException异常，执行catch_0，提示网络异
  @常错误403，当catch_0没能处理异常，那么会给catch_1处理

    .catch Lorg/apache/http/client/ClientProtocolException; {:try_start_0 .. :try_end_0} :catch_0
    .catch Ljava/io/IOException; {:try_start_0 .. :try_end_0} :catch_1

    move-result-object v0
```

第二段代码：当网络通畅没什么问题时，也就是`if-ne v2, v3, :cond_0` 跳转到`cond_0`时会执行以下代码:
