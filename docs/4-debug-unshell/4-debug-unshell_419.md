

# ASProtect1.2x-1.3x 脱壳

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-626.htm`](https://www.pediy.com/kssd/pediy07/pediy7-626.htm)

【破解作者】 layper  
【作者邮箱】 layper2002@yahoo.com.cn  
【作者主页】 http://free4.e-168.cn/layper  
【使用工具】 PEID,OD,LordPE,ImportREC  
【破解平台】 Win9x/NT/2000/XP  
【软件名称】 ABC Amber Excel Converter 1.20 多国语言  
【下载地址】 http://www.onlinedown.net/soft/28806.htm  
【软件简介】 ABC Amber Excel Converter 是一个可以帮助你快速方便的将 XLS 文件 (MS Excel)很多格式(DBF,   
  
CSV, XML, PDF, HTML, CHM, RTF, HLP, DOC 等）,方便你保存收藏,程序内置了对近 50 种语言的支持,其中包括  
  
简体中文和繁体中文!  
  
【软件大小】 1182KB  
  
【加壳方式】 ASProtect 1.2x - 1.3x \[已经注册\] -> Alexey Solodovnikov  
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）  
\--------------------------------------------------------------------------------  
【破解内容】  
  
  
OD 载入，忽略所有异常，隐藏 OD  
00401000 >  68 01B05700     push abslogde.0057B001    ;停在入口处  
00401005    E8 01000000     call abslogde.0040100B  
0040100A    C3              retn  
  
在命令行下：he GetModuleHandleA,Shift+F9 运行两次后，清除硬件断点，Alt+F9 返回到这里  
00C094AC    85C0            test eax,eax                            ; kernel32.77E40000，返回这里  
  
，F8 单步,直到 00C095D1  
00C094AE    75 07           jnz short 00C094B7                      ;  
00C094B0    53              push ebx  
00C094B1    FF95 F0314400   call dword ptr ss:\[ebp+4431F0\]  
00C094B7    8985 4D294400   mov dword ptr ss:\[ebp+44294D\],eax  
00C094BD    C785 51294400 0>mov dword ptr ss:\[ebp+442951\],0  
00C094C7    8B95 D8304400   mov edx,dword ptr ss:\[ebp+4430D8\]  
00C094CD    8B06            mov eax,dword ptr ds:\[esi\]  
00C094CF    85C0            test eax,eax  
00C094D1    75 03           jnz short 00C094D6  
00C094D3    8B46 10         mov eax,dword ptr ds:\[esi+10\]  
00C094D6    03C2            add eax,edx  
00C094D8    0385 51294400   add eax,dword ptr ss:\[ebp+442951\]  
00C094DE    8B18            mov ebx,dword ptr ds:\[eax\]  
00C094E0    8B7E 10         mov edi,dword ptr ds:\[esi+10\]  
00C094E3    03FA            add edi,edx  
00C094E5    03BD 51294400   add edi,dword ptr ss:\[ebp+442951\]  
00C094EB    85DB            test ebx,ebx  
00C094ED    0F84 A2000000   je 00C09595  
00C094F3    F7C3 00000080   test ebx,80000000  
00C094F9    75 04           jnz short 00C094FF  
00C094FB    03DA            add ebx,edx  
00C094FD    43              inc ebx  
00C094FE    43              inc ebx  
00C094FF    53              push ebx  
00C09500    81E3 FFFFFF7F   and ebx,7FFFFFFF  
00C09506    53              push ebx  
00C09507    FFB5 4D294400   push dword ptr ss:\[ebp+44294D\]  
00C0950D    FF95 E8314400   call dword ptr ss:\[ebp+4431E8\]  
00C09513    85C0            test eax,eax  
00C09515    5B              pop ebx  
00C09516    75 6F           jnz short 00C09587  
00C09518    F7C3 00000080   test ebx,80000000  
00C0951E    75 19           jnz short 00C09539  
00C09520    57              push edi  
00C09521    8B46 0C         mov eax,dword ptr ds:\[esi+C\]  
00C09524    0385 D8304400   add eax,dword ptr ss:\[ebp+4430D8\]  
00C0952A    50              push eax  
00C0952B    53              push ebx  
00C0952C    8D85 53314400   lea eax,dword ptr ss:\[ebp+443153\]  
00C09532    50              push eax  
00C09533    57              push edi  
00C09534    E9 99000000     jmp 00C095D2  
00C09539    81E3 FFFFFF7F   and ebx,7FFFFFFF  
00C0953F    8B85 DC304400   mov eax,dword ptr ss:\[ebp+4430DC\]  
00C09545    3985 4D294400   cmp dword ptr ss:\[ebp+44294D\],eax  
00C0954B    75 24           jnz short 00C09571  
00C0954D    57              push edi  
00C0954E    8BD3            mov edx,ebx  
00C09550    4A              dec edx  
00C09551    C1E2 02         shl edx,2  
00C09554    8B9D 4D294400   mov ebx,dword ptr ss:\[ebp+44294D\]  
00C0955A    8B7B 3C         mov edi,dword ptr ds:\[ebx+3C\]  
00C0955D    8B7C3B 78       mov edi,dword ptr ds:\[ebx+edi+78\]  
00C09561    035C3B 1C       add ebx,dword ptr ds:\[ebx+edi+1C\]  
00C09565    8B0413          mov eax,dword ptr ds:\[ebx+edx\]  
00C09568    0385 4D294400   add eax,dword ptr ss:\[ebp+44294D\]  
00C0956E    5F              pop edi  
00C0956F    EB 16           jmp short 00C09587  
00C09571    57              push edi  
00C09572    8B46 0C         mov eax,dword ptr ds:\[esi+C\]  
00C09575    0385 D8304400   add eax,dword ptr ss:\[ebp+4430D8\]  
00C0957B    50              push eax  
00C0957C    53              push ebx  
00C0957D    8D85 A4314400   lea eax,dword ptr ss:\[ebp+4431A4\]  
00C09583    50              push eax  
00C09584    57              push edi  
00C09585    EB 4B           jmp short 00C095D2  
00C09587    8907            mov dword ptr ds:\[edi\],eax  
00C09589    8385 51294400 0>add dword ptr ss:\[ebp+442951\],4  
00C09590  ^ E9 32FFFFFF     jmp 00C094C7  
00C09595    8906            mov dword ptr ds:\[esi\],eax  
00C09597    8946 0C         mov dword ptr ds:\[esi+C\],eax  
00C0959A    8946 10         mov dword ptr ds:\[esi+10\],eax  
00C0959D    83C6 14         add esi,14  
00C095A0    8B95 D8304400   mov edx,dword ptr ss:\[ebp+4430D8\]  
00C095A6  ^ E9 EBFEFFFF     jmp 00C09496  
00C095AB    8B85 652A4400   mov eax,dword ptr ss:\[ebp+442A65\]  
00C095B1    50              push eax  
00C095B2    0385 D8304400   add eax,dword ptr ss:\[ebp+4430D8\]  
00C095B8    5B              pop ebx  
00C095B9    0BDB            or ebx,ebx  
00C095BB    8985 112F4400   mov dword ptr ss:\[ebp+442F11\],eax  
00C095C1    61              popad  
00C095C2    75 08           jnz short 00C095CC  
00C095C4    B8 01000000     mov eax,1  
00C095C9    C2 0C00         retn 0C  
00C095CC    68 00000000     push 0  
00C095D1    C3              retn                              ;执行到这里  
  
接下来是避开 IAT 加密  
Ctrl+S 搜索命令序列：  
mov edx,dword ptr ss:\[ebp+C\]  
mov edx,dword ptr ds:\[edx\]  
mov dword ptr ds:\[edx\],eax  
刚开始我怎么也搜索不到，后来才发现第一次搜索时要选定整个区块，接着后面的搜索再把整个区块这里去掉  
  
，不知是否是我机子的问题  
搜索得以下四处，全部下断：  
(1)  
00BF58CF    8B55 0C         mov edx,dword ptr ss:\[ebp+C\]  
00BF58D2    8B12            mov edx,dword ptr ds:\[edx\]  
00BF58D4    8902            mov dword ptr ds:\[edx\],eax  
(2)  
00BF593F    8B55 0C         mov edx,dword ptr ss:\[ebp+C\]  
00BF5942    8B12            mov edx,dword ptr ds:\[edx\]  
00BF5944    8902            mov dword ptr ds:\[edx\],eax  
(3)  
00BF5953    8B55 0C         mov edx,dword ptr ss:\[ebp+C\]  
00BF5956    8B12            mov edx,dword ptr ds:\[edx\]  
00BF5958    8902            mov dword ptr ds:\[edx\],eax  
  
(4)  
00BF5961    8B55 0C         mov edx,dword ptr ss:\[ebp+C\]  
00BF5964    8B12            mov edx,dword ptr ds:\[edx\]  
00BF5966    8902            mov dword ptr ds:\[edx\],eax  
  
Shift+F9 运行，运行之后按顺序断在(1)(3)(4)三处，  
00BF58CF    8B55 0C         mov edx,dword ptr ss:\[ebp+C\]      ；断在这里，取消断点  
00BF58D2    8B12            mov edx,dword ptr ds:\[edx\]  
00BF58D4    8902            mov dword ptr ds:\[edx\],eax         ；修改为 mov dword ptr ds:\[edx\],ebx  
寄存器 ebx 保存正确的函数，继续 Shift+F9 运行  
00BF5953    8B55 0C         mov edx,dword ptr ss:\[ebp+C\]       ；断在这里，取消断点  
00BF5956    8B12            mov edx,dword ptr ds:\[edx\]  
00BF5958    8902            mov dword ptr ds:\[edx\],eax  
Shift+F9 继续运行  
00BF595C    B8 B846BF00     mov eax,0BF46B8                      ；注意这里  
00BF5961    8B55 0C         mov edx,dword ptr ss:\[ebp+C\]         ；断在这里，取消断点，F8 走  
00BF5964    8B12            mov edx,dword ptr ds:\[edx\]  
00BF5966    8902            mov dword ptr ds:\[edx\],eax            ；走到这里，此时 EAX=0BF46B8  
Ctrl+G 到达 0BF46B8 看看  
00BF46B8    55              push ebp  
00BF46B9    8BEC            mov ebp,esp  
00BF46BB    8B55 0C         mov edx,dword ptr ss:\[ebp+C\]  
00BF46BE    8B45 08         mov eax,dword ptr ss:\[ebp+8\]  
00BF46C1    3B05 F823C000   cmp eax,dword ptr ds:\[C023F8\]  
00BF46C7    75 09           jnz short 00BF46D2  
00BF46C9    8B0495 F823C000 mov eax,dword ptr ds:\[edx\*4+C023F8\]  
00BF46D0    EB 07           jmp short 00BF46D9  
00BF46D2    52              push edx  
00BF46D3    50              push eax  
00BF46D4    E8 2710FFFF     call 00BE5700                           ; jmp to   
  
kernel32.GetProcAddress  
00BF46D9    5D              pop ebp  
00BF46DA    C2 0800         retn 8  
  
原来是 GetProcAddress，Ctrl+G 到达 00BE5700  
  
00BE5700  - FF25 7852C000   jmp dword ptr ds:\[C05278\]               ; kernel32.GetProcAddress  
  
在 OD 下方可以看到  
  
ds:\[00C05278\]=77E5B332 (kernel32.GetProcAddress)  
  
返回 00BF5966，修改 EAX 的值为 77E5B332  
  
(二)Pre-Dip:用自己名字注册  
  
到这里，我们要处理 Pre-Dip 完成注册，取消前面所有断点，重新设置 OD，去掉 OD 的内存访问异常的钩选(即不  
  
忽略内存访问异常)。  
Shift+F9 运行，注意堆栈中的变化，当第二次在堆栈中看到"W4kI2ABwYqk="硬盘指纹时，来到这里  
  
00C00DE4    0172 00         add dword ptr ds:\[edx\],esi          ;来到这里  
00C00DE7    6F              outs dx,dword ptr es:\[edi\]  
00C00DE8    09D9            or ecx,ebx  
00C00DEA    67:64:8F06 0000 pop dword ptr fs:\[0\]  
00C00DF0    EB 01           jmp short 00C00DF3  
00C00DF2    9A 83C404C1 D23>call far 31D2:C104C483  
00C00DF9    BA 5A294100     mov edx,41295A  
00C00DFE    5A              pop edx  
00C00DFF    A1 9026C000     mov eax,dword ptr ds:\[C02690\]  
00C00E04    C700 DD000000   mov dword ptr ds:\[eax\],0DD  
00C00E0A    8B15 3C26C000   mov edx,dword ptr ds:\[C0263C\]  
00C00E10    A1 8C26C000     mov eax,dword ptr ds:\[C0268C\]  
  
堆栈处为  
0012FF1C   0012FF2C  指针到下一个 SEH 记录  
0012FF20   00C00CEF  SE 句柄  
0012FF24   40E2D32B  
0012FF28   00C50564  ASCII "W4kI2ABwYqk="  
0012FF2C   0012FF80  指针到下一个 SEH 记录  
0012FF30   00C0114C  SE 句柄  
  
现在可以在内存查看处下断点了，Alt+M 打开内存查看窗口，在 00401000 处下内存访问断点，Shift+F9 运行来到  
  
这里  
004A3A00    8B4424 04       mov eax,dword ptr ss:\[esp+4\]            ;停在这里 F8 网下走  
004A3A04    8B0D 38245000   mov ecx,dword ptr ds:\[502438\]  
004A3A0A    85C9            test ecx,ecx  
004A3A0C    A3 885B5100     mov dword ptr ds:\[515B88\],eax           ;走到这里，eax 保存注册名的地  
  
方，我们要注册，就要在这里对 eax 动手脚了  
004A3A11    8B4424 08       mov eax,dword ptr ss:\[esp+8\]  
004A3A15    A3 8C5B5100     mov dword ptr ds:\[515B8C\],eax  
004A3A1A    7D 05           jge short abslogde.004A3A21  
004A3A1C    A3 38245000     mov dword ptr ds:\[502438\],eax  
004A3A21    C2 0800         retn 8  
  
找一块空地写下名字，然后把 004A3A0C 处的 EAX 修改为你名字的地址就行了，我找了 004D2FE4 这里用 laype 注册  
  
修改 004A3A0C 处的 EAX=004D2FE4，处理完毕  
  
查找 OEP  
刚才我们设置的内存访问断点继续保持有效，但重新设置 OD，OD 中的内存访问异常重新打上勾(忽略所有异常)  
  
，Shift+F9 运行多次，来到这里  
  
00470B7D    6A 60           push 60                               ;停在这里，这里就是 OEP 了  
00470B7F    68 C8374D00     push abslogde.004D37C8  
00470B84    E8 77020000     call abslogde.00470E00  
00470B89    BF 94000000     mov edi,94  
00470B8E    8BC7            mov eax,edi  
00470B90    E8 9B6F0000     call abslogde.00477B30  
00470B95    8965 E8         mov dword ptr ss:\[ebp-18\],esp  
00470B98    8BF4            mov esi,esp  
00470B9A    893E            mov dword ptr ds:\[esi\],edi  
00470B9C    56              push esi  
00470B9D    FF15 E0324D00   call dword ptr ds:\[4D32E0\]              ; kernel32.GetVersionExA  
00470BA3    8B4E 10         mov ecx,dword ptr ds:\[esi+10\]  
  
不关闭 OD，用 LordPE 完整脱壳保存为 dumped，在用 ImportREC 抓取修复 dumped  
运行 ImportREC，选择这个进程。  
修改 OEP＝70B7D、自动搜索 IAT，RAV=000D3000，大小=00000608，获取输入表显示全为真修复抓取 dumped 的  
  
dumped\_ 即为脱壳后的文件。  
  
去除 2 个暗桩  
  
试运行脱壳后的 dumped\_，这个软件有暗桩，我这里是用跟踪原程序来找暗桩  
第一处：  
004A3D5A   /74 1B           je short abslogde.004A3D77              ；这里是第一处  
004A3D5C   |6A 10           push 10  
004A3D5E   |68 70904E00     push abslogde.004E9070                  ; ASCII "Absolute Log   
  
Analyzer"  
004A3D63   |68 88904E00     push abslogde.004E9088                  ; ASCII "Failed to initialize   
  
application"  
004A3D68   |53              push ebx  
004A3D69   |FF15 14344D00   call dword ptr ds:\[4D3414\]              ; user32.MessageBoxA  
  
原程序 004A3D5A 这里为跳，而脱壳后的不跳，修改脱壳后的 je 为 jmp，试运行，还有一个暗桩  
第二处:  
004C8A1C   /74 1E           je short abslogde.004C8A3C             ;这里为第二处  
004C8A1E   |6A 10           push 10  
004C8A20   |68 18934D00     push abslogde.004D9318                  ; ASCII "Absolute Log   
  
Analyzer"  
004C8A25   |68 30934D00     push abslogde.004D9330                  ; ASCII "CRC check failed.   
  
Application corrupted.  
Execution impossible."  
004C8A2A   |6A 00           push 0  
004C8A2C   |FF15 14344D00   call dword ptr ds:\[4D3414\]              ; user32.MessageBoxA  
  
同样修改 004C8A1C 的 je 为 jmp 保存，脱壳成功。  
呵呵，这篇参照了 fly 大侠的 ASProtect V2.0 脱壳——Registry Clean Expert V3.52 UnPacked + 去除自检验，在此表示谢谢了。  
\--------------------------------------------------------------------------------  
【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢! 

 4dd466d0e1e0cc123afbe504da2462ca