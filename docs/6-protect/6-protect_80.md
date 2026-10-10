

# 逆向 NP 之注入 npggNT.des

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-465.htm`](https://www.pediy.com/kssd/pediy09/pediy09-465.htm)

> NP=nProtect GameGuard  
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
> \*\*标题:逆向 NP 之注入 npggNT.des  
> \*\*作者:堕落天才  
> \*\*时间:2007-2-1  
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
>       
>     我们知道 NP 启动后会向所有进程注入模块 npggNT.des,这个过程包括:向目标进程注入执行代码及数据,创建远程线程,远程线程执行,装入  
>   
> npggNT.des。之后的事我们都知道了:npggNT.des 挂钩关键系统函数。在得到 NP 注入代码前，这个注入过程我想对于象我这样的菜鸟来说是非常  
>   
> 神秘的，于是就产生了种种幻想，装入 npggNT.des 是不是使用更底层的系统函数？还是 NP 自己构建的 LoadLibrary?又或是使用 COPY 来的系统函  
>   
> 数代码？但是这两天分析的结果令我大跌眼睛它竟然是使用 LoadLibraryA 装入 npggNT.des 的，这怎么可能？？但事实上真的是这样，下面我们  
>   
> 来看看代码就知道了。   
>      
>     1,主要代码（这里是 npggNT.des 装入的主要执行过程，要看完整的二进制代码请看附件 NPRemotThread.asm，编译后反汇编看）  
>   
> 代码:  
> 00C10000    59              pop     ecx                                
> 00C10001    58              pop     eax                                
> 00C10002    51              push    ecx                              ;这里是一般函数都会做的寄存器压栈，好像没什么好说  
> 00C10003    53              push    ebx  
> 00C10004    55              push    ebp  
> 00C10005    56              push    esi  
> 00C10006    57              push    edi  
> 00C10007    2BC9            sub     ecx, ecx                         ; ecx 清零  
> 00C10009    E8 00000000     call    00C1000E  
> 00C1000E    5B              pop     ebx                              ; ebx=00C100E ->自定位函数入口地址  
> 00C1000F    64:8B51 30      mov     edx, fs:\[ecx+30\]                 ; \[ecx+30\]=\[30\]  
> 00C10013    83C3 F2         add     ebx, -0E                         ; EBX-E=00C10000 ->线程起始地址  
> 00C10016    85C0            test    eax, eax  
> 00C10018    53              push    ebx                              ; ebx 指向线程起始地址  
> 00C10019    50              push    eax                              ; 0  
> 00C1001A    E8 23000000     call    00C10042  
>   
> 00C10042    64:FF31         push    dword ptr fs:\[ecx\]               ; fs:\[ecx\]=fs:\[0\]->SEH 安装  
> 00C10045    64:8921         mov     fs:\[ecx\], esp  
> 00C10048    89A3 93000000   mov     \[ebx+93\], esp                    ; SMC \[ebx+93\]=\[00C10093\]->保存 SEH 返回的 ESP  
> 00C1004E    78 2C           js      short 00C1007C  
> 00C10050    394A 0C         cmp     \[edx+C\], ecx                     ; \[edx+c\]=\[7FFDC00C\]=00241EA0  
> 00C10053    75 30           jnz     short 00C10085  
>   
> 00C10085    68 0000807C     push    7C800000                         ; kernel32.dll 的 HANDLE －>这里 NP 硬编码了 kernel32.dll 的  
>   
> 句柄  
> 00C1008A    53              push    ebx                              ; 线程起始地址  
> 00C1008B    50              push    eax                              ; 0  
> 00C1008C    FC              cld  
> 00C1008D    E8 16010000     call    00C101A8  
>   
> 00C101A8    E8 00000000     call    00C101AD  
> 00C101AD    5F              pop     edi                              ; EDI=00C101AD ->函数入口地址  
> 00C101AE    8B5424 0C       mov     edx, \[esp+C\]                     ; EDX=kernel32.dll HANDLE  
> 00C101B2    81C7 EF010000   add     edi, 1EF                         ; EDI=00C1039C ->npggNT.des 路径  
> 00C101B8    8D77 E4         lea     esi, \[edi-1C\]                    ; ESI=00C10380  
> 00C101BB    6A 04           push    4  
> 00C101BD    59              pop     ecx                              ; ECX=4  
> 00C101BE    AD              lods    dword ptr \[esi\]  
> 00C101BF    8B28            mov     ebp, \[eax\]                       ; 这里非常精彩，后面会详细分析  
> 00C101C1    03EA            add     ebp, edx                         ;   
> 00C101C3    896E FC         mov     \[esi-4\], ebp                     ;   
> 00C101C6  ^ E2 F6           loopd   short 00C101BE  
> 00C101C8    8B5F FE         mov     ebx, \[edi-2\]                     ; \[edi-2\]=\[00C1039A\]=3A440002  
> 00C101CB    0B5F FA         or      ebx, \[edi-6\]                     ; \[edi-6\]=\[00C10396\]=00090100 EBX=3A4D0102  
> 00C101CE    8B47 F4         mov     eax, \[edi-C\]                     ; \[edi-C\]=\[00C10390\]=00000003  
> 00C101D1    66:894F FA      mov     \[edi-6\], cx                      ; word ptr \[edi-6\]=\[00C10396\]=cx=0  
> 00C101D5    F6C7 08         test    bh, 8                            ; bh=01 and 8 =0001b and 1000b =0  
> 00C101D8    0F85 CA000000   jnz     00C102A8  
> 00C101DE    8B07            mov     eax, \[edi\]                       ; \[edi\]=\[00C1039C\]=475C3A44->npggNT.des 路径  
> 00C101E0    F6C7 02         test    bh, 2                            ; bh=01 and 2 =0001b and 0010b=0  
> 00C101E3    75 10           jnz     short 00C101F5  
> 00C101E5    8BF0            mov     esi, eax                         ; esi->npggNT.des 路径开始 4 个字节  
> 00C101E7    85C0            test    eax, eax                         ; 判断路径是不是为空  
> 00C101E9    74 02           je      short 00C101ED  
> 00C101EB    8BF7            mov     esi, edi                         ; esi=00C1039C  
> 00C101ED    84DB            test    bl, bl                           ; bl=02  
> 00C101EF    56              push    esi                              ; \\  
> 00C101F0    78 3D           js      short 00C1022F                   ;  eax=GetModuleHandleA("D:\\\\...\\\\npggNT.des")  
> 00C101F2    FF57 E4         call    \[edi-1C\]                         ; /  
> 00C101F5    F6C3 02         test    bl, 2                            ; bl=02  
> 00C101F8    75 4B           jnz     short 00C10245                   ; bl 不等于零 跳转实现  
>   
> 00C10245    F6C7 02         test    bh, 2                            ; bh=01  
> 00C10248    75 2F           jnz     short 00C10279                   ; bh=01 and 2=0001b and 0010b=0 跳转未实现  
> 00C1024A    85F6            test    esi, esi                         ; esi=00C1039C->npggNT.des 所在路径  
> 00C1024C    74 2B           je      short 00C10279                   ; 不为零  
> 00C1024E    F6C7 01         test    bh, 1                            ; bh=01 and 1=0001b and 0001b=1  
> 00C10251    74 04           je      short 00C10257  
> 00C10253    85C0            test    eax, eax                         ; eax 是执行 GetModuleHandleA("D:\\\\...\\\\npggNT.des")的结果  
> 00C10255    75 22           jnz     short 00C10279                   ; 确定 npggNT.des 未被加载  
> 00C10257    FF4F F8         dec     dword ptr \[edi-8\]                ; \[00C10394\]=00000001-1=0 jmp 回来-1=-1  
> 00C1025A    78 1D           js      short 00C10279                   ; 不为负 为负跳  
> 00C1025C    84DB            test    bl, bl                           ; bl=02  
> 00C1025E    56              push    esi                              ; \\  
> 00C1025F    78 05           js      short 00C10266                   ;  eax=LoadLibraryA("D:\\\\...\\\\npggNT.des")   
> 00C10261    FF57 E8         call    \[edi-18\]                         ; /  
> 00C10264  ^ EB F1           jmp     short 00C10257  
>   
> 00C10279    85C0            test    eax, eax                         ; eax 为模块 npggNT.des 的句柄  
> 00C1027B    75 07           jnz     short 00C10284  
> 00C1027D    B8 B6F3C2E1     mov     eax, E1C2F3B6  
> 00C10282    EB 64           jmp     short 00C102E8  
> 00C10284    8B4F F4         mov     ecx, \[edi-C\]                     ; \[edi-C\]=\[00C10390\]=00000003  
> 00C10287    F6C7 04         test    bh, 4                            ; bh=01 and 4=0001b and 0100b=0  
> 00C1028A    74 04           je      short 00C10290  
> 00C1028C    03C1            add     eax, ecx  
> 00C1028E    EB 18           jmp     short 00C102A8  
> 00C10290    85C9            test    ecx, ecx                         ; ecx=3  
> 00C10292    75 0B           jnz     short 00C1029F  
> 00C10294    8D8F 04010000   lea     ecx, \[edi+104\]  
> 00C1029A    8039 00         cmp     byte ptr \[ecx\], 0  
> 00C1029D    74 49           je      short 00C102E8  
> 00C1029F    84DB            test    bl, bl                           ; bl=02  
> 00C102A1    51              push    ecx                              ; \\  
> 00C102A2    50              push    eax                              ;   eax=GetProcAddress(HandleOfNpggNT.des,03)  
> 00C102A3  ^ 78 CB           js      short 00C10270  
> 00C102A5    FF57 F0         call    \[edi-10\]                         ; /  
> 00C102A8    85C0            test    eax, eax                         ; eax=458A1830->npggNT.des function #03  
> 00C102AA    75 07           jnz     short 00C102B3  
> 00C102AC    B8 B7F3C2E1     mov     eax, E1C2F3B7  
> 00C102B1    EB 35           jmp     short 00C102E8  
> 00C102B3    8AD7            mov     dl, bh                           ; dl=bh=01  
> 00C102B5    0FB74F FC       movzx   ecx, word ptr \[edi-4\]            ; ecx=word ptr \[edi-4\]=\[00C10398\]=0009=00000009  
> 00C102B9    8BD8            mov     ebx, eax                         ; ebx=eax=npggNT.des.func#03  
> 00C102BB    8BEC            mov     ebp, esp                         ; esp=00E1FF8C  
> 00C102BD    E3 24           jecxz   short 00C102E3                   ; ecx=9  
> 00C102BF    8DBF 0C030000   lea     edi, \[edi+30C\]                   ; \[edi+30C\]=\[00C1039C+30C\]->00C106A8  
> 00C102C5    8D748F FC       lea     esi, \[edi+ecx\*4-4\]               ; \[edi+ecx\*4-4\]=\[00C1039C+9\*4-4\]->00C106C8  
> 00C102C9    FD              std  
> 00C102CA    E8 58000000     call    00C10327                         ; 该函数用来装入某些数据 然后判断  
> 00C102CF    A8 80           test    al, 80                           ; al=01  
> 00C102D1   /75 18           jnz     short 00C102EB  
> 00C102D3   |A8 40           test    al, 40  
> 00C102D5   |75 19           jnz     short 00C102F0  
> 00C102D7   |A8 20           test    al, 20  
> 00C102D9   |75 26           jnz     short 00C10301  
> 00C102DB   |A8 10           test    al, 10  
> 00C102DD   |75 2B           jnz     short 00C1030A  
> 00C102DF   |AD              lods    dword ptr \[esi\]                  ; \[esi\]=\[00C106C8\]=0  
> 00C102E0   |50              push    eax  
> 00C102E1  ^|E2 FC           loopd   short 00C102DF  
> 00C102E3   |FC              cld  
> 00C102E4   |FFD3            call    ebx                              ; ebx=458A1830->npggNT.des.func#03 ->这里 npggNT.des 就开  
>   
> 始工作了  
> 00C102E6   |8BE5            mov     esp, ebp  
> 00C102E8   |C2 0C00         retn    0C  
>   
> 00C10092    BC 9CFFE100     mov     esp, 0E1FF9C                     ;还记得这里么？忘了请回头看 00C10048 处代码  
> 00C10097    64:8F05 0000000>pop     dword ptr fs:\[0\]                 ;SEH 异常返回这里  
> 00C1009E    59              pop     ecx  
> 00C1009F    8A4C24 02       mov     cl, \[esp+2\]  
> 00C100A3    5A              pop     edx  
> 00C100A4    8AF1            mov     dh, cl  
> 00C100A6    59              pop     ecx  
> 00C100A7    5F              pop     edi  
> 00C100A8    5E              pop     esi  
> 00C100A9    5D              pop     ebp  
> 00C100AA    5B              pop     ebx  
> 00C100AB    F6C6 40         test    dh, 40  
> 00C100AE    75 16           jnz     short 00C100C6  
> 00C100B0    F6C6 20         test    dh, 20  
> 00C100B3    74 07           je      short 00C100BC  
>   
> 00C100BC    F6C2 04         test    dl, 4  
> 00C100BF    5A              pop     edx  
> 00C100C0    58              pop     eax  
> 00C100C1    58              pop     eax  
> 00C100C2    74 2E           je      short 00C100F2  
>   
> 00C100F2    6A 00           push    0  
> 00C100F4    8BC4            mov     eax, esp  
> 00C100F6    51              push    ecx  
> 00C100F7    8BCC            mov     ecx, esp  
> 00C100F9    56              push    esi  
> 00C100FA    6A FE           push    -2  
> 00C100FC    52              push    edx  
> 00C100FD    68 00800000     push    8000  
> 00C10102    50              push    eax  
> 00C10103    51              push    ecx  
> 00C10104    6A FF           push    -1  
> 00C10106    52              push    edx  
> 00C10107    E8 AAFFFFFF     call    00C100B6  
> 00C1010C    E8 04000000     call    00C10115  
> 00C10111    03D0            add     edx, eax  
> 00C10113    FFE2            jmp     edx                             ; ntdll.ZwFreeVirtualMemory ->这个函数执行完毕后，这段代  
>   
> 码就消失了  
> 00C10115    8B15 043C927C   mov     edx, \[7C923C04\]  
> 00C1011B    C3              retn  
>   
> 数据:  
> 00C10374  47 65 74 4D 6F 64 75 6C 65 48 61 6E 28 2C 80 7C  GetModuleHan(,?  
> 00C10384  58 2F 80 7C 14 2A 80 7C B0 2C 80 7C 03 00 00 00  X/?\*???...  
> 00C10394  01 00 00 01 09 00 02 00 44 3A 5C 47 61 6D 65 5C  .....D:\\Game\\  
> 00C103A4  BE AA CC EC B6 AF B5 D8 43 61 62 61 6C 20 4F 6E  惊天动地 Cabal On  
> 00C103B4  6C 69 6E 65 5C 47 61 6D 65 47 75 61 72 64 5C 6E  line\\GameGuard\\n  
> 00C103C4  70 67 67 4E 54 2E 64 65 73                       pggNT.des  
>    
>       \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
>       2,精彩地方分析  
>   
>       看完上面代码后,不知你有没有失望的感觉。但我是有的，因为以前一直觉得神秘的东西一下子变得不神秘，而且不是自己想象中那样，  
>   
> 真的有点失望。但是代码里面也有一些精彩的地方值得学习。  
>   
>       2.1,自定位  
>   
>       在远程进程里面我们不能象在本地进程那样方便地使用数据，当我们不得已需要用到某数据时，就需要定位数据。数据 COPY 到远程进程  
>   
> 后，怎么把它找出来呢？上面可以看到 npggNT.des 路径地址在 00C1039C，但是本地线程一般不知道有个 00C1039C,我们看看 NP 怎么把它找出来。  
> 00C10009    E8 00000000     call    00C1000E  
> 00C1000E    5B              pop     ebx                              ; ebx=00C100E ->自定位函数入口地址  
> 00C1000F    64:8B51 30      mov     edx, fs:\[ecx+30\]                 ; \[ecx+30\]=\[30\]  
> 00C10013    83C3 F2         add     ebx, -0E                         ; EBX-E=00C10000 ->线程起始地址  
>       我们知道 call 指令执行完毕之后,当前 esp 保存了函数返回地址－call 指令下一条指令执行的地址，看看上面  
>       call 00C1000E  
>  00C1000E:  
>        pop  ebx   ;ebx->call 指令下一条指令的地址，那正好就是 pop ebx 的地址  
>        哈哈，定位了一条指令的地址后，原理上我们就可以把整一段代码定位了，比如上面通过 add  ebx, -0E 就把 ebx 指向线程的起始位置  
>   
> ，这样通过 ebx 相对寻址的话，整段代码的位置都可以确定了。  
>       上面还有一个自定位的地方  
> 00C101A8    E8 00000000     call    00C101AD  
> 00C101AD    5F              pop     edi          ; EDI=00C101AD ->函数入口地址  
>        原理就跟上面分析的一样了。  
>      
>       2.2 找系统函数  
>   
>       略一看上面代码，不知道你会不会决定奇怪，GetModuleHandleA,LoadLibraryA,FreeLibrary,GetProcAddress 这些 kernel32.dll 里面的  
>   
> 函数是怎么来的呢？  
>      为什么  
> 00C101F2    FF57 E4         call    \[edi-1C\]   
>      就是 GetModuleHandleA 呢？我们先分析一下这段代码:  
>   
> 00C101A8    E8 00000000     call    00C101AD  
> 00C101AD    5F              pop     edi                              ; EDI=00C101AD ->函数入口地址  
> 00C101AE    8B5424 0C       mov     edx, \[esp+C\]                     ; EDX=kernel32.dll HANDLE  
> 00C101B2    81C7 EF010000   add     edi, 1EF                         ; EDI=00C1039C ->npggNT.des 路径  
> 00C101B8    8D77 E4         lea     esi, \[edi-1C\]                    ; ESI=00C10380  
> 00C101BB    6A 04           push    4  
> 00C101BD    59              pop     ecx                              ; ECX=4  
> 00C101BE    AD              lods    dword ptr \[esi\]                  ;从 00C10380 开始依次 4 个字节地取出某数据，循环四次  
> 00C101BF    8B28            mov     ebp, \[eax\]                       ;根据数据寻址，将其指向的数据拿出来保存到 ebp   
> 00C101C1    03EA            add     ebp, edx                         ; ebp+=edx,上面可以看到 edx=kernel32.dll 的句柄也就是装  
> 00C101C3    896E FC         mov     \[esi-4\], ebp                     ; 载基地址了，基地址+偏移(基地址+RVA)，想到了么？    
>     好了，我们先看看 00C10380 跟其后面的数据是什么  
>     dword ptr \[00C10380\]=7C802C28   
>     dword ptr \[00C10384\]=7C802F58   
>     dword ptr \[00C10388\]=7C802A14  
>     dword ptr \[00C1038C\]=7C802CB0  
>     然后找到这些数据所指向的地址再看看  
>     \[7C802C28\]=B529  
>     \[7C802F58\]=1D77  
>     \[7C802A14\]=AA66  
>     \[7C802CB0\]=AC28  
>     然后把他们加上 kernel32.dll 的基地址也就是 handle(如果你忘了的话，请看 00C10085 行)  
>     7C800000+B529->GetModuleHandleA  
>     7C800000+1D77->LoadLibraryA  
>     7C800000+AA66->FreeLibrary  
>     7C800000+AC28->GetProcAddress  
>     到这里我们终于可以明白了，原来这些函数是这样来的。然后这些函数地址分别保存到 00C10380、00C10384、00C10388 及 00C1038C 中  
>    
>     2.3 自修改代码  
>   
>     不知你有没有留意，上面有一个小小的自修改。  
> 00C10048    89A3 93000000   mov     \[ebx+93\], esp          ; SMC \[ebx+93\]=\[00C10093\]->保存 SEH 返回的 ESP  
> 00C10092    BC 9CFFE100     mov     esp, 0E1FF9C           ;  
>     在 mov \[ebx+93\], esp 指令执行前 00C10092 处的代码是 mov esp,0 执行后就变成上面这个样子了。这里是把 ESP 数值直接写入代码里，呵呵  
>   
> ，可以省了一个变量。  
>       
>    \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
>    3，逆向代码(C)  
>   
>    根据上面的分析，我们已经可以理解 NP 装入 npggNT.des 的主要方法了，下面是代码，忽略了各种判断跳转。而且由于是用 C 来写，所以这里  
>   
> 没有用到自定位，而是直接把数据的地址当参数传递给线程函数（更详细代码请看附件 InjectDll.cpp）。  
>   
>    //////////////////////////////////声明 API////////////////////////////////  
> typedef HMODULE (WINAPI\*GETMODULEHANDLEA)(LPCSTR lpModuleName);             //GetModuleHandleA  
> typedef HMODULE (WINAPI\*LOADLIBRARYA)(LPCSTR lpLibFileName);                //LoadLibraryA  
> typedef BOOL    (WINAPI\*FREELIBRARY)(HMODULE hLibModule);                   //FreeLibrary  
> typedef FARPROC (WINAPI\*GETPROCADDRESS)(HMODULE hModule,LPCSTR lpProcName); //GetProcAddress  
>   
> //////////////////////////////////定义数据结构//////////////////////////////  
> typedef struct \_INJECTDATA  
> {  
>   BYTE                bName\[12\];                       //12 bytes="GetModuleHan";   
>   GETMODULEHANDLEA   \_GetModuleHandleA;                //4  bytes=hKernel32+0x2C28 －>输出函数地址表  
>   LOADLIBRARYA       \_LoadLibraryA;                    //4  bytes=hKernel32+0x2F58  
>   FREELIBRARY        \_FreeLibrary;                     //4  bytes=hKernel32+0x2A14  
>   GETPROCADDRESS     \_GetProcAddress;                  //4  bytes=hKernel32+0x2CB0  
>   BYTE               someNumber\[12\];                   //12 bytes    
>   TCHAR              szLibraryPath\[MAX\_PATH\];          //MAX\_PATH    
> }INJECTDATA,\*PINJECTDATA;  
>   
> ///////////////////////////////////定义远程线程////////////////////////////////////////////  
> static VOID WINAPI RemoteThread(LPVOID lpParam)  
> {    
>   PINJECTDATA myData=(PINJECTDATA)lpParam;  
>   DWORD       dwGetModuleHandleA,  
>             dwLoadLibraryA,  
>         dwFreeLibrary,  
>         dwGetProcAddress,  
>               hKernel32;  
>   
>   dwGetModuleHandleA= (DWORD)myData->\_GetModuleHandleA;  
>   dwLoadLibraryA    = (DWORD)myData->\_LoadLibraryA;  
>   dwFreeLibrary     = (DWORD)myData->\_FreeLibrary;  
>   dwGetProcAddress  = (DWORD)myData->\_GetProcAddress;  
>   hKernel32         =        myData->hKernel32;  
>   
>     ///////////////////////下面的汇编代码是根据输出函数地址表找到相应的函数地址的 RVA 值////////////  
>    \_asm{      
>     mov  ebx,hKernel32            // <--- 这个是 kernel32.dll 的句柄,NP 直接硬编码到这里  
>   
>     mov  eax,dwGetModuleHandleA  
>     mov  edx,\[eax\]                // <--- 根据地址表找出相应函数的 RVA 值  
>     add  edx,ebx                  // <--- 函数地址=模块加载基地址(即 handle)+相应 RVA 值  
>     mov  dwGetModuleHandleA,edx    
>   
>     mov  eax,dwLoadLibraryA  
>     mov  edx,\[eax\]  
>     add  edx,ebx  
>     mov  dwLoadLibraryA,edx  
>   
>     mov  eax,dwFreeLibrary  
>     mov  edx,\[eax\]  
>     add  edx,ebx  
>     mov  dwFreeLibrary,edx  
>   
>     mov  eax,dwGetProcAddress  
>     mov  edx,\[eax\]  
>     add  edx,ebx  
>     mov  dwGetProcAddress,edx  
>   }  
>    ///////////////////////////////////////////////////////////////////  
>   
>     myData->\_GetModuleHandleA= (GETMODULEHANDLEA)dwGetModuleHandleA;  
>     myData->\_LoadLibraryA    = (LOADLIBRARYA)    dwLoadLibraryA;  
>     myData->\_FreeLibrary     = (FREELIBRARY)     dwFreeLibrary;  
>     myData->\_GetProcAddress  = (GETPROCADDRESS)  dwGetProcAddress;  
>   
>     myData->\_LoadLibraryA(myData->szLibraryPath); // 加载 DLL  
>   ////////////////////////////////////////////////////  
>   //你可以在这里添加其他代码  
>   //////////////////////////////////////////////////  
> }  
>  \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
>    4,总结  
>    还有什么想说呢？你是不是想说“我 hook 了 LoadLibraryA 后 npggNT.des 是不是就无法载入了?” ,我想理论上是。上面的代码没有对  
>   
> LoadLibraryA 进行任何的检校，但是我们别忘了还有一个 NP 主进程 GameMon.des,这些事应该是它来干的。从反 npggNT.des 注入来说，我想这里  
>   
> 并没有比上一篇文章《反 NP 监视原理》更有价值，只是我们可以学习一点东西，明白一点东西而已。    
>   
>    最后是废话，我只是一个小菜鸟，跟我真正交流过的人都会非常认同这个观点，千万别来找我做挂，我只是对技术感兴趣。反 NP 监视、读写  
>   
> 游戏内存（NP 保护下）这两个工具在两个月前就已经发布在我所加入的所有技术 Q 群了，比两篇文章出现还早很多。我想既然文章都出来了，就  
>   
> 更没必要拿到看雪上面来浪费空间了，看看文章就知道怎么回事。实际上它们不会给你更大的惊喜，用过的朋友都知道。 

 4dd466d0e1e0cc123afbe504da2462ca