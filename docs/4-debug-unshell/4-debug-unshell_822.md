

# 很久以前的一个老壳 \[高手就不用看了\]

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-594.htm`](https://www.pediy.com/kssd/pediy08/pediy8-594.htm)

//前段时间测试破文生成器时写的一个文章，没什么价值,希望对新手有所帮助  
【文章标题】: MTV 电子相册脱壳  
【文章作者】: clide2000\[DFCG\]  
【作者邮箱】: 54arma@sina.com  
【软件名称】: MTV 电子相册  
【软件大小】: 16.6MB  
【下载地址】: 自己搜索下载  
【加壳方式】: hying 壳  
【使用工具】: OD 修改版,LordPe,ImportREC16  
【操作平台】: Win2000  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
    
  一快速定位 OEP  
  OD 载入后停在(忽略所有异常，隐藏 OD):  
  0071F000 >  55                 push ebp  
  0071F001    8BEC               mov ebp, esp  
  0071F003    6A FF              push -1  
  0071F005    68 2A2C0A00        push 0A2C2A  
  0071F00A    68 38900D00        push 0D9038  
  0071F00F    64:A1 00000000     mov eax, dword ptr fs:\[0\]  
    
  下 bp VirtualAlloc 断点，F9 运行，中断在:  
  77E64DB9 >  55                 push ebp                                   ; MTVAlbum.  
  77E64DBA    8BEC               mov ebp, esp  
  77E64DBC    FF75 14            push dword ptr ss:\[ebp+14\]  
  77E64DBF    FF75 10            push dword ptr ss:\[ebp+10\]  
  77E64DC2    FF75 0C            push dword ptr ss:\[ebp+C\]  
  77E64DC5    FF75 08            push dword ptr ss:\[ebp+8\]  
  77E64DC8    6A FF              push -1  
  77E64DCA    E8 04000000        call KERNEL32.VirtualAllocEx  
  77E64DCF    5D                 pop ebp  
  77E64DD0    C2 1000            retn 10  
    
  Alt+F9 返回后：  
  0071F1F6    50                 push eax  
  0071F1F7    8B9D E0000000      mov ebx, dword ptr ss:\[ebp+E0\]  
  0071F1FD    03DD               add ebx, ebp  
  0071F1FF    8BCD               mov ecx, ebp  
  0071F201    50                 push eax  
  0071F202    53                 push ebx  
  0071F203    E8 06000000        call MTVAlbum.0071F20E  
  0071F208    5A                 pop edx  
  0071F209    8BC2               mov eax, edx  
  0071F20B    51                 push ecx  
  0071F20C    FFE0               jmp eax  
  0071F20E    60                 pushad  
    
  现在看看 EAX 的值，这是申请到的内存空间的首地址.Ctrl+G,输入 EAX 的内容，我这里是:340000  
  来到刚申请好的内存空间：  
  00340000    0000               add byte ptr ds:\[eax\], al  
  00340002    0000               add byte ptr ds:\[eax\], al  
  00340004    0000               add byte ptr ds:\[eax\], al  
  00340006    0000               add byte ptr ds:\[eax\], al  
  00340008    0000               add byte ptr ds:\[eax\], al  
  0034000A    0000               add byte ptr ds:\[eax\], al  
  0034000C    0000               add byte ptr ds:\[eax\], al  
    
  在 00340000 上下硬件执行断点，即 he 340000,F9 运行中断在这里：  
  00340000    E8 05000000        call 0034000A  
  00340005    0F01               ???                                        ; Unknown command  
  00340007    EB 05              jmp short 0034000E  
  00340009    E8 EBFB0000        call 0034FBF9  
  0034000E    83C4 04            add esp, 4  
  00340011    B9 04000000        mov ecx, 4  
  00340016    E8 1F000000        call 0034003A  
  0034001B  ^ EB FA              jmp short 00340017  
    
  此时，壳的解压代码已经全部写到这段内存空间中了。  
  现在 Ctrl+B,搜索二进制 E7 BB EC 3E 00 FF E0 66 9C，找到：  
  003439E2    E7 BB             out 0BB, eax                               ; I/O command  
  003439E4    EC                in al, dx                                  ; I/O command  
  003439E5    3E:00FF           add bh, bh                                 ; Superfluous prefix  
  003439E8    E0 66             loopdne short 00343A50    //注意这里  
  003439EA    9C                pushfd  
    
  因为有花指令的干扰，所以在这里我们用 Ctrl+G,然后输入 003439E8-1 的地址，即 003439E7，会年到：  
  003439E7    FFE0               jmp eax  
  003439E9    66:9C              pushfw  
    
  现在取消之前所有的断点，在 003439E7 上下 F2 断点，F9 运行后中断在这：  
  003439E7  - FFE0               jmp eax      ; MTVAlbum.006440C8   //这就是 OEP 了  
  003439E9    66:9C              pushfw  
    
  取消断点，F7 后来到 OEP 006440C8:  
  006440C8    55                 push ebp  
  006440C9    8BEC               mov ebp, esp  
  006440CB    83C4 F0            add esp, -10  
  006440CE    53                 push ebx  
  006440CF    B8 A8396400        mov eax, MTVAlbum.006439A8  
  006440D4    E8 7B2EDCFF        call MTVAlbum.00406F54  
  006440D9    8B1D A4636600      mov ebx, dword ptr ds:\[6663A4\]        ; MTVAlbum.00667BF8  
  006440DF    8B03               mov eax, dword ptr ds:\[ebx\]  
  006440E1    E8 722EE4FF        call MTVAlbum.00486F58  
  006440E6    8B03               mov eax, dword ptr ds:\[ebx\]  
    
  现在可以在 006440C8 上下硬件执行断点，这样方便我们下次载入运行时直接来到 OEP  
    
  二修复  
  来到 OEP 后，按 F7 单步执行，进入到 006440D4  call MTVAlbum.00406F54:  
  00406F54    53                 push ebx  
  00406F55    8BD8               mov ebx, eax  
  00406F57    33C0               xor eax, eax  
  00406F59    A3 C4506400        mov dword ptr ds:\[6450C4\], eax  
  00406F5E    6A 00              push 0  
  00406F60    E8 2BFFFFFF        call MTVAlbum.00406E90  
  00406F65    A3 68766600        mov dword ptr ds:\[667668\], eax  
  00406F6A    A1 68766600        mov eax, dword ptr ds:\[667668\]  
  00406F6F    A3 D0506400        mov dword ptr ds:\[6450D0\], eax  
  00406F74    33C0               xor eax, eax  
  00406F76    A3 D4506400        mov dword ptr ds:\[6450D4\], eax  
  00406F7B    33C0               xor eax, eax  
  00406F7D    A3 D8506400        mov dword ptr ds:\[6450D8\], eax  
  00406F82    E8 C1FFFFFF        call MTVAlbum.00406F48  
  00406F87    BA CC506400        mov edx, MTVAlbum.006450CC  
  00406F8C    8BC3               mov eax, ebx  
  00406F8E    E8 9DD7FFFF        call MTVAlbum.00404730  
  00406F93    5B                 pop ebx  
  00406F94    C3                 retn  
    
    
  因为这是个 delphi 程序，所以在 00406F60   call MTVAlbum.00406E90 这句应该是:  
  00406F60    call   
    
  可是现在 F7 进入 call MTVAlbum.00406E90 后，却全变成了:  
  00406E90    90                 nop  
  00406E91    E8 A4CBF3FF        call 00343A3A  
  00406E96    8BC0               mov eax, eax  //这里先记下这个地址$$$$$$  
  00406E98    90                 nop  
  00406E99    E8 9CCBF3FF        call 00343A3A  
  00406E9E    8BC0               mov eax, eax  
  00406EA0    90                 nop  
  00406EA1    E8 94CBF3FF        call 00343A3A  
  00406EA6    8BC0               mov eax, eax  
  00406EA8    90                 nop  
  00406EA9    E8 8CCBF3FF        call 00343A3A  
  00406EAE    8BC0               mov eax, eax  
  00406EB0    50                 push eax  
  00406EB1    6A 40              push 40  
  00406EB3    E8 E0FFFFFF        call MTVAlbum.00406E98  
  00406EB8    C3                 retn  
    
  所有对 api 的调用都变成了 call 00343A3A，那我们就 F7 进入 call 00343A3A 里面看看：  
  00343A3A    50                 push eax    
  00343A3B    8BC4               mov eax, esp  
  00343A3D    60                 pushad  
  00343A3E    8BD8               mov ebx, eax  
  00343A40    E8 04000000        call 00343A49  
  00343A45    D5 2E              aad 2E  
  00343A47    34 00              xor al, 0  
  00343A49    5D                 pop ebp  
  00343A4A    8B6D 00            mov ebp, dword ptr ss:\[ebp\]  
  00343A4D    8B7B 04            mov edi, dword ptr ds:\[ebx+4\]  
  00343A50    8BB5 7C0C0000      mov esi, dword ptr ss:\[ebp+C7C\]  
  00343A56    03F5               add esi, ebp  
  00343A58    8B06               mov eax, dword ptr ds:\[esi\]  
  00343A5A    33D2               xor edx, edx  
  00343A5C    B9 02000000        mov ecx, 2  
  00343A61    F7E1               mul ecx  
  00343A63    D1E8               shr eax, 1  
  00343A65    3BF8               cmp edi, eax  
  00343A67    75 10              jnz short 00343A79  
  00343A69    0AD2               or dl, dl  
  00343A6B    75 07              jnz short 00343A74  
  00343A6D    E9 A6000000        jmp 00343B18  
  00343A72    EB 05              jmp short 00343A79  
  00343A74    E9 AA000000        jmp 00343B23  
  00343A79    83C6 08            add esi, 8  
  00343A7C    66:9C              pushfw  
  00343A7E    E8 04000000        call 00343A87  
  00343A83    0010               add byte ptr ds:\[eax\], dl  
  00343A85    40                 inc eax  
  00343A86    0083 C404EB04      add byte ptr ds:\[ebx+4EB04C4\], al  
  00343A8C    31C8               xor eax, ecx  
  00343A8E    3F                 aas  
  00343A8F    00EB               add bl, ch  
  00343A91    04 F0              add al, 0F0  
  00343A93    0F4000             cmovo eax, dword ptr ds:\[eax\]  
  00343A96    66:9D              popfw  
  00343A98    70 06              jo short 00343AA0  
  00343A9A    71 04              jno short 00343AA0  
  00343A9C    A8 C8              test al, 0C8  
  00343A9E    3F                 aas  
  00343A9F    0072 07            add byte ptr ds:\[edx+7\], dh  
  00343AA2    73 05              jnb short 00343AA9  
  00343AA4    880F               mov byte ptr ds:\[edi\], cl  
  00343AA6    E0 3E              loopdne short 00343AE6  
  00343AA8    00EB               add bl, ch  
  00343AAA    AD                 lods dword ptr ds:\[esi\]  
  00343AAB    74 08              je short 00343AB5  
  00343AAD    75 06              jnz short 00343AB5  
  00343AAF    9A 00E03D00 E876   call far 76E8:003DE000                ; Far call  
  00343AB6    07                 pop es                                ; Modification of segment register  
  00343AB7    77 05              ja short 00343ABE  
  00343AB9    E7 BB              out 0BB, eax                          ; I/O command  
  00343ABB    EC                 in al, dx                             ; I/O command  
  00343ABC    3E:0078 07         add byte ptr ds:\[eax+7\], bh  
  00343AC0    79 05              jns short 00343AC7  
  00343AC2    EA 000F41FF 669C   jmp far 9C66:FF410F00                 ; Far jump  
  00343AC9    EB 06              jmp short 00343AD1  
  00343ACB    E8 01AADB3D        call 3E0FE4D1  
  00343AD0    006A 02            add byte ptr ds:\[edx+2\], ch  
  00343AD3    EB 06              jmp short 00343ADB  
  00343AD5    E8 A0994700        call 007BD47A  
  00343ADA    9A 7311EB05 E80A   call far 0AE8:05EB1173                ; Far call  
  00343AE1    0029               add byte ptr ds:\[ecx\], ch  
  00343AE3    E9 E80C0000        jmp 003447D0  
  00343AE8    008B FF0F41FE      add byte ptr ds:\[ebx+FE410FFF\], cl  
  00343AEE  ^ 73 F4              jnb short 00343AE4  
  00343AF0    9A 20143100 83C4   call far C483:00311420                ; Far call  
  00343AF7    04 EB              add al, 0EB  
  00343AF9    04 D0              add al, 0D0  
  00343AFB    39C7               cmp edi, eax  
  00343AFD    00FF               add bh, bh  
  00343AFF    0C 24              or al, 24  
  00343B01    71 04              jno short 00343B07  
  00343B03    BA 88260079        mov edx, 79002688  
  00343B08    D27A 01            sar byte ptr ds:\[edx+1\], cl  
  00343B0B    E7 83              out 83, eax                           ; I/O command  
  00343B0D    C40466             les eax, fword ptr ds:\[esi\]           ; Modification of segment register  
  00343B10    9D                 popfd  
  00343B11    EB 05              jmp short 00343B18  
  00343B13    B8 EF0F410F        mov eax, 0F410FEF  
  00343B18    8B46 04            mov eax, dword ptr ds:\[esi+4\]   //在这里下 F2 断点  
  00343B1B    8903               mov dword ptr ds:\[ebx\], eax  
  00343B1D    61                 popad  
  00343B1E    58                 pop eax  
  00343B1F    8B00               mov eax, dword ptr ds:\[eax\]  
  00343B21    FFE0               jmp eax  
  00343B23    8B46 04            mov eax, dword ptr ds:\[esi+4\]   //在这里下 F2 断点  
  00343B26    8903               mov dword ptr ds:\[ebx\], eax  
  00343B28    61                 popad  
  00343B29    58                 pop eax  
  00343B2A    83C4 04            add esp, 4  
  00343B2D    8B00               mov eax, dword ptr ds:\[eax\]  
  00343B2F    FFE0               jmp eax  
    
  经跟踪分析，这是查表找到对应的 API，然后由 jmp eax 跑到 API 执行。  
  当我们在两句 mov eax, dword ptr ds:\[esi+4\] 都下好 F2 断点后，F9 运行，会发现这次我们中断在了  
  00343B23    8B46 04            mov eax, dword ptr ds:\[esi+4\]     
    
  信息窗口内容是:  
  ds:\[00343E1E\]=0067535C (MTVAlbum.0067535C)  
  eax=00406E96 (MTVAlbum.00406E96)  
    
  在数据窗口跟踪 00343E1E 得到如下一个很有规律的对应表：  
  00343E1A  96 6E 40 80  //这个就是在上面让大家记住的地址$$$$$$  
  00343E1E  5C 53 67 00  //而这里是什么，下面就看到了  
  00343E22  82 71 40 80    
  00343E26  64 53 67 00   
  00343E2A  7A 71 40 80    
  00343E2E  68 53 67 00   
  00343E32  72 71 40 80  
    
  现在上下找找这张表的开始和结束的地址：   //先记下，一会写修复代码时会用到  
  开始地址: 343C2A   
  结束地址: 344EF2   
    
    
  当 F7 来到 00343B2D 时:  
  00343B26    8903               mov dword ptr ds:\[ebx\], eax  
  00343B28    61                 popad  
  00343B29    58                 pop eax  
  00343B2A    83C4 04            add esp, 4  
  00343B2D    8B00               mov eax, dword ptr ds:\[eax\]  
  00343B2D    8B00               mov eax, dword ptr ds:\[eax\]         ; KERNEL32.GetModuleHandleA  
       
  信息窗口内容:  
  ds:\[0067535C\]=77E63DFC (KERNEL32.GetModuleHandleA)  
  eax=0067535C (MTVAlbum.0067535C)  
    
  在数据窗口跟踪 0067535C 得到一个很完整的 IAT 表。。如下   
  0067535C  kernel32.dll  013F  GetModuleHandleA  
  00675360  00000000  
  00675364  advapi32.dll  01AF  RegQueryValueExA  
  00675368  advapi32.dll  01A5  RegOpenKeyExA  
  0067536C  advapi32.dll  018C  RegCloseKey  
  00675374  00000000  
  00675374  kernel32.dll  0335  lstrcpy  
  00675378  kernel32.dll  0332  lstrcmpi  
  0067537C  kernel32.dll  032F  lstrcmp  
    
  现在上下找找 IAT 开始地址和大小：   //一会修复时用 ImportREC 取 IAT 时会用到  
  开始地址: 343C2A   
   344EF2   
    
  当执行到:    
  00343B2F    FFE0               jmp eax    // 跳到函数去了。  
    
  下面我们来分析下这个对照表的关系：  
  00406E90    90                 nop  
  00406E91    E8 A4CBF3FF        call 00343A3A //查表找到下面的 00406E96 地址对应的 API  
  00406E96    8BC0               mov eax, eax  //根据这里的地址来查，即 00406E96  
  00406E98    90                 nop  
    
  来看看之前找到的那张表：  
  00343E1A  96 6E 40 80  //这里是 00406E96  
  00343E1E  5C 53 67 00  //而这里就是 00406E96 对应的 API 的存放地址  
  00343E22  82 71 40 80    
  00343E26  64 53 67 00   
  00343E2A  7A 71 40 80    
  00343E2E  68 53 67 00   
  00343E32  72 71 40 80    
    
  同理 00343E22 放的地址是 00407182,他所对应的 API 就放在 00675364  
  这下子清晰了吧， 这个表就是 call 00343A3A 根据地址来得到对应的函数的对照表。  
    
    
  好了，下面让我们来修复吧:  
  还记得上次在 OEP 下过硬件执行断点吧，这下就用上了  
  Ctrl+F2 重新载入程序，F9 运行，会中断在 OEP 处.然后找个空白段写入以下代码：  
    
  这程序代码段尾部有足够的地方，就不去别处找了:(  
    
  00696EFF    90                 nop             //首先在这新建 EIP   
  00696F00    60                 pushad  
  00696F01    9C                 pushfd  
  00696F02    BB 2A3C3400        mov ebx, 343C2A  //对照表开始位置  
  00696F07    8B03               mov eax, dword ptr ds:\[ebx\]  
  00696F09    2D 00000080        sub eax, 80000000  
  00696F0E    66:C740 FA FF25    mov word ptr ds:\[eax-6\], 25FF  
  00696F14    8B4B 04            mov ecx, dword ptr ds:\[ebx+4\]  
  00696F17    8948 FC            mov dword ptr ds:\[eax-4\], ecx  
  00696F1A    83C3 08            add ebx, 8  
  00696F1D    81FB F24E3400      cmp ebx, 344EF2   //对照表结束位置  
  00696F23    7D 02              jge 00696F27  
  00696F25  ^ EB E0              jmp 00696F07  
  00696F27    9D                 popfd  
  00696F28    61                 popad  
  00696F29  ^ E9 9AD1FAFF        jmp 006440C8  //跳到 OEP  
  00696F2E    90                 nop  
    
  二进制代码是：  
  90 60 9C BB 2A 3C 34 00 8B 03 2D 00 00 00 80 66 C7 40 FA FF 25 8B 4B 04 89 48 FC 83 C3 08 81 FB  
  F2 4E 34 00 7D 02 EB E0 9D 61 E9 9A D1 FA FF 90  
    
  F9 运行后会再次中断在 OEP  
  打开 LordPE,先执行一下"correct ImageSize",纠正一下文件的大小,再来 DUMP 程序。  
  然后打开 ImportREC 选择程序 OPE：002440C8　，然后自动查找 IAT，获取输入表。  
  这会得到一个完好的表了。  
  修复之后运行正常。。。  
    
\--------------------------------------------------------------------------------  
【经验总结】  
  以前脱这种壳只会补区段，在学习了 xiaoboy 兄弟的“RM 转换精灵 V 7.1 主程序脱壳"一文后，自习了练习一下。做个笔记  
  ，另，对于 OEP 的查找，以前看 0.46 的版本时没有这么多花指令，很容易就能找到。现在虽然脱掉了这壳，但还不知道是什  
  么版本的，还请高手指点。  
    
  在此对所有的帮助过我的兄弟表示忠心的感谢！  
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
  
                                                       2006 年 02 月 27 日 10:17:59 

 4dd466d0e1e0cc123afbe504da2462ca