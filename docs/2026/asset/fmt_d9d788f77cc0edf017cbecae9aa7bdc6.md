
# [原创]登录抓包逆向分析学习笔记

最近在学习ARM汇编和逆向方面的基础知识，抽空跟着“无名”大神的逆向数据分析视频学习了一下，以下是本人在学习过程中的一些心得和笔记，还望各位大牛们指正。

> PS：本人已对截图做了相应的打码处理，如有侵权或涉及敏感信息，还望告知，我会第一时间做出处理。谢谢！

## 一、抓包&反编译

1. 首先在登录时，抓包，看到相应的登录信息：

![](upload/attach/201802/510716_G77GCUETXPUC8MV.png)

通过POST信息可以看到，除了时间戳、手机号码、经过MD5处理的密码以及deviceid外，有一个“sign”的认证信息，故本次逆向分析就是针对“sign”的HASH方式进行分析。

2. 拿出Android killer对该安装包尝试反编译，顺利反编译，发现搜索“sign”关键字后的搜索结果太多，于是换了关键字“pawd”进行搜索后，发现只有两条结果：

![](upload/attach/201802/510716_5SC4HF26K48GKS4.png)

根据名称可以基本定位在下方`LoginRegisterManager`这个方法中。

3. 通过反编译后的java代码，顺着`pawd`向下寻找，于是找到“sign”的代码：

![](upload/attach/201802/510716_YWFE8U9SE79AFA8.png)

4. 通过查看，可以得出`sign`是通过`KeyGenerator`的`getsign`方法操作后获取，于是进入“KeyGenerator”方法，可以查看到这里调用了动态链接库(`System.loadLibrary "XX_key"`)：

![](upload/attach/201802/510716_FJUTX4CSACWSRDD.png)

## 二、so静态分析

1. 拿出神器IDA，打开apk包lib目录下的`libxx_key.so`文件，首先下意识的在Exports中搜索关键字“Java”，查看是否有相应方法调用，发现并没有，于是需要通过`JNI_Onload`中去寻找相应方法，进入`JNI_Onload`后，F5查看伪C代码，代码相对比较简练，`RegisterNatives`此处为JNI环境注册Native方法的通用做法：

![](upload/attach/201802/510716_3A9YB7Y5W7C5S3Z.png)

2. 点击`gMethods`进入查看，这是一个典型的`JNINativeMethod methods[]`数组，通常是3个参数：

参数1：Java层方法名

参数2：方法的签名以及返回值为Java String类型 `(Ljava/lang/String;)Ljava/lang/String;`

参数3：为SO库实现的方法名，这里强制转换成了函数指针，会指向SO中的方法（`generateKeyByParams`）

![](upload/attach/201802/510716_6JD4FUV8CAPM582.png)

3. 点击第三个参数，进入到`generateKeyByParams`方法内，同样F5查看伪C代码：

![](upload/attach/201802/510716_X2AR585DFNSPNN6.png)

4. 发现伪C代码居然将`generateKeyByParams`的方法只识别了1个传参（上一张截图中可以明显看出该方法是有3个传参的），通过上一张截图可以得知，3个参数的含义：

参数1：Env指针

参数2：Java层类对象

参数3：用户输入的String

5. `_ZN7_JNIEnv17GetStringUTFCharsEP8_jstringPh` ; `_JNIEnv::GetStringUTFChars(_jstring *,uchar *)` 表示将java传入的string转换为C的string格式

6. 通过对`generateKeyByParams`方法的分析，发现它在前面做了一些初始化的工作：

![](upload/attach/201802/510716_WFGQVQU2E7QD6S8.png)

大概有40字节的内存空间初始化，然后调用了`generate_key_by_value(v19, v4)`这个方法

7. 通过以上分析，我们可以初步断定，HASH处理应该就在`generate_key_by_value`方法内了

## 三、动态调试

1. 手机打开，`adb shell`运行`android_server`，运行APP，IDA远程调试连接，选中相应的APP进程

2. 在module中通过查找“`libxx_key`”，定位到`libxx_key.so`，然后查找到“`generate_key_by_value`”，在其开始处下断点，运行

