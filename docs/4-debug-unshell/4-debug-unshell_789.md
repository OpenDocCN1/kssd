

# Obsidium V1.2 加壳的记事本脱壳

> 原文：[`www.pediy.com/kssd/pediy06/pediy6744.htm`](https://www.pediy.com/kssd/pediy06/pediy6744.htm)

Obsidium V1.2 加壳的记事本脱壳  
  
【脱文标题】 Obsidium V1.2 加壳的记事本脱壳  
  
【脱文作者】 yeyu0808  
  
【使用工具】 flyOD,Lordpe,PEID,ImportREC1.42  
  
【破解平台】 WinXp  
  
【软件名称】 notepad  
       
【加壳方式】 Obsidium V1.2  
  
【实例下载】 点击此处本地下载 或 [`bbs.pediy.com/showthread.php?s=&threadid=7212`](http://bbs.pediy.com/showthread.php?s=&threadid=7212)  
  
  
【脱壳内容】 Obs1.2 版的壳也出来这么久了，一直没人愿意放出脱壳方法，我在这里献丑了。这次 OBS 的  
  
升级没有什么太大的变化，加壳的程序跟 1.1 版的脱壳方法相似，还是第 4 次除零异常后处理 IAT，连代码  
  
都没有太大的变化，找 OEP 跟 1.1 版基本上是一模一样，只是在 IAT 加密的时候，检测内存访问断点，断不  
  
到 1.1 版直接加密 IAT 的地方，这就为找特殊函数设了一点障碍，靠你自己去研究了~！  
  
  
用 Obsidium V1.2 版加壳的记事本，难度就大降低了，我是菜鸟，太难的搞不定，只能脱个记事本~！  
  
  
  
  
一、寻找 OEP、Dump 进程  
  
  
老规矩：用 IsDebug 1.4 插件去掉 Ollydbg 的调试器标志。忽略除了“整数被除零”之外的所有其他异常选  
项，同时还要用上 UnhandledExceptionFilter2.20，F9 后出现一个内存异常，不管它，直接 Shift+F9。  
  
  
00407000 > /EB 02               jmp short NOTEPAD.00407004    载入后停在这  
003A120F    F7F0                div eax                        1 次  
003A42E5    F7F0                div eax                        2 次  
003A4413    F7F0                div eax                        3 次  
003A42E5    F7F0                div eax                        4 次，处理 IAT 的地方  
003A2C8E    F7F0                div eax                        5 次  
003A30B9    F7F0                div eax                        6 次  
003A2C8E    F7F0                div eax                        7 次  
003A30B9    F7F0                div eax                        8 次  
0040891C    F7F0                div eax                        9 次  跳 OEP 的地方  
  
  
OK，当第 5 次 003A2C8E 异常时，Ctrl+G:003A2C8E (第 4 次异常地址)   
  
到达 003A2C8E 时 Ctrl+F 搜索命令：test word ptr ds:\[esi\],20   
  
  
003A49BF    66:F706 2000        test word ptr ds:\[esi\],20//找到这里  
  
  
记下这个：003A64F0  地址，IAT 的解密还要靠它。  
  
  
9 次时的堆栈：  
0012FF4C    0012FFE0  指针到下一个 SEH 记录  
0012FF50    00408951  SE 句柄  
  
去 00408951 下断吧：  
00408951    C8 000000           enter 0,0  
00408955    EB 03               jmp short NOTEPAD.0040895A  
  
  
Ctrl+F 在当前位置下搜索命令：mov dword ptr ds:\[eax+0B8\],edx 在 004089DF 处：  
查找 CONTEXT 结构，中间会返回到系统中去。  
  
004089DF    8990 B8000000       mov dword ptr ds:\[eax+B8\],edx                   ;   
  
NOTEPAD.00408C77  
004089E5    EB 02               jmp short NOTEPAD.004089E9  
  
此时 edx=00408C77，去 00408C77 下断，F9 断下来：  
  
00408C77    E8 BB000000         call NOTEPAD.00408D37  
//加密 CALL，跟进去吧  
  
00408D37    F8                  clc  
00408D38    73 01               jnb short NOTEPAD.00408D3B  
00408D3A    59                  pop ecx  
00408D3B    60                  pushad  
00408D3C    EB 02               jmp short NOTEPAD.00408D40  
00408D3E    FC                  cld  
00408D3F    D6                  salc  
00408D40    836C24 20 05        sub dword ptr ss:\[esp+20\],5  
00408D45    EB 03               jmp short NOTEPAD.00408D4A  
00408D47    CF                  iretd  
00408D48    D4 1B               aam 1B  
00408D4A    8B4424 20           mov eax,dword ptr ss:\[esp+20\]  
00408D4E    C600 CD             mov byte ptr ds:\[eax\],0CD  
00408D51    C740 01 023B1DCD    mov dword ptr ds:\[eax+1\],CD1D3B02  
00408D58    EB 01               jmp short NOTEPAD.00408D5B  
00408D5A    65:B9 C0000000      mov ecx,0C0  
00408D60    F9                  stc  
00408D61    72 03               jb short NOTEPAD.00408D66  
00408D63    9E                  sahf  
00408D64    8480 BECD3A19       test byte ptr ds:\[eax+193ACDBE\],al  
00408D6A    66:8000 1E          add byte ptr ds:\[eax\],1E  
00408D6E    EB 02               jmp short NOTEPAD.00408D72  
00408D70    0108                add dword ptr ds:\[eax\],ecx  
00408D72    83C0 04             add eax,4  
00408D75    F9                  stc  
00408D76    72 02               jb short NOTEPAD.00408D7A  
00408D78    2E:4B               dec ebx  
00408D7A    C1CE D8             ror esi,0D8  
00408D7D    EB 02               jmp short NOTEPAD.00408D81  
00408D7F    1950 83             sbb dword ptr ds:\[eax\-7D\],edx  
00408D82  - E9 040F85E1         jmp E1C59C8B  
00408D87    FFFF                ???                                             ; 未知命令  
00408D89    FFEB                jmp far ebx                                     ; 非法使用寄  
  
存器  
00408D8B    04 F0               add al,0F0  
00408D8D    9C                  pushfd  
00408D8E    8CD3                mov bx,ss  
00408D90    61                  popad  
00408D91    C3                  retn  
  
F7 走过上面的部分：  
00408C77   /EB 02               jmp short NOTEPAD.00408C7B  
  
再次来到 00408C77 处时，Ctrl+F 在 当前位置下 搜索命令： jmp edi 在 00408D15 处，当然是 F2 再下断了  
  
，F9 会断在那里的  
  
  
00408D15    FFE7                jmp edi  
  
  
跟进后，一直 F7：  
003A84FC  - E9 D28B5600         jmp 009110D3         这里去伪 OEP  
003A8501    EB 02               jmp short 003A8505  
  
009110D3    FF15 E4639100       call dword ptr ds:\[9163E4\]  
009110D9    8BF0                mov esi,eax  
009110DB    8A00                mov al,byte ptr ds:\[eax\]  
  
补上 Stolen Code,脱其他 Obsidium 壳程序可以根据程序运行代码、伪 OEP 处的堆栈和寄存器等情况，还原  
  
OEP 处代码。  
  
009110CC    55                  push ebp  
009110CD    8BEC                mov ebp,esp  
009110CF    83EC 44             sub esp,44  
009110D2    56                  push esi  
009110D3    FF15 E4639100       call dword ptr ds:\[9163E4\]  
  
LordPE 先完全 Dump 这个进程，然后“区域脱壳”，地址=00910000，大小=00008000，得到  
Region00910000-00918000.dmp 文件。用 LordPE 打开 dumped.exe，从磁盘载入 Region00910000-  
  
00918000.dmp 区段，改其 VOffset=00510000。只保留 LordPE 的“验证 PE”选项，对 dumped.exe 重建 PE。OK  
  
，Dump 完毕！  
  
  
二、避开 IAT 加密，得到正确的输入表   
  
  
Ctrl+F2 重新来过，到达第 4 次异常的时候：  
  
003A42E5    F7F0                div eax  
  
Ctrl+G  003A49BF，下内存写入断点，注意不要下内存访问断点，不然会长时间异常没有反应，运行 12 次  
后到达第 5 次除零异常，所在在第 11 次的时候停下来，同时在转存中 Ctrl+G 003A49BF,观察 003A49BF 处的  
  
解码情况：  
  
003A4F43    C009 F9             ror byte ptr ds:\[ecx\],0F9   停在这  
003A4F46    F9                  stc  
003A4F47    72 05               jb short 003A4F4E  
003A4F49    BA A19F0F3F         mov edx,3F0F9FA1  
003A4F4E    F8                  clc  
003A4F4F    73 06               jnb short 003A4F57  
003A4F51    EB 6B               jmp short 003A4FBE  
003A4F53    D6                  salc  
003A4F54    D4 F3               aam 0F3  
003A4F56    D5 83               aad 83  
003A4F58    C104F8 73           rol dword ptr ds:\[eax+edi\*8\],73  
003A4F5C    04 31               add al,31  
003A4F5E  ^ 7C EB               jl short 003A4F4B  
003A4F60    D5 81               aad 81  
003A4F62    C0C0 FA             rol al,0FA  
003A4F65    BD 36F97203         mov ebp,372F936  
003A4F6A    3B8B 9283EB04       cmp ecx,dword ptr ds:\[ebx+4EB8392\]  
003A4F70  ^ 0F85 ACFFFFFF       jnz 003A4F22  
003A4F76    61                  popad  
003A4F77    C3                  retn  
  
当我们走到 003A4F76 的时候，发现转存 003A49BF 处变成了 66 F7 06 20 00，也就是：  
  
003A49BF    66:F706 2000        test word ptr ds:\[esi\],20  
  
程序开始对 003A49BF 解码，要开始处理 IAT 了，好，我们 Ctrl+G  003A49BF:  
  
003A49BF    66:F706 2000        test word ptr ds:\[esi\],20  
//改成 test word ptr ds:\[esi\],8  
003A49C4    74 46               je short 003A4A0C  
//改成 jnz short 003A4A0C  
003A49C6    66:F706 0200        test word ptr ds:\[esi\],2  
003A49CB    75 1F               jnz short 003A49EC  
003A49CD    66:C706 0400        mov word ptr ds:\[esi\],4  
003A49D2    8B45 14             mov eax,dword ptr ss:\[ebp+14\]  
003A49D5    6A 01               push 1  
003A49D7    6A 00               push 0  
003A49D9    FF76 04             push dword ptr ds:\[esi+4\]  
003A49DC    6A 00               push 0  
003A49DE    FF75 18             push dword ptr ss:\[ebp+18\]  
003A49E1    FF50 50             call dword ptr ds:\[eax+50\]  
003A49E4    85C0                test eax,eax  
003A49E6    74 39               je short 003A4A21  
//改成  je short 003A4A0C  
003A49E8    8907                mov dword ptr ds:\[edi\],eax  
003A49EA    EB 20               jmp short 003A4A0C  
003A49EC    66:C706 0400        mov word ptr ds:\[esi\],4  
003A49F1    8B45 14             mov eax,dword ptr ss:\[ebp+14\]  
003A49F4    0FB756 02           movzx edx,word ptr ds:\[esi+2\]  
003A49F8    6A 01               push 1  
003A49FA    52                  push edx  
003A49FB    6A 00               push 0  
003A49FD    FF76 04             push dword ptr ds:\[esi+4\]  
003A4A00    FF75 18             push dword ptr ss:\[ebp+18\]  
003A4A03    FF50 50             call dword ptr ds:\[eax+50\]  
003A4A06    85C0                test eax,eax  
////改成  je short 003A4A0C  
003A4A08    74 17               je short 003A4A21  
003A4A0A    8907                mov dword ptr ds:\[edi\],eax  
003A4A0C    83C6 08             add esi,8  
003A4A0F    83C7 04             add edi,4  
003A4A12    FF4D 08             dec dword ptr ss:\[ebp+8\]  
003A4A15  ^ 75 A8               jnz short 003A49BF  
003A4A17    33C0                xor eax,eax  
003A4A19    40                  inc eax  
003A4A1A    5F                  pop edi  
003A4A1B    5E                  pop esi  
003A4A1C    5B                  pop ebx  
003A4A1D    C9                  leave  
003A4A1E    C2 1400             retn 14  
003A4A21    33C0                xor eax,eax  
003A4A23    5F                  pop edi  
003A4A24    5E                  pop esi  
003A4A25    5B                  pop ebx  
003A4A26    C9                  leave  
003A4A27    C2 1400             retn 14  
003A4A2A    C8 000000           enter 0,0  
003A4A2E    53                  push ebx  
003A4A2F    56                  push esi  
003A4A30    57                  push edi  
003A4A31    8B5D 14             mov ebx,dword ptr ss:\[ebp+14\]  
003A4A34    8B75 10             mov esi,dword ptr ss:\[ebp+10\]  
003A4A37    8B7D 0C             mov edi,dword ptr ss:\[ebp+C\]  
003A4A3A    8B5B 04             mov ebx,dword ptr ds:\[ebx+4\]  
003A4A3D    66:833E 08          cmp word ptr ds:\[esi\],8  
003A4A41    0F85 97000000       jnz 003A4ADE  
003A4A47    8B46 04             mov eax,dword ptr ds:\[esi+4\]    
//根据 EAX 值选择不同的 CALL 解码特殊函数  
003A4A4A    83F8 00             cmp eax,0  
003A4A4D    74 44               je short 003A4A93  
003A4A4F    83F8 01             cmp eax,1  
003A4A52    74 4E               je short 003A4AA2  
003A4A54    83F8 02             cmp eax,2  
003A4A57    74 58               je short 003A4AB1  
003A4A59    83F8 03             cmp eax,3  
003A4A5C    74 11               je short 003A4A6F  
003A4A5E    83F8 04             cmp eax,4  
003A4A61    75 7B               jnz short 003A4ADE  
003A4A63    8B45 18             mov eax,dword ptr ss:\[ebp+18\]  
003A4A66    05 1F9E4100         add eax,419E1F  
003A4A6B    8907                mov dword ptr ds:\[edi\],eax  
003A4A6D    EB 6F               jmp short 003A4ADE  
003A4A6F    8B45 14             mov eax,dword ptr ss:\[ebp+14\]  
003A4A72    68 C5B1662D         push 2D66B1C5  
003A4A77    6A 00               push 0  
003A4A79    FF50 18             call dword ptr ds:\[eax+18\]   这个 CALL 是处理特殊函数的  
003A4A7C    50                  push eax  
003A4A7D    53                  push ebx  
003A4A7E    E8 EC030000         call 003A4E6F  
003A4A83    53                  push ebx  
003A4A84    E8 5B030000         call 003A4DE4  
003A4A89    8BCB                mov ecx,ebx  
003A4A8B    8D5C03 01           lea ebx,dword ptr ds:\[ebx+eax+1\]  
003A4A8F    8BC1                mov eax,ecx  
003A4A91    EB 2B               jmp short 003A4ABE  
003A4A93    8B45 14             mov eax,dword ptr ss:\[ebp+14\]  
003A4A96    68 0F1ACF4C         push 4CCF1A0F  
003A4A9B    6A 00               push 0  
003A4A9D    FF50 18             call dword ptr ds:\[eax+18\]    这个 CALL 是处理特殊函数的  
003A4AA0    EB 1C               jmp short 003A4ABE  
003A4AA2    8B45 14             mov eax,dword ptr ss:\[ebp+14\]  
003A4AA5    68 A41A86D0         push D0861AA4  
003A4AAA    6A 00               push 0  
003A4AAC    FF50 18             call dword ptr ds:\[eax+18\]   这个 CALL 是处理特殊函数的  
003A4AAF    EB 0D               jmp short 003A4ABE  
003A4AB1    8B45 14             mov eax,dword ptr ss:\[ebp+14\]  
003A4AB4    68 E313B41D         push 1DB413E3  
003A4AB9    6A 00               push 0  
003A4ABB    FF50 18             call dword ptr ds:\[eax+18\]   这个 CALL 是处理特殊函数的  
003A4ABE    C603 B8             mov byte ptr ds:\[ebx\],0B8  
003A4AC1    8943 01             mov dword ptr ds:\[ebx+1\],eax  
003A4AC4    8B55 18             mov edx,dword ptr ss:\[ebp+18\]  
003A4AC7    81C2 7A9C4100       add edx,419C7A  
003A4ACD    8D43 0A             lea eax,dword ptr ds:\[ebx+A\]  
003A4AD0    2BD0                sub edx,eax  
003A4AD2    C643 05 E9          mov byte ptr ds:\[ebx+5\],0E9  
003A4AD6    8953 06             mov dword ptr ds:\[ebx+6\],edx  
003A4AD9    891F                mov dword ptr ds:\[edi\],ebx  
003A4ADB    83C3 0A             add ebx,0A  
003A4ADE    83C6 08             add esi,8  
003A4AE1    83C7 04             add edi,4  
003A4AE4    8B45 14             mov eax,dword ptr ss:\[ebp+14\]  
003A4AE7    8958 04             mov dword ptr ds:\[eax+4\],ebx  
003A4AEA    FF4D 08             dec dword ptr ss:\[ebp+8\]  
003A4AED  ^ 0F85 4AFFFFFF       jnz 003A4A3D  
003A4AF3    5F                  pop edi  
003A4AF4    5E                  pop esi  
003A4AF5    5B                  pop ebx  
003A4AF6    C9                  leave  
003A4AF7    C2 1400             retn 14     
  
  
特殊函数我没有跟踪，偷了点懒，如果有兴趣的可以在那个几个 CALL 上下硬件执  
  
行断点，我跟过了确实能断下来，进入 CALL 后就可以得到你的函数了，当改完上  
  
面需要改的地方后，F9 运行，到达第 5 次除零异常。  
  
  
可以用 Import REC 收集 IAT 了，有三个特殊函数，对照了一下原记事本的 IAT，用  
  
插件也可以认出来，不过有一个认不出，有一个认错了。  
  
1  005163A0  kernel32.dll  00AC  ExitProcess  插件认错了  
  
1  005163A4  kernel32.dll  039E  lstrlen      插件没认出  
  
其他是填充的垃圾数据，CUT 掉就行了。改 OEP＝005110CC，FixDump！  
  
删除 00007000 区段，重建 PE 优化一下 dumped\_.exe 即可。  
  
  
  
【脱壳后记】 一开始碰到这个壳不知道怎么脱，发了贴也没人愿意回答，哎，  
  
心寒~！不过这里真的很感谢 stephenteh 和 fly 两位大哥（文中有些话是用的 fly  
  
  
大哥的以前的脱 1.1 版的话，还请 fly 大哥海涵）是他们指导了我怎么脱这个壳，  
  
还有二哥的文章带我入了门。感谢曾帮助过我的人，还有谢谢您耐心看完此文，  
  
有什么不对的地方，还请大家指正~！

 4dd466d0e1e0cc123afbe504da2462ca