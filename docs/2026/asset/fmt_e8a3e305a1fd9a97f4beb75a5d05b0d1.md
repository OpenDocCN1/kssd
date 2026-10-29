
# [原创]SO层逆向，某狗音乐登陆算法分析

## 抓包分析：

首先抓包看下参数情况

![抓包](upload/attach/202204/870029_W6HABYWTGY456P8.png)

可以看到这里未知的参数有4个

`P2`、`t1`、`t2`和`key`

---

## Java层逆向：

我们先hook一下有没有java层通用的加密算法

![java](upload/attach/202204/870029_UZVCF2RW3YDSC4R.png)

可以看到在登录调用的过程中触发了一些加密算法

我们搜索一下登录包的未知参数

看到我们有没有想要的

![java](upload/attach/202204/870029_DECQQBGJTBAHC7V.png)

定位到了`key`参数，发现是MD5加密

参数为：

`3116LnT6xpN3khm36zse0QzvmgTZ3waWdRSA104901648962363521`

接着我们连续调用几次

`3116LnT6xpN3khm36zse0QzvmgTZ3waWdRSA104901648956720446`

可以发现前面的参数是不变的，只有后面的一段数字是变化的

在登录包我们可以发现，这一段是`clienttime_ms`

![java](upload/attach/202204/870029_GY9WGEQBRXHP7A5.png)

即时间戳

剩下的`P2`、`t1`、`t2`没有hook到对应生成的地方，这里猜测可能是native层实现的加密

通过主动打印调用栈，找到登录包生成的地方

![java](upload/attach/202204/870029_C3T9DB8NY8T6BGP.png)

打开jadx反编译APK，找到对应的类对应的`a`方法：

![java](upload/attach/202204/870029_D62ANW53A9B8UDS.png)

跟踪进去：

![java](upload/attach/202204/870029_DUWRZMA7AM5H2C6.png)

可以看到我们已经定位到了登录包参数构造的地方

我们先按照登录包参数的顺序依次分析，`p2`参数来自这里

![java](upload/attach/202204/870029_8KJUCXYFKK3252A.png)

传入了一个`hashmap`，参数有两个

`clienttime_ms`跟`pwd`

跟进生成`pwd`的`a`方法中

![java](upload/attach/202204/870029_GXE2VH9QXHDWUQX.png)

可以看到传入了一个字符串，然后进行md5加密，这里就是传入的密码进行md5加密

然后我们跟进`.getEncryptText`方法中

看到`setRSAkey`方法，猜测这个过程可能跟RSA加密有关

继续跟进

![java](upload/attach/202204/870029_SSSMUYCBFBA2VV2.png)

`i`方法里面传入`j`方法生成的对象

`j`方法传入`hashmap`，遍历生成`arrayList`

![java](upload/attach/202204/870029_TR9X3YK8YASE5XP.png)

再次跟进去可以发现`_i`是native方法

![java](upload/attach/202204/870029_J4ZFNVYC5PCP5HD.png)

我们用`objection` hook一下这个方法，看看返回值是不是`p2`

![java](upload/attach/202204/870029_5E7UFUMXNBFGPMM.png)

可以发现生成的参数是一致的，至此`_i`方法为`p2`参数生成的地方

`_i`方法是非静态函数，我们找到他对应的`so`可以采用`hook RegisterNatives`找到函数注册的地方

![java](upload/attach/202204/870029_EW48JQ8DBHPKTN4.png)

![java](upload/attach/202204/870029_6BFP9RDN86XWXJT.png)

找到对应的`so`为`libj.so`，偏移地址为`0x371c5`

打开IDA进入该地址`0x371c5`

![java](upload/attach/202204/870029_8D5ZK6VUV77KN2Q.png)

可以看到最终返回值`v5`是从`v4`来的，而`v4`是从上面来的，但是上面没有看到对应参数赋值的地方。

---

## unidbg黑盒调用

构造`pair`类，因为Java中没有Android的`pair`

![unidbg](upload/attach/202204/870029_EB3ZU5DBFHGNK5P.png)

构造需要返回的`list`对象给native函数调用

![unidbg](upload/attach/202204/870029_T7H435SRFRDYNTQ.png)

### Unidbg调用：

![unidbg](upload/attach/202204/870029_PGK3MJ56GTCFVET.png)

开始补环境

![unidbg](upload/attach/202204/870029_Y7RKV8AZ26KBTW9.png)

![unidbg](upload/attach/202204/870029_SAPT345QT72Y7QW.png)

![unidbg](upload/attach/202204/870029_MGGBG9AY8J6MYKU.png) ![unidbg](upload/attach/202204/870029_2QEKQDJXD7XJ3UU.png)