3. 然后在手机上随便填写一个手机号码和12345678的密码，登录，此时IDA会停留在我们的断点处，根据前面所知，`generate_key_by_value`有两个传参，我们可以通过动态调试的方式查看`R0`，`R1`这两个寄存器中的内容，也就是说`v19`和`v4`两个参数了：

![](upload/attach/201802/510716_5S7Z5X96UKCNDGS.png)

![](upload/attach/201802/510716_E9UU2WW77A2DF8Z.png)

4. 通过查看可以发现`R0`即为我们提交时的POST封包的一些数据，而`R1`则为内存初始化的一段空间，此时为00

5. 此时记下`R1`的地址（我的是`BEFF34E4`，每个人会有所不同），然后在`generate_key_by_value`方法的结束处下断点，然后运行，IDA会停留在结束处的断点，此时在HEX-View处，按G，输入刚才记下的`R1`地址，查看`R1`寄存器中的内容：

![](upload/attach/201802/510716_ZQN7R6RSDPHVW3M.png)

可以发现`R1`的内容已经为HASH后的数据，将该段数据复制出来与抓包的`sign`数据进行比对：

![](upload/attach/201802/510716_CA2PS9TFWG2MAVX.png)

数据完全吻合，此时可以肯定，`generate_key_by_value`就是HASH方法关键所在。

## 四、算法分析

1. 为了分析HASH算法，可以在IDA静态分析中继续查看`generate_key_by_value`的伪C代码，通过查看代码，可以发现是典型的SHA1散列值函数组合生成

> 其大致的流程为`SHA1Init()`--> `SHAUpdate()` --> `SHA1Final()`

而代码中共调用了4次`SHAUpdate()`：

![](upload/attach/201802/510716_D7BG7YQTDBPBMAN.png)

`SHAUpdate(a)`，`SHAUpdate(b)`，相当于`SHAUpdate(a+b)`，于是我们可以通过代码看到这4次调用，第1次和最后1次，分别有`&v15`的一串字符串“bi2ipxazqodmw9hoety0h1wwgjvkttng”，推断大致的HASH流程为`SHAUpdate（&v15+v6+v8+v10+v12+&v15)`：

2. 为了验证是否正确，再次结合动态调试，在第一处`BL SHA1Update`处进入到`SHA1Update`方法中，在开始处下断

![](upload/attach/201802/510716_J57Y44WRHDQEXZB.png)

3. APP再次登录，IDA会停留到`SHA1Update`断点处，查看`R1`寄存器中的内容，发现“bi2ipxazqodmw9hoety0h1wwgjvkttng”字符串：

![](upload/attach/201802/510716_JUJBFWXHTXAHA7K.png)

4. 再次运行，IDA会再次停留在下断处，再次查看`R1`寄存器中的内容，此时变为“deviceid”：

![](upload/attach/201802/510716_VCAF5BFEN9GDSQB.png)

5. 继续运行，依次查看`R1`寄存器中的内容，“mobile”、“pawd”、“timestamp”、“bi2ipxazqodmw9hoety0h1wwgjvkttng”及其相应数据：

![](upload/attach/201802/510716_QMKETASX689MVDF.png)

6. 然后在`generate_key_by_value`方法的结束处下断，`F9`运行，在HEX-View中按G，查看之前记下的`R1`地址(`BEFF34E4`)，将已HASH后的数据复制出来

7. 然后将动态调试时获取到的“bi2ipxazqodmw9hoety0h1wwgjvkttng”、“deviceid”等数据放在一起，保证无空格、无换行，然后通过网上SHA1哈希后与抓包的数据比对，发现完全一致：

![](upload/attach/201802/510716_7CNSNTX95A8RUJE.png)

故之前推断的HASH流程完全正确。

未完待续，后面会补充`SHA1Update()`伪C代码分析的学习笔记。

通过两晚的学习，发现还是很有收获的，更加激发了逆向的学习动力，虽然已近不惑，凭着对逆向的热爱兴趣，一直以来都以自学和浏览帖子为主，很少有勇气发帖，希望通过看雪这个平台多与志同道合的朋友们交流，共同进步。
