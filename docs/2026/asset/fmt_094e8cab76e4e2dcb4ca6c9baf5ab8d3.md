
# [翻译]朝鲜APT组织HIDDEN COBRA远控工具NukeSped的深入分析

## 朝鲜APT组织HIDDEN COBRA远控工具NukeSped的深入分析

### 简介

高级持续性威胁（APT）组织对全球安全构成了巨大威胁，尤其是有国家背景和支持的组织。在所有的APT组织中，来自朝鲜的组织因其强破坏能力以及高持续性而尤为值得注意。美国政府特别提到，一个称为HIDDEN COBRA的APT组织与朝鲜政府相关。

<a href="elink@2fdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3L8%4u0@1K9h3&6W2N6q4)9J5k6h3y4G2L8g2)9J5c8X3k6G2M7Y4c8A6k6%4g2S2M7X3c8Q4x3V1k6@1K9s2u0W2j5i4c8Q4x3X3c8A6L8Y4c8W2L8r3I4A6k6$3g2F1j5$3g2Q4x3V1k6@1K9s2u0W2j5i4c8Q4x3X3c8J5k6i4y4W2j5i4u0U0K9q4)9J5k6h3S2@1L8h3H3`.">FortiGuard Labs</a>一直在积极监视各种APT组织，包括HIDDEN COBRA。例如，在<a href="elink@66bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3L8%4u0@1K9h3&6W2N6q4)9J5k6h3y4G2L8g2)9J5c8X3u0D9L8$3N6Q4x3V1k6@1K9s2u0W2j5i4c8Q4x3X3c8J5k6i4y4W2j5i4u0U0K9q4)9J5c8X3q4Q4x3X3c8V1k6h3g2H3i4K6u0V1k6r3W2$3k6g2)9J5k6r3q4F1j5h3I4&6M7$3W2K6i4K6u0V1L8$3k6Q4x3X3c8@1K9r3g2Q4x3X3c8X3j5h3I4D9j5$3S2A6L8r3I4Q4x3X3c8J5k6h3#2G2N6r3g2Q4x3X3c8S2k6r3#2A6L8X3W2K6N6s2u0S2N6r3W2G2L8W2)9J5k6s2c8G2L8$3I4Q4x3X3g2Z5N6r3#2D9">上一篇文章</a>中，我们介绍了FALLCHILL远控工具（RAT）。最近，我们捕获到该组织使用的一些有趣的新样本，因此我们决定做进一步研究。

### 远控样本概览

我们分析的远控样本如下：

![远控样本](upload/attach/201911/747358\_P4CN9EJ3GENNCMZ.png)

*图1：远控样本*

总体来看，它们具有以下相似特征：

- 大多数是32位
- 字符串已加密以对抗分析
- 编译时间范围为2017年5月4日10:40:47至2018年2月13日04:06:28

如下文所分析，实际上，它们的相似点要多于不同点，有的甚至复用函数。

![代码复用](upload/attach/201911/747358\_SKX7F692FEPNDVV.png)

*图2：代码复用*

让我们进一步看下资源部分，因为这部分常常可以提供一些恶意软件来源的线索。

![Language ID](upload/attach/201911/747358\_ZG2GSEU3KPYJEYT.png)

*图3：Language ID*

可以看到，每个资源都有对应的Language ID。好奇的是，多数样本Language ID都是`1042`。

![多数样本Language ID都是1042](upload/attach/201911/747358\_H2VMR2PPRKMES5V.png)

*图4：多数样本Language ID都是1042*

根据<a href="elink@0b7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3W2F1N6r3I4Q4x3V1k6D9j5h3&6Y4N6h3q4Y4k6g2)9J5k6r3W2V1k6h3&6@1K9h3k6A6k6i4u0Q4x3X3c8U0L8$3&6K6N6r3q4F1N6s2y4Q4x3X3c8S2L8X3c8Q4x3X3c8K6N6s2u0A6L8X3N6K6">权威数据</a>，`1042`（`0x0412`）是韩语的语言标识。

![LANG_KOREAN](upload/attach/201911/747358\_K6D5MQGUG38N2H6.png)

*图5：LANG_KOREAN*

### 恶意软件功能

我们首先尝试分析这种恶意软件可能对受害者系统造成的影响。通常，最好的方法是检查恶意软件从目标系统调用的函数（比如API）。让我们开始吧。

乍一看，这些恶意软件似乎并没有调用很多API。导入表很短，也没有导入很多常见的DLL和函数。我们的直觉认为它可能会动态解析函数。确实，我们很快找到了`GetProcAddress`的实例。它甚至还加密了其API名，根据我们的经验，这是对抗静态分析的常用技术，但是对动态分析无效。因此，我们跟踪了恶意软件行为活动并找出了加密的API。

![解密后的API](upload/attach/201911/747358\_SWA3EN7AQP522N6.png)

*图6：解密后的API*

可以看出，在用IDA分析之后，一切都说得通了。

下图显示了一个特例，其中函数名并没有加密，因此静态分析就可以了。

![函数表](upload/attach/201911/747358\_JPKKZXYNBNKJS8D.png)

