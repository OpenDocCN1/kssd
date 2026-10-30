
![](https://bbs.kanxue.com/upload/attach/202009/779730_MWAM9J4AFFRHWH5.png)

相比`NetrServerReqChallenge()`函数的流量特征，`NetrServerAuthenticate3()`函数多了`Negotiation options`，该字段的以`Authenticated RPC supported`和`AES support`位的设置，对漏洞的成功利用具有重要作用，因此`Negotiation options`字段的流量特征和全0的`Client Crenditial`的流量特征，可以作为识别该漏洞的最明显特征。

### 7. 补丁分析

微软在8月份发布了针对该漏洞的安全更新方案。与普通漏洞的安全更新不同，微软打算分阶段完成针对该漏洞的完整防御，但其声称在安装完8月份的安全更新后即可防御该漏洞。可以初步推断，微软打算替换现有的认证方案。考虑到域环境的复杂性，需要花费一定的时间来确认新方案的可行性。

目前，微软官方对Netlogon协议的官方说明文档已经做了更新，部分重要更新如下：

1. 102页：引入了一个新设置项：`VulnerableChannelAllowList`，以安全描述符定义语言(Security Descriptor Definition Language)表示的对Netlogon的client的设置（[MS-DTYP]第2.5.1节）不允许使用安全绑定。
2. 104页：在`session key`的协商过程中新增了一个步骤：如果client质询的前5个字节都不唯一，则server必须在不进行后续步骤的情况下使`session key`协商失败。
3. 110页：`session key`的建立过程新增了两个步骤：“4，如果未使用安全绑定，除非client处于`VulnerableChannelAllowList`设置中，否则server必须拒绝该请求。”以及“6，如果前5个字节中没有一个`ClientStoredCredential`的计算结果（第3.1.4.5节的第1步）是唯一的，server必须在不进行后续步骤的情况下使`session key`协商失败。

使用的文件版本来自Windows Server 2016，更新前文件版本号为：`10.0.14393.1737`，更新后版本号为：`10.0.14393.3866`。

#### 1. BinDiff结果

![](https://bbs.kanxue.com/upload/attach/202009/779730_XSQYX6PJ2P9764C.png)

直接看`NetrServerAuthenticate3()`函数，改动总体上较大，但尚不确定针对该漏洞具体有效的更新。看关键的大改部分：

![](https://bbs.kanxue.com/upload/attach/202009/779730_7PJDSU6QTN92PGV.png)

![](https://bbs.kanxue.com/upload/attach/202009/779730_JHPPTSB5PY5MSN8.png)

![](https://bbs.kanxue.com/upload/attach/202009/779730_763HDXMKA9U26PG.png)

在IDA中看下新增对`NlIsChallengeCredentialPairVulnerable()`函数的调用：

![](https://bbs.kanxue.com/upload/attach/202009/779730_6BN29RTABHS3BJQ.png)

![](https://bbs.kanxue.com/upload/attach/202009/779730_V4UZSK5XQMBH5ZE.png)

进入到新函数`NlIsChallengeCredentialPairVulnerable()`内部：

![](https://bbs.kanxue.com/upload/attach/202009/779730_67GVFYR9XCY3Z34.png)

根据上述代码，我们可以得到该函数的大致逻辑：

- client提供的challenge存储在`rcx`指向的buffer中；
- 检查一些全局变量，如果为1，则函数返回0，表示不存在漏洞。
- 检查`rcx`为非空，检查`rdx`为非空。
- challenge的第1个字节存储在`r9d`中，然后在循环中将接下来的4个字节与它进行比较。如果这4个字节中的任何一个与第1个字节都不相同，则该函数返回0，不存在漏洞；否则返回1，存在漏洞。

这种修复方式针对的主要是当前已公开的poc和exp，即challenge全0的情况（也可以变形成其他的全1或者全2等情况）。这种防御逻辑其实比较简单，现在还无法判定是否可以实现更进一步的补丁绕过。

## 四、漏洞检测和防御

### 1. 漏洞检测

现在针对该漏洞的检测脚本已经在互联网中进行公开，本质上来说，公开的脚本因为尝试了使用空凭据登录，不能算作无损检测。尚未发现其他的检测方法。（终端侧的补丁检测和文件版本扫描除外）

### 2. 漏洞防御

#### 1. 流量侧

流量侧防御较为简单，根据`Negotiation options`字段的流量特征和全0的`Client Crenditial`的流量特征外加爆破限制即可实现针对目前所有公开exp的攻击，本质上来说也算是从漏洞原理侧进行了防御。

#### 2. 终端侧

正如补丁分析所说，针对该漏洞的终端侧防御可以考虑使用热补丁的方，但是否存在风险需要进一步验证。

## 五、参考文献

1. `956K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6k6h3y4#2M7X3q4Q4x3X3g2U0L8$3#2Q4x3V1k6H3j5i4c8Z5N6r3!0A6L8h3N6Q4x3X3g2H3K9s2m8Q4x3@1k6A6k6q4)9K6c8o6t1H3y4e0f1`.
2. [MS-NRPC](https://bbs.kanxue.com/elink@447K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3!0H3k6h3&6K6M7r3g2U0M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4y4h3k6H3M7X3!0@1L8$3y4G2L8s2y4Q4x3V1k6E0M7#2)9J5k6r3&6J5M7r3y4Q4x3V1k6X3k6U0S2X3z5e0M7H3k6W2)9J5k6o6y4W2x3K6N6Q4x3X3b7@1x3r3j5%4i4K6u0V1j5X3b7@1j5W2)9J5k6r3q4X3y4K6x3K6y4X3f1@1y4K6V1J5k6R3`.`.)
3. cccK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2x3s2m8S2N6r3y4Z5i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9J5x3q4)9J5c8U0l9&6i4K6u0r3L8h3W2U0M7X3!0H3j5i4c8U0K9q4)9J5k6r3k6G2M7W2)9J5k6s2A6W2M7X3!0D9L8$3N6G2L8W2)9J5k6s2m8W2M7X3k6W2j5%4c8Q4x3X3g2Z5N6r3#2D9

## 六、特别声明

**看雪内部共享，未经允许，请勿擅自转载。**

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