继续调用

![unidbg](upload/attach/202204/870029_VAA42QCP6HRPGMB.png)

Unidbg模拟算法复现成功

---

## IDA算法分析

![IDA](upload/attach/202204/870029_9GXT52CPF4G9KPU.png)

首先是`H2`函数，进到里面查看

![IDA](upload/attach/202204/870029_QEU62HHJBEU4A6F.png)

里面估计是构造`json`对象

![IDA](upload/attach/202204/870029_2N56DT5CGKZVZ3B.png)

再进到`f10`里面

![IDA](upload/attach/202204/870029_49QJUHAMFTSKX22.png)

看到字符串`append`，估计是对`json`字符串的构造

![IDA](upload/attach/202204/870029_5E64UMQEGU49N4W.png)

进入`f1`

![IDA](upload/attach/202204/870029_K47FCDT6C4JJRAS.png)

进入`f13`

![IDA](upload/attach/202204/870029_42RWNESWQTXEG8K.png)

进入`f1`

![IDA](upload/attach/202204/870029_SU8JJU52U2ZN9JC.png)

发现已经定位到RSA加密的地方了，验证了我们的猜想。

HOOK对应的函数：

`PUBKEY`：

![IDA](upload/attach/202204/870029_2N9NCAQNKNT7FDR.png)

加密参数：

![IDA](upload/attach/202204/870029_ERA6547W88R2BDX.png)

![IDA](upload/attach/202204/870029_XGA9GKVZC4QBY59.png)

最终在`h12`中对密文做`hex`编码处理

## 总结

总结：由于篇幅问题这里只对`P2`参数做了分析，秘钥具体生成的过程也没有放出来，分析过程大同小异。

本文首发于公众号：秋田犬和短尾猫

## 抓包分析：

首先抓包看下参数情况

