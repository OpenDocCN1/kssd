

# 大嘴日语 5.0 之 ExeStealth 脱壳

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-641.htm`](https://www.pediy.com/kssd/pediy07/pediy7-641.htm)

【破解作者】 Pr0Zel  
【使用工具】 OllyDbg,PEid,LoadPE  
【破解平台】 Win2K+SP4  
【软件名称】 大嘴日语 5.0  
【下载地址】 http://www.onlinedown.net/soft/4089.htm  
【软件简介】 大嘴日语 融合了【逆向学习法】和【疯狂英语】这两种有效的英语学习方法的精髓，  
              并加以创新，集日语的【听、说、读、写、背、查】功能于一身， 解决了困扰国人的学习日语难的问题。  
              是一款集日语口语、日语背单词综合教育软件。  
【软件大小】 4.3M  
【加壳方式】 EXEStealth 2.75a + ASPack + UPX  
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）  
  
开始调试前,最好把 OLLYDBG.EXE 改名为 CMD.EXE 因为壳里面有些代码是检测父进程的,如下:  
0123CCA0    813F 434D442E   cmp dword ptr ds:\[edi\],2E444D43          ; \[EDI\]是检测父程序名字, 0x43,0x4D,0x44,0x2E 的 ASCII 字符是"CMD."  
0123CCA6    74 3B           je short BmJapane.0123CCE3               ; 父程序不是 CMD.EXE,就不跳,接着就 OVER 了 修改 ZF 寄存器为 1  
0123CCA8    90              nop  
0123CCA9    90              nop  
0123CCAA    90              nop  
0123CCAB    90              nop  
0123CCAC    83EE 0C         sub esi,0C  
0123CCAF    AD              lods dword ptr ds:\[esi\]  
0123CCB0    0306            add eax,dword ptr ds:\[esi\]  
0123CCB2    0346 04         add eax,dword ptr ds:\[esi+4\]  
0123CCB5    8BD8            mov ebx,eax  
0123CCB7    8DB5 A1194100   lea esi,dword ptr ss:\[ebp+4119A1\]  
0123CCBD    AD              lods dword ptr ds:\[esi\]  
0123CCBE    0BC0            or eax,eax  
0123CCC0    74 0E           je short BmJapane.0123CCD0  
0123CCC2    90              nop  
0123CCC3    90              nop  
0123CCC4    90              nop  
0123CCC5    90              nop  
0123CCC6    2BC3            sub eax,ebx  
0123CCC8    74 19           je short BmJapane.0123CCE3  
0123CCCA    90              nop  
0123CCCB    90              nop  
0123CCCC    90              nop  
0123CCCD    90              nop  
0123CCCE  ^ EB ED           jmp short BmJapane.0123CCBD  
0123CCD0    B8 90010000     mov eax,190  
0123CCD5    E8 C6EFFFFF     call BmJapane.0123BCA0  
0123CCDA    8DBD 69324000   lea edi,dword ptr ss:\[ebp+403269\]  
0123CCE0    03F8            add edi,eax  
0123CCE2    AB              stos dword ptr es:\[edi\]                  ; 若是前面不跳的话,到这里调试器就会无法处理异常,进程退出  
0123CCE3    FFB5 30FE4000   push dword ptr ss:\[ebp+40FE30\]           ; 跳到这里  
0123CCE9    50              push eax  
  
另把同目录下的 Config.ini 文件备份一个,当出来说文件不完整要重新安装程序的时候  
把备份文件覆盖了 Config 文件就行了;  
\---------------------------------------------------------------------------------------------------------  
  
用 OD 载入程序,停在这里:(第一层壳 EXEStealth)  
0124A060 > /EB 58           jmp short BmJapane.0124A0BA  
0124A062   |53              push ebx  
0124A063   |68 61726577     push 77657261  
0124A068   |61              popad  
0124A069   |72 65           jb short BmJapane.0124A0D0  
  
忽略除内存异常外的所有异常,经过 5 个内存异常(按 4 次 Shift+F9)后来到这里:  
0078700D    64:8925 0000000>mov dword ptr fs:\[0\],esp  
00787014    33C0            xor eax,eax  
00787016    8908            mov dword ptr ds:\[eax\],ecx               ; 内存异常  
00787018    0000            add byte ptr ds:\[eax\],al  
0078701A    0000            add byte ptr ds:\[eax\],al  
此时的堆栈是:  
0012FFBC   FFFFFFFF  End of SEH chain  
0012FFC0   00D9DAA8  SE handler  
0012FFC4   7C4E87F5  RETURN to kernel32.7C4E87F5  
0012FFC8   005616A8  BmJapane.005616A8  
  
在 D9DAA8 上下断点,按 Shift+F9  
00D9DAA8    B8 A7C8D9F0     mov eax,F0D9C8A7  
00D9DAAD    8D88 24120010   lea ecx,dword ptr ds:\[eax+10001224\]  
00D9DAB3    8941 01         mov dword ptr ds:\[ecx+1\],eax  
00D9DAB6    8B5424 04       mov edx,dword ptr ss:\[esp+4\]  
00D9DABA    8B52 0C         mov edx,dword ptr ds:\[edx+C\]  
00D9DABD    C602 E9         mov byte ptr ds:\[edx\],0E9  
00D9DAC0    83C2 05         add edx,5  
00D9DAC3    2BCA            sub ecx,edx  
00D9DAC5    894A FC         mov dword ptr ds:\[edx\-4\],ecx  
00D9DAC8    33C0            xor eax,eax  
00D9DACA    C3              retn  
  
  
走到 RETN 后,看原地址的代码:  
0078700D    64:8925 0000000>mov dword ptr fs:\[0\],esp  
00787014    33C0            xor eax,eax  
00787016  - E9 B06A6100     jmp BmJapane.00D9DACB                    ; 在这里下断点,程序断在这里,然后把标志寄存器的 TP 置 0,继续运行  
0078701B    0000            add byte ptr ds:\[eax\],al  
0078701D    0000            add byte ptr ds:\[eax\],al  
  
00D9DACA    C3              retn  
00D9DACB    B8 A7C8D9F0     mov eax,F0D9C8A7                         ; 原来跳到这里了  
00D9DAD0    64:8F05 0000000>pop dword ptr fs:\[0\]  
00D9DAD7    83C4 04         add esp,4  
00D9DADA    55              push ebp  
00D9DADB    53              push ebx  
00D9DADC    51              push ecx  
00D9DADD    57              push edi  
00D9DADE    56              push esi  
00D9DADF    52              push edx  
00D9DAE0    8D98 DD110010   lea ebx,dword ptr ds:\[eax+100011DD\]  
  
  
  
一直走到这里:  
00D9DB70    5E              pop esi  
00D9DB71    5F              pop edi  
00D9DB72    59              pop ecx  
00D9DB73    5B              pop ebx  
00D9DB74    5D              pop ebp  
00D9DB75  - FFE0            jmp eax                                  ; EAX=B55001  
  
\---------------------------------------------------------------------------------------------------------  
又跳到下一层去了,貌似是 ASPack 壳  
00B55001    60              pushad  
00B55002    E8 03000000     call BmJapane.00B5500A  
00B55007  - E9 EB045D45     jmp 461254F7  
00B5500C    55              push ebp  
00B5500D    C3              retn  
00B5500E    E8 01000000     call BmJapane.00B55014  
00B55013    EB 5D           jmp short BmJapane.00B55072  
  
直接用 Ctrl+S 来找 popad  
来到这里:  
00B553A9    8985 A8030000   mov dword ptr ss:\[ebp+3A8\],eax  
00B553AF    61              popad  
00B553B0    75 08           jnz short BmJapane.00B553BA  
00B553B2    B8 01000000     mov eax,1  
00B553B7    C2 0C00         retn 0C  
00B553BA    68 00000000     push 0                                   ; 运行到这里会变成 push 90fc10  
00B553BF    C3              retn  
  
\---------------------------------------------------------------------------------------------------------  
到达下一层壳,是 UPX 壳  
0090FC10    60              pushad  
0090FC11    BE 00707800     mov esi,BmJapane.00787000  
0090FC16    8DBE 00A0C7FF   lea edi,dword ptr ds:\[esi+FFC7A000\]  
0090FC1C    57              push edi  
0090FC1D    83CD FF         or ebp,FFFFFFFF  
0090FC20    EB 10           jmp short BmJapane.0090FC32  
0090FC22    90              nop  
0090FC23    90              nop  
0090FC24    90              nop  
0090FC25    90              nop  
0090FC26    90              nop  
0090FC27    90              nop  
0090FC28    8A06            mov al,byte ptr ds:\[esi\]  
0090FC2A    46              inc esi  
0090FC2B    8807            mov byte ptr ds:\[edi\],al  
0090FC2D    47              inc edi  
0090FC2E    01DB            add ebx,ebx  
0090FC30    75 07           jnz short BmJapane.0090FC39  
  
同样往下找 popad,找到如下:  
0090FD9B    66:8B07         mov ax,word ptr ds:\[edi\]  
0090FD9E    83C7 02         add edi,2  
0090FDA1  ^ EB E2           jmp short BmJapane.0090FD85  
0090FDA3    61              popad  
0090FDA4  ^ E9 8B1DD7FF     jmp BmJapane.00681B34  
  
\---------------------------------------------------------------------------------------------------------  
到达程序的 OPE 了,用 OllDump 把程序 DUMP 出来,用 LordPE 来 DUMP 也行,不过要用 IR 修复一下 IAT  
00681B34    55              push ebp  
00681B35    8BEC            mov ebp,esp  
00681B37    83C4 F0         add esp,-10  
00681B3A    53              push ebx  
00681B3B    B8 9C146800     mov eax,BmJapane.0068149C  
00681B40    E8 175DD8FF     call BmJapane.0040785C  
00681B45    8B1D D0906800   mov ebx,dword ptr ds:\[6890D0\]            ; BmJapane.0068AC18  
00681B4B    8B03            mov eax,dword ptr ds:\[ebx\]  
00681B4D    E8 CE22E1FF     call BmJapane.00493E20  
00681B52    8B03            mov eax,dword ptr ds:\[ebx\]  
00681B54    BA 7C1E6800     mov edx,BmJapane.00681E7C  
00681B59    E8 AA1EE1FF     call BmJapane.00493A08  
00681B5E    8B0B            mov ecx,dword ptr ds:\[ebx\]  
  
\---------------------------------------------------------------------------------------------------------  
  
运行一下,程序报错  
"Access violation at address 1000100A. Read of address 1000100A."  
用 LordPE 看看内存空间:  
  
ListView 的 Item 总数: 26  
\-------------------------  
Path                                             ImageBase      ImageSize        
\-------------------------------------------------------------------------------  
d:\\bmjapanese\\bmjapanese.exe                     00400000       00001000         
c:\\winnt\\system32\\ntdll.dll                      77F80000       0007B000         
c:\\winnt\\system32\\kernel32.dll                   7C4E0000       000B9000         
c:\\winnt\\system32\\user32.dll                     77E10000       00065000         
c:\\winnt\\system32\\gdi32.dll                      77F40000       0003C000         
c:\\winnt\\system32\\imm32.dll                      75E60000       0001A000         
c:\\winnt\\system32\\advapi32.dll                   7C2D0000       00062000         
c:\\winnt\\system32\\rpcrt4.dll                     77D30000       00071000         
d:\\bmjapanese\\data\\data.dat                      10000000       00037000         
c:\\winnt\\system32\\comctl32.dll                   71710000       00084000         
c:\\winnt\\system32\\comdlg32.dll                   76B30000       0003E000         
c:\\winnt\\system32\\shlwapi.dll                    70BD0000       00065000         
c:\\winnt\\system32\\msvcrt.dll                     78000000       00045000         
c:\\winnt\\system32\\shell32.dll                    782F0000       00248000         
c:\\winnt\\system32\\hhctrl.ocx                     5D300000       00080000         
c:\\winnt\\system32\\ole32.dll                      77A50000       000F7000         
c:\\winnt\\system32\\oleaut32.dll                   779B0000       0009B000         
c:\\winnt\\system32\\msacm32.dll                    77410000       00013000         
c:\\winnt\\system32\\winmm.dll                      77570000       00030000         
c:\\winnt\\system32\\version.dll                    77820000       00007000         
c:\\winnt\\system32\\lz32.dll                       759B0000       00006000         
c:\\winnt\\system32\\winspool.drv                   77800000       0001E000         
c:\\winnt\\system32\\mpr.dll                        76620000       00011000         
c:\\winnt\\system32\\wsock32.dll                    75050000       00008000         
c:\\winnt\\system32\\ws2\_32.dll                     75030000       00014000         
c:\\winnt\\system32\\ws2help.dll                    75020000       00008000         
  
地址是 data.dat 的空间,用 PEID 一查,是个 ASPack 壳的 DLL 文件,查输出表,只有两个函数,  
分别是 HDSerialNumRead 和 WEP  
  
还差一个 DLL 没有 LOAD 进去,当然会出错了.  
重新载入原文件,下 LoadLibraryA 断点,经过几次后,断在这里:  
01240873    53              push ebx  
01240874    FF95 94E24100   call dword ptr ss:\[ebp+41E294\]           ; LoadLibraryA  .dat\\data\\data.dat  
0124087A    0BC0            or eax,eax  
0124087C    75 34           jnz short BmJapane.012408B2  
  
然后来到这里:  
01240941    53              push ebx  
01240942    FFB5 17FC4000   push dword ptr ss:\[ebp+40FC17\]  
01240948    FF95 8CE24100   call dword ptr ss:\[ebp+41E28C\]           ; kernel32.GetProcAddress  
0124094E    3B9D 1FFC4000   cmp ebx,dword ptr ss:\[ebp+40FC1F\]  
  
得到 HDSerialNumRead 的地址  
用 Alt+M,打开内存镜像,  
在 10001000 处按 F2 下断点,取消刚才的 LoadLibraryA 断点,按 F9,来到这里  
10001003    CC              int3  
10001004    CC              int3  
10001005    E9 66010000     jmp data.10001170  
1000100A >  E9 310B0000     jmp data.10001B40      -|jmp  
1000100F >  E9 3C0C0000     jmp data.10001C50  
10001014    E9 C7030000     jmp data.100013E0  
10001019    E9 52000000     jmp data.10001070  
1000101E    E9 0D020000     jmp data.10001230  
10001023    E9 78080000     jmp data.100018A0  
10001028    E9 93020000     jmp data.100012C0  
1000102D    E9 BE040000     jmp data.100014F0  
10001032    E9 89070000     jmp data.100017C0  
10001037    CC              int3  
10001038    CC              int3  
\----------------------------------------------------|  
  
10001B40    55              push ebp  
10001B41    8BEC            mov ebp,esp  
10001B43    81EC D4070000   sub esp,7D4  
10001B49    53              push ebx  
10001B4A    56              push esi  
  
按 CTRL+F9,然后返回主程序:  
0059C567    8D45 C4         lea eax,dword ptr ss:\[ebp\-3C\]  
0059C56A    E8 518AE6FF     call BmJapane.00404FC0  
0059C56F    E8 D0F1FFFF     call BmJapane.0059B744                   ; jmp to data.HDSerialNumRead  
0059C574    8BD0            mov edx,eax  
0059C576    8D45 C4         lea eax,dword ptr ss:\[ebp\-3C\]  
  
原来 CALL 59B744 是 data.HDSerialNumRead 的函数的地址 CALL,直接到上面看看:  
0059B744  - FF25 2CCC6800   jmp dword ptr ds:\[68CC2C\]                ; data.HDSerialNumRead  
嗯,只要把 68CC20 填充为正确的地址就可以了  
  
用 OD 载入 DUMP 出来的程序,下断点 LoadLibraryA,程序中断后按 Alt+F9 返回用户代码:  
0047E97B   /0F85 EE000000   jnz oll.0047EA6F  
0047E981   |68 ACEA4700     push oll.0047EAAC                        ; ASCII "imm32.dll"  
0047E986   |E8 D594F8FF     call <jmp.&kernel32.LoadLibraryA>  
0047E98B   |A3 88326800     mov dword ptr ds:\[683288\],eax            ; IMM32.75E60000  
0047E990   |833D 88326800 0>cmp dword ptr ds:\[683288\],0  
0047E997   |0F84 D2000000   je oll.0047EA6F  
  
往下找代码,找到出口处:  
0047EA8D    5B              pop ebx                                  ; USER32.77E10000  
0047EA8E    59              pop ecx  
0047EA8F    5D              pop ebp  
0047EA90    C3              retn  
0047EA91    0000            add byte ptr ds:\[eax\],al  
  
找一个地方加上自己的代码:  
区块 ExeS 已经不用了,正好用来写代码:  
注:LoadLibraryA 的 CALL 是"call 00407E60" ;GetProcAddress 的 CALL 是"call 00407D50"  
情况不同者要相应作出修改.  
  
0047EA8D  - E9 6EB5DC00     jmp oll.0124A000  
0047EA92    C3              retn  
  
0124A000    68 30A02401     push oll.0124A030                        ; ASCII ".\\data\\Data.dat"  
0124A005    E8 56DE1BFF     call <jmp.&kernel32.LoadLibraryA>  
0124A00A    68 40A02401     push oll.0124A040                        ; ASCII "HDSerialNumRead"  
0124A00F    50              push eax                                 ; EAX 是 LoadLibraryA 后的 DLL 句柄  
0124A010    E8 3BDD1BFF     call <jmp.&kernel32.GetProcAddress>  
0124A015    A3 2CCC6800     mov dword ptr ds:\[68CC2C\],eax  
0124A01A    5B              pop ebx  
0124A01B    59              pop ecx  
0124A01C    5D              pop ebp  
0124A01D    90              nop  
0124A01E  - E9 6F4A23FF     jmp oll.0047EA92  
0124A023    90              nop  
  
\---------------------------------------------------------------------------------------------------------  
去程序的 ANTI 代码  
0067D1E9  ^\\EB CE           jmp short dumped.0067D1B9  
0067D1EB    E8 80C4E4FF     call dumped.004C9670  
0067D1F0    68 FFFF0000     push 0FFFF  
0067D1F5    6A 0D           push 0D  
0067D1F7    E8 7CB2D8FF     call <jmp.&user32.ExitWindowsEx>         ; 关机了~  
0067D1FC    E9 CD060000     jmp dumped.0067D8CE  
  
这样危险的 API 当然不能让程序乱用的,要它指向自己的代码,直接在 IAT 里面改:  
0067D1F7    E8 7CB2D8FF     call <jmp.&user32.ExitWindowsEx> 本来是 call 00408478  
  
去 408478 上看看:  
00408478  - FF25 50C96800   jmp dword ptr ds:\[<&user32.ExitWindowsEx>; user32.ExitWindowsEx  
  
自己写的代码:  
0124A000    83C4 0C         add esp,0C  
0124A003    FF7424 F4       push dword ptr ss:\[esp\-C\]                ; 还原调用 CALL 以后 RETN 的地址  
0124A007    C3              retn  
  
更改 IAT 表:  
00408478  - E9 831BE400     jmp exit\_fix.0124A000  
0040847D    90              nop  
  
  
运行,程序 OK,....  
脱壳完成  
  
后记,EXEStealth 的 IAT 还原部分,比较有趣:  
00FDE535    8B07            mov eax,dword ptr ds:\[edi\]               ; EDI 是加密函数名结构(可以这样说)的基地址  
00FDE537    09C0            or eax,eax  
00FDE539    74 3C           je short BmJapane.00FDE577               ; IAT 表是否填充完成,完成的话跳  
00FDE53B    8B5F 04         mov ebx,dword ptr ds:\[edi+4\]  
00FDE53E    8D8430 BC43BE00 lea eax,dword ptr ds:\[eax+esi+BE43BC\]  
00FDE545    01F3            add ebx,esi  
00FDE547    50              push eax  
00FDE548    83C7 08         add edi,8                                ; 函数名结构基地址+8byte,指向第 3 项  
00FDE54B    FF96 1045BE00   call dword ptr ds:\[esi+BE4510\]           ; LoadLibraryA  
00FDE551    95              xchg eax,ebp  
00FDE552    8A07            mov al,byte ptr ds:\[edi\]                 ; 读取第 3 项数值  
00FDE554    47              inc edi                                  ; 函数名结构基地址+1,指向第 4 项,即函数名  
00FDE555    08C0            or al,al                                 ; 看 AL(函数名第 3 项)是否为空  
00FDE557  ^ 74 DC           je short BmJapane.00FDE535  
00FDE559    89F9            mov ecx,edi                              ; 读取函数名,放入 ECX 里面  
00FDE55B    57              push edi                                 ; 函数名  
00FDE55C    48              dec eax  
00FDE55D    F2:AE           repne scas byte ptr es:\[edi\]             ; 这条指令作用实际上是计算下一个函数的基地址  
00FDE55F    55              push ebp  
00FDE560    FF96 1445BE00   call dword ptr ds:\[esi+BE4514\]           ; GetProcAddress  
00FDE566    09C0            or eax,eax  
00FDE568    74 07           je short BmJapane.00FDE571               ; 正常情况下不会跳下去的,这里是处理无法取得函数地址的情况  
00FDE56A    8903            mov dword ptr ds:\[ebx\],eax               ; 取得地址后填充到\[ebx\]里面(IAT 表)  
00FDE56C    83C3 04         add ebx,4                                ; IAT 表地址+4bytes  
00FDE56F  ^ EB E1           jmp short BmJapane.00FDE552  
  
以下是其中一个函数的函数结构,与上面联系的:  
  
35 02 00 00 24 B9 99 00 01 61 63 6D 4D 65 74 72  5..$箼.acmMetr  
|----1----|  |----2----| 3  |----4-----  
69 63 73                                         ics  
\------|  
1->用 lea eax,dword ptr ds:\[eax+esi+BE43BC\]来标志该函数是哪个 DLL 文件里面的,把 DLL 文件的地址传给 EAX  
  
2-> 这部分加上基地址(401000)后用为填充程序 IAT 表的地址  
  
3->标识函数所在的 DLL 是否与前一个函数的 DLL 相同  
  
4->函数名 

 4dd466d0e1e0cc123afbe504da2462ca