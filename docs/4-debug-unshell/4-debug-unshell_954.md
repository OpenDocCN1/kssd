

# 用 OllyDbg 手脱 RLPack V1.17 加壳的 DLL

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-490.htm`](https://www.pediy.com/kssd/pediy09/pediy09-490.htm)

> 用 OllyDbg 手脱 RLPack V1.17 加壳的 DLL  
>   
>   
> 【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教  
> 【调试环境】：WinXP、OllyDbg、PEiD、LordPE、ImportREC  
>   
> \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_  
> 【脱壳过程】：  
>   
>   
> RLPack 是 ap0x 新出的一款压缩壳，这篇教程以 RLPack V1.17 完整版自带的 iBox.dll 演示 RLPack 加壳的 DLL 手动脱壳。  
> RLPack 加壳的 eXe 脱壳方法可以参看我以前写的脚本  
> 
> 引用: RLPack.Basic.Edition.V1.0b-V1.17+Full.Edition.V1.16-V1.17.UnPacK.Script  
> http://www.unpack.cn/viewthread.php?tid=9965 
> 
> 值得一提的是，RLPack V1.18 越来越像 RLProtect，其 Code Splicing 和 Import Elimination 是很多的  
>   
> 设置 OllyDbg 暂停在 WinMain，忽略所有异常选项，开始吧  
> \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_  
> 一.OEP  
>   
> 通常压缩壳加壳的 DLL 找 OEP 是比较简单的  
> DLL 卸载时会再次从 EP 处运行，几个跳转后就会到 OEP 了  
>   
> 0094BEA0    807C24 08 01       cmp byte ptr ss:\[esp+8\],1  
> //进入 OllyDBG 后暂停在 EP  
> 0094BEA5    0F85 7E010000      jnz 0094C029  
> //这里在 DLL 卸载时会跳转，就是去 OEP 的捷径了  
>   
> 0094C029    E9 BE3AFAFF        jmp 008EFAEC  
> //这里就是跳 OEP 了  
>   
>   
> \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_  
> 二.输入表  
>   
> RLPack V1.1X Full Edition 加壳 exe 文件会加密某些输入表，而加壳 DLL 则很少加密输入表的  
> BP GetProcAddress  
> Shift+F9，中断后取消断点，Alt+F9 返回  
>   
> 0094BF57    56                 push esi  
> 0094BF58    FF95 E3090000      call near dword ptr ss:\[ebp+9E3\] ; kernel32.LoadLibraryA  
> 0094BF5E    8985 4E0A0000      mov dword ptr ss:\[ebp+A4E\],eax  
> 0094BF64    85C0               test eax,eax  
> 0094BF66    0F84 C2000000      je 0094C02E  
> 0094BF6C    8BC6               mov eax,esi  
> 0094BF6E    EB 5F              jmp short 0094BFCF  
> 0094BF70    8B85 520A0000      mov eax,dword ptr ss:\[ebp+A52\]  
> 0094BF76    8B00               mov eax,dword ptr ds:\[eax\]  
> 0094BF78    A9 00000080        test eax,80000000  
> 0094BF7D    74 14              je short 0094BF93  
> 0094BF7F    35 00000080        xor eax,80000000  
> 0094BF84    50                 push eax  
> 0094BF85    8B85 520A0000      mov eax,dword ptr ss:\[ebp+A52\]  
> 0094BF8B    C700 20202000      mov dword ptr ds:\[eax\],202020 ; UNICODE " Hercegovina"  
> 0094BF91    EB 06              jmp short 0094BF99  
> 0094BF93    FFB5 520A0000      push dword ptr ss:\[ebp+A52\]  
> 0094BF99    FFB5 4E0A0000      push dword ptr ss:\[ebp+A4E\]  
> 0094BF9F    FF95 E7090000      call near dword ptr ss:\[ebp+9E7\] ; kernel32.GetProcAddress  
> 0094BFA5    85C0               test eax,eax  
> //返回这里  
> 0094BFA7    0F84 81000000      je 0094C02E  
> 0094BFAD    8907               mov dword ptr ds:\[edi\],eax ; ntdll.RtlDeleteCriticalSection  
> //填充系统函数地址  
> //EDI=008F3154        注意观察这个地址  
> 0094BFAF    83C7 04            add edi,4  
> 0094BFB2    8B85 520A0000      mov eax,dword ptr ss:\[ebp+A52\]  
> 0094BFB8    EB 01              jmp short 0094BFBB  
> 0094BFBA    40                 inc eax  
> 0094BFBB    8038 00            cmp byte ptr ds:\[eax\],0  
> 0094BFBE    75 FA              jnz short 0094BFBA  
> 0094BFC0    40                 inc eax  
> 0094BFC1    8985 520A0000      mov dword ptr ss:\[ebp+A52\],eax  
> 0094BFC7    66:8178 02 0080    cmp word ptr ds:\[eax+2\],8000  
> 0094BFCD    74 A1              je short 0094BF70  
> 0094BFCF    8038 00            cmp byte ptr ds:\[eax\],0  
> 0094BFD2    75 9C              jnz short 0094BF70  
> 0094BFD4    EB 01              jmp short 0094BFD7  
> 0094BFD6    46                 inc esi  
> 0094BFD7    803E 00            cmp byte ptr ds:\[esi\],0  
> 0094BFDA    75 FA              jnz short 0094BFD6  
> 0094BFDC    46                 inc esi  
> 0094BFDD    40                 inc eax  
> 0094BFDE    8B38               mov edi,dword ptr ds:\[eax\]  
> 0094BFE0    E8 4B000000        call 0094C030  
> 0094BFE5    83C0 04            add eax,4  
> 0094BFE8    8985 520A0000      mov dword ptr ss:\[ebp+A52\],eax  
> 0094BFEE    803E 01            cmp byte ptr ds:\[esi\],1  
> 0094BFF1    0F85 60FFFFFF      jnz 0094BF57  
> //循环处理输入表  
>   
> 现在来手动确定输入表的 RVA 和 Size  
> 在左下角的数据窗口 Ctrl+G:008F3154，点右键->Long->Address  
> 008F3150  00000000  
> 008F3154  7C93188A  ntdll.RtlDeleteCriticalSection  
> 008F3158  7C9210ED  ntdll.RtlLeaveCriticalSection  
> ……  
> 008F37E0  7D610EC0  shell32.ShellExecuteA  
> 008F37E4  00000000  
> 008F37E8  76337CD8  
> 008F37EC  7632311E  
> 008F37F0  00000000  
>   
> 输入表开始 RVA=008F3154-00870000=00083154  
> 输入表 Size=008F37F0-008F3154=0000069C  
>   
>   
> \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_  
> 三.重定位表  
>   
> 其实写这篇教程的价值就在于这部分了  
> 经过跟踪发现 RLPack 没有加密重定位表，这就为我们脱壳减少了麻烦  
>   
> 0094BFF7    68 00400000        push 4000  
> 0094BFFC    68 54180000        push 1854  
> 0094C001    FFB5 560A0000      push dword ptr ss:\[ebp+A56\]  
> 0094C007    FF95 EF090000      call near dword ptr ss:\[ebp+9EF\] ; kernel32.VirtualFree  
> 0094C00D    68 00400000        push 4000  
> 0094C012    68 00200C00        push 0C2000  
> 0094C017    FFB5 3A0A0000      push dword ptr ss:\[ebp+A3A\]  
> 0094C01D    FF95 EF090000      call near dword ptr ss:\[ebp+9EF\] ; kernel32.VirtualFree  
> //清理战场了  
> 0094C023    E8 55000000        call 0094C07D  
> //重定位处理\[/code\]\[code\]0094C07D    60                 pushad  
> 0094C07E    8BB5 460A0000      mov esi,dword ptr ss:\[ebp+A46\]  
> //\[ebp+A46\]=00087000        重定位表 RVA  ★  
> 0094C084    0BF6               or esi,esi  
> 0094C086    74 67              je short 0094C0EF  
> 0094C088    8BBD 3E0A0000      mov edi,dword ptr ss:\[ebp+A3E\]  
> //\[ebp+A3E\]=00400000        文件基址  
> 0094C08E    8B4424 48          mov eax,dword ptr ss:\[esp+48\]  
> //\[esp+48\]=00870000                映像基址  
> 0094C092    8985 420A0000      mov dword ptr ss:\[ebp+A42\],eax  
> 0094C098    3BC7               cmp eax,edi  
> //比较是否相同  
> 0094C09A    74 53              je short 0094C0EF  
> //不同不跳就需要重定位处理了  
> //注意：此时程序没有重定位，可以现在 Dump，这样脱壳后就不需要修改 dump 文件基址了  ★  
> 0094C09C    03F0               add esi,eax  
> //ESI=00087000+00870000=008F7000                重定位表 VA  
> 0094C09E    EB 4A              jmp short 0094C0EA  
> 0094C0A0    8B16               mov edx,dword ptr ds:\[esi\]  
> 0094C0A2    8B46 04            mov eax,dword ptr ds:\[esi+4\]  
> 0094C0A5    8985 4A0A0000      mov dword ptr ss:\[ebp+A4A\],eax  
> 0094C0AB    01B5 4A0A0000      add dword ptr ss:\[ebp+A4A\],esi  
> 0094C0B1    83C6 08            add esi,8  
> 0094C0B4    EB 2C              jmp short 0094C0E2  
> 0094C0B6    0FB706             movzx eax,word ptr ds:\[esi\]  
> 0094C0B9    8BD8               mov ebx,eax  
> 0094C0BB    C1EB 0C            shr ebx,0C  
> 0094C0BE    8BCB               mov ecx,ebx  
> 0094C0C0    69DB 00100000      imul ebx,ebx,1000  
> 0094C0C6    2BC3               sub eax,ebx  
> 0094C0C8    03C2               add eax,edx  
> 0094C0CA    0385 420A0000      add eax,dword ptr ss:\[ebp+A42\]  
> 0094C0D0    83F9 03            cmp ecx,3  
> 0094C0D3    75 0A              jnz short 0094C0DF  
> 0094C0D5    2938               sub dword ptr ds:\[eax\],edi  
> 0094C0D7    8B8D 420A0000      mov ecx,dword ptr ss:\[ebp+A42\]  
> 0094C0DD    0108               add dword ptr ds:\[eax\],ecx  
> 0094C0DF    83C6 02            add esi,2  
> 0094C0E2    3BB5 4A0A0000      cmp esi,dword ptr ss:\[ebp+A4A\]  
> 0094C0E8    72 CC              jb short 0094C0B6  
> 0094C0EA    833E 00            cmp dword ptr ds:\[esi\],0  
> 0094C0ED    75 B1              jnz short 0094C0A0  
> //循环重定位处理  
> 0094C0EF    61                 popad  
> //处理完后 ESI=009000F8  
> //Relocation Table Size=009000F8-008F7000=000090F8  ★  
> 0094C0F0    C3                 retn  
>   
>   
> \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_  
> 四.完成脱壳  
>   
> 0094C028    61                 popad  
> 0094C029    E9 BE3AFAFF        jmp 008EFAEC  
> //飞向光明之巅  
> 0094C02E    61                 popad  
> 0094C02F    C3                 retn  
>   
> 008EFAEC    55                 push ebp  
> //OEP RVA=008EFAEC-00870000=0007FAEC  
> 008EFAED    8BEC               mov ebp,esp  
> 008EFAEF    83C4 C4            add esp,-3C  
> 008EFAF2    B8 04F98E00        mov eax,008EF904  
> 008EFAF7    E8 CC6DF8FF        call 008768C8  
> 008EFAFC    33C0               xor eax,eax  
> 008EFAFE    A3 442C8F00        mov dword ptr ds:\[8F2C44\],eax  
> 008EFB03    E8 DC4BF8FF        call 008746E4  
>   
> 运行 ImportREC，由于此 DLL 加载后已经进行重定位处理，所以去掉“Use PE Header From Disk”选项  
> 选择 OllyDbg 的 loaddll.exe 进程，Pick DLL 选择 iBox.dll  
> 填入 OEP RVA=0007FAEC，输入表 RVA=00083154，输入表 Size=0000069C，Get Imports  
> 可以新增区段修复，也可以把输入表放在程序无用的空白处。  
>   
> 使用 LordPE 修改 dumped\_.dll 的 Relocation Table RVA=00087000，Relocation Table Size=000090F8  
> 附件中 iBox.UnPacKed.dll 只是简单优化，如果想优化的完美点那就要多费时间了。  
> 脱壳完成  
>   
>   
> \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_  
>          ,     \_/   
>         /| \_.-~/            \\\_     ,        青春都一晌  
>        ( /~   /              \\~-.\_ |\\  
>        \`\\\\  \_/                \\   ~\\ )          忍把浮名   
>    \_-~~~-.)  )\_\_/;;,.          \\\_  //'  
>   /'\_,\\   --~   \\ ~~~-  ,;;\\\_\_\_(  (.-~~~-.        换了脱壳轻狂  
> \`~ \_( ,\_..--\\ (     ,;'' /    ~--   /.\_\`\\   
>   /~~//'   /' \`~\\         ) /--..\_, )\_  \`~  
>   "  \`~"  "      \`"      /~'\`\\    \`\\\\~~\\     
>                          "     "   "~'  ""  
>                UnPacKed By : fly  
>              http://www.unpack.cn  
>                2007.03.25 18:00 

 4dd466d0e1e0cc123afbe504da2462ca