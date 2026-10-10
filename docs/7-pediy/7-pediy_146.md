

# Inline Patch 注册 AsPrtect V2.X 保护程序——Tag&Rename V3.2 RC3

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-697.htm`](https://www.pediy.com/kssd/pediy07/pediy7-697.htm)

Inline Patch 注册 AsPrtect V2.X 保护程序——Tag&Rename V3.2 RC3  
              
             
               
下载页面：  http://www.skycn.com/soft/2893.html  
软件大小：  2577 KB  
软件语言：  英文  
软件类别：  国外软件 / 共享版 / 文件更名  
应用平台：  Win9x/NT/2000/XP  
加入时间：  2005-11-11 10:09:47  
下载次数：  10274  
开 发 商：  http://www.softpointer.com  
软件介绍：  使用 MP3/VQF 的 Tag Info 批量为文件改名。  
           
【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教  
               
【调试环境】：WinXP、OllyDBD、PEiD、LordPE、IDA  
               
—————————————————————————————————   
【脱壳过程】：  
            
           
AsPrtect V2.X 脱壳越来越麻烦，某些时候 Inline Patch 会省点事。国外不少人用此方法来对付 AsPrtect，JohnWho 写过教程，近来看到 temerata 也做了不少 Inline Patch 破解 AsPrtect 加壳程序。关于为何 Patch 可以参看以前的《Patch 注册 ASProtect V1.X 壳保护程序的方法》。  
下面的记录算是再学习一次 Inline Patch，没有新的成果。代码也较多，不适合新手练习。  
  
感谢前人的研究成果。感谢 shoooo 的帮忙分析自校验。  
下面以 Tag&Rename.V3.2.RC3\[3.2.56.119\]为例来演示  
   
—————————————————————————————————  
一、获取壳代码重定位基址  
  
  
前面部分的几个解压代码留在 Patch 部分分析，下面先简单跟踪一下流程。  
设置 Ollydbg 忽略所有的异常选项，用 IsDebug 插件去掉 Ollydbg 的调试器标志。  
  
00401000     68 01A08700        push 87A001  
//进入 Ollydbg 后暂停在这  
00401005     E8 01000000        call 0040100B  
0040100A     C3                 retn  
  
HE VirtualAlloc    Shift+F9，中断 2 次后取消断点，Alt+F9 返回  
  
0087A517     FF95 F0030000      call dword ptr ss:\[ebp+3F0\] ; kernel32.VirtualAlloc  
0087A51D     8985 31040000      mov dword ptr ss:\[ebp+431\],eax  
//\[ebp+431\]=\[0087A869\]=01280000 壳代码重定位基址  ★  
0087A523     8985 D0010000      mov dword ptr ss:\[ebp+1D0\],eax  
//\[ebp+1D0\]=\[0087A608\]=01280000  
  
  
—————————————————————————————————  
二、壳代码完全解压  
  
  
HE GetModuleHandleA    Shift+F9，中断 2 次后取消断点，Alt+F9 返回  
  
012B14A6     FF95 EC314400      call dword ptr ss:\[ebp+4431EC\]  
012B14AC     85C0               test eax,eax  
012B14AE     75 07              jnz short 012B14B7  
  
Ctrl+F 向下找 popad，看到 AsPrtect 解压壳代码的那个循环  
  
012B15C1     61                 popad  
//记住这里  ★  
012B15C2     75 08              jnz short 012B15CC  
012B15C4     B8 01000000        mov eax,1  
012B15C9     C2 0C00            retn 0C  
012B15CC     68 DC892A01        push 12A89DC  
012B15D1     C3                 retn  
  
  
—————————————————————————————————  
三、自校验：CreateFileMapping  
  
  
自从 AsPrtect V1.X 被直接 Patch 后作者在 V2.X 里面增强了自校验。  
要想办法找到可以搞定自校验的时机，AsPrtect 开始准备自校验的同时也给我们提供了解决它的机会。  
Ctrl+G:CreateFileMappingA    在函数尾下断，中断后取消断点，Alt+F9 返回  
  
01298630     6A 00              push 0  
01298632     6A 00              push 0  
01298634     6A 00              push 0  
01298636     6A 02              push 2  
//需要修改 push 8  ★  
01298638     6A 00              push 0  
0129863A     53                 push ebx  
0129863B     A1 E4972A01        mov eax,dword ptr ds:\[12A97E4\]  
01298640     8B40 1C            mov eax,dword ptr ds:\[eax+1C\]  
01298643     FFD0               call eax ; kernel32.CreateFileMappingA  
01298645     A3 14B42A01        mov dword ptr ds:\[12AB414\],eax  
0129864A     53                 push ebx  
0129864B     A1 E4972A01        mov eax,dword ptr ds:\[12A97E4\]  
01298650     8B40 18            mov eax,dword ptr ds:\[eax+18\]  
01298653     FFD0               call eax  
01298655     833D 14B42A01 00   cmp dword ptr ds:\[12AB414\],0  
0129865C     0F84 66040000      je 01298AC8  
01298662     6A 00              push 0  
01298664     6A 00              push 0  
01298666     6A 00              push 0  
01298668     6A 04              push 4  
//需要修改 push 1  ★  
0129866A     A1 14B42A01        mov eax,dword ptr ds:\[12AB414\]  
0129866F     50                 push eax  
01298670     A1 E4972A01        mov eax,dword ptr ds:\[12A97E4\]  
01298675     8B40 08            mov eax,dword ptr ds:\[eax+8\]  
01298678     FFD0               call eax ; kernel32.MapViewOfFileEx  
0129867A     8BD8               mov ebx,eax  
//EAX 值是映像文件的开始地址  ★  
0129867C     50                 push eax  
0129867D     E8 4A010000        call 012987CC  
  
  
为何不仅仅修改 MapViewOfFileEx 的参数？看看 MSDN  
  
HANDLE CreateFileMapping(  
  HANDLE hFile,                       // handle to file  
  LPSECURITY\_ATTRIBUTES lpAttributes, // security  
  DWORD flProtect,                    // protection  
  DWORD dwMaximumSizeHigh,            // high-order DWORD of size  
  DWORD dwMaximumSizeLow,             // low-order DWORD of size  
  LPCTSTR lpName                      // object name  
);  
  
flProtect :  
2=PAGE\_READONLY  
4=PAGE\_READWRITE  
8=PAGE\_WRITECOPY   //需要修改成这个参数  
  
Win9X 平台上若不修改 CreateFileMapping，则 MapViewOfFileEx 的 FILE\_MAP\_COPY 会失败  
  
LPVOID MapViewOfFileEx(  
  HANDLE hFileMappingObject,   // handle to file-mapping object  
  DWORD dwDesiredAccess,       // access mode  
  DWORD dwFileOffsetHigh,      // high-order DWORD of offset  
  DWORD dwFileOffsetLow,       // low-order DWORD of offset  
  SIZE\_T dwNumberOfBytesToMap, // number of bytes to map  
  LPVOID lpBaseAddress         // starting address  
);  
  
dwDesiredAccess :  
1=FILE\_MAP\_COPY  
4=FILE\_MAP\_READ  
  
Copy on write access. If you create the map with PAGE\_WRITECOPY and the view with FILE\_MAP\_COPY, you will receive a view to the file. If you write to it, the pages are automatically swappable and the modifications you make will not go to the original data file.   
Windows 95/98/Me: You must pass PAGE\_WRITECOPY to CreateFileMapping; otherwise, an error will be returned.  
  
  
—————————————————————————————————  
四、获得 ASProtect 注册名 Pre-Dip 处理的地址  
  
  
现在设置 Ollydbg 忽略除了“内存访问异常、同时忽略以下制定异常”之外的所有其它异常选项。  
Shift+F9 运行 N 次，直至堆栈中第 2 次看见硬盘指纹:  
  
0013FF1C   0013FF28  指针到下一个 SEH 记录  
0013FF20   0129C356  SE 句柄  
0013FF24   00000000  
0013FF28   0013FF80  指针到下一个 SEH 记录  
0013FF2C   0129D173  SE 句柄  
0013FF30   0013FF78  
0013FF34   01280000  
0013FF38   01240000  
0013FF3C   0129B470  
0013FF40   00000000  
0013FF44   012D5468  ASCII "wWDDDAAgLTg="  
0013FF48   012D5434  ASCII "AFBC4FD7-E635"  
  
Alt+M 打开内存查看窗口，在 00401000 段“设置内存访问断点”，Shift+F9，断下  
  
004B7B44     55                 push ebp  
//中断在这里  
004B7B45     8BEC               mov ebp,esp  
004B7B47     A1 B0037100        mov eax,dword ptr ds:\[7103B0\]  
004B7B4C     A3 AC037100        mov dword ptr ds:\[7103AC\],eax  
004B7B51     8B45 08            mov eax,dword ptr ss:\[ebp+8\]  
004B7B54     A3 B0037100        mov dword ptr ds:\[7103B0\],eax  
004B7B59     5D                 pop ebp  
004B7B5A     C2 0400            retn 4  
  
取消内存断点，向上看：  
004B7B18     55                 push ebp  
//下断，Shift+F9 直至中断在这里，取消断点  
004B7B19     8BEC               mov ebp,esp  
004B7B1B     8B45 08            mov eax,dword ptr ss:\[ebp+8\]  
//\[ebp+8\]=\[0012FF24\]=01283A29   注册名保存地址  
004B7B1E     85C0               test eax,eax  
004B7B20     74 0C              je short 004B7B2E  
004B7B22     8038 00            cmp byte ptr ds:\[eax\],0  
004B7B25     74 07              je short 004B7B2E  
004B7B27     C605 B4037100 01   mov byte ptr ds:\[7103B4\],1  
//注意：\[7103B4\]应该置 1       ★  
004B7B2E     8B15 A8037100      mov edx,dword ptr ds:\[7103A8\]  
004B7B34     8915 A4037100      mov dword ptr ds:\[7103A4\],edx  
004B7B3A     A3 A8037100        mov dword ptr ds:\[7103A8\],eax  
//注册名保存地址放入\[7103A8\]   ★  
004B7B3F     5D                 pop ebp  
004B7B40     C2 0400            retn 4  
//返回 0129CABF  
  
0129CAB9     50                 push eax  
//注册名保存地址               ★  
0129CABA     8B47 04            mov eax,dword ptr ds:\[edi+4\]  
0129CABD     FFD0               call eax ; 004B7B18  
  
  
—————————————————————————————————  
五、程序中需要修改的地方  
  
  
但是这里我们直接修改此 Pre-Dip 后程序还是显示未注册，呵呵  
Tag&Rename 和 AlfaClock、Bee Icons 等不同，还需要修改程序  
  
1、判断是否是注册版  
  
0070CFC5     E8 FE4FDBFF        call 004C1FC8  
//进入修改     
0070CFCA     84C0               test al,al  
0070CFCC     0F85 0E010000      jnz 0070D0E0  
0070CFD2     33C9               xor ecx,ecx  
0070CFD4     B2 01              mov dl,1  
0070CFD6     A1 C45E6A00        mov eax,dword ptr ds:\[6A5EC4\]  
0070CFDB     E8 B485D8FF        call 00495594  
0070CFE0     8B15 D88F7100      mov edx,dword ptr ds:\[718FD8\]  
0070CFE6     8902               mov dword ptr ds:\[edx\],eax  
0070CFE8     33C0               xor eax,eax  
0070CFEA     55                 push ebp  
0070CFEB     68 B9D07000        push 70D0B9  
0070CFF0     64:FF30            push dword ptr fs:\[eax\]  
0070CFF3     64:8920            mov dword ptr fs:\[eax\],esp  
0070CFF6     A1 D88F7100        mov eax,dword ptr ds:\[718FD8\]  
0070CFFB     8B00               mov eax,dword ptr ds:\[eax\]  
0070CFFD     E8 9ED8D8FF        call 0049A8A0  
0070D002     A1 D88F7100        mov eax,dword ptr ds:\[718FD8\]  
0070D007     8B00               mov eax,dword ptr ds:\[eax\]  
0070D009     8B10               mov edx,dword ptr ds:\[eax\]  
0070D00B     FF92 88000000      call dword ptr ds:\[edx+88\]  
0070D011     8B0D B0867100      mov ecx,dword ptr ds:\[7186B0\]  
0070D017     A1 B48B7100        mov eax,dword ptr ds:\[718BB4\]  
0070D01C     8B00               mov eax,dword ptr ds:\[eax\]  
0070D01E     8B15 6C876E00      mov edx,dword ptr ds:\[6E876C\]  
0070D024     E8 CF1AD9FF        call 0049EAF8  
0070D029     A1 D88F7100        mov eax,dword ptr ds:\[718FD8\]  
0070D02E     8B00               mov eax,dword ptr ds:\[eax\]  
0070D030     8B10               mov edx,dword ptr ds:\[eax\]  
0070D032     FF92 88000000      call dword ptr ds:\[edx+88\]  
0070D038     68 DC050000        push 5DC  
0070D03D     E8 3A4DD0FF        call 00411D7C  
0070D042     A1 B0867100        mov eax,dword ptr ds:\[7186B0\]  
0070D047     8B00               mov eax,dword ptr ds:\[eax\]  
0070D049     BA 70D27000        mov edx,70D270 ; ASCII "Tag&Rename 3.2 rc 3 UNREGISTERED"  
0070D04E     E8 418AD6FF        call 00475A94  
  
004C20B1     C645 FB 00         mov byte ptr ss:\[ebp-5\],0  
//Patch 点:  C645 FB 01         mov byte ptr ss:\[ebp-5\],1  ★  
004C20B5     EB 2E              jmp short 004C20E5  
  
————————————————————————  
2、显示注册名  
  
00684611     E8 8230D8FF        call 00407698  
00684616     8D45 E4            lea eax,dword ptr ss:\[ebp-1C\]  
00684619     8B55 F8            mov edx,dword ptr ss:\[ebp-8\]  
//\[ebp-8\]指向注册名，最后一起修改  
0068461C     E8 770DD8FF        call 00405398  
  
不要问我是如何找的，大家都会调试，只不过某些人在这方面用的时间多点而已。  
  
  
—————————————————————————————————  
六、RegOpenKeyExA  
  
  
Ctrl+F 在“整个段块”搜索命令：push 20019  
  
0128A289     68 19000200        push 20019  
//找到这里  
0128A28E     6A 00              push 0  
0128A290     8BC7               mov eax,edi  
0128A292     E8 8D97FFFF        call 01283A24  
0128A297     50                 push eax  
0128A298     56                 push esi  
0128A299     E8 36B4FFFF        call 012856D4 ; jmp to advapi32.RegOpenKeyExA  
//Call RegOpenKeyExA  
0128A29E     85C0               test eax,eax  
0128A2A0     0F85 84000000      jnz 0128A32A  
  
点右键，搜索全部命令：call 012856D4  
  
地址       反汇编             注释  
0128A299   call 012856D4      jmp to advapi32.RegOpenKeyExA  
0128A376   call 012856D4      jmp to advapi32.RegOpenKeyExA  
0128A3E2   call 012856D4      jmp to advapi32.RegOpenKeyExA  
0128A495   call 012856D4      jmp to advapi32.RegOpenKeyExA  
0128A57D   call 012856D4      jmp to advapi32.RegOpenKeyExA  
0129E1C3   call 012856D4      jmp to advapi32.RegOpenKeyExA  
0129E260   call 012856D4      jmp to advapi32.RegOpenKeyExA  
  
一般修改后面 3 个 CALL 后的跳转就行了，记住这几个地方  
其实在本例中不需要修改这里，但是为了解说也 Patch 了  
  
0128A57D     E8 52B1FFFF        call 012856D4 ; jmp to advapi32.RegOpenKeyExA  
0128A582     85C0               test eax,eax  
0128A584     75 30              jnz short 0128A5B6  
0129E1C3     E8 0C75FEFF        call 012856D4 ; jmp to advapi32.RegOpenKeyExA  
0129E1C8     85C0               test eax,eax  
0129E1CA     75 42              jnz short 0129E20E  
0129E260     E8 6F74FEFF        call 012856D4 ; jmp to advapi32.RegOpenKeyExA  
0129E265     85C0               test eax,eax  
0129E267     0F85 88000000      jnz 0129E2F5  
  
  
—————————————————————————————————  
七、一起 Patch 吧  
  
  
大体流程清楚了，下面开始 Patch 吧。复制 TagRename.exe 改名为 Patch.eXe  
  
看看 Patch.eXe 的 PE 信息，SizeOfImage=004A2000，区段信息如下：  
No    Name        VSize        VOffset      RSize        ROffset      Charact  
01                0030E000     00001000     000F4A00     00000400     E0000040  
02                0000B000     0030F000     00004000     000F4E00     E0000040  
03                00002000     0031A000     00000000     000F8E00     E0000040  
04                00004000     0031C000     00003800     000F8E00     E0000040  
05                00001000     00320000     00000200     000FC600     E0000040  
06                00001000     00321000     00000000     000FC800     E0000040  
07                00001000     00322000     00000200     000FC800     E0000040  
08                00029000     00323000     00000000     000FCA00     E0000040  
09    .rsrc       0012E000     0034C000     0009BA00     000FCA00     E0000040  
0A    .0000       00027000     0047A000     00026400     00198400     E0000040  
0B    .adata      00001000     004A1000     00000000     001BE800     E0000040  
  
用 LordPE 把最后一个区段 VSize 和 RSize 都改为 1500，保存后 LordPE 自动把 SizeOfImage 改为 004A2500  
用 WinHex 在 Patch.eXe 末尾添加 1500H 长度的 00 数据。当然，如果代码不多的话可以在程序中找空地。这样添加 1500H 长度后我们就获得了后面 500H 空白可以用，而不必管是否会被其他覆盖了。008A2000（偏移 0X1BF800）处我们就可以写代码了。  
  
新开 OllyDBG 载入 Patch.eXe，我们要先 Patch 几个循环解码，再 Patch 上面六步分析的地方。  
————————————————————————  
1、第 1 个点  
  
00401000     68 01A08700        push 87A001  
//进入 Ollydbg 后暂停在这  
00401005     E8 01000000        call 0040100B  
0040100A     C3                 retn  
  
F7 单步走，看 ASProtect 解压壳代码  
  
0087A0FD     BE A93C493D        mov esi,3D493CA9  
0087A102     81F6 543D493D      xor esi,3D493D54  
0087A108     B8 EB8B342E        mov eax,2E348BEB  
0087A10D     8B0A               mov ecx,dword ptr ds:\[edx\]  
0087A10F     0F8B 08000000      jpo 0087A11D  
0087A115     68 924FB86F        push 6FB84F92  
0087A11A     8BFB               mov edi,ebx  
0087A11C     5F                 pop edi  
0087A11D     81C1 21A29555      add ecx,5595A221  
0087A123     80DB 8D            sbb bl,8D  
0087A126     81C1 4646A24E      add ecx,4EA24646  
0087A12C     51                 push ecx  
0087A12D     68 AF332832        push 322833AF  
0087A132     B0 5B              mov al,5B  
0087A134     5F                 pop edi  
0087A135     58                 pop eax  
0087A136     81C1 0770A50C      add ecx,0CA57007  
0087A13C     0F81 00000000      jno 0087A142  
0087A142     890A               mov dword ptr ds:\[edx\],ecx  
0087A144     81EA 31F64154      sub edx,5441F631  
0087A14A     BB A208717E        mov ebx,7E7108A2  
0087A14F     81C2 2DF64154      add edx,5441F62D  
0087A155     8BC7               mov eax,edi  
0087A157     83EE 01            sub esi,1  
0087A15A     0F85 ADFFFFFF      jnz 0087A10D  
//解码循环   第 1 个点  接口   ★  
//Patch ①、 E9 E17E0200        jmp 008A2040  
  
Patch 代码：  
008A2040     0F85 C780FDFF      jnz 0087A10D  
//0087A15A 代码挪这里执行  
008A2046     C705 5AA18700 0F85>mov dword ptr ds:\[87A15A\],FFAD850F  
008A2050     66:C705 5EA18700 F>mov word ptr ds:\[87A15E\],0FFFF  
//还原 0087A15A 处修改的代码  
008A2059     C705 D8A18700 8C7E>mov dword ptr ds:\[87A1D8\],27E8C  
//Patch 第 2 个点  
008A2063     E9 F880FDFF        jmp 0087A160  
//继续流程  
  
  
————————————————————————  
2、第 2 个点  
  
0087A192     FF30               push dword ptr ds:\[eax\]  
0087A194     B9 3390F94E        mov ecx,4EF99033  
0087A199     5F                 pop edi  
0087A19A     0FB7CB             movzx ecx,bx  
0087A19D     81F7 1D90B243      xor edi,43B2901D  
0087A1A3     8BD7               mov edx,edi  
0087A1A5     81EF 92625E0E      sub edi,0E5E6292  
0087A1AB     81D1 52F1012A      adc ecx,2A01F152  
0087A1B1     81C7 63A4242F      add edi,2F24A463  
0087A1B7     8AEF               mov ch,bh  
0087A1B9     8938               mov dword ptr ds:\[eax\],edi  
0087A1BB     B3 B8              mov bl,0B8  
0087A1BD     83E8 04            sub eax,4  
0087A1C0     68 4D3C7652        push 52763C4D  
0087A1C5     51                 push ecx  
0087A1C6     80CA 05            or dl,5  
0087A1C9     59                 pop ecx  
0087A1CA     5A                 pop edx  
0087A1CB     4E                 dec esi  
0087A1CC     0F85 18000000      jnz 0087A1EA  
0087A1D2     66:81F1 80F3       xor cx,0F380  
0087A1D7     E9 3A000000        jmp 0087A216  
//解码循环   第 2 个点  
//Patch ②、 E9 8C7E0200        jmp 008A2068  
  
Patch 代码：  
008A2068     C705 D8A18700 3A00>mov dword ptr ds:\[87A1D8\],3A  
//还原 0087A1D8 处修改的代码  
008A2072     C705 CFA28700 E9B6>mov dword ptr ds:\[87A2CF\],27DB6E9  
//Patch 第 3 个点  
008A207C     66:C705 D3A28700 0>mov word ptr ds:\[87A2D3\],0FF00  
008A2085     E9 4D81FDFF        jmp 0087A1D7  
  
  
————————————————————————  
3、第 3 个点  
  
0087A2C4     83EA 04            sub edx,4  
0087A2C7     BB E633514B        mov ebx,4B5133E6  
0087A2CC     83EE 01            sub esi,1  
0087A2CF     0F85 87FFFFFF      jnz 0087A25C  
//解码循环   第 3 个点  
//Patch ③、 E9 B67D0200        jmp 008A208A  
  
Patch 代码：  
008A208A     0F85 CC81FDFF      jnz 0087A25C  
008A2090     C705 CFA28700 0F85>mov dword ptr ds:\[87A2CF\],FF87850F  
008A209A     66:C705 D3A28700 F>mov word ptr ds:\[87A2D3\],0FFFF  
//还原 0087A2CF 处修改的代码  
008A20A3     C705 7FA38700 E937>mov dword ptr ds:\[87A37F\],27D37E9  
008A20AD     66:C705 83A38700 0>mov word ptr ds:\[87A383\],0FF00  
//Patch 第 4 个点  
008A20B6     E9 1A82FDFF        jmp 0087A2D5  
  
  
————————————————————————  
4、第 4 个点  
  
0087A34D     81EF F17FF02D      sub edi,2DF07FF1  
0087A353     81F3 B6D34803      xor ebx,348D3B6  
0087A359     66:BF 62DB         mov di,0DB62  
0087A35D     81C3 B7C8470B      add ebx,0B47C8B7  
0087A363     8918               mov dword ptr ds:\[eax\],ebx  
0087A365     81D7 61475018      adc edi,18504761  
0087A36B     81E8 47B71B45      sub eax,451BB747  
0087A371     B1 80              mov cl,80  
0087A373     81C0 43B71B45      add eax,451BB743  
0087A379     BA 0C25D675        mov edx,75D6250C  
0087A37E     4E                 dec esi  
0087A37F     0F85 B4FFFFFF      jnz 0087A339  
//解码循环   第 4 个点  
//Patch ④、 E9 377D0200        jmp 008A20BB  
  
Patch 代码：  
008A20BB     0F85 7882FDFF      jnz 0087A339  
//0087A37F 代码挪这里执行  
008A20C1     C705 7FA38700 0F85>mov dword ptr ds:\[87A37F\],FFB4850F  
008A20CB     66:C705 83A38700 F>mov word ptr ds:\[87A383\],0FFFF  
//还原 0087A37F 处修改的代码  
008A20D4     C705 27A48700 E9C0>mov dword ptr ds:\[87A427\],27CC0E9  
008A20DE     66:C705 2BA48700 0>mov word ptr ds:\[87A42B\],0FF00  
//Patch 第 5 个点  
008A20E7     E9 9982FDFF        jmp 0087A385  
  
  
————————————————————————  
5、第 5 个点  
  
0087A3F4     59                 pop ecx  
0087A3F5     5A                 pop edx  
0087A3F6     81E8 9A2CB57B      sub eax,7BB52C9A  
0087A3FC     50                 push eax  
0087A3FD     8F041F             pop dword ptr ds:\[edi+ebx\]  
0087A400     66:BE D174         mov si,74D1  
0087A404     66:BE 09ED         mov si,0ED09  
0087A408     81EB 2FB9E712      sub ebx,12E7B92F  
0087A40E     B9 7DD3FF76        mov ecx,76FFD37D  
0087A413     81C3 2BB9E712      add ebx,12E7B92B  
0087A419     0F8B 02000000      jpo 0087A421  
0087A41F     B2 72              mov dl,72  
0087A421     81FB D8FAFFFF      cmp ebx,-528  
0087A427     0F85 91FFFFFF      jnz 0087A3BE  
//解码循环   第 5 个点  
//Patch ⑤、 E9 C07C0200        jmp 008A20EC  
  
Patch 代码：  
008A20EC     0F85 CC82FDFF      jnz 0087A3BE  
//0087A427 代码挪这里执行  
008A20F2     C705 27A48700 0F85>mov dword ptr ds:\[87A427\],FF91850F  
008A20FC     66:C705 2BA48700 F>mov word ptr ds:\[87A42B\],0FFFF  
//还原 0087A427 处修改的代码  
008A2105     C705 97A58700 E981>mov dword ptr ds:\[87A597\],27B81E9  
008A210F     66:C705 9BA58700 0>mov word ptr ds:\[87A59B\],0  
//Patch 第 6 个点  
008A2118     E9 1083FDFF        jmp 0087A42D  
  
  
————————————————————————  
6、第 6 个点   获取壳代码重定位基址  
  
0087A517     FF95 F0030000      call dword ptr ss:\[ebp+3F0\] ; kernel32.VirtualAlloc  
0087A51D     8985 31040000      mov dword ptr ss:\[ebp+431\],eax  
//\[ebp+431\]=\[0087A869\]=01280000 壳代码基址  
0087A523     8985 D0010000      mov dword ptr ss:\[ebp+1D0\],eax  
//\[ebp+1D0\]=\[0087A608\]=01280000  
0087A529     64:67:A1 0000      mov eax,dword ptr fs:\[0\]  
0087A52E     8985 2D040000      mov dword ptr ss:\[ebp+42D\],eax  
0087A534     8B55 5B            mov edx,dword ptr ss:\[ebp+5B\]  
0087A537     8B85 D0010000      mov eax,dword ptr ss:\[ebp+1D0\]  
0087A53D     8902               mov dword ptr ds:\[edx\],eax  
0087A53F     8B85 08040000      mov eax,dword ptr ss:\[ebp+408\]  
0087A545     8942 04            mov dword ptr ds:\[edx+4\],eax  
0087A548     8D85 9F030000      lea eax,dword ptr ss:\[ebp+39F\]  
0087A54E     8B40 55            mov eax,dword ptr ds:\[eax+55\]  
0087A551     8942 08            mov dword ptr ds:\[edx+8\],eax  
0087A554     8B85 EC030000      mov eax,dword ptr ss:\[ebp+3EC\]  
0087A55A     8942 10            mov dword ptr ds:\[edx+10\],eax  
0087A55D     8B85 E8030000      mov eax,dword ptr ss:\[ebp+3E8\]  
0087A563     8942 14            mov dword ptr ds:\[edx+14\],eax  
0087A566     8B95 CC010000      mov edx,dword ptr ss:\[ebp+1CC\]  
0087A56C     BB F8010000        mov ebx,1F8  
0087A571     8B7C1A 0C          mov edi,dword ptr ds:\[edx+ebx+C\]  
0087A575     0BFF               or edi,edi  
0087A577     74 1E              je short 0087A597  
0087A579     8B4C1A 10          mov ecx,dword ptr ds:\[edx+ebx+10\]  
0087A57D     0BC9               or ecx,ecx  
0087A57F     74 11              je short 0087A592  
0087A581     03BD D0010000      add edi,dword ptr ss:\[ebp+1D0\]  
0087A587     8B741A 14          mov esi,dword ptr ds:\[edx+ebx+14\]  
0087A58B     03F2               add esi,edx  
0087A58D     C1F9 02            sar ecx,2  
0087A590     F3:A5              rep movs dword ptr es:\[edi\],dword ptr ds:\[esi\]  
0087A592     83C3 28            add ebx,28  
0087A595     EB DA              jmp short 0087A571  
0087A597     8B85 CC010000      mov eax,dword ptr ss:\[ebp+1CC\]  
//解码循环   第 6 个点  
//Patch ⑥、 E9 817B0200        jmp 008A211D  
  
Patch 代码：  
008A211D     C705 97A58700 8B85>mov dword ptr ds:\[87A597\],1CC858B  
008A2127     C605 9BA58700 00   mov byte ptr ds:\[87A59B\],0  
//还原 0087A597 处修改的代码  
008A212E     8B85 31040000      mov eax,dword ptr ss:\[ebp+431\]  
//取得壳代码基址  
008A2134     C780 ED100300 684C>mov dword ptr ds:\[eax+310ED\],8A214C68  
008A213E     66:C780 F1100300 0>mov word ptr ds:\[eax+310F1\],0C300  
//Patch 第 7 个点 012B10ED  
008A2147     E9 4B84FDFF        jmp 0087A597  
  
  
————————————————————————  
7、第 7 个点  
  
012B10C4     FF95 79294400      call dword ptr ss:\[ebp+442979\]  
012B10CA     8985 75294400      mov dword ptr ss:\[ebp+442975\],eax  
012B10D0     8D9D 452A4400      lea ebx,dword ptr ss:\[ebp+442A45\]  
012B10D6     50                 push eax  
012B10D7     53                 push ebx  
012B10D8     E8 74050000        call 012B1651  
012B10DD     8BC8               mov ecx,eax  
012B10DF     8DBD 452A4400      lea edi,dword ptr ss:\[ebp+442A45\]  
012B10E5     8BB5 75294400      mov esi,dword ptr ss:\[ebp+442975\]  
012B10EB     F3:A4              rep movs byte ptr es:\[edi\],byte ptr ds:\[esi\]  
012B10ED     8B85 75294400      mov eax,dword ptr ss:\[ebp+442975\]  
//第 7 个点  
//Patch ⑦、  
012B10ED     68 4C218A00        push 8A214C  
012B10F2     C3                 retn  
  
Patch 代码：  
008A214C     A1 69A88700        mov eax,dword ptr ds:\[87A869\]  
//取得壳代码基址  
008A2151     C780 ED100300 8B85>mov dword ptr ds:\[eax+310ED\],2975858B  
008A215B     66:C780 F1100300 4>mov word ptr ds:\[eax+310F1\],44  
//还原 012B10ED 处修改的代码  
008A2164     C780 C2150300 687E>mov dword ptr ds:\[eax+315C2\],8A217E68  
008A216E     66:C780 C6150300 0>mov word ptr ds:\[eax+315C6\],0C300  
//Patch 第 8 个点 012B15C2  
008A2177     05 ED100300        add eax,310ED  
008A217C     FFE0               jmp eax  
//返回 012B10ED 处继续执行  
  
  
————————————————————————  
8、第 8 个点   CreateFileMappingA 处理  
  
012B15C1     61                 popad  
012B15C2     75 08              jnz short 012B15CC  
//第 8 个点       
//Patch ⑧、   
012B15C2     68 7E218A00        push 8A217E  
012B15C7     C3                 retn  
  
Patch 代码：  
008A217E     A1 69A88700        mov eax,dword ptr ds:\[87A869\]  
//取得壳代码基址  
008A2183     C780 C2150300 7508>mov dword ptr ds:\[eax+315C2\],1B80875  
008A218D     66:C780 C6150300 0>mov word ptr ds:\[eax+315C6\],0  
//还原 012B15C2 处修改的代码  
008A2196     C680 37860100 08   mov byte ptr ds:\[eax+18637\],8  
//修改 01298636 处的 push 2 为 push 8  ★  CreateFileMappingA 参数  
008A219D     C680 69860100 01   mov byte ptr ds:\[eax+18669\],1  
//修改 01298668 处的 push 4 为 push 1  ★  MapViewOfFileEx 参数  
008A21A4     C780 7A860100 68BE>mov dword ptr ds:\[eax+1867A\],8A21BE68  
008A21AE     66:C780 7E860100 0>mov word ptr ds:\[eax+1867E\],0C300  
//Patch 第 9 个点 0129867A  
008A21B7     05 C2150300        add eax,315C2  
008A21BC     FFE0               jmp eax  
//返回 012B15C2 继续执行  
  
  
————————————————————————  
9、第 9 个点   自校验处理  
  
01298630     6A 00              push 0  
01298632     6A 00              push 0  
01298634     6A 00              push 0  
01298636     6A 02              push 2  
01298638     6A 00              push 0  
0129863A     53                 push ebx  
0129863B     A1 E4972A01        mov eax,dword ptr ds:\[12A97E4\]  
01298640     8B40 1C            mov eax,dword ptr ds:\[eax+1C\]  
01298643     FFD0               call eax ; kernel32.CreateFileMappingA  
01298645     A3 14B42A01        mov dword ptr ds:\[12AB414\],eax  
0129864A     53                 push ebx  
0129864B     A1 E4972A01        mov eax,dword ptr ds:\[12A97E4\]  
01298650     8B40 18            mov eax,dword ptr ds:\[eax+18\]  
01298653     FFD0               call eax  
01298655     833D 14B42A01 00   cmp dword ptr ds:\[12AB414\],0  
0129865C     0F84 66040000      je 01298AC8  
01298662     6A 00              push 0  
01298664     6A 00              push 0  
01298666     6A 00              push 0  
01298668     6A 04              push 4  
0129866A     A1 14B42A01        mov eax,dword ptr ds:\[12AB414\]  
0129866F     50                 push eax  
01298670     A1 E4972A01        mov eax,dword ptr ds:\[12A97E4\]  
01298675     8B40 08            mov eax,dword ptr ds:\[eax+8\]  
01298678     FFD0               call eax ; kernel32.MapViewOfFileEx  
0129867A     8BD8               mov ebx,eax  
//第 9 个点       
//Patch ⑨、  
0129867A     68 BE218A00        push 8A21BE  
0129867F     C3                 retn  
  
Patch 代码：  
008A21BE     8B1D 69A88700      mov ebx,dword ptr ds:\[87A869\]  
//取得壳代码基址  
008A21C4     C683 37860100 02   mov byte ptr ds:\[ebx+18637\],2  
//还原 01298636 处修改的代码  
008A21CB     C683 69860100 04   mov byte ptr ds:\[ebx+18669\],4  
//还原 01298668 处修改的代码  
008A21D2     C783 7A860100 8BD8>mov dword ptr ds:\[ebx+1867A\],E850D88B  
008A21DC     66:C783 7E860100 4>mov word ptr ds:\[ebx+1867E\],14A  
//还原 0129867A 处修改的代码  
008A21E5     C780 50010000 0020>mov dword ptr ds:\[eax+150\],4A2000  
//EAX 值是映像文件的开始地址，还原映像文件的 SizeOfImage  
008A21EF     66:C780 90030000 0>mov word ptr ds:\[eax+390\],1000  
//还原映像文件最后一个区段的 VSize 为 1000  
008A21F8     66:C780 98030000 0>mov word ptr ds:\[eax+398\],0  
//还原映像文件最后一个区段的 RSize 为 0000  
008A2201     C780 5A851900 0F85>mov dword ptr ds:\[eax+19855A\],FFAD850F  
008A220B     66:C780 5E851900 F>mov word ptr ds:\[eax+19855E\],0FFFF  
//还原映像文件中第一个接口修改的代码  
008A2214     C783 B9CA0100 682F>mov dword ptr ds:\[ebx+1CAB9\],8A222F68  
008A221E     66:C783 BDCA0100 0>mov word ptr ds:\[ebx+1CABD\],0C300  
//Patch 第 10 个点 0129CAB9  
008A2227     81C3 7A860100      add ebx,1867A  
008A222D     FFE3               jmp ebx  
//返回 0129867A 继续执行  
  
  
————————————————————————  
10、第 10 个点   注册名 Pre-Dip 处理  
  
0129CAB9     50                 push eax  
//第 10 个点       
0129CABA     8B47 04            mov eax,dword ptr ds:\[edi+4\]  
0129CABD     FFD0               call eax  
//注册名 Pre-Dip 处理  
  
//Patch 10、  
0129CAB9     68 2F228A00        push 8A222F  
0129CABE     C3                 retn  
  
Patch 代码：  
008A222F     A1 69A88700        mov eax,dword ptr ds:\[87A869\]  
008A2234     C780 B9CA0100 508B>mov dword ptr ds:\[eax+1CAB9\],4478B50  
008A223E     66:C780 BDCA0100 F>mov word ptr ds:\[eax+1CABD\],0D0FF  
//还原 0129CAB9 处修改的代码  
008A2247     C680 84A50000 EB   mov byte ptr ds:\[eax+A584\],0EB  
008A224E     C680 CAE10100 EB   mov byte ptr ds:\[eax+1E1CA\],0EB  
008A2255     C780 67E20100 E989>mov dword ptr ds:\[eax+1E267\],89E9  
008A225F     C605 B4204C00 01   mov byte ptr ds:\[4C20B4\],1  
//修改程序中判断是否注册的标志位  
008A2266     C705 1D466800 60DC>mov dword ptr ds:\[68461D\],21DC60  
//Patch 第 11 个点  0068461C  
008A2270     68 04208A00        push 8A2004  ; ASCII "fly \[2005.12.01\]"  
//Push 注册名地址，预先在 8A2004 处写下注册名  
008A2275     A1 69A88700        mov eax,dword ptr ds:\[87A869\]  
008A227A     05 BACA0100        add eax,1CABA  
008A227F     FFE0               jmp eax  
//返回 0129CABA 继续执行  
  
  
————————————————————————  
11、第 11 个点   注册名显示  
  
00684616     8D45 E4            lea eax,dword ptr ss:\[ebp-1C\]  
00684619     8B55 F8            mov edx,dword ptr ss:\[ebp-8\]  
0068461C     E8 770DD8FF        call 00405398  
//第 11 个点  
//Patch 11、  
0068461C     E8 60DC2100        call 008A2281  
  
Patch 代码：  
008A2281     BA 04208A00        mov edx,8A2004  
//Patch 注册名地址，注意 8A2004-4 处应该是字符串长度  
008A2286     E9 0D31B6FF        jmp 00405398  
//继续流程  
  
  
—————————————————————————————————  
八、Patch 代码汇总  
  
  
保存一下 Patch 代码，以后再做会省点事吧。  
  
008A2040     0F85 C780FDFF               jnz 0087A10D  
008A2046     C705 5AA18700 0F85ADFF      mov dword ptr ds:\[87A15A\],FFAD850F  
008A2050     66:C705 5EA18700 FFFF       mov word ptr ds:\[87A15E\],0FFFF  
008A2059     C705 D8A18700 8C7E0200      mov dword ptr ds:\[87A1D8\],27E8C  
008A2063     E9 F880FDFF                 jmp 0087A160  
008A2068     C705 D8A18700 3A000000      mov dword ptr ds:\[87A1D8\],3A  
008A2072     C705 CFA28700 E9B67D02      mov dword ptr ds:\[87A2CF\],27DB6E9  
008A207C     66:C705 D3A28700 00FF       mov word ptr ds:\[87A2D3\],0FF00  
008A2085     E9 4D81FDFF                 jmp 0087A1D7  
008A208A     0F85 CC81FDFF               jnz 0087A25C  
008A2090     C705 CFA28700 0F8587FF      mov dword ptr ds:\[87A2CF\],FF87850F  
008A209A     66:C705 D3A28700 FFFF       mov word ptr ds:\[87A2D3\],0FFFF  
008A20A3     C705 7FA38700 E9377D02      mov dword ptr ds:\[87A37F\],27D37E9  
008A20AD     66:C705 83A38700 00FF       mov word ptr ds:\[87A383\],0FF00  
008A20B6     E9 1A82FDFF                 jmp 0087A2D5  
008A20BB     0F85 7882FDFF               jnz 0087A339  
008A20C1     C705 7FA38700 0F85B4FF      mov dword ptr ds:\[87A37F\],FFB4850F  
008A20CB     66:C705 83A38700 FFFF       mov word ptr ds:\[87A383\],0FFFF  
008A20D4     C705 27A48700 E9C07C02      mov dword ptr ds:\[87A427\],27CC0E9  
008A20DE     66:C705 2BA48700 00FF       mov word ptr ds:\[87A42B\],0FF00  
008A20E7     E9 9982FDFF                 jmp 0087A385  
008A20EC     0F85 CC82FDFF               jnz 0087A3BE  
008A20F2     C705 27A48700 0F8591FF      mov dword ptr ds:\[87A427\],FF91850F  
008A20FC     66:C705 2BA48700 FFFF       mov word ptr ds:\[87A42B\],0FFFF  
008A2105     C705 97A58700 E9817B02      mov dword ptr ds:\[87A597\],27B81E9  
008A210F     66:C705 9BA58700 0000       mov word ptr ds:\[87A59B\],0  
008A2118     E9 1083FDFF                 jmp 0087A42D  
008A211D     C705 97A58700 8B85CC01      mov dword ptr ds:\[87A597\],1CC858B  
008A2127     C605 9BA58700 00            mov byte ptr ds:\[87A59B\],0  
008A212E     8B85 31040000               mov eax,dword ptr ss:\[ebp+431\]  
008A2134     C780 ED100300 684C218A      mov dword ptr ds:\[eax+310ED\],8A214C6>  
008A213E     66:C780 F1100300 00C3       mov word ptr ds:\[eax+310F1\],0C300  
008A2147     E9 4B84FDFF                 jmp 0087A597  
008A214C     A1 69A88700                 mov eax,dword ptr ds:\[87A869\]  
008A2151     C780 ED100300 8B857529      mov dword ptr ds:\[eax+310ED\],2975858>  
008A215B     66:C780 F1100300 4400       mov word ptr ds:\[eax+310F1\],44  
008A2164     C780 C2150300 687E218A      mov dword ptr ds:\[eax+315C2\],8A217E6>  
008A216E     66:C780 C6150300 00C3       mov word ptr ds:\[eax+315C6\],0C300  
008A2177     05 ED100300                 add eax,310ED  
008A217C     FFE0                        jmp eax  
008A217E     A1 69A88700                 mov eax,dword ptr ds:\[87A869\]  
008A2183     C780 C2150300 7508B801      mov dword ptr ds:\[eax+315C2\],1B80875  
008A218D     66:C780 C6150300 0000       mov word ptr ds:\[eax+315C6\],0  
008A2196     C680 37860100 08            mov byte ptr ds:\[eax+18637\],8  
008A219D     C680 69860100 01            mov byte ptr ds:\[eax+18669\],1  
008A21A4     C780 7A860100 68BE218A      mov dword ptr ds:\[eax+1867A\],8A21BE6>  
008A21AE     66:C780 7E860100 00C3       mov word ptr ds:\[eax+1867E\],0C300  
008A21B7     05 C2150300                 add eax,315C2  
008A21BC     FFE0                        jmp eax  
008A21BE     8B1D 69A88700               mov ebx,dword ptr ds:\[87A869\]  
008A21C4     C683 37860100 02            mov byte ptr ds:\[ebx+18637\],2  
008A21CB     C683 69860100 04            mov byte ptr ds:\[ebx+18669\],4  
008A21D2     C783 7A860100 8BD850E8      mov dword ptr ds:\[ebx+1867A\],E850D88>  
008A21DC     66:C783 7E860100 4A01       mov word ptr ds:\[ebx+1867E\],14A  
008A21E5     C780 50010000 00204A00      mov dword ptr ds:\[eax+150\],4A2000  
008A21EF     66:C780 90030000 0010       mov word ptr ds:\[eax+390\],1000  
008A21F8     66:C780 98030000 0000       mov word ptr ds:\[eax+398\],0  
008A2201     C780 5A851900 0F85ADFF      mov dword ptr ds:\[eax+19855A\],FFAD85>  
008A220B     66:C780 5E851900 FFFF       mov word ptr ds:\[eax+19855E\],0FFFF  
008A2214     C783 B9CA0100 682F228A      mov dword ptr ds:\[ebx+1CAB9\],8A222F6>  
008A221E     66:C783 BDCA0100 00C3       mov word ptr ds:\[ebx+1CABD\],0C300  
008A2227     81C3 7A860100               add ebx,1867A  
008A222D     FFE3                        jmp ebx  
008A222F     A1 69A88700                 mov eax,dword ptr ds:\[87A869\]  
008A2234     C780 B9CA0100 508B4704      mov dword ptr ds:\[eax+1CAB9\],4478B50  
008A223E     66:C780 BDCA0100 FFD0       mov word ptr ds:\[eax+1CABD\],0D0FF  
008A2247     C680 84A50000 EB            mov byte ptr ds:\[eax+A584\],0EB  
008A224E     C680 CAE10100 EB            mov byte ptr ds:\[eax+1E1CA\],0EB  
008A2255     C780 67E20100 E9890000      mov dword ptr ds:\[eax+1E267\],89E9  
008A225F     C605 B4204C00 01            mov byte ptr ds:\[4C20B4\],1  
008A2266     C705 1D466800 60DC2100      mov dword ptr ds:\[68461D\],21DC60  
008A2270     68 04208A00                 push 8A2004  
008A2275     A1 69A88700                 mov eax,dword ptr ds:\[87A869\]  
008A227A     05 BACA0100                 add eax,1CABA  
008A227F     FFE0                        jmp eax  
008A2281     BA 04208A00                 mov edx,8A2004  
008A2286     E9 0D31B6FF                 jmp 00405398  
  
从 OllyDBG 中二进制代码复制如下：  
10 00 00 00 66 6C 79 20 5B 32 30 30 35 2E 31 32 2E 30 31 5D 00 00 00 00 00 00 00 00 00 00 00 00  
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  
0F 85 C7 80 FD FF C7 05 5A A1 87 00 0F 85 AD FF 66 C7 05 5E A1 87 00 FF FF C7 05 D8 A1 87 00 8C  
7E 02 00 E9 F8 80 FD FF C7 05 D8 A1 87 00 3A 00 00 00 C7 05 CF A2 87 00 E9 B6 7D 02 66 C7 05 D3  
A2 87 00 00 FF E9 4D 81 FD FF 0F 85 CC 81 FD FF C7 05 CF A2 87 00 0F 85 87 FF 66 C7 05 D3 A2 87  
00 FF FF C7 05 7F A3 87 00 E9 37 7D 02 66 C7 05 83 A3 87 00 00 FF E9 1A 82 FD FF 0F 85 78 82 FD  
FF C7 05 7F A3 87 00 0F 85 B4 FF 66 C7 05 83 A3 87 00 FF FF C7 05 27 A4 87 00 E9 C0 7C 02 66 C7  
05 2B A4 87 00 00 FF E9 99 82 FD FF 0F 85 CC 82 FD FF C7 05 27 A4 87 00 0F 85 91 FF 66 C7 05 2B  
A4 87 00 FF FF C7 05 97 A5 87 00 E9 81 7B 02 66 C7 05 9B A5 87 00 00 00 E9 10 83 FD FF C7 05 97  
A5 87 00 8B 85 CC 01 C6 05 9B A5 87 00 00 8B 85 31 04 00 00 C7 80 ED 10 03 00 68 4C 21 8A 66 C7  
80 F1 10 03 00 00 C3 E9 4B 84 FD FF A1 69 A8 87 00 C7 80 ED 10 03 00 8B 85 75 29 66 C7 80 F1 10  
03 00 44 00 C7 80 C2 15 03 00 68 7E 21 8A 66 C7 80 C6 15 03 00 00 C3 05 ED 10 03 00 FF E0 A1 69  
A8 87 00 C7 80 C2 15 03 00 75 08 B8 01 66 C7 80 C6 15 03 00 00 00 C6 80 37 86 01 00 08 C6 80 69  
86 01 00 01 C7 80 7A 86 01 00 68 BE 21 8A 66 C7 80 7E 86 01 00 00 C3 05 C2 15 03 00 FF E0 8B 1D  
69 A8 87 00 C6 83 37 86 01 00 02 C6 83 69 86 01 00 04 C7 83 7A 86 01 00 8B D8 50 E8 66 C7 83 7E  
86 01 00 4A 01 C7 80 50 01 00 00 00 20 4A 00 66 C7 80 90 03 00 00 00 10 66 C7 80 98 03 00 00 00  
00 C7 80 5A 85 19 00 0F 85 AD FF 66 C7 80 5E 85 19 00 FF FF C7 83 B9 CA 01 00 68 2F 22 8A 66 C7  
83 BD CA 01 00 00 C3 81 C3 7A 86 01 00 FF E3 A1 69 A8 87 00 C7 80 B9 CA 01 00 50 8B 47 04 66 C7  
80 BD CA 01 00 FF D0 C6 80 84 A5 00 00 EB C6 80 CA E1 01 00 EB C7 80 67 E2 01 00 E9 89 00 00 C6  
05 B4 20 4C 00 01 C7 05 1D 46 68 00 60 DC 21 00 68 04 20 8A 00 A1 69 A8 87 00 05 BA CA 01 00 FF  
E0 BA 04 20 8A 00 E9 0D 31 B6 FF 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  
  
  
**Game Over**  
—————————————————————————————————                               
         ,     \_/   
        /| \_.-~/            \\\_     ,        青春都一晌  
       ( /~   /              \\~-.\_ |\\  
       \`\\\\  \_/                \\   ~\\ )          忍把浮名   
   \_-~~~-.)  )\_\_/;;,.          \\\_  //'  
  /'\_,\\   --~   \\ ~~~-  ,;;\\\_\_\_(  (.-~~~-.        换了脱壳轻狂  
 \`~ \_( ,\_..--\\ (     ,;'' /    ~--   /.\_\`\\   
  /~~//'   /' \`~\\         ) /--..\_, )\_  \`~  
  "  \`~"  "      \`"      /~'\`\\    \`\\\\~~\\     
                         "     "   "~'  ""  
      
              UnPacKed By :  fly  
               2005-12-02 24:00 

 4dd466d0e1e0cc123afbe504da2462ca