
```java
public String a(String str, int i, String str2, String str3) {
    StringBuilder sb = new StringBuilder(str);
    StringBuilder sb2 = new StringBuilder();
    sb.append("?");
    sb.append("appid=");
    sb.append(this.b);
    sb2.append("appid=");
    sb2.append(this.b);
    sb.append("&clienttime=");
    sb.append(i);
    sb2.append("clienttime=");
    sb2.append(i);
    int version = getVersion();
    sb.append("&clientver=");
    sb.append(version);
    sb2.append("clientver=");
    sb2.append(version);
    sb.append("&dfid=");
    sb.append(str2);
    sb2.append("dfid=");
    sb2.append(str2);
    String fxDeviceId = getFxDeviceId();
    sb.append("&mid=");
    sb.append(fxDeviceId);
    sb2.append("mid=");
    sb2.append(fxDeviceId);
    String androidId = getAndroidId();
    sb.append("&uuid=");
    sb.append(androidId);
    sb2.append("uuid=");
    sb2.append(androidId);
    String a2 = com.xxx.fanxing.allinone.common.utils.ao.a(this.e + sb2.toString() + str3 + this.e);
    sb.append("&signature=");
    sb.append(a2);
    return sb.toString();
}
```

> (agent) [013697] Arguments `com.xxx.fanxing.allinone.watch.common.protocol.user.a.a(<mark class="encrypted">2a5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9L8$3N6A6L8Y4y4W2M7Y4k6A6j5$3g2Q4x3X3g2C8N6h3N6G2N6g2)9J5k6h3y4G2L8g2)9J5c8Y4j5&6i4K6u0r3L8r3!0Y4K9h3&6Q4y4h3k6T1P5g2)9#2k6Y4m8%4k6q4)9J5b7H3`.`</mark>`

`1650353888,`

`1tRbIu1gsMjC2Htvln120WPp,`

`{"params":"2CA44E5F5D8F189BC68662C0438844D7043484C37D09E7AD1A8A138D44CDCBB0465684EE3C6FD64263174944AD6F2027D0782D9BF64A904432F9FD764CF1C96FF5C90B16DB2932E15FDC56178940207F","clienttime_ms":"1650353888654","support_face_verify":"1","dfid":"1tRbIu1gsMjC2Htvln120WPp","dev":"Pixel%204","plat":"1","pk":"5F03A05AFD4E10B7283A9F2634153FEFA0AF318DD781687BB907DA770614C021921AEB30AB7A808714C132ECA24285E43E6EA2134A91BB79893B10B4024040E32C8EA448463251173D9E412A5B640F240AA601690DDF9A180599F73AA5CF45F8570F6562ABEFA31196CBB0BB31BB6E781974C56A4640CB70D25955D04F3CC7EA","t1":"cdb7d88e278cab7c596af28cfcd873ba","support_verify":"1","support_multi":"1","t2":"c397de1fd7e436639fa90f367c0b47d290831b64f19d803fc09a91ee47bcc1648e5a87f1f934ed09cfb7046dad52253cf564684d5bdef1d67fc91c74d6245335d459ddad1015bb5e96678cc0185b5ea20cc2563d5fc8f040aa77b49c039e40c5e2ca000392eeebfce750e3612ff280f6","key":"ffa858faa437189eb024cdeeb37e372d","username":"130*****669"}`

> `com.xxx.fanxing.allinone.common.utils.ao.a(`

> `4lu0l3cujt2KWIjcM374F8oX5N2lGY59`

> `appid=1131clienttime=1650356391clientver=55400dfid=1tRbIu1gsMjC2Htvln120WPpmid=5c539aee628111af4fc4645c761885bfuuid=5c539aee628111af4fc4645c761885bf`

