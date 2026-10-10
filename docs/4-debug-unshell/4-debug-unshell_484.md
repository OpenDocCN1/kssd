

# Magic NetTrace V2.1.3 

> 原文：[`www.pediy.com/kssd/pediy06/pediy6712.htm`](https://www.pediy.com/kssd/pediy06/pediy6712.htm)

下载页面：  [`bbs.pediy.com/showthread.php?s=&threadid=5311&perpage=10&pagenumber=7`](http://bbs.pediy.com/showthread.php?s=&threadid=5311&perpage=10&pagenumber=7)   
软件大小：  1.32 M  
软件语言：  英文  
软件类别：  国外软件 / 共享版 / 网络辅助  
应用平台：  Win9x/NT/2000/XP  
加入时间：  2003-12  
推荐等级：  \*\*\*\*  
软件介绍：  这个程序结合 ping，WHOIS 和 traceroute 功能。  
  
【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！  
  
【调试环境】：WinXP、Ollydbg、PEiD、LordPE、ImportREC  
  
—————————————————————————————————   
【脱壳过程】：  
            
           
jingulong 说起这个程序，我脱壳时使用了壳代码来修复。  
Thanks: VolX、jingulong、lipton  
—————————————————————————————————  
一、壳代码解压  
               
               
设置 Ollydbg 忽略所有的异常选项。老规矩：用 IsDebug V1.4 插件去掉 Ollydbg 的调试器标志。  
  
00401000     68 01505A00         push mTrace.005A5001  
//进入 OD 后停在这  
00401005     E8 01000000         call mTrace.0040100B  
0040100A     C3                  retn  
  
下断：BP GetModuleHandleA  
Shift+F9，中断 2 次后，就可以取消断点，Alt+F9 返回。  
直接 F4 至下面的 popad 处  
  
00B834A6     FF95 EC314400       call dword ptr ss:\[ebp+4431EC\]  
00B834AC     85C0                test eax,eax                        ; kernel32.77E40000  
00B834AE     75 07               jnz short 00B834B7  
00B834B0     53                  push ebx  
00B834B1     FF95 F0314400       call dword ptr ss:\[ebp+4431F0\]  
00B834B7     8985 4D294400       mov dword ptr ss:\[ebp+44294D\],eax  
00B834BD     C785 51294400 00000>mov dword ptr ss:\[ebp+442951\],0  
00B834C7     8B95 D8304400       mov edx,dword ptr ss:\[ebp+4430D8\]  
00B834CD     8B06                mov eax,dword ptr ds:\[esi\]  
00B834CF     85C0                test eax,eax  
00B834D1     75 03               jnz short 00B834D6  
00B834D3     8B46 10             mov eax,dword ptr ds:\[esi+10\]  
00B834D6     03C2                add eax,edx  
00B834D8     0385 51294400       add eax,dword ptr ss:\[ebp+442951\]  
00B834DE     8B18                mov ebx,dword ptr ds:\[eax\]  
00B834E0     8B7E 10             mov edi,dword ptr ds:\[esi+10\]  
00B834E3     03FA                add edi,edx  
00B834E5     03BD 51294400       add edi,dword ptr ss:\[ebp+442951\]  
00B834EB     85DB                test ebx,ebx  
00B834ED     0F84 A2000000       je 00B83595  
00B834F3     F7C3 00000080       test ebx,80000000  
00B834F9     75 04               jnz short 00B834FF  
00B834FB     03DA                add ebx,edx  
00B834FD     43                  inc ebx  
00B834FE     43                  inc ebx  
00B834FF     53                  push ebx  
00B83500     81E3 FFFFFF7F       and ebx,7FFFFFFF  
00B83506     53                  push ebx  
00B83507     FFB5 4D294400       push dword ptr ss:\[ebp+44294D\]  
00B8350D     FF95 E8314400       call dword ptr ss:\[ebp+4431E8\]  
00B83513     85C0                test eax,eax  
00B83515     5B                  pop ebx  
00B83516     75 6F               jnz short 00B83587  
00B83518     F7C3 00000080       test ebx,80000000  
00B8351E     75 19               jnz short 00B83539  
00B83520     57                  push edi  
00B83521     8B46 0C             mov eax,dword ptr ds:\[esi+C\]  
00B83524     0385 D8304400       add eax,dword ptr ss:\[ebp+4430D8\]  
00B8352A     50                  push eax  
00B8352B     53                  push ebx  
00B8352C     8D85 53314400       lea eax,dword ptr ss:\[ebp+443153\]  
00B83532     50                  push eax  
00B83533     57                  push edi  
00B83534     E9 99000000         jmp 00B835D2  
00B83539     81E3 FFFFFF7F       and ebx,7FFFFFFF  
00B8353F     8B85 DC304400       mov eax,dword ptr ss:\[ebp+4430DC\]  
00B83545     3985 4D294400       cmp dword ptr ss:\[ebp+44294D\],eax  
00B8354B     75 24               jnz short 00B83571  
00B8354D     57                  push edi  
00B8354E     8BD3                mov edx,ebx  
00B83550     4A                  dec edx  
00B83551     C1E2 02             shl edx,2  
00B83554     8B9D 4D294400       mov ebx,dword ptr ss:\[ebp+44294D\]  
00B8355A     8B7B 3C             mov edi,dword ptr ds:\[ebx+3C\]  
00B8355D     8B7C3B 78           mov edi,dword ptr ds:\[ebx+edi+78\]  
00B83561     035C3B 1C           add ebx,dword ptr ds:\[ebx+edi+1C\]  
00B83565     8B0413              mov eax,dword ptr ds:\[ebx+edx\]  
00B83568     0385 4D294400       add eax,dword ptr ss:\[ebp+44294D\]  
00B8356E     5F                  pop edi  
00B8356F     EB 16               jmp short 00B83587  
00B83571     57                  push edi  
00B83572     8B46 0C             mov eax,dword ptr ds:\[esi+C\]  
00B83575     0385 D8304400       add eax,dword ptr ss:\[ebp+4430D8\]  
00B8357B     50                  push eax  
00B8357C     53                  push ebx  
00B8357D     8D85 A4314400       lea eax,dword ptr ss:\[ebp+4431A4\]  
00B83583     50                  push eax  
00B83584     57                  push edi  
00B83585     EB 4B               jmp short 00B835D2  
00B83587     8907                mov dword ptr ds:\[edi\],eax  
00B83589     8385 51294400 04    add dword ptr ss:\[ebp+442951\],4  
00B83590     E9 32FFFFFF         jmp 00B834C7  
00B83595     8906                mov dword ptr ds:\[esi\],eax  
00B83597     8946 0C             mov dword ptr ds:\[esi+C\],eax  
00B8359A     8946 10             mov dword ptr ds:\[esi+10\],eax  
00B8359D     83C6 14             add esi,14  
00B835A0     8B95 D8304400       mov edx,dword ptr ss:\[ebp+4430D8\]  
00B835A6     E9 EBFEFFFF         jmp 00B83496  
00B835AB     8B85 652A4400       mov eax,dword ptr ss:\[ebp+442A65\]  
00B835B1     50                  push eax  
00B835B2     0385 D8304400       add eax,dword ptr ss:\[ebp+4430D8\]  
00B835B8     5B                  pop ebx  
00B835B9     0BDB                or ebx,ebx  
00B835BB     8985 112F4400       mov dword ptr ss:\[ebp+442F11\],eax  
00B835C1     61                  popad  
//直接 F4 到这里  
00B835C2     75 08               jnz short 00B835CC  
00B835C4     B8 01000000         mov eax,1  
00B835C9     C2 0C00             retn 0C  
00B835CC     68 24BEB700         push 0B7BE24  
00B835D1     C3                  retn  
//壳代码解压完毕  
  
  
—————————————————————————————————  
二、处理输入表，避开加密  
             
             
壳代码已经解压完毕，可以开始处理输入表了，手动 Patch，避开输入表的加密。  
             
Ctrl+S 在“整个段块”搜索命令序列：  
add esp,44  
pop ebp  
pop edi  
pop esi  
pop ebx  
retn  
找到在 00B779F3 处。上面就是输入表的处理代码段。  
下面是分析过程。可以在搜索到地址后直接修改了，免得检验出错。  
  
00B775E0     8B07                mov eax,dword ptr ds:\[edi\]  
00B775E2     8B18                mov ebx,dword ptr ds:\[eax\]  
00B775E4     8307 04             add dword ptr ds:\[edi\],4  
00B775E7     8B07                mov eax,dword ptr ds:\[edi\]  
00B775E9     8A00                mov al,byte ptr ds:\[eax\]  
00B775EB     884424 1E           mov byte ptr ss:\[esp+1E\],al  
00B775EF     FF07                inc dword ptr ds:\[edi\]  
00B775F1     85DB                test ebx,ebx  
//这里下断，Shift+F9 中断后就可以开始处理了  
00B775F3     0F85 99000000       jnz 00B77692  
00B775F9     EB 01               jmp short 00B775FC  
//IAT 处理完毕则自这里跳转  
  
00B7769E     8B07                mov eax,dword ptr ds:\[edi\]  
00B776A0     8A00                mov al,byte ptr ds:\[eax\]  
00B776A2     FF07                inc dword ptr ds:\[edi\]  
00B776A4     33D2                xor edx,edx  
00B776A6     8AD0                mov dl,al  
00B776A8     8B4424 2C           mov eax,dword ptr ss:\[esp+2C\]  
00B776AC     E8 E3F3FFFF         call 00B76A94  
00B776B1     894424 30           mov dword ptr ss:\[esp+30\],eax  
00B776B5     8B07                mov eax,dword ptr ds:\[edi\]  
00B776B7     8A00                mov al,byte ptr ds:\[eax\]  
00B776B9     FF07                inc dword ptr ds:\[edi\]  
00B776BB     84C0                test al,al  
00B776BD     75 44               jnz short 00B77703  
//下面根据 AL 值分别进行处理  
00B776BF     EB 01               jmp short 00B776C2  
  
  
————————————————————————  
1、AL=1 时的处理  
  
  
00B7784E     3C 01               cmp al,1  
//AL=1   ?  
00B77850     0F85 9C000000       jnz 00B778F2  
00B77856     EB 01               jmp short 00B77859  
00B77859     895C24 18           mov dword ptr ss:\[esp+18\],ebx  
00B7785D     8B07                mov eax,dword ptr ds:\[edi\]  
00B7785F     8B28                mov ebp,dword ptr ds:\[eax\]  
00B77861     8307 04             add dword ptr ds:\[edi\],4  
00B77864     8BC5                mov eax,ebp  
00B77866     FF5424 04           call dword ptr ss:\[esp+4\]  
00B7786A     8BE8                mov ebp,eax  
00B7786C     8B07                mov eax,dword ptr ds:\[edi\]  
00B7786E     8A00                mov al,byte ptr ds:\[eax\]  
00B77870     884424 1D           mov byte ptr ss:\[esp+1D\],al  
00B77874     FF07                inc dword ptr ds:\[edi\]  
00B77876     807C24 3C 00        cmp byte ptr ss:\[esp+3C\],0  
00B7787B     74 11               je short 00B7788E  
00B7787D     A1 64E7B700         mov eax,dword ptr ds:\[B7E764\]  
00B77882     A3 68E7B700         mov dword ptr ds:\[B7E768\],eax  
00B77887     C64424 3C 00        mov byte ptr ss:\[esp+3C\],0  
00B7788C     EB 0B               jmp short 00B77899  
00B7788E     8B4424 34           mov eax,dword ptr ss:\[esp+34\]  
00B77892     8B00                mov eax,dword ptr ds:\[eax\]  
00B77894     A3 68E7B700         mov dword ptr ds:\[B7E768\],eax  
00B77899     8B0D 68E7B700       mov ecx,dword ptr ds:\[B7E768\]  
00B7789F     8A5424 1D           mov dl,byte ptr ss:\[esp+1D\]  
00B778A3     8B07                mov eax,dword ptr ds:\[edi\]  
00B778A5     E8 1AD5FFFF         call 00B74DC4  
00B778AA     8B4424 24           mov eax,dword ptr ss:\[esp+24\]  
00B778AE     E8 91ACFEFF         call 00B62544  
00B778B3     8BF0                mov esi,eax  
00B778B5     8B07                mov eax,dword ptr ds:\[edi\]  
00B778B7     50                  push eax  
00B778B8     8B4424 34           mov eax,dword ptr ss:\[esp+34\]  
00B778BC     50                  push eax  
00B778BD     A1 1CC6B700         mov eax,dword ptr ds:\[B7C61C\]  
00B778C2     8B00                mov eax,dword ptr ds:\[eax\]  
00B778C4     FFD0                call eax  
//GetProcAddress        ★  
00B778C6     894424 38           mov dword ptr ss:\[esp+38\],eax  
00B778CA     6A 00               push 0  
00B778CC     8D5424 38           lea edx,dword ptr ss:\[esp+38\]  
00B778D0     8D4424 3C           lea eax,dword ptr ss:\[esp+3C\]  
00B778D4     8BCD                mov ecx,ebp  
00B778D6     E8 EDF8FFFF         call 00B771C8  
//加密 CALL！进入修改 ★  
00B778DB     0305 B8C4B700       add eax,dword ptr ds:\[B7C4B8\]  
00B778E1     8906                mov dword ptr ds:\[esi\],eax  
00B778E3     8933                mov dword ptr ds:\[ebx\],esi  
//加密地址写入程序！NOP 掉 ★  
00B778E5     33C0                xor eax,eax  
00B778E7     8A4424 1D           mov al,byte ptr ss:\[esp+1D\]  
00B778EB     0107                add dword ptr ds:\[edi\],eax  
00B778ED     E9 EEFCFFFF         jmp 00B775E0  
//循环  
  
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆  
★ Patch 处理①&② ★ ：  
            
00B771C8     55                  push ebp  
00B771C9     8BEC                mov ebp,esp  
00B771CB     81C4 50FDFFFF       add esp,-2B0  
00B771D1     53                  push ebx  
00B771D2     56                  push esi  
00B771D3     57                  push edi  
00B771D4     894D F8             mov dword ptr ss:\[ebp\-8\],ecx  
00B771D7     8955 FC             mov dword ptr ss:\[ebp\-4\],edx  
00B771DA     8BF8                mov edi,eax  
00B771DC     33C0                xor eax,eax  
//此时\[EDI\]=正确的函数地址 ★  
00B771DE     8D95 93FDFFFF       lea edx,dword ptr ss:\[ebp\-26D\]  
00B771E4     8955 EC             mov dword ptr ss:\[ebp\-14\],edx  
00B771E7     33F6                xor esi,esi  
//修改①：jmp 00B83908    ★   跳到 Patch 部分  
00B771E9     33DB                xor ebx,ebx  
00B771EB     8B17                mov edx,dword ptr ds:\[edi\]  
00B771ED     8955 E4             mov dword ptr ss:\[ebp\-1C\],edx  
00B771F0     33D2                xor edx,edx  
00B771F2     8955 F0             mov dword ptr ss:\[ebp\-10\],edx  
00B771F5     833F 00             cmp dword ptr ds:\[edi\],0  
00B771F8     0F84 01020000       je 00B773FF  
00B771FE     E8 C9C9FFFF         call 00B73BCC  
00B77203     EB 17               jmp short 00B7721C  
  
  
在程序下面找快空地，写入 Patch 代码。00B83908 下面都是空地，就这里吧  
提前在\[B84ED0\]处写入现在 DLL 的基址，在\[B84EE0\]处写入准备放输入表的地址：00563000  
  
00B83908     51                  push ecx  
00B83909     52                  push edx  
00B8390A     3E:8B9424 00030000  mov edx,dword ptr ds:\[esp+300\]; kernel32.77E40000  
//\[esp+300\]＝0012FF2C 此处保存的是处理的 DLL 基址 ★  
00B83912     3B15 D04EB800       cmp edx,dword ptr ds:\[B84ED0\] ; kernel32.77E40000  
//在\[B84ED0\]处提前写入现在处理的 DLL 的基址：77E40000  
00B83918     74 0D               je short 00B83927  
//判断与上次 DLL 是否相同 ★  
00B8391A     8915 D04EB800       mov dword ptr ds:\[B84ED0\],edx  
00B83920     8305 E04EB800 04    add dword ptr ds:\[B84EE0\],4  
//不同则地址+4 ★  
00B83927     8B0D E04EB800       mov ecx,dword ptr ds:\[B84EE0\] ; 00563008  
//提前在\[B84EE0\]处写入准备放输入表的地址：00563000  
00B8392D     8B17                mov edx,dword ptr ds:\[edi\]    ; kernel32.GetFileType  
00B8392F     8B0A                mov ecx,dword ptr ds:\[edx\]  
//正确的函数写入  
00B83931     8B19                mov ebx,dword ptr ds:\[ecx\]  
//取代原先入壳的跳转地址  
00B83933     C643 FF 25          mov byte ptr ds:\[ebx\-1\],25  
//把\[ebx-2\]处的 FF15 修改成 FF25  
00B83937     8305 E04EB800 04    add dword ptr ds:\[B84EE0\],4  
00B8393E     5A                  pop edx  
00B8393F     59                  pop ecx  
00B83940     33F6                xor esi,esi  
//原先 00B771E7 处代码挪到这里来运行  
00B83942     33DB                xor ebx,ebx  
00B83944     8B17                mov edx,dword ptr ds:\[edi\]  
00B83946     E9 A238FFFF         jmp 00B771ED  
//跳回去继续流程  
  
  
————————————————————————  
2、AL=2 时的处理  
  
  
00B77703     3C 02               cmp al,2  
//AL=2   ?  
00B77705     0F85 43010000       jnz 00B7784E  
00B7770B     EB 01               jmp short 00B7770E  
00B7770E     33ED                xor ebp,ebp  
00B77710     8B07                mov eax,dword ptr ds:\[edi\]  
00B77712     8A00                mov al,byte ptr ds:\[eax\]  
00B77714     FF07                inc dword ptr ds:\[edi\]  
00B77716     2C 01               sub al,1  
00B77718     73 30               jnb short 00B7774A  
00B7771A     8B07                mov eax,dword ptr ds:\[edi\]  
00B7771C     8A00                mov al,byte ptr ds:\[eax\]  
00B7771E     FF07                inc dword ptr ds:\[edi\]  
00B77720     8B17                mov edx,dword ptr ds:\[edi\]  
00B77722     8B12                mov edx,dword ptr ds:\[edx\]  
00B77724     8307 04             add dword ptr ds:\[edi\],4  
00B77727     807C24 1E 01        cmp byte ptr ss:\[esp+1E\],1  
00B7772C     75 0F               jnz short 00B7773D  
00B7772E     6A 01               push 1  
00B77730     8D4C24 44           lea ecx,dword ptr ss:\[esp+44\]  
00B77734     E8 EFFCFFFF         call 00B77428  
00B77739     8BE8                mov ebp,eax  
00B7773B     EB 0D               jmp short 00B7774A  
00B7773D     6A 00               push 0  
00B7773F     8D4C24 44           lea ecx,dword ptr ss:\[esp+44\]  
00B77743     E8 E0FCFFFF         call 00B77428  
00B77748     8BE8                mov ebp,eax  
00B7774A     8B07                mov eax,dword ptr ds:\[edi\]  
00B7774C     8B00                mov eax,dword ptr ds:\[eax\]  
00B7774E     894424 10           mov dword ptr ss:\[esp+10\],eax  
00B77752     8307 04             add dword ptr ds:\[edi\],4  
00B77755     8B4424 24           mov eax,dword ptr ss:\[esp+24\]  
00B77759     E8 E6ADFEFF         call 00B62544  
00B7775E     8BF0                mov esi,eax  
00B77760     8B5424 10           mov edx,dword ptr ss:\[esp+10\]  
00B77764     8B4424 30           mov eax,dword ptr ss:\[esp+30\]  
00B77768     E8 AFF0FFFF         call 00B7681C  
00B7776D     894424 38           mov dword ptr ss:\[esp+38\],eax  
00B77771     EB 01               jmp short 00B77774  
00B77774     85ED                test ebp,ebp  
00B77776     0F85 9B000000       jnz 00B77817  
00B7777C     6A 01               push 1  
00B7777E     8D5424 38           lea edx,dword ptr ss:\[esp+38\]  
00B77782     8D4424 3C           lea eax,dword ptr ss:\[esp+3C\]  
00B77786     33C9                xor ecx,ecx  
00B77788     E8 3BFAFFFF         call 00B771C8  
00B7778D     2B05 B8C4B700       sub eax,dword ptr ds:\[B7C4B8\]  
00B77793     8906                mov dword ptr ds:\[esi\],eax  
00B77795     807C24 1E 01        cmp byte ptr ss:\[esp+1E\],1  
00B7779A     0F85 A7000000       jnz 00B77847  
00B777A0     8B4424 24           mov eax,dword ptr ss:\[esp+24\]  
00B777A4     83C0 32             add eax,32  
00B777A7     E8 98ADFEFF         call 00B62544  
00B777AC     8BE8                mov ebp,eax  
00B777AE     55                  push ebp  
00B777AF     A1 5CE7B700         mov eax,dword ptr ds:\[B7E75C\]  
00B777B4     50                  push eax  
00B777B5     E8 76C8FFFF         call 00B74030  
00B777BA     A1 5CE7B700         mov eax,dword ptr ds:\[B7E75C\]  
00B777BF     50                  push eax  
00B777C0     E8 7FD4FFFF         call 00B74C44  
00B777C5     A1 5CE7B700         mov eax,dword ptr ds:\[B7E75C\]  
00B777CA     E8 81C8FFFF         call 00B74050  
00B777CF     03C5                add eax,ebp  
00B777D1     894424 28           mov dword ptr ss:\[esp+28\],eax  
00B777D5     B8 02000000         mov eax,2  
00B777DA     E8 E5AFFEFF         call 00B627C4  
00B777DF     48                  dec eax  
00B777E0     75 17               jnz short 00B777F9  
00B777E2     8B16                mov edx,dword ptr ds:\[esi\]  
00B777E4     2B5424 28           sub edx,dword ptr ss:\[esp+28\]  
00B777E8     83EA 05             sub edx,5  
00B777EB     8B4424 28           mov eax,dword ptr ss:\[esp+28\]  
00B777EF     E8 2CEBFFFF         call 00B76320  
00B777F4     EB 1D               jmp short 00B77813  
00B77817     892E                mov dword ptr ds:\[esi\],ebp  
00B77819     8BC5                mov eax,ebp  
00B7781B     034424 40           add eax,dword ptr ss:\[esp+40\]  
00B7781F     C600 68             mov byte ptr ds:\[eax\],68  
00B77822     6A 01               push 1  
00B77824     8D5424 38           lea edx,dword ptr ss:\[esp+38\]  
00B77828     8D4424 3C           lea eax,dword ptr ss:\[esp+3C\]  
00B7782C     33C9                xor ecx,ecx  
00B7782E     E8 95F9FFFF         call 00B771C8  
//同样的加密 CALL！进入修改 ★   Patch 处理②方法同 Patch 处理①  
00B77833     8BD5                mov edx,ebp  
00B77835     035424 40           add edx,dword ptr ss:\[esp+40\]  
00B77839     42                  inc edx  
00B7783A     8902                mov dword ptr ds:\[edx\],eax  
00B7783C     036C24 40           add ebp,dword ptr ss:\[esp+40\]  
00B77840     83C5 05             add ebp,5  
00B77843     C645 00 C3          mov byte ptr ss:\[ebp\],0C3  
00B77847     8933                mov dword ptr ds:\[ebx\],esi  
//加密地址写入程序！NOP 掉 ★  
00B77849     E9 92FDFFFF         jmp 00B775E0  
//循环  
  
  
————————————————————————  
3、AL=3 时的处理  
            
            
处理特殊函数：GetProcAddress 和 FindResourceA  
  
00B77952     3C 03               cmp al,3  
//AL=3   ?  
00B77954     75 7D               jnz short 00B779D3  
00B77956     EB 01               jmp short 00B77959  
00B77959     8B07                mov eax,dword ptr ds:\[edi\]  
00B7795B     8A00                mov al,byte ptr ds:\[eax\]  
00B7795D     FF07                inc dword ptr ds:\[edi\]  
00B7795F     FEC8                dec al  
00B77961     74 09               je short 00B7796C  
00B77963     FEC8                dec al  
00B77965     74 30               je short 00B77997  
00B77967     E9 74FCFFFF         jmp 00B775E0  
00B7796C     8B07                mov eax,dword ptr ds:\[edi\]  
00B7796E     8A00                mov al,byte ptr ds:\[eax\]  
00B77970     884424 1C           mov byte ptr ss:\[esp+1C\],al  
00B77974     FF07                inc dword ptr ds:\[edi\]  
00B77976     8B4424 24           mov eax,dword ptr ss:\[esp+24\]  
00B7797A     E8 C5ABFEFF         call 00B62544  
00B7797F     8BF0                mov esi,eax  
//修改③：jmp 00B8398C   ★   跳到 Patch 部分  
00B77981     8933                mov dword ptr ds:\[ebx\],esi  
00B77983     8A4C24 1C           mov cl,byte ptr ss:\[esp+1C\]  
00B77987     8B5424 30           mov edx,dword ptr ss:\[esp+30\]  
00B7798B     8BC6                mov eax,esi  
00B7798D     E8 B2F4FFFF         call 00B76E44  
00B77992     E9 49FCFFFF         jmp 00B775E0  
//循环  
00B77997     8B07                mov eax,dword ptr ds:\[edi\]  
00B77999     8B30                mov esi,dword ptr ds:\[eax\]  
00B7799B     8307 04             add dword ptr ds:\[edi\],4  
00B7799E     8BC6                mov eax,esi  
00B779A0     E8 9FABFEFF         call 00B62544  
00B779A5     894424 20           mov dword ptr ss:\[esp+20\],eax  
00B779A9     8BCE                mov ecx,esi  
00B779AB     8B17                mov edx,dword ptr ds:\[edi\]  
00B779AD     8B4424 20           mov eax,dword ptr ss:\[esp+20\]  
00B779B1     E8 76DCFEFF         call 00B6562C  
00B779B6     0137                add dword ptr ds:\[edi\],esi  
00B779B8     8B4424 24           mov eax,dword ptr ss:\[esp+24\]  
00B779BC     E8 83ABFEFF         call 00B62544  
00B779C1     8BF0                mov esi,eax  
00B779C3     8B4424 20           mov eax,dword ptr ss:\[esp+20\]  
00B779C7     8906                mov dword ptr ds:\[esi\],eax  
00B779C9     8933                mov dword ptr ds:\[ebx\],esi  
00B779CB     E9 10FCFFFF         jmp 00B775E0  
00B779D3     BA FC79B700         mov edx,0B779FC ; ASCII "43"  
00B779D8     A1 28C6B700         mov eax,dword ptr ds:\[B7C628\]  
00B779DD     8B00                mov eax,dword ptr ds:\[eax\]  
00B779DF     E8 48F0FFFF         call 00B76A2C  
//Game Over  
00B779E4     E9 F7FBFFFF         jmp 00B775E0  
  
  
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆  
★ Patch 处理③ ★ ：  
  
00B8398C     3E:8B4424 30        mov eax,dword ptr ds:\[esp+30\]  
00B83991     3B05 D04EB800       cmp eax,dword ptr ds:\[B84ED0\]  
00B83997     74 0C               je short 00B839A5  
00B83999     A3 D04EB800         mov dword ptr ds:\[B84ED0\],eax  
00B8399E     8305 E04EB800 04    add dword ptr ds:\[B84EE0\],4  
00B839A5     A1 E04EB800         mov eax,dword ptr ds:\[B84EE0\]  
//放入我们程序中的地址  
00B839AA     8BF0                mov esi,eax  
//00B7797F 处代码挪到这里执行  
00B839AC     8933                mov dword ptr ds:\[ebx\],esi  
00B839AE     8A4C24 1C           mov cl,byte ptr ss:\[esp+1C\]  
00B839B2     8305 E04EB800 04    add dword ptr ds:\[B84EE0\],4  
00B839B9     E9 C93FFFFF         jmp 00B77987  
  
————————————————————————  
4、AL=4 时的处理  
  
  
00B778F2     3C 04               cmp al,4  
//AL=4   ?  
00B778F4     75 5C               jnz short 00B77952  
00B778F6     EB 01               jmp short 00B778F9  
00B778F9     8B07                mov eax,dword ptr ds:\[edi\]  
00B778FB     8B28                mov ebp,dword ptr ds:\[eax\]  
00B778FD     8307 04             add dword ptr ds:\[edi\],4  
00B77900     8B07                mov eax,dword ptr ds:\[edi\]  
00B77902     8A00                mov al,byte ptr ds:\[eax\]  
00B77904     884424 1D           mov byte ptr ss:\[esp+1D\],al  
00B77908     FF07                inc dword ptr ds:\[edi\]  
00B7790A     8BCD                mov ecx,ebp  
00B7790C     8A5424 1D           mov dl,byte ptr ss:\[esp+1D\]  
00B77910     8B07                mov eax,dword ptr ds:\[edi\]  
00B77912     E8 ADD4FFFF         call 00B74DC4  
00B77917     8B4424 24           mov eax,dword ptr ss:\[esp+24\]  
00B7791B     E8 24ACFEFF         call 00B62544  
00B77920     8BF0                mov esi,eax  
00B77922     8B07                mov eax,dword ptr ds:\[edi\]  
00B77924     50                  push eax  
00B77925     8B4424 34           mov eax,dword ptr ss:\[esp+34\]  
00B77929     50                  push eax  
00B7792A     A1 1CC6B700         mov eax,dword ptr ds:\[B7C61C\]  
00B7792F     8B00                mov eax,dword ptr ds:\[eax\]  
00B77931     FFD0                call eax  
00B77933     894424 38           mov dword ptr ss:\[esp+38\],eax  
00B77937     8B4424 38           mov eax,dword ptr ss:\[esp+38\]  
00B7793B     2B05 B8C4B700       sub eax,dword ptr ds:\[B7C4B8\]  
00B77941     8906                mov dword ptr ds:\[esi\],eax  
00B77943     8933                mov dword ptr ds:\[ebx\],esi  
//修改④：jmp 00B839C2   ★   跳到 Patch 部分  
00B77945     33C0                xor eax,eax  
00B77947     8A4424 1D           mov al,byte ptr ss:\[esp+1D\]  
00B7794B     0107                add dword ptr ds:\[edi\],eax  
00B7794D     E9 8EFCFFFF         jmp 00B775E0  
  
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆  
★ Patch 处理④ ★ ：  
  
00B839C2     52                  push edx  
00B839C3     3E:8B5424 34        mov edx,dword ptr ds:\[esp+34\]  
00B839C8     3B15 D04EB800       cmp edx,dword ptr ds:\[B84ED0\]  
00B839CE     74 0D               je short 00B839DD  
00B839D0     8915 D04EB800       mov dword ptr ds:\[B84ED0\],edx  
00B839D6     8305 E04EB800 04    add dword ptr ds:\[B84EE0\],4  
00B839DD     8B35 E04EB800       mov esi,dword ptr ds:\[B84EE0\]  
00B839E3     8906                mov dword ptr ds:\[esi\],eax  
00B839E5     8933                mov dword ptr ds:\[ebx\],esi  
//取代原先入壳的跳转地址  
00B839E7     8305 E04EB800 04    add dword ptr ds:\[B84EE0\],4  
00B839EE     5A                  pop edx  
00B839EF     33C0                xor eax,eax  
//00B77945 处代码挪到这里执行  
00B839F1     8A4424 1D           mov al,byte ptr ss:\[esp+1D\]  
00B839F5     E9 513FFFFF         jmp 00B7794B  
  
  
—————————————————————————————————  
三、壳自校验  
  
  
修改完以上部分后在 00B775FC 处下断，Shift+F9 中断  
  
00B775FC     837C24 2C 00        cmp dword ptr ss:\[esp+2C\],0  
//此处下断，IAT 处理完毕，开始检验部分  
00B77601     74 0C               je short 00B7760F  
00B77603     8B4424 2C           mov eax,dword ptr ss:\[esp+2C\]  
00B77607     E8 50AFFEFF         call 00B6255C  
00B7760C     EB 01               jmp short 00B7760F  
00B7760F     BA 047AB700         mov edx,0B77A04  
00B77614     BB 886FB700         mov ebx,0B76F88  
00B77619     2BD3                sub edx,ebx  
00B7761B     83EA 04             sub edx,4  
00B7761E     8BC3                mov eax,ebx  
00B77620     E8 13F9FFFF         call 00B76F38  
00B77625     3B05 40E7B700       cmp eax,dword ptr ds:\[B7E740\]  
//不等就 Over 啦  ★  
00B7762B     74 12               je short 00B7763F  
//改标志位 Z=1 使这里跳转！  
00B7762D     A1 BCC5B700         mov eax,dword ptr ds:\[B7C5BC\]  
00B77632     C600 36             mov byte ptr ds:\[eax\],36  
00B77635     8B4424 2C           mov eax,dword ptr ss:\[esp+2C\]  
00B77639     0105 B8C4B700       add dword ptr ds:\[B7C4B8\],eax  
00B7763F     A1 5CE7B700         mov eax,dword ptr ds:\[B7E75C\]  
00B77644     E8 5FB4FEFF         call 00B62AA8  
00B77649     EB 01               jmp short 00B7764C  
  
取消以前的所有断点，开始第五步操作。  
  
  
—————————————————————————————————  
四、附注：看看 AL=3 的 00B76E44 处几个分支处理  
  
  
00B76E44     53                  push ebx  
00B76E45     56                  push esi  
00B76E46     8BF2                mov esi,edx  
00B76E48     8BD8                mov ebx,eax  
00B76E4A     33C0                xor eax,eax  
00B76E4C     8AC1                mov al,cl  
00B76E4E     83F8 05             cmp eax,5  
00B76E51     0F87 9D000000       ja 00B76EF4  
00B76E57     FF2485 5E6EB700     jmp dword ptr ds:\[eax\*4+B76E5E\]  
//根据 EAX 值进行分支处理，这个程序用了 2 个分支处理  
  
————————————————————————  
1、EAX=1     0B75200 是 GetProcAddres  
  
  
00B76E76     B8 0052B700         mov eax,0B75200  
//EAX=1 跳到这里  0B75200  
00B76E7B     8903                mov dword ptr ds:\[ebx\],eax  
00B76E7D     5E                  pop esi  
00B76E7E     5B                  pop ebx  
00B76E7F     C3                  retn  
  
00B75200     55                  push ebp  
00B75201     8BEC                mov ebp,esp  
00B75203     8B55 0C             mov edx,dword ptr ss:\[ebp+C\]  
00B75206     8B45 08             mov eax,dword ptr ss:\[ebp+8\]  
00B75209     8B0D 1CC5B700       mov ecx,dword ptr ds:\[B7C51C\]  
00B7520F     8B09                mov ecx,dword ptr ds:\[ecx\]  
00B75211     3BC8                cmp ecx,eax  
00B75213     75 09               jnz short 00B7521E  
00B75215     8B0495 68C4B700     mov eax,dword ptr ds:\[edx\*4+B7C468\]  
00B7521C     EB 07               jmp short 00B75225  
00B7521E     52                  push edx  
00B7521F     50                  push eax  
00B75220     E8 2F03FFFF         call 00B65554 ; jmp to kernel32.GetProcAddress  
//这里！ ★  
00B75225     5D                  pop ebp  
00B75226     C2 0800             retn 8  
  
————————————————————————  
2、EAX=5     0B76DE8 是 FindResourceA  
  
  
00B76EDD     A1 1CC6B700         mov eax,dword ptr ds:\[B7C61C\]  
//EAX=5 跳到这里  
00B76EE2     8B40 0C             mov eax,dword ptr ds:\[eax+C\]  
00B76EE5     A3 50E7B700         mov dword ptr ds:\[B7E750\],eax  
00B76EEA     B8 E86DB700         mov eax,0B76DE8  
//0B76DE8  
00B76EEF     8903                mov dword ptr ds:\[ebx\],eax  
00B76EF1     5E                  pop esi  
00B76EF2     5B                  pop ebx  
00B76EF3     C3                  retn  
  
00B76DE8     FF35 50E7B700       push dword ptr ds:\[B7E750\]  
00B76DEE     53                  push ebx  
00B76DEF     56                  push esi  
00B76DF0     57                  push edi  
00B76DF1     EB 01               jmp short 00B76DF4  
  
00B76DF4     89E3                mov ebx,esp  
00B76DF6     53                  push ebx  
00B76DF7     E8 10FEFFFF         call 00B76C0C  
00B76DFC     EB 01               jmp short 00B76DFF  
00B76DFF     5F                  pop edi  
00B76E00     5E                  pop esi  
00B76E01     5B                  pop ebx  
00B76E02     C3                  retn  
  
00B99F7C     6A 20               push 20  
00B99F7E     68 D814E677         push 77E614D8  
//FindResourceA 函数的前 2 条指令  
00B99F83     65:EB 01            jmp short 00B99F87  
00B99F87     68 96CAE477         push 77E4CA96  
00B99F8B     EB 01               jmp short 00B99F8E  
00B99F8E     68 86BBE777         push 77E7BB86  
00B99F93     64:A1 00000000      mov eax,dword ptr fs:\[0\]  
00B99F99     50                  push eax  
00B99F9A     64:8925 00000000    mov dword ptr fs:\[0\],esp  
00B99FA1     8B4424 10           mov eax,dword ptr ss:\[esp+10\]  
00B99FA5     896C24 10           mov dword ptr ss:\[esp+10\],ebp  
00B99FA9     8D6C24 10           lea ebp,dword ptr ss:\[esp+10\]  
00B99FAD     2BE0                sub esp,eax  
00B99FAF     53                  push ebx  
00B99FB0     56                  push esi  
00B99FB1     57                  push edi  
00B99FB2     8B45 F8             mov eax,dword ptr ss:\[ebp\-8\]  
00B99FB5     8965 E8             mov dword ptr ss:\[ebp\-18\],esp  
00B99FB8     50                  push eax  
00B99FB9     8B45 FC             mov eax,dword ptr ss:\[ebp\-4\]  
00B99FBC     C745 FC FFFFFFFF    mov dword ptr ss:\[ebp\-4\],-1  
00B99FC3     8945 F8             mov dword ptr ss:\[ebp\-8\],eax  
//把 CALL call kernel32.77E574E1 放在这里执行  
00B99FC6     F2:                 prefix repne:  
00B99FC7     EB 01               jmp short 00B99FCA  
  
00B99FCA     68 1975E577         push 77E57519  
00B99FCF     C3                  retn  
  
77E4CA8A     6A 20               push 20  
//FindResourceA   
77E4CA8C     68 D814E677         push kernel32.77E614D8  
77E4CA91     E8 4BAA0000         call kernel32.77E574E1  
77E4CA96     33F6                xor esi,esi  ; mTrace.00400000  
//返回这里  
  
————————————————————————  
3、未知  
  
  
00B76EA1     68 106FB700         push 0B76F10 ; ASCII "\_except\_handler2"  
00B76EA6     56                  push esi  
00B76EA7     A1 1CC6B700         mov eax,dword ptr ds:\[B7C61C\]  
00B76EAC     8B00                mov eax,dword ptr ds:\[eax\]  
00B76EAE     FFD0                call eax  
00B76EB0     A3 44E7B700         mov dword ptr ds:\[B7E744\],eax  
00B76EB5     B8 086EB700         mov eax,0B76E08  
00B76EBA     8903                mov dword ptr ds:\[ebx\],eax  
00B76EBC     5E                  pop esi  
00B76EBD     5B                  pop ebx  
00B76EBE     C3                  retn  
00B76EBF     68 246FB700         push 0B76F24 ; ASCII "\_except\_handler3"  
00B76EC4     56                  push esi  
00B76EC5     A1 1CC6B700         mov eax,dword ptr ds:\[B7C61C\]  
00B76ECA     8B00                mov eax,dword ptr ds:\[eax\]  
00B76ECC     FFD0                call eax  
00B76ECE     A3 48E7B700         mov dword ptr ds:\[B7E748\],eax  
00B76ED3     B8 246EB700         mov eax,0B76E24  
00B76ED8     8903                mov dword ptr ds:\[ebx\],eax  
00B76EDA     5E                  pop esi  
00B76EDB     5B                  pop ebx  
00B76EDC     C3                  retn  
  
  
—————————————————————————————————  
五、Pre-Dip：用自己的名字注册  
  
  
00B7AC61     3100                xor dword ptr ds:\[eax\],eax  
//异常，第 2 次看见硬盘指纹  
  
0012FF2C   0012FF34   指针到下一个 SEH 记录  
0012FF30   00B7AC18   SE 句柄  
0012FF34   0012FF40   指针到下一个 SEH 记录  
0012FF38   00B7B236   SE 句柄  
0012FF3C   0012FF90  
0012FF40   0012FFE0   指针到下一个 SEH 记录  
0012FF44   00B7B25B   SE 句柄  
0012FF48   0012FF90  
0012FF4C   00B60000  
0012FF50   00B30000  
0012FF54   00B7A684   ASCII "U 嬱岴鴭"  
0012FF58   00000000  
0012FF5C   00BEC1A4   ASCII "8wh3JAAQjOI="//硬盘指纹  
  
Alt+M 打开内存查看窗口，在 00401000 段下 内存访问断点，Shift+F9 通过异常，断下  
  
0052FE20     55                  push ebp  
//中断在这里  
0052FE21     8BEC                mov ebp,esp  
0052FE23     8B45 08             mov eax,dword ptr ss:\[ebp+8\]  
0052FE26     A3 E05D5400         mov dword ptr ds:\[545DE0\],eax  
//保存注册名地址  
0052FE2B     5D                  pop ebp  
0052FE2C     C2 0400             retn 4  
  
过了 0052FE26 之后，\[545DE0\]处应该是保存的注册用户名，找一段空地，在 00563880 处写入：fly  \[CUG\]，然后把\[545DE0\]处的值改为 00563880。OK，这个 Pre-Dip 处理完毕。  
  
  
—————————————————————————————————  
六、以壳解壳——ASProtect V1.3 壳的 Stolen Code 简便解决方案  
  
  
取消以前的所有断点。继续 Shift+F9 运行，直至中断在最后一次典型异常。  
  
00B79CB5     3100                xor dword ptr ds:\[eax\],eax  
//ASProtect V1.3B 最后 1 次典型异常，和 V1.23RC4 是一样的  
00B79CB7     64:8F05 00000000    pop dword ptr fs:\[0\]  
00B79CBE     58                  pop eax  
00B79CBF     833D C4E7B700 00    cmp dword ptr ds:\[B7E7C4\],0  
00B79CC6     74 14               je short 00B79CDC  
00B79CC8     6A 0C               push 0C  
00B79CCA     B9 C4E7B700         mov ecx,0B7E7C4  
00B79CCF     8D45 F8             lea eax,dword ptr ss:\[ebp\-8\]  
00B79CD2     BA 04000000         mov edx,4  
00B79CD7     E8 6075FFFF         call 00B7123C  
00B79CDC     FF75 FC             push dword ptr ss:\[ebp\-4\]  
00B79CDF     FF75 F8             push dword ptr ss:\[ebp\-8\]  
00B79CE2     8B45 F4             mov eax,dword ptr ss:\[ebp\-C\]  
00B79CE5     8338 00             cmp dword ptr ds:\[eax\],0  
00B79CE8     74 02               je short 00B79CEC  
00B79CEA     FF30                push dword ptr ds:\[eax\]  
00B79CEC     FF75 F0             push dword ptr ss:\[ebp\-10\]  
00B79CEF     FF75 EC             push dword ptr ss:\[ebp\-14\]  
00B79CF2     C3                  retn  
//此处下断，Shift+F9 断下  
  
看看堆栈：  
0012FF38   00BA0004  
0012FF3C   00400000   ASCII "MZP"  
0012FF40   59C79F56  
0012FF44   0012FFA4  
  
0012FF44＝0012FFA4，下断：HR 0012FF44。F9 运行，中断下来  
  
00BA00D1     EB 44               jmp short 00BA0117  
//中断在这  
  
00BA0117     03C3                add eax,ebx  
00BA0119     BB 08060000         mov ebx,608  
//从这里可以使用“以壳解壳”来处理 Stolen Code ★  
//在这儿用 PETools 完全 DUMP 这个进程  
00BA011E     0BDB                or ebx,ebx  
00BA0120     75 07               jnz short dumped\_.00BA0129  
00BA0122     894424 1C           mov dword ptr ss:\[esp+1C\],eax  
00BA0126     61                  popad  
00BA0127     50                  push eax  
00BA0128     C3                  retn  
00BA0129     E8 00000000         call 00BA012E  
00BA012E     5D                  pop ebp  
00BA012F     81ED D9B54B00       sub ebp,dumped\_.004BB5D9  
00BA0135     8D85 7EB54B00       lea eax,dword ptr ss:\[ebp+4BB57E\]  
00BA013B     8D8D 20B64B00       lea ecx,dword ptr ss:\[ebp+4BB620\]  
00BA0141     03CB                add ecx,ebx  
00BA0143     8941 01             mov dword ptr ds:\[ecx+1\],eax  
00BA0146     8D85 C2B54B00       lea eax,dword ptr ss:\[ebp+4BB5C2\]  
00BA014C     8D8D 86B54B00       lea ecx,dword ptr ss:\[ebp+4BB586\]  
00BA0152     8901                mov dword ptr ds:\[ecx\],eax  
00BA0154     B8 5E280000         mov eax,285E  
00BA0159     8D8D 8BB54B00       lea ecx,dword ptr ss:\[ebp+4BB58B\]  
00BA015F     8901                mov dword ptr ds:\[ecx\],eax  
00BA0161     8D8D 20B64B00       lea ecx,dword ptr ss:\[ebp+4BB620\]  
00BA0167     8D85 20DB4B00       lea eax,dword ptr ss:\[ebp+4BDB20\]  
00BA016D     51                  push ecx  
00BA016E     50                  push eax  
00BA016F     E8 76FFFFFF         call 00BA00EA  
00BA0174     61                  popad  
00BA0175     EB 01               jmp short 00BA0178  
  
  
—————————————————————————————————  
七、Dump、修复输入表  
  
  
仅仅区域脱壳出上面的处理 Stolen Code 壳代码部分是不行的。  
为了修复 CALL EAX 和 FindResourceA 的问题，所以共区域脱壳出以下部分壳代码：  
00B60000-00B85000.dmp  
00B90000-00B94000.dmp  
00B98000-00B9C000.dmp  
00BA0000-00BA4000.dmp  
00BC0000-00BF0000.dmp  
“组装”一下 dumped.exe，从磁盘载入刚才部分脱壳的五部分。  
减去基址后分别修改其虚拟地址为 00760000、00790000、00798000、009A0000、009C0000  
只保留 PETools 的“验证 PE 文件”选项，重建 PE。  
  
  
运行 ImportREC，填入 RVA=00163000、Size=0000083C  
获取输入表，会发现有 2 个无效指针  
00163078  ?  0000  00B75200//GetProcAddress  
0016319C  ?  0000  00B76DE8//FindResourceA  
001631E0  ?  0000  00B75200//GetProcAddress  
通过 AL=3 的分析，我们知道应该把 00B75200 修复为 GetProcAddress，把 00B76DE8 修复为 FindResourceA  
修改 OEP=007A0000，FixDump！  
  
FindResourceA 这个部分有问题，借用壳代码来处理吧。  
00407478     FF25 9C315600       jmp dword ptr ds:\[56319C\]; FindResourceA  
把 OEP 处修改为：  
00BA0000     C705 9C315600 E86DB>mov dword ptr ds:\[56319C\],00B76DE8  
//用原来的壳代码来处理  
00BA000A     E9 0A010000         jmp 00BA0119  
  
  
—————————————————————————————————  
八、暗桩修复  
  
  
0040218F     803D 30745400 00    cmp byte ptr ds:\[547430\],0  
//把 54E430 处的 01 修改为 00，  ★    让其重新申请内存  
00402196     75 13               jnz short dumped\_.004021AB  
00402198     E8 13F9FFFF         call dumped\_.00401AB0  
0040219D     84C0                test al,al  
0040219F     75 0A               jnz short dumped\_.004021AB  
004021A1     33C0                xor eax,eax  
004021A3     8945 FC             mov dword ptr ss:\[ebp\-4\],eax  
004021A6     E9 54010000         jmp dumped\_.004022FF  
004021AB     33C9                xor ecx,ecx  
004021AD     55                  push ebp  
004021AE     68 F8224000         push dumped\_.004022F8  
004021B3     64:FF31             push dword ptr fs:\[ecx\]  
004021B6     64:8921             mov dword ptr fs:\[ecx\],esp  
004021B9     803D 49705400 00    cmp byte ptr ds:\[547049\],0  
004021C0     74 0A               je short dumped\_.004021CC  
004021C2     68 38745400         push dumped\_.00547438  
004021C7     E8 40F2FFFF         call <jmp.&kernel32.EnterCriticalSection>  
004021CC     83C3 07             add ebx,7  
004021CF     83E3 FC             and ebx,FFFFFFFC  
004021D2     83FB 0C             cmp ebx,0C  
004021D5     7D 05               jge short dumped\_.004021DC  
004021D7     BB 0C000000         mov ebx,0C  
004021DC     81FB 00100000       cmp ebx,1000  
004021E2     0F8F 93000000       jg dumped\_.0040227B  
004021E8     8BC3                mov eax,ebx  
004021EA     85C0                test eax,eax  
004021EC     79 03               jns short dumped\_.004021F1  
004021EE     83C0 03             add eax,3  
004021F1     C1F8 02             sar eax,2  
004021F4     8B15 90745400       mov edx,dword ptr ds:\[547490\]  
004021FA     8B5482 F4           mov edx,dword ptr ds:\[edx+eax\*4-C\]  
//异常 ①  
004021FE     85D2                test edx,edx  
00402200     74 79               je short dumped\_.0040227B  
  
  
00401426     BE 54745400         mov esi,dumped\_.00547454  
0040142B     833E 00             cmp dword ptr ds:\[esi\],0  
//把\[00547454\]处的 0014EDA4 修改为 00000000  ★  
0040142E     75 3A               jnz short dumped\_.0040146A  
00401430     68 44060000         push 644  
00401435     6A 00               push 0  
00401437     E8 A8FFFFFF         call <jmp.&kernel32.LocalAlloc>  
0040143C     8BC8                mov ecx,eax  
0040143E     85C9                test ecx,ecx  
00401440     75 05               jnz short dumped\_.00401447  
00401442     33C0                xor eax,eax  
00401444     5E                  pop esi  
00401445     5B                  pop ebx  
00401446     C3                  retn  
00401447     A1 50745400         mov eax,dword ptr ds:\[547450\]  
0040144C     8901                mov dword ptr ds:\[ecx\],eax  
0040144E     890D 50745400       mov dword ptr ds:\[547450\],ecx  
00401454     33D2                xor edx,edx  
00401456     8BC2                mov eax,edx  
00401458     03C0                add eax,eax  
0040145A     8D44C1 04           lea eax,dword ptr ds:\[ecx+eax\*8+4\]  
0040145E     8B1E                mov ebx,dword ptr ds:\[esi\]  
00401460     8918                mov dword ptr ds:\[eax\],ebx  
00401462     8906                mov dword ptr ds:\[esi\],eax  
00401464     42                  inc edx  
00401465     83FA 64             cmp edx,64  
00401468     75 EC               jnz short dumped\_.00401456  
0040146A     8B06                mov eax,dword ptr ds:\[esi\]  
0040146C     8B10                mov edx,dword ptr ds:\[eax\]  
//异常 ②  
0040146E     8916                mov dword ptr ds:\[esi\],edx  
00401470     5E                  pop esi  
00401471     5B                  pop ebx  
00401472     C3                  retn  
  
  
—————————————————————————————————  
         ,     \_/   
        /| \_.-~/            \\\_     ,        青春都一晌  
       ( /~   /              \\~-.\_ |\\  
       \`\\\\  \_/                \\   ~\\ )          忍把浮名   
   \_-~~~-.)  )\_\_/;;,.          \\\_  //'  
  /'\_,\\   --~   \\ ~~~-  ,;;\\\_\_\_(  (.-~~~-.        换了破解轻狂  
 \`~ \_( ,\_..--\\ (     ,;'' /    ~--   /.\_\`\\   
  /~~//'   /' \`~\\         ) /--..\_, )\_  \`~  
  "  \`~"  "      \`"      /~'\`\\    \`\\\\~~\\     
                         "     "   "~'  ""  
     
              UnPacked By :  fly  
               2004-10-07 02:00

  
   4dd466d0e1e0cc123afbe504da2462ca