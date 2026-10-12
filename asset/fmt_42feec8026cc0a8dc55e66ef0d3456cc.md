
# Windows 系统专家 V2.5 Full 分析旅程

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-311.htm`](https://www.pediy.com/kssd/pediy07/pediy7-311.htm)

【破文标题】：Windows 系统专家 V2.5 Full 分析旅程

【破文作者】：`KuNgBiM[DFCG]`

【作者邮箱】：`gb_1227@163.com`

【软件名称】：Windows 系统专家 V2.5 Full

【保护方式】：注册码 + 启动 NAG + 功能限制

【编译语言】：Microsoft Visual Basic 5.0 / 6.0

—————————————————————————————————

【破解过程】：

1. 用 `PEiD` 查壳，无壳，Microsoft Visual Basic 5.0 / 6.0 编译。

2. 运行主程序注册，输入试炼信息，确认！程序提示注册信息错误，VB 程序无非就只有那么几个断点，这里使用字符比较断点：`bp __vbaStrCmp`

3. 使用 `Ollydbg` 载入主程序，下断：`bp __vbaStrCmp`，回车，`F9` 运行：

```
660E8A03  >  FF7424 08       push dword ptr ss:[esp+8]             ; 这里断下，F9 继续走！

660E8A07    FF7424 08       push dword ptr ss:[esp+8]

660E8A0B    6A 00           push 0

660E8A0D    E8 B426F3FF     call MSVBVM60.__vbaStrComp

660E8A12    C2 0800         retn 8
```

————————————————————————————————————————————

```
EAX 00153710 ASCII "xgK"                                        ; 软件黑名单

ECX 00000000

EDX 001C938C UNICODE "8324ADA5C4250E43A1D731349F6247AA"          ; 软件黑名单对应注册码

EBX 00000001

ESP 0012E99C ASCII "htD"

EBP 0012F804

ESI 0012F8E0

EDI 0012F810

EIP 660E8A03 MSVBVM60.__vbaStrCmp
```

————————————————————————————————————————————

大约 25 次 `F9` 后，寄存器有了变化：

————————————————————————————————————————————

```
EAX 00000000

ECX 00000000

EDX 001D212C UNICODE "8324ADA5C4250E43A1D731349F6247AA"          ; 再次检测注册码

EBX 00000001

ESP 0012FA78

EBP 0012FB7C

ESI 0012FC58

EDI 0012FB88

EIP 660E8A03 MSVBVM60.__vbaStrCmp
```

————————————————————————————————————————————

再 `F9` 一次，打开启动注册界面：

依次输入“用户名”“注册码”：

`/////// 试炼信息 ///////`

用户名：`KuNgBiM`

注册码：`9876543210`

`////////////////////////`

单击“注册”按钮，完成注册(寄存器又有了变化)：

————————————————————————————————————————————

```
EAX 001C97D4 UNICODE "9876543210"                                ; 输入的假码

ECX 001DB15C UNICODE "A43846F4A175F15A08CD2EDAB75023B6"          ; 真注册码

EDX 004B66F4 Windows?004B66F4

EBX 00000000

ESP 0012F5F0 ASCII "qwJ"

EBP 0012F6A4

ESI 001E75C8

EDI 001CB818

EIP 660E8A03 MSVBVM60.__vbaStrCmp
```

————————————————————————————————————————————

`Alt + F9` 返回程序领空：

```
004A7763    8B45 E0         mov eax,dword ptr ss:[ebp-20]

004A7766    8B4D DC         mov ecx,dword ptr ss:[ebp-24]

004A7769    50              push eax

004A776A    51              push ecx                                ; ★这里就可以做内存注册机★

004A776B    FF15 E8104000   call dword ptr ds:[<&MSVBVM60.__vbaStrCmp>] ; MSVBVM60.__vbaStrCmp

004A7771    8BF0            mov esi,eax                              ; 返回到这里，取消所有断点，然后向上看

004A7773    8D55 DC         lea edx,dword ptr ss:[ebp-24]

004A7776    F7DE            neg esi

004A7778    1BF6            sbb esi,esi

004A777A    8D45 E0         lea eax,dword ptr ss:[ebp-20]

004A777D    52              push edx

004A777E    46              inc esi

004A777F    50              push eax

004A7780    6A 02           push 2

004A7782    F7DE            neg esi

004A7784    FF15 F4114000   call dword ptr ds:[<&MSVBVM60.__vbaFreeStrL>]; MSVBVM60.__vbaFreeStrList

004A778A    83C4 0C         add esp,0C

004A778D    8D4D D4         lea ecx,dword ptr ss:[ebp-2C]

004A7790    FF15 84124000   call dword ptr ds:[<&MSVBVM60.__vbaFreeObj>] ; MSVBVM60.__vbaFreeObj

004A7796    66:3BF3         cmp si,bx

004A7799    0F84 D5020000   je Windows?004A7A74                     ; ★爆破点★
```

★★★★★★ 内存注册机 ★★★★★★

中断地址：`4A776A`

中断次数：`1`

第一字节：`51`

指令长度：`1`

内存方式 --> 寄存器 `ECX` --> 宽字符串

★★★★★★★★★★★★★★★★★★