> `{"params":"3FF4D2AB2A549FDBD95EBA40F155635B1E9A5EB353EF147EC459EC47FE18EDE2ED465FAE1E974ABB8605499F1434228E6A30779E0E07DB12A79775A6073EE748709951692C97506CD6384513250870B3","clienttime_ms":"1650356391589","support_face_verify":"1","dfid":"1tRbIu1gsMjC2Htvln120WPp","dev":"Pixel%204","plat":"1","pk":"62A6F1633AD5B9F85007AD597B90157C854A796FEA8D965E110C9D8FB152170E18CF6868F4497E1FAA62F13B257758DF803B2E4ABCF28980207534BE8B8728DFA50A4F00AD2626307B31C850E8ACED450393643C5CE8DD679E1F77186D3CED20DB72E8003C4EF2048EC1942035159EDB2590095D09B6E7A706A72F5E96408212","t1":"512640caae5a66bb98402eda86b5b026","support_verify":"1","support_multi":"1","t2":"c397de1fd7e436639fa90f367c0b47d290831b64f19d803fc09a91ee47bcc1648e5a87f1f934ed09cfb7046dad52253cf564684d5bdef1d67fc91c74d6245335d459ddad1015bb5e96678cc0185b5ea2a4865a6be23bed4a31f7625c6927379348d014b77b354e00181c2e0b1530397a","key":"7e8cfb92bb4be58f9a62e0af4c040783","username":"130*****669"}`

> `4lu0l3cujt2KWIjcM374F8oX5N2lGY59)`

`signature=MD5(appkey+appid=1131clienttime=1650356391clientver=55400dfid=1tRbIu1gsMjC2Htvln120WPpmid=5c539aee628111af4fc4645c761885bfuuid=5c539aee628111af4fc4645c761885bf+ post data + appkey)`

# 还原

