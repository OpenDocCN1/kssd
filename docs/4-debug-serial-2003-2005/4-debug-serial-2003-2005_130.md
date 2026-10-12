
# File Shredder 3.7

> 原文：[`www.pediy.com/kssd/pediy06/pediy6293.htm`](https://www.pediy.com/kssd/pediy06/pediy6293.htm)

再来一篇 `OLLYDBG` 的万能断点-----`File Shredder 3.7` 的算法探析

## 软件介绍

*   软件名称：`File Shredder 3.7`
*   软件大小：`105KB`
*   软件语言：英文
*   软件类别：国外软件/共享版/卸载工具
*   运行环境：`Win9x/Me/NT/2000/XP`
*   加入时间：`2003-10-8 16:05:51`
*   下载次数：`525`
*   软件评级：
*   联系人：未知
*   开发商：`Home Page`
*   软件描述：文件彻底删除工具，直接在资源管理器点击右键选择相应选项即可彻底删除文件，可选择对其进行多遍复写，以达到永远无法恢复的目的。
*   下载地址：[`http://www.onlinedown.net/soft/10694.htm`](http://www.onlinedown.net/soft/10694.htm)
*   破解作者：`yzez[DFCG]`
*   破解工具：`ODBG109B`
*   破解环境：`WINDOWS XP` 操作系统
*   破解目的：不为破解而破解,只因为技术而破解

## 破解过程

1. 只使用了查壳工具，没有使用其他工具，经查无壳，`VC` 编程。用 `OD` 载入程序，在最下边的命令框里输入：`bpx hmemcpy` 万能断点,按 `Enter` 键，出现互相调变的 `CALL` 对话框，在框里右键点击，在所有的 `CALL` 上都设置好断点，再按 `F9`，也可以边按 `F9`，边按 `F2` 去掉无用的断点，到出现注册信息框后，输入任意注册信息。我输入，用户名：`yzez`，用户单位：`yzez[DFCG]`，试验码：`12345678`，点确定程序被断在 `004059BB` 处，按 `F8` 往下走。

```
004059BB  CALL    DWORD PTR DS:[<&USER32.GetDlgItem>]  *****点注册确定后我们中断在此，按 F
004059C1  POP     EDI                           *****8 往下走，因为还没有到核心段！
004059C2  POP     ESI                           *****注意在这个断点会返回三次！
004059C3  MOV     EAX, 1
004059C8  POP     EBX
004059C9  RETN******************************在这里程序返回，第三次后返回的代码如下

==================================================================================
00402264  MOV     AL, BYTE PTR DS:[ESI]*****第三次后程序返回到这里！把用户名的第一位 y
****************************************送入 AL，AL 的值是 79 即 y 的 ASCII 码值！
00402266  ADD     ESP, 30
00402269  TEST    AL, AL************************测试是否为 0
0040226B  JNZ     SHORT Shredder.0040228B***不为 0 往下跳！
0040226D  CMP     BYTE PTR DS:[EDI], 0
00402270  JNZ     SHORT Shredder.0040228B
00402272  MOV     BYTE PTR DS:[ESI], 0
00402275  MOV     BYTE PTR DS:[EDI], 0
00402278  MOV     DWORD PTR DS:[ESI+64], 0
0040227F  MOV     DWORD PTR DS:[ESI+68], 1
00402286  JMP     Shredder.00402334
0040228B  LEA     EAX, DWORD PTR SS:[ESP+10]**上面跳到此处！赋试验码的地址值给 EAX
0040228F  PUSH    EAX*************************试验码入栈！
00402290  CALL    Shredder.0040CE90*****此 CALL 把试验码转换成十六进制值，
****************************即 12345678 转换成十六进制值是：BC614E
00402295  PUSH    ESI****************用户名：yzez 入栈！
00402296  MOV     EBX, EAX*****转换成十六进制值的试验码保存到 EBX 中
00402298  CALL    Shredder.00405920***对用户名运算，得到的值是由 3811A，代码看后面
0040229D  ADD     ESP, 8
004022A0  CMP     EAX, 119A792******比较 93811A 与 119A792
004022A5  JNZ     SHORT Shredder.004022BF****不相等就跳！
004022A7  MOV     EBX, DWORD PTR DS:[<&KERNEL32.ls
004022AD  PUSH    Shredder.00418084
004022B2  PUSH    ESI
004022B3  CALL    EBX
004022B5  PUSH    Shredder.00418074
004022BA  PUSH    EDI
004022BB  CALL    EBX
004022BD  JMP     SHORT Shredder.004022C6
004022BF  CMP     EAX, 0D5FCE3C********跳到这里比较
004022C4  JNZ     SHORT Shredder.004022D2****不相等再跳！
004022C6  PUSH    EDI
004022C7  PUSH    ESI
004022C8  CALL    Shredder.004056A0
004022CD  ADD     ESP, 8
004022D0  MOV     EBX, EAX
004022D2  PUSH    EDI****************跳到这里
004022D3  PUSH    ESI
004022D4  CALL    Shredder.004056A0*****关键 CALL 按 F7 跟进！
004022D9  ADD     ESP, 8
004022DC  CMP     EBX, EAX********EAX 是 97C02543，EAX 是试验码：BC614E
004022DE  JE      SHORT Shredder.004022FE*相等就跳，一跳就恭喜了，注册成功！
004022E0  PUSH    0EACF
004022E5  PUSH    1388
004022EA  PUSH    EBP
004022EB  CALL    Shredder.004056D0
004022F0  ADD     ESP, 0C
004022F3  POP     EDI
004022F4  POP     ESI                          ;  Shredder.0041EE38
004022F5  POP     EBP
004022F6  POP     EBX
004022F7  ADD     ESP, 100
004022FD  RETN

=================================================================================

******************关键 CALL**********************************************************
004056A0  MOV     EAX, DWORD PTR SS:[ESP+4]               ;  Shredder.0041EE38
004056A4  PUSH    ESI
004056A5  MOV     ESI, DWORD PTR DS:[424358]****424358 地址处存放的值 95989598 入 ESI
004056AB  PUSH    EAX
004056AC  OR      ESI, 378****************************ESI OR 378=95989598 OR 378=959897F8
004056B2  CALL    Shredder.00405920*****算法 CALL（1），对用户名运算，按 F7 跟进！
004056B7  MOV     ECX, DWORD PTR SS:[ESP+10]
004056BB  ADD     ESI, EAX***********ESI=ESI+EAX=959897F8+93811A=962C1912
004056BD  PUSH    ECX
004056BE  CALL    Shredder.00405920*****算法 CALL（2）再对用户单位同上运算，得到
*****************************的值是：1940C31
004056C3  ADD     ESP, 8
004056C6  ADD     EAX, ESI***********EAX=ESI+EAX=959897F8+1940C31=97C02543
*****************************把它转化成十进制值就是我们要找的注册码！
004056C8  POP     ESI                          ;  Shredder.0041EE38
004056C9  RETN

==================================================================================
********算法 CALL 的代码！**********************************************************
00405920  PUSH    ECX****跟进算法 CALL 后我们停在这里！
00405921  PUSH    EBX
00405922  MOV     EBX, DWORD PTR SS:[ESP+C]
00405926  PUSH    ESI
00405927  XOR     ESI, ESI
00405929  PUSH    EBX************************用户名入栈！
0040592A  MOV     DWORD PTR SS:[ESP+C], ESI
0040592E  CALL    DWORD PTR DS:[<&KERNEL32.lstrlen>**取得用户名的位数：4
00405934  TEST    EBX, EBX************************测试用户名输入了吗？
00405936  JE      SHORT Shredder.00405987******没有输入就跳走，跳就失败
00405938  TEST    EAX, EAX
0040593A  JE      SHORT Shredder.00405987
0040593C  XOR     EDX, EDX************************EDX 清 0
0040593E  TEST    EAX, EAX
00405940  JLE     SHORT Shredder.00405987
00405942  PUSH    EBP
00405943  PUSH    EDI***********用户单位：yzez[DFCG]入栈
00405944  MOV     ESI, Shredder.0041972C* "|b!pz*ls;rn|lf$vi^Axpe)rx5aic&          9/2m5lsi4@0dmZw94cmqpfhw"这是软件内置的一张表！把这张表移入 ESI
00405949  MOV     EDI, 1**************************赋 EDI 的值为 1
0040594E  SUB     ESI, EBX******************ESI=ESI-EBX，即减去用户名的位数 4
00405950  MOV     ECX, EBX*******************用户名移入 ECX 中
00405952  SUB     EDI, EBX*******************EDI 减去 EBX
00405954  MOVSX   EBX, BYTE PTR DS:[ESI+ECX]*密码表中的第一位“|”的 HEX 值 7C 扩展到 EBX
00405958  MOVSX   EBP, BYTE PTR DS:[EAX+EDX+4196F4]*EAX+EDX+4196F4=4196F8 存放的值 B
**************************的 ASCII 码值 42 扩展到 EBP
00405960  IMUL    EBX, EBP***EBX=EBX*EBP=7C*42=1FF8
00405963  LEA     EBP, DWORD PTR DS:[EDI+ECX]**移入 1 到 EBP，可能是其位数第一位！
00405966  IMUL    EBX, EBP******EBX=EBX*EBP=1FF8*1=1FF8
00405969  MOVSX   EBP, BYTE PTR DS:[ECX]*用户名的第一位“y”的 HEX 值 79 扩展到 EBP
```



```
0040596C  IMUL    EBX, EBP****EBX=EBX*EBP=1FF8*79=F1C38
0040596F  MOV     EBP, DWORD PTR SS:[ESP+10]**赋 EBP 的值为 0
00405973  ADD     EBP, EBX*****EBP=EBP+EBX，第一次循环 EBP 的值为 F1C38
00405975  INC     EDX******EDX 加 1
00405976  INC     ECX******ECX 减 1
00405977  CMP     EDX, EAX****比较 1 和 4
00405979  MOV     DWORD PTR SS:[ESP+10], EBP***保存值 F1C38
0040597D  JL      SHORT Shredder.00405954*****小于就跳，循环！最后的值是：65BF00，***再把上述的值相加即得到：93811A
0040597F  MOV     EAX, EBP***把 93811A 移入 EAX！
00405981  POP     EDI
00405982  POP     EBP
00405983  POP     ESI
00405984  POP     EBX
00405985  POP     ECX
00405986  RETN**************************************这个子程序结束，我们回到上面！
```

附上密码表：

```
004196F1  25 64 00 23 73 65 72 42  %d.#serB
004196F9  26 6E 7A 7C 6D 66 4D 31  &nz|mfM1
00419701  2F 35 28 21 73 64 24 4D  /5(!sd$M
00419709  71 2E 7B 73 5D 2B 73 46  q.{s]+sF
00419711  6A 74 4B 70 7A 53 64 74  jtKpzSdt
00419719  7A 6F 58 71 6D 62 5E 41  zoXqmb^A
00419721  6C 40 64 76 3A 73 3F 78  l@dv:s?x
00419729  2F 00 00 7C 62 21 70 7A  /..|b!pz
00419731  2A 6C 73 3B 72 6E 7C 6C  *ls;rn|l
00419739  66 24 76 69 5E 41 78 70  f$vi^Axp
00419741  65 29 72 78 35 61 69 63  e)rx5aic
00419749  26 39 2F 32 6D 35 6C 73  &9/2m5ls
00419751  69 34 40 30 64 6D 5A 77  i4@0dmZw
00419759  39 34 63 6D 71 70 66 68  94cmqpfh
00419761  77                      w
```

一个可用的注册码：用户名：`yzez`，用户单位：`yzez[DFCG]`，注册码是：`2545952067`

`4dd466d0e1e0cc123afbe504da2462ca`
