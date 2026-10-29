
# [原创]xx医美登陆协议分析

在开始正文之前, 说一些题外话。

1. 这是本人在论坛注册后的第三次发帖, 如果有格式错误, 发布的版块错误, 主题不合规侵犯什么权利之类的, 请各位明示指出。管理员也可以删帖。
2. 因为本人水平有限, 某些地方写的可能是错的, 也请各位大佬指正, 不胜感激。
3. 当前文章是xx医美8月的版本，8月份当月分析了大部分字段，写文章拖到了9月份，写完之后一直不满意，因为还有一些字段没有分析清楚就草草了结，而且整个思路也不是特别清楚，像流水账一样。现在在翻之前笔记的时候翻到了这个，想着不发出来比较可惜，发出来后大佬们指出来我一些错误我也能更好的进步，所以发出来跟大佬们交流下。
4. 文章于9月首发于易锦，作者为本人，当前版本是发布后两天内修改，修改内容不多。

以下正文：

## 写文章初衷

1. 最近在学习协议分析这块，很多时候一些apk只是简单看一下，大概看清楚流程就不想继续详细去分析了。没有很详细的去分析一个apk，想借这次写文章，梳理一下整个流程。
2. 答应了一个大佬要写一篇文章发出来。
3. 因为水平比较菜且时间不足，这次就选了一个简单的apk，虽然还有几个同样简单的apk在备选之列，但是我每天出入工地的时候，电梯广告总是在播放xx医美的广告，做女人整好 这个广告我每天都会看到几遍，那就撸它把。

