
# 对一个apk的协议分析

过年的时候看见一个apk，因为肺炎在屋头宅的发霉，拿出来玩玩，小白图个乐，大佬乎喷。

## Apk信息

![](https://bbs.kanxue.com/upload/attach/202003/840122_ZA5TBNPQDDGN4ZB.png)

这个app的协议是明文，只有一个`sign`签名。

### 1 APK的协议

#### 长链接的心跳

![](https://bbs.kanxue.com/upload/attach/202003/840122_6ZD8RUYBC7C9PM6.png)

返回值

```
{"cod":"0","type":"5","msg":"\u83b7\u53d6\u6210\u529f","username":"ii12qqaa","vip":"\u666e\u901a\u4f1a\u5458","time":"8888888888","jifen":"0","login":"106","userid":"4232206e2a","token":"9f1ff9900178b5d0fe3db2a37d078ac5201c02380ebb0ba71a3bbf71bb2a8b2c0e18907e0383e8341dc00ee45fbc6923be0313f558197894"}
```

#### 登录和注册

![](https://bbs.kanxue.com/upload/attach/202003/840122_CG2ZXU5ZAKB5P3F.png)

`Type` 1=登录 2=注册

`Username` 和 `password` 是账户和密码

`imei` 手机序列号

`t` 是时间戳

`sign` 是加密验证

返回值：

```
{"cod":"0","type":"2","msg":"\u6ce8\u518c\u6210\u529f","username":"yyyyyy1234qq","vip":"\u666e\u901a\u4f1a\u5458","time":"8888888888","jifen":"0","login":"0","userid":"06ef899b13","token":"9f1ff9900178b5d0fe3db2a77951decf2913053b5cbb0ba71a3bbf71bb2a8b2c0e18907e0383e8341dc00ee45fbc6923be0313f558197894"}
```

### 2，搜索协议的逻辑

`Name` 是搜索内容

![](https://bbs.kanxue.com/upload/attach/202003/840122_H93KVC89FS3ZCF7.png)

应用上点击列表后向服务器发送 `urn:btih:` ，时间戳，`sign`

![](https://bbs.kanxue.com/upload/attach/202003/840122_46ND366MES2D9ZN.png)

返回值

```
{"msg":"0","list":[{"name":"1.rm","data":"83a371d890c1178e8ecc8c060f17d2acf4a0eff707955035be407f7ce707d8ed49d5689a4abbffb8","size":""}]}
```

再次在应用上点击

![](https://bbs.kanxue.com/upload/attach/202003/840122_PEPAD8JNPXSPMYY.png)

返回值：

```
{"url":"976K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5J5x3q4)9J5k6e0M7$3i4K6u0W2x3U0f1H3i4K6u0W2x3e0k6Q4x3V1k6$3K9h3c8W2L8#2)9J5k6i4m8Z5M7q4)9K6c8X3S2G2M7%4c8Q4x3@1c8K6P5W2)9J5k6h3u0@1k6Y4y4Q4x3X3g2E0j5h3W2D9i4K6u0W2k6Y4c8F1i4K6u0W2M7i4q4Q4x3X3g2U0L8$3#2Q4x3U0k6S2L8i4m8Q4x3@1u0U0L8$3c8W2i4K6y4p5z5o6N6V1x3h3f1H3z5r3b7H3z5e0u0U0x3h3b7H3y4e0f1J5y4K6V1@1y4K6M7#2y4r3b7#2y4o6t1H3z5e0g2S2j5h3c8W2x3$3c8T1x3r3y4X3y4o6j5I4z5h3c8X3x3e0N6W2z5o6q4V1x3e0m8T1y4X3f1@1x3e0c8U0y4r3k6W2x3e0l9I4k6e0x3$3x3U0u0T1j5X3y4U0x3o6W2X3y4o6f1@1x3o6c8U0x3e0N6T1y4U0l9I4k6e0l9#2k6o6t1^5y4U0q4T1k6X3t1@1y4K6V1&6j5U0M7@1x3r3f1H3z5o6N6W2x3X3b7&6y4r3q4T1z5o6R3$3k6U0N6Q4x3U0k6S2L8i4m8Q4x3@1u0V1j5i4c8S2i4K6y4p5z5o6y4S2x3K6M7I4k6o6R3&6x3r3x3I4x3e0M7^5k6e0S2W2j5$3x3^5j5K6l9$3x3r3j5I4y4$3b7J5j5h3y4X3y4r3p5H3k6h3k6X3y4K6l9%4z5e0f1#2x3o6x3#2j5X3f1@1x3o6N6X3y4$3y4W2y4K6l9%4k6o6S2W2k6o6b7&6k6o6f1$3z5o6W2S2y4r3q4T1j5X3k6X3j5U0R3I4k6o6x3H3j5h3q4V1j5K6N6X3j5U0R3#2k6e0l9^5x3U0j5#2z5r3x3%4","cookie":"b4d29c58","msg":"0"}
```

这个就是视频源了，开始寻找`sign`的签名算法。

### 2 apk源码

在`com.Soku.jxActivity`的内部类`madaper`中`onBindViewHolder`方法发送`hash`以及`data`拿到视频地址

![](https://bbs.kanxue.com/upload/attach/202003/840122_H9E83RQE89PPTTB.png)

`sign`是调用`com.Soku.Sign.getsign`方法取返回值,传入的参数为 `hash+time`

`BuildConfig.VERSION_NAME`的值是 `6.3"`

![](https://bbs.kanxue.com/upload/attach/202003/840122_ZA7QKAWA4SC6AR4.png)

接下来调用 `suritySignature.sign()`，这个 类位于`ibaba.wireless.security.jaq.SecuritySignature`；

`SecureSignatureDefine.OPEN_KEY_SIGN_INPUT`值为`"INPUT"`

![](https://bbs.kanxue.com/upload/attach/202003/840122_77WFZHYE683TSVN.png)

之后是一个接口类`IsecureSignatureComponent`；他的实现在`lib\armeabi\d1libsgmain.so`中

但是这并不是一个so文件而是一个jar包。

我通过ddms绘制函数轨迹得到完整的调用链。

`com.alibaba.wireless.security.a.h.a.sign()`->

`com.taobao.wireless.security.adapter.a.b.a()`->    //判断传入的参数map类型

`com.taobao.wireless.security.adapter.a.b.a()`;->  //重载的函数， 这个函数的作用是将Unicode字节码转成UTF-8

`com.taobao.wireless.security.adapter.a.b.a()`;->

`com.taobao.wireless.security.adapter.JNICLibrary.doCommandNative`（int i, Object... objArr）；

最后调用了native函数。传入的参数为`10401`，`new object{ hash+time+“6.3“,"3be780bc-8cf2-446c-9e03-de3a8fd9d0a7", Integer.valueOf(3),"0335"}`

![](https://bbs.kanxue.com/upload/attach/202003/840122_DSTXYUQHYE79HKP.png)

![](https://bbs.kanxue.com/upload/attach/202003/840122_84W8XDGCGTEAHVQ.png)

接下来就开始对 `libsgmain-5.4.56.so`的分析

### 3 libsgmainso.so的分析

Jar包中的`lib\armeabi\libsgmain-5.4.56.so`，简单粗暴的将所有节头的偏移和大小置0后使用ida打开

这个so里面很多的流程混淆，具体可以参考这篇大佬的文章

[详细分析一款移动端浏览器安全性](https://bbs.kanxue.com/elink@38bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0p5%4z5e0l9^5x3l9`.`.)

03cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0p5%4z5e0l9^5x3l9`.`.

很多的函数并不能F5

经过分析，函数开始处与函数结束处取一个内存中的值判断是否相等，如果不相等就跳转另一个地址

并且这个值并不参与运算中来，所以修改后并不影响我们的分析并且可以愉快的F5了

![](https://bbs.kanxue.com/upload/attach/202003/840122_HC3C79H476S6QQN.jpg)![](https://bbs.kanxue.com/upload/attach/202003/840122_SEJ7HCQJVRT7ZUE.png)

执行流程

首先so会去寻找 `Boolean`，`String`，`Integer`并做方法全局化引用。

然后`0xb9b0`处解密加载

`com/taobao/wireless/security/adapter/common/HttpUti`，

解密

函数

`sub_903F0` 他的第一个参数是要解密的数据，第二个是返回值，三是

要解密的数据的大小

首先申请一段56byte的内存(就叫`malloc_56`)再申请11byte的内存，将后面这个11byte的内存地址放入

`malloc_56`

的首地址。

这

11byte的内存存入的是10字节的密钥RC4算法，

`malloc_56[1]`是0，

`malloc_56[2]` ,是11，后面的都是一些函数地址了

![](https://bbs.kanxue.com/upload/attach/202003/840122_ETWV9QGW7QHT3JY.png)

密钥`DcO/lcK+h? m3c*q@` （好像其他版本的密钥都是一样的）

![](https://bbs.kanxue.com/upload/attach/202003/840122_WEAAK8Q6PSRP4SZ.png)

![](https://bbs.kanxue.com/upload/attach/202003/840122_EDQ4W8N27WWHZE2.png)

接着再申请一个`malloc_56`，首地址存放要解密的数据地址。通过一个函数`75DDA`里的switch去调用`sub_7C700`

这就是RC4解密函数

`RC4_Initialization`初始化xor盒子

`RC4_Decrypt`解密

![](https://bbs.kanxue.com/upload/attach/202003/840122_VES5CSNWAFSVZHX.jpg)

`sub_7CA82`  就是

`RC4_Decrypt`

![](https://bbs.kanxue.com/upload/attach/202003/840122_NTX8SVH866MBQQH.png)

经过几次解密加载其他一些类后才会在偏移`0xB986`解密并加载`com.taobao.wireless.security.adapter.JNICLibrary`。

![](https://bbs.kanxue.com/upload/attach/202003/840122_UN7QYKRNC2K77EV.png)

在`JNICLibrary`加载之后并不会马上调用`RegisterNatives`函数，而是继续解密加载其他的类并创建全局变量。最后在偏移`0xbdce`处注册`doCommandNative`函数，注册函数地址为`0xbc38`。

![](https://bbs.kanxue.com/upload/attach/202003/840122_NBE9KPU8PFKDDRT.png)

`doCommandNative` 函数，

`doCommandNative` 的返回值是`v6`，这个值是由`loc_9CD0`函数得到的

![](https://bbs.kanxue.com/upload/attach/202003/840122_5RATMK2TRBKCZFU.png)

可以看到返回值`v9`是由`off`函数得到的，这个函数由`sub_9630`解密得出

`doCommandNative`传入的参数为`1`，`4`，`1`，`1`，`m_off`，`0`。

得到的地址为`0x29d0d`

![](https://bbs.kanxue.com/upload/attach/202003/840122_QE5ANUPU4HZQF4D.png)

`m_off + 16` 是Java传入的`objarray`，`m_off + 12`是`env`（这个方法到`sub_29F9C`并没有结束，只是跳转过去了而已）

```
void __fastcall 29E6C(int m_off, _DWORD *a2, int a3, int a4, int a5, int a6, )
{
  v21 = m_off;
  v22 = *(_DWORD *)off_ABBA8;//获得这个位置的值在函数末尾再次获取
  *a2 = 99;
  v23 = *(_DWORD *)(m_off + 16);
  (*(void (**)(void))(**(_DWORD **)(m_off + 12) + 692))();// JNI::GetObjectArrayElement 
  v24 = *(_DWORD *)(v21 + 12);
  sub_9A568();//处理age[0] (hash+time+6.3)
  v25 = *(_DWORD *)(v21 + 16);
  sub_9C6E0(*(JNIEnv **)(v21 + 12));//处理age[1] key
  sub_9C7AC(*(_DWORD *)(v21 + 12), *(_DWORD *)(v21 + 16), 2);//处理age[2] 调用intValue
  v26 = *(_DWORD *)(v21 + 16);
  sub_9C6E0(*(JNIEnv **)(v21 + 12));处理age[3]
  sub_29F9C()
}
```

`sub_9A568`函数 malloc一段内存空间，大小为`ArrayLength*4`

![](https://bbs.kanxue.com/upload/attach/202003/840122_ZBG7BQPUZFUWZ43.png)

这里有个小bug

```
     v8 = ((int (__fastcall *)(JNIEnv *, int, int))(*v4)->GetObjectArrayElement)(v4, v12, v7) == 0;
        v9 = 0;
        if ( !v8 )
          sub_9A138();
        *(_DWORD *)(v13 + 4 * v7++) = v9;
```

`if(!v8)`?  ?  `V8`是`objectarray`,获取成功了直接丢了不要？？？以至于后面对这个`malloc`加载获取值时 `LDR R0 ，0x0000000`？？？

困扰了我好多天，还以为后面还有函数对这个`malloc`做了操作

汇编代码

![](https://bbs.kanxue.com/upload/attach/202003/840122_6E6RFN8H3YM4CZR.png)

`BLX R3`就是去执行的`GetObjectArrayElement`。

`BEQ`相等则跳转，转换成伪c 应该是`if（v8）`

![](https://bbs.kanxue.com/upload/attach/202003/840122_7JXU3WSWS26FW53.png)

在`sub_9A138`函数中调用Java层方法`String.getBytes`

(

刚刚得到的`ObjectArray`

)，

再得到它的`ArrayLength`和`ByteArrayElements`

申请`malloc`空间大小为56(`malloc_56`)，跟解密类名一样的结构体

再将

`ByteArrayElements`拷贝到 `malloc_arr`里面 ， `sub_9A568`函数结束.

![](https://bbs.kanxue.com/upload/attach/202003/840122_XVDT8RG6M4AG3JQ.png)

处理`key`的函数`sub_9C6E0`与这个差不多 获取`key`的`ObjectArray`，调用`GetStringUTFChars`，申请`malloc_32`,`malloc_128`,将

`malloc_128`地址放入

`malloc_32[0]`,

大小放入

`malloc_32[2]`,再就是一些函数地址。之后将`utf8`格式的`key`拼接到
