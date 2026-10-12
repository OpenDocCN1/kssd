
# 凌鹏光盘出租与销售系统(网络版) 5.0

> 原文：[`www.pediy.com/kssd/pediy06/pediy6230.htm`](https://www.pediy.com/kssd/pediy06/pediy6230.htm)

**软件名称：**

凌鹏光盘出租与销售系统(网络版) 5.0

**下载地址：** `http://www.onlinedown.net/soft/18269.htm`

----------------------------------------------------------------

**破解作者：**

`yzez[DFCG]`

**破解工具：**

`w32dasm`、`0llydbg1.09`

**破解目的：**

本不为破解而破解，只因为技术而破解!

----------------------------------------------------------------

## 破解过程

**详细过程：**

1、用 `w32dasm` 反汇编，查有用信息："注册成功"，双击我们来到这里：

```
:004D76B0 55             push ebp
:004D76B1 8BEC           mov ebp, esp
:004D76B3 B905000000     mov ecx, 00000005

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004D76BD(C)
|
:004D76B8 6A00           push 00000000
:004D76BA 6A00           push 00000000
:004D76BC 49             dec ecx
:004D76BD 75F9           jne 004D76B8
:004D76BF 51             push ecx
:004D76C0 53             push ebx
:004D76C1 56             push esi
:004D76C2 8BD8           mov ebx, eax
:004D76C4 33C0           xor eax, eax
:004D76C6 55             push ebp
:004D76C7 6816784D00     push 004D7816
:004D76CC 64FF30         push dword ptr fs:[eax]
:004D76CF 648920         mov dword ptr fs:[eax], esp
:004D76D2 8D55FC         lea edx, dword ptr [ebp-04]
:004D76D5 8B831C030000   mov eax, dword ptr [ebx+0000031C]
:004D76DB E8A410F7FF     call 00448784====>在这里我们设置好断点！断点不妨设前一点！
:004D76E0 8B45FC         mov eax, dword ptr [ebp-04]
:004D76E3 E88C1DF3FF     call 00409474
:004D76E8 8BF0           mov esi, eax
:004D76EA 8D55F8         lea edx, dword ptr [ebp-08]
:004D76ED 8B8314030000   mov eax, dword ptr [ebx+00000314]
:004D76F3 E88C10F7FF     call 00448784
:004D76F8 8B45F8         mov eax, dword ptr [ebp-08]
:004D76FB E8741DF3FF     call 00409474
:004D7700 03C0           add eax, eax
:004D7702 058F568700     add eax, 0087568F
:004D7707 056B5A8700     add eax, 00875A6B
:004D770C 05A25D4F00     add eax, 004F5DA2
:004D7711 8D0480         lea eax, dword ptr [eax+4*eax]
:004D7714 3BF0           cmp esi, eax===>这是一个比较指令！比较什么？
:004D7716 0F85AF000000   jne 004D77CB===>这里是一个跳转指令，跳向何处？往下看：my god

=====================================>下面一行是什么？"注册失败"？！一定不能跳呀！

:004D771C 6A00           push 00000000

* Possible StringData Ref from Code Obj ->"注册成功"
                                      |
:004D771E B924784D00     mov ecx, 004D7824===>双击我们来到此！我们往上找跳转指令！

* Possible StringData Ref from Code Obj ->"你已经注册成功，现在你可以毫无限制的使用本软件"
                                      ->"了,多谢你的信任!"
                                      |
:004D7723 BA30784D00     mov edx, 004D7830
:004D7728 A120135800     mov eax, dword ptr [00581320]
:004D772D 8B00           mov eax, dword ptr [eax]
:004D772F E8E014F9FF     call 00468C14

* Possible StringData Ref from Code Obj ->"注：本软件已经注册，请你放心使用"
                                      |
:004D7734 BA78784D00     mov edx, 004D7878
:004D7739 8B8308030000   mov eax, dword ptr [ebx+00000308]
:004D773F E87010F7FF     call 004487B4
:004D7744 8B8330030000   mov eax, dword ptr [ebx+00000330]
:004D774A 8B8078010000   mov eax, dword ptr [eax+00000178]
:004D7750 50             push eax
:004D7751 8B00           mov eax, dword ptr [eax]
:004D7753 FF9098000000   call dword ptr [eax+00000098]
:004D7759 E8C2F1F2FF     call 00406920
:004D775E 8D55E4         lea edx, dword ptr [ebp-1C]
:004D7761 8B831C030000   mov eax, dword ptr [ebx+0000031C]
:004D7767 E81810F7FF     call 00448784
:004D776C 8B45E4         mov eax, dword ptr [ebp-1C]
:004D776F E8001DF3FF     call 00409474
:004D7774 8BD0           mov edx, eax
:004D7776 8D45E8         lea eax, dword ptr [ebp-18]
:004D7779 B1FC           mov cl, FC
:004D777B E83CE1F2FF     call 004058BC
:004D7780 FF75F4         push [ebp-0C]
:004D7783 FF75F0         push [ebp-10]
:004D7786 FF75EC         push [ebp-14]
:004D7789 FF75E8         push [ebp-18]
:004D778C 8D45D4         lea eax, dword ptr [ebp-2C]

* Possible StringData Ref from Code Obj ->"注册码"
                                      |
:004D778F BAA4784D00     mov edx, 004D78A4
:004D7794 E813E1F2FF     call 004058AC
:004D7799 FF75E0         push [ebp-20]
:004D779C FF75DC         push [ebp-24]
:004D779F FF75D8         push [ebp-28]
:004D77A2 FF75D4         push [ebp-2C]
:004D77A5 8B8330030000   mov eax, dword ptr [ebx+00000330]
:004D77AB 8B8078010000   mov eax, dword ptr [eax+00000178]
:004D77B1 50             push eax
:004D77B2 8B00           mov eax, dword ptr [eax]
:004D77B4 FF90AC000000   call dword ptr [eax+000000AC]
:004D77BA E861F1F2FF     call 00406920
:004D77BF A1742D5800     mov eax, dword ptr [00582D74]
:004D77C4 E87FDBF8FF     call 00465348
:004D77C9 EB18           jmp 004D77E3

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004D7716(C)
|
:004D77CB 6A00           push 00000000====>上面的跳跳到这里呀！

* Possible StringData Ref from Code Obj ->"注册失败"
                                      |
:004D77CD B9AC784D00     mov ecx, 004D78AC
```

2、用 `0llydbg` 动态调试，设断在：`:004D76DB |. CALL discsyst.00448784`，以下是程序代码：

```
004D76BF  |.  PUSH    ECX
004D76C0  |.  PUSH    EBX
004D76C1  |.  PUSH    ESI
004D76C2  |.  MOV     EBX, EAX
004D76C4  |.  XOR     EAX, EAX
004D76C6  |.  PUSH    EBP
004D76C7  |.  PUSH    discsyst.004D7816
004D76CC  |.  PUSH    DWORD PTR FS:[EAX]
004D76CF  |.  MOV     DWORD PTR FS:[EAX], ESP
004D76D2  |.  LEA     EDX, [LOCAL.1]
004D76D5  |.  MOV     EAX, DWORD PTR DS:[EBX+31C]
004D76DB  |.  CALL    discsyst.00448784===>在此 CALL 中断，按 F8 往下！
004D76E0  |.  MOV     EAX, [LOCAL.1]＝＝＝>输入的假注册码入 EAX
004D76E3  |.  CALL    discsyst.00409474＝＝>此 CALL 把输入的假注册码由十进制转化成十六进制！

＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝>我输入假码：12345678，转换成十六进制为：BC614E！

004D76E8  |.  MOV     ESI, EAX＝＝＝＝＝>把转换后的假码保存在：ESI！
004D76EA  |.  LEA     EDX, [LOCAL.2]
004D76ED  |.  MOV     EAX, DWORD PTR DS:[EBX+314]
004D76F3  |.  CALL    discsyst.00448784
004D76F8  |.  MOV     EAX, [LOCAL.2] ＝＝>机器码移入 EAX，我的机器码为：1021131!
004D76FB  |.  CALL    discsyst.00409474==>此 CALL 把机器码进行转换，由十进制转换成十六进制值：

＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝>F94CB，并把值保存在 EAX 中！

004D7700  |.  ADD     EAX, EAX\==>这里开始计算注册码！EAX＝EAX＋EAX＝F94CB＋F94CB＝1F2996
004D7702  |.  ADD     EAX, 87568F＝＝>EAX＝EAX＋常数：87568F＝1F2996＋87568F＝A68025
004D7707  |.  ADD     EAX, 875A6B＝＝>EAX＝EAX＋常数：875A6B＝1F2996＋875A6B＝12DDA90
004D770C  |.  ADD     EAX, discsyst.004F5DA2==>EAX＝EAX＋地址值：4F5DA2＝12DDA90＋4F5DA2＝17D3832
004D7711  |.  LEA     EAX, DWORD PTR DS:[EAX+EAX*4\]==>地址传送指令，把 DS:[EAX+EAX*4\]的值移入 EAX，

＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝>EAX\=[EAX+EAX*4\]＝17D3832＋17D3832*4＝77218FA

004D7714  |.  CMP     ESI, EAX\===>比较 ESI 与 EAX 的值，ESI 是：BC614E，EAX 的值是：77218FA
004D7716  |.  JNZ     discsyst.004D77CB＝＝＞不相等则跳，一跳就送你见上帝！阿弥陀佛不能跳！
004D771C  |.  PUSH    0
```
