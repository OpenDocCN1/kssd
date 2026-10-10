

# 一流 QQ 连连看助手 4.93

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-251.htm`](https://www.pediy.com/kssd/pediy08/pediy8-251.htm)

【文章标题】: 另辟蹊径解除某外挂试用版功能限制  
【文章作者】: 不知所谓  
【软件名称】: 一流 QQ 连连看助手 4.93 试用版  
【下载地址】: 华军软件园  
【使用工具】: OD 、Peid 、vb6  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教! 菜鸟水平，高手请移步～～  
\--------------------------------------------------------------------------------  
【详细过程】  
  任务目的：  
  1、去除弹出网页  
  2、去除启动 1 分钟后自动关闭功能  
    
  软件文件基本信息：  
  文件名        大小      Peid 查壳结果   
  LLKWG.exe     18kb      Borland Delphi 6.0 - 7.0  
  hook.dll     541kb      ASProtect 2.1x SKE -> Alexey Solodovnikov  
    
  外挂运行原理分析：  
  用 HideOD 插件隐藏 OD，载入 LLKWG.exe 并运行，主程序在 3 秒钟内就退出，外挂主界面却还在，可推知外挂主要功能在 Hook.dll 文件里，寄生于其它进程而运行。此时查看进程信息，发现外挂寄生在 Explorer.exe 里。  
  hook.dll 的输出表只有两个函数：HookOn、HookOff，函数实现相当简单，HookOn 里只调用了 SetWindowsHookEx，HookOff 里只调用 UnhookWindowsHookEx，显然二者都不是重点。HookProc 里只调用 CallNextHookEx，也不是我们想要的。  
  我们来看看主程序入口处的代码：  
  00403504 > $  55            push    ebp                              ;  程序入口  
  00403505   .  8BEC          mov     ebp, esp  
  00403507   .  83C4 F0       add     esp, -10  
  0040350A   .  B8 C4344000   mov     eax, 004034C4  
  0040350F   .  E8 B4FDFFFF   call    004032C8  
  00403514   .  68 00364000   push    00403600                         ; /MutexName = "EXE\_MUTEX\_LLK"  
  00403519   .  6A 00         push    0                                ; |Inheritable = FALSE  
  0040351B   .  68 01001F00   push    1F0001                           ; |Access = 1F0001  
  00403520   .  E8 9FFEFFFF   call    <jmp.&kernel32.OpenMutexA>       ; \\OpenMutexA  
  00403525   .  85C0          test    eax, eax  
  00403527   .  0F85 CC000000 jnz     004035F9  
  0040352D   .  68 10364000   push    00403610                         ; /MutexName = "DLL\_MUTEX\_LLK"  
  00403532   .  6A 00         push    0                                ; |Inheritable = FALSE  
  00403534   .  68 01001F00   push    1F0001                           ; |Access = 1F0001  
  00403539   .  E8 86FEFFFF   call    <jmp.&kernel32.OpenMutexA>       ; \\OpenMutexA  
  0040353E   .  85C0          test    eax, eax  
  00403540   .  0F85 B3000000 jnz     004035F9  
  00403546   .  68 00364000   push    00403600                         ; /Arg3 = 00403600 ASCII "EXE\_MUTEX\_LLK"  
  0040354B   .  6A 00         push    0                                ; |Arg2 = 00000000  
  0040354D   .  6A 00         push    0                                ; |Arg1 = 00000000  
  0040354F   .  E8 40FEFFFF   call    00403394                         ; \\LLKWG.00403394  
  00403554   .  8B15 A4404000 mov     edx, \[4040A4\]                    ;  LLKWG.00405668  
  0040355A   .  8902          mov     \[edx\], eax  
  0040355C   .  68 20364000   push    00403620                         ; /MapName = "SFILEMAP\_LLK"  
  00403561   .  6A 04         push    4                                ; |MaximumSizeLow = 4  
  00403563   .  6A 00         push    0                                ; |MaximumSizeHigh = 0  
  00403565   .  6A 04         push    4                                ; |Protection = PAGE\_READWRITE  
  00403567   .  6A 00         push    0                                ; |pSecurity = NULL  
  00403569   .  6A FF         push    -1                               ; |hFile = FFFFFFFF  
  0040356B   .  E8 14FEFFFF   call    <jmp.&kernel32.CreateFileMapping>; \\CreateFileMappingA  
  00403570   .  8B15 A8404000 mov     edx, \[4040A8\]                    ;  LLKWG.0040566C  
  00403576   .  8902          mov     \[edx\], eax  
  00403578   .  6A 00         push    0                                ; /MapSize = 0  
  0040357A   .  6A 00         push    0                                ; |OffsetLow = 0  
  0040357C   .  6A 00         push    0                                ; |OffsetHigh = 0  
  0040357E   .  6A 02         push    2                                ; |AccessMode = FILE\_MAP\_WRITE  
  00403580   .  A1 A8404000   mov     eax, \[4040A8\]                    ; |  
  00403585   .  8B00          mov     eax, \[eax\]                       ; |  
  00403587   .  50            push    eax                              ; |hMapObject  
  00403588   .  E8 2FFEFFFF   call    <jmp.&kernel32.MapViewOfFile>    ; \\MapViewOfFile  
  0040358D   .  8B15 A0404000 mov     edx, \[4040A0\]                    ;  LLKWG.00405670  
  00403593   .  8902          mov     \[edx\], eax  
  00403595   .  E8 1AFEFFFF   call    <jmp.&kernel32.GetCurrentThreadI>; \[GetCurrentThreadId  
  0040359A   .  8B15 A0404000 mov     edx, \[4040A0\]                    ;  LLKWG.00405670  
  004035A0   .  8B12          mov     edx, \[edx\]  
  004035A2   .  8902          mov     \[edx\], eax  
  004035A4   .  A1 A0404000   mov     eax, \[4040A0\]  
  004035A9   .  8B00          mov     eax, \[eax\]  
  004035AB   .  50            push    eax                              ; /BaseAddress  
  004035AC   .  E8 1BFEFFFF   call    <jmp.&kernel32.UnmapViewOfFile>  ; \\UnmapViewOfFile  
  004035B1   .  E8 D6FEFFFF   call    <jmp.&Hook.HookOn>               ; 调用后外挂主界面出现  
  004035B6   >  6A 00         push    0                                ; /MsgFilterMax = 0  
  004035B8   .  6A 00         push    0                                ; |MsgFilterMin = 0  
  004035BA   .  6A 00         push    0                                ; |hWnd = NULL  
  004035BC   .  68 78564000   push    00405678                         ; |pMsg = LLKWG.00405678  
  004035C1   .  E8 0EFEFFFF   call    <jmp.&user32.GetMessageA>        ; \\GetMessageA  
  004035C6   .  85C0          test    eax, eax  
  004035C8   .^ 75 EC         jnz     short 004035B6  
  004035CA   .  E8 C5FEFFFF   call    <jmp.&Hook.HookOff>  
  004035CF   .  A1 A8404000   mov     eax, \[4040A8\]  
  004035D4   .  8B00          mov     eax, \[eax\]  
  004035D6   .  50            push    eax                              ; /hObject  
  004035D7   .  E8 A0FDFFFF   call    <jmp.&kernel32.CloseHandle>      ; \\CloseHandle  
  004035DC   .  6A 00         push    0                                ; /lParam = 0  
  004035DE   .  6A 00         push    0                                ; |wParam = 0  
  004035E0   .  6A 1A         push    1A                               ; |Message = WM\_WININICHANGE  
  004035E2   .  68 FFFF0000   push    0FFFF                            ; |hWnd = HWND\_BROADCAST  
  004035E7   .  E8 F0FDFFFF   call    <jmp.&user32.PostMessageA>       ; \\PostMessageA   
  004035EC   .  A1 A4404000   mov     eax, \[4040A4\]  
  004035F1   .  8B00          mov     eax, \[eax\]  
  004035F3   .  50            push    eax                              ; /hObject  
  004035F4   .  E8 83FDFFFF   call    <jmp.&kernel32.CloseHandle>      ; \\CloseHandle  
  ...............................  
    
  显然，外挂用全局 Hook 进入其它进程内部，而判断是否 Explorer.exe 的代码在 DllMain 里。我决定带壳调试，用 Memory Patch 方法破解。用 OD 调试 Explorer.exe 会带来很多不便，这里先做一个 Loader，让外挂运行在 Loader 里。用 vb6 把上面的代码写一遍，代码如下  
    
  Dim hFileMapping As Long, BaseAddress As Long, Buffer As Long  
  CreateMutex 0, 0, "EXE\_MUTEX\_LLK"  
  hFileMapping = CreateFileMapping(&HFFFFFFFF, 0, 4, 0, 4, "SFILEMAP\_LLK")  
  UnmapViewOfFile MapViewOfFile(hFileMapping, 2, 0, 0, 0)  
  BaseAddress = LoadLibrary("C:\\Program Files\\16xia\\QQLLK\\hook.dll")  
    
  把 Loader 命名为 Explorer.exe，外挂正常运行！  
    
  第一步先去除弹出网页，由于 DllMain 里有相关代码，我们在脱壳前又没办法碰它，只好对 ShellExecuteA 打补丁，Loader 代码如下  
    
  Dim hModule As Long, pShellExecuteA As Long, Buffer As Long  
  hModule = LoadLibrary("shell32.dll")  
  pShellExecuteA = GetProcAddress(hModule, "ShellExecuteA")  
  Buffer = &H18C2              'retn 18H  
  WriteProcessMemoryByLong &HFFFFFFFF, pShellExecuteA, Buffer, 3, ByVal 0&  
    
  然后去除 1 分钟后关闭的功能。下 API 断点，SetSystemTime、GetLocalTime、SetLocalTime、GetTickCount、SetTimer，均没有收获，晕了～～～～～～～～  
  只好硬着头皮上咯。幸好外挂退出时没忘记跟我们道别～～OD 载入并运行 Loader 后下断点 bp MessageBoxA，大约 1 分钟后断下，跟着堆栈信息一路出来，发现如下可疑代码  
  011D76D7   .  33F6          xor     esi, esi  
  011D76D9   >  8B45 F4       mov     eax, \[ebp-C\]  
  011D76DC   .  8B10          mov     edx, \[eax\]  
  011D76DE   .  FF52 14       call    \[edx+14\]  
  011D76E1   .  48            dec     eax  
  011D76E2   .  7C 31         jl      short 011D7715  
  011D76E4   .  8D4D E8       lea     ecx, \[ebp-18\]  
  011D76E7   .  33D2          xor     edx, edx  
  011D76E9   .  8B45 F4       mov     eax, \[ebp-C\]  
  011D76EC   .  8B18          mov     ebx, \[eax\]  
  011D76EE   .  FF53 0C       call    \[ebx+C\]  
  011D76F1   .  8B45 E8       mov     eax, \[ebp-18\]  
  011D76F4   .  BA 34781D01   mov     edx, 011D7834                    ;  ASCII "STOP"  
  011D76F9   .  E8 C6CCF6FF   call    011443C4  
  011D76FE   .  74 15         je      short 011D7715  
  011D7700   .  8D4D E4       lea     ecx, \[ebp-1C\]  
  011D7703   .  33D2          xor     edx, edx  
  011D7705   .  8B45 F4       mov     eax, \[ebp-C\]  
  011D7708   .  8B18          mov     ebx, \[eax\]  
  011D770A   .  FF53 0C       call    \[ebx+C\]  
  011D770D   .  8B45 E4       mov     eax, \[ebp-1C\]  
  011D7710   .  E8 E7020000   call    011D79FC  
  011D7715   >  46            inc     esi  
  011D7716   .  BB E9030000   mov     ebx, 3E9                         ;  (3E9)H=(1001)D，循环计数器  
  011D771B   >  8B45 FC       mov     eax, \[ebp-4\]  
  011D771E   .  8078 0D 00    cmp     byte ptr \[eax+D\], 0  
  011D7722   .  74 0D         je      short 011D7731  
  011D7724   .  33C0          xor     eax, eax  
  011D7726   .  5A            pop     edx  
  011D7727   .  59            pop     ecx  
  011D7728   .  59            pop     ecx  
  011D7729   .  64:8910       mov     fs:\[eax\], edx  
  011D772C   .  E9 D2000000   jmp     011D7803  
  011D7731   >  6A 0A         push    0A                               ; /Timeout = 10. ms  
  011D7733   .  E8 C05CF7FF   call    0114D3F8                         ; \\Sleep  
  011D7738   .  4B            dec     ebx  
  011D7739   .^ 75 E0         jnz     short 011D771B                   ;  10 秒一个循环  
  011D773B   .  83FE 06       cmp     esi, 6                           ;  是否运行了 6 个 10 秒?  
  011D773E    ^ 7C 99         jl      short 011D76D9                   ;  爆破点，改为 JMP  
  011D7740   .  8B45 F4       mov     eax, \[ebp-C\]  
  011D7743   .  8B10          mov     edx, \[eax\]  
  011D7745   .  FF52 14       call    \[edx+14\]  
  011D7748   .  8BD0          mov     edx, eax  
  011D774A   .  4A            dec     edx  
  011D774B   .  8D4D E0       lea     ecx, \[ebp-20\]  
  011D774E   .  8B45 F4       mov     eax, \[ebp-C\]  
  011D7751   .  8B18          mov     ebx, \[eax\]  
  011D7753   .  FF53 0C       call    \[ebx+C\]  
  011D7756   .  8B45 E0       mov     eax, \[ebp-20\]  
  011D7759   .  BA 34781D01   mov     edx, 011D7834                    ;  ASCII "STOP"  
  011D775E   .  E8 61CCF6FF   call    011443C4  
  011D7763   .  74 2E         je      short 011D7793  
  011D7765   .  8B45 F4       mov     eax, \[ebp-C\]  
  011D7768   .  8B10          mov     edx, \[eax\]  
  011D776A   .  FF52 14       call    \[edx+14\]  
  011D776D   .  8BD0          mov     edx, eax  
  011D776F   .  4A            dec     edx  
  011D7770   .  8D4D DC       lea     ecx, \[ebp-24\]  
  011D7773   .  8B45 F4       mov     eax, \[ebp-C\]  
  011D7776   .  8B18          mov     ebx, \[eax\]  
  011D7778   .  FF53 0C       call    \[ebx+C\]  
  011D777B   .  8B45 DC       mov     eax, \[ebp-24\]  
  011D777E   .  50            push    eax  
  011D777F   .  B9 ED030000   mov     ecx, 3ED  
  011D7784   .  BA 44781D01   mov     edx, 011D7844  
  011D7789   .  B8 58781D01   mov     eax, 011D7858                    ;  ASCII "#32770"  
  011D778E   .  E8 C9070000   call    011D7F5C  
  011D7793   >  8B45 F4       mov     eax, \[ebp-C\]  
  011D7796   .  E8 C1B9F6FF   call    0114315C  
  011D779B   .  C605 9CBA1D01>mov     byte ptr \[11DBA9C\], 0  
  011D77A2   .  68 30000100   push    10030                            ;  UNICODE "C:\\Documents and Settings\\All Users"  
  011D77A7   .  68 60781D01   push    011D7860  
  011D77AC   .  68 70781D01   push    011D7870  
  011D77B1   .  8B45 FC       mov     eax, \[ebp-4\]  
  011D77B4   .  8B40 44       mov     eax, \[eax+44\]  
  011D77B7   .  50            push    eax  
  011D77B8   .  E8 A7F3F6FF   call    01146B64                         ;  这里进去后 MessageBox 弹出  
  011D77BD   .  6A 01         push    1  
  011D77BF   .  6A 00         push    0  
  011D77C1   .  6A 00         push    0  
  011D77C3   .  68 10791D01   push    011D7910                         ;  ASCII "http://www.168reg.com/view\_dev.asp?id=602"  
  011D77C8   .  6A 00         push    0  
  011D77CA   .  8B45 FC       mov     eax, \[ebp-4\]  
  011D77CD   .  8B40 44       mov     eax, \[eax+44\]  
  011D77D0   .  50            push    eax  
  011D77D1   .  E8 6615F9FF   call    01168D3C  
  011D77D6   .  6A 00         push    0                                ; /lParam = 0  
  011D77D8   .  6A 00         push    0                                ; |wParam = 0  
  011D77DA   .  6A 10         push    10                               ; |Message = WM\_CLOSE  
  011D77DC   .  8B45 FC       mov     eax, \[ebp-4\]                     ; |  
  011D77DF   .  8B40 44       mov     eax, \[eax+44\]                    ; |  
  011D77E2   .  50            push    eax                              ; |hWnd  
  011D77E3   .  E8 B4F3F6FF   call    01146B9C                         ; \\PostMessageA  
  011D77E8   .  33C0          xor     eax, eax  
  011D77EA   .  5A            pop     edx  
  011D77EB   .  59            pop     ecx  
  011D77EC   .  59            pop     ecx  
  011D77ED   .  64:8910       mov     fs:\[eax\], edx  
  011D77F0   .  EB 11         jmp     short 011D7803  
  011D77F2   .^ E9 05BEF6FF   jmp     011435FC  
  011D77F7   .  E8 68C1F6FF   call    01143964  
  011D77FC   .  EB 05         jmp     short 011D7803  
  011D77FE   .  E8 61C1F6FF   call    01143964  
  011D7803   >  33C0          xor     eax, eax  
  011D7805   .  5A            pop     edx  
  011D7806   .  59            pop     ecx  
  011D7807   .  59            pop     ecx  
  011D7808   .  64:8910       mov     fs:\[eax\], edx  
  011D780B   .  68 25781D01   push    011D7825  
  011D7810   >  8D45 DC       lea     eax, \[ebp-24\]  
  011D7813   .  BA 06000000   mov     edx, 6  
  011D7818   .  E8 BFC7F6FF   call    01143FDC  
  011D781D   .  C3            retn  
    
  查看内存中 Hook.dll 的基址为 0x01140000，爆破点偏移量为 0x011D773E-0x01140000=0x9773E，至此所有信息收集完毕～～～  
    
  下面给出完整 Loader 代码：  
  'API 声明  
  Private Declare Function LoadLibrary Lib "kernel32" Alias "LoadLibraryA" (ByVal lpLibFileName As String) As Long  
  Private Declare Function CreateMutex Lib "kernel32" Alias "CreateMutexA" (ByVal lpMutexAttributes As Long, ByVal bInitialOwner As Long, ByVal lpName As String) As Long  
  Private Declare Function CreateFileMapping Lib "kernel32" Alias "CreateFileMappingA" (ByVal hFile As Long, ByVal lpFileMappigAttributes As Long, ByVal flProtect As Long, ByVal dwMaximumSizeHigh As Long, ByVal dwMaximumSizeLow As Long, ByVal lpName As String) As Long  
  Private Declare Function MapViewOfFile Lib "kernel32" (ByVal hFileMappingObject As Long, ByVal dwDesiredAccess As Long, ByVal dwFileOffsetHigh As Long, ByVal dwFileOffsetLow As Long, ByVal dwNumberOfBytesToMap As Long) As Long  
  Private Declare Function UnmapViewOfFile Lib "kernel32" (ByVal lpBaseAddress As Long) As Long  
  Private Declare Function WriteProcessMemoryByLong Lib "kernel32" Alias "WriteProcessMemory" (ByVal hProcess As Long, ByVal lpBaseAddress As Long, ByRef lpBuffer As Long, ByVal nSize As Long, lpNumberOfBytesWritten As Long) As Long  
  Private Declare Function GetProcAddress Lib "kernel32" (ByVal hModule As Long, ByVal lpProcName As String) As Long  
  Private Declare Function FindWindow Lib "user32" Alias "FindWindowA" (ByVal lpClassName As String, ByVal lpWindowName As String) As Long  
  Private Declare Sub Sleep Lib "kernel32" (ByVal dwMilliseconds As Long)  
    
  Private Sub Form\_Load()  
  Dim hFileMapping As Long, BaseAddress As Long, Buffer As Long, hModule As Long, pShellExecuteA As Long  
    
  '对 ShellExecute 打补丁，去处弹出网页  
  hModule = LoadLibrary("shell32.dll")  
  pShellExecuteA = GetProcAddress(hModule, "ShellExecuteA")  
  Buffer = &H18C2              'retn 18H  
  WriteProcessMemoryByLong &HFFFFFFFF, pShellExecuteA, Buffer, 3, ByVal 0&  
    
  '启动外挂  
  CreateMutex 0, 0, "EXE\_MUTEX\_LLK"  
  hFileMapping = CreateFileMapping(&HFFFFFFFF, 0, 4, 0, 4, "SFILEMAP\_LLK")  
  UnmapViewOfFile MapViewOfFile(hFileMapping, 2, 0, 0, 0)  
  BaseAddress = LoadLibrary("C:\\Program Files\\16xia\\QQLLK\\hook.dll")  
    
  '去除定时关闭  
  Buffer = &HEB    'JMP  
  WriteProcessMemoryByLong &HFFFFFFFF, BaseAddress + &H9773E, Buffer, 1, ByVal 0&  
    
  '检测外挂是否已退出  
  Sleep 3000  
  Timer1.Interval = 100  
  Timer1.Enabled = True  
  Form1.Hide  
  End Sub  
    
  Private Sub Timer1\_Timer()  
  If FindWindow("TFormMain", "试用版 V4.93") = 0 Then End  
  End Sub  
 --------------------------------------------------------------------------------  
  
【后记】:写这篇小东东并非因为我喜欢玩连连看，而是觉得这外挂的保护方式比较特别，有可以借鉴的地方～～而在定时关闭时，没有使用常规的计时函数，而是另开时线程，使用 Sleep 函数计时～～这些我都没见过，呵呵～～～   
\--------------------------------------------------------------------------------  
  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                                                                     2006 年 07 月 22 日 

 4dd466d0e1e0cc123afbe504da2462ca