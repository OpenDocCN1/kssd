
# [原创]浅析:隐藏在游戏外挂中的盗号木马-光明中的黑暗-CatGames

文件名称：`病毒样本.exe` SHA256：9d2e499ddc4b7a1ed3a288f0bda807b09b5d701936d53b670f61862a06ec57c1

---

首先在文章开头先@腾讯电脑管家 因为此病毒不管是腾讯电脑管家指定病毒扫描还是直接运行 腾讯电脑管家都不给予报毒和拦截处理（文章中有测试GIF图）

## 0x00

今日在分析一款游戏辅助的执行流程是发现的一些蹊跷，此款辅助会在C盘写入一个文件，文件LOGO是dota的LOGO经验告诉我 这玩意是有问题

![](https://bbs.kanxue.com/upload/attach/202001/93_GTZSZR59WYM5F6V.jpg)

![](https://bbs.kanxue.com/upload/attach/201904/809626_BP93DWKQAGXQHDE.jpg)

## 0x01

外挂只有一个执行程序，我在分析的时候发现此款辅助会释放一个`EXE`

索性我就把它提取出来了 PE DUMP即可 一般这种捆绑的`exe`都放在`Resource`里面

所以提取吧

![](https://bbs.kanxue.com/upload/attach/201904/809626_T2F3995MDNSYPE5.jpg)

![](https://bbs.kanxue.com/upload/attach/201904/809626_DUQCJAMEJHBFJMX.jpg)

我把提取出来的分为两类

![](https://bbs.kanxue.com/upload/attach/201904/809626_TRAGM7M65Z29HKH.jpg)

![](https://bbs.kanxue.com/upload/attach/201904/809626_NDYYVFN59KPNJXT.jpg)

`UPX` 好家伙 手脱ESP定律等方法 我这里快捷直接使用脱壳机

![](https://bbs.kanxue.com/upload/attach/201904/809626_NSNSKEZZ76X2XSW.jpg)

脱壳完成

![](https://bbs.kanxue.com/upload/attach/201904/809626_4H6D5JSSZBCE5WD.jpg)

我下了一些注册表断点发现这个程序居然点进行直接停止了

![](https://bbs.kanxue.com/upload/attach/201904/809626_BT296S7R6DDH9CZ.jpg)

![](https://bbs.kanxue.com/upload/attach/201904/809626_6XD3M6KX6AN3T89.jpg)

个人觉得 要么这个病毒模块的`Resource`中还有`exe`,`dll` 释放到其他进程 要么就是他有反调试

我对于易语言常用的几个反调试的API进行了拦截 都是直接运行程序停止

那么至此 也可以得出结论 这玩意绝对`Resource`中还有别的东西

![](https://bbs.kanxue.com/upload/attach/201904/809626_NGV34PQFYMSGHBP.jpg)

果不其然

![](https://bbs.kanxue.com/upload/attach/201904/809626_U4PXM3GJJVA7WCR.jpg)

通过`EXEinfo`对外挂模块进行提取，发现中间还真有一个`dll`那么对于这个`dll` 开搞之前，先看看这个`exe`是如何把这个`dll`释放到其他进程的,还是做了其他事情

运行时把自身拷贝到`C:\Users\ADMINI~1\AppData\Local\Temp\10412832\TemporaryFile`

![](https://bbs.kanxue.com/upload/attach/201904/809626_WGKPSPN8JWJ9AZE.jpg)

并把自身后缀修改

![](https://bbs.kanxue.com/upload/attach/201904/809626_Y6KV6PBDGZR3A4K.jpg)

运行时修改后缀即可

我们自行加一个后缀`.exe`

![](https://bbs.kanxue.com/upload/attach/201904/809626_7GQ2PVG3CCGCBFK.jpg)

![](https://bbs.kanxue.com/upload/attach/201904/809626_KT4TVX6V2JWUBQW.jpg)

即可看到病毒程序

![](https://bbs.kanxue.com/upload/attach/201904/809626_EAQWYDKEEJMXZ9W.jpg)

获取系统中所有运行中的进程和线程快照

![](https://bbs.kanxue.com/upload/attach/201904/809626_3VVK9JRVP23F7CJ.jpg)

`CreateToolhelp32Snapshot()`获得当前运行进程的快照后，再通过`process32First`函数获得第一个进程句柄

![](https://bbs.kanxue.com/upload/attach/201904/809626_XH4VBTBEPRMDURS.jpg)

通过`Process32Next`获取下一个进程句柄

![](https://bbs.kanxue.com/upload/attach/201904/809626_XSQ73TWWWV7GTSR.jpg)

这里按F9之后 直接停止在此界面 此病毒正在获取我所有的进程快照并通过`Process32Next`获取下一个进程的句柄

通过这个函数往堆栈的上面找可以看到病毒程序获取的进程信息

![](https://bbs.kanxue.com/upload/attach/201904/809626_5X2NPPC28CCWNKQ.jpg)

![](https://bbs.kanxue.com/upload/attach/201904/809626_MWWU7MH6P4SCMQB.jpg)

![](https://bbs.kanxue.com/upload/attach/201904/809626_NGZEX6QCAKPKUDM.jpg)

继续F9运行

![](https://bbs.kanxue.com/upload/attach/201904/809626_75KHE4QGK934USA.jpg)

![](https://bbs.kanxue.com/upload/attach/201904/809626_JQU5WGK89V44GTZ.jpg)

病毒结束

![](https://bbs.kanxue.com/upload/attach/201904/809626_N46UJQJJBWWGPAR.jpg)

我们回到`C:\Users\ADMINI~1\AppData\Local\Temp\10412832\TemporaryFile`

找到那个`TemporaryFile`查看那个病毒拷贝自身的`exe`

我至此还以为这个病毒有什么新颖的地方

![](https://bbs.kanxue.com/upload/attach/201904/809626_A5UTD86HADKZB6M.jpg)

![](https://bbs.kanxue.com/upload/attach/201904/809626_HGQSD9HMUSGUW75.jpg)

### 终究依旧是这样 获取`QQCookie` 然后传回去

![](https://bbs.kanxue.com/upload/attach/201904/809626_ZFANF47W8HA3J8X.jpg)

![](https://bbs.kanxue.com/upload/attach/201904/809626_2WKMHJ5SB9TKC3Y.jpg)

这是关于获取Steam进程后 再获取的授权文件 `ssfn2`这种东西也就是市面上黑号啊 什么的啊那些叫什么91 94授权的东西 写一个文件到steam某目录 然后再登陆黑号 需要登陆的黑号就不再需要邮箱steam令牌等各种验证 可以简单理解为一个身份令牌 有了这个令牌 你的电脑对于steam来说就是可信的 那么也没什么看的 就是正常的获取返回给病毒客户端 为什么我说这个是病毒客户端呢？后面我会找例子说明

![](https://bbs.kanxue.com/upload/attach/201904/809626_K99JQJKK9AE7A6X.jpg)

## 0x02

我之前还说过这个病毒模块里面有个`dll` 那么我们先看看这个`dll`做了什么

`UPX` 好家伙 上脱壳机吧

![](https://bbs.kanxue.com/upload/attach/201904/809626_K48CCB5Q6GJHVBX.jpg)

![](https://bbs.kanxue.com/upload/attach/201904/809626_7FDXJEGTAQQ8H4R.jpg)

![](https://bbs.kanxue.com/upload/attach/201904/809626_J8JCC7JTXW943PU.jpg)

从这个`dll`字符串来看 基本可以判定是获取steam `ssfn`文件的一个模块

![](https://bbs.kanxue.com/upload/attach/201904/809626_Z5VBCWVVHPD4QFN.jpg)

![](https://bbs.kanxue.com/upload/attach/201904/809626_RUMG32YDP6VJNJH.jpg)

具体这些东西是什么`ssfn`等各种steam的问题 请自行研究 。（ps：如出于技术研究是好的，但是盗号等请三思而后行！）

那么我前面说了 为什么我说病毒传回的是客户端呢？ 这就要关系到现在庞大的黑号产业链中萌生的一种新型软件，叫撸号器，顾名思义，盗号就完事了

我找遍了全网很多家专卖外挂的卡盟 终于找到了一家有卖撸号器的（其实卖的很多 只是我运气不好才找到一家）

![](https://bbs.kanxue.com/upload/attach/201904/809626_CTTGUMWKWYTAM95.jpg)

![](https://bbs.kanxue.com/upload/attach/201904/809626_YR7A3ZV85PMJ7Y5.jpg)

说白了也就是这些东西

![](https://bbs.kanxue.com/upload/attach/201904/809626_AEUCU46GDYGBH3J.jpg)

具体撸号器这玩意怎么用的 干嘛的 有啥用 真的能免杀？

![](https://bbs.kanxue.com/upload/attach/201904/809626_EADESF6P8SB7RRH.jpg)

我想用一张关系图表示 流程就是这么个流程

![](https://bbs.kanxue.com/upload/attach/201904/809626_FMWSXJ3DPYG2NEK.jpg)

## 0x03

我下载了目前常用的三款杀毒软件

![](https://bbs.kanxue.com/upload/attach/201904/809626_B246SSF65QGZ8BG.jpg)

虚拟机安装后挨个测试免杀能力（至少我在微步分析这个的时候 360腾讯并未报毒）

![](https://bbs.kanxue.com/upload/attach/201904/809626_ZMZ9UEC24TEB96Y.jpg)

![](https://bbs.kanxue.com/upload/attach/201904/809626_ZJ236HGU8BAJ8M3.jpg)

1.首先是腾讯电脑管家(ps:被扫描的病毒是未脱壳的，是为了模拟正常用户使用情况）

![](https://bbs.kanxue.com/upload/attach/201904/809626_7KRP7CTJCX3DV4X.jpg)

我们把病毒放在桌面新建文件夹`kill me` 用腾讯管家的指定杀毒

![](https://bbs.kanxue.com/upload/attach/201904/809626_8WWYWFYHRUVU6Z3.jpg)

![](https://bbs.kanxue.com/upload/attach/201904/809626_9WUC9YCSF7S7DQ3.jpg)

腾讯安全管家运行情况

![](https://bbs.kanxue.com/upload/attach/201904/809626_Y8CC6HAAEHV5RKN.gif)

未进行拦截（我会把样本放在文章末尾）

![](https://bbs.kanxue.com/upload/attach/201904/809626_F48X2H5TMKQHDAD.jpg)

2.360安全卫士测试情况如下

指定病毒扫描结果如下

![](https://bbs.kanxue.com/upload/attach/201904/809626_W3ABG484SV7FEQU.jpg)

运行结果如下

![](https://bbs.kanxue.com/upload/attach/201904/809626_GEKC5HKE39NAECK.gif)

3.火绒测试结果如下

指定位置扫描结果

![](https://bbs.kanxue.com/upload/attach/201904/809626_WCMZ3FEHG6UJV8A.jpg)

运行结果

![](https://bbs.kanxue.com/upload/attach/201904/809626_NQU9S3S92CVZD4K.jpg)

至此杀软测试已结束

病毒作者服务器：`43.248.186.95`

![](https://bbs.kanxue.com/upload/attach/201904/809626_P6NYURUC9EZHBEQ.jpg)

我联系到了服务器提供商的客服

已提交反馈（至于他们处不处理 那就随他们 我能做的都做了：））

![](https://bbs.kanxue.com/upload/attach/201904/809626_2B7VE72NPTRRUC9.jpg)

![](https://bbs.kanxue.com/upload/attach/201904/809626_ZMB4D2E4JQPNQRJ.jpg)

![](https://bbs.kanxue.com/upload/attach/201904/809626_VQBRPD65SVMU8HW.jpg)

什么？还要我报警？？？？我只做分析 其他的关我啥事啊。。。。。不受理我也没办法咯：）

![](https://bbs.kanxue.com/upload/attach/201904/809626_RSJJJFUH4D5KRJ6.jpg)

对于杀毒软件我还有一点我想说的 不管是腾讯管家 360卫士 火绒 报不报毒不重要 追根溯源 这个东西是捆绑在外挂里面的 用户需求这个东西 就算报毒了 用户也会自行添加白名单信任此程序 另外大概一两个月前 发现某盗号病毒宣传可突破QQ邮箱二级密码等  希望腾讯检查一下是否存在此漏洞 样本下载链接： 262K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2D9j5h3&6\*7L8%4g2K6i4K6u0W2j5$3!0E0i4K6u0r3K9e0y4#2k6Y4W2^5j5b7`.`.

需要具体的外挂程序请发邮件到`3331234895@qq.com`获取 [CatGames的小窝](https://bbs.kanxue.com/elink@c84K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8V1y4S2N6p5N6S2L8h3g2K6i4K6u0W2j5$3^5`.)

GLHF!

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2020-1-31 17:15 被kanxue编辑 ，原因：

[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)
