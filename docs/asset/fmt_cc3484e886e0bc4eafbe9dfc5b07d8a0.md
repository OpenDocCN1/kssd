
# Power Copy 1.92 算法分析及 VC 算法注册机

> 原文：[`www.pediy.com/kssd/pediy11/126755.html`](https://www.pediy.com/kssd/pediy11/126755.html)

【破文标题】Power Copy 1.92 算法分析及 VC 算法注册机  

【破文作者】zaas[PYG]  

【破解工具】`OllyICE`，`PEiD v0.94`  

【破解平台】`WinXP`  

【软件授权】：共享版  

【软件介绍】：对于坏的软盘或光碟中的数据进行强制复制解码，取代 `Windows` 中自带拷贝工具。使您真正拥有强大稳定无敌的拷贝工具。  

【破解声明】我是一只小菜鸟，偶得一点心得，愿与大家分享  

--------------------------------------------------------------  

【破解内容】

查找字符串，有：

地址=`0040B36E`

反汇编=`push 0044112C`

文本字符串=`Software\Youtica\Power Copy`

双击跟进，猜想一下。

代码:

```
0040B36E  |.  68 2C114400   push    0044112C            ;  Software\Youtica\Power Copy
0040B373  |.  E8 E869FFFF   call    00401D60
0040B378  |.  8945 BC       mov     dword ptr [ebp-44], eax
0040B37B  |.  E8 00EBFFFF   call    00409E80            ;  关键 call
0040B380  |.  83C4 0C       add     esp, 0C
0040B383  |.  8845 BB       mov     byte ptr [ebp-45], al        ;  标志位
0040B386  |.  0FB655 BB     movzx   edx, byte ptr [ebp-45]
0040B38A  |.  85D2          test    edx, edx
0040B38C  |.  74 78         je      short 0040B406       ;  跳向死亡
0040B38E  |.  51            push    ecx
0040B38F  |.  8BCC          mov     ecx, esp
0040B391  |.  8965 D4       mov     dword ptr [ebp-2C], esp
0040B394  |.  68 2C114400   push    0044112C            ;  Software\Youtica\Power Copy
0040B399  |.  E8 C269FFFF   call    00401D60            ;  写入注册表
0040B39E  |.  8945 B4       mov     dword ptr [ebp-4C], eax

```

跟进关键 call，核心算法很容易找到。代码:

```
00409F03  |.  E8 88010000   call    0040A090            ;  算法 call
00409F08  |.  8845 9F       mov     byte ptr [ebp-61], al        ;  标志位出来了

```

跟进 `call 0040A090`，大致一看，就像是算法，下断跟跟。程序 `settimer` 了，频繁断下，不怕。

代码:
