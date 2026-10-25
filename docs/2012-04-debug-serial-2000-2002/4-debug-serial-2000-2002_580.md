
# 博奥彩票

> 原文：[`www.pediy.com/kssd/pediy03/forum480.htm`](https://www.pediy.com/kssd/pediy03/forum480.htm)

破解软件：博奥彩票分析系统千禧版

下载 URL：`http://www.boao.com.cn/lottery/download/rtsetup.exe`

软件大小：`3594531 byte`

分析：该软件也是注册码保护。每次启动时调用：`c:\\program file\\rotate\\rotate.ini`

文件中输入的注册码是否正确？

启动 TRW，打开：帮助=》注册=》注册码，你会看见注册框里有随机码如下：

`79879967780002577-`

把它们删除掉，然后输入`'79797979'`，因为我们要使用 79 大法了。

下中断：`bpx hmemcpy`，被中断后，开始按 F10 跟踪，一直按到此：

```
15F:54E266    CALL 66AF08
15F:54E26B    POP ECX
15F:54E26C    TEST CL,CL
15F:54E26E    JZ 54E51E
```

当`CL`为 0 时，则跳走，也就是注册会失败。所以从此处抬头望青天，见到什么，对：

```
15F:54E24F    LEA EAX,[EBP-40]
15F:54E252    LEA EDX,[EBP-04]
15F:54E255    CALL 66AFEC    =>按 F8 键进入此 CALL 观摩！
```

============================

```
15F:66AFEC    PUSH EBP
15F:66AFED    MOV EBP,ESP
15F:66AFEF    PUSH EBX
15F:66AFF0    MOV EAX,[EAX]    =>用 D 命令可以看见什么，好熟悉，对是 79797979
15F:66AFF2    MOV EDX,[EDX]    =>用 D 命令可以看见真正的注册码了，
```

赶快用笔记下来吧：`5484-8430-1111-578`

OK，破解完工！耗时 5 分钟，1 分钟整理！

2000 年 12 月 29 日 19:54

coolfly

`http://coolfly.126.com`  `4dd466d0e1e0cc123afbe504da2462ca`