*图7：函数表*

该特殊样本的哈希值为`b05aae59b3c1d024b19c88448811debef1eada2f51761a5c41e70da3db7615a9`。聪明的读者可能已经注意到，此样本加载的函数的顺序与其他样本非常相似。

![DLL](upload/attach/201911/747358\_XVS4JHT39D3EXUF.png)

*图8：DLL*

用IDA查看函数名后，我们可以清楚地看到该恶意软件使用了注册表（`Advapi32.dll`）、网络（`ws2_32.dll`）等核心函数。

为了保持持续性，该恶意软件将其自身注入到`Run`键中：

![持续性](upload/attach/201911/747358\_BQ6K75PVW2WH9AS.png)

*图9：持续性*

某些情况下，该恶意软件会伪装成服务进程。

![服务](upload/attach/201911/747358\_UKJ8299WJ4AY47P.png)

*图10：服务*

可以看到，DLL原始名隐藏于此。

### 恶意软件猫腻

我们来看下NukeSped的主要功能：远程控制。

经过逆向分析，我们找到了解密字符串的算法。

![解密算法](upload/attach/201911/747358\_6DMNVV53AVFNQ2M.png)

*图11：解密算法*

简而言之，恶意软件使用基于`xor`的自定义加密方式。反过来，我们用`decodeCmd`来解密远程攻击者的命令。

![解密命令](upload/attach/201911/747358\_KSPFCDUM93EJEHT.png)

*图12：解密命令*

![shell的逻辑](upload/attach/201911/747358\_VVU84DMG8X43S4N.png)

*图13：shell的逻辑*

正如典型的远控工具，该恶意软件会监听、执行和响应远程命令。完整的控制流程图（CFG）如下所示：

![控制流程图](upload/attach/201911/747358\_QB4T7SNW2RH5TDV.png)

*图14：控制流程图*

从图14可以看出，shell的控制流程很清楚。首先是解析命令及其参数的通用逻辑，然后接入一个非常庞大的switch-case来处理每个命令。

我们逆向分析了该远控工具的逻辑，发现了许多经典的远控功能：

- 遍历文件
- 以其他用户的身份创建进程
- 遍历进程和模块
- 终止进程
- 创建进程
- 写文件
- 读文件
- 连接到远程主机
- 移动文件
- 从云端获取并运行payload
- 获取磁盘信息，包括磁盘类型和磁盘的可用空间大小
- 获取当前目录
- 切换到其他目录
- 从受害系统中删除主文件和其他关联文件

### 归因

归因几乎总是不精准的艺术，但让我们考虑以下关键证据：

- 加密字符串的方式，以及加载API时调用字符串的方式（如图8）
- 功能集和主要功能（远控）的结构让人想起<a href="elink@853K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3L8%4u0@1K9h3&6W2N6q4)9J5k6h3y4G2L8g2)9J5c8X3u0D9L8$3N6Q4x3V1k6@1K9s2u0W2j5i4c8Q4x3X3c8J5k6i4y4W2j5i4u0U0K9q4)9J5c8X3q4Q4x3X3c8V1k6h3g2H3i4K6u0V1k6r3W2$3k6g2)9J5k6r3q4F1j5h3I4&6M7$3W2K6i4K6u0V1L8$3k6Q4x3X3c8@1K9r3g2Q4x3X3c8X3j5h3I4D9j5$3S2A6L8r3I4Q4x3X3c8J5k6h3#2G2N6r3g2Q4x3X3c8S2k6r3#2A6L8X3W2K6N6s2u0S2N6r3W2G2L8W2)9J5k6s2c8G2L8$3I4Q4x3X3g2Z5N6r3#2D9">FALLCHILL</a>（如下）

![FALLCHILL的shell逻辑](upload/attach/201911/747358\_43R6DE7VWV67RDR.png)

*图15：FALLCHILL的shell逻辑*

- NukeSped的大多数样本都包含以下blob加密（如图16）。有趣的是，样本都包含相似的blob加密：

![blob加密](upload/attach/201911/747358\_PC9A2BDQMRG44XQ.png)

*图16：blob加密*

- 有趣的是，还有与<a href="elink@e33K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2#2M7#2)9J5k6r3y4W2M7Y4c8Q4x3X3g2Y4L8%4k6Q4x3V1k6F1j5$3q4K6i4K6u0r3j5h3&6S2L8s2W2K6K9i4y4Q4x3X3c8J5k6i4m8G2M7Y4c8K6i4K6u0r3b7g2t1I4z5g2)9J5k6o6p5H3x3p5p5`.">HOPLIGHT</a>关联的文件名

![dump文件](upload/attach/201911/747358\_32S2FWPSBEE2YUW.png)

*图17：dump文件*

![HOPLIGHT分析](upload/attach/201911/747358\_Y3UX5WHD36Y4H9W.png)

*图18：HOPLIGHT分析*

- 大多数NukeSped样本（约70%）语言为韩文（如图4）

鉴于到目前为止的所有证据，我们可以得出结论，NukeSped远控工具与朝鲜APT组织（HIDDEN COBRA）有关。