![](https://bbs.kanxue.com/upload/attach/201910/807993_3CUQWGNDG2JE3BN.jpg)

这里选择了分析用账号密码登陆的登陆协议

![](https://bbs.kanxue.com/upload/attach/201910/807993_3ZC764P7HMZ7ADX.jpg)

测试账号：13355556666

测试密码：kkk111222

burpsuite 抓登陆包：

![](https://bbs.kanxue.com/upload/attach/201910/807993_NU5NRM9GGSNJNWQ.jpg)

![](https://bbs.kanxue.com/upload/attach/201910/807993_DSG7Q44CPRSTPMH.jpg)

格式下各个字段 看着清楚一些

```
POST /v8/passport/apploginnew HTTP/1.1

Cookie: __usersign__=1568523653313812392; expires=Tue, 15-Oct-2019 15:59:59 GMT; Max-Age=2631546; path=/; domain=.soyoung.com

Content-Type: application/x-www-form-urlencoded

Content-Length: 458

Host: api.soyoung.com

Connection: close

Accept-Encoding: gzip, deflate

User-Agent: okhttp/3.10.0



xy_device_token=1402b247ff996e94f16ce53a27377309e2

&password=kkk111222

&request_id=00e09fba6857bf87ed88030d6fd6680f

&uuid=ffffffff-9dbd-915d-ffff-ffffde5a499b

&app_id=2

&lver=7.32.2

&login_name=13355556666

&_sign=%242a%2405%24xWMKvGsGdANgOytr88Tqf.mPsuq6BAFUnKr2ITTyWJriXaidIyEO6

&device_id=175988620

&sys=2

&key=2f77dcb9f66cacdc4cb1f592974816f2

&uid=0

&sm_device_id=20190813220351b85597663f19089a4a4550cbc15f62f4014b4f187300ca7f

&pinyin=soyoung

&_time=1568523793

&cityId=0
```

当我看到密码是明文的时候 真的是一口血吐了出来  很久没看到这么坦诚相见的apk了

当前app字符串并没有加密 因为要分析密码登录的协议  所以 可以搜索类

`login_name` `password` 这样的字段

尝试搜索后发现 `password` 在源码中引用太多 不太好定位分析 所以选择 `login_name` 字段定位

![](https://bbs.kanxue.com/upload/attach/201910/807993_QJQW4HPB2FSUEE4.jpg)

当前app搜索 `login_name` 可以 直接到到关键位置的  如下图

![](https://bbs.kanxue.com/upload/attach/201910/807993_944YRS6NJ7G47CG.jpg)

但是谨慎起见 为了防止流程跟踪出现错误

我一般会结合 `monitor` 的方法跟踪 相互印证  打开 `monitor` 按下登陆键

![](https://bbs.kanxue.com/upload/attach/201910/807993_296FNSJWQ2HUJ9U.jpg)

![](https://bbs.kanxue.com/upload/attach/201910/807993_EEEZVT5KGHPH7SV.jpg)

一般此类登陆协议 整理方法调用的时候 我一般会搜索 `onclick()` 这个按键的处理的方法 因为是按了登陆键才触发登陆流程的

`onclick()` 一路跟随 整理出相关调用层级

```
onclick()

void onViewClick(View)

void a(PasswordLoginFragment)

void userlogin()

void getContent(String, String)      

void pwdLoginRequest(String, String)

void passwordLoginRequest(String, String)

Observable passwordLoginRequest(String, String)

Observable post(String, HashMap<String, String>)    

TreeMap getCommonParasm(HashMap<String, String>) 

TreeMap getCommonParasm(HashMap<String, String> , String)
```

整理调用层级的时候 可以用 `jadx-gui` 看一下这些方法的源码  方便后续分析

比较有亮点的是以下两个方法

1. `void passwordLoginRequest(String, String)`

![](https://bbs.kanxue.com/upload/attach/201910/807993_4266UDKE6QSESUS.jpg)

2. `TreeMap getCommonParasm(HashMap<String, String> , String)`

![](https://bbs.kanxue.com/upload/attach/201910/807993_ZZD2YKC7BWEP5XM.jpg)

上面方法中 看到了登陆数据包的大量字段  所以决定重点分析两个方法

这里准备动态调试结合静态分析  所以先反编译apk

![](https://bbs.kanxue.com/upload/attach/201910/807993_QVJWGPU6YJ8RHKE.jpg)

```
am start -D com.youxiang.soyoungapp/.ui.SplashActivity
```

调试启动app

![](https://bbs.kanxue.com/upload/attach/201910/807993_S7SEYS7WFHXPFJ7.jpg)

转发端口

```
adb forward tcp:8700 jdwp:20951
```

动态调试观察第一个 比较有料的函数  `void passwordLoginRequest(String, String)`

![](https://bbs.kanxue.com/upload/attach/201910/807993_J4VK2GAF323ADVQ.jpg)

![](https://bbs.kanxue.com/upload/attach/201910/807993_PDCQ63P8B57V625.jpg)

1. `void passwordLoginRequest(String, String)`

调试得知 两个参数是明文的 手机号和密码  根据调试结果 修改变量名 增加可读性

```
   public Observable passwordLoginRequest(String str_phone, String str_pass) {

        HashMap map_login_data = new HashMap();

        String str_key = FlagSpUtils.getKey(str_phone, str_pass);

        map_login_data.put("login_name", str_phone);  

        map_login_data.put("password", str_pass);

        map_login_data.put("key", str_key);

        map_login_data.put("sm_device_id", SmAntiFraud.getDeviceId());

        return this.post(LoginUrl.PWD_LOGIN, map_login_data);

    }
```

上面的函数里有4个字段

```
login_name   		明文

password     		明文

key          		getkey返回值

sm_device_id 		getDeviceId()
```

`getkey()` 方法

![](https://bbs.kanxue.com/upload/attach/201910/807993_ERBU57SW64DYYR3.jpg)

![](https://bbs.kanxue.com/upload/attach/201910/807993_VFF326Y9YZDNERB.jpg)

![](https://bbs.kanxue.com/upload/attach/201910/807993_ZDBEVTDYA7KSXBE.jpg)

![](https://bbs.kanxue.com/upload/attach/201910/807993_HYNXS3DRYFGVDYA.jpg)

```
login_name   明文

password     明文

key          md5(lavion_soyoung@2013_ + "手机号" + "_" + "密码")

sm_device_id getDeviceId()
```

`getDeviceId()`

![](https://bbs.kanxue.com/upload/attach/201910/807993_JM3DES9MEKZDRHK.jpg)

![](https://bbs.kanxue.com/upload/attach/201910/807993_FHCQZH8JW85HDD8.jpg)

搜索公司名字后 找到了 sdk的官网

![](https://bbs.kanxue.com/upload/attach/201910/807993_9K3UVAHBQXDSR5A.jpg)

看起来就是这个sdk产品 生产了唯一标识 可能防范黑产表哥把

![](https://bbs.kanxue.com/upload/attach/201910/807993_VHR4X9BTVQK8MVE.jpg)

一路跟踪 找到产生 `id` 的函数

![](https://bbs.kanxue.com/upload/attach/201910/807993_7R7H3QPSS4V9KWK.jpg)

![](https://bbs.kanxue.com/upload/attach/201910/807993_QP5ZJAAWEAR85TN.jpg)

当前sdk混淆做的比较彻底 变量方基本都是 `000OOooo` 之类的命名

对当前方法简单分析后 修改变量名 如下图

![](https://bbs.kanxue.com/upload/attach/201910/807993_Z93UCWYAVZG2P38.jpg)

![](https://bbs.kanxue.com/upload/attach/201910/807993_KWZ5PC8F4MMDJHC.jpg)

这里不继续分析  因为继续分析下去篇幅太大 有兴趣的小伙伴可以下载sdk 看看源码 或者自己直接逆向

```
login_name   明文

password     明文

key          md5(lavion_soyoung@2013_ + "手机号" + "_" + "密码")

sm_device_id getDeviceId() 第三方sdk返回值
```

第一个有字段的函数分析完毕  下面开始第二个

---

2. `getCommonParasm(HashMap<String, String> hashMap, String str)` 函数

大部分字段在这里完成拼接 下面是 `jadx-gui` 反编译的源码
