
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21 |
| --- |
| `private HttpClient getNewHttpClient() {`  `try`  `{`  `KeyStore trustStore` `=` `KeyStore.getInstance(KeyStore`  `.getDefaultType());`  `trustStore.load(null, null);`  `SSLSocketFactory sf` `=` `new SSLSocketFactory(trustStore);`  `sf.setHostnameVerifier(SSLSocketFactory.ALLOW_ALL_HOSTNAME_VERIFIER);` `//这里信任了所以的hostname，导致可能存在中间人攻击`  `HttpParams params` `=` `new BasicHttpParams();`  `HttpProtocolParams.setVersion(params, HttpVersion.HTTP_1_1);`  `HttpProtocolParams.setContentCharset(params, HTTP.UTF_8);`  `SchemeRegistry registry` `=` `new SchemeRegistry();`  `registry.register(new Scheme("http", PlainSocketFactory`  `.getSocketFactory(),` `80));`  `registry.register(new Scheme("https", sf,` `443));`  `ClientConnectionManager ccm` `=` `new ThreadSafeClientConnManager(`  `params, registry);`  `return` `new DefaultHttpClient(ccm, params);`  `} catch (Exception e) {`  `return` `new DefaultHttpClient();`  `}`  `}` |

**案例二：[WPS存在信息泄漏漏洞](https://bbs.kanxue.com/elink@f47K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8Y4k6V1i4K6u0W2L8%4u0Y4i4K6u0W2j5$3&6Q4x3V1k6X3L8r3q4%4i4K6u0r3M7$3S2G2N6#2)9J5c8U0b7H3y4U0V1I4x3o6j5`.)**

WPS采用HTTPs进行通信但是由于证书校验问题，可以被获取到敏感信息，从而导致信息泄漏漏洞，这里和上面一致，就不再重新演示了

![image-20211205155437510](https://bbs.kanxue.com/upload/attach/202112/905443_R9J9B4SYAH5ZXX3.png)

安全防护：

| 1  2 |
| --- |
| `（1）在自定义实现HostnameVerifier时，在verify中对Hostname进行严格校验`  `（2）建议setHostnameVerifier方法中使用STRICT_HOSTNAME_VERIFIER进行严格证书校验，避免使用ALLOW_ALL_HOSTNAME_VERIFIER` |

##### <3>作业帮存在https升级劫持漏洞

大家都知道https是采用加密方式来进行通信，一般来说除非证书的设置方面存在漏洞，否则很难直接去截获报文信息，但是在我挖掘漏洞的过程中，发现一个新的思路，可能这是很多厂商比较懒的原因，直接升级https后，传输的报文数据还是原来的数据，所以我们可以选择采用http旧版本的APP，抓取明文信息，修改后，使用于新版的信息，也可以导致劫持的漏洞

当然这个过程中也需要解决接收方证书信任的问题，还需要模拟https的请求方式，这里可以使用[stunnel配置](https://bbs.pediy.com/thread-268459.htm)，这里主要提供一种思路，其他操作步骤和上述一直，就不再重复演示了

#### （2）漏洞防护

针对于Android https的开发过程中常见的安全缺陷：

| 1  2  3  4 |
| --- |
| `1)在自定义实现X509TrustManager时，checkServerTrusted中没有检查证书是否可信，导致通信过程中可能存在中间人攻击，造成敏感数据劫持危害。`  `2)在重写WebViewClient的onReceivedSslError方法时，调用proceed忽略证书验证错误信息继续加载页面，导致通信过程中可能存在中间人攻击，造成敏感数据劫持危害。`  `3)在自定义实现HostnameVerifier时，没有在verify中进行严格证书校验，导致通信过程中可能存在中间人攻击，造成敏感数据劫持危害。`  `4)在setHostnameVerifier方法中使用ALLOW_ALL_HOSTNAME_VERIFIER，信任所有Hostname，导致通信过程中可能存在中间人攻击，造成敏感数据劫持危害。` |

## 四、Android 绕过https的SSL Pining

我们在对Android APP抓包时，经常会出现HTTPS报文通过MITM代理后不被信任的问题，有些https在设置好证书后，会出现：

| 1  2  3 |
| --- |
| `unknown`  `加密的乱码`  `报错无法抓包` |

这是因为对方的https采用了 SSL pinning

### 1.SSL pinning

SSL pining = 证书绑定 = SSL证书绑定

表示对方的app只允许承认自己特定的证书，这导致MITM的证书不被识别，不运行，从而导致MITM无法解密看到https的明文数据

### 2.Android 7.0后破解https 的ssl pinning

Android7.0后，系统做了改动：

| 1 |
| --- |
| `APP 默认不信任用户域的证书。之前把MITM的ssl证书，安装到 受信任的凭据` `-` `> 用户 就没用了，因为不受信任了。只信任（安装到）系统域的证书` |

因此这导致我们使用如Fiddler、Charles等抓包软件导入证书后，仍然不能在捕获https的密文，甚至无法解析请求

解决思路：

| 1  2  3  4  5 |
| --- |
| `（1）让系统信任Charles的ssl证书`  `改自己的app的配置，允许https抓包，这就需要有app的源码`  `把证书放到受系统信任的系统证书中去。前提是手机已root`  `（2）绕开https不去校验`  `使用基于Xposed等框架的JustTrustMe、基于Frida框架的r0capyure等` |

### 3.证书绕过原理

我们基于上文提出第二种思路，详细解析当下的JustTrustMe为代表的证书绕过原理

通过前面我们了解到，证书验证中到关键是 TrustManager，而绕过证书验证就需要从它入手。xpsoed 上证书校验的绕过插件就是这么干的，目前比较流行的两款基于 xposed 的绕过证书验证的模块有两款 JustTrustMe 和 SSLkiller，针对HttpsURLConnection，OkHttp 框架各自的证书校验函数

这两款工具通过 hook 这些关键函数，或替换 TrustManager(信任所有证书)或令其验证函数直接失效(函数替换，不做任何校验)，以达到绕过的目的

绕过证书的实现原理图，下图参考博客[安卓 https 证书校验和绕过](https://bbs.kanxue.com/elink@317K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6B7N6h3g2B7K9h3&6Q4x3X3g2U0L8W2)9J5c8Y4m8G2M7%4c8Q4x3V1j5$3z5e0V1J5z5o6b7@1z5e0l9^5y4K6R3^5y4K6p5I4y4o6x3^5)：

![image-20211205155437510](https://bbs.kanxue.com/upload/attach/202112/905443_GQTKJ7GVBKAX8FB.png)

### 4.工具使用

#### （1）Xposed+JustTrustMe

[JustTrustMe下载地址](https://bbs.kanxue.com/elink@083K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6r3N6i4A6A6L8$3^5J5y4q4)9J5c8V1A6#2M7%4c8f1M7Y4g2K6N6p5#2W2)

使用步骤十分简单，就在手机上安装xposed框架，具体安装参考前文帖子，然后将JustTrustMe模块安装就可以使用了

![image-20211205184421953](https://bbs.kanxue.com/upload/attach/202112/905443_5N6SW2AWNKXQKU4.png)

#### （2）Frida脚本

下面是两种比较火的frida抓包脚本

[frida-android-unpinning-ssl](https://bbs.kanxue.com/elink@fb2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3c8W2M7$3S2S2M7X3g2Q4x3X3g2X3M7X3W2V1j5g2)9J5k6i4u0W2i4K6u0r3i4K6b7H3L8h3q4K6j5X3!0Y4i4K6u0r3k6Y4u0A6k6r3q4Q4x3X3c8S2L8X3c8J5L8$3W2V1i4K6u0V1N6h3&6H3K9h3&6F1K9h3&6Y4i4K6u0V1M7%4y4D9i4K6u0r3)

[r0capture](https://bbs.kanxue.com/elink@6d0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5x3s2W2K6N6h3g2Q4x3V1k6J5x3r3y4S2M7s2c8#2M7X3f1`.)

使用步骤：

开启frida\_server注入脚本就可以了，具体可以参考博客网址

## 五、实验总结

本文从Android Http/Https通信过程出发，讲述了Android Http/Https通信漏洞产生的原因，也拿了很多的漏洞复现实例来进行一一说明，最后还简单介绍了当下对https转包的处理和原因，当然这部分东西很多还需要进一步深入的研究，本文可能还未归纳完全所有的情况，就请大家指正了

github首页：[github](https://bbs.kanxue.com/elink@2a3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6i4K9h3&6V1h3r3q4S2i4K6u0r3b7h3&6V1M7X3!0A6k6q4)9J5k6q4k6#2L8r3&6W2M7X3q4T1K9h3I4A6N6s2W2Q4x3X3c8y4K9h3&6A6L8X3M7`.)

## 六、参考文献

Android http/https原理解析：

| 1  2  3  4 |
| --- |
| `第一行代码`  `Frida 逆向和抓包实战`  `https:``/``/``zhuanlan.zhihu.com``/``p``/``330393659`  `https:``/``/``xiaoyue26.github.io``/``2018``/``09``/``26``/``2018``-``09``/``%``E9``%``80``%``9A``%``E4``%``BF``%``97``%``E7``%``90``%``86``%``E8``%``A7``%``A3SSL``-``TLS``%``E5``%``8D``%``8F``%``E8``%``AE``%``AE``%``E5``%``8C``%``BA``%``E5``%``88``%``AB``%``E4``%``B8``%``8E``%``E5``%``8E``%``9F``%``E7``%``90``%``86``/` |

Android https漏洞挖掘：

| 1  2  3 |
| --- |
| `https:``/``/``www.geek``-``share.com``/``detail``/``2727192403.html`  `https:``/``/``` 250K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4^5P5i4A6B7k6q4)9J5k6h3y4G2L8b7`.`. ```/``article``/``u010982507``/``85258477`  `https:``/``/``` 318K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3A6A6j5h3&6K6K9s2g2Q4x3X3g2U0L8$3@1`. ```/``p``/``84df0a40127c` |

Android 证书绕过：

| 1  2  3 |
| --- |
| `https:``/``/``juejin.cn``/``post``/``6992844908788711438`  `https:``/``/``` 9b4K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3A6A6j5h3&6K6K9s2g2Q4x3X3g2U0L8$3@1`. ```/``p``/``34912804bf08`  `https:``/``/``ce3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8S2L8X3q4A6K9s2g2S2i4K6u0W2j5$3!0E0``/``android``-``catchhttp``/` |

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2022-4-12 17:03

被随风而行aa编辑

，原因：

[#基础理论](https://bbs.kanxue.com/forum-161-1-117.htm)

[#漏洞相关](https://bbs.kanxue.com/forum-161-1-123.htm)

[#源码框架](https://bbs.kanxue.com/forum-161-1-127.htm)

[#其他](https://bbs.kanxue.com/forum-161-1-129.htm)
