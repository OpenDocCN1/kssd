
# TRW2k1.23 除 Bug 记

> 原文：[`www.pediy.com/kssd/pediy03/forum1139.htm`](https://www.pediy.com/kssd/pediy03/forum1139.htm)

## 【声明】

我写这篇文章是以技术交流为主，希望大家在转载时能保持文章的完整性

相信玩破解的朋友都知道 `Trw2000`。（废话）。`Trw2000` 和 `ICE` 相比，体积小。使用时不用重新起到计算机。还可以脱壳。等等诸多优点。但是，稳定性差强人意。对硬件要求高。对很多的硬件不兼容！

这不小弟的电脑就属于那种不能使用 `trw2000`。显示卡不支持!!!!呜.....呜.....痛苦啊！

先看看小弟的机器配置(是不是惨不能睹)

```
CPU    Pentium MMX233   

内  存  64MB   

主  版  华硕 TX97-E   

显示卡  创新 Graphics Blaster Exxtreme 4MB (Permedia 2V 显示芯片)  <<--就是他惹的祸！
```

情况是这样的，运行 `Trw2k1.23` 后，计算机的按键盘、移到鼠标都没有反应。同死机了一样。`Ctrl+Alt+Del` 都不能重新启动计算机。必需按机箱上的 `RESET`。才可以重新启动计算机。

如果在一开始就载入了 `ICE` 的话，这时 `Ctrl+D` 进入 `S-ice` 可以发现在下面。

```
EAX=000B8000  EBX=00000000  ECX=CA76AE1C  EDX=000003DA  ESI=000B8000      


EDI=C1773E60  EBP=CA76AE10  ESP=CA76ADF0  EIP=C18FB458  o d I s Z a P c   


CS=0028  DS=0030  SS=0030  ES=0030  FS=0078  GS=0030                       


\==================================================byte==============PROT==?0)==


0030:00000000 CC 00 FF 2F 2B 04 70 00-C3 E2 00 F0 2B 04 70 00  .../+.p.....+.p.


0030:00000010 2B 04 70 00 54 FF 00 F0-08 80 00 F0 6F EF 00 F0  +.p.T.......o...


0030:00000020 00 00 00 C8 87 E9 00 F0-6F EF 00 F0 6F EF 00 F0  ........o...o...


0030:00000030 6F EF 00 F0 6F EF 00 F0-57 EF 00 F0 6F EF 00 F0  o...o...W...o...


\==========================================================================ROT32


0028:C18FB42D  ADD      \[EAX\],AL                                                   


0028:C18FB42F  PUSHAD                                                         


0028:C18FB430  MOV      \[C193EE00\],ESI                                     


0028:C18FB436  MOV      \[C193EE04\],EDI                                     


0028:C18FB43C  MOV      \[C193EDF8\],EAX                                     


0028:C18FB441  CALL     C191ACC4                                            


0028:C18FB446  POPAD                                                           


0028:C18FB447  RET                                                             


0028:C18FB448  PUSH     EDX                                                   


0028:C18FB449  PUSH     EAX                                                   


0028:C18FB44A  MOV      EDX,000003D4                                         


0028:C18FB44F  ADD      EDX,06                                                


0028:C18FB452  IN        AL,DX                                                  


0028:C18FB453  TEST      AL,01                                                  


0028:C18FB455  JNZ       C18FB452                                                 


0028:C18FB457  IN        AL,DX                   <--|al\=0 跳不出这个循环             


0028:C18FB458  TEST      AL,01     <<--光标停在此处。    |形成死循环。             


0028:C18FB45A  JZ        C18FB457                <--|这是造成计算机的原因.


0028:C18FB45C  POP       EAX                                                 


0028:C18FB45D  POP       EDX                                                 


0028:C18FB45E  RET                                                             


0028:C18FB45F  PUSH      EBP                                                     


0028:C18FB460  MOV       EBP,ESP                                                 


0028:C18FB462  PUSH      EDX                                                     


0028:C18FB463  PUSH      EBX                                                     


\====================================TRW2000(01)+DF6D============================


:pagein n 1.txt                                                                


:pagein n 2.txt                                                                


PAGEIN address                                                                Trw2000
```

解决的方法:`0028:C18FB45A  JNZ      C18FB457`

这个方法同样适用于 `trw2000` 以前的各个版本.但是只能适用于在创新 `Graphics Blaster Exxtreme 4MB (Permedia 2V 显示芯片)`上使用，不能在其他显示卡使用。不然的话同样会死机。

xiaqin

China Cracking Group

2001.06.15 `4dd466d0e1e0cc123afbe504da2462ca`
