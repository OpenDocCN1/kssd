

# forgot 的 Unpack Me v0.1.5 bugfix 脱壳

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-492.htm`](https://www.pediy.com/kssd/pediy07/pediy7-492.htm)

                            forgot 的 Unpack Me v0.1.5 bugfix 脱壳  
   
　　　　　日期：2005 年 2 月 4 日　　　脱壳人：csjwaman\[DFCG\]  
———————————————————————————————————————————  
   
   
【软件名称】：forgot 的 Unpack Me v0.1.5 bugfix  
【下载地址】：[`www.pediy.com/bbs/pediy7/pediy7-492.rar`](http://www.pediy.com/bbs/pediy7/pediy7-492.rar)  
【脱壳声明】：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！  
【操作系统】：win2k  
【脱壳工具】：OD 等传统工具  
  
———————————————————————————————————————————  
　  
【脱壳过程】：  
  
  
第一步，找 d-process 处：  
  
用 OD 载入程序，忽略所有异常，bp WriteProcessMemory，F9 运行程序，断下：  
  
77E7ADB9 >  55              PUSH EBP////断下。  
77E7ADBA    8BEC            MOV EBP,ESP  
77E7ADBC    51              PUSH ECX  
77E7ADBD    51              PUSH ECX  
77E7ADBE    8B45 0C         MOV EAX,DWORD PTR SS:\[EBP+C\]  
77E7ADC1    53              PUSH EBX  
77E7ADC2    8945 F8         MOV DWORD PTR SS:\[EBP-8\],EAX  
  
观察堆栈数据：  
  
00A185C0   00A1BEBF  /CALL 到 WriteProcessMemory 来自 00A1BEBA  
00A185C4   00000028  |hProcess = 00000028 (window)  
00A185C8   00A1D5BA  |Address = A1D5BA/////重要!记下备用。  
00A185CC   00A1CF28  |Buffer = 00A1CF28  
00A185D0   0000029A  |BytesToWrite = 29A (666.)/////大小。  
00A185D4   00000000  \\pBytesWritten = NULL  
  
好，F9 直接运行程序，等出现界面时，运行 LordPE 选中映像文件小的那个进程，然后部分 DUMP，把从 A1D5BA 处开始大小为 29A 的数据 DUMP 下来，命名为 A1D5BA.bin。  
  
第二步，变双进程为单进程：  
  
重新载入程序，bp CreateProcessA，运行程序，断下：  
  
77E73F8F >  55              PUSH EBP////断在这儿。  
77E73F90    8BEC            MOV EBP,ESP  
77E73F92    FF75 2C         PUSH DWORD PTR SS:\[EBP+2C\]  
77E73F95    FF75 28         PUSH DWORD PTR SS:\[EBP+28\]  
77E73F98    FF75 24         PUSH DWORD PTR SS:\[EBP+24\]  
77E73F9B    FF75 20         PUSH DWORD PTR SS:\[EBP+20\]  
77E73F9E    FF75 1C         PUSH DWORD PTR SS:\[EBP+1C\]  
  
观察堆栈数据：  
  
00A185AC   00A1AF45  /CALL 到 CreateProcessA 来自 00A1AF40  
00A185B0   00A19CB2  |ModuleFileName = "C:\\Documents and Settings\\Administrator\\桌面\\015\\复件 1.exe"  
00A185B4   00A1AF36  |CommandLine = "X"/////调试标志!  
00A185B8   00000000  |pProcessSecurity = NULL  
00A185BC   00000000  |pThreadSecurity = NULL  
00A185C0   00000000  |InheritHandles = FALSE  
00A185C4   00000003  |CreationFlags = DEBUG\_PROCESS|DEBUG\_ONLY\_THIS\_PROCESS  
00A185C8   00000000  |pEnvironment = NULL  
00A185CC   00000000  |CurrentDir = NULL  
00A185D0   00A18938  |pStartupInfo = 00A18938  
00A185D4   00A18928  \\pProcessInfo = 00A18928  
  
  
  
重新载入程序，bp GetCommandLineA，运行程序，断下：  
  
77E6FB03 >  A1 9406EC77     MOV EAX,DWORD PTR DS:\[77EC0694\]////断在这里。取消断点。  
77E6FB08    C3              RETN////返回。  
77E6FB09 >  55              PUSH EBP  
77E6FB0A    8BEC            MOV EBP,ESP  
77E6FB0C    56              PUSH ESI  
77E6FB0D    64:A1 18000000  MOV EAX,DWORD PTR FS:\[18\]  
  
返回到：  
  
00A1909F    68 9F6F56B6     PUSH B6566F9F  
00A190A4    50              PUSH EAX  
00A190A5    E8 5D000000     CALL 00A19107  
00A190AA    EB FF           JMP SHORT 00A190AB  
00A190AC    71 78           JNO SHORT 00A19126  
00A190AE    C2 5000         RETN 50  
00A190B1  ^ EB D3           JMP SHORT 00A19086  
00A190B3    5B              POP EBX  
00A190B4    F3:             PREFIX REP:                              ; 多余的前缀  
00A190B5    68 895C2448     PUSH 48245C89  
00A190BA    5C              POP ESP  
00A190BB    24 58           AND AL,58  
00A190BD    FF8D 5C24585B   DEC DWORD PTR SS:\[EBP+5B58245C\]  
  
搜索二进制字符“803e58”，找到：  
  
00A19330  ^\\EB FA           JMP SHORT 00A1932C  
00A19332  ^ EB 83           JMP SHORT 00A192B7  
00A19334    C017 EB         RCL BYTE PTR DS:\[EDI\],0EB                ; 移动常数超出 1..31 的范围  
00A19337    FF70 ED         PUSH DWORD PTR DS:\[EAX-13\]  
00A1933A  ^ 71 EB           JNO SHORT 00A19327  
00A1933C  ^ EB FA           JMP SHORT 00A19338  
00A1933E  ^ EB 80           JMP SHORT 00A192C0/////找到这里。这里有个花指令。  
00A19340    3E:58           POP EAX                                  ; 多余的前缀  
00A19342    0F84 CB3F0000   JE 00A1D313  
00A19348    68 9F6F56B6     PUSH B6566F9F  
00A1934D    50              PUSH EAX  
00A1934E    E8 5D000000     CALL 00A193B0  
00A19353    EB FF           JMP SHORT 00A19354  
00A19355    71 78           JNO SHORT 00A193CF  
  
NOP 掉花指令后：  
  
00A1933E    90              NOP  
00A1933F    803E 58         CMP BYTE PTR DS:\[ESI\],58////58 为调试标志。用于判断是否为子进程。  
00A19342    0F84 CB3F0000   JE 00A1D313////此处改为 JMP!  
  
第三步，修补程序：  
  
修改上面这个跳转后，用十六进制工具把 A1D5BA.bin 的数据复盖掉 OD 的 DUMP 区从 0A1D5BA 处开始的 29A 个字节数据。  
  
第四步，查找入口：  
  
做完上述工作后，在 OD 的 CPU 窗口，Ctrl+G,输入 0A1D5BA，点确定后来到：  
  
00A1D5B3  ^\\EB FA           JMP SHORT 00A1D5AF  
00A1D5B5  ^ EB F0           JMP SHORT 00A1D5A7////这里有个花指令。  
00A1D5B7    0FC7C8          CMPXCHG8B EAX                            ; 非法使用寄存器  
00A1D5BA    FFC3            INC EBX/////来到这里。  
00A1D5BC    FFC8            DEC EAX  
00A1D5BE    F7D8            NEG EAX  
00A1D5C0    F7C1 2DF16825   TEST ECX,2568F12D  
00A1D5C6    8BD9            MOV EBX,ECX  
  
取消花指令后：  
  
00A1D5B5    90              NOP  
00A1D5B6    F0:0FC7C8       LOCK CMPXCHG8B EAX ////NOP 掉!  
  
好，接着 Alt+M 打开内存镜像，在.data 区段下内存访问断点。  
  
F9 运行程序，断在：  
  
00A18B06    AC              LODS BYTE PTR DS:\[ESI\]////断在这里，清除内存断点。  
00A18B07    F9              STC  
00A18B08    2AC1            SUB AL,CL  
00A18B0A    F8              CLC  
00A18B0B    EB 01           JMP SHORT 00A18B0E  
00A18B0D  - E9 C0C0D7C0     JMP C1794BD2  
  
再次 Alt+M 打开内存镜像，在.text 区段下内存访问断点。  
  
F9 运行程序，断在：  
  
00403A3F    68 34D04000     PUSH 复件\_1.0040D034////断在这里!这儿就是入口。DUMP 出来。  
00403A44    E8 A9840000     CALL 复件\_1.0040BEF2                       ; JMP to comdlg32.GetOpenFileNameA  
00403A49    85C0            TEST EAX,EAX  
00403A4B    74 26           JE SHORT 复件\_1.00403A73  
00403A4D    68 00B00400     PUSH 4B000  
00403A52    6A 40           PUSH 40  
00403A54    E8 75840000     CALL 复件\_1.0040BECE                       ; JMP to kErNeL32.GlobalAlloc  
00403A59    A3 D0D14000     MOV DWORD PTR DS:\[40D1D0\],EAX  
00403A5E    68 BDD04000     PUSH 复件\_1.0040D0BD  
00403A63    E8 10000000     CALL 复件\_1.00403A78  
  
  
DUMP 要注意纠正映像大小，然后删除 SHIT 区段，再用 ImportREC v142+修复 IAT，优化一下，388K--->50.7K     OK 完工!

 4dd466d0e1e0cc123afbe504da2462ca