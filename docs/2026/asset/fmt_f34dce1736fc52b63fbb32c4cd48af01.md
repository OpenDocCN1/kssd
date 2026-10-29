
进入`/path/to/xxx`目录，找到`xxx.json`。

在表达式`cvar = avar + bvar`中，加法运算符（`+`）将`avar`与`bvar`相加，得到它们的和`cvar`

在`List.of(arg0, arg1, arg2)`中，`List`接口的工厂方法`of()`接受一系列的元素，返回包含它们的只读列表。

之后我们这样调用`cmd`命令：`cmd arg0 arg1 arg2`。

```
if (condVar > someVal) {console.log("xxx")}
```
[content]
```
（1）在自定义实现HostnameVerifier时，在verify中对Hostname进行严格校验
（2）建议setHostnameVerifier方法中使用STRICT_HOSTNAME_VERIFIER进行严格证书校验，避免使用ALLOW_ALL_HOSTNAME_VERIFIER
```

### <3>作业帮存在https升级劫持漏洞

大家都知道https是采用加密方式来进行通信，一般来说除非证书的设置方面存在漏洞，否则很难直接去截获报文信息，但是在我挖掘漏洞的过程中，发现一个新的思路，可能这是很多厂商比较懒的原因，直接升级https后，传输的报文数据还是原来的数据，所以我们可以选择采用http旧版本的APP，抓取明文信息，修改后，使用于新版的信息，也可以导致劫持的漏洞

