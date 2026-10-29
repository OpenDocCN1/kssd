
同理，我们可以发现验证码校验的一些段落，包括四位验证码等。

最后经过分析，得出就是服务端会对每次发送的手机号、系统时间和手机号编码产生的密钥进行校验，会对 60s 时间进行校验，以及会对当前手机系统时间进行校验。

这里我们可以简单进行 60s 间隔的短信轰炸，可以简单实现效果如下：

![image-20220411105257847](upload/attach/202204/905443\_472RVBNCVDDPRGA.png)

因为现在短信轰炸漏洞越来越少，这里没找到更加合适的案例，这里收集一个大佬写的短信漏洞轰炸的文章，可以参考下：[挖洞技巧：绕过短信&邮箱轰炸限制以及后续](elink@8c1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0V1K6z5o6M7^5。

#### （3）安全防护

```text
(1)增加验证码发送校验时间间隔
(2)采用代理池对ip进行限制
```

### 3. 验证码泄露风险与万能验证码漏洞

#### （1）原理分析

验证码执行【找回密码】操作时，输入手机号，获取验证码，服务器会向手机发送验证码。通过 `Burp Suite` 抓包工具，查看返回的响应数据包中如果包含验证码，则可能导致用户密码恶意重置、绑定手机号被恶意更换等风险。

万能验证码一般是开发在业务未上线的时候为了方便测试用的，上线后忘记删除了，例如 `8888`、`0000`、`1234` 等等。

#### （2）安全防护

因为当下这种漏洞较少，这里没有找到合适案例，就只简单介绍下原理了。

```text
（1）禁止验证码在本地客户端生成，应采用服务器端验证码生成机制；
（2）设置验证码的时效性，如180秒过期；
（3）验证码应随机生成，且使用一次即失效。
```

### 4. 修改返回包绕过登录漏洞

#### （1）原理分析

在 APP 验证码测试过程中，我们发现很多 APP 并未对登录成功的响应报文进行校验，导致我们获取登录成功的响应报文后，就可以通过正确的响应报文实现任意号码的登录，导致信息泄露。

#### （2）漏洞复现

案例：XXAPP

![image-20220411110615223](upload/attach/202204/905443\_NTP9WGTZ3YUK6PM.png)

我们先使用手机号进行正确的登录，同时我们使用 `Burp Suite` 抓取正确登录的响应信息。

**发送的请求：**

![image-20220411110709567](upload/attach/202204/905443\_D77N86AQXUWKHPU.png)

**响应的报文：**

![image-20220411110756668](upload/attach/202204/905443\_T4ER7D8ZM5ZSDA5.png)

此时我们可以获取正确响应的报文，我们只需要对在下次登录时对该响应请求进行替换即可。

![image-20220411110840086](upload/attach/202204/905443\_Z52SFTF4D27RC4S.png)

此时我们再次登录，这里我们随便输入验证码 `5555`。

![image-20220411110840086](upload/attach/202204/905443\_D9QC88RR9XASBEF.png)

然后我们注意设置对响应进行抓取。

![image-20220411111006306](upload/attach/202204/905443\_T3H628SBEM6RM8H.png)

![image-20220411111006306](upload/attach/202204/905443\_8689HK9SJKW2ZE9.png)

![image-20220411111055714](upload/attach/202204/905443\_4N8TBSM9UWVBP73.png)

然后进行替换。

![image-20220411111136053](upload/attach/202204/905443\_KH6N7PZ3G5CT78R.png)

最后我们替换报文发现成功登录：

![image-20220411111203716](upload/attach/202204/905443\_27J7VXEVNXJY98G.png)

#### （3）安全防护

```text
不要在前端利用服务端返回的值判断是否可以修改密码 要把整个效验环节交给服务端
```

## 五、实验总结

本文详细的介绍了 APP 中的抓包技术，以及讲述了当下 APP 端验证码漏洞挖掘实现的一些常见的方式，验证码漏洞还有更多的漏洞，后续会进一步完善，本文拿了一些实例具体复现了漏洞的挖掘步骤，并提交 CNVD 获得了一些中高危漏洞，一些更详细资料放在 GitHub 和星球中。

GitHub 首页：[GitHub](elink@0dcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6i4K9h3&6V1h3r3q4S2i4K6u0r3b7h3&6V1M7X3!0A6k6q4)9J5k6q4k6#2L8r3&6W2M7X3q4T1K9h3I4A6N6s2W2Q4x3X3c8y4K9h3&6A6L8X3M7`).

## 六、参考文献

### 参考书籍

```text
安卓 Frida 逆向与抓包实战
```

### 验证码漏洞

```text
https://www.52pojie.cn/thread-1613856-1-1.html
https://www.jianshu.com/p/d2b4c6a54d68
https://www.bilibili.com/read/cv4218190
```

### 短信轰炸漏洞

```text
https://www.anquanke.com/post/id/93878
https://bbs.ichunqiu.com/forum.php?mod=viewthread&tid=27614
```

# Android APP 漏洞之战（9）——验证码漏洞详解

## 目录

- [Android APP 漏洞之战（9）——验证码漏洞详解](#android-app-漏洞之战9验证码漏洞详解)
- [一、前言](#一前言)
- [二、基础知识](#二基础知识)
- [1. APP 抓包技术详解](#1-app-抓包技术详解)
  - [（1）高版本 Android 系统证书导入](#1高版本-android-系统证书导入)
  - [（2）HTTPS 双向认证及解决方案](#2https-双向认证及解决方案)
  - [（3）VPN 代理机制](#3vpn-代理机制)
  - [（4）SSL pinning](#4ssl-pinning)
  - [（5）VPN + Charles + Burp Suite 环境配置](#5vpn--charles--burp-suite-环境配置)
- [三、验证码漏洞安全场景和分类](#三验证码漏洞安全场景和分类)
- [1. 验证码漏洞安全场景](#1-验证码漏洞安全场景)
- [2. 验证码漏洞分类](#2-验证码漏洞分类)
- [四、验证码漏洞原理分析和复现](#四验证码漏洞原理分析和复现)
- [1. 验证码暴力破解漏洞](#1-验证码暴力破解漏洞)
  - [（1）原理分析](#1原理分析)
  - [（2）漏洞复现](#2漏洞复现)
  - [（3）安全防护](#3安全防护)
- [2. 验证码无限次数发送（短信轰炸）](#2验证码无限次数发送短信轰炸)
  - [（1）原理分析](#1原理分析-1)
  - [（2）漏洞复现](#2漏洞复现-1)
  - [（3）安全防护](#3安全防护-1)
- [3. 验证码泄露风险与万能验证码漏洞](#3验证码泄露风险与万能验证码漏洞)
  - [（1）原理分析](#1原理分析-2)
  - [（2）安全防护](#2安全防护-1)
- [4. 修改返回包绕过登录漏洞](#4修改返回包绕过登录漏洞)
  - [（1）原理分析](#1原理分析-3)
  - [（2）漏洞复现](#2漏洞复现-2)
  - [（3）安全防护](#3安全防护-2)
- [五、实验总结](#五实验总结)
- [六、参考文献](#六参考文献)

## 一、前言

验证码漏洞也是当下十分常见的 APP 端漏洞，本文对验证码漏洞原理做了一个初步的讲解，并复现了当下一些常见的验证码相关的漏洞，本文 APP 抓包技术参考了肉丝大佬的书籍《安卓 Frida 逆向和抓包实战》。

本文第二节主要讲述 Android APP 端抓包的基础知识。

本文第三节主要讲述 APP 验证码漏洞的种类和安全场景。

本文第四节主要讲述 APP 验证码原理，并复现了一些常见的验证码相关漏洞。

## 二、基础知识

APP 验证码漏洞挖掘，需要掌握基本的 Android 抓包技术。现在越来越多的 APP 开始进行安全防护，所以如何才能绕过一些基本的防护技术，合理地抓取到报文，是验证码漏洞挖掘的先决条件。学习下面知识之前，可以先学习 [Android APP 漏洞之战（6）——HTTP/HTTPS 通信漏洞详解](https://bbs.pediy.com/thread-270634.htm) 的基础理论知识。

### 1. APP 抓包技术详解

#### （1）高版本 Android 系统证书导入

**Android 4.4 后**

开始引入 CA 证书校验机制，这里先会涉及到 HTTPS 的单向认证机制。

![](https://bbs.kanxue.com/upload/attach/202204/905443_ZT8NQZT7BSBNVND.png)

上述步骤 4 就进行证书校验的环节，而这里的证书便是将手机内置的证书和客户端的证书进行校验，来判断是否合法。

| 步骤 | 说明 |
| --- | --- |
| 1、2、3 | HTTPS 原理：非对称 + 对称连接。<br>（1）非对称主要是为了传输对称连接所需要的密钥和证书校验。<br>（2）对称连接是加密密码可以安全传输，就可以采用对称连接。 |

**Android 4.4—Android 7.0**

这个阶段，Android 系统信任用户下的证书，所以逆向人员可以将抓包工具的证书导入用户目录下，实现对 HTTPS 报文的抓取，这里案例可以参考 [Android 协议分析（一）——Fiddler 安装和使用](https://bbs.pediy.com/thread-268445.htm)。

**Android 7.0+**

Android 7.0 后，Android 系统不再信任用户目录下证书，只信任根目录下证书，所以我们要获取 HTTPS 报文，需要将抓包工具的证书移动到根目录下。

**方法一：将文件系统先挂载起来，然后移动**

```bash
# root 权限下：
mount -o remount,rw /system
cp * /etc/security/cacerts/
chmod 777 /etc/security/cacerts/*
mount -o remount,ro /system
reboot
```

**方法二：使用 Magisk 证书模块**

[Move Certificates](https://bbs.kanxue.com/elink@bb7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4j5h3N6A6M7$3E0Q4x3X3c8y4L8$3c8#2L8r3g2K6i4K6u0V1f1X3g2H3L8#2)9J5c8X3#2G2N6X3g2U0k6i4u0@1)

我们只需要将模块安装到 Magisk 中，这样就可以成功地抓取常规状态下的 HTTPS 报文了。这里一定要注意，抓包时必须保证 Android 手机时间正确，否则也会报错。

#### （2）HTTPS 双向认证及解决方案

HTTPS 一般采用单向认证，但是一些特殊的 APP 在服务端也会对证书进行再次认证，这样的认证机制我们称为 HTTPS 双向认证。

![](https://bbs.kanxue.com/upload/attach/202204/905443_G5UUZ434UTCWTQF.png)

上图中步骤 5 就对客户端证书进行了校验，这里导致我们就算客户端导入了抓包工具的证书并信任，也无法通过服务端的认证。

**解决办法：**

| 步骤 | 说明 |
| --- | --- |
| 1、2 | （1）获取客户端的证书，可以通过编写 Hook 脚本的方式，获取客户端证书及密码。<br>（2）将客户端证书导入到抓包软件 Charles 中，使得 Charles 能够获取服务端信任（这里一般可以使用 Charles、Burp Suite 软件）。 |

**方法一：**

这里我们一般可以先去分析开发中进行保护的函数，再对对应的函数进行 Hook。

例如 `java.security.KeyStore`。

这里可以参考肉丝大佬的文章 [实用 Frida 进阶：内存漫游、Hook Anywhere、抓包](https://bbs.kanxue.com/elink@66cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0p5&6y4K6j5#2y4#2)9J5x3$3R3K6i4K6u0V1x3e0p5)。

![](https://bbs.kanxue.com/upload/attach/202204/905443_T2W8R2TKJ57VHXT.png)

这里考虑篇幅问题，给一个参考的实操案例：[某交友 APP 的双向认证 crack](https://bbs.kanxue.com/elink@bc4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0K9o6y4F1P5h3g2Q4x3X3g2@1L8%4m8Q4x3V1k6Q4x3U0g2q4x3#2)9J5y4e0R3H3i4K6t1#2z5e0m8Q4x3U0g2q4y4g2)9J5y4f1q4q4i4K6t1#2z5f1g2Q4x3U0g2q4y4W2)9J5y4e0R3^5i4K6t1#2z5e0S2Q4x3U0g2q4x3#2)9J5y4e0R3H3i4K6t1#2z5e0q4Q4x3U0g2q4y4W2)9J5y4e0W2r3i4K6t1#2z5o6c8Q4x3U0g2q4y4g2)9J5y4e0S2r3i4K6t1#2z5p5y4Q4x3U0g2q4y4g2)9J5y4e0V1H3i4K6t1#2z5e0q4Q4x3U0g2q4z5q4)9J5y4f1q4q4i4K6t1#2b7e0c8Q4x3U0g2q4z5q4)9J5y4f1q4r3i4K6t1#2z5o6q4U0M7X3q4U0K9#2)9J5c8R3.`)。

**方法二：**

直接在 APK 文件下搜索 `.p12` 后缀的文件，这里搜索出来的很有可能就是证书文件，当然有些需要密码的就还是要进行 Hook 了。

```bash
tree -NCfhl | grep -i p12
```

![image-20220409205835643](https://bbs.kanxue.com/upload/attach/202204/905443_G4WJWBMG4DQPK59.png)

#### （3）VPN 代理机制

如果 APP 中编写了检测代理抓包软件代码，也可能会导致我们抓不到包。

```java
System.getProperty("http.proxyHost");
System.getProperty("http.proxyPort");
```

这样我们使用代理软件进行抓包要更加合适。当然也有一些检测 VPN 的情况，这种直接 Hook 就可以了。

![image-20220410094013590](https://bbs.kanxue.com/upload/attach/202204/905443_VESS3FGUPQSNNGC.png)
