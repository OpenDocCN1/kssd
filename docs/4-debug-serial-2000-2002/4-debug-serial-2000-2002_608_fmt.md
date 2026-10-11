
# 电子邮件地址搜索器

> 原文：[`www.pediy.com/kssd/pediy03/forum1074.htm`](https://www.pediy.com/kssd/pediy03/forum1074.htm)

**软件名称**：电子邮件地址搜索器

**软件简介**：收集电子邮件地址的软件。通过本软件收集的电子邮件地址全部是有效的。

**软件下载**：`http://www.csdn.net/cnshare/soft/7/7589.html`

**破解难度**：极易（适合菜鸟练习）

**破解者**：`Edea[BCG]` `QQ：3849036`

先用`Fi`分析，发现该软件没有加壳，为`Delphi`编写。

用`WDASM`反汇编，在反汇编的同时打开软件（呵呵，主要是为了节约时间），填入注册码：`9876543210`，弹出“注册码错误”的对话框。

在反汇编出的代码中我们找到这样一段：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0047DF33(C)
|
:0047DF87 8B55F0                       mov edx, dword ptr [ebp-10]
:0047DF8A 8D45F8                       lea eax, dword ptr [ebp-08]
:0047DF8D E8B25BF8FF                   call 00403B44
:0047DF92 8D55E8                       lea edx, dword ptr [ebp-18]
:0047DF95 8B83DC020000                 mov eax, dword ptr [ebx+000002DC]
:0047DF9B E8981CFBFF                   call 0042FC38
:0047DFA0 8B45E8                       mov eax, dword ptr [ebp-18]
:0047DFA3 8D55EC                       lea edx, dword ptr [ebp-14]
:0047DFA6 E871A2F8FF                   call 0040821C
:0047DFAB 8B55EC                       mov edx, dword ptr [ebp-14]
:0047DFAE 8B45F8                       mov eax, dword ptr [ebp-08]
:0047DFB1 E8865EF8FF                   call 00403E3C
:0047DFB6 7536                         jne 0047DFEE                 ------〉这里一跳就死翘翘了

* Possible StringData Ref from Code Obj ->"注册成功"
                                      |
:0047DFB8 B854E04700                 mov eax, 0047E054
:0047DFBD E87699FDFF                 call 00457938
:0047DFC2 8B45F8                     mov eax, dword ptr [ebp-08]
:0047DFC5 50                         push eax
:0047DFC6 8D45E4                     lea eax, dword ptr [ebp-1C]
:0047DFC9 E8FEA7FEFF                 call 004687CC
:0047DFCE 8B45E4                     mov eax, dword ptr [ebp-1C]

* Possible StringData Ref from Code Obj ->"RegistNo"
                                      |
:0047DFD1 B968E04700                 mov ecx, 0047E068

* Possible StringData Ref from Code Obj ->"Regist"
                                      |
:0047DFD6 BA7CE04700                 mov edx, 0047E07C
:0047DFDB E8DCA4FEFF                 call 004684BC

* Possible StringData Ref from Code Obj ->"已经注册"
                                      |
:0047DFE0 BA8CE04700                 mov edx, 0047E08C
:0047DFE5 8BC3                       mov eax, ebx
:0047DFE7 E87C1CFBFF                 call 0042FC68
:0047DFEC EB18                       jmp 0047E006

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0047DFB6(C)
|
* Possible StringData Ref from Code Obj ->"注册码错误！"
                                      |
:0047DFEE B8A0E04700                 mov eax, 0047E0A0
:0047DFF3 E84099FDFF                 call 00457938
:0047DFF8 8B83DC020000               mov eax, dword ptr [ebx+000002DC]
:0047DFFE 8B10                       mov edx, dword ptr [eax]
:0047E000 FF92B0000000               call dword ptr [edx+000000B0]
************************************************************
```

好了，打开`TRW2000`，下断点`bpx 0047DF87`，再随便填入注册码`9876543210`，点注册，被拦下，我们来到：

```
0177:0047DF87  MOV       EDX,[EBP-10]    ------>在这里我们看到程序把“AXAMBGWH”放入 EDX，很可疑
0177:0047DF8A  LEA       EAX,[EBP-08]                          （你的有可能不是 AXAMBGWH）
0177:0047DF8D  CALL      00403B44
0177:0047DF92  LEA       EDX,[EBP-18]
0177:0047DF95  MOV       EAX,[EBX+02DC]
0177:0047DF9B  CALL      0042FC38
0177:0047DFA0  MOV       EAX,[EBP-18]    ------>9876543210 => EAX
0177:0047DFA3  LEA       EDX,[EBP-14]
0177:0047DFA6  CALL      0040821C
0177:0047DFAB  MOV       EDX,[EBP-14]    ------>9876543210 => EDX
0177:0047DFAE  MOV       EAX,[EBP-08]    ------>AXAMBGWH => EAX
0177:0047DFB1  CALL      00403E3C        ------>跟入
0177:0047DFB6  JNZ       0047DFEE        ------>跳则死
0177:0047DFB8  MOV       EAX,0047E054
0177:0047DFBD  CALL      00457938
0177:0047DFC2  MOV       EAX,[EBP-08]
0177:0047DFC5  PUSH      EAX

Call from 0047DFB1:

0177:00403E39  LEA       EAX,[EAX+00]
0177:00403E3C  PUSH      EBX
0177:00403E3D  PUSH      ESI
0177:00403E3E  PUSH      EDI
0177:00403E3F  MOV       ESI,EAX
0177:00403E41  MOV       EDI,EDX
0177:00403E43  CMP       EAX,EDX       ------>在这里我们看到程序拿 AXAMBGWH 和我的假注册码比较，嘿嘿，剩下的就不用我说了吧
0177:00403E45  JZ        NEAR 00403EDA
0177:00403E4B  TEST      ESI,ESI
0177:00403E4D  JZ        00403EB7
0177:00403E4F  TEST      EDI,EDI
0177:00403E51  JZ        00403EBE
0177:00403E53  MOV       EAX,[ESI-04]
```

我的注册码：`AXAMBGWH`

这个软件的注册码是根据本机安装号码算出来的，所以你的注册码十有八九与我的不同。

注册成功后，软件所在目录下出现一个名为`System.sch`的文件，储存你的注册码。

呵呵，一个星期没破软件了，随便拉一个来练练手。 `4dd466d0e1e0cc123afbe504da2462ca`
