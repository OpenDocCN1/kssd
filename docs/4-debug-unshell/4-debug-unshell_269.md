

# Arm V4.4 CopyMemII+Debugblocker+IAT Elimination+Code Splicing 记事本脱壳【原创】

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-565.htm`](https://www.pediy.com/kssd/pediy08/pediy8-565.htm)

Armadillo V4.4 CopyMemII+Debugblocker+IAT Elimination+Code Splicing 保护方式记事本脱壳  
  
【脱壳软件】：用 WIN98 记事本加壳。  
【作者声明】：出于学习目的，失误之处敬请诸位大侠赐教。  
【调试工具】：WinXP、OllyDbg、PEiD、LordPE,arminline0.7、pupe、ArmaDetach、RecImport。  
【保护方式】： Armadillo V4.4 CopyMemII+Debugblocker+IAT Elimination+Code Splicing 保护方式脱壳  
  
初学脱壳，蒙 fly 兄弟的帮助与鼓励，总算完成了脱壳过程，在此首先感谢 fly 兄弟。  
也非常感谢各位大侠们的有关教程参考。  
注册《看雪》已很长时间了，出于对各位大侠的尊重，不敢发一个主题“班门弄斧”。这是首个主题，算是给《看雪》一个汇报吧。  
  
OD 载入。。。。。。  
00449C43 >  55              push ebp  
00449C44    8BEC            mov ebp,esp  
00449C46    6A FF           push -1  
00449C48    68 902F4700     push 乱序+远?00472F90  
00449C4D    68 80994400     push 乱序+远?00449980  
00449C52    64:A1 00000000  mov eax,dword ptr fs:\[0\]  
00449C58    50              push eax  
00449C59    64:8925 0000000>mov dword ptr fs:\[0\],esp  
\---------------------------------------------------------------------------------------------  
下断:BP WaitForDebugEvent  
  
7C85A268 >  8BFF            mov edi,edi  
7C85A26A    55              push ebp  
7C85A26B    8BEC            mov ebp,esp  
7C85A26D    83EC 68         sub esp,68  
7C85A270    56              push esi  
7C85A271    FF75 0C         push dword ptr ss:\[ebp+C\]  
7C85A274    8D45 F8         lea eax,dword ptr ss:\[ebp-8\]  
7C85A277    50              push eax  
7C85A278    E8 F381FAFF     call kernel32.7C802470  
进程将要复制给子进程第一个块的信息,在堆栈窗口中：  
\========================================================================  
0012DC8C   00439D86   /CALL 到 WaitForDebugEvent 来自 乱序+远?00439D80  
0012DC90   0012ED7C   |pDebugEvent = 0012ED7C  
0012DC94   000003E8   \\Timeout = 1000. ms  
\========================================================================  
▲pDebugEvent=0012CD90 数据窗口跟随后，数据窗口一直保持在这里，到结束。▲  
  
  
BC WaitForDebugEvent 清除这个断点。当然也可以 F2 取消。  
\--------------------------------------------------------------------------------------------  
\--子进程 OEP、PID--  
  
BP WriteProcessMemory，Shirt+F9 ,到 BytesToWrite=1000。  
  
7C80220F >  8BFF            mov edi,edi  
7C802211    55              push ebp  
7C802212    8BEC            mov ebp,esp  
7C802214    51              push ecx  
7C802215    51              push ecx  
7C802216    8B45 0C         mov eax,dword ptr ss:\[ebp+C\]  
7C802219    53              push ebx  
7C80221A    8B5D 14         mov ebx,dword ptr ss:\[ebp+14\]  
  
\--子进程 OEP、PID--  
\=======================================================================  
0012DB2C   0043DD57   /CALL 到 WriteProcessMemory 来自 乱序+远?0043DD51    
0012DB30   0000004C   |hProcess = 0000004C (window)  
0012DB34   00401000   |Address = 401000  
0012DB38   003C3560   |Buffer = 003C3560  
0012DB3C   00001000   |BytesToWrite = 1000 (4096.)  
0012DB40   0012DC48   \\pBytesWritten = 0012DC48  
\=======================================================================  
PID=0FA4  OEP=4010CC  
\==========================================================================  
0012ED7C  01 00 00 00 A4 0F 00 00 7C 0C 00 00 01 00 00 80  ...?..|.....€  
0012ED8C  00 00 00 00 00 00 00 00 CC 10 40 00 02 00 00 00  ........?@....  
0012ED9C  00 00 00 00 CC 10 40 00 CC 10 40 00 06 00 00 00  ....?@.?@....  
0012EDAC  00 00 00 00 E8 1C 88 B4 00 00 00 00 00 00 00 00  ....?埓........  
0012EDBC  13 00 00 00 00 0D 88 00 64 CD 45 B4 F1 2F 4E 80  .....?d 虴瘩/N€  
0012EDCC  00 00 00 00 CC 10 40 00 01 00 00 00 01 00 00 00  ....?@.......  
0012EDDC  01 EE 12 00 00 00 00 00 00 00 00 00 00 00 00 00  ?.............  
\==========================================================================  
\--nop 加密 CLL--  
  
在堆栈窗口右键-->反汇编窗口  
\========================================================================  
0012DB2C   0043DD57   /CALL 到 WriteProcessMemory 来自 乱序+远?0043DD51 ; 右键-->反汇编窗口跟随  
\========================================================================  
反汇编窗口跟随到这里，下断，F9 一次。  
  
0043DD57    85C0            test eax,eax  
0043DD59    75 4B           jnz short IAT+SP.0043DDA6  
0043DD5B    50              push eax  
0043DD5C    F7D0            not eax  
0043DD5E    0FC8            bswap eax  
0043DD60    58              pop eax  
0043DD61    73 00           jnb short IAT+SP.0043DD63  
  
在堆栈窗口向下回滚，到这里。  
\==========================================================================  
0012DC38   00000020  
0012DC3C   00401000   乱序+远?00401000  
0012DC40   00000020  
0012DC44   003C4560  
0012DC48   00001000  
0012DC4C   003C4560  
0012DC50  /0012DC84  
0012DC54  |0043C9C5   返回到 乱序+远?0043C9C5 来自 乱序+远?0043CD0D  
\==========================================================================  
在反汇编窗口 Ctrl+G: 0043CD0D，到这里。  
  
0043CD0D    55              push ebp  
0043CD0E    8BEC            mov ebp,esp  
0043CD10    81EC 00010000   sub esp,100  
0043CD16    53              push ebx  
0043CD17    56              push esi  
0043CD18    57              push edi  
0043CD19    8B45 08         mov eax,dword ptr ss:\[ebp+8\]  
  
Ctrl+R，选择第二个 CALL 双击。  
\=====================================================================  
参考位于 乱序+远?.text1 到 0043CD0D  
地址       反汇编                               注释  
0043C9C0   call 乱序+远?0043CD0D  
0043CC95   call 乱序+远?0043CD0D                      ; 选择这个 CALL  
0043CD0D   push ebp                             （初始 CPU 选择）  
\=====================================================================  
到这里了，将 0043CC95 这个 CALL，nop 掉。  
0043CC95    E8 73000000     call 乱序+远?0043CD0D     ; nop 这个 CALL  
0043CC9A    83C4 0C         add esp,0C  
0043CC9D    50              push eax  
0043CC9E    F7D0            not eax  
0043CCA0    0FC8            bswap eax  
0043CCA2    58              pop eax  
  
在堆栈窗口 Ctrl+G：0012ED7C  
\=======================================  
0012ED94  |004010CC   乱序+远?004010CC  
0012ED98  |00000002  
0012ED9C  |00000000  
0012EDA0  |004010CC   乱序+远?004010CC  
0012EDA4  |004010CC   乱序+远?004010CC  
\=======================================  
▲子进程 OEP 为 004010CC,注意，我们稍后会需要这三个地址。▲  
  
\---------------------------------------------------------------------------------------------  
  
  
\--形成循环--  
  
打开 PUPE,打开程序(记得使用正确的进程),键入子进程的 OEP=004010CC。  
     
选择子进程 PID=0FA4  
     
原字节: 55 B8， 修改为：EBFE。  
  
关闭 PUPE。  
  
回到 Olly。  
  
清除所有断点。  
  
下断 bp WaitForDebugEvent ，F9。  
  
取消 bc WaitForDebugEvent  
  
7C85A268 >  8BFF            mov edi,edi  
7C85A26A    55              push ebp  
7C85A26B    8BEC            mov ebp,esp  
7C85A26D    83EC 68         sub esp,68  
7C85A270    56              push esi  
7C85A271    FF75 0C         push dword ptr ss:\[ebp+C\]  
7C85A274    8D45 F8         lea eax,dword ptr ss:\[ebp-8\]  
  
中断后,在 0012DC8C"在反汇编窗口中跟随"。  
  
堆栈中：  
\=======================================================================  
0012DC8C   00439D86   /CALL 到 WaitForDebugEvent 来自 乱序+远?00439D80  ; 在反汇编窗口中跟随  
0012DC90   0012ED7C   |pDebugEvent = 0012ED7C  
0012DC94   000003E8   \\Timeout = 1000. ms  
\=======================================================================  
\--新建 EIP--  
  
在反汇编窗口中跟随到这里，右键-->新建 EIP，积存器中 EIP 也变了。  
00439D86    85C0            test eax,eax             ; 新建 EIP。  
  
nop 掉 00439D76~~00439D81。  
  
00439D76   |0300            add eax,dword ptr ds:\[eax\]  
00439D78   |008B 95DCF5FF   add byte ptr ds:\[ebx+FFF5DC95\],c>  
00439D7E   |FF52 FF         call dword ptr ds:\[edx-1\]  
00439D81   |15 E0D04600     adc eax,<&KERNEL32.WaitForDebugE>  
  
\--改变跳转--  
  
将 00439D88 改变成 jmp 00401000。  
00439D86    85C0            test eax,eax  
00439D88    0F84 64270000   je 乱序+远?0043C4F2                ; 改变跳转。  
00439D8E    8B85 FCFDFFFF   mov eax,dword ptr ss:\[ebp-204\]  
00439D94    25 FF000000     and eax,0FF  
00439D99    85C0            test eax,eax  
00439D9B    74 13           je short 乱序+远?00439DB0  
  
改动后。  
\==========================================================  
00439D85    90              nop  
00439D86    85C0            test eax,eax  
00439D88  - E9 7372FCFF     jmp 乱序+远?00401000               ; 改动后。  
00439D8D    90              nop                                ; 记住这个地址！  
00439D8E    8B85 FCFDFFFF   mov eax,dword ptr ss:\[ebp-204\]  
\---------------------------------------------------------------------------------------------  
\--修改子进程的 3 个 OEP--  
  
在数据窗口，选择子进程的 3 个 OEP 值，Ctrl+E,编辑改成 400000。  
  
在转存中修改子进程 OEP=00400000。   
\===========================================================================  
0012ED7C  01 00 00 00 A4 0F 00 00 7C 0C 00 00 01 00 00 80  ...?..|.....€  
0012ED8C  00 00 00 00 00 00 00 00 00 00 40 00 02 00 00 00  ..........@....  
0012ED9C  00 00 00 00 00 00 40 00 00 00 40 00 06 00 00 00  ......@...@....  
0012EDAC  00 00 00 00 E8 1C 88 B4 00 00 00 00 00 00 00 00  ....?埓........  
0012EDBC  13 00 00 00 00 0D 88 00 64 CD 45 B4 F1 2F 4E 80  .....?d 虴瘩/N€  
0012EDCC  00 00 00 00 CC 10 40 00 01 00 00 00 01 00 00 00  ....?@.......  
0012EDDC  01 EE 12 00 00 00 00 00 00 00 00 00 00 00 00 00  ?.............  
\===========================================================================  
  
Ctrl+G：0012ED94，记住下面地址  
\=======================================  
0012ED94   00400000   乱序+远?00400000        ; 补丁用地址 1  
0012ED98   00000002  
0012ED9C   00000000  
0012EDA0   00400000   乱序+远?00400000        ; 补丁用地址 2  
0012EDA4   00400000   乱序+远?00400000        ; 补丁用地址 3  
\=======================================  
现在，可以从 00439D88 开始 F8 步进，在 00439D88 跳转处，跳到了 00401000。  
  
注意：清除所有断点！  
\---------------------------------------------------------------------------------------------  
\--我们开始键入以下补丁--  
  
00401000    8105 A8CD1200 > add dword ptr ds:\[12ED94\],1000      ; 补丁用地址 1  
0040100A    8105 B4CD1200 > add dword ptr ds:\[12EDA0\],1000      ; 补丁用地址 2  
00401014    8105 B8CD1200 > add dword ptr ds:\[12EDA4\],1000      ; 补丁用地址 3  
0040101E    813D B8CD1200 > cmp dword ptr ds:\[12EDA4\],00405000  
00401028  - 0F85 B8544F00   jnz 00439D8D                          
0040102E    68 2C090000     push 0FA4          ★PID★  
00401033    E8 5993457C     call DebugActiveProcessStop  
00401038    90              nop                                  ; 这里中断。  
00401039    90              nop                                   
  
中断后堆栈中。  
\===========================================================================  
0012ED7C  01 00 00 00 A4 0F 00 00 7C 0C 00 00 01 00 00 80  ...?..|.....€  
0012ED8C  00 00 00 00 00 00 00 00 00 50 40 00 02 00 00 00  .........P@....  
0012ED9C  00 00 00 00 00 50 40 00 00 50 40 00 06 00 00 00  .....P@..P@....  
0012EDAC  00 00 00 00 E8 1C 88 B4 00 00 00 00 00 00 00 00  ....?埓........  
0012EDBC  13 00 00 00 00 0D 88 00 64 CD 45 B4 F1 2F 4E 80  .....?d 虴瘩/N€  
0012EDCC  00 00 00 00 CC 10 40 00 01 00 00 00 01 00 00 00  ....?@.......  
0012EDDC  01 EE 12 00 00 00 00 00 00 00 00 00 00 00 00 00  ?.............  
\===========================================================================  
看积存器中，EAX=1，OK。  
EAX=00000001，就意味着儿子脱离父亲了。  
\-----------------------------------------------------------------------------------------------  
关闭 OllyDbg，再重新打开，选择附加--->子进程 PID=0FA4，现在只有一个。  
  
停在这里了，F9，F12。  
  
7C921231    C3        retn  
7C921232    8BFF      mov edi,edi  
7C921234    90        nop  
7C921235    90        nop  
  
到了这里。  
004010CC  - EB FE     jmp short IAT+SP.004010CC  
004010CE    EC        in al,dx  
004010CF    83EC 44   sub esp,44  
004010D2    56        push esi  
004010D3    FF15 6414>call dword ptr ds:\[D91464\]  
  
Ctrl+E，将子进程的入口改回来， EB FE-->55 8B    
  
改变后。  
004010CC    55        push ebp  
004010CD    8BEC      mov ebp,esp  
004010CF    83EC 44   sub esp,44  
004010D2    56        push esi  
004010D3    FF15 6414>call dword ptr ds:\[D91464\]  
  
Ctrl+B， FF 25  
  
到这里了，右键-->数据窗口跟随-->内存地址，数据窗口右键-->长型-->地址。  
  
00404FAA  - FF25 4811>jmp dword ptr ds:\[D91148\]      ; comdlg32.CommDlgExtendedError  
00404FB0  - FF25 9813>jmp dword ptr ds:\[D91398\]      ; comdlg32.GetSaveFileNameA  
00404FB6  - FF25 5811>jmp dword ptr ds:\[D91158\]      ; comdlg32.PageSetupDlgA  
00404FBC  - FF25 F011>jmp dword ptr ds:\[D911F0\]      ; comdlg32.FindTextA  
00404FC2  - FF25 3413>jmp dword ptr ds:\[D91334\]      ; comdlg32.ChooseFontA  
00404FC8  - FF25 5814>jmp dword ptr ds:\[D91458\]      ; comdlg32.GetOpenFileNameA  
00404FCE  - FF25 3013>jmp dword ptr ds:\[D91330\]      ; comdlg32.GetFileTitleA  
00404FD4    0000      add byte ptr ds:\[eax\],al  
  
到数据窗口中。  
00D91148  763300CE  comdlg32.CommDlgExtendedError  
00D9114C  00BA7405  
00D91150  00BA73EE  
00D91154  00BA751D  
00D91158  763447B1  comdlg32.PageSetupDlgA  
00D9115C  7C826E0C  kernel32.GetDateFormatA  
  
向上回滚，到 IAT START  
00D91104  77EF6CA6  GDI32.DeleteDC  
00D91108  00BA7446  
00D9110C  00BA7519  
00D91110  77EF5D0B  GDI32.SetBkMode  
00D91114  77D1DED3  USER32.SetWindowLongA  
00D91118  7C822D88  kernel32.LocalLock  
00D9111C  77F0E45F  GDI32.SetViewportExtEx  
。。。。。。  
向下回滚，到 IAT END  
00D91510  77D2190B  USER32.CreateWindowExA  
00D91514  00BA73B9  
00D91518  77EFA8F7  GDI32.SetMapMode  
00D9151C  00BA74E9  
00D91520  00BA73F7  
00D91524  7C822D47  kernel32.GetProfileStringA  
  
IAT START：00D91104  
IAT   END：00D91524  
Size: 00D91524-00D91104=420  
  
选择 IAT START 到 IAT END,等待粘贴新的 IAT。  
\----------------------------------------------------------------------------------------------  
打开 ArmaDetach，拖拽未脱壳的程序到它的上面，稍候数秒启动结束。  
注意上面有子进程 PID，入口改动的 4 个字节。  
\=============================  
DONE!  
Child process ID: 00000B14  
Entry point: 00449C43  
Original bytes: 558B  
\=============================  
再新打开一个 OD，选择新启动的子进程 PID=B14，附加。F9，F12，修改入口字节。  
7C921231    C3        retn  
7C921232    8BFF      mov edi,edi  
7C921234    90        nop  
7C921235    90        nop  
  
F9，F12。修改入口字节。  
00449C43 >- EB FE     jmp short 乱序+远?<ModuleEntryPoint>  
00449C45    EC        in al,dx  
00449C46    6A FF     push -1  
00449C48    68 902F47>push 乱序+远?00472F90  
00449C4D    68 809944>push 乱序+远?00449980  
00449C52    64:A1 000>mov eax,dword ptr fs:\[0\]  
  
Ctrl+E，修改入口字节 55 8B。  
00449C43 >  55        push ebp  
00449C44    8BEC      mov ebp,esp  
00449C46    6A FF     push -1  
00449C48    68 902F47>push 乱序+远?00472F90  
00449C4D    68 809944>push 乱序+远?00449980  
\---------------------------------------------------------------------------------------------  
\--Magic Jump--  
  
下断：he GetModuleHandleA。  
到这里了。  
7C80B529 >  8BFF      mov edi,edi  
7C80B52B    55        push ebp  
7C80B52C    8BEC      mov ebp,esp  
7C80B52E    837D 08 0>cmp dword ptr ss:\[ebp+8\],0  
7C80B532    74 18     je short kernel32.7C80B54C  
\=====================================================================  
0012FF38   00449D0B  /CALL 到 GetModuleHandleA 来自 乱序+远?00449D05  
0012FF3C   00000000  \\pModule = NULL  
  
0012E5F0   74683BEE  /CALL 到 GetModuleHandleA 来自 74683BE8  
0012E5F4   0012E5F8  \\pModule = "D:\\WINDOWS.0\\system32\\ntdll.dll"  
  
0012E5F8   74683BEE  /CALL 到 GetModuleHandleA 来自 74683BE8  
0012E5FC   0012E600  \\pModule = "D:\\WINDOWS.0\\system32\\imm32.dll"  
  
0012E544   74683BEE  /CALL 到 GetModuleHandleA 来自 74683BE8  
0012E548   0012E54C  \\pModule = "D:\\WINDOWS.0\\system32\\KERNEL32"  
  
0012ECFC   7365D4A4  /CALL 到 GetModuleHandleA 来自 msctfime.7365D49E  
0012ED00   0012ED04  \\pModule = "D:\\WINDOWS.0\\system32\\ntdll.dll"  
  
0012EE4C   5D175394  /CALL 到 GetModuleHandleA 来自 5D17538E  
0012EE50   5D1753E0  \\pModule = "kernel32.dll"  
  
0012EF0C   77F45BB0  /CALL 到 GetModuleHandleA 来自 SHLWAPI.77F45BAA  
0012EF10   77F44FF4  \\pModule = "KERNEL32.DLL"  
  
0012F73C   004348F3  /CALL 到 GetModuleHandleA 来自 乱序+远?004348ED  
0012F740   00000000  \\pModule = NULL  
  
00129528   00BB7105  /CALL 到 GetModuleHandleA 来自 00BB70FF  
0012952C   00BCBC1C  \\pModule = "kernel32.dll"  
00129530   00BCCEC4  ASCII "VirtualAlloc"  
  
00129528   00BB7122  /CALL 到 GetModuleHandleA 来自 00BB711C  
0012952C   00BCBC1C  \\pModule = "kernel32.dll"  
00129530   00BCCEB8  ASCII "VirtualFree"  
  
0012928C   00BA5FC9  /CALL 到 GetModuleHandleA 来自 00BA5FC3  
00129290   001293DC  \\pModule = "kernel32.dll"  
  
到了，取消断点，Alt+F9 返回。  
  
00BA5FC9    8B0D AC40>mov ecx,dword ptr ds:\[BD40AC\]  
00BA5FCF    89040E    mov dword ptr ds:\[esi+ecx\],eax  
00BA5FD2    A1 AC40BD>mov eax,dword ptr ds:\[BD40AC\]  
00BA5FD7    391C06    cmp dword ptr ds:\[esi+eax\],ebx  
00BA5FDA    75 16     jnz short 00BA5FF2  
00BA5FDC    8D85 B4FE>lea eax,dword ptr ss:\[ebp-14C\]  
00BA5FE2    50        push eax  
00BA5FE3    FF15 BC62>call dword ptr ds:\[BC62BC\]                
00BA5FE9    8B0D AC40>mov ecx,dword ptr ds:\[BD40AC\]  
00BA5FEF    89040E    mov dword ptr ds:\[esi+ecx\],eax  
00BA5FF2    A1 AC40BD>mov eax,dword ptr ds:\[BD40AC\]  
00BA5FF7    391C06    cmp dword ptr ds:\[esi+eax\],ebx  
00BA5FFA    0F84 2F01>je 00BA612F                          ; Magic Jump，改 jmp！  
00BA6000    33C9      xor ecx,ecx  
00BA6002    8B07      mov eax,dword ptr ds:\[edi\]  
00BA6004    3918      cmp dword ptr ds:\[eax\],ebx  
00BA6006    74 06     je short 00BA600E  
\----------------------------------------------------------------------------------------------  
\--OEP--   
  
下断 BP GetCurrentThreadId。  
  
7C809737 >  64:A1 180>mov eax,dword ptr fs:\[18\]  
7C80973D    8B40 24   mov eax,dword ptr ds:\[eax+24\]  
7C809740    C3        retn  
\=====================================================================  
00128B34   66001E3A  /CALL 到 GetCurrentThreadId 来自 66001E34  
00128B38   00000001  
00128B3C   66001C1E  返回到 66001C1E 来自 66001DE6  
00128B40   66001B64  返回到 66001B64 来自 66001B90  
  
00128B14   66003505  /CALL 到 GetCurrentThreadId 来自 660034FF  
00128B18   00000000  
00128B1C   6610E470  
  
0012F648   746908FE  /CALL 到 GetCurrentThreadId 来自 MSCTF.746908F8  
0012F64C   00000004  
0012F650   00050004  
  
0012F620   7468FA53  /CALL 到 GetCurrentThreadId 来自 MSCTF.7468FA4D  
0012F624   00000000  
  
0012F648   746908FE  /CALL 到 GetCurrentThreadId 来自 MSCTF.746908F8  
0012F64C   00000004  
0012F650   00050004  
  
0012F648   746908FE  /CALL 到 GetCurrentThreadId 来自 MSCTF.746908F8  
0012F64C   00000004  
0012F650   00050004  
  
0012F6A0   77D6B4E2  /CALL 到 GetCurrentThreadId 来自 USER32.77D6B4DC  
0012F6A4   0012F6AC  
0012F6A8   0072A768  
  
0012F720   00BA63D4  /CALL 到 GetCurrentThreadId 来自 00BA63CE  
0012F724   00000000  
0012F728   00000164  
  
到了，取消断点，Alt+F9 返回。  
00BA63D4    50              push eax  
00BA63D5    FF75 FC         push dword ptr ss:\[ebp-4\]  
00BA63D8    E8 05000000     call 00BA63E2  
00BA63DD    83C4 0C         add esp,0C  
00BA63E0    C9              leave  
00BA63E1    C3              retn  
。。。。。。。。。。。。  
来到这里  
00BBF9A5    030D FC00BD00   add ecx,dword ptr ds:\[BD00FC\]           
00BBF9AB    8B17            mov edx,dword ptr ds:\[edi\]  
00BBF9AD    85D2            test edx,edx  
00BBF9AF    75 18           jnz short 00BBF9C9  
00BBF9B1    FF77 18         push dword ptr ds:\[edi+18\]  
00BBF9B4    FF77 14         push dword ptr ds:\[edi+14\]  
00BBF9B7    FF77 10         push dword ptr ds:\[edi+10\]  
00BBF9BA    8B50 60         mov edx,dword ptr ds:\[eax+60\]  
00BBF9BD    3350 44         xor edx,dword ptr ds:\[eax+44\]  
00BBF9C0    3350 1C         xor edx,dword ptr ds:\[eax+1C\]  
00BBF9C3    2BCA            sub ecx,edx  
00BBF9C5    FFD1            call ecx  
00BBF9C7    EB 1D           jmp short 00BBF9E6  
00BBF9C9    83FA 01         cmp edx,1  
00BBF9CC    75 1B           jnz short 00BBF9E9  
00BBF9CE    FF77 04         push dword ptr ds:\[edi+4\]  
00BBF9D1    FF77 08         push dword ptr ds:\[edi+8\]  
00BBF9D4    6A 00           push 0  
00BBF9D6    FF77 0C         push dword ptr ds:\[edi+C\]  
00BBF9D9    8B50 60         mov edx,dword ptr ds:\[eax+60\]  
00BBF9DC    3350 44         xor edx,dword ptr ds:\[eax+44\]  
00BBF9DF    3350 1C         xor edx,dword ptr ds:\[eax+1C\]  
00BBF9E2    2BCA            sub ecx,edx  
00BBF9E4    FFD1            call ecx                               ; 里面就是 OEP，进去看！  
00BBF9E6    8945 E4         mov dword ptr ss:\[ebp-1C\],eax  
00BBF9E9    8B45 E4         mov eax,dword ptr ss:\[ebp-1C\]  
进来看，是吧。  
004010CC    55              push ebp                               ; OEP！  
004010CD    8BEC            mov ebp,esp  
004010CF    83EC 44         sub esp,44  
004010D2    56              push esi  
004010D3    FF15 E4634000   call dword ptr ds:\[4063E4\]             ;   
  
kernel32.GetCommandLineA  
004010D9    8BF0            mov esi,eax  
004010DB    8A00            mov al,byte ptr ds:\[eax\]  
004010DD    3C 22           cmp al,22  
004010DF    75 1B           jnz short TEST-2.004010FC  
\----------------------------------------------------------------------------------------------  
\--找到干净的 IAT--  
  
在“命令”中键入“? USER32.DispatchMessageA”，回车。  
Alt+M 到内存窗口, Ctrl+B，搜索“命令”行右边，HEX：XXXXXXXX  
OK，搜索。  
Alt+C 到 CPU 窗口, Ctrl+G：00D81930  转存窗口首行地址。  
右键-->在数据窗口跟随-->选择部分，在数据窗口向上回滚，找到 IAT  
  
在数据窗口向上回滚，找到 IAT START  
00D8194C  77EF6CA6  GDI32.DeleteDC  
00D81950  00BA7446  
00D81954  00BA7519  
00D81958  77EF5D0B  GDI32.SetBkMode  
00D8195C  77D1DED3  USER32.SetWindowLongA  
00D81960  7C822D88  kernel32.LocalLock  
00D81964  77F0E45F  GDI32.SetViewportExtEx  
。。。。。。  
在数据窗口向下回滚，找到 IAT END  
00D81D60  77EFA8F7  GDI32.SetMapMode  
00D81D64  00BA74E9  
00D81D68  00BA73F7  
00D81D6C  7C822D47  kernel32.GetProfileStringA  
00D81D70  7C827778  kernel32.\_lcreat  
  
IAT START：00D8194C  
IAT   END：00D81D70  
Size: 00D81D70-00D8194C=424  
  
选择 IAT START 到 IAT END,二进制复制 IAT。  
\----------------------------------------------------------------------------------------------  
\--IAT Elimination + Code Splicing 修复--  
  
到另一个 OD，即附加子进程，选择 IAT 开始到 IAT 结束的部分二进制粘贴  
  
我们已经找到并中断在 OEP 处，开始解决 IAT Elimination+code splicing 了。  
  
Alt+M， 打开内存窗口，  
  
内存映射，项目 23  
 地址=00401000  
 大小=00004000 (16384.)  
 物主=IAT+SP   00400000  
 区段=.text  
 类型=Imag 01001002  
 访问=R  
 初始访问=RWE  
  
点访问---->排序,找第一个类型为 priv 访问=RWE 的项目。  
  
内存映射，项目 52  
 地址=01E20000  
 大小=00012000 (73728.)  
 物主=         01E20000 （自身）  
 区段=  
 类型=Priv 00021020  
 访问=R E  
 初始访问=RWE  
  
打开 arminline0.7。  
  
  
\[sleiv\] process ID = 看文件-->附加-->ID  
Start  Of Target Code = 00401000    
Length Of Target Code = 00004000  
  
在 ArmInline 的 Code Splicing 选项添入：  
\===================================  
Start  Of Spliced Code = 01E20000  
Length Of spliced Code = 00012000  
  
点“remove splicing”。  
Patch succesful 修复成功。  
\-----------------------------------  
在 Import Elimination 选项添入：  
Base Of Existing IAT=00D91104  
Length Of Existing IAT=424  
New Base RVA Of IAT=497000  
  
点“rebase IAT”。  
IAT Elimination 修复成功。  
\===================================  
打开 loardpe  选择 antitracks dump 完整转存。  
  
再打开 importrec1.6   选择 antitracks   
  
填 OEP=10CC, RVA=97000 ,  Size=500.  
  
getimport 获取输入表，cut 无效函数，有 138 个指针。  
  
fixdump  修复抓取文件，保存为 dump.exe。  
  
用 PeiD 看为 Microsoft Visual C++ 6.0 SPx Method。  
\----------------------------------------------------------------------------------------------  
OK，结束。  
  
为方便朋友们练手，附上加壳记事本。 

 4dd466d0e1e0cc123afbe504da2462ca