★★★★★★★★★ 爆破注册法 ★★★★★★★★★

```
004A7799    0F84 D5020000   je Windows?004A7A74

改为：（nop 掉）

004A7799    90              nop

004A779A    90              nop

004A779B    90              nop

004A779C    90              nop

004A779D    90              nop

004A779E    90              nop
```

★★★★★★★★★★★★★★★★★★★★★★★★

到此我们已经可以做初级和中级的 Cracker 了，不过该文着重是研究注册算法，所以继续下面的旅程：

```
004A7660    FF51 04         call dword ptr ds:[ecx+4]

004A7663    8B17            mov edx,dword ptr ds:[edi]

004A7665    33DB            xor ebx,ebx

004A7667    57              push edi

004A7668    895D E8         mov dword ptr ss:[ebp-18],ebx

004A766B    895D E4         mov dword ptr ss:[ebp-1C],ebx

004A766E    895D E0         mov dword ptr ss:[ebp-20],ebx

004A7671    895D DC         mov dword ptr ss:[ebp-24],ebx

004A7674    895D D8         mov dword ptr ss:[ebp-28],ebx

004A7677    895D D4         mov dword ptr ss:[ebp-2C],ebx

004A767A    895D C4         mov dword ptr ss:[ebp-3C],ebx

004A767D    895D B4         mov dword ptr ss:[ebp-4C],ebx

004A7680    895D 90         mov dword ptr ss:[ebp-70],ebx

004A7683    895D 8C         mov dword ptr ss:[ebp-74],ebx

004A7686    FF92 FC020000   call dword ptr ds:[edx+2FC]

004A768C    50              push eax                                 ; eax=01AA75AC

004A768D    8D45 D4         lea eax,dword ptr ss:[ebp-2C]

004A7690    50              push eax

004A7691    FF15 94104000   call dword ptr ds:[<&MSVBVM60.__vbaObjSet>] ; MSVBVM60.__vbaObjSet

004A7697    8BF0            mov esi,eax                              ; eax=01AA75AC，esi=0012F730

004A7699    8D55 E0         lea edx,dword ptr ss:[ebp-20]            ; edx=00DA5768

004A769C    52              push edx

004A769D    56              push esi                                 ; eax=01AA75AC

004A769E    8B0E            mov ecx,dword ptr ds:[esi]

004A76A0    FF91 A0000000   call dword ptr ds:[ecx+A0]

004A76A6    3BC3            cmp eax,ebx                              ; eax,ebx 清零

004A76A8    DBE2            fclex                                    ; 清除异常

004A76AA    7D 12           jge short Windows?004A76BE               ; 清除完毕则跳！

004A76AC    68 A0000000     push 0A0

004A76B1    68 24F74100     push Windows?0041F724

004A76B6    56              push esi

004A76B7    50              push eax

004A76B8    FF15 74104000   call dword ptr ds:[<&MSVBVM60.__vbaHresultC>]; MSVBVM60.__vbaHresultCheckObj

004A76BE    8B45 E0         mov eax,dword ptr ss:[ebp-20]            ; 取用户名

004A76C1    50              push eax                                 ; 用户名压栈，eax=001E8B24, (UNICODE "KuNgBiM")

004A76C2    68 68EE4100     push Windows?0041EE68                    ; 取固定字符串，(UNICODE "WSYS1688")

004A76C7    FF15 5C104000   call dword ptr ds:[<&MSVBVM60.__vbaStrCat>]; 字符串连接，MSVBVM60.__vbaStrCat

004A76CD    8BD0            mov edx,eax                              ; eax=001DB584, (UNICODE "KuNgBiMWSYS1688")，edx=2

004A76CF    8D4D E8         lea ecx,dword ptr ss:[ebp-18]            ; ecx=3

004A76D2    FF15 58124000   call dword ptr ds:[<&MSVBVM60.__vbaStrMove>]; MSVBVM60.__vbaStrMove

004A76D8    8D4D E0         lea ecx,dword ptr ss:[ebp-20]

004A76DB    FF15 88124000   call dword ptr ds:[<&MSVBVM60.__vbaFreeStr>]; MSVBVM60.__vbaFreeStr

004A76E1    8D4D D4         lea ecx,dword ptr ss:[ebp-2C]            ; ecx=00149888

004A76E4    FF15 84124000   call dword ptr ds:[<&MSVBVM60.__vbaFreeObj>]; MSVBVM60.__vbaFreeObj

004A76EA    8B0F            mov ecx,dword ptr ds:[edi]               ; ecx=01AA5DBC

004A76EC    57              push edi                                 ; edi=001CB7F0

004A76ED    FF91 00030000   call dword ptr ds:[ecx+300]              ; ★算法 CALL_1★，F7 跟进！
```

——————————————————————————————————————————————

跟进★算法 `CALL_1`★：

```
661043E8    E8 F8190000     call MSVBVM60.66105DE5                   ; 跟进到这里，F7 继续跟进！

661043ED    8D49 00         lea ecx,dword ptr ds:[ecx]

661043F0    E8 F0190000     call MSVBVM60.66105DE5

661043F5    8D49 00         lea ecx,dword ptr ds:[ecx]
```
