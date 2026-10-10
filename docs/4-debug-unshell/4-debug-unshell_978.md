

# SDProtector v1.12 外壳完整分析(年终篇)

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-770.htm`](https://www.pediy.com/kssd/pediy07/pediy7-770.htm)

【目     标】:N/A  
【工     具】:Olydbg1.1(diy 版)、LORDPE、ImportREC1.6F   
【任     务】:分析外壳    
【操作平台】:Windows xp sp2   
【作     者】: LOVEBOOM\[DFCG\]\[FCG\]\[CUG\]  
【相关链接】: N/A  
【简要说明】: 传说这个外壳比较强悍，我以前没时间分析的时间时候，用 OD 试了几次挂了，后面一直没有时间分析这个外壳，今天终于可以完成自己的心愿，拨开它的神秘面纱，看看这个壳是是不是真的这么神秘，这么强劲。不过说真的，等自己分析完了，第一个感觉就是上当，作者的写壳能力是我见过的最有待改进的。不过不管怎么说，什么事都是说容易做的时候难，也许有一天我自己写的时候也会出不少问题.题外话：用 Word 写分析文章还是真不方便，抓下来的代码给自己的感觉就是乱乱的。但不用 WORD 也不好办，文件太长了，做成 txt 的打开不方便，做成 html 的修改格式就麻烦，没办法，只能选个择中的方法。  
加上这个外壳的分析，算起来就快到十个外壳完全分析了，恭喜下自己，向 20 进军。关于文章的其它说明，文章已经不再是以前的教你按几次 F9，按什么键等操作性文章，如果你只是想知道怎么脱壳，那么很抱歉文章不适用于你。  
【详细过程】:  
   壳的保护：  
    这个外壳用的 Anti-Debug 还是比较多,可以大概的分为两部分，一部分是 OD 和 sice 共用的反调试部分，另一部分是分别针对 sice 或 OD 的，还好，这个壳对 OD 的检测还只是一般的水平。共用的反调试部分也是这个壳的灵魂部分：用时间差来检测调试器，不断的检测时间差(从另一角度来看，这也是这个壳的失败之处，对于稍微有经验的朋友来说，时间差他们没有起到多大的作用，只可能让壳的性能降低，浪费 CPU 资源。)，这个壳的检测时间的方法为 RDTSC、GetProcessTimes 和 GetTickcount.。另一部分中的针对 Od 的检测: 主要用 ZwQueryInformationProcess 和 IsDebuggerPresent,还有一个是多线程(多线程对 SICE 来讲没有很大的阻碍)；针对 sice 的检测就比较多有常用的 INT3 擦除硬件断点(注：虽然这个对 OD 也有效，但作用并不大)，防止单步调试检测，CreateFileA 检测(这个检测 同时会检测常用的调试器和调试器相关的东西，如 Trw,icedump 等等)ZwQuerySystemInformation 检测驱动名(还会检测 IceExt,感觉多余，因为 iceext 安装时可以自定义名字的)。  
  我自己是用 OD 分析，检测 sice 之类的对我来说就不重要了，重要的是公用检测和 OD 检测部分，好了，接下来开始慢慢分析，并从中找出跳过检测的方法。同前面的文章差不多，我喜欢用 IDA 来静态注释，那样看起来很方便。  
  准备工作:  
    开始文章之前你要对壳有个大概的了解，对汇编有一定的了解，对调试工具也要有一定的了解。我分析过几次之后发现这个外壳和其它外壳一样有，花指令基本上是用宏来写的，根据它的特性我随手写了个简单的清除花指令的脚本，因为我自己担心会影响到程序的，所以只是简单的处理了下花指令没有完全去除，不过这样已经很方便我们看代码,附件中的 csdp.txt 便为清楚花指令脚本。准备好了相关工作后，开始进入分析过程。  
用 OD 载入目标程序：  
; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047A000  
SDPI:0047A000 ; 入口处开始一堆花指令  
SDPI:0047A000  
SDPI:0047A000                 public start  
SDPI:0047A000 start           proc near  
SDPI:0047A000  
SDPI:0047A000 ; FUNCTION CHUNK AT SDPI:0047A022 SIZE 00000024 BYTES  
SDPI:0047A000  
SDPI:0047A000                 jz      short loc\_47A009  
SDPI:0047A002                 jnz     short loc\_47A009  
SDPI:0047A002 ; ----------------------------------------------------------------------------  
SDPI:0047A004 a2gss           db 19h,'2g 梃'  
SDPI:0047A009 ; ----------------------------------------------------------------------------  
SDPI:0047A009  
SDPI:0047A009 loc\_47A009:                                       ; CODE XREF: start j  
SDPI:0047A009                                                   ; start+2 j  
SDPI:0047A009                 jz      short loc\_47A02A  
SDPI:0047A00B                 jnz     short loc\_47A02A  
SDPI:0047A00B ; ----------------------------------------------------------------------------  
SDPI:0047A00D                 db 0E8h ; ?  
SDPI:0047A00E ; ----------------------------------------------------------------------------  
SDPI:0047A00E  
SDPI:0047A00E loc\_47A00E:                                       ; CODE XREF: SDPI:0047A04F j  
SDPI:0047A00E                                                   ; SDPI:0047A051 j  
SDPI:0047A00E                 push    0CD4439h  
SDPI:0047A013                 pop     ecx  
SDPI:0047A014                 pushf  
SDPI:0047A015                 push    eax  
SDPI:0047A016                 jz      short loc\_47A022  
SDPI:0047A018                 jnz     short loc\_47A022  
SDPI:0047A018 ; ----------------------------------------------------------------------------  
SDPI:0047A01A                 db 0E8h  
SDPI:0047A01B ; ----------------------------------------------------------------------------  
SDPI:0047A01B  
SDPI:0047A01B loc\_47A01B:                                       ; CODE XREF: start:loc\_47A022 p  
SDPI:0047A01B                 pop     ecx  
和一般的保存壳一样，EP 开头部分多数是一大堆的花指令，这个就是考你脱壳的耐力，看这篇文章也是一样，要耐心看才行。嗯跑题了，继续继续跟进去。  
                 call    loc\_47A3BA  
SDPI:0047A3B9                 nop  
SDPI:0047A3BA  
SDPI:0047A3BA loc\_47A3BA:                                       ; CODE XREF: SDPI:0047A3B4 p  
SDPI:0047A3BA                 pop     edx  
SDPI:0047A3BB                 add     edx, 9835h  
SDPI:0047A3C1                 call    loc\_47A3C7  
SDPI:0047A3C6                 nop  
SDPI:0047A3C7  
SDPI:0047A3C7 loc\_47A3C7:                                       ; CODE XREF: SDPI:0047A3C1 p  
SDPI:0047A3C7                 pop     eax  
SDPI:0047A3C8                 add     eax, 0FFFFFDE2h  
SDPI:0047A3CD                 mov     ecx, 10h  
SDPI:0047A3D2                 call    De\_Code                   ; invoke De\_Code,ecx(counter),eax(key),edx(end)  
SDPI:0047A3D2                                                   ; 用于解密代码，解密起始地址就是 call 进来  
SDPI:0047A3D2                                                   ; 的下一行代码地址  
SDPI:0047A3D7                 mov     eax, 0  
SDPI:0047A3DC                 push    eax  
SDPI:0047A3DD                 call    loc\_47A3E3                ; 变形 call  
SDPI:0047A3DD                                                   ; 这里实际就是:  
SDPI:0047A3DD                                                   ; push 0  
SDPI:0047A3DD                                                   ; call 481a21  
SDPI:0047A3E2                 nop  
SDPI:0047A3E3  
SDPI:0047A3E3 loc\_47A3E3:                                       ; CODE XREF: SDPI:0047A3DD p  
SDPI:0047A3E3                 pop     eax                       ; 变形 call  
SDPI:0047A3E3                                                   ; 这里实际就是:  
SDPI:0047A3E3                                                   ; push 0  
SDPI:0047A3E3                                                   ; call 481a21  
SDPI:0047A3E4                 add     eax, 11h  
SDPI:0047A3E9                 push    eax  
SDPI:0047A3EA                 jmp     Disposal\_IMP              ; 跳去处理第一个解密壳数据的 call  
SDPI:0047A3EF ; ----------------------------------------------------------------------------  
SDPI:0047A3EF                 nop  
0047A3F3                 call    loc\_47A3F9  
SDPI:0047A3F8                 nop  
SDPI:0047A3F9  
SDPI:0047A3F9 loc\_47A3F9:                                       ; CODE XREF: SDPI:0047A3F3 p  
SDPI:0047A3F9                 pop     eax  
SDPI:0047A3FA                 add     eax, 11h  
SDPI:0047A3FF                 push    eax  
SDPI:0047A400                 jmp     loc\_4813BB                ; 这里进行计算 MD5 值，不明有什么作用  
SDPI:0047A405 ; ----------------------------------------------------------------------------  
SDPI:0047A405                 nop  
SDPI:0047A406                 nop  
SDPI:0047A407                 nop  
SDPI:0047A408                 nop  
SDPI:0047A409                 push    1  
SDPI:0047A40B                 call    loc\_47A411  
SDPI:0047A410                 nop  
SDPI:0047A411  
SDPI:0047A411 loc\_47A411:                                       ; CODE XREF: SDPI:0047A40B p  
SDPI:0047A411                 pop     eax  
SDPI:0047A412                 add     eax, 11h  
SDPI:0047A417                 push    eax  
SDPI:0047A418                 jmp     Alloc\_Sp\_480825           ; push 1  
SDPI:0047A418                                                   ; call 480825  
SDPI:0047A41D ; ----------------------------------------------------------------------------  
SDPI:0047A41D                 nop  
SDPI:0047A41E                 nop  
SDPI:0047A41F                 nop  
SDPI:0047A420                 nop  
SDPI:0047A421                 call    loc\_47A427  
SDPI:0047A426                 nop  
SDPI:0047A427  
SDPI:0047A427 loc\_47A427:                                       ; CODE XREF: SDPI:0047A421 p  
SDPI:0047A427                 pop     eax  
SDPI:0047A428                 add     eax, 11h  
SDPI:0047A42D                 push    eax  
SDPI:0047A42E                 jmp     Get\_Version  
SDPI:0047A433 ; ----------------------------------------------------------------------------  
SDPI:0047A433                 nop  
SDPI:0047A434                 nop  
SDPI:0047A435                 nop  
SDPI:0047A436                 nop  
SDPI:0047A437                 cmp     eax, 80000000h  
SDPI:0047A43C                 jb      isWinNT\_\_47A4CE  
SDPI:0047A442                 sub     esp, 8  
SDPI:0047A445                 sidt    qword ptr \[esp\]           ; 如果是 Win9x 大于 80000000  
SDPI:0047A449                 mov     eax, \[esp+2\]  
SDPI:0047A44D                 mov     cx, \[eax+0Eh\]  
SDPI:0047A451                 mov     dx, \[eax+6\]  
SDPI:0047A455                 mov     bx, \[eax+1Eh\]  
SDPI:0047A459                 add     esp, 8  
SDPI:0047A45C                 cmp     cx, dx  
SDPI:0047A45F                 jnz     short loc\_47A466  
SDPI:0047A461                 cmp     bx, dx  
SDPI:0047A464                 jz      short isWinNT\_\_47A4CE  
SDPI:0047A466  
SDPI:0047A466 loc\_47A466:                                       ; CODE XREF: SDPI:0047A45F j  
SDPI:0047A466                 nop  
SDPI:0047A467                 nop  
SDPI:0047A468                 nop  
SDPI:0047A469                 nop  
SDPI:0047A46A                 nop  
SDPI:0047A46B                 call    sub\_47A471  
SDPI:0047A470                 nop  
SDPI:0047A471  
SDPI:0047A471 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047A471  
SDPI:0047A471  
SDPI:0047A471 sub\_47A471      proc near                         ; CODE XREF: SDPI:0047A46B p  
SDPI:0047A471                 pop     eax  
SDPI:0047A472                 add     eax, 5Eh  
SDPI:0047A477                 mov     edx, eax  
SDPI:0047A479                 add     edx, 32h  
SDPI:0047A47C                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047A47C                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047A47C                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047A47C                                                   ; 用来解密  
SDPI:0047A481                 call    sub\_47A487  
SDPI:0047A486                 nop  
SDPI:0047A486 sub\_47A471      endp  
SDPI:0047A486  
SDPI:0047A487  
SDPI:0047A487 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047A487  
SDPI:0047A487  
SDPI:0047A487 sub\_47A487      proc near                         ; CODE XREF: sub\_47A471+10 p  
SDPI:0047A487                 pop     eax  
SDPI:0047A488                 add     eax, 4C16h  
SDPI:0047A48D                 call    sub\_47A493  
SDPI:0047A492                 nop  
SDPI:0047A492 sub\_47A487      endp  
SDPI:0047A492  
SDPI:0047A493  
SDPI:0047A493 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047A493  
SDPI:0047A493  
SDPI:0047A493 sub\_47A493      proc near                         ; CODE XREF: sub\_47A487+6 p  
SDPI:0047A493                 pop     ecx  
SDPI:0047A494                 add     ecx, 4CB7h  
SDPI:0047A49A                 push    0  
SDPI:0047A49C                 push    ecx  
SDPI:0047A49D                 push    eax  
SDPI:0047A49E                 push    0  
SDPI:0047A4A0                 call    sub\_47A4A6  
SDPI:0047A4A5                 nop  
SDPI:0047A4A5 sub\_47A493      endp  
SDPI:0047A4A5  
SDPI:0047A4A6  
SDPI:0047A4A6 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047A4A6  
SDPI:0047A4A6  
SDPI:0047A4A6 sub\_47A4A6      proc near                         ; CODE XREF: sub\_47A493+D p  
SDPI:0047A4A6                 pop     eax  
SDPI:0047A4A7                 add     eax, 11h  
SDPI:0047A4AC                 push    eax  
SDPI:0047A4AD                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047A4AD sub\_47A4A6      endp                              ; 判断函数的前 5 位是否为 CC  
SDPI:0047A4AD                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047A4B2 ; ----------------------------------------------------------------------------  
SDPI:0047A4B2                 nop  
SDPI:0047A4B3                 nop  
SDPI:0047A4B4                 nop  
SDPI:0047A4B5                 nop  
SDPI:0047A4B6                 push    7  
SDPI:0047A4B8                 call    sub\_47A4BE  
SDPI:0047A4BD                 nop  
SDPI:0047A4BE  
SDPI:0047A4BE ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047A4BE  
SDPI:0047A4BE  
SDPI:0047A4BE sub\_47A4BE      proc near                         ; CODE XREF: SDPI:0047A4B8 p  
SDPI:0047A4BE                 pop     eax  
SDPI:0047A4BF                 add     eax, 11h  
SDPI:0047A4C4                 push    eax  
SDPI:0047A4C5                 jmp     ExitProcess  
SDPI:0047A4C5 sub\_47A4BE      endp  
SDPI:0047A4C5  
SDPI:0047A4CA ; ----------------------------------------------------------------------------  
SDPI:0047A4CA                 nop  
SDPI:0047A4CB                 nop  
SDPI:0047A4CC                 nop  
SDPI:0047A4CD                 nop  
SDPI:0047A4CE  
SDPI:0047A4CE isWinNT\_\_47A4CE:                                  ; CODE XREF: SDPI:0047A43C j  
SDPI:0047A4CE                                                   ; SDPI:0047A464 j  
SDPI:0047A4CE                 jo      short loc\_47A4DE  
SDPI:0047A4D0                 jno     short loc\_47A4DE  
SDPI:0047A4D0 ; ----------------------------------------------------------------------------  
SDPI:0047A4D2                 db 0  
SDPI:0047A4D3                 db  10h  
SDPI:0047A4D4                 db 40h  
SDPI:0047A4D5                 db 0  
SDPI:0047A4D6                 db 0BFh ; ?  
SDPI:0047A4D7                 db  56h ; V  
SDPI:0047A4D8                 db  7Ch ; |  
SDPI:0047A4D9                 db  21h ; !  
SDPI:0047A4DA                 db  76h ; v  
SDPI:0047A4DB                 db 12h  
SDPI:0047A4DC                 db  80h ; €  
SDPI:0047A4DD                 db  0Eh  
SDPI:0047A4DE ; ----------------------------------------------------------------------------  
SDPI:0047A4DE  
SDPI:0047A4DE loc\_47A4DE:                                       ; CODE XREF: SDPI:isWinNT\_\_47A4CE j  
SDPI:0047A4DE                                                   ; SDPI:0047A4D0 j  
SDPI:0047A4DE                 mov     ecx, 769E3CF2h  
SDPI:0047A4E3                 call    loc\_47A4E9  
SDPI:0047A4E8                 nop  
SDPI:0047A4E9  
SDPI:0047A4E9 loc\_47A4E9:                                       ; CODE XREF: SDPI:0047A4E3 p  
SDPI:0047A4E9                 pop     eax  
SDPI:0047A4EA                 add     eax, 5FEh  
SDPI:0047A4EF                 call    loc\_47A4F5  
SDPI:0047A4F4                 nop  
SDPI:0047A4F5  
SDPI:0047A4F5 loc\_47A4F5:                                       ; CODE XREF: SDPI:0047A4EF p  
SDPI:0047A4F5                 pop     edx  
SDPI:0047A4F6                 add     edx, 8E3h  
SDPI:0047A4FC                 call    Crypt\_Code                ; 把前面 De\_code 解出的代码再加密回去  
SDPI:0047A4FC                                                   ; 作者这里很阴险的用计算后的 MD5 值来加  
SDPI:0047A4FC                                                   ; 密回去，如果代码修改了，MD5 值肯定不对了.  
SDPI:0047A4FC                                                   ; 第一次加密地址:0047AAE6  
SDPI:0047A501                 push    eax  
SDPI:0047A502                 xor     eax, eax  
SDPI:0047A504                 call    loc\_47A50A  
SDPI:0047A509                 nop  
SDPI:0047A50A  
SDPI:0047A50A loc\_47A50A:                                       ; CODE XREF: SDPI:0047A504 p  
SDPI:0047A50A                 pop     edi  
SDPI:0047A50B                 add     edi, 61h  
SDPI:0047A511                 mov     ebx, \[edi\]  
SDPI:0047A513                 mov     edx, \[edi+4\]  
SDPI:0047A516                 jz      short loc\_47A522  
SDPI:0047A518                 jnz     short loc\_47A522  
SDPI:0047A518 ; ----------------------------------------------------------------------------  
SDPI:0047A51A                 dd 401000h  
SDPI:0047A51E                 dd 9F7AB0Bh  
SDPI:0047A522 ; ----------------------------------------------------------------------------  
SDPI:0047A522  
SDPI:0047A522 loc\_47A522:                                       ; CODE XREF: SDPI:0047A516 j  
SDPI:0047A522                                                   ; SDPI:0047A518 j  
SDPI:0047A522                 call    loc\_47A528  
SDPI:0047A527                 nop  
SDPI:0047A528  
SDPI:0047A528 loc\_47A528:                                       ; CODE XREF: SDPI:loc\_47A522 p  
SDPI:0047A528                 pop     esi  
SDPI:0047A529                 add     esi, 59h  
SDPI:0047A52F                 mov     ecx, 3  
SDPI:0047A534                 jl      short loc\_47A53D  
SDPI:0047A536  
SDPI:0047A536 loc\_47A536:                                       ; CODE XREF: SDPI:loc\_47A53D j  
SDPI:0047A536                 jmp     short loc\_47A53F  
SDPI:0047A536 ; ----------------------------------------------------------------------------  
SDPI:0047A538                 db    0  
SDPI:0047A539                 db  10h  
SDPI:0047A53A                 db  40h ; @  
SDPI:0047A53B                 db    0  
SDPI:0047A53C                 db 0E8h ; ?  
SDPI:0047A53D ; ----------------------------------------------------------------------------  
SDPI:0047A53D  
SDPI:0047A53D loc\_47A53D:                                       ; CODE XREF: SDPI:0047A534 j  
SDPI:0047A53D                 jz      short loc\_47A536  
SDPI:0047A53F  
SDPI:0047A53F loc\_47A53F:                                       ; CODE XREF: SDPI:loc\_47A536 j  
SDPI:0047A53F                 jb      short loc\_47A553  
SDPI:0047A541                 jnb     short loc\_47A553  
SDPI:0047A541 ; ----------------------------------------------------------------------------  
SDPI:0047A543                 dd 401000h  
SDPI:0047A547 dword\_47A547    dd 72C303E8h                      ; CODE XREF: SDPI:0047A55A j  
SDPI:0047A54B                 dd 19731Bh  
SDPI:0047A54F                 dd 0E8004010h  
SDPI:0047A553 ; ----------------------------------------------------------------------------  
SDPI:0047A553  
SDPI:0047A553 loc\_47A553:                                       ; CODE XREF: SDPI:loc\_47A53F j  
SDPI:0047A553                                                   ; SDPI:0047A541 j  
SDPI:0047A553                 pushfw  
SDPI:0047A555                 push    eax  
SDPI:0047A556                 xor     eax, eax  
SDPI:0047A558                 cmp     ebx, eax  
SDPI:0047A55A                 jz      short near ptr dword\_47A547+1  
SDPI:0047A55C                 call    loc\_47A566  
SDPI:0047A55C ; ----------------------------------------------------------------------------  
SDPI:0047A561                 dd 401000h  
SDPI:0047A565                 db 0E8h ; ?  
SDPI:0047A566 ; ----------------------------------------------------------------------------  
SDPI:0047A566  
SDPI:0047A566 loc\_47A566:                                       ; CODE XREF: SDPI:0047A55C p  
SDPI:0047A566                 pop     eax  
SDPI:0047A567                 pop     eax  
SDPI:0047A568                 popfw  
SDPI:0047A56A                 rep movsw                         ; shit Junk code  
SDPI:0047A56D                 call    sub\_47A5EE                ; 这里是花指令来的，防止单步跟踪  
SDPI:0047A56D                                                   ; 实际 movsw 后是 EB 01 XX EB 03 XXXXXX  
SDPI:0047A56D                                                   ;  
SDPI:0047A572                 call    INT3\_47a65E               ; 这里进去 CC 异常，并还原上面的花指令  
SDPI:0047A577                 call    near ptr 87B57Ch  
SDPI:0047A57C                 mov     al, 89h  
SDPI:0047A57E                 pushf  
SDPI:0047A57F                 add     al, 0EBh  
SDPI:0047A581                 add     \[eax-6F6FFC15h\], edx  
SDPI:0047A587                 nop  
SDPI:0047A588                 nop  
SDPI:0047A589                 nop  
SDPI:0047A58A                 nop  
SDPI:0047A58B                 call    sub\_47A591  
SDPI:0047A590                 nop  
SDPI:0047A591  
SDPI:0047A591 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047A591  
SDPI:0047A591  
SDPI:0047A591 sub\_47A591      proc near                         ; CODE XREF: SDPI:0047A58B p  
SDPI:0047A591                 pop     eax  
SDPI:0047A592                 add     eax, 5Eh  
SDPI:0047A597                 mov     edx, eax  
SDPI:0047A599                 add     edx, 32h  
SDPI:0047A59C                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047A59C                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047A59C                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047A59C                                                   ; 用来解密  
SDPI:0047A5A1                 call    sub\_47A5A7  
SDPI:0047A5A6                 nop  
SDPI:0047A5A6 sub\_47A591      endp  
SDPI:0047A5A6  
SDPI:0047A5A7  
SDPI:0047A5A7 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047A5A7  
SDPI:0047A5A7  
SDPI:0047A5A7 sub\_47A5A7      proc near                         ; CODE XREF: sub\_47A591+10 p  
SDPI:0047A5A7                 pop     eax  
SDPI:0047A5A8                 add     eax, 4AF6h  
SDPI:0047A5AD                 call    sub\_47A5B3  
SDPI:0047A5B2                 nop  
SDPI:0047A5B2 sub\_47A5A7      endp  
SDPI:0047A5B2  
SDPI:0047A5B3  
SDPI:0047A5B3 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047A5B3  
SDPI:0047A5B3  
SDPI:0047A5B3 sub\_47A5B3      proc near                         ; CODE XREF: sub\_47A5A7+6 p  
SDPI:0047A5B3                 pop     ecx  
SDPI:0047A5B4                 add     ecx, 4B97h  
SDPI:0047A5BA                 push    0  
SDPI:0047A5BC                 push    ecx  
SDPI:0047A5BD                 push    eax  
SDPI:0047A5BE                 push    0  
SDPI:0047A5C0                 call    sub\_47A5C6  
SDPI:0047A5C5                 nop  
SDPI:0047A5C5 sub\_47A5B3      endp  
SDPI:0047A5C5  
SDPI:0047A5C6  
SDPI:0047A5C6 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047A5C6  
SDPI:0047A5C6  
SDPI:0047A5C6 sub\_47A5C6      proc near                         ; CODE XREF: sub\_47A5B3+D p  
SDPI:0047A5C6                 pop     eax  
SDPI:0047A5C7                 add     eax, 11h  
SDPI:0047A5CC                 push    eax  
SDPI:0047A5CD                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047A5CD sub\_47A5C6      endp                              ; 判断函数的前 5 位是否为 CC  
SDPI:0047A5CD                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047A5CD ; ----------------------------------------------------------------------------  
SDPI:0047A5D2                 db  90h ; ?  
SDPI:0047A5D3                 db  90h ; ?  
SDPI:0047A5D4                 db  90h ; ?  
SDPI:0047A5D5                 db  90h ; ?  
SDPI:0047A5D6                 db  6Ah ; j  
SDPI:0047A5D7                 db    7  
SDPI:0047A5D8                 db 0E8h ; ?  
SDPI:0047A5D9                 db    1  
SDPI:0047A5DA                 db    0  
SDPI:0047A5DB                 db    0  
SDPI:0047A5DC                 db    0  
SDPI:0047A5DD                 db  90h ; ?  
SDPI:0047A5DE                 db  58h ; X  
SDPI:0047A5DF                 db    5  
SDPI:0047A5E0                 db  11h  
SDPI:0047A5E1                 db    0  
SDPI:0047A5E2                 db    0  
SDPI:0047A5E3                 db    0  
SDPI:0047A5E4                 db  50h ; P  
SDPI:0047A5E5                 db 0E9h ; ?  
SDPI:0047A5E6                 db  28h ; (  
SDPI:0047A5E7                 db  90h ; ?  
SDPI:0047A5E8                 db    0  
SDPI:0047A5E9                 db    0  
SDPI:0047A5EA                 db  90h ; ?  
SDPI:0047A5EB                 db  90h ; ?  
SDPI:0047A5EC                 db  90h ; ?  
SDPI:0047A5ED                 db  90h ; ?  
SDPI:0047A5EE  
SDPI:0047A5EE ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047A5EE  
SDPI:0047A5EE  
SDPI:0047A5EE sub\_47A5EE      proc near                         ; CODE XREF: SDPI:0047A56D p  
SDPI:0047A5EE                 nop  
SDPI:0047A5EF                 nop  
SDPI:0047A5F0                 nop  
SDPI:0047A5F1                 nop  
SDPI:0047A5F2                 nop  
SDPI:0047A5F3                 call    sub\_47A5F9  
SDPI:0047A5F8                 nop  
SDPI:0047A5F8 sub\_47A5EE      endp  
SDPI:0047A5F8  
SDPI:0047A5F9  
SDPI:0047A5F9 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047A5F9  
SDPI:0047A5F9  
SDPI:0047A5F9 sub\_47A5F9      proc near                         ; CODE XREF: sub\_47A5EE+5 p  
SDPI:0047A5F9                 pop     eax  
SDPI:0047A5FA                 add     eax, 5Eh  
SDPI:0047A5FF                 mov     edx, eax  
SDPI:0047A601                 add     edx, 32h  
SDPI:0047A604                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047A604                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047A604                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047A604                                                   ; 用来解密  
SDPI:0047A609                 call    sub\_47A60F  
SDPI:0047A60E                 nop  
SDPI:0047A60E sub\_47A5F9      endp  
SDPI:0047A60E  
SDPI:0047A60F  
SDPI:0047A60F ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047A60F  
SDPI:0047A60F  
SDPI:0047A60F sub\_47A60F      proc near                         ; CODE XREF: sub\_47A5F9+10 p  
SDPI:0047A60F                 pop     eax  
SDPI:0047A610                 add     eax, 4A8Eh  
SDPI:0047A615                 call    sub\_47A61B  
SDPI:0047A61A                 nop  
SDPI:0047A61A sub\_47A60F      endp  
SDPI:0047A61A  
SDPI:0047A61B  
SDPI:0047A61B ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047A61B  
SDPI:0047A61B  
SDPI:0047A61B sub\_47A61B      proc near                         ; CODE XREF: sub\_47A60F+6 p  
SDPI:0047A61B                 pop     ecx  
SDPI:0047A61C                 add     ecx, 4B2Fh  
SDPI:0047A622                 push    0  
SDPI:0047A624                 push    ecx  
SDPI:0047A625                 push    eax  
SDPI:0047A626                 push    0  
SDPI:0047A628                 call    sub\_47A62E  
SDPI:0047A62D                 nop  
SDPI:0047A62D sub\_47A61B      endp  
SDPI:0047A62D  
SDPI:0047A62E  
SDPI:0047A62E ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047A62E  
SDPI:0047A62E  
SDPI:0047A62E sub\_47A62E      proc near                         ; CODE XREF: sub\_47A61B+D p  
SDPI:0047A62E                 pop     eax  
SDPI:0047A62F                 add     eax, 11h  
SDPI:0047A634                 push    eax  
SDPI:0047A635                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047A635 sub\_47A62E      endp                              ; 判断函数的前 5 位是否为 CC  
SDPI:0047A635                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047A635 ; ----------------------------------------------------------------------------  
SDPI:0047A63A                 db  90h ; ?  
SDPI:0047A63B                 db  90h ; ?  
SDPI:0047A63C                 db  90h ; ?  
SDPI:0047A63E ; ----------------------------------------------------------------------------  
SDPI:0047A63E                 push    7  
SDPI:0047A640                 call    sub\_47A646  
SDPI:0047A645                 nop  
SDPI:0047A646  
SDPI:0047A646 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047A646  
SDPI:0047A646  
SDPI:0047A646 sub\_47A646      proc near                         ; CODE XREF: SDPI:0047A640 p  
SDPI:0047A646                 pop     eax  
SDPI:0047A647                 add     eax, 11h  
SDPI:0047A64C                 push    eax  
SDPI:0047A64D                 jmp     ExitProcess  
SDPI:0047A64D sub\_47A646      endp  
SDPI:0047A64D  
SDPI:0047A64D ; ----------------------------------------------------------------------------  
SDPI:0047A652                 db  90h ; ?  
SDPI:0047A653                 db  90h ; ?  
SDPI:0047A654                 db  90h ; ?  
SDPI:0047A655                 db  90h ; ?  
SDPI:0047A656                 db  90h ; ?  
SDPI:0047A657                 db  90h ; ?  
SDPI:0047A658                 db  90h ; ?  
SDPI:0047A659                 db  90h ; ?  
SDPI:0047A65E  
SDPI:0047A65E INT3\_47a65E:                                      ; CODE XREF: SDPI:0047A572 p  
SDPI:0047A65E                 call    loc\_47A664  
SDPI:0047A663                 nop  
SDPI:0047A664  
SDPI:0047A664 loc\_47A664:                                       ; CODE XREF: SDPI:INT3\_47a65E p  
SDPI:0047A664                 pop     edi  
SDPI:0047A665                 add     edi, 0FFFFFF07h  
SDPI:0047A66B                 mov     \[edi\], ebx                ; 还原前面的花指令,  
SDPI:0047A66D                 mov     \[edi+4\], edx  
SDPI:0047A670                 pop     eax  
SDPI:0047A671                 call    loc\_47A677  
SDPI:0047A676                 nop  
SDPI:0047A677  
SDPI:0047A677 loc\_47A677:                                       ; CODE XREF: SDPI:0047A671 p  
SDPI:0047A677                 pop     eax  
SDPI:0047A678                 add     eax, 124h  
SDPI:0047A67D                 push    eax  
SDPI:0047A67E                 xor     eax, eax  
SDPI:0047A680                 push    dword ptr fs:\[eax\]        ; 设置 SEH  
SDPI:0047A683                 mov     fs:\[eax\], esp  
SDPI:0047A686                 mov     ebp, 300EF1D3h  
SDPI:0047A68B                 add     ebp, 12345678h  
SDPI:0047A691                 mov     ax, 17h  
SDPI:0047A695                 sub     ax, 13h  
SDPI:0047A699                 jl      short loc\_47A6A2  
SDPI:0047A69B  
SDPI:0047A69B loc\_47A69B:                                       ; CODE XREF: SDPI:loc\_47A6A2 j  
SDPI:0047A69B                 jmp     short loc\_47A6A4  
SDPI:0047A69B ; ----------------------------------------------------------------------------  
SDPI:0047A69D                 db    0  
SDPI:0047A69E                 db  10h  
SDPI:0047A69F                 db  40h ; @  
SDPI:0047A6A0                 db    0  
SDPI:0047A6A1                 db 0E8h ; ?  
SDPI:0047A6A2 ; ----------------------------------------------------------------------------  
SDPI:0047A6A2  
SDPI:0047A6A2 loc\_47A6A2:                                       ; CODE XREF: SDPI:0047A699 j  
SDPI:0047A6A2                 jz      short loc\_47A69B  
SDPI:0047A6A4  
SDPI:0047A6A4 loc\_47A6A4:                                       ; CODE XREF: SDPI:loc\_47A69B j  
SDPI:0047A6A4                 jb      short loc\_47A6B8  
SDPI:0047A6A6                 jnb     short loc\_47A6B8  
SDPI:0047A6A6 ; ----------------------------------------------------------------------------  
SDPI:0047A6A8                 dd 401000h  
SDPI:0047A6AC dword\_47A6AC    dd 72C303E8h                      ; CODE XREF: SDPI:0047A6BF j  
SDPI:0047A6B0                 dd 19731Bh  
SDPI:0047A6B4                 dd 0E8004010h  
SDPI:0047A6B8 ; ----------------------------------------------------------------------------  
SDPI:0047A6B8  
SDPI:0047A6B8 loc\_47A6B8:                                       ; CODE XREF: SDPI:loc\_47A6A4 j  
SDPI:0047A6B8                                                   ; SDPI:0047A6A6 j  
SDPI:0047A6B8                 pushfw  
SDPI:0047A6BA                 push    eax  
SDPI:0047A6BB                 xor     eax, eax  
SDPI:0047A6BD                 cmp     ebx, eax  
SDPI:0047A6BF                 jz      short near ptr dword\_47A6AC+1  
SDPI:0047A6C1                 call    loc\_47A6CB  
SDPI:0047A6C1 ; ----------------------------------------------------------------------------  
SDPI:0047A6C6                 dd 401000h                        ; 好多类似这样的垃圾代码  
SDPI:0047A6CA                 db 0E8h ; ?  
SDPI:0047A6CB ; ----------------------------------------------------------------------------  
SDPI:0047A6CB  
SDPI:0047A6CB loc\_47A6CB:                                       ; CODE XREF: SDPI:0047A6C1 p  
SDPI:0047A6CB                 pop     eax  
SDPI:0047A6CC                 pop     eax  
SDPI:0047A6CD                 popfw  
SDPI:0047A6CF                 nop  
SDPI:0047A6D0                 nop  
SDPI:0047A6D1                 nop  
SDPI:0047A6D2                 nop  
SDPI:0047A6D3                 nop  
SDPI:0047A6D4                 int     3                         ; Trap to Debugger  
SDPI:0047A6D5                 nop  
SDPI:0047A6D6                 cmp     al, 4  
SDPI:0047A6D8                 jz      short INT3\_DONE\_7A74B     ; 如果 al 不为 4 则 over 了  
SDPI:0047A6DA  
SDPI:0047A6DA Over\_47a6da:                                      ; CODE XREF: SDPI:0047A764 j  
SDPI:0047A6DA                                                   ; SDPI:0047A77C j ...  
SDPI:0047A6DA                 nop                               ; 这里和前面的 INT3 一样，提示出错信息  
SDPI:0047A6DB                 nop  
SDPI:0047A6DC                 nop  
SDPI:0047A6DD                 nop  
SDPI:0047A6DE                 nop  
SDPI:0047A6DF                 call    loc\_47A6E5  
SDPI:0047A6E4                 nop  
SDPI:0047A6E5  
SDPI:0047A6E5 loc\_47A6E5:                                       ; CODE XREF: SDPI:0047A6DF p  
SDPI:0047A6E5                 pop     eax  
SDPI:0047A6E6                 add     eax, 5Eh  
SDPI:0047A6EB                 mov     edx, eax  
SDPI:0047A6ED                 add     edx, 32h  
SDPI:0047A6F0                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047A6F0                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047A6F0                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047A6F0                                                   ; 用来解密  
SDPI:0047A6F5                 call    loc\_47A6FB  
SDPI:0047A6FA                 nop  
SDPI:0047A6FB  
SDPI:0047A6FB loc\_47A6FB:                                       ; CODE XREF: SDPI:0047A6F5 p  
SDPI:0047A6FB                 pop     eax  
SDPI:0047A6FC                 add     eax, 49A2h  
SDPI:0047A701                 call    loc\_47A707  
SDPI:0047A706                 nop  
SDPI:0047A707  
SDPI:0047A707 loc\_47A707:                                       ; CODE XREF: SDPI:0047A701 p  
SDPI:0047A707                 pop     ecx  
SDPI:0047A708                 add     ecx, 4A43h  
SDPI:0047A70E                 push    0  
SDPI:0047A710                 push    ecx  
SDPI:0047A711                 push    eax  
SDPI:0047A712                 push    0  
SDPI:0047A714                 call    loc\_47A71A  
SDPI:0047A719                 nop  
SDPI:0047A71A  
SDPI:0047A71A loc\_47A71A:                                       ; CODE XREF: SDPI:0047A714 p  
SDPI:0047A71A                 pop     eax  
SDPI:0047A71B                 add     eax, 11h  
SDPI:0047A720                 push    eax  
SDPI:0047A721                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047A721                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047A721                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047A721 ; ----------------------------------------------------------------------------  
SDPI:0047A726                 db  90h ; ?  
INT3\_DONE\_7A74B:                                  ; CODE XREF: SDPI:0047A6D8 j  
SDPI:0047A74B                 pop     large dword ptr fs:0  
SDPI:0047A752                 add     esp, 4  
SDPI:0047A755                 call    loc\_47A75B  
SDPI:0047A75A                 nop  
SDPI:0047A75B  
SDPI:0047A75B loc\_47A75B:                                       ; CODE XREF: SDPI:0047A755 p  
SDPI:0047A75B                 pop     eax  
SDPI:0047A75C                 add     eax, 0FFFFFE1Dh  
SDPI:0047A761                 cmp     byte ptr \[eax\], 0E9h      ; SEH 时改成了 0E9,所以这里判断是否为 0e9  
SDPI:0047A761                                                   ; 不是则 over  
SDPI:0047A764                 jnz     Over\_47a6da               ; 这里和前面的 INT3 一样，提示出错信息  
SDPI:0047A76A                 mov     byte ptr \[eax\], 0E8h      ; 改回原代码  
SDPI:0047A76D                 rdtsc  
SDPI:0047A76F                 mov     ecx, eax  
SDPI:0047A771                 mov     ebx, edx  
SDPI:0047A773                 rdtsc  
SDPI:0047A775                 sub     eax, ecx  
SDPI:0047A777                 sbb     edx, ebx  
SDPI:0047A779                 cmp     edx, 0                    ; int3 后面紧跟着时间检测  
SDPI:0047A77C                 jnz     Over\_47a6da               ; 这里和前面的 INT3 一样，提示出错信息  
SDPI:0047A782                 cmp     eax, 30000000h  
SDPI:0047A787                 ja      Over\_47a6da               ; 这里和前面的 INT3 一样，提示出错信息  
SDPI:0047A78D                 jz      short Nodbg\_47A7D8  
SDPI:0047A78F                 jnz     short Nodbg\_47A7D8  
SDPI:0047A78F ; ----------------------------------------------------------------------------  
SDPI:0047A791                 db 0E8h  
SDPI:0047A792                 db    0  
SDPI:0047A793                 db  10h  
SDPI:0047A794                 db  40h ; @  
SDPI:0047A795                 db    0  
SDPI:0047A796                 db 0B0h  
SDPI:0047A797                 db  89h ; ?  
SDPI:0047A798 ; ----------------------------------------------------------------------------  
SDPI:0047A798                 pushf  
SDPI:0047A798 ; ----------------------------------------------------------------------------  
SDPI:0047A799                 db 4  
SDPI:0047A79A ; ----------------------------------------------------------------------------  
SDPI:0047A79A                 mov     eax, \[esp+4\]              ; SEH 处理处  
SDPI:0047A79E                 mov     ecx, \[esp+0Ch\]  
SDPI:0047A7A2                 inc     dword ptr \[ecx+0B8h\]      ; reg\[EIP\]+1  
SDPI:0047A7A8                 mov     eax, \[eax\]  
SDPI:0047A7AA                 sub     eax, 80000003h            ; 判断是否为 CC 异常  
SDPI:0047A7AF                 jnz     short locret\_47A7D7  
SDPI:0047A7B1                 call    sub\_47A7B7  
SDPI:0047A7B6                 nop  
SDPI:0047A7B7  
SDPI:0047A7B7 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047A7B7  
SDPI:0047A7B7  
SDPI:0047A7B7 sub\_47A7B7      proc near                         ; CODE XREF: SDPI:0047A7B1 p  
SDPI:0047A7B7                 pop     eax  
SDPI:0047A7B8                 add     eax, 0FFFFFDC1h  
SDPI:0047A7BD                 cmp     byte ptr \[eax\], 0E8h      ; 判断 0047A577 处是否为 0E8,不则则 over 了  
SDPI:0047A7BD                                                   ; 如果是则改写为 0E9  
SDPI:0047A7BD                                                   ;  
SDPI:0047A7C0                 jnz     Over\_47a6da               ; 这里和前面的 INT3 一样，提示出错信息  
SDPI:0047A7C6                 mov     byte ptr \[eax\], 0E9h  
SDPI:0047A7C9                 xor     eax, eax  
SDPI:0047A7CB                 mov     \[ecx+4\], eax  
SDPI:0047A7CE                 mov     \[ecx+8\], eax  
SDPI:0047A7D1                 mov     \[ecx+0Ch\], eax  
SDPI:0047A7D4                 mov     \[ecx+10h\], eax  
SDPI:0047A7D7  
SDPI:0047A7D7 locret\_47A7D7:                                    ; CODE XREF: SDPI:0047A7AF j  
SDPI:0047A7D7                 retn  
SDPI:0047A7D7 sub\_47A7B7      endp ; sp =  4  
SDPI:0047A7D7  
SDPI:0047A7D8 ; ----------------------------------------------------------------------------  
SDPI:0047A7D8  
SDPI:0047A7D8 Nodbg\_47A7D8:                                     ; CODE XREF: SDPI:0047A78D j  
SDPI:0047A7D8                                                   ; SDPI:0047A78F j  
SDPI:0047A7D8                 pop     eax  
SDPI:0047A7D9                 call    Call\_GetTickCount         ; 这里开始有几个 GetTickCount  
SDPI:0047A7D9                                                   ; 来检测调试器.没处理好这里很容易被挂的:-)  
SDPI:0047A7D9 ; ----------------------------------------------------------------------------  
SDPI:0047A7DE                 dd 401000h  
SDPI:0047A7E2                 dd 15C56BEh  
SDPI:0047A7E6 ; ----------------------------------------------------------------------------  
SDPI:0047A7E6  
SDPI:0047A7E6 junk\_47a7e6:                                      ; CODE XREF: SDPI:0047A9E4 p  
SDPI:0047A7E6                 pop     ebp  
SDPI:0047A7E7                 pop     eax  
SDPI:0047A7E8                 jmp     loc\_47A9E9  
SDPI:0047A7ED ; ----------------------------------------------------------------------------  
SDPI:0047A7ED                 mov     ecx, 0FFFFFF00h  
SDPI:0047A7F2                 push    fs  
SDPI:0047A7F4                 jz      short loc\_47A800  
SDPI:0047A7F6                 jnz     short loc\_47A800  
SDPI:0047A7F6 ; ----------------------------------------------------------------------------  
SDPI:0047A7F8                 dd 401000h  
SDPI:0047A7FC                 dd 49C89B0h  
SDPI:0047A800 ; ----------------------------------------------------------------------------  
SDPI:0047A800  
SDPI:0047A800 loc\_47A800:                                       ; CODE XREF: SDPI:0047A7F4 j  
SDPI:0047A800                                                   ; SDPI:0047A7F6 j  
SDPI:0047A800                 pushfw  
SDPI:0047A802                 push    eax  
SDPI:0047A803                 mov     eax, ebx                  ; junk  
SDPI:0047A805                 push    ebx  
SDPI:0047A806                 mov     eax, ecx                  ; mov eax,-100  
SDPI:0047A808                 push    eax  
SDPI:0047A809                 add     eax, edx                  ; 这也实际上是 mov eax,edx  
SDPI:0047A809                                                   ; sub eax,100  
SDPI:0047A809                                                   ; mov ebx,eax  
SDPI:0047A80B                 mov     ebx, eax  
SDPI:0047A80D                 push    ebx  
SDPI:0047A80E                 pop     eax  
SDPI:0047A80F                 push    edx  
SDPI:0047A810                 call    loc\_47A81D  
SDPI:0047A810 ; ----------------------------------------------------------------------------  
SDPI:0047A815                 dd 401000h  
SDPI:0047A819                 dd 132BD7B0h  
SDPI:0047A81D ; ----------------------------------------------------------------------------  
SDPI:0047A81D  
SDPI:0047A81D loc\_47A81D:                                       ; CODE XREF: SDPI:0047A810 p  
SDPI:0047A81D                 pop     eax  
SDPI:0047A81E                 call    loc\_47A824  
SDPI:0047A823                 nop  
SDPI:0047A824  
SDPI:0047A824 loc\_47A824:                                       ; CODE XREF: SDPI:0047A81E p  
SDPI:0047A824                 pop     eax  
SDPI:0047A825                 add     eax, 11h  
SDPI:0047A82A                 push    eax  
SDPI:0047A82B                 jmp     GetTickCount  
SDPI:0047A82B ; ----------------------------------------------------------------------------  
SDPI:0047A830                 db  90h ; ?  
SDPI:0047A831                 db  90h ; ?  
SDPI:0047A832                 db  90h ; ?  
SDPI:0047A833                 db  90h ; ?  
SDPI:0047A834 ; ----------------------------------------------------------------------------  
SDPI:0047A834                 push    eax  
SDPI:0047A835                 mov     eax, edx  
SDPI:0047A837                 push    eax  
SDPI:0047A838                 call    loc\_47A83E  
SDPI:0047A83D                 nop  
SDPI:0047A83E  
SDPI:0047A83E loc\_47A83E:                                       ; CODE XREF: SDPI:0047A838 p  
SDPI:0047A83E                 pop     edx  
SDPI:0047A83F                 add     edx, 52h  
SDPI:0047A845                 push    edx  
SDPI:0047A846                 add     edx, 401846h  
SDPI:0047A84C                 push    edx  
SDPI:0047A84D                 jo      short loc\_47A8A2  
SDPI:0047A84F                 jno     short loc\_47A8A2  
SDPI:0047A851  
SDPI:0047A851 loc\_47A851:                                       ; CODE XREF: SDPI:0047A895 p  
SDPI:0047A851                 pop     eax  
SDPI:0047A852                 pop     ebx  
SDPI:0047A853                 call    loc\_47A859  
SDPI:0047A858                 nop  
SDPI:0047A859  
SDPI:0047A859 loc\_47A859:                                       ; CODE XREF: SDPI:0047A853 p  
SDPI:0047A859                 pop     eax  
SDPI:0047A85A                 add     eax, 11h  
SDPI:0047A85F                 push    eax  
SDPI:0047A860                 jmp     GetTickCount  
SDPI:0047A860 ; ----------------------------------------------------------------------------  
SDPI:0047A865                 db  90h ; ?  
SDPI:0047A866                 db  90h ; ?  
SDPI:0047A867                 db  90h ; ?  
SDPI:0047A868                 db  90h ; ?  
SDPI:0047A869 ; ----------------------------------------------------------------------------  
SDPI:0047A869                 pop     ebx  
SDPI:0047A86A                 add     ebx, 1F4h                 ; ===========  
SDPI:0047A86A                                                   ; 这里注意了，每比较第二次的时间  
SDPI:0047A86A                                                   ; 第一次的时间值放到堆栈中  
SDPI:0047A86A                                                   ; 这里不能跳了，跳就 over  
SDPI:0047A870                 sub     ebx, eax  
SDPI:0047A872                 js      short Over\_47A8B6         ; 因为壳很多地方用到的变形 call,  
SDPI:0047A872                                                   ; 所以不是很好分模块  
SDPI:0047A874                 call    loc\_47A87A  
SDPI:0047A879                 nop  
SDPI:0047A87A  
SDPI:0047A87A loc\_47A87A:                                       ; CODE XREF: SDPI:0047A874 p  
SDPI:0047A87A                 pop     ebx  
SDPI:0047A87B                 add     ebx, 0A5h  
SDPI:0047A881                 push    ebx  
SDPI:0047A882                 call    sub\_47A8AC  
SDPI:0047A887                 add     \[eax\], dl  
SDPI:0047A889                 inc     eax  
SDPI:0047A88A                 add     \[eax+58058C88h\], dh  
SDPI:0047A890                 mov     edx, eax  
SDPI:0047A892                 mov     eax, ebx  
SDPI:0047A894                 push    eax  
SDPI:0047A895                 call    loc\_47A851  
SDPI:0047A89A                 add     \[eax\], dl  
SDPI:0047A89C                 inc     eax  
SDPI:0047A89D                 add     \[ecx\], bh  
SDPI:0047A89D ; ----------------------------------------------------------------------------  
SDPI:0047A89F                 db  36h ; 6  
SDPI:0047A8A0                 db  83h ; ?  
SDPI:0047A8A1                 db    1  
SDPI:0047A8A2 ; ----------------------------------------------------------------------------  
SDPI:0047A8A2  
SDPI:0047A8A2 loc\_47A8A2:                                       ; CODE XREF: SDPI:0047A84D j  
SDPI:0047A8A2                                                   ; SDPI:0047A84F j  
SDPI:0047A8A2                 pop     eax  
SDPI:0047A8A3                 retn  
SDPI:0047A8A3 ; ----------------------------------------------------------------------------  
SDPI:0047A8A4                 db    0  
SDPI:0047A8A5                 db  10h  
SDPI:0047A8A6                 db  40h ; @  
SDPI:0047A8A7                 db    0  
SDPI:0047A8A8                 db  3Eh ; >  
SDPI:0047A8A9                 db  56h ; V  
SDPI:0047A8AA                 db  7Ch ; |  
SDPI:0047A8AB                 db    7  
SDPI:0047A8AC  
SDPI:0047A8AC ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047A8AC  
SDPI:0047A8AC  
SDPI:0047A8AC sub\_47A8AC      proc near                         ; CODE XREF: SDPI:0047A882 p  
SDPI:0047A8AC                 pop     edx  
SDPI:0047A8AD                 retn  
SDPI:0047A8AD sub\_47A8AC      endp ; sp =  4  
SDPI:0047A8AD  
SDPI:0047A8AD ; ----------------------------------------------------------------------------  
SDPI:0047A8AE                 db    0  
SDPI:0047A8AF                 db  10h  
SDPI:0047A8B0                 db  40h ; @  
SDPI:0047A8B1                 db    0  
SDPI:0047A8B2                 db 0EFh ; ?  
SDPI:0047A8B3                 db  53h ; S  
SDPI:0047A8B4                 db 0EDh ; ?  
SDPI:0047A8B5                 db    1  
SDPI:0047A8B6  
SDPI:0047A8B6 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047A8B6  
SDPI:0047A8B6 ; 因为壳很多地方用到的变形 call,  
SDPI:0047A8B6 ; 所以不是很好分模块  
SDPI:0047A8B6  
SDPI:0047A8B6 Over\_47A8B6     proc near                         ; CODE XREF: SDPI:0047A872 j  
SDPI:0047A8B6                 nop  
SDPI:0047A8B7                 nop  
SDPI:0047A8B8                 nop  
SDPI:0047A8B9                 nop  
SDPI:0047A8BA                 nop  
SDPI:0047A8BB                 call    loc\_47A8C1  
SDPI:0047A8C0                 nop  
SDPI:0047A8C1  
SDPI:0047A8C1 loc\_47A8C1:                                       ; CODE XREF: Over\_47A8B6+5 p  
SDPI:0047A8C1                 pop     eax  
SDPI:0047A8C2                 add     eax, 5Eh  
SDPI:0047A8C7                 mov     edx, eax  
SDPI:0047A8C9                 add     edx, 32h  
SDPI:0047A8CC                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047A8CC                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047A8CC                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047A8CC                                                   ; 用来解密  
SDPI:0047A8D1                 call    loc\_47A8D7  
SDPI:0047A8D6                 nop  
SDPI:0047A8D7  
SDPI:0047A8D7 loc\_47A8D7:                                       ; CODE XREF: Over\_47A8B6+1B p  
SDPI:0047A8D7                 pop     eax  
SDPI:0047A8D8                 add     eax, 47C6h  
SDPI:0047A8DD                 call    loc\_47A8E3  
SDPI:0047A8E2                 nop  
SDPI:0047A8E3  
SDPI:0047A8E3 loc\_47A8E3:                                       ; CODE XREF: Over\_47A8B6+27 p  
SDPI:0047A8E3                 pop     ecx  
SDPI:0047A8E4                 add     ecx, 4867h  
SDPI:0047A8EA                 push    0  
SDPI:0047A8EC                 push    ecx  
SDPI:0047A8ED                 push    eax  
SDPI:0047A8EE                 push    0  
SDPI:0047A8F0                 call    loc\_47A8F6  
SDPI:0047A8F5                 nop  
SDPI:0047A8F6  
SDPI:0047A8F6 loc\_47A8F6:                                       ; CODE XREF: Over\_47A8B6+3A p  
SDPI:0047A8F6                 pop     eax  
SDPI:0047A8F7                 add     eax, 11h  
SDPI:0047A8FC                 push    eax  
SDPI:0047A8FD                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047A8FD                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047A8FD                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047A8FD ; ----------------------------------------------------------------------------  
SDPI:0047A902                 db  90h ; ?  
SDPI:0047A903                 db  90h ; ?  
SDPI:0047A904                 db  90h ; ?  
SDPI:0047A905                 db  90h ; ?  
SDPI:0047A906 ; ----------------------------------------------------------------------------  
SDPI:0047A906                 push    7  
SDPI:0047A908                 call    loc\_47A90E  
SDPI:0047A90D                 nop  
SDPI:0047A90E  
SDPI:0047A90E loc\_47A90E:                                       ; CODE XREF: Over\_47A8B6+52 p  
SDPI:0047A90E                 pop     eax  
SDPI:0047A90F                 add     eax, 11h  
SDPI:0047A914                 push    eax  
SDPI:0047A915                 jmp     ExitProcess  
SDPI:0047A915 ; ----------------------------------------------------------------------------  
SDPI:0047A91A                 db  90h ; ?  
SDPI:0047A91B                 db  90h ; ?  
SDPI:0047A91C                 db  90h ; ?  
SDPI:0047A91D                 db  90h ; ?  
SDPI:0047A91D Over\_47A8B6     endp  
SDPI:0047A91D  
SDPI:0047A91E ; ----------------------------------------------------------------------------  
SDPI:0047A91E                 pop     edx                       ; 第一次计算出的时间出栈  
SDPI:0047A91F                 mov     eax, ecx  
SDPI:0047A921                 add     eax, edx  
SDPI:0047A923                 inc     ecx  
SDPI:0047A924                 push    eax  
SDPI:0047A925                 inc     ecx  
SDPI:0047A926                 pop     ebx  
SDPI:0047A927                 pop     ecx  
SDPI:0047A928                 push    eax  
SDPI:0047A929                 sub     eax, 8  
SDPI:0047A92C                 pop     ebx  
SDPI:0047A92D                 pop     ebx  
SDPI:0047A92E                 inc     eax  
SDPI:0047A92F                 add     eax, ebx  
SDPI:0047A931                 pop     eax  
SDPI:0047A932                 pushfw  
SDPI:0047A934                 popfw  
SDPI:0047A936                 popfw  
SDPI:0047A938                 pop     es  
SDPI:0047A939                 mov     eax, 12345678h  
SDPI:0047A93E                 push    eax  
SDPI:0047A93F                 call    loc\_47A945  
SDPI:0047A944                 nop  
SDPI:0047A945  
SDPI:0047A945 loc\_47A945:                                       ; CODE XREF: SDPI:0047A93F p  
SDPI:0047A945                 pop     eax  
SDPI:0047A946                 add     eax, 12Ch  
SDPI:0047A94B                 push    eax  
SDPI:0047A94C                 pop     ebx  
SDPI:0047A94D                 add     eax, 12h                  ; 一堆交换来交换去的代码:-(  
SDPI:0047A950                 pop     edx  
SDPI:0047A951                 add     eax, edx  
SDPI:0047A953                 mov     edx, eax  
SDPI:0047A955                 push    ebx  
SDPI:0047A956                 mov     ebx, es:\[ecx+100h\]  
SDPI:0047A95D                 push    ebx  
SDPI:0047A95E                 mov     eax, esp  
SDPI:0047A960                 mov     ebx, eax  
SDPI:0047A962                 push    ebx  
SDPI:0047A963                 pop     edx  
SDPI:0047A964                 mov     es:\[ecx+100h\], eax  
SDPI:0047A96B                 xor     eax, eax  
SDPI:0047A96D                 jle     short loc\_47A976  
SDPI:0047A96F                 jg      short loc\_47A976  
SDPI:0047A971                 add     \[eax\], dl  
SDPI:0047A973                 inc     eax  
SDPI:0047A974                 add     al, ch  
SDPI:0047A976  
SDPI:0047A976 loc\_47A976:                                       ; CODE XREF: SDPI:0047A96D j  
SDPI:0047A976                                                   ; SDPI:0047A96F j  
SDPI:0047A976                 pushfw  
SDPI:0047A978                 push    ecx  
SDPI:0047A979                 xor     ecx, ecx  
SDPI:0047A97B                 jcxz    loc\_47A983  
SDPI:0047A97E                 add     \[eax\], dl  
SDPI:0047A980                 inc     eax  
SDPI:0047A981                 add     al, ch  
SDPI:0047A983  
SDPI:0047A983 loc\_47A983:                                       ; CODE XREF: SDPI:0047A97B j  
SDPI:0047A983                 pop     ecx  
SDPI:0047A984                 nop  
SDPI:0047A985                 nop  
SDPI:0047A986                 nop  
SDPI:0047A987                 nop  
SDPI:0047A988                 nop  
SDPI:0047A989                 nop  
SDPI:0047A98A                 nop  
SDPI:0047A98B                 nop  
SDPI:0047A98C                 nop  
SDPI:0047A98D                 nop  
SDPI:0047A98E                 nop  
SDPI:0047A98F                 nop  
SDPI:0047A990                 nop  
SDPI:0047A991                 nop  
SDPI:0047A992                 nop  
SDPI:0047A993                 nop  
SDPI:0047A994                 nop  
SDPI:0047A995                 nop  
SDPI:0047A996                 nop  
SDPI:0047A997                 nop  
SDPI:0047A998                 nop  
SDPI:0047A999                 nop  
SDPI:0047A99A                 nop  
SDPI:0047A99B                 popfw  
SDPI:0047A99D                 jo      short loc\_47A9A5  
SDPI:0047A99F                 jno     short loc\_47A9A5  
SDPI:0047A99F ; ----------------------------------------------------------------------------  
SDPI:0047A9A1                 dd 401000h  
SDPI:0047A9A5 ; ----------------------------------------------------------------------------  
SDPI:0047A9A5  
SDPI:0047A9A5 loc\_47A9A5:                                       ; CODE XREF: SDPI:0047A99D j  
SDPI:0047A9A5                                                   ; SDPI:0047A99F j  
SDPI:0047A9A5                 int     3                         ; Trap to Debugger  
SDPI:0047A9A6                 nop                               ; 这里发生异常后跳去 SEH(0047AA70)处  
SDPI:0047A9A7                 xor     eax, eax  
SDPI:0047A9A9                 mov     dword ptr \[eax\], 401AA9h  ; 这里是第三个异常了  
SDPI:0047A9AF                 jp      short Call\_GetTickCount  
SDPI:0047A9B1                 jnp     short Call\_GetTickCount  
SDPI:0047A9B3                 add     \[eax\], dl  
SDPI:0047A9B5                 inc     eax  
SDPI:0047A9B6                 add     \[ebx+3Dh\], bh  
SDPI:0047A9B9                 inc     eax  
SDPI:0047A9B9 ; ----------------------------------------------------------------------------  
SDPI:0047A9BA                 db    0  
SDPI:0047A9BB ; ----------------------------------------------------------------------------  
SDPI:0047A9BB  
SDPI:0047A9BB Call\_GetTickCount:                                ; CODE XREF: SDPI:0047A7D9 p  
SDPI:0047A9BB                                                   ; SDPI:0047A9AF j ...  
SDPI:0047A9BB                 call    loc\_47A9C1  
SDPI:0047A9C0                 nop  
SDPI:0047A9C1  
SDPI:0047A9C1 loc\_47A9C1:                                       ; CODE XREF: SDPI:Call\_GetTickCount p  
SDPI:0047A9C1                 pop     eax  
SDPI:0047A9C2                 add     eax, 11h  
SDPI:0047A9C7                 push    eax  
SDPI:0047A9C8                 jmp     GetTickCount  
SDPI:0047A9C8 ; ----------------------------------------------------------------------------  
SDPI:0047A9CD                 db  90h ; ?  
SDPI:0047A9CE                 db  90h ; ?  
SDPI:0047A9CF                 db  90h ; ?  
SDPI:0047A9D0                 db  90h ; ?  
SDPI:0047A9D1 ; ----------------------------------------------------------------------------  
SDPI:0047A9D1                 call    loc\_47A9D7  
SDPI:0047A9D6                 nop  
SDPI:0047A9D7  
SDPI:0047A9D7 loc\_47A9D7:                                       ; CODE XREF: SDPI:0047A9D1 p  
SDPI:0047A9D7                 pop     edx  
SDPI:0047A9D8                 add     edx, 0FFFFFB04h  
SDPI:0047A9DE                 mov     \[edx\], eax                ; 保存第一次获取的时间 01B85F98  
SDPI:0047A9E0                 pop     ebp  
SDPI:0047A9E1                 add     eax, edx  
SDPI:0047A9E3                 push    eax  
SDPI:0047A9E4                 call    junk\_47a7e6  
SDPI:0047A9E9  
SDPI:0047A9E9 loc\_47A9E9:                                       ; CODE XREF: SDPI:0047A7E8 j  
SDPI:0047A9E9                 call    sub\_47A9EF  
SDPI:0047A9EE                 nop  
SDPI:0047A9EF  
SDPI:0047A9EF ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047A9EF  
SDPI:0047A9EF  
SDPI:0047A9EF sub\_47A9EF      proc near                         ; CODE XREF: SDPI:loc\_47A9E9 p  
SDPI:0047A9EF                 pop     edx  
SDPI:0047A9F0                 add     edx, 0FFFFFDFFh  
SDPI:0047A9F6                 add     edx, eax  
SDPI:0047A9F8                 push    edx  
SDPI:0047A9F9                 pop     ecx  
SDPI:0047A9FA                 sub     ecx, eax  
SDPI:0047A9FC                 push    ecx  
SDPI:0047A9FD                 retn    4  
SDPI:0047A9FD sub\_47A9EF      endp  
SDPI:0047A9FD  
SDPI:0047AA00 ; ----------------------------------------------------------------------------  
SDPI:0047AA00  
SDPI:0047AA00 OVER\_47AA00:                                      ; CODE XREF: SDPI:0047AAA3 j  
SDPI:0047AA00                                                   ; SDPI:0047AAAB j ...  
SDPI:0047AA00                 nop                               ; 发现了调试器则这里显示错误信息  
SDPI:0047AA01                 nop  
SDPI:0047AA02                 nop  
SDPI:0047AA03                 nop  
SDPI:0047AA04                 nop  
SDPI:0047AA05                 call    sub\_47AA0B  
SDPI:0047AA0A                 nop  
SDPI:0047AA0B  
SDPI:0047AA0B ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047AA0B  
SDPI:0047AA0B  
SDPI:0047AA0B sub\_47AA0B      proc near                         ; CODE XREF: SDPI:0047AA05 p  
SDPI:0047AA0B                 pop     eax  
SDPI:0047AA0C                 add     eax, 5Eh  
SDPI:0047AA11                 mov     edx, eax  
SDPI:0047AA13                 add     edx, 32h  
SDPI:0047AA16                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047AA16                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047AA16                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047AA16                                                   ; 用来解密  
SDPI:0047AA1B                 call    sub\_47AA21  
SDPI:0047AA20                 nop  
SDPI:0047AA20 sub\_47AA0B      endp  
SDPI:0047AA20  
SDPI:0047AA21  
SDPI:0047AA21 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047AA21  
SDPI:0047AA21  
SDPI:0047AA21 sub\_47AA21      proc near                         ; CODE XREF: sub\_47AA0B+10 p  
SDPI:0047AA21                 pop     eax  
SDPI:0047AA22                 add     eax, 467Ch  
SDPI:0047AA27                 call    sub\_47AA2D  
SDPI:0047AA2C                 nop  
SDPI:0047AA2C sub\_47AA21      endp  
SDPI:0047AA2C  
SDPI:0047AA2D  
SDPI:0047AA2D ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047AA2D  
SDPI:0047AA2D  
SDPI:0047AA2D sub\_47AA2D      proc near                         ; CODE XREF: sub\_47AA21+6 p  
SDPI:0047AA2D                 pop     ecx  
SDPI:0047AA2E                 add     ecx, 471Dh  
SDPI:0047AA34                 push    0  
SDPI:0047AA36                 push    ecx  
SDPI:0047AA37                 push    eax  
SDPI:0047AA38                 push    0  
SDPI:0047AA3A                 call    sub\_47AA40  
SDPI:0047AA3F                 nop  
SDPI:0047AA3F sub\_47AA2D      endp  
SDPI:0047AA3F  
SDPI:0047AA40  
SDPI:0047AA40 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047AA40  
SDPI:0047AA40  
SDPI:0047AA40 sub\_47AA40      proc near                         ; CODE XREF: sub\_47AA2D+D p  
SDPI:0047AA40                 pop     eax  
SDPI:0047AA41                 add     eax, 11h  
SDPI:0047AA46                 push    eax  
SDPI:0047AA47                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047AA47 sub\_47AA40      endp                              ; 判断函数的前 5 位是否为 CC  
SDPI:0047AA47                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047AA47 ; ----------------------------------------------------------------------------  
SDPI:0047AA4C                 db  90h ; ?  
SDPI:0047AA4D                 db  90h ; ?  
SDPI:0047AA4E                 db  90h ; ?  
SDPI:0047AA4F                 db  90h ; ?  
SDPI:0047AA50 ; ----------------------------------------------------------------------------  
SDPI:0047AA50                 push    7  
SDPI:0047AA52                 call    sub\_47AA58  
SDPI:0047AA57                 nop  
SDPI:0047AA58  
SDPI:0047AA58 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047AA58  
SDPI:0047AA58  
SDPI:0047AA58 sub\_47AA58      proc near                         ; CODE XREF: SDPI:0047AA52 p  
SDPI:0047AA58                 pop     eax  
SDPI:0047AA59                 add     eax, 11h  
SDPI:0047AA5E                 push    eax  
SDPI:0047AA5F                 jmp     ExitProcess  
SDPI:0047AA5F sub\_47AA58      endp  
SDPI:0047AA5F  
SDPI:0047AA5F ; ----------------------------------------------------------------------------  
SDPI:0047AA64                 db  90h ; ?  
SDPI:0047AA65                 db  90h ; ?  
SDPI:0047AA66                 db  90h ; ?  
SDPI:0047AA67                 db  90h ; ?  
SDPI:0047AA68                 db    0  
SDPI:0047AA69                 db  10h  
SDPI:0047AA6A                 db  40h ; @  
SDPI:0047AA6B                 db    0  
SDPI:0047AA6C                 db 0BEh ; ?  
SDPI:0047AA6D                 db  56h ; V  
SDPI:0047AA6E                 db  5Ch ; \\  
SDPI:0047AA6F                 db    1  
SDPI:0047AA70 ; ----------------------------------------------------------------------------  
SDPI:0047AA70  
SDPI:0047AA70 SEH\_HND\_47A9A5:                                   ; SEH HANDLE\_0047A9A5  
SDPI:0047AA70                 mov     esp, \[esp+8\]  
SDPI:0047AA74                 pop     large dword ptr fs:0  
SDPI:0047AA7B                 call    loc\_47AA81  
SDPI:0047AA80                 nop  
SDPI:0047AA81  
SDPI:0047AA81 loc\_47AA81:                                       ; CODE XREF: SDPI:0047AA7B p  
SDPI:0047AA81                 pop     eax  
SDPI:0047AA82                 add     eax, 11h  
SDPI:0047AA87                 push    eax  
SDPI:0047AA88                 jmp     GetTickCount  
SDPI:0047AA88 ; ----------------------------------------------------------------------------  
SDPI:0047AA8D                 db  90h ; ?  
SDPI:0047AA8E                 db  90h ; ?  
SDPI:0047AA8F                 db  90h ; ?  
SDPI:0047AA90                 db  90h ; ?  
SDPI:0047AA91 ; ----------------------------------------------------------------------------  
SDPI:0047AA91                 call    loc\_47AA97  
SDPI:0047AA96                 nop  
SDPI:0047AA97  
SDPI:0047AA97 loc\_47AA97:                                       ; CODE XREF: SDPI:0047AA91 p  
SDPI:0047AA97                 pop     edx  
SDPI:0047AA98                 add     edx, 0FFFFFA44h  
SDPI:0047AA9E                 mov     ecx, \[edx\]  
SDPI:0047AAA0                 cmp     ecx, 0                    ; 判断时间是否为 0  
SDPI:0047AAA0                                                   ; 也就是判断是否被我们手工修改过  
SDPI:0047AAA0                                                   ; GetTickCount  
SDPI:0047AAA3                 jz      OVER\_47AA00               ; 发现了调试器则这里显示错误信息  
SDPI:0047AAA9                 sub     eax, ecx  
SDPI:0047AAAB                 js      OVER\_47AA00               ; 发现了调试器则这里显示错误信息  
SDPI:0047AAB1                 sub     eax, 7D0h                 ; 这几个地方都不能跳了，跳就 over 了  
SDPI:0047AAB6                 jns     OVER\_47AA00               ; 发现了调试器则这里显示错误信息  
SDPI:0047AABC                 mov     eax, 0E801276h  
SDPI:0047AAC1                 mov     \[edx\], eax  
SDPI:0047AAC3                 call    loc\_47AAC9  
SDPI:0047AAC8                 nop  
SDPI:0047AAC9  
SDPI:0047AAC9 loc\_47AAC9:                                       ; CODE XREF: SDPI:0047AAC3 p  
SDPI:0047AAC9                 pop     edx  
SDPI:0047AACA                 add     edx, 30Fh  
SDPI:0047AAD0                 call    loc\_47AAD6  
SDPI:0047AAD5                 nop  
SDPI:0047AAD6  
SDPI:0047AAD6 loc\_47AAD6:                                       ; CODE XREF: SDPI:0047AAD0 p  
SDPI:0047AAD6                 pop     eax  
SDPI:0047AAD7                 add     eax, 0FFFFF67Bh  
SDPI:0047AADC                 mov     ecx, 10h                  ; 这里就是前面用 MD5 加密处  
SDPI:0047AAE1                 call    De\_Code                   ; invoke De\_Code,ecx(counter),eax(key),edx(end)  
SDPI:0047AAE1                                                   ; 用于解密代码，解密起始地址就是 call 进来  
SDPI:0047AAE1                                                   ; 的下一行代码地址  
SDPI:0047AAE6                 call    loc\_47AAEC  
SDPI:0047AAEB                 nop  
SDPI:0047AAEC  
SDPI:0047AAEC loc\_47AAEC:                                       ; CODE XREF: SDPI:0047AAE6 p  
SDPI:0047AAEC                 pop     eax  
SDPI:0047AAED                 add     eax, 11h  
SDPI:0047AAF2                 push    eax  
SDPI:0047AAF3                 jmp     CMPHASH\_48147D            ; 计算并比较 MD5 值,判断代码是否被修改过  
SDPI:0047AAF3 ; ----------------------------------------------------------------------------  
SDPI:0047AAF8                 db  90h ; ?  
SDPI:0047AAF9                 db  90h ; ?  
SDPI:0047AAFA                 db  90h ; ?  
SDPI:0047AAFB                 db  90h ; ?  
SDPI:0047AAFC ; ----------------------------------------------------------------------------  
SDPI:0047AAFC                 call    loc\_47AB02  
SDPI:0047AB01                 nop  
SDPI:0047AB02  
SDPI:0047AB02 loc\_47AB02:                                       ; CODE XREF: SDPI:0047AAFC p  
SDPI:0047AB02                 pop     eax  
SDPI:0047AB03                 add     eax, 11h  
SDPI:0047AB08                 push    eax  
SDPI:0047AB09                 jmp     Anti\_DBG\_482360           ; 进去还是检测调试器:-(,  
SDPI:0047AB09                                                   ; 光检测调试器不干活了  
SDPI:0047AB09 ; ----------------------------------------------------------------------------  
SDPI:0047AB0E                 db  90h ; ?  
SDPI:0047AB0F                 db  90h ; ?  
SDPI:0047AB10                 db  90h ; ?  
SDPI:0047AB11                 db  90h ; ?  
SDPI:0047AB12 ; ----------------------------------------------------------------------------  
SDPI:0047AB12                 sub     ebx, eax  
SDPI:0047AB14                 add     ecx, ebx  
SDPI:0047AB16                 xor     ebx, ebx  
SDPI:0047AB18                 sub     eax, 0D246534Fh           ; 又是比较，这个比较好对付  
SDPI:0047AB18                                                   ; 直接在那个 call 里返回就行了  
SDPI:0047AB1D                 jle     short loc\_47AB26  
SDPI:0047AB1F                 jg      short loc\_47AB26  
SDPI:0047AB1F ; ----------------------------------------------------------------------------  
SDPI:0047AB21                 dd 401000h  
SDPI:0047AB25                 db 0E8h ; ?  
SDPI:0047AB26 ; ----------------------------------------------------------------------------  
SDPI:0047AB26  
SDPI:0047AB26 loc\_47AB26:                                       ; CODE XREF: SDPI:0047AB1D j  
SDPI:0047AB26                                                   ; SDPI:0047AB1F j  
SDPI:0047AB26                 pushfw  
SDPI:0047AB28                 push    ecx                       ; 很多地方有类似这样的垃圾代码  
SDPI:0047AB29                 xor     ecx, ecx  
SDPI:0047AB2B                 jcxz    loc\_47AB33  
SDPI:0047AB2E                 add     \[eax\], dl  
SDPI:0047AB30                 inc     eax  
SDPI:0047AB31                 add     al, ch  
SDPI:0047AB33  
SDPI:0047AB33 loc\_47AB33:                                       ; CODE XREF: SDPI:0047AB2B j  
SDPI:0047AB33                 pop     ecx  
SDPI:0047AB34                 nop  
SDPI:0047AB35                 nop  
SDPI:0047AB36                 nop  
SDPI:0047AB37                 nop  
SDPI:0047AB38                 nop  
SDPI:0047AB39                 nop  
SDPI:0047AB3A                 nop  
SDPI:0047AB49                 nop  
SDPI:0047AB4A                 nop  
SDPI:0047AB4B                 popfw  
SDPI:0047AB4D                 jz      short Pass\_47ABB7         ; 这里不跳就 over 了  
SDPI:0047AB4F  
SDPI:0047AB4F ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047AB4F  
SDPI:0047AB4F  
SDPI:0047AB4F FINDDBG\_47AB4F  proc near  
SDPI:0047AB4F                 nop  
SDPI:0047AB50                 nop  
SDPI:0047AB51                 nop  
SDPI:0047AB52                 nop  
SDPI:0047AB53                 nop  
SDPI:0047AB54                 call    loc\_47AB5A  
SDPI:0047AB59                 nop  
SDPI:0047AB5A  
SDPI:0047AB5A loc\_47AB5A:                                       ; CODE XREF: FINDDBG\_47AB4F+5 p  
SDPI:0047AB5A                 pop     eax  
SDPI:0047AB5B                 add     eax, 5Eh  
SDPI:0047AB60                 mov     edx, eax  
SDPI:0047AB62                 add     edx, 32h  
SDPI:0047AB65                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047AB65                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047AB65                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047AB65                                                   ; 用来解密  
SDPI:0047AB6A                 call    loc\_47AB70  
SDPI:0047AB6F                 nop  
SDPI:0047AB70  
SDPI:0047AB70 loc\_47AB70:                                       ; CODE XREF: FINDDBG\_47AB4F+1B p  
SDPI:0047AB70                 pop     eax  
SDPI:0047AB71                 add     eax, 452Dh  
SDPI:0047AB76                 call    loc\_47AB7C  
SDPI:0047AB7B                 nop  
SDPI:0047AB7C  
SDPI:0047AB7C loc\_47AB7C:                                       ; CODE XREF: FINDDBG\_47AB4F+27 p  
SDPI:0047AB7C                 pop     ecx  
SDPI:0047AB7D                 add     ecx, 45CEh  
SDPI:0047AB83                 push    0  
SDPI:0047AB85                 push    ecx  
SDPI:0047AB86                 push    eax  
SDPI:0047AB87                 push    0  
SDPI:0047AB89                 call    loc\_47AB8F  
SDPI:0047AB8E                 nop  
SDPI:0047AB8F  
SDPI:0047AB8F loc\_47AB8F:                                       ; CODE XREF: FINDDBG\_47AB4F+3A p  
SDPI:0047AB8F                 pop     eax  
SDPI:0047AB90                 add     eax, 11h  
SDPI:0047AB95                 push    eax  
SDPI:0047AB96                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047AB96                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047AB96                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047AB96 ; ----------------------------------------------------------------------------  
SDPI:0047AB9B                 db  90h ; ?  
SDPI:0047AB9C                 db  90h ; ?  
SDPI:0047AB9D                 db  90h ; ?  
SDPI:0047AB9E                 db  90h ; ?  
SDPI:0047AB9F ; ----------------------------------------------------------------------------  
SDPI:0047AB9F                 push    7  
SDPI:0047ABA1                 call    loc\_47ABA7  
SDPI:0047ABA6                 nop  
SDPI:0047ABA7  
SDPI:0047ABA7 loc\_47ABA7:                                       ; CODE XREF: FINDDBG\_47AB4F+52 p  
SDPI:0047ABA7                 pop     eax  
SDPI:0047ABA8                 add     eax, 11h  
SDPI:0047ABAD                 push    eax  
SDPI:0047ABAE                 jmp     ExitProcess  
SDPI:0047ABAE ; ----------------------------------------------------------------------------  
SDPI:0047ABB3                 db  90h ; ?  
SDPI:0047ABB4                 db  90h ; ?  
SDPI:0047ABB5                 db  90h ; ?  
SDPI:0047ABB6                 db  90h ; ?  
SDPI:0047ABB6 FINDDBG\_47AB4F  endp  
SDPI:0047ABB6  
SDPI:0047ABB7 ; ----------------------------------------------------------------------------  
SDPI:0047ABB7  
SDPI:0047ABB7 Pass\_47ABB7:                                      ; CODE XREF: SDPI:0047AB4D j  
SDPI:0047ABB7                 call    loc\_47ABBD  
SDPI:0047ABBC                 nop  
SDPI:0047ABBD  
SDPI:0047ABBD loc\_47ABBD:                                       ; CODE XREF: SDPI:Pass\_47ABB7 p  
SDPI:0047ABBD                 pop     eax  
SDPI:0047ABBE                 add     eax, 11h  
SDPI:0047ABC3                 push    eax  
SDPI:0047ABC4                 jmp     AntiDBG\_482535  
SDPI:0047ABC4 ; ----------------------------------------------------------------------------  
SDPI:0047ABC9                 db  90h ; ?  
SDPI:0047ABCA                 db  90h ; ?  
SDPI:0047ABCB                 db  90h ; ?  
SDPI:0047ABCC                 db  90h ; ?  
SDPI:0047ABCD ; ----------------------------------------------------------------------------  
SDPI:0047ABCD                 cmp     eax, 80000000h  
SDPI:0047ABCD ; ----------------------------------------------------------------------------  
SDPI:0047ABD2                 dd 7EB077Ch                       ; 垃圾代码  
SDPI:0047ABD6                 dd 401000h  
SDPI:0047ABDA                 dd 72F774E8h  
SDPI:0047ABDE aS@sRS@s        db 12h,'s',10h,0,10h,'@',0,'?,3,'胷',1Bh,'s',19h,0,10h,'@',0,'?  
SDPI:0047ABF1 aFP3TS@sxxf     db 'f 淧 3?豻扈',5,0,0,0,0,10h,'@',0,'鑈 Xf'  
SDPI:0047AC07                 db  9Dh ; ?                      ; 上面都是花指令来的，  
SDPI:0047AC07                                                   ; 我直接让 IDA 分析成字符串  
SDPI:0047AC08 ; ----------------------------------------------------------------------------  
SDPI:0047AC08                 jz      short Pass\_47AC72         ; 这里比较跳，不跳 over 了  
SDPI:0047AC0A  
SDPI:0047AC0A ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047AC0A  
SDPI:0047AC0A  
SDPI:0047AC0A FNDDBG\_47AC0A   proc near  
SDPI:0047AC0A                 nop  
SDPI:0047AC0B                 nop  
SDPI:0047AC0C                 nop  
SDPI:0047AC0D                 nop  
SDPI:0047AC0E                 nop  
SDPI:0047AC0F                 call    loc\_47AC15  
SDPI:0047AC14                 nop  
SDPI:0047AC15  
SDPI:0047AC15 loc\_47AC15:                                       ; CODE XREF: FNDDBG\_47AC0A+5 p  
SDPI:0047AC15                 pop     eax  
SDPI:0047AC16                 add     eax, 5Eh  
SDPI:0047AC1B                 mov     edx, eax  
SDPI:0047AC1D                 add     edx, 32h  
SDPI:0047AC20                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047AC20                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047AC20                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047AC20                                                   ; 用来解密  
SDPI:0047AC25                 call    loc\_47AC2B  
SDPI:0047AC2A                 nop  
SDPI:0047AC2B  
SDPI:0047AC2B loc\_47AC2B:                                       ; CODE XREF: FNDDBG\_47AC0A+1B p  
SDPI:0047AC2B                 pop     eax  
SDPI:0047AC2C                 add     eax, 4472h  
SDPI:0047AC31                 call    loc\_47AC37  
SDPI:0047AC36                 nop  
SDPI:0047AC37  
SDPI:0047AC37 loc\_47AC37:                                       ; CODE XREF: FNDDBG\_47AC0A+27 p  
SDPI:0047AC37                 pop     ecx  
SDPI:0047AC38                 add     ecx, 4513h  
SDPI:0047AC3E                 push    0  
SDPI:0047AC40                 push    ecx  
SDPI:0047AC41                 push    eax  
SDPI:0047AC42                 push    0  
SDPI:0047AC44                 call    loc\_47AC4A  
SDPI:0047AC49                 nop  
SDPI:0047AC4A  
SDPI:0047AC4A loc\_47AC4A:                                       ; CODE XREF: FNDDBG\_47AC0A+3A p  
SDPI:0047AC4A                 pop     eax  
SDPI:0047AC4B                 add     eax, 11h  
SDPI:0047AC50                 push    eax  
SDPI:0047AC51                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047AC51                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047AC51                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047AC51 ; ----------------------------------------------------------------------------  
SDPI:0047AC56                 db  90h ; ?  
SDPI:0047AC57                 db  90h ; ?  
SDPI:0047AC58                 db  90h ; ?  
SDPI:0047AC59                 db  90h ; ?  
SDPI:0047AC5A ; ----------------------------------------------------------------------------  
SDPI:0047AC5A  
SDPI:0047AC5A ExitProc:  
SDPI:0047AC5A                 push    7  
SDPI:0047AC5C                 call    loc\_47AC62  
SDPI:0047AC61                 nop  
SDPI:0047AC62  
SDPI:0047AC62 loc\_47AC62:                                       ; CODE XREF: FNDDBG\_47AC0A+52 p  
SDPI:0047AC62                 pop     eax  
SDPI:0047AC63                 add     eax, 11h  
SDPI:0047AC68                 push    eax  
SDPI:0047AC69                 jmp     ExitProcess  
SDPI:0047AC69 ; ----------------------------------------------------------------------------  
SDPI:0047AC6E                 db  90h ; ?  
SDPI:0047AC6F                 db  90h ; ?  
SDPI:0047AC70                 db  90h ; ?  
SDPI:0047AC71                 db  90h ; ?  
SDPI:0047AC71 FNDDBG\_47AC0A   endp  
SDPI:0047AC71  
SDPI:0047AC72 ; ----------------------------------------------------------------------------  
SDPI:0047AC72  
SDPI:0047AC72 Pass\_47AC72:                                      ; CODE XREF: SDPI:0047AC08 j  
SDPI:0047AC72                 call    loc\_47AC78  
SDPI:0047AC77                 nop  
SDPI:0047AC78  
SDPI:0047AC78 loc\_47AC78:                                       ; CODE XREF: SDPI:Pass\_47AC72 p  
SDPI:0047AC78                 pop     eax  
SDPI:0047AC79                 add     eax, 11h  
SDPI:0047AC7E                 push    eax  
SDPI:0047AC7F                 jmp     Check\_Mode                ; 检测是父进程还是子进程  
SDPI:0047AC7F ; ----------------------------------------------------------------------------  
SDPI:0047AC84                 db  90h ; ?  
SDPI:0047AC85                 db  90h ; ?  
SDPI:0047AC86                 db  90h ; ?  
SDPI:0047AC87                 db  90h ; ?  
SDPI:0047AC88 ; ----------------------------------------------------------------------------  
SDPI:0047AC88                 mov     ebx, 80000000h  
SDPI:0047AC8D                 add     ebx, eax  
SDPI:0047AC8F                 xor     eax, 87EAF247h  
SDPI:0047AC94                 sub     eax, 0BC1D12FAh           ; 这里关键了，如果相减为 0 表示是字进程  
SDPI:0047AC94 ; ----------------------------------------------------------------------------  
SDPI:0047AC99 JUNK\_47AC99     db '|',7,'?,7,0,10h,'@',0,'鑤鱮',12h,'s',10h,0,10h,'@',0,'?,3,'?  
SDPI:0047AC99                 db 'r',1Bh,'s',19h,0,10h,'@',0,'鑖淧 3?豻扈',5,0,0,0,0,10h,'@',0  
SDPI:0047AC99                 db '鑈 Xf 漼',7,'x',5,0,10h,'@',0,'?  
SDPI:0047ACD8 ; ----------------------------------------------------------------------------  
SDPI:0047ACD8                 jz      Subroutine\_Mode           ; 跳去子进程部分  
SDPI:0047ACDE                 jnz     short loc\_47ACE5  
SDPI:0047ACDE ; ----------------------------------------------------------------------------  
SDPI:0047ACE0                 dd 401000h  
SDPI:0047ACE4                 db 0E8h ; ?  
SDPI:0047ACE5 ; ----------------------------------------------------------------------------  
SDPI:0047ACE5  
SDPI:0047ACE5 loc\_47ACE5:                                       ; CODE XREF: SDPI:0047ACDE j  
SDPI:0047ACE5                 call    loc\_47ACEB  
SDPI:0047ACEA                 nop  
SDPI:0047ACEB  
SDPI:0047ACEB loc\_47ACEB:                                       ; CODE XREF: SDPI:loc\_47ACE5 p  
SDPI:0047ACEB                 pop     eax  
SDPI:0047ACEC                 add     eax, 11h  
SDPI:0047ACF1                 push    eax  
SDPI:0047ACF2                 jmp     WritTMPF\_481537           ; 写入临时文件  
SDPI:0047ACF2 ; ----------------------------------------------------------------------------  
SDPI:0047ACF7                 db  90h ; ?  
SDPI:0047ACF8                 db  90h ; ?  
SDPI:0047ACF9                 db  90h ; ?  
SDPI:0047ACFA                 db  90h ; ?  
SDPI:0047ACFB ; ----------------------------------------------------------------------------  
SDPI:0047ACFB                 sub     eax, 8  
SDPI:0047ACFB ; ----------------------------------------------------------------------------  
SDPI:0047ACFE Junk\_47ACFE     db '~',7,'',5,0,10h,'@',0,'鑖淨 3 蒰?,5,0,10h,'@',0,'鑉悙悙悙悙?  
SDPI:0047ACFE                 db '悙悙悙悙悙悙悙 f 漸',7,'t',5,0,10h,'@',0  
SDPI:0047AD36                 db 0E8h ; ?  
SDPI:0047AD37 ; ----------------------------------------------------------------------------  
SDPI:0047AD37                 jz      Subroutine\_Mode  
SDPI:0047AD3D                 call    loc\_47AD43  
SDPI:0047AD42                 nop  
SDPI:0047AD43  
SDPI:0047AD43 loc\_47AD43:                                       ; CODE XREF: SDPI:0047AD3D p  
SDPI:0047AD43                 pop     eax  
SDPI:0047AD44                 add     eax, 11h  
SDPI:0047AD49                 push    eax  
SDPI:0047AD4A                 jmp     apiGetCmdLine  
SDPI:0047AD4A ; ----------------------------------------------------------------------------  
SDPI:0047AD4F                 db  90h ; ?  
SDPI:0047AD50                 db  90h ; ?  
SDPI:0047AD51                 db  90h ; ?  
SDPI:0047AD52                 db  90h ; ?  
SDPI:0047AD53 ; ----------------------------------------------------------------------------  
SDPI:0047AD53                 push    eax  
SDPI:0047AD54                 mov     edi, eax  
SDPI:0047AD56                 xor     al, al  
SDPI:0047AD58                 mov     ecx, 0FFFFFFFFh  
SDPI:0047AD5D                 repne scasb  
SDPI:0047AD5F                 neg     ecx  
SDPI:0047AD61                 dec     ecx                       ; 获取命令行长度  
SDPI:0047AD62                 pop     esi  
SDPI:0047AD63                 call    loc\_47AD69  
SDPI:0047AD68                 nop  
SDPI:0047AD69  
SDPI:0047AD69 loc\_47AD69:                                       ; CODE XREF: SDPI:0047AD63 p  
SDPI:0047AD69                 pop     edi  
SDPI:0047AD6A                 add     edi, 0FFFFF4E4h  
SDPI:0047AD70                 rep movsb                         ; 复制命令行  
SDPI:0047AD72                 call    loc\_47AD78  
SDPI:0047AD77                 nop  
SDPI:0047AD78  
SDPI:0047AD78 loc\_47AD78:                                       ; CODE XREF: SDPI:0047AD72 p  
SDPI:0047AD78                 pop     edi  
SDPI:0047AD79                 add     edi, 0FFFFF4D5h  
SDPI:0047AD7F                 call    loc\_47AD85  
SDPI:0047AD84                 nop  
SDPI:0047AD85  
SDPI:0047AD85 loc\_47AD85:                                       ; CODE XREF: SDPI:0047AD7F p  
SDPI:0047AD85                 pop     eax  
SDPI:0047AD86                 add     eax, 0FFFFF5CCh  
SDPI:0047AD8B                 call    loc\_47AD91  
SDPI:0047AD90                 nop  
SDPI:0047AD91  
SDPI:0047AD91 loc\_47AD91:                                       ; CODE XREF: SDPI:0047AD8B p  
SDPI:0047AD91                 pop     ebx  
SDPI:0047AD92                 add     ebx, 0FFFFF4ACh  
SDPI:0047AD98                 push    ebx  
SDPI:0047AD99                 push    eax  
SDPI:0047AD9A                 push    0  
SDPI:0047AD9C                 push    0  
SDPI:0047AD9E                 push    0  
SDPI:0047ADA0                 push    1  
SDPI:0047ADA2                 push    0  
SDPI:0047ADA4                 push    0  
SDPI:0047ADA6                 push    edi  
SDPI:0047ADA7                 push    0  
SDPI:0047ADA9                 call    loc\_47ADAF  
SDPI:0047ADAE                 nop  
SDPI:0047ADAF  
SDPI:0047ADAF loc\_47ADAF:                                       ; CODE XREF: SDPI:0047ADA9 p  
SDPI:0047ADAF                 pop     eax  
SDPI:0047ADB0                 add     eax, 11h  
SDPI:0047ADB5                 push    eax  
SDPI:0047ADB6                 jmp     apiCreateProcess          ; 创建新进程  
SDPI:0047ADB6 ; ----------------------------------------------------------------------------  
SDPI:0047ADBB                 db  90h ; ?  
SDPI:0047ADBC                 db  90h ; ?  
SDPI:0047ADBD                 db  90h ; ?  
SDPI:0047ADBE                 db  90h ; ?  
SDPI:0047ADBF ; ----------------------------------------------------------------------------  
SDPI:0047ADBF                 push    0  
SDPI:0047ADC1                 call    loc\_47ADC7  
SDPI:0047ADC6                 nop  
SDPI:0047ADC7  
SDPI:0047ADC7 loc\_47ADC7:                                       ; CODE XREF: SDPI:0047ADC1 p  
SDPI:0047ADC7                 pop     eax  
SDPI:0047ADC8                 add     eax, 11h  
SDPI:0047ADCD                 push    eax  
SDPI:0047ADCE                 jmp     ExitProcess  
SDPI:0047ADCE ; ----------------------------------------------------------------------------  
SDPI:0047ADD3                 db  90h ; ?  
到这里启动进程就结束了，继续跳去被启动部分。  
SDPI:0047ADD7 ; ----------------------------------------------------------------------------  
SDPI:0047ADD7  
SDPI:0047ADD7 Subroutine\_Mode:                                  ; CODE XREF: SDPI:0047ACD8 j  
SDPI:0047ADD7                                                   ; SDPI:0047AD37 j  
SDPI:0047ADD7                 call    loc\_47ADDD  
SDPI:0047ADDC                 nop  
SDPI:0047ADDD  
SDPI:0047ADDD loc\_47ADDD:                                       ; CODE XREF: SDPI:Subroutine\_Mode p  
SDPI:0047ADDD                 pop     eax  
SDPI:0047ADDE                 add     eax, 11h  
SDPI:0047ADE3                 push    eax  
SDPI:0047ADE4                 jmp     CMPHASH\_48147D            ; 计算并比较 MD5 值,判断代码是否被修改过  
SDPI:0047ADE4 ; ----------------------------------------------------------------------------  
SDPI:0047ADE9 dword\_47ADE9    dd 90909090h  
SDPI:0047ADED aPQ@VVA         db 'p',0Eh,'q',0Ch,0,10h,'@',0,'縑|!v',12h,'€',0Eh  
SDPI:0047ADFD ; ----------------------------------------------------------------------------  
SDPI:0047ADFD                 mov     ecx, 769E3CF2h  
SDPI:0047AE02                 call    loc\_47AE08  
SDPI:0047AE07                 nop  
SDPI:0047AE08  
SDPI:0047AE08 loc\_47AE08:                                       ; CODE XREF: SDPI:0047AE02 p  
SDPI:0047AE08                 pop     eax  
SDPI:0047AE09                 add     eax, 5FEh  
SDPI:0047AE0E                 call    loc\_47AE14  
SDPI:0047AE13                 nop  
SDPI:0047AE14  
SDPI:0047AE14 loc\_47AE14:                                       ; CODE XREF: SDPI:0047AE0E p  
SDPI:0047AE14                 pop     edx  
SDPI:0047AE15                 add     edx, 6ECh  
SDPI:0047AE1B                 call    Crypt\_Code                ; MD5 值加密代码,调用方法:  
SDPI:0047AE1B                                                   ; invoke Crypt\_Code,End,Start  
SDPI:0047AE1B                                                   ; end 加密结束地址,地址在 EDX 中  
SDPI:0047AE1B                                                   ; start 加密起始地址,地址在 eax 中  
SDPI:0047AE1B                                                   ;  
SDPI:0047AE20                 push    eax                       ; 用够阴险，再用计算出来的 md5 值  
SDPI:0047AE20                                                   ; 加密代码,  
SDPI:0047AE20                                                   ; 第二次加密地址为:0047B405  
SDPI:0047AE21                 xor     eax, eax  
SDPI:0047AE23                 call    loc\_47AE29  
SDPI:0047AE28                 nop  
SDPI:0047AE29  
SDPI:0047AE29 loc\_47AE29:                                       ; CODE XREF: SDPI:0047AE23 p  
SDPI:0047AE29                 pop     edi  
SDPI:0047AE2A                 add     edi, 61h  
SDPI:0047AE30                 mov     ebx, \[edi\]  
SDPI:0047AE32                 mov     edx, \[edi+4\]  
SDPI:0047AE32 ; ----------------------------------------------------------------------------  
SDPI:0047AE35 aTU@L           db 't',0Ah  
SDPI:0047AE35                 db 'u',8,0,10h,'@',0,0Bh,'',9  
SDPI:0047AE41 ; ----------------------------------------------------------------------------  
SDPI:0047AE41                 call    loc\_47AE47  
SDPI:0047AE46                 nop  
SDPI:0047AE47  
SDPI:0047AE47 loc\_47AE47:                                       ; CODE XREF: SDPI:0047AE41 p  
SDPI:0047AE47                 pop     esi  
SDPI:0047AE48                 add     esi, 59h  
SDPI:0047AE4E                 mov     ecx, 3  
SDPI:0047AE4E ; ----------------------------------------------------------------------------  
SDPI:0047AE53 Junk\_47AE53     db '|',7,'?,7,0,10h,'@',0,'鑤鱮',12h,'s',10h,0,10h,'@',0,'?,3,'?  
SDPI:0047AE53                 db 'r',1Bh,'s',19h,0,10h,'@',0,'鑖淧 3?豻扈',5,0,0,0,0,10h,'@',0  
SDPI:0047AE53                 db '鑈 Xf?  
SDPI:0047AE89 ; ----------------------------------------------------------------------------  
SDPI:0047AE89                 rep movsw  
SDPI:0047AE8C                 call    FNDDBG\_47AF0D  
SDPI:0047AE91                 call    Int3\_47AF7D               ; 第四处 INT3 了  
SDPI:0047AE91 ; ----------------------------------------------------------------------------  
SDPI:0047AE96 JUNK\_47AE96     db '?,0,10h,'@',0,'皦?,4,'?,1,'愲',3,'悙'  
SDPI:0047AEA6  
SDPI:0047AEA6 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047AEA6  
SDPI:0047AEA6  
SDPI:0047AEA6 FndDBG\_47AEA6   proc near  
SDPI:0047AEA6                 nop  
SDPI:0047AEA7                 nop  
SDPI:0047AEA8                 nop  
SDPI:0047AEA9                 nop  
SDPI:0047AEAA                 call    loc\_47AEB0  
SDPI:0047AEAF                 nop  
SDPI:0047AEB0  
SDPI:0047AEB0 loc\_47AEB0:                                       ; CODE XREF: FndDBG\_47AEA6+4 p  
SDPI:0047AEB0                 pop     eax  
SDPI:0047AEB1                 add     eax, 5Eh  
SDPI:0047AEB6                 mov     edx, eax  
SDPI:0047AEB8                 add     edx, 32h  
SDPI:0047AEBB                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047AEBB                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047AEBB                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047AEBB                                                   ; 用来解密  
SDPI:0047AEC0                 call    loc\_47AEC6  
SDPI:0047AEC5                 nop  
SDPI:0047AEC6  
SDPI:0047AEC6 loc\_47AEC6:                                       ; CODE XREF: FndDBG\_47AEA6+1A p  
SDPI:0047AEC6                 pop     eax  
SDPI:0047AEC7                 add     eax, 41D7h  
SDPI:0047AECC                 call    loc\_47AED2  
SDPI:0047AED1                 nop  
SDPI:0047AED2  
SDPI:0047AED2 loc\_47AED2:                                       ; CODE XREF: FndDBG\_47AEA6+26 p  
SDPI:0047AED2                 pop     ecx  
SDPI:0047AED3                 add     ecx, 4278h  
SDPI:0047AED9                 push    0  
SDPI:0047AEDB                 push    ecx  
SDPI:0047AEDC                 push    eax  
SDPI:0047AEDD                 push    0  
SDPI:0047AEDF                 call    loc\_47AEE5  
SDPI:0047AEE4                 nop  
SDPI:0047AEE5  
SDPI:0047AEE5 loc\_47AEE5:                                       ; CODE XREF: FndDBG\_47AEA6+39 p  
SDPI:0047AEE5                 pop     eax  
SDPI:0047AEE6                 add     eax, 11h  
SDPI:0047AEEB                 push    eax  
SDPI:0047AEEC                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047AEEC                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047AEEC                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047AEEC ; ----------------------------------------------------------------------------  
SDPI:0047AEF1                 db  90h ; ?  
SDPI:0047AEF2                 db  90h ; ?  
SDPI:0047AEF3                 db  90h ; ?  
SDPI:0047AEF4                 db  90h ; ?  
SDPI:0047AEF5 ; ----------------------------------------------------------------------------  
SDPI:0047AEF5                 push    7  
SDPI:0047AEF7                 call    loc\_47AEFD  
SDPI:0047AEFC                 nop  
SDPI:0047AEFD  
SDPI:0047AEFD loc\_47AEFD:                                       ; CODE XREF: FndDBG\_47AEA6+51 p  
SDPI:0047AEFD                 pop     eax  
SDPI:0047AEFE                 add     eax, 11h  
SDPI:0047AF03                 push    eax  
SDPI:0047AF04                 jmp     ExitProcess  
SDPI:0047AF04 ; ----------------------------------------------------------------------------  
SDPI:0047AF09                 db  90h ; ?  
SDPI:0047AF0A                 db  90h ; ?  
SDPI:0047AF0B                 db  90h ; ?  
SDPI:0047AF0C                 db  90h ; ?  
SDPI:0047AF0C FndDBG\_47AEA6   endp  
SDPI:0047AF0C  
SDPI:0047AF0D  
SDPI:0047AF0D ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047AF0D  
SDPI:0047AF0D  
SDPI:0047AF0D FNDDBG\_47AF0D   proc near                         ; CODE XREF: SDPI:0047AE8C p  
SDPI:0047AF0D                 nop  
SDPI:0047AF0E                 nop  
SDPI:0047AF0F                 nop  
SDPI:0047AF10                 nop  
SDPI:0047AF11                 nop  
SDPI:0047AF12                 call    loc\_47AF18  
SDPI:0047AF17                 nop  
SDPI:0047AF18  
SDPI:0047AF18 loc\_47AF18:                                       ; CODE XREF: FNDDBG\_47AF0D+5 p  
SDPI:0047AF18                 pop     eax  
SDPI:0047AF19                 add     eax, 5Eh  
SDPI:0047AF1E                 mov     edx, eax  
SDPI:0047AF20                 add     edx, 32h  
SDPI:0047AF23                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047AF23                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047AF23                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047AF23                                                   ; 用来解密  
SDPI:0047AF28                 call    loc\_47AF2E  
SDPI:0047AF2D                 nop  
SDPI:0047AF2E  
SDPI:0047AF2E loc\_47AF2E:                                       ; CODE XREF: FNDDBG\_47AF0D+1B p  
SDPI:0047AF2E                 pop     eax  
SDPI:0047AF2F                 add     eax, 416Fh  
SDPI:0047AF34                 call    loc\_47AF3A  
SDPI:0047AF39                 nop  
SDPI:0047AF3A  
SDPI:0047AF3A loc\_47AF3A:                                       ; CODE XREF: FNDDBG\_47AF0D+27 p  
SDPI:0047AF3A                 pop     ecx  
SDPI:0047AF3B                 add     ecx, 4210h  
SDPI:0047AF41                 push    0  
SDPI:0047AF43                 push    ecx  
SDPI:0047AF44                 push    eax  
SDPI:0047AF45                 push    0  
SDPI:0047AF47                 call    loc\_47AF4D  
SDPI:0047AF4C                 nop  
SDPI:0047AF4D  
SDPI:0047AF4D loc\_47AF4D:                                       ; CODE XREF: FNDDBG\_47AF0D+3A p  
SDPI:0047AF4D                 pop     eax  
SDPI:0047AF4E                 add     eax, 11h  
SDPI:0047AF53                 push    eax  
SDPI:0047AF54                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047AF54                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047AF54                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047AF54 ; ----------------------------------------------------------------------------  
SDPI:0047AF59                 db  90h ; ?  
SDPI:0047AF5A                 db  90h ; ?  
SDPI:0047AF5B                 db  90h ; ?  
SDPI:0047AF5C                 db  90h ; ?  
SDPI:0047AF5D ; ----------------------------------------------------------------------------  
SDPI:0047AF5D                 push    7  
SDPI:0047AF5F                 call    loc\_47AF65  
SDPI:0047AF64                 nop  
SDPI:0047AF65  
SDPI:0047AF65 loc\_47AF65:                                       ; CODE XREF: FNDDBG\_47AF0D+52 p  
SDPI:0047AF65                 pop     eax  
SDPI:0047AF66                 add     eax, 11h  
SDPI:0047AF6B                 push    eax  
SDPI:0047AF6C                 jmp     ExitProcess  
SDPI:0047AF6C ; ----------------------------------------------------------------------------  
SDPI:0047AF71 JUNK\_47AF71     db '悙悙',0,10h,'@',0,'鄩?,6  
SDPI:0047AF71 FNDDBG\_47AF0D   endp  
SDPI:0047AF71  
SDPI:0047AF7D ; ----------------------------------------------------------------------------  
SDPI:0047AF7D  
SDPI:0047AF7D Int3\_47AF7D:                                      ; CODE XREF: SDPI:0047AE91 p  
SDPI:0047AF7D                 call    loc\_47AF83                ; 第四处 INT3 了  
SDPI:0047AF82                 nop  
SDPI:0047AF83  
SDPI:0047AF83 loc\_47AF83:                                       ; CODE XREF: SDPI:Int3\_47AF7D p  
SDPI:0047AF83                 pop     edi  
SDPI:0047AF84                 add     edi, 0FFFFFF07h  
SDPI:0047AF8A                 mov     \[edi\], ebx  
SDPI:0047AF8C                 mov     \[edi+4\], edx  
SDPI:0047AF8F                 pop     eax  
SDPI:0047AF90                 call    loc\_47AF96  
SDPI:0047AF95                 nop  
SDPI:0047AF96  
SDPI:0047AF96 loc\_47AF96:                                       ; CODE XREF: SDPI:0047AF90 p  
SDPI:0047AF96                 pop     eax  
SDPI:0047AF97                 add     eax, 124h  
SDPI:0047AF9C                 push    eax  
SDPI:0047AF9D                 xor     eax, eax  
SDPI:0047AF9F                 push    dword ptr fs:\[eax\]  
SDPI:0047AFA2                 mov     fs:\[eax\], esp  
SDPI:0047AFA5                 mov     ebp, 300EF1D3h  
SDPI:0047AFAA                 add     ebp, 12345678h  
SDPI:0047AFB0                 mov     ax, 17h  
SDPI:0047AFB4                 sub     ax, 13h  
SDPI:0047AFB4 ; ----------------------------------------------------------------------------  
SDPI:0047AFB8 JUNK\_47AFB8     db '|',7,'?,7,0,10h,'@',0,'鑤鱮',12h,'s',10h,0,10h,'@',0,'?,3,'?  
SDPI:0047AFB8                 db 'r',1Bh,'s',19h,0,10h,'@',0,'鑖淧 3?豻扈',5,0,0,0,0,10h,'@',0  
SDPI:0047AFB8                 db '鑈 Xf?  
SDPI:0047AFEE ; ----------------------------------------------------------------------------  
SDPI:0047AFEE                 nop  
SDPI:0047AFEF                 nop  
SDPI:0047AFF0                 nop  
SDPI:0047AFF1                 nop                               ; 第四次 INT3 异常后 SEH 句柄在  
SDPI:0047AFF1                                                   ; 0047B0B9 处  
SDPI:0047AFF2                 nop  
SDPI:0047AFF3                 int     3                         ; Trap to Debugger  
SDPI:0047AFF4                 nop  
SDPI:0047AFF5                 cmp     al, 4  
SDPI:0047AFF7                 jz      short Pass\_47B06A         ; 通过 int3 异常则跳  
SDPI:0047AFF9  
SDPI:0047AFF9 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047AFF9  
SDPI:0047AFF9 ; 如果发现调试器则和前面一样显示错误信息  
SDPI:0047AFF9  
SDPI:0047AFF9 FNDDBG\_47AFF9   proc near                         ; CODE XREF: SDPI:0047B083 j  
SDPI:0047AFF9                                                   ; SDPI:0047B09B j ...  
SDPI:0047AFF9                 nop  
SDPI:0047AFFA                 nop  
SDPI:0047AFFB                 nop  
SDPI:0047AFFC                 nop  
SDPI:0047AFFD                 nop  
SDPI:0047AFFE                 call    loc\_47B004  
SDPI:0047B003                 nop  
SDPI:0047B004  
SDPI:0047B004 loc\_47B004:                                       ; CODE XREF: FNDDBG\_47AFF9+5 p  
SDPI:0047B004                 pop     eax  
SDPI:0047B005                 add     eax, 5Eh  
SDPI:0047B00A                 mov     edx, eax  
SDPI:0047B00C                 add     edx, 32h  
SDPI:0047B00F                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047B00F                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047B00F                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047B00F                                                   ; 用来解密  
SDPI:0047B014                 call    loc\_47B01A  
SDPI:0047B019                 nop  
SDPI:0047B01A  
SDPI:0047B01A loc\_47B01A:                                       ; CODE XREF: FNDDBG\_47AFF9+1B p  
SDPI:0047B01A                 pop     eax  
SDPI:0047B01B                 add     eax, 4083h  
SDPI:0047B020                 call    loc\_47B026  
SDPI:0047B025                 nop  
SDPI:0047B026  
SDPI:0047B026 loc\_47B026:                                       ; CODE XREF: FNDDBG\_47AFF9+27 p  
SDPI:0047B026                 pop     ecx  
SDPI:0047B027                 add     ecx, 4124h  
SDPI:0047B02D                 push    0  
SDPI:0047B02F                 push    ecx  
SDPI:0047B030                 push    eax  
SDPI:0047B031                 push    0  
SDPI:0047B033                 call    loc\_47B039  
SDPI:0047B038                 nop  
SDPI:0047B039  
SDPI:0047B039 loc\_47B039:                                       ; CODE XREF: FNDDBG\_47AFF9+3A p  
SDPI:0047B039                 pop     eax  
SDPI:0047B03A                 add     eax, 11h  
SDPI:0047B03F                 push    eax  
SDPI:0047B040                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047B040                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047B040                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047B040 ; ----------------------------------------------------------------------------  
SDPI:0047B045                 db  90h ; ?  
SDPI:0047B046                 db  90h ; ?  
SDPI:0047B047                 db  90h ; ?  
SDPI:0047B048                 db  90h ; ?  
SDPI:0047B049 ; ----------------------------------------------------------------------------  
SDPI:0047B049                 push    7  
SDPI:0047B04B                 call    loc\_47B051  
SDPI:0047B050                 nop  
SDPI:0047B051  
SDPI:0047B051 loc\_47B051:                                       ; CODE XREF: FNDDBG\_47AFF9+52 p  
SDPI:0047B051                 pop     eax  
SDPI:0047B052                 add     eax, 11h  
SDPI:0047B057                 push    eax  
SDPI:0047B058                 jmp     ExitProcess  
SDPI:0047B058 ; ----------------------------------------------------------------------------  
SDPI:0047B05D JUNK\_47B05D     db '悙悙?,0,10h,'@',0,'皦?,4  
SDPI:0047B05D FNDDBG\_47AFF9   endp  
SDPI:0047B05D  
SDPI:0047B06A ; ----------------------------------------------------------------------------  
SDPI:0047B06A  
SDPI:0047B06A Pass\_47B06A:                                      ; CODE XREF: SDPI:0047AFF7 j  
SDPI:0047B06A                 pop     large dword ptr fs:0  
SDPI:0047B071                 add     esp, 4  
SDPI:0047B074                 call    loc\_47B07A  
SDPI:0047B079                 nop  
SDPI:0047B07A  
SDPI:0047B07A loc\_47B07A:                                       ; CODE XREF: SDPI:0047B074 p  
SDPI:0047B07A                 pop     eax  
SDPI:0047B07B                 add     eax, 0FFFFFE1Dh  
SDPI:0047B080                 cmp     byte ptr \[eax\], 0E9h  
SDPI:0047B083                 jnz     FNDDBG\_47AFF9             ; 如果发现调试器则和前面一样显示错误信息  
SDPI:0047B089                 mov     byte ptr \[eax\], 0E8h  
SDPI:0047B08C                 rdtsc  
SDPI:0047B08E                 mov     ecx, eax  
SDPI:0047B090                 mov     ebx, edx  
SDPI:0047B092                 rdtsc  
SDPI:0047B094                 sub     eax, ecx  
SDPI:0047B096                 sbb     edx, ebx  
SDPI:0047B098                 cmp     edx, 0                    ; 又是时间检测  
SDPI:0047B09B                 jnz     FNDDBG\_47AFF9             ; 如果发现调试器则和前面一样显示错误信息  
SDPI:0047B0A1                 cmp     eax, 30000000h  
SDPI:0047B0A6                 ja      FNDDBG\_47AFF9             ; 如果发现调试器则和前面一样显示错误信息  
SDPI:0047B0AC                 jz      short pass\_47B0F7  
SDPI:0047B0AE                 jnz     short pass\_47B0F7  
SDPI:0047B0AE ; ----------------------------------------------------------------------------  
SDPI:0047B0B0 JUNK\_47B0B0     db '?,0,10h,'@',0,'皦?,4  
SDPI:0047B0B9 ; ----------------------------------------------------------------------------  
SDPI:0047B0B9                 mov     eax, \[esp+4\]              ; 第四处 INT3 异常的处理句柄  
SDPI:0047B0BD                 mov     ecx, \[esp+0Ch\]  
SDPI:0047B0C1                 inc     dword ptr \[ecx+0B8h\]      ; REG\[EIP\]+1  
SDPI:0047B0C7                 mov     eax, \[eax\]  
SDPI:0047B0C9                 sub     eax, EXCEPTION\_BREAKPOINT ; 判断是否为异常中断  
SDPI:0047B0CE                 jnz     short locret\_47B0F6  
SDPI:0047B0D0                 call    loc\_47B0D6  
SDPI:0047B0D5                 nop  
SDPI:0047B0D6  
SDPI:0047B0D6 loc\_47B0D6:                                       ; CODE XREF: SDPI:0047B0D0 p  
SDPI:0047B0D6                 pop     eax  
SDPI:0047B0D7                 add     eax, 0FFFFFDC1h  
SDPI:0047B0DC                 cmp     byte ptr \[eax\], 0E8h      ; 这里和前面一样判断是否为 0E8,  
SDPI:0047B0DC                                                   ; 是否被修改过  
SDPI:0047B0DF                 jnz     FNDDBG\_47AFF9             ; 如果发现调试器则和前面一样显示错误信息  
SDPI:0047B0E5                 mov     byte ptr \[eax\], 0E9h  
SDPI:0047B0E8                 xor     eax, eax  
SDPI:0047B0EA                 mov     \[ecx+4\], eax              ; 清除硬件断点  
SDPI:0047B0ED                 mov     \[ecx+8\], eax  
SDPI:0047B0F0                 mov     \[ecx+0Ch\], eax  
SDPI:0047B0F3                 mov     \[ecx+10h\], eax  
SDPI:0047B0F6  
SDPI:0047B0F6 locret\_47B0F6:                                    ; CODE XREF: SDPI:0047B0CE j  
SDPI:0047B0F6                 retn  
SDPI:0047B0F7 ; ----------------------------------------------------------------------------  
SDPI:0047B0F7  
SDPI:0047B0F7 pass\_47B0F7:                                      ; CODE XREF: SDPI:0047B0AC j  
SDPI:0047B0F7                                                   ; SDPI:0047B0AE j  
SDPI:0047B0F7                 pop     eax  
SDPI:0047B0F8                 call    CallGetTickCount          ; 注意一点的话就会发现和前  
SDPI:0047B0F8                                                   ; 一个 INT3 的代码基本上是一样的  
SDPI:0047B0F8                                                   ; 作者很喜欢作宏?  
SDPI:0047B0F8 ; ----------------------------------------------------------------------------  
SDPI:0047B0FD a@V             db 0,10h,'@',0,'綱',1  
SDPI:0047B105 ; ----------------------------------------------------------------------------  
SDPI:0047B105  
SDPI:0047B105 loc\_47B105:                                       ; CODE XREF: SDPI:0047B303 p  
SDPI:0047B105                 pop     ebp  
SDPI:0047B106                 pop     eax  
SDPI:0047B107                 jmp     near ptr unk\_47B308  
SDPI:0047B10C ; ----------------------------------------------------------------------------  
SDPI:0047B10C                 mov     ecx, 0FFFFFF00h  
SDPI:0047B111                 push    fs  
SDPI:0047B111 ; ----------------------------------------------------------------------------  
SDPI:0047B113 aTU@I           db 't',0Ah  
SDPI:0047B113                 db 'u',8,0,10h,'@',0,'皦?,4  
SDPI:0047B11F ; ----------------------------------------------------------------------------  
SDPI:0047B11F                 pushfw  
SDPI:0047B121                 push    eax  
SDPI:0047B122                 mov     eax, ebx  
SDPI:0047B124                 push    ebx  
SDPI:0047B125                 mov     eax, ecx  
SDPI:0047B127                 push    eax  
SDPI:0047B128                 add     eax, edx  
SDPI:0047B12A                 mov     ebx, eax  
SDPI:0047B12C                 push    ebx  
SDPI:0047B12D                 pop     eax  
SDPI:0047B12E                 push    edx  
SDPI:0047B12F                 call    loc\_47B13C  
SDPI:0047B12F ; ----------------------------------------------------------------------------  
SDPI:0047B134                 dd 401000h  
SDPI:0047B138                 dd 132BD7B0h  
SDPI:0047B13C ; ----------------------------------------------------------------------------  
SDPI:0047B13C  
SDPI:0047B13C loc\_47B13C:                                       ; CODE XREF: SDPI:0047B12F p  
SDPI:0047B13C                 pop     eax  
SDPI:0047B13D                 call    loc\_47B143  
SDPI:0047B142                 nop  
SDPI:0047B143  
SDPI:0047B143 loc\_47B143:                                       ; CODE XREF: SDPI:0047B13D p  
SDPI:0047B143                 pop     eax  
SDPI:0047B144                 add     eax, 11h  
SDPI:0047B149                 push    eax  
SDPI:0047B14A                 jmp     GetTickCount  
SDPI:0047B14A ; ----------------------------------------------------------------------------  
SDPI:0047B14F                 db  90h ; ?  
SDPI:0047B150                 db  90h ; ?  
SDPI:0047B151                 db  90h ; ?  
SDPI:0047B152                 db  90h ; ?  
SDPI:0047B153 ; ----------------------------------------------------------------------------  
SDPI:0047B153                 push    eax  
SDPI:0047B154                 mov     eax, edx  
SDPI:0047B156                 push    eax  
SDPI:0047B157                 call    loc\_47B15D  
SDPI:0047B15C                 nop  
SDPI:0047B15D  
SDPI:0047B15D loc\_47B15D:                                       ; CODE XREF: SDPI:0047B157 p  
SDPI:0047B15D                 pop     edx  
SDPI:0047B15E                 add     edx, 52h  
SDPI:0047B164                 push    edx  
SDPI:0047B165                 add     edx, 402165h  
SDPI:0047B16B                 push    edx  
SDPI:0047B16C                 jo      short loc\_47B1C1  
SDPI:0047B16E                 jno     short loc\_47B1C1  
SDPI:0047B170  
SDPI:0047B170 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047B170  
SDPI:0047B170  
SDPI:0047B170 sub\_47B170      proc near                         ; CODE XREF: SDPI:0047B1B4 p  
SDPI:0047B170                 pop     eax  
SDPI:0047B171                 pop     ebx  
SDPI:0047B172                 call    sub\_47B178  
SDPI:0047B177                 nop  
SDPI:0047B177 sub\_47B170      endp  
SDPI:0047B177  
SDPI:0047B178  
SDPI:0047B178 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047B178  
SDPI:0047B178  
SDPI:0047B178 sub\_47B178      proc near                         ; CODE XREF: sub\_47B170+2 p  
SDPI:0047B178                 pop     eax  
SDPI:0047B179                 add     eax, 11h  
SDPI:0047B17E                 push    eax  
SDPI:0047B17F                 jmp     GetTickCount  
SDPI:0047B17F sub\_47B178      endp  
SDPI:0047B17F  
SDPI:0047B17F ; ----------------------------------------------------------------------------  
SDPI:0047B184                 db  90h ; ?  
SDPI:0047B185                 db  90h ; ?  
SDPI:0047B186                 db  90h ; ?  
SDPI:0047B187                 db  90h ; ?  
SDPI:0047B188 ; ----------------------------------------------------------------------------  
SDPI:0047B188                 pop     ebx  
SDPI:0047B189                 add     ebx, 1F4h  
SDPI:0047B18F                 sub     ebx, eax  
SDPI:0047B191                 js      short OVER\_47B1D5         ; 同前一个是一样的,不能跳  
SDPI:0047B193                 call    loc\_47B199  
SDPI:0047B198                 nop  
SDPI:0047B199  
SDPI:0047B199 loc\_47B199:                                       ; CODE XREF: SDPI:0047B193 p  
SDPI:0047B199                 pop     ebx  
SDPI:0047B19A                 add     ebx, 0A5h  
SDPI:0047B1A0                 push    ebx  
SDPI:0047B1A1                 call    loc\_47B1CB  
SDPI:0047B1A1 ; ----------------------------------------------------------------------------  
SDPI:0047B1A6                 dd 401000h  
SDPI:0047B1AA                 dd 58C88B0h  
SDPI:0047B1AE ; ----------------------------------------------------------------------------  
SDPI:0047B1AE                 pop     eax  
SDPI:0047B1AF                 mov     edx, eax  
SDPI:0047B1B1                 mov     eax, ebx  
SDPI:0047B1B3                 push    eax  
SDPI:0047B1B4                 call    sub\_47B170  
SDPI:0047B1B4 ; ----------------------------------------------------------------------------  
SDPI:0047B1B9                 dd 401000h  
SDPI:0047B1BD                 dd 1833639h  
SDPI:0047B1C1 ; ----------------------------------------------------------------------------  
SDPI:0047B1C1  
SDPI:0047B1C1 loc\_47B1C1:                                       ; CODE XREF: SDPI:0047B16C j  
SDPI:0047B1C1                                                   ; SDPI:0047B16E j  
SDPI:0047B1C1                 pop     eax  
SDPI:0047B1C2                 retn  
SDPI:0047B1C2 ; ----------------------------------------------------------------------------  
SDPI:0047B1C3 JUNK\_47B1C3     db 0,10h,'@',0,'>V|',7  
SDPI:0047B1CB ; ----------------------------------------------------------------------------  
SDPI:0047B1CB  
SDPI:0047B1CB loc\_47B1CB:                                       ; CODE XREF: SDPI:0047B1A1 p  
SDPI:0047B1CB                 pop     edx  
SDPI:0047B1CC                 retn  
SDPI:0047B1CC ; ----------------------------------------------------------------------------  
SDPI:0047B1CD JUNK\_47B1cD     db 0,10h,'@',0,'颯?,1  
SDPI:0047B1D5  
SDPI:0047B1D5 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047B1D5  
SDPI:0047B1D5  
SDPI:0047B1D5 OVER\_47B1D5     proc near                         ; CODE XREF: SDPI:0047B191 j  
SDPI:0047B1D5                 nop  
SDPI:0047B1D6                 nop  
SDPI:0047B1D7                 nop  
SDPI:0047B1D8                 nop  
SDPI:0047B1D9                 nop  
SDPI:0047B1DA                 call    loc\_47B1E0  
SDPI:0047B1DF                 nop  
SDPI:0047B1E0  
SDPI:0047B1E0 loc\_47B1E0:                                       ; CODE XREF: OVER\_47B1D5+5 p  
SDPI:0047B1E0                 pop     eax  
SDPI:0047B1E1                 add     eax, 5Eh  
SDPI:0047B1E6                 mov     edx, eax  
SDPI:0047B1E8                 add     edx, 32h  
SDPI:0047B1EB                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047B1EB                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047B1EB                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047B1EB                                                   ; 用来解密  
SDPI:0047B1F0                 call    loc\_47B1F6  
SDPI:0047B1F5                 nop  
SDPI:0047B1F6  
SDPI:0047B1F6 loc\_47B1F6:                                       ; CODE XREF: OVER\_47B1D5+1B p  
SDPI:0047B1F6                 pop     eax  
SDPI:0047B1F7                 add     eax, 3EA7h  
SDPI:0047B1FC                 call    loc\_47B202  
SDPI:0047B201                 nop  
SDPI:0047B202  
SDPI:0047B202 loc\_47B202:                                       ; CODE XREF: OVER\_47B1D5+27 p  
SDPI:0047B202                 pop     ecx  
SDPI:0047B203                 add     ecx, 3F48h  
SDPI:0047B209                 push    0  
SDPI:0047B20B                 push    ecx  
SDPI:0047B20C                 push    eax  
SDPI:0047B20D                 push    0  
SDPI:0047B20F                 call    loc\_47B215  
SDPI:0047B214                 nop  
SDPI:0047B215  
SDPI:0047B215 loc\_47B215:                                       ; CODE XREF: OVER\_47B1D5+3A p  
SDPI:0047B215                 pop     eax  
SDPI:0047B216                 add     eax, 11h  
SDPI:0047B21B                 push    eax  
SDPI:0047B21C                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047B21C                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047B21C                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047B21C ; ----------------------------------------------------------------------------  
SDPI:0047B221                 db  90h ; ?  
SDPI:0047B222                 db  90h ; ?  
SDPI:0047B223                 db  90h ; ?  
SDPI:0047B224                 db  90h ; ?  
SDPI:0047B225 ; ----------------------------------------------------------------------------  
SDPI:0047B225                 push    7  
SDPI:0047B227                 call    loc\_47B22D  
SDPI:0047B22C                 nop  
SDPI:0047B22D  
SDPI:0047B22D loc\_47B22D:                                       ; CODE XREF: OVER\_47B1D5+52 p  
SDPI:0047B22D                 pop     eax  
SDPI:0047B22E                 add     eax, 11h  
SDPI:0047B233                 push    eax  
SDPI:0047B234                 jmp     ExitProcess  
SDPI:0047B234 ; ----------------------------------------------------------------------------  
SDPI:0047B239                 db  90h ; ?  
SDPI:0047B23A                 db  90h ; ?  
SDPI:0047B23B                 db  90h ; ?  
SDPI:0047B23C                 db  90h ; ?  
SDPI:0047B23C OVER\_47B1D5     endp  
SDPI:0047B23C  
SDPI:0047B23D ; ----------------------------------------------------------------------------  
SDPI:0047B23D                 pop     edx  
SDPI:0047B23E                 mov     eax, ecx  
SDPI:0047B240                 add     eax, edx  
SDPI:0047B242                 inc     ecx  
SDPI:0047B243                 push    eax                       ; 和上一个 INT3 差不多，我也就不多讲了  
SDPI:0047B244                 inc     ecx  
SDPI:0047B245                 pop     ebx  
SDPI:0047B246                 pop     ecx  
SDPI:0047B247                 push    eax  
SDPI:0047B248                 sub     eax, 8  
SDPI:0047B24B                 pop     ebx  
SDPI:0047B24C                 pop     ebx  
SDPI:0047B24D                 inc     eax  
SDPI:0047B24E                 add     eax, ebx  
SDPI:0047B250                 pop     eax  
SDPI:0047B251                 pushfw  
SDPI:0047B253                 popfw  
SDPI:0047B255                 popfw  
SDPI:0047B257                 pop     es  
SDPI:0047B258                 mov     eax, 12345678h  
SDPI:0047B25D                 push    eax  
SDPI:0047B25E                 call    loc\_47B264  
SDPI:0047B263                 nop  
SDPI:0047B264  
SDPI:0047B264 loc\_47B264:                                       ; CODE XREF: SDPI:0047B25E p  
SDPI:0047B264                 pop     eax  
SDPI:0047B265                 add     eax, 12Ch  
SDPI:0047B26A                 push    eax  
SDPI:0047B26B                 pop     ebx  
SDPI:0047B26C                 add     eax, 12h  
SDPI:0047B26F                 pop     edx  
SDPI:0047B270                 add     eax, edx  
SDPI:0047B272                 mov     edx, eax  
SDPI:0047B274                 push    ebx  
SDPI:0047B275                 mov     ebx, es:\[ecx+100h\]  
SDPI:0047B27C                 push    ebx  
SDPI:0047B27D                 mov     eax, esp  
SDPI:0047B27F                 mov     ebx, eax  
SDPI:0047B281                 push    ebx  
SDPI:0047B282                 pop     edx  
SDPI:0047B283                 mov     es:\[ecx+100h\], eax  
SDPI:0047B28A                 xor     eax, eax  
SDPI:0047B28A ; ----------------------------------------------------------------------------  
SDPI:0047B28C JUNK\_47B28C     db '~',7,'',5,0,10h,'@',0,'鑖淨 3 蒰?,5,0,10h,'@',0,'鑉悙悙悙悙?  
SDPI:0047B28C                 db '悙悙悙悙悙悙悙 f 漰',6  
SDPI:0047B2BE aQ@             db 'q',4,0,10h,'@',0              ; 迎接第五个 int3 了  
SDPI:0047B2C4 ; ----------------------------------------------------------------------------  
SDPI:0047B2C4                 int     3                         ; Trap to Debugger  
SDPI:0047B2C5                 nop                               ; INT3 SEH 句柄在 0047B38F 处  
SDPI:0047B2C6                 xor     eax, eax  
SDPI:0047B2C8                 mov     dword ptr \[eax\], 4023C8h  
SDPI:0047B2C8 ; ----------------------------------------------------------------------------  
SDPI:0047B2CE JUNK\_47B2CE     db 'z',0Ah  
SDPI:0047B2CE                 db '{',8,0,10h,'@',0,'{=@',0  
SDPI:0047B2DA ; ----------------------------------------------------------------------------  
SDPI:0047B2DA  
SDPI:0047B2DA CallGetTickCount:                                 ; CODE XREF: SDPI:0047B0F8 p  
SDPI:0047B2DA                 call    loc\_47B2E0  
SDPI:0047B2DF                 nop  
SDPI:0047B2E0  
SDPI:0047B2E0 loc\_47B2E0:                                       ; CODE XREF: SDPI:CallGetTickCount p  
SDPI:0047B2E0                 pop     eax  
SDPI:0047B2E1                 add     eax, 11h  
SDPI:0047B2E6                 push    eax  
SDPI:0047B2E7                 jmp     GetTickCount  
SDPI:0047B2E7 ; ----------------------------------------------------------------------------  
SDPI:0047B2EC                 db  90h ; ?  
SDPI:0047B2ED                 db  90h ; ?  
SDPI:0047B2EE                 db  90h ; ?  
SDPI:0047B2EF                 db  90h ; ?  
SDPI:0047B2F0 ; ----------------------------------------------------------------------------  
SDPI:0047B2F0                 call    loc\_47B2F6  
SDPI:0047B2F5                 nop  
SDPI:0047B2F6  
SDPI:0047B2F6 loc\_47B2F6:                                       ; CODE XREF: SDPI:0047B2F0 p  
SDPI:0047B2F6                 pop     edx  
SDPI:0047B2F7                 add     edx, 0FFFFFB04h  
SDPI:0047B2FD                 mov     \[edx\], eax  
SDPI:0047B2FF                 pop     ebp  
SDPI:0047B300                 add     eax, edx  
SDPI:0047B302                 push    eax  
SDPI:0047B303                 call    loc\_47B105  
SDPI:0047B303 ; ----------------------------------------------------------------------------  
SDPI:0047B308 unk\_47B308      db 0E8h ; ?                      ; CODE XREF: SDPI:0047B107 j  
SDPI:0047B309                 db    1  
SDPI:0047B30A                 db    0  
SDPI:0047B30B                 db    0  
SDPI:0047B30C ; ----------------------------------------------------------------------------  
SDPI:0047B30C                 add     \[eax-3D7EA6h\], dl  
SDPI:0047B312                 std  
SDPI:0047B312 ; ----------------------------------------------------------------------------  
SDPI:0047B313                 db 0FFh  
SDPI:0047B314                 db 0FFh  
SDPI:0047B315                 db    3  
SDPI:0047B316                 db 0D0h ; ?  
SDPI:0047B317                 db  52h ; R  
SDPI:0047B318                 db  59h ; Y  
SDPI:0047B319                 db  2Bh ; +  
SDPI:0047B31A                 db 0C8h ; ?  
SDPI:0047B31B                 db  51h ; Q  
SDPI:0047B31C                 db 0C2h ; ?  
SDPI:0047B31D                 db    4  
SDPI:0047B31E                 db    0  
SDPI:0047B31F  
SDPI:0047B31F ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047B31F  
SDPI:0047B31F  
SDPI:0047B31F OVER\_47B31F     proc near                         ; CODE XREF: SDPI:0047B3C2 j  
SDPI:0047B31F                                                   ; SDPI:0047B3CA j ...  
SDPI:0047B31F                 nop  
SDPI:0047B320                 nop  
SDPI:0047B321                 nop  
SDPI:0047B322                 nop  
SDPI:0047B323                 nop  
SDPI:0047B324                 call    loc\_47B32A  
SDPI:0047B329                 nop  
SDPI:0047B32A  
SDPI:0047B32A loc\_47B32A:                                       ; CODE XREF: OVER\_47B31F+5 p  
SDPI:0047B32A                 pop     eax  
SDPI:0047B32B                 add     eax, 5Eh  
SDPI:0047B330                 mov     edx, eax  
SDPI:0047B332                 add     edx, 32h  
SDPI:0047B335                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047B335                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047B335                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047B335                                                   ; 用来解密  
SDPI:0047B33A                 call    loc\_47B340  
SDPI:0047B33F                 nop  
SDPI:0047B340  
SDPI:0047B340 loc\_47B340:                                       ; CODE XREF: OVER\_47B31F+1B p  
SDPI:0047B340                 pop     eax  
SDPI:0047B341                 add     eax, 3D5Dh  
SDPI:0047B346                 call    loc\_47B34C  
SDPI:0047B34B                 nop  
SDPI:0047B34C  
SDPI:0047B34C loc\_47B34C:                                       ; CODE XREF: OVER\_47B31F+27 p  
SDPI:0047B34C                 pop     ecx  
SDPI:0047B34D                 add     ecx, 3DFEh  
SDPI:0047B353                 push    0  
SDPI:0047B355                 push    ecx  
SDPI:0047B356                 push    eax  
SDPI:0047B357                 push    0  
SDPI:0047B359                 call    loc\_47B35F  
SDPI:0047B35E                 nop  
SDPI:0047B35F  
SDPI:0047B35F loc\_47B35F:                                       ; CODE XREF: OVER\_47B31F+3A p  
SDPI:0047B35F                 pop     eax  
SDPI:0047B360                 add     eax, 11h  
SDPI:0047B365                 push    eax  
SDPI:0047B366                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047B366                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047B366                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047B366 ; ----------------------------------------------------------------------------  
SDPI:0047B36B                 db  90h ; ?  
SDPI:0047B36C                 db  90h ; ?  
SDPI:0047B36D                 db  90h ; ?  
SDPI:0047B36E                 db  90h ; ?  
SDPI:0047B36F ; ----------------------------------------------------------------------------  
SDPI:0047B36F                 push    7  
SDPI:0047B371                 call    loc\_47B377  
SDPI:0047B376                 nop  
SDPI:0047B377  
SDPI:0047B377 loc\_47B377:                                       ; CODE XREF: OVER\_47B31F+52 p  
SDPI:0047B377                 pop     eax  
SDPI:0047B378                 add     eax, 11h  
SDPI:0047B37D                 push    eax  
SDPI:0047B37E                 jmp     ExitProcess  
SDPI:0047B37E ; ----------------------------------------------------------------------------  
SDPI:0047B383 aRrrr@V         db '悙悙',0,10h,'@',0,'綱',1  
SDPI:0047B383 OVER\_47B31F     endp  
SDPI:0047B383  
SDPI:0047B38F ; ----------------------------------------------------------------------------  
SDPI:0047B38F                 mov     esp, \[esp+8\]              ; 第五处 int3 的处理句柄  
SDPI:0047B393                 pop     large dword ptr fs:0  
SDPI:0047B39A                 call    loc\_47B3A0  
SDPI:0047B39F                 nop  
SDPI:0047B3A0  
SDPI:0047B3A0 loc\_47B3A0:                                       ; CODE XREF: SDPI:0047B39A p  
SDPI:0047B3A0                 pop     eax  
SDPI:0047B3A1                 add     eax, 11h  
SDPI:0047B3A6                 push    eax  
SDPI:0047B3A7                 jmp     GetTickCount  
SDPI:0047B3A7 ; ----------------------------------------------------------------------------  
SDPI:0047B3AC                 db  90h ; ?  
SDPI:0047B3AD                 db  90h ; ?  
SDPI:0047B3AE                 db  90h ; ?  
SDPI:0047B3AF                 db  90h ; ?  
SDPI:0047B3B0 ; ----------------------------------------------------------------------------  
SDPI:0047B3B0                 call    loc\_47B3B6  
SDPI:0047B3B5                 nop  
SDPI:0047B3B6  
SDPI:0047B3B6 loc\_47B3B6:                                       ; CODE XREF: SDPI:0047B3B0 p  
SDPI:0047B3B6                 pop     edx  
SDPI:0047B3B7                 add     edx, 0FFFFFA44h  
SDPI:0047B3BD                 mov     ecx, \[edx\]  
SDPI:0047B3BF                 cmp     ecx, 0  
SDPI:0047B3C2                 jz      OVER\_47B31F  
SDPI:0047B3C8                 sub     eax, ecx  
SDPI:0047B3CA                 js      OVER\_47B31F  
SDPI:0047B3D0                 sub     eax, 7D0h  
SDPI:0047B3D5                 jns     OVER\_47B31F               ; 没有特殊的了,宏了上面的代码而已  
SDPI:0047B3DB                 mov     eax, 0E801276h  
SDPI:0047B3E0                 mov     \[edx\], eax  
SDPI:0047B3E2                 call    loc\_47B3E8  
SDPI:0047B3E7                 nop  
SDPI:0047B3E8  
SDPI:0047B3E8 loc\_47B3E8:                                       ; CODE XREF: SDPI:0047B3E2 p  
SDPI:0047B3E8                 pop     edx  
SDPI:0047B3E9                 add     edx, 118h  
SDPI:0047B3EF                 call    loc\_47B3F5  
SDPI:0047B3F4                 nop  
SDPI:0047B3F5  
SDPI:0047B3F5 loc\_47B3F5:                                       ; CODE XREF: SDPI:0047B3EF p  
SDPI:0047B3F5                 pop     eax  
SDPI:0047B3F6                 add     eax, 0FFFFED5Ch  
SDPI:0047B3FB                 mov     ecx, 10h  
SDPI:0047B400                 call    De\_Code                   ; invoke De\_Code,ecx(counter),eax(key),edx(end)  
SDPI:0047B400                                                   ; 用于解密代码，解密起始地址就是 call 进来  
SDPI:0047B400                                                   ; 的下一行代码地址  
SDPI:0047B400 ; ----------------------------------------------------------------------------  
SDPI:0047B405 JUNK\_47B405     db '~',7,'',5,0,10h,'@',0,'鑖淨 3 蒰?,5,0,10h,'@',0,'鑉悙悙悙悙?  
SDPI:0047B405                 db '悙悙悙悙悙悙悙 f?  
SDPI:0047B435 ; ----------------------------------------------------------------------------  
SDPI:0047B435                 call    loc\_47B43B  
SDPI:0047B43A                 nop  
SDPI:0047B43B  
SDPI:0047B43B loc\_47B43B:                                       ; CODE XREF: SDPI:0047B435 p  
SDPI:0047B43B                 pop     eax  
SDPI:0047B43C                 add     eax, 11h  
SDPI:0047B441                 push    eax  
SDPI:0047B442                 jmp     GetStart\_Info             ; 获取程序启动的相关信息：  
SDPI:0047B442                                                   ; 如程序句柄，系统目录  
SDPI:0047B442                                                   ; windows 目录，  
SDPI:0047B442                                                   ; 程序完整程序等  
SDPI:0047B442 ; ----------------------------------------------------------------------------  
SDPI:0047B447                 db  90h ; ?  
SDPI:0047B448                 db  90h ; ?  
SDPI:0047B449                 db  90h ; ?  
SDPI:0047B44A                 db  90h ; ?  
SDPI:0047B44B ; ----------------------------------------------------------------------------  
SDPI:0047B44B                 call    loc\_47B451  
SDPI:0047B450                 nop  
SDPI:0047B451  
SDPI:0047B451 loc\_47B451:                                       ; CODE XREF: SDPI:0047B44B p  
SDPI:0047B451                 pop     eax  
SDPI:0047B452                 add     eax, 11h  
SDPI:0047B457                 push    eax  
SDPI:0047B458                 jmp     CMP\_HASH\_481275           ; 进去又是 MD5 检测  
SDPI:0047B458 ; ----------------------------------------------------------------------------  
SDPI:0047B45D                 db  90h ; ?  
SDPI:0047B45E                 db  90h ; ?  
SDPI:0047B45F                 db  90h ; ?  
SDPI:0047B460                 db  90h ; ?  
SDPI:0047B461 ; ----------------------------------------------------------------------------  
SDPI:0047B461                 cmp     eax, 0FE5F3AFEh  
SDPI:0047B461 ; ----------------------------------------------------------------------------  
SDPI:0047B466 JUNK\_47B466     db '|',7,'?,7,0,10h,'@',0,'鑤鱮',12h,'s',10h,0,10h,'@',0,'?,3,'?  
SDPI:0047B466                 db 'r',1Bh,'s',19h,0,10h,'@',0,'鑖淧 3?豻扈',5,0,0,0,0,10h,'@',0  
SDPI:0047B466                 db '鑈 Xf?  
SDPI:0047B49C ; ----------------------------------------------------------------------------  
SDPI:0047B49C                 jz      OVER\_47B5E7  
SDPI:0047B4A2                 jnz     short loc\_47B4A9  
SDPI:0047B4A2 ; ----------------------------------------------------------------------------  
SDPI:0047B4A4                 dd 401000h  
SDPI:0047B4A8                 db 0E8h ; ?  
SDPI:0047B4A9 ; ----------------------------------------------------------------------------  
SDPI:0047B4A9  
SDPI:0047B4A9 loc\_47B4A9:                                       ; CODE XREF: SDPI:0047B4A2 j  
SDPI:0047B4A9                 call    loc\_47B4AF  
SDPI:0047B4AE                 nop  
SDPI:0047B4AF  
SDPI:0047B4AF loc\_47B4AF:                                       ; CODE XREF: SDPI:loc\_47B4A9 p  
SDPI:0047B4AF                 pop     eax  
SDPI:0047B4B0                 add     eax, 11h  
SDPI:0047B4B5                 push    eax  
SDPI:0047B4B6                 jmp     IsDebuggerPresent         ; 这里应该存在问题，  
SDPI:0047B4B6                                                   ; 如果是 Win9x 这里还能检测到吗??  
SDPI:0047B4B6 ; ----------------------------------------------------------------------------  
SDPI:0047B4BB                 db  90h ; ?  
SDPI:0047B4BC                 db  90h ; ?  
SDPI:0047B4BD                 db  90h ; ?  
SDPI:0047B4BE                 db  90h ; ?  
SDPI:0047B4BF ; ----------------------------------------------------------------------------  
SDPI:0047B4BF                 cmp     eax, 0  
SDPI:0047B4BF ; ----------------------------------------------------------------------------  
SDPI:0047B4C2 JUNK\_47B4C2     db '~',7,'',5,0,10h,'@',0,'鑖淨 3 蒰?,5,0,10h,'@',0,'鑉悙悙悙悙?  
SDPI:0047B4C2                 db '悙悙悙悙悙悙悙 f?  
SDPI:0047B4F2 ; ----------------------------------------------------------------------------  
SDPI:0047B4F2                 jnz     OVER\_47B5E7  
SDPI:0047B4F8                 jz      short loc\_47B4FF  
SDPI:0047B4F8 ; ----------------------------------------------------------------------------  
SDPI:0047B4FA                 dd 401000h  
SDPI:0047B4FE                 db 0E8h ; ?  
SDPI:0047B4FF ; ----------------------------------------------------------------------------  
SDPI:0047B4FF  
SDPI:0047B4FF loc\_47B4FF:                                       ; CODE XREF: SDPI:0047B4F8 j  
SDPI:0047B4FF                 call    loc\_47B505  
SDPI:0047B504                 nop  
SDPI:0047B505  
SDPI:0047B505 loc\_47B505:                                       ; CODE XREF: SDPI:loc\_47B4FF p  
SDPI:0047B505                 pop     eax  
SDPI:0047B506                 add     eax, 0FFFFED3Ch  
SDPI:0047B50B                 call    loc\_47B511  
SDPI:0047B510                 nop  
SDPI:0047B511  
SDPI:0047B511 loc\_47B511:                                       ; CODE XREF: SDPI:0047B50B p  
SDPI:0047B511                 pop     ebx  
SDPI:0047B512                 add     ebx, 0FFFFEC7Ch  
SDPI:0047B518                 mov     ecx, \[eax\]  
SDPI:0047B51A                 mov     \[ebx\], ecx  
SDPI:0047B51C                 call    loc\_47B522  
SDPI:0047B521                 nop  
SDPI:0047B522  
SDPI:0047B522 loc\_47B522:                                       ; CODE XREF: SDPI:0047B51C p  
SDPI:0047B522                 pop     eax  
SDPI:0047B523                 add     eax, 0FFFFED23h  
SDPI:0047B528                 call    loc\_47B52E  
SDPI:0047B52D                 nop  
SDPI:0047B52E  
SDPI:0047B52E loc\_47B52E:                                       ; CODE XREF: SDPI:0047B528 p  
SDPI:0047B52E                 pop     ebx  
SDPI:0047B52F                 add     ebx, 0FFFFEC63h  
SDPI:0047B535                 mov     ecx, \[eax\]  
SDPI:0047B537                 mov     \[ebx\], ecx  
SDPI:0047B539                 call    loc\_47B53F  
SDPI:0047B53E                 nop  
SDPI:0047B53F  
SDPI:0047B53F loc\_47B53F:                                       ; CODE XREF: SDPI:0047B539 p  
SDPI:0047B53F                 pop     eax  
SDPI:0047B540                 add     eax, 11h  
SDPI:0047B545                 push    eax  
SDPI:0047B546                 jmp     CreateThread2             ; 创建两个新线程  
SDPI:0047B546                                                   ; 线程地址分别为:  
SDPI:0047B546                                                   ; 00482100  
SDPI:0047B546                                                   ; 00482269  
SDPI:0047B546                                                   ; 所幸的是如果是双 CPU 就不会运行，  
SDPI:0047B546                                                   ; 这也说明那两个新线程肯定不是  
SDPI:0047B546                                                   ; 什么好东西，也不重要的东西  
SDPI:0047B546 ; ----------------------------------------------------------------------------  
SDPI:0047B54B                 db  90h ; ?  
SDPI:0047B54C                 db  90h ; ?  
SDPI:0047B54D                 db  90h ; ?  
SDPI:0047B54E                 db  90h ; ?  
SDPI:0047B54F ; ----------------------------------------------------------------------------  
SDPI:0047B54F                 mov     ecx, 10h  
SDPI:0047B554                 call    loc\_47B55A  
SDPI:0047B559                 nop  
SDPI:0047B55A  
SDPI:0047B55A loc\_47B55A:                                       ; CODE XREF: SDPI:0047B554 p  
SDPI:0047B55A                 pop     eax  
SDPI:0047B55B                 add     eax, 0FFFFECE3h  
SDPI:0047B560                 call    FillZero\_47F375  
SDPI:0047B565                 mov     ecx, 104h  
SDPI:0047B56A                 call    loc\_47B570  
SDPI:0047B56F                 nop  
SDPI:0047B570  
SDPI:0047B570 loc\_47B570:                                       ; CODE XREF: SDPI:0047B56A p  
SDPI:0047B570                 pop     eax  
SDPI:0047B571                 add     eax, 0FFFFECDDh  
SDPI:0047B576                 call    FillZero\_47F375  
SDPI:0047B57B                 mov     ecx, 64h  
SDPI:0047B580                 call    loc\_47B586  
SDPI:0047B585                 nop  
SDPI:0047B586  
SDPI:0047B586 loc\_47B586:                                       ; CODE XREF: SDPI:0047B580 p  
SDPI:0047B586                 pop     eax  
SDPI:0047B587                 add     eax, 0FFFFEDCBh  
SDPI:0047B58C                 call    FillZero\_47F375  
SDPI:0047B591                 call    loc\_47B597  
SDPI:0047B596                 nop  
SDPI:0047B597  
SDPI:0047B597 loc\_47B597:                                       ; CODE XREF: SDPI:0047B591 p  
SDPI:0047B597                 pop     edx  
SDPI:0047B598                 add     edx, 0FFFFECA2h  
SDPI:0047B59E                 mov     ebx, \[edx\]                ; \[EDX\]=DS:\[0047A238\]=E821C800  
SDPI:0047B59E                                                   ; EBX=E821C800  
SDPI:0047B5A0                 cmp     ebx, 0E821C800h           ; 这里是标志，不相等则 over  
SDPI:0047B5A6                 jnz     short OVER\_47B5E7  
SDPI:0047B5A8                 call    loc\_47B5AE  
SDPI:0047B5AD                 nop  
SDPI:0047B5AE  
SDPI:0047B5AE loc\_47B5AE:                                       ; CODE XREF: SDPI:0047B5A8 p  
SDPI:0047B5AE                 pop     eax  
SDPI:0047B5AF                 add     eax, 0FFFFEA57h  
SDPI:0047B5B4                 mov     ecx, \[eax\]  
SDPI:0047B5B6                 cmp     ecx, 0E8673219h  
SDPI:0047B5BC                 jz      Pass\_47B64F  
SDPI:0047B5C2  
SDPI:0047B5C2 OVer\_47B5C2:  
SDPI:0047B5C2                 call    loc\_47B5C8  
SDPI:0047B5C7                 nop  
SDPI:0047B5C8  
SDPI:0047B5C8 loc\_47B5C8:                                       ; CODE XREF: SDPI:OVer\_47B5C2 p  
SDPI:0047B5C8                 pop     eax  
SDPI:0047B5C9                 add     eax, 11h  
SDPI:0047B5CE                 push    eax  
SDPI:0047B5CF                 jmp     Get\_Version  
SDPI:0047B5CF ; ----------------------------------------------------------------------------  
SDPI:0047B5D4                 db  90h ; ?  
SDPI:0047B5D5                 db  90h ; ?  
SDPI:0047B5D6                 db  90h ; ?  
SDPI:0047B5D7                 db  90h ; ?  
SDPI:0047B5D8 ; ----------------------------------------------------------------------------  
SDPI:0047B5D8                 call    loc\_47B5DE  
SDPI:0047B5DD                 nop  
SDPI:0047B5DE  
SDPI:0047B5DE loc\_47B5DE:                                       ; CODE XREF: SDPI:0047B5D8 p  
SDPI:0047B5DE                 pop     edx  
SDPI:0047B5DF                 add     edx, 0FFFFEC5Bh  
SDPI:0047B5E5                 mov     \[edx\], eax  
SDPI:0047B5E7  
SDPI:0047B5E7 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047B5E7  
SDPI:0047B5E7  
SDPI:0047B5E7 OVER\_47B5E7     proc near                         ; CODE XREF: SDPI:0047B49C j  
SDPI:0047B5E7                                                   ; SDPI:0047B4F2 j ...  
SDPI:0047B5E7                 nop  
SDPI:0047B5E8                 nop  
SDPI:0047B5E9                 nop  
SDPI:0047B5EA                 nop  
SDPI:0047B5EB                 nop  
SDPI:0047B5EC                 call    loc\_47B5F2  
SDPI:0047B5F1                 nop  
SDPI:0047B5F2  
SDPI:0047B5F2 loc\_47B5F2:                                       ; CODE XREF: OVER\_47B5E7+5 p  
SDPI:0047B5F2                 pop     eax  
SDPI:0047B5F3                 add     eax, 5Eh  
SDPI:0047B5F8                 mov     edx, eax  
SDPI:0047B5FA                 add     edx, 32h  
SDPI:0047B5FD                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047B5FD                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047B5FD                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047B5FD                                                   ; 用来解密  
SDPI:0047B602                 call    loc\_47B608  
SDPI:0047B607                 nop  
SDPI:0047B608  
SDPI:0047B608 loc\_47B608:                                       ; CODE XREF: OVER\_47B5E7+1B p  
SDPI:0047B608                 pop     eax  
SDPI:0047B609                 add     eax, 3A95h  
SDPI:0047B60E                 call    loc\_47B614  
SDPI:0047B613                 nop  
SDPI:0047B614  
SDPI:0047B614 loc\_47B614:                                       ; CODE XREF: OVER\_47B5E7+27 p  
SDPI:0047B614                 pop     ecx  
SDPI:0047B615                 add     ecx, 3B36h  
SDPI:0047B61B                 push    0  
SDPI:0047B61D                 push    ecx  
SDPI:0047B61E                 push    eax  
SDPI:0047B61F                 push    0  
SDPI:0047B621                 call    loc\_47B627  
SDPI:0047B626                 nop  
SDPI:0047B627  
SDPI:0047B627 loc\_47B627:                                       ; CODE XREF: OVER\_47B5E7+3A p  
SDPI:0047B627                 pop     eax  
SDPI:0047B628                 add     eax, 11h  
SDPI:0047B62D                 push    eax  
SDPI:0047B62E                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047B62E                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047B62E                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047B62E ; ----------------------------------------------------------------------------  
SDPI:0047B633                 db  90h ; ?  
SDPI:0047B634                 db  90h ; ?  
SDPI:0047B635                 db  90h ; ?  
SDPI:0047B636                 db  90h ; ?  
SDPI:0047B637 ; ----------------------------------------------------------------------------  
SDPI:0047B637                 push    7  
SDPI:0047B639                 call    loc\_47B63F  
SDPI:0047B63E                 nop  
SDPI:0047B63F  
SDPI:0047B63F loc\_47B63F:                                       ; CODE XREF: OVER\_47B5E7+52 p  
SDPI:0047B63F                 pop     eax  
SDPI:0047B640                 add     eax, 11h  
SDPI:0047B645                 push    eax  
SDPI:0047B646                 jmp     ExitProcess  
SDPI:0047B646 ; ----------------------------------------------------------------------------  
SDPI:0047B64B                 db  90h ; ?  
SDPI:0047B64C                 db  90h ; ?  
SDPI:0047B64D                 db  90h ; ?  
SDPI:0047B64E                 db  90h ; ?  
SDPI:0047B64E OVER\_47B5E7     endp  
SDPI:0047B64E  
SDPI:0047B64F ; ----------------------------------------------------------------------------  
SDPI:0047B64F  
SDPI:0047B64F Pass\_47B64F:                                      ; CODE XREF: SDPI:0047B5BC j  
SDPI:0047B64F                 call    loc\_47B655  
SDPI:0047B654                 nop  
SDPI:0047B655  
SDPI:0047B655 loc\_47B655:                                       ; CODE XREF: SDPI:Pass\_47B64F p  
SDPI:0047B655                 pop     eax  
SDPI:0047B656                 add     eax, 3Ch                  ; 加密起始地址 47B690  
SDPI:0047B65B                 call    loc\_47B661  
SDPI:0047B660                 nop  
SDPI:0047B661  
SDPI:0047B661 loc\_47B661:                                       ; CODE XREF: SDPI:0047B65B p  
SDPI:0047B661                 pop     edx  
SDPI:0047B662                 add     edx, 1D11h                ; 加密结束地址:0047D371  
SDPI:0047B668                 call    Crypt\_Decrypt\_CODE        ; 由于后面的 DE\_CODE 要解密，  
SDPI:0047B668                                                   ; 而代码已经解出来了，  
SDPI:0047B668                                                   ; 因此这里再用一个加密加回去  
SDPI:0047B66D                 call    loc\_47B673  
SDPI:0047B672                 nop  
SDPI:0047B673  
SDPI:0047B673 loc\_47B673:                                       ; CODE XREF: SDPI:0047B66D p  
SDPI:0047B673                 pop     edx  
SDPI:0047B674                 add     edx, 1CFFh  
SDPI:0047B67A                 call    loc\_47B680  
SDPI:0047B67F                 nop  
SDPI:0047B680  
SDPI:0047B680 loc\_47B680:                                       ; CODE XREF: SDPI:0047B67A p  
SDPI:0047B680                 pop     eax  
SDPI:0047B681                 add     eax, 0FFFFEB19h  
SDPI:0047B686                 mov     ecx, 10h  
SDPI:0047B68B  
SDPI:0047B68B loc\_47B68B:                                       ; invoke De\_Code,ecx(counter),eax(key),edx(end)  
SDPI:0047B68B                 call    De\_Code                   ; 用于解密代码，解密起始地址就是 call 进来  
SDPI:0047B68B                                                   ; 的下一行代码地址  
SDPI:0047B690                 jo      short loc\_47B6A0  
SDPI:0047B692                 jno     short loc\_47B6A0  
SDPI:0047B692 ; ----------------------------------------------------------------------------  
SDPI:0047B694 JUNK\_47B694     db 0,10h,'@',0,'縑|!v',12h,'€',0Eh  
SDPI:0047B6A0 ; ----------------------------------------------------------------------------  
SDPI:0047B6A0  
SDPI:0047B6A0 loc\_47B6A0:                                       ; CODE XREF: SDPI:0047B690 j  
SDPI:0047B6A0                                                   ; SDPI:0047B692 j  
SDPI:0047B6A0                 mov     ecx, 769E3CF2h  
SDPI:0047B6A5                 call    loc\_47B6AB  
SDPI:0047B6AA                 nop  
SDPI:0047B6AB  
SDPI:0047B6AB loc\_47B6AB:                                       ; CODE XREF: SDPI:0047B6A5 p  
SDPI:0047B6AB                 pop     eax  
SDPI:0047B6AC                 add     eax, 5FEh                 ; 加密起始地址:0047BCA8  
SDPI:0047B6B1                 call    loc\_47B6B7  
SDPI:0047B6B6                 nop  
SDPI:0047B6B7  
SDPI:0047B6B7 loc\_47B6B7:                                       ; CODE XREF: SDPI:0047B6B1 p  
SDPI:0047B6B7                 pop     edx  
SDPI:0047B6B8                 add     edx, 0E8Eh                ; 加密结束地址,再次用 MD5 值把  
SDPI:0047B6B8                                                   ; 已解密的代码加密回去  
SDPI:0047B6B8                                                   ; 如果被修改过 MD5 值就肯定不同  
SDPI:0047B6B8                                                   ; 那解密出来的也肯定是无效代码  
SDPI:0047B6BE                 call    Crypt\_Code                ; 第三次加密地址为:0047BCA8  
SDPI:0047B6C3                 push    eax  
SDPI:0047B6C4                 xor     eax, eax  
SDPI:0047B6C6                 call    loc\_47B6CC  
SDPI:0047B6CB                 nop  
SDPI:0047B6CC  
SDPI:0047B6CC loc\_47B6CC:                                       ; CODE XREF: SDPI:0047B6C6 p  
SDPI:0047B6CC                 pop     edi  
SDPI:0047B6CD                 add     edi, 61h  
SDPI:0047B6D3                 mov     ebx, \[edi\]  
SDPI:0047B6D5                 mov     edx, \[edi+4\]  
SDPI:0047B6D8                 jz      short loc\_47B6E4  
SDPI:0047B6DA                 jnz     short loc\_47B6E4  
SDPI:0047B6DA ; ----------------------------------------------------------------------------  
SDPI:0047B6DC                 dd 401000h  
SDPI:0047B6E0                 dd 9F7AB0Bh  
SDPI:0047B6E4 ; ----------------------------------------------------------------------------  
SDPI:0047B6E4  
SDPI:0047B6E4 loc\_47B6E4:                                       ; CODE XREF: SDPI:0047B6D8 j  
SDPI:0047B6E4                                                   ; SDPI:0047B6DA j  
SDPI:0047B6E4                 call    loc\_47B6EA  
SDPI:0047B6E9                 nop  
SDPI:0047B6EA  
SDPI:0047B6EA loc\_47B6EA:                                       ; CODE XREF: SDPI:loc\_47B6E4 p  
SDPI:0047B6EA                 pop     esi  
SDPI:0047B6EB                 add     esi, 59h  
SDPI:0047B6F1                 mov     ecx, 3  
SDPI:0047B6F1 ; ----------------------------------------------------------------------------  
SDPI:0047B6F6 JUNK\_47B6F6     db '|',7,'?,7,0,10h,'@',0,'鑤鱮',12h,'s',10h,0,10h,'@',0,'?,3,'?  
SDPI:0047B6F6                 db 'r',1Bh,'s',19h,0,10h,'@',0,'鑖淧 3?豻扈',5,0,0,0,0,10h,'@',0  
SDPI:0047B6F6                 db '鑈 Xf?  
SDPI:0047B72C ; ----------------------------------------------------------------------------  
SDPI:0047B72C                 rep movsw  
SDPI:0047B72F                 call    Local\_47B7B0  
SDPI:0047B734                 call    loc\_47B820  
SDPI:0047B739                 call    near ptr 87C73Eh  
SDPI:0047B73E                 mov     al, 89h  
SDPI:0047B740                 pushf  
SDPI:0047B741                 add     al, 0EBh  
SDPI:0047B743                 add     \[eax-6F6FFC15h\], edx  
SDPI:0047B749  
SDPI:0047B749 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047B749  
SDPI:0047B749  
SDPI:0047B749 FNDDBG\_47B749   proc near  
SDPI:0047B749                 nop  
SDPI:0047B74A                 nop  
SDPI:0047B74B                 nop  
SDPI:0047B74C                 nop  
SDPI:0047B74D                 call    loc\_47B753  
SDPI:0047B752                 nop  
SDPI:0047B753  
SDPI:0047B753 loc\_47B753:                                       ; CODE XREF: FNDDBG\_47B749+4 p  
SDPI:0047B753                 pop     eax  
SDPI:0047B754                 add     eax, 5Eh  
SDPI:0047B759                 mov     edx, eax  
SDPI:0047B75B                 add     edx, 32h  
SDPI:0047B75E                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047B75E                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047B75E                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047B75E                                                   ; 用来解密  
SDPI:0047B763                 call    loc\_47B769  
SDPI:0047B768                 nop  
SDPI:0047B769  
SDPI:0047B769 loc\_47B769:                                       ; CODE XREF: FNDDBG\_47B749+1A p  
SDPI:0047B769                 pop     eax  
SDPI:0047B76A                 add     eax, 3934h  
SDPI:0047B76F                 call    loc\_47B775  
SDPI:0047B774                 nop  
SDPI:0047B775  
SDPI:0047B775 loc\_47B775:                                       ; CODE XREF: FNDDBG\_47B749+26 p  
SDPI:0047B775                 pop     ecx  
SDPI:0047B776                 add     ecx, 39D5h  
SDPI:0047B77C                 push    0  
SDPI:0047B77E                 push    ecx  
SDPI:0047B77F                 push    eax  
SDPI:0047B780                 push    0  
SDPI:0047B782                 call    loc\_47B788  
SDPI:0047B787                 nop  
SDPI:0047B788  
SDPI:0047B788 loc\_47B788:                                       ; CODE XREF: FNDDBG\_47B749+39 p  
SDPI:0047B788                 pop     eax  
SDPI:0047B789                 add     eax, 11h  
SDPI:0047B78E                 push    eax  
SDPI:0047B78F                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047B78F                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047B78F                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047B78F ; ----------------------------------------------------------------------------  
SDPI:0047B794                 db  90h ; ?  
SDPI:0047B795                 db  90h ; ?  
SDPI:0047B796                 db  90h ; ?  
SDPI:0047B797                 db  90h ; ?  
SDPI:0047B798 ; ----------------------------------------------------------------------------  
SDPI:0047B798                 push    7  
SDPI:0047B79A                 call    loc\_47B7A0  
SDPI:0047B79F                 nop  
SDPI:0047B7A0  
SDPI:0047B7A0 loc\_47B7A0:                                       ; CODE XREF: FNDDBG\_47B749+51 p  
SDPI:0047B7A0                 pop     eax  
SDPI:0047B7A1                 add     eax, 11h  
SDPI:0047B7A6                 push    eax  
SDPI:0047B7A7                 jmp     ExitProcess  
SDPI:0047B7A7 ; ----------------------------------------------------------------------------  
SDPI:0047B7AC                 db  90h ; ?  
SDPI:0047B7AD                 db  90h ; ?  
SDPI:0047B7AE                 db  90h ; ?  
SDPI:0047B7AF                 db  90h ; ?  
SDPI:0047B7AF FNDDBG\_47B749   endp  
SDPI:0047B7AF  
SDPI:0047B7B0 ; ----------------------------------------------------------------------------  
SDPI:0047B7B0  
SDPI:0047B7B0 Local\_47B7B0:                                     ; CODE XREF: SDPI:0047B72F p  
SDPI:0047B7B0                 nop  
SDPI:0047B7B1                 nop  
SDPI:0047B7B2                 nop  
SDPI:0047B7B3                 nop  
SDPI:0047B7B4                 nop  
SDPI:0047B7B5                 call    loc\_47B7BB  
SDPI:0047B7BA                 nop  
SDPI:0047B7BB  
SDPI:0047B7BB loc\_47B7BB:                                       ; CODE XREF: SDPI:0047B7B5 p  
SDPI:0047B7BB                 pop     eax  
SDPI:0047B7BC                 add     eax, 5Eh  
SDPI:0047B7C1                 mov     edx, eax  
SDPI:0047B7C3                 add     edx, 32h  
SDPI:0047B7C6                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047B7C6                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047B7C6                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047B7C6                                                   ; 用来解密  
SDPI:0047B7CB                 call    loc\_47B7D1  
SDPI:0047B7D0                 nop  
SDPI:0047B7D1  
SDPI:0047B7D1 loc\_47B7D1:                                       ; CODE XREF: SDPI:0047B7CB p  
SDPI:0047B7D1                 pop     eax  
SDPI:0047B7D2                 add     eax, 38CCh  
SDPI:0047B7D7                 call    loc\_47B7DD  
SDPI:0047B7DC                 nop  
SDPI:0047B7DD  
SDPI:0047B7DD loc\_47B7DD:                                       ; CODE XREF: SDPI:0047B7D7 p  
SDPI:0047B7DD                 pop     ecx  
SDPI:0047B7DE                 add     ecx, 396Dh  
SDPI:0047B7E4                 push    0  
SDPI:0047B7E6                 push    ecx  
SDPI:0047B7E7                 push    eax  
SDPI:0047B7E8                 push    0  
SDPI:0047B7EA                 call    loc\_47B7F0  
SDPI:0047B7EF                 nop  
SDPI:0047B7F0  
SDPI:0047B7F0 loc\_47B7F0:                                       ; CODE XREF: SDPI:0047B7EA p  
SDPI:0047B7F0                 pop     eax  
SDPI:0047B7F1                 add     eax, 11h  
SDPI:0047B7F6                 push    eax  
SDPI:0047B7F7                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047B7F7                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047B7F7                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047B7F7 ; ----------------------------------------------------------------------------  
SDPI:0047B7FC                 dd 90909090h  
SDPI:0047B800 ; ----------------------------------------------------------------------------  
SDPI:0047B800                 push    7  
SDPI:0047B802                 call    loc\_47B808  
SDPI:0047B807                 nop  
SDPI:0047B808  
SDPI:0047B808 loc\_47B808:                                       ; CODE XREF: SDPI:0047B802 p  
SDPI:0047B808                 pop     eax  
SDPI:0047B809                 add     eax, 11h  
SDPI:0047B80E                 push    eax  
SDPI:0047B80F                 jmp     ExitProcess  
SDPI:0047B80F ; ----------------------------------------------------------------------------  
SDPI:0047B814                 dd 90909090h  
SDPI:0047B818                 dd 401000h  
SDPI:0047B81C                 dd 69C89E0h  
SDPI:0047B820 ; ----------------------------------------------------------------------------  
SDPI:0047B820  
SDPI:0047B820 loc\_47B820:                                       ; CODE XREF: SDPI:0047B734 p  
SDPI:0047B820                 call    loc\_47B826  
SDPI:0047B825                 nop  
SDPI:0047B826  
SDPI:0047B826 loc\_47B826:                                       ; CODE XREF: SDPI:loc\_47B820 p  
SDPI:0047B826                 pop     edi  
SDPI:0047B827                 add     edi, 0FFFFFF07h  
SDPI:0047B82D                 mov     \[edi\], ebx  
SDPI:0047B82F                 mov     \[edi+4\], edx  
SDPI:0047B832                 pop     eax  
SDPI:0047B833                 call    loc\_47B839  
SDPI:0047B838                 nop  
SDPI:0047B839  
SDPI:0047B839 loc\_47B839:                                       ; CODE XREF: SDPI:0047B833 p  
SDPI:0047B839                 pop     eax  
SDPI:0047B83A                 add     eax, 124h  
SDPI:0047B83F                 push    eax  
SDPI:0047B840                 xor     eax, eax  
SDPI:0047B842                 push    dword ptr fs:\[eax\]  
SDPI:0047B845                 mov     fs:\[eax\], esp  
SDPI:0047B848                 mov     ebp, 300EF1D3h  
SDPI:0047B84D                 add     ebp, 12345678h  
SDPI:0047B853                 mov     ax, 17h  
SDPI:0047B857                 sub     ax, 13h  
SDPI:0047B857 ; ----------------------------------------------------------------------------  
SDPI:0047B85B aI@stRS@sRS@sfP3TS@s db '|',7,'?,7,0,10h,'@',0,'鑤鱮',12h,'s',10h,0,10h,'@',0,'?,3,'?  
SDPI:0047B85B                 db 'r',1Bh,'s',19h,0,10h,'@',0,'鑖淧 3?豻扈',5,0,0,0,0,10h,'@',0  
SDPI:0047B85B                 db '鑈 Xf?  
SDPI:0047B891                 dd 90909090h  
SDPI:0047B895 ; ----------------------------------------------------------------------------  
SDPI:0047B895                 nop  
SDPI:0047B896                 int     3                         ; Trap to Debugger  
SDPI:0047B897                 nop                               ; SEH 句柄:0047B95C  
SDPI:0047B898                 cmp     al, 4  
SDPI:0047B89A                 jz      short Pass\_47B90D  
SDPI:0047B89C  
SDPI:0047B89C ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047B89C  
SDPI:0047B89C  
SDPI:0047B89C fnddbg\_47B89C   proc near                         ; CODE XREF: SDPI:0047B926 j  
SDPI:0047B89C                                                   ; SDPI:0047B93E j ...  
SDPI:0047B89C                 nop  
SDPI:0047B89D                 nop  
SDPI:0047B89E                 nop  
SDPI:0047B89F                 nop  
SDPI:0047B8A0                 nop  
SDPI:0047B8A1                 call    loc\_47B8A7  
SDPI:0047B8A6                 nop  
SDPI:0047B8A7  
SDPI:0047B8A7 loc\_47B8A7:                                       ; CODE XREF: fnddbg\_47B89C+5 p  
SDPI:0047B8A7                 pop     eax  
SDPI:0047B8A8                 add     eax, 5Eh  
SDPI:0047B8AD                 mov     edx, eax  
SDPI:0047B8AF                 add     edx, 32h  
SDPI:0047B8B2                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047B8B2                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047B8B2                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047B8B2                                                   ; 用来解密  
SDPI:0047B8B7                 call    loc\_47B8BD  
SDPI:0047B8BC                 nop  
SDPI:0047B8BD  
SDPI:0047B8BD loc\_47B8BD:                                       ; CODE XREF: fnddbg\_47B89C+1B p  
SDPI:0047B8BD                 pop     eax  
SDPI:0047B8BE                 add     eax, 37E0h  
SDPI:0047B8C3                 call    loc\_47B8C9  
SDPI:0047B8C8                 nop  
SDPI:0047B8C9  
SDPI:0047B8C9 loc\_47B8C9:                                       ; CODE XREF: fnddbg\_47B89C+27 p  
SDPI:0047B8C9                 pop     ecx  
SDPI:0047B8CA                 add     ecx, 3881h  
SDPI:0047B8D0                 push    0  
SDPI:0047B8D2                 push    ecx  
SDPI:0047B8D3                 push    eax  
SDPI:0047B8D4                 push    0  
SDPI:0047B8D6                 call    loc\_47B8DC  
SDPI:0047B8DB                 nop  
SDPI:0047B8DC  
SDPI:0047B8DC loc\_47B8DC:                                       ; CODE XREF: fnddbg\_47B89C+3A p  
SDPI:0047B8DC                 pop     eax  
SDPI:0047B8DD                 add     eax, 11h  
SDPI:0047B8E2                 push    eax  
SDPI:0047B8E3                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047B8E3                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047B8E3                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047B8E3 ; ----------------------------------------------------------------------------  
SDPI:0047B8E8                 db  90h ; ?  
SDPI:0047B8E9                 db  90h ; ?  
SDPI:0047B8EA                 db  90h ; ?  
SDPI:0047B8EB                 db  90h ; ?  
SDPI:0047B8EC ; ----------------------------------------------------------------------------  
SDPI:0047B8EC                 push    7  
SDPI:0047B8EE                 call    loc\_47B8F4  
SDPI:0047B8F3                 nop  
SDPI:0047B8F4  
SDPI:0047B8F4 loc\_47B8F4:                                       ; CODE XREF: fnddbg\_47B89C+52 p  
SDPI:0047B8F4                 pop     eax  
SDPI:0047B8F5                 add     eax, 11h  
SDPI:0047B8FA                 push    eax  
SDPI:0047B8FB                 jmp     ExitProcess  
SDPI:0047B8FB ; ----------------------------------------------------------------------------  
SDPI:0047B900 aRrrrs@I        db '悙悙?,0,10h,'@',0,'皦?,4  
SDPI:0047B900 fnddbg\_47B89C   endp  
SDPI:0047B900  
SDPI:0047B90D ; ----------------------------------------------------------------------------  
SDPI:0047B90D  
SDPI:0047B90D Pass\_47B90D:                                      ; CODE XREF: SDPI:0047B89A j  
SDPI:0047B90D                 pop     large dword ptr fs:0  
SDPI:0047B914                 add     esp, 4  
SDPI:0047B917                 call    loc\_47B91D  
SDPI:0047B91C                 nop  
SDPI:0047B91D  
SDPI:0047B91D loc\_47B91D:                                       ; CODE XREF: SDPI:0047B917 p  
SDPI:0047B91D                 pop     eax  
SDPI:0047B91E                 add     eax, 0FFFFFE1Dh  
SDPI:0047B923                 cmp     byte ptr \[eax\], 0E9h  
SDPI:0047B926                 jnz     fnddbg\_47B89C  
SDPI:0047B92C                 mov     byte ptr \[eax\], 0E8h  
SDPI:0047B92F                 rdtsc  
SDPI:0047B931                 mov     ecx, eax  
SDPI:0047B933                 mov     ebx, edx  
SDPI:0047B935                 rdtsc  
SDPI:0047B937                 sub     eax, ecx  
SDPI:0047B939                 sbb     edx, ebx  
SDPI:0047B93B                 cmp     edx, 0  
SDPI:0047B93E                 jnz     fnddbg\_47B89C  
SDPI:0047B944                 cmp     eax, 30000000h  
SDPI:0047B949                 ja      fnddbg\_47B89C             ; 作用用的宏，因此没什么好介绍的了  
SDPI:0047B94F                 jz      short PASS\_47B99A  
SDPI:0047B951                 jnz     short PASS\_47B99A  
SDPI:0047B951 ; ----------------------------------------------------------------------------  
SDPI:0047B953 aS@I            db '?,0,10h,'@',0,'皦?,4  
SDPI:0047B95C ; ----------------------------------------------------------------------------  
SDPI:0047B95C                 mov     eax, \[esp+4\]              ; SEH 处理处  
SDPI:0047B960                 mov     ecx, \[esp+0Ch\]  
SDPI:0047B964                 inc     dword ptr \[ecx+0B8h\]  
SDPI:0047B96A                 mov     eax, \[eax\]  
SDPI:0047B96C                 sub     eax, 80000003h  
SDPI:0047B971                 jnz     short locret\_47B999  
SDPI:0047B973                 call    sub\_47B979  
SDPI:0047B978                 nop  
SDPI:0047B979  
SDPI:0047B979 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047B979  
SDPI:0047B979  
SDPI:0047B979 sub\_47B979      proc near                         ; CODE XREF: SDPI:0047B973 p  
SDPI:0047B979                 pop     eax  
SDPI:0047B97A                 add     eax, 0FFFFFDC1h  
SDPI:0047B97F                 cmp     byte ptr \[eax\], 0E8h  
SDPI:0047B982                 jnz     fnddbg\_47B89C  
SDPI:0047B988                 mov     byte ptr \[eax\], 0E9h  
SDPI:0047B98B                 xor     eax, eax  
SDPI:0047B98D                 mov     \[ecx+4\], eax  
SDPI:0047B990                 mov     \[ecx+8\], eax  
SDPI:0047B993                 mov     \[ecx+0Ch\], eax  
SDPI:0047B996                 mov     \[ecx+10h\], eax  
SDPI:0047B999  
SDPI:0047B999 locret\_47B999:                                    ; CODE XREF: SDPI:0047B971 j  
SDPI:0047B999                 retn  
SDPI:0047B999 sub\_47B979      endp ; sp =  4  
SDPI:0047B999  
SDPI:0047B99A ; ----------------------------------------------------------------------------  
SDPI:0047B99A  
SDPI:0047B99A PASS\_47B99A:                                      ; CODE XREF: SDPI:0047B94F j  
SDPI:0047B99A                                                   ; SDPI:0047B951 j  
SDPI:0047B99A                 pop     eax  
SDPI:0047B99B                 call    loc\_47BB7D  
SDPI:0047B99B ; ----------------------------------------------------------------------------  
SDPI:0047B9A0                 dd 401000h  
SDPI:0047B9A4                 dd 15C56BEh  
SDPI:0047B9A8 ; ----------------------------------------------------------------------------  
SDPI:0047B9A8  
SDPI:0047B9A8 loc\_47B9A8:                                       ; CODE XREF: SDPI:0047BBA6 p  
SDPI:0047B9A8                 pop     ebp  
SDPI:0047B9A9                 pop     eax  
SDPI:0047B9AA                 jmp     loc\_47BBAB  
SDPI:0047B9AF ; ----------------------------------------------------------------------------  
SDPI:0047B9AF                 mov     ecx, 0FFFFFF00h  
SDPI:0047B9B4                 push    fs  
SDPI:0047B9B6                 jz      short loc\_47B9C2  
SDPI:0047B9B8                 jnz     short loc\_47B9C2  
SDPI:0047B9B8 ; ----------------------------------------------------------------------------  
SDPI:0047B9BA                 dd 401000h  
SDPI:0047B9BE                 dd 49C89B0h  
SDPI:0047B9C2 ; ----------------------------------------------------------------------------  
SDPI:0047B9C2  
SDPI:0047B9C2 loc\_47B9C2:                                       ; CODE XREF: SDPI:0047B9B6 j  
SDPI:0047B9C2                                                   ; SDPI:0047B9B8 j  
SDPI:0047B9C2                 pushfw  
SDPI:0047B9C4                 push    eax  
SDPI:0047B9C5                 mov     eax, ebx  
SDPI:0047B9C7                 push    ebx  
SDPI:0047B9C8                 mov     eax, ecx  
SDPI:0047B9CA                 push    eax  
SDPI:0047B9CB                 add     eax, edx  
SDPI:0047B9CD                 mov     ebx, eax  
SDPI:0047B9CF                 push    ebx  
SDPI:0047B9D0                 pop     eax  
SDPI:0047B9D1                 push    edx  
SDPI:0047B9D2                 call    loc\_47B9DF  
SDPI:0047B9D2 ; ----------------------------------------------------------------------------  
SDPI:0047B9D7                 dd 401000h  
SDPI:0047B9DB                 dd 132BD7B0h  
SDPI:0047B9DF ; ----------------------------------------------------------------------------  
SDPI:0047B9DF  
SDPI:0047B9DF loc\_47B9DF:                                       ; CODE XREF: SDPI:0047B9D2 p  
SDPI:0047B9DF                 pop     eax  
SDPI:0047B9E0                 call    loc\_47B9E6  
SDPI:0047B9E5                 nop  
SDPI:0047B9E6  
SDPI:0047B9E6 loc\_47B9E6:                                       ; CODE XREF: SDPI:0047B9E0 p  
SDPI:0047B9E6                 pop     eax  
SDPI:0047B9E7                 add     eax, 11h  
SDPI:0047B9EC                 push    eax  
SDPI:0047B9ED                 jmp     GetTickCount  
SDPI:0047B9ED ; ----------------------------------------------------------------------------  
SDPI:0047B9F2                 dd 90909090h  
SDPI:0047B9F6 ; ----------------------------------------------------------------------------  
SDPI:0047B9F6                 push    eax  
SDPI:0047B9F7                 mov     eax, edx  
SDPI:0047B9F9                 push    eax  
SDPI:0047B9FA                 call    loc\_47BA00  
SDPI:0047B9FF                 nop  
SDPI:0047BA00  
SDPI:0047BA00 loc\_47BA00:                                       ; CODE XREF: SDPI:0047B9FA p  
SDPI:0047BA00                 pop     edx  
SDPI:0047BA01                 add     edx, 52h  
SDPI:0047BA07                 push    edx  
SDPI:0047BA08                 add     edx, 402A08h  
SDPI:0047BA0E                 push    edx  
SDPI:0047BA0F                 jo      short loc\_47BA64  
SDPI:0047BA11                 jno     short loc\_47BA64  
SDPI:0047BA13  
SDPI:0047BA13 loc\_47BA13:                                       ; CODE XREF: SDPI:0047BA57 p  
SDPI:0047BA13                 pop     eax  
SDPI:0047BA14                 pop     ebx  
SDPI:0047BA15                 call    loc\_47BA1B  
SDPI:0047BA1A                 nop  
SDPI:0047BA1B  
SDPI:0047BA1B loc\_47BA1B:                                       ; CODE XREF: SDPI:0047BA15 p  
SDPI:0047BA1B                 pop     eax  
SDPI:0047BA1C                 add     eax, 11h  
SDPI:0047BA21                 push    eax  
SDPI:0047BA22                 jmp     GetTickCount  
SDPI:0047BA22 ; ----------------------------------------------------------------------------  
SDPI:0047BA27                 dd 90909090h  
SDPI:0047BA2B ; ----------------------------------------------------------------------------  
SDPI:0047BA2B                 pop     ebx  
SDPI:0047BA2C                 add     ebx, 1F4h  
SDPI:0047BA32                 sub     ebx, eax                  ; 这里同上，跳则 over  
SDPI:0047BA34                 js      short FNDDBG\_47BA78  
SDPI:0047BA36                 call    loc\_47BA3C  
SDPI:0047BA3B                 nop  
SDPI:0047BA3C  
SDPI:0047BA3C loc\_47BA3C:                                       ; CODE XREF: SDPI:0047BA36 p  
SDPI:0047BA3C                 pop     ebx  
SDPI:0047BA3D                 add     ebx, 0A5h  
SDPI:0047BA43                 push    ebx  
SDPI:0047BA44                 call    loc\_47BA6E  
SDPI:0047BA44 ; ----------------------------------------------------------------------------  
SDPI:0047BA49                 dd 401000h  
SDPI:0047BA4D                 dd 58C88B0h  
SDPI:0047BA51 ; ----------------------------------------------------------------------------  
SDPI:0047BA51                 pop     eax  
SDPI:0047BA52                 mov     edx, eax  
SDPI:0047BA54                 mov     eax, ebx  
SDPI:0047BA56                 push    eax  
SDPI:0047BA57                 call    loc\_47BA13  
SDPI:0047BA57 ; ----------------------------------------------------------------------------  
SDPI:0047BA5C                 dd 401000h  
SDPI:0047BA60                 dd 1833639h  
SDPI:0047BA64 ; ----------------------------------------------------------------------------  
SDPI:0047BA64  
SDPI:0047BA64 loc\_47BA64:                                       ; CODE XREF: SDPI:0047BA0F j  
SDPI:0047BA64                                                   ; SDPI:0047BA11 j  
SDPI:0047BA64                 pop     eax  
SDPI:0047BA65                 retn  
SDPI:0047BA65 ; ----------------------------------------------------------------------------  
SDPI:0047BA66                 dd 401000h  
SDPI:0047BA6A                 dd 77C563Eh  
SDPI:0047BA6E ; ----------------------------------------------------------------------------  
SDPI:0047BA6E  
SDPI:0047BA6E loc\_47BA6E:                                       ; CODE XREF: SDPI:0047BA44 p  
SDPI:0047BA6E                 pop     edx  
SDPI:0047BA6F                 retn  
SDPI:0047BA6F ; ----------------------------------------------------------------------------  
SDPI:0047BA70 JUNK\_47BA70     db 0,10h,'@',0,'颯?,1  
SDPI:0047BA78  
SDPI:0047BA78 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047BA78  
SDPI:0047BA78  
SDPI:0047BA78 FNDDBG\_47BA78   proc near                         ; CODE XREF: SDPI:0047BA34 j  
SDPI:0047BA78                 nop  
SDPI:0047BA79                 nop  
SDPI:0047BA7A                 nop  
SDPI:0047BA7B                 nop  
SDPI:0047BA7C                 nop  
SDPI:0047BA7D                 call    loc\_47BA83  
SDPI:0047BA82                 nop  
SDPI:0047BA83  
SDPI:0047BA83 loc\_47BA83:                                       ; CODE XREF: FNDDBG\_47BA78+5 p  
SDPI:0047BA83                 pop     eax  
SDPI:0047BA84                 add     eax, 5Eh  
SDPI:0047BA89                 mov     edx, eax  
SDPI:0047BA8B                 add     edx, 32h  
SDPI:0047BA8E                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047BA8E                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047BA8E                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047BA8E                                                   ; 用来解密  
SDPI:0047BA93                 call    loc\_47BA99  
SDPI:0047BA98                 nop  
SDPI:0047BA99  
SDPI:0047BA99 loc\_47BA99:                                       ; CODE XREF: FNDDBG\_47BA78+1B p  
SDPI:0047BA99                 pop     eax  
SDPI:0047BA9A                 add     eax, 3604h  
SDPI:0047BA9F                 call    loc\_47BAA5  
SDPI:0047BAA4                 nop  
SDPI:0047BAA5  
SDPI:0047BAA5 loc\_47BAA5:                                       ; CODE XREF: FNDDBG\_47BA78+27 p  
SDPI:0047BAA5                 pop     ecx  
SDPI:0047BAA6                 add     ecx, 36A5h  
SDPI:0047BAAC                 push    0  
SDPI:0047BAAE                 push    ecx  
SDPI:0047BAAF                 push    eax  
SDPI:0047BAB0                 push    0  
SDPI:0047BAB2                 call    loc\_47BAB8  
SDPI:0047BAB7                 nop  
SDPI:0047BAB8  
SDPI:0047BAB8 loc\_47BAB8:                                       ; CODE XREF: FNDDBG\_47BA78+3A p  
SDPI:0047BAB8                 pop     eax  
SDPI:0047BAB9                 add     eax, 11h  
SDPI:0047BABE                 push    eax  
SDPI:0047BABF                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047BABF                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047BABF                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047BABF ; ----------------------------------------------------------------------------  
SDPI:0047BAC4                 dd 90909090h  
SDPI:0047BAC8 ; ----------------------------------------------------------------------------  
SDPI:0047BAC8                 push    7  
SDPI:0047BACA                 call    loc\_47BAD0  
SDPI:0047BACF                 nop  
SDPI:0047BAD0  
SDPI:0047BAD0 loc\_47BAD0:                                       ; CODE XREF: FNDDBG\_47BA78+52 p  
SDPI:0047BAD0                 pop     eax  
SDPI:0047BAD1                 add     eax, 11h  
SDPI:0047BAD6                 push    eax  
SDPI:0047BAD7                 jmp     ExitProcess  
SDPI:0047BAD7 ; ----------------------------------------------------------------------------  
SDPI:0047BADC                 dd 90909090h  
SDPI:0047BADC FNDDBG\_47BA78   endp  
SDPI:0047BADC  
SDPI:0047BAE0 ; ----------------------------------------------------------------------------  
SDPI:0047BAE0                 pop     edx  
SDPI:0047BAE1                 mov     eax, ecx  
SDPI:0047BAE3                 add     eax, edx  
SDPI:0047BAE5                 inc     ecx  
SDPI:0047BAE6                 push    eax  
SDPI:0047BAE7                 inc     ecx  
SDPI:0047BAE8                 pop     ebx  
SDPI:0047BAE9                 pop     ecx  
SDPI:0047BAEA                 push    eax  
SDPI:0047BAEB                 sub     eax, 8  
SDPI:0047BAEE                 pop     ebx  
SDPI:0047BAEF                 pop     ebx  
SDPI:0047BAF0                 inc     eax  
SDPI:0047BAF1                 add     eax, ebx  
SDPI:0047BAF3                 pop     eax  
SDPI:0047BAF4                 pushfw  
SDPI:0047BAF6                 popfw  
SDPI:0047BAF8                 popfw  
SDPI:0047BAFA                 pop     es  
SDPI:0047BAFB                 mov     eax, 12345678h  
SDPI:0047BB00                 push    eax  
SDPI:0047BB01                 call    loc\_47BB07  
SDPI:0047BB06                 nop  
SDPI:0047BB07  
SDPI:0047BB07 loc\_47BB07:                                       ; CODE XREF: SDPI:0047BB01 p  
SDPI:0047BB07                 pop     eax  
SDPI:0047BB08                 add     eax, 12Ch  
SDPI:0047BB0D                 push    eax  
SDPI:0047BB0E                 pop     ebx  
SDPI:0047BB0F                 add     eax, 12h  
SDPI:0047BB12                 pop     edx  
SDPI:0047BB13                 add     eax, edx  
SDPI:0047BB15                 mov     edx, eax  
SDPI:0047BB17                 push    ebx  
SDPI:0047BB18                 mov     ebx, es:\[ecx+100h\]  
SDPI:0047BB1F                 push    ebx  
SDPI:0047BB20                 mov     eax, esp  
SDPI:0047BB22                 mov     ebx, eax  
SDPI:0047BB24                 push    ebx  
SDPI:0047BB25                 pop     edx  
SDPI:0047BB26                 mov     es:\[ecx+100h\], eax  
SDPI:0047BB2D                 xor     eax, eax  
SDPI:0047BB2F                 jle     short loc\_47BB38  
SDPI:0047BB31                 jg      short loc\_47BB38  
SDPI:0047BB31 ; ----------------------------------------------------------------------------  
SDPI:0047BB33                 dd 401000h  
SDPI:0047BB37                 db 0E8h ; ?  
SDPI:0047BB38 ; ----------------------------------------------------------------------------  
SDPI:0047BB38  
SDPI:0047BB38 loc\_47BB38:                                       ; CODE XREF: SDPI:0047BB2F j  
SDPI:0047BB38                                                   ; SDPI:0047BB31 j  
SDPI:0047BB38                 pushfw  
SDPI:0047BB3A                 push    ecx  
SDPI:0047BB3B                 xor     ecx, ecx  
SDPI:0047BB3D                 jcxz    loc\_47BB45  
SDPI:0047BB40                 add     \[eax\], dl  
SDPI:0047BB42                 inc     eax  
SDPI:0047BB43                 add     al, ch  
SDPI:0047BB45  
SDPI:0047BB45 loc\_47BB45:                                       ; CODE XREF: SDPI:0047BB3D j  
SDPI:0047BB45                 pop     ecx  
SDPI:0047BB46                 nop  
SDPI:0047BB47                 nop  
SDPI:0047BB48                 nop  
SDPI:0047BB49                 nop  
SDPI:0047BB4A                 nop  
SDPI:0047BB4B                 nop  
SDPI:0047BB4C                 nop  
SDPI:0047BB4D                 nop  
SDPI:0047BB4E                 nop  
SDPI:0047BB4F                 nop  
SDPI:0047BB50                 nop  
SDPI:0047BB51                 nop  
SDPI:0047BB52                 nop  
SDPI:0047BB53                 nop  
SDPI:0047BB54                 nop  
SDPI:0047BB55                 nop  
SDPI:0047BB56                 nop  
SDPI:0047BB57                 nop  
SDPI:0047BB58                 nop  
SDPI:0047BB59                 nop  
SDPI:0047BB5A                 nop  
SDPI:0047BB5B                 nop  
SDPI:0047BB5C                 nop  
SDPI:0047BB5D                 popfw  
SDPI:0047BB5F                 jo      short loc\_47BB67  
SDPI:0047BB61                 jno     short loc\_47BB67  
SDPI:0047BB61 ; ----------------------------------------------------------------------------  
SDPI:0047BB63                 dd 401000h  
SDPI:0047BB67 ; ----------------------------------------------------------------------------  
SDPI:0047BB67  
SDPI:0047BB67 loc\_47BB67:                                       ; CODE XREF: SDPI:0047BB5F j  
SDPI:0047BB67                                                   ; SDPI:0047BB61 j  
SDPI:0047BB67                 int     3                         ; Trap to Debugger  
SDPI:0047BB68                 nop                               ; SEH 位置:0047BC32  
SDPI:0047BB69                 xor     eax, eax  
SDPI:0047BB6B                 mov     dword ptr \[eax\], 402C6Bh  
SDPI:0047BB71                 jp      short loc\_47BB7D  
SDPI:0047BB73                 jnp     short loc\_47BB7D  
SDPI:0047BB73 ; ----------------------------------------------------------------------------  
SDPI:0047BB75                 dd 401000h  
SDPI:0047BB79                 dd 403D7Bh  
SDPI:0047BB7D ; ----------------------------------------------------------------------------  
SDPI:0047BB7D  
SDPI:0047BB7D loc\_47BB7D:                                       ; CODE XREF: SDPI:0047B99B p  
SDPI:0047BB7D                                                   ; SDPI:0047BB71 j ...  
SDPI:0047BB7D                 call    loc\_47BB83  
SDPI:0047BB82                 nop  
SDPI:0047BB83  
SDPI:0047BB83 loc\_47BB83:                                       ; CODE XREF: SDPI:loc\_47BB7D p  
SDPI:0047BB83                 pop     eax  
SDPI:0047BB84                 add     eax, 11h  
SDPI:0047BB89                 push    eax  
SDPI:0047BB8A                 jmp     GetTickCount  
SDPI:0047BB8A ; ----------------------------------------------------------------------------  
SDPI:0047BB8F                 dd 90909090h  
SDPI:0047BB93 ; ----------------------------------------------------------------------------  
SDPI:0047BB93                 call    loc\_47BB99  
SDPI:0047BB98                 nop  
SDPI:0047BB99  
SDPI:0047BB99 loc\_47BB99:                                       ; CODE XREF: SDPI:0047BB93 p  
SDPI:0047BB99                 pop     edx  
SDPI:0047BB9A                 add     edx, 0FFFFFB04h  
SDPI:0047BBA0                 mov     \[edx\], eax  
SDPI:0047BBA2                 pop     ebp  
SDPI:0047BBA3                 add     eax, edx  
SDPI:0047BBA5                 push    eax  
SDPI:0047BBA6                 call    loc\_47B9A8  
SDPI:0047BBAB  
SDPI:0047BBAB loc\_47BBAB:                                       ; CODE XREF: SDPI:0047B9AA j  
SDPI:0047BBAB                 call    loc\_47BBB1  
SDPI:0047BBB0                 nop  
SDPI:0047BBB1  
SDPI:0047BBB1 loc\_47BBB1:                                       ; CODE XREF: SDPI:loc\_47BBAB p  
SDPI:0047BBB1                 pop     edx  
SDPI:0047BBB2                 add     edx, 0FFFFFDFFh  
SDPI:0047BBB8                 add     edx, eax  
SDPI:0047BBBA                 push    edx  
SDPI:0047BBBB                 pop     ecx  
SDPI:0047BBBC                 sub     ecx, eax  
SDPI:0047BBBE                 push    ecx  
SDPI:0047BBBF                 retn    4  
SDPI:0047BBC2  
SDPI:0047BBC2 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047BBC2  
SDPI:0047BBC2  
SDPI:0047BBC2 FNDDBG\_47BBC2   proc near                         ; CODE XREF: SDPI:0047BC65 j  
SDPI:0047BBC2                                                   ; SDPI:0047BC6D j ...  
SDPI:0047BBC2                 nop  
SDPI:0047BBC3                 nop  
SDPI:0047BBC4                 nop  
SDPI:0047BBC5                 nop  
SDPI:0047BBC6                 nop  
SDPI:0047BBC7                 call    loc\_47BBCD  
SDPI:0047BBCC                 nop  
SDPI:0047BBCD  
SDPI:0047BBCD loc\_47BBCD:                                       ; CODE XREF: FNDDBG\_47BBC2+5 p  
SDPI:0047BBCD                 pop     eax  
SDPI:0047BBCE                 add     eax, 5Eh  
SDPI:0047BBD3                 mov     edx, eax  
SDPI:0047BBD5                 add     edx, 32h  
SDPI:0047BBD8                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047BBD8                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047BBD8                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047BBD8                                                   ; 用来解密  
SDPI:0047BBDD                 call    loc\_47BBE3  
SDPI:0047BBE2                 nop  
SDPI:0047BBE3  
SDPI:0047BBE3 loc\_47BBE3:                                       ; CODE XREF: FNDDBG\_47BBC2+1B p  
SDPI:0047BBE3                 pop     eax  
SDPI:0047BBE4                 add     eax, 34BAh  
SDPI:0047BBE9                 call    loc\_47BBEF  
SDPI:0047BBEE                 nop  
SDPI:0047BBEF  
SDPI:0047BBEF loc\_47BBEF:                                       ; CODE XREF: FNDDBG\_47BBC2+27 p  
SDPI:0047BBEF                 pop     ecx  
SDPI:0047BBF0                 add     ecx, 355Bh  
SDPI:0047BBF6                 push    0  
SDPI:0047BBF8                 push    ecx  
SDPI:0047BBF9                 push    eax  
SDPI:0047BBFA                 push    0  
SDPI:0047BBFC                 call    loc\_47BC02  
SDPI:0047BC01                 nop  
SDPI:0047BC02  
SDPI:0047BC02 loc\_47BC02:                                       ; CODE XREF: FNDDBG\_47BBC2+3A p  
SDPI:0047BC02                 pop     eax  
SDPI:0047BC03                 add     eax, 11h  
SDPI:0047BC08                 push    eax  
SDPI:0047BC09                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047BC09                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047BC09                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047BC09 ; ----------------------------------------------------------------------------  
SDPI:0047BC0E                 dd 90909090h  
SDPI:0047BC12 ; ----------------------------------------------------------------------------  
SDPI:0047BC12                 push    7  
SDPI:0047BC14                 call    loc\_47BC1A  
SDPI:0047BC19                 nop  
SDPI:0047BC1A  
SDPI:0047BC1A loc\_47BC1A:                                       ; CODE XREF: FNDDBG\_47BBC2+52 p  
SDPI:0047BC1A                 pop     eax  
SDPI:0047BC1B                 add     eax, 11h  
SDPI:0047BC20                 push    eax  
SDPI:0047BC21                 jmp     ExitProcess  
SDPI:0047BC21 ; ----------------------------------------------------------------------------  
SDPI:0047BC26 aRrrr@V\_0       db '悙悙',0,10h,'@',0,'綱',1  
SDPI:0047BC26 FNDDBG\_47BBC2   endp  
SDPI:0047BC26  
SDPI:0047BC32 ; ----------------------------------------------------------------------------  
SDPI:0047BC32                 mov     esp, \[esp+8\]  
SDPI:0047BC36                 pop     large dword ptr fs:0  
SDPI:0047BC3D                 call    loc\_47BC43  
SDPI:0047BC42                 nop  
SDPI:0047BC43  
SDPI:0047BC43 loc\_47BC43:                                       ; CODE XREF: SDPI:0047BC3D p  
SDPI:0047BC43                 pop     eax  
SDPI:0047BC44                 add     eax, 11h  
SDPI:0047BC49                 push    eax  
SDPI:0047BC4A                 jmp     GetTickCount  
SDPI:0047BC4A ; ----------------------------------------------------------------------------  
SDPI:0047BC4F                 dd 90909090h  
SDPI:0047BC53 ; ----------------------------------------------------------------------------  
SDPI:0047BC53                 call    loc\_47BC59  
SDPI:0047BC58                 nop  
SDPI:0047BC59  
SDPI:0047BC59 loc\_47BC59:                                       ; CODE XREF: SDPI:0047BC53 p  
SDPI:0047BC59                 pop     edx  
SDPI:0047BC5A                 add     edx, 0FFFFFA44h  
SDPI:0047BC60                 mov     ecx, \[edx\]  
SDPI:0047BC62                 cmp     ecx, 0  
SDPI:0047BC65                 jz      FNDDBG\_47BBC2  
SDPI:0047BC6B                 sub     eax, ecx  
SDPI:0047BC6D                 js      FNDDBG\_47BBC2  
SDPI:0047BC73                 sub     eax, 7D0h  
SDPI:0047BC78                 jns     FNDDBG\_47BBC2  
SDPI:0047BC7E                 mov     eax, 0E801276h  
SDPI:0047BC83                 mov     \[edx\], eax  
SDPI:0047BC85                 call    loc\_47BC8B  
SDPI:0047BC8A                 nop  
SDPI:0047BC8B  
SDPI:0047BC8B loc\_47BC8B:                                       ; CODE XREF: SDPI:0047BC85 p  
SDPI:0047BC8B                 pop     edx  
SDPI:0047BC8C                 add     edx, 8BAh                 ; 解密终止地址 47c544  
SDPI:0047BC92                 call    loc\_47BC98  
SDPI:0047BC97                 nop  
SDPI:0047BC98  
SDPI:0047BC98 loc\_47BC98:                                       ; CODE XREF: SDPI:0047BC92 p  
SDPI:0047BC98                 pop     eax  
SDPI:0047BC99                 add     eax, 0FFFFE4B9h  
SDPI:0047BC9E                 mov     ecx, 10h  
SDPI:0047BCA3                 call    De\_Code                   ; invoke De\_Code,ecx(counter),eax(key),edx(end)  
SDPI:0047BCA3                                                   ; 用于解密代码，解密起始地址就是 call 进来  
SDPI:0047BCA3                                                   ; 的下一行代码地址  
SDPI:0047BCA8                 call    loc\_47BCAE  
SDPI:0047BCAD                 nop  
SDPI:0047BCAE  
SDPI:0047BCAE loc\_47BCAE:                                       ; CODE XREF: SDPI:0047BCA8 p  
SDPI:0047BCAE                 pop     eax  
SDPI:0047BCAF                 add     eax, 11h  
SDPI:0047BCB4                 push    eax  
SDPI:0047BCB5                 jmp     CHK\_R0D\_4825DA            ; 检测 RING 0 级调试器  
SDPI:0047BCB5                                                   ; 只检测 ICE 和 ICEEXT  
SDPI:0047BCB5                                                   ; 看来私下的秘密武器还是有  
SDPI:0047BCB5                                                   ; 很大作用^\_^  
SDPI:0047BCB5 ; ----------------------------------------------------------------------------  
SDPI:0047BCBA                 dd 90909090h  
SDPI:0047BCBE ; ----------------------------------------------------------------------------  
SDPI:0047BCBE                 call    loc\_47BCC4  
SDPI:0047BCC3                 nop  
SDPI:0047BCC4  
SDPI:0047BCC4 loc\_47BCC4:                                       ; CODE XREF: SDPI:0047BCBE p  
SDPI:0047BCC4                 pop     eax  
SDPI:0047BCC5                 add     eax, 11h  
SDPI:0047BCCA                 push    eax  
SDPI:0047BCCB                 jmp     Get\_Version  
SDPI:0047BCCB ; ----------------------------------------------------------------------------  
SDPI:0047BCD0                 db  90h ; ?  
SDPI:0047BCD1                 db  90h ; ?  
SDPI:0047BCD2                 db  90h ; ?  
SDPI:0047BCD3                 db  90h ; ?  
SDPI:0047BCD4 ; ----------------------------------------------------------------------------  
SDPI:0047BCD4                 call    loc\_47BCDA  
SDPI:0047BCD9                 nop  
SDPI:0047BCDA  
SDPI:0047BCDA loc\_47BCDA:                                       ; CODE XREF: SDPI:0047BCD4 p  
SDPI:0047BCDA                 pop     edx  
SDPI:0047BCDB                 add     edx, 0FFFFE32Bh  
SDPI:0047BCE1                 mov     \[edx\], eax                ; 保存 Version info  
SDPI:0047BCE3                 pushf  
SDPI:0047BCE4                 pop     eax                       ; 反单步跟踪  
SDPI:0047BCE5                 test    eax, 100h  
SDPI:0047BCEA                 jz      short Pass\_47BD54  
SDPI:0047BCEC  
SDPI:0047BCEC ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047BCEC  
SDPI:0047BCEC ; 检测到调试器则 OVER 了  
SDPI:0047BCEC  
SDPI:0047BCEC FNDDBG\_47BCEC   proc near  
SDPI:0047BCEC                 nop  
SDPI:0047BCED                 nop  
SDPI:0047BCEE                 nop  
SDPI:0047BCEF                 nop  
SDPI:0047BCF0                 nop  
SDPI:0047BCF1                 call    loc\_47BCF7  
SDPI:0047BCF6                 nop  
SDPI:0047BCF7  
SDPI:0047BCF7 loc\_47BCF7:                                       ; CODE XREF: FNDDBG\_47BCEC+5 p  
SDPI:0047BCF7                 pop     eax  
SDPI:0047BCF8                 add     eax, 5Eh  
SDPI:0047BCFD                 mov     edx, eax  
SDPI:0047BCFF                 add     edx, 32h  
SDPI:0047BD02                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047BD02                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047BD02                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047BD02                                                   ; 用来解密  
SDPI:0047BD07                 call    loc\_47BD0D  
SDPI:0047BD0C                 nop  
SDPI:0047BD0D  
SDPI:0047BD0D loc\_47BD0D:                                       ; CODE XREF: FNDDBG\_47BCEC+1B p  
SDPI:0047BD0D                 pop     eax  
SDPI:0047BD0E                 add     eax, 3390h  
SDPI:0047BD13                 call    loc\_47BD19  
SDPI:0047BD18                 nop  
SDPI:0047BD19  
SDPI:0047BD19 loc\_47BD19:                                       ; CODE XREF: FNDDBG\_47BCEC+27 p  
SDPI:0047BD19                 pop     ecx  
SDPI:0047BD1A                 add     ecx, 3431h  
SDPI:0047BD20                 push    0  
SDPI:0047BD22                 push    ecx  
SDPI:0047BD23                 push    eax  
SDPI:0047BD24                 push    0  
SDPI:0047BD26                 call    loc\_47BD2C  
SDPI:0047BD2B                 nop  
SDPI:0047BD2C  
SDPI:0047BD2C loc\_47BD2C:                                       ; CODE XREF: FNDDBG\_47BCEC+3A p  
SDPI:0047BD2C                 pop     eax  
SDPI:0047BD2D                 add     eax, 11h  
SDPI:0047BD32                 push    eax  
SDPI:0047BD33                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047BD33                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047BD33                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047BD33 ; ----------------------------------------------------------------------------  
SDPI:0047BD38                 db  90h ; ?  
SDPI:0047BD39                 db  90h ; ?  
SDPI:0047BD3A                 db  90h ; ?  
SDPI:0047BD3B                 db  90h ; ?  
SDPI:0047BD3C ; ----------------------------------------------------------------------------  
SDPI:0047BD3C                 push    7  
SDPI:0047BD3E                 call    loc\_47BD44  
SDPI:0047BD43                 nop  
SDPI:0047BD44  
SDPI:0047BD44 loc\_47BD44:                                       ; CODE XREF: FNDDBG\_47BCEC+52 p  
SDPI:0047BD44                 pop     eax  
SDPI:0047BD45                 add     eax, 11h  
SDPI:0047BD4A                 push    eax  
SDPI:0047BD4B                 jmp     ExitProcess  
SDPI:0047BD4B ; ----------------------------------------------------------------------------  
SDPI:0047BD50                 dd 90909090h  
SDPI:0047BD50 FNDDBG\_47BCEC   endp  
SDPI:0047BD50  
SDPI:0047BD54 ; ----------------------------------------------------------------------------  
SDPI:0047BD54  
SDPI:0047BD54 Pass\_47BD54:                                      ; CODE XREF: SDPI:0047BCEA j  
SDPI:0047BD54                 call    loc\_47BD5A  
SDPI:0047BD59                 nop  
SDPI:0047BD5A  
SDPI:0047BD5A loc\_47BD5A:                                       ; CODE XREF: SDPI:Pass\_47BD54 p  
SDPI:0047BD5A                 pop     eax  
SDPI:0047BD5B                 add     eax, 11h  
SDPI:0047BD60                 push    eax  
SDPI:0047BD61                 jmp     CHK\_IsREGED\_481774        ; 跳去判断加壳的主程序是否已注册  
SDPI:0047BD61 ; ----------------------------------------------------------------------------  
SDPI:0047BD66                 db  90h ; ?  
SDPI:0047BD67                 db  90h ; ?  
SDPI:0047BD68                 db  90h ; ?  
SDPI:0047BD69                 db  90h ; ?  
SDPI:0047BD6A ; ----------------------------------------------------------------------------  
SDPI:0047BD6A                 xor     eax, 87EAF247h  
SDPI:0047BD6F                 sub     eax, 254653EFh  
SDPI:0047BD6F ; ----------------------------------------------------------------------------  
SDPI:0047BD74 JUNK\_46BD74     db '|',7,'?,7,0,10h,'@',0,'鑤鱮',12h,'s',10h,0,10h,'@',0,'?,3,'?  
SDPI:0047BD74                 db 'r',1Bh,'s',19h,0,10h,'@',0,'鑖淧 3?豻扈',5,0,0,0,0,10h,'@',0  
SDPI:0047BD74                 db '鑈 Xf?  
SDPI:0047BDAA ; ----------------------------------------------------------------------------  
SDPI:0047BDAA                 jz      short IsReged\_47BE17  
SDPI:0047BDAA ; ----------------------------------------------------------------------------  
SDPI:0047BDAC junk\_47bdac     db '|',7,'?,7,0,10h,'@',0,'鑤鱮',12h,'s',10h,0,10h,'@',0,'?,3,'?  
SDPI:0047BDAC                 db 'r',1Bh,'s',19h,0,10h,'@',0,'鑖淧 3?豻扈',5,0,0,0,0,10h,'@',0  
SDPI:0047BDAC                 db '鑈 Xf?  
SDPI:0047BDE2  
SDPI:0047BDE2 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047BDE2  
SDPI:0047BDE2 ; 显示未注册信息框  
SDPI:0047BDE2  
SDPI:0047BDE2 UnReg\_MSG       proc near  
SDPI:0047BDE2                 call    loc\_47BDE8  
SDPI:0047BDE7                 nop  
SDPI:0047BDE8  
SDPI:0047BDE8 loc\_47BDE8:                                       ; CODE XREF: UnReg\_MSG p  
SDPI:0047BDE8                 pop     eax  
SDPI:0047BDE9                 add     eax, 336Ch  
SDPI:0047BDEE                 call    loc\_47BDF4  
SDPI:0047BDF3                 nop  
SDPI:0047BDF4  
SDPI:0047BDF4 loc\_47BDF4:                                       ; CODE XREF: UnReg\_MSG+C p  
SDPI:0047BDF4                 pop     ecx  
SDPI:0047BDF5                 add     ecx, 3457h  
SDPI:0047BDFB                 push    0  
SDPI:0047BDFD                 push    ecx  
SDPI:0047BDFE                 push    eax  
SDPI:0047BDFF                 push    0  
SDPI:0047BE01                 call    loc\_47BE07  
SDPI:0047BE06                 nop  
SDPI:0047BE07  
SDPI:0047BE07 loc\_47BE07:                                       ; CODE XREF: UnReg\_MSG+1F p  
SDPI:0047BE07                 pop     eax  
SDPI:0047BE08                 add     eax, 11h  
SDPI:0047BE0D                 push    eax  
SDPI:0047BE0E                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047BE0E                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047BE0E                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047BE0E ; ----------------------------------------------------------------------------  
SDPI:0047BE13                 db  90h ; ?  
SDPI:0047BE14                 db  90h ; ?  
SDPI:0047BE15                 db  90h ; ?  
SDPI:0047BE16                 db  90h ; ?  
SDPI:0047BE16 UnReg\_MSG       endp  
SDPI:0047BE16  
SDPI:0047BE17 ; ----------------------------------------------------------------------------  
SDPI:0047BE17  
SDPI:0047BE17 IsReged\_47BE17:                                   ; CODE XREF: SDPI:0047BDAA j  
SDPI:0047BE17                 jo      short loc\_47BE27  
SDPI:0047BE19                 jno     short loc\_47BE27  
SDPI:0047BE19 ; ----------------------------------------------------------------------------  
SDPI:0047BE1B                 dd 401000h  
SDPI:0047BE1F                 dd 217C56BFh  
SDPI:0047BE23                 dd 0E801276h  
SDPI:0047BE27 ; ----------------------------------------------------------------------------  
SDPI:0047BE27  
SDPI:0047BE27 loc\_47BE27:                                       ; CODE XREF: SDPI:IsReged\_47BE17 j  
SDPI:0047BE27                                                   ; SDPI:0047BE19 j  
SDPI:0047BE27                 push    eax  
SDPI:0047BE28                 xor     eax, eax  
SDPI:0047BE2A                 call    loc\_47BE30  
SDPI:0047BE2F                 nop  
SDPI:0047BE30  
SDPI:0047BE30 loc\_47BE30:                                       ; CODE XREF: SDPI:0047BE2A p  
SDPI:0047BE30                 pop     edi  
SDPI:0047BE31                 add     edi, 61h  
SDPI:0047BE37                 mov     ebx, \[edi\]  
SDPI:0047BE39                 mov     edx, \[edi+4\]  
SDPI:0047BE3C                 jz      short loc\_47BE48  
SDPI:0047BE3E                 jnz     short loc\_47BE48  
SDPI:0047BE3E ; ----------------------------------------------------------------------------  
SDPI:0047BE40                 dd 401000h  
SDPI:0047BE44                 dd 9F7AB0Bh  
SDPI:0047BE48 ; ----------------------------------------------------------------------------  
SDPI:0047BE48  
SDPI:0047BE48 loc\_47BE48:                                       ; CODE XREF: SDPI:0047BE3C j  
SDPI:0047BE48                                                   ; SDPI:0047BE3E j  
SDPI:0047BE48                 call    loc\_47BE4E  
SDPI:0047BE4D                 nop  
SDPI:0047BE4E  
SDPI:0047BE4E loc\_47BE4E:                                       ; CODE XREF: SDPI:loc\_47BE48 p  
SDPI:0047BE4E                 pop     esi  
SDPI:0047BE4F                 add     esi, 59h  
SDPI:0047BE55                 mov     ecx, 3  
SDPI:0047BE55 ; ----------------------------------------------------------------------------  
SDPI:0047BE5A JUNK\_47BE5A     db '|',7,'?,7,0,10h,'@',0,'鑤鱮',12h,'s',10h,0,10h,'@',0,'?,3,'?  
SDPI:0047BE5A                 db 'r',1Bh,'s',19h,0,10h,'@',0,'鑖淧 3?豻扈',5,0,0,0,0,10h,'@',0  
SDPI:0047BE5A                 db '鑈 Xf?  
SDPI:0047BE90 ; ----------------------------------------------------------------------------  
SDPI:0047BE90                 rep movsw  
SDPI:0047BE93                 call    FNDDBG\_47BF14  
SDPI:0047BE98                 call    INT3\_47BF84  
SDPI:0047BE98 ; ----------------------------------------------------------------------------  
SDPI:0047BE9D JUNK\_47BE9D     db '?,0,10h,'@',0,'皦?,4,'?,1,'愲',3,'悙?  
SDPI:0047BEAE  
SDPI:0047BEAE ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047BEAE  
SDPI:0047BEAE  
SDPI:0047BEAE FNDDBG\_47BEAE   proc near  
SDPI:0047BEAE                 nop  
SDPI:0047BEAF                 nop  
SDPI:0047BEB0                 nop  
SDPI:0047BEB1                 call    loc\_47BEB7  
SDPI:0047BEB6                 nop  
SDPI:0047BEB7  
SDPI:0047BEB7 loc\_47BEB7:                                       ; CODE XREF: FNDDBG\_47BEAE+3 p  
SDPI:0047BEB7                 pop     eax  
SDPI:0047BEB8                 add     eax, 5Eh  
SDPI:0047BEBD                 mov     edx, eax  
SDPI:0047BEBF                 add     edx, 32h  
SDPI:0047BEC2                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047BEC2                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047BEC2                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047BEC2                                                   ; 用来解密  
SDPI:0047BEC7                 call    loc\_47BECD  
SDPI:0047BECC                 nop  
SDPI:0047BECD  
SDPI:0047BECD loc\_47BECD:                                       ; CODE XREF: FNDDBG\_47BEAE+19 p  
SDPI:0047BECD                 pop     eax  
SDPI:0047BECE                 add     eax, 31D0h  
SDPI:0047BED3                 call    loc\_47BED9  
SDPI:0047BED8                 nop  
SDPI:0047BED9  
SDPI:0047BED9 loc\_47BED9:                                       ; CODE XREF: FNDDBG\_47BEAE+25 p  
SDPI:0047BED9                 pop     ecx  
SDPI:0047BEDA                 add     ecx, 3271h  
SDPI:0047BEE0                 push    0  
SDPI:0047BEE2                 push    ecx  
SDPI:0047BEE3                 push    eax  
SDPI:0047BEE4                 push    0  
SDPI:0047BEE6                 call    loc\_47BEEC  
SDPI:0047BEEB                 nop  
SDPI:0047BEEC  
SDPI:0047BEEC loc\_47BEEC:                                       ; CODE XREF: FNDDBG\_47BEAE+38 p  
SDPI:0047BEEC                 pop     eax  
SDPI:0047BEED                 add     eax, 11h  
SDPI:0047BEF2                 push    eax  
SDPI:0047BEF3                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047BEF3                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047BEF3                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047BEF3 ; ----------------------------------------------------------------------------  
SDPI:0047BEF8                 dd 90909090h  
SDPI:0047BEFC ; ----------------------------------------------------------------------------  
SDPI:0047BEFC                 push    7  
SDPI:0047BEFE                 call    loc\_47BF04  
SDPI:0047BF03                 nop  
SDPI:0047BF04  
SDPI:0047BF04 loc\_47BF04:                                       ; CODE XREF: FNDDBG\_47BEAE+50 p  
SDPI:0047BF04                 pop     eax  
SDPI:0047BF05                 add     eax, 11h  
SDPI:0047BF0A                 push    eax  
SDPI:0047BF0B                 jmp     ExitProcess  
SDPI:0047BF0B ; ----------------------------------------------------------------------------  
SDPI:0047BF10                 dd 90909090h  
SDPI:0047BF10 FNDDBG\_47BEAE   endp  
SDPI:0047BF10  
SDPI:0047BF14  
SDPI:0047BF10  
SDPI:0047BF14  
SDPI:0047BF14 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047BF14  
SDPI:0047BF14  
SDPI:0047BF14 FNDDBG\_47BF14   proc near                         ; CODE XREF: SDPI:0047BE93 p  
SDPI:0047BF14                 nop  
SDPI:0047BF15                 nop  
SDPI:0047BF16                 nop  
SDPI:0047BF17                 nop  
SDPI:0047BF18                 nop  
SDPI:0047BF19                 call    loc\_47BF1F  
SDPI:0047BF1E                 nop  
SDPI:0047BF1F  
SDPI:0047BF1F loc\_47BF1F:                                       ; CODE XREF: FNDDBG\_47BF14+5 p  
SDPI:0047BF1F                 pop     eax  
SDPI:0047BF20                 add     eax, 5Eh  
SDPI:0047BF25                 mov     edx, eax  
SDPI:0047BF27                 add     edx, 32h  
SDPI:0047BF2A                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047BF2A                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047BF2A                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047BF2A                                                   ; 用来解密  
SDPI:0047BF2F                 call    loc\_47BF35  
SDPI:0047BF34                 nop  
SDPI:0047BF35  
SDPI:0047BF35 loc\_47BF35:                                       ; CODE XREF: FNDDBG\_47BF14+1B p  
SDPI:0047BF35                 pop     eax  
SDPI:0047BF36                 add     eax, 3168h  
SDPI:0047BF3B                 call    loc\_47BF41  
SDPI:0047BF40                 nop  
SDPI:0047BF41  
SDPI:0047BF41 loc\_47BF41:                                       ; CODE XREF: FNDDBG\_47BF14+27 p  
SDPI:0047BF41                 pop     ecx  
SDPI:0047BF42                 add     ecx, 3209h  
SDPI:0047BF48                 push    0  
SDPI:0047BF4A                 push    ecx  
SDPI:0047BF4B                 push    eax  
SDPI:0047BF4C                 push    0  
SDPI:0047BF4E                 call    loc\_47BF54  
SDPI:0047BF53                 nop  
SDPI:0047BF54  
SDPI:0047BF54 loc\_47BF54:                                       ; CODE XREF: FNDDBG\_47BF14+3A p  
SDPI:0047BF54                 pop     eax  
SDPI:0047BF55                 add     eax, 11h  
SDPI:0047BF5A                 push    eax  
SDPI:0047BF5B                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047BF5B                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047BF5B                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047BF5B ; ----------------------------------------------------------------------------  
SDPI:0047BF60                 dd 90909090h  
SDPI:0047BF64 ; ----------------------------------------------------------------------------  
SDPI:0047BF64                 push    7  
SDPI:0047BF66                 call    loc\_47BF6C  
SDPI:0047BF6B                 nop  
SDPI:0047BF6C  
SDPI:0047BF6C loc\_47BF6C:                                       ; CODE XREF: FNDDBG\_47BF14+52 p  
SDPI:0047BF6C                 pop     eax  
SDPI:0047BF6D                 add     eax, 11h  
SDPI:0047BF72                 push    eax  
SDPI:0047BF73                 jmp     ExitProcess  
SDPI:0047BF73 ; ----------------------------------------------------------------------------  
SDPI:0047BF78 JUNK\_47BF78     db '悙悙',0,10h,'@',0,'鄩?,6  
SDPI:0047BF78 FNDDBG\_47BF14   endp  
SDPI:0047BF78  
SDPI:0047BF84 ; ----------------------------------------------------------------------------  
SDPI:0047BF84  
SDPI:0047BF84 INT3\_47BF84:                                      ; CODE XREF: SDPI:0047BE98 p  
SDPI:0047BF84                 call    loc\_47BF8A  
SDPI:0047BF89                 nop  
SDPI:0047BF8A  
SDPI:0047BF8A loc\_47BF8A:                                       ; CODE XREF: SDPI:INT3\_47BF84 p  
SDPI:0047BF8A                 pop     edi  
SDPI:0047BF8B                 add     edi, 0FFFFFF07h  
SDPI:0047BF91                 mov     \[edi\], ebx  
SDPI:0047BF93                 mov     \[edi+4\], edx  
SDPI:0047BF96                 pop     eax  
SDPI:0047BF97                 call    loc\_47BF9D  
SDPI:0047BF9C                 nop  
SDPI:0047BF9D  
SDPI:0047BF9D loc\_47BF9D:                                       ; CODE XREF: SDPI:0047BF97 p  
SDPI:0047BF9D                 pop     eax  
SDPI:0047BF9E                 add     eax, 124h  
SDPI:0047BFA3                 push    eax  
SDPI:0047BFA4                 xor     eax, eax  
SDPI:0047BFA6                 push    dword ptr fs:\[eax\]  
SDPI:0047BFA9                 mov     fs:\[eax\], esp  
SDPI:0047BFAC                 mov     ebp, 300EF1D3h  
SDPI:0047BFB1                 add     ebp, 12345678h  
SDPI:0047BFB7                 mov     ax, 17h  
SDPI:0047BFBB                 sub     ax, 13h  
SDPI:0047BFBB ; ----------------------------------------------------------------------------  
SDPI:0047BFBF JUNK\_47BFBF     db '|',7,'?,7,0,10h,'@',0,'鑤鱮',12h,'s',10h,0,10h,'@',0,'?,3,'?  
SDPI:0047BFBF                 db 'r',1Bh,'s',19h,0,10h,'@',0,'鑖淧 3?豻扈',5,0,0,0,0,10h,'@',0  
SDPI:0047BFBF                 db '鑈 Xf 潗悙悙'  
SDPI:0047BFFA ; ----------------------------------------------------------------------------  
SDPI:0047BFFA                 int     3                         ; Trap to Debugger  
SDPI:0047BFFB                 nop  
SDPI:0047BFFC                 cmp     al, 4                     ; SEH 句柄:0047C0C0  
SDPI:0047BFFE                 jz      short Done\_47C071  
SDPI:0047C000  
SDPI:0047C000 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047C000  
SDPI:0047C000  
SDPI:0047C000 FNDDBG\_47C000   proc near                         ; CODE XREF: SDPI:0047C08A j  
SDPI:0047C000                                                   ; SDPI:0047C0A2 j ...  
SDPI:0047C000                 nop  
SDPI:0047C001                 nop  
SDPI:0047C002                 nop  
SDPI:0047C003                 nop  
SDPI:0047C004                 nop  
SDPI:0047C005                 call    loc\_47C00B  
SDPI:0047C00A                 nop  
SDPI:0047C00B  
SDPI:0047C00B loc\_47C00B:                                       ; CODE XREF: FNDDBG\_47C000+5 p  
SDPI:0047C00B                 pop     eax  
SDPI:0047C00C                 add     eax, 5Eh  
SDPI:0047C011                 mov     edx, eax  
SDPI:0047C013                 add     edx, 32h  
SDPI:0047C016                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047C016                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047C016                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047C016                                                   ; 用来解密  
SDPI:0047C01B                 call    loc\_47C021  
SDPI:0047C020                 nop  
SDPI:0047C021  
SDPI:0047C021 loc\_47C021:                                       ; CODE XREF: FNDDBG\_47C000+1B p  
SDPI:0047C021                 pop     eax  
SDPI:0047C022                 add     eax, 307Ch  
SDPI:0047C027                 call    loc\_47C02D  
SDPI:0047C02C                 nop  
SDPI:0047C02D  
SDPI:0047C02D loc\_47C02D:                                       ; CODE XREF: FNDDBG\_47C000+27 p  
SDPI:0047C02D                 pop     ecx  
SDPI:0047C02E                 add     ecx, 311Dh  
SDPI:0047C034                 push    0  
SDPI:0047C036                 push    ecx  
SDPI:0047C037                 push    eax  
SDPI:0047C038                 push    0  
SDPI:0047C03A                 call    loc\_47C040  
SDPI:0047C03F                 nop  
SDPI:0047C040  
SDPI:0047C040 loc\_47C040:                                       ; CODE XREF: FNDDBG\_47C000+3A p  
SDPI:0047C040                 pop     eax  
SDPI:0047C041                 add     eax, 11h  
SDPI:0047C046                 push    eax  
SDPI:0047C047                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047C047                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047C047                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047C047 ; ----------------------------------------------------------------------------  
SDPI:0047C04C                 dd 90909090h  
SDPI:0047C050 ; ----------------------------------------------------------------------------  
SDPI:0047C050                 push    7  
SDPI:0047C052                 call    loc\_47C058  
SDPI:0047C057                 nop  
SDPI:0047C058  
SDPI:0047C058 loc\_47C058:                                       ; CODE XREF: FNDDBG\_47C000+52 p  
SDPI:0047C058                 pop     eax  
SDPI:0047C059                 add     eax, 11h  
SDPI:0047C05E                 push    eax  
SDPI:0047C05F                 jmp     ExitProcess  
SDPI:0047C05F ; ----------------------------------------------------------------------------  
SDPI:0047C064 JUNK\_47C064     db '悙悙?,0,10h,'@',0,'皦?,4  
SDPI:0047C064 FNDDBG\_47C000   endp  
SDPI:0047C064  
SDPI:0047C071 ; ----------------------------------------------------------------------------  
SDPI:0047C071  
SDPI:0047C071 Done\_47C071:                                      ; CODE XREF: SDPI:0047BFFE j  
SDPI:0047C071                 pop     large dword ptr fs:0  
SDPI:0047C078                 add     esp, 4  
SDPI:0047C07B                 call    loc\_47C081  
SDPI:0047C080                 nop  
SDPI:0047C081  
SDPI:0047C081 loc\_47C081:                                       ; CODE XREF: SDPI:0047C07B p  
SDPI:0047C081                 pop     eax  
SDPI:0047C082                 add     eax, 0FFFFFE1Dh  
SDPI:0047C087                 cmp     byte ptr \[eax\], 0E9h  
SDPI:0047C08A                 jnz     FNDDBG\_47C000  
SDPI:0047C090                 mov     byte ptr \[eax\], 0E8h  
SDPI:0047C093                 rdtsc  
SDPI:0047C095                 mov     ecx, eax  
SDPI:0047C097                 mov     ebx, edx  
SDPI:0047C099                 rdtsc  
SDPI:0047C09B                 sub     eax, ecx  
SDPI:0047C09D                 sbb     edx, ebx  
SDPI:0047C09F                 cmp     edx, 0  
SDPI:0047C0A2                 jnz     FNDDBG\_47C000  
SDPI:0047C0A8                 cmp     eax, 30000000h  
SDPI:0047C0AD                 ja      FNDDBG\_47C000  
SDPI:0047C0B3                 jz      short PASS\_47C0FE  
SDPI:0047C0B5                 jnz     short PASS\_47C0FE  
SDPI:0047C0B5 ; ----------------------------------------------------------------------------  
SDPI:0047C0B7                 dd 401000E8h  
SDPI:0047C0BB                 dd 9C89B000h  
SDPI:0047C0BF                 db 4  
SDPI:0047C0C0 ; ----------------------------------------------------------------------------  
SDPI:0047C0C0                 mov     eax, \[esp+4\]              ; SEH 处理处  
SDPI:0047C0C4                 mov     ecx, \[esp+0Ch\]  
SDPI:0047C0C8                 inc     dword ptr \[ecx+0B8h\]  
SDPI:0047C0CE                 mov     eax, \[eax\]  
SDPI:0047C0D0                 sub     eax, 80000003h  
SDPI:0047C0D5                 jnz     short locret\_47C0FD  
SDPI:0047C0D7                 call    sub\_47C0DD  
SDPI:0047C0DC                 nop  
SDPI:0047C0DD  
SDPI:0047C0DD ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047C0DD  
SDPI:0047C0DD  
SDPI:0047C0DD sub\_47C0DD      proc near                         ; CODE XREF: SDPI:0047C0D7 p  
SDPI:0047C0DD                 pop     eax  
SDPI:0047C0DE                 add     eax, 0FFFFFDC1h  
SDPI:0047C0E3                 cmp     byte ptr \[eax\], 0E8h  
SDPI:0047C0E6                 jnz     FNDDBG\_47C000  
SDPI:0047C0EC                 mov     byte ptr \[eax\], 0E9h  
SDPI:0047C0EF                 xor     eax, eax  
SDPI:0047C0F1                 mov     \[ecx+4\], eax  
SDPI:0047C0F4                 mov     \[ecx+8\], eax  
SDPI:0047C0F7                 mov     \[ecx+0Ch\], eax  
SDPI:0047C0FA                 mov     \[ecx+10h\], eax  
SDPI:0047C0FD  
SDPI:0047C0FD locret\_47C0FD:                                    ; CODE XREF: SDPI:0047C0D5 j  
SDPI:0047C0FD                 retn  
SDPI:0047C0FD sub\_47C0DD      endp ; sp =  4  
SDPI:0047C0FD  
SDPI:0047C0FE ; ----------------------------------------------------------------------------  
SDPI:0047C0FE  
SDPI:0047C0FE PASS\_47C0FE:                                      ; CODE XREF: SDPI:0047C0B3 j  
SDPI:0047C0FE                                                   ; SDPI:0047C0B5 j  
SDPI:0047C0FE                 pop     eax  
SDPI:0047C0FF                 call    loc\_47C2FF  
SDPI:0047C0FF ; ----------------------------------------------------------------------------  
SDPI:0047C104                 dd 90909090h  
SDPI:0047C108                 dd 90909090h  
SDPI:0047C10C ; ----------------------------------------------------------------------------  
SDPI:0047C10C  
SDPI:0047C10C loc\_47C10C:                                       ; CODE XREF: SDPI:0047C328 p  
SDPI:0047C10C                 pop     ebp  
SDPI:0047C10D                 pop     eax  
SDPI:0047C10E                 jmp     loc\_47C32D  
SDPI:0047C113 ; ----------------------------------------------------------------------------  
SDPI:0047C113                 call    loc\_47C119  
SDPI:0047C118                 nop  
SDPI:0047C119  
SDPI:0047C119 loc\_47C119:                                       ; CODE XREF: SDPI:0047C113 p  
SDPI:0047C119                 pop     eax  
SDPI:0047C11A                 add     eax, 312h                 ; EAX==0047C42A  
SDPI:0047C11F                 call    loc\_47C125  
SDPI:0047C124                 nop  
SDPI:0047C125  
SDPI:0047C125 loc\_47C125:                                       ; CODE XREF: SDPI:0047C11F p  
SDPI:0047C125                 pop     edx  
SDPI:0047C126                 add     edx, 11EEh                ; EDX==0047D312  
SDPI:0047C12C                 call    Crypt\_Decrypt\_CODE        ; 把 47C42A 到 47D312 处的代码加密回去  
SDPI:0047C131                 mov     ecx, 0FFFFFF00h  
SDPI:0047C136                 push    fs  
SDPI:0047C138                 nop  
SDPI:0047C139                 nop  
SDPI:0047C13A                 nop  
SDPI:0047C13B                 nop  
SDPI:0047C13C                 nop  
SDPI:0047C13D                 nop  
SDPI:0047C13E                 nop  
SDPI:0047C13F                 nop  
SDPI:0047C140                 nop  
SDPI:0047C141                 nop  
SDPI:0047C142                 nop  
SDPI:0047C143                 nop  
SDPI:0047C144                 pushfw  
SDPI:0047C146                 push    eax  
SDPI:0047C147                 mov     eax, ebx  
SDPI:0047C149                 push    ebx  
SDPI:0047C14A                 mov     eax, ecx  
SDPI:0047C14C                 push    eax  
SDPI:0047C14D                 add     eax, edx  
SDPI:0047C14F                 mov     ebx, eax  
SDPI:0047C151                 push    ebx  
SDPI:0047C152                 pop     eax  
SDPI:0047C153                 push    edx  
SDPI:0047C154                 call    loc\_47C161  
SDPI:0047C159                 nop  
SDPI:0047C15A                 nop  
SDPI:0047C15B                 nop  
SDPI:0047C15C                 nop  
SDPI:0047C15D                 nop  
SDPI:0047C15E                 nop  
SDPI:0047C15F                 nop  
SDPI:0047C160                 nop  
SDPI:0047C161  
SDPI:0047C161 loc\_47C161:                                       ; CODE XREF: SDPI:0047C154 p  
SDPI:0047C161                 pop     eax  
SDPI:0047C162                 call    loc\_47C168  
SDPI:0047C167                 nop  
SDPI:0047C168  
SDPI:0047C168 loc\_47C168:                                       ; CODE XREF: SDPI:0047C162 p  
SDPI:0047C168                 pop     eax  
SDPI:0047C169                 add     eax, 11h  
SDPI:0047C16E                 push    eax  
SDPI:0047C16F                 jmp     GetTickCount  
SDPI:0047C16F ; ----------------------------------------------------------------------------  
SDPI:0047C174                 dd 90909090h  
SDPI:0047C178 ; ----------------------------------------------------------------------------  
SDPI:0047C178                 push    eax  
SDPI:0047C179                 mov     eax, edx  
SDPI:0047C17B                 push    eax  
SDPI:0047C17C                 call    loc\_47C182  
SDPI:0047C181                 nop  
SDPI:0047C182  
SDPI:0047C182 loc\_47C182:                                       ; CODE XREF: SDPI:0047C17C p  
SDPI:0047C182                 pop     edx  
SDPI:0047C183                 add     edx, 52h  
SDPI:0047C189                 push    edx  
SDPI:0047C18A                 add     edx, 40318Ah  
SDPI:0047C190                 push    edx  
SDPI:0047C191                 jo      short loc\_47C1E6  
SDPI:0047C193                 jno     short loc\_47C1E6  
SDPI:0047C195  
SDPI:0047C195 loc\_47C195:                                       ; CODE XREF: SDPI:0047C1D9 p  
SDPI:0047C195                 pop     eax  
SDPI:0047C196                 pop     ebx  
SDPI:0047C197                 call    loc\_47C19D  
SDPI:0047C19C                 nop  
SDPI:0047C19D  
SDPI:0047C19D loc\_47C19D:                                       ; CODE XREF: SDPI:0047C197 p  
SDPI:0047C19D                 pop     eax  
SDPI:0047C19E                 add     eax, 11h  
SDPI:0047C1A3                 push    eax  
SDPI:0047C1A4                 jmp     GetTickCount  
SDPI:0047C1A4 ; ----------------------------------------------------------------------------  
SDPI:0047C1A9                 dd 90909090h  
SDPI:0047C1AD ; ----------------------------------------------------------------------------  
SDPI:0047C1AD                 pop     ebx  
SDPI:0047C1AE                 add     ebx, 1F4h  
SDPI:0047C1B4                 sub     ebx, eax  
SDPI:0047C1B6                 js      short OVER\_47C1FA  
SDPI:0047C1B8                 call    loc\_47C1BE  
SDPI:0047C1BD                 nop  
SDPI:0047C1BE  
SDPI:0047C1BE loc\_47C1BE:                                       ; CODE XREF: SDPI:0047C1B8 p  
SDPI:0047C1BE                 pop     ebx  
SDPI:0047C1BF                 add     ebx, 0A5h  
SDPI:0047C1C5                 push    ebx  
SDPI:0047C1C6                 call    loc\_47C1F0  
SDPI:0047C1C6 ; ----------------------------------------------------------------------------  
SDPI:0047C1CB DB90\_471cb      db 8 dup(90h)  
SDPI:0047C1D3 ; ----------------------------------------------------------------------------  
SDPI:0047C1D3                 pop     eax  
SDPI:0047C1D4                 mov     edx, eax  
SDPI:0047C1D6                 mov     eax, ebx  
SDPI:0047C1D8                 push    eax  
SDPI:0047C1D9                 call    loc\_47C195  
SDPI:0047C1D9 ; ----------------------------------------------------------------------------  
SDPI:0047C1DE                 db 8 dup(90h)  
SDPI:0047C1E6 ; ----------------------------------------------------------------------------  
SDPI:0047C1E6  
SDPI:0047C1E6 loc\_47C1E6:                                       ; CODE XREF: SDPI:0047C191 j  
SDPI:0047C1E6                                                   ; SDPI:0047C193 j  
SDPI:0047C1E6                 pop     eax  
SDPI:0047C1E7                 retn  
SDPI:0047C1E7 ; ----------------------------------------------------------------------------  
SDPI:0047C1E8                 db 8 dup(90h)  
SDPI:0047C1F0 ; ----------------------------------------------------------------------------  
SDPI:0047C1F0  
SDPI:0047C1F0 loc\_47C1F0:                                       ; CODE XREF: SDPI:0047C1C6 p  
SDPI:0047C1F0                 pop     edx  
SDPI:0047C1F1                 retn  
SDPI:0047C1F1 ; ----------------------------------------------------------------------------  
SDPI:0047C1F2                 db 8 dup(90h)  
SDPI:0047C1FA  
SDPI:0047C1FA ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047C1FA  
SDPI:0047C1FA  
SDPI:0047C1FA OVER\_47C1FA     proc near                         ; CODE XREF: SDPI:0047C1B6 j  
SDPI:0047C1FA                 nop  
SDPI:0047C1FB                 nop  
SDPI:0047C1FC                 nop  
SDPI:0047C1FD                 nop  
SDPI:0047C1FE                 nop  
SDPI:0047C1FF                 call    loc\_47C205  
SDPI:0047C204                 nop  
SDPI:0047C205  
SDPI:0047C205 loc\_47C205:                                       ; CODE XREF: OVER\_47C1FA+5 p  
SDPI:0047C205                 pop     eax  
SDPI:0047C206                 add     eax, 5Eh  
SDPI:0047C20B                 mov     edx, eax  
SDPI:0047C20D                 add     edx, 32h  
SDPI:0047C210                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047C210                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047C210                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047C210                                                   ; 用来解密  
SDPI:0047C215                 call    loc\_47C21B  
SDPI:0047C21A                 nop  
SDPI:0047C21B  
SDPI:0047C21B loc\_47C21B:                                       ; CODE XREF: OVER\_47C1FA+1B p  
SDPI:0047C21B                 pop     eax  
SDPI:0047C21C                 add     eax, 2E82h  
SDPI:0047C221                 call    loc\_47C227  
SDPI:0047C226                 nop  
SDPI:0047C227  
SDPI:0047C227 loc\_47C227:                                       ; CODE XREF: OVER\_47C1FA+27 p  
SDPI:0047C227                 pop     ecx  
SDPI:0047C228                 add     ecx, 2F23h  
SDPI:0047C22E                 push    0  
SDPI:0047C230                 push    ecx  
SDPI:0047C231                 push    eax  
SDPI:0047C232                 push    0  
SDPI:0047C234                 call    loc\_47C23A  
SDPI:0047C239                 nop  
SDPI:0047C23A  
SDPI:0047C23A loc\_47C23A:                                       ; CODE XREF: OVER\_47C1FA+3A p  
SDPI:0047C23A                 pop     eax  
SDPI:0047C23B                 add     eax, 11h  
SDPI:0047C240                 push    eax  
SDPI:0047C241                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047C241                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047C241                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047C241 ; ----------------------------------------------------------------------------  
SDPI:0047C246                 db 4 dup(90h)  
SDPI:0047C24A ; ----------------------------------------------------------------------------  
SDPI:0047C24A                 push    7  
SDPI:0047C24C                 call    loc\_47C252  
SDPI:0047C251                 nop  
SDPI:0047C252  
SDPI:0047C252 loc\_47C252:                                       ; CODE XREF: OVER\_47C1FA+52 p  
SDPI:0047C252                 pop     eax  
SDPI:0047C253                 add     eax, 11h  
SDPI:0047C258                 push    eax  
SDPI:0047C259                 jmp     ExitProcess  
SDPI:0047C259 ; ----------------------------------------------------------------------------  
SDPI:0047C25E                 db 4 dup(90h)  
SDPI:0047C25E OVER\_47C1FA     endp  
SDPI:0047C25E  
SDPI:0047C262 ; ----------------------------------------------------------------------------  
SDPI:0047C262                 pop     edx  
SDPI:0047C263                 mov     eax, ecx  
SDPI:0047C265                 add     eax, edx  
SDPI:0047C267                 inc     ecx  
SDPI:0047C268                 push    eax  
SDPI:0047C269                 inc     ecx  
SDPI:0047C26A                 pop     ebx  
SDPI:0047C26B                 pop     ecx  
SDPI:0047C26C                 push    eax  
SDPI:0047C26D                 sub     eax, 8  
SDPI:0047C270                 pop     ebx  
SDPI:0047C271                 pop     ebx  
SDPI:0047C272                 inc     eax  
SDPI:0047C273                 add     eax, ebx  
SDPI:0047C275                 pop     eax  
SDPI:0047C276                 pushfw  
SDPI:0047C278                 popfw  
SDPI:0047C27A                 popfw  
SDPI:0047C27C                 pop     es  
SDPI:0047C27D                 mov     eax, 12345678h  
SDPI:0047C282                 push    eax  
SDPI:0047C283                 call    loc\_47C289  
SDPI:0047C288                 nop  
SDPI:0047C289  
SDPI:0047C289 loc\_47C289:                                       ; CODE XREF: SDPI:0047C283 p  
SDPI:0047C289                 pop     eax  
SDPI:0047C28A                 add     eax, 12Ch  
SDPI:0047C28F                 push    eax  
SDPI:0047C290                 pop     ebx  
SDPI:0047C291                 add     eax, 12h  
SDPI:0047C294                 pop     edx  
SDPI:0047C295                 add     eax, edx  
SDPI:0047C297                 mov     edx, eax  
SDPI:0047C299                 push    ebx  
SDPI:0047C29A                 mov     ebx, es:\[ecx+100h\]  
SDPI:0047C2A1                 push    ebx  
SDPI:0047C2A2                 mov     eax, esp  
SDPI:0047C2A4                 mov     ebx, eax  
SDPI:0047C2A6                 push    ebx  
SDPI:0047C2A7                 pop     edx  
SDPI:0047C2A8                 mov     es:\[ecx+100h\], eax  
SDPI:0047C2AF                 xor     eax, eax  
SDPI:0047C2AF ; ----------------------------------------------------------------------------  
SDPI:0047C2B1                 db 38h dup(90h)  
SDPI:0047C2E9 ; ----------------------------------------------------------------------------  
SDPI:0047C2E9                 int     3                         ; Trap to Debugger  
SDPI:0047C2EA                 nop                               ; SEH 句柄:0047C3B4  
SDPI:0047C2EB                 xor     eax, eax  
SDPI:0047C2ED                 mov     dword ptr \[eax\], 4033EDh  
SDPI:0047C2F3                 jp      short loc\_47C2FF  
SDPI:0047C2F5                 jnp     short loc\_47C2FF  
SDPI:0047C2F7                 nop  
SDPI:0047C2F8                 nop  
SDPI:0047C2F9                 nop  
SDPI:0047C2FA                 nop  
SDPI:0047C2FB                 nop  
SDPI:0047C2FC                 nop  
SDPI:0047C2FD                 nop  
SDPI:0047C2FE                 nop  
SDPI:0047C2FF  
SDPI:0047C2FF loc\_47C2FF:                                       ; CODE XREF: SDPI:0047C0FF p  
SDPI:0047C2FF                                                   ; SDPI:0047C2F3 j ...  
SDPI:0047C2FF                 call    loc\_47C305  
SDPI:0047C304                 nop  
SDPI:0047C305  
SDPI:0047C305 loc\_47C305:                                       ; CODE XREF: SDPI:loc\_47C2FF p  
SDPI:0047C305                 pop     eax  
SDPI:0047C306                 add     eax, 11h  
SDPI:0047C30B                 push    eax  
SDPI:0047C30C                 jmp     GetTickCount  
SDPI:0047C30C ; ----------------------------------------------------------------------------  
SDPI:0047C311                 db 4 dup(90h)  
SDPI:0047C315 ; ----------------------------------------------------------------------------  
SDPI:0047C315                 call    loc\_47C31B  
SDPI:0047C31A                 nop  
SDPI:0047C31B  
SDPI:0047C31B loc\_47C31B:                                       ; CODE XREF: SDPI:0047C315 p  
SDPI:0047C31B                 pop     edx  
SDPI:0047C31C                 add     edx, 0FFFFFB09h  
SDPI:0047C322                 mov     \[edx\], eax  
SDPI:0047C324                 pop     ebp  
SDPI:0047C325                 add     eax, edx  
SDPI:0047C327                 push    eax  
SDPI:0047C328                 call    loc\_47C10C  
SDPI:0047C32D  
SDPI:0047C32D loc\_47C32D:                                       ; CODE XREF: SDPI:0047C10E j  
SDPI:0047C32D                 call    loc\_47C333  
SDPI:0047C332                 nop  
SDPI:0047C333  
SDPI:0047C333 loc\_47C333:                                       ; CODE XREF: SDPI:loc\_47C32D p  
SDPI:0047C333                 pop     edx  
SDPI:0047C334                 add     edx, 0FFFFFDE1h  
SDPI:0047C33A                 add     edx, eax  
SDPI:0047C33C                 push    edx  
SDPI:0047C33D                 pop     ecx  
SDPI:0047C33E                 sub     ecx, eax  
SDPI:0047C340                 push    ecx  
SDPI:0047C341                 retn    4  
SDPI:0047C344  
SDPI:0047C344 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047C344  
SDPI:0047C344  
SDPI:0047C344 OVER\_47C344     proc near                         ; CODE XREF: SDPI:0047C3E7 j  
SDPI:0047C344                                                   ; SDPI:0047C3EF j ...  
SDPI:0047C344                 nop  
SDPI:0047C345                 nop  
SDPI:0047C346                 nop  
SDPI:0047C347                 nop  
SDPI:0047C348                 nop  
SDPI:0047C349                 call    loc\_47C34F  
SDPI:0047C34E                 nop  
SDPI:0047C34F  
SDPI:0047C34F loc\_47C34F:                                       ; CODE XREF: OVER\_47C344+5 p  
SDPI:0047C34F                 pop     eax  
SDPI:0047C350                 add     eax, 5Eh  
SDPI:0047C355                 mov     edx, eax  
SDPI:0047C357                 add     edx, 32h  
SDPI:0047C35A                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047C35A                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047C35A                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047C35A                                                   ; 用来解密  
SDPI:0047C35F                 call    loc\_47C365  
SDPI:0047C364                 nop  
SDPI:0047C365  
SDPI:0047C365 loc\_47C365:                                       ; CODE XREF: OVER\_47C344+1B p  
SDPI:0047C365                 pop     eax  
SDPI:0047C366                 add     eax, 2D38h  
SDPI:0047C36B                 call    loc\_47C371  
SDPI:0047C370                 nop  
SDPI:0047C371  
SDPI:0047C371 loc\_47C371:                                       ; CODE XREF: OVER\_47C344+27 p  
SDPI:0047C371                 pop     ecx  
SDPI:0047C372                 add     ecx, 2DD9h  
SDPI:0047C378                 push    0  
SDPI:0047C37A                 push    ecx  
SDPI:0047C37B                 push    eax  
SDPI:0047C37C                 push    0  
SDPI:0047C37E                 call    loc\_47C384  
SDPI:0047C383                 nop  
SDPI:0047C384  
SDPI:0047C384 loc\_47C384:                                       ; CODE XREF: OVER\_47C344+3A p  
SDPI:0047C384                 pop     eax  
SDPI:0047C385                 add     eax, 11h  
SDPI:0047C38A                 push    eax  
SDPI:0047C38B                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047C38B                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047C38B                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047C38B ; ----------------------------------------------------------------------------  
SDPI:0047C390                 db 4 dup(90h)  
SDPI:0047C394 ; ----------------------------------------------------------------------------  
SDPI:0047C394                 push    7  
SDPI:0047C396                 call    loc\_47C39C  
SDPI:0047C39B                 nop  
SDPI:0047C39C  
SDPI:0047C39C loc\_47C39C:                                       ; CODE XREF: OVER\_47C344+52 p  
SDPI:0047C39C                 pop     eax  
SDPI:0047C39D                 add     eax, 11h  
SDPI:0047C3A2                 push    eax  
SDPI:0047C3A3                 jmp     ExitProcess  
SDPI:0047C3A3 ; ----------------------------------------------------------------------------  
SDPI:0047C3A8                 db 0Ch dup(90h)  
SDPI:0047C3A8 OVER\_47C344     endp  
SDPI:0047C3A8  
SDPI:0047C3B4 ; ----------------------------------------------------------------------------  
SDPI:0047C3B4                 mov     esp, \[esp+8\]              ; SEH 处理  
SDPI:0047C3B8                 pop     large dword ptr fs:0  
SDPI:0047C3BF                 call    loc\_47C3C5  
SDPI:0047C3C4                 nop  
SDPI:0047C3C5  
SDPI:0047C3C5 loc\_47C3C5:                                       ; CODE XREF: SDPI:0047C3BF p  
SDPI:0047C3C5                 pop     eax  
SDPI:0047C3C6                 add     eax, 11h  
SDPI:0047C3CB                 push    eax  
SDPI:0047C3CC                 jmp     GetTickCount  
SDPI:0047C3CC ; ----------------------------------------------------------------------------  
SDPI:0047C3D1                 db 4 dup(90h)  
SDPI:0047C3D5 ; ----------------------------------------------------------------------------  
SDPI:0047C3D5                 call    loc\_47C3DB  
SDPI:0047C3DA                 nop  
SDPI:0047C3DB  
SDPI:0047C3DB loc\_47C3DB:                                       ; CODE XREF: SDPI:0047C3D5 p  
SDPI:0047C3DB                 pop     edx  
SDPI:0047C3DC                 add     edx, 0FFFFFA49h  
SDPI:0047C3E2                 mov     ecx, \[edx\]  
SDPI:0047C3E4                 cmp     ecx, 0  
SDPI:0047C3E7                 jz      OVER\_47C344  
SDPI:0047C3ED                 sub     eax, ecx  
SDPI:0047C3EF                 js      OVER\_47C344  
SDPI:0047C3F5                 sub     eax, 7D0h  
SDPI:0047C3FA                 jns     OVER\_47C344  
SDPI:0047C400                 mov     eax, 0E801276h  
SDPI:0047C405                 mov     \[edx\], eax  
SDPI:0047C407                 call    loc\_47C40D  
SDPI:0047C40C                 nop  
SDPI:0047C40D  
SDPI:0047C40D loc\_47C40D:                                       ; CODE XREF: SDPI:0047C407 p  
SDPI:0047C40D                 pop     edx  
SDPI:0047C40E                 add     edx, 0F06h                ; 上面加密回去的代码，再解密出来  
SDPI:0047C414                 call    loc\_47C41A                ; 解密结束地址:0047D312  
SDPI:0047C419                 nop  
SDPI:0047C41A  
SDPI:0047C41A loc\_47C41A:                                       ; CODE XREF: SDPI:0047C414 p  
SDPI:0047C41A                 pop     eax  
SDPI:0047C41B                 add     eax, 0FFFFDD7Fh  
SDPI:0047C420                 mov     ecx, 10h  
SDPI:0047C425                 call    De\_Code                   ; invoke De\_Code,ecx(counter),eax(key),edx(end)  
SDPI:0047C425                                                   ; 用于解密代码，解密起始地址就是 call 进来  
SDPI:0047C425                                                   ; 的下一行代码地址  
SDPI:0047C42A                 jmp     short loc\_47C4A2  
SDPI:0047C42A ; ----------------------------------------------------------------------------  
SDPI:0047C42C                 db 0Eh dup(90h)  
SDPI:0047C43A  
SDPI:0047C43A ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047C43A  
SDPI:0047C43A  
SDPI:0047C43A OVER\_47C43A     proc near                         ; CODE XREF: SDPI:0047C4C7 j  
SDPI:0047C43A                                                   ; SDPI:0047CB8B j  
SDPI:0047C43A                 nop  
SDPI:0047C43B                 nop  
SDPI:0047C43C                 nop  
SDPI:0047C43D                 nop  
SDPI:0047C43E                 nop  
SDPI:0047C43F                 call    loc\_47C445  
SDPI:0047C444                 nop  
SDPI:0047C445  
SDPI:0047C445 loc\_47C445:                                       ; CODE XREF: OVER\_47C43A+5 p  
SDPI:0047C445                 pop     eax  
SDPI:0047C446                 add     eax, 5Eh  
SDPI:0047C44B                 mov     edx, eax  
SDPI:0047C44D                 add     edx, 32h  
SDPI:0047C450                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047C450                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047C450                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047C450                                                   ; 用来解密  
SDPI:0047C455                 call    loc\_47C45B  
SDPI:0047C45A                 nop  
SDPI:0047C45B  
SDPI:0047C45B loc\_47C45B:                                       ; CODE XREF: OVER\_47C43A+1B p  
SDPI:0047C45B                 pop     eax  
SDPI:0047C45C                 add     eax, 2C42h  
SDPI:0047C461                 call    loc\_47C467  
SDPI:0047C466                 nop  
SDPI:0047C467  
SDPI:0047C467 loc\_47C467:                                       ; CODE XREF: OVER\_47C43A+27 p  
SDPI:0047C467                 pop     ecx  
SDPI:0047C468                 add     ecx, 2CE3h  
SDPI:0047C46E                 push    0  
SDPI:0047C470                 push    ecx  
SDPI:0047C471                 push    eax  
SDPI:0047C472                 push    0  
SDPI:0047C474                 call    loc\_47C47A  
SDPI:0047C479                 nop  
SDPI:0047C47A  
SDPI:0047C47A loc\_47C47A:                                       ; CODE XREF: OVER\_47C43A+3A p  
SDPI:0047C47A                 pop     eax  
SDPI:0047C47B                 add     eax, 11h  
SDPI:0047C480                 push    eax  
SDPI:0047C481                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047C481                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047C481                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047C481 ; ----------------------------------------------------------------------------  
SDPI:0047C486                 db 4 dup(90h)  
SDPI:0047C48A ; ----------------------------------------------------------------------------  
SDPI:0047C48A                 push    7  
SDPI:0047C48C                 call    loc\_47C492  
SDPI:0047C491                 nop  
SDPI:0047C492  
SDPI:0047C492 loc\_47C492:                                       ; CODE XREF: OVER\_47C43A+52 p  
SDPI:0047C492                 pop     eax  
SDPI:0047C493                 add     eax, 11h  
SDPI:0047C498                 push    eax  
SDPI:0047C499                 jmp     ExitProcess  
SDPI:0047C499 ; ----------------------------------------------------------------------------  
SDPI:0047C49E                 db 4 dup(90h)  
SDPI:0047C49E OVER\_47C43A     endp  
SDPI:0047C49E  
SDPI:0047C4A2 ; ----------------------------------------------------------------------------  
SDPI:0047C4A2  
SDPI:0047C4A2 loc\_47C4A2:                                       ; CODE XREF: SDPI:0047C42A j  
SDPI:0047C4A2                 call    sub\_47C4A8  
SDPI:0047C4A7                 nop  
SDPI:0047C4A8  
SDPI:0047C4A8 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047C4A8  
SDPI:0047C4A8  
SDPI:0047C4A8 sub\_47C4A8      proc near                         ; CODE XREF: SDPI:loc\_47C4A2 p  
SDPI:0047C4A8                 pop     eax  
SDPI:0047C4A9                 add     eax, 11h  
SDPI:0047C4AE                 push    eax  
SDPI:0047C4AF                 jmp     Get\_Version  
SDPI:0047C4AF sub\_47C4A8      endp  
SDPI:0047C4AF  
SDPI:0047C4AF ; ----------------------------------------------------------------------------  
SDPI:0047C4B4                 db 4 dup(90h)  
SDPI:0047C4B8 ; ----------------------------------------------------------------------------  
SDPI:0047C4B8                 call    loc\_47C4BE  
SDPI:0047C4BD                 nop  
SDPI:0047C4BE  
SDPI:0047C4BE loc\_47C4BE:                                       ; CODE XREF: SDPI:0047C4B8 p  
SDPI:0047C4BE                 pop     edx  
SDPI:0047C4BF                 add     edx, 0FFFFDB47h  
SDPI:0047C4C5                 cmp     eax, \[edx\]                ; 这里再来一次判断是否修改过  
SDPI:0047C4C5                                                   ; GetVersion 的返回值  
SDPI:0047C4C7                 jnz     OVER\_47C43A  
SDPI:0047C4CD                 sub     ebx, 40000000h  
SDPI:0047C4CD ; ----------------------------------------------------------------------------  
SDPI:0047C4D3                 db 36h dup(90h)  
SDPI:0047C509 ; ----------------------------------------------------------------------------  
SDPI:0047C509                 sub     eax, 80000000h  
SDPI:0047C509 ; ----------------------------------------------------------------------------  
SDPI:0047C50E                 db 30h dup(90h)  
SDPI:0047C53E ; ----------------------------------------------------------------------------  
SDPI:0047C53E                 jb      NotIsWin9x\_47D312         ; 判断系统是否为 winnt,如果是则跳  
SDPI:0047C53E ; --------------------------------------------------------------------  
  
&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&  
              Win9x 下的调试器检测部分  
&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&  
由于我自己没有 Win9x, 也不怎么了解 Win9x 下的  
调试器检测，因此下面也没有怎么进 anti-dbg 部分  
不过应该和以前公布出来的检测方式差不多  
SDPI:0047C553 ; ----------------------------------------------------------------------------  
SDPI:0047C553                 push    cs  
SDPI:0047C554                 push    eax  
SDPI:0047C555                 xor     eax, eax  
SDPI:0047C557                 call    loc\_47C55D  
SDPI:0047C55C                 nop  
SDPI:0047C55D  
SDPI:0047C55D loc\_47C55D:                                       ; CODE XREF: SDPI:0047C557 p  
SDPI:0047C55D                 pop     edi  
SDPI:0047C55E                 add     edi, 61h  
SDPI:0047C564                 mov     ebx, \[edi\]  
SDPI:0047C566                 mov     edx, \[edi+4\]  
SDPI:0047C566 ; ----------------------------------------------------------------------------  
SDPI:0047C569                 db 0Ch dup(90h)  
SDPI:0047C575 ; ----------------------------------------------------------------------------  
SDPI:0047C575                 call    loc\_47C57B  
SDPI:0047C57A                 nop  
SDPI:0047C57B  
SDPI:0047C57B loc\_47C57B:                                       ; CODE XREF: SDPI:0047C575 p  
SDPI:0047C57B                 pop     esi  
SDPI:0047C57C                 add     esi, 59h  
SDPI:0047C582                 mov     ecx, 3  
SDPI:0047C587                 nop  
SDPI:0047C588                 nop  
SDPI:0047C589                 nop  
SDPI:0047C58A                 nop  
SDPI:0047C58B                 nop  
SDPI:0047C58C                 nop  
SDPI:0047C58D                 nop  
SDPI:0047C58E                 nop  
SDPI:0047C58F                 nop  
SDPI:0047C590                 nop  
SDPI:0047C591                 nop  
SDPI:0047C592                 nop  
SDPI:0047C593                 nop  
SDPI:0047C594                 nop  
SDPI:0047C595                 nop  
SDPI:0047C596                 nop  
SDPI:0047C597                 nop  
SDPI:0047C598                 nop  
SDPI:0047C599                 nop  
SDPI:0047C59A                 nop  
SDPI:0047C59B                 nop  
SDPI:0047C59C                 nop  
SDPI:0047C59D                 nop  
SDPI:0047C59E                 nop  
SDPI:0047C59F                 nop  
SDPI:0047C5A0                 nop  
SDPI:0047C5A1                 nop  
SDPI:0047C5A2                 nop  
SDPI:0047C5A3                 nop  
SDPI:0047C5A4                 nop  
SDPI:0047C5A5                 nop  
SDPI:0047C5A6                 nop  
SDPI:0047C5A7                 nop  
SDPI:0047C5A8                 nop  
SDPI:0047C5A9                 nop  
SDPI:0047C5AA                 nop  
SDPI:0047C5AB                 nop  
SDPI:0047C5AC                 nop  
SDPI:0047C5AD                 nop  
SDPI:0047C5AE                 nop  
SDPI:0047C5AF                 nop  
SDPI:0047C5B0                 nop  
SDPI:0047C5B1                 nop  
SDPI:0047C5B2                 nop  
SDPI:0047C5B3                 nop  
SDPI:0047C5B4                 nop  
SDPI:0047C5B5                 nop  
SDPI:0047C5B6                 nop  
SDPI:0047C5B7                 nop  
SDPI:0047C5B8                 nop  
SDPI:0047C5B9                 nop  
SDPI:0047C5BA                 nop  
SDPI:0047C5BB                 nop  
SDPI:0047C5BC                 nop  
SDPI:0047C5BD                 rep movsw  
SDPI:0047C5C0                 call    fnddbg\_47C641  
SDPI:0047C5C5                 call    int3\_47C6B1  
SDPI:0047C5C5 ; ----------------------------------------------------------------------------  
SDPI:0047C5CA                 dd 401000E8h  
SDPI:0047C5CE                 dd 9C89B000h  
SDPI:0047C5D2                 dd 9001EB04h  
SDPI:0047C5D6                 dd 909003EBh  
SDPI:0047C5DA  
SDPI:0047C5DA ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047C5DA  
SDPI:0047C5DA  
SDPI:0047C5DA FNDDBG\_47C5DA   proc near  
SDPI:0047C5DA                 nop  
SDPI:0047C5DB                 nop  
SDPI:0047C5DC                 nop  
SDPI:0047C5DD                 nop  
SDPI:0047C5DE                 call    loc\_47C5E4  
SDPI:0047C5E3                 nop  
SDPI:0047C5E4  
SDPI:0047C5E4 loc\_47C5E4:                                       ; CODE XREF: FNDDBG\_47C5DA+4 p  
SDPI:0047C5E4                 pop     eax  
SDPI:0047C5E5                 add     eax, 5Eh  
SDPI:0047C5EA                 mov     edx, eax  
SDPI:0047C5EC                 add     edx, 32h  
SDPI:0047C5EF                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047C5EF                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047C5EF                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047C5EF                                                   ; 用来解密  
SDPI:0047C5F4                 call    loc\_47C5FA  
SDPI:0047C5F9                 nop  
SDPI:0047C5FA  
SDPI:0047C5FA loc\_47C5FA:                                       ; CODE XREF: FNDDBG\_47C5DA+1A p  
SDPI:0047C5FA                 pop     eax  
SDPI:0047C5FB                 add     eax, 2AA3h  
SDPI:0047C600                 call    loc\_47C606  
SDPI:0047C605                 nop  
SDPI:0047C606  
SDPI:0047C606 loc\_47C606:                                       ; CODE XREF: FNDDBG\_47C5DA+26 p  
SDPI:0047C606                 pop     ecx  
SDPI:0047C607                 add     ecx, 2B44h  
SDPI:0047C60D                 push    0  
SDPI:0047C60F                 push    ecx  
SDPI:0047C610                 push    eax  
SDPI:0047C611                 push    0  
SDPI:0047C613                 call    loc\_47C619  
SDPI:0047C618                 nop  
SDPI:0047C619  
SDPI:0047C619 loc\_47C619:                                       ; CODE XREF: FNDDBG\_47C5DA+39 p  
SDPI:0047C619                 pop     eax  
SDPI:0047C61A                 add     eax, 11h  
SDPI:0047C61F                 push    eax  
SDPI:0047C620                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047C620                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047C620                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047C620 ; ----------------------------------------------------------------------------  
SDPI:0047C625                 db 4 dup(90h)  
SDPI:0047C629 ; ----------------------------------------------------------------------------  
SDPI:0047C629                 push    7  
SDPI:0047C62B                 call    loc\_47C631  
SDPI:0047C630                 nop  
SDPI:0047C631  
SDPI:0047C631 loc\_47C631:                                       ; CODE XREF: FNDDBG\_47C5DA+51 p  
SDPI:0047C631                 pop     eax  
SDPI:0047C632                 add     eax, 11h  
SDPI:0047C637                 push    eax  
SDPI:0047C638                 jmp     ExitProcess  
SDPI:0047C638 ; ----------------------------------------------------------------------------  
SDPI:0047C63D                 db 4 dup(90h)  
SDPI:0047C63D FNDDBG\_47C5DA   endp  
SDPI:0047C63D  
SDPI:0047C641  
SDPI:0047C641 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047C641  
SDPI:0047C641  
SDPI:0047C641 fnddbg\_47C641   proc near                         ; CODE XREF: SDPI:0047C5C0 p  
SDPI:0047C641                 nop  
SDPI:0047C642                 nop  
SDPI:0047C643                 nop  
SDPI:0047C644                 nop  
SDPI:0047C645                 nop  
SDPI:0047C646                 call    loc\_47C64C  
SDPI:0047C64B                 nop  
SDPI:0047C64C  
SDPI:0047C64C loc\_47C64C:                                       ; CODE XREF: fnddbg\_47C641+5 p  
SDPI:0047C64C                 pop     eax  
SDPI:0047C64D                 add     eax, 5Eh  
SDPI:0047C652                 mov     edx, eax  
SDPI:0047C654                 add     edx, 32h  
SDPI:0047C657                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047C657                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047C657                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047C657                                                   ; 用来解密  
SDPI:0047C65C                 call    loc\_47C662  
SDPI:0047C661                 nop  
SDPI:0047C662  
SDPI:0047C662 loc\_47C662:                                       ; CODE XREF: fnddbg\_47C641+1B p  
SDPI:0047C662                 pop     eax  
SDPI:0047C663                 add     eax, 2A3Bh  
SDPI:0047C668                 call    loc\_47C66E  
SDPI:0047C66D                 nop  
SDPI:0047C66E  
SDPI:0047C66E loc\_47C66E:                                       ; CODE XREF: fnddbg\_47C641+27 p  
SDPI:0047C66E                 pop     ecx  
SDPI:0047C66F                 add     ecx, 2ADCh  
SDPI:0047C675                 push    0  
SDPI:0047C677                 push    ecx  
SDPI:0047C678                 push    eax  
SDPI:0047C679                 push    0  
SDPI:0047C67B                 call    loc\_47C681  
SDPI:0047C680                 nop  
SDPI:0047C681  
SDPI:0047C681 loc\_47C681:                                       ; CODE XREF: fnddbg\_47C641+3A p  
SDPI:0047C681                 pop     eax  
SDPI:0047C682                 add     eax, 11h  
SDPI:0047C687                 push    eax  
SDPI:0047C688                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047C688                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047C688                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047C688 ; ----------------------------------------------------------------------------  
SDPI:0047C68D                 db 4 dup(90h)  
SDPI:0047C691 ; ----------------------------------------------------------------------------  
SDPI:0047C691                 push    7  
SDPI:0047C693                 call    loc\_47C699  
SDPI:0047C698                 nop  
SDPI:0047C699  
SDPI:0047C699 loc\_47C699:                                       ; CODE XREF: fnddbg\_47C641+52 p  
SDPI:0047C699                 pop     eax  
SDPI:0047C69A                 add     eax, 11h  
SDPI:0047C69F                 push    eax  
SDPI:0047C6A0                 jmp     ExitProcess  
SDPI:0047C6A0 ; ----------------------------------------------------------------------------  
SDPI:0047C6A5                 db 0Ch dup(90h)  
SDPI:0047C6A5 fnddbg\_47C641   endp  
SDPI:0047C6A5  
SDPI:0047C6B1 ; ----------------------------------------------------------------------------  
SDPI:0047C6B1  
SDPI:0047C6B1 int3\_47C6B1:                                      ; CODE XREF: SDPI:0047C5C5 p  
SDPI:0047C6B1                 call    loc\_47C6B7  
SDPI:0047C6B6                 nop  
SDPI:0047C6B7  
SDPI:0047C6B7 loc\_47C6B7:                                       ; CODE XREF: SDPI:int3\_47C6B1 p  
SDPI:0047C6B7                 pop     edi  
SDPI:0047C6B8                 add     edi, 0FFFFFF07h  
SDPI:0047C6BE                 mov     \[edi\], ebx  
SDPI:0047C6C0                 mov     \[edi+4\], edx  
SDPI:0047C6C3                 pop     eax  
SDPI:0047C6C4                 call    loc\_47C6CA  
SDPI:0047C6C9                 nop  
SDPI:0047C6CA  
SDPI:0047C6CA loc\_47C6CA:                                       ; CODE XREF: SDPI:0047C6C4 p  
SDPI:0047C6CA                 pop     eax  
SDPI:0047C6CB                 add     eax, 124h  
SDPI:0047C6D0                 push    eax  
SDPI:0047C6D1                 xor     eax, eax  
SDPI:0047C6D3                 push    dword ptr fs:\[eax\]  
SDPI:0047C6D6                 mov     fs:\[eax\], esp  
SDPI:0047C6D9                 mov     ebp, 300EF1D3h  
SDPI:0047C6DE                 add     ebp, 12345678h  
SDPI:0047C6E4                 mov     ax, 17h  
SDPI:0047C6E8                 sub     ax, 13h  
SDPI:0047C6EC                 nop  
SDPI:0047C6ED                 nop  
SDPI:0047C6EE                 nop  
SDPI:0047C6EF                 nop  
SDPI:0047C6F0                 nop  
SDPI:0047C6F1                 nop  
SDPI:0047C6F2                 nop  
SDPI:0047C6F3                 nop  
SDPI:0047C6F4                 nop  
SDPI:0047C6F5                 nop  
SDPI:0047C6F6                 nop  
SDPI:0047C6F7                 nop  
SDPI:0047C6F8                 nop  
SDPI:0047C6F9                 nop  
SDPI:0047C6FA                 nop  
SDPI:0047C6FB                 nop  
SDPI:0047C6FC                 nop  
SDPI:0047C6FD                 nop  
SDPI:0047C6FE                 nop  
SDPI:0047C6FF                 nop  
SDPI:0047C700                 nop  
SDPI:0047C701                 nop  
SDPI:0047C702                 nop  
SDPI:0047C703                 nop  
SDPI:0047C704                 nop  
SDPI:0047C705                 nop  
SDPI:0047C706                 nop  
SDPI:0047C707                 nop  
SDPI:0047C708                 nop  
SDPI:0047C709                 nop  
SDPI:0047C70A                 nop  
SDPI:0047C70B                 nop  
SDPI:0047C70C                 nop  
SDPI:0047C70D                 nop  
SDPI:0047C70E                 nop  
SDPI:0047C70F                 nop  
SDPI:0047C710                 nop  
SDPI:0047C711                 nop  
SDPI:0047C712                 nop  
SDPI:0047C713                 nop  
SDPI:0047C714                 nop  
SDPI:0047C715                 nop  
SDPI:0047C716                 nop  
SDPI:0047C717                 nop  
SDPI:0047C718                 nop  
SDPI:0047C719                 nop  
SDPI:0047C71A                 nop  
SDPI:0047C71B                 nop  
SDPI:0047C71C                 nop  
SDPI:0047C71D                 nop  
SDPI:0047C71E                 nop  
SDPI:0047C71F                 nop  
SDPI:0047C720                 nop  
SDPI:0047C721                 nop  
SDPI:0047C722                 nop  
SDPI:0047C723                 nop  
SDPI:0047C724                 nop  
SDPI:0047C725                 nop  
SDPI:0047C726                 nop  
SDPI:0047C727                 int     3                         ; Trap to Debugger  
SDPI:0047C728                 nop  
SDPI:0047C729                 cmp     al, 4  
SDPI:0047C72B                 jz      short done\_47C79E  
SDPI:0047C72D  
SDPI:0047C72D ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047C72D  
SDPI:0047C72D  
SDPI:0047C72D fnddbg\_47C72D   proc near                         ; CODE XREF: SDPI:0047C7B7 j  
SDPI:0047C72D                                                   ; SDPI:0047C7CF j ...  
SDPI:0047C72D                 nop  
SDPI:0047C72E                 nop  
SDPI:0047C72F                 nop  
SDPI:0047C730                 nop  
SDPI:0047C731                 nop  
SDPI:0047C732                 call    loc\_47C738  
SDPI:0047C737                 nop  
SDPI:0047C738  
SDPI:0047C738 loc\_47C738:                                       ; CODE XREF: fnddbg\_47C72D+5 p  
SDPI:0047C738                 pop     eax  
SDPI:0047C739                 add     eax, 5Eh  
SDPI:0047C73E                 mov     edx, eax  
SDPI:0047C740                 add     edx, 32h  
SDPI:0047C743                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047C743                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047C743                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047C743                                                   ; 用来解密  
SDPI:0047C748                 call    loc\_47C74E  
SDPI:0047C74D                 nop  
SDPI:0047C74E  
SDPI:0047C74E loc\_47C74E:                                       ; CODE XREF: fnddbg\_47C72D+1B p  
SDPI:0047C74E                 pop     eax  
SDPI:0047C74F                 add     eax, 294Fh  
SDPI:0047C754                 call    loc\_47C75A  
SDPI:0047C759                 nop  
SDPI:0047C75A  
SDPI:0047C75A loc\_47C75A:                                       ; CODE XREF: fnddbg\_47C72D+27 p  
SDPI:0047C75A                 pop     ecx  
SDPI:0047C75B                 add     ecx, 29F0h  
SDPI:0047C761                 push    0  
SDPI:0047C763                 push    ecx  
SDPI:0047C764                 push    eax  
SDPI:0047C765                 push    0  
SDPI:0047C767                 call    loc\_47C76D  
SDPI:0047C76C                 nop  
SDPI:0047C76D  
SDPI:0047C76D loc\_47C76D:                                       ; CODE XREF: fnddbg\_47C72D+3A p  
SDPI:0047C76D                 pop     eax  
SDPI:0047C76E                 add     eax, 11h  
SDPI:0047C773                 push    eax  
SDPI:0047C774                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047C774                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047C774                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047C774 ; ----------------------------------------------------------------------------  
SDPI:0047C779                 db 4 dup(90h)  
SDPI:0047C77D ; ----------------------------------------------------------------------------  
SDPI:0047C77D                 push    7  
SDPI:0047C77F                 call    loc\_47C785  
SDPI:0047C784                 nop  
SDPI:0047C785  
SDPI:0047C785 loc\_47C785:                                       ; CODE XREF: fnddbg\_47C72D+52 p  
SDPI:0047C785                 pop     eax  
SDPI:0047C786                 add     eax, 11h  
SDPI:0047C78B                 push    eax  
SDPI:0047C78C                 jmp     ExitProcess  
SDPI:0047C78C ; ----------------------------------------------------------------------------  
SDPI:0047C791                 db 0Dh dup(90h)  
SDPI:0047C791 fnddbg\_47C72D   endp  
SDPI:0047C791  
SDPI:0047C79E ; ----------------------------------------------------------------------------  
SDPI:0047C79E  
SDPI:0047C79E done\_47C79E:                                      ; CODE XREF: SDPI:0047C72B j  
SDPI:0047C79E                 pop     large dword ptr fs:0  
SDPI:0047C7A5                 add     esp, 4  
SDPI:0047C7A8                 call    loc\_47C7AE  
SDPI:0047C7AD                 nop  
SDPI:0047C7AE  
SDPI:0047C7AE loc\_47C7AE:                                       ; CODE XREF: SDPI:0047C7A8 p  
SDPI:0047C7AE                 pop     eax  
SDPI:0047C7AF                 add     eax, 0FFFFFE1Dh  
SDPI:0047C7B4                 cmp     byte ptr \[eax\], 0E9h  
SDPI:0047C7B7                 jnz     fnddbg\_47C72D  
SDPI:0047C7BD                 mov     byte ptr \[eax\], 0E8h  
SDPI:0047C7C0                 rdtsc  
SDPI:0047C7C2                 mov     ecx, eax  
SDPI:0047C7C4                 mov     ebx, edx  
SDPI:0047C7C6                 rdtsc  
SDPI:0047C7C8                 sub     eax, ecx  
SDPI:0047C7CA                 sbb     edx, ebx  
SDPI:0047C7CC                 cmp     edx, 0  
SDPI:0047C7CF                 jnz     fnddbg\_47C72D  
SDPI:0047C7D5                 cmp     eax, 30000000h  
SDPI:0047C7DA                 ja      fnddbg\_47C72D  
SDPI:0047C7E0                 jmp     short PASS\_47C82B  
SDPI:0047C7E0 ; ----------------------------------------------------------------------------  
SDPI:0047C7E2                 db 0Bh dup(90h)  
SDPI:0047C7ED ; ----------------------------------------------------------------------------  
SDPI:0047C7ED                 mov     eax, \[esp+4\]  
SDPI:0047C7F1                 mov     ecx, \[esp+0Ch\]  
SDPI:0047C7F5                 inc     dword ptr \[ecx+0B8h\]  
SDPI:0047C7FB                 mov     eax, \[eax\]  
SDPI:0047C7FD                 sub     eax, 80000003h  
SDPI:0047C802                 jnz     short locret\_47C82A  
SDPI:0047C804                 call    loc\_47C80A  
SDPI:0047C809                 nop  
SDPI:0047C80A  
SDPI:0047C80A loc\_47C80A:                                       ; CODE XREF: SDPI:0047C804 p  
SDPI:0047C80A                 pop     eax  
SDPI:0047C80B                 add     eax, 0FFFFFDC1h  
SDPI:0047C810                 cmp     byte ptr \[eax\], 0E8h  
SDPI:0047C813                 jnz     fnddbg\_47C72D  
SDPI:0047C819                 mov     byte ptr \[eax\], 0E9h  
SDPI:0047C81C                 xor     eax, eax  
SDPI:0047C81E                 mov     \[ecx+4\], eax  
SDPI:0047C821                 mov     \[ecx+8\], eax  
SDPI:0047C824                 mov     \[ecx+0Ch\], eax  
SDPI:0047C827                 mov     \[ecx+10h\], eax  
SDPI:0047C82A  
SDPI:0047C82A locret\_47C82A:                                    ; CODE XREF: SDPI:0047C802 j  
SDPI:0047C82A                 retn  
SDPI:0047C82B ; ----------------------------------------------------------------------------  
SDPI:0047C82B  
SDPI:0047C82B PASS\_47C82B:                                      ; CODE XREF: SDPI:0047C7E0 j  
SDPI:0047C82B                 pop     eax  
SDPI:0047C82C                 call    loc\_47CA2C  
SDPI:0047C831                 nop  
SDPI:0047C832                 nop  
SDPI:0047C833                 nop  
SDPI:0047C834                 nop  
SDPI:0047C835                 nop  
SDPI:0047C836                 nop  
SDPI:0047C837                 nop  
SDPI:0047C838                 nop  
SDPI:0047C839  
SDPI:0047C839 loc\_47C839:                                       ; CODE XREF: SDPI:0047CA55 p  
SDPI:0047C839                 pop     ebp  
SDPI:0047C83A                 pop     eax  
SDPI:0047C83B                 jmp     loc\_47CA5A  
SDPI:0047C840 ; ----------------------------------------------------------------------------  
SDPI:0047C840                 call    loc\_47C846  
SDPI:0047C845                 nop  
SDPI:0047C846  
SDPI:0047C846 loc\_47C846:                                       ; CODE XREF: SDPI:0047C840 p  
SDPI:0047C846                 pop     eax  
SDPI:0047C847                 add     eax, 312h                 ; 把 0047CB57 的代码加密回去  
SDPI:0047C84C                 call    loc\_47C852  
SDPI:0047C851                 nop  
SDPI:0047C852  
SDPI:0047C852 loc\_47C852:                                       ; CODE XREF: SDPI:0047C84C p  
SDPI:0047C852                 pop     edx  
SDPI:0047C853                 add     edx, 38Ah                 ; 结束地址:0047CBDB  
SDPI:0047C859                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047C859                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047C859                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047C859                                                   ; 用来解密  
SDPI:0047C85E                 mov     ecx, 0FFFFFF00h  
SDPI:0047C863                 push    fs  
SDPI:0047C865                 nop  
SDPI:0047C866                 nop  
SDPI:0047C867                 nop  
SDPI:0047C868                 nop  
SDPI:0047C869                 nop  
SDPI:0047C86A                 nop  
SDPI:0047C86B                 nop  
SDPI:0047C86C                 nop  
SDPI:0047C86D                 nop  
SDPI:0047C86E                 nop  
SDPI:0047C86F                 nop  
SDPI:0047C870                 nop  
SDPI:0047C871                 pushfw  
SDPI:0047C873                 push    eax  
SDPI:0047C874                 mov     eax, ebx  
SDPI:0047C876                 push    ebx  
SDPI:0047C877                 mov     eax, ecx  
SDPI:0047C879                 push    eax  
SDPI:0047C87A                 add     eax, edx  
SDPI:0047C87C                 mov     ebx, eax  
SDPI:0047C87E                 push    ebx  
SDPI:0047C87F                 pop     eax  
SDPI:0047C880                 push    edx  
SDPI:0047C881                 call    loc\_47C88E  
SDPI:0047C886                 nop  
SDPI:0047C887                 nop  
SDPI:0047C888                 nop  
SDPI:0047C889                 nop  
SDPI:0047C88A                 nop  
SDPI:0047C88B                 nop  
SDPI:0047C88C                 nop  
SDPI:0047C88D                 nop  
SDPI:0047C88E  
SDPI:0047C88E loc\_47C88E:                                       ; CODE XREF: SDPI:0047C881 p  
SDPI:0047C88E                 pop     eax  
SDPI:0047C88F                 call    loc\_47C895  
SDPI:0047C894                 nop  
SDPI:0047C895  
SDPI:0047C895 loc\_47C895:                                       ; CODE XREF: SDPI:0047C88F p  
SDPI:0047C895                 pop     eax  
SDPI:0047C896                 add     eax, 11h  
SDPI:0047C89B                 push    eax  
SDPI:0047C89C                 jmp     GetTickCount  
SDPI:0047C89C ; ----------------------------------------------------------------------------  
SDPI:0047C8A1                 db 4 dup(90h)  
SDPI:0047C8A5 ; ----------------------------------------------------------------------------  
SDPI:0047C8A5                 push    eax  
SDPI:0047C8A6                 mov     eax, edx  
SDPI:0047C8A8                 push    eax  
SDPI:0047C8A9                 call    loc\_47C8AF  
SDPI:0047C8AE                 nop  
SDPI:0047C8AF  
SDPI:0047C8AF loc\_47C8AF:                                       ; CODE XREF: SDPI:0047C8A9 p  
SDPI:0047C8AF                 pop     edx  
SDPI:0047C8B0                 add     edx, 52h  
SDPI:0047C8B6                 push    edx  
SDPI:0047C8B7                 add     edx, 4038B7h  
SDPI:0047C8BD                 push    edx  
SDPI:0047C8BE                 jmp     short loc\_47C913  
SDPI:0047C8BE ; ----------------------------------------------------------------------------  
SDPI:0047C8C0                 db 2 dup(90h)  
SDPI:0047C8C2  
SDPI:0047C8C2 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047C8C2  
SDPI:0047C8C2  
SDPI:0047C8C2 sub\_47C8C2      proc near                         ; CODE XREF: SDPI:0047C906 p  
SDPI:0047C8C2                 pop     eax  
SDPI:0047C8C3                 pop     ebx  
SDPI:0047C8C4                 call    sub\_47C8CA  
SDPI:0047C8C9                 nop  
SDPI:0047C8C9 sub\_47C8C2      endp  
SDPI:0047C8C9  
SDPI:0047C8CA  
SDPI:0047C8CA ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047C8CA  
SDPI:0047C8CA  
SDPI:0047C8CA sub\_47C8CA      proc near                         ; CODE XREF: sub\_47C8C2+2 p  
SDPI:0047C8CA                 pop     eax  
SDPI:0047C8CB                 add     eax, 11h  
SDPI:0047C8D0                 push    eax  
SDPI:0047C8D1                 jmp     GetTickCount  
SDPI:0047C8D1 sub\_47C8CA      endp  
SDPI:0047C8D1  
SDPI:0047C8D1 ; ----------------------------------------------------------------------------  
SDPI:0047C8D6                 db 4 dup(90h)  
SDPI:0047C8DA ; ----------------------------------------------------------------------------  
SDPI:0047C8DA                 pop     ebx  
SDPI:0047C8DB                 add     ebx, 1F4h  
SDPI:0047C8E1                 sub     ebx, eax  
SDPI:0047C8E3                 js      short OVER\_47C927  
SDPI:0047C8E5                 call    loc\_47C8EB  
SDPI:0047C8EA                 nop  
SDPI:0047C8EB  
SDPI:0047C8EB loc\_47C8EB:                                       ; CODE XREF: SDPI:0047C8E5 p  
SDPI:0047C8EB                 pop     ebx  
SDPI:0047C8EC                 add     ebx, 0A5h  
SDPI:0047C8F2                 push    ebx  
SDPI:0047C8F3                 call    loc\_47C91D  
SDPI:0047C8F8                 nop  
SDPI:0047C8F9                 nop  
SDPI:0047C8FA                 nop  
SDPI:0047C8FB                 nop  
SDPI:0047C8FC                 nop  
SDPI:0047C8FD                 nop  
SDPI:0047C8FE                 nop  
SDPI:0047C8FF                 nop  
SDPI:0047C900                 pop     eax  
SDPI:0047C901                 mov     edx, eax  
SDPI:0047C903                 mov     eax, ebx  
SDPI:0047C905                 push    eax  
SDPI:0047C906                 call    sub\_47C8C2  
SDPI:0047C90B                 nop  
SDPI:0047C90C                 nop  
SDPI:0047C90D                 nop  
SDPI:0047C90E                 nop  
SDPI:0047C90F                 nop  
SDPI:0047C910                 nop  
SDPI:0047C911                 nop  
SDPI:0047C912                 nop  
SDPI:0047C913  
SDPI:0047C913 loc\_47C913:                                       ; CODE XREF: SDPI:0047C8BE j  
SDPI:0047C913                 pop     eax  
SDPI:0047C914                 retn  
SDPI:0047C914 ; ----------------------------------------------------------------------------  
SDPI:0047C915                 db 8 dup(90h)  
SDPI:0047C91D ; ----------------------------------------------------------------------------  
SDPI:0047C91D  
SDPI:0047C91D loc\_47C91D:                                       ; CODE XREF: SDPI:0047C8F3 p  
SDPI:0047C91D                 pop     edx  
SDPI:0047C91E                 retn  
SDPI:0047C91E ; ----------------------------------------------------------------------------  
SDPI:0047C91F                 db 8 dup(90h)  
SDPI:0047C927  
SDPI:0047C927 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047C927  
SDPI:0047C927  
SDPI:0047C927 OVER\_47C927     proc near                         ; CODE XREF: SDPI:0047C8E3 j  
SDPI:0047C927                 nop  
SDPI:0047C928                 nop  
SDPI:0047C929                 nop  
SDPI:0047C92A                 nop  
SDPI:0047C92B                 nop  
SDPI:0047C92C                 call    loc\_47C932  
SDPI:0047C931                 nop  
SDPI:0047C932  
SDPI:0047C932 loc\_47C932:                                       ; CODE XREF: OVER\_47C927+5 p  
SDPI:0047C932                 pop     eax  
SDPI:0047C933                 add     eax, 5Eh  
SDPI:0047C938                 mov     edx, eax  
SDPI:0047C93A                 add     edx, 32h  
SDPI:0047C93D                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047C93D                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047C93D                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047C93D                                                   ; 用来解密  
SDPI:0047C942                 call    loc\_47C948  
SDPI:0047C947                 nop  
SDPI:0047C948  
SDPI:0047C948 loc\_47C948:                                       ; CODE XREF: OVER\_47C927+1B p  
SDPI:0047C948                 pop     eax  
SDPI:0047C949                 add     eax, 2755h  
SDPI:0047C94E                 call    loc\_47C954  
SDPI:0047C953                 nop  
SDPI:0047C954  
SDPI:0047C954 loc\_47C954:                                       ; CODE XREF: OVER\_47C927+27 p  
SDPI:0047C954                 pop     ecx  
SDPI:0047C955                 add     ecx, 27F6h  
SDPI:0047C95B                 push    0  
SDPI:0047C95D                 push    ecx  
SDPI:0047C95E                 push    eax  
SDPI:0047C95F                 push    0  
SDPI:0047C961                 call    loc\_47C967  
SDPI:0047C966                 nop  
SDPI:0047C967  
SDPI:0047C967 loc\_47C967:                                       ; CODE XREF: OVER\_47C927+3A p  
SDPI:0047C967                 pop     eax  
SDPI:0047C968                 add     eax, 11h  
SDPI:0047C96D                 push    eax  
SDPI:0047C96E                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047C96E                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047C96E                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047C96E ; ----------------------------------------------------------------------------  
SDPI:0047C973                 db 4 dup(90h)  
SDPI:0047C977 ; ----------------------------------------------------------------------------  
SDPI:0047C977                 push    7  
SDPI:0047C979                 call    loc\_47C97F  
SDPI:0047C97E                 nop  
SDPI:0047C97F  
SDPI:0047C97F loc\_47C97F:                                       ; CODE XREF: OVER\_47C927+52 p  
SDPI:0047C97F                 pop     eax  
SDPI:0047C980                 add     eax, 11h  
SDPI:0047C985                 push    eax  
SDPI:0047C986                 jmp     ExitProcess  
SDPI:0047C986 ; ----------------------------------------------------------------------------  
SDPI:0047C98B                 db 4 dup(90h)  
SDPI:0047C98B OVER\_47C927     endp  
SDPI:0047C98B  
SDPI:0047C98F ; ----------------------------------------------------------------------------  
SDPI:0047C98F                 pop     edx  
SDPI:0047C990                 mov     eax, ecx  
SDPI:0047C992                 add     eax, edx  
SDPI:0047C994                 inc     ecx  
SDPI:0047C995                 push    eax  
SDPI:0047C996                 inc     ecx  
SDPI:0047C997                 pop     ebx  
SDPI:0047C998                 pop     ecx  
SDPI:0047C999                 push    eax  
SDPI:0047C99A                 sub     eax, 8  
SDPI:0047C99D                 pop     ebx  
SDPI:0047C99E                 pop     ebx  
SDPI:0047C99F                 inc     eax  
SDPI:0047C9A0                 add     eax, ebx  
SDPI:0047C9A2                 pop     eax  
SDPI:0047C9A3                 pushfw  
SDPI:0047C9A5                 popfw  
SDPI:0047C9A7                 popfw  
SDPI:0047C9A9                 pop     es  
SDPI:0047C9AA                 mov     eax, 12345678h  
SDPI:0047C9AF                 push    eax  
SDPI:0047C9B0                 call    loc\_47C9B6  
SDPI:0047C9B5                 nop  
SDPI:0047C9B6  
SDPI:0047C9B6 loc\_47C9B6:                                       ; CODE XREF: SDPI:0047C9B0 p  
SDPI:0047C9B6                 pop     eax  
SDPI:0047C9B7                 add     eax, 12Ch  
SDPI:0047C9BC                 push    eax  
SDPI:0047C9BD                 pop     ebx  
SDPI:0047C9BE                 add     eax, 12h  
SDPI:0047C9C1                 pop     edx  
SDPI:0047C9C2                 add     eax, edx  
SDPI:0047C9C4                 mov     edx, eax  
SDPI:0047C9C6                 push    ebx  
SDPI:0047C9C7                 mov     ebx, es:\[ecx+100h\]  
SDPI:0047C9CE                 push    ebx  
SDPI:0047C9CF                 mov     eax, esp  
SDPI:0047C9D1                 mov     ebx, eax  
SDPI:0047C9D3                 push    ebx  
SDPI:0047C9D4                 pop     edx  
SDPI:0047C9D5                 mov     es:\[ecx+100h\], eax  
SDPI:0047C9DC                 xor     eax, eax  
SDPI:0047C9DC ; ----------------------------------------------------------------------------  
SDPI:0047C9DE                 db 38h dup(90h)  
SDPI:0047CA16 ; ----------------------------------------------------------------------------  
SDPI:0047CA16                 int     3                         ; Trap to Debugger  
SDPI:0047CA17                 nop  
SDPI:0047CA18                 xor     eax, eax  
SDPI:0047CA1A                 mov     dword ptr \[eax\], 403B1Ah  
SDPI:0047CA20                 nop  
SDPI:0047CA21                 nop  
SDPI:0047CA22                 nop  
SDPI:0047CA23                 nop  
SDPI:0047CA24                 nop  
SDPI:0047CA25                 nop  
SDPI:0047CA26                 nop  
SDPI:0047CA27                 nop  
SDPI:0047CA28                 nop  
SDPI:0047CA29                 nop  
SDPI:0047CA2A                 nop  
SDPI:0047CA2B                 nop  
SDPI:0047CA2C  
SDPI:0047CA2C loc\_47CA2C:                                       ; CODE XREF: SDPI:0047C82C p  
SDPI:0047CA2C                 call    loc\_47CA32  
SDPI:0047CA31                 nop  
SDPI:0047CA32  
SDPI:0047CA32 loc\_47CA32:                                       ; CODE XREF: SDPI:loc\_47CA2C p  
SDPI:0047CA32                 pop     eax  
SDPI:0047CA33                 add     eax, 11h  
SDPI:0047CA38                 push    eax  
SDPI:0047CA39                 jmp     GetTickCount  
SDPI:0047CA39 ; ----------------------------------------------------------------------------  
SDPI:0047CA3E                 db 4 dup(90h)  
SDPI:0047CA42 ; ----------------------------------------------------------------------------  
SDPI:0047CA42                 call    loc\_47CA48  
SDPI:0047CA47                 nop  
SDPI:0047CA48  
SDPI:0047CA48 loc\_47CA48:                                       ; CODE XREF: SDPI:0047CA42 p  
SDPI:0047CA48                 pop     edx  
SDPI:0047CA49                 add     edx, 0FFFFFB09h  
SDPI:0047CA4F                 mov     \[edx\], eax  
SDPI:0047CA51                 pop     ebp  
SDPI:0047CA52                 add     eax, edx  
SDPI:0047CA54                 push    eax  
SDPI:0047CA55                 call    loc\_47C839  
SDPI:0047CA5A  
SDPI:0047CA5A loc\_47CA5A:                                       ; CODE XREF: SDPI:0047C83B j  
SDPI:0047CA5A                 call    loc\_47CA60  
SDPI:0047CA5F                 nop  
SDPI:0047CA60  
SDPI:0047CA60 loc\_47CA60:                                       ; CODE XREF: SDPI:loc\_47CA5A p  
SDPI:0047CA60                 pop     edx  
SDPI:0047CA61                 add     edx, 0FFFFFDE1h  
SDPI:0047CA67                 add     edx, eax  
SDPI:0047CA69                 push    edx  
SDPI:0047CA6A                 pop     ecx  
SDPI:0047CA6B                 sub     ecx, eax  
SDPI:0047CA6D                 push    ecx  
SDPI:0047CA6E                 retn    4  
SDPI:0047CA71  
SDPI:0047CA71 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047CA71  
SDPI:0047CA71  
SDPI:0047CA71 Over\_47ca71     proc near                         ; CODE XREF: SDPI:0047CB14 j  
SDPI:0047CA71                                                   ; SDPI:0047CB1C j ...  
SDPI:0047CA71                 nop  
SDPI:0047CA72                 nop  
SDPI:0047CA73                 nop  
SDPI:0047CA74                 nop  
SDPI:0047CA75                 nop  
SDPI:0047CA76                 call    loc\_47CA7C  
SDPI:0047CA7B                 nop  
SDPI:0047CA7C  
SDPI:0047CA7C loc\_47CA7C:                                       ; CODE XREF: Over\_47ca71+5 p  
SDPI:0047CA7C                 pop     eax  
SDPI:0047CA7D                 add     eax, 5Eh  
SDPI:0047CA82                 mov     edx, eax  
SDPI:0047CA84                 add     edx, 32h  
SDPI:0047CA87                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047CA87                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047CA87                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047CA87                                                   ; 用来解密  
SDPI:0047CA8C                 call    loc\_47CA92  
SDPI:0047CA91                 nop  
SDPI:0047CA92  
SDPI:0047CA92 loc\_47CA92:                                       ; CODE XREF: Over\_47ca71+1B p  
SDPI:0047CA92                 pop     eax  
SDPI:0047CA93                 add     eax, 260Bh  
SDPI:0047CA98                 call    loc\_47CA9E  
SDPI:0047CA9D                 nop  
SDPI:0047CA9E  
SDPI:0047CA9E loc\_47CA9E:                                       ; CODE XREF: Over\_47ca71+27 p  
SDPI:0047CA9E                 pop     ecx  
SDPI:0047CA9F                 add     ecx, 26ACh  
SDPI:0047CAA5                 push    0  
SDPI:0047CAA7                 push    ecx  
SDPI:0047CAA8                 push    eax  
SDPI:0047CAA9                 push    0  
SDPI:0047CAAB                 call    loc\_47CAB1  
SDPI:0047CAB0                 nop  
SDPI:0047CAB1  
SDPI:0047CAB1 loc\_47CAB1:                                       ; CODE XREF: Over\_47ca71+3A p  
SDPI:0047CAB1                 pop     eax  
SDPI:0047CAB2                 add     eax, 11h  
SDPI:0047CAB7                 push    eax  
SDPI:0047CAB8                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047CAB8                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047CAB8                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047CAB8 ; ----------------------------------------------------------------------------  
SDPI:0047CABD                 db 4 dup(90h)  
SDPI:0047CAC1 ; ----------------------------------------------------------------------------  
SDPI:0047CAC1                 push    7  
SDPI:0047CAC3                 call    loc\_47CAC9  
SDPI:0047CAC8                 nop  
SDPI:0047CAC9  
SDPI:0047CAC9 loc\_47CAC9:                                       ; CODE XREF: Over\_47ca71+52 p  
SDPI:0047CAC9                 pop     eax  
SDPI:0047CACA                 add     eax, 11h  
SDPI:0047CACF                 push    eax  
SDPI:0047CAD0                 jmp     ExitProcess  
SDPI:0047CAD0 ; ----------------------------------------------------------------------------  
SDPI:0047CAD5                 db 4 dup(90h), 0, 10h, 40h, 0, 0BEh, 56h, 5Ch, 1  
SDPI:0047CAD5 Over\_47ca71     endp  
SDPI:0047CAD5  
SDPI:0047CAE1 ; ----------------------------------------------------------------------------  
SDPI:0047CAE1                 mov     esp, \[esp+8\]  
SDPI:0047CAE5                 pop     large dword ptr fs:0  
SDPI:0047CAEC                 call    loc\_47CAF2  
SDPI:0047CAF1                 nop  
SDPI:0047CAF2  
SDPI:0047CAF2 loc\_47CAF2:                                       ; CODE XREF: SDPI:0047CAEC p  
SDPI:0047CAF2                 pop     eax  
SDPI:0047CAF3                 add     eax, 11h  
SDPI:0047CAF8                 push    eax  
SDPI:0047CAF9                 jmp     GetTickCount  
SDPI:0047CAF9 ; ----------------------------------------------------------------------------  
SDPI:0047CAFE                 db 4 dup(90h)  
SDPI:0047CB02 ; ----------------------------------------------------------------------------  
SDPI:0047CB02                 call    loc\_47CB08  
SDPI:0047CB07                 nop  
SDPI:0047CB08  
SDPI:0047CB08 loc\_47CB08:                                       ; CODE XREF: SDPI:0047CB02 p  
SDPI:0047CB08                 pop     edx  
SDPI:0047CB09                 add     edx, 0FFFFFA49h  
SDPI:0047CB0F                 mov     ecx, \[edx\]  
SDPI:0047CB11                 cmp     ecx, 0  
SDPI:0047CB14                 jz      Over\_47ca71  
SDPI:0047CB1A                 sub     eax, ecx  
SDPI:0047CB1C                 js      Over\_47ca71  
SDPI:0047CB22                 sub     eax, 7D0h  
SDPI:0047CB27                 jns     Over\_47ca71  
SDPI:0047CB2D                 mov     eax, 0E801276h  
SDPI:0047CB32                 mov     \[edx\], eax  
SDPI:0047CB34                 call    loc\_47CB3A  
SDPI:0047CB39                 nop  
SDPI:0047CB3A  
SDPI:0047CB3A loc\_47CB3A:                                       ; CODE XREF: SDPI:0047CB34 p  
SDPI:0047CB3A                 pop     edx  
SDPI:0047CB3B                 add     edx, 0A2h  
SDPI:0047CB41                 call    loc\_47CB47  
SDPI:0047CB46                 nop  
SDPI:0047CB47  
SDPI:0047CB47 loc\_47CB47:                                       ; CODE XREF: SDPI:0047CB41 p  
SDPI:0047CB47                 pop     eax  
SDPI:0047CB48                 add     eax, 0FFFFD652h  
SDPI:0047CB4D                 mov     ecx, 10h                  ; 再次解密代码  
SDPI:0047CB52                 call    De\_Code                   ; invoke De\_Code,ecx(counter),eax(key),edx(end)  
SDPI:0047CB52                                                   ; 用于解密代码，解密起始地址就是 call 进来  
SDPI:0047CB52                                                   ; 的下一行代码地址  
SDPI:0047CB52 ; ----------------------------------------------------------------------------  
SDPI:0047CB57                 db 0Fh dup(90h)  
SDPI:0047CB66 ; ----------------------------------------------------------------------------  
SDPI:0047CB66                 call    loc\_47CB6C  
SDPI:0047CB6B                 nop  
SDPI:0047CB6C  
SDPI:0047CB6C loc\_47CB6C:                                       ; CODE XREF: SDPI:0047CB66 p  
SDPI:0047CB6C                 pop     eax  
SDPI:0047CB6D                 add     eax, 11h  
SDPI:0047CB72                 push    eax  
SDPI:0047CB73                 jmp     Get\_Version  
SDPI:0047CB73 ; ----------------------------------------------------------------------------  
SDPI:0047CB78                 db 4 dup(90h)  
SDPI:0047CB7C ; ----------------------------------------------------------------------------  
SDPI:0047CB7C                 call    loc\_47CB82  
SDPI:0047CB81                 nop  
SDPI:0047CB82  
SDPI:0047CB82 loc\_47CB82:                                       ; CODE XREF: SDPI:0047CB7C p  
SDPI:0047CB82                 pop     edx  
SDPI:0047CB83                 add     edx, 0FFFFD483h  
SDPI:0047CB89                 cmp     eax, \[edx\]                ; 又判断是否修改了 GetVersion 的返回值  
SDPI:0047CB8B                 jnz     OVER\_47C43A  
SDPI:0047CB91                 cmp     eax, 80000000h  
SDPI:0047CB96                 jb      NotIsWin9x\_47D312         ; 再次判断是否为 Winnt 系统  
SDPI:0047CB9C                 mov     ah, 43h                   ; 如果是 Win9x 则产生 int 68 异常  
SDPI:0047CB9E                 int     68h                       ;  - APPC/PC  
SDPI:0047CBA0                 cmp     ax, 0F386h  
SDPI:0047CBA4                 jnz     NODBG\_47CC43  
SDPI:0047CBAA                 jz      short near ptr aU4Rrrrr+6  
SDPI:0047CBAC                 push    ebx  
SDPI:0047CBAD                 push    edi  
SDPI:0047CBAE                 push    es  
SDPI:0047CBAE ; ----------------------------------------------------------------------------  
SDPI:0047CBAF a9              db '~',6,'',4,'9 窿',0Dh  
SDPI:0047CBB7 ; ----------------------------------------------------------------------------  
SDPI:0047CBB7                 xor     di, di  
SDPI:0047CBBA                 db      66h  
SDPI:0047CBBA                 mov     es, di  
SDPI:0047CBBD                 mov     ax, 1684h  
SDPI:0047CBC1                 mov     bx, 202h  
SDPI:0047CBC5                 int     2Fh  
SDPI:0047CBC7                 mov     ax, es  
SDPI:0047CBCA                 add     ax, di  
SDPI:0047CBCD                 pop     es  
SDPI:0047CBCE                 pop     edi  
SDPI:0047CBCF                 pop     ebx  
SDPI:0047CBD0                 test    ax, ax  
SDPI:0047CBD3                 jz      short NODBG\_47CC43  
SDPI:0047CBD3 ; ----------------------------------------------------------------------------  
SDPI:0047CBD5 aU4Rrrrr        db 'u',4,'$4',5,0,'悙悙?         ; CODE XREF: SDPI:0047CBAA j  
SDPI:0047CBE0  
SDPI:0047CBE0 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047CBE0  
SDPI:0047CBE0  
SDPI:0047CBE0 FNDDBG\_46CBE0   proc near  
SDPI:0047CBE0                 call    loc\_47CBE6  
SDPI:0047CBE5                 nop  
SDPI:0047CBE6  
SDPI:0047CBE6 loc\_47CBE6:                                       ; CODE XREF: FNDDBG\_46CBE0 p  
SDPI:0047CBE6                 pop     eax  
SDPI:0047CBE7                 add     eax, 5Eh  
SDPI:0047CBEC                 mov     edx, eax  
SDPI:0047CBEE                 add     edx, 32h  
SDPI:0047CBF1                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047CBF1                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047CBF1                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047CBF1                                                   ; 用来解密  
SDPI:0047CBF6                 call    loc\_47CBFC  
SDPI:0047CBFB                 nop  
SDPI:0047CBFC  
SDPI:0047CBFC loc\_47CBFC:                                       ; CODE XREF: FNDDBG\_46CBE0+16 p  
SDPI:0047CBFC                 pop     eax  
SDPI:0047CBFD                 add     eax, 24A1h  
SDPI:0047CC02                 call    loc\_47CC08  
SDPI:0047CC07                 nop  
SDPI:0047CC08  
SDPI:0047CC08 loc\_47CC08:                                       ; CODE XREF: FNDDBG\_46CBE0+22 p  
SDPI:0047CC08                 pop     ecx  
SDPI:0047CC09                 add     ecx, 2542h  
SDPI:0047CC0F                 push    0  
SDPI:0047CC11                 push    ecx  
SDPI:0047CC12                 push    eax  
SDPI:0047CC13                 push    0  
SDPI:0047CC15                 call    loc\_47CC1B  
SDPI:0047CC1A                 nop  
SDPI:0047CC1B  
SDPI:0047CC1B loc\_47CC1B:                                       ; CODE XREF: FNDDBG\_46CBE0+35 p  
SDPI:0047CC1B                 pop     eax  
SDPI:0047CC1C                 add     eax, 11h  
SDPI:0047CC21                 push    eax  
SDPI:0047CC22                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047CC22                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047CC22                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047CC22 ; ----------------------------------------------------------------------------  
SDPI:0047CC27                 db  90h ; ?  
SDPI:0047CC28                 db  90h ; ?  
SDPI:0047CC29                 db  90h ; ?  
SDPI:0047CC2A                 db  90h ; ?  
SDPI:0047CC2B ; ----------------------------------------------------------------------------  
SDPI:0047CC2B                 push    7  
SDPI:0047CC2D                 call    loc\_47CC33  
SDPI:0047CC32                 nop  
SDPI:0047CC33  
SDPI:0047CC33 loc\_47CC33:                                       ; CODE XREF: FNDDBG\_46CBE0+4D p  
SDPI:0047CC33                 pop     eax  
SDPI:0047CC34                 add     eax, 11h  
SDPI:0047CC39                 push    eax  
SDPI:0047CC3A                 jmp     ExitProcess  
SDPI:0047CC3A ; ----------------------------------------------------------------------------  
SDPI:0047CC3F                 db 4 dup(90h)  
SDPI:0047CC3F FNDDBG\_46CBE0   endp  
SDPI:0047CC3F  
SDPI:0047CC43 ; ----------------------------------------------------------------------------  
SDPI:0047CC43  
SDPI:0047CC43 NODBG\_47CC43:                                     ; CODE XREF: SDPI:0047CBA4 j  
SDPI:0047CC43                                                   ; SDPI:0047CBD3 j  
SDPI:0047CC43                 nop  
SDPI:0047CC44                 nop  
SDPI:0047CC45                 nop  
SDPI:0047CC46                 nop  
SDPI:0047CC47                 nop  
SDPI:0047CC48                 nop  
SDPI:0047CC49                 nop  
SDPI:0047CC4A                 nop  
SDPI:0047CC4B                 nop  
SDPI:0047CC4C                 nop  
SDPI:0047CC4D                 nop  
SDPI:0047CC4E                 nop  
SDPI:0047CC4F                 nop  
SDPI:0047CC50                 nop  
SDPI:0047CC51                 nop  
SDPI:0047CC52                 push    cs  
SDPI:0047CC53                 push    eax  
SDPI:0047CC54                 xor     eax, eax  
SDPI:0047CC56                 call    loc\_47CC5C  
SDPI:0047CC5B                 nop  
SDPI:0047CC5C  
SDPI:0047CC5C loc\_47CC5C:                                       ; CODE XREF: SDPI:0047CC56 p  
SDPI:0047CC5C                 pop     edi  
SDPI:0047CC5D                 add     edi, 61h  
SDPI:0047CC63                 mov     ebx, \[edi\]  
SDPI:0047CC65                 mov     edx, \[edi+4\]  
SDPI:0047CC65 ; ----------------------------------------------------------------------------  
SDPI:0047CC68                 db 0Ch dup(90h)  
SDPI:0047CC74 ; ----------------------------------------------------------------------------  
SDPI:0047CC74                 call    loc\_47CC7A  
SDPI:0047CC79                 nop  
SDPI:0047CC7A  
SDPI:0047CC7A loc\_47CC7A:                                       ; CODE XREF: SDPI:0047CC74 p  
SDPI:0047CC7A                 pop     esi  
SDPI:0047CC7B                 add     esi, 59h  
SDPI:0047CC81                 mov     ecx, 3  
SDPI:0047CC81 ; ----------------------------------------------------------------------------  
SDPI:0047CC86                 db 34h dup(90h)  
SDPI:0047CCBA                 db 2 dup(90h)  
SDPI:0047CCBC ; ----------------------------------------------------------------------------  
SDPI:0047CCBC                 rep movsw  
SDPI:0047CCBC ; ----------------------------------------------------------------------------  
SDPI:0047CCBF                 db 0E8h, 7Ch, 3 dup(0), 0E8h, 0E7h, 3 dup(0), 0E8h, 0  
SDPI:0047CCBF                 db 10h, 40h, 0, 0B0h, 89h, 9Ch, 4, 0EBh, 1, 90h, 0EBh  
SDPI:0047CCBF                 db 3, 6 dup(90h)  
SDPI:0047CCDD  
SDPI:0047CCDD ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047CCDD  
SDPI:0047CCDD  
SDPI:0047CCDD FNDDBG\_47CCDD   proc near  
SDPI:0047CCDD                 call    loc\_47CCE3  
SDPI:0047CCE2                 nop  
SDPI:0047CCE3  
SDPI:0047CCE3 loc\_47CCE3:                                       ; CODE XREF: FNDDBG\_47CCDD p  
SDPI:0047CCE3                 pop     eax  
SDPI:0047CCE4                 add     eax, 5Eh  
SDPI:0047CCE9                 mov     edx, eax  
SDPI:0047CCEB                 add     edx, 32h  
SDPI:0047CCEE                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047CCEE                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047CCEE                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047CCEE                                                   ; 用来解密  
SDPI:0047CCF3                 call    loc\_47CCF9  
SDPI:0047CCF8                 nop  
SDPI:0047CCF9  
SDPI:0047CCF9 loc\_47CCF9:                                       ; CODE XREF: FNDDBG\_47CCDD+16 p  
SDPI:0047CCF9                 pop     eax  
SDPI:0047CCFA                 add     eax, 23A4h  
SDPI:0047CCFF                 call    loc\_47CD05  
SDPI:0047CD04                 nop  
SDPI:0047CD05  
SDPI:0047CD05 loc\_47CD05:                                       ; CODE XREF: FNDDBG\_47CCDD+22 p  
SDPI:0047CD05                 pop     ecx  
SDPI:0047CD06                 add     ecx, 2445h  
SDPI:0047CD0C                 push    0  
SDPI:0047CD0E                 push    ecx  
SDPI:0047CD0F                 push    eax  
SDPI:0047CD10                 push    0  
SDPI:0047CD12                 call    loc\_47CD18  
SDPI:0047CD17                 nop  
SDPI:0047CD18  
SDPI:0047CD18 loc\_47CD18:                                       ; CODE XREF: FNDDBG\_47CCDD+35 p  
SDPI:0047CD18                 pop     eax  
SDPI:0047CD19                 add     eax, 11h  
SDPI:0047CD1E                 push    eax  
SDPI:0047CD1F                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047CD1F                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047CD1F                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047CD1F ; ----------------------------------------------------------------------------  
SDPI:0047CD24                 db 4 dup(90h)  
SDPI:0047CD28 ; ----------------------------------------------------------------------------  
SDPI:0047CD28                 push    7  
SDPI:0047CD2A                 call    loc\_47CD30  
SDPI:0047CD2F                 nop  
SDPI:0047CD30  
SDPI:0047CD30 loc\_47CD30:                                       ; CODE XREF: FNDDBG\_47CCDD+4D p  
SDPI:0047CD30                 pop     eax  
SDPI:0047CD31                 add     eax, 11h  
SDPI:0047CD36                 push    eax  
SDPI:0047CD37                 jmp     ExitProcess  
SDPI:0047CD37 ; ----------------------------------------------------------------------------  
SDPI:0047CD3C                 db 4 dup(90h)  
SDPI:0047CD3C FNDDBG\_47CCDD   endp  
SDPI:0047CD3C  
SDPI:0047CD40  
SDPI:0047CD40 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047CD40  
SDPI:0047CD40  
SDPI:0047CD40 FNDDBG\_47CD40   proc near  
SDPI:0047CD40                 nop  
SDPI:0047CD41                 nop  
SDPI:0047CD42                 nop  
SDPI:0047CD43                 nop  
SDPI:0047CD44                 nop  
SDPI:0047CD45                 call    loc\_47CD4B  
SDPI:0047CD4A                 nop  
SDPI:0047CD4B  
SDPI:0047CD4B loc\_47CD4B:                                       ; CODE XREF: FNDDBG\_47CD40+5 p  
SDPI:0047CD4B                 pop     eax  
SDPI:0047CD4C                 add     eax, 5Eh  
SDPI:0047CD51                 mov     edx, eax  
SDPI:0047CD53                 add     edx, 32h  
SDPI:0047CD56                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047CD56                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047CD56                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047CD56                                                   ; 用来解密  
SDPI:0047CD5B                 call    loc\_47CD61  
SDPI:0047CD60                 nop  
SDPI:0047CD61  
SDPI:0047CD61 loc\_47CD61:                                       ; CODE XREF: FNDDBG\_47CD40+1B p  
SDPI:0047CD61                 pop     eax  
SDPI:0047CD62                 add     eax, 233Ch  
SDPI:0047CD67                 call    loc\_47CD6D  
SDPI:0047CD6C                 nop  
SDPI:0047CD6D  
SDPI:0047CD6D loc\_47CD6D:                                       ; CODE XREF: FNDDBG\_47CD40+27 p  
SDPI:0047CD6D                 pop     ecx  
SDPI:0047CD6E                 add     ecx, 23DDh  
SDPI:0047CD74                 push    0  
SDPI:0047CD76                 push    ecx  
SDPI:0047CD77                 push    eax  
SDPI:0047CD78                 push    0  
SDPI:0047CD7A                 call    loc\_47CD80  
SDPI:0047CD7F                 nop  
SDPI:0047CD80  
SDPI:0047CD80 loc\_47CD80:                                       ; CODE XREF: FNDDBG\_47CD40+3A p  
SDPI:0047CD80                 pop     eax  
SDPI:0047CD81                 add     eax, 11h  
SDPI:0047CD86                 push    eax  
SDPI:0047CD87                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047CD87                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047CD87                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047CD87 ; ----------------------------------------------------------------------------  
SDPI:0047CD8C                 db 4 dup(90h)  
SDPI:0047CD90 ; ----------------------------------------------------------------------------  
SDPI:0047CD90                 push    7  
SDPI:0047CD92                 call    loc\_47CD98  
SDPI:0047CD97                 nop  
SDPI:0047CD98  
SDPI:0047CD98 loc\_47CD98:                                       ; CODE XREF: FNDDBG\_47CD40+52 p  
SDPI:0047CD98                 pop     eax  
SDPI:0047CD99                 add     eax, 11h  
SDPI:0047CD9E                 push    eax  
SDPI:0047CD9F                 jmp     ExitProcess  
SDPI:0047CD9F ; ----------------------------------------------------------------------------  
SDPI:0047CDA4                 db 4 dup(90h), 0, 10h, 40h, 0, 0E0h, 89h, 9Ch, 6  
SDPI:0047CDA4 FNDDBG\_47CD40   endp  
SDPI:0047CDA4  
SDPI:0047CDB0 ; ----------------------------------------------------------------------------  
SDPI:0047CDB0                 call    loc\_47CDB6  
SDPI:0047CDB5                 nop  
SDPI:0047CDB6  
SDPI:0047CDB6 loc\_47CDB6:                                       ; CODE XREF: SDPI:0047CDB0 p  
SDPI:0047CDB6                 pop     edi  
SDPI:0047CDB7                 add     edi, 0FFFFFF07h  
SDPI:0047CDBD                 mov     \[edi\], ebx  
SDPI:0047CDBF                 mov     \[edi+4\], edx  
SDPI:0047CDC2                 pop     eax  
SDPI:0047CDC3                 call    loc\_47CDC9  
SDPI:0047CDC8                 nop  
SDPI:0047CDC9  
SDPI:0047CDC9 loc\_47CDC9:                                       ; CODE XREF: SDPI:0047CDC3 p  
SDPI:0047CDC9                 pop     eax  
SDPI:0047CDCA                 add     eax, 124h  
SDPI:0047CDCF                 push    eax  
SDPI:0047CDD0                 xor     eax, eax  
SDPI:0047CDD2                 push    dword ptr fs:\[eax\]  
SDPI:0047CDD5                 mov     fs:\[eax\], esp  
SDPI:0047CDD8                 mov     ebp, 300EF1D3h  
SDPI:0047CDDD                 add     ebp, 12345678h  
SDPI:0047CDE3                 mov     ax, 17h  
SDPI:0047CDE7                 sub     ax, 13h  
SDPI:0047CDEB                 nop  
SDPI:0047CDEC                 nop  
SDPI:0047CDED                 nop  
SDPI:0047CDEE                 nop  
SDPI:0047CDEF                 nop  
SDPI:0047CDF0                 nop  
SDPI:0047CDF1                 nop  
SDPI:0047CDF2                 nop  
SDPI:0047CDF3                 nop  
SDPI:0047CDF4                 nop  
SDPI:0047CDF5                 nop  
SDPI:0047CDF6                 nop  
SDPI:0047CDF7                 nop  
SDPI:0047CDF8                 nop  
SDPI:0047CDF9                 nop  
SDPI:0047CDFA                 nop  
SDPI:0047CDFB                 nop  
SDPI:0047CDFC                 nop  
SDPI:0047CDFD                 nop  
SDPI:0047CDFE                 nop  
SDPI:0047CDFF                 nop  
SDPI:0047CE00                 nop  
SDPI:0047CE01                 nop  
SDPI:0047CE02                 nop  
SDPI:0047CE03                 nop  
SDPI:0047CE04                 nop  
SDPI:0047CE05                 nop  
SDPI:0047CE06                 nop  
SDPI:0047CE07                 nop  
SDPI:0047CE08                 nop  
SDPI:0047CE09                 nop  
SDPI:0047CE0A                 nop  
SDPI:0047CE0B                 nop  
SDPI:0047CE0C                 nop  
SDPI:0047CE0D                 nop  
SDPI:0047CE0E                 nop  
SDPI:0047CE0F                 nop  
SDPI:0047CE10                 nop  
SDPI:0047CE11                 nop  
SDPI:0047CE12                 nop  
SDPI:0047CE13                 nop  
SDPI:0047CE14                 nop  
SDPI:0047CE15                 nop  
SDPI:0047CE16                 nop  
SDPI:0047CE17                 nop  
SDPI:0047CE18                 nop  
SDPI:0047CE19                 nop  
SDPI:0047CE1A                 nop  
SDPI:0047CE1B                 nop  
SDPI:0047CE1C                 nop  
SDPI:0047CE1D                 nop  
SDPI:0047CE1E                 nop  
SDPI:0047CE1F                 nop  
SDPI:0047CE20                 nop  
SDPI:0047CE21                 nop  
SDPI:0047CE22                 nop  
SDPI:0047CE23                 nop  
SDPI:0047CE24                 nop  
SDPI:0047CE25                 nop  
SDPI:0047CE26                 int     3                         ; Trap to Debugger  
SDPI:0047CE27                 nop  
SDPI:0047CE28                 cmp     al, 4  
SDPI:0047CE2A                 jz      short loc\_47CE9D  
SDPI:0047CE2C  
SDPI:0047CE2C ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047CE2C  
SDPI:0047CE2C  
SDPI:0047CE2C FNDDBG\_47CE2C   proc near                         ; CODE XREF: SDPI:0047CEB6 j  
SDPI:0047CE2C                                                   ; SDPI:0047CECE j ...  
SDPI:0047CE2C                 nop  
SDPI:0047CE2D                 nop  
SDPI:0047CE2E                 nop  
SDPI:0047CE2F                 nop  
SDPI:0047CE30                 nop  
SDPI:0047CE31                 call    loc\_47CE37  
SDPI:0047CE36                 nop  
SDPI:0047CE37  
SDPI:0047CE37 loc\_47CE37:                                       ; CODE XREF: FNDDBG\_47CE2C+5 p  
SDPI:0047CE37                 pop     eax  
SDPI:0047CE38                 add     eax, 5Eh  
SDPI:0047CE3D                 mov     edx, eax  
SDPI:0047CE3F                 add     edx, 32h  
SDPI:0047CE42                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047CE42                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047CE42                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047CE42                                                   ; 用来解密  
SDPI:0047CE47                 call    loc\_47CE4D  
SDPI:0047CE4C                 nop  
SDPI:0047CE4D  
SDPI:0047CE4D loc\_47CE4D:                                       ; CODE XREF: FNDDBG\_47CE2C+1B p  
SDPI:0047CE4D                 pop     eax  
SDPI:0047CE4E                 add     eax, 2250h  
SDPI:0047CE53                 call    loc\_47CE59  
SDPI:0047CE58                 nop  
SDPI:0047CE59  
SDPI:0047CE59 loc\_47CE59:                                       ; CODE XREF: FNDDBG\_47CE2C+27 p  
SDPI:0047CE59                 pop     ecx  
SDPI:0047CE5A                 add     ecx, 22F1h  
SDPI:0047CE60                 push    0  
SDPI:0047CE62                 push    ecx  
SDPI:0047CE63                 push    eax  
SDPI:0047CE64                 push    0  
SDPI:0047CE66                 call    loc\_47CE6C  
SDPI:0047CE6B                 nop  
SDPI:0047CE6C  
SDPI:0047CE6C loc\_47CE6C:                                       ; CODE XREF: FNDDBG\_47CE2C+3A p  
SDPI:0047CE6C                 pop     eax  
SDPI:0047CE6D                 add     eax, 11h  
SDPI:0047CE72                 push    eax  
SDPI:0047CE73                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047CE73                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047CE73                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047CE73 ; ----------------------------------------------------------------------------  
SDPI:0047CE78                 db 4 dup(90h)  
SDPI:0047CE7C ; ----------------------------------------------------------------------------  
SDPI:0047CE7C                 push    7  
SDPI:0047CE7E                 call    loc\_47CE84  
SDPI:0047CE83                 nop  
SDPI:0047CE84  
SDPI:0047CE84 loc\_47CE84:                                       ; CODE XREF: FNDDBG\_47CE2C+52 p  
SDPI:0047CE84                 pop     eax  
SDPI:0047CE85                 add     eax, 11h  
SDPI:0047CE8A                 push    eax  
SDPI:0047CE8B                 jmp     ExitProcess  
SDPI:0047CE8B ; ----------------------------------------------------------------------------  
SDPI:0047CE90                 db 4 dup(90h)  
SDPI:0047CE94                 db 0E8h, 0, 10h, 40h, 0  
SDPI:0047CE99                 db 0B0h, 89h, 9Ch, 4  
SDPI:0047CE99 FNDDBG\_47CE2C   endp  
SDPI:0047CE99  
SDPI:0047CE9D ; ----------------------------------------------------------------------------  
SDPI:0047CE9D  
SDPI:0047CE9D loc\_47CE9D:                                       ; CODE XREF: SDPI:0047CE2A j  
SDPI:0047CE9D                 pop     large dword ptr fs:0  
SDPI:0047CEA4                 add     esp, 4  
SDPI:0047CEA7                 call    loc\_47CEAD  
SDPI:0047CEAC                 nop  
SDPI:0047CEAD  
SDPI:0047CEAD loc\_47CEAD:                                       ; CODE XREF: SDPI:0047CEA7 p  
SDPI:0047CEAD                 pop     eax  
SDPI:0047CEAE                 add     eax, 0FFFFFE1Dh  
SDPI:0047CEB3                 cmp     byte ptr \[eax\], 0E9h  
SDPI:0047CEB6                 jnz     FNDDBG\_47CE2C  
SDPI:0047CEBC                 mov     byte ptr \[eax\], 0E8h  
SDPI:0047CEBF                 rdtsc  
SDPI:0047CEC1                 mov     ecx, eax  
SDPI:0047CEC3                 mov     ebx, edx  
SDPI:0047CEC5                 rdtsc  
SDPI:0047CEC7                 sub     eax, ecx  
SDPI:0047CEC9                 sbb     edx, ebx  
SDPI:0047CECB                 cmp     edx, 0  
SDPI:0047CECE                 jnz     FNDDBG\_47CE2C  
SDPI:0047CED4                 cmp     eax, 30000000h  
SDPI:0047CED9                 ja      FNDDBG\_47CE2C  
SDPI:0047CEDF                 jz      short loc\_47CF2A  
SDPI:0047CEE1                 jnz     short loc\_47CF2A  
SDPI:0047CEE1 ; ----------------------------------------------------------------------------  
SDPI:0047CEE3                 dd 401000E8h  
SDPI:0047CEE7                 dd 9C89B000h  
SDPI:0047CEEB                 db 4  
SDPI:0047CEEC ; ----------------------------------------------------------------------------  
SDPI:0047CEEC                 mov     eax, \[esp+4\]  
SDPI:0047CEF0                 mov     ecx, \[esp+0Ch\]  
SDPI:0047CEF4                 inc     dword ptr \[ecx+0B8h\]  
SDPI:0047CEFA                 mov     eax, \[eax\]  
SDPI:0047CEFC                 sub     eax, EXCEPTION\_BREAKPOINT  
SDPI:0047CF01                 jnz     short locret\_47CF29  
SDPI:0047CF03                 call    loc\_47CF09  
SDPI:0047CF08                 nop  
SDPI:0047CF09  
SDPI:0047CF09 loc\_47CF09:                                       ; CODE XREF: SDPI:0047CF03 p  
SDPI:0047CF09                 pop     eax  
SDPI:0047CF0A                 add     eax, 0FFFFFDC1h  
SDPI:0047CF0F                 cmp     byte ptr \[eax\], 0E8h  
SDPI:0047CF12                 jnz     FNDDBG\_47CE2C  
SDPI:0047CF18                 mov     byte ptr \[eax\], 0E9h  
SDPI:0047CF1B                 xor     eax, eax  
SDPI:0047CF1D                 mov     \[ecx+4\], eax  
SDPI:0047CF20                 mov     \[ecx+8\], eax  
SDPI:0047CF23                 mov     \[ecx+0Ch\], eax  
SDPI:0047CF26                 mov     \[ecx+10h\], eax  
SDPI:0047CF29  
SDPI:0047CF29 locret\_47CF29:                                    ; CODE XREF: SDPI:0047CF01 j  
SDPI:0047CF29                 retn  
SDPI:0047CF2A ; ----------------------------------------------------------------------------  
SDPI:0047CF2A  
SDPI:0047CF2A loc\_47CF2A:                                       ; CODE XREF: SDPI:0047CEDF j  
SDPI:0047CF2A                                                   ; SDPI:0047CEE1 j  
SDPI:0047CF2A                 pop     eax  
SDPI:0047CF2B                 call    loc\_47D12B  
SDPI:0047CF2B ; ----------------------------------------------------------------------------  
SDPI:0047CF30                 dd 401000h  
SDPI:0047CF34                 dd 15C56BEh  
SDPI:0047CF38  
SDPI:0047CF38 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047CF38  
SDPI:0047CF38  
SDPI:0047CF38 sub\_47CF38      proc near                         ; CODE XREF: SDPI:0047D154 p  
SDPI:0047CF38                 pop     ebp  
SDPI:0047CF39                 pop     eax  
SDPI:0047CF3A                 jmp     loc\_47D159  
SDPI:0047CF3A sub\_47CF38      endp  
SDPI:0047CF3A  
SDPI:0047CF3F ; ----------------------------------------------------------------------------  
SDPI:0047CF3F                 call    loc\_47CF45  
SDPI:0047CF44                 nop  
SDPI:0047CF45  
SDPI:0047CF45 loc\_47CF45:                                       ; CODE XREF: SDPI:0047CF3F p  
SDPI:0047CF45                 pop     eax  
SDPI:0047CF46                 add     eax, 312h  
SDPI:0047CF4B                 call    loc\_47CF51  
SDPI:0047CF50                 nop  
SDPI:0047CF51  
SDPI:0047CF51 loc\_47CF51:                                       ; CODE XREF: SDPI:0047CF4B p  
SDPI:0047CF51                 pop     edx  
SDPI:0047CF52                 add     edx, 3C2h                 ; 和前面的一样了，加密代码回去  
SDPI:0047CF58                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047CF58                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047CF58                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047CF58                                                   ; 用来解密  
SDPI:0047CF5D                 mov     ecx, 0FFFFFF00h  
SDPI:0047CF62                 push    fs  
SDPI:0047CF64                 nop  
SDPI:0047CF65                 nop  
SDPI:0047CF66                 nop  
SDPI:0047CF67                 nop  
SDPI:0047CF68                 nop  
SDPI:0047CF69                 nop  
SDPI:0047CF6A                 nop  
SDPI:0047CF6B                 nop  
SDPI:0047CF6C                 nop  
SDPI:0047CF6D                 nop  
SDPI:0047CF6E                 nop  
SDPI:0047CF6F                 nop  
SDPI:0047CF70                 pushfw  
SDPI:0047CF72                 push    eax  
SDPI:0047CF73                 mov     eax, ebx  
SDPI:0047CF75                 push    ebx  
SDPI:0047CF76                 mov     eax, ecx  
SDPI:0047CF78                 push    eax  
SDPI:0047CF79                 add     eax, edx  
SDPI:0047CF7B                 mov     ebx, eax  
SDPI:0047CF7D                 push    ebx  
SDPI:0047CF7E                 pop     eax  
SDPI:0047CF7F                 push    edx  
SDPI:0047CF80                 call    loc\_47CF8D  
SDPI:0047CF80 ; ----------------------------------------------------------------------------  
SDPI:0047CF85                 dd 401000h  
SDPI:0047CF89                 dd 132BD7B0h  
SDPI:0047CF8D ; ----------------------------------------------------------------------------  
SDPI:0047CF8D  
SDPI:0047CF8D loc\_47CF8D:                                       ; CODE XREF: SDPI:0047CF80 p  
SDPI:0047CF8D                 pop     eax  
SDPI:0047CF8E                 call    loc\_47CF94  
SDPI:0047CF93                 nop  
SDPI:0047CF94  
SDPI:0047CF94 loc\_47CF94:                                       ; CODE XREF: SDPI:0047CF8E p  
SDPI:0047CF94                 pop     eax  
SDPI:0047CF95                 add     eax, 11h  
SDPI:0047CF9A                 push    eax  
SDPI:0047CF9B                 jmp     GetTickCount  
SDPI:0047CF9B ; ----------------------------------------------------------------------------  
SDPI:0047CFA0                 db 4 dup(90h)  
SDPI:0047CFA4 ; ----------------------------------------------------------------------------  
SDPI:0047CFA4                 push    eax  
SDPI:0047CFA5                 mov     eax, edx  
SDPI:0047CFA7                 push    eax  
SDPI:0047CFA8                 call    loc\_47CFAE  
SDPI:0047CFAD                 nop  
SDPI:0047CFAE  
SDPI:0047CFAE loc\_47CFAE:                                       ; CODE XREF: SDPI:0047CFA8 p  
SDPI:0047CFAE                 pop     edx  
SDPI:0047CFAF                 add     edx, 52h  
SDPI:0047CFB5                 push    edx  
SDPI:0047CFB6                 add     edx, 403FB6h  
SDPI:0047CFBC                 push    edx  
SDPI:0047CFBD                 jo      short loc\_47D012  
SDPI:0047CFBF                 jno     short loc\_47D012  
SDPI:0047CFC1  
SDPI:0047CFC1 loc\_47CFC1:                                       ; CODE XREF: SDPI:0047D005 p  
SDPI:0047CFC1                 pop     eax  
SDPI:0047CFC2                 pop     ebx  
SDPI:0047CFC3                 call    loc\_47CFC9  
SDPI:0047CFC8                 nop  
SDPI:0047CFC9  
SDPI:0047CFC9 loc\_47CFC9:                                       ; CODE XREF: SDPI:0047CFC3 p  
SDPI:0047CFC9                 pop     eax  
SDPI:0047CFCA                 add     eax, 11h  
SDPI:0047CFCF                 push    eax  
SDPI:0047CFD0                 jmp     GetTickCount  
SDPI:0047CFD0 ; ----------------------------------------------------------------------------  
SDPI:0047CFD5                 db 4 dup(90h)  
SDPI:0047CFD9 ; ----------------------------------------------------------------------------  
SDPI:0047CFD9                 pop     ebx  
SDPI:0047CFDA                 add     ebx, 1F4h  
SDPI:0047CFE0                 sub     ebx, eax  
SDPI:0047CFE2                 js      short OVER\_47D026  
SDPI:0047CFE4                 call    loc\_47CFEA  
SDPI:0047CFE9                 nop  
SDPI:0047CFEA  
SDPI:0047CFEA loc\_47CFEA:                                       ; CODE XREF: SDPI:0047CFE4 p  
SDPI:0047CFEA                 pop     ebx  
SDPI:0047CFEB                 add     ebx, 0A5h  
SDPI:0047CFF1                 push    ebx  
SDPI:0047CFF2                 call    loc\_47D01C  
SDPI:0047CFF7                 nop  
SDPI:0047CFF8                 nop  
SDPI:0047CFF9                 nop  
SDPI:0047CFFA                 nop  
SDPI:0047CFFB                 nop  
SDPI:0047CFFC                 nop  
SDPI:0047CFFD                 nop  
SDPI:0047CFFE                 nop  
SDPI:0047CFFF                 pop     eax  
SDPI:0047D000                 mov     edx, eax  
SDPI:0047D002                 mov     eax, ebx  
SDPI:0047D004                 push    eax  
SDPI:0047D005                 call    loc\_47CFC1  
SDPI:0047D005 ; ----------------------------------------------------------------------------  
SDPI:0047D00A                 dd 401000h  
SDPI:0047D00E                 dd 1833639h  
SDPI:0047D012 ; ----------------------------------------------------------------------------  
SDPI:0047D012  
SDPI:0047D012 loc\_47D012:                                       ; CODE XREF: SDPI:0047CFBD j  
SDPI:0047D012                                                   ; SDPI:0047CFBF j  
SDPI:0047D012                 pop     eax  
SDPI:0047D013                 retn  
SDPI:0047D013 ; ----------------------------------------------------------------------------  
SDPI:0047D014                 dd 401000h  
SDPI:0047D018                 dd 77C563Eh  
SDPI:0047D01C ; ----------------------------------------------------------------------------  
SDPI:0047D01C  
SDPI:0047D01C loc\_47D01C:                                       ; CODE XREF: SDPI:0047CFF2 p  
SDPI:0047D01C                 pop     edx  
SDPI:0047D01D                 retn  
SDPI:0047D01D ; ----------------------------------------------------------------------------  
SDPI:0047D01E                 dd 401000h  
SDPI:0047D022                 dd 1ED53EFh  
SDPI:0047D026  
SDPI:0047D026 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047D026  
SDPI:0047D026  
SDPI:0047D026 OVER\_47D026     proc near                         ; CODE XREF: SDPI:0047CFE2 j  
SDPI:0047D026                 nop  
SDPI:0047D027                 nop  
SDPI:0047D028                 nop  
SDPI:0047D029                 nop  
SDPI:0047D02A                 nop  
SDPI:0047D02B                 call    loc\_47D031  
SDPI:0047D030                 nop  
SDPI:0047D031  
SDPI:0047D031 loc\_47D031:                                       ; CODE XREF: OVER\_47D026+5 p  
SDPI:0047D031                 pop     eax  
SDPI:0047D032                 add     eax, 5Eh  
SDPI:0047D037                 mov     edx, eax  
SDPI:0047D039                 add     edx, 32h  
SDPI:0047D03C                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047D03C                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047D03C                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047D03C                                                   ; 用来解密  
SDPI:0047D041                 call    loc\_47D047  
SDPI:0047D046                 nop  
SDPI:0047D047  
SDPI:0047D047 loc\_47D047:                                       ; CODE XREF: OVER\_47D026+1B p  
SDPI:0047D047                 pop     eax  
SDPI:0047D048                 add     eax, 2056h  
SDPI:0047D04D                 call    loc\_47D053  
SDPI:0047D052                 nop  
SDPI:0047D053  
SDPI:0047D053 loc\_47D053:                                       ; CODE XREF: OVER\_47D026+27 p  
SDPI:0047D053                 pop     ecx  
SDPI:0047D054                 add     ecx, 20F7h  
SDPI:0047D05A                 push    0  
SDPI:0047D05C                 push    ecx  
SDPI:0047D05D                 push    eax  
SDPI:0047D05E                 push    0  
SDPI:0047D060                 call    loc\_47D066  
SDPI:0047D065                 nop  
SDPI:0047D066  
SDPI:0047D066 loc\_47D066:                                       ; CODE XREF: OVER\_47D026+3A p  
SDPI:0047D066                 pop     eax  
SDPI:0047D067                 add     eax, 11h  
SDPI:0047D06C                 push    eax  
SDPI:0047D06D                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047D06D                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047D06D                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047D06D ; ----------------------------------------------------------------------------  
SDPI:0047D072                 db 4 dup(90h)  
SDPI:0047D076 ; ----------------------------------------------------------------------------  
SDPI:0047D076                 push    7  
SDPI:0047D078                 call    loc\_47D07E  
SDPI:0047D07D                 nop  
SDPI:0047D07E  
SDPI:0047D07E loc\_47D07E:                                       ; CODE XREF: OVER\_47D026+52 p  
SDPI:0047D07E                 pop     eax  
SDPI:0047D07F                 add     eax, 11h  
SDPI:0047D084                 push    eax  
SDPI:0047D085                 jmp     ExitProcess  
SDPI:0047D085 ; ----------------------------------------------------------------------------  
SDPI:0047D08A                 db 4 dup(90h)  
SDPI:0047D08A OVER\_47D026     endp  
SDPI:0047D08A  
SDPI:0047D08E ; ----------------------------------------------------------------------------  
SDPI:0047D08E                 pop     edx  
SDPI:0047D08F                 mov     eax, ecx  
SDPI:0047D091                 add     eax, edx  
SDPI:0047D093                 inc     ecx  
SDPI:0047D094                 push    eax  
SDPI:0047D095                 inc     ecx  
SDPI:0047D096                 pop     ebx  
SDPI:0047D097                 pop     ecx  
SDPI:0047D098                 push    eax  
SDPI:0047D099                 sub     eax, 8  
SDPI:0047D09C                 pop     ebx  
SDPI:0047D09D                 pop     ebx  
SDPI:0047D09E                 inc     eax  
SDPI:0047D09F                 add     eax, ebx  
SDPI:0047D0A1                 pop     eax  
SDPI:0047D0A2                 pushfw  
SDPI:0047D0A4                 popfw  
SDPI:0047D0A6                 popfw  
SDPI:0047D0A8                 pop     es  
SDPI:0047D0A9                 mov     eax, 12345678h  
SDPI:0047D0AE                 push    eax  
SDPI:0047D0AF                 call    loc\_47D0B5  
SDPI:0047D0B4                 nop  
SDPI:0047D0B5  
SDPI:0047D0B5 loc\_47D0B5:                                       ; CODE XREF: SDPI:0047D0AF p  
SDPI:0047D0B5                 pop     eax  
SDPI:0047D0B6                 add     eax, 12Ch  
SDPI:0047D0BB                 push    eax  
SDPI:0047D0BC                 pop     ebx  
SDPI:0047D0BD                 add     eax, 12h  
SDPI:0047D0C0                 pop     edx  
SDPI:0047D0C1                 add     eax, edx  
SDPI:0047D0C3                 mov     edx, eax  
SDPI:0047D0C5                 push    ebx  
SDPI:0047D0C6                 mov     ebx, es:\[ecx+100h\]  
SDPI:0047D0CD                 push    ebx  
SDPI:0047D0CE                 mov     eax, esp  
SDPI:0047D0D0                 mov     ebx, eax  
SDPI:0047D0D2                 push    ebx  
SDPI:0047D0D3                 pop     edx  
SDPI:0047D0D4                 mov     es:\[ecx+100h\], eax  
SDPI:0047D0DB                 xor     eax, eax  
SDPI:0047D0DB ; ----------------------------------------------------------------------------  
SDPI:0047D0DD JUNK\_47D0DD     db '~',7,'',5,0,10h,'@',0,'鑖淨 3 蒰?,5,0,10h,'@',0,'鑉悙悙悙悙?  
SDPI:0047D0DD                 db '悙悙悙悙悙悙悙 f 漽',6,'{',4,0,10h,'@',0  
SDPI:0047D115 ; ----------------------------------------------------------------------------  
SDPI:0047D115                 int     3                         ; Trap to Debugger  
SDPI:0047D116                 nop  
SDPI:0047D117                 xor     eax, eax  
SDPI:0047D119                 mov     dword ptr \[eax\], 404219h  
SDPI:0047D11F                 jp      short loc\_47D12B  
SDPI:0047D121                 jnp     short loc\_47D12B  
SDPI:0047D121 ; ----------------------------------------------------------------------------  
SDPI:0047D123                 dd 401000h  
SDPI:0047D127                 dd 403D7Bh  
SDPI:0047D12B ; ----------------------------------------------------------------------------  
SDPI:0047D12B  
SDPI:0047D12B loc\_47D12B:                                       ; CODE XREF: SDPI:0047CF2B p  
SDPI:0047D12B                                                   ; SDPI:0047D11F j ...  
SDPI:0047D12B                 call    loc\_47D131  
SDPI:0047D130                 nop  
SDPI:0047D131  
SDPI:0047D131 loc\_47D131:                                       ; CODE XREF: SDPI:loc\_47D12B p  
SDPI:0047D131                 pop     eax  
SDPI:0047D132                 add     eax, 11h  
SDPI:0047D137                 push    eax  
SDPI:0047D138                 jmp     GetTickCount  
SDPI:0047D138 ; ----------------------------------------------------------------------------  
SDPI:0047D13D                 db 4 dup(90h)  
SDPI:0047D141 ; ----------------------------------------------------------------------------  
SDPI:0047D141                 call    loc\_47D147  
SDPI:0047D146                 nop  
SDPI:0047D147  
SDPI:0047D147 loc\_47D147:                                       ; CODE XREF: SDPI:0047D141 p  
SDPI:0047D147                 pop     edx  
SDPI:0047D148                 add     edx, 0FFFFFB09h  
SDPI:0047D14E                 mov     \[edx\], eax  
SDPI:0047D150                 pop     ebp  
SDPI:0047D151                 add     eax, edx  
SDPI:0047D153                 push    eax  
SDPI:0047D154                 call    sub\_47CF38  
SDPI:0047D159  
SDPI:0047D159 loc\_47D159:                                       ; CODE XREF: sub\_47CF38+2 j  
SDPI:0047D159                 call    loc\_47D15F  
SDPI:0047D15E                 nop  
SDPI:0047D15F  
SDPI:0047D15F loc\_47D15F:                                       ; CODE XREF: SDPI:loc\_47D159 p  
SDPI:0047D15F                 pop     edx  
SDPI:0047D160                 add     edx, 0FFFFFDE1h  
SDPI:0047D166                 add     edx, eax  
SDPI:0047D168                 push    edx  
SDPI:0047D169                 pop     ecx  
SDPI:0047D16A                 sub     ecx, eax  
SDPI:0047D16C                 push    ecx  
SDPI:0047D16D                 retn    4  
SDPI:0047D170  
SDPI:0047D170 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047D170  
SDPI:0047D170  
SDPI:0047D170 FNDDBG\_47D170   proc near                         ; CODE XREF: SDPI:0047D213 j  
SDPI:0047D170                                                   ; SDPI:0047D21B j ...  
SDPI:0047D170                 nop  
SDPI:0047D171                 nop  
SDPI:0047D172                 nop  
SDPI:0047D173                 nop  
SDPI:0047D174                 nop  
SDPI:0047D175                 call    loc\_47D17B  
SDPI:0047D17A                 nop  
SDPI:0047D17B  
SDPI:0047D17B loc\_47D17B:                                       ; CODE XREF: FNDDBG\_47D170+5 p  
SDPI:0047D17B                 pop     eax  
SDPI:0047D17C                 add     eax, 5Eh  
SDPI:0047D181                 mov     edx, eax  
SDPI:0047D183                 add     edx, 32h  
SDPI:0047D186                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047D186                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047D186                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047D186                                                   ; 用来解密  
SDPI:0047D18B                 call    loc\_47D191  
SDPI:0047D190                 nop  
SDPI:0047D191  
SDPI:0047D191 loc\_47D191:                                       ; CODE XREF: FNDDBG\_47D170+1B p  
SDPI:0047D191                 pop     eax  
SDPI:0047D192                 add     eax, 1F0Ch  
SDPI:0047D197                 call    loc\_47D19D  
SDPI:0047D19C                 nop  
SDPI:0047D19D  
SDPI:0047D19D loc\_47D19D:                                       ; CODE XREF: FNDDBG\_47D170+27 p  
SDPI:0047D19D                 pop     ecx  
SDPI:0047D19E                 add     ecx, 1FADh  
SDPI:0047D1A4                 push    0  
SDPI:0047D1A6                 push    ecx  
SDPI:0047D1A7                 push    eax  
SDPI:0047D1A8                 push    0  
SDPI:0047D1AA                 call    loc\_47D1B0  
SDPI:0047D1AF                 nop  
SDPI:0047D1B0  
SDPI:0047D1B0 loc\_47D1B0:                                       ; CODE XREF: FNDDBG\_47D170+3A p  
SDPI:0047D1B0                 pop     eax  
SDPI:0047D1B1                 add     eax, 11h  
SDPI:0047D1B6                 push    eax  
SDPI:0047D1B7                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047D1B7                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047D1B7                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047D1B7 ; ----------------------------------------------------------------------------  
SDPI:0047D1BC                 db 4 dup(90h)  
SDPI:0047D1C0 ; ----------------------------------------------------------------------------  
SDPI:0047D1C0                 push    7  
SDPI:0047D1C2                 call    loc\_47D1C8  
SDPI:0047D1C7                 nop  
SDPI:0047D1C8  
SDPI:0047D1C8 loc\_47D1C8:                                       ; CODE XREF: FNDDBG\_47D170+52 p  
SDPI:0047D1C8                 pop     eax  
SDPI:0047D1C9                 add     eax, 11h  
SDPI:0047D1CE                 push    eax  
SDPI:0047D1CF                 jmp     ExitProcess  
SDPI:0047D1CF ; ----------------------------------------------------------------------------  
SDPI:0047D1D4                 db 4 dup(90h)  
SDPI:0047D1D8                 dd 401000h  
SDPI:0047D1DC                 dd 15C56BEh  
SDPI:0047D1DC FNDDBG\_47D170   endp  
SDPI:0047D1DC  
SDPI:0047D1E0 ; ----------------------------------------------------------------------------  
SDPI:0047D1E0                 mov     esp, \[esp+8\]  
SDPI:0047D1E4                 pop     large dword ptr fs:0  
SDPI:0047D1EB                 call    loc\_47D1F1  
SDPI:0047D1F0                 nop  
SDPI:0047D1F1  
SDPI:0047D1F1 loc\_47D1F1:                                       ; CODE XREF: SDPI:0047D1EB p  
SDPI:0047D1F1                 pop     eax  
SDPI:0047D1F2                 add     eax, 11h  
SDPI:0047D1F7                 push    eax  
SDPI:0047D1F8                 jmp     GetTickCount  
SDPI:0047D1F8 ; ----------------------------------------------------------------------------  
SDPI:0047D1FD                 db 4 dup(90h)  
SDPI:0047D201 ; ----------------------------------------------------------------------------  
SDPI:0047D201                 call    loc\_47D207  
SDPI:0047D206                 nop  
SDPI:0047D207  
SDPI:0047D207 loc\_47D207:                                       ; CODE XREF: SDPI:0047D201 p  
SDPI:0047D207                 pop     edx  
SDPI:0047D208                 add     edx, 0FFFFFA49h  
SDPI:0047D20E                 mov     ecx, \[edx\]  
SDPI:0047D210                 cmp     ecx, 0  
SDPI:0047D213                 jz      FNDDBG\_47D170  
SDPI:0047D219                 sub     eax, ecx  
SDPI:0047D21B                 js      FNDDBG\_47D170  
SDPI:0047D221                 sub     eax, 7D0h  
SDPI:0047D226                 jns     FNDDBG\_47D170  
SDPI:0047D22C                 mov     eax, 0E801276h  
SDPI:0047D231                 mov     \[edx\], eax  
SDPI:0047D233                 call    loc\_47D239  
SDPI:0047D238                 nop  
SDPI:0047D239  
SDPI:0047D239 loc\_47D239:                                       ; CODE XREF: SDPI:0047D233 p  
SDPI:0047D239                 pop     edx  
SDPI:0047D23A                 add     edx, 0DAh  
SDPI:0047D240                 call    loc\_47D246  
SDPI:0047D245                 nop  
SDPI:0047D246  
SDPI:0047D246 loc\_47D246:                                       ; CODE XREF: SDPI:0047D240 p  
SDPI:0047D246                 pop     eax  
SDPI:0047D247                 add     eax, 0FFFFCF53h  
SDPI:0047D24C                 mov     ecx, 10h                  ; 再次解开代码，  
SDPI:0047D24C                                                   ; 这里全都是 Win9x 下的调试器检测  
SDPI:0047D251                 call    De\_Code                   ; invoke De\_Code,ecx(counter),eax(key),edx(end)  
SDPI:0047D251                                                   ; 用于解密代码，解密起始地址就是 call 进来  
SDPI:0047D251                                                   ; 的下一行代码地址  
SDPI:0047D256                 sub     esp, 8  
SDPI:0047D259                 sidt    qword ptr \[esp\]           ; 在 Win9x 下检测检测调试器  
SDPI:0047D25D                 mov     eax, \[esp+2\]  
SDPI:0047D261                 mov     cx, \[eax+0Eh\]  
SDPI:0047D265                 mov     dx, \[eax+6\]  
SDPI:0047D269                 mov     bx, \[eax+1Eh\]  
SDPI:0047D26D                 add     esp, 8  
SDPI:0047D270                 cmp     cx, dx  
SDPI:0047D273                 jnz     short FNDDBG\_47D2AA  
SDPI:0047D275                 cmp     bx, dx  
SDPI:0047D278                 jnz     short FNDDBG\_47D2AA  
SDPI:0047D27A                 sub     esp, 8  
SDPI:0047D27D                 sidt    qword ptr \[esp\]  
SDPI:0047D281                 mov     edx, \[esp+2\]  
SDPI:0047D285                 add     edx, 4Eh  
SDPI:0047D288                 mov     edx, \[edx\]  
SDPI:0047D28A                 ror     edx, 10h  
SDPI:0047D28D                 mov     ecx, 0C00h  
SDPI:0047D292                 add     esp, 8  
SDPI:0047D295  
SDPI:0047D295 loc\_47D295:                                       ; CODE XREF: SDPI:0047D2A6 j  
SDPI:0047D295                 cmp     dword ptr \[edx\], 48455245h  
SDPI:0047D29B                 jz      short FNDDBG\_47D2AA  
SDPI:0047D29D                 cmp     dword ptr \[edx\], 53474F52h  
SDPI:0047D2A3                 jz      short FNDDBG\_47D2AA  
SDPI:0047D2A5                 inc     edx  
SDPI:0047D2A6                 loop    loc\_47D295  
SDPI:0047D2A8                 jmp     short NotIsWin9x\_47D312   ; Win9x 下检测调试器完毕后跳去  
SDPI:0047D2A8                                                   ; 下一步，如果是 WINNT 系统则跳过  
SDPI:0047D2A8                                                   ; Win9x 下的调试器检测部分  
SDPI:0047D2AA  
SDPI:0047D2AA ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047D2AA  
SDPI:0047D2AA  
SDPI:0047D2AA FNDDBG\_47D2AA   proc near                         ; CODE XREF: SDPI:0047D273 j  
SDPI:0047D2AA                                                   ; SDPI:0047D278 j ...  
SDPI:0047D2AA                 nop  
SDPI:0047D2AB                 nop  
SDPI:0047D2AC                 nop  
SDPI:0047D2AD                 nop  
SDPI:0047D2AE                 nop  
SDPI:0047D2AF                 call    loc\_47D2B5  
SDPI:0047D2B4                 nop  
SDPI:0047D2B5  
SDPI:0047D2B5 loc\_47D2B5:                                       ; CODE XREF: FNDDBG\_47D2AA+5 p  
SDPI:0047D2B5                 pop     eax  
SDPI:0047D2B6                 add     eax, 5Eh  
SDPI:0047D2BB                 mov     edx, eax  
SDPI:0047D2BD                 add     edx, 32h  
SDPI:0047D2C0                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047D2C0                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047D2C0                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047D2C0                                                   ; 用来解密  
SDPI:0047D2C5                 call    loc\_47D2CB  
SDPI:0047D2CA                 nop  
SDPI:0047D2CB  
SDPI:0047D2CB loc\_47D2CB:                                       ; CODE XREF: FNDDBG\_47D2AA+1B p  
SDPI:0047D2CB                 pop     eax  
SDPI:0047D2CC                 add     eax, 1DD2h  
SDPI:0047D2D1                 call    loc\_47D2D7  
SDPI:0047D2D6                 nop  
SDPI:0047D2D7  
SDPI:0047D2D7 loc\_47D2D7:                                       ; CODE XREF: FNDDBG\_47D2AA+27 p  
SDPI:0047D2D7                 pop     ecx  
SDPI:0047D2D8                 add     ecx, 1E73h  
SDPI:0047D2DE                 push    0  
SDPI:0047D2E0                 push    ecx  
SDPI:0047D2E1                 push    eax  
SDPI:0047D2E2                 push    0  
SDPI:0047D2E4                 call    loc\_47D2EA  
SDPI:0047D2E9                 nop  
SDPI:0047D2EA  
SDPI:0047D2EA loc\_47D2EA:                                       ; CODE XREF: FNDDBG\_47D2AA+3A p  
SDPI:0047D2EA                 pop     eax  
SDPI:0047D2EB                 add     eax, 11h  
SDPI:0047D2F0                 push    eax  
SDPI:0047D2F1                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047D2F1                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047D2F1                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047D2F1 ; ----------------------------------------------------------------------------  
SDPI:0047D2F6                 db 4 dup(90h)  
SDPI:0047D2FA ; ----------------------------------------------------------------------------  
SDPI:0047D2FA                 push    7  
SDPI:0047D2FC                 call    loc\_47D302  
SDPI:0047D301                 nop  
SDPI:0047D302  
SDPI:0047D302 loc\_47D302:                                       ; CODE XREF: FNDDBG\_47D2AA+52 p  
SDPI:0047D302                 pop     eax  
SDPI:0047D303                 add     eax, 11h  
SDPI:0047D308                 push    eax  
SDPI:0047D309                 jmp     ExitProcess  
SDPI:0047D309 ; ----------------------------------------------------------------------------  
SDPI:0047D30E                 db 4 dup(90h)                     ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047D30E FNDDBG\_47D2AA   endp                              ;            Win9x 调试器检测完毕  
SDPI:0047D30E                                                   ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047D312 ; ----------------------------------------------------------------------------  
SDPI:0047D312  
??  
SDPI:0047D312 NotIsWin9x\_47D312:                                ; CODE XREF: SDPI:0047C53E j  
SDPI:0047D312                                                   ; SDPI:0047CB96 j ...  
SDPI:0047D312                 call    loc\_47D318  
SDPI:0047D317                 nop  
SDPI:0047D318  
SDPI:0047D318 loc\_47D318:                                       ; CODE XREF: SDPI:NotIsWin9x\_47D312 p  
SDPI:0047D318                 pop     eax  
SDPI:0047D319                 add     eax, 5Ah  
SDPI:0047D31E                 call    loc\_47D324                ; EAX=0047D371,EDX=0047D3B7  
SDPI:0047D323                 nop  
SDPI:0047D324  
SDPI:0047D324 loc\_47D324:                                       ; CODE XREF: SDPI:0047D31E p  
SDPI:0047D324                 pop     edx                       ; EAX=0047D371,EDX=0047D3B7  
SDPI:0047D325                 add     edx, 94h                  ; 再次加密代码回去  
SDPI:0047D32B                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047D32B                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047D32B                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047D32B                                                   ; 用来解密  
SDPI:0047D330                 call    loc\_47D336  
SDPI:0047D335                 nop  
SDPI:0047D336  
SDPI:0047D336 loc\_47D336:                                       ; CODE XREF: SDPI:0047D330 p  
SDPI:0047D336                 pop     eax  
SDPI:0047D337                 add     eax, 21AFh  
SDPI:0047D33C                 call    loc\_47D342                ; EAX=0047F4E4,edx=00482DE2  
SDPI:0047D341                 nop  
SDPI:0047D342  
SDPI:0047D342 loc\_47D342:                                       ; CODE XREF: SDPI:0047D33C p  
SDPI:0047D342                 pop     edx                       ; EAX=0047F4E4,edx=00482DE2  
SDPI:0047D343                 add     edx, 5AA1h  
SDPI:0047D349                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047D349                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047D349                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047D349                                                   ; 用来解密  
SDPI:0047D34E                 call    loc\_47D354  
SDPI:0047D353                 nop  
SDPI:0047D354  
SDPI:0047D354 loc\_47D354:                                       ; CODE XREF: SDPI:0047D34E p  
SDPI:0047D354                 pop     edx  
SDPI:0047D355                 add     edx, 64h  
SDPI:0047D35B                 call    loc\_47D361  
SDPI:0047D360                 nop  
SDPI:0047D361  
SDPI:0047D361 loc\_47D361:                                       ; CODE XREF: SDPI:0047D35B p  
SDPI:0047D361                 pop     eax  
SDPI:0047D362                 add     eax, 0FFFFCE38h  
SDPI:0047D367                 mov     ecx, 10h  
SDPI:0047D36C                 call    De\_Code                   ; invoke De\_Code,ecx(counter),eax(key),edx(end)  
SDPI:0047D36C                                                   ; 用于解密代码，解密起始地址就是 call 进来  
SDPI:0047D36C                                                   ; 的下一行代码地址  
SDPI:0047D371                 call    loc\_47D377  
SDPI:0047D376                 nop  
SDPI:0047D377  
SDPI:0047D377 loc\_47D377:                                       ; CODE XREF: SDPI:0047D371 p  
SDPI:0047D377                 pop     eax  
SDPI:0047D378                 add     eax, 216Eh  
SDPI:0047D37D                 call    loc\_47D383                ; EAX=0047F4E4,edx=00482DE2  
SDPI:0047D382                 nop  
SDPI:0047D383  
SDPI:0047D383 loc\_47D383:                                       ; CODE XREF: SDPI:0047D37D p  
SDPI:0047D383                 pop     edx                       ; EAX=0047F4E4,edx=00482DE2  
SDPI:0047D384                 add     edx, 5A60h                ; 这里把上面那次加密代码解回去  
SDPI:0047D38A                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047D38A                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047D38A                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047D38A                                                   ; 用来解密  
SDPI:0047D38F                 call    loc\_47D395  
SDPI:0047D394                 nop  
SDPI:0047D395  
SDPI:0047D395 loc\_47D395:                                       ; CODE XREF: SDPI:0047D38F p  
SDPI:0047D395                 pop     edx  
SDPI:0047D396                 add     edx, 0FFFFCEA4h  
SDPI:0047D39C                 mov     eax, 0E821C800h  
SDPI:0047D3A1                 mov     \[edx\], eax  
SDPI:0047D3A3                 call    loc\_47D3A9  
SDPI:0047D3A8                 nop  
SDPI:0047D3A9  
SDPI:0047D3A9 loc\_47D3A9:                                       ; CODE XREF: SDPI:0047D3A3 p  
SDPI:0047D3A9                 pop     edx  
SDPI:0047D3AA                 add     edx, 0FFFFCC5Ch  
SDPI:0047D3B0                 mov     eax, 0E8673219h  
SDPI:0047D3B5                 mov     \[edx\], eax  
SDPI:0047D3B5 ; ----------------------------------------------------------------------------  
SDPI:0047D3B7 jUNK\_47D3B7     db 'p',0Eh,'q',0Ch,0,10h,'@',0,'縑|!v',12h,'€'  
SDPI:0047D3C6 ; ----------------------------------------------------------------------------  
SDPI:0047D3C6                 push    cs  
SDPI:0047D3C7                 mov     ecx, 769E3CF2h  
SDPI:0047D3CC                 call    loc\_47D3D2  
SDPI:0047D3D1                 nop  
SDPI:0047D3D2  
SDPI:0047D3D2 loc\_47D3D2:                                       ; CODE XREF: SDPI:0047D3CC p  
SDPI:0047D3D2                 pop     eax  
SDPI:0047D3D3                 add     eax, 5FEh  
SDPI:0047D3D8                 call    loc\_47D3DE  
SDPI:0047D3DD                 nop  
SDPI:0047D3DE  
SDPI:0047D3DE loc\_47D3DE:                                       ; CODE XREF: SDPI:0047D3D8 p  
SDPI:0047D3DE                 pop     edx  
SDPI:0047D3DF                 add     edx, 94Fh                 ; EAX=0047D9CF,EDX=0047DD2C  
SDPI:0047D3E5                 call    Crypt\_Code                ; MD5 值加密代码,调用方法:  
SDPI:0047D3E5                                                   ; invoke Crypt\_Code,End,Start  
SDPI:0047D3E5                                                   ; end 加密结束地址,地址在 EDX 中  
SDPI:0047D3E5                                                   ; start 加密起始地址,地址在 eax 中  
SDPI:0047D3E5                                                   ;  
SDPI:0047D3EA                 push    eax  
SDPI:0047D3EB                 xor     eax, eax  
SDPI:0047D3ED                 call    loc\_47D3F3  
SDPI:0047D3F2                 nop  
SDPI:0047D3F3  
SDPI:0047D3F3 loc\_47D3F3:                                       ; CODE XREF: SDPI:0047D3ED p  
SDPI:0047D3F3                 pop     edi  
SDPI:0047D3F4                 add     edi, 61h  
SDPI:0047D3FA                 mov     ebx, \[edi\]  
SDPI:0047D3FC                 mov     edx, \[edi+4\]  
SDPI:0047D3FF                 jz      short loc\_47D40B  
SDPI:0047D401                 jnz     short loc\_47D40B  
SDPI:0047D401 ; ----------------------------------------------------------------------------  
SDPI:0047D403 a@L             db 0,10h,'@',0,0Bh,'',9  
SDPI:0047D40B ; ----------------------------------------------------------------------------  
SDPI:0047D40B  
SDPI:0047D40B loc\_47D40B:                                       ; CODE XREF: SDPI:0047D3FF j  
SDPI:0047D40B                                                   ; SDPI:0047D401 j  
SDPI:0047D40B                 call    loc\_47D411  
SDPI:0047D410                 nop  
SDPI:0047D411  
SDPI:0047D411 loc\_47D411:                                       ; CODE XREF: SDPI:loc\_47D40B p  
SDPI:0047D411                 pop     esi  
SDPI:0047D412                 add     esi, 59h  
SDPI:0047D418                 mov     ecx, 3  
SDPI:0047D418 ; ----------------------------------------------------------------------------  
SDPI:0047D41D JUNK\_47D41D     db '|',7,'?,7,0,10h,'@',0,'鑤鱮',12h,'s',10h,0,10h,'@',0,'?,3,'?  
SDPI:0047D41D                 db 'r',1Bh,'s',19h,0,10h,'@',0,'鑖淧 3?豻扈',5,0,0,0,0,10h,'@',0  
SDPI:0047D41D                 db '鑈 Xf?  
SDPI:0047D453 ; ----------------------------------------------------------------------------  
SDPI:0047D453                 rep movsw  
SDPI:0047D456                 call    fnddbg\_47D4D7             ; 又是一大堆的宏来判断是否有调试器，  
SDPI:0047D456                                                   ; 让我想起了一首歌曲<<无尽的爱>>  
SDPI:0047D456                                                   ; 这壳叫无尽的检测\*\_\*  
SDPI:0047D45B                 call    loc\_47D547  
SDPI:0047D45B ; ----------------------------------------------------------------------------  
SDPI:0047D460 JUNK\_47D460     db '?,0,10h,'@',0,'皦?,4,'?,1,'愲',3,'悙'  
SDPI:0047D470  
SDPI:0047D470 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047D470  
SDPI:0047D470  
SDPI:0047D470 FNDDBG\_47D470   proc near  
SDPI:0047D470                 nop  
SDPI:0047D471                 nop  
SDPI:0047D472                 nop  
SDPI:0047D473                 nop  
SDPI:0047D474                 call    loc\_47D47A  
SDPI:0047D479                 nop  
SDPI:0047D47A  
SDPI:0047D47A loc\_47D47A:                                       ; CODE XREF: FNDDBG\_47D470+4 p  
SDPI:0047D47A                 pop     eax  
SDPI:0047D47B                 add     eax, 5Eh  
SDPI:0047D480                 mov     edx, eax  
SDPI:0047D482                 add     edx, 32h  
SDPI:0047D485                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047D485                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047D485                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047D485                                                   ; 用来解密  
SDPI:0047D48A                 call    loc\_47D490  
SDPI:0047D48F                 nop  
SDPI:0047D490  
SDPI:0047D490 loc\_47D490:                                       ; CODE XREF: FNDDBG\_47D470+1A p  
SDPI:0047D490                 pop     eax  
SDPI:0047D491                 add     eax, 1C0Dh  
SDPI:0047D496                 call    loc\_47D49C  
SDPI:0047D49B                 nop  
SDPI:0047D49C  
SDPI:0047D49C loc\_47D49C:                                       ; CODE XREF: FNDDBG\_47D470+26 p  
SDPI:0047D49C                 pop     ecx  
SDPI:0047D49D                 add     ecx, 1CAEh  
SDPI:0047D4A3                 push    0  
SDPI:0047D4A5                 push    ecx  
SDPI:0047D4A6                 push    eax  
SDPI:0047D4A7                 push    0  
SDPI:0047D4A9                 call    loc\_47D4AF  
SDPI:0047D4AE                 nop  
SDPI:0047D4AF  
SDPI:0047D4AF loc\_47D4AF:                                       ; CODE XREF: FNDDBG\_47D470+39 p  
SDPI:0047D4AF                 pop     eax  
SDPI:0047D4B0                 add     eax, 11h  
SDPI:0047D4B5                 push    eax  
SDPI:0047D4B6                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047D4B6                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047D4B6                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047D4B6 ; ----------------------------------------------------------------------------  
SDPI:0047D4BB                 db 4 dup(90h)  
SDPI:0047D4BF ; ----------------------------------------------------------------------------  
SDPI:0047D4BF                 push    7  
SDPI:0047D4C1                 call    loc\_47D4C7  
SDPI:0047D4C6                 nop  
SDPI:0047D4C7  
SDPI:0047D4C7 loc\_47D4C7:                                       ; CODE XREF: FNDDBG\_47D470+51 p  
SDPI:0047D4C7                 pop     eax  
SDPI:0047D4C8                 add     eax, 11h  
SDPI:0047D4CD                 push    eax  
SDPI:0047D4CE                 jmp     ExitProcess  
SDPI:0047D4CE ; ----------------------------------------------------------------------------  
SDPI:0047D4D3                 db 4 dup(90h)  
SDPI:0047D4D3 FNDDBG\_47D470   endp  
SDPI:0047D4D3  
SDPI:0047D4D7  
SDPI:0047D4D7 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047D4D7  
SDPI:0047D4D7 ; 又是一大堆的宏来判断是否有调试器，  
SDPI:0047D4D7 ; 让我想起了一首歌曲<<无尽的爱>>  
SDPI:0047D4D7 ; 这壳叫无尽的检测\*\_\*  
SDPI:0047D4D7  
SDPI:0047D4D7 fnddbg\_47D4D7   proc near                         ; CODE XREF: SDPI:0047D456 p  
SDPI:0047D4D7                 nop  
SDPI:0047D4D8                 nop  
SDPI:0047D4D9                 nop  
SDPI:0047D4DA                 nop  
SDPI:0047D4DB                 nop  
SDPI:0047D4DC                 call    loc\_47D4E2  
SDPI:0047D4E1                 nop  
SDPI:0047D4E2  
SDPI:0047D4E2 loc\_47D4E2:                                       ; CODE XREF: fnddbg\_47D4D7+5 p  
SDPI:0047D4E2                 pop     eax  
SDPI:0047D4E3                 add     eax, 5Eh  
SDPI:0047D4E8                 mov     edx, eax  
SDPI:0047D4EA                 add     edx, 32h  
SDPI:0047D4ED                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047D4ED                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047D4ED                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047D4ED                                                   ; 用来解密  
SDPI:0047D4F2                 call    loc\_47D4F8  
SDPI:0047D4F7                 nop  
SDPI:0047D4F8  
SDPI:0047D4F8 loc\_47D4F8:                                       ; CODE XREF: fnddbg\_47D4D7+1B p  
SDPI:0047D4F8                 pop     eax  
SDPI:0047D4F9                 add     eax, 1BA5h  
SDPI:0047D4FE                 call    loc\_47D504  
SDPI:0047D503                 nop  
SDPI:0047D504  
SDPI:0047D504 loc\_47D504:                                       ; CODE XREF: fnddbg\_47D4D7+27 p  
SDPI:0047D504                 pop     ecx  
SDPI:0047D505                 add     ecx, 1C46h  
SDPI:0047D50B                 push    0  
SDPI:0047D50D                 push    ecx  
SDPI:0047D50E                 push    eax  
SDPI:0047D50F                 push    0  
SDPI:0047D511                 call    loc\_47D517  
SDPI:0047D516                 nop  
SDPI:0047D517  
SDPI:0047D517 loc\_47D517:                                       ; CODE XREF: fnddbg\_47D4D7+3A p  
SDPI:0047D517                 pop     eax  
SDPI:0047D518                 add     eax, 11h  
SDPI:0047D51D                 push    eax  
SDPI:0047D51E                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047D51E                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047D51E                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047D51E ; ----------------------------------------------------------------------------  
SDPI:0047D523                 db 4 dup(90h)  
SDPI:0047D527 ; ----------------------------------------------------------------------------  
SDPI:0047D527                 push    7  
SDPI:0047D529                 call    loc\_47D52F  
SDPI:0047D52E                 nop  
SDPI:0047D52F  
SDPI:0047D52F loc\_47D52F:                                       ; CODE XREF: fnddbg\_47D4D7+52 p  
SDPI:0047D52F                 pop     eax  
SDPI:0047D530                 add     eax, 11h  
SDPI:0047D535                 push    eax  
SDPI:0047D536                 jmp     ExitProcess  
SDPI:0047D536 ; ----------------------------------------------------------------------------  
SDPI:0047D53B                 db 4 dup(90h), 0, 10h, 40h, 0, 0E0h, 89h, 9Ch, 6  
SDPI:0047D53B fnddbg\_47D4D7   endp  
SDPI:0047D53B  
SDPI:0047D547 ; ----------------------------------------------------------------------------  
SDPI:0047D547  
SDPI:0047D547 loc\_47D547:                                       ; CODE XREF: SDPI:0047D45B p  
SDPI:0047D547                 call    loc\_47D54D  
SDPI:0047D54C                 nop  
SDPI:0047D54D  
SDPI:0047D54D loc\_47D54D:                                       ; CODE XREF: SDPI:loc\_47D547 p  
SDPI:0047D54D                 pop     edi  
SDPI:0047D54E                 add     edi, 0FFFFFF07h  
SDPI:0047D554                 mov     \[edi\], ebx  
SDPI:0047D556                 mov     \[edi+4\], edx  
SDPI:0047D559                 pop     eax  
SDPI:0047D55A                 call    loc\_47D560  
SDPI:0047D55F                 nop  
SDPI:0047D560  
SDPI:0047D560 loc\_47D560:                                       ; CODE XREF: SDPI:0047D55A p  
SDPI:0047D560                 pop     eax  
SDPI:0047D561                 add     eax, 124h  
SDPI:0047D566                 push    eax  
SDPI:0047D567                 xor     eax, eax  
SDPI:0047D569                 push    dword ptr fs:\[eax\]  
SDPI:0047D56C                 mov     fs:\[eax\], esp  
SDPI:0047D56F                 mov     ebp, 300EF1D3h  
SDPI:0047D574                 add     ebp, 12345678h  
SDPI:0047D57A                 mov     ax, 17h  
SDPI:0047D57E                 sub     ax, 13h  
SDPI:0047D57E ; ----------------------------------------------------------------------------  
SDPI:0047D582 JUNK\_47D582     db '|',7,'?,7,0,10h,'@',0,'鑤鱮',12h,'s',10h,0,10h,'@',0,'?,3,'?  
SDPI:0047D582                 db 'r',1Bh,'s',19h,0,10h,'@',0,'鑖淧 3?豻扈',5,0,0,0,0,10h,'@',0  
SDPI:0047D582                 db '鑈 Xf 潗悙悙'  
SDPI:0047D5BD ; ----------------------------------------------------------------------------  
SDPI:0047D5BD                 int     3                         ; Trap to Debugger  
SDPI:0047D5BE                 nop  
SDPI:0047D5BF                 cmp     al, 4  
SDPI:0047D5C1                 jz      short DONE\_47D634  
SDPI:0047D5C3  
SDPI:0047D5C3 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047D5C3  
SDPI:0047D5C3  
SDPI:0047D5C3 FNDDBG\_47D5C3   proc near                         ; CODE XREF: SDPI:0047D64D j  
SDPI:0047D5C3                                                   ; SDPI:0047D665 j ...  
SDPI:0047D5C3                 nop  
SDPI:0047D5C4                 nop  
SDPI:0047D5C5                 nop  
SDPI:0047D5C6                 nop  
SDPI:0047D5C7                 nop  
SDPI:0047D5C8                 call    loc\_47D5CE  
SDPI:0047D5CD                 nop  
SDPI:0047D5CE  
SDPI:0047D5CE loc\_47D5CE:                                       ; CODE XREF: FNDDBG\_47D5C3+5 p  
SDPI:0047D5CE                 pop     eax  
SDPI:0047D5CF                 add     eax, 5Eh  
SDPI:0047D5D4                 mov     edx, eax  
SDPI:0047D5D6                 add     edx, 32h  
SDPI:0047D5D9                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047D5D9                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047D5D9                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047D5D9                                                   ; 用来解密  
SDPI:0047D5DE                 call    loc\_47D5E4  
SDPI:0047D5E3                 nop  
SDPI:0047D5E4  
SDPI:0047D5E4 loc\_47D5E4:                                       ; CODE XREF: FNDDBG\_47D5C3+1B p  
SDPI:0047D5E4                 pop     eax  
SDPI:0047D5E5                 add     eax, 1AB9h  
SDPI:0047D5EA                 call    loc\_47D5F0  
SDPI:0047D5EF                 nop  
SDPI:0047D5F0  
SDPI:0047D5F0 loc\_47D5F0:                                       ; CODE XREF: FNDDBG\_47D5C3+27 p  
SDPI:0047D5F0                 pop     ecx  
SDPI:0047D5F1                 add     ecx, 1B5Ah  
SDPI:0047D5F7                 push    0  
SDPI:0047D5F9                 push    ecx  
SDPI:0047D5FA                 push    eax  
SDPI:0047D5FB                 push    0  
SDPI:0047D5FD                 call    loc\_47D603  
SDPI:0047D602                 nop  
SDPI:0047D603  
SDPI:0047D603 loc\_47D603:                                       ; CODE XREF: FNDDBG\_47D5C3+3A p  
SDPI:0047D603                 pop     eax  
SDPI:0047D604                 add     eax, 11h  
SDPI:0047D609                 push    eax  
SDPI:0047D60A                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047D60A                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047D60A                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047D60A ; ----------------------------------------------------------------------------  
SDPI:0047D60F                 db 4 dup(90h)  
SDPI:0047D613 ; ----------------------------------------------------------------------------  
SDPI:0047D613                 push    7  
SDPI:0047D615                 call    loc\_47D61B  
SDPI:0047D61A                 nop  
SDPI:0047D61B  
SDPI:0047D61B loc\_47D61B:                                       ; CODE XREF: FNDDBG\_47D5C3+52 p  
SDPI:0047D61B                 pop     eax  
SDPI:0047D61C                 add     eax, 11h  
SDPI:0047D621                 push    eax  
SDPI:0047D622                 jmp     ExitProcess  
SDPI:0047D622 ; ----------------------------------------------------------------------------  
SDPI:0047D627                 db 4 dup(90h), 0E8h, 0, 10h, 40h, 0, 0B0h, 89h, 9Ch, 4  
SDPI:0047D627 FNDDBG\_47D5C3   endp  
SDPI:0047D627  
SDPI:0047D634 ; ----------------------------------------------------------------------------  
SDPI:0047D634  
SDPI:0047D634 DONE\_47D634:                                      ; CODE XREF: SDPI:0047D5C1 j  
SDPI:0047D634                 pop     large dword ptr fs:0  
SDPI:0047D63B                 add     esp, 4  
SDPI:0047D63E                 call    loc\_47D644  
SDPI:0047D643                 nop  
SDPI:0047D644  
SDPI:0047D644 loc\_47D644:                                       ; CODE XREF: SDPI:0047D63E p  
SDPI:0047D644                 pop     eax  
SDPI:0047D645                 add     eax, 0FFFFFE1Dh  
SDPI:0047D64A                 cmp     byte ptr \[eax\], 0E9h  
SDPI:0047D64D                 jnz     FNDDBG\_47D5C3  
SDPI:0047D653                 mov     byte ptr \[eax\], 0E8h  
SDPI:0047D656                 rdtsc  
SDPI:0047D658                 mov     ecx, eax  
SDPI:0047D65A                 mov     ebx, edx  
SDPI:0047D65C                 rdtsc  
SDPI:0047D65E                 sub     eax, ecx  
SDPI:0047D660                 sbb     edx, ebx  
SDPI:0047D662                 cmp     edx, 0  
SDPI:0047D665                 jnz     FNDDBG\_47D5C3  
SDPI:0047D66B                 cmp     eax, 30000000h  
SDPI:0047D670                 ja      FNDDBG\_47D5C3  
SDPI:0047D676                 jz      short PASS\_47D6C1  
SDPI:0047D678                 jnz     short PASS\_47D6C1  
SDPI:0047D678 ; ----------------------------------------------------------------------------  
SDPI:0047D67A                 dd 401000E8h  
SDPI:0047D67E                 dd 9C89B000h  
SDPI:0047D682                 db 4  
SDPI:0047D683 ; ----------------------------------------------------------------------------  
SDPI:0047D683                 mov     eax, \[esp+4\]  
SDPI:0047D687                 mov     ecx, \[esp+0Ch\]  
SDPI:0047D68B                 inc     dword ptr \[ecx+0B8h\]  
SDPI:0047D691                 mov     eax, \[eax\]  
SDPI:0047D693                 sub     eax, EXCEPTION\_BREAKPOINT  
SDPI:0047D698                 jnz     short locret\_47D6C0  
SDPI:0047D69A                 call    loc\_47D6A0  
SDPI:0047D69F                 nop  
SDPI:0047D6A0  
SDPI:0047D6A0 loc\_47D6A0:                                       ; CODE XREF: SDPI:0047D69A p  
SDPI:0047D6A0                 pop     eax  
SDPI:0047D6A1                 add     eax, 0FFFFFDC1h  
SDPI:0047D6A6                 cmp     byte ptr \[eax\], 0E8h  
SDPI:0047D6A9                 jnz     FNDDBG\_47D5C3  
SDPI:0047D6AF                 mov     byte ptr \[eax\], 0E9h  
SDPI:0047D6B2                 xor     eax, eax  
SDPI:0047D6B4                 mov     \[ecx+4\], eax              ; 清除 Drx  
SDPI:0047D6B7                 mov     \[ecx+8\], eax  
SDPI:0047D6BA                 mov     \[ecx+0Ch\], eax  
SDPI:0047D6BD                 mov     \[ecx+10h\], eax  
SDPI:0047D6C0  
SDPI:0047D6C0 locret\_47D6C0:                                    ; CODE XREF: SDPI:0047D698 j  
SDPI:0047D6C0                 retn  
SDPI:0047D6C1 ; ----------------------------------------------------------------------------  
SDPI:0047D6C1  
SDPI:0047D6C1 PASS\_47D6C1:                                      ; CODE XREF: SDPI:0047D676 j  
SDPI:0047D6C1                                                   ; SDPI:0047D678 j  
SDPI:0047D6C1                 pop     eax  
SDPI:0047D6C2                 call    loc\_47D8A4  
SDPI:0047D6C2 ; ----------------------------------------------------------------------------  
SDPI:0047D6C7                 dd 401000h  
SDPI:0047D6CB                 dd 15C56BEh  
SDPI:0047D6CF  
SDPI:0047D6CF ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047D6CF  
SDPI:0047D6CF  
SDPI:0047D6CF sub\_47D6CF      proc near                         ; CODE XREF: SDPI:0047D8CD p  
SDPI:0047D6CF                 pop     ebp  
SDPI:0047D6D0                 pop     eax  
SDPI:0047D6D1                 jmp     loc\_47D8D2  
SDPI:0047D6D1 sub\_47D6CF      endp  
SDPI:0047D6D1  
SDPI:0047D6D6 ; ----------------------------------------------------------------------------  
SDPI:0047D6D6                 mov     ecx, 0FFFFFF00h  
SDPI:0047D6DB                 push    fs  
SDPI:0047D6DB ; ----------------------------------------------------------------------------  
SDPI:0047D6DD JUNK\_47D6DD     db 't',0Ah  
SDPI:0047D6DD                 db 'u',8,0,10h,'@',0,'皦?,4  
SDPI:0047D6E9 ; ----------------------------------------------------------------------------  
SDPI:0047D6E9                 pushfw  
SDPI:0047D6EB                 push    eax  
SDPI:0047D6EC                 mov     eax, ebx  
SDPI:0047D6EE                 push    ebx  
SDPI:0047D6EF                 mov     eax, ecx  
SDPI:0047D6F1                 push    eax  
SDPI:0047D6F2                 add     eax, edx  
SDPI:0047D6F4                 mov     ebx, eax  
SDPI:0047D6F6                 push    ebx  
SDPI:0047D6F7                 pop     eax  
SDPI:0047D6F8                 push    edx  
SDPI:0047D6F9                 call    loc\_47D706  
SDPI:0047D6F9 ; ----------------------------------------------------------------------------  
SDPI:0047D6FE                 dd 401000h  
SDPI:0047D702                 dd 132BD7B0h  
SDPI:0047D706 ; ----------------------------------------------------------------------------  
SDPI:0047D706  
SDPI:0047D706 loc\_47D706:                                       ; CODE XREF: SDPI:0047D6F9 p  
SDPI:0047D706                 pop     eax  
SDPI:0047D707                 call    loc\_47D70D  
SDPI:0047D70C                 nop  
SDPI:0047D70D  
SDPI:0047D70D loc\_47D70D:                                       ; CODE XREF: SDPI:0047D707 p  
SDPI:0047D70D                 pop     eax  
SDPI:0047D70E                 add     eax, 11h  
SDPI:0047D713                 push    eax  
SDPI:0047D714                 jmp     GetTickCount  
SDPI:0047D714 ; ----------------------------------------------------------------------------  
SDPI:0047D719                 db 4 dup(90h)  
SDPI:0047D71D ; ----------------------------------------------------------------------------  
SDPI:0047D71D                 push    eax  
SDPI:0047D71E                 mov     eax, edx  
SDPI:0047D720                 push    eax  
SDPI:0047D721                 call    loc\_47D727  
SDPI:0047D726                 nop  
SDPI:0047D727  
SDPI:0047D727 loc\_47D727:                                       ; CODE XREF: SDPI:0047D721 p  
SDPI:0047D727                 pop     edx  
SDPI:0047D728                 add     edx, 52h  
SDPI:0047D72E                 push    edx  
SDPI:0047D72F                 add     edx, 40472Fh  
SDPI:0047D735                 push    edx  
SDPI:0047D736                 jo      short loc\_47D78B  
SDPI:0047D738                 jno     short loc\_47D78B  
SDPI:0047D73A  
SDPI:0047D73A loc\_47D73A:                                       ; CODE XREF: SDPI:0047D77E p  
SDPI:0047D73A                 pop     eax  
SDPI:0047D73B                 pop     ebx  
SDPI:0047D73C                 call    loc\_47D742  
SDPI:0047D741                 nop  
SDPI:0047D742  
SDPI:0047D742 loc\_47D742:                                       ; CODE XREF: SDPI:0047D73C p  
SDPI:0047D742                 pop     eax  
SDPI:0047D743                 add     eax, 11h  
SDPI:0047D748                 push    eax  
SDPI:0047D749                 jmp     GetTickCount  
SDPI:0047D749 ; ----------------------------------------------------------------------------  
SDPI:0047D74E                 db 4 dup(90h)  
SDPI:0047D752 ; ----------------------------------------------------------------------------  
SDPI:0047D752                 pop     ebx  
SDPI:0047D753                 add     ebx, 1F4h  
SDPI:0047D759                 sub     ebx, eax  
SDPI:0047D75B                 js      short FNDDBG\_47D79F  
SDPI:0047D75D                 call    loc\_47D763  
SDPI:0047D762                 nop  
SDPI:0047D763  
SDPI:0047D763 loc\_47D763:                                       ; CODE XREF: SDPI:0047D75D p  
SDPI:0047D763                 pop     ebx  
SDPI:0047D764                 add     ebx, 0A5h  
SDPI:0047D76A                 push    ebx  
SDPI:0047D76B                 call    loc\_47D795  
SDPI:0047D76B ; ----------------------------------------------------------------------------  
SDPI:0047D770                 dd 401000h  
SDPI:0047D774                 dd 58C88B0h  
SDPI:0047D778 ; ----------------------------------------------------------------------------  
SDPI:0047D778                 pop     eax  
SDPI:0047D779                 mov     edx, eax  
SDPI:0047D77B                 mov     eax, ebx  
SDPI:0047D77D                 push    eax  
SDPI:0047D77E                 call    loc\_47D73A  
SDPI:0047D77E ; ----------------------------------------------------------------------------  
SDPI:0047D783                 dd 401000h  
SDPI:0047D787                 dd 1833639h  
SDPI:0047D78B ; ----------------------------------------------------------------------------  
SDPI:0047D78B  
SDPI:0047D78B loc\_47D78B:                                       ; CODE XREF: SDPI:0047D736 j  
SDPI:0047D78B                                                   ; SDPI:0047D738 j  
SDPI:0047D78B                 pop     eax  
SDPI:0047D78C                 retn  
SDPI:0047D78C ; ----------------------------------------------------------------------------  
SDPI:0047D78D                 dd 401000h  
SDPI:0047D791                 dd 77C563Eh  
SDPI:0047D795 ; ----------------------------------------------------------------------------  
SDPI:0047D795  
SDPI:0047D795 loc\_47D795:                                       ; CODE XREF: SDPI:0047D76B p  
SDPI:0047D795                 pop     edx  
SDPI:0047D796                 retn  
SDPI:0047D796 ; ----------------------------------------------------------------------------  
SDPI:0047D797                 dd 401000h  
SDPI:0047D79B                 dd 1ED53EFh  
SDPI:0047D79F  
SDPI:0047D79F ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047D79F  
SDPI:0047D79F  
SDPI:0047D79F FNDDBG\_47D79F   proc near                         ; CODE XREF: SDPI:0047D75B j  
SDPI:0047D79F                 nop  
SDPI:0047D7A0                 nop  
SDPI:0047D7A1                 nop  
SDPI:0047D7A2                 nop  
SDPI:0047D7A3                 nop  
SDPI:0047D7A4                 call    loc\_47D7AA  
SDPI:0047D7A9                 nop  
SDPI:0047D7AA  
SDPI:0047D7AA loc\_47D7AA:                                       ; CODE XREF: FNDDBG\_47D79F+5 p  
SDPI:0047D7AA                 pop     eax  
SDPI:0047D7AB                 add     eax, 5Eh  
SDPI:0047D7B0                 mov     edx, eax  
SDPI:0047D7B2                 add     edx, 32h  
SDPI:0047D7B5                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047D7B5                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047D7B5                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047D7B5                                                   ; 用来解密  
SDPI:0047D7BA                 call    loc\_47D7C0  
SDPI:0047D7BF                 nop  
SDPI:0047D7C0  
SDPI:0047D7C0 loc\_47D7C0:                                       ; CODE XREF: FNDDBG\_47D79F+1B p  
SDPI:0047D7C0                 pop     eax  
SDPI:0047D7C1                 add     eax, 18DDh  
SDPI:0047D7C6                 call    loc\_47D7CC  
SDPI:0047D7CB                 nop  
SDPI:0047D7CC  
SDPI:0047D7CC loc\_47D7CC:                                       ; CODE XREF: FNDDBG\_47D79F+27 p  
SDPI:0047D7CC                 pop     ecx  
SDPI:0047D7CD                 add     ecx, 197Eh  
SDPI:0047D7D3                 push    0  
SDPI:0047D7D5                 push    ecx  
SDPI:0047D7D6                 push    eax  
SDPI:0047D7D7                 push    0  
SDPI:0047D7D9                 call    loc\_47D7DF  
SDPI:0047D7DE                 nop  
SDPI:0047D7DF  
SDPI:0047D7DF loc\_47D7DF:                                       ; CODE XREF: FNDDBG\_47D79F+3A p  
SDPI:0047D7DF                 pop     eax  
SDPI:0047D7E0                 add     eax, 11h  
SDPI:0047D7E5                 push    eax  
SDPI:0047D7E6                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047D7E6                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047D7E6                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047D7E6 ; ----------------------------------------------------------------------------  
SDPI:0047D7EB                 db 4 dup(90h)  
SDPI:0047D7EF ; ----------------------------------------------------------------------------  
SDPI:0047D7EF                 push    7  
SDPI:0047D7F1                 call    loc\_47D7F7  
SDPI:0047D7F6                 nop  
SDPI:0047D7F7  
SDPI:0047D7F7 loc\_47D7F7:                                       ; CODE XREF: FNDDBG\_47D79F+52 p  
SDPI:0047D7F7                 pop     eax  
SDPI:0047D7F8                 add     eax, 11h  
SDPI:0047D7FD                 push    eax  
SDPI:0047D7FE                 jmp     ExitProcess  
SDPI:0047D7FE ; ----------------------------------------------------------------------------  
SDPI:0047D803                 db 4 dup(90h)  
SDPI:0047D803 FNDDBG\_47D79F   endp  
SDPI:0047D803  
SDPI:0047D807 ; ----------------------------------------------------------------------------  
SDPI:0047D807                 pop     edx  
SDPI:0047D808                 mov     eax, ecx  
SDPI:0047D80A                 add     eax, edx  
SDPI:0047D80C                 inc     ecx  
SDPI:0047D80D                 push    eax  
SDPI:0047D80E                 inc     ecx  
SDPI:0047D80F                 pop     ebx  
SDPI:0047D810                 pop     ecx  
SDPI:0047D811                 push    eax  
SDPI:0047D812                 sub     eax, 8  
SDPI:0047D815                 pop     ebx  
SDPI:0047D816                 pop     ebx  
SDPI:0047D817                 inc     eax  
SDPI:0047D818                 add     eax, ebx  
SDPI:0047D81A                 pop     eax  
SDPI:0047D81B                 pushfw  
SDPI:0047D81D                 popfw  
SDPI:0047D81F                 popfw  
SDPI:0047D821                 pop     es  
SDPI:0047D822                 mov     eax, 12345678h  
SDPI:0047D827                 push    eax  
SDPI:0047D828                 call    loc\_47D82E  
SDPI:0047D82D                 nop  
SDPI:0047D82E  
SDPI:0047D82E loc\_47D82E:                                       ; CODE XREF: SDPI:0047D828 p  
SDPI:0047D82E                 pop     eax  
SDPI:0047D82F                 add     eax, 12Ch  
SDPI:0047D834                 push    eax  
SDPI:0047D835                 pop     ebx  
SDPI:0047D836                 add     eax, 12h  
SDPI:0047D839                 pop     edx  
SDPI:0047D83A                 add     eax, edx  
SDPI:0047D83C                 mov     edx, eax  
SDPI:0047D83E                 push    ebx  
SDPI:0047D83F                 mov     ebx, es:\[ecx+100h\]  
SDPI:0047D846                 push    ebx  
SDPI:0047D847                 mov     eax, esp  
SDPI:0047D849                 mov     ebx, eax  
SDPI:0047D84B                 push    ebx  
SDPI:0047D84C                 pop     edx  
SDPI:0047D84D                 mov     es:\[ecx+100h\], eax  
SDPI:0047D854                 xor     eax, eax  
SDPI:0047D854 ; ----------------------------------------------------------------------------  
SDPI:0047D856 JUNK\_47D856     db '~',7,'',5,0,10h,'@',0,'鑖淨 3 蒰?,5,0,10h,'@',0,'鑉悙悙悙悙?  
SDPI:0047D856                 db '悙悙悙悙悙悙悙 f 漰',6,'q',4,0,10h,'@',0  
SDPI:0047D88E ; ----------------------------------------------------------------------------  
SDPI:0047D88E                 int     3                         ; Trap to Debugger  
SDPI:0047D88F                 nop  
SDPI:0047D890                 xor     eax, eax  
SDPI:0047D892                 mov     dword ptr \[eax\], 404992h  
SDPI:0047D898                 jp      short loc\_47D8A4  
SDPI:0047D89A                 jnp     short loc\_47D8A4  
SDPI:0047D89A ; ----------------------------------------------------------------------------  
SDPI:0047D89C                 dd 401000h  
SDPI:0047D8A0                 dd 403D7Bh  
SDPI:0047D8A4 ; ----------------------------------------------------------------------------  
SDPI:0047D8A4  
SDPI:0047D8A4 loc\_47D8A4:                                       ; CODE XREF: SDPI:0047D6C2 p  
SDPI:0047D8A4                                                   ; SDPI:0047D898 j ...  
SDPI:0047D8A4                 call    loc\_47D8AA  
SDPI:0047D8A9                 nop  
SDPI:0047D8AA  
SDPI:0047D8AA loc\_47D8AA:                                       ; CODE XREF: SDPI:loc\_47D8A4 p  
SDPI:0047D8AA                 pop     eax  
SDPI:0047D8AB                 add     eax, 11h  
SDPI:0047D8B0                 push    eax  
SDPI:0047D8B1                 jmp     GetTickCount  
SDPI:0047D8B1 ; ----------------------------------------------------------------------------  
SDPI:0047D8B6                 db 4 dup(90h)  
SDPI:0047D8BA ; ----------------------------------------------------------------------------  
SDPI:0047D8BA                 call    loc\_47D8C0  
SDPI:0047D8BF                 nop  
SDPI:0047D8C0  
SDPI:0047D8C0 loc\_47D8C0:                                       ; CODE XREF: SDPI:0047D8BA p  
SDPI:0047D8C0                 pop     edx  
SDPI:0047D8C1                 add     edx, 0FFFFFB04h  
SDPI:0047D8C7                 mov     \[edx\], eax  
SDPI:0047D8C9                 pop     ebp  
SDPI:0047D8CA                 add     eax, edx  
SDPI:0047D8CC                 push    eax  
SDPI:0047D8CD                 call    sub\_47D6CF  
SDPI:0047D8D2  
SDPI:0047D8D2 loc\_47D8D2:                                       ; CODE XREF: sub\_47D6CF+2 j  
SDPI:0047D8D2                 call    loc\_47D8D8  
SDPI:0047D8D7                 nop  
SDPI:0047D8D8  
SDPI:0047D8D8 loc\_47D8D8:                                       ; CODE XREF: SDPI:loc\_47D8D2 p  
SDPI:0047D8D8                 pop     edx  
SDPI:0047D8D9                 add     edx, 0FFFFFDFFh  
SDPI:0047D8DF                 add     edx, eax  
SDPI:0047D8E1                 push    edx  
SDPI:0047D8E2                 pop     ecx  
SDPI:0047D8E3                 sub     ecx, eax  
SDPI:0047D8E5                 push    ecx  
SDPI:0047D8E6                 retn    4  
SDPI:0047D8E9  
SDPI:0047D8E9 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047D8E9  
SDPI:0047D8E9  
SDPI:0047D8E9 FNDDBG\_47D8E9   proc near                         ; CODE XREF: SDPI:0047D98C j  
SDPI:0047D8E9                                                   ; SDPI:0047D994 j ...  
SDPI:0047D8E9                 nop  
SDPI:0047D8EA                 nop  
SDPI:0047D8EB                 nop  
SDPI:0047D8EC                 nop  
SDPI:0047D8ED                 nop  
SDPI:0047D8EE                 call    loc\_47D8F4  
SDPI:0047D8F3                 nop  
SDPI:0047D8F4  
SDPI:0047D8F4 loc\_47D8F4:                                       ; CODE XREF: FNDDBG\_47D8E9+5 p  
SDPI:0047D8F4                 pop     eax  
SDPI:0047D8F5                 add     eax, 5Eh  
SDPI:0047D8FA                 mov     edx, eax  
SDPI:0047D8FC                 add     edx, 32h  
SDPI:0047D8FF                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047D8FF                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047D8FF                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047D8FF                                                   ; 用来解密  
SDPI:0047D904                 call    loc\_47D90A  
SDPI:0047D909                 nop  
SDPI:0047D90A  
SDPI:0047D90A loc\_47D90A:                                       ; CODE XREF: FNDDBG\_47D8E9+1B p  
SDPI:0047D90A                 pop     eax  
SDPI:0047D90B                 add     eax, 1793h  
SDPI:0047D910                 call    loc\_47D916  
SDPI:0047D915                 nop  
SDPI:0047D916  
SDPI:0047D916 loc\_47D916:                                       ; CODE XREF: FNDDBG\_47D8E9+27 p  
SDPI:0047D916                 pop     ecx  
SDPI:0047D917                 add     ecx, 1834h  
SDPI:0047D91D                 push    0  
SDPI:0047D91F                 push    ecx  
SDPI:0047D920                 push    eax  
SDPI:0047D921                 push    0  
SDPI:0047D923                 call    loc\_47D929  
SDPI:0047D928                 nop  
SDPI:0047D929  
SDPI:0047D929 loc\_47D929:                                       ; CODE XREF: FNDDBG\_47D8E9+3A p  
SDPI:0047D929                 pop     eax  
SDPI:0047D92A                 add     eax, 11h  
SDPI:0047D92F                 push    eax  
SDPI:0047D930                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047D930                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047D930                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047D930 ; ----------------------------------------------------------------------------  
SDPI:0047D935                 db 4 dup(90h)  
SDPI:0047D939 ; ----------------------------------------------------------------------------  
SDPI:0047D939                 push    7  
SDPI:0047D93B                 call    loc\_47D941  
SDPI:0047D940                 nop  
SDPI:0047D941  
SDPI:0047D941 loc\_47D941:                                       ; CODE XREF: FNDDBG\_47D8E9+52 p  
SDPI:0047D941                 pop     eax  
SDPI:0047D942                 add     eax, 11h  
SDPI:0047D947                 push    eax  
SDPI:0047D948                 jmp     ExitProcess  
SDPI:0047D948 ; ----------------------------------------------------------------------------  
SDPI:0047D94D                 db 4 dup(90h), 0, 10h, 40h, 0, 0BEh, 56h, 5Ch, 1  
SDPI:0047D94D FNDDBG\_47D8E9   endp  
SDPI:0047D94D  
SDPI:0047D959 ; ----------------------------------------------------------------------------  
SDPI:0047D959                 mov     esp, \[esp+8\]  
SDPI:0047D95D                 pop     large dword ptr fs:0  
SDPI:0047D964                 call    loc\_47D96A  
SDPI:0047D969                 nop  
SDPI:0047D96A  
SDPI:0047D96A loc\_47D96A:                                       ; CODE XREF: SDPI:0047D964 p  
SDPI:0047D96A                 pop     eax  
SDPI:0047D96B                 add     eax, 11h  
SDPI:0047D970                 push    eax  
SDPI:0047D971                 jmp     GetTickCount  
SDPI:0047D971 ; ----------------------------------------------------------------------------  
SDPI:0047D976                 db 4 dup(90h)  
SDPI:0047D97A ; ----------------------------------------------------------------------------  
SDPI:0047D97A                 call    loc\_47D980  
SDPI:0047D97F                 nop  
SDPI:0047D980  
SDPI:0047D980 loc\_47D980:                                       ; CODE XREF: SDPI:0047D97A p  
SDPI:0047D980                 pop     edx  
SDPI:0047D981                 add     edx, 0FFFFFA44h  
SDPI:0047D987                 mov     ecx, \[edx\]  
SDPI:0047D989                 cmp     ecx, 0  
SDPI:0047D98C                 jz      FNDDBG\_47D8E9  
SDPI:0047D992                 sub     eax, ecx  
SDPI:0047D994                 js      FNDDBG\_47D8E9  
SDPI:0047D99A                 sub     eax, 7D0h  
SDPI:0047D99F                 jns     FNDDBG\_47D8E9  
SDPI:0047D9A5                 mov     eax, 0E801276h  
SDPI:0047D9AA                 mov     \[edx\], eax  
SDPI:0047D9AC                 call    loc\_47D9B2  
SDPI:0047D9B1                 nop  
SDPI:0047D9B2  
SDPI:0047D9B2 loc\_47D9B2:                                       ; CODE XREF: SDPI:0047D9AC p  
SDPI:0047D9B2                 pop     edx  
SDPI:0047D9B3                 add     edx, 37Bh  
SDPI:0047D9B9                 call    loc\_47D9BF                ; 渡过漫长的 int3 后来再来看这里  
SDPI:0047D9BE                 nop  
SDPI:0047D9BF  
SDPI:0047D9BF loc\_47D9BF:                                       ; CODE XREF: SDPI:0047D9B9 p  
SDPI:0047D9BF                 pop     eax  
SDPI:0047D9C0                 add     eax, 0FFFFC792h  
SDPI:0047D9C5                 mov     ecx, 10h  
SDPI:0047D9CA                 call    De\_Code                   ; invoke De\_Code,ecx(counter),eax(key),edx(end)  
SDPI:0047D9CA                                                   ; 用于解密代码，解密起始地址就是 call 进来  
SDPI:0047D9CA                                                   ; 的下一行代码地址  
SDPI:0047D9CF                 call    loc\_47D9D5  
SDPI:0047D9D4                 nop  
SDPI:0047D9D5  
SDPI:0047D9D5 loc\_47D9D5:                                       ; CODE XREF: SDPI:0047D9CF p  
SDPI:0047D9D5                 pop     eax  
SDPI:0047D9D6                 add     eax, 11h  
SDPI:0047D9DB                 push    eax  
SDPI:0047D9DC                 jmp     CHKDBGFLG\_48178A          ; 再次检测是否有调试器  
SDPI:0047D9E1 ; ----------------------------------------------------------------------------  
SDPI:0047D9E1                 nop  
SDPI:0047D9E2                 nop  
SDPI:0047D9E3                 nop  
SDPI:0047D9E4                 nop  
SDPI:0047D9E5                 xor     eax, 87EAF247h  
SDPI:0047D9EA                 cmp     eax, 0F234543Eh  
SDPI:0047D9EA ; ----------------------------------------------------------------------------  
SDPI:0047D9EF aI@stRS@sRS@sfP3TS\_0 db '|',7,'?,7,0,10h,'@',0,'鑤鱮',12h,'s',10h,0,10h,'@',0,'?,3,'?  
SDPI:0047D9EF                 db 'r',1Bh,'s',19h,0,10h,'@',0,'鑖淧 3?豻扈',5,0,0,0,0,10h,'@',0  
SDPI:0047D9EF                 db '鑈 Xf'  
SDPI:0047DA24                 db  9Dh ; ?  
SDPI:0047DA25 ; ----------------------------------------------------------------------------  
SDPI:0047DA25                 jz      short Pass\_47DA8F  
SDPI:0047DA27  
SDPI:0047DA27 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047DA27  
SDPI:0047DA27  
SDPI:0047DA27 FNDDBG\_47DA27   proc near  
SDPI:0047DA27                 nop  
SDPI:0047DA28                 nop  
SDPI:0047DA29                 nop  
SDPI:0047DA2A                 nop  
SDPI:0047DA2B                 nop  
SDPI:0047DA2C                 call    loc\_47DA32  
SDPI:0047DA31                 nop  
SDPI:0047DA32  
SDPI:0047DA32 loc\_47DA32:                                       ; CODE XREF: FNDDBG\_47DA27+5 p  
SDPI:0047DA32                 pop     eax  
SDPI:0047DA33                 add     eax, 5Eh  
SDPI:0047DA38                 mov     edx, eax  
SDPI:0047DA3A                 add     edx, 32h  
SDPI:0047DA3D                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047DA3D                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047DA3D                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047DA3D                                                   ; 用来解密  
SDPI:0047DA42                 call    loc\_47DA48  
SDPI:0047DA47                 nop  
SDPI:0047DA48  
SDPI:0047DA48 loc\_47DA48:                                       ; CODE XREF: FNDDBG\_47DA27+1B p  
SDPI:0047DA48                 pop     eax  
SDPI:0047DA49                 add     eax, 1655h  
SDPI:0047DA4E                 call    loc\_47DA54  
SDPI:0047DA53                 nop  
SDPI:0047DA54  
SDPI:0047DA54 loc\_47DA54:                                       ; CODE XREF: FNDDBG\_47DA27+27 p  
SDPI:0047DA54                 pop     ecx  
SDPI:0047DA55                 add     ecx, 16F6h  
SDPI:0047DA5B                 push    0  
SDPI:0047DA5D                 push    ecx  
SDPI:0047DA5E                 push    eax  
SDPI:0047DA5F                 push    0  
SDPI:0047DA61                 call    loc\_47DA67  
SDPI:0047DA66                 nop  
SDPI:0047DA67  
SDPI:0047DA67 loc\_47DA67:                                       ; CODE XREF: FNDDBG\_47DA27+3A p  
SDPI:0047DA67                 pop     eax  
SDPI:0047DA68                 add     eax, 11h  
SDPI:0047DA6D                 push    eax  
SDPI:0047DA6E                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047DA6E                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047DA6E                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047DA6E ; ----------------------------------------------------------------------------  
SDPI:0047DA73                 db 4 dup(90h)  
SDPI:0047DA77 ; ----------------------------------------------------------------------------  
SDPI:0047DA77                 push    7  
SDPI:0047DA79                 call    loc\_47DA7F  
SDPI:0047DA7E                 nop  
SDPI:0047DA7F  
SDPI:0047DA7F loc\_47DA7F:                                       ; CODE XREF: FNDDBG\_47DA27+52 p  
SDPI:0047DA7F                 pop     eax  
SDPI:0047DA80                 add     eax, 11h  
SDPI:0047DA85                 push    eax  
SDPI:0047DA86                 jmp     ExitProcess  
SDPI:0047DA86 ; ----------------------------------------------------------------------------  
SDPI:0047DA8B                 db 4 dup(90h)  
SDPI:0047DA8B FNDDBG\_47DA27   endp  
SDPI:0047DA8B  
SDPI:0047DA8F ; ----------------------------------------------------------------------------  
SDPI:0047DA8F  
SDPI:0047DA8F Pass\_47DA8F:                                      ; CODE XREF: SDPI:0047DA25 j  
SDPI:0047DA8F                 call    loc\_47DA95  
SDPI:0047DA94                 nop  
SDPI:0047DA95  
SDPI:0047DA95 loc\_47DA95:                                       ; CODE XREF: SDPI:Pass\_47DA8F p  
SDPI:0047DA95                 pop     ecx  
SDPI:0047DA96                 add     ecx, 0FFFFC680h  
SDPI:0047DA9C                 push    ecx  
SDPI:0047DA9D                 push    0  
SDPI:0047DA9F                 call    loc\_47DAA5  
SDPI:0047DAA4                 nop  
SDPI:0047DAA5  
SDPI:0047DAA5 loc\_47DAA5:                                       ; CODE XREF: SDPI:0047DA9F p  
SDPI:0047DAA5                 pop     eax  
SDPI:0047DAA6                 add     eax, 11h  
SDPI:0047DAAB                 push    eax  
SDPI:0047DAAC                 jmp     api\_GetModule  
SDPI:0047DAAC ; ----------------------------------------------------------------------------  
SDPI:0047DAB1                 db 4 dup(90h)  
SDPI:0047DAB5 ; ----------------------------------------------------------------------------  
SDPI:0047DAB5                 pop     ecx                       ; 获取程序的 HMODULE  
SDPI:0047DAB6                 add     eax, \[ecx\]  
SDPI:0047DAB8                 push    eax  
SDPI:0047DAB9                 call    loc\_47DABF  
SDPI:0047DABE                 nop  
SDPI:0047DABF  
SDPI:0047DABF loc\_47DABF:                                       ; CODE XREF: SDPI:0047DAB9 p  
SDPI:0047DABF                 pop     eax  
SDPI:0047DAC0                 add     eax, 11h  
SDPI:0047DAC5                 push    eax  
SDPI:0047DAC6                 jmp     UnKnow\_48171A  
SDPI:0047DAC6 ; ----------------------------------------------------------------------------  
SDPI:0047DACB                 db 4 dup(90h)  
SDPI:0047DACF ; ----------------------------------------------------------------------------  
SDPI:0047DACF                 mov     ebx, eax  
SDPI:0047DAD1                 pop     eax  
SDPI:0047DAD2                 call    loc\_47DAD8  
SDPI:0047DAD7                 nop  
SDPI:0047DAD8  
SDPI:0047DAD8 loc\_47DAD8:                                       ; CODE XREF: SDPI:0047DAD2 p  
SDPI:0047DAD8                 pop     ecx  
SDPI:0047DAD9                 add     ecx, 0FFFFC69Dh  
SDPI:0047DADF                 mov     \[ecx\], ebx  
SDPI:0047DAE1                 call    loc\_47DAE7  
SDPI:0047DAE6                 nop  
SDPI:0047DAE7  
SDPI:0047DAE7 loc\_47DAE7:                                       ; CODE XREF: SDPI:0047DAE1 p  
SDPI:0047DAE7                 pop     ecx  
SDPI:0047DAE8                 add     ecx, 0FFFFC632h  
SDPI:0047DAEE                 add     \[ecx\], ebx  
SDPI:0047DAF0                 call    loc\_47DAF6  
SDPI:0047DAF5                 nop  
SDPI:0047DAF6  
SDPI:0047DAF6 loc\_47DAF6:                                       ; CODE XREF: SDPI:0047DAF0 p  
SDPI:0047DAF6                 pop     ecx  
SDPI:0047DAF7                 add     ecx, 0FFFFC627h  
SDPI:0047DAFD                 add     \[ecx\], ebx  
SDPI:0047DAFF                 call    loc\_47DB05  
SDPI:0047DB04                 nop  
SDPI:0047DB05  
SDPI:0047DB05 loc\_47DB05:                                       ; CODE XREF: SDPI:0047DAFF p  
SDPI:0047DB05                 pop     ecx  
SDPI:0047DB06                 add     ecx, 0FFFFC628h  
SDPI:0047DB0C                 add     \[ecx\], ebx  
SDPI:0047DB0E                 cmp     ebx, 0                    ; 这几个地方就不知道有什么作用了  
SDPI:0047DB0E                                                   ; 我这里 ebx==0  
SDPI:0047DB11                 jz      short loc\_47DB2A  
SDPI:0047DB13                 push    eax  
SDPI:0047DB14                 mov     ax, bx  
SDPI:0047DB17                 shr     ebx, 10h  
SDPI:0047DB1A                 mov     dx, bx  
SDPI:0047DB1D                 mov     bx, 400h  
SDPI:0047DB21                 div     bx  
SDPI:0047DB24                 xor     ecx, ecx  
SDPI:0047DB26                 mov     cx, ax  
SDPI:0047DB29                 pop     eax  
SDPI:0047DB2A  
SDPI:0047DB2A loc\_47DB2A:                                       ; CODE XREF: SDPI:0047DB11 j  
SDPI:0047DB2A                 mov     ecx, 0  
SDPI:0047DB2F                 call    loc\_47DB35  
SDPI:0047DB34                 nop  
SDPI:0047DB35  
SDPI:0047DB35 loc\_47DB35:                                       ; CODE XREF: SDPI:0047DB2F p  
SDPI:0047DB35                 pop     ebx  
SDPI:0047DB36                 add     ebx, 0Ch  
SDPI:0047DB3C                 push    ebx                       ; EBX 返回地址:0047DB40  
SDPI:0047DB3D                 push    ecx  
SDPI:0047DB3E                 jmp     eax                       ; 跳去 004740C0 解压程序代码  
SDPI:0047DB40 ; ----------------------------------------------------------------------------  
SDPI:0047DB40                 sub     esp, 2000h  
SDPI:0047DB46                 call    CHKTARGET\_482BEF          ; 这里进去一大堆的时间检测了  
SDPI:0047DB46                                                   ; 文件检测,调试器检测等  
SDPI:0047DB4B                 call    loc\_47DB51  
SDPI:0047DB50                 nop  
SDPI:0047DB51  
SDPI:0047DB51 loc\_47DB51:                                       ; CODE XREF: SDPI:0047DB4B p  
SDPI:0047DB51                 pop     eax  
SDPI:0047DB52                 add     eax, 11h  
SDPI:0047DB57                 push    eax  
SDPI:0047DB58                 jmp     Get\_Version  
SDPI:0047DB58 ; ----------------------------------------------------------------------------  
SDPI:0047DB5D                 db 4 dup(90h)  
SDPI:0047DB61 ; ----------------------------------------------------------------------------  
SDPI:0047DB61                 sub     ebx, 60000000h  
SDPI:0047DB61 ; ----------------------------------------------------------------------------  
SDPI:0047DB67 JUNK\_47db67     db '|',7,'?,7,0,10h,'@',0,'鑤鱮',12h,'s',10h,0,10h,'@',0,'?,3,'?  
SDPI:0047DB67                 db 'r',1Bh,'s',19h,0,10h,'@',0,'鑖淧 3?豻扈',5,0,0,0,0,10h,'@',0  
SDPI:0047DB67                 db '鑈 Xf?  
SDPI:0047DB9D ; ----------------------------------------------------------------------------  
SDPI:0047DB9D                 sub     eax, 20000000h  
SDPI:0047DB9D ; ----------------------------------------------------------------------------  
SDPI:0047DBA2 junk\_47dba2     db '~',7,'',5,0,10h,'@',0,'鑖淨 3 蒰?,5,0,10h,'@',0,'鑉悙悙悙悙?  
SDPI:0047DBA2                 db '悙悙悙悙悙悙悙 f?  
SDPI:0047DBD2 ; ----------------------------------------------------------------------------  
SDPI:0047DBD2                 jnb     NotIsWinNT\_47DC9C         ; 如果是 Winnt 系统则还有  
SDPI:0047DBD2                                                   ; 一个 int1 来检测调试器  
SDPI:0047DBD2 ; ----------------------------------------------------------------------------  
SDPI:0047DBD8                 dw 172h  
SDPI:0047DBDA                 db 0FFh  
SDPI:0047DBDB ; ----------------------------------------------------------------------------  
SDPI:0047DBDB                 call    loc\_47DBE1  
SDPI:0047DBE0                 nop  
SDPI:0047DBE1  
SDPI:0047DBE1 loc\_47DBE1:                                       ; CODE XREF: SDPI:0047DBDB p  
SDPI:0047DBE1                 pop     eax  
SDPI:0047DBE2                 add     eax, 8Eh  
SDPI:0047DBE7                 push    eax  
SDPI:0047DBE8                 xor     eax, eax  
SDPI:0047DBEA                 push    dword ptr fs:\[eax\]  
SDPI:0047DBED                 mov     fs:\[eax\], esp  
SDPI:0047DBF0                 int     1                         ; - internal hardware - SINGLE-STEP  
SDPI:0047DBF0                                                   ; generated at end of each machine instruction if TF bit in FLAGS is set  
SDPI:0047DBF2                 retn                              ; SEH  句柄 47db6e  
SDPI:0047DBF3 ; ----------------------------------------------------------------------------  
SDPI:0047DBF3                 pop     large dword ptr fs:0  
SDPI:0047DBFA                 add     esp, 4  
SDPI:0047DBFD                 jmp     NotIsWinNT\_47DC9C  
SDPI:0047DC02  
SDPI:0047DC02 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047DC02  
SDPI:0047DC02  
SDPI:0047DC02 FNDDBG\_47DC02   proc near                         ; CODE XREF: SDPI:0047DC7D j  
SDPI:0047DC02                 nop  
SDPI:0047DC03                 nop  
SDPI:0047DC04                 nop  
SDPI:0047DC05                 nop  
SDPI:0047DC06                 nop  
SDPI:0047DC07                 call    loc\_47DC0D  
SDPI:0047DC0C                 nop  
SDPI:0047DC0D  
SDPI:0047DC0D loc\_47DC0D:                                       ; CODE XREF: FNDDBG\_47DC02+5 p  
SDPI:0047DC0D                 pop     eax  
SDPI:0047DC0E                 add     eax, 5Eh  
SDPI:0047DC13                 mov     edx, eax  
SDPI:0047DC15                 add     edx, 32h  
SDPI:0047DC18                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047DC18                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047DC18                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047DC18                                                   ; 用来解密  
SDPI:0047DC1D                 call    loc\_47DC23  
SDPI:0047DC22                 nop  
SDPI:0047DC23  
SDPI:0047DC23 loc\_47DC23:                                       ; CODE XREF: FNDDBG\_47DC02+1B p  
SDPI:0047DC23                 pop     eax  
SDPI:0047DC24                 add     eax, 147Ah  
SDPI:0047DC29                 call    loc\_47DC2F  
SDPI:0047DC2E                 nop  
SDPI:0047DC2F  
SDPI:0047DC2F loc\_47DC2F:                                       ; CODE XREF: FNDDBG\_47DC02+27 p  
SDPI:0047DC2F                 pop     ecx  
SDPI:0047DC30                 add     ecx, 151Bh  
SDPI:0047DC36                 push    0  
SDPI:0047DC38                 push    ecx  
SDPI:0047DC39                 push    eax  
SDPI:0047DC3A                 push    0  
SDPI:0047DC3C                 call    loc\_47DC42  
SDPI:0047DC41                 nop  
SDPI:0047DC42  
SDPI:0047DC42 loc\_47DC42:                                       ; CODE XREF: FNDDBG\_47DC02+3A p  
SDPI:0047DC42                 pop     eax  
SDPI:0047DC43                 add     eax, 11h  
SDPI:0047DC48                 push    eax  
SDPI:0047DC49                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047DC49                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047DC49                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047DC49 ; ----------------------------------------------------------------------------  
SDPI:0047DC4E                 db 4 dup(90h)  
SDPI:0047DC52 ; ----------------------------------------------------------------------------  
SDPI:0047DC52                 push    7  
SDPI:0047DC54                 call    loc\_47DC5A  
SDPI:0047DC59                 nop  
SDPI:0047DC5A  
SDPI:0047DC5A loc\_47DC5A:                                       ; CODE XREF: FNDDBG\_47DC02+52 p  
SDPI:0047DC5A                 pop     eax  
SDPI:0047DC5B                 add     eax, 11h  
SDPI:0047DC60                 push    eax  
SDPI:0047DC61                 jmp     ExitProcess  
SDPI:0047DC61 ; ----------------------------------------------------------------------------  
SDPI:0047DC66                 db 4 dup(90h), 0, 10h, 40h, 0  
SDPI:0047DC66 FNDDBG\_47DC02   endp  
SDPI:0047DC66  
SDPI:0047DC6E ; ----------------------------------------------------------------------------  
SDPI:0047DC6E                 mov     eax, \[esp+4\]              ; INT 1 SEH 处理处  
SDPI:0047DC72                 mov     ecx, \[esp+0Ch\]  
SDPI:0047DC76                 mov     eax, \[eax\]  
SDPI:0047DC78                 cmp     eax, EXCEPTION\_SINGLE\_STEP ; 判断是否为单步异常  
SDPI:0047DC7D                 jz      short FNDDBG\_47DC02  
SDPI:0047DC7F                 cmp     eax, EXCEPTION\_ACCESS\_VIOLATION  
SDPI:0047DC84                 jnz     short locret\_47DC9B  
SDPI:0047DC86                 sub     dword ptr \[ecx+0B8h\], 0FFFFFFFDh ; 异常位置+3  
SDPI:0047DC8D                 xor     eax, eax  
SDPI:0047DC8F                 mov     \[ecx+4\], eax              ; 清除 Drx  
SDPI:0047DC92                 mov     \[ecx+8\], eax  
SDPI:0047DC95                 mov     \[ecx+0Ch\], eax  
SDPI:0047DC98                 mov     \[ecx+10h\], eax  
SDPI:0047DC9B  
SDPI:0047DC9B locret\_47DC9B:                                    ; CODE XREF: SDPI:0047DC84 j  
SDPI:0047DC9B                 retn  
SDPI:0047DC9C ; ----------------------------------------------------------------------------  
SDPI:0047DC9C  
SDPI:0047DC9C NotIsWinNT\_47DC9C:                                ; CODE XREF: SDPI:0047DBD2 j  
SDPI:0047DC9C                                                   ; SDPI:0047DBFD j  
SDPI:0047DC9C                 mov     eax, 1  
SDPI:0047DCA1                 push    eax  
SDPI:0047DCA2                 call    loc\_47DCA8  
SDPI:0047DCA7                 nop  
SDPI:0047DCA8  
SDPI:0047DCA8 loc\_47DCA8:                                       ; CODE XREF: SDPI:0047DCA2 p  
SDPI:0047DCA8                 pop     eax  
SDPI:0047DCA9                 add     eax, 11h  
SDPI:0047DCAE                 push    eax  
SDPI:0047DCAF                 jmp     Disposal\_IMP              ; 用于解压密出壳的输入表信息  
SDPI:0047DCAF                                                   ; 如果操作成功 EAX 返回 1  
SDPI:0047DCAF ; ----------------------------------------------------------------------------  
SDPI:0047DCB4                 db 4 dup(90h)  
SDPI:0047DCB8 ; ----------------------------------------------------------------------------  
SDPI:0047DCB8                 add     esp, 2000h  
SDPI:0047DCBE                 call    loc\_47DCC4  
SDPI:0047DCC3                 nop  
SDPI:0047DCC4  
SDPI:0047DCC4 loc\_47DCC4:                                       ; CODE XREF: SDPI:0047DCBE p  
SDPI:0047DCC4                 pop     eax  
SDPI:0047DCC5                 add     eax, 11h  
SDPI:0047DCCA                 push    eax  
SDPI:0047DCCB                 jmp     loc\_480DC7  
SDPI:0047DCCB ; ----------------------------------------------------------------------------  
SDPI:0047DCD0                 db  90h ; ?  
SDPI:0047DCD1                 db  90h ; ?  
SDPI:0047DCD2                 db  90h ; ?  
SDPI:0047DCD3                 db  90h ; ?  
SDPI:0047DCD4 ; ----------------------------------------------------------------------------  
SDPI:0047DCD4                 call    loc\_47DCDA  
SDPI:0047DCD9                 nop  
SDPI:0047DCDA  
SDPI:0047DCDA loc\_47DCDA:                                       ; CODE XREF: SDPI:0047DCD4 p  
SDPI:0047DCDA                 pop     eax  
SDPI:0047DCDB                 add     eax, 11h  
SDPI:0047DCE0                 push    eax  
SDPI:0047DCE1                 jmp     CHK\_UnHndexcptCC          ; 没多大作用检测,检测 UnhandleExceptxx  
SDPI:0047DCE1                                                   ; API 的第一个字节是否下断了  
SDPI:0047DCE1 ; ----------------------------------------------------------------------------  
SDPI:0047DCE6                 db 4 dup(90h)  
SDPI:0047DCEA ; ----------------------------------------------------------------------------  
SDPI:0047DCEA                 call    loc\_47DCF0  
SDPI:0047DCEF                 nop  
SDPI:0047DCF0  
SDPI:0047DCF0 loc\_47DCF0:                                       ; CODE XREF: SDPI:0047DCEA p  
SDPI:0047DCF0                 pop     eax  
SDPI:0047DCF1                 add     eax, 11h  
SDPI:0047DCF6                 push    eax  
SDPI:0047DCF7                 jmp     WaitThread                ; 这里再次判断是否为单 CPU，  
SDPI:0047DCF7                                                   ; 如果是单 CPU 则等待前面创建两个新线程  
SDPI:0047DCF7                                                   ; 的动作结束  
SDPI:0047DCF7 ; ----------------------------------------------------------------------------  
SDPI:0047DCFC                 db 4 dup(90h)  
SDPI:0047DD00 ; ----------------------------------------------------------------------------  
SDPI:0047DD00                 call    loc\_47DD06  
SDPI:0047DD05                 nop  
SDPI:0047DD06  
SDPI:0047DD06 loc\_47DD06:                                       ; CODE XREF: SDPI:0047DD00 p  
SDPI:0047DD06                 pop     eax  
SDPI:0047DD07                 add     eax, 11h  
SDPI:0047DD0C                 push    eax  
SDPI:0047DD0D                 jmp     Erase\_PEHEADER            ; 修改 PE 头部信息  
SDPI:0047DD0D ; ----------------------------------------------------------------------------  
SDPI:0047DD12                 db 4 dup(90h)  
SDPI:0047DD16 ; ----------------------------------------------------------------------------  
SDPI:0047DD16                 call    loc\_47DD1C  
SDPI:0047DD1B                 nop  
SDPI:0047DD1C  
SDPI:0047DD1C loc\_47DD1C:                                       ; CODE XREF: SDPI:0047DD16 p  
SDPI:0047DD1C                 pop     eax  
SDPI:0047DD1D                 add     eax, 11h  
SDPI:0047DD22                 push    eax  
SDPI:0047DD23                 jmp     MD5CHK\_4813E4             ; 再次检测 MD5 值是否相等  
SDPI:0047DD23 ; ----------------------------------------------------------------------------  
SDPI:0047DD28                 db 4 dup(90h)  
SDPI:0047DD2C ; ----------------------------------------------------------------------  
很多相似的调试器检测，跳过一堆重复代码的分析,来到这里：  
; ----------------------------------------------------------------------------  
SDPI:0047E35D                 call    loc\_47E363  
SDPI:0047E362                 nop  
SDPI:0047E363  
SDPI:0047E363 loc\_47E363:                                       ; CODE XREF: SDPI:0047E35D p  
SDPI:0047E363                 pop     eax  
SDPI:0047E364                 add     eax, 11h  
SDPI:0047E369                 push    eax  
SDPI:0047E36A                 jmp     Stolen\_CODE               ; 这里开始抽程序的入口代码  
SDPI:0047E36A ; ----------------------------------------------------------------------------  
SDPI:0047E36F                 db  90h ; ?  
SDPI:0047E370                 db  90h ; ?  
SDPI:0047E371                 db  90h ; ?  
SDPI:0047E372                 db  90h ; ?  
又跳过一段无用的代码来到这里：  
   
SDPI:0047E9B7 ; ----------------------------------------------------------------------------  
SDPI:0047E9B7                 pop     ecx  
SDPI:0047E9B8                 add     eax, \[ecx\]  
SDPI:0047E9BA                 mov     edi, eax  
SDPI:0047E9BC                 push    edi  
SDPI:0047E9BD                 call    loc\_47E9C3  
SDPI:0047E9C2                 nop  
SDPI:0047E9C3  
SDPI:0047E9C3 loc\_47E9C3:                                       ; CODE XREF: SDPI:0047E9BD p  
SDPI:0047E9C3                 pop     eax  
SDPI:0047E9C4                 add     eax, 0FFFFB76Eh  
SDPI:0047E9C9                 mov     ecx, \[eax\]  
SDPI:0047E9CB                 call    loc\_47E9D1  
SDPI:0047E9D0                 nop  
SDPI:0047E9D1  
SDPI:0047E9D1 loc\_47E9D1:                                       ; CODE XREF: SDPI:0047E9CB p  
SDPI:0047E9D1                 pop     eax  
SDPI:0047E9D2                 add     eax, 0FFFFB75Ch  
SDPI:0047E9D7                 add     ecx, \[eax\]  
SDPI:0047E9D9                 call    loc\_47E9DF  
SDPI:0047E9DE                 nop  
SDPI:0047E9DF  
SDPI:0047E9DF loc\_47E9DF:                                       ; CODE XREF: SDPI:0047E9D9 p  
SDPI:0047E9DF                 pop     eax  
SDPI:0047E9E0                 add     eax, 0FFFFB73Ah  
SDPI:0047E9E5                 sub     ecx, \[eax\]  
SDPI:0047E9E7                 cld  
SDPI:0047E9E8                 mov     al, 0C3h  
SDPI:0047E9EA  
SDPI:0047E9EA loop\_47E9EA:                                      ; CODE XREF: SDPI:0047E9FF j  
SDPI:0047E9EA                 repne scasb                       ; 从 OEP 处开始搜索 0C3  
SDPI:0047E9EC                 cmp     ecx, 0  
SDPI:0047E9EF                 jle     short loc\_47EA04  
SDPI:0047E9F1                 jmp     short loc\_47E9F4  
SDPI:0047E9F1 ; ----------------------------------------------------------------------------  
SDPI:0047E9F3                 db  90h ; ?  
SDPI:0047E9F4 ; ----------------------------------------------------------------------------  
SDPI:0047E9F4  
SDPI:0047E9F4 loc\_47E9F4:                                       ; CODE XREF: SDPI:0047E9F1 j  
SDPI:0047E9F4                 dec     edi  
SDPI:0047E9F5                 call    edi  
SDPI:0047E9F7                 nop  
SDPI:0047E9F8                 nop  
SDPI:0047E9F9                 nop  
SDPI:0047E9FA                 nop  
SDPI:0047E9FB                 nop  
SDPI:0047E9FC                 push    edi                       ; 这里的不断 push 也就造成了后面的 N 个 ret  
SDPI:0047E9FD                 inc     edi  
SDPI:0047E9FE                 dec     ecx  
SDPI:0047E9FF                 jg      short loop\_47E9EA         ; 从 OEP 处开始搜索 0C3  
SDPI:0047EA01                 jmp     short loc\_47EA04  
SDPI:0047EA01 ; ----------------------------------------------------------------------------  
SDPI:0047EA03                 db  90h ; ?  
SDPI:0047EA04 ; ----------------------------------------------------------------------------  
SDPI:0047EA04  
SDPI:0047EA04 loc\_47EA04:                                       ; CODE XREF: SDPI:0047E9EF j  
SDPI:0047EA04                                                   ; SDPI:0047EA01 j  
SDPI:0047EA04                 sub     esp, 1000h  
省略 N 多代码到这里:  
047F023                                                   ; 的下一行代码地址  
SDPI:0047F028                 add     esp, 1000h  
SDPI:0047F02E                 jp      short loc\_47F036  
SDPI:0047F030                 jnp     short loc\_47F036  
SDPI:0047F032                 xor     al, 0EFh  
SDPI:0047F034                 sub     al, \[ecx\]  
SDPI:0047F036  
SDPI:0047F036 loc\_47F036:                                       ; CODE XREF: SDPI:0047F02E j  
SDPI:0047F036                                                   ; SDPI:0047F030 j  
SDPI:0047F036                 xor     eax, eax  
SDPI:0047F038                 mov     ecx, 4E22h  
SDPI:0047F03D                 call    sub\_47F043  
SDPI:0047F042                 nop  
SDPI:0047F043  
SDPI:0047F043 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047F043  
SDPI:0047F043  
SDPI:0047F043 sub\_47F043      proc near                         ; CODE XREF: SDPI:0047F03D p  
SDPI:0047F043                 pop     edi  
SDPI:0047F044                 add     edi, 0FFFFB1EAh  
SDPI:0047F04A                 jo      short loc\_47F056          ; 清场处理:-),结束了跳去 N 个 Ret,然后返回到程序 oep  
SDPI:0047F04C                 jno     short loc\_47F056          ; 清场处理:-),结束了跳去 N 个 Ret,然后返回到程序 oep  
SDPI:0047F04E                 scasb  
SDPI:0047F04F                 mov     al, 42h  
SDPI:0047F051                 add     eax, \[eax\]  
SDPI:0047F053                 adc     \[eax+0\], al  
SDPI:0047F056  
SDPI:0047F056 loc\_47F056:                                       ; CODE XREF: sub\_47F043+7 j  
SDPI:0047F056                                                   ; sub\_47F043+9 j  
SDPI:0047F056                 rep stosb                         ; 清场处理:-),结束了跳去 N 个 Ret,然后返回到程序 oep  
SDPI:0047F058                 retn  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*以下是各模块\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047F061  
SDPI:0047F061 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047F061  
SDPI:0047F061 ; 获取实际地址  
SDPI:0047F061  
SDPI:0047F061 ; int \_\_stdcall GetAbsAddress(int relativeAddress)  
SDPI:0047F061 GetAbsAddress   proc near                         ; CODE XREF: GetFileMD5Val\_47F542+3D p  
SDPI:0047F061                                                   ; sub\_47F6AA+22 p ...  
SDPI:0047F061  
SDPI:0047F061 relativeAddress = dword ptr  4  
SDPI:0047F061  
SDPI:0047F061                 call    $+5  
SDPI:0047F066                 pop     eax  
SDPI:0047F067                 sub     eax, 406066h  
SDPI:0047F06C                 add     eax, \[esp+relativeAddress\]  
SDPI:0047F070                 retn    4  
SDPI:0047F070 GetAbsAddress   endp  
SDPI:0047F070  
SDPI:0047F070 ; ----------------------------------------------------------------------  
……  
SDPI:0047F077 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047F077  
SDPI:0047F077  
SDPI:0047F077 Get\_BASE        proc near                         ; CODE XREF: CRC\_480467+6 p  
SDPI:0047F077                                                   ; CRC\_480467+1A p ...  
SDPI:0047F077                 call    loc\_47F07D  
SDPI:0047F07C                 nop  
SDPI:0047F07D  
SDPI:0047F07D loc\_47F07D:                                       ; CODE XREF: Get\_BASE p  
SDPI:0047F07D                 pop     eax  
SDPI:0047F07E                 add     eax, 0FFFFB084h  
SDPI:0047F083                 retn  
SDPI:0047F083 Get\_BASE        endp ; sp =  4  
SDPI:0047F083  
SDPI:0047F083 ; ----------------------------------------------------------------------  
……  
SDPI:0047F088 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047F088  
SDPI:0047F088  
SDPI:0047F088 Decode\_47F088   proc near                         ; CODE XREF: CRC\_480467+89 p  
SDPI:0047F088                                                   ; MD5CHK\_4813E4+2A p ...  
SDPI:0047F088                 pop     eax  
SDPI:0047F089                 push    eax                       ; EDX  
SDPI:0047F08A                 add     eax, 64h  
SDPI:0047F08D                 mov     edx, eax  
SDPI:0047F08F                 add     edx, 32h  
SDPI:0047F092                 call    Crypt\_Decrypt\_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047F092                                                   ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047F092                                                   ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047F092                                                   ; 用来解密  
SDPI:0047F097                 retn  
SDPI:0047F097 Decode\_47F088   endp ; sp =  8  
SDPI:0047F097  
SDPI:0047F097 ; ----------------------------------------------------------------------------  
连他的 Information 都抓 出来的说.  
SDPI:0047F098                 dd 62AEF34h  
SDPI:0047F09C aDebuggerDetect db 'Debugger detected  - please close it down and restart!',0Dh,0Ah  
SDPI:0047F09C                 db 0Dh,0Ah  
SDPI:0047F09C                 db 'For some debuggers, such as SoftIce, you must restart ',0Dh,0Ah  
SDPI:0047F09C                 db 'this machine without it enabled to run this application!',0Dh  
SDPI:0047F09C                 db 0Ah,0  
SDPI:0047F149 aExit\_\_\_        db 'Exit...',0Dh,0Ah,0  
SDPI:0047F153 aThisProgramIsProtec db 'This program is protected by unregistered version of SoftDef'  
SDPI:0047F153                 db 'ender',0Dh,0Ah  
SDPI:0047F153                 db 0Dh,0Ah  
SDPI:0047F153                 db 'This message will not appear on programs protected by a regi'  
SDPI:0047F153                 db 'stered ',0Dh,0Ah  
SDPI:0047F153                 db 'verson of SoftDefender, please register it from:',0Dh,0Ah  
SDPI:0047F153                 db 0Dh,0Ah  
SDPI:0047F153                 db '                 http://www.softdefender.com/order.htm',0Dh,0Ah  
SDPI:0047F153                 db 0  
SDPI:0047F24A aUnregisteredVersion db 'Unregistered Version',0Dh,0Ah,0  
  
  
SDPI:0047F261  
SDPI:0047F261 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047F261  
SDPI:0047F261 ; 检测调试器是否存在  
SDPI:0047F261  
SDPI:0047F261 INT3\_47F261     proc near                         ; CODE XREF: Disposal\_IMP+A p  
SDPI:0047F261                                                   ; CreateThread2+3 p ...  
SDPI:0047F261  
SDPI:0047F261 var\_4           = dword ptr -4  
SDPI:0047F261 arg\_0           = dword ptr  4  
SDPI:0047F261  
SDPI:0047F261                 jz      short loc\_47F26D  
SDPI:0047F263                 jnz     short loc\_47F26D  
SDPI:0047F263 ; ----------------------------------------------------------------------------  
SDPI:0047F265                 db 0  
SDPI:0047F266                 db  10h  
SDPI:0047F267                 db 40h  
SDPI:0047F268                 db 0  
SDPI:0047F269                 db  0Bh  
SDPI:0047F26A                 db 0ABh  
SDPI:0047F26B                 db 0F7h ; ?  
SDPI:0047F26C                 db    9  
SDPI:0047F26D ; ----------------------------------------------------------------------------  
SDPI:0047F26D  
SDPI:0047F26D loc\_47F26D:                                       ; CODE XREF: INT3\_47F261 j  
SDPI:0047F26D                                                   ; INT3\_47F261+2 j  
SDPI:0047F26D                 call    loc\_47F273  
SDPI:0047F272                 nop  
SDPI:0047F273  
SDPI:0047F273 loc\_47F273:                                       ; CODE XREF: INT3\_47F261:loc\_47F26D p  
SDPI:0047F273                 pop     eax  
SDPI:0047F274                 add     eax, 0D1h  
SDPI:0047F279                 push    eax                       ; EDX  
SDPI:0047F27A                 xor     eax, eax  
SDPI:0047F27C                 push    dword ptr fs:\[eax\]  
SDPI:0047F27F                 mov     fs:\[eax\], esp  
SDPI:0047F282                 mov     ebp, 300EF1D3h            ; 设置 Magic  
SDPI:0047F287                 add     ebp, 12345678h  
SDPI:0047F28D                 mov     ax, 17h  
SDPI:0047F291                 sub     ax, 13h                   ; 产生异常  
SDPI:0047F295                 int     3                         ; Trap to Debugger  
SDPI:0047F296                 nop  
SDPI:0047F297                 cmp     al, 4                     ; 如果用了 sice int3 之后返回的就不是 4 了  
SDPI:0047F299                 jz      short NOICE\_47F30C  
SDPI:0047F29B  
SDPI:0047F29B OVER\_47F29B:                                      ; CODE XREF: INT3\_47F261+C4 j  
SDPI:0047F29B                                                   ; INT3\_47F261+CF j  
SDPI:0047F29B                 nop  
SDPI:0047F29C                 nop  
SDPI:0047F29D                 nop  
SDPI:0047F29E                 nop  
SDPI:0047F29F                 nop  
SDPI:0047F2A0                 call    loc\_47F2A6  
SDPI:0047F2A5                 nop  
SDPI:0047F2A6  
SDPI:0047F2A6 loc\_47F2A6:                                       ; CODE XREF: INT3\_47F261+3F p  
SDPI:0047F2A6                 pop     eax  
SDPI:0047F2A7                 add     eax, 5Eh  
SDPI:0047F2AC                 mov     edx, eax  
SDPI:0047F2AE                 add     edx, 32h                  ; 0047F335  
SDPI:0047F2B1                 call    Crypt\_Decrypt\_CODE        ; 解密 47f303 处的代码  
SDPI:0047F2B1                                                   ; 大小为 32h  
SDPI:0047F2B6                 call    loc\_47F2BC  
SDPI:0047F2BB                 nop  
SDPI:0047F2BC  
SDPI:0047F2BC loc\_47F2BC:                                       ; CODE XREF: INT3\_47F261+55 p  
SDPI:0047F2BC                 pop     eax  
SDPI:0047F2BD                 add     eax, 0FFFFFDE1h           ; eax 指向 aDebuggerDetect(0047F09C)  
SDPI:0047F2C2                 call    loc\_47F2C8  
SDPI:0047F2C7                 nop  
SDPI:0047F2C8  
SDPI:0047F2C8 loc\_47F2C8:                                       ; CODE XREF: INT3\_47F261+61 p  
SDPI:0047F2C8                 pop     ecx  
SDPI:0047F2C9                 add     ecx, 0FFFFFE82h           ; ecx 指向"exit"(0047F149)  
SDPI:0047F2CF                 push    0  
SDPI:0047F2D1                 push    ecx  
SDPI:0047F2D2                 push    eax  
SDPI:0047F2D3                 push    0  
SDPI:0047F2D5                 call    loc\_47F2DB  
SDPI:0047F2DA                 nop  
SDPI:0047F2DB  
SDPI:0047F2DB loc\_47F2DB:                                       ; CODE XREF: INT3\_47F261+74 p  
SDPI:0047F2DB                 pop     eax  
SDPI:0047F2DC                 add     eax, 11h  
SDPI:0047F2E1                 push    eax  
SDPI:0047F2E2                 jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047F2E2                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047F2E2                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047F2E7 ; ----------------------------------------------------------------------------  
SDPI:0047F2E7                 nop                               ; 作者设计是有误的，这里检测到调试器后  
SDPI:0047F2E7                                                   ; 肯定出不来 MessageBox 的.  
SDPI:0047F2E7                                                   ; 因为这里不管是鸡生蛋还是蛋生鸡，  
SDPI:0047F2E7                                                   ; 鸡和蛋都没有，所以不可能会出信息框的  
SDPI:0047F2E8                 nop  
SDPI:0047F2E9                 nop  
SDPI:0047F2EA                 nop  
SDPI:0047F2EB                 push    7  
SDPI:0047F2ED                 call    loc\_47F2F3                ; 这里也同样  
SDPI:0047F2F2                 nop  
SDPI:0047F2F3  
SDPI:0047F2F3 loc\_47F2F3:                                       ; CODE XREF: INT3\_47F261+8C p  
SDPI:0047F2F3                 pop     eax  
SDPI:0047F2F4                 add     eax, 11h  
SDPI:0047F2F9                 push    eax  
SDPI:0047F2FA                 jmp     ExitProcess  
SDPI:0047F2FF ; ----------------------------------------------------------------------------  
SDPI:0047F2FF                 nop  
SDPI:0047F300                 nop  
SDPI:0047F301                 nop  
SDPI:0047F302                 nop  
SDPI:0047F303                 call    near ptr 880308h  
SDPI:0047F308                 mov     al, 89h  
SDPI:0047F30A                 pushf  
SDPI:0047F30A ; ----------------------------------------------------------------------------  
SDPI:0047F30B                 db    4  
SDPI:0047F30C ; ----------------------------------------------------------------------------  
SDPI:0047F30C  
SDPI:0047F30C NOICE\_47F30C:                                     ; CODE XREF: INT3\_47F261+38 j  
SDPI:0047F30C                 pop     large dword ptr fs:0  
SDPI:0047F313                 add     esp, 4                    ; 如果没有检测到 ICE，就进行时间检测  
SDPI:0047F316                 rdtsc  
SDPI:0047F318                 mov     ecx, eax  
SDPI:0047F31A                 mov     ebx, edx  
SDPI:0047F31C                 rdtsc  
SDPI:0047F31E                 sub     eax, ecx  
SDPI:0047F320                 sbb     edx, ebx  
SDPI:0047F322                 cmp     edx, 0  
SDPI:0047F325                 jnz     OVER\_47F29B               ; 如果 edx 不为 0 则 over  
SDPI:0047F32B                 cmp     eax, 30000000h  
SDPI:0047F330                 ja      OVER\_47F29B               ; 如果时间大于 30000000h 也 over 了  
SDPI:0047F336                 jz      short Continue\_47F369  
SDPI:0047F338                 jnz     short Continue\_47F369  
SDPI:0047F338 ; ----------------------------------------------------------------------------  
SDPI:0047F33A                 db 0E8h                           ; junk  
SDPI:0047F33B                 db    0  
SDPI:0047F33C                 db  10h  
SDPI:0047F33D                 db  40h ; @  
SDPI:0047F33E                 db    0  
SDPI:0047F33F                 db 0B0h  
SDPI:0047F340                 db  89h ; ?  
SDPI:0047F341                 db 9Ch  
SDPI:0047F342                 db    4  
SDPI:0047F343 ; ----------------------------------------------------------------------------  
SDPI:0047F343                 mov     eax, \[esp+arg\_0\]  
SDPI:0047F347                 mov     ecx, \[esp+0Ch\]  
SDPI:0047F34B                 inc     dword ptr \[ecx+0B8h\]      ; reg\[EIP\]+1  
SDPI:0047F351                 mov     eax, \[eax\]  
SDPI:0047F353                 sub     eax, 80000003h            ; 判断是否为 INT3 异常  
SDPI:0047F358                 jnz     short locret\_47F368  
SDPI:0047F35A                 xor     eax, eax                  ; 如果为 INT3 异常则清除硬件断点  
SDPI:0047F35C                 mov     \[ecx+4\], eax  
SDPI:0047F35F                 mov     \[ecx+8\], eax  
SDPI:0047F362                 mov     \[ecx+0Ch\], eax  
SDPI:0047F365                 mov     \[ecx+10h\], eax  
SDPI:0047F368  
SDPI:0047F368 locret\_47F368:                                    ; CODE XREF: INT3\_47F261+F7 j  
SDPI:0047F368                 retn  
SDPI:0047F369 ; ----------------------------------------------------------------------------  
SDPI:0047F369  
SDPI:0047F369 Continue\_47F369:                                  ; CODE XREF: INT3\_47F261+D5 j  
SDPI:0047F369                                                   ; INT3\_47F261+D7 j  
SDPI:0047F369                 retn  
SDPI:0047F369 INT3\_47F261     endp  
SDPI:0047F369  
SDPI:0047F369 ; ----------------------------------------------------------------------  
  
X?  
SDPI:0047F381 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047F381  
SDPI:0047F381 ; MD5 值加密代码,调用方法:  
SDPI:0047F381 ; invoke Crypt\_Code,End,Start  
SDPI:0047F381 ; end 加密结束地址,地址在 EDX 中  
SDPI:0047F381 ; start 加密起始地址,地址在 eax 中  
SDPI:0047F381 ;  
SDPI:0047F381  
SDPI:0047F381 ; int \_\_stdcall Crypt\_Code(int endedx,int starteax)  
SDPI:0047F381 Crypt\_Code      proc near                         ; CODE XREF: SDPI:0047A4FC p  
SDPI:0047F381                                                   ; SDPI:0047AE1B p ...  
SDPI:0047F381  
SDPI:0047F381 endedx          = dword ptr  4  
SDPI:0047F381 starteax        = dword ptr  8  
SDPI:0047F381  
SDPI:0047F381                 jb      short loc\_47F38D  
SDPI:0047F383                 jnb     short loc\_47F397  
SDPI:0047F383 ; ----------------------------------------------------------------------------  
SDPI:0047F385                 dd 401000h  
SDPI:0047F389                 dd 56C9FFCDh  
SDPI:0047F38D ; ----------------------------------------------------------------------------  
SDPI:0047F38D  
SDPI:0047F38D loc\_47F38D:                                       ; CODE XREF: Crypt\_Code j  
SDPI:0047F38D                 jb      short loc\_47F3A1  
SDPI:0047F38D ; ----------------------------------------------------------------------------  
SDPI:0047F38F                 dd 401000h  
SDPI:0047F393                 dd 8A29B3D7h  
SDPI:0047F397 ; ----------------------------------------------------------------------------  
SDPI:0047F397  
SDPI:0047F397 loc\_47F397:                                       ; CODE XREF: Crypt\_Code+2 j  
SDPI:0047F397                 jnb     short loc\_47F3A1  
SDPI:0047F397 ; ----------------------------------------------------------------------------  
SDPI:0047F399 a@LY            db 0,10h,'@',0,'蝎',0Ah,'y'  
SDPI:0047F3A1 ; ----------------------------------------------------------------------------  
SDPI:0047F3A1  
SDPI:0047F3A1 loc\_47F3A1:                                       ; CODE XREF: Crypt\_Code:loc\_47F38D j  
SDPI:0047F3A1                                                   ; Crypt\_Code:loc\_47F397 j  
SDPI:0047F3A1                 push    ecx  
SDPI:0047F3A2                 push    ebx  
SDPI:0047F3A3                 push    esi  
SDPI:0047F3A4                 push    edi  
SDPI:0047F3A5                 mov     esi, eax  
SDPI:0047F3A7                 cmp     ecx, 0  
SDPI:0047F3AA                 jz      short loc\_47F412  
SDPI:0047F3AC                 push    esi  
SDPI:0047F3AD                 push    eax  
SDPI:0047F3AE                 push    edx  
SDPI:0047F3AF                 cmp     ecx, 459E3C92h  
SDPI:0047F3B5                 jz      short loc\_47F3E1  
SDPI:0047F3B7                 call    loc\_47F3BD  
SDPI:0047F3BC                 nop  
SDPI:0047F3BD  
SDPI:0047F3BD loc\_47F3BD:                                       ; CODE XREF: Crypt\_Code+36 p  
SDPI:0047F3BD                 pop     ebx  
SDPI:0047F3BE                 add     ebx, 0FFFFAE24h  
SDPI:0047F3C4                 push    ebx  
SDPI:0047F3C5                 push    ebx  
SDPI:0047F3C6                 call    loc\_47F3CC  
SDPI:0047F3CB                 nop  
SDPI:0047F3CC  
SDPI:0047F3CC loc\_47F3CC:                                       ; CODE XREF: Crypt\_Code+45 p  
SDPI:0047F3CC                 pop     eax  
SDPI:0047F3CD                 add     eax, 11h  
SDPI:0047F3D2                 push    eax  
SDPI:0047F3D3                 jmp     HASH\_MD5\_812E8            ; 计算 MD5 值,其实这里存在 BUG 了  
SDPI:0047F3D3                                                   ; 如果第一次检测 MD5 值时  
SDPI:0047F3D3                                                   ; 代码没有被修改那么后面再  
SDPI:0047F3D3                                                   ; 调用这个函数时的缓冲地址里  
SDPI:0047F3D3                                                   ; 就是保存了正确的 MD5 值  
SDPI:0047F3D3 ; ----------------------------------------------------------------------------  
SDPI:0047F3D8                 dd 90909090h  
SDPI:0047F3DC ; ----------------------------------------------------------------------------  
SDPI:0047F3DC                 jp      short loc\_47F406  
SDPI:0047F3DE                 jnp     short loc\_47F406  
SDPI:0047F3DE ; ----------------------------------------------------------------------------  
SDPI:0047F3E0                 db 0E8h  
SDPI:0047F3E1 ; ----------------------------------------------------------------------------  
SDPI:0047F3E1  
SDPI:0047F3E1 loc\_47F3E1:                                       ; CODE XREF: Crypt\_Code+34 j  
SDPI:0047F3E1                 call    loc\_47F3E7  
SDPI:0047F3E6                 nop  
SDPI:0047F3E7  
SDPI:0047F3E7 loc\_47F3E7:                                       ; CODE XREF: Crypt\_Code:loc\_47F3E1 p  
SDPI:0047F3E7                 pop     ebx  
SDPI:0047F3E8                 add     ebx, 0FFFFAE1Ah  
SDPI:0047F3EE                 push    ebx  
SDPI:0047F3EF                 push    ebx  
SDPI:0047F3F0                 call    loc\_47F3F6  
SDPI:0047F3F5                 nop  
SDPI:0047F3F6  
SDPI:0047F3F6 loc\_47F3F6:                                       ; CODE XREF: Crypt\_Code+6F p  
SDPI:0047F3F6                 pop     eax  
SDPI:0047F3F7                 add     eax, 11h  
SDPI:0047F3FC                 push    eax  
SDPI:0047F3FD                 jmp     sub\_481362  
SDPI:0047F3FD ; ----------------------------------------------------------------------------  
SDPI:0047F402                 dd 90909090h  
SDPI:0047F406 ; ----------------------------------------------------------------------------  
SDPI:0047F406  
SDPI:0047F406 loc\_47F406:                                       ; CODE XREF: Crypt\_Code+5B j  
SDPI:0047F406                                                   ; Crypt\_Code+5D j  
SDPI:0047F406                 pop     ebx  
SDPI:0047F407                 pop     edx  
SDPI:0047F408                 pop     eax  
SDPI:0047F409                 pop     esi  
SDPI:0047F40A                 jp      short loc\_47F41F  
SDPI:0047F40C                 jnp     short loc\_47F41F  
SDPI:0047F40C ; ----------------------------------------------------------------------------  
SDPI:0047F40E                 dd 401000h  
SDPI:0047F412 ; ----------------------------------------------------------------------------  
SDPI:0047F412  
SDPI:0047F412 loc\_47F412:                                       ; CODE XREF: Crypt\_Code+29 j  
SDPI:0047F412                 call    loc\_47F418  
SDPI:0047F417                 nop  
SDPI:0047F418  
SDPI:0047F418 loc\_47F418:                                       ; CODE XREF: Crypt\_Code:loc\_47F412 p  
SDPI:0047F418                 pop     ebx  
SDPI:0047F419                 add     ebx, 0FFFFADD9h  
SDPI:0047F41F  
SDPI:0047F41F loc\_47F41F:                                       ; CODE XREF: Crypt\_Code+89 j  
SDPI:0047F41F                                                   ; Crypt\_Code+8B j  
SDPI:0047F41F                 jp      short loc\_47F42B  
SDPI:0047F421                 jnp     short loc\_47F42B  
SDPI:0047F421 ; ----------------------------------------------------------------------------  
SDPI:0047F423                 dd 401000h  
SDPI:0047F427                 dd 769E3CF2h  
SDPI:0047F42B ; ----------------------------------------------------------------------------  
SDPI:0047F42B  
SDPI:0047F42B loc\_47F42B:                                       ; CODE XREF: Crypt\_Code:loc\_47F41F j  
SDPI:0047F42B                                                   ; Crypt\_Code+A0 j  
SDPI:0047F42B                 nop  
SDPI:0047F42C                 nop  
SDPI:0047F42D                 nop  
SDPI:0047F42E                 nop  
SDPI:0047F42F                 nop  
SDPI:0047F430  
SDPI:0047F430 loc\_47F430:                                       ; CODE XREF: Crypt\_Code+C3 j  
SDPI:0047F430                 xor     edi, edi  
SDPI:0047F432                 mov     ecx, 10h                  ; 循环长度 10h  
SDPI:0047F437  
SDPI:0047F437 loc\_47F437:                                       ; CODE XREF: Crypt\_Code+C1 j  
SDPI:0047F437                 cmp     esi, edx  
SDPI:0047F439                 jnb     short loc\_47F446  
SDPI:0047F43B                 mov     al, \[edi+ebx\]             ; 这里的 KEY 来源，就是  
SDPI:0047F43B                                                   ; 上面计算出来的 md5 值  
SDPI:0047F43B                                                   ; 所在地址:0047A1E0  
SDPI:0047F43E                 xor     \[esi\], al                 ; ESI==0047AAE6  
SDPI:0047F440                 inc     esi  
SDPI:0047F441                 inc     edi  
SDPI:0047F442                 loop    loc\_47F437  
SDPI:0047F444                 jmp     short loc\_47F430  
SDPI:0047F446 ; ------------------------------------------------------------------  
  
\]D  
SDPI:0047F469 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047F469  
SDPI:0047F469 ; 传入 edx(结束地址)传入 EAX(起始地址)  
SDPI:0047F469 ; 用于加密或解密代码,加解密操作是互换的  
SDPI:0047F469 ; 是只简单的 xor,因此可以用来加密也可以  
SDPI:0047F469 ; 用来解密  
SDPI:0047F469  
SDPI:0047F469 ; int \_\_stdcall Crypt\_Decrypt\_CODE(int EDX,int eax)  
SDPI:0047F469 Crypt\_Decrypt\_CODE proc near                      ; CODE XREF: sub\_47A471+B p  
SDPI:0047F469                                                   ; sub\_47A591+B p ...  
SDPI:0047F469  
SDPI:0047F469 EDX             = dword ptr  4  
SDPI:0047F469 eax             = dword ptr  8  
SDPI:0047F469  
SDPI:0047F469                 jb      short near ptr JUNK\_47F46D+8  
SDPI:0047F46B                 jnb     short loc\_47F47F  
SDPI:0047F46B ; ----------------------------------------------------------------------------  
SDPI:0047F46D JUNK\_47F46D     db 0,10h,'@',0,'?,0FFh,'蒝 r',12h,0,10h,'@',0,'壮)?  
SDPI:0047F46D                                                   ; CODE XREF: Crypt\_Decrypt\_CODE j  
SDPI:0047F47F ; ----------------------------------------------------------------------------  
SDPI:0047F47F  
SDPI:0047F47F loc\_47F47F:                                       ; CODE XREF: Crypt\_Decrypt\_CODE+2 j  
SDPI:0047F47F                 jnb     short loc\_47F489  
SDPI:0047F47F ; ----------------------------------------------------------------------------  
SDPI:0047F481 JUNK\_47F481     db 0,10h,'@',0,'蝎',0Ah,'y'  
SDPI:0047F489 ; ----------------------------------------------------------------------------  
SDPI:0047F489  
SDPI:0047F489 loc\_47F489:                                       ; CODE XREF: Crypt\_Decrypt\_CODE:loc\_47F47F j  
SDPI:0047F489                 push    ebx  
SDPI:0047F48A                 push    esi  
SDPI:0047F48B                 push    edi  
SDPI:0047F48C                 mov     esi, eax  
SDPI:0047F48E                 call    loc\_47F494  
SDPI:0047F493                 nop  
SDPI:0047F494  
SDPI:0047F494 loc\_47F494:                                       ; CODE XREF: Crypt\_Decrypt\_CODE+25 p  
SDPI:0047F494                 pop     ebx  
SDPI:0047F495                 add     ebx, 0FFFFAD05h  
SDPI:0047F49B                 jp      short loc\_47F4A7  
SDPI:0047F49D                 jnp     short loc\_47F4A7  
SDPI:0047F49D ; ----------------------------------------------------------------------------  
SDPI:0047F49F JUNK\_47F49F     db 0,10h,'@',0,'?瀡'  
SDPI:0047F4A7 ; ----------------------------------------------------------------------------  
SDPI:0047F4A7  
SDPI:0047F4A7 loc\_47F4A7:                                       ; CODE XREF: Crypt\_Decrypt\_CODE+32 j  
SDPI:0047F4A7                                                   ; Crypt\_Decrypt\_CODE+34 j  
SDPI:0047F4A7                 nop  
SDPI:0047F4A8                 nop  
SDPI:0047F4A9                 nop  
SDPI:0047F4AA                 nop  
SDPI:0047F4AB                 nop  
SDPI:0047F4AC  
SDPI:0047F4AC loc\_47F4AC:                                       ; CODE XREF: Crypt\_Decrypt\_CODE+57 j  
SDPI:0047F4AC                 xor     edi, edi  
SDPI:0047F4AE                 mov     ecx, 10h  
SDPI:0047F4B3  
SDPI:0047F4B3 loc\_47F4B3:                                       ; CODE XREF: Crypt\_Decrypt\_CODE+55 j  
SDPI:0047F4B3                 cmp     esi, edx  
SDPI:0047F4B5                 jnb     short loc\_47F4C2  
SDPI:0047F4B7                 mov     al, \[edi+ebx\]  
SDPI:0047F4BA                 xor     \[esi\], al  
SDPI:0047F4BC                 inc     esi  
SDPI:0047F4BD                 inc     edi  
SDPI:0047F4BE                 loop    loc\_47F4B3  
SDPI:0047F4C0                 jmp     short loc\_47F4AC  
SDPI:0047F4C2 ; ----------------------------------------------------------------------------  
SDPI:0047F4C2  
SDPI:0047F4C2 loc\_47F4C2:                                       ; CODE XREF: Crypt\_Decrypt\_CODE+4C j  
SDPI:0047F4C2                 call    loc\_47F4C8  
SDPI:0047F4C7                 nop  
SDPI:0047F4C8  
SDPI:0047F4C8 loc\_47F4C8:                                       ; CODE XREF: Crypt\_Decrypt\_CODE:loc\_47F4C2 p  
SDPI:0047F4C8                 pop     eax  
SDPI:0047F4C9                 add     eax, 19h  
SDPI:0047F4CE                 sub     esp, 0ED0h  
SDPI:0047F4D4                 push    eax  
SDPI:0047F4D5                 retn    0ED0h  
SDPI:0047F4D5 ; ----------------------------------------------------------------------------  
SDPI:0047F4D8 JUNK\_47F4d8     db 0,10h,'@',0,'檠 J',3  
SDPI:0047F4E0 ; ----------------------------------------------------------------------------  
SDPI:0047F4E0                 pop     edi  
SDPI:0047F4E1                 pop     esi  
SDPI:0047F4E2                 pop     ebx  
SDPI:0047F4E3                 retn  
SDPI:0047F4E3 Crypt\_Decrypt\_CODE endp ; sp =  0ED4h  
SDPI:0047F4E3  
SDPI:0047F4E3 ; ----------------------------------------------------------------------  
  
D  
SDPI:0047F524 aOutOfMemoryInHeapal db 'Out Of Memory in HeapAlloc1!',0Ah,0  
SDPI:0047F542  
SDPI:0047F542 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047F542  
SDPI:0047F542  
SDPI:0047F542 GetFileMD5Val\_47F542 proc near                    ; CODE XREF: File\_CRC\_CHK+87 p  
SDPI:0047F542  
SDPI:0047F542 InBuffer        = dword ptr -100h  
SDPI:0047F542 RetBuffer       = dword ptr -0CCh  
SDPI:0047F542 var\_A4          = dword ptr -0A4h  
SDPI:0047F542 var\_A0          = dword ptr -0A0h  
SDPI:0047F542 var\_8C          = dword ptr -8Ch  
SDPI:0047F542 initBuffer      = dword ptr -88h  
SDPI:0047F542 BufferSize      = dword ptr -6Ch  
SDPI:0047F542 NumberOfBytesRead= dword ptr -58h  
SDPI:0047F542 arg\_4           = dword ptr  8  
SDPI:0047F542  
SDPI:0047F542                 sub     esp, 58h  
SDPI:0047F545                 push    esi  
SDPI:0047F546                 mov     esi, \[esp+5Ch+arg\_4\]  
SDPI:0047F54A                 push    edi  
SDPI:0047F54B                 push    esi  
SDPI:0047F54C                 call    sub\_47F6AA  
SDPI:0047F551                 mov     edi, eax  
SDPI:0047F553                 test    edi, edi  
SDPI:0047F555                 jnz     short loc\_47F55F  
SDPI:0047F557                 pop     edi  
SDPI:0047F558                 pop     esi  
SDPI:0047F559                 add     esp, 58h  
SDPI:0047F55C                 retn    8  
SDPI:0047F55F ; ----------------------------------------------------------------------------  
SDPI:0047F55F  
SDPI:0047F55F loc\_47F55F:                                       ; CODE XREF: GetFileMD5Val\_47F542+13 j  
SDPI:0047F55F                 push    ebx  
SDPI:0047F560                 push    80000h  
SDPI:0047F565                 push    8  
SDPI:0047F567                 call    GetProcessHeap  
SDPI:0047F56C                 push    eax  
SDPI:0047F56D                 call    allocateHeap  
SDPI:0047F572                 mov     ebx, eax  
SDPI:0047F574                 test    ebx, ebx  
SDPI:0047F576                 jnz     short loc\_47F592  
SDPI:0047F578                 push    eax                       ; uType  
SDPI:0047F579                 push    eax                       ; lpCaption  
SDPI:0047F57A                 push    406524h                   ; relativeAddress  
SDPI:0047F57F                 call    GetAbsAddress             ; 获取实际地址  
SDPI:0047F584                 push    eax                       ; lpText  
SDPI:0047F585                 push    ebx                       ; hWnd  
SDPI:0047F586                 call    MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047F586                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:0047F586                                                   ; 也就是判断有没有下 int3 断点  
SDPI:0047F58B                 push    2                         ; uExitCode  
SDPI:0047F58D                 call    ExitProcess  
SDPI:0047F592  
SDPI:0047F592 loc\_47F592:                                       ; CODE XREF: GetFileMD5Val\_47F542+34 j  
SDPI:0047F592                 push    0                         ; dwMoveMethod  
SDPI:0047F594                 push    0                         ; lpDistanceToMoveHigh  
SDPI:0047F596                 push    0                         ; lDistanceToMove  
SDPI:0047F598                 push    esi                       ; hFile  
SDPI:0047F599                 call    SetFilePointer  
SDPI:0047F59E                 lea     eax, \[esp+0Ch\]  
SDPI:0047F5A2                 push    eax                       ; initBuffer  
SDPI:0047F5A3                 call    INIT\_MD5  
SDPI:0047F5A8                 add     esp, 4  
SDPI:0047F5AB                 push    1                         ; dwMoveMethod  
SDPI:0047F5AD                 push    0                         ; lpDistanceToMoveHigh  
SDPI:0047F5AF                 push    0                         ; lDistanceToMove  
SDPI:0047F5B1                 push    esi                       ; hFile  
SDPI:0047F5B2                 call    SetFilePointer  
SDPI:0047F5B7                 cmp     eax, edi  
SDPI:0047F5B9                 jnb     short loc\_47F61C  
SDPI:0047F5BB  
SDPI:0047F5BB loc\_47F5BB:                                       ; CODE XREF: GetFileMD5Val\_47F542+D8 j  
SDPI:0047F5BB                 push    1                         ; dwMoveMethod  
SDPI:0047F5BD                 push    0                         ; lpDistanceToMoveHigh  
SDPI:0047F5BF                 push    0                         ; lDistanceToMove  
SDPI:0047F5C1                 push    esi                       ; hFile  
SDPI:0047F5C2                 call    SetFilePointer  
SDPI:0047F5C7                 mov     ecx, edi  
SDPI:0047F5C9                 sub     ecx, eax  
SDPI:0047F5CB                 cmp     ecx, 80000h  
SDPI:0047F5D1                 jbe     short loc\_47F5DA  
SDPI:0047F5D3                 mov     ecx, 80000h  
SDPI:0047F5D8                 jmp     short loc\_47F5EA  
SDPI:0047F5DA ; ----------------------------------------------------------------------------  
SDPI:0047F5DA  
SDPI:0047F5DA loc\_47F5DA:                                       ; CODE XREF: GetFileMD5Val\_47F542+8F j  
SDPI:0047F5DA                 push    1                         ; dwMoveMethod  
SDPI:0047F5DC                 push    0                         ; lpDistanceToMoveHigh  
SDPI:0047F5DE                 push    0                         ; lDistanceToMove  
SDPI:0047F5E0                 push    esi                       ; hFile  
SDPI:0047F5E1                 call    SetFilePointer  
SDPI:0047F5E6                 mov     ecx, edi  
SDPI:0047F5E8                 sub     ecx, eax  
SDPI:0047F5EA  
SDPI:0047F5EA loc\_47F5EA:                                       ; CODE XREF: GetFileMD5Val\_47F542+96 j  
SDPI:0047F5EA                 lea     edx, \[esp+6Ch\]  
SDPI:0047F5EE                 push    0                         ; lpOverlapped  
SDPI:0047F5F0                 push    edx                       ; lpNumberOfBytesRead  
SDPI:0047F5F1                 push    ecx                       ; nNumberOfBytesToRead  
SDPI:0047F5F2                 push    ebx                       ; lpBuffer  
SDPI:0047F5F3                 push    esi                       ; hFile  
SDPI:0047F5F4                 call    ReadFile  
SDPI:0047F5F9                 lea     ecx, \[esp+0Ch\]  
SDPI:0047F5FD                 mov     eax, \[esp+0C4h+NumberOfBytesRead\]  
SDPI:0047F601                 push    eax                       ; BufferSize  
SDPI:0047F602                 push    ebx                       ; CalcBuffer  
SDPI:0047F603                 push    ecx                       ; RetBuffer  
SDPI:0047F604                 call    Hash\_MD5  
SDPI:0047F609                 add     esp, 0Ch  
SDPI:0047F60C                 push    1                         ; dwMoveMethod  
SDPI:0047F60E                 push    0                         ; lpDistanceToMoveHigh  
SDPI:0047F610                 push    0                         ; lDistanceToMove  
SDPI:0047F612                 push    esi                       ; hFile  
SDPI:0047F613                 call    SetFilePointer  
SDPI:0047F618                 cmp     eax, edi  
SDPI:0047F61A                 jb      short loc\_47F5BB  
SDPI:0047F61C  
SDPI:0047F61C loc\_47F61C:                                       ; CODE XREF: GetFileMD5Val\_47F542+77 j  
SDPI:0047F61C                 push    1                         ; dwMoveMethod  
SDPI:0047F61E                 push    0                         ; lpDistanceToMoveHigh  
SDPI:0047F620                 push    10h                       ; lDistanceToMove  
SDPI:0047F622                 push    esi                       ; hFile  
SDPI:0047F623                 call    SetFilePointer  
SDPI:0047F628                 mov     dword ptr \[esp+6Ch\], 80000h  
SDPI:0047F630  
SDPI:0047F630 loc\_47F630:                                       ; CODE XREF: GetFileMD5Val\_47F542+11D j  
SDPI:0047F630                 lea     edx, \[esp+6Ch\]  
SDPI:0047F634                 push    0                         ; lpOverlapped  
SDPI:0047F636                 push    edx                       ; lpNumberOfBytesRead  
SDPI:0047F637                 push    80000h                    ; nNumberOfBytesToRead  
SDPI:0047F63C                 push    ebx                       ; lpBuffer  
SDPI:0047F63D                 push    esi                       ; hFile  
SDPI:0047F63E                 call    ReadFile  
SDPI:0047F643                 lea     ecx, \[esp+0Ch\]  
SDPI:0047F647                 mov     eax, \[esp+0F8h+var\_8C\]  
SDPI:0047F64B                 push    eax                       ; BufferSize  
SDPI:0047F64C                 push    ebx                       ; CalcBuffer  
SDPI:0047F64D                 push    ecx                       ; RetBuffer  
SDPI:0047F64E                 call    Hash\_MD5  
SDPI:0047F653                 mov     eax, \[esp+104h+var\_8C\]  
SDPI:0047F657                 add     esp, 0Ch  
SDPI:0047F65A                 cmp     eax, 80000h  
SDPI:0047F65F                 jz      short loc\_47F630  
SDPI:0047F661                 mov     esi, \[esp+68h\]  
SDPI:0047F665                 lea     edx, \[esp+0Ch\]  
SDPI:0047F669                 push    edx                       ; InBuffer  
SDPI:0047F66A                 push    esi                       ; RetBuffer  
SDPI:0047F66B                 call    Calculate\_MD5  
SDPI:0047F670                 add     esp, 8  
SDPI:0047F673                 push    ebx  
SDPI:0047F674                 push    0  
SDPI:0047F676                 call    GetProcessHeap  
SDPI:0047F67B                 push    eax  
SDPI:0047F67C                 call    FreeHeap  
SDPI:0047F681                 pop     ebx  
SDPI:0047F682                 mov     eax, esi  
SDPI:0047F684                 pop     edi  
SDPI:0047F685                 pop     esi  
SDPI:0047F686                 add     esp, 58h  
SDPI:0047F689                 retn    8  
SDPI:0047F689 GetFileMD5Val\_47F542 endp ; sp = -0A0h  
SDPI:0047F689  
SDPI:0047F689 ; ---------------------------------------------------------------  
  
褀   
  
SDPI:0047F766 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047F766  
SDPI:0047F766  
SDPI:0047F766 Get\_PEHEADER    proc near                         ; CODE XREF: sub\_47F6AA+69 p  
SDPI:0047F766                                                   ; CRC\_480467+15 p ...  
SDPI:0047F766  
SDPI:0047F766 arg\_0           = dword ptr  4  
SDPI:0047F766  
SDPI:0047F766                 mov     ecx, \[esp+arg\_0\]  
SDPI:0047F76A                 cmp     word ptr \[ecx\], 5A4Dh  
SDPI:0047F76F                 jnz     short loc\_47F78A  
SDPI:0047F771                 mov     eax, \[ecx+3Ch\]  
SDPI:0047F774                 add     eax, ecx  
SDPI:0047F776                 jz      short loc\_47F78A  
SDPI:0047F778                 add     ecx, 0FF08h  
SDPI:0047F77E                 cmp     eax, ecx  
SDPI:0047F780                 ja      short loc\_47F78A  
SDPI:0047F782                 cmp     dword ptr \[eax\], 4550h  
SDPI:0047F788                 jz      short locret\_47F78C  
SDPI:0047F78A  
SDPI:0047F78A loc\_47F78A:                                       ; CODE XREF: Get\_PEHEADER+9 j  
SDPI:0047F78A                                                   ; Get\_PEHEADER+10 j ...  
SDPI:0047F78A                 xor     eax, eax  
SDPI:0047F78C  
SDPI:0047F78C locret\_47F78C:                                    ; CODE XREF: Get\_PEHEADER+22 j  
SDPI:0047F78C                 retn    4  
SDPI:0047F78C Get\_PEHEADER    endp  
SDPI:0047F78C  
SDPI:0047F78C ; ---------------------------------------------------------------------  
  
  
  
SDPI:0047F89D ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047F89D  
SDPI:0047F89D  
SDPI:0047F89D ; int \_\_cdecl INIT\_MD5(int initBuffer)  
SDPI:0047F89D INIT\_MD5        proc near                         ; CODE XREF: GetFileMD5Val\_47F542+61 p  
SDPI:0047F89D                                                   ; CRC\_480467+33 p ...  
SDPI:0047F89D  
SDPI:0047F89D initBuffer      = dword ptr  4  
SDPI:0047F89D  
SDPI:0047F89D                 mov     eax, \[esp+initBuffer\]  
SDPI:0047F8A1                 xor     ecx, ecx  
SDPI:0047F8A3                 mov     \[eax+14h\], ecx  
SDPI:0047F8A6                 mov     \[eax+10h\], ecx  
SDPI:0047F8A9                 mov     dword ptr \[eax\], 67452301h  
SDPI:0047F8AF                 mov     dword ptr \[eax+4\], 0EFCDAB89h  
SDPI:0047F8B6                 mov     dword ptr \[eax+8\], 98BADCFEh  
SDPI:0047F8BD                 mov     dword ptr \[eax+0Ch\], 10325476h  
SDPI:0047F8C4                 retn  
SDPI:0047F8C4 INIT\_MD5        endp  
  
\]D  
SDPI:0047F8C5 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047F8C5  
SDPI:0047F8C5  
SDPI:0047F8C5 ; int \_\_cdecl Hash\_MD5(int RetBuffer,int CalcBuffer,int BufferSize)  
SDPI:0047F8C5 Hash\_MD5        proc near                         ; CODE XREF: GetFileMD5Val\_47F542+C2 p  
SDPI:0047F8C5                                                   ; GetFileMD5Val\_47F542+10C p ...  
SDPI:0047F8C5  
SDPI:0047F8C5 RetBuffer       = dword ptr  10h  
SDPI:0047F8C5 CalcBuffer      = dword ptr  14h  
SDPI:0047F8C5 BufferSize      = dword ptr  18h  
SDPI:0047F8C5  
SDPI:0047F8C5                 push    ebx  
SDPI:0047F8C6                 push    ebp  
SDPI:0047F8C7                 push    esi  
SDPI:0047F8C8                 mov     esi, \[esp+RetBuffer\]  
SDPI:0047F8CC                 push    edi  
SDPI:0047F8CD                 mov     edi, \[esp+4+BufferSize\]  
SDPI:0047F8D1                 mov     ecx, \[esi+10h\]  
SDPI:0047F8D4                 mov     eax, ecx  
SDPI:0047F8D6                 lea     edx, ds:0\[edi\*8\]  
SDPI:0047F8DD                 shr     eax, 3  
SDPI:0047F8E0                 lea     ecx, \[ecx+edi\*8\]  
SDPI:0047F8E3                 and     eax, 3Fh  
SDPI:0047F8E6                 cmp     ecx, edx  
SDPI:0047F8E8                 mov     \[esi+10h\], ecx  
SDPI:0047F8EB                 jnb     short loc\_47F8F0  
SDPI:0047F8ED                 inc     dword ptr \[esi+14h\]  
SDPI:0047F8F0  
SDPI:0047F8F0 loc\_47F8F0:                                       ; CODE XREF: Hash\_MD5+26 j  
SDPI:0047F8F0                 mov     ebx, \[esi+14h\]  
SDPI:0047F8F3                 mov     ecx, edi  
SDPI:0047F8F5                 shr     ecx, 1Dh  
SDPI:0047F8F8                 add     ebx, ecx  
SDPI:0047F8FA                 mov     \[esi+14h\], ebx  
SDPI:0047F8FD                 mov     ebx, 40h  
SDPI:0047F902                 sub     ebx, eax  
SDPI:0047F904                 cmp     edi, ebx  
SDPI:0047F906                 jb      short loc\_47F94E  
SDPI:0047F908                 mov     edx, \[esp+4+CalcBuffer\]  
SDPI:0047F90C                 push    ebx  
SDPI:0047F90D                 lea     eax, \[esi+eax+18h\]  
SDPI:0047F911                 push    edx  
SDPI:0047F912                 push    eax  
SDPI:0047F913                 call    sub\_4800D4  
SDPI:0047F918                 lea     ecx, \[esi+18h\]  
SDPI:0047F91B                 push    ecx  
SDPI:0047F91C                 push    esi  
SDPI:0047F91D                 call    MD5\_Function              ; MD5 算法过程  
SDPI:0047F922                 mov     ebp, ebx  
SDPI:0047F924                 add     ebx, 3Fh  
SDPI:0047F927                 add     esp, 14h  
SDPI:0047F92A                 cmp     ebx, edi  
SDPI:0047F92C                 jnb     short loc\_47F94A  
SDPI:0047F92E  
SDPI:0047F92E loc\_47F92E:                                       ; CODE XREF: Hash\_MD5+83 j  
SDPI:0047F92E                 mov     edx, \[esp+4+CalcBuffer\]  
SDPI:0047F932                 lea     eax, \[ebx+edx-3Fh\]  
SDPI:0047F936                 push    eax  
SDPI:0047F937                 push    esi  
SDPI:0047F938                 call    MD5\_Function              ; MD5 算法过程  
SDPI:0047F93D                 add     ebx, 40h  
SDPI:0047F940                 add     esp, 8  
SDPI:0047F943                 add     ebp, 40h  
SDPI:0047F946                 cmp     ebx, edi  
SDPI:0047F948                 jb      short loc\_47F92E  
SDPI:0047F94A  
SDPI:0047F94A loc\_47F94A:                                       ; CODE XREF: Hash\_MD5+67 j  
SDPI:0047F94A                 xor     eax, eax  
SDPI:0047F94C                 jmp     short loc\_47F950  
SDPI:0047F94E ; ----------------------------------------------------------------------------  
SDPI:0047F94E  
SDPI:0047F94E loc\_47F94E:                                       ; CODE XREF: Hash\_MD5+41 j  
SDPI:0047F94E                 xor     ebp, ebp  
SDPI:0047F950  
SDPI:0047F950 loc\_47F950:                                       ; CODE XREF: Hash\_MD5+87 j  
SDPI:0047F950                 mov     ecx, \[esp+4+CalcBuffer\]  
SDPI:0047F954                 sub     edi, ebp  
SDPI:0047F956                 add     ebp, ecx  
SDPI:0047F958                 push    edi  
SDPI:0047F959                 lea     edx, \[esi+eax+18h\]  
SDPI:0047F95D                 push    ebp  
SDPI:0047F95E                 push    edx  
SDPI:0047F95F                 call    sub\_4800D4  
SDPI:0047F964                 add     esp, 0Ch  
SDPI:0047F967                 pop     edi  
SDPI:0047F968                 pop     esi  
SDPI:0047F969                 pop     ebp  
SDPI:0047F96A                 pop     ebx  
SDPI:0047F96B                 retn  
SDPI:0047F96B Hash\_MD5        endp  
  
\]D  
SDPI:0047F96C ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0047F96C  
SDPI:0047F96C  
SDPI:0047F96C ; int \_\_cdecl Calculate\_MD5(int RetBuffer,int InBuffer)  
SDPI:0047F96C Calculate\_MD5   proc near                         ; CODE XREF: GetFileMD5Val\_47F542+129 p  
SDPI:0047F96C                                                   ; CRC\_480467+6A p ...  
SDPI:0047F96C  
SDPI:0047F96C CalcBuffer      = dword ptr -8  
SDPI:0047F96C RetBuffer       = dword ptr  4  
SDPI:0047F96C InBuffer        = dword ptr  8  
SDPI:0047F96C  
SDPI:0047F96C                 sub     esp, 8  
SDPI:0047F96F                 lea     eax, \[esp+8+CalcBuffer\]  
SDPI:0047F972                 push    esi  
SDPI:0047F973                 push    edi  
SDPI:0047F974                 mov     edi, \[esp+10h+InBuffer\]  
SDPI:0047F978                 push    8  
SDPI:0047F97A                 lea     esi, \[edi+10h\]  
SDPI:0047F97D                 push    esi  
SDPI:0047F97E                 push    eax  
SDPI:0047F97F                 call    sub\_48008B  
SDPI:0047F984                 mov     eax, \[esi\]  
SDPI:0047F986                 add     esp, 0Ch  
SDPI:0047F989                 shr     eax, 3  
SDPI:0047F98C                 and     eax, 3Fh  
SDPI:0047F98F                 mov     ecx, 38h  
SDPI:0047F994                 cmp     eax, 38h  
SDPI:0047F997                 jb      short loc\_47F99E  
SDPI:0047F999                 mov     ecx, 78h  
SDPI:0047F99E  
SDPI:0047F99E loc\_47F99E:                                       ; CODE XREF: Calculate\_MD5+2B j  
SDPI:0047F99E                 sub     ecx, eax  
SDPI:0047F9A0                 push    ecx                       ; BufferSize  
SDPI:0047F9A1                 push    4064E4h                   ; relativeAddress  
SDPI:0047F9A6                 call    GetAbsAddress             ; 获取实际地址  
SDPI:0047F9AB                 push    eax                       ; CalcBuffer  
SDPI:0047F9AC                 push    edi                       ; RetBuffer  
SDPI:0047F9AD                 call    Hash\_MD5  
SDPI:0047F9B2                 lea     ecx, \[esp+1Ch+CalcBuffer\]  
SDPI:0047F9B6                 push    8                         ; BufferSize  
SDPI:0047F9B8                 push    ecx                       ; CalcBuffer  
SDPI:0047F9B9                 push    edi                       ; RetBuffer  
SDPI:0047F9BA                 call    Hash\_MD5  
SDPI:0047F9BF                 mov     edx, \[esp+28h+RetBuffer\]  
SDPI:0047F9C3                 push    10h  
SDPI:0047F9C5                 push    edi  
SDPI:0047F9C6                 push    edx  
SDPI:0047F9C7                 call    sub\_48008B  
SDPI:0047F9CC                 push    58h  
SDPI:0047F9CE                 push    0  
SDPI:0047F9D0                 push    edi  
SDPI:0047F9D1                 call    sub\_4800F2  
SDPI:0047F9D6                 add     esp, 30h  
SDPI:0047F9D9                 pop     edi  
SDPI:0047F9DA                 pop     esi  
SDPI:0047F9DB                 add     esp, 8  
SDPI:0047F9DE                 retn  
SDPI:0047F9DE Calculate\_MD5   endp  
SDPI:0047F9DE  
SDPI:0047F9DF  
  
  
  
SDPI:00480121 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:00480121  
SDPI:00480121 ; 复制内存  
SDPI:00480121  
SDPI:00480121 ; int \_\_stdcall RtlMoveMemory(DWORD to,DWORD from,int size)  
SDPI:00480121 RtlMoveMemory   proc near                         ; CODE XREF: Stolen\_CODE+106 p  
SDPI:00480121                                                   ; Stolen\_CODE+132 p ...  
SDPI:00480121  
SDPI:00480121 to              = dword ptr  8  
SDPI:00480121 from            = dword ptr  0Ch  
SDPI:00480121 size            = dword ptr  10h  
SDPI:00480121  
SDPI:00480121                 push    esi  
SDPI:00480122                 mov     esi, \[esp+size\]  
SDPI:00480126                 test    esi, esi  
SDPI:00480128                 jbe     short loc\_480147  
SDPI:0048012A                 mov     edx, \[esp+from\]  
SDPI:0048012E                 push    edi  
SDPI:0048012F                 mov     edi, \[esp+4+to\]  
SDPI:00480133                 mov     ecx, edi  
SDPI:00480135                 sub     edx, edi  
SDPI:00480137  
SDPI:00480137 loc\_480137:                                       ; CODE XREF: RtlMoveMemory+1D j  
SDPI:00480137                 mov     al, \[ecx+edx\]  
SDPI:0048013A                 mov     \[ecx\], al  
SDPI:0048013C                 inc     ecx  
SDPI:0048013D                 dec     esi  
SDPI:0048013E                 jnz     short loc\_480137  
SDPI:00480140                 mov     eax, edi  
SDPI:00480142                 pop     edi  
SDPI:00480143                 pop     esi  
SDPI:00480144                 retn    0Ch  
SDPI:00480147 ; ---------------------------------------------------------------------  
  
  
S  
SDPI:0048014F  
SDPI:0048014F ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0048014F  
SDPI:0048014F ; 清除数据  
SDPI:0048014F  
SDPI:0048014F ; int \_\_stdcall Clear\_Data(LPCSTR,char,int)  
SDPI:0048014F Clear\_Data      proc near                         ; CODE XREF: Stolen\_CODE+11A p  
SDPI:0048014F                                                   ; Stolen\_CODE+1C4 p ...  
SDPI:0048014F  
SDPI:0048014F arg\_0           = dword ptr  4  
SDPI:0048014F arg\_4           = byte ptr  8  
SDPI:0048014F arg\_8           = dword ptr  0Ch  
SDPI:0048014F  
SDPI:0048014F                 mov     ecx, \[esp+arg\_8\]  
SDPI:00480153                 test    ecx, ecx  
SDPI:00480155                 jbe     short loc\_480186  
SDPI:00480157                 mov     al, \[esp+arg\_4\]  
SDPI:0048015B                 push    ebx  
SDPI:0048015C                 mov     bl, al  
SDPI:0048015E                 push    esi  
SDPI:0048015F                 mov     esi, \[esp+8+arg\_0\]  
SDPI:00480163                 mov     bh, bl  
SDPI:00480165                 mov     eax, ebx  
SDPI:00480167                 mov     edx, ecx  
SDPI:00480169                 shl     eax, 10h  
SDPI:0048016C                 push    edi  
SDPI:0048016D                 mov     edi, esi  
SDPI:0048016F                 mov     ax, bx  
SDPI:00480172                 shr     ecx, 2  
SDPI:00480175                 rep stosd  
SDPI:00480177                 mov     ecx, edx  
SDPI:00480179                 and     ecx, 3  
SDPI:0048017C                 rep stosb  
SDPI:0048017E                 mov     eax, esi  
SDPI:00480180                 pop     edi  
SDPI:00480181                 pop     esi  
SDPI:00480182                 pop     ebx  
SDPI:00480183                 retn    0Ch  
SDPI:00480186 ; -------------------------------------------------------------------  
  
 D  
SDPI:004801C9 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:004801C9  
SDPI:004801C9 ; 其中 CurrentAPI 指当然要操作的 API  
SDPI:004801C9 ; SpecialAPI 指壳指的特殊 API  
SDPI:004801C9  
SDPI:004801C9 ; int \_\_stdcall StrCompare(LPCSTR CurrentAPI,LPCSTR specialAPI)  
SDPI:004801C9 StrCompare      proc near                         ; CODE XREF: API\_GetProcAddr+5C p  
SDPI:004801C9                                                   ; Disposal\_IMP+407 p ...  
SDPI:004801C9  
SDPI:004801C9 CurrentAPI      = dword ptr  4  
SDPI:004801C9 specialAPI      = dword ptr  8  
SDPI:004801C9  
SDPI:004801C9                 mov     ecx, \[esp+CurrentAPI\]  
SDPI:004801CD                 mov     edx, \[esp+specialAPI\]  
SDPI:004801D1                 push    ebx  
SDPI:004801D2                 mov     al, \[ecx\]  
SDPI:004801D4                 mov     bl, \[edx\]  
SDPI:004801D6                 cmp     al, bl  
SDPI:004801D8                 jnz     short loc\_4801EA  
SDPI:004801DA  
SDPI:004801DA loc\_4801DA:                                       ; CODE XREF: StrCompare+1F j  
SDPI:004801DA                 test    al, al  
SDPI:004801DC                 jz      short loc\_4801EA  
SDPI:004801DE                 mov     al, \[ecx+1\]  
SDPI:004801E1                 mov     bl, \[edx+1\]  
SDPI:004801E4                 inc     ecx  
SDPI:004801E5                 inc     edx  
SDPI:004801E6                 cmp     al, bl  
SDPI:004801E8                 jz      short loc\_4801DA  
SDPI:004801EA  
SDPI:004801EA loc\_4801EA:                                       ; CODE XREF: StrCompare+F j  
SDPI:004801EA                                                   ; StrCompare+13 j  
SDPI:004801EA                 movsx   eax, byte ptr \[ecx\]  
SDPI:004801ED                 movsx   ecx, byte ptr \[edx\]  
SDPI:004801F0                 sub     eax, ecx  
SDPI:004801F2                 pop     ebx  
SDPI:004801F3                 retn    8  
SDPI:004801F3 StrCompare      endp  
SDPI:004801F3  
SDPI:004801F6 ; --------------------------------------------------------------------  
  
?9 E  
SDPI:00480240 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:00480240  
SDPI:00480240 ; 把 Calculate\_value 改为 Original\_value  
SDPI:00480240 ; 就通过检测了  
SDPI:00480240  
SDPI:00480240 ; int \_\_stdcall Compare\_HASH(int Calculate\_value,int Original\_value,int size)  
SDPI:00480240 Compare\_HASH    proc near                         ; CODE XREF: CRC\_480467+7D p  
SDPI:00480240                                                   ; Stolen\_CODE+F5 p ...  
SDPI:00480240  
SDPI:00480240 Calculate\_value = dword ptr  4  
SDPI:00480240 Original\_value  = dword ptr  8  
SDPI:00480240 size            = dword ptr  0Ch  
SDPI:00480240  
SDPI:00480240                 mov     edx, \[esp+size\]  
SDPI:00480244                 mov     ecx, \[esp+Calculate\_value\]  
SDPI:00480248                 xor     eax, eax  
SDPI:0048024A                 push    esi  
SDPI:0048024B                 test    edx, edx  
SDPI:0048024D                 push    edi  
SDPI:0048024E                 jbe     short loc\_480267  
SDPI:00480250                 mov     esi, \[esp+8+Original\_value\]  
SDPI:00480254                 sub     esi, ecx  
SDPI:00480256  
SDPI:00480256 loc\_480256:                                       ; CODE XREF: Compare\_HASH+25 j  
SDPI:00480256                 test    eax, eax  
SDPI:00480258                 jnz     short loc\_480267  
SDPI:0048025A                 movsx   edi, byte ptr \[ecx+esi\]  
SDPI:0048025E                 movsx   eax, byte ptr \[ecx\]  
SDPI:00480261                 sub     eax, edi  
SDPI:00480263                 inc     ecx  
SDPI:00480264                 dec     edx  
SDPI:00480265                 jnz     short loc\_480256  
SDPI:00480267  
SDPI:00480267 loc\_480267:                                       ; CODE XREF: Compare\_HASH+E j  
SDPI:00480267                                                   ; Compare\_HASH+18 j  
SDPI:00480267                 pop     edi  
SDPI:00480268                 pop     esi  
SDPI:00480269                 retn    0Ch  
SDPI:00480269 Compare\_HASH    endp  
  
  
SDPI:00480757 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:00480757  
SDPI:00480757  
SDPI:00480757 ; int \_\_stdcall API\_GetProcAddr(HMODULE hdll,LPCSTR APIName)  
SDPI:00480757 API\_GetProcAddr proc near                         ; CODE XREF: GetStart\_Info+CE p  
SDPI:00480757                                                   ; Disposal\_IMP+3E p ...  
SDPI:00480757  
SDPI:00480757 hdll            = dword ptr  0Ch  
SDPI:00480757 APIName         = dword ptr  10h  
SDPI:00480757  
SDPI:00480757                 push    ebx  
SDPI:00480758                 push    ebp                       ; 获取 api 地址  
SDPI:00480758                                                   ; 和 api 函数 GetProcAddress 一样的操作  
SDPI:00480758                                                   ; invoke GetProcAddress,hDll,addr SzAPINAME  
SDPI:00480759                 mov     ebp, \[esp+APIName\]  
SDPI:0048075D                 push    esi  
SDPI:0048075E                 push    edi  
SDPI:0048075F                 mov     edi, \[esp+8+hdll\]  
SDPI:00480763                 test    edi, edi  
SDPI:00480765                 jz      loc\_4807FA  
SDPI:0048076B                 cmp     word ptr \[edi\], 5A4Dh  
SDPI:00480770                 jnz     loc\_4807FA  
SDPI:00480776                 mov     eax, \[edi+3Ch\]  
SDPI:00480779                 add     eax, edi  
SDPI:0048077B                 cmp     dword ptr \[eax\], 4550h  
SDPI:00480781                 jnz     short loc\_4807FA  
SDPI:00480783                 mov     ecx, \[eax+7Ch\]  
SDPI:00480786                 test    ecx, ecx  
SDPI:00480788                 jz      short loc\_4807FA  
SDPI:0048078A                 mov     esi, \[eax+78h\]  
SDPI:0048078D                 add     esi, edi  
SDPI:0048078F                 add     ecx, esi  
SDPI:00480791                 test    ebp, 0FFFF0000h  
SDPI:00480797                 mov     \[esp+8+hdll\], ecx  
SDPI:0048079B                 jz      short loc\_4807D7  
SDPI:0048079D                 mov     eax, \[esi+18h\]  
SDPI:004807A0                 xor     ebx, ebx  
SDPI:004807A2                 test    eax, eax  
SDPI:004807A4                 jbe     short loc\_4807C4  
SDPI:004807A6  
SDPI:004807A6 loc\_4807A6:                                       ; CODE XREF: API\_GetProcAddr+6B j  
SDPI:004807A6                 mov     eax, \[esi+20h\]  
SDPI:004807A9                 push    ebp                       ; specialAPI  
SDPI:004807AA                 lea     ecx, \[eax+ebx\*4\]  
SDPI:004807AD                 mov     edx, \[edi+ecx\]  
SDPI:004807B0                 add     edx, edi  
SDPI:004807B2                 push    edx                       ; CurrentAPI  
SDPI:004807B3                 call    StrCompare                ; 其中 CurrentAPI 指当然要操作的 API  
SDPI:004807B3                                                   ; SpecialAPI 指壳指的特殊 API  
SDPI:004807B8                 test    eax, eax  
SDPI:004807BA                 jz      short loc\_4807C4  
SDPI:004807BC                 mov     eax, \[esi+18h\]  
SDPI:004807BF                 inc     ebx  
SDPI:004807C0                 cmp     ebx, eax  
SDPI:004807C2                 jb      short loc\_4807A6  
SDPI:004807C4  
SDPI:004807C4 loc\_4807C4:                                       ; CODE XREF: API\_GetProcAddr+4D j  
SDPI:004807C4                                                   ; API\_GetProcAddr+63 j  
SDPI:004807C4                 cmp     \[esi+18h\], ebx  
SDPI:004807C7                 jbe     short loc\_4807FA  
SDPI:004807C9                 mov     eax, \[esi+24h\]  
SDPI:004807CC                 lea     ecx, \[eax+ebx\*2\]  
SDPI:004807CF                 xor     eax, eax  
SDPI:004807D1                 mov     ax, \[edi+ecx\]  
SDPI:004807D5                 jmp     short loc\_4807DE  
SDPI:004807D7 ; ----------------------------------------------------------------------------  
SDPI:004807D7  
SDPI:004807D7 loc\_4807D7:                                       ; CODE XREF: API\_GetProcAddr+44 j  
SDPI:004807D7                 mov     ecx, \[esi+10h\]  
SDPI:004807DA                 mov     eax, ebp  
SDPI:004807DC                 sub     eax, ecx  
SDPI:004807DE  
SDPI:004807DE loc\_4807DE:                                       ; CODE XREF: API\_GetProcAddr+7E j  
SDPI:004807DE                 cmp     \[esi+14h\], eax  
SDPI:004807E1                 jbe     short loc\_4807FA  
SDPI:004807E3                 mov     edx, \[esi+1Ch\]  
SDPI:004807E6                 lea     eax, \[edx+eax\*4\]  
SDPI:004807E9                 mov     eax, \[edi+eax\]  
SDPI:004807EC                 add     eax, edi  
SDPI:004807EE                 jz      short loc\_4807FA  
SDPI:004807F0                 cmp     eax, esi  
SDPI:004807F2                 jb      short loc\_480801  
SDPI:004807F4                 cmp     eax, \[esp+8+hdll\]  
SDPI:004807F8                 ja      short loc\_480801  
SDPI:004807FA  
SDPI:004807FA loc\_4807FA:                                       ; CODE XREF: API\_GetProcAddr+E j  
SDPI:004807FA                                                   ; API\_GetProcAddr+19 j ...  
SDPI:004807FA                 push    ebp  
SDPI:004807FB                 push    edi  
SDPI:004807FC                 call    sub\_48359A  
SDPI:00480801  
SDPI:00480801 loc\_480801:                                       ; CODE XREF: API\_GetProcAddr+9B j  
SDPI:00480801                                                   ; API\_GetProcAddr+A1 j  
SDPI:00480801                 pop     edi  
SDPI:00480802                 pop     esi  
SDPI:00480803                 pop     ebp  
SDPI:00480804                 pop     ebx  
SDPI:00480805                 retn    8  
SDPI:00480805 API\_GetProcAddr endp ; sp = -8  
SDPI:00480805  
SDPI:00480805 ; ---------------------------------------------------------------------  
  
  
  
SDPI:00480825 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:00480825  
SDPI:00480825 ; 申请空间，并使空间有可写读和运行权限  
SDPI:00480825  
SDPI:00480825 ; int \_\_stdcall Alloc\_Sp\_480825(int)  
SDPI:00480825 Alloc\_Sp\_480825 proc near                         ; CODE XREF: SDPI:0047A418 j  
SDPI:00480825                                                   ; sub\_4808D3+38 p  
SDPI:00480825  
SDPI:00480825 flOldProtect    = dword ptr -4  
SDPI:00480825 push1           = dword ptr  10h  
SDPI:00480825  
SDPI:00480825                 push    ecx  
SDPI:00480826                 push    esi  
SDPI:00480827                 push    edi  
SDPI:00480828                 call    Get\_BASE  
SDPI:0048082D                 mov     edi, eax  
SDPI:0048082F                 mov     eax, \[esp+push1\]  
SDPI:00480833                 test    eax, eax  
SDPI:00480835                 jz      short loc\_480859  
SDPI:00480837                 push    0Ch  
SDPI:00480839                 push    8  
SDPI:0048083B                 call    GetProcessHeap  
SDPI:00480840                 push    eax  
SDPI:00480841                 call    allocateHeap              ; 申请空间  
SDPI:00480846                 mov     esi, eax  
SDPI:00480848                 test    esi, esi                  ; 判断申请空间是否成功  
SDPI:0048084A                 jnz     short loc\_480851  
SDPI:0048084C                 call    failed\_480730  
SDPI:00480851  
SDPI:00480851 loc\_480851:                                       ; CODE XREF: Alloc\_Sp\_480825+25 j  
SDPI:00480851                 mov     \[edi+0C8h\], esi  
SDPI:00480857                 jmp     short loc\_48085F  
SDPI:00480859 ; ----------------------------------------------------------------------------  
SDPI:00480859  
SDPI:00480859 loc\_480859:                                       ; CODE XREF: Alloc\_Sp\_480825+10 j  
SDPI:00480859                 mov     esi, \[edi+0C8h\]  
SDPI:0048085F  
SDPI:0048085F loc\_48085F:                                       ; CODE XREF: Alloc\_Sp\_480825+32 j  
SDPI:0048085F                 push    4000h  
SDPI:00480864                 push    8  
SDPI:00480866                 call    GetProcessHeap  
SDPI:0048086B                 push    eax  
SDPI:0048086C                 call    allocateHeap  
SDPI:00480871                 mov     edi, eax  
SDPI:00480873                 test    edi, edi                  ; 再次申请空间  
SDPI:00480875                 jnz     short loc\_48087C  
SDPI:00480877                 call    failed\_480730  
SDPI:0048087C  
SDPI:0048087C loc\_48087C:                                       ; CODE XREF: Alloc\_Sp\_480825+50 j  
SDPI:0048087C                 lea     eax, \[esp+0Ch+flOldProtect\]  
SDPI:00480880                 mov     \[esi\], edi  
SDPI:00480882                 push    eax                       ; /pOldProtect = 0012FFB8  
SDPI:00480883                 push    40h                       ; |NewProtect = PAGE\_EXECUTE\_READWRITE  
SDPI:00480885                 push    4000h                     ; |Size = 4000 (16384.)  
SDPI:0048088A                 push    edi                       ; |Address = 00143A58  
SDPI:0048088B                 mov     dword ptr \[esi+4\], 0      ; |  
SDPI:00480892                 mov     dword ptr \[esi+8\], 4000h  ; |使前面申请的空间有写读和运行权限  
SDPI:00480899                 call    VirtualProtect            ; \\CALL to VirtualProtect  
SDPI:0048089E                 pop     edi  
SDPI:0048089F                 pop     esi  
SDPI:004808A0                 test    eax, eax  
SDPI:004808A2                 jnz     short loc\_4808CA  
SDPI:004808A4                 push    10h                       ; uType  
SDPI:004808A6                 push    offset strError           ; relativeAddress  
SDPI:004808AB                 call    GetAbsAddress             ; 获取实际地址  
SDPI:004808B0                 push    eax                       ; lpCaption  
SDPI:004808B1                 push    offset unk\_407808         ; relativeAddress  
SDPI:004808B6                 call    GetAbsAddress             ; 获取实际地址  
SDPI:004808BB                 push    eax                       ; lpText  
SDPI:004808BC                 push    0                         ; hWnd  
SDPI:004808BE                 call    MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:004808BE                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:004808BE                                                   ; 也就是判断有没有下 int3 断点  
SDPI:004808C3                 push    0FFFFFFFFh                ; uExitCode  
SDPI:004808C5                 call    ExitProcess  
SDPI:004808CA  
SDPI:004808CA loc\_4808CA:                                       ; CODE XREF: Alloc\_Sp\_480825+7D j  
SDPI:004808CA                 mov     eax, 1  
SDPI:004808CF                 pop     ecx  
SDPI:004808D0                 retn    4  
SDPI:004808D0 Alloc\_Sp\_480825 endp ; sp = -0Ch  
  
  
SDPI:00480A8B ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:00480A8B  
SDPI:00480A8B ; 获取程序启动的相关信息：  
SDPI:00480A8B ; 如程序句柄，系统目录  
SDPI:00480A8B ; windows 目录，  
SDPI:00480A8B ; 程序完整程序等  
SDPI:00480A8B  
SDPI:00480A8B GetStart\_Info   proc near                         ; CODE XREF: SDPI:0047B442 j  
SDPI:00480A8B                 push    esi  
SDPI:00480A8C                 push    edi  
SDPI:00480A8D                 push    62Ch  
SDPI:00480A92                 push    8  
SDPI:00480A94                 call    GetProcessHeap  
SDPI:00480A99                 push    eax  
SDPI:00480A9A                 call    allocateHeap  
SDPI:00480A9F                 mov     edi, eax  
SDPI:00480AA1                 call    Get\_BASE  
SDPI:00480AA6                 mov     esi, eax  
SDPI:00480AA8                 mov     \[esi+0CCh\], edi  
SDPI:00480AAE                 call    apiGetCmdLine  
SDPI:00480AB3                 push    eax  
SDPI:00480AB4                 mov     eax, \[esi+0CCh\]  
SDPI:00480ABA                 add     eax, 4F4h  
SDPI:00480ABF                 push    eax  
SDPI:00480AC0                 call    memcpy  
SDPI:00480AC5                 call    GetCurrentProcessId  
SDPI:00480ACA                 mov     ecx, \[esi+0CCh\]  
SDPI:00480AD0                 mov     \[ecx+4\], eax  
SDPI:00480AD3                 call    Get\_Version  
SDPI:00480AD8                 mov     edx, \[esi+0CCh\]  
SDPI:00480ADE                 mov     \[edx\], eax  
SDPI:00480AE0                 mov     eax, \[esi+0CCh\]  
SDPI:00480AE6                 mov     dword ptr \[eax+0Ch\], 94h  
SDPI:00480AED                 mov     ecx, \[esi+0CCh\]  
SDPI:00480AF3                 add     ecx, 0Ch  
SDPI:00480AF6                 push    ecx                       ; lpVersionInformation  
SDPI:00480AF7                 call    GetVersionExA  
SDPI:00480AFC                 mov     edx, \[esi+0CCh\]  
SDPI:00480B02                 add     edx, 0A0h  
SDPI:00480B08                 push    edx                       ; lpStartupInfo  
SDPI:00480B09                 call    GetStartupInfoA  
SDPI:00480B0E                 mov     eax, \[esi+0CCh\]  
SDPI:00480B14                 push    104h                      ; nSize  
SDPI:00480B19                 add     eax, 0E4h  
SDPI:00480B1E                 push    eax                       ; lpFilename  
SDPI:00480B1F                 push    0                         ; hModule  
SDPI:00480B21                 call    GetModuleFileNameA  
SDPI:00480B26                 mov     ecx, \[esi+0CCh\]  
SDPI:00480B2C                 push    104h                      ; uSize  
SDPI:00480B31                 add     ecx, 2ECh  
SDPI:00480B37                 push    ecx                       ; lpBuffer  
SDPI:00480B38                 call    GetSystemDirectoryA  
SDPI:00480B3D                 push    offset strGetWinDir       ; relativeAddress  
SDPI:00480B42                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00480B47                 push    eax  
SDPI:00480B48                 push    offset strkerdll          ; relativeAddress  
SDPI:00480B4D                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00480B52                 push    eax                       ; APIName  
SDPI:00480B53                 call    api\_GetModule  
SDPI:00480B58                 push    eax                       ; hdll  
SDPI:00480B59                 call    API\_GetProcAddr  
SDPI:00480B5E                 mov     edx, \[esi+0CCh\]  
SDPI:00480B64                 push    104h  
SDPI:00480B69                 add     edx, 1E8h                 ; 获取 Windows 目录  
SDPI:00480B6F                 push    edx  
SDPI:00480B70                 call    eax                       ; GetWindowsDirectory  
SDPI:00480B72                 mov     eax, \[esi+0CCh\]  
SDPI:00480B78                 add     eax, 3F0h  
SDPI:00480B7D                 push    eax                       ; lpBuffer  
SDPI:00480B7E                 push    104h                      ; nBufferLength  
SDPI:00480B83                 call    GetCurrentDirectoryA  
SDPI:00480B88                 push    0  
SDPI:00480B8A                 call    api\_GetModule  
SDPI:00480B8F                 mov     ecx, \[esi+0CCh\]  
SDPI:00480B95                 pop     edi  
SDPI:00480B96                 pop     esi  
SDPI:00480B97                 mov     \[ecx+8\], eax  
SDPI:00480B9A                 mov     eax, 1  
SDPI:00480B9F                 retn  
SDPI:00480B9F GetStart\_Info   endp ; sp = -4  
SDPI:00480B9F  
SDPI:00480B9F ; ----------------------------------------------------------------------  
  
  
  
SDPI:00480E25 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:00480E25  
SDPI:00480E25 ; 这里开始抽程序的入口代码  
SDPI:00480E25  
SDPI:00480E25 Stolen\_CODE     proc near                         ; CODE XREF: SDPI:0047E36A j  
SDPI:00480E25  
SDPI:00480E25 to              = dword ptr -3Ch  
SDPI:00480E25 from            = dword ptr -38h  
SDPI:00480E25 Original\_value  = dword ptr -34h  
SDPI:00480E25 var\_30          = byte ptr -30h  
SDPI:00480E25 var\_2C          = byte ptr -2Ch  
SDPI:00480E25 var\_2A          = byte ptr -2Ah  
SDPI:00480E25 var\_29          = byte ptr -29h  
SDPI:00480E25 var\_28          = byte ptr -28h  
SDPI:00480E25 var\_27          = byte ptr -27h  
SDPI:00480E25 var\_26          = byte ptr -26h  
SDPI:00480E25 var\_24          = byte ptr -24h  
SDPI:00480E25 var\_22          = byte ptr -22h  
SDPI:00480E25 var\_21          = byte ptr -21h  
SDPI:00480E25 var\_20          = byte ptr -20h  
SDPI:00480E25 var\_1F          = byte ptr -1Fh  
SDPI:00480E25 var\_1E          = byte ptr -1Eh  
SDPI:00480E25 var\_1C          = byte ptr -1Ch  
SDPI:00480E25 var\_1A          = byte ptr -1Ah  
SDPI:00480E25 var\_19          = byte ptr -19h  
SDPI:00480E25 var\_18          = byte ptr -18h  
SDPI:00480E25 var\_17          = byte ptr -17h  
SDPI:00480E25 var\_16          = byte ptr -16h  
SDPI:00480E25 var\_15          = byte ptr -15h  
SDPI:00480E25 var\_14          = byte ptr -14h  
SDPI:00480E25 var\_13          = byte ptr -13h  
SDPI:00480E25 var\_10          = byte ptr -10h  
SDPI:00480E25 var\_E           = byte ptr -0Eh  
SDPI:00480E25 var\_D           = byte ptr -0Dh  
SDPI:00480E25 var\_C           = byte ptr -0Ch  
SDPI:00480E25 var\_B           = byte ptr -0Bh  
SDPI:00480E25 var\_A           = byte ptr -0Ah  
SDPI:00480E25 var\_9           = byte ptr -9  
SDPI:00480E25 var\_8           = byte ptr -8  
SDPI:00480E25 var\_7           = byte ptr -7  
SDPI:00480E25  
SDPI:00480E25                 sub     esp, 38h  
SDPI:00480E28                 push    ebx  
SDPI:00480E29                 push    esi  
SDPI:00480E2A                 push    edi  
SDPI:00480E2B                 call    Get\_BASE  
SDPI:00480E30                 push    0  
SDPI:00480E32                 mov     ebx, eax  
SDPI:00480E34                 call    api\_GetModule  
SDPI:00480E39                 mov     ecx, \[ebx+18h\]            ; BASE+18H 处保存原程序的 OEP  
SDPI:00480E3C                 mov     edi, eax  
SDPI:00480E3E                 add     edi, ecx  
SDPI:00480E40                 mov     dl, 55h  
SDPI:00480E42                 mov     cl, 8Bh  
SDPI:00480E44                 mov     al, 0ECh  
SDPI:00480E46                 push    4  
SDPI:00480E48                 mov     byte ptr \[esp+4Ch+Original\_value\], dl ; 这里是设计出各类程序的入口代码样式  
SDPI:00480E4C                 mov     byte ptr \[esp+4Ch+Original\_value+1\], cl  
SDPI:00480E50                 mov     byte ptr \[esp+4Ch+Original\_value+2\], al  
SDPI:00480E54                 mov     byte ptr \[esp+4Ch+Original\_value+3\], 6Ah  
SDPI:00480E59                 mov     \[esp+4Ch+var\_30\], 0FFh  
SDPI:00480E5E                 mov     \[esp+4Ch+var\_2C\], dl  
SDPI:00480E62                 mov     \[esp+21h\], cl  
SDPI:00480E66                 mov     \[esp+4Ch+var\_2A\], al  
SDPI:00480E6A                 mov     \[esp+4Ch+var\_29\], 83h  
SDPI:00480E6F                 mov     \[esp+4Ch+var\_28\], 0C4h  
SDPI:00480E74                 mov     \[esp+4Ch+var\_27\], 0F4h  
SDPI:00480E79                 mov     \[esp+4Ch+var\_26\], 0B8h  
SDPI:00480E7E                 mov     \[esp+4Ch+var\_1C\], dl  
SDPI:00480E82                 mov     \[esp+31h\], cl  
SDPI:00480E86                 mov     \[esp+4Ch+var\_1A\], al  
SDPI:00480E8A                 mov     \[esp+4Ch+var\_19\], 83h  
SDPI:00480E8F                 mov     \[esp+4Ch+var\_18\], 0C4h  
SDPI:00480E94                 mov     \[esp+4Ch+var\_17\], 0F4h  
SDPI:00480E99                 mov     \[esp+4Ch+var\_16\], 53h  
SDPI:00480E9E                 mov     \[esp+4Ch+var\_15\], 56h  
SDPI:00480EA3                 mov     \[esp+4Ch+var\_14\], 57h  
SDPI:00480EA8                 mov     \[esp+4Ch+var\_13\], 0B8h  
SDPI:00480EAD                 mov     \[esp+4Ch+var\_24\], dl  
SDPI:00480EB1                 mov     \[esp+29h\], cl  
SDPI:00480EB5                 mov     \[esp+4Ch+var\_22\], al  
SDPI:00480EB9                 mov     \[esp+4Ch+var\_21\], 83h  
SDPI:00480EBE                 mov     \[esp+4Ch+var\_20\], 0C4h  
SDPI:00480EC3                 mov     \[esp+4Ch+var\_1F\], 0F8h  
SDPI:00480EC8                 mov     \[esp+4Ch+var\_1E\], 0B8h  
SDPI:00480ECD                 mov     \[esp+4Ch+var\_10\], dl  
SDPI:00480ED1                 mov     \[esp+3Dh\], cl  
SDPI:00480ED5                 mov     \[esp+4Ch+var\_E\], al  
SDPI:00480ED9                 mov     \[esp+4Ch+var\_D\], 83h  
SDPI:00480EDE                 mov     \[esp+4Ch+var\_C\], 0C4h  
SDPI:00480EE3                 mov     \[esp+4Ch+var\_B\], 0F8h  
SDPI:00480EE8                 mov     \[esp+4Ch+var\_A\], 53h  
SDPI:00480EED                 mov     \[esp+4Ch+var\_9\], 56h  
SDPI:00480EF2                 mov     \[esp+4Ch+var\_8\], 57h  
SDPI:00480EF7                 mov     \[esp+4Ch+var\_7\], 0B8h  
SDPI:00480EFC                 call    sub\_4808D3  
SDPI:00480F01                 push    20h  
SDPI:00480F03                 mov     \[esp+4Ch+to\], eax  
SDPI:00480F07                 call    sub\_4808D3  
SDPI:00480F0C                 mov     esi, eax  
SDPI:00480F0E                 push    5                         ; size  
SDPI:00480F10                 mov     \[esp+4Ch+from\], eax  
SDPI:00480F14                 lea     eax, \[esp+4Ch+Original\_value\]  
SDPI:00480F18                 push    eax                       ; Original\_value  
SDPI:00480F19                 push    edi                       ; Calculate\_value  
SDPI:00480F1A                 call    Compare\_HASH              ; 把 Calculate\_value 改为 Original\_value  
SDPI:00480F1A                                                   ; 就通过检测了  
SDPI:00480F1F                 test    eax, eax                  ; 判断是否为 C 程序入口:  
SDPI:00480F1F                                                   ; push ebp  
SDPI:00480F1F                                                   ; mov ebp,esp  
SDPI:00480F1F                                                   ; push -1  
SDPI:00480F21                 jnz     short NotIsCAPP  
SDPI:00480F23                 mov     byte ptr \[esi\], 58h  
SDPI:00480F26                 inc     esi  
SDPI:00480F27                 push    0Fh                       ; size  
SDPI:00480F29                 push    edi                       ; from  
SDPI:00480F2A                 push    esi                       ; to  
SDPI:00480F2B                 call    RtlMoveMemory             ; 复制内存  
SDPI:00480F30                 add     esi, 0Fh  
SDPI:00480F33                 push    0Fh                       ; int  
SDPI:00480F35                 push    0                         ; char  
SDPI:00480F37                 push    edi                       ; LPCSTR  
SDPI:00480F38                 mov     byte ptr \[esi\], 0FFh  
SDPI:00480F3B                 mov     byte ptr \[esi+1\], 0E0h  
SDPI:00480F3F                 call    Clear\_Data                ; 清除数据  
SDPI:00480F44                 lea     eax, \[edi+9\]  
SDPI:00480F47                 lea     ecx, \[esp+48h+to\]  
SDPI:00480F4B                 push    4                         ; size  
SDPI:00480F4D                 push    ecx                       ; from  
SDPI:00480F4E                 mov     byte ptr \[eax\], 0FFh  
SDPI:00480F51                 inc     eax  
SDPI:00480F52                 mov     byte ptr \[eax\], 15h  
SDPI:00480F55                 inc     eax  
SDPI:00480F56                 push    eax                       ; to  
SDPI:00480F57                 call    RtlMoveMemory             ; 复制内存  
SDPI:00480F5C                 lea     edx, \[esp+48h+from\]  
SDPI:00480F60                 push    4                         ; size  
SDPI:00480F62                 mov     eax, \[esp+4Ch+to\]  
SDPI:00480F66                 push    edx                       ; from  
SDPI:00480F67                 push    eax                       ; to  
SDPI:00480F68                 call    RtlMoveMemory             ; 复制内存  
SDPI:00480F6D                 mov     eax, \[ebx+18h\]  
SDPI:00480F70                 pop     edi  
SDPI:00480F71                 add     eax, 9  
SDPI:00480F74                 pop     esi  
SDPI:00480F75                 mov     \[ebx+18h\], eax  
SDPI:00480F78                 pop     ebx  
SDPI:00480F79                 add     esp, 38h  
SDPI:00480F7C                 retn  
SDPI:00480F7D ; ----------------------------------------------------------------------------  
SDPI:00480F7D  
SDPI:00480F7D NotIsCAPP:                                        ; CODE XREF: Stolen\_CODE+FC j  
SDPI:00480F7D                 lea     ecx, \[esp+48h+var\_2C\]  
SDPI:00480F81                 push    7                         ; size  
SDPI:00480F83                 push    ecx                       ; Original\_value  
SDPI:00480F84                 push    edi                       ; Calculate\_value  
SDPI:00480F85                 call    Compare\_HASH              ; 把 Calculate\_value 改为 Original\_value  
SDPI:00480F85                                                   ; 就通过检测了  
SDPI:00480F8A                 test    eax, eax                  ; 判断是否为 Delphi 的程序  
SDPI:00480F8C                 jz      IsDelphiApp  
SDPI:00480F92                 lea     edx, \[esp+48h+var\_24\]  
SDPI:00480F96                 push    7                         ; size  
SDPI:00480F98                 push    edx                       ; Original\_value  
SDPI:00480F99                 push    edi                       ; Calculate\_value  
SDPI:00480F9A                 call    Compare\_HASH              ; 把 Calculate\_value 改为 Original\_value  
SDPI:00480F9A                                                   ; 就通过检测了  
SDPI:00480F9F                 test    eax, eax  
SDPI:00480FA1                 jz      IsDelphiApp  
SDPI:00480FA7                 lea     eax, \[esp+48h+var\_1C\]  
SDPI:00480FAB                 push    0Ah                       ; size  
SDPI:00480FAD                 push    eax                       ; Original\_value  
SDPI:00480FAE                 push    edi                       ; Calculate\_value  
SDPI:00480FAF                 call    Compare\_HASH              ; 把 Calculate\_value 改为 Original\_value  
SDPI:00480FAF                                                   ; 就通过检测了  
SDPI:00480FB4                 test    eax, eax  
SDPI:00480FB6                 jz      short loc\_480FCD  
SDPI:00480FB8                 lea     ecx, \[esp+48h+var\_10\]  
SDPI:00480FBC                 push    0Ah                       ; size  
SDPI:00480FBE                 push    ecx                       ; Original\_value  
SDPI:00480FBF                 push    edi                       ; Calculate\_value  
SDPI:00480FC0                 call    Compare\_HASH              ; 把 Calculate\_value 改为 Original\_value  
SDPI:00480FC0                                                   ; 就通过检测了  
SDPI:00480FC5                 test    eax, eax  
SDPI:00480FC7                 jnz     DontSteaCode              ; 如果都不符合条件则结束过程  
SDPI:00480FCD  
SDPI:00480FCD loc\_480FCD:                                       ; CODE XREF: Stolen\_CODE+191 j  
SDPI:00480FCD                 mov     byte ptr \[esi\], 59h  
SDPI:00480FD0                 inc     esi  
SDPI:00480FD1                 push    0Eh                       ; size  
SDPI:00480FD3                 push    edi                       ; from  
SDPI:00480FD4                 push    esi                       ; 这些复制代码没有什么特别之处  
SDPI:00480FD5                 call    RtlMoveMemory             ; 复制内存  
SDPI:00480FDA                 add     esi, 0Eh                  ; 也算是作者的又一大败笔吧  
SDPI:00480FDA                                                   ; 抽程序的入口代码，不是加壳时  
SDPI:00480FDA                                                   ; 抽而是在这里再判断的抽。  
SDPI:00480FDD                 push    0Eh                       ; int  
SDPI:00480FDF                 push    0                         ; char  
SDPI:00480FE1                 push    edi                       ; LPCSTR  
SDPI:00480FE2                 mov     byte ptr \[esi\], 0FFh  
SDPI:00480FE5                 mov     byte ptr \[esi+1\], 0E1h  
SDPI:00480FE9                 call    Clear\_Data                ; 清除数据  
SDPI:00480FEE                 lea     eax, \[edi+8\]  
SDPI:00480FF1                 lea     edx, \[esp+48h+to\]  
SDPI:00480FF5                 push    4                         ; size  
SDPI:00480FF7                 push    edx                       ; from  
SDPI:00480FF8                 mov     byte ptr \[eax\], 0FFh  
SDPI:00480FFB                 inc     eax  
SDPI:00480FFC                 mov     byte ptr \[eax\], 15h  
SDPI:00480FFF                 inc     eax  
SDPI:00481000                 push    eax                       ; to  
SDPI:00481001                 call    RtlMoveMemory             ; 复制内存  
SDPI:00481006                 lea     eax, \[esp+48h+from\]  
SDPI:0048100A                 push    4                         ; size  
SDPI:0048100C                 mov     ecx, \[esp+4Ch+to\]  
SDPI:00481010                 push    eax                       ; from  
SDPI:00481011                 push    ecx                       ; to  
SDPI:00481012                 call    RtlMoveMemory             ; 复制内存  
SDPI:00481017                 mov     eax, \[ebx+18h\]  
SDPI:0048101A                 pop     edi  
SDPI:0048101B                 add     eax, 8  
SDPI:0048101E                 pop     esi  
SDPI:0048101F                 mov     \[ebx+18h\], eax  
SDPI:00481022                 pop     ebx  
SDPI:00481023                 add     esp, 38h  
SDPI:00481026                 retn  
SDPI:00481027 ; ----------------------------------------------------------------------------  
SDPI:00481027  
SDPI:00481027 IsDelphiApp:                                      ; CODE XREF: Stolen\_CODE+167 j  
SDPI:00481027                                                   ; Stolen\_CODE+17C j  
SDPI:00481027                 mov     byte ptr \[esi\], 59h  
SDPI:0048102A                 inc     esi  
SDPI:0048102B                 push    0Bh                       ; size  
SDPI:0048102D                 push    edi                       ; from  
SDPI:0048102E                 push    esi                       ; to  
SDPI:0048102F                 call    RtlMoveMemory             ; 复制内存  
SDPI:00481034                 add     esi, 0Bh  
SDPI:00481037                 push    0Bh                       ; int  
SDPI:00481039                 push    0                         ; char  
SDPI:0048103B                 push    edi                       ; LPCSTR  
SDPI:0048103C                 mov     byte ptr \[esi\], 0FFh  
SDPI:0048103F                 mov     byte ptr \[esi+1\], 0E1h  
SDPI:00481043                 call    Clear\_Data                ; 清除数据  
SDPI:00481048                 lea     eax, \[edi+5\]  
SDPI:0048104B                 lea     edx, \[esp+48h+to\]  
SDPI:0048104F                 push    4                         ; size  
SDPI:00481051                 push    edx                       ; from  
SDPI:00481052                 mov     byte ptr \[eax\], 0FFh  
SDPI:00481055                 inc     eax  
SDPI:00481056                 mov     byte ptr \[eax\], 15h  
SDPI:00481059                 inc     eax  
SDPI:0048105A                 push    eax                       ; to  
SDPI:0048105B                 call    RtlMoveMemory             ; 复制内存  
SDPI:00481060                 lea     eax, \[esp+48h+from\]  
SDPI:00481064                 push    4                         ; size  
SDPI:00481066                 mov     ecx, \[esp+4Ch+to\]  
SDPI:0048106A                 push    eax                       ; from  
SDPI:0048106B                 push    ecx                       ; to  
SDPI:0048106C                 call    RtlMoveMemory             ; 复制内存  
SDPI:00481071                 add     dword ptr \[ebx+18h\], 5  
SDPI:00481075  
SDPI:00481075 DontSteaCode:                                     ; CODE XREF: Stolen\_CODE+1A2 j  
SDPI:00481075                 pop     edi  
SDPI:00481076                 pop     esi  
SDPI:00481077                 pop     ebx  
SDPI:00481078                 add     esp, 38h  
SDPI:0048107B                 retn  
SDPI:0048107B Stolen\_CODE     endp ; sp = -4  
  
  
  
SDPI:0048107C ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0048107C  
SDPI:0048107C ; 修改文件头部信息  
SDPI:0048107C  
SDPI:0048107C Erase\_PEHEADER  proc near                         ; CODE XREF: SDPI:0047DD0D j  
SDPI:0048107C  
SDPI:0048107C flNewProtect    = dword ptr -40h  
SDPI:0048107C flOldProtect    = dword ptr -30h  
SDPI:0048107C Oldp            = \_MEMORY\_BASIC\_INFORMATION ptr -20h  
SDPI:0048107C  
SDPI:0048107C                 sub     esp, 20h  
SDPI:0048107F                 push    ebx  
SDPI:00481080                 push    edi  
SDPI:00481081                 push    0  
SDPI:00481083                 call    api\_GetModule  
SDPI:00481088                 mov     ebx, eax  
SDPI:0048108A                 test    ebx, ebx  
SDPI:0048108C                 jz      Done\_4811A7  
SDPI:00481092                 cmp     word ptr \[ebx\], 'ZM'      ; 这里几个判断是多余的  
SDPI:00481092                                                   ; 如果不是 PE 文件，壳还能加上去吗？  
SDPI:00481092                                                   ; 更不可能会运行到这里  
SDPI:00481097                 jnz     Done\_4811A7  
SDPI:0048109D                 mov     edi, \[ebx+3Ch\]  
SDPI:004810A0                 add     edi, ebx  
SDPI:004810A2                 cmp     dword ptr \[edi\], 'EP'     ; 这里还判断是否为 PE 文件  
SDPI:004810A2                                                   ; 简直是乱搞  
SDPI:004810A8                 jnz     Done\_4811A7  
SDPI:004810AE                 push    ebp  
SDPI:004810AF                 push    esi  
SDPI:004810B0                 call    GetTickCount  
SDPI:004810B5                 mov     esi, eax  
SDPI:004810B7                 lea     eax, \[esp+34h+Oldp\]  
SDPI:004810BB                 push    1Ch                       ; dwLength  
SDPI:004810BD                 push    eax                       ; lpBuffer  
SDPI:004810BE                 push    ebx                       ; lpAddress  
SDPI:004810BF                 call    VirtualQuery  
SDPI:004810C4                 lea     ecx, \[esp+10h\]  
SDPI:004810C8                 mov     edx, \[esp+34h+Oldp.Protect\]  
SDPI:004810CC                 push    ecx                       ; lpflOldProtect  
SDPI:004810CD                 and     edx, 0FFFFFFDDh  
SDPI:004810D0                 or      edx, 4  
SDPI:004810D3                 push    edx                       ; flNewProtect  
SDPI:004810D4                 push    200h                      ; dwSize  
SDPI:004810D9                 push    ebx                       ; lpAddress  
SDPI:004810DA                 call    VirtualProtect            ; 使文件头具有可写属性  
SDPI:004810DA                                                   ; 作者设计上又一失误  
SDPI:004810DA                                                   ; 如果操作失败那后面的怎么办?  
SDPI:004810DF                 mov     eax, esi  
SDPI:004810E1                 xor     edx, edx  
SDPI:004810E3                 mov     ecx, 0Ah  
SDPI:004810E8                 mov     ebp, \[edi+50h\]            ; 这里开始壳对 PE 文件头，乱改一通.  
SDPI:004810EB                 div     ecx  
SDPI:004810ED                 mov     eax, \[edi+30h\]  
SDPI:004810F0                 shl     edx, 0Ch  
SDPI:004810F3                 add     eax, edx  
SDPI:004810F5                 add     ebp, edx  
SDPI:004810F7                 mov     \[edi+30h\], eax  
SDPI:004810FA                 xor     eax, eax  
SDPI:004810FC                 mov     ax, \[edi+14h\]  
SDPI:00481100                 mov     \[edi+50h\], ebp  
SDPI:00481103                 mov     ecx, \[edi+eax+2Ch\]  
SDPI:00481107                 lea     eax, \[edi+eax+18h\]  
SDPI:0048110B                 add     ecx, edx  
SDPI:0048110D                 mov     edi, 1000h  
SDPI:00481112                 mov     ebp, \[eax+8\]  
SDPI:00481115                 mov     \[eax+14h\], ecx  
SDPI:00481118                 mov     ecx, esi  
SDPI:0048111A                 and     esi, 1  
SDPI:0048111D                 shl     esi, 0Ch  
SDPI:00481120                 and     ecx, 3  
SDPI:00481123                 sub     edi, esi  
SDPI:00481125                 shl     ecx, 0Ch  
SDPI:00481128                 add     ebp, edi  
SDPI:0048112A                 mov     \[eax+0Ch\], ecx  
SDPI:0048112D                 mov     \[eax+8\], ebp  
SDPI:00481130                 mov     edi, ebp  
SDPI:00481132                 add     eax, 28h  
SDPI:00481135                 add     ecx, edi  
SDPI:00481137                 mov     edi, \[eax+14h\]  
SDPI:0048113A                 mov     ebp, \[eax+8\]  
SDPI:0048113D                 add     edi, edx  
SDPI:0048113F                 mov     \[eax+0Ch\], ecx  
SDPI:00481142                 mov     \[eax+14h\], edi  
SDPI:00481145                 mov     edi, 1000h  
SDPI:0048114A                 sub     edi, esi  
SDPI:0048114C                 add     eax, 28h  
SDPI:0048114F                 add     ebp, edi  
SDPI:00481151                 mov     \[eax-20h\], ebp  
SDPI:00481154                 mov     edi, ebp  
SDPI:00481156                 add     ecx, edi  
SDPI:00481158                 mov     edi, \[eax+14h\]  
SDPI:0048115B                 mov     ebp, \[eax+8\]  
SDPI:0048115E                 add     edi, edx  
SDPI:00481160                 mov     \[eax+14h\], edi  
SDPI:00481163                 mov     edi, 1000h  
SDPI:00481168                 sub     edi, esi  
SDPI:0048116A                 mov     \[eax+0Ch\], ecx  
SDPI:0048116D                 add     ebp, edi  
SDPI:0048116F                 mov     \[eax+8\], ebp  
SDPI:00481172                 mov     edi, ebp  
SDPI:00481174                 add     eax, 28h  
SDPI:00481177                 add     ecx, edi  
SDPI:00481179                 mov     edi, \[eax+14h\]  
SDPI:0048117C                 mov     \[eax+0Ch\], ecx  
SDPI:0048117F                 mov     ecx, 1000h  
SDPI:00481184                 add     edi, edx  
SDPI:00481186                 mov     edx, \[eax+8\]  
SDPI:00481189                 sub     ecx, esi  
SDPI:0048118B                 mov     \[eax+14h\], edi  
SDPI:0048118E                 add     edx, ecx  
SDPI:00481190                 mov     \[eax+8\], edx  
SDPI:00481193                 mov     edx, \[esp+10h\]  
SDPI:00481197                 push    0                         ; lpflOldProtect  
SDPI:00481199                 push    edx                       ; flNewProtect  
SDPI:0048119A                 push    200h                      ; dwSize  
SDPI:0048119F                 push    ebx                       ; lpAddress  
SDPI:004811A0                 call    VirtualProtect  
SDPI:004811A5                 pop     esi                       ; 改完后再把文件头属性还原回去  
SDPI:004811A6                 pop     ebp  
SDPI:004811A7  
SDPI:004811A7 Done\_4811A7:                                      ; CODE XREF: Erase\_PEHEADER+10 j  
SDPI:004811A7                                                   ; Erase\_PEHEADER+1B j ...  
SDPI:004811A7                 pop     edi  
SDPI:004811A8                 pop     ebx  
SDPI:004811A9                 add     esp, 20h  
SDPI:004811AC                 retn  
SDPI:004811AC Erase\_PEHEADER  endp ; sp = -24h  
  
  
  
SDPI:004811F8 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:004811F8  
SDPI:004811F8  
SDPI:004811F8 ; int \_\_stdcall DeCrypt\_Functions(int fromDeAddr,int siz2,int,int size1,DWORD Stack\_to,DWORD Stack\_from)  
SDPI:004811F8 DeCrypt\_Functions proc near                       ; CODE XREF: Disposal\_IMP+152 p  
SDPI:004811F8                                                   ; Disposal\_IMP+193 p ...  
SDPI:004811F8  
SDPI:004811F8 fromDeAddr      = dword ptr  4  
SDPI:004811F8 siz2            = dword ptr  8  
SDPI:004811F8 arg\_8           = dword ptr  0Ch  
SDPI:004811F8 size1           = dword ptr  10h  
SDPI:004811F8 Stack\_to        = dword ptr  14h  
SDPI:004811F8 Stack\_from      = dword ptr  18h  
SDPI:004811F8  
SDPI:004811F8                 mov     eax, \[esp+Stack\_from\]  
SDPI:004811FC                 push    esi  
SDPI:004811FD                 mov     esi, \[esp+4+Stack\_to\]  
SDPI:00481201                 push    edi  
SDPI:00481202                 push    102h                      ; size  
SDPI:00481207                 push    eax                       ; from  
SDPI:00481208                 push    esi                       ; to  
SDPI:00481209                 call    RtlMoveMemory             ; 复制内存  
SDPI:0048120E                 push    esi  
SDPI:0048120F                 mov     esi, \[esp+0Ch+siz2\]  
SDPI:00481213                 mov     edi, \[esp+0Ch+fromDeAddr\]  
SDPI:00481217                 push    esi  
SDPI:00481218                 push    edi  
SDPI:00481219                 call    sub\_4805C2  
SDPI:0048121E                 mov     ecx, \[esp+14h+size1\]  
SDPI:00481222                 mov     edx, \[esp+14h+arg\_8\]  
SDPI:00481226                 add     esp, 0Ch  
SDPI:00481229                 push    ecx  
SDPI:0048122A                 push    edx  
SDPI:0048122B                 push    esi  
SDPI:0048122C                 push    edi  
SDPI:0048122D                 call    sub\_4811AD  
SDPI:00481232                 pop     edi  
SDPI:00481233                 mov     eax, 1  
SDPI:00481238                 pop     esi  
SDPI:00481239                 retn    18h  
SDPI:00481239 DeCrypt\_Functions endp  
  
  
  
SDPI:00481275 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:00481275  
SDPI:00481275 ; 判断 MD5 效验值是否相等  
SDPI:00481275 ; 如果相等则 eax==edi 的值  
SDPI:00481275 ; 不相等返回 FE5F3AFE  
SDPI:00481275  
SDPI:00481275 CMP\_HASH\_481275 proc near                         ; CODE XREF: SDPI:0047B458 j  
SDPI:00481275  
SDPI:00481275 RetBuffer       = dword ptr -68h  
SDPI:00481275 InBuffer        = dword ptr -58h  
SDPI:00481275  
SDPI:00481275                 sub     esp, 68h  
SDPI:00481278                 push    edi  
SDPI:00481279                 call    GetTickCount  
SDPI:0048127E                 mov     edi, eax  
SDPI:00481280                 cmp     edi, 0FE5F3AFEh  
SDPI:00481286                 jnz     short loc\_48128D  
SDPI:00481288                 mov     edi, 0FE5F3AFFh  
SDPI:0048128D  
SDPI:0048128D loc\_48128D:                                       ; CODE XREF: CMP\_HASH\_481275+11 j  
SDPI:0048128D                 push    esi  
SDPI:0048128E                 call    Get\_BASE  
SDPI:00481293                 mov     esi, eax  
SDPI:00481295                 lea     eax, \[esp+70h+InBuffer\]  
SDPI:00481299                 push    eax                       ; initBuffer  
SDPI:0048129A                 call    INIT\_MD5  
SDPI:0048129F                 lea     ecx, \[esi+0B8h\]  
SDPI:004812A5                 push    10h                       ; BufferSize  
SDPI:004812A7                 lea     edx, \[esp+78h+InBuffer\]  
SDPI:004812AB                 push    ecx                       ; CalcBuffer  
SDPI:004812AC                 push    edx                       ; RetBuffer  
SDPI:004812AD                 call    Hash\_MD5  
SDPI:004812B2                 lea     eax, \[esp+80h+InBuffer\]  
SDPI:004812B6                 lea     ecx, \[esp+80h+RetBuffer\]  
SDPI:004812BA                 push    eax                       ; InBuffer  
SDPI:004812BB                 push    ecx                       ; RetBuffer  
SDPI:004812BC                 call    Calculate\_MD5  
SDPI:004812C1                 add     esp, 18h  
SDPI:004812C4                 add     esi, 0D0h  
SDPI:004812CA                 lea     edx, \[esp+70h+RetBuffer\]  
SDPI:004812CE                 push    10h                       ; size  
SDPI:004812D0                 push    esi                       ; Original\_value  
SDPI:004812D1                 push    edx                       ; Calculate\_value  
SDPI:004812D2                 call    Compare\_HASH              ; 把 Calculate\_value 改为 Original\_value  
SDPI:004812D2                                                   ; 就通过检测了  
SDPI:004812D7                 test    eax, eax  
SDPI:004812D9                 pop     esi  
SDPI:004812DA                 mov     eax, 0FE5F3AFEh  
SDPI:004812DF                 jnz     short loc\_4812E3  
SDPI:004812E1                 mov     eax, edi  
SDPI:004812E3  
SDPI:004812E3 loc\_4812E3:                                       ; CODE XREF: CMP\_HASH\_481275+6A j  
SDPI:004812E3                 pop     edi  
SDPI:004812E4                 add     esp, 68h  
SDPI:004812E7                 retn  
SDPI:004812E7 CMP\_HASH\_481275 endp  
SDPI:004812E7  
  
  
  
SDPI:0048179E ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0048179E  
SDPI:0048179E ; 载入目标 DLL  
SDPI:0048179E  
SDPI:0048179E ; int \_\_stdcall LoadDll(int lpLibFileName)  
SDPI:0048179E LoadDll         proc near                         ; CODE XREF: Disposal\_IMP+C0 p  
SDPI:0048179E                                                   ; Disposal\_IMP+22A p  
SDPI:0048179E  
SDPI:0048179E lpLibFileName   = dword ptr  8  
SDPI:0048179E  
SDPI:0048179E                 push    esi  
SDPI:0048179F                 mov     esi, \[esp+lpLibFileName\]  
SDPI:004817A3                 push    esi  
SDPI:004817A4                 call    api\_GetModule  
SDPI:004817A9                 test    eax, eax  
SDPI:004817AB                 jnz     short loc\_4817B3  
SDPI:004817AD                 push    esi  
SDPI:004817AE                 call    api\_LoadLibraryA  
SDPI:004817B3  
SDPI:004817B3 loc\_4817B3:                                       ; CODE XREF: LoadDll+D j  
SDPI:004817B3                 pop     esi  
SDPI:004817B4                 retn    4  
SDPI:004817B4 LoadDll         endp ; sp = -8  
SDPI:004817B4  
SDPI:004817B7  
  
  
  
SDPI:004817E0 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:004817E0  
SDPI:004817E0 ; 加密输入表,把 call api 搬到壳里去  
SDPI:004817E0  
SDPI:004817E0 ; int \_\_stdcall HOOK\_API\_JMP(DWORD APIAddress)  
SDPI:004817E0 HOOK\_API\_JMP    proc near                         ; CODE XREF: Disposal\_IMP+364 p  
SDPI:004817E0                                                   ; Disposal\_IMP+5A3 p  
SDPI:004817E0  
SDPI:004817E0 from            = dword ptr -20h  
SDPI:004817E0 var\_1C          = dword ptr -1Ch  
SDPI:004817E0 var\_18          = dword ptr -18h  
SDPI:004817E0 var\_14          = dword ptr -14h  
SDPI:004817E0 var\_10          = dword ptr -10h  
SDPI:004817E0 var\_C           = dword ptr -0Ch  
SDPI:004817E0 var\_8           = dword ptr -8  
SDPI:004817E0 APIAddress      = dword ptr  4  
SDPI:004817E0  
SDPI:004817E0                 sub     esp, 1Ch  
SDPI:004817E3                 push    edi  
SDPI:004817E4                 push    20h  
SDPI:004817E6                 call    sub\_4808D3  
SDPI:004817EB                 mov     edi, eax  
SDPI:004817ED                 test    edi, edi  
SDPI:004817EF                 jz      loc\_4818B4  
SDPI:004817F5                 push    esi  
SDPI:004817F6                 lea     ecx, \[esp+24h+var\_1C\]  
SDPI:004817FA                 push    2                         ; size  
SDPI:004817FC                 lea     eax, \[edi+16h\]  
SDPI:004817FF                 push    ecx                       ; from  
SDPI:00481800                 push    edi                       ; to  
SDPI:00481801                 mov     \[esp+30h+var\_1C\], 0E74h  
SDPI:00481809                 mov     \[esp+30h+var\_18\], 0C75h  
SDPI:00481811                 mov     \[esp+30h+var\_8\], 25FFh  
SDPI:00481819                 mov     \[esp+30h+var\_14\], 53321AE8h  
SDPI:00481821                 mov     \[esp+30h+var\_10\], 40A64Eh  
SDPI:00481829                 mov     \[esp+30h+var\_C\], 40A59Ah  
SDPI:00481831                 mov     \[esp+2Ch\], eax  
SDPI:00481835                 call    RtlMoveMemory             ; 复制内存  
SDPI:0048183A                 lea     edx, \[esp+24h+var\_18\]  
SDPI:0048183E                 lea     esi, \[edi+2\]  
SDPI:00481841                 push    2                         ; size  
SDPI:00481843                 push    edx                       ; from  
SDPI:00481844                 push    esi                       ; to  
SDPI:00481845                 call    RtlMoveMemory             ; 复制内存  
SDPI:0048184A                 lea     eax, \[esp+24h+var\_14\]  
SDPI:0048184E                 add     esi, 2  
SDPI:00481851                 push    4                         ; size  
SDPI:00481853                 push    eax                       ; from  
SDPI:00481854                 push    esi                       ; to  
SDPI:00481855                 call    RtlMoveMemory             ; 复制内存  
SDPI:0048185A                 lea     ecx, \[esp+24h+var\_10\]  
SDPI:0048185E                 add     esi, 4  
SDPI:00481861                 push    4                         ; size  
SDPI:00481863                 push    ecx                       ; from  
SDPI:00481864                 push    esi                       ; to  
SDPI:00481865                 call    RtlMoveMemory             ; 复制内存  
SDPI:0048186A                 lea     edx, \[esp+24h+var\_C\]  
SDPI:0048186E                 add     esi, 4  
SDPI:00481871                 push    4                         ; size  
SDPI:00481873                 push    edx                       ; from  
SDPI:00481874                 push    esi                       ; to  
SDPI:00481875                 call    RtlMoveMemory             ; 复制内存  
SDPI:0048187A                 lea     eax, \[esp+24h+var\_8\]  
SDPI:0048187E                 add     esi, 4  
SDPI:00481881                 push    2                         ; size  
SDPI:00481883                 push    eax                       ; from  
SDPI:00481884                 push    esi                       ; to  
SDPI:00481885                 call    RtlMoveMemory             ; 复制内存  
SDPI:0048188A                 lea     ecx, \[esp+20h\]  
SDPI:0048188E                 add     esi, 2  
SDPI:00481891                 push    4                         ; size  
SDPI:00481893                 push    ecx                       ; from  
SDPI:00481894                 push    esi                       ; to  
SDPI:00481895                 call    RtlMoveMemory             ; 复制内存  
SDPI:0048189A                 lea     edx, \[esp+24h+APIAddress\]  
SDPI:0048189E                 push    4                         ; size  
SDPI:004818A0                 add     esi, 4  
SDPI:004818A3                 push    edx                       ; from  
SDPI:004818A4                 push    esi                       ; to  
SDPI:004818A5                 call    RtlMoveMemory             ; 复制内存  
SDPI:004818AA                 mov     eax, edi  
SDPI:004818AC                 pop     esi  
SDPI:004818AD                 pop     edi  
SDPI:004818AE                 add     esp, 1Ch  
SDPI:004818B1                 retn    4  
SDPI:004818B4 ; --------------------------------------------------------------------  
  
  
  
SDPI:004818BC ; ----------------------------------------------------------------------------  
SDPI:004818BF aGetprocessheap db 'GetProcessHeap',0  
SDPI:004818CE aHeapalloc      db 'HeapAlloc',0  
SDPI:004818D8 aLoadlibrarya   db 'LoadLibraryA',0  
SDPI:004818E5 aUser32\_dll     db 'user32.dll',0  
SDPI:004818F0 aGdi32\_dll      db 'gdi32.dll',0  
SDPI:004818FA aAdvapi32\_dll   db 'advapi32.dll',0  
SDPI:00481907 aShell32\_dll    db 'shell32.dll',0  
SDPI:00481913 aCanNotLoadSPleaseMa db 'Can not load %s, please make sure it exist in this',0Ah  
SDPI:00481913                 db 'or system folder, then try to launch again.',0  
SDPI:00481972 aGetsystemdirectorya db 'GetSystemDirectoryA',0  
SDPI:00481986 aGetversionexa  db 'GetVersionExA',0  
SDPI:00481994 aGetcommandlinea db 'GetCommandLineA',0  
SDPI:004819A4 aGetcurrentprocessid db 'GetCurrentProcessId',0  
SDPI:004819B8 aGetcurrentdirectory db 'GetCurrentDirectoryA',0  
SDPI:004819CD aSetcurrentdirectory db 'SetCurrentDirectoryA',0  
SDPI:004819E2 aGetmoudlehandlea db 'GetMoudleHandleA',0  
SDPI:004819F3 aGetmoudlefilenamea db 'GetMoudleFileNameA',0  
SDPI:00481A06 aGetversion     db 'GetVersion',0  
SDPI:00481A11 aGetstartupinfoa db 'GetStartupInfoA',0  
SDPI:00481A21  
SDPI:00481A21 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:00481A21  
SDPI:00481A21 ; 用于解压密出壳的输入表信息  
SDPI:00481A21 ; 如果操作成功 EAX 返回 1  
SDPI:00481A21  
SDPI:00481A21 ; int \_\_stdcall Disposal\_IMP(int IsClient\_FLG)  
SDPI:00481A21 Disposal\_IMP    proc near                         ; CODE XREF: SDPI:0047A3EA j  
SDPI:00481A21                                                   ; SDPI:0047DCAF j  
SDPI:00481A21  
SDPI:00481A21 var\_448         = dword ptr -448h  
SDPI:00481A21 var\_444         = dword ptr -444h  
SDPI:00481A21 var\_440         = dword ptr -440h  
SDPI:00481A21 var\_43C         = dword ptr -43Ch  
SDPI:00481A21 var\_438         = dword ptr -438h  
SDPI:00481A21 hdll            = dword ptr -434h  
SDPI:00481A21 var\_430         = dword ptr -430h  
SDPI:00481A21 var\_42C         = dword ptr -42Ch  
SDPI:00481A21 var\_428         = dword ptr -428h  
SDPI:00481A21 hMem\_IMPVA      = dword ptr -424h  
SDPI:00481A21 var\_420         = dword ptr -420h  
SDPI:00481A21 var\_31C         = dword ptr -31Ch  
SDPI:00481A21 var\_218         = dword ptr -218h  
SDPI:00481A21 var\_14          = dword ptr -14h  
SDPI:00481A21 IsClient\_FLG    = dword ptr  4  
SDPI:00481A21  
SDPI:00481A21                 sub     esp, 430h  
SDPI:00481A27                 push    ebx  
SDPI:00481A28                 push    ebp  
SDPI:00481A29                 push    esi  
SDPI:00481A2A                 push    edi  
SDPI:00481A2B                 call    INT3\_47F261               ; 检测调试器是否存在  
SDPI:00481A30                 mov     ebp, \[esp+440h+IsClient\_FLG\]  
SDPI:00481A37                 mov     \[esp+440h+hMem\_IMPVA\], 0  ; 判断是否为程序的输入表处理  
SDPI:00481A37                                                   ; 如果是程序的输入表处理则  
SDPI:00481A37                                                   ; 传入的 is\_client\_flg 标志为 1  
SDPI:00481A3F                 test    ebp, ebp  
SDPI:00481A41                 jnz     short IsClient\_Mode  
SDPI:00481A43                 push    offset strGetpheap        ; relativeAddress  
SDPI:00481A48                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481A4D                 push    eax  
SDPI:00481A4E                 push    offset strkerdll          ; relativeAddress  
SDPI:00481A53                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481A58                 push    eax                       ; APIName  
SDPI:00481A59                 call    api\_LoadLibraryA  
SDPI:00481A5E                 push    eax                       ; hdll  
SDPI:00481A5F                 call    API\_GetProcAddr  
SDPI:00481A64                 push    offset unk\_4088CE         ; relativeAddress  
SDPI:00481A69                 mov     edi, eax  
SDPI:00481A6B                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481A70                 push    eax  
SDPI:00481A71                 push    offset strkerdll          ; relativeAddress  
SDPI:00481A76                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481A7B                 push    eax                       ; APIName  
SDPI:00481A7C                 call    api\_LoadLibraryA  
SDPI:00481A81                 push    eax                       ; hdll  
SDPI:00481A82                 call    API\_GetProcAddr  
SDPI:00481A87                 mov     esi, eax  
SDPI:00481A89                 call    Get\_BASE  
SDPI:00481A8E                 test    esi, esi  
SDPI:00481A90                 mov     ebx, eax  
SDPI:00481A92                 jz      short loc\_481AA8  
SDPI:00481A94                 test    edi, edi  
SDPI:00481A96                 jz      short loc\_481AA8  
SDPI:00481A98                 push    320h  
SDPI:00481A9D                 push    8  
SDPI:00481A9F                 call    edi  
SDPI:00481AA1                 push    eax  
SDPI:00481AA2                 call    esi  
SDPI:00481AA4                 mov     \[esp+440h+hMem\_IMPVA\], eax  
SDPI:00481AA8  
SDPI:00481AA8 loc\_481AA8:                                       ; CODE XREF: Disposal\_IMP+71 j  
SDPI:00481AA8                                                   ; Disposal\_IMP+75 j  
SDPI:00481AA8                 mov     eax, \[esp+440h+hMem\_IMPVA\]  
SDPI:00481AAC                 mov     \[ebx+70h\], eax  
SDPI:00481AAF  
SDPI:00481AAF IsClient\_Mode:                                    ; CODE XREF: Disposal\_IMP+20 j  
SDPI:00481AAF                 mov     \[esp+440h+var\_428\], 0  
SDPI:00481AB7                 call    sub\_481748  
SDPI:00481ABC                 cmp     eax, 13B8C8B9h  
SDPI:00481AC1                 jnz     short loc\_481ACB  
SDPI:00481AC3                 mov     \[esp+440h+var\_428\], 1  
SDPI:00481ACB  
SDPI:00481ACB loc\_481ACB:                                       ; CODE XREF: Disposal\_IMP+A0 j  
SDPI:00481ACB                 push    offset strLoadLibraryA    ; relativeAddress  
SDPI:00481AD0                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481AD5                 push    eax                       ; APIName  
SDPI:00481AD6                 push    offset strkerdll          ; relativeAddress  
SDPI:00481ADB                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481AE0                 push    eax                       ; lpLibFileName  
SDPI:00481AE1                 call    LoadDll                   ; 载入目标 DLL  
SDPI:00481AE6                 push    eax                       ; hdll  
SDPI:00481AE7                 call    API\_GetProcAddr  
SDPI:00481AEC                 test    eax, eax  
SDPI:00481AEE                 mov     \[esp+30h\], eax  
SDPI:00481AF2                 jnz     short loc\_481AFC  
SDPI:00481AF4                 mov     dword ptr \[esp+30h\], offset unk\_40A64E  
SDPI:00481AFC  
SDPI:00481AFC loc\_481AFC:                                       ; CODE XREF: Disposal\_IMP+D1 j  
SDPI:00481AFC                 push    0  
SDPI:00481AFE                 call    api\_GetModule  
SDPI:00481B03                 mov     ebx, eax  
SDPI:00481B05                 mov     \[esp+34h\], ebx  
SDPI:00481B09                 call    Get\_BASE  
SDPI:00481B0E                 mov     edi, eax  
SDPI:00481B10                 lea     ecx, \[esp+38h\]  
SDPI:00481B14                 push    ecx                       ; retBuffer  
SDPI:00481B15                 push    10h                       ; size  
SDPI:00481B17                 lea     eax, \[edi+0A8h\]  
SDPI:00481B1D                 push    eax                       ; sFrom  
SDPI:00481B1E                 mov     \[esp+450h+var\_430\], eax  
SDPI:00481B22                 call    DECode  
SDPI:00481B27                 add     esp, 0Ch  
SDPI:00481B2A                 test    ebp, ebp  
SDPI:00481B2C                 jz      short loc\_481B33  
SDPI:00481B2E                 mov     esi, \[edi+1Ch\]  
SDPI:00481B31                 jmp     short loc\_481B36  
SDPI:00481B33 ; ----------------------------------------------------------------------------  
SDPI:00481B33  
SDPI:00481B33 loc\_481B33:                                       ; CODE XREF: Disposal\_IMP+10B j  
SDPI:00481B33                 mov     esi, \[edi+0Ch\]  
SDPI:00481B36  
SDPI:00481B36 loc\_481B36:                                       ; CODE XREF: Disposal\_IMP+110 j  
SDPI:00481B36                 mov     ebp, \[edi+74h\]  
SDPI:00481B39                 add     esi, ebx  
SDPI:00481B3B                 mov     \[esp+2Ch\], ebp  
SDPI:00481B3F                 call    CRC\_480467  
SDPI:00481B44                 mov     edi, esi  
SDPI:00481B46  
SDPI:00481B46 loc\_481B46:                                       ; CODE XREF: Disposal\_IMP+2AD j  
SDPI:00481B46                 mov     \[esp+444h+hMem\_IMPVA\], edi ; 保存输入表起始 VA  
SDPI:00481B4A  
SDPI:00481B4A loc\_481B4A:                                       ; CODE XREF: Disposal\_IMP+63C j  
SDPI:00481B4A                 mov     edx, \[edi\]                ; 获取输入表信息  
SDPI:00481B4C                 mov     eax, \[edi+10h\]  
SDPI:00481B4F                 or      edx, eax  
SDPI:00481B51                 jz      loc\_482062                ; 获取完跳去下一步  
SDPI:00481B57                 call    CRC\_480467                ; 很多这样的无聊的检测  
SDPI:00481B5C                 mov     edx, \[esp+444h+var\_430\]  
SDPI:00481B60                 lea     eax, \[esp+38h\]  
SDPI:00481B64                 lea     ecx, \[esp+13Ch\]  
SDPI:00481B6B                 push    eax                       ; Stack\_from  
SDPI:00481B6C                 push    ecx                       ; Stack\_to  
SDPI:00481B6D                 push    10h                       ; size1  
SDPI:00481B6F                 push    edx                       ; int  
SDPI:00481B70                 push    14h                       ; siz2  
SDPI:00481B72                 push    edi                       ; fromDeAddr  
SDPI:00481B73                 call    DeCrypt\_Functions         ; 解密函数  
SDPI:00481B78                 mov     eax, \[esp+444h\]           ; 解密出输入表结构信息  
SDPI:00481B7F                 test    eax, eax  
SDPI:00481B81                 jz      short loc\_481B90  
SDPI:00481B83                 mov     esi, \[edi+0Ch\]  
SDPI:00481B86                 add     esi, ebx  
SDPI:00481B88                 add     esi, ebp  
SDPI:00481B8A                 mov     \[esp+444h+var\_420\], esi  
SDPI:00481B8E                 jmp     short loc\_481B9B  
SDPI:00481B90 ; ----------------------------------------------------------------------------  
SDPI:00481B90  
SDPI:00481B90 loc\_481B90:                                       ; CODE XREF: Disposal\_IMP+160 j  
SDPI:00481B90                 mov     eax, \[edi+0Ch\]  
SDPI:00481B93                 add     eax, ebx  
SDPI:00481B95                 mov     \[esp+444h+var\_420\], eax  
SDPI:00481B99                 mov     esi, eax  
SDPI:00481B9B  
SDPI:00481B9B loc\_481B9B:                                       ; CODE XREF: Disposal\_IMP+16D j  
SDPI:00481B9B                 mov     edx, \[esp+444h+var\_430\]  
SDPI:00481B9F                 lea     eax, \[esp+38h\]  
SDPI:00481BA3                 lea     ecx, \[esp+13Ch\]  
SDPI:00481BAA                 push    eax                       ; Stack\_from  
SDPI:00481BAB                 mov     eax, \[edi+4\]  
SDPI:00481BAE                 push    ecx                       ; Stack\_to  
SDPI:00481BAF                 push    10h                       ; size1  
SDPI:00481BB1                 push    edx                       ; int  
SDPI:00481BB2                 push    eax                       ; siz2  
SDPI:00481BB3                 push    esi                       ; fromDeAddr  
SDPI:00481BB4                 call    DeCrypt\_Functions         ; 这里解密出 DLL 名称  
SDPI:00481BB9                 call    CRC\_480467  
SDPI:00481BBE  
SDPI:00481BBE loc\_481BBE:                                       ; CODE XREF: Disposal\_IMP+286 j  
SDPI:00481BBE                 push    offset strkerdll          ; relativeAddress  
SDPI:00481BC3                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481BC8                 push    eax                       ; LPCTSTR  
SDPI:00481BC9                 push    esi                       ; LPCTSTR  
SDPI:00481BCA                 call    lstrcmp                   ; 比较字符串  
SDPI:00481BCF                 test    eax, eax                  ; 判断是否需要特殊处理  
SDPI:00481BCF                                                   ; 只针对几个常用的 DLL 进行处理  
SDPI:00481BD1                 jz      short Set\_Sp\_FLG          ; 如果是特殊 DLL 则置特殊标志为 1  
SDPI:00481BD1                                                   ; 否则置特殊标志为 0  
SDPI:00481BD3                 push    offset strdllusr32        ; relativeAddress  
SDPI:00481BD8                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481BDD                 push    eax                       ; LPCTSTR  
SDPI:00481BDE                 push    esi                       ; LPCTSTR  
SDPI:00481BDF                 call    lstrcmp                   ; 如果相等为 0  
SDPI:00481BE4                 test    eax, eax  
SDPI:00481BE6                 jz      short Set\_Sp\_FLG          ; 如果是特殊 DLL 则置特殊标志为 1  
SDPI:00481BE6                                                   ; 否则置特殊标志为 0  
SDPI:00481BE8                 push    offset strdllgdi32        ; relativeAddress  
SDPI:00481BED                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481BF2                 push    eax                       ; LPCTSTR  
SDPI:00481BF3                 push    esi                       ; LPCTSTR  
SDPI:00481BF4                 call    lstrcmp                   ; 如果相等为 0  
SDPI:00481BF9                 test    eax, eax  
SDPI:00481BFB                 jz      short Set\_Sp\_FLG          ; 如果是特殊 DLL 则置特殊标志为 1  
SDPI:00481BFB                                                   ; 否则置特殊标志为 0  
SDPI:00481BFD                 push    offset strdlladvapi32     ; relativeAddress  
SDPI:00481C02                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481C07                 push    eax                       ; LPCTSTR  
SDPI:00481C08                 push    esi                       ; LPCTSTR  
SDPI:00481C09                 call    lstrcmp                   ; 如果相等为 0  
SDPI:00481C0E                 test    eax, eax  
SDPI:00481C10                 jz      short Set\_Sp\_FLG          ; 如果是特殊 DLL 则置特殊标志为 1  
SDPI:00481C10                                                   ; 否则置特殊标志为 0  
SDPI:00481C12                 push    offset strdllshell32      ; relativeAddress  
SDPI:00481C17                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481C1C                 push    eax                       ; LPCTSTR  
SDPI:00481C1D                 push    esi                       ; LPCTSTR  
SDPI:00481C1E                 call    lstrcmp                   ; 如果相等为 0  
SDPI:00481C23                 test    eax, eax  
SDPI:00481C25                 mov     dword ptr \[esp+28h\], 0  
SDPI:00481C2D                 jnz     short loc\_481C37  
SDPI:00481C2F  
SDPI:00481C2F Set\_Sp\_FLG:                                       ; CODE XREF: Disposal\_IMP+1B0 j  
SDPI:00481C2F                                                   ; Disposal\_IMP+1C5 j ...  
SDPI:00481C2F                 mov     dword ptr \[esp+28h\], 1    ; 如果是特殊 DLL 则置特殊标志为 1  
SDPI:00481C2F                                                   ; 否则置特殊标志为 0  
SDPI:00481C37  
SDPI:00481C37 loc\_481C37:                                       ; CODE XREF: Disposal\_IMP+20C j  
SDPI:00481C37                 mov     eax, \[esp+444h\]  
SDPI:00481C3E                 test    eax, eax  
SDPI:00481C40                 jz      short loc\_481C52  
SDPI:00481C42                 mov     eax, \[esp+444h+var\_42C\]  
SDPI:00481C46                 test    eax, eax  
SDPI:00481C48                 jz      short loc\_481C52  
SDPI:00481C4A                 push    esi                       ; lpLibFileName  
SDPI:00481C4B                 call    LoadDll                   ; 载入目标 DLL  
SDPI:00481C50                 jmp     short loc\_481C5D  
SDPI:00481C52 ; ----------------------------------------------------------------------------  
SDPI:00481C52  
SDPI:00481C52 loc\_481C52:                                       ; CODE XREF: Disposal\_IMP+21F j  
SDPI:00481C52                                                   ; Disposal\_IMP+227 j  
SDPI:00481C52                 mov     ecx, \[esp+30h\]  
SDPI:00481C56                 push    esi                       ; LPCSTR  
SDPI:00481C57                 push    ecx                       ; int  
SDPI:00481C58                 call    LOadDll\_4817B7            ; 载入相应的 DLL  
SDPI:00481C5D  
SDPI:00481C5D loc\_481C5D:                                       ; CODE XREF: Disposal\_IMP+22F j  
SDPI:00481C5D                 test    eax, eax  
SDPI:00481C5F                 mov     \[esp+444h+hdll\], eax  
SDPI:00481C63                 jnz     short loc\_481CB3  
SDPI:00481C65                 push    esi  
SDPI:00481C66                 push    offset strcanntloaddll    ; relativeAddress  
SDPI:00481C6B                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481C70                 lea     edx, \[esp+244h\]  
SDPI:00481C77                 push    eax                       ; strIn  
SDPI:00481C78                 push    edx                       ; strout  
SDPI:00481C79                 call    wsprintfA                 ; 如果不能载入 dll 则提示出错  
SDPI:00481C7E                 add     esp, 0Ch  
SDPI:00481C81                 push    12h                       ; uType  
SDPI:00481C83                 push    offset strError           ; relativeAddress  
SDPI:00481C88                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481C8D                 push    eax                       ; lpCaption  
SDPI:00481C8E                 lea     eax, \[esp+248h\]  
SDPI:00481C95                 push    eax                       ; lpText  
SDPI:00481C96                 push    0                         ; hWnd  
SDPI:00481C98                 call    MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:00481C98                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:00481C98                                                   ; 也就是判断有没有下 int3 断点  
SDPI:00481C9D                 cmp     eax, 3  
SDPI:00481CA0                 jz      short loc\_481CAC  
SDPI:00481CA2                 cmp     eax, 4  
SDPI:00481CA5                 jnz     short loc\_481CCB  
SDPI:00481CA7                 jmp     loc\_481BBE  
SDPI:00481CAC ; ----------------------------------------------------------------------------  
SDPI:00481CAC  
SDPI:00481CAC loc\_481CAC:                                       ; CODE XREF: Disposal\_IMP+27F j  
SDPI:00481CAC                 push    0                         ; uExitCode  
SDPI:00481CAE                 call    ExitProcess  
SDPI:00481CB3  
SDPI:00481CB3 loc\_481CB3:                                       ; CODE XREF: Disposal\_IMP+242 j  
SDPI:00481CB3                 mov     eax, \[esp+444h\]  
SDPI:00481CBA                 test    eax, eax  
SDPI:00481CBC                 mov     eax, \[edi\]  
SDPI:00481CBE                 lea     ecx, \[ebx+eax\]  
SDPI:00481CC1                 jz      short loc\_481CD3  
SDPI:00481CC3                 add     ecx, ebp  
SDPI:00481CC5                 lea     esi, \[ebp+ebx+0\]  
SDPI:00481CC9                 jmp     short loc\_481CD5  
SDPI:00481CCB ; ----------------------------------------------------------------------------  
SDPI:00481CCB  
SDPI:00481CCB loc\_481CCB:                                       ; CODE XREF: Disposal\_IMP+284 j  
SDPI:00481CCB                 add     edi, 14h  
SDPI:00481CCE                 jmp     loc\_481B46                ; 保存输入表起始 VA  
SDPI:00481CD3 ; ----------------------------------------------------------------------------  
SDPI:00481CD3  
SDPI:00481CD3 loc\_481CD3:                                       ; CODE XREF: Disposal\_IMP+2A0 j  
SDPI:00481CD3                 mov     esi, ebx  
SDPI:00481CD5  
SDPI:00481CD5 loc\_481CD5:                                       ; CODE XREF: Disposal\_IMP+2A8 j  
SDPI:00481CD5                 mov     edx, \[edi+10h\]  
SDPI:00481CD8                 mov     ebp, ecx  
SDPI:00481CDA                 add     esi, edx  
SDPI:00481CDC                 test    eax, eax  
SDPI:00481CDE                 jnz     short loc\_481CE2  
SDPI:00481CE0                 mov     ebp, esi  
SDPI:00481CE2  
SDPI:00481CE2 loc\_481CE2:                                       ; CODE XREF: Disposal\_IMP+2BD j  
SDPI:00481CE2                 call    CRC\_480467  
SDPI:00481CE7  
SDPI:00481CE7 loc\_481CE7:                                       ; CODE XREF: Disposal\_IMP+607 j  
SDPI:00481CE7                 cmp     dword ptr \[ebp+0\], 0      ; 上面载入 DLL 后，这里填充当前 DLL 的加载函数  
SDPI:00481CE7                                                   ; 填充完跳去处理下一个 DLL  
SDPI:00481CEB                 jz      DoneCurrDll\_48202D  
SDPI:00481CF1                 mov     edi, \[esp+444h+var\_430\]  
SDPI:00481CF5                 lea     ecx, \[esp+38h\]  
SDPI:00481CF9                 lea     edx, \[esp+13Ch\]  
SDPI:00481D00                 push    ecx                       ; Stack\_from  
SDPI:00481D01                 push    edx                       ; Stack\_to  
SDPI:00481D02                 push    10h                       ; size1  
SDPI:00481D04                 push    edi                       ; int  
SDPI:00481D05                 push    4                         ; siz2  
SDPI:00481D07                 push    ebp                       ; fromDeAddr  
SDPI:00481D08                 call    DeCrypt\_Functions         ; 解密出 Thunk Value  
SDPI:00481D0D                 lea     eax, \[esp+38h\]  
SDPI:00481D11                 lea     ecx, \[esp+13Ch\]  
SDPI:00481D18                 push    eax                       ; Stack\_from  
SDPI:00481D19                 push    ecx                       ; Stack\_to  
SDPI:00481D1A                 push    10h                       ; size1  
SDPI:00481D1C                 push    edi                       ; int  
SDPI:00481D1D                 push    4                         ; siz2  
SDPI:00481D1F                 push    esi                       ; fromDeAddr  
SDPI:00481D20                 call    DeCrypt\_Functions  
SDPI:00481D25                 mov     eax, \[ebp+0\]  
SDPI:00481D28                 mov     ecx, \[esp+444h\]  
SDPI:00481D2F                 test    ecx, ecx  
SDPI:00481D31                 jz      short loc\_481D3D  
SDPI:00481D33                 mov     edx, \[esp+2Ch\]  
SDPI:00481D37                 add     ebx, eax  
SDPI:00481D39                 add     ebx, edx  
SDPI:00481D3B                 jmp     short loc\_481D3F  
SDPI:00481D3D ; ----------------------------------------------------------------------------  
SDPI:00481D3D  
SDPI:00481D3D loc\_481D3D:                                       ; CODE XREF: Disposal\_IMP+310 j  
SDPI:00481D3D                 add     ebx, eax  
SDPI:00481D3F  
SDPI:00481D3F loc\_481D3F:                                       ; CODE XREF: Disposal\_IMP+31A j  
SDPI:00481D3F                 test    eax, 80000000h  
SDPI:00481D44                 jz      short loc\_481DA8          ; 判断是否为字符串  
SDPI:00481D46                 test    ecx, ecx  
SDPI:00481D48                 jz      short loc\_481D91  
SDPI:00481D4A                 mov     ecx, \[esp+444h+var\_42C\]  
SDPI:00481D4E                 test    ecx, ecx  
SDPI:00481D50                 jz      short loc\_481D6C  
SDPI:00481D52                 mov     edx, \[esi\]  
SDPI:00481D54                 mov     eax, \[esp+444h+hdll\]  
SDPI:00481D58                 and     edx, 7FFFFFFFh  
SDPI:00481D5E                 push    edx  
SDPI:00481D5F                 push    eax  
SDPI:00481D60                 call    sub\_48359A  
SDPI:00481D65                 mov     \[esi\], eax  
SDPI:00481D67                 jmp     loc\_48200C                ; 如果不相等则清除 ThunkValue  
SDPI:00481D6C ; ----------------------------------------------------------------------------  
SDPI:00481D6C  
SDPI:00481D6C loc\_481D6C:                                       ; CODE XREF: Disposal\_IMP+32F j  
SDPI:00481D6C                 mov     ecx, \[esp+28h\]  
SDPI:00481D70                 test    ecx, ecx  
SDPI:00481D72                 jz      short loc\_481D91  
SDPI:00481D74                 mov     ecx, \[esp+444h+hdll\]  
SDPI:00481D78                 and     eax, 7FFFFFFFh  
SDPI:00481D7D                 push    eax                       ; APIName  
SDPI:00481D7E                 push    ecx                       ; hdll  
SDPI:00481D7F                 call    API\_GetProcAddr  
SDPI:00481D84                 push    eax                       ; APIAddress  
SDPI:00481D85                 call    HOOK\_API\_JMP              ; 加密输入表,把 call api 搬到壳里去  
SDPI:00481D8A                 mov     \[esi\], eax                ; 以序号方式时同样加密后填充到原输入表位置  
SDPI:00481D8C                 jmp     loc\_48200C                ; 如果不相等则清除 ThunkValue  
SDPI:00481D91 ; ----------------------------------------------------------------------------  
SDPI:00481D91  
SDPI:00481D91 loc\_481D91:                                       ; CODE XREF: Disposal\_IMP+327 j  
SDPI:00481D91                                                   ; Disposal\_IMP+351 j  
SDPI:00481D91                 mov     edx, \[esp+444h+hdll\]  
SDPI:00481D95                 and     eax, 7FFFFFFFh  
SDPI:00481D9A                 push    eax                       ; APIName  
SDPI:00481D9B                 push    edx                       ; hdll  
SDPI:00481D9C                 call    API\_GetProcAddr  
SDPI:00481DA1                 mov     \[esi\], eax  
SDPI:00481DA3                 jmp     loc\_48200C                ; 如果不相等则清除 ThunkValue  
SDPI:00481DA8 ; ----------------------------------------------------------------------------  
SDPI:00481DA8  
SDPI:00481DA8 loc\_481DA8:                                       ; CODE XREF: Disposal\_IMP+323 j  
SDPI:00481DA8                 mov     edx, \[esp+444h+var\_430\]  
SDPI:00481DAC                 lea     eax, \[esp+38h\]  
SDPI:00481DB0                 lea     ecx, \[esp+13Ch\]  
SDPI:00481DB7                 push    eax                       ; Stack\_from  
SDPI:00481DB8                 push    ecx                       ; Stack\_to  
SDPI:00481DB9                 push    10h                       ; size1  
SDPI:00481DBB                 push    edx                       ; int  
SDPI:00481DBC                 push    2                         ; siz2  
SDPI:00481DBE                 push    ebx                       ; fromDeAddr  
SDPI:00481DBF                 call    DeCrypt\_Functions         ; 解密出序号 Hint  
SDPI:00481DC4                 lea     eax, \[esp+38h\]  
SDPI:00481DC8                 lea     ecx, \[esp+13Ch\]  
SDPI:00481DCF                 mov     edx, \[esp+444h+var\_430\]  
SDPI:00481DD3                 push    eax                       ; Stack\_from  
SDPI:00481DD4                 xor     eax, eax  
SDPI:00481DD6                 push    ecx                       ; Stack\_to  
SDPI:00481DD7                 mov     ax, \[ebx\]  
SDPI:00481DDA                 push    10h                       ; size1  
SDPI:00481DDC                 lea     edi, \[ebx+2\]  
SDPI:00481DDF                 push    edx                       ; int  
SDPI:00481DE0                 push    eax                       ; siz2  
SDPI:00481DE1                 push    edi                       ; fromDeAddr  
SDPI:00481DE2                 call    DeCrypt\_Functions         ; 解密出函数名  
SDPI:00481DE7                 mov     eax, \[esp+444h\]  
SDPI:00481DEE                 test    eax, eax  
SDPI:00481DF0                 jz      loc\_481FCD  
SDPI:00481DF6                 mov     eax, \[esp+444h+var\_42C\]  
SDPI:00481DFA                 test    eax, eax  
SDPI:00481DFC                 jz      short loc\_481E10  
SDPI:00481DFE                 mov     ecx, \[esp+444h+hdll\]  
SDPI:00481E02                 push    edi  
SDPI:00481E03                 push    ecx  
SDPI:00481E04                 call    sub\_48359A  
SDPI:00481E09                 mov     \[esi\], eax  
SDPI:00481E0B                 jmp     loc\_481FF6                ; 如果是需要特殊处理的 DLL  
SDPI:00481E0B                                                   ; 则不跳转，比较是否有特殊函数  
SDPI:00481E10 ; ----------------------------------------------------------------------------  
SDPI:00481E10  
SDPI:00481E10 loc\_481E10:                                       ; CODE XREF: Disposal\_IMP+3DB j  
SDPI:00481E10                 mov     eax, \[esp+28h\]  
SDPI:00481E14                 test    eax, eax  
SDPI:00481E16                 jz      loc\_481FCD  
SDPI:00481E1C                 push    offset strGetWinDir       ; relativeAddress  
SDPI:00481E21                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481E26                 push    eax                       ; specialAPI  
SDPI:00481E27                 push    edi                       ; CurrentAPI  
SDPI:00481E28                 call    StrCompare                ; 其中 CurrentAPI 指当然要操作的 API  
SDPI:00481E28                                                   ; SpecialAPI 指壳指的特殊 API  
SDPI:00481E2D                 test    eax, eax  
SDPI:00481E2F                 jnz     short loc\_481E42  
SDPI:00481E31                 push    offset strGetSysdir       ; relativeAddress  
SDPI:00481E36                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481E3B                 mov     \[esi\], eax  
SDPI:00481E3D                 jmp     loc\_481FF6  
SDPI:00481E42 ; ----------------------------------------------------------------------------  
SDPI:00481E42  
SDPI:00481E42 loc\_481E42:                                       ; CODE XREF: Disposal\_IMP+40E j  
SDPI:00481E42                 push    offset unk\_408972         ; relativeAddress  
SDPI:00481E47                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481E4C                 push    eax                       ; specialAPI  
SDPI:00481E4D                 push    edi                       ; CurrentAPI  
SDPI:00481E4E                 call    StrCompare                ; 其中 CurrentAPI 指当然要操作的 API  
SDPI:00481E4E                                                   ; SpecialAPI 指壳指的特殊 API  
SDPI:00481E53                 test    eax, eax  
SDPI:00481E55                 jnz     short loc\_481E68  
SDPI:00481E57                 push    offset unk\_40799E         ; relativeAddress  
SDPI:00481E5C                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481E61                 mov     \[esi\], eax  
SDPI:00481E63                 jmp     loc\_481FF6  
SDPI:00481E68 ; ----------------------------------------------------------------------------  
SDPI:00481E68  
SDPI:00481E68 loc\_481E68:                                       ; CODE XREF: Disposal\_IMP+434 j  
SDPI:00481E68                 push    offset strGetVersionExA   ; relativeAddress  
SDPI:00481E6D                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481E72                 push    eax                       ; specialAPI  
SDPI:00481E73                 push    edi                       ; CurrentAPI  
SDPI:00481E74                 call    StrCompare                ; 其中 CurrentAPI 指当然要操作的 API  
SDPI:00481E74                                                   ; SpecialAPI 指壳指的特殊 API  
SDPI:00481E79                 test    eax, eax  
SDPI:00481E7B                 jnz     short loc\_481E8E  
SDPI:00481E7D                 push    offset unk\_4079EC         ; relativeAddress  
SDPI:00481E82                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481E87                 mov     \[esi\], eax  
SDPI:00481E89                 jmp     loc\_481FF6  
SDPI:00481E8E ; ----------------------------------------------------------------------------  
SDPI:00481E8E  
SDPI:00481E8E loc\_481E8E:                                       ; CODE XREF: Disposal\_IMP+45A j  
SDPI:00481E8E                 push    offset strGetCMDLine      ; relativeAddress  
SDPI:00481E93                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481E98                 push    eax                       ; specialAPI  
SDPI:00481E99                 push    edi                       ; CurrentAPI  
SDPI:00481E9A                 call    StrCompare                ; 其中 CurrentAPI 指当然要操作的 API  
SDPI:00481E9A                                                   ; SpecialAPI 指壳指的特殊 API  
SDPI:00481E9F                 test    eax, eax  
SDPI:00481EA1                 jnz     short loc\_481EB4  
SDPI:00481EA3                 push    offset unk\_407A29         ; relativeAddress  
SDPI:00481EA8                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481EAD                 mov     \[esi\], eax  
SDPI:00481EAF                 jmp     loc\_481FF6  
SDPI:00481EB4 ; ----------------------------------------------------------------------------  
SDPI:00481EB4  
SDPI:00481EB4 loc\_481EB4:                                       ; CODE XREF: Disposal\_IMP+480 j  
SDPI:00481EB4                 push    offset strGetCurrProcId   ; relativeAddress  
SDPI:00481EB9                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481EBE                 push    eax                       ; specialAPI  
SDPI:00481EBF                 push    edi                       ; CurrentAPI  
SDPI:00481EC0                 call    StrCompare                ; 其中 CurrentAPI 指当然要操作的 API  
SDPI:00481EC0                                                   ; SpecialAPI 指壳指的特殊 API  
SDPI:00481EC5                 test    eax, eax  
SDPI:00481EC7                 jnz     short loc\_481EDA  
SDPI:00481EC9                 push    offset unk\_407A5A         ; relativeAddress  
SDPI:00481ECE                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481ED3                 mov     \[esi\], eax  
SDPI:00481ED5                 jmp     loc\_481FF6  
SDPI:00481EDA ; ----------------------------------------------------------------------------  
SDPI:00481EDA  
SDPI:00481EDA loc\_481EDA:                                       ; CODE XREF: Disposal\_IMP+4A6 j  
SDPI:00481EDA                 push    offset strGetCurrDir      ; relativeAddress  
SDPI:00481EDF                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481EE4                 push    eax                       ; specialAPI  
SDPI:00481EE5                 push    edi                       ; CurrentAPI  
SDPI:00481EE6                 call    StrCompare                ; 其中 CurrentAPI 指当然要操作的 API  
SDPI:00481EE6                                                   ; SpecialAPI 指壳指的特殊 API  
SDPI:00481EEB                 test    eax, eax  
SDPI:00481EED                 jnz     short loc\_481F00  
SDPI:00481EEF                 push    offset unk\_407BA0         ; relativeAddress  
SDPI:00481EF4                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481EF9                 mov     \[esi\], eax  
SDPI:00481EFB                 jmp     loc\_481FF6  
SDPI:00481F00 ; ----------------------------------------------------------------------------  
SDPI:00481F00  
SDPI:00481F00 loc\_481F00:                                       ; CODE XREF: Disposal\_IMP+4CC j  
SDPI:00481F00                 push    offset strSetCurrDir      ; relativeAddress  
SDPI:00481F05                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481F0A                 push    eax                       ; specialAPI  
SDPI:00481F0B                 push    edi                       ; CurrentAPI  
SDPI:00481F0C                 call    StrCompare                ; 其中 CurrentAPI 指当然要操作的 API  
SDPI:00481F0C                                                   ; SpecialAPI 指壳指的特殊 API  
SDPI:00481F11                 test    eax, eax  
SDPI:00481F13                 jnz     short loc\_481F26  
SDPI:00481F15                 push    407CA8h                   ; relativeAddress  
SDPI:00481F1A                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481F1F                 mov     \[esi\], eax  
SDPI:00481F21                 jmp     loc\_481FF6  
SDPI:00481F26 ; ----------------------------------------------------------------------------  
SDPI:00481F26  
SDPI:00481F26 loc\_481F26:                                       ; CODE XREF: Disposal\_IMP+4F2 j  
SDPI:00481F26                 push    offset strGetModuleA      ; relativeAddress  
SDPI:00481F2B                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481F30                 push    eax                       ; specialAPI  
SDPI:00481F31                 push    edi                       ; CurrentAPI  
SDPI:00481F32                 call    StrCompare                ; 其中 CurrentAPI 指当然要操作的 API  
SDPI:00481F32                                                   ; SpecialAPI 指壳指的特殊 API  
SDPI:00481F37                 test    eax, eax  
SDPI:00481F39                 jnz     short loc\_481F4C  
SDPI:00481F3B                 push    offset unk\_407C86         ; relativeAddress  
SDPI:00481F40                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481F45                 mov     \[esi\], eax  
SDPI:00481F47                 jmp     loc\_481FF6  
SDPI:00481F4C ; ----------------------------------------------------------------------------  
SDPI:00481F4C  
SDPI:00481F4C loc\_481F4C:                                       ; CODE XREF: Disposal\_IMP+518 j  
SDPI:00481F4C                 push    offset strGetModuleName   ; relativeAddress  
SDPI:00481F51                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481F56                 push    eax                       ; specialAPI  
SDPI:00481F57                 push    edi                       ; CurrentAPI  
SDPI:00481F58                 call    StrCompare                ; 其中 CurrentAPI 指当然要操作的 API  
SDPI:00481F58                                                   ; SpecialAPI 指壳指的特殊 API  
SDPI:00481F5D                 test    eax, eax  
SDPI:00481F5F                 jnz     short loc\_481F72  
SDPI:00481F61                 push    offset unk\_407C04         ; relativeAddress  
SDPI:00481F66                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481F6B                 mov     \[esi\], eax  
SDPI:00481F6D                 jmp     loc\_481FF6  
SDPI:00481F72 ; ----------------------------------------------------------------------------  
SDPI:00481F72  
SDPI:00481F72 loc\_481F72:                                       ; CODE XREF: Disposal\_IMP+53E j  
SDPI:00481F72                 push    offset strGetVersion      ; relativeAddress  
SDPI:00481F77                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481F7C                 push    eax                       ; specialAPI  
SDPI:00481F7D                 push    edi                       ; CurrentAPI  
SDPI:00481F7E                 call    StrCompare                ; 其中 CurrentAPI 指当然要操作的 API  
SDPI:00481F7E                                                   ; SpecialAPI 指壳指的特殊 API  
SDPI:00481F83                 test    eax, eax  
SDPI:00481F85                 jnz     short loc\_481F95  
SDPI:00481F87                 push    407942h                   ; relativeAddress  
SDPI:00481F8C                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481F91                 mov     \[esi\], eax  
SDPI:00481F93                 jmp     short loc\_481FF6  
SDPI:00481F95 ; ----------------------------------------------------------------------------  
SDPI:00481F95  
SDPI:00481F95 loc\_481F95:                                       ; CODE XREF: Disposal\_IMP+564 j  
SDPI:00481F95                 push    offset strGetStartupInfo  ; relativeAddress  
SDPI:00481F9A                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481F9F                 push    eax                       ; specialAPI  
SDPI:00481FA0                 push    edi                       ; CurrentAPI  
SDPI:00481FA1                 call    StrCompare                ; 其中 CurrentAPI 指当然要操作的 API  
SDPI:00481FA1                                                   ; SpecialAPI 指壳指的特殊 API  
SDPI:00481FA6                 test    eax, eax  
SDPI:00481FA8                 jnz     short loc\_481FB8  
SDPI:00481FAA                 push    offset unk\_407A3A         ; relativeAddress  
SDPI:00481FAF                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00481FB4                 mov     \[esi\], eax  
SDPI:00481FB6                 jmp     short loc\_481FF6  
SDPI:00481FB8 ; ----------------------------------------------------------------------------  
SDPI:00481FB8  
SDPI:00481FB8 loc\_481FB8:                                       ; CODE XREF: Disposal\_IMP+587 j  
SDPI:00481FB8                 mov     edx, \[esp+444h+hdll\]  
SDPI:00481FBC                 push    edi                       ; APIName  
SDPI:00481FBD                 push    edx                       ; hdll  
SDPI:00481FBE                 call    API\_GetProcAddr  
SDPI:00481FC3                 push    eax                       ; APIAddress  
SDPI:00481FC4                 call    HOOK\_API\_JMP              ; 加密输入表,把 call api 搬到壳里去  
SDPI:00481FC9                 mov     \[esi\], eax                ; 填充运算后的加密地址  
SDPI:00481FCB                 jmp     short loc\_481FF6  
SDPI:00481FCD ; ----------------------------------------------------------------------------  
SDPI:00481FCD  
SDPI:00481FCD loc\_481FCD:                                       ; CODE XREF: Disposal\_IMP+3CF j  
SDPI:00481FCD                                                   ; Disposal\_IMP+3F5 j  
SDPI:00481FCD                 mov     eax, \[esp+444h+hdll\]  
SDPI:00481FD1                 push    edi                       ; APIName  
SDPI:00481FD2                 push    eax                       ; hdll  
SDPI:00481FD3                 call    API\_GetProcAddr           ; 获取 API 地址  
SDPI:00481FD8                 mov     \[esi\], eax                ; 保存 API,把特殊 DLL 处理改成非特殊 DLL  
SDPI:00481FD8                                                   ; 后直接到这里，然后 patch 下就可以获取  
SDPI:00481FD8                                                   ; 到完整的输入表了  
SDPI:00481FDA                 mov     ecx, \[esp+444h\]  
SDPI:00481FE1                 test    ecx, ecx  
SDPI:00481FE3                 jnz     short loc\_481FF6  
SDPI:00481FE5                 mov     ecx, \[esp+444h+var\_428\]  
SDPI:00481FE9                 test    ecx, ecx  
SDPI:00481FEB                 jz      short loc\_481FF6  
SDPI:00481FED                 mov     \[ecx\], eax                ; 把 api 保存到申请的空间里  
SDPI:00481FEF                 add     ecx, 4  
SDPI:00481FF2                 mov     \[esp+444h+var\_428\], ecx  
SDPI:00481FF6  
SDPI:00481FF6 loc\_481FF6:                                       ; CODE XREF: Disposal\_IMP+3EA j  
SDPI:00481FF6                                                   ; Disposal\_IMP+41C j ...  
SDPI:00481FF6                 xor     ecx, ecx  
SDPI:00481FF8                 mov     cx, \[ebx\]                 ; 如果要获取完整的输入表，  
SDPI:00481FF8                                                   ; 则这里不能让壳清除掉  
SDPI:00481FFB                 push    ecx                       ; int  
SDPI:00481FFC                 push    0FFh                      ; char  
SDPI:00482001                 push    edi                       ; LPCSTR  
SDPI:00482002                 call    Clear\_Data                ; 清除数据  
SDPI:00482007                 mov     word ptr \[ebx\], 0         ; 清除序号名  
SDPI:0048200C  
SDPI:0048200C loc\_48200C:                                       ; CODE XREF: Disposal\_IMP+346 j  
SDPI:0048200C                                                   ; Disposal\_IMP+36B j ...  
SDPI:0048200C                 cmp     ebp, esi                  ; 如果不相等则清除 ThunkValue  
SDPI:0048200E                 jz      short loc\_48201A  
SDPI:00482010                 push    4                         ; int  
SDPI:00482012                 push    0                         ; char  
SDPI:00482014                 push    ebp                       ; LPCSTR  
SDPI:00482015                 call    Clear\_Data                ; 擦除输入表信息  
SDPI:0048201A  
SDPI:0048201A loc\_48201A:                                       ; CODE XREF: Disposal\_IMP+5ED j  
SDPI:0048201A                 mov     ebx, \[esp+34h\]  
SDPI:0048201E                 mov     edi, \[esp+444h+hMem\_IMPVA\]  
SDPI:00482022                 add     ebp, 4  
SDPI:00482025                 add     esi, 4  
SDPI:00482028                 jmp     loc\_481CE7                ; 上面载入 DLL 后，这里填充当前 DLL 的加载函数  
SDPI:00482028                                                   ; 填充完跳去处理下一个 DLL  
SDPI:0048202D ; ----------------------------------------------------------------------------  
SDPI:0048202D  
SDPI:0048202D DoneCurrDll\_48202D:                               ; CODE XREF: Disposal\_IMP+2CA j  
SDPI:0048202D                 call    CRC\_480467  
SDPI:00482032                 mov     edx, \[edi+4\]  
SDPI:00482035                 mov     eax, \[esp+444h+var\_420\]   ; 当前 DLL 处理完毕则，清除 DLL 名  
SDPI:00482039                 push    edx                       ; int  
SDPI:0048203A                 push    0FFh                      ; char  
SDPI:0048203F                 push    eax                       ; LPCSTR  
SDPI:00482040                 call    Clear\_Data                ; 这里也是擦除输入表信息，  
SDPI:00482040                                                   ; 不能让壳清除掉输入表信息  
SDPI:00482045                 push    14h                       ; int  
SDPI:00482047                 push    0FFh                      ; char  
SDPI:0048204C                 push    edi                       ; LPCSTR  
SDPI:0048204D                 call    Clear\_Data                ; 清除数据  
SDPI:00482052                 add     edi, 14h  
SDPI:00482055                 mov     ebp, \[esp+2Ch\]  
SDPI:00482059                 mov     \[esp+444h+hMem\_IMPVA\], edi  
SDPI:0048205D                 jmp     loc\_481B4A                ; 获取输入表信息  
SDPI:00482062 ; ----------------------------------------------------------------------------  
SDPI:00482062  
SDPI:00482062 loc\_482062:                                       ; CODE XREF: Disposal\_IMP+130 j  
SDPI:00482062                 pop     edi                       ; 输入表处理完毕跳来这里  
SDPI:00482063                 pop     esi  
SDPI:00482064                 pop     ebp  
SDPI:00482065                 mov     eax, 1  
SDPI:0048206A                 pop     ebx  
SDPI:0048206B                 add     esp, 430h  
SDPI:00482071                 retn    4  
SDPI:00482071 Disposal\_IMP    endp ; sp = -4  
SDPI:00482071  
SDPI:00482071 ; -----------------------------------------------------------------------  
  
  
   
SDPI:0048233E ; ----------------------------------------------------------------------------  
SDPI:00482343 aGetprocesstimes db 'GetProcessTimes',0  
SDPI:00482353 aKernel32\_dll   db 'Kernel32.dll',0  
SDPI:00482360  
SDPI:00482360 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:00482360  
SDPI:00482360  
SDPI:00482360 Anti\_DBG\_482360 proc near                         ; CODE XREF: SDPI:0047AB09 j  
SDPI:00482360  
SDPI:00482360 var\_2C          = dword ptr -2Ch  
SDPI:00482360 var\_28          = dword ptr -28h  
SDPI:00482360 var\_24          = dword ptr -24h  
SDPI:00482360 var\_20          = dword ptr -20h  
SDPI:00482360 var\_14          = dword ptr -14h  
SDPI:00482360 var\_C           = dword ptr -0Ch  
SDPI:00482360  
SDPI:00482360                 sub     esp, 10h  
SDPI:00482363                 push    esi  
SDPI:00482364                 call    Get\_Version               ; 获取操作系统  
SDPI:00482369                 cmp     eax, 80000000h  
SDPI:0048236E                 jnb     short IsWin9x\_4823EE      ; 如果操作系统是 Win9x 则跳过处理  
SDPI:00482370                 push    offset strGetProcTimes    ; 获取进程时间  
SDPI:00482375                 call    GetAbsAddress             ; 获取实际地址  
SDPI:0048237A                 push    eax  
SDPI:0048237B                 push    offset strDllKer32        ; relativeAddress  
SDPI:00482380                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00482385                 push    eax                       ; APIName  
SDPI:00482386                 call    api\_LoadLibraryA  
SDPI:0048238B                 push    eax                       ; hdll  
SDPI:0048238C                 call    API\_GetProcAddr           ; 获取 GetProcessTimes 的地址  
SDPI:00482391                 mov     esi, eax  
SDPI:00482393                 test    esi, esi  
SDPI:00482395                 jz      short IsWin9x\_4823EE      ; 如果获取失败也跳过检测  
SDPI:00482397                 lea     eax, \[esp+18h+var\_14\]  
SDPI:0048239B                 lea     ecx, \[esp+18h+var\_14\]  
SDPI:0048239F                 push    eax  
SDPI:004823A0  
SDPI:004823A0 loc\_4823A0:  
SDPI:004823A0                 lea     edx, \[esp+1Ch+var\_14\]  
SDPI:004823A4                 push    ecx  
SDPI:004823A5                 lea     eax, \[esp+20h+var\_C\]  
SDPI:004823A9                 push    edx  
SDPI:004823AA                 push    eax                       ; 这里怎么放过 CC 检测了  
SDPI:004823AB                 call    apiGetCurProcess  
SDPI:004823B0                 push    eax                       ; 获取进程时间  
SDPI:004823B1                 call    esi                       ; /GetProcessTimes  
SDPI:004823B1                                                   ; |hProcess = FFFFFFFF  
SDPI:004823B1                                                   ; |lpCreationTime = 0012FFB8  
SDPI:004823B1                                                   ; |lpExitTime = 0012FFB0  
SDPI:004823B1                                                   ; |lpKernelTime = 0012FFB0  
SDPI:004823B1                                                   ; \\lpUserTime = 0012FFB0  
SDPI:004823B3                 test    eax, eax  
SDPI:004823B5                 jz      short IsWin9x\_4823EE      ; 如果获取系统时间失败则跳  
SDPI:004823B7  
SDPI:004823B7 loc\_4823B7:  
SDPI:004823B7                 lea     ecx, \[esp+4\]  
SDPI:004823BB                 push    ecx  
SDPI:004823BC                 call    apiGetsystasft            ; GetSystemTimeAsFileTime  
SDPI:004823C1                 mov     eax, \[esp+8\]  
SDPI:004823C5                 mov     ecx, \[esp+10h\]            ; 比较是否相同，如果相等则跳去取时间  
SDPI:004823C9                 sub     eax, ecx  
SDPI:004823CB                 jz      short loc\_4823F8          ; 这几个地方不能跳了  
SDPI:004823CB                                                   ; 当然你想搞清楚什么跟他跳没关系的  
SDPI:004823CD                 cmp     eax, 1  
SDPI:004823D0                 ja      short GetTick\_48240A  
SDPI:004823D2                 mov     eax, \[esp+18h+var\_14\]  
SDPI:004823D6                 mov     ecx, \[esp+18h+var\_C\]  
SDPI:004823DA                 shr     eax, 4  
SDPI:004823DD                 shr     ecx, 4  
SDPI:004823E0                 add     eax, 10000000h  
SDPI:004823E5                 sub     eax, ecx  
SDPI:004823E7                 cmp     eax, 1000000h  
SDPI:004823EC                 ja      short GetTick\_48240A  
SDPI:004823EE  
SDPI:004823EE IsWin9x\_4823EE:                                   ; CODE XREF: Anti\_DBG\_482360+E j  
SDPI:004823EE                                                   ; Anti\_DBG\_482360+35 j ...  
SDPI:004823EE                 mov     eax, 0D246534Fh  
SDPI:004823F3                 pop     esi  
SDPI:004823F4                 add     esp, 10h  
SDPI:004823F7                 retn  
SDPI:004823F8 ; -----------------------------------------------------------------------  
  
  
  
SDPI:00482414 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:00482414  
SDPI:00482414 ; 这里再次判断是否为单 CPU，  
SDPI:00482414 ; 如果是单 CPU 则等待前面创建两个新线程  
SDPI:00482414 ; 的动作结束  
SDPI:00482414  
SDPI:00482414 WaitThread      proc near                         ; CODE XREF: SDPI:0047DCF7 j  
SDPI:00482414  
SDPI:00482414 sysinfobuffer   = dword ptr -24h  
SDPI:00482414 NumberOfCpu     = dword ptr -14h  
SDPI:00482414  
SDPI:00482414                 sub     esp, 24h  
SDPI:00482417                 lea     eax, \[esp+24h+sysinfobuffer\]  
SDPI:0048241A                 push    esi  
SDPI:0048241B                 push    eax                       ; lpSystemInfo  
SDPI:0048241C                 call    GetSystemInfo  
SDPI:00482421                 cmp     \[esp+2Ch+NumberOfCpu\], 1  ; 判断是否只有一个 cpu  
SDPI:00482426                 jnz     short loc\_48245A  
SDPI:00482428                 call    Get\_BASE  
SDPI:0048242D                 mov     esi, eax  
SDPI:0048242F                 push    0AF113BFAh  
SDPI:00482434                 lea     ecx, \[esi+94h\]  
SDPI:0048243A                 push    ecx  
SDPI:0048243B                 call    SetNoDBG\_FLG\_4816FA  
SDPI:00482440  
SDPI:00482440 loc\_482440:                                       ; CODE XREF: WaitThread+44 j  
SDPI:00482440                 mov     eax, \[esi+80h\]  
SDPI:00482446                 test    eax, eax  
SDPI:00482448                 jz      short loc\_482451  
SDPI:0048244A                 mov     eax, \[esi+7Ch\]  
SDPI:0048244D                 test    eax, eax  
SDPI:0048244F                 jnz     short loc\_48245A  
SDPI:00482451  
SDPI:00482451 loc\_482451:                                       ; CODE XREF: WaitThread+34 j  
SDPI:00482451                 push    0                         ; dwMilliseconds  
SDPI:00482453                 call    Sleep  
SDPI:00482458                 jmp     short loc\_482440  
SDPI:0048245A ; ----------------------------------------------------------------------------  
SDPI:0048245A  
SDPI:0048245A loc\_48245A:                                       ; CODE XREF: WaitThread+12 j  
SDPI:0048245A                                                   ; WaitThread+3B j  
SDPI:0048245A                 pop     esi  
SDPI:0048245B                 add     esp, 24h  
SDPI:0048245E                 retn  
SDPI:0048245E WaitThread      endp ; sp = -4  
SDPI:0048245E  
SDPI:0048245F  
  
  
  
SDPI:0048245F ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0048245F  
SDPI:0048245F ; Attributes: bp-based frame  
SDPI:0048245F  
SDPI:0048245F isDebuggerPresent proc near                       ; CODE XREF: AntiDBG\_482535:CHECK\_R3D p  
SDPI:0048245F  
SDPI:0048245F var\_4           = dword ptr -4  
SDPI:0048245F  
SDPI:0048245F                 push    ebp  
SDPI:00482460                 mov     ebp, esp  
SDPI:00482462                 push    ecx  
SDPI:00482463                 push    eax  
SDPI:00482464                 push    ecx  
SDPI:00482465                 mov     eax, large fs:18h  
SDPI:0048246B                 mov     eax, \[eax+30h\]  
SDPI:0048246E                 mov     ecx, \[eax\]  
SDPI:00482470                 mov     \[ebp+var\_4\], ecx  
SDPI:00482473                 pop     ecx  
SDPI:00482474                 pop     eax  
SDPI:00482475                 mov     eax, \[ebp+var\_4\]  
SDPI:00482478                 shr     eax, 10h  
SDPI:0048247B                 and     eax, 1  
SDPI:0048247E                 mov     esp, ebp  
SDPI:00482480                 pop     ebp  
SDPI:00482481                 retn  
SDPI:00482481 isDebuggerPresent endp  
SDPI:00482481  
  
  
SDPI:0048248E ; ----------------------------------------------------------------------------  
SDPI:0048248F aUnhandledexceptionf db 'UnhandledExceptionFilter',0  
SDPI:004824A8 aDebuggerDetected db 'Debugger detected!',0  
SDPI:004824BB  
SDPI:004824BB ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:004824BB  
SDPI:004824BB ; 没多大作用检测,检测 UnhandleExceptxx  
SDPI:004824BB ; API 的第一个字节是否下断了  
SDPI:004824BB  
SDPI:004824BB CHK\_UnHndexcptCC proc near                        ; CODE XREF: SDPI:0047DCE1 j  
SDPI:004824BB                 push    offset strUnhandledExcept ; relativeAddress  
SDPI:004824C0                 call    GetAbsAddress             ; 获取实际地址  
SDPI:004824C5                 push    eax  
SDPI:004824C6                 push    offset strkerdll          ; relativeAddress  
SDPI:004824CB                 call    GetAbsAddress             ; 获取实际地址  
SDPI:004824D0                 push    eax                       ; APIName  
SDPI:004824D1                 call    api\_LoadLibraryA  
SDPI:004824D6                 push    eax                       ; hdll  
SDPI:004824D7                 call    API\_GetProcAddr  
SDPI:004824DC                 test    eax, eax                  ; 判断 Unxxx 是否被下断了  
SDPI:004824DE                 jz      short locret\_482510  
SDPI:004824E0                 cmp     byte ptr \[eax\], 0CCh  
SDPI:004824E3                 jnz     short locret\_482510  
SDPI:004824E5                 call    Decode\_47F088  
SDPI:004824EA                 push    0                         ; uType  
SDPI:004824EC                 push    offset strError           ; relativeAddress  
SDPI:004824F1                 call    GetAbsAddress             ; 获取实际地址  
SDPI:004824F6                 push    eax                       ; lpCaption  
SDPI:004824F7                 push    offset unk\_4094A8         ; relativeAddress  
SDPI:004824FC                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00482501                 push    eax                       ; lpText  
SDPI:00482502                 push    0                         ; hWnd  
SDPI:00482504                 call    MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:00482504                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:00482504                                                   ; 也就是判断有没有下 int3 断点  
SDPI:00482509                 push    9                         ; uExitCode  
SDPI:0048250B                 call    ExitProcess  
SDPI:00482510  
SDPI:00482510 locret\_482510:                                    ; CODE XREF: CHK\_UnHndexcptCC+23 j  
SDPI:00482510                                                   ; CHK\_UnHndexcptCC+28 j  
SDPI:00482510                 retn  
SDPI:00482510 CHK\_UnHndexcptCC endp ; sp = -4  
SDPI:00482510  
SDPI:00482510 ; -----------------------------------------------------------------------  
  
  
  
SDPI:00482510 ; ----------------------------------------------------------------------------  
SDPI:00482511 aZwqueryinformationp db 'ZwQueryInformationProcess',0  
SDPI:0048252B aNtdll\_dll      db 'ntdll.dll',0  
SDPI:00482535  
SDPI:00482535 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:00482535  
SDPI:00482535  
SDPI:00482535 AntiDBG\_482535  proc near                         ; CODE XREF: SDPI:0047ABC4 j  
SDPI:00482535  
SDPI:00482535 var\_18          = dword ptr -18h  
SDPI:00482535 var\_4           = dword ptr -4  
SDPI:00482535  
SDPI:00482535                 push    ecx  
SDPI:00482536                 call    Get\_Version  
SDPI:0048253B                 cmp     eax, 80000000h  
SDPI:00482540                 jnb     short isWin9x\_4825A5      ; 如果是 Win9x 则跳  
SDPI:00482542                 push    esi  
SDPI:00482543                 push    edi  
SDPI:00482544                 call    GetTickCount  
SDPI:00482549                 push    offset strZwQInfoProc     ; 获取 ZwQueryInformationProcess 的地址  
SDPI:0048254E                 mov     edi, eax  
SDPI:00482550                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00482555                 push    eax  
SDPI:00482556                 push    offset strdllNTDLL        ; relativeAddress  
SDPI:0048255B                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00482560                 push    eax                       ; APIName  
SDPI:00482561                 call    api\_LoadLibraryA  
SDPI:00482566                 push    eax                       ; hdll  
SDPI:00482567                 call    API\_GetProcAddr  
SDPI:0048256C                 mov     esi, eax  
SDPI:0048256E                 test    esi, esi                  ; 获取 ZwQueryInformationProcess 的地址  
SDPI:0048256E                                                   ; 如果获取失败则直接跳去 IsDebuggerPresent  
SDPI:0048256E                                                   ; 检测是否有 Ring3 级调试器  
SDPI:00482570                 jz      short CHECK\_R3D  
SDPI:00482572                 push    0  
SDPI:00482574                 lea     eax, \[esp+10h+var\_4\]  
SDPI:00482578                 push    4  
SDPI:0048257A                 push    eax  
SDPI:0048257B                 push    7  
SDPI:0048257D                 call    apiGetCurProcess  
SDPI:00482582                 push    eax  
SDPI:00482583                 call    esi                       ; 检测调试器是否存在  
SDPI:00482585                 test    eax, eax  
SDPI:00482587                 jnz     short CHECK\_R3D  
SDPI:00482589                 mov     eax, \[esp+20h+var\_18\]  
SDPI:0048258D                 test    eax, eax  
SDPI:0048258F                 jnz     short FoundDBG\_48259F     ; 如果检测到调试器则 over 了  
SDPI:00482591  
SDPI:00482591 CHECK\_R3D:                                        ; CODE XREF: AntiDBG\_482535+3B j  
SDPI:00482591                                                   ; AntiDBG\_482535+52 j  
SDPI:00482591                 call    isDebuggerPresent  
SDPI:00482596                 test    eax, eax  
SDPI:00482598                 jnz     short FoundDBG\_48259F  
SDPI:0048259A                 mov     edi, 80000000h            ; 没有检测到调试器则  
SDPI:0048259A                                                   ; mov eax,80000000h  
SDPI:0048259F  
SDPI:0048259F FoundDBG\_48259F:                                  ; CODE XREF: AntiDBG\_482535+5A j  
SDPI:0048259F                                                   ; AntiDBG\_482535+63 j  
SDPI:0048259F                 mov     eax, edi  
SDPI:004825A1                 pop     edi  
SDPI:004825A2                 pop     esi  
SDPI:004825A3                 pop     ecx  
SDPI:004825A4                 retn  
SDPI:004825A5 ; ----------------------------------------------------------------------------  
SDPI:004825A5  
  
  
體局駑#   @?  
SDPI:004825AB ; ----------------------------------------------------------------------------  
SDPI:004825AC aNtquerysysteminform db 'NtQuerySystemInformation',0  
SDPI:004825C5 aNtice\_sys      db 'ntice.sys',0  
SDPI:004825CF aIceext\_sys     db 'IceExt.sys',0  
SDPI:004825DA  
SDPI:004825DA ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:004825DA  
SDPI:004825DA ; 检测 RING 0 级调试器  
SDPI:004825DA ; 只检测 ICE 和 ICEEXT  
SDPI:004825DA ; 看来私下的秘密武器还是有  
SDPI:004825DA ; 很大作用^\_^  
SDPI:004825DA  
SDPI:004825DA CHK\_R0D\_4825DA  proc near                         ; CODE XREF: SDPI:0047BCB5 j  
SDPI:004825DA  
SDPI:004825DA var\_4A          = byte ptr -4Ah  
SDPI:004825DA var\_3A          = dword ptr -3Ah  
SDPI:004825DA var\_34          = dword ptr -34h  
SDPI:004825DA var\_28          = dword ptr -28h  
SDPI:004825DA var\_1C          = dword ptr -1Ch  
SDPI:004825DA var\_C           = dword ptr -0Ch  
SDPI:004825DA @HMODEL         = dword ptr -4  
SDPI:004825DA  
SDPI:004825DA                 sub     esp, 0Ch  
SDPI:004825DD                 push    ebx  
SDPI:004825DE                 push    ebp  
SDPI:004825DF                 call    Get\_BASE  
SDPI:004825E4                 mov     ebx, eax  
SDPI:004825E6                 mov     \[esp+14h+@HMODEL\], ebx  
SDPI:004825EA                 call    Get\_Version  
SDPI:004825EF                 cmp     eax, 80000000h  
SDPI:004825F4                 jnb     IsWin9X\_4826E8  
SDPI:004825FA                 push    offset strNtQuerySINFO    ; relativeAddress  
SDPI:004825FF                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00482604                 push    eax  
SDPI:00482605                 push    offset strdllNTDLL        ; relativeAddress  
SDPI:0048260A                 call    GetAbsAddress             ; 获取实际地址  
SDPI:0048260F                 push    eax                       ; APIName  
SDPI:00482610                 call    api\_LoadLibraryA  
SDPI:00482615                 push    eax                       ; hdll  
SDPI:00482616                 call    API\_GetProcAddr  
SDPI:0048261B                 mov     ebp, eax  
SDPI:0048261D                 test    ebp, ebp  
SDPI:0048261F                 jz      IsWin9X\_4826E8  
SDPI:00482625                 push    esi  
SDPI:00482626                 lea     eax, \[esp+1Ch+var\_C\]  
SDPI:0048262A                 push    edi  
SDPI:0048262B                 push    eax  
SDPI:0048262C                 lea     ecx, \[esp+24h+var\_C\]  
SDPI:00482630                 push    0  
SDPI:00482632                 push    ecx  
SDPI:00482633                 push    0Bh  
SDPI:00482635                 call    ebp  
SDPI:00482637                 mov     edx, \[esp+30h+var\_1C\]  
SDPI:0048263B                 lea     eax, ds:0\[edx\*4\]  
SDPI:00482642                 push    eax  
SDPI:00482643                 push    8  
SDPI:00482645                 call    GetProcessHeap  
SDPI:0048264A                 push    eax  
SDPI:0048264B                 call    allocateHeap  
SDPI:00482650                 mov     esi, eax  
SDPI:00482652                 push    0  
SDPI:00482654                 mov     ecx, \[esp+40h+var\_28\]  
SDPI:00482658                 lea     edx, ds:0\[ecx\*4\]  
SDPI:0048265F                 push    edx  
SDPI:00482660                 push    esi  
SDPI:00482661                 push    0Bh  
SDPI:00482663                 call    ebp  
SDPI:00482665                 mov     eax, \[esi\]  
SDPI:00482667                 xor     edi, edi  
SDPI:00482669                 test    eax, eax  
SDPI:0048266B                 jbe     short loc\_4826AB  
SDPI:0048266D                 lea     ebx, \[esi+20h\]  
SDPI:00482670  
SDPI:00482670 LP\_CHK\_R0D\_482670:                                ; CODE XREF: CHK\_R0D\_4825DA+CB j  
SDPI:00482670                 push    offset strntice\_sys       ; relativeAddress  
SDPI:00482675                 call    GetAbsAddress             ; 获取实际地址  
SDPI:0048267A                 push    eax                       ; lpString2  
SDPI:0048267B                 push    ebx                       ; lpString1  
SDPI:0048267C                 call    lstrcmpA                  ; 检测 RING 0 级 DEBUG，  
SDPI:0048267C                                                   ; sice 和 iceext  
SDPI:00482681                 test    eax, eax  
SDPI:00482683                 jnz     short fNDDBG\_4826D2  
SDPI:00482685                 push    offset strICEExt\_sys      ; relativeAddress  
SDPI:0048268A                 call    GetAbsAddress             ; 获取实际地址  
SDPI:0048268F                 push    eax                       ; lpString2  
SDPI:00482690                 push    ebx                       ; lpString1  
SDPI:00482691                 call    lstrcmpA  
SDPI:00482696                 test    eax, eax  
SDPI:00482698                 jnz     short fNDDBG\_4826D2  
SDPI:0048269A                 mov     eax, \[esi\]  
SDPI:0048269C                 inc     edi  
SDPI:0048269D                 add     ebx, 11Ch  
SDPI:004826A3                 cmp     edi, eax  
SDPI:004826A5                 jb      short LP\_CHK\_R0D\_482670   ; 通过 ZwQueryInformation,  
SDPI:004826A5                                                   ; 来循环检测 RING0 级驱动  
SDPI:004826A5                                                   ; 然后判断是否为 NTICE 或 ICEEXT 加载  
SDPI:004826A7                 mov     ebx, \[esp+4Ch+var\_34\]  
SDPI:004826AB  
SDPI:004826AB loc\_4826AB:                                       ; CODE XREF: CHK\_R0D\_4825DA+91 j  
SDPI:004826AB                 push    0  
SDPI:004826AD                 lea     eax, \[esp+50h+var\_3A\]  
SDPI:004826B1                 push    2  
SDPI:004826B3                 push    eax  
SDPI:004826B4                 push    23h  
SDPI:004826B6                 call    ebp  
SDPI:004826B8                 test    eax, eax  
SDPI:004826BA                 jnz     short loc\_4826C4  
SDPI:004826BC                 mov     al, \[esp+5Ch+var\_4A\]  
SDPI:004826C0                 test    al, al  
SDPI:004826C2                 jnz     short fNDDBG\_4826D2  
SDPI:004826C4  
SDPI:004826C4 loc\_4826C4:                                       ; CODE XREF: CHK\_R0D\_4825DA+E0 j  
SDPI:004826C4                 add     ebx, 78h  
SDPI:004826C7                 push    0F234543Eh  
SDPI:004826CC                 push    ebx  
SDPI:004826CD                 call    SetNoDBG\_FLG\_4816FA  
SDPI:004826D2  
SDPI:004826D2 fNDDBG\_4826D2:                                    ; CODE XREF: CHK\_R0D\_4825DA+A9 j  
SDPI:004826D2                                                   ; CHK\_R0D\_4825DA+BE j ...  
SDPI:004826D2                 push    esi  
SDPI:004826D3                 push    0  
SDPI:004826D5                 call    GetProcessHeap  
SDPI:004826DA                 push    eax  
SDPI:004826DB                 call    FreeHeap  
SDPI:004826E0                 pop     edi  
SDPI:004826E1                 pop     esi  
SDPI:004826E2                 pop     ebp  
SDPI:004826E3                 pop     ebx  
SDPI:004826E4                 add     esp, 0Ch  
SDPI:004826E7                 retn  
SDPI:004826E8 ; ----------------------------------------------------------------------------  
SDPI:004826E8  
SDPI:004826E8 IsWin9X\_4826E8:                                   ; CODE XREF: CHK\_R0D\_4825DA+1A j  
SDPI:004826E8                                                   ; CHK\_R0D\_4825DA+45 j  
SDPI:004826E8                 add     ebx, 78h  
SDPI:004826EB                 push    0F234543Eh  
SDPI:004826F0                 push    ebx  
SDPI:004826F1                 call    SetNoDBG\_FLG\_4816FA  
SDPI:004826F6                 pop     ebp  
SDPI:004826F7                 pop     ebx  
SDPI:004826F8                 add     esp, 0Ch  
SDPI:004826FB                 retn  
SDPI:004826FB CHK\_R0D\_4825DA  endp  
SDPI:004826FB  
SDPI:004826FB ; -----------------------------------------------------------------------  
  
  
  
SDPI:004826FB ; ----------------------------------------------------------------------------  
SDPI:004826FC aNoErrors       db 'No Errors !!',0Ah,0  
SDPI:0048270A  
SDPI:0048270A ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0048270A  
SDPI:0048270A ; 检测文件 MD5 值,判断文件是否被修改过  
SDPI:0048270A  
SDPI:0048270A File\_CRC\_CHK    proc near                         ; CODE XREF: CHKTARGET\_482BEF+77 p  
SDPI:0048270A                 push    esi  
SDPI:0048270B                 mov     esi, ecx  
SDPI:0048270D                 push    edi  
SDPI:0048270E                 mov     dword ptr \[esi+31Ch\], 0  
SDPI:00482718                 call    INT3\_47F261               ; 检测调试器是否存在  
SDPI:0048271D                 push    0  
SDPI:0048271F                 call    api\_GetModule  
SDPI:00482724                 push    eax  
SDPI:00482725                 mov     \[esi+8\], eax  
SDPI:00482728                 call    Get\_PEHEADER  
SDPI:0048272D                 lea     edi, \[esi+118h\]  
SDPI:00482733                 push    offset strNoErrors        ; relativeAddress  
SDPI:00482738                 mov     \[esi+0Ch\], eax  
SDPI:0048273B                 mov     \[esi+318h\], edi  
SDPI:00482741                 mov     dword ptr \[esi\], 209h  
SDPI:00482747                 call    GetAbsAddress             ; 获取实际地址  
SDPI:0048274C                 push    eax                       ; strIn  
SDPI:0048274D                 push    edi                       ; strout  
SDPI:0048274E                 call    wsprintfA  
SDPI:00482753                 add     esp, 8  
SDPI:00482756                 lea     eax, \[esi+14h\]  
SDPI:00482759                 push    104h                      ; nSize  
SDPI:0048275E                 push    eax                       ; lpFilename  
SDPI:0048275F                 push    0                         ; hModule  
SDPI:00482761                 call    GetModuleFileNameA  
SDPI:00482766                 push    10h                       ; size  
SDPI:00482768                 call    Get\_BASE  
SDPI:0048276D                 add     eax, 0A8h  
SDPI:00482772                 lea     ecx, \[esi+330h\]  
SDPI:00482778                 push    eax                       ; from  
SDPI:00482779                 push    ecx                       ; to  
SDPI:0048277A                 call    RtlMoveMemory             ; 复制内存  
SDPI:0048277F                 mov     ecx, esi  
SDPI:00482781                 call    OPNSELF\_4827F7            ; 打开可执行文件本身,  
SDPI:00482781                                                   ; 打开失败则提示错误信息  
SDPI:00482786                 mov     edx, \[esi+4\]              ; mov,EDX,hFile  
SDPI:00482789                 lea     edi, \[esi+320h\]  
SDPI:0048278F                 push    edx  
SDPI:00482790                 push    edi  
SDPI:00482791                 call    GetFileMD5Val\_47F542      ; 还要获取文件的 MD5 值:-(  
SDPI:00482796                 push    10h                       ; size  
SDPI:00482798                 push    edi                       ; from  
SDPI:00482799                 call    Get\_BASE  
SDPI:0048279E                 add     eax, 0F0h  
SDPI:004827A3                 push    eax                       ; to  
SDPI:004827A4                 call    RtlMoveMemory             ; 复制内存  
SDPI:004827A9                 push    10h                       ; size  
SDPI:004827AB                 push    edi                       ; Original\_value  
SDPI:004827AC                 call    Get\_BASE                  ; 文件正确 MD5 值保存在 EP+140H 处  
SDPI:004827AC                                                   ; 比如 EP 为 47A000 那么文件 MD5 值  
SDPI:004827AC                                                   ; 保存在 47A140 处。  
SDPI:004827AC                                                   ; 正确的内存检验值保存在 EP+150H 处  
SDPI:004827AC                                                   ; 比如 EP 为 47A000 那么内存 MD5 值  
SDPI:004827AC                                                   ; 保存在 47A150 处。  
SDPI:004827B1                 add     eax, 40h                  ; 比较文件的 MD5 值  
SDPI:004827B1                                                   ; 判断文件本身是否被修改过.  
SDPI:004827B1                                                   ; 比如 filepatch 等  
SDPI:004827B4                 push    eax                       ; Calculate\_value  
SDPI:004827B5                 call    Compare\_HASH              ; 把 Calculate\_value 改为 Original\_value  
SDPI:004827B5                                                   ; 就通过检测了  
SDPI:004827BA                 test    eax, eax  
SDPI:004827BC                 jz      short loc\_4827C8  
SDPI:004827BE                 mov     dword ptr \[esi+31Ch\], 1   ; 如果不相等，设置标志位  
SDPI:004827C8  
SDPI:004827C8 loc\_4827C8:                                       ; CODE XREF: File\_CRC\_CHK+B2 j  
SDPI:004827C8                 mov     eax, esi  
SDPI:004827CA                 pop     edi  
SDPI:004827CB                 pop     esi  
SDPI:004827CC                 retn  
SDPI:004827CC File\_CRC\_CHK    endp ; sp = -4  
SDPI:004827CC  
SDPI:004827CC ; -----------------------------------------------------------------------  
  
  
  
SDPI:004827CC ; ----------------------------------------------------------------------------  
SDPI:004827CD aCanNotOpenExecutabl db 'Can Not Open Executable "%s" its self !!',0Ah,0  
SDPI:004827F7  
SDPI:004827F7 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:004827F7  
SDPI:004827F7 ; 打开可执行文件本身,  
SDPI:004827F7 ; 打开失败则提示错误信息  
SDPI:004827F7  
SDPI:004827F7 OPNSELF\_4827F7  proc near                         ; CODE XREF: File\_CRC\_CHK+77 p  
SDPI:004827F7                 push    esi  
SDPI:004827F8                 push    edi  
SDPI:004827F9                 push    0                         ; hTemplateFile  
SDPI:004827FB                 mov     esi, ecx  
SDPI:004827FD                 push    10000000h                 ; dwFlagsAndAttributes  
SDPI:00482802                 push    3                         ; dwCreationDisposition  
SDPI:00482804                 push    0                         ; lpSecurityAttributes  
SDPI:00482806                 lea     edi, \[esi+14h\]  
SDPI:00482809                 push    1                         ; dwShareMode  
SDPI:0048280B                 push    80000000h                 ; dwDesiredAccess  
SDPI:00482810                 push    edi                       ; lpFileName  
SDPI:00482811                 call    CreateFileA  
SDPI:00482816                 cmp     eax, 0FFFFFFFFh  
SDPI:00482819                 mov     \[esi+4\], eax  
SDPI:0048281C                 jnz     short OPENOK\_482851       ; 保存 hFile  
SDPI:0048281E                 push    edi  
SDPI:0048281F                 push    offset strCanNotOpenExe   ; relativeAddress  
SDPI:00482824                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00482829                 push    eax                       ; strIn  
SDPI:0048282A                 mov     eax, \[esi+318h\]  
SDPI:00482830                 push    eax                       ; strout  
SDPI:00482831                 call    wsprintfA  
SDPI:00482836                 mov     ecx, \[esi+318h\]  
SDPI:0048283C                 add     esp, 0Ch  
SDPI:0048283F                 add     ecx, eax  
SDPI:00482841                 mov     dword ptr \[esi+31Ch\], 20h  
SDPI:0048284B                 mov     \[esi+318h\], ecx  
SDPI:00482851  
SDPI:00482851 OPENOK\_482851:                                    ; CODE XREF: OPNSELF\_4827F7+25 j  
SDPI:00482851                 mov     eax, \[esi+4\]              ; 保存 hFile  
SDPI:00482854                 pop     edi  
SDPI:00482855                 pop     esi  
SDPI:00482856                 retn  
SDPI:00482856 OPNSELF\_4827F7  endp ; sp = -1Ch  
  
  
   
  
SDPI:00482857 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:00482857  
SDPI:00482857  
SDPI:00482857 ClosehFile\_482857 proc near                       ; CODE XREF: CHKTARGET\_482BEF+1E7 p  
SDPI:00482857                 mov     eax, \[ecx+4\]  
SDPI:0048285A                 push    eax                       ; hObject  
SDPI:0048285B                 call    CloseHandle  
SDPI:00482860                 retn  
SDPI:00482860 ClosehFile\_482857 endp  
SDPI:00482860  
  
  
   
SDPI:00482867 ; ----------------------------------------------------------------------------  
SDPI:00482868 a\_Sice          db '\\\\.\\SICE',0  
SDPI:00482871 a\_Ntice         db '\\\\.\\NTICE',0  
SDPI:0048287B a\_Siwdebug      db '\\\\.\\SIWDEBUG',0  
SDPI:00482888 a\_Siwvid        db '\\\\.\\SIWVID',0  
SDPI:00482893 a\_Filemon       db '\\\\.\\FILEMON',0  
SDPI:0048289F a\_GlobalFilemon db '\\\\.\\Global\\FILEMON',0  
SDPI:004828B2 a\_Regmon        db '\\\\.\\REGMON',0  
SDPI:004828BD a\_GlobalRegmon  db '\\\\.\\Global\\REGMON',0  
SDPI:004828CF a\_Filevxd\_vxd   db '\\\\.\\FILEVXD.VXD',0  
SDPI:004828DF a\_Regvxd\_vxd    db '\\\\.\\REGVXD.VXD',0  
SDPI:004828EE a\_Trw           db '\\\\.\\TRW',0  
SDPI:004828F6 a\_Trwdebug      db '\\\\.\\TRWDEBUG',0  
SDPI:00482903 a\_Icedump       db '\\\\.\\ICEDUMP',0  
SDPI:0048290F a\_Frogsice      db '\\\\.\\FROGSICE',0  
SDPI:0048291C a\_Iceext        db '\\\\.\\IceExt',0  
SDPI:00482927 a\_Rvtracerdevice0 db '\\\\.\\RvtracerDevice0',0  
SDPI:0048293B  
SDPI:0048293B ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0048293B  
SDPI:0048293B ; 没什么特别的,CreateFile 来检测调试器  
SDPI:0048293B ; 无聊的做法  
SDPI:0048293B  
SDPI:0048293B CFCHKDBG\_48293B proc near                         ; CODE XREF: CHKTARGET\_482BEF+C0 p  
SDPI:0048293B  
SDPI:0048293B var\_7C          = dword ptr -7Ch  
SDPI:0048293B var\_60          = dword ptr -60h  
SDPI:0048293B var\_5C          = dword ptr -5Ch  
SDPI:0048293B var\_44          = dword ptr -44h  
SDPI:0048293B var\_40          = dword ptr -40h  
SDPI:0048293B var\_3C          = dword ptr -3Ch  
SDPI:0048293B var\_38          = dword ptr -38h  
SDPI:0048293B var\_34          = dword ptr -34h  
SDPI:0048293B var\_30          = dword ptr -30h  
SDPI:0048293B var\_2C          = dword ptr -2Ch  
SDPI:0048293B var\_28          = dword ptr -28h  
SDPI:0048293B var\_24          = dword ptr -24h  
SDPI:0048293B var\_20          = dword ptr -20h  
SDPI:0048293B var\_1C          = dword ptr -1Ch  
SDPI:0048293B var\_18          = dword ptr -18h  
SDPI:0048293B var\_14          = dword ptr -14h  
SDPI:0048293B var\_10          = dword ptr -10h  
SDPI:0048293B var\_C           = dword ptr -0Ch  
SDPI:0048293B var\_8           = dword ptr -8  
SDPI:0048293B var\_4           = dword ptr -4  
SDPI:0048293B  
SDPI:0048293B                 sub     esp, 44h  
SDPI:0048293E                 push    ebx  
SDPI:0048293F                 push    ebp  
SDPI:00482940                 push    esi  
SDPI:00482941                 push    edi  
SDPI:00482942                 mov     ebx, ecx  
SDPI:00482944                 push    offset strSICE            ; relativeAddress  
SDPI:00482949                 call    GetAbsAddress             ; 获取实际地址  
SDPI:0048294E                 push    offset NTICE              ; relativeAddress  
SDPI:00482953                 mov     \[esp+58h+var\_40\], eax  
SDPI:00482957                 call    GetAbsAddress             ; 获取实际地址  
SDPI:0048295C                 push    offset SIWDEBUG           ; relativeAddress  
SDPI:00482961                 mov     \[esp+58h+var\_3C\], eax  
SDPI:00482965                 call    GetAbsAddress             ; 获取实际地址  
SDPI:0048296A                 push    offset SIWVID             ; relativeAddress  
SDPI:0048296F                 mov     \[esp+58h+var\_38\], eax  
SDPI:00482973                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00482978                 push    offset FILEMON            ; relativeAddress  
SDPI:0048297D                 mov     \[esp+58h+var\_34\], eax  
SDPI:00482981                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00482986                 push    offset GlobalFILEMON      ; relativeAddress  
SDPI:0048298B                 mov     \[esp+58h+var\_30\], eax  
SDPI:0048298F                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00482994                 push    offset REGMON             ; relativeAddress  
SDPI:00482999                 mov     \[esp+58h+var\_2C\], eax  
SDPI:0048299D                 call    GetAbsAddress             ; 获取实际地址  
SDPI:004829A2                 push    offset GlobalREGMON       ; relativeAddress  
SDPI:004829A7                 mov     \[esp+58h+var\_28\], eax  
SDPI:004829AB                 call    GetAbsAddress             ; 获取实际地址  
SDPI:004829B0                 push    offset FILEVXD\_VXD        ; relativeAddress  
SDPI:004829B5                 mov     \[esp+58h+var\_24\], eax  
SDPI:004829B9                 call    GetAbsAddress             ; 获取实际地址  
SDPI:004829BE                 push    offset REGVXD\_VXD         ; relativeAddress  
SDPI:004829C3                 mov     \[esp+58h+var\_20\], eax  
SDPI:004829C7                 call    GetAbsAddress             ; 获取实际地址  
SDPI:004829CC                 push    offset TRW                ; relativeAddress  
SDPI:004829D1                 mov     \[esp+58h+var\_1C\], eax  
SDPI:004829D5                 call    GetAbsAddress             ; 获取实际地址  
SDPI:004829DA                 push    offset TRWDEBUG           ; relativeAddress  
SDPI:004829DF                 mov     \[esp+58h+var\_18\], eax  
SDPI:004829E3                 call    GetAbsAddress             ; 获取实际地址  
SDPI:004829E8                 push    offset ICEDUMP            ; relativeAddress  
SDPI:004829ED                 mov     \[esp+58h+var\_14\], eax  
SDPI:004829F1                 call    GetAbsAddress             ; 获取实际地址  
SDPI:004829F6                 push    offset FROGSICE           ; relativeAddress  
SDPI:004829FB                 mov     \[esp+58h+var\_10\], eax  
SDPI:004829FF                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00482A04                 push    offset IceExt             ; relativeAddress  
SDPI:00482A09                 mov     \[esp+58h+var\_C\], eax  
SDPI:00482A0D                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00482A12                 push    offset RvtracerDevice0    ; relativeAddress  
SDPI:00482A17                 mov     \[esp+58h+var\_8\], eax  
SDPI:00482A1B                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00482A20                 mov     \[esp+54h+var\_4\], eax  
SDPI:00482A24                 mov     \[esp+54h+var\_44\], 6  
SDPI:00482A2C                 mov     ebp, 3  
SDPI:00482A31  
SDPI:00482A31 loc\_482A31:                                       ; CODE XREF: CFCHKDBG\_48293B+154 j  
SDPI:00482A31                 xor     edi, edi  
SDPI:00482A33                 lea     esi, \[esp+54h+var\_40\]  
SDPI:00482A37  
SDPI:00482A37 loc\_482A37:                                       ; CODE XREF: CFCHKDBG\_48293B+149 j  
SDPI:00482A37                 mov     eax, \[esi\]  
SDPI:00482A39                 push    0                         ; hTemplateFile  
SDPI:00482A3B                 push    80h                       ; dwFlagsAndAttributes  
SDPI:00482A40                 push    ebp                       ; dwCreationDisposition  
SDPI:00482A41                 push    0                         ; lpSecurityAttributes  
SDPI:00482A43                 push    1                         ; dwShareMode  
SDPI:00482A45                 push    80000000h                 ; dwDesiredAccess  
SDPI:00482A4A                 push    eax                       ; lpFileName  
SDPI:00482A4B                 call    CreateFileA  
SDPI:00482A50                 cmp     eax, 0FFFFFFFFh  
SDPI:00482A53                 jz      short loc\_482A6D  
SDPI:00482A55                 cmp     edi, 4  
SDPI:00482A58                 jge     short loc\_482A65  
SDPI:00482A5A                 cmp     edi, 7  
SDPI:00482A5D                 jle     short loc\_482A65  
SDPI:00482A5F                 mov     \[ebx+31Ch\], ebp  
SDPI:00482A65  
SDPI:00482A65 loc\_482A65:                                       ; CODE XREF: CFCHKDBG\_48293B+11D j  
SDPI:00482A65                                                   ; CFCHKDBG\_48293B+122 j  
SDPI:00482A65                 push    eax                       ; hObject  
SDPI:00482A66                 call    CloseHandle  
SDPI:00482A6B                 jmp     short loc\_482A7D  
SDPI:00482A6D ; ----------------------------------------------------------------------------  
SDPI:00482A6D  
SDPI:00482A6D loc\_482A6D:                                       ; CODE XREF: CFCHKDBG\_48293B+118 j  
SDPI:00482A6D                 call    sub\_483A0E  
SDPI:00482A72                 cmp     eax, 5  
SDPI:00482A75                 jnz     short loc\_482A7D  
SDPI:00482A77                 mov     \[ebx+31Ch\], ebp  
SDPI:00482A7D  
SDPI:00482A7D loc\_482A7D:                                       ; CODE XREF: CFCHKDBG\_48293B+130 j  
SDPI:00482A7D                                                   ; CFCHKDBG\_48293B+13A j  
SDPI:00482A7D                 inc     edi  
SDPI:00482A7E                 add     esi, 4  
SDPI:00482A81                 cmp     edi, 10h  
SDPI:00482A84                 jb      short loc\_482A37  
SDPI:00482A86                 mov     eax, \[esp+54h+var\_44\]  
SDPI:00482A8A                 dec     eax  
SDPI:00482A8B                 mov     \[esp+54h+var\_44\], eax  
SDPI:00482A8F                 jnz     short loc\_482A31  
SDPI:00482A91                 call    IsDebuggerPresent  
SDPI:00482A96                 test    eax, eax  
SDPI:00482A98                 jz      short loc\_482AA0  
SDPI:00482A9A                 mov     \[ebx+31Ch\], ebp  
SDPI:00482AA0  
SDPI:00482AA0 loc\_482AA0:                                       ; CODE XREF: CFCHKDBG\_48293B+15D j  
SDPI:00482AA0                 mov     \[esp+54h+var\_44\], 7  
SDPI:00482AA8  
SDPI:00482AA8 loc\_482AA8:                                       ; CODE XREF: CFCHKDBG\_48293B+1D0 j  
SDPI:00482AA8                 xor     edi, edi  
SDPI:00482AAA                 lea     esi, \[esp+54h+var\_40\]  
SDPI:00482AAE  
SDPI:00482AAE loc\_482AAE:                                       ; CODE XREF: CFCHKDBG\_48293B+1C0 j  
SDPI:00482AAE                 mov     ecx, \[esi\]  
SDPI:00482AB0                 push    0                         ; hTemplateFile  
SDPI:00482AB2                 push    80h                       ; dwFlagsAndAttributes  
SDPI:00482AB7                 push    ebp                       ; dwCreationDisposition  
SDPI:00482AB8                 push    0                         ; lpSecurityAttributes  
SDPI:00482ABA                 push    1                         ; dwShareMode  
SDPI:00482ABC                 push    80000000h                 ; dwDesiredAccess  
SDPI:00482AC1                 push    ecx                       ; lpFileName  
SDPI:00482AC2                 call    CreateFileA  
SDPI:00482AC7                 cmp     eax, 0FFFFFFFFh  
SDPI:00482ACA                 jz      short loc\_482AE4  
SDPI:00482ACC                 cmp     edi, 4  
SDPI:00482ACF                 jge     short loc\_482ADC  
SDPI:00482AD1                 cmp     edi, 7  
SDPI:00482AD4                 jle     short loc\_482ADC  
SDPI:00482AD6                 mov     \[ebx+31Ch\], ebp  
SDPI:00482ADC  
SDPI:00482ADC loc\_482ADC:                                       ; CODE XREF: CFCHKDBG\_48293B+194 j  
SDPI:00482ADC                                                   ; CFCHKDBG\_48293B+199 j  
SDPI:00482ADC                 push    eax                       ; hObject  
SDPI:00482ADD                 call    CloseHandle  
SDPI:00482AE2                 jmp     short loc\_482AF4  
SDPI:00482AE4 ; ----------------------------------------------------------------------------  
SDPI:00482AE4  
SDPI:00482AE4 loc\_482AE4:                                       ; CODE XREF: CFCHKDBG\_48293B+18F j  
SDPI:00482AE4                 call    sub\_483A0E  
SDPI:00482AE9                 cmp     eax, 5  
SDPI:00482AEC                 jnz     short loc\_482AF4  
SDPI:00482AEE                 mov     \[ebx+31Ch\], ebp  
SDPI:00482AF4  
SDPI:00482AF4 loc\_482AF4:                                       ; CODE XREF: CFCHKDBG\_48293B+1A7 j  
SDPI:00482AF4                                                   ; CFCHKDBG\_48293B+1B1 j  
SDPI:00482AF4                 inc     edi  
SDPI:00482AF5                 add     esi, 4  
SDPI:00482AF8                 cmp     edi, 10h  
SDPI:00482AFB                 jb      short loc\_482AAE  
SDPI:00482AFD                 mov     ecx, \[ebx\]  
SDPI:00482AFF                 mov     eax, \[esp+70h+var\_60\]  
SDPI:00482B03                 inc     ecx  
SDPI:00482B04                 dec     eax  
SDPI:00482B05                 mov     \[ebx\], ecx  
SDPI:00482B07                 mov     \[esp+70h+var\_60\], eax  
SDPI:00482B0B                 jnz     short loc\_482AA8  
SDPI:00482B0D                 call    IsDebuggerPresent  
SDPI:00482B12                 test    eax, eax  
SDPI:00482B14                 jz      short loc\_482B1C  
SDPI:00482B16                 mov     \[ebx+31Ch\], ebp  
SDPI:00482B1C  
SDPI:00482B1C loc\_482B1C:                                       ; CODE XREF: CFCHKDBG\_48293B+1D9 j  
SDPI:00482B1C                 mov     eax, \[ebx\]  
SDPI:00482B1E                 pop     edi  
SDPI:00482B1F                 inc     eax  
SDPI:00482B20                 pop     esi  
SDPI:00482B21                 mov     \[ebx\], eax  
SDPI:00482B23                 pop     ebp  
SDPI:00482B24                 mov     eax, 1  
SDPI:00482B29                 pop     ebx  
SDPI:00482B2A                 add     esp, 44h  
SDPI:00482B2D                 retn  
SDPI:00482B2D CFCHKDBG\_48293B endp ; sp = -1Ch  
SDPI:00482B2D  
SDPI:00482B2D ; ---------------------------------------------------------------------  
  
  
  
SDPI:00482B2D ; ----------------------------------------------------------------------------  
SDPI:00482B2E aDebuggerFound  db 'Debugger found!',0  
SDPI:00482B3E aError          db 'Error',0  
SDPI:00482B44  
SDPI:00482B44 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:00482B44  
SDPI:00482B44 ; 这里检测上面检测后留下的标志，  
SDPI:00482B44 ; 如果发现标志，则提示错误信息.  
SDPI:00482B44 ; 标志 1 和 2 为时间的,3 为检测调试器的  
SDPI:00482B44  
SDPI:00482B44 chkflg\_482B44   proc near                         ; CODE XREF: CHKTARGET\_482BEF+109 p  
SDPI:00482B44                 push    esi  
SDPI:00482B45                 mov     esi, ecx  
SDPI:00482B47                 mov     eax, \[esi+31Ch\]  
SDPI:00482B4D                 cmp     eax, 1  
SDPI:00482B50                 jz      short OVER\_482B7C  
SDPI:00482B52                 cmp     eax, 2  
SDPI:00482B55                 jz      short OVER\_482B7C  
SDPI:00482B57                 cmp     eax, 3  
SDPI:00482B5A                 jnz     short loc\_482B70  
SDPI:00482B5C                 push    offset strdbgfnd\_409B2E   ; relativeAddress  
SDPI:00482B61                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00482B66                 push    eax  
SDPI:00482B67                 lea     eax, \[esi+118h\]  
SDPI:00482B6D                 push    eax  
SDPI:00482B6E                 jmp     short loc\_482B8E  
SDPI:00482B70 ; ----------------------------------------------------------------------------  
SDPI:00482B70  
SDPI:00482B70 loc\_482B70:                                       ; CODE XREF: chkflg\_482B44+16 j  
SDPI:00482B70                 cmp     eax, 4  
SDPI:00482B73                 jnz     short loc\_482B96  
SDPI:00482B75                 call    CRC\_480467  
SDPI:00482B7A                 jmp     short loc\_482B96  
SDPI:00482B7C ; ----------------------------------------------------------------------------  
SDPI:00482B7C  
SDPI:00482B7C OVER\_482B7C:                                      ; CODE XREF: chkflg\_482B44+C j  
SDPI:00482B7C                                                   ; chkflg\_482B44+11 j  
SDPI:00482B7C                 push    offset strFileisCorruped  ; relativeAddress  
SDPI:00482B81                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00482B86                 lea     ecx, \[esi+118h\]  
SDPI:00482B8C                 push    eax                       ; strIn  
SDPI:00482B8D                 push    ecx                       ; strout  
SDPI:00482B8E  
SDPI:00482B8E loc\_482B8E:                                       ; CODE XREF: chkflg\_482B44+2A j  
SDPI:00482B8E                 call    wsprintfA  
SDPI:00482B93                 add     esp, 8  
SDPI:00482B96  
SDPI:00482B96 loc\_482B96:                                       ; CODE XREF: chkflg\_482B44+2F j  
SDPI:00482B96                                                   ; chkflg\_482B44+36 j  
SDPI:00482B96                 mov     eax, \[esi+31Ch\]  
SDPI:00482B9C                 test    eax, eax  
SDPI:00482B9E                 jz      short loc\_482BC7  
SDPI:00482BA0                 call    Decode\_47F088  
SDPI:00482BA5                 push    0                         ; uType  
SDPI:00482BA7                 push    offset strError\_409B3E    ; relativeAddress  
SDPI:00482BAC                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00482BB1  
SDPI:00482BB1 loc\_482BB1:  
SDPI:00482BB1                 add     esi, 118h  
SDPI:00482BB7                 push    eax                       ; lpCaption  
SDPI:00482BB8                 push    esi                       ; lpText  
SDPI:00482BB9                 push    0                         ; hWnd  
SDPI:00482BBB                 call    MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:00482BBB                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:00482BBB                                                   ; 也就是判断有没有下 int3 断点  
SDPI:00482BC0                 push    0FFFFFFFFh                ; uExitCode  
SDPI:00482BC2                 call    ExitProcess  
SDPI:00482BC7  
SDPI:00482BC7 loc\_482BC7:                                       ; CODE XREF: chkflg\_482B44+5A j  
SDPI:00482BC7                 call    CRC\_480467  
SDPI:00482BCC                 pop     esi  
SDPI:00482BCD                 retn  
SDPI:00482BCD chkflg\_482B44   endp  
SDPI:00482BCD  
SDPI:00482BCD ; ----------------------------------------------------------------------  
  
  
?  
SDPI:00482BCD ; ----------------------------------------------------------------------------  
SDPI:00482BCE aInitializationTimeO db 'Initialization time out, exit...',0  
SDPI:00482BEF  
SDPI:00482BEF ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:00482BEF  
SDPI:00482BEF ; 检测目标文件是否被修改过  
SDPI:00482BEF ; 当中还有几个时间反调试  
SDPI:00482BEF  
SDPI:00482BEF CHKTARGET\_482BEF proc near                        ; CODE XREF: SDPI:0047DB46 p  
SDPI:00482BEF  
SDPI:00482BEF var\_3F8         = dword ptr -3F8h  
SDPI:00482BEF var\_3E4         = dword ptr -3E4h  
SDPI:00482BEF var\_3D0         = dword ptr -3D0h  
SDPI:00482BEF var\_3CC         = dword ptr -3CCh  
SDPI:00482BEF var\_3BC         = dword ptr -3BCh  
SDPI:00482BEF var\_3A8         = dword ptr -3A8h  
SDPI:00482BEF var\_394         = dword ptr -394h  
SDPI:00482BEF var\_390         = dword ptr -390h  
SDPI:00482BEF Text            = byte ptr -380h  
SDPI:00482BEF var\_37C         = dword ptr -37Ch  
SDPI:00482BEF var\_368         = dword ptr -368h  
SDPI:00482BEF CRCFILEFLG      = dword ptr -354h  
SDPI:00482BEF  
SDPI:00482BEF                 sub     esp, 380h  
SDPI:00482BF5                 push    esi  
SDPI:00482BF6                 push    edi  
SDPI:00482BF7                 push    offset strInitTimeOut     ; relativeAddress  
SDPI:00482BFC                 mov     edi, 201h                 ; 获取超时的字符串所在的位置  
SDPI:00482C01                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00482C06                 push    eax                       ; strIn  
SDPI:00482C07                 lea     eax, \[esp+38Ch+Text\]  
SDPI:00482C0B                 push    eax                       ; strout  
SDPI:00482C0C                 call    wsprintfA                 ; 格式化 Initialization time out, exit...  
SDPI:00482C11                 add     esp, 8  
SDPI:00482C14                 call    GetTickCount              ; 第一次获取时间保存在 ESI 中  
SDPI:00482C19                 mov     esi, eax  
SDPI:00482C1B                 call    CRC\_480467                ; 还对代码检测一次:-(  
SDPI:00482C20                 call    GetTickCount              ; 第二次取时间  
SDPI:00482C25                 test    esi, esi                  ; 判断前一次 GetTickCount  
SDPI:00482C25                                                   ; 返回的值是否为 0,也就是  
SDPI:00482C25                                                   ; 判断 API 是否被修改成了直接返回  
SDPI:00482C27                 jz      short OVER\_482C42  
SDPI:00482C29                 test    eax, eax                  ; 这里也是判断是否返回为 0  
SDPI:00482C2B                 jz      short OVER\_482C42  
SDPI:00482C2D                 mov     ecx, eax                  ; 判断第二次和第一次的时间差是否大于  
SDPI:00482C2D                                                   ; 1000ms,如果大于则 Over  
SDPI:00482C2F                 sub     ecx, esi  
SDPI:00482C31                 cmp     ecx, 1000  
SDPI:00482C37                 ja      short OVER\_482C42  
SDPI:00482C39                 mov     esi, eax                  ; 把第二次取的时间保存到 ESI 中  
SDPI:00482C3B                 mov     edi, 20Bh  
SDPI:00482C40                 jmp     short loc\_482C62  
SDPI:00482C42 ; ----------------------------------------------------------------------------  
SDPI:00482C42  
SDPI:00482C42 OVER\_482C42:                                      ; CODE XREF: CHKTARGET\_482BEF+38 j  
SDPI:00482C42                                                   ; CHKTARGET\_482BEF+3C j ...  
SDPI:00482C42                 push    0                         ; uType  
SDPI:00482C44                 push    offset strError\_409B3E    ; relativeAddress  
SDPI:00482C49                 call    GetAbsAddress             ; 获取实际地址  
SDPI:00482C4E                 lea     edx, \[esp+38Ch+Text\]  
SDPI:00482C52                 push    eax                       ; lpCaption  
SDPI:00482C53                 push    edx                       ; lpText  
SDPI:00482C54                 push    0                         ; hWnd  
SDPI:00482C56                 call    MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:00482C56                                                   ; 判断函数的前 5 位是否为 CC  
SDPI:00482C56                                                   ; 也就是判断有没有下 int3 断点  
SDPI:00482C5B                 push    1                         ; uExitCode  
SDPI:00482C5D                 call    ExitProcess  
SDPI:00482C62  
SDPI:00482C62 loc\_482C62:                                       ; CODE XREF: CHKTARGET\_482BEF+51 j  
SDPI:00482C62                 lea     ecx, \[esp+48h\]  
SDPI:00482C66                 call    File\_CRC\_CHK              ; 检测文件 MD5 值,判断文件是否被修改过  
SDPI:00482C6B                 call    GetTickCount  
SDPI:00482C70                 test    esi, esi                  ; 判断第二次的时间是否为 0  
SDPI:00482C70                                                   ; 是则 over  
SDPI:00482C72                 jz      short Over\_482C8B  
SDPI:00482C74                 test    eax, eax                  ; 判断第三次返回的时间是否为 0  
SDPI:00482C74                                                   ; 是则 over  
SDPI:00482C76                 jz      short Over\_482C8B  
SDPI:00482C78                 mov     ecx, eax                  ; 保存第三次时间到 ecx 中  
SDPI:00482C7A                 sub     ecx, esi  
SDPI:00482C7C                 cmp     ecx, 4000                 ; 比较延时是否超过 4 秒 4000ms  
SDPI:00482C82                 ja      short Over\_482C8B  
SDPI:00482C84                 mov     esi, eax                  ; 保存第三的时间到 esi 中  
SDPI:00482C86                 add     edi, 0Ah  
SDPI:00482C89                 jmp     short loc\_482CAB  
SDPI:00482C8B ; ----------------------------------------------------------------------------  
SDPI:00482C8B  
  
  
  
SDPI:0048424D ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SDPI:0048424D  
SDPI:0048424D ; invoke De\_Code,ecx(counter),eax(key),edx(end)  
SDPI:0048424D ; 用于解密代码，解密起始地址就是 call 进来  
SDPI:0048424D ; 的下一行代码地址  
SDPI:0048424D  
SDPI:0048424D De\_Code         proc near                         ; CODE XREF: SDPI:0047A3D2 p  
SDPI:0048424D                                                   ; SDPI:0047AAE1 p ...  
SDPI:0048424D  
SDPI:0048424D loop\_counter    = dword ptr -4  
SDPI:0048424D  
SDPI:0048424D                 nop  
SDPI:0048424E                 nop  
SDPI:0048424F                 nop  
SDPI:00484250                 nop  
SDPI:00484251                 nop  
SDPI:00484252                 nop  
SDPI:00484253                 nop  
SDPI:00484254                 nop  
SDPI:00484255                 nop  
SDPI:00484256                 nop  
SDPI:00484257                 nop  
SDPI:00484258                 nop  
SDPI:00484259                 nop  
SDPI:0048425A                 nop  
SDPI:0048425B                 nop  
SDPI:0048425C                 nop  
SDPI:0048425D                 nop  
SDPI:0048425E                 nop  
SDPI:0048425F                 nop  
SDPI:00484260                 nop  
SDPI:00484261                 nop  
SDPI:00484262                 nop  
SDPI:00484263                 nop  
SDPI:00484264                 nop  
SDPI:00484265                 nop  
SDPI:00484266                 nop  
SDPI:00484267                 nop  
SDPI:00484268                 nop  
SDPI:00484269                 nop  
SDPI:0048426A                 nop  
SDPI:0048426B                 nop  
SDPI:0048426C                 nop  
SDPI:0048426D                 push    ebx                       ; 把 key 的地址放到 ebx 中  
SDPI:0048426E                 push    esi  
SDPI:0048426F                 push    edi  
SDPI:00484270                 mov     ebx, eax  
SDPI:00484272                 mov     esi, \[esp+0Ch\]            ; 取出返回地址，也就是解密开始处  
SDPI:00484276                 push    ecx                       ; 循环次数入栈  
SDPI:00484277  
SDPI:00484277 loc\_484277:                                       ; CODE XREF: De\_Code+3C j  
SDPI:00484277                 mov     ecx, \[esp+4+loop\_counter\]  
SDPI:0048427A                 xor     edi, edi  
SDPI:0048427C  
SDPI:0048427C loc\_48427C:                                       ; CODE XREF: De\_Code+3A j  
SDPI:0048427C                 cmp     esi, edx                  ; 判断是否解密完成  
SDPI:0048427E                 jnb     short de\_done\_428b  
SDPI:00484280                 mov     al, \[edi+ebx\]             ; 进行简单的 xor 解密操作  
SDPI:00484280                                                   ; key 长度为 10h  
SDPI:00484280                                                   ; 如果取完了 10h 位就跳回开始处  
SDPI:00484283                 xor     \[esi\], al  
SDPI:00484285                 inc     esi  
SDPI:00484286                 inc     edi  
SDPI:00484287                 loop    loc\_48427C                ; 判断是否解密完成  
SDPI:00484289                 jmp     short loc\_484277  
SDPI:0048428B ; ----------------------------------------------------------------------------  
SDPI:0048428B  
SDPI:0048428B de\_done\_428b:                                     ; CODE XREF: De\_Code+31 j  
SDPI:0048428B                 call    loc\_484291  
SDPI:00484290                 nop  
SDPI:00484291  
SDPI:00484291 loc\_484291:                                       ; CODE XREF: De\_Code:de\_done\_428b p  
SDPI:00484291                 pop     eax  
SDPI:00484292                 add     eax, 0Dh  
SDPI:00484297                 push    eax  
SDPI:00484298                 retn  
SDPI:00484299 ; ----------------------------------------------------------------------------  
SDPI:00484299                 nop  
SDPI:0048429A                 nop  
SDPI:0048429B                 nop  
SDPI:0048429C                 nop  
SDPI:0048429D                 pop     ecx  
SDPI:0048429E                 pop     edi  
SDPI:0048429F                 pop     esi  
SDPI:004842A0                 pop     ebx  
SDPI:004842A1                 retn  
SDPI:004842A1 De\_Code         endp  
SDPI:004842A1  
SDPI:004842A1 ; --------------------------------------------------------------------  
OK 到这里分析完毕!回头看看又感觉好像其实也没有什么特别，还有很多地方写的不好。那个抽代码和擦除文件头看起来就是那么鸡肋，值的注意一下就是那个 CreateThread，如果你不想办法去掉那两个线程的话对后面修改代码来说就不是那么方便了。越时感觉没有什么导致我的文章也龙头虎尾一样。。总的来的说这个壳除了时间差检测外，其它方面没有什么。  
这也是我开头说有所失望的原因。  
  
LoVe WeN  
  
Greetz:  
 Fly.Jingulong,yock,tDasm.David.hexer,hmimys,ahao.UFO(brother).alan(sister).all of my friends and you!  
  
By loveboom\[DFCG\]\[FCG\]\[US\]  
http://blog.csdn.net/bmd2chen  
Email:loveboom#163.com  
Date:12/22/2005 3:47:00 AM  

 4dd466d0e1e0cc123afbe504da2462ca