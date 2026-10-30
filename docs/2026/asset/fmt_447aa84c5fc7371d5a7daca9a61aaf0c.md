
# [原创] CTF2017 第8题 loudy Crackme 破解过程（迟来的成功）

## 序言：

这几天有点感冒，看反汇编代码就头晕，看不下去了休息、休息浪费了一天比赛时间。算法在比赛结束前已经搞得清清楚楚，只可惜没来得及用`python`写个高效率穷举的程序。在群里发誓要把它破出来，所以现在补上破文，以免浪费了时间和精力。对第三段注册码的穷举我使用了彩虹表（索引）的办法，弥补了`python`执行效率低的问题，脑洞大开，代码都贴上了，希望思路可以供今后借鉴。

本题作者构思全面，涵盖了TLS、虚拟机、内存校验、SMC、多线程、密码学算法（RSA）、大数运算等反调试、反分析手段。难度算高的了，也许作者是奔着防御组冠军去的，如果对算法第三段部分再进行严谨构思，应该算很棒的作品了。

按以往惯例，文章末尾都会总结一下，本文破例，就写在前面了，需要建议各位新手的是对付这样的算法很复杂、流程很乱的作品，应合理使用`OD`的`RUN`跟踪功能，记录执行过的每条重要指令和跳转，动态记录注释信息和寄存器内容，随时重放函数和执行流程，助你准确调试分析，动态图演示一下：

[![](https://bbs.kanxue.com/upload/attach/201706/52087_mvuu4pbsydbhbic.gif)](https://bbs.kanxue.com/upload/attach/201706/52087_mvuu4pbsydbhbic.gif)

## 一、查壳、PE格式，了解作品（套路了）

//查壳 Microsoft Visual C/C++(2005)[`msvcrt`]

//算法侦查：Big number：406490  Referenced at 40213C-->提示了作品可能包含大数运算

//PE格式检查：TLS 3 FUNC：402D00 402D20 402D30 -->提示了作品可能利用TLS反调试

//双击运行查看界面，命令行模式，输入错误注册码，弹出对话框，运行时间略久，怀疑计算量不少

![](https://bbs.kanxue.com/upload/attach/201706/52087_mixmgcugcak3mtw.png)

## 二、TLS反调试

```
//TLS 1 反调试：
public TlsCallback_0

TlsCallback_0 proc near

var_4= dword ptr -4

push    ecx
push    eax
mov     eax, large fs:30h
movzx   eax, byte ptr [eax+2]
mov     [esp+8+var_4], eax
pop     eax
pop     ecx
retn    0Ch

TlsCallback_0 endp
```

```
//TLS 2 反调试
public TlsCallback_1

TlsCallback_1 proc near

call    sub_401000
retn    0Ch

TlsCallback_1 endp
```

```
//TLS 3 反调试
public TlsCallback_2

TlsCallback_2 proc near

arg_4= dword ptr  8
cmp     [esp+arg_4], 1
jnz     short locret_402D3C
call    sub_402C70
locret_402D3C:
retn    0Ch

TlsCallback_2 endp
```
//`OD`载入，断TLS，勾选隐藏`OD`插件配置，TLS跟踪一遍发现对开了插件的`OD`没有影响，开了很多线程，比较卡。

## 三、主流程、VM分析

粗略翻一下代码，找到`402D40`关键函数，在`402D40`下断

```
00402D40    55              PUSH EBP
00402D41    8BEC            MOV EBP,ESP
00402D43    83E4 F8         AND ESP,0xFFFFFFF8
00402D46    81EC AC010000   SUB ESP,0x1AC
00402D4C    A1 00604000     MOV EAX,DWORD PTR DS:[0x406000]
```

很快就能跟到虚拟机部分：

![](https://bbs.kanxue.com/upload/attach/201706/52087_zkp34kuc412fvom.png)

//`ECX`指向了下表`Array1`

```
0040355F    3A11            CMP DL,BYTE PTR DS:[ECX]
```

```
//Array1
0012FEE8  18 0D 16 16 45 0D 02 11 49 03 18 4C 03 01 1B 50  .E.ILP
0012FEF8  03 1B 14 1C 01 57 93 7C 18 0D 16 44 02 09 13 48  W搢.D.H
0012FF08  1D 02 0E 4C 1F 07 08 18 05 52 18 11 0C 57 00 00  LR.W..
0012FF18  AA 15 20 01 00 00 AA 15 40 01 00 00 A0 10 00 00  ? ..?@..?.. 
0012FF28  00 00 A8 A0 10 F0 00 00 00 A8 A0 10 60 01 00 00  ..?..`. 
0012FF38  A7 AA 11 80 00 00 00 AA 10 60 00 00 00 AA 12 B0  И€...?`...? 
0012FF48  00 00 00 A9 A2 EA A6 0E A0 10 20 01 00 00 A0 11  ...軎? ..? 
0012FF58  10 01 00 00 A4 A5 A0 10 40 01 00 00 A0 11 10 01  ..ぅ?@..? 
0012FF68  00 00 A4 A5
```

//`ESI`指向了下表`Array2`，`[ESI+0x14]`指向了`EIP`，`EIP`包含操作数同时又指向了函数入口

```
00403576    8B4E 14         MOV ECX,DWORD PTR DS:[ESI+0x14]                   ; 貌似取虚拟指令
```

```
//Array2
0012FDD0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0012FDE0  00 00 00 00 1E FF 12 00 A0 00 00 00 C0 34 40 00  .....?..?@.
0012FDF0  A1 00 00 00 C0 33 40 00 A2 00 00 00 D0 33 40 00  ?..?@.?..?@.
0012FE00  A4 00 00 00 40 34 40 00 A5 00 00 00 70 34 40 00  ?..@4@.?..p4@.
0012FE10  A3 00 00 00 30 34 40 00 A6 00 00 00 00 34 40 00  ?..04@.?...4@.
0012FE20  A7 00 00 00 80 34 40 00 A8 00 00 00 A0 34 40 00  ?..€4@.?..?@.
0012FE30  A9 00 00 00 00 27 40 00 AA 00 00 00 20 27 40 00  ?...'@.?.. '@.
```

//`Array2`表整理如下：

```
A0 —— 4034C0
A1 —— 4033C0
A2 —— 4033D0
A4 —— 403440
A5 —— 403470
A3 —— 403430
A6 —— 403400
A7 —— 403480
A8 —— 4034A0
A9 —— 402700
AA —— 402720
```

//`Array1`表整理如下：