### 解决方案

FortiGuard Labs的内部测试表明，受Fortinet解决方案保护的网络和设备，且安装了最新的订阅服务更新，可免受此恶意软件的影响。

尤其是，FortiGuard Antivirus服务可检测以下样本：

- `1a01b8a4c505db70f9e199337ce7f497b3dd42f25ad06487e29385580bca3676` <a href="elink@925K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2$3K9i4u0#2M7%4c8G2N6r3q4D9i4K6u0W2j5$3!0E0i4K6u0r3k6%4g2A6i4K6u0r3M7$3g2S2M7X3y4Z5i4K6u0r3k6X3!0J5N6r3W2F1k6i4c8Q4x3U0f1J5y4e0y4m8i4K6t1#2x3U0f1J5x3W2M7$3y4q4)9J5y4e0t1#2x3V1k6t1K9h3c8o6L8$3u0J5j5g2)9J5k6f1q4Q4x3U0q4@1M7W2)9J5y4e0t1#2x3U0t1`.">W64/HidCobra.A!tr</a>
- `8a1d57ee05d29a730864299376b830a7e127f089e500e148d96d0868b7c5b520` <a href="elink@5f0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2$3K9i4u0#2M7%4c8G2N6r3q4D9i4K6u0W2j5$3!0E0i4K6u0r3k6%4g2A6i4K6u0r3M7$3g2S2M7X3y4Z5i4K6u0r3k6X3!0J5N6r3W2F1k6i4c8Q4x3U0f1J5y4e0y4m8i4K6t1#2x3U0f1J5x3W2M7K6x3W2)9J5y4e0t1#2x3V1k6z5N6h3E0W2f1%4m8W2k6q4)9J5k6f1q4g2i4K6t1I4N6s2u0Q4x3U0f1J5y4e0t1J5">W32/NukeSped.AU!tr</a>
- `32ec329301aa4547b4ef4800159940feb950785f1ab68d85a14d363e0ff2bc11` <a href="elink@497K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2$3K9i4u0#2M7%4c8G2N6r3q4D9i4K6u0W2j5$3!0E0i4K6u0r3k6%4g2A6i4K6u0r3M7$3g2S2M7X3y4Z5i4K6u0r3k6X3!0J5N6r3W2F1k6i4c8Q4x3U0f1J5y4e0y4m8i4K6t1#2x3U0f1J5x3W2M7K6x3W2)9J5y4e0t1#2x3V1k6f1M7X3!0B7j5h3&6Q4x3X3g2r3f1p5W2m8i4K6t1I4N6s2u0Q4x3U0f1J5y4e0t1J5">W32/Trojan.FPIA!tr</a>
- `73dcb7639c1f81d3f7c4931d32787bdf07bd98550888c4b29b1058b2d5a7ca33` <a href="elink@261K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2$3K9i4u0#2M7%4c8G2N6r3q4D9i4K6u0W2j5$3!0E0i4K6u0r3k6%4g2A6i4K6u0r3M7$3g2S2M7X3y4Z5i4K6u0r3k6X3!0J5N6r3W2F1k6i4c8Q4x3U0f1J5y4e0y4m8i4K6t1#2x3U0f1J5x3W2M7K6x3W2)9J5y4e0t1#2x3V1k6z5N6h3E0W2f1%4m8W2k6q4)9J5k6f1q4g2i4K6t1I4N6s2u0Q4x3U0f1J5y4e0t1J5">W32/NukeSped.AU!tr</a>
- `084b21bc32ee19af98f85aee8204a148032ce7eabef668481b919195dd62b319` <a href="elink@471K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2$3K9i4u0#2M7%4c8G2N6r3q4D9i4K6u0W2j5$3!0E0i4K6u0r3k6%4g2A6i4K6u0r3M7$3g2S2M7X3y4Z5i4K6u0r3k6X3!0J5N6r3W2F1k6i4c8Q4x3U0f1J5y4e0y4m8i4K6t1#2x3U0f1J5x3W2M7K6x3W2)9J5y4e0t1#2x3V1k6t1K9h3c8o6L8$3u0J5j5g2)9J5k6e0W2o6c8V1u0Q4x3U0q4@1M7W2)9J5y4e0t1#2x3U0t1`.">W32/HidCobra.9CFB!tr</a>
- `0608e411348905145a267a9beaf5cd3527f11f95c4afde4c45998f066f418571` <a href="elink@2f8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2$3K9i4u0#2M7%4c8G2N6r3q4D9i4K6u0W2j5$3!0E0i4K6u0r3k6%4g2A6i4K6u0r3M7$3g2S2M7X3y4Z5i4K6u0r3k6X3!0J5N6r3W2F1k6i4c8Q4x3U0f1J5y4e0y4m8i4K6t1#2x3U0f1J5x3W2M7K6x3W2)9J5y4e0t1#2x3V1k6z5N6h3E0W2f1%4m8W2k6q4)9J5k6f1q4g2i4K6t1I4N6s2u0Q4x3U0f1J5y4e0t1J5">W32/NukeSped.AU!tr</a>
