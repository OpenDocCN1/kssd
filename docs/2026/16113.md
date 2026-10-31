

# [原创] 某猫直播登录算法分析

做个 App 的协议分析，仅供学习研究。初次发帖，水平有限。

## 目标 App

某猫直播最新版本（`1.1.6`）。

## 分析目的

分析登录算法。

## App 信息

![](https://bbs.kanxue.com/upload/attach/202003/698087_8SNCCA7END3ZV7Y.png)

## 抓包分析

通过 `HttpCanary` 抓登录包，得到登录 URL：

`http://XXX.XXX.XX.XX:XXXX/api/users/login`

登录请求数据如下（登录邮箱我填的是 `123@qq.com`，密码 `123456`）：

```text
_app_version=1.1.6
&_device_type=M3s
&_device_version=5.1
&_sdk_version=22
&data=920A95B81FD74AC1590F909BABF482191DC7DCE2DF3D842673E30ABCB9F37CC752FDEAA30F2EA3895AC9B816915AAFD6
&sig=457dd421bd83a3a8c6d70c48f1678109
```

登录响应数据如下：

```text
B08020D0FACFDAF81DB46890E4040EDBB8613DA5ABF038F8B86BD44525D2E27B26E22ACD06388112D8467FD688C79CC7EA83F27440577350E8168C2560368616
```

其实提示的是密码错误，这是个密文，后面会说到。

主要分析 `data` 和 `sig` 的算法就可以了。`sig` 一看就是 `MD5` 加密的形式。

## 反编译分析

把 APK 拖入 `AndroidKiller`，可惜 `AndroidKiller` 没能反编译出 Java 源码，一直提示“反编译源码中”，具体原因还不知道（有知道的可以告诉我一下吗）：

![](https://bbs.kanxue.com/upload/attach/202003/698087_WR4ST9RC9ME8THY.png)

算了，直接用 `jadx` 打开，搜索“`_app_version`”。由于是 POST 请求，找到 `PostFormBuilder` 这个类，找到了如下代码：

![](https://bbs.kanxue.com/upload/attach/202003/698087_VRJXZA47JJC2KX9.png)

通过 `EncryptUtils.encryptMD5ToString(v3.toString().substring(1) + "XXX")` 知道了 `sig` 就是把上面的那几个参数拼起来，再加上一个字符串，然后做 MD5 计算。

但 `sig` 的结果肯定和 `data` 的值有关联。从 `getParams()` 这个方法看，`data` 并没有 `add` 到 `this.params` 这个 `map` 中，`data` 的值是怎么计算出来的呢？

然后我搜索了下“`data`”，发现这个“`data`”是在调用 `getParams` 方法之前，通过 `addParams` 方法动态 `add` 上去的：

![](https://bbs.kanxue.com/upload/attach/202003/698087_7S6ZGPNMYSZM2QF.png)

`PostFormBuilder` 类的 `getParams` 方法的上面就有个 `addParams` 方法。这个方法有个 AES 加密计算：

`AesEncryptionUtil.encrypt(val, Api.AES_PWD, Api.AES_IV)`

![](https://bbs.kanxue.com/upload/attach/202003/698087_HRN8VMAKUQ64YKT.png)

搜索“`addParams(`”，来到 `LoginViewActivity` 这个登录类的 `doLogin` 方法，看到了关键的信息：

`((PostFormBuilder) OkHttpUtils.post().addParams("data", jsonObject.toString()).getParams().url(path)).build().execute(new StringCallback() {…})`

如图：

![](https://bbs.kanxue.com/upload/attach/202003/698087_Q4DD4K63JU4XAG6.png)

原来 `data` 是在这里添加进 `PostFormBuilder` 类的 `addParams` 方法的。`PostFormBuilder` 类的 `addParams` 方法里，对用户名和密码组成的 JSON 字符串进行了 `AES/CBC/PKCS5Padding` 加密。

到这登录算法逻辑大体就分析完了，下面就是写 Java 代码实现了：

![](https://bbs.kanxue.com/upload/attach/202004/698087_4VQGASYMNDAXE53.png)

![](https://bbs.kanxue.com/upload/attach/202004/698087_MRMR95DWKF6BAP5.png)

## 模拟登录

最后用 `Postman` 模拟登录：

![](https://bbs.kanxue.com/upload/attach/202003/698087_JDQ3YMEMM9Q857J.png)

`B08020D0FACFDAF81DB46890E4040EDBB8613DA5ABF038F8B86BD44525D2E27B26E22ACD06388112D8467FD688C79CC7EA83F27440577350E8168C2560368616` 这串字符其实就是“密码错误”用 AES 加密的密文，也就是文章开头留下的疑问。

哈哈，这个 App 的数据协议就分析完了。本文只针对登录部分，但视频加载的算法也是走的这套协议。由于都是 Java 层的协议分析，比较简单。第一次发技术帖，大神勿喷。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2020-4-17 14:11 被树上的猫编辑，原因：完善隐私信息保护

