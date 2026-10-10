

# 大侠勿进！Armadillo 最强保护方式\_XP 变脸王脱壳笔记【原创】

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-550.htm`](https://www.pediy.com/kssd/pediy08/pediy8-550.htm)

Armadillo 最强保护方式\_XP 变脸王脱壳笔记  
  
【脱壳软件】XP 变脸王 v7.0.4.0  
【脱壳工具】OllyDBG LordPE Import REC ArmInline 0.71  
【保护方式】Protected Armadillo  
 CopyMem-II  
 Enable Strategic Code Splicing  
 Enable Nanomites Processing  
 Enable Memory-Patching Protections  
  
  
OD 载入。。。。。。  
  
0078BF43 >  55               push ebp                                           ; 入口  
0078BF44    8BEC             mov ebp,esp  
0078BF46    6A FF            push -1  
0078BF48    68 E84F7B00      push XPStyle.007B4FE8  
0078BF4D    68 80BC7800      push XPStyle.0078BC80  
0078BF52    64:A1 00000000   mov eax,dword ptr fs:\[0\]  
0078BF58    50               push eax  
0078BF59    64:8925 00000000 mov dword ptr fs:\[0\],esp  
0078BF60    83EC 58          sub esp,58  
0078BF63    53               push ebx  
0078BF64    56               push esi  
0078BF65    57               push edi  
0078BF66    8965 E8          mov dword ptr ss:\[ebp-18\],esp  
0078BF69    FF15 8CF17A00    call dword ptr ds:\[<&KERNEL32.GetV>; kernel32.GetVersion  
\----------------------------------------------------------------------------------------------------------  
下断:BP WaitForDebugEvent  
  
7C85A268 >  8BFF             mov edi,edi  
7C85A26A    55               push ebp  
7C85A26B    8BEC             mov ebp,esp  
7C85A26D    83EC 68          sub esp,68  
7C85A270    56               push esi  
7C85A271    FF75 0C          push dword ptr ss:\[ebp+C\]  
7C85A274    8D45 F8          lea eax,dword ptr ss:\[ebp-8\]  
7C85A277    50               push eax  
7C85A278    E8 F381FAFF      call kernel32.7C802470  
  
堆栈窗口(Stack window)看到关于这个 API 的所有参数信息。  
\========================================================================  
0012DC8C   0077C056  /CALL 到 WaitForDebugEvent 来自 XPStyle.0077C050  
0012DC90   0012ED7C  |pDebugEvent = 0012ED7C  
0012DC94   000003E8  \\Timeout = 1000. ms  
\========================================================================  
▲pDebugEvent=0012ED7C 数据窗口跟随，数据窗口一直保持在这里，到结束。▲  
  
BC WaitForDebugEvent 清除这个断点。当然也可以 F2 取消。  
\----------------------------------------------------------------------------------------------------------  
2、BP WriteProcessMemory，Shirt+F9 ,到 BytesToWrite=1000。  
  
7C80220F >  8BFF             mov edi,edi  
7C802211    55               push ebp  
7C802212    8BEC             mov ebp,esp  
7C802214    51               push ecx  
7C802215    51               push ecx  
7C802216    8B45 0C          mov eax,dword ptr ss:\[ebp+C\]  
7C802219    53               push ebx  
7C80221A    8B5D 14          mov ebx,dword ptr ss:\[ebp+14\]  
7C80221D    56               push esi  
7C80221E    8B35 B812807C    mov esi,dword ptr ds:\[<&ntdll.NtPr>; ntdll.ZwProtectVirtualMemory  
  
父进程将要复制给子进程第一个块的信息,在堆栈窗口中：   
\===========================================================================  
0012DB2C   00780055  /CALL 到 WriteProcessMemory 来自 XPStyle.0078004F  
0012DB30   0000004C  |hProcess = 0000004C (window)  
0012DB34   00689000  |Address = 689000  
0012DB38   01230048  |Buffer = 01230048  
0012DB3C   00001000  |BytesToWrite = 1000 (4096.)  
0012DB40   0012DC48  \\pBytesWritten = 0012DC48  
\============================================================================  
数据窗口中：  
0012ED7C  01 00 00 00 58 09 00 00 04 09 00 00 01 00 00 80  ...X........€  
0012ED8C  00 00 00 00 00 00 00 00 BC 92 68 00 02 00 00 00  ........紥 h....  
0012ED9C  00 00 00 00 BC 92 68 00 BC 92 68 00 06 00 00 00  ....紥 h.紥 h....  
0012EDAC  00 00 00 00 E8 8C 5C B4 00 00 00 00 00 00 00 00  ....鑼\\?.......  
0012EDBC  13 00 00 00 00 7D 5C 00 64 6D A1 B4 F1 2F 4E 80  ....}\\.dm〈?N€  
0012EDCC  00 00 00 00 BC 92 68 00 01 00 00 00 01 00 00 00  ....紥 h.......  
0012EDDC  01 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ...............  
  
得到：OEP=6892BC  PID=0958  
取消断点：BC WriteProcessMemory  
\============================================================================  
\--nop 加密 CLL--  
  
在堆栈窗口，向下看有一个主程序返回和小循环。  
  
0012DC3C   00689000  XPStyle.00689000  
0012DC40   00000020  
0012DC44   01231048  
0012DC48   00001000  
0012DC4C   01231048  
0012DC50  /0012DC84  
0012DC54  |0077ECC3  返回到 XPStyle.0077ECC3 来自 XPStyle.0077F00B           ; 在这里。  
  
在 CPU 窗口，Ctrl+G：0077F00B  
0077F00B    55               push ebp                                        ; 到这里。    
0077F00C    8BEC             mov ebp,esp  
0077F00E    81EC 00010000    sub esp,100  
0077F014    53               push ebx  
0077F015    56               push esi  
0077F016    57               push edi  
0077F017    8B45 08          mov eax,dword ptr ss:\[ebp+8\]  
  
Alt+R  
\==============================================================================  
参考位于 XPStyle:.text 到 0077F00B  
地址        反汇编                                           注释  
0077ECBE    call XPStyle.0077F00B  
0077EF93    call XPStyle.0077F00B       ☆双击  
0077F00B    push ebp                                         （初始 CPU 选择）  
\==============================================================================  
双击 0077EF93 行，来到这里。  
0077EF93    E8 73000000      call XPStyle.0077F00B                           ; nop 这个 CALL  
0077EF98    83C4 0C          add esp,0C  
0077EF9B    50               push eax  
0077EF9C    F7D0             not eax  
0077EF9E    0FC8             bswap eax  
0077EFA0    58               pop eax  
0077EFA1    73 00            jnb short XPStyle.0077EFA3  
\---------------------------------------------------------------------------------------------------------  
回到堆栈窗口。在反汇编窗口中跟随下面。  
0012DB2C   00780055  /CALL 到 WriteProcessMemory 来自 XPStyle.0078004F       ; 在这里跟随。  
  
到了这里。在 00780055 下断，F9，注意数据窗口中父进程 OEP 的变化后停止。  
0078004F    FF15 14F17A00    call dword ptr ds:\[<&KERNEL32.Writ>; kernel32.WriteProcessMemory  
00780055    85C0             test eax,eax  
00780057    75 4B            jnz short XPStyle.007800A4  
00780059    50               push eax  
0078005A    F7D0             not eax  
0078005C    0FC8             bswap eax  
0078005E    58               pop eax  
0078005F    73 00            jnb short XPStyle.00780061  
\----------------------------------------------------------------------------------------------------------  
\---修改 OEP 处的子进程为无限循环---  
  
回到堆栈窗口。在反汇编窗口中跟随。  
  
好了，从 6892BC 变成 407010，这个就是子进程的 OEP！PID=0950  
\=============================================================================  
0012ED7C  01 00 00 00 50 09 00 00 64 03 00 00 01 00 00 80  ...P...d....€  
0012ED8C  00 00 00 00 00 00 00 00 10 70 40 00 02 00 00 00  ........p@....  
0012ED9C  00 00 00 00 10 70 40 00 10 70 40 00 06 00 00 00  ....p@.p@....  
0012EDAC  00 00 00 00 E8 8C 5C B4 00 00 00 00 00 00 00 00  ....鑼\\?.......  
0012EDBC  13 00 00 00 00 7D 5C 00 64 1D 92 B2 F1 2F 4E 80  ....}\\.d 挷?N€  
0012EDCC  00 00 00 00 10 70 40 00 01 00 00 00 01 00 00 00  ....p@.......  
0012EDDC  01 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ...............  
\=============================================================================  
在堆栈窗口 Ctrl+G：0012ED7C  
\========================================================  
0012ED94  |00407010  XPStyle.00407010    ★子进程 OEP★  
0012ED98  |00000002  
0012ED9C  |00000000  
0012EDA0  |00407010  XPStyle.00407010  
0012EDA4  |00407010  XPStyle.00407010  
\=======================================================  
▲子进程 OEP 为 00407010,注意，我们稍后会需要这三个地址。▲  
  
打开 PUPE,打开程序(记得使用正确的进程),键入子进程的 OEP=00407010。  
     
选择子进程 PID=D0C  
     
记下原字节: 55 B8， 修改为：EBFE。  
  
关闭 PUPE，回到 Olly。  
  
清除 BC 00780055  
\----------------------------------------------------------------------------------------------------------  
\--新建 EIP--  
  
下断 bp WaitForDebugEvent ，F9。  
\===================================================================================  
0012DC8C   0077C056  /CALL 到 WaitForDebugEvent 来自 XPStyle.0077C050 ☆在 CPU 中跟随  
0012DC90   0012ED7C  |pDebugEvent = 0012ED7C  
0012DC94   000003E8  \\Timeout = 1000. ms  
\===================================================================================  
取消 bc WaitForDebugEvent  
  
在反汇编窗口中跟随到这里，右键-->新建 EIP，积存器中 EIP 也变了。  
跟随到这里，0077C056 新建 EIP。  
0077C056    85C0             test eax,eax                         
0077C058    0F84 64270000    je XPStyle.0077E7C2  
0077C05E    8B85 FCFDFFFF    mov eax,dword ptr ss:\[ebp-204\]  
0077C064    25 FF000000      and eax,0FF  
0077C069    85C0             test eax,eax  
0077C06B    74 13            je short XPStyle.0077C080  
0077C06D    8B0D 40657B00    mov ecx,dword ptr ds:\[7B6540\]  
0077C073    8379 20 00       cmp dword ptr ds:\[ecx+20\],0  
0077C077    74 07            je short XPStyle.0077C080  
  
往上看，nop 掉以下部分。  
0077C046    0300             add eax,dword ptr ds:\[eax\]  
0077C048    008B 95DCF5FF    add byte ptr ds:\[ebx+FFF5DC95\],cl  
0077C04E    FF52 FF          call dword ptr ds:\[edx-1\]  
0077C051    15 E4F07A00      adc eax,<&KERNEL32.WaitForDebugEve>  
  
\--改变跳转--  
  
将 0077C058 改变成 jmp 00401000。  
0077C058   /0F84 64270000    je XPStyle.0077E7C2  
0077C05E   |8B85 FCFDFFFF    mov eax,dword ptr ss:\[ebp-204\]  
0077C064   |25 FF000000      and eax,0FF  
0077C069   |85C0             test eax,eax  
0077C06B   |74 13            je short XPStyle.0077C080  
  
改动后。  
0077C056    85C0             test eax,eax                       ; 改动后。  
0077C058  - 0F84 A24FC8FF    je XPStyle.00401000                ; 记住这个地址！  
0077C05E    8B85 FCFDFFFF    mov eax,dword ptr ss:\[ebp-204\]  
0077C064    25 FF000000      and eax,0FF  
0077C069    85C0             test eax,eax  
0077C06B    74 13            je short XPStyle.0077C080  
  
\--修改子进程的 3 个 OEP--  
  
在数据窗口，选择子进程的 3 个 OEP 值，Ctrl+E,编辑改成 400000。  
  
在转存中修改子进程 OEP=00400000。   
\======================================  
0012ED94   00400000  ASCII "MZP"         ; 补丁用地址 1  
0012ED98   00000002  
0012ED9C   00000000  
0012EDA0   00400000  ASCII "MZP"         ; 补丁用地址 2  
0012EDA4   00400000  ASCII "MZP"         ; 补丁用地址 3  
\======================================  
现在，可以从 0077C056 开始 F8 步进，在 0077C058 跳转处，跳到了 00401000。  
  
注意：清除所有断点！  
\----------------------------------------------------------------------------------------------------------  
\--我们开始键入以下补丁--  
  
00401000    8105 A8CD1200 > add dword ptr ds:\[12ED94\],1000      ; 补丁用地址 1  
0040100A    8105 B4CD1200 > add dword ptr ds:\[12EDA0\],1000      ; 补丁用地址 2  
00401014    8105 B8CD1200 > add dword ptr ds:\[12EDA4\],1000      ; 补丁用地址 3  
0040101E    813D B8CD1200 > cmp dword ptr ds:\[12EDA4\],68A000  
00401028  - 0F85 B8544F00   jnz 0077C05D                          
0040102E    68 2C090000     push 0FA4          ★PID★  
00401033    E8 5993457C     call DebugActiveProcessStop  
00401038    90              nop                                  ; 这里中断。  
00401039    90              nop                                   
  
看积存器中，EAX=1，OK。  
EAX=00000001，就意味着子进程完全脱离父进程了。  
关闭这个 OD，  
\----------------------------------------------------------------------------------------------------------  
下面对子进程开始修复。  
  
再启动 OD(调试 OD)，文件--->附加--->PID(尽管我们关闭了 OD，子进程还在内存中)。  
  
停在这里了。  
7C921231    C3               retn  
7C921232    8BFF             mov edi,edi  
7C921234    90               nop  
7C921235    90               nop  
7C921236    90               nop  
7C921237    90               nop  
7C921238    90               nop  
7C921239 >  CC               int3  
7C92123A    C3               retn  
  
F9，F12。  
  
006892BC  - EB FE            jmp short XPStyle.006892BC          
006892BE    EC               in al,dx  
006892BF    83C4 EC          add esp,-14  
006892C2    33C0             xor eax,eax  
006892C4    8945 EC          mov dword ptr ss:\[ebp-14\],eax  
006892C7    B8 9C8B6800      mov eax,XPStyle.00688B9C  
006892CC    E8 3FDDD7FF      call XPStyle.00407010  
  
Ctrl+E，将子进程的入口改回来， EB FE-->558B  
  
006892BC    55               push ebp                            
006892BD    8BEC             mov ebp,esp  
006892BF    83C4 EC          add esp,-14  
006892C2    33C0             xor eax,eax  
006892C4    8945 EC          mov dword ptr ss:\[ebp-14\],eax  
006892C7    B8 9C8B6800      mov eax,XPStyle.00688B9C  
006892CC    E8 3FDDD7FF      call XPStyle.00407010  
  
  
F7 步进到 006892CC 这个 CALL，到子进程 OEP 了。  
00407010    53               push ebx  
00407011    8BD8             mov ebx,eax                       ; XPStyle.00688B9C  
00407013    33C0             xor eax,eax  
00407015    A3 CCA06800      mov dword ptr ds:\[68A0CC\],eax  
0040701A    6A 00            push 0  
0040701C    E8 2BFFFFFF      call XPStyle.00406F4C  
  
继续 F7 到 0040701C 这个 CALL，看见 IAT 表。  
可是，第一个 00406F4C \[727338\]内缺少函数值！我们先来解决这个问题。  
  
00406F4A    C3               retn  
00406F4B    90               nop  
00406F4C  - FF25 38737200    jmp dword ptr ds:\[727338\]  
00406F52    8BC0             mov eax,eax  
00406F54  - FF25 34737200    jmp dword ptr ds:\[727334\]         ; kernel32.LocalAlloc  
00406F5A    8BC0             mov eax,eax  
00406F5C  - FF25 30737200    jmp dword ptr ds:\[727330\]         ; kernel32.TlsGetValue  
00406F62    8BC0             mov eax,eax  
00406F64  - FF25 2C737200    jmp dword ptr ds:\[72732C\]         ; kernel32.TlsSetValue  
00406F6A    8BC0             mov eax,eax  
00406F6C    50               push eax  
00406F6D    6A 40            push 40  
00406F6F    E8 E0FFFFFF      call XPStyle.00406F54             ; jmp to kernel32.LocalAlloc  
00406F74    C3               retn  
  
右键--->在数据窗口跟随--->内存地址。  
  
确实没有数值。  
  
用一个 arm 小工具 ArmaDetach，拖拽 XPStyle.exe 到 ArmaDetach 上。  
DONE!  
Child process ID: 00000CC4   PID  
Entry point: 0078BF43          
Original bytes: 558B         入口字节  
\----------------------------------------------------------------------------------------------------------  
再启动一个 OD(参照 OD)，文件--->附加--->PID=CC4  
  
F9，F12。  
0078BF43 >- EB FE            jmp short XPStyle.<ModuleEntryPoi>  
0078BF45    EC               in al,dx  
0078BF46    6A FF            push -1  
0078BF48    68 E84F7B00      push XPStyle.007B4FE8  
0078BF4D    68 80BC7800      push XPStyle.0078BC80  
0078BF52    64:A1 00000000   mov eax,dword ptr fs:\[0\]  
0078BF58    50               push eax  
0078BF59    64:8925 00000000 mov dword ptr fs:\[0\],esp  
  
Ctrl+E，将子进程的入口改回来， EB FE-->558B  
  
0078BF43 >  55               push ebp  
0078BF44    8BEC             mov ebp,esp  
0078BF46    6A FF            push -1  
0078BF48    68 E84F7B00      push XPStyle.007B4FE8  
0078BF4D    68 80BC7800      push XPStyle.0078BC80  
0078BF52    64:A1 00000000   mov eax,dword ptr fs:\[0\]  
0078BF58    50               push eax  
0078BF59    64:8925 00000000 mov dword ptr fs:\[0\],esp  
0078BF60    83EC 58          sub esp,58  
0078BF63    53               push ebx  
0078BF64    56               push esi  
0078BF65    57               push edi  
0078BF66    8965 E8          mov dword ptr ss:\[ebp-18\],esp  
0078BF69    FF15 8CF17A00    call dword ptr ds:\[<&KERNEL32.Get>; kernel32.GetVersion  
\----------------------------------------------------------------------------------------------------------  
这个参照 OD，现在已经变成了单进程，手动过程省略过。  
建议用 fly 大侠的 Armadillo V4.0-V4.4.Standard.Protection 脚本完成。  
  
006892BC    47               inc edi                           ; This is the OEP!  Found By: fly  
006892BD    019D 07D66642    add dword ptr ss:\[ebp+4266D607\],e>  
006892C3    44               inc esp  
006892C4    9B               wait  
006892C5    CF               iretd  
006892C6    9D               popfd  
006892C7    3C 8E            cmp al,8E  
006892C9    0119             add dword ptr ds:\[ecx\],ebx  
006892CB    84FA             test dl,bh  
006892CD    B5 AC            mov ch,0AC  
006892CF    53               push ebx  
006892D0    ED               in eax,dx  
006892D1    B9 B1D17A68      mov ecx,687AD1B1  
006892D6  ^ E2 EC            loopd short XPStyle.006892C4  
  
我们从调试 OD 知道，IAT 在地址 727000 附近，在数据窗口 CTRL+G：727000  
  
向下回滚，发现 IAT 开始：  
0072722C  00000000  
00727230  7C93188A  ntdll.RtlDeleteCriticalSection  
00727234  7C9210ED  ntdll.RtlLeaveCriticalSection  
00727238  7C921005  ntdll.RtlEnterCriticalSection  
0072723C  7C809FA1  kernel32.InitializeCriticalSection  
00727240  7C809B14  kernel32.VirtualFree  
00727244  7C809A81  kernel32.VirtualAlloc  
00727248  7C80995D  kernel32.LocalFree  
。。。。。。。。。。。。  
00727B7C  77455011  SHELL32.RestartDialog  
00727B80  01347550  
00727B84  73ACFB78  avifil32.AVIStreamGetFrameClose  
00727B88  73AC5E5F  avifil32.AVIStreamRelease  
00727B8C  73AC5E5F  avifil32.AVIStreamRelease  
00727B90  73AC589F  avifil32.AVIFileExit  
00727B94  0134748C  
  
选择 00727230~00727B90 段二进制复制。  
\----------------------------------------------------------------------------------------------------------  
回到调试 OD，选择 IAT 开始 00727340~00727BA4 IAT 结束部分。  
0072733C  01347451  
00727340  77DAEBE7  ADVAPI32.RegSetValueExA  
00727344  77DA7883  ADVAPI32.RegQueryValueExA  
00727348  77DCC1B5  ADVAPI32.RegQueryInfoKeyA  
0072734C  77DA761B  ADVAPI32.RegOpenKeyExA  
00727350  77DBB908  ADVAPI32.RegFlushKey  
00727354  77DCC8C1  ADVAPI32.RegEnumKeyExA  
00727358  77DAEDE5  ADVAPI32.RegDeleteValueA  
0072735C  77DCC123  ADVAPI32.RegDeleteKeyA  
00727360  0134AD7E  
。。。。。。。。。。。。。  
00727B90  73AC589F  avifil32.AVIFileExit  
00727B94  0134748C  
00727B98  73B423CA  MSVFW32.DrawDibRealize  
00727B9C  73B441C7  MSVFW32.DrawDibOpen  
00727BA0  73B4191A  MSVFW32.DrawDibDraw  
00727BA4  73B49AAD  MSVFW32.DrawDibClose  
  
00727BA4-00727340=864  
  
二进制粘贴，反汇编窗口已经补上，在数据窗口中有很多红色的改变项。  
这样，参考 OD 的任务已经完成，可以关闭了。  
  
00406F4A    C3               retn  
00406F4B    90               nop  
00406F4C  - FF25 38737200    jmp dword ptr ds:\[727338\]    ; kernel32.GetModuleHandleA  
00406F52    8BC0             mov eax,eax  
00406F54  - FF25 34737200    jmp dword ptr ds:\[727334\]    ; kernel32.LocalAlloc  
00406F5A    8BC0             mov eax,eax  
00406F5C  - FF25 30737200    jmp dword ptr ds:\[727330\]    ; kernel32.TlsGetValue  
00406F62    8BC0             mov eax,eax  
00406F64  - FF25 2C737200    jmp dword ptr ds:\[72732C\]    ; kernel32.TlsSetValue  
00406F6A    8BC0             mov eax,eax  
00406F6C    50               push eax  
00406F6D    6A 40            push 40  
00406F6F    E8 E0FFFFFF      call XPStyle.00406F54        ; jmp to kernel32.LocalAlloc  
\---------------------------------------------------------------------------------------------------------  
现在可以修复 Strategic Code Splicing+Nanomites Processing。  

 4dd466d0e1e0cc123afbe504da2462ca