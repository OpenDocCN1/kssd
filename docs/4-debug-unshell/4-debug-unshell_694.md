

# forgot 的 unpackme 1.7 的简单脱壳

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-515.htm`](https://www.pediy.com/kssd/pediy07/pediy7-515.htm)

forgot 的 unpackme 1.7 的简单脱壳  
   
　　　　　日期：2005 年 3 月 6 日　　　脱壳人：csjwaman\[DFCG\]  
———————————————————————————————————————————  
   
   
【软件名称】：forgot 的 unpackme 1.7   
【下载地址】：[`www.pediy.com/bbs/pediy7/pediy7-515.rar`](http://www.pediy.com/bbs/pediy7/pediy7-515.rar)  
【脱壳声明】：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！  
【操作系统】：winxp  
【脱壳工具】：OD 等传统工具  
  
———————————————————————————————————————————  
　  
【脱壳过程】：  
  
forgot 的 unpackme 1.7 和 pack v0.1.6 的变化不大，只是加了 IAT 简单加密。另外和 1.6 一样加了入口代码修改，但修改字节并不多。  
如果看过我脱 015 版和 016 版的文章，那么可以省略相关的章节。  
  
  
第一步，找 d-process 处：  
  
用 OD 载入程序，忽略所有异常，bp WriteProcessMemory，F9 运行程序，断下：  
  
77E41A94 >  55              PUSH EBP////断下。  
77E41A95    8BEC            MOV EBP,ESP  
77E41A97    51              PUSH ECX  
77E41A98    51              PUSH ECX  
77E41A99    8B45 0C         MOV EAX,DWORD PTR SS:\[EBP+C\]  
77E41A9C    53              PUSH EBX  
  
观察堆栈数据：  
  
0032797C   0032B349  /CALL 到 WriteProcessMemory 来自 0032B344  
00327980   00000034  |hProcess = 00000034  
00327984   0032CA56  |Address = 32CA56////重要！记下备用。  
00327988   0032C3C5  |Buffer = 0032C3C5  
0032798C   0000029A  |BytesToWrite = 29A (666.)////重要！记下备用。  
00327990   00000000  \\pBytesWritten = NULL  
  
  
  
好，F9 直接运行程序，等出现界面时，运行 LordPE 选中映像文件小的那个进程，然后部分 DUMP，把从 32CA56 处开始大小为 29A 的数据 DUMP 下来，命名为 32CA56.bin。  
  
第二步，变双进程为单进程：  
  
重新载入程序，bp CreateProcessA，运行程序，断下：  
  
77E41BBC >  55              PUSH EBP////断在这儿。  
77E41BBD    8BEC            MOV EBP,ESP  
77E41BBF    6A 00           PUSH 0  
77E41BC1    FF75 2C         PUSH DWORD PTR SS:\[EBP+2C\]  
77E41BC4    FF75 28         PUSH DWORD PTR SS:\[EBP+28\]  
77E41BC7    FF75 24         PUSH DWORD PTR SS:\[EBP+24\]  
  
观察堆栈数据：  
  
00327968   0032A22C  /CALL 到 CreateProcessA 来自 0032A227  
0032796C   00328F99  |ModuleFileName = "C:\\Documents and Settings\\csjwaman\\桌面\\1.7\\1.7.exe"  
00327970   0032A21D  |CommandLine = "X"////调试标志。  
00327974   00000000  |pProcessSecurity = NULL  
00327978   00000000  |pThreadSecurity = NULL  
0032797C   00000000  |InheritHandles = FALSE  
00327980   00000003  |CreationFlags = DEBUG\_PROCESS|DEBUG\_ONLY\_THIS\_PROCESS  
00327984   00000000  |pEnvironment = NULL  
00327988   00000000  |CurrentDir = NULL  
0032798C   00327CF4  |pStartupInfo = 00327CF4  
00327990   00327CE4  \\pProcessInfo = 00327CE4  
  
  
  
重新载入程序，bp GetCommandLineA，运行程序，断下：  
  
77E5E358 >  A1 1476EB77     MOV EAX,DWORD PTR DS:\[77EB7614\]////断在这里。取消断点。  
77E5E35D    C3              RETN////返回。  
  
返回到：  
  
0032837C    68 9F6F56B6     PUSH B6566F9F  
00328381    50              PUSH EAX  
00328382    E8 5D000000     CALL 003283E4  
00328387    EB FF           JMP SHORT 00328388  
00328389    71 78           JNO SHORT 00328403  
0032838B    C2 5000         RETN 50  
0032838E  ^ EB D3           JMP SHORT 00328363  
00328390    5B              POP EBX  
00328391    F3:             PREFIX REP:       ; 多余的前缀  
  
  
  
搜索二进制字符“803e58”，找到：  
  
00328617  ^\\71 EB           JNO SHORT 00328604  
00328619  ^ EB FA           JMP SHORT 00328615  
0032861B  ^ EB 80           JMP SHORT 0032859D/////找到这里。这里有个花指令。  
0032861D    3E:58           POP EAX        
0032861F    0F84 8A410000   JE 0032C7AF  
00328625    68 9F6F56B6     PUSH B6566F9F  
0032862A    50              PUSH EAX  
0032862B    E8 5D000000     CALL 0032868D  
00328630    EB FF           JMP SHORT 00328631  
  
  
NOP 掉花指令后：  
  
0032861B    90              NOP  
0032861C    803E 58         CMP BYTE PTR DS:\[ESI\],58////58 为调试标志。用于判断是否为子进程。  
0032861F    0F84 8A410000   JE 0032C7AF////此处改为 JMP!可以变为单进程。  
00328625    68 9F6F56B6     PUSH B6566F9F  
0032862A    50              PUSH EAX  
0032862B    E8 5D000000     CALL 0032868D  
  
  
00A19217    90              NOP  
00A19218    803E 58         CMP BYTE PTR DS:\[ESI\],58  
00A1921B    0F84 CA3F0000   JE 00A1D1EB  
00A19221    68 9F6F56B6     PUSH B6566F9F  
00A19226    50              PUSH EAX  
00A19227    E8 5D000000     CALL 00A19289  
  
  
第三步，修补程序：  
  
修改上面这个跳转后，用十六进制工具把 32CA56.bin 的数据复盖掉 OD 的 DUMP 区从 0032CA56 处开始的 29A 个字节数据。  
  
第四步，查找入口：  
  
做完上述工作后，在 OD 的 CPU 窗口，Ctrl+G,输入 0032CA56，点确定后来到：  
  
0032CA4D  ^\\71 EB           JNO SHORT 0032CA3A  
0032CA4F  ^ EB FA           JMP SHORT 0032CA4B  
0032CA51  ^ EB F0           JMP SHORT 0032CA43////NOP 掉。  
0032CA53    0FC7C8          CMPXCHG8B EAX////NOP 掉。  
0032CA56    FFC3            INC EBX/////来到这里。  
0032CA58    FFC8            DEC EAX  
0032CA5A    F7D8            NEG EAX  
0032CA5C    F7C1 2DF16825   TEST ECX,2568F12D  
0032CA62    8BD9            MOV EBX,ECX  
0032CA64    01CB            ADD EBX,ECX  
0032CA66    85CB            TEST EBX,ECX  
0032CA68    8BDA            MOV EBX,EDX  
0032CA6A    C1CB F3         ROR EBX,0F3                              ; 移动常数超出 1..31 的范围  
0032CA6D    C7C3 BE948305   MOV EBX,58394BE  
0032CA73    87D8            XCHG EAX,EBX  
0032CA75    0FA5D3          SHLD EBX,EDX,CL  
0032CA78    87C0            XCHG EAX,EAX  
  
搜索二进制字符“6150C3”，找到：  
  
0032FDFE  ^\\71 EB           JNO SHORT 0032FDEB  
0032FE00  ^ EB FA           JMP SHORT 0032FDFC  
0032FE02    EB 50           JMP SHORT 0032FE54  
0032FE04    8D78 02         LEA EDI,DWORD PTR DS:\[EAX+2\]  
0032FE07    EB 02           JMP SHORT 0032FE0B////在这里 F2 下断。  
0032FE09    61              POPAD////找到这里。  
0032FE0A    50              PUSH EAX  
0032FE0B    C3              RETN////返回入口！  
  
在 0032FE07 处断下后，EAX＝403A3F，这就是入口地址。  
  
00403A3F    FFD7            CALL NEAR EDI////返回到这里。入口代码被修改了。  
00403A41    58              POP EAX  
00403A42    90              NOP  
00403A43    90              NOP  
00403A44    E8 63910000     CALL 1\_7.0040CBAC  
00403A49    85C0            TEST EAX,EAX  
00403A4B    74 26           JE SHORT 1\_7.00403A73  
00403A4D    68 00000200     PUSH 20000  
00403A52    6A 40           PUSH 40  
00403A54    E8 2F910000     CALL 1\_7.0040CB88  
00403A59    A3 C0E14000     MOV DWORD PTR DS:\[40E1C0\],EAX  
00403A5E    68 BCE04000     PUSH 1\_7.0040E0BC  
00403A63    E8 10000000     CALL 1\_7.00403A78  
  
到入口后观察堆栈区：  
  
00327D68   0040E034  1\_7.0040E034////这就是被偷的代码。  
00327D6C   0042F73C  返回到 1\_7.0042F73C  
  
补回代码后：  
  
00403A3F    68 34E04000     PUSH 1\_7.0040E034////这才象入口嘛:)  
00403A44    E8 63910000     CALL 1\_7.0040CBAC  
00403A49    85C0            TEST EAX,EAX  
00403A4B    74 26           JE SHORT 1\_7.00403A73  
00403A4D    68 00000200     PUSH 20000  
00403A52    6A 40           PUSH 40  
00403A54    E8 2F910000     CALL 1\_7.0040CB88  
00403A59    A3 C0E14000     MOV DWORD PTR DS:\[40E1C0\],EAX  
00403A5E    68 BCE04000     PUSH 1\_7.0040E0BC  
00403A63    E8 10000000     CALL 1\_7.00403A78  
00403A68    FF35 C0E14000   PUSH DWORD PTR DS:\[40E1C0\]  
00403A6E    E8 1B910000     CALL 1\_7.0040CB8E  
  
第五步，查找 IAT 加密处  
  
到入口后 F7 跟踪：  
  
00403A3F    68 34E04000     PUSH 1\_7.0040E034////F7  
00403A44    E8 63910000     CALL 1\_7.0040CBAC////F7 进入！  
00403A49    85C0            TEST EAX,EAX  
00403A4B    74 26           JE SHORT 1\_7.00403A73  
  
进入后：  
  
0040CB6A    68 F0A6E577     PUSH kernel32.CloseHandle  
0040CB6F    C3              RETN  
0040CB70    68 76B4E577     PUSH kernel32.CreateFileA  
0040CB75    C3              RETN  
0040CB76    68 FD98E577     PUSH kernel32.ExitProcess  
0040CB7B    C3              RETN  
0040CB7C    68 6416E677     PUSH kernel32.GetFileSize  
0040CB81    C3              RETN  
0040CB82    68 9BA2E577     PUSH kernel32.GetTickCount  
0040CB87    C3              RETN  
0040CB88    68 9E56E577     PUSH kernel32.GlobalAlloc  
0040CB8D    C3              RETN  
0040CB8E    68 D957E577     PUSH kernel32.GlobalFree  
0040CB93    C3              RETN  
0040CB94    68 4EABE577     PUSH kernel32.ReadFile                   ; ASCII "j h"  
0040CB99    C3              RETN  
0040CB9A    68 2EF0E577     PUSH kernel32.SetFilePointer  
0040CB9F    C3              RETN  
0040CBA0    68 3AF1E577     PUSH kernel32.WriteFile  
0040CBA5    C3              RETN  
0040CBA6    68 0AB0D377     PUSH user32.MessageBoxA  
0040CBAB    C3              RETN  
0040CBAC    68 90233276     PUSH comdlg32.GetOpenFileNameA////到这里。  
0040CBB1    C3              RETN  
  
跳转表都被改成　PUSH　XXXXXXXX　然后　RETN　的形式了！看开始处是 40CB6A。  
  
好，重新载入程序，按上述方法到第四步后，在 40CB6A 处下内存访问断点，然后 F9 运行程序，断下几次后会来到：  
  
0032F976    03BD 66CB4000   ADD EDI,DWORD PTR SS:\[EBP+40CB66\]  
0032F97C    81C7 00100000   ADD EDI,1000  
0032F982    66:8139 FF25    CMP WORD PTR DS:\[ECX\],25FF////会断在这里。这里开始处理 IAT 跳转表。  
0032F987    74 02           JE SHORT 0032F98B////修改为 JMP　32F9C8  
0032F989    EB 05           JMP SHORT 0032F990  
0032F98B    3941 02         CMP DWORD PTR DS:\[ECX+2\],EAX  
0032F98E    72 07           JB SHORT 0032F997  
0032F990    41              INC ECX  
0032F991    3BCF            CMP ECX,EDI  
0032F993  ^ 72 ED           JB SHORT 0032F982  
0032F995    EB 27           JMP SHORT 0032F9BE  
0032F997    8B59 02         MOV EBX,DWORD PTR DS:\[ECX+2\]  
0032F99A    8B1B            MOV EBX,DWORD PTR DS:\[EBX\]  
0032F99C    C601 68         MOV BYTE PTR DS:\[ECX\],68////IAT 跳转改为 PUSH xxxxxxxx，然后再 RETN 的形式。  
0032F99F    8959 01         MOV DWORD PTR DS:\[ECX+1\],EBX  
0032F9A2    C641 05 C3      MOV BYTE PTR DS:\[ECX+5\],0C3////RETN  
0032F9A6    83C1 06         ADD ECX,6  
0032F9A9    66:8139 FF25    CMP WORD PTR DS:\[ECX\],25FF  
0032F9AE    75 04           JNZ SHORT 0032F9B4  
0032F9B0    3BCF            CMP ECX,EDI  
0032F9B2  ^ 72 CE           JB SHORT 0032F982  
0032F9B4    83C1 02         ADD ECX,2  
0032F9B7    66:8139 FF25    CMP WORD PTR DS:\[ECX\],25FF  
0032F9BC  ^ 74 D9           JE SHORT 0032F997  
0032F9BE    83C1 24         ADD ECX,24  
0032F9C1    66:8139 FF25    CMP WORD PTR DS:\[ECX\],25FF  
0032F9C6  ^ 74 CF           JE SHORT 0032F997  
0032F9C8    61              POPAD  
  
把 0032F987 处直接修改为 JMP　32F9C8　就可以得到完整的 IAT 跳转表了。  
  
到入口后就可以用 PETOOLS 来 DUMP 了（因为用 LORDPE 不行！），再用 ImportREC v142+修复 IAT， OK 完工!

forgot 的 unpackme 1.7xxx 的脱壳(灌水)  
   
　　　　　日期：2005 年 3 月 7 日　　　脱壳人：csjwaman\[DFCG\]  
———————————————————————————————————  
  
————————  
   
   
【软件名称】：forgot 的 unpackme 1.7xxx  
【下载地址】：见附件。  
【脱壳声明】：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠  
  
赐教！  
【操作系统】：win2k  
【脱壳工具】：OD 等传统工具  
  
———————————————————————————————————  
  
————————  
　  
【脱壳过程】：  
  
看了 simonzh2000 的脱文感觉太复杂。菜如我者还是找个简单点办法吧:)当然如果要深入理解还是请看 simonzh2000 的脱文。  
  
forgot 的 unpackme 1.7xxx 和 unpackme 1.7 的变化，我感觉只是加了代码修改检  
  
测和内在断点检测。当时由于没注意，死 FORGOT 让我重启了机子。后来注意了这  
  
两个检测，就通过了。但脱壳的文件还是不能运行!原来是我把被偷的代码看错了，把 40F034 看成了 40F304!!!难怪无法正常运行，还好 FLY 兄弟指出。在此谢谢了。下面请看过程。现在还发这个可能有灌水之嫌噢:)  
  
  
第一步，找 d-process 处：  
  
用 OD 载入程序，忽略所有异常，bp WriteProcessMemory，F9 运行程序，断下：  
  
77E878F8 >  55              PUSH EBP////断下。  
77E878F9    8BEC            MOV EBP,ESP  
77E878FB    51              PUSH ECX  
77E878FC    51              PUSH ECX  
77E878FD    8B45 0C         MOV EAX,DWORD PTR SS:\[EBP+C\]  
77E87900    53              PUSH EBX  
77E87901    8945 F8         MOV DWORD PTR SS:\[EBP-8\],EAX  
77E87904    56              PUSH ESI  
77E87905    8B5D 14         MOV EBX,DWORD PTR SS:\[EBP+14\]  
77E87908    8D45 14         LEA EAX,DWORD PTR SS:\[EBP+14\]  
  
  
观察堆栈数据：  
  
003271E4   0032B337  /CALL 到 WriteProcessMemory 来自 0032B332  
003271E8   00000028  |hProcess = 00000028  
003271EC   0032CA44  |Address = 32CA44////重要！记下备用。  
003271F0   0032C3B3  |Buffer = 0032C3B3  
003271F4   0000029A  |BytesToWrite = 29A (666.)////重要！记下备用。  
003271F8   00000000  \\pBytesWritten = NULL  
  
  
好，F9 直接运行程序，等出现界面时，运行 LordPE 选中映像文件小的那个进程，  
  
然后部分 DUMP，把从 32CA44 处开始大小为 29A 的数据 DUMP 下来，命名为 32CA44.bin。  
  
第二步，变双进程为单进程：  
  
重新载入程序，bp CreateProcessA，运行程序，断下：  
  
77E84FA8 >  55              PUSH EBP////断在这儿。  
77E84FA9    8BEC            MOV EBP,ESP  
77E84FAB    FF75 2C         PUSH DWORD PTR SS:\[EBP+2C\]  
77E84FAE    FF75 28         PUSH DWORD PTR SS:\[EBP+28\]  
77E84FB1    FF75 24         PUSH DWORD PTR SS:\[EBP+24\]  
77E84FB4    FF75 20         PUSH DWORD PTR SS:\[EBP+20\]  
77E84FB7    FF75 1C         PUSH DWORD PTR SS:\[EBP+1C\]  
77E84FBA    FF75 18         PUSH DWORD PTR SS:\[EBP+18\]  
  
观察堆栈数据：  
  
003271D0   0032A21A  /CALL 到 CreateProcessA 来自 0032A215  
003271D4   00328F87  |ModuleFileName =   
  
"d:\\我的文档\\aman\\crack\\17x\\17x.exe"  
003271D8   0032A20B  |CommandLine = "X"////调试标志。  
003271DC   00000000  |pProcessSecurity = NULL  
003271E0   00000000  |pThreadSecurity = NULL  
003271E4   00000000  |InheritHandles = FALSE  
003271E8   00000003  |CreationFlags =   
  
DEBUG\_PROCESS|DEBUG\_ONLY\_THIS\_PROCESS  
003271EC   00000000  |pEnvironment = NULL  
003271F0   00000000  |CurrentDir = NULL  
003271F4   0032755C  |pStartupInfo = 0032755C  
003271F8   0032754C  \\pProcessInfo = 0032754C  
  
  
重新载入程序，bp GetCommandLineA，运行程序，断下：  
  
77E86DF3 >  A1 4CB3EB77     MOV EAX,DWORD PTR DS:\[77EBB34C\]////断在这里。取消断点。  
77E86DF8    C3              RETN  
  
返回到：  
  
0032836A    68 9F6F56B6     PUSH B6566F9F////返回到这里。  
0032836F    50              PUSH EAX  
00328370    E8 5D000000     CALL 003283D2  
00328375    EB FF           JMP SHORT 00328376  
00328377    71 78           JNO SHORT 003283F1  
00328379    C2 5000         RETN 50  
  
  
搜索二进制字符“803e58”，找到：  
  
00328602    FF70 ED         PUSH DWORD PTR DS:\[EAX-13\]  
00328605  ^ 71 EB           JNO SHORT 003285F2  
00328607  ^ EB FA           JMP SHORT 00328603  
00328609  ^ EB 80           JMP SHORT 0032858B/////找到这里。这里有个花指令。  
0032860B    3E:58           POP EAX    
0032860D    0F84 8A410000   JE 0032C79D  
00328613    68 9F6F56B6     PUSH B6566F9F  
00328618    50              PUSH EAX  
  
  
NOP 掉花指令后：  
  
00328609    90              NOP////弄清楚了后就恢复代码，“有病治病，无病防病”，省得后面有代码完整性检测的麻烦:)  
0032860A    803E 58         CMP BYTE PTR DS:\[ESI\],58////58 为调试标志。  
  
用于判断是否为子进程。  
0032860D    0F84 8A410000   JE 0032C79D////此处改 Z 标志位让跳!可以变为单进程。  
00328613    68 9F6F56B6     PUSH B6566F9F  
00328618    50              PUSH EAX  
  
  
第三步，修补程序：  
  
修改上面这个跳转后，用十六进制工具把 32CA44.bin 的数据复盖掉 OD 的 DUMP 区从 0032CA44 处开始的 29A 个字节数据。  
  
第四步，查找入口：  
  
做完上述工作后，在 OD 的 CPU 窗口，Ctrl+G,输入 0032CA44，点确定后来到：  
  
0032CA3B  ^\\71 EB           JNO SHORT 0032CA28  
0032CA3D  ^ EB FA           JMP SHORT 0032CA39  
0032CA3F  ^ EB F0           JMP SHORT 0032CA31////NOP 掉。  
0032CA41    0FC7C8          CMPXCHG8B EAX////NOP 掉。  
0032CA44    FFC3            INC EBX/////来到这里。  
0032CA46    FFC8            DEC EAX  
0032CA48    F7D8            NEG EAX  
0032CA4A    F7C1 2DF16825   TEST ECX,2568F12D  
0032CA50    8BD9            MOV EBX,ECX  
0032CA52    01CB            ADD EBX,ECX  
0032CA54    85CB            TEST EBX,ECX  
  
  
在 0032CA44 处 F2 下断，F9 断下后取消断点!这里好象不必恢复刚才 NOP 掉的代码。  
  
搜索二进制字符“6150C3”，找到：  
  
0032FDF6  ^\\71 EB           JNO SHORT 0032FDE3  
0032FDF8  ^ EB FA           JMP SHORT 0032FDF4  
0032FDFA    EB 50           JMP SHORT 0032FE4C  
0032FDFC    8D78 02         LEA EDI,DWORD PTR DS:\[EAX+2\]  
0032FDFF    EB 02           JMP SHORT 0032FE03////在这里 F2 下断。  
0032FE01    61              POPAD////找到这里。  
0032FE02    50              PUSH EAX  
0032FE03    C3              RETN////返回入口！  
  
在 0032FDFF 处断下后，EAX＝403A3F，这就是入口地址。  
  
00403A3F    FFD7            CALL NEAR EDI ; 17x.00403A41////返回到这里  
  
。入口代码被修改了。  
00403A41    58              POP EAX  
00403A42    90              NOP  
00403A43    90              NOP  
00403A44    E8 FD980000     CALL 17x.0040D346  
00403A49    85C0            TEST EAX,EAX  
00403A4B    74 26           JE SHORT 17x.00403A73  
00403A4D    68 00000200     PUSH 20000  
00403A52    6A 40           PUSH 40  
  
00403A3F    FFD7            CALL NEAR EDI  
00403A41    58              POP EAX  
00403A42    90              NOP  
00403A43    90              NOP  
00403A44    E8 63910000     CALL 1\_7.0040CBAC  
00403A49    85C0            TEST EAX,EAX  
00403A4B    74 26           JE SHORT 1\_7.00403A73  
00403A4D    68 00000200     PUSH 20000  
00403A52    6A 40           PUSH 40  
00403A54    E8 2F910000     CALL 1\_7.0040CB88  
00403A59    A3 C0E14000     MOV DWORD PTR DS:\[40E1C0\],EAX  
00403A5E    68 BCE04000     PUSH 1\_7.0040E0BC  
00403A63    E8 10000000     CALL 1\_7.00403A78  
  
到入口后观察堆栈区：  
  
003275D0   0040F034  17x.0040F034////这就是被偷的代码。  
003275D4   004327F9  返回到 17x.004327F9  
  
  
补回代码后：  
  
00403A3F    68 34F04000     PUSH 17x.0040F034////这才象入口嘛:)  
00403A44    E8 FD980000     CALL 17x.0040D346  
00403A49    85C0            TEST EAX,EAX  
00403A4B    74 26           JE SHORT 17x.00403A73  
00403A4D    68 00000200     PUSH 20000  
00403A52    6A 40           PUSH 40  
00403A54    E8 C9980000     CALL 17x.0040D322  
00403A59    A3 20F24000     MOV DWORD PTR DS:\[40F220\],EAX  
00403A5E    68 11F14000     PUSH 17x.0040F111  
  
起初看错了堆栈区数据，把 0040F034 看成了 0040F304。害得我以为脱壳有问题:(  
  
看来脱壳要细心呀!  
  
第五步，查找 IAT 加密处  
  
到入口后 F7 跟踪：  
  
00403A3F    68 34F04000     PUSH 17x.0040F034////F7  
00403A44    E8 FD980000     CALL 17x.0040D346////F7 进入！  
  
进入后：  
  
0040D304    68 6D7EE677     PUSH kErNeL32.CloseHandle  
0040D309    C3              RETN  
0040D30A    68 ABC1E777     PUSH kErNeL32.CreateFileA  
0040D30F    C3              RETN  
0040D310    68 7269E877     PUSH kErNeL32.ExitProcess  
0040D315    C3              RETN  
0040D316    68 926BE777     PUSH kErNeL32.GetFileSize  
0040D31B    C3              RETN  
0040D31C    68 D4EDE677     PUSH kErNeL32.GetTickCount  
0040D321    C3              RETN  
0040D322    68 7DDBE777     PUSH kErNeL32.GlobalAlloc  
0040D327    C3              RETN  
0040D328    68 E7E4E777     PUSH kErNeL32.GlobalFree  
0040D32D    C3              RETN  
0040D32E    68 6861E777     PUSH kErNeL32.ReadFile  
0040D333    C3              RETN  
0040D334    68 4265E777     PUSH kErNeL32.SetFilePointer  
0040D339    C3              RETN  
0040D33A    68 5063E777     PUSH kErNeL32.WriteFile  
0040D33F    C3              RETN  
0040D340    68 8733E177     PUSH user32.MessageBoxA  
0040D345    C3              RETN  
0040D346    68 869DAF76     PUSH comdlg32.GetOpenFileNameA////到这里。  
0040D34B    C3              RETN  
  
  
跳转表都被改成　PUSH　XXXXXXXX　然后　RETN　的形式了！看开始处是 0040D304。  
  
好，重新载入程序，按上述方法到第四步后，在 0040D304 处下硬件访问断点(不能下内在访问断点了!否则。。。)，然后 F9 运行程序，断下几次后会来到：  
  
0032F969    03BD FFD24000   ADD EDI,DWORD PTR SS:\[EBP+40D2FF\]  
0032F96F    81C7 00100000   ADD EDI,1000  
0032F975    66:8139 FF25    CMP WORD PTR DS:\[ECX\],25FF  
0032F97A    74 02           JE SHORT 0032F97E////会断在这里。这里开始处理 IAT 跳转表。改为 JMP　32F9BB 后 F8。  
0032F97C    EB 05           JMP SHORT 0032F983  
0032F97E    3941 02         CMP DWORD PTR DS:\[ECX+2\],EAX  
0032F981    72 07           JB SHORT 0032F98A  
0032F983    41              INC ECX  
0032F984    3BCF            CMP ECX,EDI  
0032F986  ^ 72 ED           JB SHORT 0032F975  
0032F988    EB 27           JMP SHORT 0032F9B1  
0032F98A    8B59 02         MOV EBX,DWORD PTR DS:\[ECX+2\]  
0032F98D    8B1B            MOV EBX,DWORD PTR DS:\[EBX\]  
0032F98F    C601 68         MOV BYTE PTR DS:\[ECX\],68////IAT 跳转改为 PUSH xxxxxxxx，然后再 RETN 的形式。  
0032F992    8959 01         MOV DWORD PTR DS:\[ECX+1\],EBX  
0032F995    C641 05 C3      MOV BYTE PTR DS:\[ECX+5\],0C3////RETN  
0032F999    83C1 06         ADD ECX,6  
0032F99C    66:8139 FF25    CMP WORD PTR DS:\[ECX\],25FF  
0032F9A1    75 04           JNZ SHORT 0032F9A7  
0032F9A3    3BCF            CMP ECX,EDI  
0032F9A5  ^ 72 CE           JB SHORT 0032F975  
0032F9A7    83C1 02         ADD ECX,2  
0032F9AA    66:8139 FF25    CMP WORD PTR DS:\[ECX\],25FF  
0032F9AF  ^ 74 D9           JE SHORT 0032F98A  
0032F9B1    83C1 24         ADD ECX,24  
0032F9B4    66:8139 FF25    CMP WORD PTR DS:\[ECX\],25FF  
0032F9B9  ^ 74 CF           JE SHORT 0032F98A  
0032F9BB    61              POPAD////到这里后恢复刚才修改的代码。  
0032F9BC    E8 0A84FFFF     CALL 00327DCB  
0032F9C1    68 9F6F56B6     PUSH B6566F9F  
0032F9C6    50              PUSH EAX  
0032F9C7    E8 5D000000     CALL 0032FA29  
  
把 0032F97A 处直接修改为 JMP　32F9BB　就可以得到完整的 IAT 跳转表了。  
  
到入口后就可以用 PETOOLS 来 DUMP 了（因为用 LORDPE 不行！），再用 ImportRECv142+修复 IAT， OK 完工

forgot 的 unpackme 1.7yyy 的脱壳  
   
　　　　　日期：2005 年 3 月 13 日　　　脱壳人：csjwaman\[DFCG\]  
———————————————————————————————————————————  
   
   
【软件名称】：forgot 的 unpackme 1.7yyy  
【脱壳声明】：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！  
【操作系统】：winxp  
【脱壳工具】：OD 等传统工具  
  
———————————————————————————————————————————  
　  
【脱壳过程】：  
  
  
forgot 说：有的壳脱了，它还活着。  
  
Lenus 说：有的壳没脱，他已经死了:)  
  
下面就看看这壳是怎么死的:)  
  
第一步，找 d-process 处：  
  
用 OD 载入程序，忽略所有异常，bp WriteProcessMemory，F9 运行程序，断下：  
  
77E41A94 k>  55                      push ebp////断下。取消断点（我习惯及时取消断点，这样可以避免一些不必要的麻烦。）。  
77E41A95     8BEC                    mov ebp,esp  
77E41A97     51                      push ecx  
77E41A98     51                      push ecx  
77E41A99     8B45 0C                 mov eax,dword ptr ss:\[ebp+C\]  
77E41A9C     53                      push ebx  
77E41A9D     8B5D 14                 mov ebx,dword ptr ss:\[ebp+14\]  
77E41AA0     56                      push esi  
77E41AA1     8B35 B012E477           mov esi,dword ptr ds:\[<&ntdll.N>; ntdll.ZwProtectVirtualMemory  
77E41AA7     57                      push edi  
  
  
观察堆栈数据：  
  
003C7054    003CB1A7  /CALL 到 WriteProcessMemory 来自 003CB1A2  
003C7058    00000034  |hProcess = 00000034 (window)  
003C705C    003CC8B4  |Address = 3CC8B4////重要！记下备用。  
003C7060    003CC223  |Buffer = 003CC223  
003C7064    0000029A  |BytesToWrite = 29A (666.)////重要！记下备用。  
003C7068    00000000  \\pBytesWritten = NULL  
  
好，F9 直接运行程序，等出现界面时，运行 LordPE 选中映像文件小的那个进程，然后部分 DUMP，把从 3CC8B4 处开始大小为 29A 的数据 DUMP 下来，命名为 3CC8B4.bin。  
  
第二步，变双进程为单进程：  
  
重新载入程序，bp CreateProcessA，运行程序，断下：  
  
77E41BBC k>  55                      push ebp////断下。取消断点。  
77E41BBD     8BEC                    mov ebp,esp  
77E41BBF     6A 00                   push 0  
77E41BC1     FF75 2C                 push dword ptr ss:\[ebp+2C\]  
77E41BC4     FF75 28                 push dword ptr ss:\[ebp+28\]  
77E41BC7     FF75 24                 push dword ptr ss:\[ebp+24\]  
77E41BCA     FF75 20                 push dword ptr ss:\[ebp+20\]  
  
  
观察堆栈数据：  
  
003C7040    003CA08A  /CALL 到 CreateProcessA 来自 003CA085  
003C7044    003C8DF7  |ModuleFileName = "C:\\Documents and Settings\\csjwaman\\桌面\\17y.rar\_828\\1.exe"  
003C7048    003CA07B  |CommandLine = "X"////调试标志。  
003C704C    00000000  |pProcessSecurity = NULL  
003C7050    00000000  |pThreadSecurity = NULL  
003C7054    00000000  |InheritHandles = FALSE  
003C7058    00000003  |CreationFlags = DEBUG\_PROCESS|DEBUG\_ONLY\_THIS\_PROCESS  
003C705C    00000000  |pEnvironment = NULL  
003C7060    00000000  |CurrentDir = NULL  
003C7064    003C73CC  |pStartupInfo = 003C73CC  
003C7068    003C73BC  \\pProcessInfo = 003C73BC  
  
  
重新载入程序，bp GetCommandLineA，运行程序，断下：  
  
77E5E358 k>  A1 1476EB77             mov eax,dword ptr ds:\[77EB7614\]////断在这里。取消断点。  
77E5E35D     C3                      retn  
  
返回到：  
  
003C81DA     68 9F6F56B6             push B6566F9F////返回到这里。  
003C81DF     50                      push eax  
003C81E0     E8 5D000000             call 003C8242  
003C81E5     EB FF                   jmp short 003C81E6  
003C81E7     71 78                   jno short 003C8261  
003C81E9     C2 5000                 retn 50  
003C81EC   ^ EB D3                   jmp short 003C81C1  
  
  
搜索二进制字符“803e58”，找到：  
  
003C8475   ^\\71 EB                   jno short 003C8462  
003C8477   ^ EB FA                   jmp short 003C8473  
003C8479   ^ EB 80                   jmp short 003C83FB/////找到这里。这里有个花指令。  
003C847B     3E:58                   pop eax  
003C847D     0F84 8A410000           je 003CC60D////在此 F2 断下后，取消断点，然后改 Z 标志位让跳!可以变为单进程。  
  
  
第三步，修补程序：  
  
修改上面这个跳转后，用十六进制工具把 3CC8B4.bin 的数据复盖掉 OD 的 DUMP 区从 003CC8B4 处开始的 29A 个字节数据。  
  
第四步，修复 IAT 及查找入口：  
  
做完上述工作后，在 OD 的 CPU 窗口，Ctrl+G,输入 003CC8B4，点确定后来到：  
  
003CC8AB   ^\\71 EB                   jno short 003CC898  
003CC8AD   ^ EB FA                   jmp short 003CC8A9  
003CC8AF   ^ EB F0                   jmp short 003CC8A1////NOP 掉。  
003CC8B1     0FC7C8                  cmpxchg8b eax  ////NOP 掉。  
003CC8B4     FFC3                    inc ebx/////来到这里。  
003CC8B6     FFC8                    dec eax  
003CC8B8     F7D8                    neg eax  
003CC8BA     F7C1 2DF16825           test ecx,2568F12D  
003CC8C0     8BD9                    mov ebx,ecx  
003CC8C2     01CB                    add ebx,ecx  
003CC8C4     85CB                    test ebx,ecx  
003CC8C6     8BDA                    mov ebx,edx  
  
  
在 003CC8B4 处硬件执行断点，F9 断下后取消断点!恢复刚才 NOP 掉的代码。  
  
然后直接往下找，来到：  
  
003CF7AF     8B8D 63D54000           mov ecx,dword ptr ss:\[ebp+40D563\]  
003CF7B5     0349 3C                 add ecx,dword ptr ds:\[ecx+3C\]  
003CF7B8     8B41 50                 mov eax,dword ptr ds:\[ecx+50\]  
003CF7BB     0385 63D54000           add eax,dword ptr ss:\[ebp+40D563\]  
003CF7C1     8B8D 63D54000           mov ecx,dword ptr ss:\[ebp+40D563\]  
003CF7C7     81C1 00100000           add ecx,1000  
003CF7CD     8BBD 63D54000           mov edi,dword ptr ss:\[ebp+40D563\]  
003CF7D3     037F 3C                 add edi,dword ptr ds:\[edi+3C\]  
003CF7D6     8B7F 1C                 mov edi,dword ptr ds:\[edi+1C\]  
003CF7D9     03BD 63D54000           add edi,dword ptr ss:\[ebp+40D563\]  
003CF7DF     81C7 00100000           add edi,1000  
003CF7E5     66:8139 FF25            cmp word ptr ds:\[ecx\],25FF  
003CF7EA     74 02                   je short 003CF7EE////这里开始处理 IAT 跳转表。改为 JMP　003CF82B。  
003CF7EC     EB 05                   jmp short 003CF7F3  
003CF7EE     3941 02                 cmp dword ptr ds:\[ecx+2\],eax  
003CF7F1     72 07                   jb short 003CF7FA  
003CF7F3     41                      inc ecx  
003CF7F4     3BCF                    cmp ecx,edi  
003CF7F6   ^ 72 ED                   jb short 003CF7E5  
003CF7F8     EB 27                   jmp short 003CF821  
003CF7FA     8B59 02                 mov ebx,dword ptr ds:\[ecx+2\]  
003CF7FD     8B1B                    mov ebx,dword ptr ds:\[ebx\]  
003CF7FF     C601 68                 mov byte ptr ds:\[ecx\],68////IAT 跳转改为 PUSH xxxxxxxx，然后再 RETN 的形式。  
003CF802     8959 01                 mov dword ptr ds:\[ecx+1\],ebx  
003CF805     C641 05 C3              mov byte ptr ds:\[ecx+5\],0C3////RETN  
003CF809     83C1 06                 add ecx,6  
003CF80C     66:8139 FF25            cmp word ptr ds:\[ecx\],25FF  
003CF811     75 04                   jnz short 003CF817  
003CF813     3BCF                    cmp ecx,edi  
003CF815   ^ 72 CE                   jb short 003CF7E5  
003CF817     83C1 02                 add ecx,2  
003CF81A     66:8139 FF25            cmp word ptr ds:\[ecx\],25FF  
003CF81F   ^ 74 D9                   je short 003CF7FA  
003CF821     83C1 24                 add ecx,24  
003CF824     66:8139 FF25            cmp word ptr ds:\[ecx\],25FF  
003CF829   ^ 74 CF                   je short 003CF7FA  
003CF82B     61                      popad////下硬件执行断点，然后 SHIFT+F9，通过几次异常后会断在这里，然后取消断点。  
003CF82C     E8 0A84FFFF             call 003C7C3B  
003CF831     68 9F6F56B6             push B6566F9F  
003CF836     50                      push eax  
003CF837     E8 5D000000             call 003CF899  
003CF83C     EB FF                   jmp short 003CF83D  
003CF83E     71 78                   jno short 003CF8B8  
003CF840     C2 5000                 retn 50  
  
跳转表处理和 1.7xxx 一样都改成　PUSH　XXXXXXXX　然后　RETN　的形式！  
  
跳过 IAT 加密后，继续直接往下找：  
  
003CFDEB   ^\\71 EB                   jno short 003CFDD8  
003CFDED   ^ EB FA                   jmp short 003CFDE9  
003CFDEF   ^ EB 83                   jmp short 003CFD74  
003CFDF1     C017 EB                 rcl byte ptr ds:\[edi\],0EB  
003CFDF4     FF70 ED                 push dword ptr ds:\[eax-13\]  
003CFDF7   ^ 71 EB                   jno short 003CFDE4  
003CFDF9   ^ EB FA                   jmp short 003CFDF5  
003CFDFB     EB 50                   jmp short 003CFE4D  
003CFDFD     8D78 02                 lea edi,dword ptr ds:\[eax+2\]  
003CFE00     EB 02                   jmp short 003CFE04 下硬件执行断点，F9，断下后取消断点。  
003CFE02     61                      popad  
003CFE03     50                      push eax  
003CFE04     C3                      retn////返回入口！  
003CFE05     0200                    add al,byte ptr ds:\[eax\]  
003CFE07     0000                    add byte ptr ds:\[eax\],al  
  
  
在 0032FDFF 处断下后，EAX＝403A3F，这就是入口地址。  
  
00403A3F     FFD7                    call edi     ; 1.00403A41////返回到这里。入口代码被修改了。  
00403A41     58                      pop eax  
00403A42     90                      nop  
00403A43     90                      nop  
00403A44     E8 5B9B0000             call 1.0040D5A4                 ; jmp to comdlg32.GetOpenFileNameA  
00403A49     85C0                    test eax,eax  
00403A4B     74 17                   je short 1.00403A64  
00403A4D     68 15F14000             push 1.0040F115  
00403A52     E8 2C000000             call 1.00403A83  
00403A57     8B0D 04F04000           mov ecx,dword ptr ds:\[40F004\]  
00403A5D     E3 05                   jecxz short 1.00403A64  
00403A5F     E8 229B0000             call 1.0040D586                 ; jmp to kernel32.GlobalFree  
00403A64     E8 059B0000             call 1.0040D56E                 ; jmp to kernel32.ExitProcess  
00403A69     55                      push ebp  
  
  
到入口后观察堆栈区：  
  
003C7440    0040F038  1.0040F038////这就是被偷的代码。  
003C7444    00432ED9  返回到 1.00432ED9  
  
  
补回代码后：  
  
00403A3F     68 38F04000             push 1.0040F038////这才象入口嘛:)  
00403A44     E8 5B9B0000             call 1.0040D5A4                 ; jmp to comdlg32.GetOpenFileNameA  
00403A49     85C0                    test eax,eax  
00403A4B     74 17                   je short 1.00403A64  
00403A4D     68 15F14000             push 1.0040F115  
00403A52     E8 2C000000             call 1.00403A83  
00403A57     8B0D 04F04000           mov ecx,dword ptr ds:\[40F004\]  
00403A5D     E3 05                   jecxz short 1.00403A64  
00403A5F     E8 229B0000             call 1.0040D586                 ; jmp to kernel32.GlobalFree  
00403A64     E8 059B0000             call 1.0040D56E                 ; jmp to kernel32.ExitProcess  
00403A69     55                      push ebp  
  
  
到入口后设置区段为完全权限，然后就可以用 PETOOLS 来 DUMP 了（因为用 LORDPE 不行！），删除 FUCK 区段及其数据，再用 ImportREC v142+修复 IAT。保存为 DUMP\_.EXE ,程序可以运行了！  
  
  
第五步，修复区段：  
  
  
加壳试试。晕！不能加壳。总是在这里出错：  
  
004024C3    .  F3:AA                 rep stos byte ptr es:\[edi\]////出错！  
004024C5    >  61                    popad  
004024C6    .  C9                    leave  
004024C7    .  C3                    retn  
  
  
ecx=00008B10 (十进制 35600.)  
al=FF  
es:\[edi\]=\[00477000\]=???////无 477000 内存地址！  
  
  
00403918    .  8B75 28               mov esi,dword ptr ss:\[ebp+28\]  
0040391B    .  8B4D 2C               mov ecx,dword ptr ss:\[ebp+2C\]  
0040391E    .  898D C4FEFFFF         mov dword ptr ss:\[ebp-13C\],ecx  
00403924       F3:A4                 rep movs byte ptr es:\[edi\],byte ptr ds:\[esi\]////出错！  
00403926    .  FF75 30               push dword ptr ss:\[ebp+30\]  
00403929    .  8F85 C8FEFFFF         pop dword ptr ss:\[ebp-138\]  
0040392F    .  33D2                  xor edx,edx  
00403931    >  42                    inc edx  
00403932    .  FF75 58               push dword ptr ss:\[ebp+58\]  
  
  
ecx=00004E6F (十进制 20079.)  
ds:\[esi\]=\[004086F3\]=C0  
es:\[edi\]=\[0045A000\]=???////无 45A000 内存地址！  
  
用 OD 载入脱壳后的程序，到入口后观察内存镜像：  
  
  
内存映射  
地址       大小       Owner                         区段        包含             类型                访问      初始访问     映射为  
  
00400000   00001000   Dumped\_  00400000 (itself)                PE header        Imag 01001002       R         RWE  
00401000   0000D000   Dumped\_  00400000             .text       code             Imag 01001002       R         RWE  
0040E000   00001000   Dumped\_  00400000             .rdata                       Imag 01001002       R         RWE  
0040F000   00001000   Dumped\_  00400000             .data                        Imag 01001002       R         RWE  
00410000   00002000   Dumped\_  00400000             .rsrc       data,resources   Imag 01001002       R         RWE  
00412000   00033000   Dumped\_  00400000             FUCK                         Imag 01001002       R         RWE  
00445000   00001000   Dumped\_  00400000             .mackt      imports          Imag 01001002       R         RWE  
00450000   00005000            00450000 (itself)                                 Priv 00021004       RW        RW  
  
  
观察原程序到入口进的内存镜像：  
  
  
内存映射  
地址       大小       Owner                         区段        包含             类型                访问      初始访问     映射为  
内存映射  
地址       大小       Owner                         区段        包含             类型                访问      初始访问     映射为  
00010000   00001000            00010000 (itself)                                 Priv 00021004       RW        RW  
00020000   00001000            00020000 (itself)                                 Priv 00021004       RW        RW  
003C3000   00001000            00030000                                          Priv 00021104       RW  防护    RW  
003C4000   0000C000            00030000                         stack of 主要线     Priv 00021104       RW  防护    RW  
003D0000   00001000            003D0000 (itself)                                 Map  00041002       R         R  
003E0000   00006000            003E0000 (itself)                                 Priv 00021004       RW        RW  
003F0000   00001000            003F0000 (itself)                                 Map  00041004       RW        RW  
00400000   00001000   1        00400000 (itself)                PE header        Imag 01001002       R         RWE  
00401000   0000D000   1        00400000             .text       code             Imag 01001002       R         RWE  
0040E000   00001000   1        00400000             .rdata                       Imag 01001002       R         RWE  
0040F000   00001000   1        00400000             .data                        Imag 01001002       R         RWE  
00410000   00002000   1        00400000             .rsrc       data,resources   Imag 01001002       R         RWE  
00412000   00033000   1        00400000             FUCK        SFX,imports      Imag 01001002       R         RWE  
00450000   00031000            00450000 (itself)                                 Priv 00021004       RW        RW  
  
呵呵，看出问题来了吧！脱壳后的程序 450000 区段只有 5000H 大小，而原程序却有 31000H 大小！两者相差 2C000H。  
  
知道了原因就好办了。在 DUMP\_.EXE 文件的最后增加一个大小为 31000H 的区段！然后用 LordPE 重建一下就可以了。不过重建只选 PE 验证选项噢！260K->272

 4dd466d0e1e0cc123afbe504da2462ca