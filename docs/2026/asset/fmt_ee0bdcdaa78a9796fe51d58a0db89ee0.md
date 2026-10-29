
![](https://bbs.kanxue.com/upload/attach/202104/878914_HQW8JKS4PBTARJ3.png))

随便打开一个文件查看，发现都是加密状态，这个和之前遇到的lua 比较像， 资源文件都是加密的

![](https://bbs.kanxue.com/upload/attach/202104/878914_3HWNY23VACD29PW.png))

之后查看了其他的一些文件，都是加密的

## 分析 so 文件

apk 只有一个 so `libsec.so`

![](https://bbs.kanxue.com/upload/attach/202104/878914_XMCW2FVN7VHNYR5.png))

使用 ida 打开后 `libsec.so` 文件， 查看一下流程和段结构

![](https://bbs.kanxue.com/upload/attach/202104/878914_2UAREQYF9AFZ3ZN.png))

![](https://bbs.kanxue.com/upload/attach/202104/878914_UDVAG2C78QF6W3C.png))

so 文件段结构正常，流程简单清晰明了，可以发现没有做任何安全防护，那 so 层就很容易分析了。

### 搜索字符串

快捷键 `shift+F12` 搜索字符串

> 科普一下：我们知道 JNI 分为 动态调试和动态调试，静态注册，每次调用 native 方法，都要去寻找，方法都是以 Java+包名+类名+方法名 这种结构，根据函数名建立起 java 方法与 JNI 函数的对应关系，这种方式简单明了，但是必须遵守注册规则，效率低，基本很少在 app 中遇到，而动态注册，是通过 `RegisterNatives` 方法手动完成 native 方法和 so 方法的绑定，通过函数映射表查找函数，因此运行效率要比静态注册高。方法一般可以通过分析 `Jni_onload` 得到

先搜索 `java_` 没有返回，那就没有使用 JNI 静态注册

![](https://bbs.kanxue.com/upload/attach/202104/878914_9N49NDH4E7VE6PF.png))

然后在查看 `jni_onload`

![](https://bbs.kanxue.com/upload/attach/202104/878914_KH75W48JYYY9XPY.png))

跟进去，发现字符串也没加密，arm 汇编也很清晰

![](https://bbs.kanxue.com/upload/attach/202104/878914_XDMM6A55NAVUYGA.png))

先修改一下参数类型，ida 自己识别不出来，需要我们手工改一下，修改为正确的类型，`JNI_onload` 参数是 `JavaVM*`

![](https://bbs.kanxue.com/upload/attach/202104/878914_ZHQC4E4JYVZPXV8.png))

重新识别一下，得到了 `GetEnv`

![](https://bbs.kanxue.com/upload/attach/202104/878914_6XDBJ3YH7GE54BS.png))

在修改一下 `GetEnv` 的第一个参数，为 `JNIEnv *` ,改完后就可以得到清晰的逻辑了

![](https://bbs.kanxue.com/upload/attach/202104/878914_9828468DGV34SHP.png))

识别一下

![](https://bbs.kanxue.com/upload/attach/202104/878914_F2HG74QA7WMK5PP.png))

识别后的结果和跳转地址都有了，根据刚才的介绍，我们知道函数的映射关系都在 `RegisterNatives`， 通过 `off_8804` 跳转

![](https://bbs.kanxue.com/upload/attach/202104/878914_XUWPTYGJXM8DQ6X.png))

找到注册函数后，可以看到 JNI 的所有方法都在这块，分为三部分： 参数类型，方法地址和函数名

![](https://bbs.kanxue.com/upload/attach/202104/878914_RGMKQKP4D4HZ67A.png))

方法很清晰

![](https://bbs.kanxue.com/upload/attach/202104/878914_DA7J735KCQ479X7.png))

接下来 就很容易了，找到 so 加载的地方，查看调用 native 的函数，`frida hook` 即可

参考： APICloud 解密本地资源到逆向 APP 算法到通用资源解密

## 分析资源加载流程

先思考 APP 对资源的加载流程

可能为:

1) `WEBVIEW` -> 加载页面 -> 拦截/查找本地文件 有 -> 解密/写回数据
2) `WEBVIEW` -> 加载页面 -> 拦截/查找本地文件 无 -> 请求网络文件

这里有个共同的点都是需要 拦截,而 `WebView` 只有一个实现这个功能的接口: `WebViewClient.shouldInterceptRequest`

![](https://bbs.kanxue.com/upload/attach/202104/878914_NFNN9KS9PWJBZRY.png)

参考： APICloud 解密本地资源到逆向 APP 算法到通用资源解密

搜索 关键词 `shouldinterceptResquest`

![](https://bbs.kanxue.com/upload/attach/202104/878914_ET7SXD67GV2Z36E.png))

跟进第一个，分析函数逻辑

![](https://bbs.kanxue.com/upload/attach/202104/878914_5QQ83B7578KSXQB.png))

在跟 `b.d(url)` 进去查看

![](https://bbs.kanxue.com/upload/attach/202104/878914_2EQ78DXDAYTYAWA.png))

调用另一个方法，继续跟进去

![](https://bbs.kanxue.com/upload/attach/202104/878914_2ZANA255Y2FXNBD.png)-20210420110621781.png)

然后根据函数名称可以猜出他是从 url 获取文件扩展名

返回 `shouldInterceptRequest` 继续往下跟

![](https://bbs.kanxue.com/upload/attach/202104/878914_WR4882V7V567MWV.png))

对得到的后缀扩展名进行判断

![](https://bbs.kanxue.com/upload/attach/202104/878914_HNKD6VX6YJUNDJR.png))

这里的 `a` 记录的是扩展名列表

![](https://bbs.kanxue.com/upload/attach/202104/878914_6CJ2H9JENKFA2WS.png))

继续往下跟

![](https://bbs.kanxue.com/upload/attach/202104/878914_76WAX5VA3S4GYFE.png))

跟进去

![](https://bbs.kanxue.com/upload/attach/202104/878914_XNY9SY25UV9GFJU.png))

获取文件路径，

![](https://bbs.kanxue.com/upload/attach/202104/878914_UWWM4GP8EU9GEUD.png))

到这里 发现已经开始做解密工作了

![](https://bbs.kanxue.com/upload/attach/202104/878914_88H6ZKJWJZV6RQW.png))

继续往下

![](https://bbs.kanxue.com/upload/attach/202104/878914_H9ZT3ASS5RW64T3.png))

基本到这里就分析出来了，下面就被 SDK 接管

![](https://bbs.kanxue.com/upload/attach/202104/878914_EJHTSQ7CD4G7F44.png))

从上一步 分析 下图位置，拿到结果 就可以 dump 出 source 了

![](https://bbs.kanxue.com/upload/attach/202104/878914_ZBQ2RARS86Y5TX3.png))

这里用的构造函数，在 `frida hook` 时 要用 `$init`

![](https://bbs.kanxue.com/upload/attach/202104/878914_S2WSH6DKQFVY4BW.png))

编写 hook 脚本

![](https://bbs.kanxue.com/upload/attach/202104/878914_VAH5EHCQEXWGEGA.png))

查看 dump 结果

![](https://bbs.kanxue.com/upload/attach/202104/878914_GP7YSQ9Y59E3Y7U.png))

通过 `adb pull` 到本地

![](https://bbs.kanxue.com/upload/attach/202104/878914_AYGAMRWHD8674WE.png))

在和之前的对比

![](https://bbs.kanxue.com/upload/attach/202104/878914_6BV6XYPT5UKBA8B.png))

解密就结束了，下来就是愉快的抓包 找 api 捡漏洞了 。

---

## 参考

https://bbs.pediy.com/thread-218656.htm

bd2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2j5i4m8A6j5$3I4G2N6h3c8Q4x3X3g2U0L8$3#2Q4x3V1k6p5k6i4k6Q4x3X3c8s2N6h3W2V1k6g2)9J5c8W2y4#2M7r3g2J5g2$3g2T1N6X3W2W2N6#2)9J5k6r3N6#2K9h3c8W2i4K6u0V1k6X3!0J5i4K6u0V1j5h3&6V1M7X3!0A6k6l9`.`.

https://bbs.pediy.com/thread-249457.htm

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)
