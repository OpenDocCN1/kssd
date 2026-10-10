

# Ultra Protect 脱壳+暗桩解除——股市风暴 V6.0

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-551.htm`](https://www.pediy.com/kssd/pediy07/pediy7-551.htm)

Ultra Protect 脱壳+暗桩解除——股市风暴 V6.0 Build 156  
   
   
   
下载地址：  http://www.jfe99.com/day/wlsetup.exe   
软件大小：  4.05M  
运行平台：  Windows9X/ME/NT/2000/XP  
更新日期：  2005-04-17  
软件简介：  股票软件  
  
【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教  
  
【调试环境】：WinXP、flyODBG、PEiD、LordPE  
  
—————————————————————————————————   
【脱壳过程】：  
            
           
设置 Ollydbg 忽略所有的异常选项。用 IsDebug 插件去掉 Ollydbg 的调试器标志。  
—————————————————————————————————  
一、壳中壳的 IAT  
  
  
00720000    60              pushad  
00720001    50              push eax  
00720002    E8 01000000     call 00720008  
//进入 Ollydbg 后暂停在这  
00720007    79 83           jns short 0071FF8C  
  
下断：HE GetModuleHandleA     
Shift+F9 运行，中断后取消断点。Alt+F9 返回  
  
00732622    FF95 20854100   call dword ptr ss:\[ebp+418520\]  
00732628    EB 17           jmp short 00732641  
//返回这里  
  
下面这段是处理壳所使用的输入表函数，因为 EMbedded Protector 要使用壳代码，所以我们要保留这些函数  
Ctrl+F 在当前位置下搜索命令：rep stos byte ptr es:\[edi\]  
找到在 007328C8 处，直接 F4 至 007328C8  
  
007328B8    8DBD 83FB4000   lea edi,dword ptr ss:\[ebp+40FB83\]  
007328BE    8D8D F1FC4000   lea ecx,dword ptr ss:\[ebp+40FCF1\]  
007328C4    2BCF            sub ecx,edi  
007328C6    33C0            xor eax,eax  
007328C8    F3:AA           rep stos byte ptr es:\[edi\]  
//清除 DLL、函数名            不让壳清除，NOP 掉 ★  
007328CA    C3              retn  
  
00735306    E8 F0D2FFFF     call 007325FB  
0073530B    E8 38F9FFFF     call 00734C48  
//返回这里  
  
记住 call 007325FB 是处理壳输入表的地方  
  
  
—————————————————————————————————  
二、搞定输入表  
  
  
继续下断：HE GetModuleHandleA     
Shift+F9 运行，中断后取消断点。Alt+F9 返回  
  
00734473    8B95 46F84000   mov edx,dword ptr ss:\[ebp+40F846\]  
00734479    8BB5 07F94000   mov esi,dword ptr ss:\[ebp+40F907\]  
0073447F    03F2            add esi,edx  
00734481    8B46 0C         mov eax,dword ptr ds:\[esi+C\]  
00734484    0BC0            or eax,eax  
00734486    0F84 25020000   je 007346B1  
0073448C    8366 0C 00      and dword ptr ds:\[esi+C\],0  
//这里清空 ImageImportDescriptor 的 Name！     NOP 掉 ①  ★  
当我们中断后返回 007344AF 时，这里已经运行过一次了，清除了第一个 Name 指针。  
可以根据当时的寄存器情况来恢复这个指针。在 007344AF 时 ESI＝0062B000，\[esi+C\]＝\[0062B00C\]=00 00，而 EBX＝0062BB34 ASCII "KERNEL32.dll" 是第一个处理的 DLL 名，所以可以确定\[0062B00C\]=0022BB34，修改之，否则 DLLName 会有错误 ★  
  
00734490    03C2            add eax,edx  
00734492    8BD8            mov ebx,eax  
00734494    56              push esi  
00734495    57              push edi  
00734496    50              push eax  
00734497    8BF3            mov esi,ebx  
00734499    8BFB            mov edi,ebx  
0073449B    AC              lods byte ptr ds:\[esi\]  
0073449C    C0C0 03         rol al,3  
//解码出 DLL 名 ★  
0073449F    AA              stos byte ptr es:\[edi\]  
007344A0    803F 00         cmp byte ptr ds:\[edi\],0  
007344A3    75 F6           jnz short 0073449B  
007344A5    58              pop eax  
007344A6    5F              pop edi  
007344A7    5E              pop esi  
007344A8    50              push eax  
007344A9    FF95 20854100   call dword ptr ss:\[ebp+418520\]  
007344AF    0BC0            or eax,eax  
//返回这里  
//此时 ESI＝0062B000-00400000＝0022B000   输入表的 RVA  ★  
007344B1    75 43           jnz short 007344F6  
007344B3    90              nop  
007344B4    90              nop  
007344B5    90              nop  
007344B6    90              nop  
007344B7    53              push ebx  
007344B8    FF95 24854100   call dword ptr ss:\[ebp+418524\]  
007344BE    0BC0            or eax,eax  
007344C0    75 34           jnz short 007344F6  
007344C2    90              nop  
007344C3    90              nop  
007344C4    90              nop  
007344C5    90              nop  
007344C6    8B95 46F84000   mov edx,dword ptr ss:\[ebp+40F846\]  
007344CC    0195 351B4000   add dword ptr ss:\[ebp+401B35\],edx  
007344D2    0195 391B4000   add dword ptr ss:\[ebp+401B39\],edx  
007344D8    6A 00           push 0  
007344DA    FFB5 351B4000   push dword ptr ss:\[ebp+401B35\]  
007344E0    FFB5 391B4000   push dword ptr ss:\[ebp+401B39\]  
007344E6    6A 00           push 0  
007344E8    FF95 2C854100   call dword ptr ss:\[ebp+41852C\]  
007344EE    6A 00           push 0  
007344F0    FF95 28854100   call dword ptr ss:\[ebp+418528\]  
007344F6    60              pushad  
007344F7    2BC0            sub eax,eax  
007344F9    8803            mov byte ptr ds:\[ebx\],al  
//用完之后清空 DLL 名   NOP 掉 ②！ ★  
007344FB    43              inc ebx  
007344FC    3803            cmp byte ptr ds:\[ebx\],al  
007344FE    75 F9           jnz short 007344F9  
00734500    61              popad  
00734501    8985 3EF84000   mov dword ptr ss:\[ebp+40F83E\],eax  
//保存 DLL 基址  
00734507    C785 42F84000 0>mov dword ptr ss:\[ebp+40F842\],0  
00734511    8B95 46F84000   mov edx,dword ptr ss:\[ebp+40F846\]  
00734517    8B06            mov eax,dword ptr ds:\[esi\]  
00734519    0BC0            or eax,eax  
0073451B    75 07           jnz short 00734524  
0073451D    90              nop  
0073451E    90              nop  
0073451F    90              nop  
00734520    90              nop  
00734521    8B46 10         mov eax,dword ptr ds:\[esi+10\]  
00734524    03C2            add eax,edx  
00734526    0385 42F84000   add eax,dword ptr ss:\[ebp+40F842\]  
0073452C    8B18            mov ebx,dword ptr ds:\[eax\]  
0073452E    8B7E 10         mov edi,dword ptr ds:\[esi+10\]  
00734531    03FA            add edi,edx  
00734533    03BD 42F84000   add edi,dword ptr ss:\[ebp+40F842\]  
00734539    85DB            test ebx,ebx  
0073453B    0F84 62010000   je 007346A3  
00734541    F7C3 00000080   test ebx,80000000  
00734547    75 1D           jnz short 00734566  
00734549    90              nop  
0073454A    90              nop  
0073454B    90              nop  
0073454C    90              nop  
0073454D    03DA            add ebx,edx  
0073454F    83C3 02         add ebx,2  
00734552    56              push esi  
00734553    57              push edi  
00734554    50              push eax  
00734555    8BF3            mov esi,ebx  
00734557    8BFB            mov edi,ebx  
00734559    AC              lods byte ptr ds:\[esi\]  
0073455A    C0C0 03         rol al,3  
//解码出函数名 ★  
0073455D    AA              stos byte ptr es:\[edi\]  
0073455E    803F 00         cmp byte ptr ds:\[edi\],0  
00734561    75 F6           jnz short 00734559  
00734563    58              pop eax  
00734564    5F              pop edi  
00734565    5E              pop esi  
00734566    3B9D 46F84000   cmp ebx,dword ptr ss:\[ebp+40F846\]  
0073456C    7C 11           jl short 0073457F  
0073456E    90              nop  
0073456F    90              nop  
00734570    90              nop  
00734571    90              nop  
00734572    83BD 1A204000 0>cmp dword ptr ss:\[ebp+40201A\],0  
00734579    75 0A           jnz short 00734585  
0073457B    90              nop  
0073457C    90              nop  
0073457D    90              nop  
0073457E    90              nop  
0073457F    81E3 FFFFFF0F   and ebx,0FFFFFFF  
00734585    53              push ebx  
00734586    FFB5 3EF84000   push dword ptr ss:\[ebp+40F83E\]  
0073458C    FF95 1C854100   call dword ptr ss:\[ebp+41851C\]  
00734592    3B9D 46F84000   cmp ebx,dword ptr ss:\[ebp+40F846\]  
00734598    7C 0F           jl short 007345A9  
0073459A    90              nop  
0073459B    90              nop  
0073459C    90              nop  
0073459D    90              nop  
0073459E    60              pushad  
0073459F    2BC0            sub eax,eax  
007345A1    8803            mov byte ptr ds:\[ebx\],al  
//用完之后清空函数名   NOP 掉 ③！ ★  
007345A3    43              inc ebx  
007345A4    3803            cmp byte ptr ds:\[ebx\],al  
007345A6    75 F9           jnz short 007345A1  
007345A8    61              popad  
007345A9    0BC0            or eax,eax  
007345AB    0F84 15FFFFFF   je 007344C6  
007345B1    3B85 2C854100   cmp eax,dword ptr ss:\[ebp+41852C\]  
//是否是 MessageBoxA ？EMbedded Protector 专用 APT 接口  
007345B7    74 20           je short 007345D9  
007345B9    90              nop  
007345BA    90              nop  
007345BB    90              nop  
007345BC    90              nop  
007345BD    3B85 C4FD4000   cmp eax,dword ptr ss:\[ebp+40FDC4\]  
//是否是 RegisterHotKey ？  
007345C3    74 09           je short 007345CE  
007345C5    90              nop  
007345C6    90              nop  
007345C7    90              nop  
007345C8    90              nop  
007345C9    EB 14           jmp short 007345DF  
007345CB    90              nop  
007345CC    90              nop  
007345CD    90              nop  
007345CE    8D85 31FE4000   lea eax,dword ptr ss:\[ebp+40FE31\]  
007345D4    EB 09           jmp short 007345DF  
007345D6    90              nop  
007345D7    90              nop  
007345D8    90              nop  
007345D9    8D85 4BFE4000   lea eax,dword ptr ss:\[ebp+40FE4B\]  
007345DF    56              push esi  
007345E0    FFB5 3EF84000   push dword ptr ss:\[ebp+40F83E\]  
007345E6    5E              pop esi  
007345E7    39B5 12204000   cmp dword ptr ss:\[ebp+402012\],esi  
//比较是否是 Kernel32.DLL 基址  
007345ED    74 15           je short 00734604  
007345EF    90              nop  
007345F0    90              nop  
007345F1    90              nop  
007345F2    90              nop  
007345F3    39B5 16204000   cmp dword ptr ss:\[ebp+402016\],esi  
//比较是否是 User32.DLL 基址  
007345F9    74 09           je short 00734604  
007345FB    90              nop  
007345FC    90              nop  
007345FD    90              nop  
007345FE    90              nop  
007345FF    EB 63           jmp short 00734664  
00734601    90              nop  
00734602    90              nop  
00734603    90              nop  
00734604    80BD 16564100 0>cmp byte ptr ss:\[ebp+415616\],0  
0073460B    74 57           je short 00734664  
//Magic Jump！ 如果用 ImportREC 修复输入表，则可以修改这里为：jmp 00734664  
0073460D    90              nop  
0073460E    90              nop  
0073460F    90              nop  
00734610    90              nop  
00734611    EB 07           jmp short 0073461A  
//下面就是加密了  
00734613    90              nop  
00734614    90              nop  
00734615    90              nop  
00734616    0100            add dword ptr ds:\[eax\],eax  
00734618    0000            add byte ptr ds:\[eax\],al  
0073461A    8BB5 0BF94000   mov esi,dword ptr ss:\[ebp+40F90B\]  
00734620    83C6 0D         add esi,0D  
00734623    81EE 02184000   sub esi,stocksto.00401802  
00734629    2BF5            sub esi,ebp  
0073462B    83FE 00         cmp esi,0  
0073462E    7F 34           jg short 00734664  
00734630    90              nop  
00734631    90              nop  
00734632    90              nop  
00734633    90              nop  
00734634    8BB5 0BF94000   mov esi,dword ptr ss:\[ebp+40F90B\]  
0073463A    53              push ebx  
0073463B    50              push eax  
0073463C    E8 8DB2FFFF     call 0072F8CE  
00734641    8BD8            mov ebx,eax  
00734643    58              pop eax  
00734644    33C3            xor eax,ebx  
//加密只是简单异或  
00734646    C606 68         mov byte ptr ds:\[esi\],68  
00734649    8946 01         mov dword ptr ds:\[esi+1\],eax  
0073464C    C746 05 8134240>mov dword ptr ds:\[esi+5\],243481  
00734653    895E 08         mov dword ptr ds:\[esi+8\],ebx  
00734656    C646 0C C3      mov byte ptr ds:\[esi+C\],0C3  
0073465A    5B              pop ebx  
0073465B    8BC6            mov eax,esi  
0073465D    8385 0BF94000 0>add dword ptr ss:\[ebp+40F90B\],0D  
00734664    5E              pop esi  
00734665    60              pushad  
00734666    8BD0            mov edx,eax  
00734668    2BBD 46F84000   sub edi,dword ptr ss:\[ebp+40F846\]  
0073466E    8BC7            mov eax,edi  
00734670    B9 01010000     mov ecx,101  
00734675    8DBD EBEC4000   lea edi,dword ptr ss:\[ebp+40ECEB\]  
0073467B    F2:AF           repne scas dword ptr es:\[edi\]  
0073467D    0BC9            or ecx,ecx  
0073467F    74 13           je short 00734694  
00734681    90              nop  
00734682    90              nop  
00734683    90              nop  
00734684    90              nop  
00734685    81E9 01010000   sub ecx,101  
0073468B    F7D1            not ecx  
0073468D    89948D EBE84000 mov dword ptr ss:\[ebp+ecx\*4+40E8EB\],edx  
00734694    61              popad  
00734695    8907            mov dword ptr ds:\[edi\],eax  
//API 函数的系统地址（或者加密地址）填充到 IAT 中    NOP 掉 ④！★  
00734697    8385 42F84000 0>add dword ptr ss:\[ebp+40F842\],4  
0073469E    E9 6EFEFFFF     jmp 00734511  
007346A3    83C6 14         add esi,14  
007346A6    8B95 46F84000   mov edx,dword ptr ss:\[ebp+40F846\]  
007346AC    E9 D0FDFFFF     jmp 00734481  
//循环处理  
007346B1    8DBD EBEC4000   lea edi,dword ptr ss:\[ebp+40ECEB\]  
//修改上面 4 处后直接 F4 到这里   输入表处理完毕了  
007346B7    33C0            xor eax,eax  
007346B9    B9 00010000     mov ecx,100  
007346BE    F3:AB           rep stos dword ptr es:\[edi\]  
007346C0    60              pushad  
007346C1    E8 00000000     call 007346C6  
  
  
—————————————————————————————————  
三、第 2 区段内存断点法，飞向光明之巅  
  
  
Alt+M 打开内存查看窗口，在 401000 第二区段上设置内存访问断点，Shift+F9 运行  
  
0061A654    55              push ebp  
//直接中断在 OEP ★  运行 LordPE 完全 Dump 出这个进程  
0061A655    8BEC            mov ebp,esp  
0061A657    83C4 F0         add esp,-10  
0061A65A    53              push ebx  
0061A65B    B8 5C9D6100     mov eax,00619D5C  
0061A660    E8 C3CEDEFF     call 00407528  
0061A665    8B1D 70096200   mov ebx,dword ptr ds:\[620970\]  
0061A66B    8B03            mov eax,dword ptr ds:\[ebx\]  
0061A66D    E8 BA28E7FF     call 0048CF2C  
  
用 LordPE 修正 dumped.exe 的 OEP RVA=0021A654，Import Table RVA=0022B000  
  
  
—————————————————————————————————  
四、EMbedded Protector + 修复  
  
  
目前脱壳后的程序还是无法运行的，程序使用了 Ultra Protect 的 EMbedded Protector 技术，需要修复  
  
0059D650      53            push ebx  
0059D651      8BD8          mov ebx,eax  
0059D653      60            pushad  
0059D654      6A 05         push 5  
0059D656      6A 00         push 0  
0059D658      6A 00         push 0  
0059D65A      6A FF         push -1  
0059D65C    E8 27ADE6FF     call 00408388 ; <jmp.&user32.MessageBoxA>  
//这里的调用其实是 EMbedded Protector 接口  
0059D661      61            popad  
  
00408388    FF25 B4B76200   jmp dword ptr ds:\[62B7B4\] ; user32.MessageBoxA  
00408388    FF25 B4B76200   jmp dword ptr ds:\[62B7B4\] ; stocksto.0072EE4B  
//跟踪原程序发现这里是：072EE4B  
  
在 EMbedded Protector 中会检测壳的一系列特征：  
  
0059EFC6    3B85 50164000   cmp eax,dword ptr ss:\[ebp+401650\]  
0059EFCC    74 7B           je short 0059F049  
0059EFCE    90              nop  
0059EFCF    90              nop  
0059EFD0    90              nop  
0059EFD1    90              nop  
0059EFD2    3B85 54164000   cmp eax,dword ptr ss:\[ebp+401654\]  
0059EFD8    75 6F           jnz short 0059F049  
//检测壳原来的 EP  
0059EFDE    0FB747 06       movzx eax,word ptr ds:\[edi+6\]  
0059EFE2    48              dec eax  
0059EFE3    3D 08000000     cmp eax,8  
//检测区段数  
0059EFE8    75 5F           jnz short 0059F049   
  
0059F002    3B9D 54164000   cmp ebx,dword ptr ss:\[ebp+401654\]  
//检测壳原来的 EP  
0059F008    75 3F           jnz short 0059F049  
0059F00F    813E 2E706572   cmp dword ptr ds:\[esi\],7265702E  
0059F015    75 32           jnz short 0059F049  
0059F017    90              nop  
0059F018    90              nop  
0059F019    90              nop  
0059F01A    90              nop  
0059F01B    817E 04 706C657>cmp dword ptr ds:\[esi+4\],78656C70  
0059F022    75 25           jnz short 0059F049  
//检验有无.perplex 壳区段  
0059F028    8B85 54164000   mov eax,dword ptr ss:\[ebp+401654\]  
0059F02E    8BBD 28164000   mov edi,dword ptr ss:\[ebp+401628\]  
0059F034    0FB61C07        movzx ebx,byte ptr ds:\[edi+eax\]  
0059F038    80EB 30         sub bl,30  
0059F03B    80FB 30         cmp bl,30  
//检验壳 EP 处的第一个字节  
0059F03E    75 09           jnz short 0059F049  
  
  
—————————————————————————————————  
五、壳中初始化数据修复  
  
  
0072F792    8BB48D 3D1B4000 mov esi,dword ptr ss:\[ebp+ecx\*4+401B3D\]  
0072F799    03B5 46F84000   add esi,dword ptr ss:\[ebp+40F846\]  
0072F79F    8B948D CD1C4000 mov edx,dword ptr ss:\[ebp+ecx\*4+401CCD\]  
0072F7A6    8BBC8D 5D1E4000 mov edi,dword ptr ss:\[ebp+ecx\*4+401E5D\]  
0072F7AD    87CA            xchg edx,ecx  
0072F7AF    F3:A4           rep movs byte ptr es:\[edi\],byte ptr ds:\[esi\]  
//这里异常  \[edi\]=\[00155668\]  
  
在数据窗口中 Ctrl+G：ebp+edx\*4+401E5D  
\[00720E5D\]=00155668  
这里是壳中初始化了，重新运行原程序，在 00720E5D 处设置硬件写入断点  
Shift+F9 几次后中断在 0073509A 处  
  
0073506A    60              pushad  
0073506B    E8 90B0FFFF     call 00730100  
00735070    33D2            xor edx,edx  
00735072    8BB495 3D1B4000 mov esi,dword ptr ss:\[ebp+edx\*4+401B3D\]  
00735079    0BF6            or esi,esi  
0073507B    74 31           je short 007350AE  
0073507D    90              nop  
0073507E    90              nop  
0073507F    90              nop  
00735080    90              nop  
00735081    03B5 46F84000   add esi,dword ptr ss:\[ebp+40F846\]  
00735087    8B8C95 CD1C4000 mov ecx,dword ptr ss:\[ebp+edx\*4+401CCD\]  
0073508E    60              pushad  
0073508F    52              push edx  
00735090    51              push ecx  
00735091    6A 40           push 40  
00735093    FF95 FFFC4000   call dword ptr ss:\[ebp+40FCFF\]; kernel32.GlobalAlloc  
00735099    5A              pop edx  
0073509A    898495 5D1E4000 mov dword ptr ss:\[ebp+edx\*4+401E5D\],eax  
//写入申请的内存地址值  
007350A1    61              popad  
007350A2    8BBC95 5D1E4000 mov edi,dword ptr ss:\[ebp+edx\*4+401E5D\]  
007350A9    F3:A4           rep movs byte ptr es:\[edi\],byte ptr ds:\[esi\]  
007350AB    42              inc edx  
007350AC    EB C4           jmp short 00735072  
007350AE    61              popad  
007350AF    C3              retn  
  
我们可以直接调用 Call 0073506A 来重新初始化这里  
             
  
—————————————————————————————————  
六、OEP 处修复代码  
  
  
把 OEP RVA 改为 00320000,最终修复代码如下  
  
00720000    60              pushad  
//保留壳 EP 原来的第一个字节  
00720001    A1 3CB26200     mov eax,dword ptr ds:\[62B23C\]; kernel32.GetModuleHandleA  
00720006    A3 20757300     mov dword ptr ds:\[737520\],eax  
//\[737520\]->GetModuleHandleA  
0072000B    A1 38B26200     mov eax,dword ptr ds:\[62B238\]; kernel32.GetProcAddress  
00720010    A3 1C757300     mov dword ptr ds:\[73751C\],eax  
//\[73751C\]->GetProcAddress  
00720015    E8 E1250100     call 007325FB   
//重新获取壳中壳 IAT  
0072001A    E8 4B500100     call 0073506A   
//重新初始化  
0072001F    C705 B4B76200 4>mov dword ptr ds:\[62B7B4\],72EE4B  
//EMbedded Protector 地址填充\[<&user32.MessageBoxA>\]处  
00720029    E9 26A6EFFF     jmp 0061A654  
//回到原来的 OEP 处执行  
  
现在可以正常运行了，脱壳完成。  
  
  
—————————————————————————————————  
七、检测版本升级  
  
  
00607155    B8 78726000     mov eax,00607278          ; ASCII "stockstorm.exe"  
0060715A    E8 D1FDFFFF     call 00606F30  
0060715F    84C0            test al,al  
00607161    74 71           je short 006071D4  
00607163    6A 00           push 0  
00607165    8D45 E4         lea eax,dword ptr ss:\[ebp-1C\]  
00607168    50              push eax  
00607169    B9 90726000     mov ecx,00607290          ; ASCII "datetime"  
0060716E    BA 78726000     mov edx,00607278          ; ASCII "stockstorm.exe"  
00607173    8BC3            mov eax,ebx  
00607175    8B18            mov ebx,dword ptr ds:\[eax\]  
00607177    FF13            call dword ptr ds:\[ebx\]  
00607179    8B45 E4         mov eax,dword ptr ss:\[ebp-1C\]  
0060717C    E8 2F6BE0FF     call 0040DCB0  
00607181    DC5D F0         fcomp qword ptr ss:\[ebp-10\]  
00607184    DFE0            fstsw ax  
00607186    9E              sahf  
00607187    76 4B           jbe short 006071D4  
//检测是否有新版本升级，不升级可以 jmp 006071D4  
  
  
—————————————————————————————————  
八、解决碰到黑名单上的程序则重启电脑的暗桩  
  
  
程序检测 trw2000、RegMon、FileMon、SoftICE 等黑名单程序，有则重启电脑  
  
004F41F1    E8 1236F1FF     call <jmp.&kernel32.CreateFileA>  
004F41F6    8BF8            mov edi,eax  
004F41F8    83FF FF         cmp edi,-1  
004F41FB    0F84 84000000   je 004F4285  
//修改为：jmp 004F4292  
004F4201    8D55 CC         lea edx,dword ptr ss:\[ebp-34\]  
004F4204    B8 5C454F00     mov eax,004F455C  
004F4209    E8 0AF6FFFF     call 004F3818  
004F420E    FF75 CC         push dword ptr ss:\[ebp-34\]  
004F4211    68 00464F00     push 004F4600  
004F4216    8D55 C8         lea edx,dword ptr ss:\[ebp-38\]  
004F4219    8BC6            mov eax,esi  
004F421B    E8 945FF1FF     call 0040A1B4  
004F4220    FF75 C8         push dword ptr ss:\[ebp-38\]  
004F4223    8D45 D0         lea eax,dword ptr ss:\[ebp-30\]  
004F4226    BA 03000000     mov edx,3  
004F422B    E8 6C10F1FF     call 0040529C  
004F4230    8B45 D0         mov eax,dword ptr ss:\[ebp-30\]  
004F4233    E8 C460F4FF     call 0043A2FC  
004F4238    57              push edi  
004F4239    E8 A235F1FF     call <jmp.&kernel32.CloseHandle>  
004F423E    E8 A983F1FF     call 0040C5EC  
004F4243    83C4 F8         add esp,-8  
004F4246    DD1C24          fstp qword ptr ss:\[esp\]  
004F4249    9B              wait  
004F424A    8D45 F8         lea eax,dword ptr ss:\[ebp-8\]  
004F424D    50              push eax  
004F424E    8D4D FA         lea ecx,dword ptr ss:\[ebp-6\]  
004F4251    8D55 FC         lea edx,dword ptr ss:\[ebp-4\]  
004F4254    8D45 FE         lea eax,dword ptr ss:\[ebp-2\]  
004F4257    E8 907FF1FF     call 0040C1EC  
004F425C    66:817D F8 4A01 cmp word ptr ss:\[ebp-8\],14A  
004F4262    73 0B           jnb short 004F426F  
004F4264    6A 00           push 0  
004F4266    6A 02           push 2  
004F4268    E8 5B3EF1FF     call <jmp.&user32.ExitWindowsEx>  
//Game Over  
004F426D    EB 16           jmp short 004F4285  
004F426F    66:817D F8 9E02 cmp word ptr ss:\[ebp-8\],29E  
004F4275    73 02           jnb short 004F4279  
004F4277    EB FE           jmp short 004F4277  
004F4279    A1 70096200     mov eax,dword ptr ds:\[620970\]  
004F427E    8B00            mov eax,dword ptr ds:\[eax\]  
004F4280    E8 2B8EF9FF     call 0048D0B0  
004F4285    46              inc esi  
004F4286    83C3 04         add ebx,4  
004F4289    83FE 08         cmp esi,8  
004F428C    0F85 3AFFFFFF   jnz 004F41CC  
//循环  
004F4292    8D45 C4         lea eax,dword ptr ss:\[ebp-3C\]  
004F4295    E8 62040000     call 004F46FC  
  
  
—————————————————————————————————  
九、解决删除本目录下其他 exe 文件的暗桩  
  
  
006131E0    BA 78356100     mov edx,00613578 ; ASCII "\*.\*"  
006131E5    E8 FA1FDFFF     call 004051E4  
006131EA    8B85 9CFEFFFF   mov eax,dword ptr ss:\[ebp-164\]  
006131F0    8D8D A0FEFFFF   lea ecx,dword ptr ss:\[ebp-160\]  
006131F6    BA 3F000000     mov edx,3F  
006131FB    E8 7876DFFF     call 0040A878  
00613200    85C0            test eax,eax  
00613202    0F85 4C020000   jnz 00613454  
//修改为 jmp 00613454 即可解决这个暗桩  ★  
00613208    8D95 94FEFFFF   lea edx,dword ptr ss:\[ebp-16C\]  
0061320E    8B85 ACFEFFFF   mov eax,dword ptr ss:\[ebp-154\]  
00613214    E8 3768DFFF     call 00409A50  
00613219    8B95 94FEFFFF   mov edx,dword ptr ss:\[ebp-16C\]  
0061321F    B8 84356100     mov eax,00613584 ; ASCII "unins"  
00613224    E8 F722DFFF     call 00405520  
00613229    85C0            test eax,eax  
0061322B    7E 64           jle short 00613291  
…… ……  
006132B3    8B95 84FEFFFF   mov edx,dword ptr ss:\[ebp-17C\]  
006132B9    B8 94356100     mov eax,00613594 ; ASCII ".EXE"  
006132BE    E8 5D22DFFF     call 00405520  
//查找本目录下的.exe 文件  
006132C3    85C0            test eax,eax  
006132C5    0F8E DA000000   jle 006133A5  
006132CB    8D95 78FEFFFF   lea edx,dword ptr ss:\[ebp-188\]  
006132D1    8B85 ACFEFFFF   mov eax,dword ptr ss:\[ebp-154\]  
006132D7    E8 C877DFFF     call 0040AAA4  
006132DC    8B85 78FEFFFF   mov eax,dword ptr ss:\[ebp-188\]  
006132E2    8D95 7CFEFFFF   lea edx,dword ptr ss:\[ebp-184\]  
006132E8    E8 6367DFFF     call 00409A50  
006132ED    8B85 7CFEFFFF   mov eax,dword ptr ss:\[ebp-184\]  
006132F3    BA A4356100     mov edx,006135A4 ; ASCII "STOCKSTORM.EXE"  
006132F8    E8 2B20DFFF     call 00405328  
//是否是 STOCKSTORM.EXE  
006132FD    74 34           je short 00613333  
006132FF    8D95 70FEFFFF   lea edx,dword ptr ss:\[ebp-190\]  
00613305    8B85 ACFEFFFF   mov eax,dword ptr ss:\[ebp-154\]  
0061330B    E8 9477DFFF     call 0040AAA4  
00613310    8B85 70FEFFFF   mov eax,dword ptr ss:\[ebp-190\]  
00613316    8D95 74FEFFFF   lea edx,dword ptr ss:\[ebp-18C\]  
0061331C    E8 2F67DFFF     call 00409A50  
00613321    8B85 74FEFFFF   mov eax,dword ptr ss:\[ebp-18C\]  
00613327    BA BC356100     mov edx,006135BC ; ASCII "UPDATE.EXE"  
0061332C    E8 F71FDFFF     call 00405328  
//是否是 UPDATE.EXE  
00613331    75 0C           jnz short 0061333F  
00613333    81BD A4FEFFFF 0>cmp dword ptr ss:\[ebp-15C\],19000  
0061333D    7D 66           jge short 006133A5  
0061333F    8D95 68FEFFFF   lea edx,dword ptr ss:\[ebp-198\]  
00613345    A1 70096200     mov eax,dword ptr ds:\[620970\]  
0061334A    8B00            mov eax,dword ptr ds:\[eax\]  
0061334C    E8 83A2E7FF     call 0048D5D4  
00613351    8B85 68FEFFFF   mov eax,dword ptr ss:\[ebp-198\]  
00613357    8D95 6CFEFFFF   lea edx,dword ptr ss:\[ebp-194\]  
0061335D    E8 BA76DFFF     call 0040AA1C  
00613362    8D85 6CFEFFFF   lea eax,dword ptr ss:\[ebp-194\]  
00613368    8B95 ACFEFFFF   mov edx,dword ptr ss:\[ebp-154\]  
0061336E    E8 711EDFFF     call 004051E4  
00613373    8B85 6CFEFFFF   mov eax,dword ptr ss:\[ebp-194\]  
00613379    E8 3614EEFF     call 004F47B4  
//如果目录下有非 STOCKSTORM.EXE 和 UPDATE.EXE 之外的其他 exe 文件，对不起，删之  
  
下面就是在 C 盘下创建一个批处理文件，格式如下：  
:try  
del "G:\\UnPack\\ACPrtect\\股市风暴 V6.0\\UnPacKed.exe"  
if exist "G:\\UnPack\\ACPrtect\\股市风暴 V6.0\\UnPacKed.exe" goto try  
del %0  
  
004F480F    B9 AC494F00     mov ecx,004F49AC ; ASCII ".bat"  
004F4814    E8 0F0AF1FF     call 00405228  
004F4819    8D45 F8         lea eax,dword ptr ss:\[ebp-8\]  
004F481C    8B4D F4         mov ecx,dword ptr ss:\[ebp-C\]  
004F481F    BA BC494F00     mov edx,004F49BC  
004F4824    E8 FF09F1FF     call 00405228  
004F4829    8B55 F8         mov edx,dword ptr ss:\[ebp-8\]  
004F482C    8D85 28FEFFFF   lea eax,dword ptr ss:\[ebp-1D8\]  
004F4832    E8 8DE7F0FF     call 00402FC4  
004F4837    8D85 28FEFFFF   lea eax,dword ptr ss:\[ebp-1D8\]  
004F483D    E8 1EE5F0FF     call 00402D60  
004F4842    E8 71E1F0FF     call 004029B8  
004F4847    BA C8494F00     mov edx,004F49C8 ; ASCII ":try"  
004F484C    8D85 28FEFFFF   lea eax,dword ptr ss:\[ebp-1D8\]  
004F4852    E8 A10DF1FF     call 004055F8  
004F4857    E8 7CF1F0FF     call 004039D8  
004F485C    E8 57E1F0FF     call 004029B8  
004F4861    68 D8494F00     push 004F49D8 ; ASCII "del ""  
004F4866    FF75 FC         push dword ptr ss:\[ebp-4\]  
004F4869    68 E8494F00     push 004F49E8  
004F486E    8D85 CCFDFFFF   lea eax,dword ptr ss:\[ebp-234\]  
004F4874    BA 03000000     mov edx,3  
004F4879    E8 1E0AF1FF     call 0040529C  
004F487E    8B95 CCFDFFFF   mov edx,dword ptr ss:\[ebp-234\]  
004F4884    8D85 28FEFFFF   lea eax,dword ptr ss:\[ebp-1D8\]  
004F488A    E8 690DF1FF     call 004055F8  
004F488F    E8 44F1F0FF     call 004039D8  
004F4894    E8 1FE1F0FF     call 004029B8  
004F4899    68 F4494F00     push 004F49F4 ; ASCII "if exist ""  
004F489E    FF75 FC         push dword ptr ss:\[ebp-4\]  
004F48A1    68 E8494F00     push 004F49E8  
004F48A6    68 084A4F00     push 004F4A08 ; ASCII " goto try"  
004F48AB    8D85 C8FDFFFF   lea eax,dword ptr ss:\[ebp-238\]  
004F48B1    BA 04000000     mov edx,4  
004F48B6    E8 E109F1FF     call 0040529C  
004F48BB    8B95 C8FDFFFF   mov edx,dword ptr ss:\[ebp-238\]  
004F48C1    8D85 28FEFFFF   lea eax,dword ptr ss:\[ebp-1D8\]  
004F48C7    E8 2C0DF1FF     call 004055F8  
004F48CC    E8 07F1F0FF     call 004039D8  
004F48D1    E8 E2E0F0FF     call 004029B8  
004F48D6    BA 1C4A4F00     mov edx,004F4A1C ; ASCII "del %0"  
004F48DB    8D85 28FEFFFF   lea eax,dword ptr ss:\[ebp-1D8\]  
004F48E1    E8 120DF1FF     call 004055F8  
004F48E6    E8 EDF0F0FF     call 004039D8  
004F48EB    E8 C8E0F0FF     call 004029B8  
004F48F0    8D85 28FEFFFF   lea eax,dword ptr ss:\[ebp-1D8\]  
004F48F6    E8 45E8F0FF     call 00403140  
004F48FB    E8 B8E0F0FF     call 004029B8  
004F4900    8D85 D4FDFFFF   lea eax,dword ptr ss:\[ebp-22C\]  
004F4906    33C9            xor ecx,ecx  
004F4908    BA 44000000     mov edx,44  
004F490D    E8 C2EAF0FF     call 004033D4  
004F4912    C785 00FEFFFF 0>mov dword ptr ss:\[ebp-200\],1  
004F491C    66:C785 04FEFFF>mov word ptr ss:\[ebp-1FC\],0  
004F4925    8D85 18FEFFFF   lea eax,dword ptr ss:\[ebp-1E8\]  
004F492B    50              push eax  
004F492C    8D85 D4FDFFFF   lea eax,dword ptr ss:\[ebp-22C\]  
004F4932    50              push eax  
004F4933    6A 00           push 0  
004F4935    6A 00           push 0  
004F4937    6A 40           push 40  
004F4939    6A 00           push 0  
004F493B    6A 00           push 0  
004F493D    6A 00           push 0  
004F493F    8B45 F8         mov eax,dword ptr ss:\[ebp-8\]  
004F4942    E8 950AF1FF     call 004053DC  
004F4947    50              push eax  
004F4948    6A 00           push 0  
004F494A    E8 E92EF1FF     call <jmp.&kernel32.CreateProcessA>  
//运行删除文件的批处理程序  
004F494F    85C0            test eax,eax  
004F4951    74 18           je short 004F496B  
  
  
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
      
              UnPacKed By :  fly  
               2005-04-18 12:00 

 4dd466d0e1e0cc123afbe504da2462ca