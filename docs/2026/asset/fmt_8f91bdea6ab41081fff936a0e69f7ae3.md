
# 蛇从暗黑中袭来——响尾蛇(SideWinder) APT组织2020年上半年活动总结报告

```
注意事项:

1.本报告由追影小组原创,未经许可禁止转载

2.本文一共3245字,36张图,预计阅读时间8分钟

3.本文涉及的敏感内容皆以打码,并且不公开C2和hash.所造成的恶劣影响与本公众号和本团队无关
```

## 0x00.前言:

响尾蛇(又称**SideWinder**，**T-APT-04**)是一个背景可能来源于印度的 APT 组织，该组织此前已对巴基斯坦和东南亚各国发起过多次攻击，该组织以窃取政府，能源，军事，矿产等领域的机密信息为主要目的。

在今年年初的时候，**Gcow安全团队**的**追影小组**发布了关于`SiderWinder` **APT组织**的报告——**《游荡于中巴两国的魅影——响尾蛇(SideWinder) APT组织针对巴基斯坦最近的活动以及2019年该组织的活动总结》**。本小组也一直对该小组的活动加以跟踪。

在2020上半年的活动中该组织的主要目标依然是**巴基斯坦，中国，孟加拉国以及其他的东南亚国家**，其主要是集中在**政府，军事**领域。不过值得注意的是在本次活动也出现了**体育比赛方面**的话题。同时该组织在针对某重点单位的时候采取的使用钓鱼网站的方式窃取相关人员的凭据，目前所发现这种方式主要针对的是与军事有关的部门。其实该组织也同样利用关于**COVID-19**的信息作为诱饵对中巴的教育机构以及政府机构进行攻击活动。

从样本攻击流程来看，该组织的技术并没有太多的革新。主要的样本形式有两类，第一类为带有`CVE-2017-11882`漏洞的**RTF(富文本)**文件;另一类为使用`mshta.exe`执行远程hta脚本的**LNK文件**。不过该组织在针对我国某所大学的攻击活动中所采取的载荷较为不同，其采用了`CVE-2017-0199`漏洞结合`CVE-2020-0674`漏洞的方式进行攻击，在后续的内容中我们会详细的介绍该组织使用的新手法。具体的流程会在后文中以流程图的形式展现出来。

`Gcow`安全团队**追影小组**初步统计了关于2020年上半年国内外各大厂商以及安全团队所发布的披露**SideWinder APT组织**的相关报告信息，并把相关报告链接放在了文末的相关链接上(若有不全欢迎私信补充)

