

# MyVM\_#1\_KeygenMe\_by\_BUBlic 虚拟机粗略分析记录

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-560.htm`](https://www.pediy.com/kssd/pediy09/pediy09-560.htm)

> 【文章标题】: MyVM\_#1\_KeygenMe\_by\_BUBlic 虚拟机粗略分析记录  
> 【文章作者】: layper  
> 【作者邮箱】: layper@yahoo.com.cn  
> 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
> \--------------------------------------------------------------------------------  
> 【详细过程】  
>   分析虚拟机确实非常痛苦的事,虽然 MyVM\_#1\_KeygenMe\_by\_BUBlic 虚拟机比较简单,分析它我还是费了九牛二虎之力.  
>   注:  
>   水平有限,而且是第一次分析虚拟机,错漏难免.  
>   由于没有分析出算法,不涉及算法部分.  
>   一 call 401000:  
>   这个 call 是 crack 主体部分,分析如下:  
>   .text:00401000 sub\_401000      proc near               ; CODE XREF: DialogFunc:loc\_4011B5p  
>   .text:00401000  
>   .text:00401000 var\_C           = dword ptr -0Ch  
>   .text:00401000 var\_8           = dword ptr -8  
>   .text:00401000 var\_1           = byte ptr -1  
>   .text:00401000  
>   .text:00401000                 push    ebp  
>   .text:00401001                 mov     ebp, esp  
>   .text:00401003                 sub     esp, 0Ch  
>   .text:00401006                 push    ebx  
>   .text:00401007                 push    esi  
>   .text:00401008                 push    edi  
>   .text:00401009                 mov     edi, ds:GetDlgItemTextA  
>   .text:0040100F                 mov     ebx, 0FFh  
>   .text:00401014                 push    ebx             ; cchMax  
>   .text:00401015                 mov     esi, offset NameString ; esi 为注册名  
>   .text:0040101A                 push    esi             ; lpString  
>   .text:0040101B                 push    3EBh            ; nIDDlgItem  
>   .text:00401020                 push    hDlg            ; hDlg  
>   .text:00401026                 call    edi ; GetDlgItemTextA  
>   .text:00401026  
>   .text:00401028                 push    ebx             ; cchMax  
>   .text:00401029                 mov     ebx, offset SerialString ; ebx 为注册码  
>   .text:0040102E                 push    ebx             ; lpString  
>   .text:0040102F                 push    3E9h            ; nIDDlgItem  
>   .text:00401034                 push    hDlg            ; hDlg  
>   .text:0040103A                 call    edi ; GetDlgItemTextA  
>   .text:0040103A  
>   .text:0040103C                 push    esi             ; Str  
>   .text:0040103D                 call    \_strlen  
>   .text:0040103D  
>   .text:00401042                 mov     edi, eax  
>   .text:00401044                 push    ebx             ; Str  
>   .text:00401045                 mov     \[ebp+var\_C\], edi ; 用户名长度  
>   .text:00401048                 call    \_strlen  
>   .text:00401048  
>   .text:0040104D                 add     edi, 0FFFFFFFDh  
>   .text:00401050                 cmp     edi, 33h        ; 比较名称长度是否大于 33h  
>   .text:00401053                 pop     ecx  
>   .text:00401054                 pop     ecx  
>   .text:00401055                 mov     \[ebp+var\_8\], eax ; eax 为注册码长度  
>   .text:00401058                 ja      NameError  
>   .text:00401058  
>   .text:0040105E                 mov     ecx, eax  
>   .text:00401060                 and     ecx, 80000001h  ; 与运算  
>   .text:00401066                 jns     short loc\_40106D ; sf=0 则跳转  
>   .text:00401066  
>   .text:00401068                 dec     ecx             ; 减 1  
>   .text:00401069                 or      ecx, 0FFFFFFFEh ; 或运算  
>   .text:0040106C                 inc     ecx             ; 加一  
>   .text:0040106C  
>   .text:0040106D  
>   .text:0040106D loc\_40106D:                             ; CODE XREF: sub\_401000+66j  
>   .text:0040106D                 jnz     Wrongserial     ; ZF 不等于 0 则出错  
>   .text:0040106D  
>   .text:00401073                 cmp     eax, 10h        ; 注册码运算后的长度是否小于 10h,小于则出错  
>   .text:00401076                 jl      Wrongserial  
>   .text:00401076  
>   .text:0040107C                 xor     edi, edi        ; 清除  
>   .text:0040107E                 test    eax, eax  
>   .text:00401080                 mov     ebx, offset dword\_40FE88  
>   .text:00401085                 jz      short loc\_4010AC ; eax 为 0 则跳  
>   .text:00401085  
>   .text:00401087  
>   .text:00401087 loc\_401087:                             ; CODE XREF: sub\_401000+AAj  
>   .text:00401087                 mov     eax, edi        ; 第一次先清 0  
>   .text:00401089                 cdq                     ; 把 EDX 的所有位都设成 EAX 最高位的值  
>   .text:00401089                                         ; 当 EAX <80000000, EDX 00000000;  
>   .text:00401089                                         ; 当 EAX >= 80000000, EDX 则为 FFFFFFFF).  
>   .text:0040108A                 sub     eax, edx        ; eax-0 或者 eax-(-1)  
>   .text:0040108C                 sar     eax, 1          ; 左移一位  
>   .text:0040108E                 add     eax, ebx        ; 第一次:40FE88+eax  
>   .text:00401090                 push    eax  
>   .text:00401091                 lea     eax, SerialString\[edi\] ; 注册码的\[edi\]位  
>   .text:00401097                 push    offset s\_2x     ; "%2X"  
>   .text:0040109C                 push    eax             ; char \*  
>   .text:0040109D                 call    \_sscanf         ; 执行从字符串中的格式化  
>   .text:0040109D  
>   .text:004010A2                 add     esp, 0Ch        ; esp 退后 10 位  
>   .text:004010A5                 inc     edi  
>   .text:004010A6                 inc     edi             ; edi 加 2  
>   .text:004010A7                 cmp     edi, \[ebp+var\_8\] ; 与注册码长度比较  
>   .text:004010AA                 jnz     short loc\_401087 ; 不等循环  
>   .text:004010AA  
>   .text:004010AC  
>   .text:004010AC loc\_4010AC:                             ; CODE XREF: sub\_401000+85j  
>   .text:004010AC                 push    480h            ; Size  
>   .text:004010B1                 call    operator new(uint) ; 分配 480h 内存  
>   .text:004010B1  
>   .text:004010B6                 xor     edi, edi  
>   .text:004010B8                 cmp     eax, edi        ; 分配返回地址  
>   .text:004010BA                 pop     ecx  
>   .text:004010BB                 jz      short loc\_4010D2 ; 为 0 则跳走  
>   .text:004010BB  
>   .text:004010BD                 mov     edx, offset unk\_40E000  
>   .text:004010C2                 push    edx             ; int  
>   .text:004010C3                 push    0BC1h           ; Size  
>   .text:004010C8                 push    edx             ; int  
>   .text:004010C9                 mov     ecx, eax        ; 分配返回地址  
>   .text:004010CB                 call    sub\_401BDC      ; arg\_0 与 arg\_8 初始值为 40E000  
>   .text:004010CB                                         ; 使用这个函数后  
>   .text:004010CB                                         ; esi,edi,ecx 与进入是一样  
>   .text:004010CB                                         ; esi 为注册名,edi=0  
>   .text:004010CB                                         ; eax 为第一次分配返回地址  
>   .text:004010CB                                         ; \[第一次分配返回地址+?\]填充了大量函数偏移  
>   .text:004010CB                                         ; \[第一次分配返回地址+48h\]为 1000h 大小,  
>   .text:004010CB                                         ; \[第一次分配返回地址+4Ch\]取\[40E000+20h\]值  
>   .text:004010CB  
>   .text:004010D0                 jmp     short loc\_4010D4  
>   .text:004010D0  
>   .text:004010D2 ; ---------------------------------------------------------------------------  
>   .text:004010D2  
>   .text:004010D2 loc\_4010D2:                             ; CODE XREF: sub\_401000+BBj  
>   .text:004010D2                 xor     eax, eax  
>   .text:004010D2  
>   .text:004010D4  
>   .text:004010D4 loc\_4010D4:                             ; CODE XREF: sub\_401000+D0j  
>   .text:004010D4                 lea     ecx, \[ebp+var\_1\] ; ecx 指向 var\_1  
>   .text:004010D7                 push    ecx  
>   .text:004010D8                 push    ebx  
>   .text:004010D9                 push    \[ebp+var\_C\]     ; 用户名长度  
>   .text:004010DC                 mov     \[ebp+var\_1\], 1  
>   .text:004010E0                 push    esi             ; 注册名  
>   .text:004010E1                 push    edi             ; 0  
>   .text:004010E2                 push    eax             ; eax 为第一次分配返回地址  
>   .text:004010E3                 call    sub\_40156A  
>   .text:004010E3  
>   .text:004010E8                 add     esp, 18h  
>   .text:004010EB                 cmp     \[ebp+var\_1\], 0  
>   .text:004010EF                 push    edi  
>   .text:004010F0                 jnz     short GootOK  
>   .text:004010F0  
>   .text:004010F2                 push    offset s\_BadBoy ; "Bad boy"  
>   .text:004010F7                 push    offset s\_KeepOnTrying ; "Keep on trying!"  
>   .text:004010FC                 jmp     short UseMessageBoxA  
>   .text:004010FC  
>   .text:004010FE ; ---------------------------------------------------------------------------  
>   .text:004010FE  
>   .text:004010FE GootOK:                                 ; CODE XREF: sub\_401000+F0j  
>   .text:004010FE                 push    offset s\_GoodBoy ; "Good boy"  
>   .text:00401103                 push    offset s\_YouDidIt ; "You did it!"  
>   .text:00401108                 jmp     short UseMessageBoxA  
>   .text:00401108  
>   .text:0040110A ; ---------------------------------------------------------------------------  
>   .text:0040110A  
>   .text:0040110A Wrongserial:                            ; CODE XREF: sub\_401000:loc\_40106Dj  
>   .text:0040110A                                         ; sub\_401000+76j  
>   .text:0040110A                 push    10h  
>   .text:0040110C                 push    offset Caption  ; "ERROR"  
>   .text:00401111                 push    offset s\_WrongSerial ; "Wrong serial!"  
>   .text:00401116                 jmp     short UseMessageBoxA  
>   .text:00401116  
>   .text:00401118 ; ---------------------------------------------------------------------------  
>   .text:00401118  
>   .text:00401118 NameError:                              ; CODE XREF: sub\_401000+58j  
>   .text:00401118                 push    10h             ; uType  
>   .text:0040111A                 push    offset Caption  ; "ERROR"  
>   .text:0040111F                 push    offset Text     ; "Use some other name!"  
>   .text:0040111F  
>   .text:00401124  
>   .text:00401124 UseMessageBoxA:                         ; CODE XREF: sub\_401000+FCj  
>   .text:00401124                                         ; sub\_401000+108j  
>   .text:00401124                                         ; sub\_401000+116j  
>   .text:00401124                 push    hDlg            ; hWnd  
>   .text:0040112A                 call    ds:MessageBoxA  
>   .text:0040112A  
>   .text:00401130                 pop     edi  
>   .text:00401131                 pop     esi  
>   .text:00401132                 pop     ebx  
>   .text:00401133                 leave  
>   .text:00401134                 retn  
>   .text:00401134  
>   .text:00401134 sub\_401000      endp  
>   .text:00401134  
>     
>   二、VM 初始化  
>   这个 VM 是"非标准"的虚拟机,初始化时它所做的主要是把 VM 指令指针保存在一个分配好的内存地址中,为以下调用作准备.  
>     
>   .text:00401BDC ; arg\_0 与 arg\_8 初始值为 40E000  
>   .text:00401BDC ; 使用这个函数后  
>   .text:00401BDC ; esi,edi,ecx 与进入是一样  
>   .text:00401BDC ; esi 为注册名,edi=0  
>   .text:00401BDC ; eax 为第一次分配返回地址  
>   .text:00401BDC ; \[第一次分配返回地址+?\]填充了大量函数偏移  
>   .text:00401BDC ; \[第一次分配返回地址+48h\]为 1000h 大小,  
>   .text:00401BDC ; \[第一次分配返回地址+4Ch\]取\[40E000+20h\]值  
>   .text:00401BDC  
>   .text:00401BDC ; int \_\_stdcall sub\_401BDC(int,size\_t Size,int)  
>   .text:00401BDC sub\_401BDC      proc near               ; CODE XREF: sub\_401000+CBp  
>   .text:00401BDC  
>   .text:00401BDC arg\_0           = dword ptr  4  
>   .text:00401BDC Size            = dword ptr  8  
>   .text:00401BDC arg\_8           = dword ptr  0Ch  
>   .text:00401BDC  
>   .text:00401BDC                 push    esi  
>   .text:00401BDD                 push    edi  
>   .text:00401BDE                 push    24h             ; Size  
>   .text:00401BE0                 mov     esi, ecx        ; 分配返回地址  
>   .text:00401BE2                 call    operator new(uint)  
>   .text:00401BE2  
>   .text:00401BE7                 mov     edi, 1000h  
>   .text:00401BEC                 push    edi             ; Size  
>   .text:00401BED                 mov     \[esi+7Ch\], eax  
>   .text:00401BF0                 call    operator new(uint)  
>   .text:00401BF0  
>   .text:00401BF5                 mov     \[esi+40h\], edi  
>   .text:00401BF8                 mov     edi, \[esp+10h+Size\]  
>   .text:00401BFC                 push    edi             ; Size  
>   .text:00401BFD                 mov     \[esi+3Ch\], eax  
>   .text:00401C00                 call    operator new(uint)  
>   .text:00401C00  
>   .text:00401C05                 push    edi  
>   .text:00401C06                 push    \[esp+18h+arg\_0\] ; 40E000+18h  
>   .text:00401C0A                 mov     \[esi+44h\], eax  
>   .text:00401C0D                 push    eax  
>   .text:00401C0E                 call    unknown\_libname\_1 ; Microsoft VisualC 2-8/net runtime  
>   .text:00401C0E  
>   .text:00401C13                 mov     eax, \[esp+20h+arg\_8\] ; 取\[40E000+20h\]  
>   .text:00401C17                 add     esp, 18h  
>   .text:00401C1A                 mov     \[esi+48h\], edi  ; edi 为 1000h 大小,  
>   .text:00401C1A                                         ; esi 为第一次分配返回地址  
>   .text:00401C1D                 mov     \[esi+4Ch\], eax  ; 取\[40E000+20h\]  
>   .text:00401C20                 pop     edi  
>   .text:00401C21                 mov     dword ptr \[esi+80h\], offset VMEip3  
>   .text:00401C2B                 mov     dword ptr \[esi+84h\], offset VMAdd  
>   .text:00401C35                 mov     dword ptr \[esi+88h\], offset VMMOV  
>   .text:00401C3F                 mov     dword ptr \[esi+8Ch\], offset VMMov\_1  
>   .text:00401C49                 mov     dword ptr \[esi+90h\], offset VMMov1 ;  
>   .text:00401C49                                         ; \[ebp+var\_4\]指向\[\[esi+20h\]\]  
>   .text:00401C49                                         ; \[esi+20h\]加 4  
>   .text:00401C53                 mov     dword ptr \[esi+94h\], offset VMAND\_DOWRD  
>   .text:00401C5D                 mov     dword ptr \[esi+98h\], offset VMMOV  
>   .text:00401C67                 mov     dword ptr \[esi+9Ch\], offset VMImul  
>   .text:00401C71                 mov     dword ptr \[esi+0A0h\], offset VMDiv  
>   .text:00401C7B                 mov     dword ptr \[esi+0A4h\], offset VMMOV\_BYTE  
>   .text:00401C85                 mov     dword ptr \[esi+0A8h\], offset VMSub  
>   .text:00401C8F                 mov     dword ptr \[esi+0ACh\], offset VMFlag  
>   .text:00401C99                 mov     dword ptr \[esi+0B0h\], offset VMNextEip1  
>   .text:00401CA3                 mov     dword ptr \[esi+0B4h\], offset VMEipAddress  
>   .text:00401CAD                 mov     dword ptr \[esi+0B8h\], offset VMEipAddress1  
>   .text:00401CB7                 mov     dword ptr \[esi+0BCh\], offset VMEipAddress\_1  
>   .text:00401CC1                 mov     dword ptr \[esi+0C0h\], offset VMEipAddress\_0  
>   .text:00401CCB                 mov     dword ptr \[esi+0C4h\], offset VMNextEip  
>   .text:00401CD5                 mov     dword ptr \[esi+0C8h\], offset VMNextEip2  
>   .text:00401CDF                 mov     dword ptr \[esi+0CCh\], offset VMShl  
>   .text:00401CE9                 mov     dword ptr \[esi+0D0h\], offset VMShr  
>   .text:00401CF3                 mov     dword ptr \[esi+0D4h\], offset VMAnd  
>   .text:00401CFD                 mov     dword ptr \[esi+0D8h\], offset VMOr  
>   .text:00401D07                 mov     dword ptr \[esi+0DCh\], offset VMXor  
>   .text:00401D11                 mov     dword ptr \[esi+0E0h\], offset VMNot  
>   .text:00401D1B                 mov     dword ptr \[esi+0E4h\], offset VMEipAddress\_0  
>   .text:00401D25                 mov     dword ptr \[esi+0E8h\], offset VMEipAddress\_2  
>   .text:00401D2F                 mov     dword ptr \[esi+0ECh\], offset VMEipAddress\_2  
>   .text:00401D39                 mov     dword ptr \[esi+0F0h\], offset VMMOV  
>   .text:00401D43                 mov     dword ptr \[esi+0F4h\], offset VMFlag1  
>   .text:00401D4D                 mov     eax, esi        ; eax 为第一次分配返回地址  
>   .text:00401D4F                 pop     esi  
>   .text:00401D50                 retn    0Ch  
>   .text:00401D50  
>   .text:00401D50 sub\_401BDC      endp  
>     
>   可以看出,esi 作为指针地址寄存器,指向一个结构或者说是一个表,所以在下面我们就要注意\[esi+xx\]的代码.  
>     
>   三 VM 执行循环  
>   .text:0040156A sub\_40156A      proc near               ; CODE XREF: sub\_401000+E3p  
>   .text:0040156A  
>   .text:0040156A arg\_0           = dword ptr  4  
>   .text:0040156A arg\_4           = byte ptr  8  
>   .text:0040156A arg\_8           = byte ptr  0Ch  
>   .text:0040156A  
>   .text:0040156A                 push    ebx  
>   .text:0040156B                 push    esi  
>   .text:0040156C                 mov     esi, \[esp+8+arg\_0\]  
>   .text:00401570                 push    edi  
>   .text:00401571                 xor     ebx, ebx  
>   .text:00401573                 lea     edi, \[esp+0Ch+arg\_4\] ; 指向 var\_1  
>   .text:00401577                 mov     \[esi+79h\], bl  
>   .text:0040157A                 sub     edi, 8  
>   .text:0040157D                 mov     eax, \[edi\]  
>   .text:0040157F                 push    30h             ; Size  
>   .text:00401581                 mov     \[esi+70h\], eax  
>   .text:00401584                 lea     eax, \[esi+4\]  
>   .text:00401587                 push    ebx             ; Val  
>   .text:00401588                 push    eax             ; Dst  
>   .text:00401589                 call    \_memset         ; 对内存块的每个字节初始化,  
>   .text:00401589  
>   .text:0040158E                 push    5               ; Size  
>   .text:00401590                 lea     eax, \[esi+34h\]  
>   .text:00401593                 push    ebx             ; Val  
>   .text:00401594                 push    eax             ; Dst  
>   .text:00401595                 call    \_memset         ; 用来对一段内存空间全部设置为某个字符，  
>   .text:00401595                                         ; 一般用在对定义的字符串进行初始化为  
>   .text:00401595                                         ; ‘ ’或‘\\0’  
>   .text:00401595  
>   .text:0040159A                 push    24h             ; Size  
>   .text:0040159C                 push    ebx             ; Val  
>   .text:0040159D                 push    dword ptr \[esi+7Ch\] ; Dst  
>   .text:004015A0                 call    \_memset         ; 用来对一段内存空间全部设置为某个字符，  
>   .text:004015A0                                         ; 一般用在对定义的字符串进行初始化为  
>   .text:004015A0                                         ; ‘ ’或‘\\0’  
>   .text:004015A0  
>   .text:004015A5                 push    1000h           ; Size  
>   .text:004015AA                 push    ebx             ; Val  
>   .text:004015AB                 push    dword ptr \[esi+3Ch\] ; Dst  
>   .text:004015AE                 call    \_memset         ; 用来对一段内存空间全部设置为某个字符，  
>   .text:004015AE                                         ; 一般用在对定义的字符串进行初始化为  
>   .text:004015AE                                         ; ‘ ’或‘\\0’  
>   .text:004015AE  
>   .text:004015B3                 mov     eax, \[esi+3Ch\]  
>   .text:004015B6                 push    4  
>   .text:004015B8                 add     eax, 800h  
>   .text:004015BD                 push    edi  
>   .text:004015BE                 push    eax  
>   .text:004015BF                 mov     \[esi+20h\], eax  
>   .text:004015C2                 call    unknown\_libname\_1 ; Microsoft VisualC 2-8/net runtime  
>   .text:004015C2  
>   .text:004015C7                 push    800h  
>   .text:004015CC                 lea     eax, \[esp+4Ch+arg\_8\]  
>   .text:004015D0                 push    eax  
>   .text:004015D1                 mov     eax, \[esi+20h\]  
>   .text:004015D4                 add     eax, 4  
>   .text:004015D7                 push    eax  
>   .text:004015D8                 call    unknown\_libname\_1 ; Microsoft VisualC 2-8/net runtime  
>   .text:004015D8  
>   .text:004015DD                 add     esp, 48h  
>   .text:004015E0                 mov     \[esi+74h\], ebx  
>   .text:004015E3                 jmp     short loc\_4015FE  
>   .text:004015E3  
>   .text:004015E5 ; ---------------------------------------------------------------------------  
>   .text:004015E5  
>   .text:004015E5 VMLoop:                                 ; CODE XREF: sub\_40156A+97j  
>   .text:004015E5                 inc     dword ptr \[esi+74h\]  
>   .text:004015E8                 mov     ecx, esi  
>   .text:004015EA                 call    sub\_4013DF      ; 解码 key  
>   .text:004015EA  
>   .text:004015EF                 mov     eax, \[esi+7Ch\]  
>   .text:004015F2                 movzx   eax, byte ptr \[eax\]  
>   .text:004015F5                 mov     ecx, esi  
>   .text:004015F7                 call    dword ptr \[esi+eax\*4+80h\] ; 执行 VM 指令  
>   .text:004015F7  
>   .text:004015FE  
>   .text:004015FE loc\_4015FE:                             ; CODE XREF: sub\_40156A+79j  
>   .text:004015FE                 cmp     \[esi+79h\], bl  
>   .text:00401601                 jz      short VMLoop  
>   .text:00401601  
>   .text:00401603                 pop     edi  
>   .text:00401604                 pop     esi  
>   .text:00401605                 pop     ebx  
>   .text:00401606                 retn  
>   .text:00401606  
>   .text:00401606 sub\_40156A      endp  
>     
>   VMLoop 处循环就是执行 VM 的地方,其中 sub\_4013DF 把数据段的一大段数据进行解码运算,运算后按 dword 保存在另一个内存空间,  
>   而 call    dword ptr \[esi+eax\*4+80h\]就是执行 VM 指令的 call.  
>     
>   四 VM 指令分析  
>   刚开始分析觉得非常奇怪,为什么它初始化是为什么没有很明显的 context 环境,仅仅是把指令地址保存,而如 VM 寄存器和标志  
>   位都未曾说明,深入分析后才知道它把这些融入 VM 指令中.  
>     
>   先分析两个重要 call,因为很多 VM 指令都调用它们.  
>   (1)sub\_401271  
>   .text:00401271 ; 相当于指令机器码判断  
>   .text:00401271 ; arg\_0 计算后值放入 arg\_4  
>   .text:00401271 ; mov \[arg\_4\],\[arg\_0\]  
>   .text:00401271 ; Attributes: bp-based frame  
>   .text:00401271  
>   .text:00401271 ; int \_\_stdcall sub\_401271(int,int)  
>   .text:00401271 sub\_401271      proc near               ; CODE XREF: VMAdd+1Dp  
>   .text:00401271                                         ; VMAdd+2Cp VMSub+1Dp  
>   .text:00401271                                         ; VMSub+2Cp VMMOV+17p  
>   .text:00401271                                         ; VMMov\_1+16p ...  
>   .text:00401271  
>   .text:00401271 arg\_0           = dword ptr  8          ; 内存地址  
>   .text:00401271 arg\_4           = dword ptr  0Ch        ; 堆栈地址  
>   .text:00401271  
>   .text:00401271                 push    ebp  
>   .text:00401272                 mov     ebp, esp  
>   .text:00401274                 mov     eax, \[ebp+arg\_0\]  
>   .text:00401277                 movzx   edx, byte ptr \[eax\] ; 逐位取字节  
>   .text:0040127A                 sub     edx, 0          ; 是否为 0  
>   .text:0040127D                 jz      short oneis0denextcode  
>   .text:0040127D  
>   .text:0040127F                 dec     edx             ; 是否为 1  
>   .text:00401280                 jz      short oneis1denextcode  
>   .text:00401280  
>   .text:00401282                 dec     edx             ; 是否为 2  
>   .text:00401283                 jnz     locreturn0  
>   .text:00401283  
>   .text:00401289                 movzx   ecx, byte ptr \[eax+1\]  
>   .text:0040128D                 sub     ecx, edx        ; 下一位-上一位+2 是否为 0  
>   .text:0040128F                 jz      short movaleaxadd4value  
>   .text:0040128F  
>   .text:00401291                 dec     ecx             ; 下一位-上一位+1 是否为 0  
>   .text:00401292                 jz      short movaleaxadd4value  
>   .text:00401292  
>   .text:00401294                 dec     ecx             ; 下一位-上一位是否为 0  
>   .text:00401295                 jz      short movaxeaxadd4value  
>   .text:00401295  
>   .text:00401297                 dec     ecx             ; 下一位-上一位-1 是否为 0  
>   .text:00401298                 jnz     locreturn0  
>   .text:00401298  
>   .text:0040129E                 mov     eax, \[eax+4\]    ; 第五位  
>   .text:004012A1                 jmp     short movarg4eax  
>   .text:004012A1  
>   .text:004012A3 ; ---------------------------------------------------------------------------  
>   .text:004012A3  
>   .text:004012A3 movaxeaxadd4value:                      ; CODE XREF: sub\_401271+24j  
>   .text:004012A3                 mov     ax, \[eax+4\]     ; 取之后第四位  
>   .text:004012A7                 jmp     short movarg4ax  
>   .text:004012A7  
>   .text:004012A9 ; ---------------------------------------------------------------------------  
>   .text:004012A9  
>   .text:004012A9 movaleaxadd4value:                      ; CODE XREF: sub\_401271+1Ej  
>   .text:004012A9                                         ; sub\_401271+21j  
>   .text:004012A9                 mov     al, \[eax+4\]     ; 取之后第四位  
>   .text:004012AC                 jmp     short movarg4al  
>   .text:004012AC  
>   .text:004012AE ; ---------------------------------------------------------------------------  
>   .text:004012AE  
>   .text:004012AE oneis1denextcode:                       ; CODE XREF: sub\_401271+Fj  
>   .text:004012AE                 movzx   ecx, byte ptr \[eax+1\]  
>   .text:004012B2                 sub     ecx, 0  
>   .text:004012B5                 jz      short oneis1twois0or1denextcode  
>   .text:004012B5  
>   .text:004012B7                 dec     ecx  
>   .text:004012B8                 jz      short oneis1twois0or1denextcode  
>   .text:004012B8  
>   .text:004012BA                 dec     ecx  
>   .text:004012BB                 jz      short oneis1twois2denextcode  
>   .text:004012BB  
>   .text:004012BD                 dec     ecx  
>   .text:004012BE                 jnz     short locreturn0  
>   .text:004012BE  
>   .text:004012C0                 mov     eax, \[eax+4\]    ; 取下四位  
>   .text:004012C3                 mov     eax, \[eax\]      ; 取下四位的值  
>   .text:004012C5                 jmp     short movarg4eax  
>   .text:004012C5  
>   .text:004012C7 ; ---------------------------------------------------------------------------  
>   .text:004012C7  
>   .text:004012C7 oneis1twois2denextcode:                 ; CODE XREF: sub\_401271+4Aj  
>   .text:004012C7                 mov     eax, \[eax+4\]  
>   .text:004012CA                 mov     ax, \[eax\]       ; 取之后第四位作为指针,继续取值  
>   .text:004012CD                 jmp     short movarg4ax  
>   .text:004012CD  
>   .text:004012CF ; ---------------------------------------------------------------------------  
>   .text:004012CF  
>   .text:004012CF oneis1twois0or1denextcode:              ; CODE XREF: sub\_401271+44j  
>   .text:004012CF                                         ; sub\_401271+47j  
>   .text:004012CF                 mov     eax, \[eax+4\]  
>   .text:004012D2                 mov     al, \[eax\]  
>   .text:004012D4                 jmp     short movarg4al  
>   .text:004012D4  
>   .text:004012D6 ; ---------------------------------------------------------------------------  
>   .text:004012D6  
>   .text:004012D6 oneis0denextcode:                       ; CODE XREF: sub\_401271+Cj  
>   .text:004012D6                 movzx   edx, byte ptr \[eax+1\] ; 是 0 取下一位  
>   .text:004012DA                 sub     edx, 0          ; 是否为 0  
>   .text:004012DD                 jz      short oneis0twois0denextcode  
>   .text:004012DD  
>   .text:004012DF                 dec     edx  
>   .text:004012E0                 jz      short oneis0twois1denextcode  
>   .text:004012E0  
>   .text:004012E2                 dec     edx  
>   .text:004012E3                 jz      short oneis0twois2denextcode  
>   .text:004012E3  
>   .text:004012E5                 dec     edx  
>   .text:004012E6                 jnz     short locreturn0  
>   .text:004012E6  
>   .text:004012E8                 mov     eax, \[eax+4\]  
>   .text:004012EB                 mov     eax, \[ecx+eax\*4+4\]  
>   .text:004012EB  
>   .text:004012EF  
>   .text:004012EF movarg4eax:                             ; CODE XREF: sub\_401271+30j  
>   .text:004012EF                                         ; sub\_401271+54j  
>   .text:004012EF                 mov     ecx, \[ebp+arg\_4\]  
>   .text:004012F2                 mov     \[ecx\], eax  
>   .text:004012F4                 jmp     short locreturn0  
>   .text:004012F4  
>   .text:004012F6 ; ---------------------------------------------------------------------------  
>   .text:004012F6  
>   .text:004012F6 oneis0twois2denextcode:                 ; CODE XREF: sub\_401271+72j  
>   .text:004012F6                 mov     eax, \[eax+4\]  
>   .text:004012F9                 mov     ax, \[ecx+eax\*4+4\]  
>   .text:004012F9  
>   .text:004012FE  
>   .text:004012FE movarg4ax:                              ; CODE XREF: sub\_401271+36j  
>   .text:004012FE                                         ; sub\_401271+5Cj  
>   .text:004012FE                 mov     ecx, \[ebp+arg\_4\]  
>   .text:00401301                 mov     \[ecx\], ax  
>   .text:00401304                 jmp     short locreturn0  
>   .text:00401304  
>   .text:00401306 ; ---------------------------------------------------------------------------  
>   .text:00401306  
>   .text:00401306 oneis0twois1denextcode:                 ; CODE XREF: sub\_401271+6Fj  
>   .text:00401306                 mov     eax, \[eax+4\]  
>   .text:00401309                 mov     eax, \[ecx+eax\*4+4\]  
>   .text:0040130D                 shr     eax, 8  
>   .text:00401310                 jmp     short movarg4al  
>   .text:00401310  
>   .text:00401312 ; ---------------------------------------------------------------------------  
>   .text:00401312  
>   .text:00401312 oneis0twois0denextcode:                 ; CODE XREF: sub\_401271+6Cj  
>   .text:00401312                 mov     eax, \[eax+4\]  
>   .text:00401315                 mov     al, \[ecx+eax\*4+4\]  
>   .text:00401315  
>   .text:00401319  
>   .text:00401319 movarg4al:                              ; CODE XREF: sub\_401271+3Bj  
>   .text:00401319                                         ; sub\_401271+63j  
>   .text:00401319                                         ; sub\_401271+9Fj  
>   .text:00401319                 mov     ecx, \[ebp+arg\_4\]  
>   .text:0040131C                 mov     \[ecx\], al  
>   .text:0040131C  
>   .text:0040131E  
>   .text:0040131E locreturn0:                             ; CODE XREF: sub\_401271+12j  
>   .text:0040131E                                         ; sub\_401271+27j  
>   .text:0040131E                                         ; sub\_401271+4Dj  
>   .text:0040131E                                         ; sub\_401271+75j  
>   .text:0040131E                                         ; sub\_401271+83j  
>   .text:0040131E                                         ; sub\_401271+93j  
>   .text:0040131E                 pop     ebp  
>   .text:0040131F                 retn    8  
>   .text:0040131F  
>   .text:0040131F sub\_401271      endp  
>   函数 sub\_4013DF 解码数据段 unk\_40E000,key 后这里再一次判断或者说是解码,这个函数有两个参数,一个指向堆栈地址,一个指向内存地址,  
>   运算结果保存在堆栈中.  
>     
>   (2)sub\_401322  
>   这个函数也这个函数有两个参数,一个指向堆栈地址,一个指向内存地址,运算结果保存在内存地址中.在 VM 指令中相当于把运  
>   算结果保存到指定内存地址.  
>   .text:00401322 ; 相当于 VM 存储器、寄存器等  
>   .text:00401322 ; Attributes: bp-based frame  
>   .text:00401322  
>   .text:00401322 ; int \_\_stdcall sub\_401322(int,int)  
>   .text:00401322 sub\_401322      proc near               ; CODE XREF: VMAdd+3Ep  
>   .text:00401322                                         ; VMSub+3Ep VMMOV+26p  
>   .text:00401322                                         ; VMMov1+22p VMImul+46p  
>   .text:00401322                                         ; VMShl+3Ep ...  
>   .text:00401322  
>   .text:00401322 arg\_0           = dword ptr  8          ; 内存地址  
>   .text:00401322 arg\_4           = dword ptr  0Ch        ; 堆栈地址  
>   .text:00401322  
>   .text:00401322                 push    ebp  
>   .text:00401323                 mov     ebp, esp  
>   .text:00401325                 mov     eax, \[ebp+arg\_0\]  
>   .text:00401328                 movzx   edx, byte ptr \[eax\]  
>   .text:0040132B                 sub     edx, 0  
>   .text:0040132E                 jz      short oneis0denextcode  
>   .text:0040132E  
>   .text:00401330                 dec     edx  
>   .text:00401331                 jnz     return1  
>   .text:00401331  
>   .text:00401337                 movzx   ecx, byte ptr \[eax+1\]  
>   .text:0040133B                 sub     ecx, edx  
>   .text:0040133D                 jz      short onedaimadec1dengyutwodaimaoradd1  
>   .text:0040133D  
>   .text:0040133F                 dec     ecx  
>   .text:00401340                 jz      short onedaimadec1dengyutwodaimaoradd1  
>   .text:00401340  
>   .text:00401342                 dec     ecx  
>   .text:00401343                 jz      short onedaimadec1dengyutwodaimaadd2  
>   .text:00401343  
>   .text:00401345                 dec     ecx  
>   .text:00401346                 jnz     return1  
>   .text:00401346  
>   .text:0040134C                 mov     ecx, \[ebp+arg\_4\]  
>   .text:0040134F                 mov     eax, \[eax+4\]  
>   .text:00401352                 mov     ecx, \[ecx\]  
>   .text:00401354                 jmp     mov\_eax\_ecx  
>   .text:00401354  
>   .text:00401359 ; ---------------------------------------------------------------------------  
>   .text:00401359  
>   .text:00401359 onedaimadec1dengyutwodaimaadd2:         ; CODE XREF: sub\_401322+21j  
>   .text:00401359                 mov     ecx, \[ebp+arg\_4\]  
>   .text:0040135C                 mov     eax, \[eax+4\]  
>   .text:0040135F                 mov     cx, \[ecx\]  
>   .text:00401362                 mov     \[eax\], cx       ; 局部变量指针值存入\[eax+4\]  
>   .text:00401365                 jmp     short return1  
>   .text:00401365  
>   .text:00401367 ; ---------------------------------------------------------------------------  
>   .text:00401367  
>   .text:00401367 onedaimadec1dengyutwodaimaoradd1:       ; CODE XREF: sub\_401322+1Bj  
>   .text:00401367                                         ; sub\_401322+1Ej  
>   .text:00401367                 mov     ecx, \[ebp+arg\_4\] ; 堆栈局部变量  
>   .text:0040136A                 mov     eax, \[eax+4\]  
>   .text:0040136D                 mov     cl, \[ecx\]       ; 局部变量指针值  
>   .text:0040136F                 mov     \[eax\], cl       ; 局部变量指针值存入\[eax+4\]  
>   .text:00401371                 jmp     short return1  
>   .text:00401371  
>   .text:00401373 ; ---------------------------------------------------------------------------  
>   .text:00401373  
>   .text:00401373 oneis0denextcode:                       ; CODE XREF: sub\_401322+Cj  
>   .text:00401373                 movzx   edx, byte ptr \[eax+1\]  
>   .text:00401377                 sub     edx, 0  
>   .text:0040137A                 jz      short oneis0twois0denextcode  
>   .text:0040137A  
>   .text:0040137C                 dec     edx  
>   .text:0040137D                 jz      short oneis0twois1denextcode  
>   .text:0040137D  
>   .text:0040137F                 dec     edx  
>   .text:00401380                 jz      short oneis0twois2denextcode  
>   .text:00401380  
>   .text:00401382                 dec     edx  
>   .text:00401383                 jnz     short return1  
>   .text:00401383  
>   .text:00401385                 mov     edx, \[ebp+arg\_4\]  
>   .text:00401388                 mov     eax, \[eax+4\]  
>   .text:0040138B                 mov     edx, \[edx\]  
>   .text:0040138D                 mov     \[ecx+eax\*4+4\], edx ; 局部变量指针值存入\[ecx+eax\*4+4\]  
>   .text:00401391                 jmp     short return1  
>   .text:00401391  
>   .text:00401393 ; ---------------------------------------------------------------------------  
>   .text:00401393  
>   .text:00401393 oneis0twois2denextcode:                 ; CODE XREF: sub\_401322+5Ej  
>   .text:00401393                 mov     eax, \[eax+4\]  
>   .text:00401396                 mov     edx, \[ebp+arg\_4\]  
>   .text:00401399                 movsx   edx, word ptr \[edx\] ; 局部变量指针值  
>   .text:0040139C                 lea     eax, \[ecx+eax\*4+4\]  
>   .text:004013A0                 mov     ecx, \[eax\]  
>   .text:004013A2                 and     ecx, 0FFFF0000h ; ecx=\[ecx+eax\*4+4\] and 0FFFF0000h  
>   .text:004013A8                 jmp     short orecxedx  
>   .text:004013A8  
>   .text:004013AA ; ---------------------------------------------------------------------------  
>   .text:004013AA  
>   .text:004013AA oneis0twois1denextcode:                 ; CODE XREF: sub\_401322+5Bj  
>   .text:004013AA                 mov     eax, \[eax+4\]  
>   .text:004013AD                 mov     edx, \[ebp+arg\_4\]  
>   .text:004013B0                 lea     eax, \[ecx+eax\*4+4\]  
>   .text:004013B4                 xor     ecx, ecx  
>   .text:004013B6                 mov     ch, \[edx\]  
>   .text:004013B8                 mov     edx, \[eax\]  
>   .text:004013BA                 and     edx, 0FFFF00FFh  
>   .text:004013C0                 jmp     short orecxedx  
>   .text:004013C0  
>   .text:004013C2 ; ---------------------------------------------------------------------------  
>   .text:004013C2  
>   .text:004013C2 oneis0twois0denextcode:                 ; CODE XREF: sub\_401322+58j  
>   .text:004013C2                 mov     eax, \[eax+4\]  
>   .text:004013C5                 mov     edx, \[ebp+arg\_4\]  
>   .text:004013C8                 movzx   edx, byte ptr \[edx\]  
>   .text:004013CB                 lea     eax, \[ecx+eax\*4+4\]  
>   .text:004013CF                 mov     ecx, \[eax\]  
>   .text:004013D1                 and     ecx, 0FFFFFF00h  
>   .text:004013D1  
>   .text:004013D7  
>   .text:004013D7 orecxedx:                               ; CODE XREF: sub\_401322+86j  
>   .text:004013D7                                         ; sub\_401322+9Ej  
>   .text:004013D7                 or      ecx, edx        ; 局部指针值 or (\[ecx+eax\*4+4\] and 0FFFF0000h)存入 ecx=\[ecx+eax\*4+4\] and 0FFFF0000h  
>   .text:004013D7  
>   .text:004013D9  
>   .text:004013D9 mov\_eax\_ecx:                            ; CODE XREF: sub\_401322+32j  
>   .text:004013D9                 mov     \[eax\], ecx      ; oneis0 时,  
>   .text:004013D9                                         ; 局部指针值 or (\[ecx+eax\*4+4\] and 0FFFF0000h)  
>   .text:004013D9                                         ; 存入\[ecx+eax\*4+4\] and 0FFFF0000h.  
>   .text:004013D9                                         ; one 不是 0 时,  
>   .text:004013D9                                         ; 局部变量指针值存入\[eax+4\].  
>   .text:004013D9  
>   .text:004013DB  
>   .text:004013DB return1:                                ; CODE XREF: sub\_401322+Fj  
>   .text:004013DB                                         ; sub\_401322+24j  
>   .text:004013DB                                         ; sub\_401322+43j  
>   .text:004013DB                                         ; sub\_401322+4Fj  
>   .text:004013DB                                         ; sub\_401322+61j  
>   .text:004013DB                                         ; sub\_401322+6Fj  
>   .text:004013DB                 pop     ebp  
>   .text:004013DC                 retn    8  
>   .text:004013DC  
>   .text:004013DC sub\_401322      endp  
>     
>   分析了以上这两个函数后,VM 指令就好识别多了.  
>     
>   (3)寻找下一条 key 地址的代码特征(unk\_40E000 为开始)  
>   mov     eax, \[esi+7Ch\]   
>   mov     eax, \[eax+4\] ;本条 key 长度  
>   add     \[esi+4Ch\], eax ;下一 key 的地址  
>     
>   (4)经过上面三步分析,识别 VM 指令就简单多了,一般来说,每个 VM 指令调用 sub\_401271 和 sub\_401322 中间部分就是识别 VM 指令  
>   的关键了.  
>   这里仅列举两个 VM 指令,具体看 MyVM #1 KeygenMe by BUBlic.idb 文件  
>     
>   标志位指令 VMFlag:  
>   .text:004017E3 VMFlag          proc near               ; DATA XREF: sub\_401BDC+B3o  
>   .text:004017E3  
>   .text:004017E3 var\_8           = dword ptr -8  
>   .text:004017E3 var\_4           = dword ptr -4  
>   .text:004017E3  
>   .text:004017E3                 push    ebp  
>   .text:004017E4                 mov     ebp, esp  
>   .text:004017E6                 push    ecx  
>   .text:004017E7                 push    ecx  
>   .text:004017E8                 push    ebx  
>   .text:004017E9                 push    esi  
>   .text:004017EA                 push    edi  
>   .text:004017EB                 mov     esi, ecx  
>   .text:004017ED                 mov     edi, \[esi+7Ch\]  
>   .text:004017F0                 lea     eax, \[ebp+var\_8\]  
>   .text:004017F3                 push    eax             ; int  
>   .text:004017F4                 lea     eax, \[edi+0Ch\]  
>   .text:004017F7                 xor     ebx, ebx  
>   .text:004017F9                 push    eax             ; int  
>   .text:004017FA                 mov     \[ebp+var\_8\], ebx  
>   .text:004017FD                 mov     \[ebp+var\_4\], ebx ; var\_4,var\_8 为 0  
>   .text:00401800                 call    sub\_401271      ; 相当于指令机器码判断  
>   .text:00401800                                         ; arg\_0 计算后值放入 arg\_4  
>   .text:00401800                                         ; mov \[arg\_4\],\[arg\_0\]  
>   .text:00401800  
>   .text:00401805                 lea     eax, \[ebp+var\_4\]  
>   .text:00401808                 push    eax             ; int  
>   .text:00401809                 lea     eax, \[edi+14h\]  
>   .text:0040180C                 push    eax             ; int  
>   .text:0040180D                 mov     ecx, esi  
>   .text:0040180F                 call    sub\_401271      ; 相当于指令机器码判断  
>   .text:0040180F                                         ; arg\_0 计算后值放入 arg\_4  
>   .text:0040180F                                         ; mov \[arg\_4\],\[arg\_0\]  
>   .text:0040180F  
>   .text:00401814                 mov     ecx, \[ebp+var\_8\]  
>   .text:00401817                 cmp     ecx, \[ebp+var\_4\] ; 判断取值是否为 0  
>   .text:0040181A                 jnb     short loc\_401821  
>   .text:0040181A  
>   .text:0040181C                 mov     \[esi+3Ah\], bl  
>   .text:0040181F                 jmp     short loc\_401829  
>   .text:0040181F  
>   .text:00401821 ; ---------------------------------------------------------------------------  
>   .text:00401821  
>   .text:00401821 loc\_401821:                             ; CODE XREF: VMFlag+37j  
>   .text:00401821                 setnbe  al  
>   .text:00401824                 inc     al  
>   .text:00401826                 mov     \[esi+3Ah\], al  
>   .text:00401826  
>   .text:00401829  
>   .text:00401829 loc\_401829:                             ; CODE XREF: VMFlag+3Cj  
>   .text:00401829                 movzx   eax, byte ptr \[edi+0Dh\]  
>   .text:0040182D                 sub     eax, ebx  
>   .text:0040182F                 jz      short loc\_40183C  
>   .text:0040182F  
>   .text:00401831                 dec     eax  
>   .text:00401832                 jz      short loc\_40183C  
>   .text:00401832  
>   .text:00401834                 dec     eax  
>   .text:00401835                 jnz     short loc\_401842  
>   .text:00401835  
>   .text:00401837                 movsx   eax, cx  
>   .text:0040183A                 jmp     short loc\_40183F  
>   .text:0040183A  
>   .text:0040183C ; ---------------------------------------------------------------------------  
>   .text:0040183C  
>   .text:0040183C loc\_40183C:                             ; CODE XREF: VMFlag+4Cj  
>   .text:0040183C                                         ; VMFlag+4Fj  
>   .text:0040183C                 movzx   eax, cl  
>   .text:0040183C  
>   .text:0040183F  
>   .text:0040183F loc\_40183F:                             ; CODE XREF: VMFlag+57j  
>   .text:0040183F                 mov     \[ebp+var\_8\], eax  
>   .text:0040183F  
>   .text:00401842  
>   .text:00401842 loc\_401842:                             ; CODE XREF: VMFlag+52j  
>   .text:00401842                 mov     ecx, esi  
>   .text:00401844                 call    sub\_401225  
>   .text:00401844  
>   .text:00401849                 sub     eax, ebx  
>   .text:0040184B                 jz      short loc\_401859  
>   .text:0040184B  
>   .text:0040184D                 dec     eax  
>   .text:0040184E                 jz      short loc\_401859  
>   .text:0040184E  
>   .text:00401850                 dec     eax  
>   .text:00401851                 jnz     short loc\_401860  
>   .text:00401851  
>   .text:00401853                 movsx   eax, word ptr \[ebp+var\_4\]  
>   .text:00401857                 jmp     short loc\_40185D  
>   .text:00401857  
>   .text:00401859 ; ---------------------------------------------------------------------------  
>   .text:00401859  
>   .text:00401859 loc\_401859:                             ; CODE XREF: VMFlag+68j  
>   .text:00401859                                         ; VMFlag+6Bj  
>   .text:00401859                 movzx   eax, byte ptr \[ebp+var\_4\]  
>   .text:00401859  
>   .text:0040185D  
>   .text:0040185D loc\_40185D:                             ; CODE XREF: VMFlag+74j  
>   .text:0040185D                 mov     \[ebp+var\_4\], eax  
>   .text:0040185D  
>   .text:00401860  
>   .text:00401860 loc\_401860:                             ; CODE XREF: VMFlag+6Ej  
>   .text:00401860                 mov     eax, \[ebp+var\_8\]  
>   .text:00401863                 cmp     eax, \[ebp+var\_4\]  
>   .text:00401866                 jge     short loc\_40186D  
>   .text:00401866  
>   .text:00401868                 mov     \[esi+3Bh\], bl  
>   .text:0040186B                 jmp     short loc\_401875  
>   .text:0040186B  
>   .text:0040186D ; ---------------------------------------------------------------------------  
>   .text:0040186D  
>   .text:0040186D loc\_40186D:                             ; CODE XREF: VMFlag+83j  
>   .text:0040186D                 setnle  al  
>   .text:00401870                 inc     al  
>   .text:00401872                 mov     \[esi+3Bh\], al  
>   .text:00401872  
>   .text:00401875  
>   .text:00401875 loc\_401875:                             ; CODE XREF: VMFlag+88j  
>   .text:00401875                 mov     eax, \[edi+4\]  
>   .text:00401878                 add     \[esi+4Ch\], eax  
>   .text:0040187B                 pop     edi  
>   .text:0040187C                 pop     esi  
>   .text:0040187D                 pop     ebx  
>   .text:0040187E                 leave  
>   .text:0040187F                 retn  
>   .text:0040187F  
>   .text:0040187F VMFlag          endp  
>     
>   条件跳转指令 VMEipAddress:  
>   .text:004018A4 VMEipAddress    proc near               ; DATA XREF: sub\_401BDC+C7o  
>   .text:004018A4  
>   .text:004018A4 var\_4           = dword ptr -4  
>   .text:004018A4  
>   .text:004018A4                 push    ebp  
>   .text:004018A5                 mov     ebp, esp  
>   .text:004018A7                 push    ecx  
>   .text:004018A8                 and     \[ebp+var\_4\], 0  
>   .text:004018AC                 push    esi  
>   .text:004018AD                 push    edi  
>   .text:004018AE                 mov     esi, ecx  
>   .text:004018B0                 mov     edi, \[esi+7Ch\]  
>   .text:004018B3                 lea     eax, \[ebp+var\_4\]  
>   .text:004018B6                 push    eax             ; int  
>   .text:004018B7                 lea     eax, \[edi+0Ch\]  
>   .text:004018BA                 push    eax             ; int  
>   .text:004018BB                 call    sub\_401271      ; 相当于指令机器码判断  
>   .text:004018BB                                         ; arg\_0 计算后值放入 arg\_4  
>   .text:004018BB                                         ; mov \[arg\_4\],\[arg\_0\]  
>   .text:004018BB  
>   .text:004018C0                 cmp     byte ptr \[esi+3Bh\], 1 ;比较标志位  
>   .text:004018C4                 mov     eax, \[ebp+var\_4\]  
>   .text:004018C7                 jz      short loc\_4018CC  
>   .text:004018C7  
>   .text:004018C9                 mov     eax, \[edi+4\]  
>   .text:004018C9  
>   .text:004018CC  
>   .text:004018CC loc\_4018CC:                             ; CODE XREF: VMEipAddress+23j  
>   .text:004018CC                 add     \[esi+4Ch\], eax  ;跳转到的地址  
>   .text:004018CF                 pop     edi  
>   .text:004018D0                 pop     esi  
>   .text:004018D1                 leave  
>   .text:004018D2                 retn  
>   .text:004018D2  
>   .text:004018D2 VMEipAddress    endp  
>     
>   收工.:)  
>     
> \--------------------------------------------------------------------------------  
> 【版权声明】: 转载请注明作者并保持文章的完整, 谢谢!  
>   
>                                                        2007 年 11 月 08 日 13:46:03 

 4dd466d0e1e0cc123afbe504da2462ca