[算法还原Python版](elink@7bdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6^5K9h3q4G2K9$3q4F1k6$3S2#2j5W2)9J5c8X3E0#2k6$3!0#2)

# 前言

**本篇文章仅为学习交流所用，涉及的数据已做脱敏处理，请勿用于不当途径，侵权请联系**

# 环境

1. 主机：win10
2. 手机：Pixel 4，Android 10
3. APP版本：V5.54.0

# 工具

IDA、JADX、Frida、Charles

# 登录逆向分析

## RegisterNatives

![](https://bbs.kanxue.com/upload/attach/202204/951706_24EFN6CG6E3258J.png)

![](https://bbs.kanxue.com/upload/attach/202204/951706_TUWT7UFKZJ8UJ79.png)

## Request Params

首先抓包看一下登录包：

![](https://bbs.kanxue.com/upload/attach/202204/951706_4PEGFTGXE3UQY26.png)

![](https://bbs.kanxue.com/upload/attach/202204/951706_KPKSKB2KCDSMJPF.png)

### verifydata

> ```json
> {
>   "params": "5F1D4B282F5E89548F98FC30853F4F9289D874AA82EDB0ED82330B07CF5D208ECAAF446BAEA4D4848BB7515573123514EFDEA8E6C1000FEF8F30A9901382B7119822C74B9C91394F9DDFF8B239E241F9",
>   "clienttime_ms": "1647312358378",
>   "support_face_verify": "1",
>   "dfid": "-",
>   "dev": "Pixel%204",
>   "plat": "1",
>   "pk": "7C9E722B7BCCE5B56BAB9D42902B2A4F238BAA40EACCCEECACE72C3EE46A85B1B833F7800E6D2B968EE809303485376180BD8494EBA84F006DBD050E1F876B3713DE64AFADDA4F51FCC522C6DAC0AFFC8A04AABAF47F95FE6D3F2FB6726A16FC4664AB66CE065E3FCFD7D0FACD3DCA05AA96D74DD96EC74DBF54CA72D088E42C",
>   "t1": "bfaf46951c6a5c58d4bf618c517354de",
>   "support_verify": "1",
>   "support_multi": "1",
>   "t2": "c397de1fd7e436639fa90f367c0b47d290831b64f19d803fc09a91ee47bcc1648e5a87f1f934ed09cfb7046dad52253cf564684d5bdef1d67fc91c74d6245335237a78074f157641a1ddfb5ca4a66c5d72855feae18282b981d896c6f170cb54ca7c21aad3811d69c80a73ad3482c340",
>   "key": "8d73d8b5da06bad3b466d864812d488b",
>   "username": "130**\***696"
> }
> ```

### params参数解析

通过JADX搜索“params”字符串定位到位置：

|  |  |
| --- | --- |
| 1 2 3 | `this.j.put(``"params"``, a.c(jSONObject.toString(), this.g));` 进入c方法内可以看到： |
|  |  |
| --- | --- |
| 1 2 3 | `public static String c(String str, String str2) throws Exception {`<br>`return a(str, "utf-8", ao.a(str2).substring(0, 32), ao.a(str2).substring(16, 32));`<br>`}` |

通过objection hook该方法内的a方法，得到如下结果：

> `(agent) [302556] Called com.xxxx.fanxing.allinone.common.utils.a.a(java.lang.String, java.lang.String, java.lang.String, java.lang.String)`

---

> `(agent) [302556] Arguments com.xxx.fanxing.allinone.common.utils.a.a({"username":"13**\***96","clienttime_ms":"1647313862199","pwd":"557998555"}, utf-8, 0dd6da40aea47d05b5f6612596fd2e7f, b5f6612596fd2e7f)`

---

> `(agent) [793424] Backtrace:`

> `com.xxx.fanxing.allinone.common.utils.a.a(Native Method)`

> `com.xxx.fanxing.allinone.common.utils.a.a(SourceFile:117)`

> `com.xxx.fanxing.allinone.common.utils.a.a(Native Method)`

> `com.xxx.fanxing.allinone.common.utils.a.c(SourceFile:106)`

> `com.xxx.fanxing.core.protocol.g.e.c(SourceFile:85)`

> `com.xxx.fanxing.core.protocol.g.e.c(Native Method)`

> `com.xxx.fanxing.core.protocol.g.a.d(SourceFile:76)`

> `com.xxx.fanxing.core.modul.user.login.c.a(SourceFile:50)`

> `com.xxx.fanxing.core.modul.user.login.f.a(SourceFile:228)`

> `com.xxx.fanxing.core.modul.user.ui.a.a(SourceFile:577)`

> `com.xxx.fanxing.core.modul.user.ui.a.j(SourceFile:554)`

> `com.xxx.fanxing.core.modul.user.ui.a.i(SourceFile:520)`

> `com.xxx.fanxing.core.modul.user.ui.a.onClick(SourceFile:417)`

> `android.view.View.performClick(View.java:7259)`

> `android.view.View.performClickInternal(View.java:7236)`

> `android.view.View.access$3600(View.java:801)`

> `android.view.View$PerformClick.run(View.java:27892)`

> `android.os.Handler.handleCallback(Handler.java:883)`

> `android.os.Handler.dispatchMessage(Handler.java:100)`

> `android.os.Looper.loop(Looper.java:214)`

> `android.app.ActivityThread.main(ActivityThread.java:7356)`

> `java.lang.reflect.Method.invoke(Native Method)`

> `com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:492)`

> `com.android.internal.os.ZygoteInit.main(ZygoteInit.java:930)`

---

这样我们可以得到`a.c(jSONObject.toString(), this.g)`中两个参数的来源，其中第一个参数为JSON结构：`{"username":"13**\***96","clienttime_ms":"1647313862199","pwd":"557998555"}`，第二个参数为AES的KEY，最终将KEY拆分为`(0,32)`和`(16,32)`，具体生成方式如下：

|  |  |
| --- | --- |
| 1 | `this.g = com.xxx.fanxing.allinone.common.utils.a.a(com.kugou.fanxing.allinone.common.constant.c.hz() ? 128 : 64);` |

a方法如下：

|  |  |
| --- | --- |
