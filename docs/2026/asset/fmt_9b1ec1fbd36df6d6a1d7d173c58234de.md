
---

然后来看看这个加密的字段`access_token`：

![图片描述](upload/attach/202208/939330_TNMCB6W59K6PN7P.png)

然后就要分析这个东西了：跳到`C5414a.f16229h`看看：

![图片描述](upload/attach/202208/939330_US3N7FVCVUGXYXF.png)

接着跟进去分析，终于找到了定义他的地方了：

```
static {
    String str = f16227f + Build.PRODUCT + MyApplication.getNetworkType() + MyApplication.getDeviceId();
    crack.log(str);
    f16229h = C6448r.m23455a(str);
}
```

这个字段`f16227f`是获取`versionname`：

![图片描述](upload/attach/202208/939330_MHSG9RD8ZBDTDWP.png)

后面的几个是`deviceid`，还有网络信息：

![图片描述](upload/attach/202208/939330_TQC9N8Q9Z4YUC53.png)

最后再看看这个函数`C6448r.m23455a()`干了什么：

![图片描述](upload/attach/202208/939330_R3ZH5ZRRJVV63H9.png)

就是个`md5`加密就完了。

---

最后重点看这个`codesign`字段：

![图片描述](upload/attach/202208/939330_26HPWGAMU6VAF8S.png)

第一个参数`relaceall`是随机生成的数字，然后进行了字符串的替换。

第二个参数`a`是之前传入的：

![图片描述](upload/attach/202208/939330_8SYGKTFKCKCZWWF.png)

`jsonobject`中的`data`中的数据，就是明文的用户名和密码。

第三个参数是当前的时间。

然后就要跟进分析这个函数了：

![图片描述](upload/attach/202208/939330_TSJW27UJGH6B6UN.png)

这个函数的三个参数都分析完了，只有这个参数我们还不知道是什么：`m23483a()`：

![图片描述](upload/attach/202208/939330_GHSKQU56J8SQBWK.png)

```
public static final int forum_key = 2131230990;
```

然后就要看这个函数了：`C6420af.m23337b()`。

![图片描述](upload/attach/202208/939330_6WSFU6NN9TXBVK6.png)

这个是获取资源文件的函数。

所以要想得到这个字段的值，就要从资源文件中看看了：

```
<string name="forum_key">94ac5cfb69e87bd7</string>
```

这样所有字段的值都得到了，最后直接看看这个函数就好了：`C6448r.m23455a()`还是刚刚那个`md5`加密的函数：

![图片描述](upload/attach/202208/939330_6WBWBF45RV2QRCY.png)

这样这款`app`就分析完了。

分析完上面两款`app`，大家也可以发现，这些工具没有谁最好，只是都得使用，各有各的好处。上次说那个`JEB yyds`就打脸了，哈哈，这个`jadx`也挺好的，一些代码直接翻译成人能看懂的了。

---

## 继续继续，还有第三个 app

这款`app`我用的真机，模拟器出了点问题，先登录：

![图片描述](upload/attach/202208/939330_JWXPEPZXJUSU55W.png)

然后抓包。

登录包中只有这两个数据：

![图片描述](upload/attach/202208/939330_KX253T5CCJ6TFCZ.png)

这里如果只在`key`上面下断点是断不下来程序的（分析到后面就知道了，这个传输的字段叫`key_id`），所以这里我打算从`post`字段上下手。

```
POST http://passport.xxxx.com.cn/login_jsonp_active.do HTTP/1.1
Content-Type: application/x-www-form-urlencoded; charset=utf-8
Content-Length: 302
Host: passport.xxxx.com.cn
Connection: Keep-Alive
Accept-Encoding: gzip
Cookie: JSESSIONID=aaapcYHe7KtGb7NDScYhy
User-Agent: okhttp/3.1.2
key=dTdlMmtGQjZIQk9CNmdudi95QURUbUNrZ2xKWFRNc0t0Z3g4NnpKRkZDYjRGc25RU05CL0wzSjQ2ZFYrMmxqd1ZaU2JtTVJvaURudWJhVnpFZGRsRmZGQldGQzBxbE0xVFVNVER5TDRpNkFpc1E4eVJVK0VnZWxBUUdaR0lvZ0Y5NTdFKzJKRVFtNlR0SDN5SGtCY1FOZnFIYnpNdmZqa3FPVnc2SkNGUzJ1SWUrb2xBby9wbGUrSUh1bU1wK2pTbW1XYkhzajNsYmM9P2tleUlkPTE
```

