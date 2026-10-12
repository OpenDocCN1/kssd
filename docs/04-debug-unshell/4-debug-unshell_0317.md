
# 菜鸟对 13.8.2 ASPack 外壳分析

> 原文：[`www.pediy.com/kssd/pediy10/77009.html`](https://www.pediy.com/kssd/pediy10/77009.html)

**标题**：菜鸟对 13.8.2 ASPack 外壳分析
**作者**：`guhegu`
**日期**：2008-11-19
**使用工具**：`OllyDBG`
**平台**：`WinXP sp2`
**目标程序**：`加密与解密映像光盘 13.8.2 EdrLib.dll`

**作者声明**：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

看书两个月了请教了很多大侠，终于有一点心得希望能跟大家分享。第一次写这样的文章，不足之处请指正。

```
003ED001 >   60                pushad
003ED002    E8 03000000      call    003ED00A  ; 没有 003ED00A，所以为花指令，F7 进入
003ED007  - E9 EB045D45      jmp     459BD4F7
003ED00C      55                push    ebp
003ED00D      C3                retn
003ED00A      5D                pop     ebp                 ; EdrLib.003ED007
003ED00B      45                inc     ebp
003ED00C      55                push    ebp
003ED00D      C3                retn
003ED00E      E8 01000000     call    003ED014  ; F7 进入
003ED014      5D                pop     ebp                 ; EdrLib.003ED013
003ED015      BB EDFFFFFF     mov     ebx, -13
003ED01A      03DD             add     ebx, ebp  ; EBX 值为 3ED000
003ED01C      81EB 00D00000   sub     ebx, 0D000  ; 得到实际基址 3E0000
003ED022      83BD 22040000 0 > cmp     dword ptr [ebp+422], 0 ; 改地址数据为 0
003ED029      899D 22040000   mov     dword ptr [ebp+422], ebx
003ED02F      0F85 65030000   jnz     003ED39A  ; 这里不跳

003ED035      8D85 2E040000   lea     eax, dword ptr [ebp+42E]
; 地址=003ED441, (ASCII "kernel32.dll")
003ED03B      50                push    eax
003ED03C      FF95 4D0F0000   call    dword ptr [ebp+F4D]
; 调用 ss:[003EDF60]=7C80B6A1 (kernel32.GetModuleHandleA) 取得 kernel32.dll 的句柄

003ED042      8985 26040000   mov     dword ptr [ebp+426], eax
; KERNEL32.DLL 的句柄存入地址中去，方便下次调用

003ED048      8BF8             mov     edi, eax
003ED04A      8D5D 5E         lea     ebx, dword ptr [ebp+5E]
; 地址=003ED071, (ASCII "VirtualAlloc") 为获得 VirtualAlloc 函数地址做准备
003ED04D      53                push    ebx   ; 函数名进栈
003ED04E      50                push    eax   ; DLL 文件句柄进栈
003ED04F      FF95 490F0000   call    dword ptr [ebp+F49]
; 调用 ss:[003EDF5C]=7C80ADA0 (kernel32.GetProcAddress) 获得函数地址

003ED055      8985 4D050000   mov     dword ptr [ebp+54D], eax ; 存入该地址方便下次调用
003ED05B      8D5D 6B         lea     ebx, dword ptr [ebp+6B]
; 地址=003ED07E, (ASCII "VirtualFree")

003ED05E      53                push    ebx ; 函数名地址进栈
003ED05F      57                push    edi ; DLL 文件句柄进栈
003ED060      FF95 490F0000   call    dword ptr [ebp+F49]
; 调用 ss:[003EDF5C]=7C80ADA0 (kernel32.GetProcAddress) 获得函数地址

003ED066      8985 51050000   mov     dword ptr [ebp+551], eax
; 函数地址存入内存中去，方便调用

003ED06C      8D45 77         lea     eax, dword ptr [ebp+77]
; 调用 ss:[003EDF5C]=7C80ADA0 (kernel32.GetProcAddress) 获得函数地址
; 地址=003ED08A

003ED06F      FFE0             jmp     eax == jmp 003ED08A

003ED08A      8B9D 31050000   mov     ebx, dword ptr [ebp+531] ; ebx=0
003ED090      0BDB             or      ebx, ebx
003ED092      74 0A            je      short 003ED09E ; 跳转实现
003ED094      8B03             mov     eax, dword ptr [ebx]
003ED096      8785 35050000   xchg    dword ptr [ebp+535], eax
003ED09C      8903             mov     dword ptr [ebx], eax

003ED09E      8DB5 69050000   lea     esi, dword ptr [ebp+569]
; ESI=003ED57C

003ED0A4      833E 00         cmp     dword ptr [esi], 0
; DS:[003ED57C]=1000
003ED0A7      0F84 21010000   je      003ED1CE ; 不跳

003ED0AD      6A 04            push    4
003ED0AF      68 00100000     push    1000
003ED0B4      68 00180000     push    1800
003ED0B9      6A 00            push    0
003ED0BB      FF95 4D050000   call    dword ptr [ebp+54D]
; 前面保存的 VIRTUALALLOC 函数的地址，意思是分配一个大小为 1000 的空间

003ED0C1      8985 56010000   mov     dword ptr [ebp+156], eax
; EAX 为函数返回的值即系统分配的基址，我这里为 3F0000，每台机器应该不一样

003ED0C7      8B46 04         mov     eax, dword ptr [esi+4] ; EAX=6000
003ED0CA      05 0E010000     add     eax, 10E ; EAX=610E
003ED0CF      6A 04            push    4
003ED0D1      68 00100000     push    1000
003ED0D6      50                push    eax
003ED0D7      6A 00            push    0
003ED0D9      FF95 4D050000   call    dword ptr [ebp+54D]
; 继续分配一个空间

003ED0DF      8985 52010000   mov     dword ptr [ebp+152], eax
; EAX=870000

003ED0E5      56                push    esi
003ED0E6      8B1E             mov     ebx, dword ptr [esi] ; EBX=1000
003ED0E8      039D 22040000   add     ebx, dword ptr [ebp+422]
; 1000+3E0000(上面一段语句保存的基址)=3E1000

003ED0EE      FFB5 56010000   push    dword ptr [ebp+156]
003ED0F4      FF76 04         push    dword ptr [esi+4] ; 6000
003ED0F7      50                push    eax ; 870000
003ED0F8      53                push    ebx ; 3E1000
003ED0F9      E8 6E050000     call    003ED66C ; 解压代码到 3F0000 中去

003ED0FE      B3 00            mov     bl, 0
003ED100      80FB 00         cmp     bl, 0
003ED103      75 5E            jnz     short 003ED163 ; 不跳

003ED105      FE85 EC000000   inc     byte ptr [ebp+EC]
003ED10B      8B3E             mov     edi, dword ptr [esi] ; EDI=1000
003ED10D      03BD 22040000   add     edi, dword ptr [ebp+422] ; EDI=3E1000
003ED113      FF37             push    dword ptr [edi] ; AA424320 进栈
003ED115      C607 C3         mov     byte ptr [edi], 0C3
003ED118      FFD7             call    edi ; 垃圾调用
003ED11A      8F07             pop     dword ptr [edi] ; 实际就是把 AA424320 写到 3E1000 处

003ED11C      50                push    eax
003ED11D      51                push    ecx
003ED11E      56                push    esi
003ED11F      53                push    ebx
003ED120      8BC8             mov     ecx, eax
003ED122      83E9 06         sub     ecx, 6
003ED125      8BB5 52010000   mov     esi, dword ptr [ebp+152] ; ESI=870000
003ED12B      33DB             xor     ebx, ebx ; EBX 清 0
003ED12D      0BC9             or      ecx, ecx
003ED12F      74 2E            je      short 003ED15F
003ED131      78 2C            js      short 003ED15F ; 两个都不跳

003ED133      AC                lods    byte ptr [esi] ; 0B8 送到 AL 中去 SI++
003ED134      3C E8            cmp     al, 0E8
003ED136      74 0A            je      short 003ED142
003ED138      EB 00            jmp     short 003ED13A
003ED13A      3C E9            cmp     al, 0E9
003ED13C      74 04            je      short 003ED142
003ED13E      43                inc     ebx
003ED13F      49                dec     ecx
003ED140    ^ EB EB           jmp     short 003ED12D

003ED142      8B06             mov     eax, dword ptr [esi]
003ED144      EB 00            jmp     short 003ED146
003ED146      803E 01         cmp     byte ptr [esi], 1
003ED149    ^ 75 F3          jnz     short 003ED13E
003ED14B      24 00            and     al, 0
003ED14D      C1C0 18         rol     eax, 18
003ED150      2BC3             sub     eax, ebx
003ED152      8906             mov     dword ptr [esi], eax
003ED154      83C3 05         add     ebx, 5
003ED157      83C6 04         add     esi, 4
003ED15A      83E9 05         sub     ecx, 5
003ED15D    ^ EB CE          jmp     short 003ED12D

; 003ED12D 到这里都是在解码，放到 870000 中去

003ED15F      5B                pop     ebx
003ED160      5E                pop     esi
003ED161      59                pop     ecx
003ED162      58                pop     eax
003ED163      EB 08            jmp     short 003ED16D

003ED16D      8BC8             mov     ecx, eax
003ED16F      8B3E             mov     edi, dword ptr [esi]
003ED171      03BD 22040000   add     edi, dword ptr [ebp+422] ; EDI=3E1000
003ED177      8BB5 52010000   mov     esi, dword ptr [ebp+152] ; ESI=870000
003ED17D      C1F9 02         sar     ecx, 2
003ED180      F3:A5           rep     movs dword ptr es:[edi], dword ptr [esi]
; 解码
003ED182      8BC8             mov     ecx, eax
003ED184      83E1 03         and     ecx, 3
003ED187      F3:A4           rep     movs byte ptr es:[edi], byte ptr [esi]
003ED189      5E                pop     esi
003ED18A      68 00800000     push    8000
```



```
003ED18F    6A 00             push    0  
003ED191    FFB5 52010000    push    dword ptr [ebp+152] 870000  
003ED197    FF95 51050000    call    dword ptr [ebp+551]  
```

`Virtualfree` 释放 `870000` 的空间

```
003ED19D    83C6 08          add     esi, 8  
003ED1A0    833E 00        cmp     dword ptr [esi], 0  
003ED1A3  ^ 0F85 1EFFFFFF   jnz     003ED0C7 整个大循环就是在解码  
003ED1A9    68 00800000      push    8000  
003ED1AE    6A 00           push    0  
003ED1B0    FFB5 56010000    push    dword ptr [ebp+156]  
003ED1B6    FF95 51050000    call    dword ptr [ebp+551]  
```

释放 `370000` 的空间

```
003ED1BC    8B9D 31050000    mov     ebx, dword ptr [ebp+531]  
003ED1C2    0BDB            or      ebx, ebx  
003ED1C4    74 08           je      short 003ED1CE 跳  
003ED1C6    8B03            mov     eax, dword ptr [ebx]  
003ED1C8    8785 35050000    xchg    dword ptr [ebp+535], eax  
```

接下来就是开始重定位了

```
003ED1CE    8B95 22040000    mov     edx, dword ptr [ebp+422] 实际基址  
003ED1D4    8B85 2D050000    mov     eax, dword ptr [ebp+52D] 默认基址  
003ED1DA    2BD0            sub     edx, eax 实际基址-默认基址不等于 0 所以要进行重定位 差值保存到 `EDX`  
003ED1DC    74 79           je      short 003ED257  
003ED1DE    8BC2            mov     eax, edx EDX 差值放到 `EAX`  
003ED1E0    C1E8 10         shr     eax, 10   右移 10 位  
003ED1E3    33DB            xor     ebx, ebx ebx 清 0  
003ED1E5    8BB5 39050000    mov     esi, dword ptr [ebp+539] 取重定位表 `RVA` 这里是 `C000`  
003ED1EB    03B5 22040000    add     esi, dword ptr [ebp+422] 得到实际重定位表地址  
003ED1F1    833E 00        cmp     dword ptr [esi], 0  
003ED1F4    74 61           je      short 003ED257 不跳  
```

下面两个嵌套循环完成重定位

```
003ED1F6    8B4E 04        mov     ecx, dword ptr [esi+4] 重定位表指向下一位（重定位单元的大小）  
003ED1F9    83E9 08        sub     ecx, 8  
003ED1FC    D1E9            shr     ecx, 1  `ECX`=8A  
003ED1FE    8B3E            mov     edi, dword ptr [esi]  
003ED200    03BD 22040000    add     edi, dword ptr [ebp+422] `edi`=3e1000  
003ED206    83C6 08        add     esi, 8  源操作数+8  
003ED209    66:8B1E        mov     bx, word ptr [esi] `BX`=301D  
003ED20C    C1EB 0C        shr     ebx, 0C   `EBX`=3  这里应该是用 `EBX` 分类  
003ED20F    83FB 01        cmp     ebx, 1  
003ED212    74 0C           je      short 003ED220  
003ED214    83FB 02        cmp     ebx, 2  
003ED217    74 16           je      short 003ED22F  
003ED219    83FB 03        cmp     ebx, 3  
003ED21C    74 20           je      short 003ED23E 跳转实现  
003ED21E    EB 2C           jmp     short 003ED24C  
003ED220    66:8B1E        mov     bx, word ptr [esi]  
003ED223    81E3 FF0F0000   and     ebx, 0FFF  
003ED229    66:01041F      add     word ptr [edi+ebx], ax  
003ED22D    EB 1D           jmp     short 003ED24C  
003ED22F    66:8B1E        mov     bx, word ptr [esi]  
003ED232    81E3 FF0F0000   and     ebx, 0FFF  
003ED238    66:01141F      add     word ptr [edi+ebx], dx  
003ED23C    EB 0E           jmp     short 003ED24C  
003ED23E    66:8B1E        mov     bx, word ptr [esi]  
003ED241    81E3 FF0F0000   and     ebx, 0FFF  `EBX`=1D  
003ED247    01141F          add     dword ptr [edi+ebx], edx  
```

这里就是重定位了 `EDX` 保存的差值

```
003ED24A    EB 00           jmp     short 003ED24C  
003ED24C    66:830E FF      or      word ptr [esi], 0FFFF  
003ED250    83C6 02        add     esi, 2 指向下一个  
003ED253  ^ E2 B4         loopd   short 003ED209  
003ED255  ^ EB 9A         jmp     short 003ED1F1  
```

开始处理输入表

```
003ED257    8B95 22040000    mov     edx, dword ptr [ebp+422]实际基址  
003ED25D    8BB5 41050000    mov     esi, dword ptr [ebp+541]  
003ED263    0BF6            or      esi, esi  
003ED265    74 11           je      short 003ED278 跳转实现  
003ED267    03F2            add     esi, edx  
003ED269    AD              lods    dword ptr [esi]  
003ED26A    0BC0            or      eax, eax  
003ED26C    74 0A           je      short 003ED278  
003ED26E    03C2            add     eax, edx  
003ED270    8BF8            mov     edi, eax  
003ED272    66:AD           lods    word ptr [esi]  
003ED274    66:AB           stos    word ptr es:[edi]  
003ED276  ^ EB F1         jmp     short 003ED269  
003ED278    BE 94760000     mov     esi, 7694 输入表 `RVA`  
003ED27D    8B95 22040000    mov     edx, dword ptr [ebp+422]  
003ED283    03F2            add     esi, edx 得到实际的输入表地址  
003ED285    8B46 0C        mov     eax, dword ptr [esi+C] `IID` 的 `RVA`  
003ED288    85C0            test    eax, eax    
003ED28A    0F84 0A010000   je      003ED39A 不跳转  
003ED290    03C2            add     eax, edx 得到实际 `IID` 地址  
003ED292    8BD8            mov     ebx, eax  
003ED294    50              push    eax  
003ED295    FF95 4D0F0000    call    dword ptr [ebp+F4D]  
```

调用 `ss:[003EDF60]=7C80B6A1` (`kernel32.GetModuleHandleA`) 得到 `KERNEL32.DLL` 句柄

```
003ED29B    85C0            test    eax, eax  
003ED29D    75 07           jnz     short 003ED2A6 跳转实现  
003ED29F    53              push    ebx  
003ED2A0    FF95 510F0000    call    dword ptr [ebp+F51]  
003ED2A6    8985 45050000    mov     dword ptr [ebp+545], eax 句柄存入  
003ED2AC    C785 49050000 0>mov     dword ptr [ebp+549], 0 加个 0 表示结尾  
003ED2B6    8B95 22040000    mov     edx, dword ptr [ebp+422]  
003ED2BC    8B06            mov     eax, dword ptr [esi] 第一个 `IID` 的 `RVA`  
003ED2BE    85C0            test    eax, eax  
003ED2C0    75 03           jnz     short 003ED2C5 跳  
003ED2C2    8B46 10        mov     eax, dword ptr [esi+10] 下一个 `IID` `76E4`  
003ED2C5    03C2            add     eax, edx  
003ED2C7    0385 49050000   add     eax, dword ptr [ebp+549]  
003ED2CD    8B18            mov     ebx, dword ptr [eax]  7B66  
003ED2CF    8B7E 10        mov     edi, dword ptr [esi+10] 7014  
003ED2D2    03FA            add     edi, edx  
003ED2D4    03BD 49050000   add     edi, dword ptr [ebp+549]  
003ED2DA    85DB            test    ebx, ebx  
003ED2DC    0F84 A2000000   je      003ED384  
003ED2E2    F7C3 00000080   test    ebx, 80000000  
003ED2E8    75 04           jnz     short 003ED2EE  
003ED2EA    03DA            add     ebx, edx  
003ED2EC    43              inc     ebx  
003ED2ED    43              inc     ebx  找到第一个函数名地址  
003ED2EE    53              push    ebx  
003ED2EF    81E3 FFFFFFFF   and     ebx, 7FFFFFFF  
003ED2F5    53              push    ebx  
003ED2F6    FFB5 45050000    push    dword ptr [ebp+545]  
003ED2FC    FF95 490F0000    call    dword ptr [ebp+F49]  
```

`ss:[003EDF5C]=7C80ADA0` (`kernel32.GetProcAddress`) 得到 第一个函数的地址

```
003ED302    85C0            test    eax, eax 函数地址  
003ED304    5B              pop     ebx  函数名  
003ED305    75 6F           jnz     short 003ED376 跳  
003ED307    F7C3 00000080   test    ebx, 80000000  
003ED30D    75 19           jnz     short 003ED328  
003ED30F    57              push    edi  
003ED310    8B46 0C        mov     eax, dword ptr [esi+C]  
003ED313    0385 22040000   add     eax, dword ptr [ebp+422]  
003ED319    50              push    eax  
003ED31A    53              push    ebx  
003ED31B    8D85 75040000   lea     eax, dword ptr [ebp+475]  
003ED321    50              push    eax  
003ED322    57              push    edi  
003ED323    E9 98000000     jmp     003ED3C0  
003ED376    8907            mov     dword ptr [edi], eax             ; `kernel32.GetStringTypeA` 存入 `IAT`  
003ED378    8385 49050000 0>add     dword ptr [ebp+549], 4  指向下一个函数名  
003ED37F  ^ E9 32FFFFFF     jmp     003ED2B6  循环第一层  
003ED384    8906            mov     dword ptr [esi], eax  完成 `IID` 的填写  
003ED386    8946 0C        mov     dword ptr [esi+C], eax  完成 `IID` 的填写  
003ED389    8946 10        mov     dword ptr [esi+10], eax 完成 `IID` 的填写  
003ED38C    83C6 14        add     esi, 14  指向下一个 `IID`  
003ED38F    8B95 22040000    mov     edx, dword ptr [ebp+422]  
003ED395  ^ E9 EBFEFFFF     jmp     003ED285  
003ED39A    B8 40120000     mov     eax, 1240  
```



```
003ED39F    50                 push    eax  
003ED3A0    0385 22040000      add     eax, dword ptr [ebp+422]  
003ED3A6    59                 pop     ecx  
003ED3A7    0BC9               or      ecx, ecx  
003ED3A9    8985 A8030000      mov     dword ptr [ebp+3A8], eax  
003ED3AF    61                 popad  
003ED3B0    75 08              jnz     short 003ED3BA  
003ED3B2    B8 01000000        mov     eax, 1  
003ED3B7    C2 0C00            retn    0C  
003ED3BA    68 00000000        push    0  
003ED3BF    C3                 retn   回到 OEP
```