![图片1-2020上半年各大厂商发布关于SideWinder APT组织的报告](https://bbs.kanxue.com/upload/attach/202007/879321_QDQ5PPN8MMZSK95.jpg)

根据不完全统计，`Gcow`安全团队**追影小组**还绘制了该组织2020年上半年的攻击活动脉络图。

![图片2-SideWinder APT组织2020上半年的活动时间轴](https://bbs.kanxue.com/upload/attach/202007/879321_T9ENGN9MQBMPYXC.jpg)

## 0x01.样本分析:

该组织主要投放的样本类型以**LNK文件**为主，**RTF文件**为辅。基本在针对每个目标的攻击活动中，我们都发现其使用两种载荷交替攻击的情况。

### 一.针对巴基斯坦的活动:

下图为该组织针对巴基斯坦活动所投放的**LNK文件**:

![图片3-2020上半年SideWinder APT组织针对巴基斯坦所投放LNK样本](https://bbs.kanxue.com/upload/attach/202007/879321_HRXUNC98WCEJWYU.jpg)

其主要通过将`lnk`文件放到压缩包中，以达到绕过邮件网关的目的。

其释放的诱饵文件如下:

![图片4-向退役军人发放csd回扣卡](https://bbs.kanxue.com/upload/attach/202007/879321_ZBJGC9XYN6DBYYQ.jpg)

![图片5-巴基斯坦军方抗击疫情相关战略](https://bbs.kanxue.com/upload/attach/202007/879321_AFYPJXJT6JVXWJ9.jpg)

![图片6-巴基斯坦空军大学疫情期间网络课程政策](https://bbs.kanxue.com/upload/attach/202007/879321_TKXG9YVFJK9CDRZ.jpg)

其中这些LNK文件的参数为`%windir%\system32\cftmo.exe {HTA URL}`

这里的`cftmo.exe`是伪装的`mshta.exe`.该组织通过修改`sIDList`结构以迷惑受害者。

![图片7-lnk载荷的参数](https://bbs.kanxue.com/upload/attach/202007/879321_FX9Y8T2YSUT9CKW.jpg)

所请求的远程hta文件，该组织以自写的解密算法取代了之前的硬编码，不过核心与上一篇文章我们所描述的一样，在内存加载.Net的dll文件`LinkZip.dll`

![图片8-第一阶段hta脚本](https://bbs.kanxue.com/upload/attach/202007/879321_RPNGEKXJNKZB6MW.jpg)

`LinkZip.dll`被传入四个参数

```
参数1-下一段hta文件的URL地址

参数2-上传杀软信息的URL地址

参数3-base64和Gzip加密后的诱饵文档数据

参数4-诱饵文档名称
```

该`LinkZip.dll`先解密参数3，再将诱饵文档写入临时文件夹，最后运行。就会出现我们提到的那些诱饵的文档，以迷惑受害者。

![图片9-解密诱饵文档数据并运行](https://bbs.kanxue.com/upload/attach/202007/879321_CCUA58GVPBRBNFP.jpg)

将参数1的第二阶段hta文件下载下来，存在`%temp%\`目录下，利用`mshta.exe`将其运行起来，若运行不成功将会向参数2反馈在第一阶段hta脚本中收集的杀软信息以及其他异常情况。

![图片10-执行第二段hta脚本并反馈异常信息](https://bbs.kanxue.com/upload/attach/202007/879321_Y2RK6CNS9XUUFY4.jpg)

第二阶段hta脚本同样是解密后内存加载`.Net`文件`StInstaller.dll`

![图片11-第二阶段hta脚本](https://bbs.kanxue.com/upload/attach/202007/879321_TRXTGFMP47V982H.jpg)

`StInstaller.dll`被传入了三个参数

```
参数1-加密的duser.dll数据

参数2-加密的{随机名}.tmp数据

参数3-编码的回链C2地址
```

其先从`%windir%\system32\`或者`%windir%\syswow64\`拷贝`rekeywiz.exe`到该木马的指定文件夹下，解密`Duser.dll`与`{随机名}.tmp`，并将其写入木马指定的文件夹下，以组成`rekeywiz.exe`与`Duser.dll`白加黑组合进行攻击，并且通过写注册表启动项的方式将`rekeywiz.exe`设为自启动达到权限维持的效果。

![图片12-StInstaller.dll](https://bbs.kanxue.com/upload/attach/202007/879321_23TU23U9934ZJ26.jpg)

`Duser.dll`以`rekeywiz.exe`的侧加载执行起来，并且其主要功能是读取同目录下的`.tmp`文件，并且选取其前32个字节为异或的秘钥，解密后面的内容再内存加载。

![图片13-Duser.dll](https://bbs.kanxue.com/upload/attach/202007/879321_6RGWWUDVT49N4BZ.jpg)

其解密出了最后的Net文件`SystemApp.dll`

由于最后的远控与上篇文章没有什么区别，这里不再赘述。

![图片14-SystemApp.dll](https://bbs.kanxue.com/upload/attach/202007/879321_PFGMA48RSFSX2DF.jpg)

远控指令:

![图片15-远控指令](https://bbs.kanxue.com/upload/attach/202007/879321_966WPF8P7Z2UU8X.jpg)

为了方便各位看官的理解本团队特意画了一张流程图，方便各位更加直观地了解这个组织的手法。

![](https://bbs.kanxue.com/upload/attach/202007/879321_4G92DDBFZVRUUSQ.jpg)

### 二.针对孟加拉国的攻击活动:

本小组发现该组织针对孟加拉国的活动主要模仿了孟加拉国代购商协会对相关单位以及人员进行攻击活动。

![图片17-模仿孟加拉国代购商协会进行攻击](https://bbs.kanxue.com/upload/attach/202007/879321_3AYSF4GZESEXGW3.jpg)

该属于**RTF类型**的样本，主要是利用嵌入的ole对象释放`1.a`文件

![图片18-1.a文件](https://bbs.kanxue.com/upload/attach/202007/879321_YC8AFFBMV6XHEZ2.jpg)

`1.a`文件类似于上文介绍Lnk载荷中提到的第二阶段hta文件，其在内存中解密`StInstaller.dll`并释放白加黑组合，后续的白加黑组合也和上文类似，这里不再赘述。

![](https://bbs.kanxue.com/upload/attach/202007/879321_SV8ZEQKM3HQFUGF.jpg)

### 三.针对中国的攻击活动:

在本次活动之中，本小组捕获了针对中国某某重点大学的网络攻击活动，如下是其使用的文件诱饵，话题关于**2020年春季的疫情防控工作**的优秀教师推荐名单。

![图片20-针对某某重点大学的诱饵](https://bbs.kanxue.com/upload/attach/202007/879321_F2AH2445G6S7EDE.jpg)

其利用内置一个frameset组件，ID是`rId912`.该组件会指向一个远程的RTF文件，以此完成远程**远程模板注入技术**，加载远程模板，这是一种绕过杀软静态查杀的好方法。

![图片21-远程模板注入技术](https://bbs.kanxue.com/upload/attach/202007/879321_F2KGYNE2C55GNJC.jpg)

远程模板为RTF文档，其内嵌了OLE对象，并且使用了漏洞`CVE-2017-0199`加载其内置会自动更新的超链接域。

![图片22-远程模板内置的自动更新超链接域](https://bbs.kanxue.com/upload/attach/202007/879321_UDZ2MSPT3Z9ZXGC.jpg)

该超链接更新域指向一个hta文件，hta文件的解密算法与上文提到的类似。

不过值得注意的是，该hta文件的异或解密秘钥是从网站上获取的。

若攻击者需要及时停止攻击活动只需要撤走相关的秘钥即可。

![图片23-hta文件自解密部分](https://bbs.kanxue.com/upload/attach/202007/879321_SV23GBMZYCFNVJ9.jpg)

其解密的文件如下所示，经过调试发现其崩溃点偏移量与双星漏洞所公开的POC完全相同，均位于`jscript+0x1cfbb`偏移处.以此确定为双星0day之一的`CVE-2020-0674`漏洞的利用，属于**浏览器nday漏洞利用**范畴。

![图片24-CVE-2020-0674利用](https://bbs.kanxue.com/upload/attach/202007/879321_YEUH5ZHV2N8PUHP.jpg)

![图片25-与公开POC崩溃点偏移完全一致](https://bbs.kanxue.com/upload/attach/202007/879321_M8YT7T56HVEP7D8.jpg)

## 0x05.相关链接:

- <mark class="encrypted">332K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2N6s2u0W2L8X3c8E0K9h3y4J5L8#2)9J5k6h3y4G2L8g2)9J5c8Y4c8J5k6h3&6V1L8r3q4T1M7#2)9J5k6s2y4W2j5%4g2J5K9i4c8&6i4K6u0V1K9h3&6@1k6h3I4D9K9h3N6W2L8X3y4W2i4K6u0r3k6X3W2J5M7%4c8Q4x3X3c8S2j5%4c8A6N6X3g2Q4x3X3c8S2N6s2c8S2j5$3E0Q4x3X3c8W2P5s2m8D9L8$3W2@1K9h3&6Y4i4K6u0V1j5%4k6W2i4K6u0V1x3U0l9I4z5g2)9J5k6o6t1J5x3e0g2Q4x3X3c8X3L8%4g2F1k6q4)9J5k6r3!0F1i4K6u0V1k6$3!0G2k6$3I4W2i4K6u0V1M7r3I4S2P5g2)9J5k6r3I4A6L8X3E0W2k6q4)9J5k6s2c8G2i4K6u0V1M7$3W2V1k6i4N6A6L8X3c8W2M7W2)9J5k6r3q4H3N6q4)9J5k6r3N6J5L8%4g2H3i4K6u0r3</mark>

- <mark class="encrypted">595K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8Y4W2^5g2g2c8s2x3#2q4$3j5e0p5$3z5g2)9J5k6q4S2A6h3g2j5H3M7p5q4&6f1b7`.`.</mark>

- <mark class="encrypted">fc1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8U0W2x3k6V1g2D9c8r3u0w2b7%4u0c8h3o6q4c8P5V1N6r3d9g2y4r3f1s2M7`.</mark>

- <mark class="encrypted">b17K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8V1E0T1i4K6g2X3N6$3!0t1M7o6q4E0K9h3q4o6k6@1c8K9P5f1S2x3d9p5&6Y4b7b7`.`.</mark>

- <mark class="encrypted">0a3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8V1y4K9M7X3c8K6L8s2A6q4M7K6c8A6N6$3I4S2g2s2A6v1d9o6N6g2j5X3M7`.</mark>

- https://bbs.pediy.com/thread-259500.htm

- <mark class="encrypted">675K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4M7#2)9J5k6e0x3$3x3q4)9J5k6h3y4F1i4K6u0r3M7r3!0K6N6q4)9J5c8X3q4H3N6q4)9J5k6r3y4Q4x3X3b7H3y4W2)9#2k6U0m8V1j5i4W2Q4x3X3g2Z5N6r3#2D9</mark>

- <mark class="encrypted">9b4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4M7#2)9J5k6h3A6H3j5$3g2J5N6q4)9J5k6h3!0J5i4K6u0W2K9Y4m8Q4x3V1k6W2L8W2)9J5c8U0t1H3x3U0m8Q4x3V1j5H3y4q4)9J5c8X3W2W2i4K6u0V1k6X3W2J5k6h3k6G2P5q4)9J5k6o6m8V1j5i4W2Q4x3X3g2Z5N6r3#2D9</mark>

### C2

- `B6932A288649B3CEB9A454F808D6EB35`

- `7E461F6366681C5AE24920A31C3CFEC6`

- `nrots[.]net`

- <mark class="encrypted">d2aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3b7H3x3h3k6S2i4K6g2n7i4K6u0W2i4K6g2p5L8X3g2@1</mark>

- <mark class="encrypted">507K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3k6V1L8W2)9J5k6r3g2F1i4K6g2n7i4K6u0W2i4K6g2p5L8X3g2@1</mark>

- `ap-ms[.]net`

- `r0dps[.]net`

- `www-afc[.]chrom3[.]net`

- `cloud-apt[.]net`

- <mark class="encrypted">9adK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3I4A6L8X3E0Q4x3X3c8U0k6r3&6D9i4K6g2n7i4K6u0W2i4K6g2p5L8X3g2@1</mark>

- `kat0x[.]net`

- `www[.]au-edu[.]km01s[.]net`
