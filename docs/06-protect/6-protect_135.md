
# WeaveMacker V7.55

> 原文：[`www.pediy.com/kssd/pediy04/kanxue235.htm`](https://www.pediy.com/kssd/pediy04/kanxue235.htm)

WeaveMacker V7.55 破解

作者：crackjack  
来源：`http://tools.85560046.com/cgi-bin/wzgl/show.cgi?type=CNCG&id=20011112021153`  
日期：2001-11-12

这个软件是用圣天诺狗来加密的，当软件没有检测到加密狗时，则需要注册它。好，我们看看怎么样来破解它。

工具：`trw2000`、`hview`

软件在运行时，如果没有狗，则会跳出一个需要注册的对话框，而且也是用动态注册的方法，如果注册码不正确，`register` 按钮是灰色的。这类软件一般用断点 `bpx hmemcpy`，今天我们用另外的断点来破它，用什么断点呢？想一下要取得对话框的数据要用到什么函数，对了，是 `GetDlgItemTextA`。好，我们先输入用户名，然后输入假注册码 `12345678`，切换到 TRW，下断点 `bpx GetDlgItemTextA`，按 `F5` 返回程序，马上被中断了，按 `F10` 返回到主程序，如下：

```
:004C6798 8D852C010000             lea eax, dword ptr [ebp+0000012C]  <===返回到这里,用 d eax 可以看到我们输入的假注册码
:004C679E 50                       push eax
:004C679F 6A06                     push 00000006
:004C67A1 E8BAFFFEFF               call 004B6760
:004C67A6 83C410                   add esp, 00000010
:004C67A9 E99E000000               jmp 004C684C
------------------------------------------------
```

按几下 `F10`,到下面:

```
:004C684C 55                       push ebp
:004C684D E84E010000               call 004C69A0  <====计算和比较注册码的 CALL,所以要 F8 进入
:004C6852 83C404                   add esp, 00000004
:004C6855 85C0                     test eax, eax
:004C6857 7427                     je 004C6880
:004C6859 8B8DEC020000             mov ecx, dword ptr [ebp+000002EC]
:004C685F 51                       push ecx
:004C6860 E86B000000               call 004C68D0
:004C6865 6A00                     push 00000000
:004C6867 6A07                     push 00000007
:004C6869 E8C205FFFF               call 004B6E30
:004C686E 83C40C                   add esp, 0000000C
:004C6871 B801000000               mov eax, 00000001
:004C6876 5F                       pop edi
:004C6877 5E                       pop esi
:004C6878 5D                       pop ebp
:004C6879 81C498000000             add esp, 00000098
:004C687F C3                       ret
---------------------------------------------------
```

进入 `4C684D` 的 `CALL`:

```
:004C69A0 81EC18020000             sub esp, 00000218
:004C69A6 53                       push ebx
:004C69A7 55                       push ebp
:004C69A8 8BAC2424020000           mov ebp, dword ptr [esp+00000224]
:004C69AF 56                       push esi
:004C69B0 57                       push edi

* Possible StringData Ref from Data Obj ->"Demo"
                                 |
:004C69B1 BFD0115300               mov edi, 005311D0  <====取试用的注册码,这个注册码可以用到 2001-12-31 日,这不是我们所要的,继续 GO
:004C69B6 8DB52C010000             lea esi, dword ptr [ebp+0000012C] <====用 d esi 可以看到我们输入的假注册码
:004C69BC 8BC6                     mov eax, esi

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C69E0(C)
|
:004C69BE 8A10                     mov dl, byte ptr [eax]
:004C69C0 8A1F                     mov bl, byte ptr [edi]
:004C69C2 8ACA                     mov cl, dl
:004C69C4 3AD3                     cmp dl, bl <=====比较注册码
:004C69C6 751E                     jne 004C69E6  <=====不相等则跳到正式用户注册码运算的代码
:004C69C8 84C9                     test cl, cl
:004C69CA 7416                     je 004C69E2
:004C69CC 8A5001                   mov dl, byte ptr [eax+01]
------------------------------------------------
```

```
* Possible StringData Ref from Data Obj ->"Never"
                                 |
:004C6A82 BF10135300               mov edi, 00531310    <====取字符串 Never
:004C6A87 83C9FF                   or ecx, FFFFFFFF
:004C6A8A 8985EC020000             mov dword ptr [ebp+000002EC], eax
:004C6A90 C744241800000000          mov [esp+18], 00000000
:004C6A98 F2                       repnz
:004C6A99 AE                       scasb
:004C6A9A F7D1                     not ecx
:004C6A9C 2BF9                     sub edi, ecx
:004C6A9E 8BC1                     mov eax, ecx
:004C6AA0 8BF7                     mov esi, edi
:004C6AA2 8BFA                     mov edi, edx
:004C6AA4 C1E902                   shr ecx, 02
:004C6AA7 F3                       repz
:004C6AA8 A5                       movsd
:004C6AA9 8BC8                     mov ecx, eax
:004C6AAB 83E103                   and ecx, 00000003
:004C6AAE F3                       repz
:004C6AAF A4                       movsb

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C6A6B(U)
|
:004C6AB0 8D4C241C                 lea ecx, dword ptr [esp+1C]
:004C6AB4 51                       push ecx
:004C6AB5 53                       push ebx
:004C6AB6 E855090000               call 004C7410
:004C6ABB 8D5528                   lea edx, dword ptr [ebp+28]  <====取字符串 22
:004C6ABE 8DB544020000             lea esi, dword ptr [ebp+00000244]
:004C6AC4 52                       push edx
:004C6AC5 8D8594020000             lea eax, dword ptr [ebp+00000294]  <====取用户名
:004C6ACB 56                       push esi
:004C6ACC 8D4D3C                   lea ecx, dword ptr [ebp+3C]
:004C6ACF 50                       push eax
:004C6AD0 8D9424AC000000            lea edx, dword ptr [esp+000000AC]
:004C6AD7 51                       push ecx
:004C6AD8 52                       push edx
:004C6AD9 E862050000               call 004C7040  <====连接成运算码,比如输入的用户名为 crackjack,则连接成 22crackjack(0DH)Never(0DH)22
---------------------------------------------
```

一直走,走到:

```
:004C6B22 8D442434                 lea eax, dword ptr [esp+34]
:004C6B26 56                       push esi
:004C6B27 8D8C24AC000000            lea ecx, dword ptr [esp+000000AC]
:004C6B2E 50                       push eax
:004C6B2F 51                       push ecx
:004C6B30 E88B060000               call 004C71C0  <=====运算注册码的 CALLL,F8 进入
:004C6B35 8B44242C                 mov eax, dword ptr [esp+2C]  <=====取运算出来的注册码
:004C6B39 8B4C2438                 mov ecx, dword ptr [esp+38]  <=====取我们输入的注册码,我们可以在这里暴破它,改为 mov ecx, dword ptr [esp+2C]
:004C6B3D 83C41C                   add esp, 0000001C
:004C6B40 3BC8                     cmp ecx, eax    <=====比较注册码
:004C6B42 8985F0020000             mov dword ptr [ebp+000002F0], eax
:004C6B48 898DF4020000             mov dword ptr [ebp+000002F4], ecx
:004C6B4E 7510                     jne 004C6B60
:004C6B50 5F                       pop edi
:004C6B51 5E                       pop esi
:004C6B52 5D                       pop ebp
:004C6B53 B801000000               mov eax, 00000001
:004C6B58 5B                       pop ebx
:004C6B59 81C418020000             add esp, 00000218
:004C6B5F C3                       ret
-----------------------------------------------
```

```
:004C7260 6A00                     push 00000000
:004C7262 C1E00D                   shl eax, 0D
:004C7265 0BC6                     or eax, esi
:004C7267 8B74242C                 mov esi, dword ptr [esp+2C]
:004C726B 55                       push ebp
:004C726C 68F4535300               push 005353F4
:004C7271 8906                     mov dword ptr [esi], eax
:004C7273 8B442424                 mov eax, dword ptr [esp+24]
:004C7277 50                       push eax
:004C7278 E813FFFFFF               call 004C7190  <====F8 进入
:004C727D 8B5C2440                 mov ebx, dword ptr [esp+40]
:004C7281 83C410                   add esp, 00000010
:004C7284 85DB                     test ebx, ebx

----------------------------------------------------

:004C719C F2                       repnz
:004C719D AE                       scasb
:004C719E F7D1                     not ecx
:004C71A0 49                       dec ecx
:004C71A1 51                       push ecx
:004C71A2 52                       push edx
:004C71A3 E8584A0000               call 004CBC00  <=====F8 进入
```



#### 汇编代码段

```
:004C71A8 8B4C2418           mov ecx, dword ptr [esp+18]   

:004C71AC 83C408             add esp, 0000008   

:004C71AF 8901               mov dword ptr [ecx], eax   

:004C71B1 5F                 pop edi   

-------------------------------------------------------
```

下面就是运算注册码的代码，这个软件的运算有点特别，它用程序本身的代码数据（相当于密码表）来算出注册码，只要改一下所用到的数据，就让破解者的注册机无效了

#### 运算注册码代码分析

```
:004CBC00 56                 push esi   

:004CBC01 8B74240C           mov esi, dword ptr [esp+0C]   

:004CBC05 83C8FF             or eax, FFFFFFFF <====EAX 的初始值为 0FFFFFFFF   

:004CBC08 85F6               test esi, esi   

:004CBC0A 7E24               jle 004CBC30   

:004CBC0C 8B4C2408           mov ecx, dword ptr [esp+08]   

:004CBC10 57                 push edi   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:004CBC2D(C)   

|   

:004CBC11 33D2               xor edx, edx     <=====清 0   

:004CBC13 8BF8               mov edi, eax     <=====EDI=EAX   

:004CBC15 8A11               mov dl, byte ptr [ecx] <=====取第 N 个字符   

:004CBC17 81E7FF000000       and edi, 000000FF  <=====取 EDI 的最低位   

:004CBC1D 33D7               xor edx, edi       <=====EDX xor EDI   

:004CBC1F C1E808             shr eax, 08        <=====结果右移 8 位   

:004CBC22 8B1495D4145300     mov edx, dword ptr [4*edx+005314D4] <====取程序中地址为 EDX*4+5314d4 的代码数据,我们知道,字符的最大数是 FF,则密码表的长度为 4*FF+5314d4=5318d0,我们用 Hview 打开主文件,可知它的地址为 1314d4--1318d0,这个地址范围的数值就做为运算注册码的密码   

:004CBC29 33C2               xor eax, edx  <====EAX xor EDX,结果作为下一个字符的运算参数   

:004CBC2B 41                 inc ecx   

:004CBC2C 4E                 dec esi   

:004CBC2D 75E2               jne 004CBC11  <====运算完了吗,没有则继续运算   

:004CBC2F 5F                 pop edi   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:004CBC0A(C)   

|   

:004CBC30 F7D0               not eax  <=====将结果取非,最后的结果就是注册码的十六进制,比如 EAX=29C8BF83,则注册码为 701022083   

:004CBC32 5E                 pop esi   

:004CBC33 C3                 ret
```

由于密码表的数据比较多，我就不编写他的注册机了，只是用暴破的方法来处理它，哪位朋友有兴趣可以编出它的注册机吧，让大家来学习一下嘛. `4dd466d0e1e0cc123afbe504da2462ca`
