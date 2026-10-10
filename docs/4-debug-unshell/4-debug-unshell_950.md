

# q3 watcher 兄 unpack it 4 的脱壳过程

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-701.htm`](https://www.pediy.com/kssd/pediy07/pediy7-701.htm)

q3 watcher 兄 unpack it 4 的脱壳过程  
  
【破解作者】 wangli\_com  
  
【使用工具】 WinXP、Ollydbg、PEiD、LordPE、ImportREC 1.6F  
  
【破解平台】 WinXP  
  
【目标下载】[ http://www.pediy.com/bbs/pediy7/pediy7-701.rar](http://www.pediy.com/bbs/pediy7/pediy7-701.rar)  
  
【加壳方式】 inti+BlockInput  
  
【破解声明】 我是一菜青虫，偶得一点心得，愿与大家分享    
  
  
   一直想独立的练习脱一个壳，前一阵看见 q3 watcher 兄发的 unpack it 4，说是 anti 比较厉害，于是就想看一看，正好学习  
  
学习 anti，现在的壳对 Ollydbg 的防范真是越来越厉害，这个壳给我印象最深的就是 BlockInput 函数，作者  
  
把这个函数用到了及至，把系统代码搬到了壳中，一不小心就挂了，不过幸亏有脚本，不然重新走一编可是  
  
很累的，那么多的花指令和 anti，首先我自己对他的花指令进行了分析，总结一些特征码，在 DeJunk\_v0.12  
  
中用，很好用啊。一直用 F7 走也行，只是太累，前期跟的时候小瞧了这个壳，一直用 F7 走，走的我头晕眼花  
  
，后期就用花指令去除器 DeJunk\_v0.12 和脚本走，断断续续看了好长时间，感觉一个字“累”，不过，脱完  
  
之后，心情还是满愉快的。  
  
借用 jingulong 的小结一下：  
 1 多线程  
 2 BlockInput  
 3 GetTickCount  
 4 ZwQueryInformationProcess  
 5 ZwSetInformationThread  
 6 CheckRemoteDebuggerPresent  
 7 FindWindow  
 8 EnableWindow  
 9 Process32Next  
10 ReadProcessMemory  
11 GetProcessHeap  
12 ZwOpenProcess(...,&(clid=CsrGetProcessId()))  
13 N 多 RDTSC  
14 N^x 多花花  
15 IsDebuggerPresent  
16 改 CALL DWORD PTR DS:\[<&DLL.ApiName>\] 为 call 壳地址  
17 利用异常对 drx 清零  
18 用多个(同种异常+花花+LOOPD) 组成循环，循环次数=0x3005，改成 0x8 次就好了  
  
  
    开始我们的脱壳之旅，首先用 PEiD 看到 TLS CALL 的地址为 0043F556，\[0043F556\]=0043ee79  
  
要把\*ProcessHeap+10=40000060，清零，不然解码会出错，直接用如月写的 WMoS.dll 插件也行。  
  
  
一. TLS CallBack 中解码  
  
运行 OD 后 中断在系统断点,  f2 断点 到 TLS CallBack Entry.  
  
0043EE79    F2:                       prefix repne:     ----f2 断点  
  
  
  
0043F4FE    3102            xor dword ptr ds:\[edx\],eax    --eax=0000006E  
ds:\[0043DF70\]=F592A690     ;;ecx=8db  
  
0043F500    EB 01           jmp short RegClean.0043F503  
0043F502    90              nop  
0043F503    42              inc edx  
0043F504    EB 01           jmp short RegClean.0043F507  
0043F506    90              nop  
0043F507  ^ E2 F5           loopd short RegClean.0043F4FE  
  
//上面这段代码解压 43df70 开始，8db 长度的代码，eax 经过很长的换算。  
  
二、壳的解压代码  
  
用 PEID 查看入口点是 0043C107，下 f2 断点， 从 43C107-0043DC21 这段代码调用 BlockInput 函数，返回值要是 1  
  
0043C107    60                        pushad                    ---f2 断点  
0043C108    E8 00000000               call RegClean.0043C10D  
0043C10D    5D                        pop ebp  
0043C10E    81ED 3A1D0000             sub ebp,1D3A  
0043C114    8DB5 05080000             lea esi,dword ptr ss:\[ebp+805\]  
0043C11A    55                        push ebp  
0043C11B    56                        push esi  
0043C11C    81C5 9D3B0000             add ebp,3B9D  
0043C122    55                        push ebp  
0043C123    C3                        retn          --------返回到 0043DF70  
  
0043DF70    FEC8                      dec al  
  
  
0043E51D    8332 36         xor dword ptr ds:\[edx\],36  
0043E555    83C2 01         add edx,1  
0043E558    74 2A           je short RegClean.0043E584  
  
0043E594  ^\\E2 87           loopd short RegClean.0043E51D  
  
//上面这段代码解压 43e84b 开始，547 长度的代码  
  
0043E60A    0F31            rdtsc  
0043E60C    8BC8            mov ecx,eax  
0043E60E    0F31            rdtsc  
0043E610    2BC8            sub ecx,eax  
0043E612    F7D1            not ecx  
0043E614    81F9 33980100   cmp ecx,19833  
0043E61A    7F FF           jg short RegClean.0043E61B  
//一段时间效验的代码，下面还有很多  
  
  
0043EC16    AC              lods byte ptr ds:\[esi\]  
0043EC17    FEC0            inc al  
0043EC19    34 6B           xor al,6B  
0043EC1B    F8              clc  
0043EC1C    F9              stc  
0043EC1D    2AC1            sub al,cl  
0043EC1F    9B              wait  
0043EC20    F5              cmc  
0043EC21    F8              clc  
0043EC22    02C1            add al,cl  
0043EC24    90              nop  
0043EC25    C0C0 55         rol al,55  
0043EC28    C0C8 02         ror al,2  
0043EC41    34 60           xor al,60  
0043EC43    F9              stc  
0043EC44    34 A1           xor al,0A1  
0043EC46    FEC0            inc al  
0043EC48    34 6B           xor al,6B  
0043EC4A    F8              clc  
0043EC4B    F9              stc  
0043EC4C    2AC1            sub al,cl  
0043EC4E    9B              wait  
0043EC4F    F5              cmc  
0043EC50    F8              clc  
0043EC51    02C1            add al,cl  
0043EC53    90              nop  
0043EC54    C0C0 55         rol al,55  
0043EC57    C0C8 02         ror al,2  
0043EC7F    AA              stos byte ptr es:\[edi\]  
0043ECB5   /E2 77           loopd short RegClean.0043ED2E  
0043ECB7    51              push ecx  
0043ED2E  ^\\E9 E3FEFFFF     jmp RegClean.0043EC16  
  
  
//上面这段代码解压 43c124 开始，1e4c 长度的代码  
  
  
  
  
0043C701    8D75 54         lea esi,dword ptr ss:\[ebp+54\]----地址=0043C054, (ASCII "KERNEL32.dll")  
esi=0043DF70 (RegClean.0043DF70)  
  
0043C704    56              push esi  
0043C705    51              push ecx  
0043C706    8B4D 40         mov ecx,dword ptr ss:\[ebp+40\] -- ss:\[0043C040\]=77E59F93 (kernel32.GetModuleHandleA)  
ecx=00000000  
0043C709    E8 D4170000     call RegClean.0043DEE2      //检查 cc  
............................  
0043DEE2    8039 CC         cmp byte ptr ds:\[ecx\],0CC   //检查 cc  
0043DEE5    74 3A           je short RegClean.0043DF21  
0043DEEB    8079 01 CC      cmp byte ptr ds:\[ecx+1\],0CC  
0043DEEF    74 30           je short RegClean.0043DF21  
0043DEF5    8079 02 CC      cmp byte ptr ds:\[ecx+2\],0CC  
0043DEF9    74 26           je short RegClean.0043DF21  
0043DEFF    8079 03 CC      cmp byte ptr ds:\[ecx+3\],0CC  
0043DF03    74 1C           je short RegClean.0043DF21  
0043DF09    8079 04 CC      cmp byte ptr ds:\[ecx+4\],0CC  
0043DF0D    74 12           je short RegClean.0043DF21  
0043DF13    8079 05 CC      cmp byte ptr ds:\[ecx+5\],0CC  
0043DF17    74 08           je short RegClean.0043DF21  
0043DF1D    C3              retn  
...........................  
  
0043C772    FF55 40         call dword ptr ss:\[ebp+40\]          ; kernel32.GetModuleHandleA  
/取 KERNEL32.dll 的基址 77e40000  
0043C775    8DB5 DE000000   lea esi,dword ptr ss:\[ebp+DE\]   地址=0043C0DE, (ASCII "VirtualAlloc")  
  
0043C77B    56              push esi  
0043C77C    50              push eax  
0043C77D    51              push ecx  
0043C77E    8B4D 3C         mov ecx,dword ptr ss:\[ebp+3C\]  ----ss:\[0043C03C\]=77E5A5FD (kernel32.GetProcAddress)  
  
0043C781    E8 5C170000     call RegClean.0043DEE2  
0043C786    59              pop ecx  
0043C787    FF55 3C         call dword ptr ss:\[ebp+3C\]  --ss:\[0043C03C\]=77E5A5FD (kernel32.GetProcAddress)  
0043C78A    8985 EB000000   mov dword ptr ss:\[ebp+EB\],eax  
0043C790    59              pop ecx  
  
0043C791    E8 0C000000     call RegClean.0043C7A2  
0043C796  55 53 45 52 33 32 2E 64  USER32.d  
0043C79E  6C 6C 00 00              ll..  
  
  
0043C7A2    51              push ecx                        ; RegClean.0043ABD8  
0043C7A3    8B4D 44         mov ecx,dword ptr ss:\[ebp+44\]  ;kernel32.LoadLibraryA  
0043C7A6    E8 37170000     call RegClean.0043DEE2  //检查 cc  
0043C7AB    59              pop ecx  
0043C7AC    FF55 44         call dword ptr ss:\[ebp+44\]      ;kernel32.LoadLibraryA    
0043C7AF    83EC 04         sub esp,4   
  
0043C7B2    E8 0B000000     call RegClean.0043C7C2  
  
0043C7B7  42 6C 6F 63 6B 49 6E 70  BlockInp  
0043C7BF  75 74 00                 ut.  
  
  
0043C7C2    50              push eax                            ; User32.77D10000  
0043C7C3    51              push ecx  
0043C7C4    8B4D 3C         mov ecx,dword ptr ss:\[ebp+3C\] //kernel32.GetProcAddress  
0043C7C7    E8 16170000     call RegClean.0043DEE2       //检查 cc  
0043C7CC    59              pop ecx  
0043C7CD    FF55 3C         call dword ptr ss:\[ebp+3C\]  //kernel32.GetProcAddress  
0043C7D0    EB 23           jmp short RegClean.0043C7F5  
  
0043C8D2    04 6A           add al,6A  
0043C8D4    F2:             prefix repne:  
0043C8D5    FFD0            call eax          //调用 BlockInput 函数，返回值要是 1，他用返回值做一些效验，现在返回 0，以后会出错。  
  
0043C8D7    50              push eax  
0043C8D8    33C0            xor eax,eax  
0043C8DA    50              push eax  
0043C8DB    56              push esi  
0043C8DC    75 0F           jnz short RegClean.0043C8ED  
0043C967    51              push ecx  
0043C968    68 00FE98B7     push B798FE00  
0043C96D    50              push eax  
0043C96E    E8 5D000000     call RegClean.0043C9D0  
0043C9DB    FF30            push dword ptr ds:\[eax\]  
0043C9DD    E8 C9000000     call RegClean.0043CAAB  
  
  
  
//下面的循环会在以后多次出现，主要是效验用，eax 一般为零  
  
0043CD8F    B9 590F0000     mov ecx,0F59  
0043CDC9    0102            add dword ptr ds:\[edx\],eax------>3  
0043CDCB    EB 23           jmp short RegClean.0043CDF0  
0043CDF0    51              push ecx  
0043CDF1    E8 EDFFFFFF     call RegClean.0043CDE3  
0043CE2D    50              push eax  
0043CE2E    E8 EDFFFFFF     call RegClean.0043CE20  
0043CE00    42              inc edx                             ; RegClean.0043DF22  
0043CE01    74 2A           je short RegClean.0043CE2D  
0043CE03    EB 03           jmp short RegClean.0043CE08  
0043CE3D  ^\\E2 8A           loopd short RegClean.0043CDC9---->3  
0043CE3F    50              push eax  
  
  
  
  
seh：  
0043CF3F    64:FF35 0000000>push dword ptr fs:\[0\]  
0043CF46    64:8925 0000000>mov dword ptr fs:\[0\],esp  
0043CF4D    50              push eax  
0043CF4E    EB 01           jmp short RegClean.0043CF51  
0043CF50    90              nop  
0043CF51    FFFF            ???                                 ; 未知命令  
0043CF53    90              nop  
0043CF54    C3              retn  
  
0043CF5B    58              pop eax  
0043CF5C    64:8F05 0000000>pop dword ptr fs:\[0\]  
  
  
  
  
0043D51A    E8 00000000     call RegClean.0043D51F  
0043D51F    830424 1B       add dword ptr ss:\[esp\],1B  
0043D523    64:FF35 0000000>push dword ptr fs:\[0\]  
0043D52A    64:8925 0000000>mov dword ptr fs:\[0\],esp  
0043D531    CC              int3  
  
  
//seh：利用异常清硬件断点，下面还有很多。  
  
0043D53A    55              push ebp  
0043D53B    8BEC            mov ebp,esp  
0043D53D    53              push ebx  
0043D53E    8B45 10         mov eax,dword ptr ss:\[ebp+10\]  
0043D541    55              push ebp  
0043D542    8B98 B4000000   mov ebx,dword ptr ds:\[eax+B4\]  
0043D548    8BEB            mov ebp,ebx  
0043D54A    8D9D 00104000   lea ebx,dword ptr ss:\[ebp+401000\]  
0043D550    83EB 04         sub ebx,4  
0043D553    5D              pop ebp  
0043D554    8180 B8000000 9>add dword ptr ds:\[eax+B8\],92  
0043D55E    33DB            xor ebx,ebx  
0043D560    8958 04         mov dword ptr ds:\[eax+4\],ebx  
0043D563    8958 08         mov dword ptr ds:\[eax+8\],ebx  
0043D566    8958 0C         mov dword ptr ds:\[eax+C\],ebx  
0043D569    8958 10         mov dword ptr ds:\[eax+10\],ebx  
0043D56C    8160 14 F00FFFF>and dword ptr ds:\[eax+14\],FFFF0FF0  
0043D573    C740 18 5501000>mov dword ptr ds:\[eax+18\],155  
0043D57A    C700 17000100   mov dword ptr ds:\[eax\],10017  
0043D580    B8 00000000     mov eax,0  
0043D585    5B              pop ebx  
0043D586    C9              leave  
0043D587    C2 1000         retn 10  
  
0043D5C3    64:8F05 0000000>pop dword ptr fs:\[0\]  
0043D5CA    83C4 04         add esp,4  
0043D5CD    50              push eax  
0043D5CE    E8 24000000     call RegClean.0043D5F7  
  
  
  
//开始用 VirtualAlloc 函数申请空间，放置壳代码，申请的地址我机子上是 940000  
  
0043D94B    68 00100000     push 1000  
0043D98B    FFB5 DA000000   push dword ptr ss:\[ebp+DA\]    ;627169  
0043D9CD    6A 00           push 0  
0043DA70    8B8D EB000000   mov ecx,dword ptr ss:\[ebp+EB\]       ; kernel32.VirtualAlloc  
0043DB31    FF95 EB000000   call dword ptr ss:\[ebp+EB\]          ; kernel32.VirtualAlloc  
  
0012FFB0   00000000  |Address = NULL  
0012FFB4   00627169  |Size = 627169 (6451561.)  
0012FFB8   00001000  |AllocationType = MEM\_COMMIT  
0012FFBC   00000004  \\Protect = PAGE\_READWRITE  
  
0043DB58    50              push eax           ;940000  
0043DB59    8B9D D6000000   mov ebx,dword ptr ss:\[ebp+D6\]   ;ss:\[0043C0D6\]=00003CC0  
0043DB5F    03DD            add ebx,ebp              ;43c000+3cc0=43fcc0  
0043DB61    50              push eax  
0043DB62    53              push ebx  
  
  
0043DC21    C3              retn         返回到 00940000     //下面将进入壳代码，  
  
三、壳代码的主要部分。  
  
从 940000-0095473D 这段代码调用 BlockInput 函数，返回值要是 0  
  
009400BC    64:FF35 0000000>push dword ptr fs:\[0\]  
009400C3    64:8925 0000000>mov dword ptr fs:\[0\],esp  
009400CA    CD 01           int 1  
  
  
009400AC    8B6424 08       mov esp,dword ptr ss:\[esp+8\]  
009400B0    64:8F05 0000000>pop dword ptr fs:\[0\]  
009400B7    58              pop eax  
009400B8    EB 14           jmp short 009400CE  
  
00940213    C600 F2          mov byte ptr ds:\[eax\],0F2  
  
00940243    E8 10000000      call 00940258  
00940248    8B6424 08        mov esp,dword ptr ss:\[esp+8\]  
0094024C    64:8F05 00000000 pop dword ptr fs:\[0\]  
00940253    58               pop eax  
00940254    EB 14            jmp short 0094026A  
00940256    C783 64FF3500 00>mov dword ptr ds:\[ebx+35FF64\],64000>  
00940260    8925 00000000    mov dword ptr ds:\[0\],esp  
00940266    CD 01            int 1  
  
  
009404D0    8B6424 08       mov esp,dword ptr ss:\[esp+8\]  
009404D4    64:8F05 0000000>pop dword ptr fs:\[0\]  
009404DB    58              pop eax  
009404DC    EB 14           jmp short 009404F2  
009404DE    C78464 FF350000>mov dword ptr ss:\[esp+35FF\],8964000>  
009404E9    25 00000000     and eax,0  
009404EE    CD 01           int 1  
  
00940639    E8 24000000     call 00940662  
0094063E    8B4C24 0C       mov ecx,dword ptr ss:\[esp+C\]  
00940642    8381 B8000000 1>add dword ptr ds:\[ecx+B8\],11  
00940649    33C0            xor eax,eax  
0094064B    8941 04         mov dword ptr ds:\[ecx+4\],eax  
0094064E    8941 08         mov dword ptr ds:\[ecx+8\],eax  
00940651    8941 0C         mov dword ptr ds:\[ecx+C\],eax  
00940654    8941 10         mov dword ptr ds:\[ecx+10\],eax  
00940657    8941 14         mov dword ptr ds:\[ecx+14\],eax  
0094065A    C741 18 5501000>mov dword ptr ds:\[ecx+18\],155  
00940661    C3              retn  
00940662    33C0            xor eax,eax  
00940664    64:FF30         push dword ptr fs:\[eax\]  
00940667    64:8920         mov dword ptr fs:\[eax\],esp  
  
009406B7    64:8F05 0000000>pop dword ptr fs:\[0\]  
009406BE    EB 23           jmp short 009406E3  
  
0094081A    64:8F05 0000000>pop dword ptr fs:\[0\]  
00940821    51              push ecx  
00940822    EB 49           jmp short 0094086D  
  
  
009419C7    05 6C000000     add eax,6C             //eax=94198a+6c=9419f6    ecx=b3  
009419CC    8330 4D         xor dword ptr ds:\[eax\],4D  
009419CF    40              inc eax  
009419D0  ^ E2 FA           loopd short 009419CC  
009419D2    F2:             prefix repne:  
  
  
009419F6  58 E8 0B 00 00 00 55 53  X?...US  
009419FE  45 52 33 32 2E 64 6C 6C  ER32.dll  
  
00941A36  00 00 42 6C 6F 63 6B 49  ..BlockI  
00941A3E  6E 70 75 74 00           nput.  
  
00941A07    FF55 44         call dword ptr ss:\[ebp+44\]          ; kernel32.LoadLibraryA  
  
00941A43    50              push eax                            ; eax=77d10000  
00941A44    FF55 3C         call dword ptr ss:\[ebp+3C\]          ; kernel32.GetProcAddress  
  
00941AA3    83C4 04         add esp,4  
00941AA6    9D              popfd  
00941AA7    FFD0            call eax       ;eax=User32.BlockInput  
  
//返回值为 0  
  
  
  
  
  
00941DC2    FF65 48         jmp dword ptr ss:\[ebp+48\]           ; kernel32.ExitProcess  
  
  
  
00941E3E    B9 E5000000     mov ecx,0E5  
00941FA8    8330 51         xor dword ptr ds:\[eax\],51  
00941FAB    40              inc eax  
00941FAC  ^ E2 FA           loopd short 00941FA8  
//解压下面一段代码  
  
  
  
00942044  00 00 42 6C 6F 63 6B 49  ..BlockI  
0094204C  6E 70 75 74 00           nput.  
  
00941FF7    FF55 44         call dword ptr ss:\[ebp+44\]          ; kernel32.LoadLibraryA  
00942055    FF55 3C         call dword ptr ss:\[ebp+3C\]          ; kernel32.GetProcAddress  
009420F3    8039 C2         cmp byte ptr ds:\[ecx\],0C2  //User32.BlockInput ;ds:\[77D6935A\]=B8  
00942103    8039 E9         cmp byte ptr ds:\[ecx\],0E9  
0094214B    8039 CC         cmp byte ptr ds:\[ecx\],0CC  
00942158    8039 E8         cmp byte ptr ds:\[ecx\],0E8  
009421C8    8039 58         cmp byte ptr ds:\[ecx\],58  
00942234    8039 83         cmp byte ptr ds:\[ecx\],83  
00942294    8039 90         cmp byte ptr ds:\[ecx\],90  
  
0094229A  ^\\0F84 0BFFFFFF   je 009421AB  
009422A0    FFD0            call eax           User32.BlockInput  
  
  
0094259E    B9 1E010000     mov ecx,11E  
009425F2    0102            add dword ptr ds:\[edx\],eax  
009425F4    EB 01           jmp short 009425F7  
009425F7    83C2 04         add edx,4  
00942603  ^ E2 ED           loopd short 009425F2  
  
  
00942777    8330 00         xor dword ptr ds:\[eax\],0     //eax=9427bb  
0094277D    40              inc eax  
00942781  ^ E2 F4           loopd short 00942777  
  
  
\------------------------  
009427C9    64:A1 18000000  mov eax,dword ptr fs:\[18\]  
009427CF    8B48 30         mov ecx,dword ptr ds:\[eax+30\]  
009427D7    8B81 B0000000   mov eax,dword ptr ds:\[ecx+B0\]  // OSPlatformId  
009427DD    83F0 FE         xor eax,FFFFFFFE  
009427E3    C1E0 0E         shl eax,0E  
009427E6    0BC2            or eax,edx  
009427F1    C1E0 08         shl eax,8  
009427F4    0B81 A8000000   or eax,dword ptr ds:\[ecx+A8\]  //OSMinorVersion  
009427FF    C1E0 08         shl eax,8  
00942802    0B81 A4000000   or eax,dword ptr ds:\[ecx+A4\]   //OSMajorVersion  
00942811    84E4            test ah,ah  
00942813    74 43           je short 00942858         //不跳  
00942818    80FC 02         cmp ah,2  
0094281B    74 5D           je short 0094287A        //跳  
0094281D   /73 07           jnb short 00942826  
00942826    E8 02000000     call 0094282D  
0094282D    B8 36110000     mov eax,1136  
00942835    E8 03000000     call 0094283D  
0094283A   /EB 0F           jmp short 0094284B      //到这  
0094283D    8BD4            mov edx,esp  
0094283F    EB 01           jmp short 00942842  
00942842    0F34            sysenter  
  
//上面这段代码被重复了 N 编，把系统代码搬到了壳中。  
  
这段代码实际上就是调用 User32.BlockInput 函数：  
  
77D6935A >  B8 36110000               mov eax,1136  
77D6935F    BA 0003FE7F               mov edx,7FFE0300  
77D69364    FFD2                      call edx  
77D69366    C2 0400                   retn 4  
  
  
  
\---------------------------  
  
  
  
  
00942BC6    B9 15010000     mov ecx,115         
00942C23    0102            add dword ptr ds:\[edx\],eax   //edx=9430e2  
00942C28    83C2 04         add edx,4  
00942C2F  ^\\E2 F2           loopd short 00942C23      
  
  
//SEH  
00942D9A    50              push eax  
00942D9B    E8 24000000     call 00942DC4  
00942DA0    8B4C24 0C       mov ecx,dword ptr ss:\[esp+C\]  
00942DA4    8381 B8000000 1>add dword ptr ds:\[ecx+B8\],11  
00942DAB    33C0            xor eax,eax  
00942DAD    8941 04         mov dword ptr ds:\[ecx+4\],eax  
00942DB0    8941 08         mov dword ptr ds:\[ecx+8\],eax  
00942DB3    8941 0C         mov dword ptr ds:\[ecx+C\],eax  
00942DB6    8941 10         mov dword ptr ds:\[ecx+10\],eax  
00942DB9    8941 14         mov dword ptr ds:\[ecx+14\],eax  
00942DBC    C741 18 5501000>mov dword ptr ds:\[ecx+18\],155  
  
00942E97   /7A 4A           jpe short 00942EE3  
0094324C    8F05 00000000   pop dword ptr ds:\[0\]  
00943252    F2:             prefix repne:  
  
  
  
  
  
00953777    64:A1 30000000  mov eax,dword ptr fs:\[30\]      //7ffdf000  
0095377D    8B40 0C         mov eax,dword ptr ds:\[eax+C\]   //struct \_PEB\_LDR\_DATA \*Ldr  
  
00953783    8B40 0C         mov eax,dword ptr ds:\[eax+C\]  //InLoadOrderModuleList  ;241ee0  
00953786    8B48 14         mov ecx,dword ptr ds:\[eax+14\]  //BaseAddress  
0095378C    81E1 47030000   and ecx,347                   //ecx=0  
  
  
  
  
  
//作者一路上会友情提示，“registered”  
009441E7  72 65 67 69 73 74 65 72  register  
009441EF  65 64 00                 ed.  
  
  
  
  
00945170    8330 2A         xor dword ptr ds:\[eax\],2A  //ecx=b3,eax=945177  
00945173    40              inc eax  
00945174  ^ E2 FA           loopd short 00945170  
//解压下面一段代码  
  
  
00945187    FF55 44         call dword ptr ss:\[ebp+44\]          ; kernel32.LoadLibraryA  
009451ED    50              push eax  
009451EE    FF55 3C         call dword ptr ss:\[ebp+3C\]          ; kernel32.GetProcAddress  
  
00945458    58              pop eax  
00945459    833C24 01       cmp dword ptr ss:\[esp\],1  
0094545D    75 06           jnz short 00945465  
00945485    83C4 04         add esp,4  
00945488    9D              popfd  
00945489    FFD0            call eax                         //  User32.BlockInput  
  
00945844    B9 B3000000     mov ecx,0B3         
009458CA    8330 4D         xor dword ptr ds:\[eax\],4D  //ecx=b3,eax=9458f4  
009458CD    40              inc eax  
009458CE  ^ E2 FA           loopd short 009458CA  
//解压下面一段代码  
  
  
00945905    FF55 44         call dword ptr ss:\[ebp+44\]          ; kernel32.LoadLibraryA  
00945941    50              push eax  
00945942    FF55 3C         call dword ptr ss:\[ebp+3C\]          ; kernel32.GetProcAddress  
009459A1    83C4 04         add esp,4  
009459A4    9D              popfd  
009459A5    FFD0            call eax                 //  User32.BlockInput  
  
00945D3C    B9 E5000000     mov ecx,0E5  
00945EA6    8330 51         xor dword ptr ds:\[eax\],51   //ecx=e5,eax=945ee5  
00945EA9    40              inc eax  
00945EAA  ^ E2 FA           loopd short 00945EA6  
//解压下面一段代码  
  
  
  
00945EF5    FF55 44         call dword ptr ss:\[ebp+44\]          ; kernel32.LoadLibraryA  
00945F53    FF55 3C         call dword ptr ss:\[ebp+3C\]          ; kernel32.GetProcAddress  
  
00945FF1    8039 C2         cmp byte ptr ds:\[ecx\],0C2  
00946001    8039 E9         cmp byte ptr ds:\[ecx\],0E9  
00946049    8039 CC         cmp byte ptr ds:\[ecx\],0CC  
00946056    8039 E8         cmp byte ptr ds:\[ecx\],0E8  
009460C6    8039 58         cmp byte ptr ds:\[ecx\],58  
00946132    8039 83         cmp byte ptr ds:\[ecx\],83  
00946192    8039 90         cmp byte ptr ds:\[ecx\],90  
00946198  ^\\0F84 0BFFFFFF   je 009460A9  
0094619E    FFD0            call eax                            ; User32.BlockInput  
  
0094649C    B9 1E010000     mov ecx,11E  
009464F0    0102            add dword ptr ds:\[edx\],eax  //ecx=11e,eax=946815  
009464F5    83C2 04         add edx,4  
00946501  ^\\E2 ED           loopd short 009464F0  
//解压下面一段代码  
  
  
00946613    B9 D6000000     mov ecx,0D6  
00946675    8330 00         xor dword ptr ds:\[eax\],0   //ecx=0d6,eax=9466b8  
0094667B    40              inc eax  
0094667F  ^\\E2 F4           loopd short 00946675  
//解压下面一段代码  
  
  
009466C7    64:A1 18000000  mov eax,dword ptr fs:\[18\]  
009466CD    8B48 30         mov ecx,dword ptr ds:\[eax+30\]  
009466D5    8B81 B0000000   mov eax,dword ptr ds:\[ecx+B0\]  
009466DB    83F0 FE         xor eax,FFFFFFFE  
009466E1    C1E0 0E         shl eax,0E  
009466E4    0BC2            or eax,edx  
009466EF    C1E0 08         shl eax,8  
009466F2    0B81 A8000000   or eax,dword ptr ds:\[ecx+A8\]  
009466FD    C1E0 08         shl eax,8  
00946700    0B81 A4000000   or eax,dword ptr ds:\[ecx+A4\]  
0094670F    84E4            test ah,ah  
00946711    74 43           je short 00946756  
00946716    80FC 02         cmp ah,2  
00946719    74 5D           je short 00946778  
0094671B   /73 07           jnb short 00946724  
0094671D   |EB 01           jmp short 00946720  
0094671F   |90              nop  
00946720   |72 02           jb short 00946724  
00946722   |90              nop  
00946723   |90              nop  
00946724   \\E8 02000000     call 0094672B  
00946729    0F35            sysexit  
0094672B    B8 36110000     mov eax,1136  
00946730    EB 01           jmp short 00946733  
00946732    90              nop  
00946733    E8 03000000     call 0094673B  
00946738    EB 0F           jmp short 00946749  
0094673B    8BD4            mov edx,esp  
  
00946AC4    B9 15010000     mov ecx,115  
00946B21    0102            add dword ptr ds:\[edx\],eax  //ecx=115,eax=946fe4  
00946B26    83C2 04         add edx,4  
00946B2D  ^\\E2 F2           loopd short 00946B21  
//解压下面一段代码  
  
  
00946CB6    B9 19010000     mov ecx,119  
00946E2D    8330 5D         xor dword ptr ds:\[eax\],5D  //ecx=119,eax=946e68  
00946E30    40              inc eax  
00946E31  ^ E2 FA           loopd short 00946E2D  
//解压下面一段代码  
  
  
  
//进行 ZwQueryInformationProcess 的检查  
  
00946EB8  00 5A 77 51 75 65 72 79  .ZwQuery  
00946EC0  49 6E 66 6F 72 6D 61 74  Informat  
00946EC8  69 6F 6E 50 72 6F 63 65  ionProce  
00946ED0  73 73 00 E8 0A 00 00 00  ss.?...  
00946ED8  6E 74 64 6C 6C 2E 64 6C  ntdll.dl  
00946EE0  6C 00 50 51 0F CA F7 D2  l.PQ 树  
  
00946F17    FF55 44         call dword ptr ss:\[ebp+44\]          ; kernel32.LoadLibraryA  
00946F1A    50              push eax  
00946F1B    FF55 3C         call dword ptr ss:\[ebp+3C\]          ; kernel32.GetProcAddress  
00946F1E    8BF0            mov esi,eax                   ; ntdll.ZwQueryInformationProcess  
  
0094705E    FFD0            call eax                    1; ntdll.ZwQueryInformationProcess  
  
00947425    B9 7E010000     mov ecx,17E  
00947550    8330 58         xor dword ptr ds:\[eax\],58  //ecx=17e,eax=94758b  
00947553    40              inc eax  
00947554  ^ E2 FA           loopd short 00947550  
//解压下面一段代码  
  
  
009475D9    FF55 44         call dword ptr ss:\[ebp+44\]          ; kernel32.LoadLibraryA  
009475DC    50              push eax  
009475DD    FF55 3C         call dword ptr ss:\[ebp+3C\]          ; kernel32.GetProcAddress  
00947730    83C4 04         add esp,4  
00947733    9D              popfd  
00947734    FFD6            call esi            2; ntdll.ZwQueryInformationProcess  
00947736    9C              pushfd  
  
  
00947AD6    B9 6D010000     mov ecx,16D  
00947C0D    8330 35         xor dword ptr ds:\[eax\],35  //eax=16d,eax=947c4e  
00947C10    40              inc eax  
00947C11  ^ E2 FA           loopd short 00947C0D  
//解压下面一段代码  
  
  
00947CF2    FF55 44         call dword ptr ss:\[ebp+44\]          ; kernel32.LoadLibraryA  
00947CF5    50              push eax  
00947CF6    FF55 3C         call dword ptr ss:\[ebp+3C\]          ; kernel32.GetProcAddress  
  
00947DD5    8039 C2         cmp byte ptr ds:\[ecx\],0C2  
00947DE5    8039 E9         cmp byte ptr ds:\[ecx\],0E9  
00947E2D    8039 CC         cmp byte ptr ds:\[ecx\],0CC  
00947E3A    8039 E8         cmp byte ptr ds:\[ecx\],0E8  
00947EAA    8039 58         cmp byte ptr ds:\[ecx\],58  
00947F16    8039 83         cmp byte ptr ds:\[ecx\],83  
00947F76    8039 90         cmp byte ptr ds:\[ecx\],90  
00947F7C  ^\\0F84 0BFFFFFF   je 00947E8D  
00947F82    FFD1            call ecx            3; ntdll.ZwQueryInformationProcess  
00947F84    9C                 pushfd  
  
  
00948346    B9 16020000     mov ecx,216  
00948491    8330 68         xor dword ptr ds:\[eax\],68  //eax=216,eax=9484d2  
00948497    40              inc eax  
0094849B  ^\\E2 F4           loopd short 00948491  
0094849D    50                 push eax  
//解压下面一段代码  
  
  
0094860E    64:A1 18000000  mov eax,dword ptr fs:\[18\]  
00948617    8B48 30         mov ecx,dword ptr ds:\[eax+30\]  
00948623    8B81 B0000000   mov eax,dword ptr ds:\[ecx+B0\]  
0094862D    83F0 FE         xor eax,FFFFFFFE  
00948633    C1E0 0E         shl eax,0E  
00948641    0BC2            or eax,edx  
00948643    C1E0 08         shl eax,8  
0094864A    0B81 A8000000   or eax,dword ptr ds:\[ecx+A8\]  
00948659    C1E0 08         shl eax,8  
0094865C    0B81 A4000000   or eax,dword ptr ds:\[ecx+A4\]  
0094866B    84E4            test ah,ah  
00948671   /74 3C           je short 009486AF  
00948676    80FC 02         cmp ah,2  
0094867C   /74 57           je short 009486D5  
00948689    E8 03000000     call 00948691  
0094868E   /EB 02              jmp short 00948692  
00948691    B8 9A000000        mov eax,9A  
00948696    E8 03000000        call 0094869E  
0094869B    EB 0B              jmp short 009486A8  
0094869E    8BD4               mov edx,esp          //mov eax,0;   ret  
009486A0    EB 02              jmp short 009486A4  
009486A2    90                 nop  
009486A3    90                 nop  
009486A4    0F34               sysenter  
  
  
  
  
00949BC4    B9 B3000000       mov ecx,0B3    
00949C75    8330 2A           xor dword ptr ds:\[eax\],2A      //eax=0b6,eax=949c7c  
00949C78    40                inc eax  
00949C79  ^ E2 FA             loopd short 00949C75  
00949C7B    58                pop eax  
//解压下面一段代码  
  
  
00949F8C    04 9D             add al,9D  
00949F8E    FFD0              call eax         //  User32.BlockInput  
00949F90    9C                pushfd  
  
  
  
0094A349    B9 B3000000       mov ecx,0B3  
0094A3CF    8330 4D           xor dword ptr ds:\[eax\],4D  //eax=0b3,eax=94a3f9  
0094A3D2    40                inc eax  
0094A3D3  ^ E2 FA             loopd short 0094A3CF  
0094A3D5    F2:               prefix repne:  
//解压下面一段代码  
  
  
0094A4A6    83C4 04           add esp,4  
0094A4A9    9D                popfd  
0094A4AA    FFD0              call eax   //  User32.BlockInput  
0094A4AC    9C                pushfd  
  
0094AC9D  ^\\0F84 0BFFFFFF     je 0094ABAE  
0094ACA3    FFD0              call eax        //  User32.BlockInput  
0094ACA5    F2:               prefix repne:  
  
  
  
  
0094BA20    04 9D             add al,9D  
0094BA22    FFD0              call eax       //  User32.BlockInput  
0094BA24    9C                pushfd  
  
0094BDDD    B9 B3000000       mov ecx,0B3  
0094BE63    8330 4D           xor dword ptr ds:\[eax\],4D  
0094BE66    40                inc eax  
0094BE67  ^ E2 FA             loopd short 0094BE63  
//解压下面一段代码  
  
  
0094BF3C    04 9D             add al,9D  
0094BF3E    FFD0              call eax   //  User32.BlockInput  
0094BF40    9C                pushfd       
  
0094C731  ^\\0F84 0BFFFFFF     je 0094C642  
0094C737    FFD0              call eax    //  User32.BlockInput  
0094C739    F2:               prefix repne:  
  
  
  
0094CCCB    68 E8030000       push 3E8  
0094CCD0    00EB              add bl,ch  
0094CCD2    0FB9              ???                                  ; 未知命令  
0094CCD4    8BD4              mov edx,esp             //mov eax,0;   ret  
0094CCD6    EB 01             jmp short 0094CCD9  
0094CCD8    330F              xor ecx,dword ptr ds:\[edi\]  
0094CCDA    34 EB             xor al,0EB  
  
  
  
0094D05D    B9 15010000       mov ecx,115  
0094D0BA    0102              add dword ptr ds:\[edx\],eax  //ecx=115,edx=94d579  
0094D0BF    83C2 04           add edx,4  
0094D0C6  ^\\E2 F2             loopd short 0094D0BA  
//解压下面一段代码  
  
  
  
0094E634    B9 74100000       mov ecx,1074  
0094E6E7    8330 1C           xor dword ptr ds:\[eax\],1C   //ecx=1074,edx=94e6f4  
0094E6ED    40                inc eax  
0094E6F1  ^\\E2 F4             loopd short 0094E6E7  
//解压下面一段代码  
  
  
0094E735    81C1 08010000     add ecx,108  
  
0094E9F8    FF55 44           call dword ptr ss:\[ebp+44\]   ; kernel32.LoadLibraryA  
0094EA6B    FF55 3C           call dword ptr ss:\[ebp+3C\]   ; kernel32.GetProcAddress  
0094EA6E    50                push eax            ; kernel32.SetUnhandledExceptionFilter  
  
0094ED1D    8039 C2           cmp byte ptr ds:\[ecx\],0C2  
0094ED2D    8039 E9           cmp byte ptr ds:\[ecx\],0E9  
0094ED75    8039 CC           cmp byte ptr ds:\[ecx\],0CC  
0094ED86    8039 E8           cmp byte ptr ds:\[ecx\],0E8  
0094EDD6    8039 58           cmp byte ptr ds:\[ecx\],58  
0094EE42    8039 83           cmp byte ptr ds:\[ecx\],83  
0094EEA2    8039 90           cmp byte ptr ds:\[ecx\],90  
0094EEE3    FFD0              call eax         ; kernel32.SetUnhandledExceptionFilter  
0094EEE5    66:9C             pushfw               ;94e3f5  
0094EF19    85C0              test eax,eax  
0094EF5A   /0F84 AB000000     je 0094F00B  
0094EFE9   /0F85 FE000000     jnz 0094F0ED  
  
  
  
  
  
0094F8B6    50                push eax  
0094F8B7    E8 24000000       call 0094F8E0  
0094F8BC    8B4C24 0C         mov ecx,dword ptr ss:\[esp+C\]  
0094F8C0    8381 B8000000 11  add dword ptr ds:\[ecx+B8\],11  
0094F8C7    33C0              xor eax,eax  
  
0094FFF4    FF55 44           call dword ptr ss:\[ebp+44\]     ; kernel32.LoadLibraryA  
  
00954106    FF55 44           call dword ptr ss:\[ebp+44\]  
00954109    50                push eax                       
0095410A    FF55 3C           call dword ptr ss:\[ebp+3C\]           ; kernel32.GetProcAddress  
00954110    FFD0              call eax            ;ZwCheckDebuggerInformation          
00954115    85C0              test eax,eax             //eax=1  
0095411A   /75 0D             jnz short 00954129           
  
  
  
///异常后跳转到 94e3f5  
  
0094E3F5    66:9C                     pushfw  
0094E3F7    EB 04                     jmp short 0094E3FD  
  
///从 0095473D-00EE6213 这段代码调用 BlockInput 函数，返回值要是 1  
  
0095473D    64:A1 18000000    mov eax,dword ptr fs:\[18\]  
00954784    8B48 30           mov ecx,dword ptr ds:\[eax+30\]  
009547BB    8B81 B0000000     mov eax,dword ptr ds:\[ecx+B0\]  
009547FD    83F0 FE           xor eax,FFFFFFFE  
00954803    C1E0 0E           shl eax,0E  
00954886    0B81 A8000000     or eax,dword ptr ds:\[ecx+A8\]  
00954891    C1E0 08           shl eax,8  
009548F3    0B81 A4000000     or eax,dword ptr ds:\[ecx+A4\]  
00954902    84E4              test ah,ah  
009549AB    B8 36110000       mov eax,1136  
009549B0    EB 01             jmp short 009549B3  
009549B2    68 E8030000       push 3E8  
009549B7    00EB              add bl,ch  
009549B9    0FB9              ???                      
009549BB    8BD4              mov edx,esp              //这里的返回值是 1，mov eax,1;   ret  
009549BD    EB 01             jmp short 009549C0  
009549BF    330F              xor ecx,dword ptr ds:\[edi\]  
009549C1    34 EB             xor al,0EB  
  
00954A86    B9 390F0000        mov ecx,0F39  
00954B39    0102               add dword ptr ds:\[edx\],eax     //ecx=f39,edx=955eab  
00954B3E    42                 inc edx  
00954B43  ^\\E2 D3              loopd short 00954B18  
  
  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
下面最好写一个脚本，下面会有很多这样的代码，最后 edx=757,不知是不是代表个数。  
  
  
        find    eip, #64A118000000#             // mov eax,dword ptr fs:\[18\]  
        find    $RESULT, #B836110000#         // mov eax,1136  
        find    $RESULT, #8BD4#                  //mov edx,esp  
        mov     i, $RESULT  
        log     i  
        bp      i  
        run  
  
anti1：    
        bc  i  
        mov eax,1  
        add esp,4  
        sub i,3  
        mov eip,i  
       
       find    eip, #64A118000000#             // mov eax,dword ptr fs:\[18\]  
       find    $RESULT, #B836110000#         // mov eax,1136  
       find    $RESULT, #8BD4#                  //mov edx,esp  
       mov     i, $RESULT  
       log     i  
       bp      i  
       eob     anti1  
       run  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*    
  
  
0095511D    64:A1 18000000    mov eax,dword ptr fs:\[18\]  
00955352    68 E8030000       push 3E8  
00955357    00EB              add bl,ch  
00955359    0FB9              ???                                  ; 未知命令  
0095535B    8BD4              mov edx,esp                    //mov eax,1;   ret  
0095535D    EB 01             jmp short 00955360        
0095535F    330F              xor ecx,dword ptr ds:\[edi\]  
00955361    34 EB             xor al,0EB  
  
  
00958B36    68 E8030000       push 3E8  
00958B3B    00EB              add bl,ch  
00958B3D    0FB9              ???                                  ; 未知命令  
00958B3F    8BD4              mov edx,esp  
00958B41    EB 01             jmp short 00958B44  
  
  
00EE5618    64:A1 18000000   mov eax,dword ptr fs:\[18\]  
00EE561E    66:9C            pushfw  
00EE5620    EB 06            jmp short 00EE5628  
  
  
00EE6213    88745D 73         mov byte ptr ss:\[ebp+ebx\*2+73\],dh  
00EE6217    07                pop es  
00EE622D    68 E8030000       push 3E8  
00EE6232    00EB              add bl,ch  
00EE6234    0FB9              ???                                  ; 未知命令  
00EE6236    8BD4              mov edx,esp  
00EE6238    EB 01             jmp short 00EE623B  
  
  
//到这里这段垃圾代码终于结束了  
  
EAX 00000001  
ECX 7FFDF000  
EDX 00000757  
EBX 0012FC00  
ESP 0012FFB4  
EBP 0043EE6D RegClean.0043EE6D  
ESI 00953404  
EDI 009536FD  
EIP 00EE6233  
C 1  ES 0023 32bit 0(FFFFFFFF)  
P 1  CS 001B 32bit 0(FFFFFFFF)  
A 1  SS 0023 32bit 0(FFFFFFFF)  
Z 0  DS 0023 32bit 0(FFFFFFFF)  
S 1  FS 003B 32bit 7FFDE000(FFF)  
T 0  GS 0000 NULL  
D 0  
O 0  LastErr ERROR\_PROC\_NOT\_FOUND (0000007F)  
EFL 00000297 (NO,B,NE,BE,S,PE,L,LE)  
DR0 00000000  
DR1 00000001  
DR2 00000000  
DR3 00000000  
DR6 00000000  
DR7 00000000  
  
  
  
00EE9676    FFD0                      call eax          -------BlockInput  
00EE9678    66:9C                     pushfw  
00EE9D46    B9 55770000               mov ecx,7755  
00EEA0F1    8130 33713371             xor dword ptr ds:\[eax\],71337133  
00EEA12C    83C0 01                   add eax,1  
00EEA12F  ^\\E2 C0                     loopd short 00EEA0F1  
00EEA131    68 03EA17B3               push B317EA03  
//解压下面一段代码  
  
  
四、让我们开始新的旅行  
  
00EEAA02    64:FF35 30000000          push dword ptr fs:\[30\]  
00EEAA50    58                        pop eax  
00EEAB17    8B40 0C                   mov eax,dword ptr ds:\[eax+C\]  
00EEABE0    8B40 0C                   mov eax,dword ptr ds:\[eax+C\]  
ds:\[00241EAC\]=00241EE0  
eax=00241EA0  
00EEACA9    C740 20 03000000          mov dword ptr ds:\[eax+20\],3   ;SizeOfImage=5b000  
                                                       ; 花招，下面还有效验，最好不要改  
  
//下面开始创建 线程 2  
  
00EEFB1F     6A 00              push 0  
00EEFB56     FF7424 08          push dword ptr ss:\[esp+8\]  ;8  
00EEFCB0     51                 push ecx                   ;00eef35f   
00EEFCE8     6A 00              push 0  
00EEFD1F     6A 00              push 0  
00EEFD56     E8 0D000000        call 00EEFD68  
  
00EEFD5B  43 72 65 61 74 65 54 68  CreateTh  
00EEFD63  72 65 61 64 00           read.  
  
  
  
  
  
  
00EEFD89    E8 0D000000        call 00EEFD9B  
  
00EEFD8E  6B 65 72 6E 65 6C 33 32  kernel32  
00EEFD96  2E 64 6C 6C 00           .dll.  
  
00EEFD9B    FF55 44            call dword ptr ss:\[ebp+44\]           ; kernel32.LoadLibraryA  
00EEFDE2    50                 push eax                             ; kernel32.77E40000  
00EEFE00    FF55 3C            call dword ptr ss:\[ebp+3C\]         ; kernel32.GetProcAddress  
00EF00D2    8BC8               mov ecx,eax  
  
00EF00D7    8039 C2            cmp byte ptr ds:\[ecx\],0C2  
  
  
00EF029D    FFD0               call eax                             ; kernel32.CreateThread  
  
FF D0 66 9C EB 01 EB 57 EB 06 E8 00 21 72 54  
  
0012FFA4   00000000  |pSecurity = NULL  
0012FFA8   00000000  |StackSize = 0  
0012FFAC   00EEF35F  |ThreadFunction = 00EEF35F    //  线程 2：入口  
0012FFB0   00000008  |pThreadParm = 00000008  
0012FFB4   00000000  |CreationFlags = 0  
0012FFB8   00EEFB17  \\pThreadId = 00EEFB17  
  
00EF02D3    85C0                      test eax,eax  
  
00EF08DE    E8 0D000000               call 00EF08F0  
00EF08E3  6B 65 72 6E 65 6C 33 32  kernel32  
00EF08EB  2E 64 6C 6C 00           .dll.  
00EF092C    FF55 44                   call dword ptr ss:\[ebp+44\]  
  
00EF09BF    FF55 3C                   call dword ptr ss:\[ebp+3C\]  ; kernel32.GetProcAddress  
  
0012FFB0   77E40000  |hModule = 77E40000 (kernel32)  
0012FFB4   00EF08A3  \\ProcNameOrOrdinal = "Sleep"  
  
00EF0C64    8D85 F6040000             lea eax,dword ptr ss:\[ebp+4F6\] ;00ef1125  
  
00EF0CA4    64:FF30                   push dword ptr fs:\[eax\]  
  
00EF16F1    FFD0                      call eax      ;Sleep，在这设 00EEF35F 段点，会进入线程 2  
  
  
  
00EF80BC    FF55 3C                   call dword ptr ss:\[ebp+3C\]  
00EFB56E    FF10                      call dword ptr ds:\[eax\]   ;Sleep  
  
00EFB57E    B9 CB0E0000               mov ecx,0ECB           ;eax=efb659,ecx=0ecb  
00EFB58E    8330 33                   xor dword ptr ds:\[eax\],33  
00EFB591    83C0 01                   add eax,1  
00EFB594  ^ E2 F8                     loopd short 00EFB58E  
//解压下面一段代码  
  
  
00EFB68E    64:FF35 30000000          push dword ptr fs:\[30\]  
00EFB6F3    8B40 0C                   mov eax,dword ptr ds:\[eax+C\]  
00EFB72B    8B40 0C                   mov eax,dword ptr ds:\[eax+C\]  
00EFB7A7    8B40 18                   mov eax,dword ptr ds:\[eax+18\]   ; BaseAddress=0   
00EFB7AA   /74 2A                     je short 00EFB7D6  
  
  
  
//下面是线程 1 在循环检测一些软件：OLLYDBG、ODbyDYK、LordPE、ImportREC、PE Tools、WINHEX、EnjoyC32asm  
  
00EFBD72    6A 00                     push 0  
00EFBD74    E8 08000000               call 00EFBD81  
00EFBD79  4F 4C 4C 59 44 42 47 00  OLLYDBG.  
00EFBD81    E8 00000000               call 00EFBD86  
00EFBD86    58                        pop eax  
00EFBD87    2D 293C0000               sub eax,3C29  
00EFBD8C    8B00                      mov eax,dword ptr ds:\[eax\]   ; User32.FindWindowA  
  
00EFC2F4    FFD0                      call eax                ;FindWindowA  
00EFC2F6    85C0                      test eax,eax  
00EFC2F8  ^ 0F85 9BF1FFFF             jnz 00EFB499  
  
00EFC2FE    6A 00                     push 0  
00EFC300    E8 08000000               call 00EFC30D  
00EFC305  4F 44 62 79 44 59 4B 00  ODbyDYK.  
00EFC318    FF10                      call dword ptr ds:\[eax\]    ; User32.FindWindowA  
00EFC31A    85C0                      test eax,eax  
00EFC31C  ^ 0F85 77F1FFFF             jnz 00EFB499  
  
00EFC322    E8 1A000000               call 00EFC341  
00EFC327  5B 20 4C 6F 72 64 50 45  \[ LordPE  
00EFC32F  20 44 65 6C 75 78 65 20   Deluxe  
00EFC337  5D 20 62 79 20 79 6F 64  \] by yod  
00EFC33F  61 00                    a.  
00EFC34E    FF10                      call dword ptr ds:\[eax\]      ; User32.FindWindowA  
00EFC350    85C0                      test eax,eax  
00EFC352  ^ 0F85 41F1FFFF             jnz 00EFB499  
  
00EFC358    E8 37000000               call 00EFC394  
00EFC35D  A1 EE 20 20 49 6D 70 6F  ☆  Impo  
00EFC365  72 74 52 45 43 20 20 A1  rtREC     
00EFC3A1    FF10                      call dword ptr ds:\[eax\]       ; User32.FindWindowA  
00EFC3A3    85C0                      test eax,eax  
00EFC3A5  ^ 0F85 EEF0FFFF             jnz 00EFB499  
  
00EFC3AB    E8 41000000               call 00EFC3F1  
00EFC3B0  50 45 20 54 6F 6F 6C 73  PE Tools  
00EFC3B8  20 76 31 2E 35 20 58 6D   v1.5 Xm  
00EFC3FE    FF10                      call dword ptr ds:\[eax\]        ; User32.FindWindowA  
00EFC400    85C0                      test eax,eax  
00EFC402  ^ 0F85 91F0FFFF             jnz 00EFB499  
  
00EFC408    E8 07000000               call 00EFC414  
00EFC40D  57 49 4E 48 45 58 00     WINHEX.  
00EFC414    6A 00                     push 0  
00EFC421    FF10                      call dword ptr ds:\[eax\]         ; User32.FindWindowA  
00EFC423    85C0                      test eax,eax  
00EFC425  ^ 0F85 6EF0FFFF             jnz 00EFB499  
  
00EFC42B    E8 12000000               call 00EFC442  
00EFC430  20 2D 20 5B 45 6E 6A 6F   - \[Enjo  
00EFC438  79 20 43 33 32 61 73 6D  y C32asm  
00EFC442    6A 00                     push 0  
00EFC44F    FF10                      call dword ptr ds:\[eax\]       ; User32.FindWindowA  
00EFC451    85C0                      test eax,eax  
00EFC453  ^ 0F85 40F0FFFF             jnz 00EFB499  
  
00EFC459    E8 10000000               call 00EFC46E  
00EFC45E  66 6C 79 30 44 42 47 20  fly0DBG  
00EFC466  2D 20 5B 43 50 55 5D 00  - \[CPU\].  
00EFC47B    FF10                      call dword ptr ds:\[eax\]       ; User32.FindWindowA  
00EFC47D    85C0                      test eax,eax  
00EFC47F  ^ 0F85 14F0FFFF             jnz 00EFB499  
  
00EFC485    E8 35000000               call 00EFC4BF  
00EFC48A  A1 EE 20 20 49 6D 70 6F  ☆  Impo  
00EFC492  72 74 52 45 43 20 20 A1  rtREC    
00EFC4CC    FF10                      call dword ptr ds:\[eax\]         ; User32.FindWindowA  
00EFC4CE    85C0                      test eax,eax  
00EFC4D0  ^ 0F85 C3EFFFFF             jnz 00EFB499  
  
00EFC4D6    E8 32000000               call 00EFC50D  
00EFC4DB  50 45 20 54 6F 6F 6C 73  PE Tools  
00EFC4E3  20 76 31 2E 35 20 62 79   v1.5 by  
00EFC51A    FF10                      call dword ptr ds:\[eax\]         ; User32.FindWindowA  
00EFC51C    85C0                      test eax,eax  
00EFC51E  ^ 0F85 75EFFFFF             jnz 00EFB499  
  
00EFC524    B9 CB0E0000               mov ecx,0ECB         ;eax=efb659,ecx=0ecb  
00EFC534    8330 33                   xor dword ptr ds:\[eax\],33  
00EFC537    83C0 01                   add eax,1  
00EFC53A  ^ E2 F8                     loopd short 00EFC534  
00EFC53C    C3                        retn                          ;00EFB546  
  
  
00EFB546    64:FF35 30000000          push dword ptr fs:\[30\]  
00EFB54D    58                        pop eax  
00EFB54E    8B40 0C                   mov eax,dword ptr ds:\[eax+C\]  
00EFB551    8B40 0C                   mov eax,dword ptr ds:\[eax+C\]  
00EFB554    8378 20 03                cmp dword ptr ds:\[eax+20\],3  
00EFB558  ^\\0F85 3BFFFFFF             jnz 00EFB499  
  
00EFB56E    FF10                      call dword ptr ds:\[eax\]          ; kernel32.Sleep  
  
  
  
  
  
  
线程 2：入口 00EEF35F  
  
  
00EEF35F    E8 00000000               call 00EEF364  
00EEF40B    8B50 04                   mov edx,dword ptr ds:\[eax+4\]   ;ds:\[00EEF18D\]=00EE6402  
00EFCA48    66:9C                     pushfw  
00EFCA7C    B9 855A0000               mov ecx,5A85  
00EFCAF7    8D85 F6040000             lea eax,dword ptr ss:\[ebp+4F6\]  ;地址=00EFCFB8  
00EFCB37    64:FF30                   push dword ptr fs:\[eax\]  
00EFCBD1    CC                        int3  
00EFCBD2    90                        nop  
00EFCBEC    F7F3                      div ebx  
00EFCBEE    74 2A                     je short 00EFCC1A  
00EFD3C8   /0F83 74010000             jnb 00EFD542  
00EFD542    61                        popad  
00EFD543    E8 00000000               call 00EFD548  
  
  
00EFD5A2    8130 CCC30000             xor dword ptr ds:\[eax\],0C3CC  ;eax=efed8d,ecx=5a85  
00EFD5A8    40                        inc eax  
00EFD5A9  ^ E2 F7                     loopd short 00EFD5A2  
00EFD5AB    50                        push eax  
//解压下面一段代码  
  
  
  
  
00EFEDE9    6A 08                     push 8  
00EFF363    6A 00                     push 0  
00EFF39A    FF7424 08                 push dword ptr ss:\[esp+8\]  
00EFF52C    6A 00                     push 0  
00EFF563    6A 00                     push 0  
00EFF59A    E8 0D000000               call 00EFF5AC  
00EFF59F  43 72 65 61 74 65 54 68  CreateTh  
00EFF5A7  72 65 61 64 00           read.  
00EFF8EF    FF55 44                   call dword ptr ss:\[ebp+44\]  
00EFF954    FF55 3C                   call dword ptr ss:\[ebp+3C\]   ；CreateThread  
00EFFE25    85C0                      test eax,eax  
  
  
00F00157    FFD0                      call eax     ；CreateThread  
  
  
0106FF7C   00000000  |pSecurity = NULL  
0106FF80   00000000  |StackSize = 0  
0106FF84   00EFEEB3  |ThreadFunction = 00EFEEB3    // 线程 3：入口  
0106FF88   00000008  |pThreadParm = 00000008  
0106FF8C   00000000  |CreationFlags = 0  
0106FF90   00EFF35B  \\pThreadId = 00EFF35B  
  
00F00366    68 CC000000               push 0CC  
  
00F003DF    E8 0D000000               call 00F003F1  
00F003E4  6B 65 72 6E 65 6C 33 32  kernel32  
00F003EC  2E 64 6C 6C 00           .dll.  
00F0042D    FF55 44                call dword ptr ss:\[ebp+44\]     ; kernel32.LoadLibraryA  
00F004C0    FF55 3C                call dword ptr ss:\[ebp+3C\] ;kernel32.GetProcAddress  
  
00F00730    FFD0                      call eax         ;Sleep  在这设 00EFEEB3 段点，会进入线程 3  
00F00732    60                        pushad    
  
  
///下面的代码是线程 2 在检查非法软件，循环检查。  
  
  
00F013DE    64:A1 18000000            mov eax,dword ptr fs:\[18\]  
00F0141F    8B48 30                   mov ecx,dword ptr ds:\[eax+30\]  
00F01427    8B81 B0000000             mov eax,dword ptr ds:\[ecx+B0\]  
00F015C4    B8 E5000000               mov eax,0E5  
00F01600    E8 06000000               call 00F0160B  
00F01605    EB 01                     jmp short 00F01608  
00F0160B    8BD4                      mov edx,esp  
00F01611    0F34                      sysenter  
  
00F017D2   /E9 F8010000               jmp 00F019CF  
00F01CE2    2BD1                      sub edx,ecx  ;edx=23c,ecx=236  
00F02080    51                        push ecx  
00F01F76  ^\\0F89 17FEFFFF              jns 00F01D93  
00F02018  ^\\0F81 6EFFFFFF             jno 00F01F8C  
00F02436    E8 06000000               call 00F02441  
00F0243B  53 6C 65 65 70 00        Sleep.  
00F02453    FF55 44                   call dword ptr ss:\[ebp+44\]   ; kernel32.LoadLibraryA  
00F02457    FF55 3C                   call dword ptr ss:\[ebp+3C\]   ; kernel32.GetProcAddress  
00F02464    8901                      mov dword ptr ds:\[ecx\],eax   ；ecx=0f0245f  
  
//破坏代码  
00F02466    B9 1D000000               mov ecx,1D  
00F0247A    3110                      xor dword ptr ds:\[eax\],edx   ;0f02436  
00F0247C    40                        inc eax  
00F0247D  ^ E2 FB                     loopd short 00F0247A  
  
00F0247F    E8 19000000               call 00F0249D  
00F02484  43 72 65 61 74 65 54 6F 6F 6C 68 65 6C 70 33 32  CreateToolhelp32  
00F02494  53 6E 61 70 73 68 6F 74 00                       Snapshot.  
  
00F024AF    FF55 44                   call dword ptr ss:\[ebp+44\]   ; kernel32.LoadLibraryA  
00F024B2    50                        push eax  
00F024B3    FF55 3C                   call dword ptr ss:\[ebp+3C\]   ; kernel32.GetProcAddress  
00F024C0    8901                      mov dword ptr ds:\[ecx\],eax  ; ecx=0f024bb     eax=kernel32.CreateToolhelp32Snapshot  
  
//破坏代码  
00F024C2    B9 30000000               mov ecx,30  
00F024D6    3110                      xor dword ptr ds:\[eax\],edx   ;0f0247f  
00F024D8    40                        inc eax  
00F024D9  ^ E2 FB                     loopd short 00F024D6  
  
00F02501    FF55 44                   call dword ptr ss:\[ebp+44\]   ; kernel32.LoadLibraryA  
00F02504    50                        push eax  
00F02505    FF55 3C                   call dword ptr ss:\[ebp+3C\]  ; kernel32.GetProcAddress  
00F02512    8901                      mov dword ptr ds:\[ecx\],eax           ; ecx=0f0250d    eax=kernel32.Process32First  
  
//破坏代码  
00F02514    B9 26000000               mov ecx,26  
00F02519    0F31                      rdtsc  
00F02528    3110                      xor dword ptr ds:\[eax\],edx   ;;0f024db  
00F0252A    40                        inc eax  
00F0252B  ^ E2 FB                     loopd short 00F02528  
  
00F02550    FF55 44                   call dword ptr ss:\[ebp+44\]   ; kernel32.LoadLibraryA  
00F02553    50                        push eax                    ; kernel32.77E40000  
00F02554    FF55 3C                   call dword ptr ss:\[ebp+3C\]  ; kernel32.GetProcAddress  
00F02561    8901                      mov dword ptr ds:\[ecx\],eax           ; ecx=0f0255c    eax=kernel32.OpenProcess  
  
//破坏代码  
00F02563    B9 23000000               mov ecx,23  
00F02568    0F31                      rdtsc   
00F02577    3110                      xor dword ptr ds:\[eax\],edx      ;;0f0252d  
00F02579    40                        inc eax  
00F0257A  ^ E2 FB                     loopd short 00F02577  
  
00F025A1    FF55 44                   call dword ptr ss:\[ebp+44\]    ; kernel32.LoadLibraryA  
00F025A4    50                        push eax                      ; kernel32.77E40000  
00F025A5    FF55 3C                   call dword ptr ss:\[ebp+3C\] ; kernel32.GetProcAddress  
00F025B2    8901                      mov dword ptr ds:\[ecx\],eax ;  
EAX =77E9A6E9 kernel32.Process32Next     ecx=f025ad  
  
//破坏代码  
00F025B4    B9 25000000               mov ecx,25  
00F025B9    0F31                      rdtsc  
00F025C8    3110                      xor dword ptr ds:\[eax\],edx    ;;0f0257c  
00F025CA    40                        inc eax  
00F025CB  ^ E2 FB                     loopd short 00F025C8  
  
00F025ED    FF55 44                   call dword ptr ss:\[ebp+44\]   ; kernel32.LoadLibraryA  
00F025F0    50                        push eax  
00F025F1    FF55 3C                   call dword ptr ss:\[ebp+3C\]  
00F025FE    8901                      mov dword ptr ds:\[ecx\],eax   ; User32.BlockInput  
  
00F02600    B9 20000000               mov ecx,20  
00F02605    0F31                      rdtsc  
00F02614    3110                      xor dword ptr ds:\[eax\],edx  
00F02616    40                        inc eax  
00F02617  ^ E2 FB                     loopd short 00F02614  
  
00F02642    FF55 44                   call dword ptr ss:\[ebp+44\]   ; kernel32.LoadLibraryA  
00F02645    50                        push eax  
00F02646    FF55 3C                   call dword ptr ss:\[ebp+3C\]  
00F02653    8901                      mov dword ptr ds:\[ecx\],eax  ; kernel32.ReadProcessMemory  
  
00F02696    FF55 44                   call dword ptr ss:\[ebp+44\]   ; kernel32.LoadLibraryA  
00F02699    50                        push eax  
00F0269A    FF55 3C                   call dword ptr ss:\[ebp+3C\]  
00F026A7    8901                      mov dword ptr ds:\[ecx\],eax   ; kernel32.TerminateProcess  
  
00F026C2    6A 00                     push 0  
00F026C4    6A 02                     push 2  
00F026D1    FF10                      call dword ptr ds:\[eax\]    ; kernel32.CreateToolhelp32Snapshot  
00F026F3    C706 28010000             mov dword ptr ds:\[esi\],128      ;esi=0f017db  
00F0271E    FF10                      call dword ptr ds:\[eax\]     ; kernel32.Process32First  
00F02720    E8 46000000               call 00F0276B  
00F02725    E8 00000000               call 00F0272A  
00F0274C    FF10                      call dword ptr ds:\[eax\]      ; kernel32.Process32Next  
00F0274E    0BC0                      or eax,eax  
00F02750    74 02                     je short 00F02754  
00F02752  ^\\EB CC                     jmp short 00F02720  
00F02754    68 93000000               push 93  
00F02764    FF10                      call dword ptr ds:\[eax\]              ; kernel32.Sleep  
00F02766  ^\\E9 57FFFFFF               jmp 00F026C2  
00F0276B    51                        push ecx  
  
  
  
00F0278E    FF10                      call dword ptr ds:\[eax\]      ; kernel32.OpenProcess  
  
0106FFA4    00000011  |Access = TERMINATE|VM\_READ  
0106FFA8    00000000  |Inheritable = FALSE  
0106FFAC    00000000  \\ProcessId = 0  
  
00F02790    0BC0                      or eax,eax  
00F02792    0F84 B3170000             je 00F03F4B  
00F02798    8BF0                      mov esi,eax  
  
00F027B7    60                        pushad  
00F027E0    8B00                      mov eax,dword ptr ds:\[eax\] ; kernel32.ReadProcessMemory  
00F02D48    FFD0                      call eax             ; kernel32.ReadProcessMemory  
  
0106FF7C    00000001  |hProcess = 00000001  
0106FF80    00407622  |pBaseAddress = 407622  
0106FF84    00F01903  |Buffer = 00F01903  
0106FF88    00000010  |BytesToRead = 10 (16.)  
0106FF8C    00F01913  \\pBytesRead = 00F01913  
  
00F02D4A    0BC0                      or eax,eax  
00F02D4C    61                        popad  
00F02D4D    0F84 F8110000             je 00F03F4B  
00F03F4B    C3                        retn         ;f02752  
  
00F02D53    83C7 04                   add edi,4  
00F02D56    57                        push edi  
00F02D63    E8 E4110000               call 00F03F4C  
00F03F4C    56                        push esi  
00F03F4D    57                        push edi  
00F03F4E    8B7C24 0C                 mov edi,dword ptr ss:\[esp+C\]   ；f0191f  
00F03F52    8B7424 10                 mov esi,dword ptr ss:\[esp+10\]  ; f01903  
00F03F56    A7                        cmps dword ptr ds:\[esi\],dword ptr es>  
00F03F57   /EB 23                     jmp short 00F03F7C  
  
  
od 的内存 :407622:8B 47 1C 89 45        比较      00F0191F : 8B 47 1C 89              
  
00F03F8C   /0F85 CA040000             jnz 00F0445C  
00F03FD3    A7                        cmps dword ptr ds:\[esi\],dword ptr es>  
00F03FD4    74 2A                     je short 00F04000  
00F047CC  ^\\0F89 17FEFFFF             jns 00F045E9  
00F04811    C3                        retn    ；f02d68  
  
00F02D68    83C4 08                   add esp,8  
00F02D6B    9C                        pushfd  
00F03005    83F8 01                   cmp eax,1  
00F03234   /0F84 EC000000             je 00F03326  
00F0323A   |EB 23                     jmp short 00F0325F  
00F032A6   /E2 38                     loopd short 00F032E0  
00F03321  ^\\E9 91F4FFFF               jmp 00F027B7  
  
  
  
      
  
线程 3：入口  00EFEEB3  
  
在线程 3 中会调用三次 GetTickCount，都是做效验用的，只要使返回值第一次等于 1，第二次等于 2，第三  
次等于 3 就行了。  
  
  
00EFEEB3    E8 00000000               call 00EFEEB8  
00F04A66    8D85 F6040000             lea eax,dword ptr ss:\[ebp+4F6\]  ；00f04f27  
00F05480    9C                        pushfd  
00F054C2    E8 12000000               call 00F054D9  
00F054C7  4C 6F 63 6B 57 69 6E 64  LockWind  
00F054CF  6F 77 53 74 61 74 69 6F  owStatio  
00F054D7  6E 00                    n.  
  
00F0560B    E8 24000000               call 00F05634  
00F05698    B8 30000000               mov eax,30  
  
00F057D6    85C9                      test ecx,ecx  
00F057D8    EB 32                     jmp short 00F0580C  
00F0580F   /0F84 B4000000             je 00F058C9        //跳  
  
00F058FE    E8 0D000000               call 00F05910  
00F05903  47 65 74 54 69 63 6B 43  GetTickC  
00F0590B  6F 75 6E 74 00           ount.  
  
00F05F3E    FFD0                      call eax         ; GetTickCount  第一次，返回 1  
00F05F40    66:9C                     pushfw  
00F05F74    85C0                      test eax,eax  
  
00F063AB    E8 0D000000               call 00F063BD  
00F063B0  6B 65 72 6E 65 6C 33 32  kernel32  
00F063B8  2E 64 6C 6C 00           .dll.  
00F063BD    E8 0C000000               call 00F063CE  
00F063C2  45 78 69 74 50 72 6F 63  ExitProc  
00F063CA  65 73 73 00              ess.  
00F0643C    E8 0D000000               call 00F0644E  
00F06441  6B 65 72 6E 65 6C 33 32  kernel32  
00F06449  2E 64 6C 6C 00           .dll.  
00F0644E    E8 1A000000               call 00F0646D  
00F06453  44 65 62 75 67 53 65 74  DebugSet  
00F0645B  50 72 6F 63 65 73 73 4B  ProcessK  
00F06463  69 6C 6C 4F 6E 45 78 69  illOnExi  
00F0646B  74 00                    t.  
  
00F064B5    64:8B00                   mov eax,dword ptr fs:\[eax\]  
00F06516    8B01                      mov eax,dword ptr ds:\[ecx\]  
  
00F066DC    B9 622E0000               mov ecx,2E62  
  
00F0687A    8330 33                   xor dword ptr ds:\[eax\],33   ;eax=f06904,ecx=2e26  
00F068B2    83C0 01                   add eax,1  
00F068B8  ^\\E2 C0                     loopd short 00F0687A  
00F068BA    50                        push eax  
//解压下面一段代码  
  
  
00F06C12    E8 0D000000               call 00F06C24  
00F06C24    E8 0C000000               call 00F06C35  
00F06C29  47 65 74 46 69 6C 65 53  GetFileS  
00F06C31  69 7A 65 00              ize.  
00F06CCC    E8 12000000               call 00F06CE3  
00F06CD1  55 6E 68 6F 6F 6B 57 69  UnhookWi  
00F06CD9  6E 64 6F 77 73 48 6F 6F  ndowsHoo  
00F06CE1  6B 00                    k.  
  
00F072B5    E8 0D000000               call 00F072C7  
00F072BA  47 65 74 54 69 63 6B 43  GetTickC  
00F072C2  6F 75 6E 74 00           ount.  
  
00F0756B    FFD0                      call eax   ；kernel32.GetTickCount   第二次，返回 2  
00F0756D    66:9C                     pushfw  
00F075E2   /0F84 AB000000             je 00F07693  
  
//ZwSetInformationThread 检查  
  
00F0883E    64:A1 18000000            mov eax,dword ptr fs:\[18\]  
00F0887F    8B48 30                   mov ecx,dword ptr ds:\[eax+30\]  
00F08887    8B81 B0000000             mov eax,dword ptr ds:\[ecx+B0\]  ;OSPlatformId  
00F08972    0B81 A8000000             or eax,dword ptr ds:\[ecx+A8\]    ;OSMinorVersion  
00F089EE    0B81 A4000000             or eax,dword ptr ds:\[ecx+A4\]  ;OSMajorVersion  
  
  
00F08A24    B8 E5000000               mov eax,0E5     ;ZwSetInformationThread  
00F08A60    E8 06000000               call 00F08A6B  
00F08A65    EB 01                     jmp short 00F08A68  
00F08A6B    8BD4                      mov edx,esp  
00F08A71    0F34                      sysenter  
  
00F0954A    E8 0A000000               call 00F09559  
00F0954F  47 65 74 57 69 6E 64 6F  GetWindo  
00F09557  77 00                    w.  
  
  
00F09992    E8 AF000000               call 00F09A46  
00F09A46    64:FF35 00000000          push dword ptr fs:\[0\]      ;eax=8  ，F2 段点  
00F09A4D    64:8925 00000000          mov dword ptr fs:\[0\],esp  
00F09A54    CC                        int3  
  
  
\----------------------------------------------------  
这里有用多个(同种异常+花花+LOOPD) 组成循环，循环次数=3005,在 00F09A46 处设个 F2 段点， 令 eax=8  
  
让他执行一遍就出来了。  
  
00f09a54-f0a81a-f0b4b9-f0c20e-f0cead-f0dc73-f0e912-f0f667-f10306-f110ca-f11d48-f12ad7-f13755-f144ed-f1516b-f15ef1-f16b6f-f178f9-f18577-f0a81a  
  
  
出口：//f1516b-f188a6-f18d53-f19431-f199ce  
  
  
  
00F0B4F9   /E2 49                     loopd short 00F0B544   /  
00F0CF28   /E2 7F                     loopd short 00F0CFA9   /  
00F0E98D   /E2 0E                     loopd short 00F0E99D   /  
00F10381   /E2 7D                     loopd short 00F10400  
00F11DC3   /E2 48                     loopd short 00F11E0D  
00F137D0   /E2 51                     loopd short 00F13823  
00F151E6   /E2 3F                     loopd short 00F15227  
00F16BEA   /E2 43                     loopd short 00F16C2F  
00F185F2   /E2 54                     loopd short 00F18648  
  
\---------------------------------------------------------------  
00F1945C    E8 1E000000               call 00F1947F  
00F19461  4E 74 51 75 65 72 79 53 79 73 74 65 6D 45 6E 76  NtQuerySystemEnv  
00F19471  69 72 6F 6E 6D 65 6E 74 56 61 6C 75 65 00        ironmentValue.  
00F195ED   /E9 84000000               jmp 00F19676  
  
00F1985B    E8 18000000               call 00F19878  
00F19860  4E 74 53 65 74 49 6E 66 6F 72 6D 61 74 69 6F 6E  NtSetInformation  
00F19870  50 72 6F 63 65 73 73 00                          Process.  
  
  
  
  
  
00F1A937    8330 7A                   xor dword ptr ds:\[eax\],7E  
00F1A943    40                        inc eax  
00F1A948  ^\\E2 ED                     loopd short 00F1A937  
//解压下面一段代码  
  
  
  
00F1B687    E8 14000000               call 00F1B6A0  
00F1B68C  47 65 74 46 6F 72 65 67 72 6F 75 6E 64 57 69 6E  GetForegroundWin  
00F1B69C  64 6F 77 00                                      dow.  
  
00F1B77B    FF55 44                   call dword ptr ss:\[ebp+44\]     ; kernel32.LoadLibraryA  
00F1B814    FF55 3C                   call dword ptr ss:\[ebp+3C\]   ; kernel32.GetProcAddress  
                                               ;EAX 77D1456B User32.GetForegroundWindow  
  
00F1C191    E8 0D000000               call 00F1C1A3  
00F1C196  45 6E 61 62 6C 65 57 69 6E 64 6F 77 00           EnableWindow.  
  
00F1C582    FF55 3C                   call dword ptr ss:\[ebp+3C\]   ; kernel32.GetProcAddress  
00F1C7E4    FFD0                      call eax                      ; User32.EnableWindow  
  
0116FF90    00090142  |hWnd = 00090142 ('flyODBG - RegClean.exe - \[CPU...',class='fly\*OD\*',wndproc=03A0314C)  
0116FF94    00000000  \\Enable = FALSE        ；这里改成 1，不然的话，鼠标就不会动了  
  
  
  
  
///效验出错的话会死在这里：  
00F1D95F    C45452 71                 les edx,fword ptr ds:\[edx+edx\*2+71\]  
  
  
  
00F1D352    E8 10000000               call 00F1D367  
00F1D357  5A 77 53 75 73 70 65 6E 64 54 68 72 65 61 64 00  ZwSuspendThread.  
00F2616D    FF55 3C                   call dword ptr ss:\[ebp+3C\]    ; kernel32.GetProcAddress  
  
00F1DF8A    E8 0F000000               call 00F1DF9E  
00F1DF8F  5A 77 43 72 65 61 74 65 54 68 72 65 61 64 00     ZwCreateThread.  
  
00F1F37E    E8 06000000               call 00F1F389  
00F1F383    EB 01                     jmp short 00F1F386  
00F1F389    8BD4                      mov edx,esp  
00F1F38F    0F34                      sysenter  
  
  
  
  
00F2616D    FF55 3C                   call dword ptr ss:\[ebp+3C\]   ；kernel32.GetTickCount  
  
00F26370    FFD0                      call eax        ；kernel32.GetTickCount  第二次，返回 3  
00F26372    66:9C                     pushfw  
  
00F26936    8D85 F6040000             lea eax,dword ptr ss:\[ebp+4F6\]  
  
  
00F29B90    E8 06000000               call 00F29B9B  
00F29B95    EB 01                     jmp short 00F29B98  
00F29B9B    8BD4                      mov edx,esp  
00F29BA1    0F34                      sysenter  
  
00F2C1F9    90                        nop  
00F2C1FA    FF55 3C                   call dword ptr ss:\[ebp+3C\]  
  
                                   ;EAX = kernel32.UnhandledExceptionFilter  
  
  
  
00F2DB1F    E8 06000000               call 00F2DB2A  
00F2DB24    EB 01                     jmp short 00F2DB27  
00F2DB2A    8BD4                      mov edx,esp  
00F2DB30    0F34                      sysenter  
  
00F2F8A7    8332 6E                   xor dword ptr ds:\[edx\],6E  ;ecx=c7ba,edx=f31542  
00F2F8E6    83C2 01                   add edx,1  
00F2F91E  ^\\E2 87                     loopd short 00F2F8A7  
//解压下面一段代码  
  
  
  
00F30BBC    E8 0F000000               call 00F30BD0  
00F30BC1  5A 77 43 72 65 61 74 65 54 68 72 65 61 64 00     ZwCreateThread.  
00F32806    FF55 3C                   call dword ptr ss:\[ebp+3C\] ; kernel32.GetProcAddress  
       EAX 77D1456B User32.GetForegroundWindow  
00F32A5D    FFD0                      call eax          ; User32.GetForegroundWindow  
  
00F3322B    E8 0D000000               call 00F3323D  
00F33230  45 6E 61 62 6C 65 57 69 6E 64 6F 77 00           EnableWindow.  
  
00F33851    FFD0                      call eax           ； EnableWindow.  
00F33853    66:9C                     pushfw                 
  
0116FF90   00020102  |hWnd = 00020102 ('flyODBG - asdf.exe - \[CPU - t...',class='fly\*OD\*',wndproc=03A0314C)  
0116FF94   00000001  \\Enable = TRUE  
  
  
  
  
  
  
00F36B53    E8 06000000               call 00F36B5E  
00F36B58    EB 01                     jmp short 00F36B5B  
00F36B5E    8BD4                      mov edx,esp  
00F36B64    0F34                      sysenter  
  
//开始查看系统进程  
  
00F37D07    FF55 3C                   call dword ptr ss:\[ebp+3C\]  
00F37D0A    FFD0                      call eax         ; kernel32.CreateToolhelp32Snapshot  
0116FF90   00000002  |Flags = TH32CS\_SNAPPROCESS  
0116FF94   00000000  \\ProcessID = 0  
  
  
00F37D0C    51                        push ecx         ;eax=40  
00F37D0D    E8 00000000               call 00F37D12  
00F37D12    59                        pop ecx  
00F37D13    81E9 720D0000             sub ecx,0D72  
00F37D19    8901                      mov dword ptr ds:\[ecx\],eax   ;ecx=f36fa0,eax=40  
00F37D1C    E8 14000000               call 00F37D35  
00F37D21  47 65 74 43 75 72 72 65 6E 74 50 72 6F 63 65 73  GetCurrentProces  
00F37D31  73 49 64 00                                      sId.  
00F37D35    E8 0D000000               call 00F37D47  
  
  
00F37D47    FF55 44                   call dword ptr ss:\[ebp+44\]  ; kernel32.LoadLibraryA  
00F37D4A    50                        push eax  
00F37D4B    FF55 3C                   call dword ptr ss:\[ebp+3C\]   ; kernel32.GetProcAddress  
00F37D4E    FFD0                      call eax                ; kernel32.GetCurrentProcessId  
00F37D50    51                        push ecx        ;eax=578     
                  
77E60656 >  64:A1 18000000            mov eax,dword ptr fs:\[18\]  
77E6065C    8B40 20                   mov eax,dword ptr ds:\[eax+20\]  
77E6065F    C3                        retn  
  
  
00F37D5D    8901                      mov dword ptr ds:\[ecx\],eax  ;ecx=f36f98, ;eax=108       
00F37D70    C706 28010000             mov dword ptr ds:\[esi\],128  ;esi=f36fa4,   
00F37D8E    FF31                      push dword ptr ds:\[ecx\]     ;ecx=f36fa0 ,40  
  
00F37D90    E8 0F000000               call 00F37DA4  
00F37D95  50 72 6F 63 65 73 73 33  Process3  
00F37D9D  32 46 69 72 73 74 00     2First.  
  
00F37DB6    FF55 44                   call dword ptr ss:\[ebp+44\] ; kernel32.LoadLibraryA  
00F37DB9    50                        push eax  
00F37DBA    FF55 3C                   call dword ptr ss:\[ebp+3C\] ; kernel32.GetProcAddress  
00F37DBD    FFD0                      call eax              ; kernel32.Process32First  
  
0116FF90   00000040  |hSnapshot = 00000040 (window)  
0116FF94   00F36FA4  \\pProcessentry = 00F36FA4  
  
00F37DDC    8B19                      mov ebx,dword ptr ds:\[ecx\]  ;\[00F36F98\]=578  
00F37DDF    3B5E 08                   cmp ebx,dword ptr ds:\[esi+8\]  ;\[esi+8\]=0  
00F37DE2    75 00                     jnz short 00F37DE4  
00F37DE4    E8 00000000               call 00F37DE9  
  
  
  
00F37DFC    FF31                      push dword ptr ds:\[ecx\]   ;;\[00F36FA0\]=40  
00F37DFE    E8 0E000000               call 00F37E11  
00F37E03  50 72 6F 63 65 73 73 33  Process3  
00F37E0B  32 4E 65 78 74 00        2Next.  
00F37E23    FF55 44                   call dword ptr ss:\[ebp+44\]    ; kernel32.LoadLibraryA  
00F37E26    50                        push eax  
00F37E27    FF55 3C                   call dword ptr ss:\[ebp+3C\]   ; kernel32.GetProcAddress  
00F37E2A    FFD0                      call eax           ; kernel32.Process32Next  
  
0116FF90   00000040  |hSnapshot = 00000040 (window)  
0116FF94   00F36FA4  \\pProcessentry = 00F36FA4  
  
00F37E4C    3B5E 08                   cmp ebx,dword ptr ds:\[esi+8\]  
00F37E4F  ^ 75 93                     jnz short 00F37DE4  
00F37E51    FF76 18                   push dword ptr ds:\[esi+18\]  
  
00F37E93    FF55 44                   call dword ptr ss:\[ebp+44\]  
00F37E96    50                        push eax  
00F37E97    FF55 3C                   call dword ptr ss:\[ebp+3C\]  
00F37E9A    FFD0                      call eax              ; kernel32.CloseHandle  
  
00F37E9C    6A 00                     push 0  
00F37E9E    6A 02                     push 2  
00F37EA0    E8 19000000               call 00F37EBE  
00F37EA5  43 72 65 61 74 65 54 6F  CreateTo  
00F37EAD  6F 6C 68 65 6C 70 33 32  olhelp32  
00F37EB5  53 6E 61 70 73 68 6F 74  Snapshot  
  
00F37ED0    FF55 44                   call dword ptr ss:\[ebp+44\]   ; kernel32.LoadLibraryA  
00F37ED3    50                        push eax  
00F37ED4    FF55 3C                   call dword ptr ss:\[ebp+3C\]  
00F37ED7    FFD0                      call eax        ; kernel32.CreateToolhelp32Snapshot  
  
00F37F2F    FF55 44                   call dword ptr ss:\[ebp+44\]    ; kernel32.LoadLibraryA  
00F37F32    50                        push eax  
00F37F33    FF55 3C                   call dword ptr ss:\[ebp+3C\]  
00F37F36    FFD0                      call eax             ; kernel32.Process32First  
00F37F38    51                        push ecx           ;eax=1  
  
00F37F9C    FF55 44                   call dword ptr ss:\[ebp+44\]    ; kernel32.LoadLibraryA  
00F37F9F    50                        push eax  
00F37FA0    FF55 3C                   call dword ptr ss:\[ebp+3C\]  
00F37FA3    FFD0                      call eax            ; kernel32.Process32Next  
  
00F37FC6    8D46 24                   lea eax,dword ptr ds:\[esi+24\]  
00F37FCB    E8 08000000               call 00F37FD8  
00F37FD0  6C 73 74 72 6C 65 6E 00  lstrlen.  
  
00F37FEA    FF55 44                   call dword ptr ss:\[ebp+44\]     ; kernel32.LoadLibraryA  
00F37FED    50                        push eax  
00F37FEE    FF55 3C                   call dword ptr ss:\[ebp+3C\]  
00F37FF1    FFD0                      call eax                ; kernel32.lstrlenA  
  
00F37FFA    E8 0B000000               call 00F3800A  
00F37FFF  43 68 61 72 55 70 70 65  CharUppe  
00F38007  72 41 00                 rA.  
  
00F3801A    FF55 44                   call dword ptr ss:\[ebp+44\]     ; kernel32.LoadLibraryA  
00F3801D    50                        push eax  
00F3801E    FF55 3C                   call dword ptr ss:\[ebp+3C\]  
00F38021    FFD0                      call eax                      ; User32.CharUpperA  
  
00F38030    E8 08000000               call 00F3803D  
00F38035  6C 73 74 72 63 6D 70 00 E8 E3 00                 lstrcmp.  
  
00F3835F    FF55 44                   call dword ptr ss:\[ebp+44\]      ; kernel32.LoadLibraryA  
00F38362    50                        push eax  
00F38363    FF55 3C                   call dword ptr ss:\[ebp+3C\]  
00F38566    FFD0                      call eax                        ; kernel32.lstrcmpA  
                                                                      ；这里做比较  
00F38568    66:9C                     pushfw                           ；  mov eax,0     
00F3859C    85C0                      test eax,eax   
00F385DD   /0F84 AB000000             je 00F3868E                 ; 必须跳,  否则 Over    
  
00F388FB    E8 08000000               call 00F38908  
00F38900  6C 73 74 72 6C 65 6E 00  lstrlen.  
00F3891A    FF55 44                   call dword ptr ss:\[ebp+44\]      ; kernel32.LoadLibraryA  
00F3891D    50                        push eax  
00F3891E    FF55 3C                   call dword ptr ss:\[ebp+3C\]  
00F38921    FFD0                      call eax                             ; kernel32.lstrlenA  
  
00F3894A    FF55 44                   call dword ptr ss:\[ebp+44\]  
00F3894D    50                        push eax                             ; User32.77D10000  
00F3894E    FF55 3C                   call dword ptr ss:\[ebp+3C\]  
00F38951    FFD0                      call eax                             ; User32.CharUpperA  
  
00F38AD8    FF55 44                   call dword ptr ss:\[ebp+44\]  
00F38B4D    FF55 3C                   call dword ptr ss:\[ebp+3C\]  
00F38BDE    8039 C2                   cmp byte ptr ds:\[ecx\],0C2  
00F38BEE    8039 E9                   cmp byte ptr ds:\[ecx\],0E9  
00F38C36    8039 CC                   cmp byte ptr ds:\[ecx\],0CC  
00F38C47    8039 E8                   cmp byte ptr ds:\[ecx\],0E8  
00F38C97    8039 58                   cmp byte ptr ds:\[ecx\],58  
00F38D03    8039 83                   cmp byte ptr ds:\[ecx\],83  
00F38D63    8039 90                   cmp byte ptr ds:\[ecx\],90  
00F38DA4    FFD0                      call eax            ; kernel32.lstrcmpA  
00F38DA6    66:9C                     pushfw              ；这里做比较，mov eax,0    
  
  
00F391B3    8B00                      mov eax,dword ptr ds:\[eax\]   ;  
00F391C1    8B09                      mov ecx,dword ptr ds:\[ecx\]   ;0f388ea  
00F391C3    3BC8                      cmp ecx,eax  
00F391C5    0F84 330E0000             je 00F39FFE  
  
00F3A404    E8 10000000               call 00F3A419  
00F3A409  47 65 74 43 6F 6D 6D 61 6E 64 4C 69 6E 65 41 00  GetCommandLineA.  
  
00F3A746    E8 0D000000               call 00F3A758  
00F3A74B  6B 65 72 6E 65 6C 33 32 2E 64 6C 6C 00           kernel32.dll.  
  
00F3A782    FF55 44                   call dword ptr ss:\[ebp+44\]     ; kernel32.LoadLibraryA  
00F3ABE1    FF55 3C                   call dword ptr ss:\[ebp+3C\]    ; kernel32.GetProcAddress  
00F3AF92    FFD0                      call eax            ; kernel32.GetCommandLineA  
  
00F3C9FC    FF55 44                   call dword ptr ss:\[ebp+44\]     ; kernel32.LoadLibraryA  
00F3CAFB    FF55 3C                   call dword ptr ss:\[ebp+3C\]     ; kernel32.GetProcAddress  
00F3D636    FFD0                      call eax                     ; kernel32.CreateFileA  
00F3D638    66:9C                     pushfw               ;eax=44  
  
0116FF7C   00141EE1  |FileName = "D:\\3\\regclean\\regclean.exe"  
0116FF80   80000000  |Access = GENERIC\_READ  
0116FF84   00000000  |ShareMode = 0  
0116FF88   00000000  |pSecurity = NULL  
0116FF8C   00000003  |Mode = OPEN\_EXISTING  
0116FF90   00000000  |Attributes = 0  
0116FF94   00000000  \\hTemplateFile = NULL  
  
00F3D6AD   /0F84 AB000000             je 00F3D75E         //不跳  
  
00F3DC58    E8 00000000               call 00F3DC5D  
00F3DC5D    58                        pop eax  
00F3DC5E    2D 705B0100               sub eax,15B70  
00F3DC63    B9 D1580100               mov ecx,158D1  
00F3DC68    8130 5546C7E8             xor dword ptr ds:\[eax\],E8C74655  
00F3DC6E    8300 44                   add dword ptr ds:\[eax\],44  
00F3DC71    83C0 01                   add eax,1  
00F3DC74  ^ E2 F2                     loopd short 00F3DC68  
00F3DC76    9C                        pushfd  
  
00F3E8B3    E8 10000000               call 00F3E8C8  
00F3E8B8  43 73 72 47 65 74 50 72 6F 63 65 73 73 49 64 00  CsrGetProcessId.  
00F3EC53    FF55 44                   call dword ptr ss:\[ebp+44\]    ; kernel32.LoadLibraryA  
00F3EDCD    FF55 3C                   call dword ptr ss:\[ebp+3C\]    ; kernel32.GetProcAddress  
00F3F800    FFD0                      call eax               ; ntdll.CsrGetProcessId  
00F3F802    66:9C                     pushfw  
00F3F877   /0F84 AB000000             je 00F3F928         //不跳  
  
00F419E5    E8 0E000000               call 00F419F8  
00F419EA  5A 77 4F 70 65 6E 50 72 6F 63 65 73 73 00        ZwOpenProcess.  
00F41A7F    FF55 44                   call dword ptr ss:\[ebp+44\]   ; kernel32.LoadLibraryA  
00F41B19    FF55 3C                   call dword ptr ss:\[ebp+3C\]   ; kernel32.GetProcAddress  
00F41E20    FFD0                      call eax                ; ntdll.ZwOpenProcess  
00F41E22    66:9C                     pushfw  
00F41E97   /0F84 AB000000             je 00F41F48               
  
00F4361C    E8 1A000000               call 00F4363B  
00F43621  4E 74 51 75 65 72 79 49 6E 66 6F 72 6D 61 74 69  NtQueryInformati  
00F43631  6F 6E 50 72 6F 63 65 73 73 00                    onProcess.  
  
00F437FA    FF55 44                   call dword ptr ss:\[ebp+44\]    ; kernel32.LoadLibraryA  
00F43939    FF55 3C                   call dword ptr ss:\[ebp+3C\]   ; kernel32.GetProcAddress  
00F439CA    6A 00                     push 0  
00F43B44    6A 00                     push 0  
00F44608    6A 04                     push 4  
  
00F44750    E8 62D40100               call 00F61BB7  
00F61BB7    8039 CC                   cmp byte ptr ds:\[ecx\],0CC  
00F61BBA    74 1F                     je short 00F61BDB  
00F61BBC    8079 01 CC                cmp byte ptr ds:\[ecx+1\],0CC  
00F61BC0    74 19                     je short 00F61BDB  
00F61BC2    8079 02 CC                cmp byte ptr ds:\[ecx+2\],0CC  
00F61BC6    74 13                     je short 00F61BDB  
00F61BC8    8079 03 CC                cmp byte ptr ds:\[ecx+3\],0CC  
00F61BCC    74 0D                     je short 00F61BDB  
00F61BCE    8079 04 CC                cmp byte ptr ds:\[ecx+4\],0CC  
00F61BD2    74 07                     je short 00F61BDB  
00F61BD4    8079 05 CC                cmp byte ptr ds:\[ecx+5\],0CC  
00F61BD8    74 01                     je short 00F61BDB  
00F61BDA    C3                        retn  
  
00F44984    58                        pop eax  
00F44985    FFD6                      call esi          ; ntdll.ZwQueryInformationProcess  
00F44987    9C                        pushfd  
  
0116FF80   FFFFFFFF  |hProcess = FFFFFFFF  
0116FF84   00000007  |InfoClass = 7  
0116FF88   0116FF94  |Buffer = 0116FF94  
0116FF8C   00000004  |Bufsize = 4  
0116FF90   00000000  \\pReqsize = NULL  
  
00F44CBB   /74 1B                     je short 00F44CD8    // ; 必须跳,  否则 Over    
  
00F45220    E8 12000000               call 00F45237  
00F45225  55 6E 68 6F 6F 6B 57 69 6E 64 6F 77 73 48 6F 6F  UnhookWindowsHoo  
00F45235  6B 00                                            k.  
00F4A9C6    FF55 44                   call dword ptr ss:\[ebp+44\]    ; kernel32.LoadLibraryA  
00F4AA08    FF55 3C                   call dword ptr ss:\[ebp+3C\]  ; kernel32.GetProcAddress  
00F4AA5F    FFD0                      call eax                ; kernel32.LocalAlloc  
  
00F4B802    E8 0C000000               call 00F4B813  
00F4B807  52 65 67 4F 70 65 6E 4B 65 79 41 00              RegOpenKeyA.  
00F5176E    FF55 44                   call dword ptr ss:\[ebp+44\]  ; kernel32.LoadLibraryA  
00F5179A    FF55 3C                   call dword ptr ss:\[ebp+3C\]   ; kernel32.GetProcAddress  
  
  
//ZwSetInformationThread 的检查  
  
00F51BD7    6A FE                     push -2           ；改这里为 push 0  
00F51EDB    FFD7                      call edi           ; ntdll.ZwSetInformationThread  
00F51EDD    60                        pushad  
00F51FC5    83C0 43                   add eax,43  
  
  
00F52A5F   /77 4E                     ja short 00F52AAF     // ; 必须跳,  否则 Over    
  
  
  
  
  
  
  
//下面才是程序中真真有用一点的代码，开始解压代码段  
  
  
00F5A649    8B06                      mov eax,dword ptr ds:\[esi\]   ;ESI=0043C03C  
00F5A64B    8907                      mov dword ptr ds:\[edi\],eax   ;EDI=00F64F22  
00F5A64D    83C6 04                   add esi,4  
00F5A650    83C7 04                   add edi,4  
00F5A653  ^ E2 F4                     loopd short 00F5A649  
  
00F5A655    8D85 7E1C0000             lea eax,dword ptr ss:\[ebp+1C7E\]  
00F5A65B    8982 48EB0700             mov dword ptr ds:\[edx+7EB48\],eax  
00F5A661    8B85 EB000000             mov eax,dword ptr ss:\[ebp+EB\]  
00F5A667    8982 2CEB0700             mov dword ptr ds:\[edx+7EB2C\],eax  
00F5A66D  ^\\79 EB                     jns short 00F5A65A        ;    mov !SF, 1  
  
//这里还要 mov ebp,edx，不知为什么 ebp 到这里的值不对，改成 edx 的值就可以了  
  
  
00F5A678    E8 3A750000               call 00F61BB7     ;检查 CC  
  
00F5A67E    FF95 24EB0700             call dword ptr ss:\[ebp+7EB24\] ;kernel32.GetModuleHandleA  
00F5A684    8985 30EB0700             mov dword ptr ss:\[ebp+7EB30\],eax  ;0F64F32=asdf.00400000  
00F5A68A    8DB5 4CEB0700             lea esi,dword ptr ss:\[ebp+7EB4C\]   ;ASCII "KERNEL32.dll"  
00F5A690    56                        push esi  
00F5A691    51                        push ecx  
00F5A692    8B8D 24EB0700             mov ecx,dword ptr ss:\[ebp+7EB24\]  
00F5A698    E8 1A750000               call 00F61BB7          ;检查 CC  
00F5A69D    59                        pop ecx  
00F5A69E    FF95 24EB0700             call dword ptr ss:\[ebp+7EB24\]  
00F5A6A4    0BC0                      or eax,eax  
00F5A6A6    75 07                     jnz short 00F5A6AF  
00F5A6A8    56                        push esi  
00F5A6A9    FF95 28EB0700             call dword ptr ss:\[ebp+7EB28\]  
00F5A6AF    50                        push eax  
00F5A6B0    8BF0                      mov esi,eax  
00F5A6B2    8D9D 3FED0700             lea ebx,dword ptr ss:\[ebp+7ED3F\];地址=00F65141, (ASCII "VirtualFree")  
00F5A6B8    53                        push ebx  
00F5A6B9    56                        push esi  
00F5A6BA    51                        push ecx  
00F5A6BB    8B8D 20EB0700             mov ecx,dword ptr ss:\[ebp+7EB20\];kernel32.GetProcAddress  
00F5A6C1    E8 F1740000               call 00F61BB7  
00F5A6C6    59                        pop ecx  
00F5A6C7    FF95 20EB0700             call dword ptr ss:\[ebp+7EB20\]  
00F5A6CD    8985 29EF0700             mov dword ptr ss:\[ebp+7EF29\],eax ;eax=77E59E34 (kernel32.VirtualFree)  
  
00F5A6D3    58                        pop eax  
00F5A6D4    8BF0                      mov esi,eax  
00F5A6D6    8D9D 2DEF0700             lea ebx,dword ptr ss:\[ebp+7EF2D\]  
地址=00F6532F, (ASCII "VirtualProtect")  
00F5A6DC    53                        push ebx  
00F5A6DD    56                        push esi  
00F5A6DE    FF95 20EB0700             call dword ptr ss:\[ebp+7EB20\]  
00F5A6E4    8985 1AF10700             mov dword ptr ss:\[ebp+7F11A\],eax  
00F5A6EA    BB 1EF10700               mov ebx,7F11E  
00F5A6EF    833C2B 00                 cmp dword ptr ds:\[ebx+ebp\],0  ;ds:\[00F65520\]=00029000  
00F5A6F3    74 61                     je short 00F5A756  
00F5A6F5    53                        push ebx  
00F5A6F6    6A 04                     push 4  
00F5A6F8    68 00100000               push 1000  
00F5A6FD    FF342B                    push dword ptr ds:\[ebx+ebp\]  
00F5A700    6A 00                     push 0  
00F5A702    51                        push ecx  
00F5A703    8B8D 2CEB0700             mov ecx,dword ptr ss:\[ebp+7EB2C\];ss:\[00F64F2E\]=77E5980A (kernel32.VirtualAlloc)  
  
00F5A709    E8 A9740000               call 00F61BB7  
00F5A70E    59                        pop ecx  
00F5A70F    FF95 2CEB0700             call dword ptr ss:\[ebp+7EB2C\]  ;VirtualAlloc  
00F5A715    5B                        pop ebx          
00F5A716    8BF0                      mov esi,eax        ; EAX=3A0000  
00F5A718    8BC3                      mov eax,ebx  
00F5A71A    03C5                      add eax,ebp  
00F5A71C    8B78 04                   mov edi,dword ptr ds:\[eax+4\]     ;1000  
00F5A71F    03BD 30EB0700             add edi,dword ptr ss:\[ebp+7EB30\]    
;ss:\[00F64F32\]=00400000 (asdf.00400000)  
edi=00001000  
00F5A725    56                        push esi  
00F5A726    57                        push edi  
00F5A727    FF95 48EB0700             call dword ptr ss:\[ebp+7EB48\]  ;解压代码  
00F5A72D    8B0C2B                    mov ecx,dword ptr ds:\[ebx+ebp\]  ；29000  
00F5A730    56                        push esi  
00F5A731    F3:A4                     rep movs byte ptr es:\[edi\],byte ptr >  
00F5A733    5E                        pop esi  
00F5A734    53                        push ebx  
00F5A735    68 00800000               push 8000  
00F5A73A    6A 00                     push 0  
00F5A73C    56                        push esi  
00F5A73D    51                        push ecx  
00F5A73E    8B8D 29EF0700             mov ecx,dword ptr ss:\[ebp+7EF29\] ；ss:\[00F6532B\]=77E59E34 (kernel32.VirtualFree)  
  
00F5A744    E8 6E740000               call 00F61BB7  
00F5A749    59                        pop ecx  
00F5A74A    FF95 29EF0700             call dword ptr ss:\[ebp+7EF29\]  ;VirtualFree  
00F5A750    5B                        pop ebx  
00F5A751    83C3 0C                   add ebx,0C       ；ds:\[00F6552C\]=00011880  
00F5A754  ^ EB 99                     jmp short 00F5A6EF  
  
  
//下面开始处理 IAT  
  
00F5A756    8B85 34EB0700             mov eax,dword ptr ss:\[ebp+7EB34\] ；ss:\[00F64F36\]=00000001  
  
00F5A75C    0BC0                      or eax,eax  
00F5A75E    0F85 A4000000             jnz 00F5A808      //跳  
00F5A764    8BBD 3CEB0700             mov edi,dword ptr ss:\[ebp+7EB3C\]  
00F5A76A    03BD 30EB0700             add edi,dword ptr ss:\[ebp+7EB30\]  
00F5A770    8B77 0C                   mov esi,dword ptr ds:\[edi+C\]  
00F5A773    0BF6                      or esi,esi  
00F5A775    75 05                     jnz short 00F5A77C  
00F5A777    E9 87000000               jmp 00F5A803  
00F5A77C    03B5 30EB0700             add esi,dword ptr ss:\[ebp+7EB30\]  
00F5A782    56                        push esi  
00F5A783    51                        push ecx  
00F5A784    8B8D 24EB0700             mov ecx,dword ptr ss:\[ebp+7EB24\]  
00F5A78A    E8 28740000               call 00F61BB7  
00F5A78F    59                        pop ecx  
00F5A790    FF95 24EB0700             call dword ptr ss:\[ebp+7EB24\]  
00F5A796    0BC0                      or eax,eax  
00F5A798    75 07                     jnz short 00F5A7A1  
00F5A79A    56                        push esi  
00F5A79B    FF95 28EB0700             call dword ptr ss:\[ebp+7EB28\]  
00F5A7A1    8BF0                      mov esi,eax  
00F5A7A3    8B17                      mov edx,dword ptr ds:\[edi\]  
00F5A7A5    0BD2                      or edx,edx  
00F5A7A7    75 03                     jnz short 00F5A7AC  
00F5A7A9    8B57 10                   mov edx,dword ptr ds:\[edi+10\]  
00F5A7AC    0395 30EB0700             add edx,dword ptr ss:\[ebp+7EB30\]  
00F5A7B2    8B5F 10                   mov ebx,dword ptr ds:\[edi+10\]  
00F5A7B5    039D 30EB0700             add ebx,dword ptr ss:\[ebp+7EB30\]  
00F5A7BB    8B02                      mov eax,dword ptr ds:\[edx\]  
00F5A7BD    0BC0                      or eax,eax  
00F5A7BF    75 02                     jnz short 00F5A7C3  
00F5A7C1    EB 38                     jmp short 00F5A7FB  
00F5A7C3    53                        push ebx  
00F5A7C4    52                        push edx  
00F5A7C5    99                        cdq  
00F5A7C6    0BD2                      or edx,edx  
00F5A7C8    75 0B                     jnz short 00F5A7D5  
00F5A7CA    83C0 02                   add eax,2  
00F5A7CD    0385 30EB0700             add eax,dword ptr ss:\[ebp+7EB30\]  
00F5A7D3    EB 05                     jmp short 00F5A7DA  
00F5A7D5    25 FFFFFF7F               and eax,7FFFFFFF  
00F5A7DA    50                        push eax  
00F5A7DB    56                        push esi  
00F5A7DC    51                        push ecx  
00F5A7DD    8B8D 20EB0700             mov ecx,dword ptr ss:\[ebp+7EB20\]  
00F5A7E3    E8 CF730000               call 00F61BB7  
00F5A7E8    59                        pop ecx  
00F5A7E9    FF95 20EB0700             call dword ptr ss:\[ebp+7EB20\]  
00F5A7EF    8903                      mov dword ptr ds:\[ebx\],eax  
00F5A7F1    5A                        pop edx  
00F5A7F2    5B                        pop ebx  
00F5A7F3    83C2 04                   add edx,4  
00F5A7F6    83C3 04                   add ebx,4  
00F5A7F9  ^ EB C0                     jmp short 00F5A7BB  
  
00F5A808    8B95 3CEB0700             mov edx,dword ptr ss:\[ebp+7EB3C\] ；ss:\[00F64F3E\]=0007F3A8  
00F5A80E    03D5                      add edx,ebp  
  
00F5A810    8B3A                      mov edi,dword ptr ds:\[edx\]  
ds:\[00F657AA\]=0001B0D0    ，edi=0043C000 (asdf.0043C000)  
00F5A812    0BFF                      or edi,edi  
00F5A814    75 05                     jnz short 00F5A81B  
00F5A816    E9 9C000000               jmp 00F5A8B7            //跳到加密 iat 处  
  
00F5A81B    03BD 30EB0700             add edi,dword ptr ss:\[ebp+7EB30\]；41b0d0  
00F5A821    83C2 05                   add edx,5  
00F5A824    8BF2                      mov esi,edx    ;edx=00F657AF, (ASCII "KERNEL32.dll")  
00F5A826    56                        push esi  
00F5A827    51                        push ecx  
00F5A828    8B8D 24EB0700             mov ecx,dword ptr ss:\[ebp+7EB24\] ;GetModuleHandleA   
00F5A82E    E8 84730000               call 00F61BB7  
00F5A833    59                        pop ecx  
00F5A834    FF95 24EB0700             call dword ptr ss:\[ebp+7EB24\] ;GetModuleHandleA   
00F5A83A    0BC0                      or eax,eax  
00F5A83C    75 14                     jnz short 00F5A852  
00F5A83E    56                        push esi  
00F5A83F    51                        push ecx  
00F5A840    8B8D 28EB0700             mov ecx,dword ptr ss:\[ebp+7EB28\]  ;LoadLibraryA  
00F5A846    E8 6C730000               call 00F61BB7  
00F5A84B    59                        pop ecx  
00F5A84C    FF95 28EB0700             call dword ptr ss:\[ebp+7EB28\]   ;ss:\[00F64F2A\]=77E605D8 (kernel32.LoadLibraryA)  
  
00F5A852    0FB64E FF                 movzx ecx,byte ptr ds:\[esi-1\]   ;长度 0c  
00F5A856    03F1                      add esi,ecx  
00F5A858    8BD6                      mov edx,esi   ；f657bb  
00F5A85A    8BF0                      mov esi,eax   ;kernel32.dll 基址 77e4000  
00F5A85C    42                        inc edx  
00F5A85D    8B0A                      mov ecx,dword ptr ds:\[edx\]  ;ds:\[00F657BC\]=0000006D  
函数的数目  
00F5A85F    83C2 04                   add edx,4        
  
  //  下面是处理未加密的 iat   
00F5A862    51                        push ecx            //循环处理 iat  
00F5A863    0FB602                    movzx eax,byte ptr ds:\[edx\]   ;函数名的长度  
00F5A866    0BC0                      or eax,eax  
00F5A868    75 21                     jnz short 00F5A88B  
00F5A86A    42                        inc edx           //函数序号  
00F5A86B    52                        push edx  
00F5A86C    8B02                      mov eax,dword ptr ds:\[edx\]    
00F5A86E    50                        push eax      ;函数名字符串  
00F5A86F    56                        push esi      ;库基址  
00F5A870    51                        push ecx  
00F5A871    8B8D 20EB0700             mov ecx,dword ptr ss:\[ebp+7EB20\] ；ss:\[00F64F22\]=77E5A5FD (kernel32.GetProcAddress)  
  
00F5A877    E8 3B730000               call 00F61BB7  
00F5A87C    59                        pop ecx  
00F5A87D    FF95 20EB0700             call dword ptr ss:\[ebp+7EB20\]  ;GetProcAddress  
00F5A883    8907                      mov dword ptr ds:\[edi\],eax     //函数地址放入  
00F5A885    5A                        pop edx  
00F5A886    83C2 04                   add edx,4  
00F5A889    EB 20                     jmp short 00F5A8AB  
00F5A88B    42                        inc edx  
00F5A88C    52                        push edx  
00F5A88D    52                        push edx  
00F5A88E    56                        push esi  
00F5A88F    51                        push ecx  
00F5A890    8B8D 20EB0700             mov ecx,dword ptr ss:\[ebp+7EB20\]  
00F5A896    E8 1C730000               call 00F61BB7  
00F5A89B    59                        pop ecx  
00F5A89C    FF95 20EB0700             call dword ptr ss:\[ebp+7EB20\]  //GetProcAddress  
00F5A8A2    8907                      mov dword ptr ds:\[edi\],eax   //函数地址放入  
00F5A8A4    5A                        pop edx  
00F5A8A5    0FB642 FF                 movzx eax,byte ptr ds:\[edx-1\]  
00F5A8A9    03D0                      add edx,eax  
00F5A8AB    42                        inc edx  
00F5A8AC    83C7 04                   add edi,4  
00F5A8AF    59                        pop ecx  
00F5A8B0  ^ E2 B0                     loopd short 00F5A862  //循环处理 iat  
  
//下面是处理加密的 iat  
  
00F5A8B7    8B85 38EB0700             mov eax,dword ptr ss:\[ebp+7EB38\]  
00F5A8BD    83F8 01                   cmp eax,1  
00F5A8C0    75 27                     jnz short 00F5A8E9  
00F5A8C2    8BBD 40EB0700             mov edi,dword ptr ss:\[ebp+7EB40\]  
00F5A8C8    03FD                      add edi,ebp             ；0f66919   
00F5A8CA    8DB5 46D40700             lea esi,dword ptr ss:\[ebp+7D446\]   ;壳处理 iat 的入口地址是 00d63848  
00F5A8D0    8B07                      mov eax,dword ptr ds:\[edi\]   ;\[edi\]=要加密的 iat 函数地址  
00F5A8D2    0BC0                      or eax,eax  
00F5A8D4    75 02                     jnz short 00F5A8D8  
00F5A8D6    EB 11                     jmp short 00F5A8E9  
00F5A8D8    25 FFFFFF7F               and eax,7FFFFFFF   ；407a40  
00F5A8DD    8BDE                      mov ebx,esi  
00F5A8DF    2BD8                      sub ebx,eax     ;d63848-407a40=b5be08  
00F5A8E1    8958 FC                   mov dword ptr ds:\[eax-4\],ebx  ;加密的 iat 函数放入  
00F5A8E4    83C7 08                   add edi,8  
00F5A8E7  ^ EB E7                     jmp short 00F5A8D0  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
//修改 IAT 加密的代码为正常代码  
  
00F5A8CA    8B07                       mov eax,dword ptr ds:\[edi\]    ///  
00F5A8CC    8BDF                       mov ebx,edi                   ///  
00F5A8CE    83C3 04                    add ebx,4                     ///  
00F5A8D1    90                         nop                           ///  
00F5A8D2    0BC0                       or eax,eax  
00F5A8D4    75 02                      jnz short 00F5A8D8  
00F5A8D6    EB 11                      jmp short 00F5A8E9  
00F5A8D8    25 FFFFFF7F                and eax,7FFFFFFF  
00F5A8DD    8B1B                       mov ebx,dword ptr ds:\[ebx\]    ///      
00F5A8DF    90                         nop                           ///  
00F5A8E0    90                         nop                          ///  
00F5A8E1    8958 FC                    mov dword ptr ds:\[eax-4\],ebx  
00F5A8E4    66:C740 FA FF15            mov word ptr ds:\[eax-6\],15FF   ///     
00F5A8EA    83C7 08                    add edi,8                     ///  
00F5A8ED  ^ EB DB                      jmp short 00F5A8CA           ///     
  
  
  
8B 07 8B DF 83 C3 04 90 0B C0 75 02 EB 11 25 FF FF FF 7F 8B 1B 90 90 89 58 FC 66 C7 40 FA FF 15 83 C7 08 EB DB  
  
  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
  
  
  
00F5A8E9    68 03EA17B3               push B317EA03  
00F5AA38    51                        push ecx  
  
00F5AAC8    64:FF35 30000000          push dword ptr fs:\[30\]  
00F5AC5A    58                        pop eax                              ; 7FFDF000  
00F5AD39    0FB648 02                 movzx ecx,byte ptr ds:\[eax+2\]  
00F5AD5B    8B40 0C                   mov eax,dword ptr ds:\[eax+C\]  
00F5AD5E    8B40 0C                   mov eax,dword ptr ds:\[eax+C\]  
00F5ADA5    0AC9                      or cl,cl  
00F5AE1F   /0F85 29030000             jnz 00F5B14E          //不跳  
00F5B1AB    89AD 66E00700             mov dword ptr ss:\[ebp+7E066\],ebp  
00F5B252    8928                      mov dword ptr ds:\[eax\],ebp  ；\[eax\]=00F61DBB  
00F5B295    8BBD 30EB0700             mov edi,dword ptr ss:\[ebp+7EB30\]   ;400000  
00F5B2CF    037F 3C                   add edi,dword ptr ds:\[edi+3C\]  ;ds:\[0040003C\]=000000F0  
00F5B30D    8BB5 30EB0700             mov esi,dword ptr ss:\[ebp+7EB30\]     ; asdf.00400000  
00F5B367    8B4F 54                   mov ecx,dword ptr ds:\[edi+54\] ;ds:\[00400144\]=00001000  
00F5B4EA    6A 04                     push 4  
00F5B521    51                        push ecx    ;1000  
00F5B556    FFB5 30EB0700             push dword ptr ss:\[ebp+7EB30\]        ; asdf.00400000  
00F5B59D    FF95 1AF10700             call dword ptr ss:\[ebp+7F11A\] ; kernel32.VirtualProtect  
  
0116FFA4   00400000  |Address = asdf.00400000  
0116FFA8   00001000  |Size = 1000 (4096.)  
0116FFAC   00000004  |NewProtect = PAGE\_READWRITE  
0116FFB0   00F5B4C5  \\pOldProtect = 00F5B4C5  
  
00F5B5D8    57                        push edi         ;PE  
00F5B60E    83C7 74                   add edi,74       ;数据目录结构的数量  
00F5B64D    8B0F                      mov ecx,dword ptr ds:\[edi\]   ；6FDC0C9E  
00F5B761    8B00                      mov eax,dword ptr ds:\[eax\]   ；ds:\[00F63767\]=6FDC0C9E  
00F5B7D9    3BC1                      cmp eax,ecx  
00F5BD8E    8B85 30EB0700             mov eax,dword ptr ss:\[ebp+7EB30\]     ; asdf.00400000  
00F5BDCB    0340 3C                   add eax,dword ptr ds:\[eax+3C\]  
00F5BE09    66:0960 06                or word ptr ds:\[eax+6\],sp  ；sp=FFB4  
ds:\[004000F6\]=0003  
00F5BEC0    C740 36 00000000          mov dword ptr ds:\[eax+36\],0  ；清除基址和块对齐粒度  
00F5BEFC    C740 37 10000000          mov dword ptr ds:\[eax+37\],10 ；改基址  
00F5C1DA    05 F0000000               add eax,0F0  
00F5C267    B9 00100000               mov ecx,1000  
00F5C309    2BC8                      sub ecx,eax   ；1000-1e0=e20  
00F5C340    83E9 20                   sub ecx,20    ;e00  
00F5C3B5    C700 00000000             mov dword ptr ds:\[eax\],0     ; asdf.004001E0  
00F5C3F7    40                        inc eax                               
00F5C42D  ^\\E2 86                     loopd short 00F5C3B5  
  
//对 4001e0 长度 e00 的代码清零  
  
00F5C693    B9 3F000000               mov ecx,3F  
00F5C902    891C01                    mov dword ptr ds:\[ecx+eax\],ebx  ；文件中块对齐粒度  
00F5C949    41                        inc ecx  
00F5C97F    83F9 53                   cmp ecx,53  
00F5C9B7  ^\\0F85 5CFDFFFF             jnz 00F5C719  
  
//对 40012 长度 14 的代码清零  
  
  
00F5D0B1    8BBD 30EB0700             mov edi,dword ptr ss:\[ebp+7EB30\]  ;400000  
00F5D116    037F 3C                   add edi,dword ptr ds:\[edi+3C\]  
00F5D15A    8BB5 30EB0700             mov esi,dword ptr ss:\[ebp+7EB30\]     ; asdf.00400000  
00F5D194    8B4F 54                   mov ecx,dword ptr ds:\[edi+54\]  
00F5D385    6A 02                     push 2  
00F5D3BC    51                        push ecx  
00F5D3F4    FFB5 30EB0700             push dword ptr ss:\[ebp+7EB30\]        ; asdf.00400000  
00F5D42F    FF95 1AF10700             call dword ptr ss:\[ebp+7F11A\] ; kernel32.VirtualProtect  
00F5DC43    B9 2C330000               mov ecx,332C  
00F5DED9    3103                      xor dword ptr ds:\[ebx\],eax  
eax=B9A15A72  
ds:\[00F5A3A3\]=796807EB  
00F5DF10    83C3 01                   add ebx,1  
00F5DF4F  ^\\E2 88                     loopd short 00F5DED9  
00F5DF51   /E9 D8000000               jmp 00F5E02E         ；ebx=0f5d6cf  
  
  
//对 0F5a3a3 长度 332c 的代码加密  
  
00F5F0B4    58                        pop eax  
00F608F5    E8 0B000000               call 00F60905  
00F6129D    E8 0B000000               call 00F612AD  
00F615BD    E8 0B000000               call 00F615CD  
  
00F618CE    8B8D 44EB0700             mov ecx,dword ptr ss:\[ebp+7EB44\]  ;ss:\[00F64F46\]=000068D0   ,ecx=00EE6402  
00F61909    038D 30EB0700             add ecx,dword ptr ss:\[ebp+7EB30\]  ;400000  
00F61935    8BC1                      mov eax,ecx  
00F6196D    50                        push eax  
00F6198D    C3                        retn       ;入口点  
  
//程序的入口点，在这里 dump 出程序，ok 了  
  
004068D0    55                        push ebp  
004068D1    8BEC                      mov ebp,esp  
004068D3    6A FF                     push -1  
004068D5    68 70D24100               push RegClean.0041D270  
004068DA    68 B4944000               push RegClean.004094B4  
004068DF    64:A1 00000000            mov eax,dword ptr fs:\[0\]  
004068E5    50                        push eax  
004068E6    64:8925 00000000          mov dword ptr fs:\[0\],esp  
004068ED    83EC 58                   sub esp,58  
004068F0    53                        push ebx  
004068F1    56                        push esi  
004068F2    57                        push edi  
004068F3    8965 E8                   mov dword ptr ss:\[ebp-18\],esp  
004068F6    3E:E8 4CCFB500            call 00F63848    //CALL 进壳代码中  
  
  
//壳对 IAT 的处理比较简单，函数地址放在 CALL 地址+4 的地方，修改方法参见上面。  
  
00F63848    50                        push eax  
00F63867    8908                      mov dword ptr ds:\[eax\],ecx  
00F63DA8    68 03EA17B3               push B317EA03  
00F63DAD    53                        push ebx  
00F63DAE    E8 5D000000               call 00F63E10  
  
00F6478A    8B06                      mov eax,dword ptr ds:\[esi\]           ; RegClean.0040AC3F  
00F64798    B9 02000000               mov ecx,2  
00F647A7    F7E1                      mul ecx  
00F647B2    D1E8                      shr eax,1  
00F647C0    3BF8                      cmp edi,eax             ; eax=RegClean.00407A40  
00F647C2   /0F85 10010000             jnz 00F648D8  
00F64803   /0F82 F1000000             jb 00F648FA     ;找到对应的函数地址跳到下面  
00F648D8    83C6 08                   add esi,8  
00F648DE  ^\\0F8E A6FEFFFF             jle 00F6478A   ；没找到，加 8 后跳到上面  
  
  
00F648FA    8B46 04                   mov eax,dword ptr ds:\[esi+4\]         ; RegClean.0041B1F4  
00F64906    8903                      mov dword ptr ds:\[ebx\],eax           ;eax=0041B1F4   
00F6490F    64:FF35 30000000          push dword ptr fs:\[30\]  
00F6491B    8B49 0C                   mov ecx,dword ptr ds:\[ecx+C\]  
00F64921    8B49 0C                   mov ecx,dword ptr ds:\[ecx+C\]  
00F6492B    8B49 20                   mov ecx,dword ptr ds:\[ecx+20\]  
00F649C4    83C0 03                   add eax,3  
00F64A06    83E9 03                   sub ecx,3  
00F64A45    85C9                      test ecx,ecx  
00F64A7E  ^\\0F85 71F2FFFF             jnz 00F63CF5  
00F64A8F    8B00                      mov eax,dword ptr ds:\[eax\] ; kernel32.GetVersion  
00F64AD2    50                        push eax  
00F64AF8    C3                        retn   //返回系统函数  
  
  
最后，NumberOfRvaAndSizes =6fdc0c9e,改为 00000010，用 OD 加载时就中断在入口点 004068D0 处了。  
脚本写的太乱，大部分地址都是本地地址，所以就不发了，如果作者真的写成了通用版本的话，再写通用脚本吧。  
  
                                                  2005 年 12 月 9 日

老大的耐心真是另人钦佩，不过你能分析花指令是因为这个好象是我原来写错的哪个,当时有一个参数传递错误，造成只调用了一个宏，而实际上应该是１６＊１６＊４组，在这个数量级上分析花指令是会伤害眼睛的．其实这个壳是个盗版，而且有Ｎ多ＢＵＧ，只不过盗的东西多了就看不出来了，IAT 没有处理是因为觉得没什么用，反正是一步一步走过来的，前面的能ＫＯ，后面就没问题．我放寒假的时候会再做更新的，能有人一起玩真是开心啊．![](img/710b5589904d0d2bf58a67748fca40e1.jpg) ![](img/710b5589904d0d2bf58a67748fca40e1.jpg) ![](img/710b5589904d0d2bf58a67748fca40e1.jpg)

从 94000-00F5A649 之间只有 anti 的话，第二次跟踪的话，我会从 94000 直接跳到 00F5A649 处，修改一些关键的寄存器的值，程序也能执行啊！希望作者在下个版本中间加点东西，否则光看 anti 了。  
![](img/a7e67d4f8f4afe4c3cd0617906cb3d9d.jpg) ![](img/a7e67d4f8f4afe4c3cd0617906cb3d9d.jpg) ![](img/a7e67d4f8f4afe4c3cd0617906cb3d9d.jpg)

TO wangli\_com:我说了那个写错了,你看到是单一兵种,不是虫海,直接跳了当然会挂,直接导致后面解码错误,而且离的很远,到时候你就想不起来是那里的问题了.壳是做 ANTI 实验玩的,在不同平台上会有不同的状态,考验的不是技术,而是心理承受能力,其实我是学电子信息工程的,课余研究过行为心理学,我很清楚在心理上压跨人比在技术上超越人要现实的多.我现在正准备期末考试,等放假了一定会更新的.  
TO crackerabc:感谢老大看的起,现在的东西还是游戏阶段,不能用.等到时候改进成能拿的出手的东西就给老大传一份. 

 4dd466d0e1e0cc123afbe504da2462ca