当然这个过程中也需要解决接收方证书信任的问题，还需要模拟https的请求方式，这里可以使用[stunnel配置](https://bbs.pediy.com/thread-268459.htm)，这里主要提供一种思路，其他操作步骤和上述一直，就不再重复演示了

#### （2）漏洞防护

针对于Android https的开发过程中常见的安全缺陷：

```java
1)在自定义实现X509TrustManager时，checkServerTrusted中没有检查证书是否可信，导致通信过程中可能存在中间人攻击，造成敏感数据劫持危害。
2)在重写WebViewClient的onReceivedSslError方法时，调用proceed忽略证书验证错误信息继续加载页面，导致通信过程中可能存在中间人攻击，造成敏感数据劫持危害。
3)在自定义实现HostnameVerifier时，没有在verify中进行严格证书校验，导致通信过程中可能存在中间人攻击，造成敏感数据劫持危害。
4)在setHostnameVerifier方法中使用ALLOW_ALL_HOSTNAME_VERIFIER，信任所有Hostname，导致通信过程中可能存在中间人攻击，造成敏感数据劫持危害。
```

## 四、Android 绕过https的SSL Pining

我们在对Android APP抓包时，经常会出现HTTPS报文通过MITM代理后不被信任的问题，有些https在设置好证书后，会出现：

```
unknown
加密的乱码
报错无法抓包
```

这是因为对方的https采用了 SSL pinning

### 1.SSL pinning

SSL pining = 证书绑定 = SSL证书绑定

表示对方的app只允许承认自己特定的证书，这导致MITM的证书不被识别，不运行，从而导致MITM无法解密看到https的明文数据

### 2.Android 7.0后破解https 的ssl pinning

Android7.0后，系统做了改动：

```
APP 默认不信任用户域的证书。之前把MITM的ssl证书，安装到 受信任的凭据 -> 用户 就没用了，因为不受信任了。只信任（安装到）系统域的证书
```

因此这导致我们使用如Fiddler、Charles等抓包软件导入证书后，仍然不能在捕获https的密文，甚至无法解析请求

解决思路：

```
（1）让系统信任Charles的ssl证书
改自己的app的配置，允许https抓包，这就需要有app的源码
把证书放到受系统信任的系统证书中去。前提是手机已root
（2）绕开https不去校验
使用基于Xposed等框架的JustTrustMe、基于Frida框架的r0capyure等
```

### 3.证书绕过原理

我们基于上文提出第二种思路，详细解析当下的JustTrustMe为代表的证书绕过原理

通过前面我们了解到，证书验证中到关键是 TrustManager，而绕过证书验证就需要从它入手。xpsoed 上证书校验的绕过插件就是这么干的，目前比较流行的两款基于 xposed 的绕过证书验证的模块有两款 JustTrustMe 和 SSLkiller，针对HttpsURLConnection，OkHttp 框架各自的证书校验函数

这两款工具通过 hook 这些关键函数，或替换 TrustManager(信任所有证书)或令其验证函数直接失效(函数替换，不做任何校验)，以达到绕过的目的

绕过证书的实现原理图，下图参考博客[安卓 https 证书校验和绕过](https://elink@dc5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6B7N6h3g2B7K9h3&6Q4x3X3g2U0L8W2)9J5c8Y4m8G2M7%4c8Q4x3V1j5$3z5e0V1J5z5o6b7@1z5e0l9^5y4K6R3^5y4K6p5I4y4o6x3^5)：

![image-20211205155437510](upload/attach/202112/905443_GQTKJ7GVBKAX8FB.png)

### 4.工具使用

#### （1）Xposed+JustTrustMe

[JustTrustMe下载地址](https://elink@b9bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6r3N6i4A6A6L8$3^5J5y4q4)9J5c8V1A6#2M7%4c8f1M7Y4g2K6N6p5#2W2)

使用步骤十分简单，就在手机上安装xposed框架，具体安装参考前文帖子，然后将JustTrustMe模块安装就可以使用了

![image-20211205184421953](upload/attach/202112/905443_5N6SW2AWNKXQKU4.png)

#### （2）Frida脚本

下面是两种比较火的frida抓包脚本

[frida-android-unpinning-ssl](https://elink@677K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3c8W2M7$3S2S2M7X3g2Q4x3X3g2X3M7X3W2V1j5g2)9J5k6i4u0W2i4K6u0r3i4K6b7H3L8h3q4K6j5X3!0Y4i4K6u0r3k6Y4u0A6k6r3q4Q4x3X3c8S2L8X3c8J5L8$3W2V1i4K6u0V1N6h3&6H3K9h3&6F1K9h3&6Y4i4K6u0V1M7%4y4D9i4K6u0r3)

[r0capture](https://elink@a11K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5x3s2W2K6N6h3g2Q4x3V1k6J5x3r3y4S2M7s2c8#2M7X3f1`.)

使用步骤：

开启frida_server注入脚本就可以了，具体可以参考博客网址

## 五、实验总结

本文从Android Http/Https通信过程出发，讲述了Android Http/Https通信漏洞产生的原因，也拿了很多的漏洞复现实例来进行一一说明，最后还简单介绍了当下对https转包的处理和原因，当然这部分东西很多还需要进一步深入的研究，本文可能还未归纳完全所有的情况，就请大家指正了

github首页：[github](https://elink@631K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6i4K9h3&6V1h3r3q4S2i4K6u0r3b7h3&6V1M7X3!0A6k6q4)9J5k6q4k6#2L8r3&6W2M7X3q4T1K9h3I4A6N6s2W2Q4x3X3c8y4K9h3&6A6L8X3M7`.)

## 六、参考文献

Android http/https原理解析：

```
第一行代码
Frida 逆向和抓包实战
https://zhuanlan.zhihu.com/p/330393659
https://xiaoyue26.github.io/2018/09/26/2018-09/%E9%80%9A%E4%BF%97%E7%90%86%E8%A7%A3SSL-TLS%E5%8D%8F%E8%AE%AE%E5%8C%BA%E5%88%AB%E4%B8%8E%E5%8E%9F%E7%90%86/
```

Android https漏洞挖掘：

```
https://www.geek-share.com/detail/2727192403.html
https://www.cxyzjd.com/article/u010982507/85258477
https://www.jianshu.com/p/84df0a40127c
```

Android 证书绕过：

```
https://juejin.cn/post/6992844908788711438
https://www.jianshu.com/p/34912804bf08
https://www.panaihua.com/android-catchhttp/
```

# Android漏洞之战（6）——http/https通信安全漏洞详解

目录

- [Android漏洞之战（6）——http/https通信安全漏洞详解](https://bbs.kanxue.com/#android漏洞之战（6）——http/https通信安全漏洞详解)

- [一、前言](https://bbs.kanxue.com/#一、前言)

- [二、基础知识](https://bbs.kanxue.com/#二、基础知识)

- [1.加密算法](https://bbs.kanxue.com/#1.加密算法)

- [（1）对称加密](https://bbs.kanxue.com/#（1）对称加密)

- [（2）非对称加密](https://bbs.kanxue.com/#（2）非对称加密)

- [2.信息安全问题](https://bbs.kanxue.com/#2.信息安全问题)

- [（1）信息的保密性](https://bbs.kanxue.com/#（1）信息的保密性)

- [（2）信息的完整性（数字签名）](https://bbs.kanxue.com/#（2）信息的完整性（数字签名）)

- [（3）身份识别（数字证书）](https://bbs.kanxue.com/#（3）身份识别（数字证书）)

- [3.Http/Https详解](https://bbs.kanxue.com/#3.http/https详解)

- [（1）TLS/SSL](https://bbs.kanxue.com/#（1）tls/ssl)

- [（2）HTTPs的单向认证和双向认证](https://bbs.kanxue.com/#（2）https的单向认证和双向认证)

- <1>单向认证

- <2>双向认证

- 4.Android Http网络开发

- [（1）HttpURLConnection](https://bbs.kanxue.com/#（1）httpurlconnection)

- [（2）okhttp3](https://bbs.kanxue.com/#（2）okhttp3)

- [（3）各自证书的校验方式](https://bbs.kanxue.com/#（3）各自证书的校验方式)

- [三、HTTP/HTTPs漏洞分析和复现](https://bbs.kanxue.com/#三、http/https漏洞分析和复现)

- [1.漏洞的安全种类和危害](https://bbs.kanxue.com/#1.漏洞的安全种类和危害)

- [2.HTTP明文传输漏洞](https://bbs.kanxue.com/#2.http明文传输漏洞)

- [（1）漏洞案例](https://bbs.kanxue.com/#（1）漏洞案例)

- <1>酷我音乐APP存在逻辑缺陷漏洞

- <2> 上海任意门科技有限公司Soul APP存在信息泄露漏洞

- <3>酷狗直播存在逻辑缺陷漏洞（hash验证）

- [（2）漏洞防护](https://bbs.kanxue.com/#（2）漏洞防护)

- [3.HTTPs密文传输漏洞](https://bbs.kanxue.com/#3.https密文传输漏洞)

- [（1）漏洞案例](https://bbs.kanxue.com/#（1）漏洞案例)

- <1> 忽略SSL证书校验漏洞

- <2>忽略域名校验漏洞

- <3>作业帮存在https升级劫持漏洞

- [（2）漏洞防护](https://bbs.kanxue.com/#（2）漏洞防护)

- 四、Android 绕过https的SSL Pining

- 1.SSL pinning

- 2.Android 7.0后破解https 的ssl pinning

- [3.证书绕过原理](https://bbs.kanxue.com/#3.证书绕过原理)

- [4.工具使用](https://bbs.kanxue.com/#4.工具使用)

- [（1）Xposed+JustTrustMe](https://bbs.kanxue.com/#（1）xposed+justtrustme)

- [（2）Frida脚本](https://bbs.kanxue.com/#（2）frida脚本)

- [五、实验总结](https://bbs.kanxue.com/#五、实验总结)

- [六、参考文献](https://bbs.kanxue.com/#六、参考文献)

## 一、前言

本文主要介绍Android http/https方面的安全漏洞问题，并会从原理并结合案例来逐一讲解，本文一部分参考网络上一些博客，并在相应部分给出链接

本文第二节主要讲述Android http/https相关的基础知识

本文第三节为漏洞原理解析和漏洞复现

本文第四节为Android https转包漏洞介绍

## 二、基础知识

### 1.加密算法

#### （1）对称加密

对称加密算法是双方都持有相同的密钥进行通信，加密速度很快

![image-20211204145051628](https://bbs.kanxue.com/upload/attach/202112/905443_ZY4W66BVMZX9BGM.png)

特点：

|  |  |

| --- | --- |

| 1  2  3  4 | `a.加密和解密都是用同一个秘钥`  `b.加密、解密效率高`  `c.秘钥被窃取，容易造成数据不安全`  `常见的对称加密算法有DES、``3DES``、AES等，这里我们就不深入讲解了` |

缺点：

上面的对称加密模型最大的问题就是，对称加密模型需要一个安全的信道来传输对称密钥，但是如果真的存在一个真正安全的信道，那直接用这个信道来传输数据就可以了，这就有点矛盾了

#### （2）非对称加密

非对称加密，是为了解决对称加密中的安全问题而诞生，含有一对密钥：公钥和私钥，发送方用公钥进行加密，公钥可以被公开，接收方用私钥进行解密，私钥不可公开

![image-20211204150154475](https://bbs.kanxue.com/upload/attach/202112/905443_2NXNM9WJ2QHEHMU.png)

特点：

|  |  |

| --- | --- |

| 1  2  3  4 | `a.用公钥加密用私钥解密`  `b.加密、解密相对于对称加密效率更低，但是比对称加密更安全`  `c.公钥可能被中间人伪造，造成数据不安全`  `常见的非对称加密算法有RSA、DSA等` |

缺点：

如何保证加密的是接收方的公钥，如何安全的传输公钥

### 2.信息安全问题

我们在传输数据的过程中往往着眼于三个方面的安全问题：

|  |  |

| --- | --- |

| 1  2  3 | `（``1``）信息的保密性`  `（``2``）信息的完整性`  `（``3``）身份识别` |

#### （1）信息的保密性

我们一般会使用各种加密算法对我们传输的数据信息进行加密，即使用上面的对称加密和非对称加密来完成，但无论是对称加密还是非对称加密都存在一个共同的安全问题：`密钥如何传递，而且提高传输速率`，一般公用的方法是采用`对称加密+非对称加密结合`，即双方都在使用对称加密进行传输，但是会存在密钥不能保证安全性的问题，此时我们使用公钥对对称密钥进行加密，然后接收方使用私钥对对称密钥进行解密，这样就可以解决这个问题

![image-20211204151540175](https://bbs.kanxue.com/upload/attach/202112/905443_SCEV4D3ZG75J3JT.png)

#### （2）信息的完整性（数字签名）

数据在传输的过程中，我们的信息可能被第三方劫持篡改，所以我们要保证信息的完整性，一般通过使用散列函数如SHA1，MD5将传输内容hash依次获得hash值，即摘要。客户端使用服务端的公钥对摘要和信息内容进行加密，传输给服务端，服务端使用私钥进行解密，然后用相同的hash算法对原始内容进行hash，然后与摘要值对比，如果一直，说明信息是完整的

![image-20211204152606806](https://bbs.kanxue.com/upload/attach/202112/905443_JJTNEHW834H4AZQ.png)

举例：

|  |  |

| --- | --- |

| 1  2  3  4 | `Android APP应用一般就具有签名验证机制，以防止恶意攻击者在对APP进行逆向之后，重打包，一般来说Android APP的签名机制分为``3``类：`  `（``1``）java本地验证，在java代码中有``hash``函数验证，我们通常搜索signature定位到目标代码段，直接删除或hook该代码段即可`  `（``2``）so本地验证，为了加强逆向难度，很多公司会将APP验证写在so层，这一般我们通过IDA动态调试，获取代码段然后NOP即可`  `（``3``）网络服务器验证，一般来说这种进行网络``hash``验证，一般这种通过抓包，但有一些加密后变很难处理了` |

#### （3）身份识别（数字证书）
