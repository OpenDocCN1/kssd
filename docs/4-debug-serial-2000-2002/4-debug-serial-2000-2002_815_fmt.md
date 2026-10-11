
# Gif2Swf2.1

> 原文：[`www.pediy.com/kssd/pediy04/kanxue178.htm`](https://www.pediy.com/kssd/pediy04/kanxue178.htm)

## Gif2Swf2.1 注册算法分析

好久没有写破解文章了，手都有点生了。现在中国破解界新人辈出，我也觉得我要跟不上时代了 ^\_^

本来想一直等到 `CHiNA CrACKiNG GrOUp` 出新的 `CrackMe` 再来练练手，但是 `SunBird` 老大可能比较忙，新的 `CrackMe` 迟迟不出，所以从网上下载了一个 `Gif2Swf2.1` 来练练手 ^\_^，也不知道是不是有人已经作出 `KeyGen` 了。

用 `TRW` 载入程序，到输入注册码的对话框中输入注册信息：

*   用户名：`NYDoll`
*   注册码：`38383838`

下断点 `BPX LOCKMYTASK`。

点击确定按钮，程序被拦下。`F10` 单步跟踪，直到看到下面的代码：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004079C4(C)
|

:004079CF 83FF01                  cmp edi, 00000001
:004079D2 5F                      pop edi
:004079D3 0F85A5000000            jne 00407A7E
:004079D9 E842BDFFFF              call 00403720   /关键 Call 要知道注册算法就要跟进这个 Call，具体参见附录一
:004079DE 85C0                    test eax, eax
:004079E0 0F8485000000            je 00407A6B      /标志位比对 若注册码不正确则跳转到注册失败对话框
:004079E6 6A00                    push 00000000
:004079E8 8BCB                    mov ecx, ebx
:004079EA C7056CC5410001000000    mov dword ptr [0041C56C], 00000001

* Reference To: MFC42.Ordinal:0A55, Ord:0A55h
                          |
:004079F4 E8A17A0000          Call 0040F49A
:004079F9 A168C54100          mov eax, dword ptr [0041C568]
:004079FE 85C0                 test eax, eax
:00407A00 743A                 je 00407A3C
:00407A02 A1E03A4100          mov eax, dword ptr [00413AE0]
:00407A07 6A00                 push 00000000
:00407A09 6A00                 push 00000000
:00407A0B 6810010000          push 00000110
:00407A10 50                   push eax

* Reference To: USER32.SendMessageA, Ord:0214h
                          |
:00407A11 FF1500044100        Call dword ptr [00410400]
:00407A17 6A40                 push 00000040

* Possible StringData Ref from Data Obj ->"Congratulations"  /注册成功则出现如下信息
                          |
:00407A19 6800374100          push 00413700

* Possible StringData Ref from Data Obj ->"GIF2SWF has been successfuly registered "
                          ->"!"
                          |
:00407A1E 68D4364100          push 004136D4
:00407A23 6A00                 push 00000000

* Reference To: USER32.MessageBoxA, Ord:01BEh
                          |
:00407A25 FF15FC034100        Call dword ptr [004103FC]
:00407A2B 5E                   pop esi
:00407A2C 5B                   pop ebx
:00407A2D 8B4C2404             mov ecx, dword ptr [esp+04]
:00407A31 64890D00000000       mov dword ptr fs:[00000000], ecx
:00407A38 83C410               add esp, 00000010
:00407A3B C3                   ret

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00407A00(C)
|
:00407A3C 6A40                 push 00000040

* Possible StringData Ref from Data Obj ->"Congratulations"
                          |
:00407A3E 6800374100          push 00413700

* Possible StringData Ref from Data Obj ->"GIF2SWF has been successfuly registered "
                          ->"!"
                          |
:00407A43 68D4364100          push 004136D4
:00407A48 6A00                 push 00000000

* Reference To: USER32.MessageBoxA, Ord:01BEh
                          |
:00407A4A FF15FC034100        Call dword ptr [004103FC]
:00407A50 6A00                 push 00000000
:00407A52 E8F9B7FFFF          call 00403250
:00407A57 83C404               add esp, 00000004
:00407A5A 5E                   pop esi
:00407A5B 5B                   pop ebx
:00407A5C 8B4C2404             mov ecx, dword ptr [esp+04]
:00407A60 64890D00000000       mov dword ptr fs:[00000000], ecx
:00407A67 83C410               add esp, 00000010
:00407A6A C3                   ret

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004079E0(C)
|
:00407A6B 6A30                 push 00000030

* Possible StringData Ref from Data Obj ->"Registration"  /注册失败则显示如下信息
                          |
:00407A6D 68C4364100          push 004136C4

* Possible StringData Ref from Data Obj ->"You have entered and incorrect "
                          ->"Name or Serial number
Please "
                          ->"try again !"
```

### 附录一

```
* Referenced by a CALL at Addresses:
|:00404168  , :004079D9
|

:00403720 6AFF                   push FFFFFFFF
:00403722 68E8F94000             push 0040F9E8
:00403727 64A100000000           mov eax, dword ptr fs:[00000000]
:0040372D 50                     push eax
:0040372E 64892500000000         mov dword ptr fs:[00000000], esp
:00403735 83EC08                 sub esp, 00000008
:00403738 51                     push ecx
:00403739 8BCC                   mov ecx, esp
:0040373B 89642404               mov dword ptr [esp+04], esp
:0040373F 68D83A4100             push 00413AD8

* Reference To: MFC42.Ordinal:0217, Ord:0217h
                          |
:00403744 E827BD0000             Call 0040F470
:00403749 51                     push ecx
:0040374A C744241800000000       mov [esp+18], 00000000
:00403752 8BCC                   mov ecx, esp
:00403754 8964240C               mov dword ptr [esp+0C], esp
:00403758 68DC3A4100             push 00413ADC

* Reference To: MFC42.Ordinal:0217, Ord:0217h
                          |
:0040375D E80EBD0000             Call 0040F470
:00403762 C7442418FFFFFFFF       mov [esp+18], FFFFFFFF
:0040376A E841FEFFFF             call 004035B0  /关键 Call
:0040376F 83C408                 add esp, 00000008
:00403772 85C0                   test eax, eax
:00403774 7544                   jne 004037BA  /跟到这里的时候发现 若在这里跳转则跳出当前 Call 故怀疑 call 004035B0 为关键 Call 按 F8 跟入 具体参见附录二
:00403776 B9DC3A4100             mov ecx, 00413ADC

* Reference To: MFC42.Ordinal:104B, Ord:104Bh
                          |
:0040377B E8EABC0000             Call 0040F46A
:00403780 50                     push eax

* Possible StringData Ref from Data Obj ->"RegisteredUserName"  /推测是将验证成功的用户名信息存入注册表
                          |
:00403781 686C354100             push 0041356C
:00403786 E805FDFFFF             call 00403490
:0040378B 83C408                 add esp, 00000008
:0040378E B9D83A4100             mov ecx, 00413AD8

* Reference To: MFC42.Ordinal:104B, Ord:104Bh
                          |
:00403793 E8D2BC0000             Call 0040F46A
:00403798 50                     push eax

* Possible StringData Ref from Data Obj ->"RegisteredUserKey"  /推测是将验证成功的注册码信息存入注册表
                          |
:00403799 6858354100             push 00413558
:0040379E E8EDFCFFFF             call 00403490
:004037A3 83C408                 add esp, 00000008
:004037A6 B801000000             mov eax, 00000001
:004037AB 8B4C2408               mov ecx, dword ptr [esp+08]
:004037AF 64890D00000000         mov dword ptr fs:[00000000], ecx
:004037B6 83C414                 add esp, 00000014
:004037B9 C3                     ret
```

### 附录二

```
* Referenced by a CALL at Addresses:
|:0040376A  , :00403E22
|

:004035B0 6AFF                   push FFFFFFFF
:004035B2 68D0F94000             push 0040F9D0
:004035B7 64A100000000           mov eax, dword ptr fs:[00000000]
:004035BD 50                     push eax
```



```
:004035BE 64892500000000           mov dword ptr fs:[00000000], esp

:004035C5 83EC64                  sub esp, 00000064

:004035C8 55                      push ebp

:004035C9 56                      push esi

:004035CA 57                      push edi

:004035CB 8D8C2480000000          lea ecx, dword ptr [esp+00000080]

:004035D2 C744247801000000        mov [esp+78], 00000001

* Reference To: MFC42.Ordinal:106C, Ord:106Ch
                                |
:004035DA E8A7BD0000             Call 0040F386

:004035DF 8D8C2480000000          lea ecx, dword ptr [esp+00000080]

* Reference To: MFC42.Ordinal:104B, Ord:104Bh
                                |
:004035E6 E87FBE0000             Call 0040F46A

:004035EB 8D8C2484000000          lea ecx, dword ptr [esp+00000084]

:004035F2 8BF0                    mov esi, eax    /用户名入栈

* Reference To: MFC42.Ordinal:104B, Ord:104Bh
                                |
:004035F4 E871BE0000             Call 0040F46A  --------\

:004035F9 8BE8                    mov ebp, eax                   \

:004035FB 8BFE                    mov edi, esi

:004035FD 83C9FF                  or ecx, FFFFFFFF

:00403600 33C0                    xor eax, eax                       很常见到一种通过循环获得用户名字符数和比对是否为零的代码段

:00403602 F2                      repnz

:00403603 AE                      scasb

:00403604 F7D1                    not ecx                       /

:00403606 49                      dec ecx                     /

:00403607 0F84D2000000             je 004036DF  ----------

:0040360D 8BFD                    mov edi, ebp

:0040360F 83C9FF                  or ecx, FFFFFFFF

:00403612 F2                      repnz

:00403613 AE                      scasb

:00403614 F7D1                    not ecx  /先取反 在减一 等到字符串的位数 常见方法

:00403616 49                      dec ecx  /好像使用 MFC 编写的程序都使用这样的方法验证字符串的个数

:00403617 0F84C2000000             je 004036DF               /比对注册码是否为零

:0040361D 53                      push ebx

:0040361E 8BFE                    mov edi, esi

:00403620 83C9FF                  or ecx, FFFFFFFF

:00403623 33DB                    xor ebx, ebx

:00403625 F2                      repnz

:00403626 AE                      scasb

:00403627 F7D1                    not ecx

:00403629 49                      dec ecx

:0040362A 83F920                  cmp ecx, 00000020      /比对用户名位数是否大于 32，若小于则正常计算 若大于则只计算用户名的前 32 为

:0040362D 7E05                    jle 00403634    /小于等于则跳转

:0040362F B920000000               mov ecx, 00000020    /将用户名字符数强行赋值为 32

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040362D(C)
|
:00403634 33F6                    xor esi, esi    /ESI 寄存器清零

:00403636 85C9                    test ecx, ecx

:00403638 7E1E                    jle 00403658

:0040363A B887D61200               mov eax, 0012D687  /EAX 寄存器赋值为 1234567

:0040363F 99                      cdq

:00403640 F7F9                    idiv ecx        /EAX 寄存器的值除以 ECX 寄存器的值 设结果为常量 a，ECX 寄存器中保存的是用户名的位数

:00403642 8B942484000000          mov edx, dword ptr [esp+00000084]  /载入用户名

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00403656(C)
|
:00403649 0FBE3C16                movsx edi, byte ptr [esi+edx]  /依次取用户名的大写字符的 ASCII 码参与计算

:0040364D 0FAFF8                  imul edi, eax  /用当前字符的 ASCII 码与常量 a 相乘

:00403650 03DF                    add ebx, edi  /结果累加进 EBX 寄存器中

:00403652 46                      inc esi    /计数器加一

:00403653 40                      inc eax    /循环位数加一

:00403654 3BF1                    cmp esi, ecx    /对比循环次数和用户名位数 相等则认为循环结束 跳出循环 不相等则继续

:00403656 7CF1                    jl 00403649

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00403638(C)
|
:00403658 81C31FF97E00             add ebx, 007EF91F  /累加结果加上 8321311

:0040365E 8D442410                lea eax, dword ptr [esp+10]

:00403662 53                      push ebx

* Possible StringData Ref from Data Obj ->"%i"
                                |
:00403663 6854354100               push 00413554

:00403668 50                      push eax

* Reference To: MSVCRT.sprintf, Ord:02B2h
                                |
:00403669 FF156C034100             Call dword ptr [0041036C]

:0040366F 83C40C                  add esp, 0000000C

:00403672 8BF5                    mov esi, ebp

:00403674 8D442410                lea eax, dword ptr [esp+10]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040369A(C)
|
:00403678 8A10                    mov dl, byte ptr [eax]    /循环比对 Fuck SN 和 Real SN 是否相等

:0040367A 8A1E                    mov bl, byte ptr [esi]

:0040367C 8ACA                    mov cl, dl

:0040367E 3AD3                    cmp dl, bl

:00403680 751E                    jne 004036A0

:00403682 84C9                    test cl, cl

:00403684 7416                    je 0040369C

:00403686 8A5001                  mov dl, byte ptr [eax+01]

:00403689 8A5E01                  mov bl, byte ptr [esi+01]

:0040368C 8ACA                    mov cl, dl

:0040368E 3AD3                    cmp dl, bl

:00403690 750E                    jne 004036A0

:00403692 83C002                  add eax, 00000002

:00403695 83C602                  add esi, 00000002

:00403698 84C9                    test cl, cl

:0040369A 75DC                    jne 00403678

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00403684(C)
|
:0040369C 33F6                    xor esi, esi

:0040369E EB05                    jmp 004036A5
```

注册机正在编译中………………

`------------ Gif2Swf2.1 Cracked ------------------`

娃娃（NYDoll）

属于中国破解组织 CCG(CHiNA CrACKiNG GrOUp)

仅以此文献给我们可爱的组织 CCG 希望它能蒸蒸日上

特别献给新兴组织 CNCG `4dd466d0e1e0cc123afbe504da2462ca`
