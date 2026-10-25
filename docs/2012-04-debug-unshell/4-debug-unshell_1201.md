
# ZProtect IAT 修复分析

> 原文：[`www.pediy.com/kssd/pediy10/82665.html`](https://www.pediy.com/kssd/pediy10/82665.html)

手上也有正版的 `ZProtect` 了，就加了几个样本玩儿了一下，总结出了一些小方法，分享给大家。

本文的目的旨在讨论如何去除 `ZP` 保护的程序（不带 SDK）。主要讨论一下如何修复 `IAT`。

第一个样品是个 `VB` 程序，按默认保护了输入表。

## ZP 中的输入表保护方式

`ZP` 中的输入表的保护方式有三种：

1. 加密等级 1
2. 加密等级 2
3. 模拟

还有一个额外的输入表保护方式 `Anti-Hook`，这里不讨论修复这种保护。

默认保护下，`IAT` 中的一部分函数使用的是等级 1，剩下的使用的是等级 2。至于模拟这种保护，貌似在 `ZP1.44` 中是被动选择。只能对 `GetModuleHandleA` 这个函数使用。知道了这个，修复这个函数也就不在话下。

## 探索壳代码

闲话少说，`OD` 载入，配置好 `StrongOD`，确保程序能在 `OD` 中运行，这样会减少我们不少麻烦。

`F7` 单步走几步，探索一下壳的代码。

走了一段时间就会看清楚 `ZP` 的代码变形引擎的本质：

`ZP` 壳会把程序的代码按指令分割起来，然后用 `Jmp` 连接起来。

当然如果仅仅是这样的话，反跟踪的难度绝对不够的。还要对 `Jmp` 进行变形。

这里用下面的例子说明一下：

```assembly
006E1ACF    E8 02000000     call Farsight.006E1AD6
006E1AD4    6C             ins byte ptr es:[edi],dx---这两个是花指令
006E1AD5    19             ---
006E1AD6    872C24         xchg dword ptr ss:[esp],ebp
006E1AD9    8DAD B0F9FFFF lea ebp,dword ptr ss:[ebp-650]
006E1ADF    872C24         xchg dword ptr ss:[esp],ebp
006E1AE2    C3             retn
```

上面的代码就相当于 `Jmp 006E1A6F`

原理是这样：

`call Farsight.006E1AD6` 这条指令执行完以后会把它下面指令的地址压入堆栈，所以现在 `[ESP]=006E1AD4`。

然后 `xchg dword ptr ss:[esp],ebp` 把 `006E1AD4` 传递给 `ebp`。

接着 `lea ebp,dword ptr ss:[ebp-650]` 把 `ebp-650=006E1A6F`

在赋值给 `esp`，最后 `retn` 返回就返回到 `006E1A6F` 了。

这段代码的二进制是：

```text
E8 02 00 00 00 27 EB 87 0C 24 8D 89 63 01 00 00 87 0C 24 C3
```

`E8 02 00 00 00` 这个是 `call`  
`87 0C 24 8D 89 63 01 00 00 87 0C 24 C3`  
`27 EB` 是花指令，个数不确定，一般是 `0-2` 个  
`87 0C 24` 其中的 `0C` 决定使用的寄存器。  
`8D 89 63 01 00 00` 其中 `89` 表示寄存器，后面的四个字节就是相应的差。  

有一定编程能力的人，相信应该能写出去掉代码变形的工具。

## 到达 OEP 的方法

弄清楚代码变形的过程就不看在代码里面晃了，实在很痛苦。下面说一个通用的到达 `OEP` 的方法：

重新载入程序，然后 `F7` 走，一直看到 `Pushad` 这条指令以后，走过去，使用 ESP 定律，运行，停下来以后，走几步就到 `OEP` 了！

这个方法通用 `1.3x-1.4x` 保护的程序。其他版本没验证过…… 从这点来看，`ZP` 在技术上还只算一个压缩壳……

## 修复 IAT

好了，到了 `OEP`，下面开始修复 `IAT`。

```assembly
00412940  - E9 DBB73300     jmp manager.0074E120
00412945    90             nop
00412946  - E9 11BB3300     jmp manager.0074E45C
0041294B    90             nop
0041294C  - E9 FFBD3300     jmp manager.0074E750
00412951    90             nop
00412952  - E9 65B83300     jmp manager.0074E1BC
00412957    90             nop
00412958  - E9 7FB93300     jmp manager.0074E2DC
0041295D    90             nop
0041295E  - E9 D9B63300     jmp manager.0074E03C
00412963    90             nop
00412964  - E9 27B73300     jmp manager.0074E090
00412F0A  - E9 F5B83300     jmp manager.0074E804
00412F0F    90             nop
00412F10    68 7C5B4100     push manager.00415B7C              ; <---OEP
00412F15    E8 F0FFFFFF     call manager.00412F0A    VB 的程序这个 call 应该是 jmp.&msvbvm60.ThunRTMain 这个函数。
```

看来这处 `IAT` 调用 `00412F0A - E9 F5B83300 jmp manager.0074E804` 应该是跳转到 `ThunRTMain` 这个函数的。

我们来仔细跟踪到这里看一下，下面是去掉了 `jmp` 和变型以后的代码：

```assembly
0074E804    68 14E693AD     push AD93E614 《---注意这个值，这就是 IAT 调用的 jmp 执行以后的第一条指令
01504660    68 F9E1932D     push 2D93E1F9
01526465    9C             pushfd
01526466    E8 01000000     call 0152646C
0152646B    97             xchg eax,edi
0152646C    871424         xchg dword ptr ss:[esp],edx
0152646F    8D92 3AD3FFFF lea edx,dword ptr ds:[edx-2CC6]  === jmp 0015237A5
01526475    871424         xchg dword ptr ss:[esp],edx
01526478    C3             retn

015237A5    83EC 3E        sub esp,3E
0151D99C    896424 2C      mov dword ptr ss:[esp+2C],esp
01521064    894424 28      mov dword ptr ss:[esp+28],eax
0151BDBC    8B4424 42      mov eax,dword ptr ss:[esp+42]
0151CBF6    894424 36      mov dword ptr ss:[esp+36],eax
015212DE    8B4424 3E      mov eax,dword ptr ss:[esp+3E]
015298E8    894424 1C      mov dword ptr ss:[esp+1C],eax
0151F090    894C24 18      mov dword ptr ss:[esp+18],ecx
jmp 1524BD1

01524BD1    895424 24      mov dword ptr ss:[esp+24],edx
0151BFC3    891C24         mov dword ptr ss:[esp],ebx
jmp 151EAF8

0151EAF8    896C24 0C      mov dword ptr ss:[esp+C],ebp
01526611    897424 14      mov dword ptr ss:[esp+14],esi
015202C0    897C24 04      mov dword ptr ss:[esp+4],edi
jmp 151B386

0151B386    834424 2C 46   add dword ptr ss:[esp+2C],46
01522D55    54             push esp
jmp 151F0FB

0151F0FB    55             push ebp
0152390F    8BEC           mov ebp,esp
0152765C    83E4 F8        and esp,FFFFFFF8
0151DA6C    83EC 0C        sub esp,0C
015233A9    53             push ebx
0152823D    56             push esi
0151D68B    57             push edi
0151EBE7    68 BC945101    push 15194BC
01522654    FF15 98C65001  call dword ptr ds:[150C698]      ; ntdll.RtlEnterCriticalSection
01528791    8B0D F89A5101  mov ecx,dword ptr ds:[1519AF8]
015215F5    33F6           xor esi,esi
0152661B    90             nop
jmp 15283A9

015283A9    85C9           test ecx,ecx
015223B6   /0F84 C8510000 je 01527584
0151D8A3    A1 FC9A5101    mov eax,dword ptr ds:[1519AFC]
0151ECC7    2BC1           sub eax,ecx
jmp 15265EF

015265EF    C1F8 03        sar eax,3
0152221A    3BF0           cmp esi,eax
jmp 152905C

0152905C  ^\0F83 22E5FFFF jnb 01527584
0152065A    833CF1 00      cmp dword ptr ds:[ecx+esi*8],0
015244E0    8D3CF1         lea edi,dword ptr ds:[ecx+esi*8]
0152397C   /0F84 A8530000 je 01528D2A
01520E01    83C6 01        add esi,1
jmp 1525545

01525545   /E9 5F2E0000    jmp 015283A9
015283A9    85C9           test ecx,ecx
015223B6   /0F84 C8510000 je 01527584
0151D8A3    A1 FC9A5101    mov eax,dword ptr ds:[1519AFC]
0151ECC7    2BC1           sub eax,ecx
jmp 15265EF

015265EF    C1F8 03        sar eax,3
0152221A    3BF0           cmp esi,eax
jmp 152905C

0152905C  ^\0F83 22E5FFFF jnb 01527584
01528D2A    68 BC945101    push 15194BC
0151DEDE    C707 01000000 mov dword ptr ds:[edi],1
015205CE    FF15 9CC65001  call dword ptr ds:[150C69C]      ; ntdll.RtlLeaveCriticalSection
jmp 151C4C0

0151C4C0    8B75 08        mov esi,dword ptr ss:[ebp+8]
015243C9    897E 08        mov dword ptr ds:[esi+8],edi
jmp 151FCEC

0151FCEC    8B47 04        mov eax,dword ptr ds:[edi+4]
jmp 1524B35

01524B35    05 C0FF0000    add eax,0FFC0
jmp 152969B

0152969B    8BF8           mov edi,eax
01525B49    B9 0F000000    mov ecx,0F
0152341D    F3:A5         rep movs dword ptr es:[edi],dword ptr ds>
0152995A    8D48 FC        lea ecx,dword ptr ds:[eax-4]
jmp 151B4B6

0151B4B6    66:A5         movs word ptr es:[edi],word ptr ds:[esi]
jmp 151BCC5

0151BCC5    51             push ecx
jmp 15263C2

015263C2    8901           mov dword ptr ds:[ecx],eax
jmp 151E667

0151E667    8B6424 04      mov esp,dword ptr ss:[esp+4]
jmp 1521F66

01521F66    56             push esi
jmp 1524140

01524140    8B7424 08      mov esi,dword ptr ss:[esp+8]
jmp 151F723

0151F723    8B46 36        mov eax,dword ptr ds:[esi+36]
0151FD49    50             push eax
01527C5C    C646 34 00     mov byte ptr ds:[esi+34],0
jmp 1524DA1

01524DA1    FF15 80935101  call dword ptr ds:[1519380]
01528833    A9 00000040    test eax,40000000
01527ED8    8946 36        mov dword ptr ds:[esi+36],eax
0151C911   /0F84 D0A20000 je 01526BE7
01521FF8    8B0D A4945101  mov ecx,dword ptr ds:[15194A4]
0151D3FA    25 FFFFFF BF   and eax,BFFFFFFF
jmp 1523987

01523987    894E 10        mov dword ptr ds:[esi+10],ecx
0152797F    8946 36        mov dword ptr ds:[esi+36],eax
0152248E    8B46 36        mov eax,dword ptr ds:[esi+36]
0152851F    85C0           test eax,eax
0151B213   /0F88 26E50000 js 0152973F
01524E11    8B0D F4945101  mov ecx,dword ptr ds:[15194F4]
01528329    03C1           add eax,ecx
015218C8    56             push esi
01527F32    8946 3A        mov dword ptr ds:[esi+3A],eax
```



0152018A `FF15 B8945101` call dword ptr ds:`[15194B8]`

015290DA `8B46 30` mov eax,dword ptr ds:`[esi+30]`

0151B904 `A9 00000040` test eax,40000000

`jmp 15299BF`

015299BF `^\0F84 9BF8FFFF` je 01529260

0152939E `25 FFFFFFFF` and eax,BFFFFFFF

015273FC `8946 30` mov dword ptr ds:`[esi+30]`,eax

`jmp 1529260`

01529260 `807E 34 01` cmp byte ptr ds:`[esi+34]`,1

01521A36 `/0F84 5B7A0000` je 01529497

015298A8 `8B46 30` mov eax,dword ptr ds:`[esi+30]`

0151F0A9 `85C0` test eax,eax

015208DF `/0F88 5A8E0000` js 0152973F

`jmp 1524E11`

01526BE7 `8B15 A0945101` mov edx,dword ptr ds:`[15194A0]`

`jmp 151DB50`

0151DB50 `8956 10` mov dword ptr ds:`[esi+10]`,edx ; manager.00400000

0152248E `8B46 36` mov eax,dword ptr ds:`[esi+36]`

0152851F `85C0` test eax,eax

0151B213 `/0F88 26E50000` js 0152973F

`jmp 1524E11`

0152973F `8B0D 80965101` mov ecx,dword ptr ds:`[1519680]`

01523F52 `8D9408 00000080` lea edx,dword ptr ds:`[eax+ecx+80000000]`

`jmp 151DA83`

0151DA83 `8956 20` mov dword ptr ds:`[esi+20]`,edx

01529497 `8BC6` mov eax,esi

01527A24 `5E` pop esi ; 0013FFB0

01527C3D `C2 0400` retn 4

01527B37 `8368 2C 08` sub dword ptr ds:`[eax+2C]`,8

01522518 `8B48 2C` mov ecx,dword ptr ds:`[eax+2C]`

0151C41E `FF70 20` push dword ptr ds:`[eax+20]`

015201B8 `8F41 04` pop dword ptr ds:`[ecx+4]` ; 01591D91

01523E41 `FF70 1C` push dword ptr ds:`[eax+1C]`

0152155E `8F01` pop dword ptr ds:`[ecx]`

0151D6FA `FF70 2C` push dword ptr ds:`[eax+2C]`

0151F312 `FF70 04` push dword ptr ds:`[eax+4]` ; ntdll.7C92E900

0151F327 `FF70 14` push dword ptr ds:`[eax+14]` ; ntdll.7C930208

01522D3E `FF70 0C` push dword ptr ds:`[eax+C]`

01521BC9 `FF30` push dword ptr ds:`[eax]`

015223EF `FF70 24` push dword ptr ds:`[eax+24]` ; ntdll.KiFastSystemCallRet

015231E4 `FF70 18` push dword ptr ds:`[eax+18]`

0151BD79 `FF70 28` push dword ptr ds:`[eax+28]`

0152655A `8B48 08` mov ecx,dword ptr ds:`[eax+8]`

01524545 `C701 00000000` mov dword ptr ds:`[ecx]`,0

01527B4F `58` pop eax

0151E226 `59` pop ecx ; 0013FFB0

0151D283 `5A` pop edx ; ntdll.KiFastSystemCallRet

01520D75 `5B` pop ebx ; 7FFDA000

0151DB08 `5D` pop ebp ; 0013FFF0

0152917B `5E` pop esi ; ntdll.7C930208

01526BD1 `5F` pop edi ; ntdll.7C92E900

01528C39 `5C` pop esp ; 0013FFB0

015279F7 `9D` popfd

0151E549 `C3` retn

01591D91 `60` pushad

015900D8 `FF7424 20` push dword ptr ss:`[esp+20]`

015268F9 `A1 B0945101` mov eax,dword ptr ds:`[15194B0]`

01525C36 `80B8 E4000000 0>cmp byte ptr ds:`[eax+E4]`,0

01522CB7 `/0F84 64270000` je 01525421

01526102 `FF15 14C75001` call dword ptr ds:`[150C714]` ; kernel32.GetTickCount

0151D494 `8BC8` mov ecx,eax

015286D0 `2B0D 68935101` sub ecx,dword ptr ds:`[1519368]` ; manager.00707416

01524042 `81F9 88130000` cmp ecx,1388

0151FA0B `/0F86 105A0000` jbe 01525421

0151E6F2 `8B15 6C935101` mov edx,dword ptr ds:`[151936C]`

01528235 `52` push edx

015286B8 `A3 68935101` mov dword ptr ds:`[1519368]`,eax

015209E5 `FF15 84C65001` call dword ptr ds:`[150C684]` ; kernel32.ResumeThread

015215BA `833D 289B5101 0>cmp dword ptr ds:`[1519B28]`,3

01524837 `^\0F8C 8FF3FFFF` jl 01523BCC

01523BCC `803D D8945101 0>cmp byte ptr ds:`[15194D8]`,0

0151FEAF `B8 01000000` mov eax,1

0151D3C1 `/0F84 FA900000` je 015264C1

015264C1 `C705 289B5101 0>mov dword ptr ds:`[1519B28]`,0

0151CB28 `A2 D8945101` mov byte ptr ds:`[15194D8]`,al

01525421 `8B4424 04` mov eax,dword ptr ss:`[esp+4]` 这里给 eax 最开始装入的值

01524649 `56` push esi ; ntdll.7C930208

0151CCB3 `50` push eax

0151BD6C `FF15 84935101` call dword ptr ds:`[1519384]` 这个 call 是用来获得函数序号

0151C582 `8B0D C09A5101` mov ecx,dword ptr ds:`[1519AC0]` 这里放入 IAT 基地址

01525B6E `85C9` test ecx,ecx

0151BCA0 `8BF0` mov esi,eax 这个 eax 装入函数序号

015291A2 `^\0F84 CA46FFFF` je 0151D872

015220CC `A1 C49A5101` mov eax,dword ptr ds:`[1519AC4]` 这里保存 IAT 所在基地址的结束位置

0151DE9D `2BC1` sub eax,ecx

01527AAC `C1F8 02` sar eax,2

01520AEA `3BF0` cmp esi,eax 比较是否超过函数序号

0152076C `^\0F82 5FEFFFFF` jb 0151F6D1

0151F6D1 `8B0CB1` mov ecx,dword ptr ds:`[ecx+esi\*4]` ; msvbvm60.ThunRTMain 这里获取函数地址

1519AC0 进这里看看：

```
01810054 73480836 msvbvm60._adj_fpatan---11 这一段 IAT 是一段重复的 IAT
01810058 7346C2FE msvbvm60.__vbaVarIndexStore
0181005C 73497559 msvbvm60.__vbaFpI2
01810060 73485CA6 msvbvm60.GetMemStr
01810064 73471766 msvbvm60.__vbaStrVarMove
01810068 73474C90 msvbvm60.__vbaR8ErrVar
0181006C 73469FC7 msvbvm60.__vbaCastObjVar
01810070 73497660 msvbvm60.__vbaVarMul
01810074 7345BD78 msvbvm60.__vbaStrToUnicode
01810078 73485C74 msvbvm60.GetMem8
0181007C 73476CF5 msvbvm60.rtcRightCharVar
01810080 734893D3 msvbvm60._CIsqrt
01810084 73499957 msvbvm60.__vbaVarCmpNe
01810088 73462106 msvbvm60.rtcGetDateVar
0181008C 734755CB msvbvm60.__vbaGetOwner4
01810090 7345C195 msvbvm60.__vbaSetSystemError
01810094 73474C6D msvbvm60.__vbaI4ErrVar
01810098 73499589 msvbvm60.__vbaVarAnd
0181009C 73471819 msvbvm60.__vbaUI1I2
018100A0 7345C09F msvbvm60.__vbaAryMove
018100A4 734619A7 msvbvm60.rtcGetMonthOfYear
018100A8 73485F89 msvbvm60.PutMemNewObj
018100AC 73476E26 msvbvm60.rtcMidCharBstr
018100B0 73476A45 msvbvm60.__vbaFreeStrList
018100B4 734711BE msvbvm60.rtcHexVarFromVar
018100B8 73469F9E msvbvm60.__vbaVerifyVarObj
018100BC 7347ED2D msvbvm60._allmul
018100C0 73475594 msvbvm60.__vbaGetOwner3
018100C4 73498B5D msvbvm60.__vbaLateIdCallSt
018100C8 73470E6A msvbvm60.__vbaI2Str
018100CC 73485C8B msvbvm60.GetMemObj
018100D0 7348926B msvbvm60._CIlog
018100D4 734599CF msvbvm60.EVENT_SINK_AddRef
018100D8 73497587 msvbvm60.__vbaFpR4
018100DC 73479496 msvbvm60.__vbaStrFixstr
018100E0 73470D93 msvbvm60.rtcVarBstrFromAnsi
018100E4 73477465 msvbvm60.rtcTrimVar
018100E8 73487532 msvbvm60.rtcSaveSetting
018100EC 73480781 msvbvm60._adj_fprem1
018100F0 73499873 msvbvm60.__vbaVarTstLt
018100F4 73461B66 msvbvm60.rtcGetPresentDate
018100F8 7348017A msvbvm60._adj_fdiv_m32i
018100FC 73485DBF msvbvm60.PutMemObj
01810100 73470F58 msvbvm60.__vbaDateStr
01810104 73469DFB msvbvm60.__vbaFreeObj
01810108 7346C474 msvbvm60.__vbaLbound
0181010C 734768EF msvbvm60.__vbaLenVar
01810110 7349980F msvbvm60.__vbaVarTstNe
01810114 73470E2A msvbvm60.__vbaBoolStr
01810118 7346EDBA msvbvm60.rtcErrObj
0181011C 7349759B msvbvm60.__vbaFpR8
01810120 73485C63 msvbvm60.GetMem4
01810124 7349728D msvbvm60.__vbaVarVargNofree
01810128 73466045 msvbvm60.rtcFileLen
0181012C 7347945C msvbvm60.__vbaFixstrConstruct
01810130 734718B9 msvbvm60.__vbaStrErrVarCopy
01810134 7349606C msvbvm60.__vbaForEachCollObj
```



`01810138` `73477C6A` `msvbvm60.rtcStringBstr`

`0181013C` `7349779C` `msvbvm60.__vbaVarInt`

`01810140` `73477BFD` `msvbvm60.rtcSpaceBstr`

`01810144` `73470CFB` `msvbvm60.rtcBstrFromAnsi`

`01810148` `73470E9B` `msvbvm60.__vbaI4Str`

`0181014C` `73469E6B` `msvbvm60.__vbaVarSetObj`

`01810150` `7346CAD9` `msvbvm60.rtcRgb`

`01810154` `7346C287` `msvbvm60.__vbaVarIndexLoad`

`01810158` `7347182F` `msvbvm60.__vbaUI1I4`

`0181015C` `73495F13` `msvbvm60.__vbaNextEachCollVar`

`01810160` `73497570` `msvbvm60.__vbaFpI4`

`01810164` `73485C50` `msvbvm60.GetMem2`

`01810168` `734793DA` `msvbvm60.__vbaStrCmp`

`0181016C` `73495ECE` `msvbvm60.__vbaForEachCollVar`

`01810170` `7346C4FA` `msvbvm60.__vbaAryConstruct2`

`01810174` `734891C6` `msvbvm60._CIcos`

`01810178` `734804C9` `msvbvm60._adj_fprem`

`0181017C` `73497262` `msvbvm60.__vbaFreeVarList`

`01810180` `73496A0A` `msvbvm60.__vbaFreeVarg`

`01810184` `734694A2` `msvbvm60.__vbaCheckTypeVar`

`01810188` `7347545C` `msvbvm60.rtcFileLength`

`0181018C` `734747E2` `msvbvm60.__vbaOnError`

`01810190` `73474C74` `msvbvm60.__vbaI4Var`

`01810194` `73474624` `msvbvm60.__vbaExceptHandler`

`01810198` `7346A050` `msvbvm60.__vbaNew`

`0181019C` `73498C98` `msvbvm60.__vbaLateMemCallSt`

`018101A0` `73464E8E` `msvbvm60.__vbaPrintObj`

`018101A4` `734977C1` `msvbvm60.__vbaVarAdd`

`018101A8` `73499922` `msvbvm60.__vbaVarCmpEq`

`018101AC` `73485DA9` `msvbvm60.PutMem8`

`018101B0` `734771D1` `msvbvm60.__vbaInStrVar`

`018101B4` `73496DF6` `msvbvm60.__vbaVarDup`

`018101B8` `7346C548` `msvbvm60.__vbaAryUnlock`

`018101BC` `734993FA` `msvbvm60.__vbaVarForNext`

`018101C0` `73476EFB` `msvbvm60.rtcAnsiValueBstr`

`018101C4` `73497513` `msvbvm60.__vbaFPException`

`018101C8` `0030006B`

`018101CC` `010C01B7`

`018101D0` `00000000`

`018101D4` `73480836` `msvbvm60._adj_fpatan`---这是第一段 IAT

`018101D8` `7346C2FE` `msvbvm60.__vbaVarIndexStore`

`018101DC` `73497559` `msvbvm60.__vbaFpI2`

`018101E0` `73485CA6` `msvbvm60.GetMemStr`

`018101E4` `73471766` `msvbvm60.__vbaStrVarMove`

`018101E8` `73474C90` `msvbvm60.__vbaR8ErrVar`

`018101EC` `73469FC7` `msvbvm60.__vbaCastObjVar`

`018101F0` `73497660` `msvbvm60.__vbaVarMul`

`018101F4` `7345BD78` `msvbvm60.__vbaStrToUnicode`

`018101F8` `73485C74` `msvbvm60.GetMem8`

`018101FC` `73476CF5` `msvbvm60.rtcRightCharVar`

`01810200` `734893D3` `msvbvm60._CIsqrt`

`01810204` `73499957` `msvbvm60.__vbaVarCmpNe`

`01810208` `73462106` `msvbvm60.rtcGetDateVar`

`0181020C` `734755CB` `msvbvm60.__vbaGetOwner4`

`01810210` `7345C195` `msvbvm60.__vbaSetSystemError`

`01810214` `73474C6D` `msvbvm60.__vbaI4ErrVar`

`01810218` `73499589` `msvbvm60.__vbaVarAnd`

`0181021C` `73471819` `msvbvm60.__vbaUI1I2`

`01810220` `7345C09F` `msvbvm60.__vbaAryMove`

`01810224` `734619A7` `msvbvm60.rtcGetMonthOfYear`

`01810228` `73485F89` `msvbvm60.PutMemNewObj`

`0181022C` `73476E26` `msvbvm60.rtcMidCharBstr`

`01810230` `73476A45` `msvbvm60.__vbaFreeStrList`

`01810234` `734711BE` `msvbvm60.rtcHexVarFromVar`

`01810238` `73469F9E` `msvbvm60.__vbaVerifyVarObj`

`0181023C` `7347ED2D` `msvbvm60._allmul`

`01810240` `73475594` `msvbvm60.__vbaGetOwner3`

`01810244` `73498B5D` `msvbvm60.__vbaLateIdCallSt`

`01810248` `73470E6A` `msvbvm60.__vbaI2Str`

`0181024C` `73485C8B` `msvbvm60.GetMemObj`

`01810250` `7348926B` `msvbvm60._CIlog`

`01810254` `734599CF` `msvbvm60.EVENT_SINK_AddRef`

`01810258` `73497587` `msvbvm60.__vbaFpR4`

`0181025C` `73479496` `msvbvm60.__vbaStrFixstr`

`01810260` `73470D93` `msvbvm60.rtcVarBstrFromAnsi`

`01810264` `73477465` `msvbvm60.rtcTrimVar`

`01810268` `73487532` `msvbvm60.rtcSaveSetting`

`0181026C` `73480781` `msvbvm60._adj_fprem1`

`01810270` `73499873` `msvbvm60.__vbaVarTstLt`

`01810274` `73461B66` `msvbvm60.rtcGetPresentDate`

`01810278` `7348017A` `msvbvm60._adj_fdiv_m32i`

`0181027C` `73485DBF` `msvbvm60.PutMemObj`

`01810280` `73470F58` `msvbvm60.__vbaDateStr`

`01810284` `73469DFB` `msvbvm60.__vbaFreeObj`

`01810288` `7346C474` `msvbvm60.__vbaLbound`

`0181028C` `734768EF` `msvbvm60.__vbaLenVar`

`01810290` `7349980F` `msvbvm60.__vbaVarTstNe`

`01810294` `73470E2A` `msvbvm60.__vbaBoolStr`

`01810298` `7346EDBA` `msvbvm60.rtcErrObj`

`0181029C` `7349759B` `msvbvm60.__vbaFpR8`

`018102A0` `73485C63` `msvbvm60.GetMem4`

`018102A4` `7349728D` `msvbvm60.__vbaVarVargNofree`

`018102A8` `73466045` `msvbvm60.rtcFileLen`

`018102AC` `7347945C` `msvbvm60.__vbaFixstrConstruct`

`018102B0` `734718B9` `msvbvm60.__vbaStrErrVarCopy`

`018102B4` `7349606C` `msvbvm60.__vbaForEachCollObj`

`018102B8` `73477C6A` `msvbvm60.rtcStringBstr`

`018102BC` `7349779C` `msvbvm60.__vbaVarInt`

`018102C0` `73477BFD` `msvbvm60.rtcSpaceBstr`

`018102C4` `73470CFB` `msvbvm60.rtcBstrFromAnsi`

`018102C8` `73470E9B` `msvbvm60.__vbaI4Str`

`018102CC` `73469E6B` `msvbvm60.__vbaVarSetObj`

`018102D0` `7346CAD9` `msvbvm60.rtcRgb`

`018102D4` `7346C287` `msvbvm60.__vbaVarIndexLoad`

`018102D8` `7347182F` `msvbvm60.__vbaUI1I4`

`018102DC` `73495F13` `msvbvm60.__vbaNextEachCollVar`

`018102E0` `73497570` `msvbvm60.__vbaFpI4`

`018102E4` `73485C50` `msvbvm60.GetMem2`

`018102E8` `734793DA` `msvbvm60.__vbaStrCmp`

`018102EC` `73495ECE` `msvbvm60.__vbaForEachCollVar`

`018102F0` `7346C4FA` `msvbvm60.__vbaAryConstruct2`

`018102F4` `734891C6` `msvbvm60._CIcos`

`018102F8` `734804C9` `msvbvm60._adj_fprem`

`018102FC` `73497262` `msvbvm60.__vbaFreeVarList`

`01810300` `73496A0A` `msvbvm60.__vbaFreeVarg`

`01810304` `734694A2` `msvbvm60.__vbaCheckTypeVar`

`01810308` `7347545C` `msvbvm60.rtcFileLength`

`0181030C` `734747E2` `msvbvm60.__vbaOnError`

`01810310` `73474C74` `msvbvm60.__vbaI4Var`

`01810314` `73474624` `msvbvm60.__vbaExceptHandler`

`01810318` `7346A050` `msvbvm60.__vbaNew`

`0181031C` `73498C98` `msvbvm60.__vbaLateMemCallSt`

`01810320` `73464E8E` `msvbvm60.__vbaPrintObj`

`01810324` `734977C1` `msvbvm60.__vbaVarAdd`

`01810328` `73499922` `msvbvm60.__vbaVarCmpEq`

`0181032C` `73485DA9` `msvbvm60.PutMem8`

`01810330` `734771D1` `msvbvm60.__vbaInStrVar`

`01810334` `73496DF6` `msvbvm60.__vbaVarDup`

`01810338` `7346C548` `msvbvm60.__vbaAryUnlock`

`0181033C` `734993FA` `msvbvm60.__vbaVarForNext`

`01810340` `73476EFB` `msvbvm60.rtcAnsiValueBstr`

`01810344` `73497513` `msvbvm60.__vbaFPException`

`01810348` `734598E0` `msvbvm60.EVENT_SINK_QueryInterface`

`0181034C` `73499841` `msvbvm60.__vbaVarTstGt`

`01810350` `73489100` `msvbvm60._CIatan`

`01810354` `7346C156` `msvbvm60.__vbaRedim`

`01810358` `73497185` `msvbvm60.__vbaBoolVarNull`

`0181035C` `73480246` `msvbvm60._adj_fdivr_m16i`

`01810360` `73496AEE` `msvbvm60.__vbaVarMove`

`01810364` `73474C5F` `msvbvm60.__vbaI2ErrVar`

`01810368` `7346CF7E` `msvbvm60.rtcMsgBox`

`0181036C` `73498D8C` `msvbvm60.__vbaVarLateMemCallLd`

`01810370` `7345DF51` `msvbvm60.rtcDoEvents`

`01810374` `73474C66` `msvbvm60.__vbaI2Var`

`01810378` `7346C2D0` `msvbvm60.__vbaVarIndexLoadRefLock`

`0181037C` `734972BE` `msvbvm60.__vbaVargVar`

`01810380` `7348737C` `msvbvm60.rtcGetSetting`

`01810384` `73489484` `msvbvm60._CItan`

`01810388` `7346A083` `msvbvm60.__vbaNew2`

`0181038C` `73498CDC` `msvbvm60.__vbaLateMemStAd`

`01810390` `73469E54` `msvbvm60.__vbaObjSetAddref`

`01810394` `73488804` `msvbvm60.rtcReplace`

`01810398` `7346DCE5` `msvbvm60.VarPtr`

`0181039C` `734753CE` `msvbvm60.rtcFileLocation`

`018103A0` `73485D89` `msvbvm60.PutMem2`

`018103A4` `7348027A` `msvbvm60._adj_fdivr_m32i`

`018103A8` `73485CFF` `msvbvm60.GetMemNewObj`

`018103AC` `734640C2` `msvbvm60.rtcFreeFile`

`018103B0` `73485D9A` `msvbvm60.PutMem4`

`018103B4` `7346B5A9` `msvbvm60.__vbaRecUniToAnsi`

`018103B8` `734801AE` `msvbvm60._adj_fdivr_m32`

`018103BC` `73476BDE` `msvbvm60.rtcLeftCharBstr`

`018103C0` `73471785` `msvbvm60.__vbaStrVarVal`

`018103C4` `734960C4` `msvbvm60.__vbaNextEachCollObj`

`018103C8` `73498CB8` `msvbvm60.__vbaLateMemCall`

`018103CC` `73499564` `msvbvm60.__vbaVarNot`

`018103D0` `7347040D` `msvbvm60.__vbaStrDate`

`018103D4` `73493A13` `msvbvm60.rtcArray`

`018103D8` `73465B8E` `msvbvm60.rtcKillFiles`

`018103DC` `7347DB51` `msvbvm60._CIexp`



```
018103E0  73470B53  msvbvm60.__vbaDateVar  
018103E4  73476A74  msvbvm60.__vbaStrMove  
018103E8  7346C263  msvbvm60.__vbaRefVarAry  
018103EC  7348932E  msvbvm60._CIsin  
018103F0  7345E6A8  msvbvm60.__vbaError  
018103F4  734768DF  msvbvm60.__vbaLenBstr  
018103F8  73461964  msvbvm60.rtcGetYear  
018103FC  7346C4A6  msvbvm60.__vbaUbound  
01810400  734999C1  msvbvm60.__vbaVarCmpGt  
01810404  73496BB8  msvbvm60.__vbaVarCopy  
01810408  73485FF1  msvbvm60.SetMemObj  
0181040C  7349998C  msvbvm60.__vbaVarCmpGe  
01810410  7346A0C0  msvbvm60.__vbaHresultCheckObj  
01810414  73476E73  msvbvm60.rtcMidCharVar  
01810418  73498C75  msvbvm60.__vbaLateMemSt  
0181041C  73463B5D  msvbvm60.__vbaFileOpen  
01810420  73498BA6  msvbvm60.__vbaLateIdStAd  
01810424  7339A3BF  msvbvm60.__vbaStrToAnsi  
01810428  7347FBE9  msvbvm60._adj_fdiv_r  
0181042C  7347488D  msvbvm60.__vbaExitProc  
01810430  7346C816  msvbvm60.rtcIsNumeric  
01810434  7346B5E3  msvbvm60.__vbaRecAnsiToUni  
01810438  7346CB51  msvbvm60.rtcRandomNext  
0181043C  73497534  msvbvm60.__vbaFpUI1  
01810440  7346C525  msvbvm60.__vbaAryLock  
01810444  73495DED  msvbvm60.__vbaCastObj  
01810448  7346C6A9  msvbvm60.rtcGetTimer  
0181044C  73469E0F  msvbvm60.__vbaFreeObjList  
01810450  73498B25  msvbvm60.__vbaLateIdCallLd  
01810454  734800FA  msvbvm60._adj_fdiv_m64  
01810458  73470311  msvbvm60.__vbaStrUI1  
0181045C  73476A8E  msvbvm60.__vbaStrCopy  
01810460  7346C25C  msvbvm60.__vbaGenerateBoundsError  
01810464  73470374  msvbvm60.__vbaStrI4  
01810468  73496831  msvbvm60.__vbaFreeVar  
0181046C  734995B2  msvbvm60.__vbaVarOr  
01810470  7339A0E5  msvbvm60.DllFunctionCall  
01810474  73474826  msvbvm60.__vbaResume  
01810478  7346961D  msvbvm60.rtcCreateObject2  
0181047C  734800AE  msvbvm60._adj_fdiv_m32  
01810480  7346CB86  msvbvm60.rtcRandomize  
01810484  73465AD6  msvbvm60.rtcMakeDir  
01810488  73476CE2  msvbvm60.rtcRightCharBstr  
0181048C  73497689  msvbvm60.__vbaVarDiv  
01810490  733935A4  msvbvm60.ThunRTMain  
01810494  73493A40  msvbvm60.__vbaStr2Vec  
01810498  734977EA  msvbvm60.__vbaVarSub  
0181049C  73474B48  msvbvm60.__vbaI4Abs  
018104A0  73480146  msvbvm60._adj_fdiv_m16i  
018104A4  73469FF5  msvbvm60.__vbaObjVar  
018104A8  734999F6  msvbvm60.__vbaVarCmpLe  
018104AC  73469E3D  msvbvm60.__vbaObjSet  
018104B0  7345CCC3  msvbvm60.rtcShell  
018104B4  73485E8F  msvbvm60.PutMemStr  
018104B8  734880D9  msvbvm60.rtcInStrRev  
018104BC  73499828  msvbvm60.__vbaVarTstGe  
018104C0  734755B0  msvbvm60.__vbaPutOwner3  
018104C4  734767C1  msvbvm60.__vbaVarCat  
018104C8  73470F27  msvbvm60.__vbaR8Str  
018104CC  7346B550  msvbvm60.__vbaRecDestruct  
018104D0  73470EF7  msvbvm60.__vbaR4Str  
018104D4  73480839  msvbvm60._adj_fptan  
018104D8  73487C1B  msvbvm60.rtcSplit  
018104DC  73477C3F  msvbvm60.rtcSpaceVar  
018104E0  7348612A  ASCII "QWP="  
018104E4  73469514  msvbvm60.__vbaObjIs  
018104E8  73462148  msvbvm60.rtcGetTimeVar  
018104EC  734599E2  msvbvm60.EVENT_SINK_Release  
018104F0  73498B44  msvbvm60.__vbaLateIdSt  
018104F4  73471845  msvbvm60.__vbaI2I4  
018104F8  734801FA  msvbvm60._adj_fdivr_m64  
018104FC  7348626B  msvbvm60.rtcVarFromFormatVar  
01810500  7346BF49  msvbvm60.__vbaErase  
01810504  73477DEE  msvbvm60.rtcUpperCaseVar  
01810508  7346C2A3  msvbvm60.__vbaVarIndexLoadRef  
0181050C  734997F6  msvbvm60.__vbaVarTstEq  
01810510  73465244  msvbvm60.rtcDir  
01810514  7339A27E  msvbvm60.__vbaInStr  
01810518  73463FCA  msvbvm60.__vbaFileClose  
0181051C  00000000  
01810520  006B009F  manager.006B009F  
01810524  0108012A  
01810528  73470BB0  msvbvm60.__vbaBoolVar  ----这是第二段 IAT  
0181052C  73480836  msvbvm60._adj_fpatan  
01810530  7346C2FE  msvbvm60.__vbaVarIndexStore  
01810534  73497559  msvbvm60.__vbaFpI2  
01810538  73485CA6  msvbvm60.GetMemStr  
0181053C  73471766  msvbvm60.__vbaStrVarMove  
01810540  73474C90  msvbvm60.__vbaR8ErrVar  
01810544  73469FC7  msvbvm60.__vbaCastObjVar  
01810548  73497660  msvbvm60.__vbaVarMul  
0181054C  7345BD78  msvbvm60.__vbaStrToUnicode  
01810550  73485C74  msvbvm60.GetMem8  
01810554  73476CF5  msvbvm60.rtcRightCharVar  
01810558  734893D3  msvbvm60._CIsqrt  
0181055C  73499957  msvbvm60.__vbaVarCmpNe  
01810560  73462106  msvbvm60.rtcGetDateVar  
01810564  734755CB  msvbvm60.__vbaGetOwner4  
01810568  7345C195  msvbvm60.__vbaSetSystemError  
0181056C  73474C6D  msvbvm60.__vbaI4ErrVar  
01810570  73499589  msvbvm60.__vbaVarAnd  
01810574  73471819  msvbvm60.__vbaUI1I2  
01810578  7345C09F  msvbvm60.__vbaAryMove  
0181057C  734619A7  msvbvm60.rtcGetMonthOfYear  
01810580  73485F89  msvbvm60.PutMemNewObj  
01810584  73476E26  msvbvm60.rtcMidCharBstr  
01810588  73476A45  msvbvm60.__vbaFreeStrList  
0181058C  734711BE  msvbvm60.rtcHexVarFromVar  
01810590  73469F9E  msvbvm60.__vbaVerifyVarObj  
01810594  7347ED2D  msvbvm60._allmul  
01810598  73475594  msvbvm60.__vbaGetOwner3  
0181059C  73498B5D  msvbvm60.__vbaLateIdCallSt  
018105A0  73470E6A  msvbvm60.__vbaI2Str  
018105A4  73485C8B  msvbvm60.GetMemObj  
018105A8  7348926B  msvbvm60._CIlog  
018105AC  734599CF  msvbvm60.EVENT_SINK_AddRef  
018105B0  73497587  msvbvm60.__vbaFpR4  
018105B4  73479496  msvbvm60.__vbaStrFixstr  
018105B8  73470D93  msvbvm60.rtcVarBstrFromAnsi  
018105BC  73477465  msvbvm60.rtcTrimVar  
018105C0  73487532  msvbvm60.rtcSaveSetting  
018105C4  73480781  msvbvm60._adj_fprem1  
018105C8  73499873  msvbvm60.__vbaVarTstLt  
018105CC  73461B66  msvbvm60.rtcGetPresentDate  
018105D0  7348017A  msvbvm60._adj_fdiv_m32i  
018105D4  73485DBF  msvbvm60.PutMemObj  
018105D8  73470F58  msvbvm60.__vbaDateStr  
018105DC  73469DFB  msvbvm60.__vbaFreeObj  
018105E0  7346C474  msvbvm60.__vbaLbound  
018105E4  734768EF  msvbvm60.__vbaLenVar  
018105E8  7349980F  msvbvm60.__vbaVarTstNe  
018105EC  73470E2A  msvbvm60.__vbaBoolStr  
018105F0  7346EDBA  msvbvm60.rtcErrObj  
018105F4  7349759B  msvbvm60.__vbaFpR8  
018105F8  73485C63  msvbvm60.GetMem4  
018105FC  7349728D  msvbvm60.__vbaVarVargNofree  
01810600  73466045  msvbvm60.rtcFileLen  
01810604  7347945C  msvbvm60.__vbaFixstrConstruct  
01810608  734718B9  msvbvm60.__vbaStrErrVarCopy  
0181060C  7349606C  msvbvm60.__vbaForEachCollObj  
01810610  73477C6A  msvbvm60.rtcStringBstr  
01810614  7349779C  msvbvm60.__vbaVarInt  
01810618  73477BFD  msvbvm60.rtcSpaceBstr  
0181061C  73470CFB  msvbvm60.rtcBstrFromAnsi  
01810620  73470E9B  msvbvm60.__vbaI4Str  
01810624  73469E6B  msvbvm60.__vbaVarSetObj  
01810628  7346CAD9  msvbvm60.rtcRgb  
0181062C  7346C287  msvbvm60.__vbaVarIndexLoad  
01810630  7347182F  msvbvm60.__vbaUI1I4  
01810634  73495F13  msvbvm60.__vbaNextEachCollVar  
01810638  73497570  msvbvm60.__vbaFpI4  
0181063C  73485C50  msvbvm60.GetMem2  
01810640  734793DA  msvbvm60.__vbaStrCmp  
01810644  73495ECE  msvbvm60.__vbaForEachCollVar  
01810648  7346C4FA  msvbvm60.__vbaAryConstruct2  
0181064C  734891C6  msvbvm60._CIcos  
01810650  734804C9  msvbvm60._adj_fprem  
01810654  73497262  msvbvm60.__vbaFreeVarList  
01810658  73496A0A  msvbvm60.__vbaFreeVarg  
0181065C  734694A2  msvbvm60.__vbaCheckTypeVar  
01810660  7347545C  msvbvm60.rtcFileLength  
01810664  734747E2  msvbvm60.__vbaOnError  
01810668  73474C74  msvbvm60.__vbaI4Var  
0181066C  73474624  msvbvm60.__vbaExceptHandler  
01810670  7346A050  msvbvm60.__vbaNew  
01810674  73498C98  msvbvm60.__vbaLateMemCallSt  
01810678  73464E8E  msvbvm60.__vbaPrintObj  
0181067C  734977C1  msvbvm60.__vbaVarAdd  
01810680  73499922  msvbvm60.__vbaVarCmpEq  
01810684  73485DA9  msvbvm60.PutMem8  
01810688  734771D1  msvbvm60.__vbaInStrVar  
```



0181068C 73496DF6 `msvbvm60.__vbaVarDup`

01810690 7346C548 `msvbvm60.__vbaAryUnlock`

01810694 734993FA `msvbvm60.__vbaVarForNext`

01810698 73476EFB `msvbvm60.rtcAnsiValueBstr`

0181069C 73497513 `msvbvm60.__vbaFPException`

018106A0 734598E0 `msvbvm60.EVENT_SINK_QueryInterface`

018106A4 73499841 `msvbvm60.__vbaVarTstGt`

018106A8 73489100 `msvbvm60._CIatan`

018106AC 7346C156 `msvbvm60.__vbaRedim`

018106B0 73497185 `msvbvm60.__vbaBoolVarNull`

018106B4 73480246 `msvbvm60._adj_fdivr_m16i`

018106B8 73496AEE `msvbvm60.__vbaVarMove`

018106BC 73474C5F `msvbvm60.__vbaI2ErrVar`

018106C0 7346CF7E `msvbvm60.rtcMsgBox`

018106C4 73498D8C `msvbvm60.__vbaVarLateMemCallLd`

018106C8 7345DF51 `msvbvm60.rtcDoEvents`

018106CC 73474C66 `msvbvm60.__vbaI2Var`

018106D0 7346C2D0 `msvbvm60.__vbaVarIndexLoadRefLock`

018106D4 734972BE `msvbvm60.__vbaVargVar`

018106D8 7348737C `msvbvm60.rtcGetSetting`

018106DC 73489484 `msvbvm60._CItan`

018106E0 7346A083 `msvbvm60.__vbaNew2`

018106E4 73498CDC `msvbvm60.__vbaLateMemStAd`

018106E8 73469E54 `msvbvm60.__vbaObjSetAddref`

018106EC 73488804 `msvbvm60.rtcReplace`

018106F0 7346DCE5 `msvbvm60.VarPtr`

018106F4 734753CE `msvbvm60.rtcFileLocation`

018106F8 73485D89 `msvbvm60.PutMem2`

018106FC 7348027A `msvbvm60._adj_fdivr_m32i`

01810700 73485CFF `msvbvm60.GetMemNewObj`

01810704 734640C2 `msvbvm60.rtcFreeFile`

01810708 73485D9A `msvbvm60.PutMem4`

0181070C 7346B5A9 `msvbvm60.__vbaRecUniToAnsi`

01810710 734801AE `msvbvm60._adj_fdivr_m32`

01810714 73476BDE `msvbvm60.rtcLeftCharBstr`

01810718 73471785 `msvbvm60.__vbaStrVarVal`

0181071C 734960C4 `msvbvm60.__vbaNextEachCollObj`

01810720 73498CB8 `msvbvm60.__vbaLateMemCall`

01810724 73499564 `msvbvm60.__vbaVarNot`

01810728 7347040D `msvbvm60.__vbaStrDate`

0181072C 73493A13 `msvbvm60.rtcArray`

01810730 73465B8E `msvbvm60.rtcKillFiles`

01810734 7347DB51 `msvbvm60._CIexp`

01810738 73470B53 `msvbvm60.__vbaDateVar`

0181073C 73476A74 `msvbvm60.__vbaStrMove`

01810740 7346C263 `msvbvm60.__vbaRefVarAry`

01810744 7348932E `msvbvm60._CIsin`

01810748 7345E6A8 `msvbvm60.__vbaError`

0181074C 734768DF `msvbvm60.__vbaLenBstr`

01810750 73461964 `msvbvm60.rtcGetYear`

01810754 7346C4A6 `msvbvm60.__vbaUbound`

01810758 734999C1 `msvbvm60.__vbaVarCmpGt`

0181075C 73496BB8 `msvbvm60.__vbaVarCopy`

01810760 73485FF1 `msvbvm60.SetMemObj`

01810764 7349998C `msvbvm60.__vbaVarCmpGe`

01810768 7346A0C0 `msvbvm60.__vbaHresultCheckObj`

0181076C 73476E73 `msvbvm60.rtcMidCharVar`

01810770 73498C75 `msvbvm60.__vbaLateMemSt`

01810774 73463B5D `msvbvm60.__vbaFileOpen`

01810778 73498BA6 `msvbvm60.__vbaLateIdStAd`

0181077C 7339A3BF `msvbvm60.__vbaStrToAnsi`

01810780 7347FBE9 `msvbvm60._adj_fdiv_r`

01810784 7347488D `msvbvm60.__vbaExitProc`

01810788 7346C816 `msvbvm60.rtcIsNumeric`

0181078C 7346B5E3 `msvbvm60.__vbaRecAnsiToUni`

01810790 7346CB51 `msvbvm60.rtcRandomNext`

01810794 73497534 `msvbvm60.__vbaFpUI1`

01810798 7346C525 `msvbvm60.__vbaAryLock`

0181079C 73495DED `msvbvm60.__vbaCastObj`

018107A0 7346C6A9 `msvbvm60.rtcGetTimer`

018107A4 73469E0F `msvbvm60.__vbaFreeObjList`

018107A8 73498B25 `msvbvm60.__vbaLateIdCallLd`

018107AC 734800FA `msvbvm60._adj_fdiv_m64`

018107B0 73470311 `msvbvm60.__vbaStrUI1`

018107B4 73476A8E `msvbvm60.__vbaStrCopy`

018107B8 7346C25C `msvbvm60.__vbaGenerateBoundsError`

018107BC 73470374 `msvbvm60.__vbaStrI4`

018107C0 73496831 `msvbvm60.__vbaFreeVar`

018107C4 734995B2 `msvbvm60.__vbaVarOr`

018107C8 7339A0E5 `msvbvm60.DllFunctionCall`

018107CC 73474826 `msvbvm60.__vbaResume`

018107D0 7346961D `msvbvm60.rtcCreateObject2`

018107D4 734800AE `msvbvm60._adj_fdiv_m32`

018107D8 7346CB86 `msvbvm60.rtcRandomize`

018107DC 73465AD6 `msvbvm60.rtcMakeDir`

018107E0 73476CE2 `msvbvm60.rtcRightCharBstr`

018107E4 73497689 `msvbvm60.__vbaVarDiv`

018107E8 733935A4 `msvbvm60.ThunRTMain`

018107EC 73493A40 `msvbvm60.__vbaStr2Vec`

018107F0 734977EA `msvbvm60.__vbaVarSub`

018107F4 73474B48 `msvbvm60.__vbaI4Abs`

018107F8 73480146 `msvbvm60._adj_fdiv_m16i`

018107FC 73469FF5 `msvbvm60.__vbaObjVar`

01810800 734999F6 `msvbvm60.__vbaVarCmpLe`

01810804 73469E3D `msvbvm60.__vbaObjSet`

01810808 7345CCC3 `msvbvm60.rtcShell`

0181080C 73485E8F `msvbvm60.PutMemStr`

01810810 734880D9 `msvbvm60.rtcInStrRev`

01810814 73499828 `msvbvm60.__vbaVarTstGe`

01810818 734755B0 `msvbvm60.__vbaPutOwner3`

0181081C 734767C1 `msvbvm60.__vbaVarCat`

01810820 73470F27 `msvbvm60.__vbaR8Str`

01810824 7346B550 `msvbvm60.__vbaRecDestruct`

01810828 73470EF7 `msvbvm60.__vbaR4Str`

0181082C 73480839 `msvbvm60._adj_fptan`

01810830 73487C1B `msvbvm60.rtcSplit`

01810834 73477C3F `msvbvm60.rtcSpaceVar`

01810838 7348612A `ASCII "QWP="`

0181083C 73469514 `msvbvm60.__vbaObjIs`

01810840 73462148 `msvbvm60.rtcGetTimeVar`

01810844 734599E2 `msvbvm60.EVENT_SINK_Release`

01810848 73498B44 `msvbvm60.__vbaLateIdSt`

0181084C 73471845 `msvbvm60.__vbaI2I4`

01810850 734801FA `msvbvm60._adj_fdivr_m64`

01810854 7348626B `msvbvm60.rtcVarFromFormatVar`

01810858 7346BF49 `msvbvm60.__vbaErase`

0181085C 73477DEE `msvbvm60.rtcUpperCaseVar`

01810860 7346C2A3 `msvbvm60.__vbaVarIndexLoadRef`

01810864 734997F6 `msvbvm60.__vbaVarTstEq`

01810868 73465244 `msvbvm60.rtcDir`

0181086C 7339A27E `msvbvm60.__vbaInStr`

01810870 73463FCA `msvbvm60.__vbaFileClose`

01810874 734860A6 `msvbvm60.SetMemNewObj`

01810878 7347964E `msvbvm60.__vbaR8IntI4`

0181087C 73478E2D `msvbvm60.rtcStrConvVar2`

01810880 73479637 `msvbvm60.__vbaR8IntI2`

01810884 73498B74 `msvbvm60.__vbaLateIdCall`

01810888 73463AA5 `msvbvm60.__vbaInputFile`

0181088C 73477529 `msvbvm60.__vbaLsetFixstr`

01810890 73486018 `msvbvm60.SetMemVar`

01810894 734993CC `msvbvm60.__vbaVarForInit`

01810898 73498C4D `msvbvm60.__vbaLateMemCallLd`

0181089C 734623ED `msvbvm60.rtcDateAdd`

018108A0 73470344 `msvbvm60.__vbaStrI2`

018108A4 73498DC7 `msvbvm60.__vbaVarLateMemSt`

018108A8 73498DA8 `msvbvm60.__vbaVarLateMemCallLdRf`

018108AC 73474C9E `msvbvm60.__vbaR8Var`

018108B0 73476BF1 `msvbvm60.rtcLeftCharVar`

018108B4 73493AEC `msvbvm60.__vbaVar2Vec`

018108B8 73465D5A `msvbvm60.rtcFileCopy`

018108BC 7346417F `msvbvm60.rtcEndOfFile`

018108C0 7349985A `msvbvm60.__vbaVarTstLe`

018108C4 73477D91 `msvbvm60.rtcStringVar`

018108C8 734703A4 `msvbvm60.__vbaStrR4`

018108CC 7345BCE3 `msvbvm60.__vbaEnd`

018108D0 73474C97 `msvbvm60.__vbaR4Var`

018108D4 734768BA `msvbvm60.__vbaStrCat`

018108D8 73462628 `msvbvm60.rtcDateDiff`

018108DC 73462E44 `msvbvm60.rtcInputCountVar`

018108E0 73476AC3 `msvbvm60.__vbaLenBstrB`

018108E4 73476A30 `msvbvm60.__vbaFreeStr`

018108E8 7345E6C1 `msvbvm60.__vbaErrorOverflow`

018108EC 7346B61D `msvbvm60.__vbaRecDestructAnsi`

018108F0 73485CE0 `msvbvm60.GetMemVar`

018108F4 73485ED6 `msvbvm60.PutMemVar`

018108F8 73470483 `msvbvm60.__vbaStrVarCopy`

018108FC 7346C185 `msvbvm60.__vbaRedimPreserve`

01810900 7346C5A8 `msvbvm60.rtcBstrFromError`

01810904 7346C04A `msvbvm60.__vbaAryDestruct`

通过这个地址我们看到了`IAT`真正的位置，也找到了`IAT`的加密方法：

`ZP`按照函数的不同把函数分成不同几组，每组的处理的方式是不同。这些`IAT`组在内存中是间隔分开的。一般来说是分成两组，组内的地址是连续的。所以这个`ZP`使用的调用方式是“组起始地址+编号”的形式调用的。

在处理`IAT`上，壳还会把一部分`IAT`进行若干次的重复（看上面那段重复的部分），排列在同一个内存段。但实际上有效的`IAT`只有两组。一般来说是最下面的一组和倒数第二组。

可能有人已经想到了：两组有效的`IAT`，对应的正是两个`IAT`加密等级。

调试一下会发现，其实的这两加密等级的解码`call`是同一个，只是前面的代码是不一样的~ 这些代码的结果就是向`1519AC0`中放入不同的组的开始地址。

```
0151BD6C    FF15 84935101   call dword ptr ds:[1519384]  这个 call 是用来获得函数序号
```



### 0151C582 处 IAT 基地址写入说明
`0151C582` `8B0D C09A5101` `mov ecx,dword ptr ds:[1519AC0]`，这里放入 `IAT` 基地址。

### 修复方法
`IAT` 调用是连续的，从起始位置循环处理即可。

以第一个为例：
`00412940` `- E9 DBB73300` `jmp manager.0074E120 *`

1. 第一步：读取 `dword ptr ds:[0074E120+1]` 这个值，获取函数序号提取码。
2. 第二步：判断属于第一组还是第二组。
3. 第三步：调用壳的子函数 `call` 来获得函数序号。
4. 第四步：获取 `IAT` 组的开始地址。
5. 第五步：通过上面的两个数据获得函数真实地址，然后修改 `*` 处。
6. 第六步：循环下一个。

有兴趣的高人可以写一个脚本玩玩儿哦~

搞定了这个程序以后，继续研究一下 `Delphi` 的程序。

同样的办法到达 `OEP`，随便找一下 `IAT` 调用。

还是被壳修改了，改的形式是一样的。

### 示例 `IAT` 调用（`Farsight`）

```
00407CDC  - E9 9FA42D00     jmp Farsight.006E2180
00407CE1    90              nop
00407CE2    8BC0            mov eax,eax
00407CE4  - E9 13B02D00     jmp Farsight.006E2CFC
00407CE9    90              nop
00407CEA    8BC0            mov eax,eax
```

跟踪一下就能看出来 `IAT` 的处理方式是一模一样的。尤其是下面一段代码，几乎是很特征性的。

### 特征性代码段（`kernel32` 相关调用）

```
01526102    FF15 14C75001   call dword ptr ds:[150C714]               ; kernel32.GetTickCount
0151D494    8BC8            mov ecx,eax
015286D0    2B0D 68935101   sub ecx,dword ptr ds:[1519368]            ; manager.00707416
01524042    81F9 88130000   cmp ecx,1388
0151FA0B   /0F86 105A0000   jbe 01525421
0151E6F2    8B15 6C935101   mov edx,dword ptr ds:[151936C]
01528235    52              push edx
015286B8    A3 68935101     mov dword ptr ds:[1519368],eax
015209E5    FF15 84C65001   call dword ptr ds:[150C684]               ; kernel32.ResumeThread
015215BA    833D 289B5101 0>cmp dword ptr ds:[1519B28],3
01524837  ^\0F8C 8FF3FFFF   jl 01523BCC
01523BCC    803D D8945101 0>cmp byte ptr ds:[15194D8],0
0151FEAF    B8 01000000     mov eax,1
0151D3C1   /0F84 FA900000   je 015264C1
015264C1    C705 289B5101 0>mov dword ptr ds:[1519B28],0
```

### `FF25` 型与 `FF15` 型 `IAT` 修复
这个程序的 `IAT` 调用是 `FF25` 型的，修复的时候要先修复 `FF25`，然后找个空白地方写入 `IAT` 地址就可以了~~~

对于 `FF15` 型的程序来说，也比较好辨别，因为程序在修改调用 `call` 的时候改成了 `E9 XXXXXXXX` 形式，同 `FF15 XXXXXXXX` 的调用形式差了一个字节，所以加壳以后在 `IAT` 调用的 `call` 下面是有一个 `nop` 指令的。

### 总结
上面就是对 `ZProtect` 加壳后程序的 `IAT` 修复的方法。脚本和修复工具就不附加了。偶的编程水平实在不行。有兴趣的大侠可以写个通用的脚本或工具。到时一定捧场~

### 突破 `ZP` 注册对话框
最后说一下突破 `ZP` 的注册对话框，`y3` 大侠的 `dll` 可以过不带试用框的 `ZP` 程序，貌似是 hook 了 `DialogBoxIndirectParam` 这个函数，希望 `y3` 大侠能公开方法~~ （带试用框就不用跳了吧，直接试用，然后脱壳就行了……）

小菜心得，欢迎批评！

`HyperChem`

`02/23/2009`

### 附件
- 目标 `VB` 程序.rar [附件请到论坛下载：[`bbs.pediy.com/showthread.php?t=82665`](http://bbs.pediy.com/showthread.php?t=82665)]
- ![附件](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg)
