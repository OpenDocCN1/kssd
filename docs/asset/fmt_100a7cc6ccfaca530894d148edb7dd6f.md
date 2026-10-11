
# 谈谈 NP 和 HS 的通用 unhook

> 原文：[`www.pediy.com/kssd/pediy10/59324.html`](https://www.pediy.com/kssd/pediy10/59324.html)

祝各位新年快乐![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸") ![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")

一边听音乐一边写代码真的很舒服阿很舒服

先简单说以下，`NP` 和 `HS` 都 hook 了以下一些 `SSDT`

什么是 `SSDT`？我不想说那些复杂的理论，希望以简单的事实帮助介绍

先向大 S 道歉，下文如有冒犯，还请大 S 原谅![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")

怎么说呢，`SSDT` 本身就是一个地址表，我们当他是个地址簿，也就是写了每个人的地址的记录表

今天，偶想给大 S 送新年礼物，怎么送到大 S 家里呢？就得查看地址簿了

上面是这样写的  大 S： 游戏达人，外挂高人，破解牛人，脱壳圣人；分身有如月，堀北真希，花泽类，猪的理想；实际上是个阳光宝宝，吃的的白白胖胖的，每天都很开心；家住上海市 X 区 X 路 X 楼，进门左转、左转、再左转，敲门说找 S.H.E 就行了。

标准 `SSDT` hook 的方式就是改变这个地址，`HS` 就是八地址改写成 `abnlab` 实验室，如果你照着走就到 `abnlab` 实验室了

他们就要检查你了，这么大一箱礼物？是不是带了炸弹阿，识别你的身份阿，但是这样怎么能行呢，经过他们一审核

我的新年礼物岂不是要过年以后才能到了？不行不行

只能先联系大 S，大 S 说家在上海 X 别墅区 X，偶于是就上门了，到了别墅，门卫不让偶进去阿，理由是偶没有邀请函

```
; MmDeleteTeb(x,x)+Ep ...

.text:004129B3
.text:004129B3 BugCheckParameter1= dword ptr  8
.text:004129B3 arg_4               = dword ptr  0Ch

.text:004129B3 ; FUNCTION CHUNK AT .text:00445DD6 SIZE 00000049 BYTES

.text:004129B3
.text:004129B3                   mov     edi, edi
.text:004129B5                   push    ebp
.text:004129B6                   mov     ebp, esp
.text:004129B8                   push    esi
.text:004129B9                   push    edi
.text:004129BA                   mov     eax, large fs:124h
.text:004129C0                   mov     edi, [ebp+BugCheckParameter1]
.text:004129C3                   mov     esi, eax
.text:004129C5                   cmp     [esi+44h], edi
.text:004129C8                   jz      short loc_4129FF
.text:004129CA                   cmp     byte ptr [esi+165h], 0
.text:004129D1                   jnz     loc_445DD6
.text:004129D7                   mov     eax, large fs:994h
.text:004129DD                   test    eax, eax
.text:004129DF                   jnz     loc_445DD6
.text:004129E5                   call    ds:__imp__KeRaiseIrqlToDpcLevel@0 ; KeRaiseIrqlToDpcLevel()
.text:004129EB                   mov     byte ptr [ebp+BugCheckParameter1], al
.text:004129EE                   lea     eax, [esi+14Ch]
.text:004129F4                   push    eax
.text:004129F5                   push    [ebp+BugCheckParameter1]
.text:004129F8                   push    edi
.text:004129F9                   push    esi
.text:004129FA                   call    _KiAttachProcess@16 ; KiAttachProcess(x,x,x,x)
.text:004129FF
.text:004129FF loc_4129FF:                           ; CODE XREF: KeAttachProcess(x)+15j
.text:004129FF                   pop     edi
.text:00412A00                   pop     esi
.text:00412A01                   pop     ebp
.text:00412A02                   retn    4
.text:00412A02 _KeAttachProcess@4 endp
.text:004189DB ; __stdcall KeStackAttachProcess(x, x)
.text:004189DB                   public _KeStackAttachProcess@8
.text:004189DB _KeStackAttachProcess@8 proc near     ; CODE XREF: MmAttachSession(x,x)+58p
.text:004189DB                                       ; MiAttachToSecureProcessInSession(x)+43p ...
.text:004189DB
.text:004189DB arg_0               = dword ptr  8
.text:004189DB arg_4               = dword ptr  0Ch

.text:004189DB
.text:004189DB                   mov     edi, edi
.text:004189DD                   push    ebp
.text:004189DE                   mov     ebp, esp
.text:004189E0                   push    esi
.text:004189E1                   push    edi
.text:004189E2                   mov     eax, large fs:124h
.text:004189E8                   mov     esi, eax
.text:004189EA                   mov     eax, large fs:994h
.text:004189F0                   test    eax, eax
.text:004189F2                   jnz     loc_445DF1
.text:004189F8                   mov     edi, [ebp+arg_0]
.text:004189FB                   cmp     [esi+44h], edi
.text:004189FE                   jz      short loc_418A34
.text:00418A00                   call    ds:__imp__KeRaiseIrqlToDpcLevel@0 ; KeRaiseIrqlToDpcLevel()
.text:00418A06                   cmp     byte ptr [esi+165h], 0
.text:00418A0D                   mov     byte ptr [ebp+arg_0], al
.text:00418A10                   jnz     loc_445E0D
.text:00418A16                   lea     eax, [esi+14Ch]
.text:00418A1C                   push    eax
.text:00418A1D                   push    [ebp+arg_0]
.text:00418A20                   push    edi
.text:00418A21                   push    esi
.text:00418A22                   call    _KiAttachProcess@16 ; KiAttachProcess(x,x,x,x)
.text:00418A27                   mov     eax, [ebp+arg_4]
.text:00418A2A                   and     dword ptr [eax+10h], 0
.text:00418A2E
.text:00418A2E loc_418A2E:                           ; CODE XREF: KeStackAttachProcess(x,x)+63j
.text:00418A2E                                       ; KeAttachProcess(x)+33467j
.text:00418A2E                   pop     edi
.text:00418A2F                   pop     esi
.text:00418A30                   pop     ebp
.text:00418A31                   retn    8
```

我们有以下办法

1.直接执行函数＋5 字节处

这样可以，但是有其他问题，如果 `NP` 把函数 hook 改到函数中间或者末尾，我们很难定位

2.执行底层函数 `KiAttachProcess`

`KiAttachProcess(EPROCESS *Process,Irql)`

```
.text:00412855 ; __stdcall KiAttachProcess(x, x, x, x)
.text:00412855 _KiAttachProcess@16 proc near         ; CODE XREF: KeAttachProcess(x)+47p
.text:00412855                                       ; KeStackAttachProcess(x,x)+47p ...
.text:00412855
.text:00412855 arg_0               = dword ptr  8
.text:00412855 arg_4               = dword ptr  0Ch
.text:00412855 arg_8               = byte ptr  10h
.text:00412855 arg_C               = dword ptr  14h

.text:00412855
.text:00412855 ; FUNCTION CHUNK AT .text:00445D4F SIZE 0000001E BYTES
.text:00412855 ; FUNCTION CHUNK AT .text:00445D72 SIZE 00000064 BYTES

.text:00412855
.text:00412855                   mov     edi, edi
.text:00412857                   push    ebp
.text:00412858                   mov     ebp, esp
.text:0041285A                   push    ebx
.text:0041285B                   mov     ebx, [ebp+arg_4]
.text:0041285E                   inc     word ptr [ebx+60h]
.text:00412862                   push    esi
.text:00412863                   mov     esi, [ebp+arg_0]
.text:00412866                   push    edi
.text:00412867                   push    [ebp+arg_C]
.text:0041286A                   lea     edi, [esi+34h]
.text:0041286D                   push    edi
.text:0041286E                   call    _KiMoveApcState@8 ; KiMoveApcState(x,x)
.text:00412873                   mov     [edi+4], edi
.text:00412876                   mov     [edi], edi
.text:00412878                   lea     eax, [esi+3Ch]
.text:0041287B                   mov     [eax+4], eax
.text:0041287E                   mov     [eax], eax
.text:00412880                   lea     eax, [esi+14Ch]
.text:00412886                   cmp     [ebp+arg_C], eax
.text:00412889                   mov     [esi+44h], ebx
.text:0041288C                   mov     byte ptr [esi+48h], 0
.text:00412890                   mov     byte ptr [esi+49h], 0
.text:00412894                   mov     byte ptr [esi+4Ah], 0
.text:00412898                   jnz     short loc_4128AD
.text:0041289A                   mov     [esi+138h], eax
.text:004128A0                   mov     [esi+13Ch], edi
.text:004128A6                   mov     byte ptr [esi+165h], 1
.text:004128AD
.text:004128AD loc_4128AD:                           ; CODE XREF: KiAttachProcess(x,x,x,x)+43j
.text:004128AD                   cmp     byte ptr [ebx+65h], 0
.text:004128B1                   jnz     loc_445D72
.text:004128B7                   lea     esi, [ebx+40h]
.text:004128BA
```
