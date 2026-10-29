
# Android APP漏洞之战（9）——验证码漏洞挖掘详解

[TOC]

## 一、前言

验证码漏洞也是当下十分常见的APP端漏洞，本文对验证码漏洞原理做了一个初步的讲解，并复现了当下一些常见的验证码相关的漏洞，本文App抓包技术参考了肉丝大佬的书籍安卓Frida逆向和抓包实战

本文第二节主要讲述Android APP端抓包的基础知识

本文第三节主要讲述APP验证码漏洞的种类和安全场景

本文第四节主要讲述APP验证码原理，并复现了一些常见的验证码相关漏洞

## 二、基础知识

APP验证码漏洞挖掘，需要掌握基本的Android抓包技术，现在越来越多的APP开始进行安全防护，所以如何才能绕过一些基本的防护技术，合理的抓取到报文是验证码漏洞挖掘的先决条件。学习下面知识之前，可以先学习[Android APP漏洞之战（6）——HTTP/HTTPs通信漏洞详解](https://bbs.pediy.com/thread-270634.htm)的基础理论知识。

### 1.APP抓包技术详解

#### （1）高版本android系统证书导入

**Android4.4后**

开始引入CA证书校验机制，这里先会涉及到Https的单向认证机制

![](upload/attach/202204/905443_ZT8NQZT7BSBNVND.png)

上述步骤4就进行证书校验的环节，而这里的证书便是将手机内置的证书和客户端的证书进行校验，来判断是否合法

```
Https原理：非对称+对称连接
（1）非对称主要是为了传输对称连接所需要密钥和证书校验
（2）对称连接是加密密码可以安全传输，就可以采用对称连接
```

**Android4.4-Android7.0**

这个阶段，Android系统信任用户下的证书，所以逆向人员可以将抓包工具的证书导入用户目录下，实现对Https的报文抓取，这里案例可以参考[Android协议分析（一）——Fiddler安装和使用 ](https://bbs.pediy.com/thread-268445.htm)

**Android7.0+**

Android7.0后，Android系统不再信任用户目录下证书，只信任根目录下证书，所以我们要获取HTTPs报文，需要将抓包工具的证书移动到根目录下

方法一：将文件系统先挂载起来，然后移动

```java
root权限下：
mount -o remount,rw /system
cp * /etc/security/cacerts/
chmod 777 /etc/security/cacerts/*
mount -o remount,ro /system
reboot
```

方法二：使用Magisk证书模块

<a href="elink@fe7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4j5h3N6A6M7$3E0Q4x3X3c8y4L8$3c8#2L8r3g2K6i4K6u0V1f1X3g2H3L8#2)9J5c8X3#2G2N6X3g2U0k6i4u0@1">Move Certificates</a>

我们只需要将模块安装到Magisk中，这样就可以成功的抓取常规状态下的HTTPs的报文了，这里一定主要抓包时必须保证Android手机时间要正确，否则也会报错

#### （2）HTTPs双向认证及解决方案

Https一般采用单向认证，但是一些特殊的APP在服务端也会对证书进行再次认证，这样的认证机制我们称为HTTPs双向认证

![](upload/attach/202204/905443_G5UUZ434UTCWTQF.png)

上图中步骤5就对客户端证书进行了校验，这里导致我们就算客户端导入了抓包工具的证书并信任，也无法通过服务端的认证

**解决办法：**

```
（1）获取客户端的证书，可以通过编写hook脚本的方式，获取客户端证书及密码
（2）将客户端证书导入到抓包软件charles中，使得Charles能都获取服务端信任（这里一般可以使用charles、Burpsuit软件）
```

方法一：

这里我们一般可以先去分析开发中进行保护的函数，再对对应的函数进行hook

```
例如java.security.KeyStore
```

这里可以参考肉丝大佬的文章<a href="elink@cbcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0p5&6y4K6j5#2y4#2)9J5x3$3R3K6i4K6u0V1x3e0p5">实用FRIDA进阶：内存漫游、hook anywhere、抓包</a>

![](upload/attach/202204/905443_T2W8R2TKJ57VHXT.png)

这里考虑篇幅问题，给一个参考的实操案例<a href="elink@15cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0K9o6y4F1P5h3g2Q4x3X3g2@1L8%4m8Q4x3V1k6Q4x3U0g2q4x3#2)9J5y4e0R3H3i4K6t1#2z5e0m8Q4x3U0g2q4y4g2)9J5y4f1q4q4i4K6t1#2z5f1g2Q4x3U0g2q4y4W2)9J5y4e0R3^5i4K6t1#2z5e0S2Q4x3U0g2q4x3#2)9J5y4e0R3H3i4K6t1#2z5e0q4Q4x3U0g2q4y4W2)9J5y4e0W2r3i4K6t1#2z5e0m8Q4x3U0g2q4y4q4)9J5y4f1u0m8i4K6t1#2b7e0c8Q4x3U0g2q4y4g2)9J5y4e0S2r3i4K6t1#2z5p5u0S2M7s2m8Q4x3U0g2q4y4#2)9J5y4e0W2m8i4K6t1#2z5o6c8Q4x3U0g2q4y4g2)9J5y4e0S2r3i4K6t1#2z5p5y4Q4x3U0g2q4y4g2)9J5y4e0V1H3i4K6t1#2z5e0q4Q4x3U0g2q4z5q4)9J5y4f1q4q4i4K6t1#2b7e0c8Q4x3U0g2q4z5q4)9J5y4f1q4r3i4K6t1#2z5o6q4U0M7X3q4U0K9#2)9J5c8R3">某交友app的双向认证crack</a>

方法二：

直接在Apk文件下搜索`.p12`后缀的文件，这里搜索出来的很有可能就是证书文件，当然有些需要密码的就还是要进行hook了

```
tree -NCfhl |grep -i p12
```

![image-20220409205835643](upload/attach/202204/905443_G4WJWBMG4DQPK59.png)

#### （3）VPN代理机制

如果APP中编写了检测代理抓包软件代码，也可能会导致我们抓不到包

```
System.getProperty("http.proxyHost");
System.getProperty("http.proxyPort");
```

这样我们使用代理软件进行抓包要更加合适，当然也有一些检测VPN的情况，这种直接hook就可以了

![image-20220410094013590](upload/attach/202204/905443_VESS3FGUPQSNNGC.png)

具体如何配置使用，可以参考这篇文章<a href="elink@8ebK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8X3q4Z5f1r3u0n7f1$3k6C8K9@1u0K6N6U0c8G2P5e0t1$3y4i4u0u0x3W2p5">APP流量 “一个都不能少” ！</a>

采用VPN代理进行抓包，和Http中间人抓包有本质区别，VPN抓包本质在网络层和路由层抓包

#### （4）SSL pinning

就算我们使用Vpn代理抓包，也解决了证书双向认证的问题，但是在抓包过程中也可能会遇到证书绑定的情况

证书绑定：

```
APP在代码段中有有对服务端证书进行对比一致性，如果发现是中间人就会直接终止连接
```

这里我们提供一般的解决思路：

```
（1）识别证书绑定
（2）先使用开源框架objection或DroidSSLUnpinning先去简单hook
（3）不成功，再去分析APP使用哪种开发框架，去有针对性的进行hook
（4）如果不知道开发框架，可以采用Hook Java的File构造函数进一步去定位证书绑定代码
```

这里用一个简单的实例来说明：

**识别证书绑定：**

![image-20220410095914907](upload/attach/202204/905443_B95NWG5WCQT8EUU.png)

当我们解决了单向认证、双向认证、采用代理通信还出现上面的情况，说明APP可能采用了证书绑定

**使用开源框架hook——以objection为例：**

![image-20220410095914907](upload/attach/202204/905443_KQ4RHX5NBRWE943.png)

我们直接使用objection的hook或spwan模式就进行证书绑定hook

```
hook模式启动：android sslpinning disable
spwan模式启动：objection -g 包名 explore -s "android sslpinning disable"
```

**针对开发框架，有针对性hook:**

我们需要分析APP采用何种开源框架，如okhttp等，不过这里一般开源hook工具objection已经针对了当下的很多开源框架了，所以如果没有成功，可能是程序进行了混淆，这个时候就需要进一步去hook正确的绑定函数

**未知开源框架：**

我们对File构造函数进行hook，然后可以观察对应的调用链信息，从而分析出证书绑定的函数

![image-20220410095914907](upload/attach/202204/905443_5BV7YTXWD28FYRU.png)

再针对这里找到的函数进行正确的hook，即可

#### （5）VPN+Charles+Burpsuit环境配置

我们经过上面一系列流程基本可以解决当下常见的安全防护形式，当然针对更加安全的抓包防护策略就需要深入的解决了，这里我们介绍一下本文的抓包环境搭建，一般使用通过VPN+Charles+Burpsuit的组合抓包方式

首先先配置VPN+Charles的抓包模式

![image-20220410101546946](upload/attach/202204/905443_R5B6AYZ28BNBTNR.png)

![image-20220410101609229](upload/attach/202204/905443_VP6VPQV6U3UJHU7.png)

然后我们将Charles的流量去导入到Burpsuit中，这样就可以实现组合使用，当然根据实际情况你也可以选择合适抓包工具

![image-20220410102252123](upload/attach/202204/905443_7R3ZVHZZ5QQ7SXU.png)

![image-20220410102431749](upload/attach/202204/905443_RX44G9XM97SEFPK.png)

## 三、验证码漏洞安全场景和分类

### 1.验证码漏洞安全场景

验证码漏洞一般是APP对验证码没有进行登陆校验等安全防护导致的，攻击者一般可以利用验证码漏洞进行暴力破解实现任意号码登陆、还可以实现短信轰炸、导致验证码泄露、手机号换绑、验证码无限发送风险、万能验证码登录、修改返回包绕过验证码登录等安全问题，验证码漏洞在APP中的影响十分严重，一般会造成严重的经济损失。

### 2.验证码漏洞分类

我们根据上面的描述，可以将验证码漏洞分为如下几类：

![image-20220411111532796](upload/attach/202204/905443_D5QN867CFZSBFRV.png)

## 四、验证码漏洞原理分析和复现

### 1.验证码暴力破解漏洞

#### （1）原理分析

验证码暴力破解漏洞一般是因为用户使用手机号+验证码的方式进行登陆时，短信验证码一般由4-6位数字组成，而且APP并未对验证码做时间和失败次数校验，所以攻击者可以通过这个区间的所有数字进行暴力破解来进行攻击

#### （2）漏洞复现

案例：XX APP为例

![image-20220410105515919](upload/attach/202204/905443_V6S3UJN4M9YNK8Q.png)

我们打开APP，发现该APP并未对登录号码进行验证，而且返回的验证码是4位验证码，这里我们就十分怀疑是否能进行验证码暴力破解

然后我们使用上面搭建好的抓包环境，随便输入一个验证码去抓取错误信息

![image-20220410105515919](upload/attach/202204/905443_TY7792RA6ZMHVDF.png)

我们发现APP根本没有进行证书绑定等安全防护，甚至没对验证码进行加密，我们于是对响应的报文进行攻击

![image-20220410105515919](upload/attach/202204/905443_WRMHC58ME7GKK6A.png)

然后我们采用100个线程，就暴力破解

![image-20220410105515919](upload/attach/202204/905443_86JV5KQBSXASXUY.png)

![image-20220410105515919](upload/attach/202204/905443_VTKDHNKZA5C3KAV.png)

开始攻击

![image-20220410105515919](upload/attach/202204/905443_5SM5VG3DKTPR6BS.png)

然后我们就成功暴力破解出验证码，这里我们发现还可以输入任意的号码`18888888888`同样可以成功登录，而且可以获取其他用户的订单信息

![image-20220410110251127](upload/attach/202204/905443_A6YNNM5MC788CSN.png)

最后类似漏洞危害，如下

![image-20220410110705360](upload/attach/202204/905443_M72FT4RQQ9AZ2MX.png)

#### （3）安全防护

```
（1）设置验证码时间验证，一般建议为180s
（2）限制单位时间内验证码失败的尝试错误，如5分钟连续失败即锁定账号
（3）对验证码进行加密传输
```

### 2.验证码无限次数发送（短信轰炸）

#### （1）原理分析

验证码无限次数发送、短信轰炸漏洞一般是APP端并没有对手机号码进行次数显示，也未对访问进行时间限制

#### （2）漏洞复现

我们首先对上面的APP进行发送验证码，查看响应的报文

![image-20220410111915010](upload/attach/202204/905443_BWJ6RZQ84RAM8YE.png)

![image-20220411101714278](upload/attach/202204/905443_C7KUVF365KAFTNX.png)

然后通过暴力破解进行短信轰炸，这里轰炸自己手机号，就轰炸个50次吧

![image-20220410112221486](upload/attach/202204/905443_R3QT875EZVURBZ3.png)

![image-20220410112415310](upload/attach/202204/905443_2Y5G6BM3V87MMN3.png)

我们发现可以验证码无限次数发送，但是无法进行短信轰炸，因为验证码做了60s校验，于是尝试去分析APP代码

我们将APP拖入GDA分析

![image-20220411102019374](upload/attach/202204/905443_V9QPRZRGSEVRHHB.png)

这里我们发现APP进行360加固保护，我们就使用一代壳脱壳工具Frida_Dump进行脱壳

![image-20220411102212716](upload/attach/202204/905443_6QBU5UYGDNAQJQF.png)

我们定位到相关的代码段:

![image-20220411102327963](upload/attach/202204/905443_7C5WADS9SS7SEZQ.png)

分析了代码逻辑，本来以为可以简单的hook相关的函数，使得验证码60s时间变少，但是发现成功发送后还是显示需要60s后再次发送，这说明这个验证就是根据发送的报文，在服务端进行校验

我们查看发送的报文：

![image-20220411104546386](upload/attach/202204/905443_9G72D2TZK7R6Q3M.png)

我们发现获取验证码就是对应的手机号和time值，很显然我们推测time值是用来校验时间和对应手机号的

我们分析对应源码：

![image-20220411104714014](upload/attach/202204/905443_EZAU859PU3NVNGE.png)

![image-20220411104815174](upload/attach/202204/905443_E6XCRKWQDMWM9Z7.png)

我们可以清晰发现为什么APP验证码对当前系统时间进行校验，因为有系统时间获取函数

![image-20220411104901496](upload/attach/202204/905443_3VNZ942S7XZN99R.png)
