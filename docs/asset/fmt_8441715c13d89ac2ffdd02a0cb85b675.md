
# BubbleKing V2.63

> 原文：[`www.pediy.com/kssd/pediy06/pediy6529.htm`](https://www.pediy.com/kssd/pediy06/pediy6529.htm)

BubbleKing V2.63 完全静态破解

下载地址: [`www2.skycn.com/soft/17495.html`](http://www2.skycn.com/soft/17495.html)

刚刚放假，先找一个简单的小游戏来练练手。检查一下，无壳，VC6.0， very good...

用 `W32Dasm` 看了看，发现了大概的思路。但有几个调用的函数显不出来，于是换用 `IDA`，结果一目了然了。

```
.text:0040597D sub_40597D     proc near                 ; DATA XREF: .rdata:00408BF4o
.text:0040597D                 push    esi
.text:0040597E                 mov     esi, ecx
.text:00405980                 push    1
.text:00405982                 call    ?UpdateData@CWnd@@QAEHH@Z ; CWnd::UpdateData(int)
.text:00405987                 mov     eax, [esi+64h]
.text:0040598A                 mov     eax, [eax-8]    ;得到 NAME 长度
.text:0040598D                 test    eax, eax
.text:0040598F                 jnz     short loc_40599F;长度不能为 0
.text:00405991                 push    40h
.text:00405993                 push    offset aWarning ; "Warning"
.text:00405998                 push    offset aPleaseEnterYou ; "Please enter your name first!"
.text:0040599D                 jmp     short loc_4059DC
.text:0040599F ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0040599F
.text:0040599F loc_40599F:                            ; CODE XREF: sub_40597D+12j
.text:0040599F                 cmp     eax, 28h
.text:004059A2                 jl      short loc_4059B2;长度不能大于 28h
.text:004059A4                 push    40h
.text:004059A6                 push    offset aWarning ; "Warning"
.text:004059AB                 push    offset aYourNameIsTooL ; "Your name is too long ^_^"
.text:004059B0                 jmp     short loc_4059DC
.text:004059B2 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:004059B2
.text:004059B2 loc_4059B2:                            ; CODE XREF: sub_40597D+25j
.text:004059B2                 lea     eax, [esi+60h]
.text:004059B5                 push    eax
.text:004059B6                 call    sub_40590C      ;关键 CALL
.text:004059BB                 test    eax, eax
.text:004059BD                 pop     ecx
.text:004059BE                 jz      short loc_4059D0 ;EAX 为 0 就 OVER
.text:004059C0                 mov     ecx, esi
.text:004059C2                 call    ?OnOK@CDialog@@MAEXXZ ; CDialog::OnOK(void)
.text:004059C7                 mov     ecx, esi
.text:004059C9                 call    ?OnOK@CDialog@@MAEXXZ ; CDialog::OnOK(void)
.text:004059CE                 pop     esi
.text:004059CF                 retn
.text:004059D0 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:004059D0
.text:004059D0 loc_4059D0:                            ; CODE XREF: sub_40597D+41j
.text:004059D0                 push    40h
.text:004059D2                 push    offset aHi      ; "Hi"
.text:004059D7                 push    offset aTheRegistratio ; "The registration code you input is inva"...
.text:004059DC
.text:004059DC loc_4059DC:                            ; CODE XREF: sub_40597D+20j
.text:004059DC                                     ; sub_40597D+33j
.text:004059DC                 mov     ecx, esi
.text:004059DE                 call    ?MessageBoxA@CWnd@@QAEHPBD0I@Z ; CWnd::MessageBoxA(char const *,char const *,uint)
.text:004059E3                 pop     esi
.text:004059E4                 retn
.text:004059E4 sub_40597D     endp
```

进入关键的 CALL:

```
.text:0040590C                 push    esi
.text:0040590D                 mov     esi, [esp+arg_0]
.text:00405911                 mov     eax, [esi]                ;EAX 处为注册码
.text:00405913                 cmp     dword ptr [eax-8], 12h     ;长度必须为 12h
.text:00405917                 jnz     short loc_405979
.text:00405919                 cmp     byte ptr [eax+5], 2Dh      ;第 6 位必须为 2Dh,即"-"
.text:0040591D                 jnz     short loc_405979
.text:0040591F                 cmp     byte ptr [eax+0Ah], 2Dh    ;第 11 位必须为 2Dh,即"-"
.text:00405923                 jnz     short loc_405979
.text:00405925                 movsx   ecx, byte ptr [eax+10h]    ;取第 17 位
.text:00405929                 movsx   edx, byte ptr [eax+0Eh]    ;取第 15 位
.text:0040592D                 sub     edx, ecx                   ;相减
.text:0040592F                 movsx   ecx, byte ptr [eax+2]      ;取第 3 位
.text:00405933                 movsx   eax, byte ptr [eax]        ;取第 1 位
.text:00405936                 sub     eax, ecx                   ;相减
.text:00405938                 cmp     eax, edx                   ;结果必须相同
.text:0040593A                 jnz     short loc_405979
.text:0040593C                 push    61h                         ;查找注册码是否有 61h,即"a"
.text:0040593E                 mov     ecx, esi
.text:00405940                 call    ?Find@CString@@QBEHD@Z ; CString::Find(char)
.text:00405945                 cmp     eax, 0FFFFFFFFh
.text:00405948                 jz      short loc_405979            ;没有就 OVER
.text:0040594A                 push    62h                         ;是否有"b"
.text:0040594C                 mov     ecx, esi
.text:0040594E                 call    ?Find@CString@@QBEHD@Z ; CString::Find(char)
.text:00405953                 cmp     eax, 0FFFFFFFFh
.text:00405956                 jnz     short loc_405979            ;有就 OVER
.text:00405958                 push    64h                         ;是否有"d"
.text:0040595A                 mov     ecx, esi
.text:0040595C                 call    ?Find@CString@@QBEHD@Z ; CString::Find(char)
.text:00405961                 cmp     eax, 0FFFFFFFFh
.text:00405964                 jnz     short loc_405979            ;有就 OVER
.text:00405966                 push    63h                         ;是否有"c"
.text:00405968                 mov     ecx, esi
.text:0040596A                 call    ?Find@CString@@QBEHD@Z ; CString::Find(char)
.text:0040596F                 cmp     eax, 0FFFFFFFFh
.text:00405972                 jz      short loc_405979            ;没有就 OVER
.text:00405974                 push    1
.text:00405976                 pop     eax                         ;EAX=1,大功告成
.text:00405977                 pop     esi
.text:00405978                 retn
.text:00405979 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:00405979
.text:00405979 loc_405979:                            ; CODE XREF: sub_40590C+Bj
.text:00405979                                     ; sub_40590C+11j ...
.text:00405979                 xor     eax, eax ;注册码不符合条件跳到这里,EAX=0
.text:0040597B                 pop     esi
.text:0040597C                 retn
.text:0040597C sub_40590C     endp
```

非常简单的注册，根本没运用 `SoftICE`。

一个可用注册码: `12345-acxx-1234567` 用户名不大于 40 位，任意

`4dd466d0e1e0cc123afbe504da2462ca`
