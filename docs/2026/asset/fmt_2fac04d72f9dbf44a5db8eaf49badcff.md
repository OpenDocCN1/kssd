
- 修改 `app` 的 `AndroidManifest.xml` 的 `Application` 为 dex 壳的 `Application`

- 壳 dex 和源 dex 拼接成一个 dex，格式如下：

  - 壳 dex + (1 字节 application 名长度 + `app` 的 application 名 + 4 字节源 dex 大小 + 源 dex) + 4 字节源 dex2 大小 + [源 dex2] + 4 字节源 dex3 大小 + [源 dex3] + ... + 4 字节壳 dex 大小

  - 小括号的数据前 512 字节进行 AES 加密

  - 中括号的数据的 dex 头也就是前 112 位进行异或

- 打包签名 `app`

**壳 dex 处理：**

- 解密出 `app` 的 application 名和 dex

- 加载 `app` 的 dex

  - Android 8.0 以下版本采用 call `libart` 的 `OpenMemory` 函数实现内存加载 dex

  - Android 8.0 及以上采用系统提供的 `InMemoryDexClassLoader` 实现内存加载 dex

- 壳 `application` 替换为 `app` 的 `application`

## 4. 不落地加载壳的对抗

我们为了进一步验证不落地加载的安全性，使用恶意应用检测来验证最好，大家可以设想，如果我们的加壳技术进行升级后，防护性更强是不是说明源码不易被检测和识别，那么使用加壳的恶意应用来检测就十分能说明这个问题，例如加壳防护级别越高的恶意应用越难以检测出恶意性，那么我们检测平台的分数应该也会越低，按照这个设想我们开展实验：

### 4.1 未加壳

我们还是拿之前的恶意样本，我们先不对其加壳，查看恶意性：

![image-20221008210342685](https://bbs.kanxue.com/upload/attach/202305/905443_PXTG7HGUQN9UNKT.png)

可以发现 VirusTotal 上面有 27 家厂商识别出了恶意应用，说明这个恶意性还是很容易检测到的

### 4.2 动态加载壳

我们使用之前的一代壳加载器进行加壳，查看其恶意性：

![image-20221008210405210](https://bbs.kanxue.com/upload/attach/202305/905443_NHPFZQRVNDHKE8D.png)

这里发现分数略微降低，但是还是很高，这是因为之前一代加壳器，我们未对 dex 进行加密，而且 dex 保存本地，所以十分容易检测出来

### 4.3 不落地加载壳

我们使用本文的不落地加载壳进行加载，查看其恶意性：

![image-20221018201207513](https://bbs.kanxue.com/upload/attach/202305/905443_66G4JADGZY9FEDN.png)

这里可以发现 67 家厂商中只有 9 家厂商可以识别出来，识别率大大降低，这更加反映了不落地加载防护技术的升级

## 5. 实验总结

本文讲述了不落地加载的基本原理，也总结了和前面的落地加载的区别，本文通过分析一个开源项目详细的分析了不落地加载器实现的基本原理，最后通过实验进一步证明了不落地加载的防护性进一步升级。本文的相关资料存放知识星球：安全后厨。

github 地址：[github](https://bbs.kanxue.com/elink@e42K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6i4K9h3&6V1h3r3q4S2i4K6u0r3b7h3&6V1M7X3!0A6k6q4)9J5k6s2u0W2N6X3g2J5M7$3f1`.)

## 6. 参考文献

|  |  |

| --- | --- |

| 1  2  3 | `https://bbs.pediy.com/thread-261939.htm#msg_header_h1_2`  `https://链接失效`  `https://github.com/Frezrik/Jiagu` |

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2025-1-13 09:39

被 kanxue 编辑，原因：

[#基础理论](https://bbs.kanxue.com/forum-161-1-117.htm)

[#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)

[#脱壳反混淆](https://bbs.kanxue.com/forum-161-1-122.htm)

[#其他](https://bbs.kanxue.com/forum-161-1-129.htm)
