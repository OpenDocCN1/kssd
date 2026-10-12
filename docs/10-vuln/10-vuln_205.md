
# Mediacoder 0.7.54792 Buffer Overflow Exploit(SEH)漏洞分析

> 原文：[`www.pediy.com/kssd/pediy12/125829.html`](https://www.pediy.com/kssd/pediy12/125829.html)

12 月 1 号从 http://www.exploit-db.com/exploits/15630/ 下了 11 月 29 号的一个 poc 简单分析了下，不足的地方望大家多拍砖~~~![](img/c1d8523a7842d7b67917c210ac6dcf18.jpg "微笑")

## Exploit 信息

#Exploit Title: Mediacode 0.7.5.4792 Buffer Overflow Exploit(SEH)
#Software Link: http://www.mediacoderhq.com/mirrors.htm?file=MediaCoder-0.7.5.4792.exe
#Origianl exploit by 0v3r
#Modified by bestshow
#Test on Windows Xp xp3 CN
#Date: 2010/12/01-2010/12/02

## 测试效果

添加生成的文件“evil.lst”

![](img/8f9b1b2b8b5fcca22714610f16b5d15b.jpg)

鼠标选中该文件则触发计算器：

![](img/e758a1815e473e4eab2c09cb641c9c84.jpg)

## 分析过程

主要是为了分析触发过程，所以换了个简单的弹出计算器的 shellcode。

载入触发文件后，遇到异常，观察此时栈中的情况：

![](img/15eca8499b1ab141078086f6c6a60063.jpg)

此时的 `eip` = `41414141`，`esp` = `0012f2bc`

## 上传的附件

| ![](img/b7e61e3627e1e92d7b948dd8fdd24fa6.jpg) || Mediacoder 0.7.54792 Buffer Overflow Exploit(SEH)分析报告.doc |

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || poc_1.rar |