这里我搜索的是`login_jsonp_active`这个字段，然后定位到这里：

![图片描述](upload/attach/202208/939330_B2CJJG4EN5R3Q8R.png)

然后看那个`API_LOGIN`的交叉引用，定位到了这里：

![图片描述](upload/attach/202208/939330_FEE5VF6E9RKH2A7.png)

然后向下找找，看看有没有突破点：

这个函数中有解密字段，还有用户名和结果，看着比较像：

![图片描述](upload/attach/202208/939330_A8ESRCMBC4FMZR8.png)

然后继续看交叉引用，这里有解密的标志，所以这里看着比较像：

![图片描述](upload/attach/202208/939330_8XW3ZPURVMWZJ3G.png)

最终定位到了这里，发现了是`key_id`字段，不是`key`字段……难怪下这么多断点都找不到他：

![图片描述](upload/attach/202208/939330_KTKCZMK3E4JUAEV.png)

然后还发现了这个：发现他是`des`加密：

```
sb.append(DESedeCoder.encode(json, KEYS.get("1"))).append("?keyId=").append("1");
```

然后就要分析这个函数是怎么加密`key_id`字段了：

```
public String encrypt(Map params) {
```

这个传入的参数中的`map`数据，在`fiddler`中的数据包中可以发现：

![图片描述](upload/attach/202208/939330_24GDUHQ2XDQ4NNU.png)

然后通过搜索`sign`字段，也可以定位到在`app`中的位置：

```
public static Map<String, String> m264a(Map<String, String> map) {
    map.put("t", String.valueOf(System.currentTimeMillis()));
    StringBuilder sb = new StringBuilder(128);
    sb.append(m265b(map.get("appkey"))).append("&").append(m265b(map.get("domain"))).append("&").append(m265b(map.get("appName"))).append("&").append(m265b(map.get(SdkConstants.APP_VERSION))).append("&").append(m265b(map.get("bssid"))).append("&").append(m265b(map.get("channel"))).append("&").append(m265b(map.get(LeService.KEY_DEVICE_ID))).append("&").append(m265b(map.get("lat"))).append("&").append(m265b(map.get("lng"))).append("&").append(m265b(map.get("machine"))).append("&").append(m265b(map.get("netType"))).append("&").append(m265b(map.get("lng"))).append("&").append(m265b(map.get("platform"))).append("&").append(m265b(map.get("platformVersion"))).append("&").append(m265b(map.get("preIp"))).append("&").append(m265b(map.get("sid"))).append("&").append(m265b(map.get("t"))).append("&").append(m265b(map.get("v")));
    map.put("sign", m263a(sb.toString()));
    return map;
}
```

分析完参数是啥，就要看看函数的具体操作了：

![图片描述](upload/attach/202208/939330_42YCVPEQYUJBR2B.png)

先声明一个`stringbuffer`变量。

再将`map`中的数据转化为`string`类型。

然后就是对`key`字段进行的加密了：那个`KEYS.get("1")`在这里可以找到值，应该是密钥。

![图片描述](upload/attach/202208/939330_6CBQ49F8GJVGRWV.png)

然后就要分析这个函数了：`DESedeCoder.encode(json, KEYS.get("1"))`。

第一个参数是`map`中的数据，第二个参数是密钥。

![图片描述](upload/attach/202208/939330_CARRQT9QYNUZDYC.png)

先将传入的字符串转化为`byte`类型，然后传入`encrypt`函数，继续跟进分析：

![图片描述](upload/attach/202208/939330_YYSU2AK5A76SCPQ.png)

好了，这款`app`的加密字段也分析完了。

---

## 总结一下吧

这个协议分析和找 flag 基本上是一样的，就是定位到关键字段和函数，然后分析加密过程。这次就先分享三个 Java 层的吧，下次再分析`so`的。最近几天也是比较迷茫，一直在找课，再往深了学，能白嫖的视频越来越少了。这几天找资料也浪费了不少时间，好在找到了一个适合自己的，又是一个新的提升阶段，哈哈。看了看一共 100 多 h 的视频，之后一定做好记录分享给大家。其实看视频是一方面，更重要的是要实践，才能发现其中的问题与奥秘。