![抓包](https://bbs.kanxue.com/upload/attach/202204/870029_W6HABYWTGY456P8.png)

可以看到这里未知的参数有4个

`P2`、`t1`、`t2`和`key`

---

## Java层逆向：

我们先hook一下有没有java层通用的加密算法

![java](https://bbs.kanxue.com/upload/attach/202204/870029_UZVCF2RW3YDSC4R.png)

可以看到在登录调用的过程中触发了一些加密算法

我们搜索一下登录包的未知参数

看到我们有没有想要的

![java](https://bbs.kanxue.com/upload/attach/202204/870029_DECQQBGJTBAHC7V.png)

定位到了`key`参数，发现是MD5加密

参数为：

`3116LnT6xpN3khm36zse0QzvmgTZ3waWdRSA104901648962363521`

接着我们连续调用几次

`3116LnT6xpN3khm36zse0QzvmgTZ3waWdRSA104901648956720446`

可以发现前面的参数是不变的，只有后面的一段数字是变化的

在登录包我们可以发现，这一段是`clienttime_ms`

![java](https://bbs.kanxue.com/upload/attach/202204/870029_GY9WGEQBRXHP7A5.png)

即时间戳

剩下的`P2`、`t1`、`t2`没有hook到对应生成的地方，这里猜测可能是native层实现的加密

通过主动打印调用栈，找到登录包生成的地方

![java](https://bbs.kanxue.com/upload/attach/202204/870029_C3T9DB8NY8T6BGP.png)

打开jadx反编译APK，找到对应的类对应的`a`方法：

![java](https://bbs.kanxue.com/upload/attach/202204/870029_D62ANW53A9B8UDS.png)

跟踪进去：

![java](https://bbs.kanxue.com/upload/attach/202204/870029_DUWRZMA7AM5H2C6.png)

可以看到我们已经定位到了登录包参数构造的地方

我们先按照登录包参数的顺序依次分析，`p2`参数来自这里

![java](https://bbs.kanxue.com/upload/attach/202204/870029_8KJUCXYFKK3252A.png)

传入了一个`hashmap`，参数有两个

`clienttime_ms`跟`pwd`

跟进生成`pwd`的`a`方法中

![java](https://bbs.kanxue.com/upload/attach/202204/870029_GXE2VH9QXHDWUQX.png)

可以看到传入了一个字符串，然后进行md5加密，这里就是传入的密码进行md5加密

然后我们跟进`.getEncryptText`方法中

看到`setRSAkey`方法，猜测这个过程可能跟RSA加密有关

继续跟进

![java](https://bbs.kanxue.com/upload/attach/202204/870029_SSSMUYCBFBA2VV2.png)

`i`方法里面传入`j`方法生成的对象

`j`方法传入`hashmap`，遍历生成`arrayList`

![java](https://bbs.kanxue.com/upload/attach/202204/870029_TR9X3YK8YASE5XP.png)

再次跟进去可以发现`_i`是native方法

![java](https://bbs.kanxue.com/upload/attach/202204/870029_J4ZFNVYC5PCP5HD.png)

我们用`objection` hook一下这个方法，看看返回值是不是`p2`

![java](https://bbs.kanxue.com/upload/attach/202204/870029_5E7UFUMXNBFGPMM.png)

可以发现生成的参数是一致的，至此`_i`方法为`p2`参数生成的地方

`_i`方法是非静态函数，我们找到他对应的`so`可以采用`hook RegisterNatives`找到函数注册的地方

![java](https://bbs.kanxue.com/upload/attach/202204/870029_EW48JQ8DBHPKTN4.png)

![java](https://bbs.kanxue.com/upload/attach/202204/870029_6BFP9RDN86XWXJT.png)

找到对应的`so`为`libj.so`，偏移地址为`0x371c5`

打开IDA进入该地址`0x371c5`

![java](https://bbs.kanxue.com/upload/attach/202204/870029_8D5ZK6VUV77KN2Q.png)

可以看到最终返回值`v5`是从`v4`来的，而`v4`是从上面来的，但是上面没有看到对应参数赋值的地方。

---

## unidbg黑盒调用

构造`pair`类，因为Java中没有Android的`pair`

![unidbg](https://bbs.kanxue.com/upload/attach/202204/870029_EB3ZU5DBFHGNK5P.png)

构造需要返回的`list`对象给native函数调用

![unidbg](https://bbs.kanxue.com/upload/attach/202204/870029_T7H435SRFRDYNTQ.png)

### Unidbg调用：

![unidbg](https://bbs.kanxue.com/upload/attach/202204/870029_PGK3MJ56GTCFVET.png)

开始补环境

![unidbg](https://bbs.kanxue.com/upload/attach/202204/870029_Y7RKV8AZ26KBTW9.png)

![unidbg](https://bbs.kanxue.com/upload/attach/202204/870029_SAPT345QT72Y7QW.png)

![unidbg](https://bbs.kanxue.com/upload/attach/202204/870029_MGGBG9AY8J6MYKU.png) ![unidbg](https://bbs.kanxue.com/upload/attach/202204/870029_2QEKQDJXD7XJ3UU.png)

继续调用

![unidbg](https://bbs.kanxue.com/upload/attach/202204/870029_VAA42QCP6HRPGMB.png)

Unidbg模拟算法复现成功

---

## IDA算法分析

![IDA](https://bbs.kanxue.com/upload/attach/202204/870029_9GXT52CPF4G9KPU.png)

首先是`H2`函数，进到里面查看

![IDA](https://bbs.kanxue.com/upload/attach/202204/870029_QEU62HHJBEU4A6F.png)

里面估计是构造`json`对象

![IDA](https://bbs.kanxue.com/upload/attach/202204/870029_2N56DT5CGKZVZ3B.png)

再进到`f10`里面

![IDA](https://bbs.kanxue.com/upload/attach/202204/870029_49QJUHAMFTSKX22.png)

看到字符串`append`，估计是对`json`字符串的构造

![IDA](https://bbs.kanxue.com/upload/attach/202204/870029_5E64UMQEGU49N4W.png)

进入`f1`

![IDA](https://bbs.kanxue.com/upload/attach/202204/870029_K47FCDT6C4JJRAS.png)

进入`f13`

![IDA](https://bbs.kanxue.com/upload/attach/202204/870029_42RWNESWQTXEG8K.png)

进入`f1`

![IDA](https://bbs.kanxue.com/upload/attach/202204/870029_SU8JJU52U2ZN9JC.png)

发现已经定位到RSA加密的地方了，验证了我们的猜想。

HOOK对应的函数：

`PUBKEY`：

![IDA](https://bbs.kanxue.com/upload/attach/202204/870029_2N9NCAQNKNT7FDR.png)

加密参数：

![IDA](https://bbs.kanxue.com/upload/attach/202204/870029_ERA6547W88R2BDX.png)

![IDA](https://bbs.kanxue.com/upload/attach/202204/870029_XGA9GKVZC4QBY59.png)

最终在`h12`中对密文做`hex`编码处理

## 总结

总结：由于篇幅问题这里只对`P2`参数做了分析，秘钥具体生成的过程也没有放出来，分析过程大同小异。

本文首发于公众号：秋田犬和短尾猫

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2022-4-6 10:05

被\_Miracles\_编辑

，原因：

[#NDK分析](https://bbs.kanxue.com/forum-161-1-119.htm)

[#源码框架](https://bbs.kanxue.com/forum-161-1-127.htm)
