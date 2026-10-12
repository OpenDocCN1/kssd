
# Asp1.2X 的 Stolen code

> 原文：[`www.pediy.com/kssd/pediy06/pediy6664.htm`](https://www.pediy.com/kssd/pediy06/pediy6664.htm)

Advanced WMA Workshop 2.04b 简单脱壳破解

程序用 `asp1.24rc4` 加壳，并用壳的时间保护

作者：lordor

QQ: 88378557

Mail: lordor#163.com

来自：[www.digitalnuke.com](http://www.digitalnuke.com)

说明：`asp1.24RC4` 的 `Stolen code` 的寻找方法，这个方法对于 1.3 以上的是否管用，还在研究中。

自由转载请保

## 一、寻找伪 OEP

用 `od` 载入程序，设置内存不打钩，其它都打钩，隐藏 `od`, 到如下：

```
AWMAWork.>PUSH AWMAWork.00632001  ==>先停在这里，看一下寄存器的值
```

\--------------

```
ECX 0012FFB0
EDX 7FFE0304
EBX 7FFDF000
ESP 0012FFC4  ==>注意这个
EBP 0012FFF0  ==>
ESI 00000024
EDI 00000000
EIP 00401000 AWMAWork.<ModuleEntryPoint>
```

\--------------

经过 26 个 `seh` 后，到这里的特征代码：

```
00CB39EC  XOR DWORD PTR DS:[EAX],EAX  ==>停在这里
00CB39EE  POP DWORD PTR FS:[0]
00CB39F5  POP EAX
00CB39F6  CMP DWORD PTR DS:[CB7EB0],0
00CB39FD  JE SHORT 00CB3A13
00CB39FF  PUSH 0C
00CB3A01  MOV ECX,0CB7EB0
00CB3A06  LEA EAX,DWORD PTR SS:[EBP-8]
00CB3A09  MOV EDX,4
00CB3A0E  CALL 00CB0B40
00CB3A13  PUSH DWORD PTR SS:[EBP-4]
00CB3A16  PUSH DWORD PTR SS:[EBP-8]
00CB3A19  MOV EAX,DWORD PTR SS:[EBP-C]
00CB3A1C  CMP DWORD PTR DS:[EAX],0
00CB3A1F  JE SHORT 00CB3A23
00CB3A21  PUSH DWORD PTR DS:[EAX]
00CB3A23  PUSH DWORD PTR SS:[EBP-10]
00CB3A26  PUSH DWORD PTR SS:[EBP-14]
00CB3A29  RETN      ==>这里下断，shift+F9 运行到这里
```

打开 "Memory Map" 窗口

在 `AWMAWorkshop` 的 "code" 段下 “Set break-on access”(即按 F2)，这样当执行完壳的代码后，再执行解压后代码段的内容就会停下来。

置完后，按 F9，会到这里

```
0047DB94  PUSH AWMAWork.0047DCEC                 ; JMP to MSVCR70._except_handler3
0047DB99  MOV EAX,DWORD PTR FS:[0]
0047DB9F  PUSH EAX
0047DBA0  MOV EAX,DWORD PTR SS:[ESP+10]
0047DBA4  MOV DWORD PTR SS:[ESP+10],EBP
0047DBA8  LEA EBP,DWORD PTR SS:[ESP+10]
0047DBAC  SUB ESP,EAX    ==>这里为 stack 分配局部地址，eax 为压入的立即数
0047DBAE  PUSH EBX    ***
0047DBAF  PUSH ESI    ***   
0047DBB0  PUSH EDI    ***
0047DBB1  MOV EAX,DWORD PTR SS:[EBP-8]
0047DBB4  MOV DWORD PTR SS:[EBP-18],ESP
0047DBB7  PUSH EAX    ***
0047DBB8  MOV EAX,DWORD PTR SS:[EBP-4]
0047DBBB  MOV DWORD PTR SS:[EBP-4],-1
0047DBC2  MOV DWORD PTR SS:[EBP-8],EAX
0047DBC5  LEA EAX,DWORD PTR SS:[EBP-10]
0047DBC8  MOV DWORD PTR FS:[0],EAX
0047DBCE  RETN      ==>停在这里
0047DBCF  MOV ECX,DWORD PTR SS:[EBP-10]
0047DBD2  MOV DWORD PTR FS:[0],ECX
0047DBD9  POP ECX
0047DBDA  POP EDI
0047DBDB  POP ESI
0047DBDC  POP EBX
0047DBDD  LEAVE
0047DBDE  PUSH ECX
```

可以看到这个是 VC7 的第一个 call 的代码，看一下寄存器的值,并注意 *** 上面压入了几句(这里只有四句)，后面会用到：

\------------

```
ECX 0012FFB0
EDX 7FFE0304
EBX 7FFDF000
ESP 0012FF30  ==>注意这里
EBP 0012FFC0   
ESI 00000000
EDI 00000000
EIP 0047DBCE AWMAWork.0047DBCE
```

\------------

F8 一步，到这里

```
0047D8C5  RETN 10
0047D8C8  ADD BYTE PTR DS:[EAX],AL
0047D8CA  ADD BYTE PTR DS:[EAX],AL
0047D8CC  ADD BYTE PTR DS:[EAX],AL
0047D8CE  NOP        ==>花指令，nop 掉
0047D8CF  CALL AWMAWork.0047DB94
0047D8D4  XOR EBX,EBX      ==>到这里，伪 OEP，可以看到上一个 call(0047DB94)已经在壳中运行了
0047D8D6  PUSH EBX
0047D8D7  MOV EDI,DWORD PTR DS:[4820FC]
0047D8DD  CALL EDI
0047D8DF  CMP WORD PTR DS:[EAX],5A4D
0047D8E4  JNZ SHORT AWMAWork.0047D905
0047D8E6  MOV ECX,DWORD PTR DS:[EAX+3C]
0047D8E9  ADD ECX,EAX
```

## 二、寻找 STOLEN CODE，并 DUMP 程序

可以看到 `0047D8C8  ADD BYTE PTR DS:[EAX],AL` 处就是真正的 OEP 了，那里已经抽掉两句代码了，

可能你会问，我是怎么知道的？

找回抽掉的代码的方法一般有：

1. `TC` 跟踪得到
2. `tDasm` 老兄介绍的还原 `stolen code` 的分析法
3. `FLY` 老兄介绍的 `cdi` 猜测法，这个我还是未明白
4. `blowfish` 老兄介绍的建立 `stack` 法，这个方法非常好。

我的方法是在 `blowfish` 方法的基础上再加上：参照同类编译器生成的入口代码。

好，那么我们打开一个 VC7 的程序，看一下入口的代码

```
Service.<>PUSH 74     
```

```
0040270F  PUSH Service.00403BF8
00402714  CALL Service.0040290C
00402719  XOR EBX,EBX      ==>这里以下都是相同的语句
0040271B  MOV DWORD PTR SS:[EBP-20],EBX
0040271E  PUSH EBX                            ; /pModule => NULL
0040271F  MOV EDI,DWORD PTR DS:[<&KERNEL32.GetModu>; |kernel32.GetModuleHandleA
00402725  CALL EDI                            ; \GetModuleHandleA
00402727  CMP WORD PTR DS:[EAX],5A4D
0040272C  JNZ SHORT Service.0040274D
0040272E  MOV ECX,DWORD PTR DS:[EAX+3C]
00402731  ADD ECX,EAX
00402733  CMP DWORD PTR DS:[ECX],4550
00402739  JNZ SHORT Service.0040274D
```

所以关键就是想方法补上抽掉的两句代码。首先可以推测一下这两句的形式：

`push 立即数`

`push 地址`

那么如何那个立即数及地址呢？

这里就要查看 STACK 的内容来确定了。

先看一载入程序时的 `ESP==0012FFC4`，`EBP=0012FFF0`，

再看一下到伪 OEP 后的 STACK 的内容：

```
0012FF34   00000000  ==>现在的 esp
0012FF38   00000000
0012FF3C   7FFDF000
0012FF40   00CA0000
0012FF44   00C80000
0012FF48   0000000C
0012FF4C   00CD1F70
0012FF50   0012FF60
0012FF54   00CCDD95  RETURN to 00CCDD95 from 00CCDDA0
0012FF58   00CCDDF3  RETURN to 00CCDDF3 from 00CCDDFD
0012FF5C   00CCDDD6  RETURN to 00CCDDD6 from 00CCDDE8
0012FF60   00400000  AWMAWork.00400000
0012FF64   C0F1DD1F
0012FF68   0012FFA4
0012FF6C   00CA0000
0012FF70   00C80000
0012FF74   00CB4138
0012FF78   00CD1DA0
0012FF7C   00CCDD50
0012FF80   00400000  AWMAWork.00400000
0012FF84   00632880  AWMAWork.00632880
0012FF88   00CCDE87  RETURN to 00CCDE87 from 00CCDE93
0012FF8C   5FCEDFF7
0012FF90   00000369
0012FF94   0080FD60
0012FF98   FDC7616E
0012FF9C   0012FFF0
0012FFA0   0012FFB0
0012FFA4   00000000
0012FFA8   0012FF34
0012FFAC   0012FFE0
0012FFB0   0012FFE0  Pointer to next SEH record
0012FFB4   0047DCEC  SE handler
0012FFB8   0049E9C0  AWMAWork.0049E9C0
0012FFBC   FFFFFFFF
0012FFC0   0012FFF0  ==>注意这里
0012FFC4   77E5EB69  RETURN to kernel32.77E5EB69
```

其中(0012FFB8   0049E9C0  AWMAWork.0049E9C0)的 `0049E9C0` 即为第二句抽掉的的 push,

现在来确定那个立即数

`0012FFC0-0012FF34=8c`, 这个是我们要找的立即数吗？不是，还要减掉 `4*4` 个字节(注意上面所说的)，即

`8c-4*4=7C`, 这个就是立即数。

所以总结抽掉的代码如下：

```
push 7C
push 0049E9C0
```

注：这可不是常见的 `push ebp` 什么的啊

这两句补完后，长度刚好相等

```
0047D8C8  PUSH 7C
0047D8CA  PUSH AWMAWork.0049E9C0
0047D8CF  CALL AWMAWork.0047DB94
0047D8D4  XOR EBX,EBX
0047D8D6  PUSH EBX
```

把入口修正为 `47D8C8` 后 dump 这个程序。

## 三、修复输入表

运行 `AsprDbgr_build_106` 程序，一路确定，在提示伪 OEP 后，运行 `ImportREC1.6` 修复输入表，在 OEP 中输入 `7d8c8`, 会找到全部的 api, `Fixdump` 即可

脱完壳后，程序的 nag 提示都没有了

lordor 总结：

寻找 OEP 及修复输入表，都是极简单，可以用秒杀，但在修复 Stolen code 时我花了不少时间，参考很多文章，对 blowfish 老兄的方法觉得更有

价值，但操作很烦，后来我想到根据入口代码的情况，对照相应编译器未加壳的程序来寻找 Stolen code, 这个方法更简单实用。

By lordor 04.7.11

`4dd466d0e1e0cc123afbe504da2462ca`