`app`就不透露了（之前看各种大佬的分析也是怕侵权），什么方面的都有（也不知道是什么时候的`app`了），大家看看思路就好啦，哈哈。

## 先看第一个 app

找到登录界面，我输入的手机号是 13905376666，密码是：666666666。

先抓包：

![图片描述](https://bbs.kanxue.com/upload/attach/202208/939330_RNDVFCNHG5MU767.png)

这个是请求包：

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 | `http://m.xxxx.com.cn/v2/member?modules=cloudlogin%3A1&password=666666666&siteid=10001&sign=d01f5e4445af0a30d148d4dc451b41cf&clientid=1&system_name=android&type=android&time=1659366738640&ip=10.0.2.15&device_id=2a%3Aa5%3A33%3A2d%3Ade%3Ac9&account=13905376666` `HTTP/1.1` |

第一步，也是最重要的一步，得先定位到关键代码的位置。

（这里插一句，分析协议字段的时候一般都是最后分析`sign`字段，这个`sign`字段经过我这几天的学习总结出来的一般是这样生成的：

先将其他字段加密之后，然后有的还可能带一些盐，最后将各个字段排序之后，然后加密，`MD5`一般是。）

回到分析过程中，这里发现只有`sign`字段看着像加密的，其他的基本都是明文。然后找到两个方法，分别下断点，然后看看它走哪个就完事了。

这里我打算搜索`post`请求中的`v2/member`和`"system_name"`。

最终定位到这两个方法中比较像：

![图片描述](https://bbs.kanxue.com/upload/attach/202208/939330_99B4B7WMUZ5DKRA.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202208/939330_6WCXHS93UH786GJ.png)

然后动态调试的时候我怕断不下来，就遇见`sign`都下了断点（这也是一种方法吧，哈哈，但是它会在一些奇奇怪怪的方法中断下来，但是只要不是你想要的断点处，直接让程序继续运行就好了），然后发现我的判断没有错：（当点击登录按钮的时候，程序从这里断了下来。）

![图片描述](https://bbs.kanxue.com/upload/attach/202208/939330_5A2JWHQJ4BQNYSJ.png)

这个协议分析我也是刚刚开始自己动手，然后就打算把步骤写得详细一点：

首先看这个函数的两个参数：

第一个参数表示一个集合，在这个集合中保存发往服务器的数据包中的各个参数。然后第二个参数的翻译是上下文，所以这个参数应该就是连接上下函数的一个变量。

![图片描述](https://bbs.kanxue.com/upload/attach/202208/939330_96PANEQGSZXCH4B.png)

然后看这个`device_id`就是表示手机的串号：

![图片描述](https://bbs.kanxue.com/upload/attach/202208/939330_VCFB6W8BH5DD57G.png)

是这样声明的：

![图片描述](https://bbs.kanxue.com/upload/attach/202208/939330_WHVMX3C9T689256.png)

就是返回手机的型号，没有什么好说的。

然后解释完第一个，其他的就不用过多解释啦：

![图片描述](https://bbs.kanxue.com/upload/attach/202208/939330_JSZ3PQFEXH5HKJ8.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202208/939330_ZMFPAYS4J9QBQSJ.png)

这个`clientid`就是定值 1：`ClientID`（客户端标识号）用于标识连接到 API 服务的客户端。

然后下一个就是本机 IP 地址，还有时间`timestamp`，`siteid`站点标识符定值 10001，系统名称`system_name`，型号`type`是 Android 的。

然后重点就是这个`sign`了，这个是个签名校验。这个校验对于我们来说，就是我们自己分析算出来的`sign`值和发往服务器的`sign`值一样，就说明分析正确了。一般的校验都是看这个发往服务器的`sign`值和服务器那边计算出来的`sign`值是否一样：

![图片描述](https://bbs.kanxue.com/upload/attach/202208/939330_NU8K9KNR6AXEAJ2.png)

所以接下来就是要分析这个函数了：`m8098a(params.getURLHashMap(), timestamp + "")`。

它是这样声明的：

![图片描述](https://bbs.kanxue.com/upload/attach/202208/939330_GWK9F37JKCJ3WMF.png)

然后就要分析这个关键函数了：

|  |  |
| --- | --- |
