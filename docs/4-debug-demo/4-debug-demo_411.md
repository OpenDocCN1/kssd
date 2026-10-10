

# Streaming Audio Player (SAP) v0.9

> 原文：[`www.pediy.com/kssd/pediy06/pediy6771.htm`](https://www.pediy.com/kssd/pediy06/pediy6771.htm)

Streaming Audio Player (SAP) v0.9 是一个较老的 MP3 播放器，2000 年 7 月份发布的。但其中的 DJ 功能我确非常喜欢，并且占用资源小，界面也非常漂亮，而且支持动画。不过在使用的时候发现几个不方便的地方，就修改了一下：  
1. 当设置成 DJ 模式的时候，如果停止播放，在按 Play，系统不会  
   继续播放。  
     
   解决方法：  
       用 Ollydbg 打开 SAP.EXE，查找 ShuffleMaster，经过分析后发现  
   ShuffleMaster 为\[436488\]，当该值为 1 时就是 ShuffleMaster 模式。  
   设置在该地址读时中断，并且在 DJ 模式下按 Play，代码如下：  
  
   0040F945   |>  8B0D 88644300   MOV     ECX, \[436488\]  
   0040F94B   |.  85C9            TEST    ECX, ECX  
   0040F94D   |.  7E 0B           JLE     SHORT sap.0040F95A   如果不是 DJ 模式跳转  
   0040F94F   |.  51              PUSH    ECX  
   0040F950   |.  E8 6BAB0000     CALL    sap.0041A4C0  
   0040F955   |.  83C4 04         ADD     ESP, 4  
   0040F958   |.  5E              POP     ESI                              USER32.77D43FBE  
   0040F959   |.  C3              RETN  
   0040F95A   |>  8B0D 8C644300   MOV     ECX, \[43648C\]  
   0040F960   |.  83F9 01         CMP     ECX, 1  
   0040F963   |.  75 0A           JNZ     SHORT sap.0040F96F  
   0040F965   |.  8A88 68030000   MOV     CL, \[EAX+368\]  
   0040F96B   |.  84C9            TEST    CL, CL  
   0040F96D   |.  74 15           JE      SHORT sap.0040F984  
   0040F96F   |>  8B0D 8C644300   MOV     ECX, \[43648C\]  
  
   在地址 0040F94D 处判断如果不是 DJ 模式则跳转，是 DJ 模式就调用  
   41A4C0，然后返回。查看 41A4C0 的代码发现是创建一个新的线程，  
   显然这个线程是用于 DJ 的，而 40F95A 以下的代码则是来开始播放  
   的，因而很简单，将 40F958 和 40F959 的代码 NOP 掉即可。  
  
2. 系统在最小化到 SysTray 的时候显示 ICON 不对  
   解决方法：  
   用 Ollydbg 打开 SAP.EXE，查找函数 Shell\_NotifyIconA 的调用，到如下  
   代码：  
   0040A500   /$  8B4424 04       MOV     EAX, \[ESP+4\]                     sap.004107E4  
   0040A504   |.  83EC 58         SUB     ESP, 58  
   0040A507   |.  8B4C24 64       MOV     ECX, \[ESP+64\]                    ntdll.77F8CF62  
   0040A50B   |.  53              PUSH    EBX                              USER32.GetMessageA  
   0040A50C   |.  894424 08       MOV     \[ESP+8\], EAX  
   0040A510   |.  56              PUSH    ESI                              USER32.WaitMessage  
   0040A511   |.  894C24 10       MOV     \[ESP+10\], ECX  
   0040A515   |.  57              PUSH    EDI                              USER32.PeekMessageA  
   0040A516   |.  8B7424 74       MOV     ESI, \[ESP+74\]  
   0040A51A   |.  8B4424 78       MOV     EAX, \[ESP+78\]  
   0040A51E   |.  897424 20       MOV     \[ESP+20\], ESI                    USER32.WaitMessage  
   0040A522   |.  85C0            TEST    EAX, EAX  
   0040A524   |.  C74424 0C 58000>MOV     DWORD PTR \[ESP+C\], 58  
   0040A52C   |.  C74424 18 07000>MOV     DWORD PTR \[ESP+18\], 7  
   0040A534   |.  C74424 1C 64800>MOV     DWORD PTR \[ESP+1C\], 8064  
   0040A53C   |.  74 10           JE      SHORT sap.0040A54E  
   0040A53E   |.  6A 40           PUSH    40                              /n = 40 (64.)  
   0040A540   |.  50              PUSH    EAX                             |String2 = "H"  
   0040A541   |.  8D4424 2C       LEA     EAX, \[ESP+2C\]                   |  
   0040A545   |.  50              PUSH    EAX                             |String1 = 003C53E0  
   0040A546   |.  FF15 18254500   CALL    \[<&KERNEL32.lstrcpynA>\]         \\lstrcpynA  
   0040A54C   |.  EB 05           JMP     SHORT sap.0040A553  
   0040A54E   |>  C64424 24 00    MOV     BYTE PTR \[ESP+24\], 0  
   0040A553   |>  8D4424 0C       LEA     EAX, \[ESP+C\]  
   0040A557   |.  8B4C24 6C       MOV     ECX, \[ESP+6C\]  
   0040A55B   |.  50              PUSH    EAX  
   0040A55C   |.  51              PUSH    ECX  
   0040A55D   |.  FF15 1C264500   CALL    \[<&SHELL32.Shell\_NotifyIconA>\]   SHELL32.Shell\_NotifyIconA  
   0040A563   |.  8BF8            MOV     EDI, EAX  
   0040A565   |.  85F6            TEST    ESI, ESI                         USER32.WaitMessage  
   0040A567   |.  74 07           JE      SHORT sap.0040A570  
   0040A569   |.  56              PUSH    ESI                             /hIcon = 77D43FB2  
   0040A56A   |.  FF15 A0264500   CALL    \[<&USER32.DestroyIcon>\]         \\DestroyIcon  
   0040A570   |>  8BC7            MOV     EAX, EDI                         USER32.PeekMessageA  
   0040A572   |.  5F              POP     EDI                              USER32.77D43FBE  
   0040A573   |.  5E              POP     ESI                              USER32.77D43FBE  
   0040A574   |.  5B              POP     EBX                              USER32.77D43FBE  
   0040A575   |.  83C4 58         ADD     ESP, 58  
   0040A578   \\.  C3              RETN  
  
   经查，该函数在下面的代码中被调用：  
   0040A580   /$  53              PUSH    EBX  
   0040A581   |.  A1 B0604300     MOV     EAX, \[4360B0\]  
   0040A586   |.  56              PUSH    ESI  
   0040A587   |.  57              PUSH    EDI  
   0040A588   |.  8B78 0C         MOV     EDI, \[EAX+C\]  
   0040A58B   |.  6A FA           PUSH    -6                              /Index = GWL\_HINSTANCE  
   0040A58D   |.  57              PUSH    EDI                             |hWnd = NULL  
   0040A58E   |.  FF15 64264500   CALL    \[<&USER32.GetWindowLongA>\]      \\GetWindowLongA  
   0040A594   |.  8BF0            MOV     ESI, EAX  
   0040A596   |.  6A 00           PUSH    0  
   0040A598   |.  6A 00           PUSH    0  
   0040A59A   |.  68 57040000     PUSH    457  
   0040A59F   |.  6A 00           PUSH    0  
   0040A5A1   |.  57              PUSH    EDI  
   0040A5A2   |.  E8 59FFFFFF     CALL    SAP.0040A500  
   0040A5A7   |.  83C4 14         ADD     ESP, 14  
   0040A5AA   |.  68 4C504300     PUSH    SAP.0043504C                     ASCII "SAP"  
   0040A5AF   |.  6A 65           PUSH    65                              /RsrcName = 101.  
   0040A5B1   |.  56              PUSH    ESI                             |hInst = 00000024  
   0040A5B2   |.  FF15 88264500   CALL    \[<&USER32.LoadIconA>\]           \\LoadIconA  
   0040A5B8   |.  50              PUSH    EAX  
   0040A5B9   |.  68 57040000     PUSH    457  
   0040A5BE   |.  6A 01           PUSH    1  
   0040A5C0   |.  57              PUSH    EDI  
   0040A5C1   |.  E8 3AFFFFFF     CALL    SAP.0040A500  
   0040A5C6   |.  83C4 14         ADD     ESP, 14  
   0040A5C9   |.  5F              POP     EDI                              kernel32.77E7EB69  
   0040A5CA   |.  5E              POP     ESI                              kernel32.77E7EB69  
   0040A5CB   |.  5B              POP     EBX                              kernel32.77E7EB69  
   0040A5CC   \\.  C3              RETN  
  
   经过跟踪后发现调用 GetWindowLongA 返回的 hInstance 为 0，而不是  
   正在的句柄，所以将该代码改成如下：  
   0040A58B   |.  6A 00           PUSH    0                               /pModule = NULL  
   0040A58D   |.  90              NOP                                     |  
   0040A58E   |.  FF15 C8254500   CALL    \[<&KERNEL32.GetModuleHandleA>\]  \\GetModuleHandleA  
  
   通过 GetModuleHandle 来获取句柄就可以了。  
     
3. 还有，在设置成 Auto Hide 模式时，当鼠标移到各个角落的时候，会自动显示，  
   这些功能很烦，所以需要把它去掉。查找 Auto Hide 的代码发现是通过 SetTimer  
   实现的，代码如下：  
   0041C86F   |.  6A 00           PUSH    0  
   0041C871   |.  6A FF           PUSH    -1  
   0041C873   |.  51              PUSH    ECX  
   0041C874   |.  FFD6            CALL    ESI                              USER32.SetWindowPos  
   0041C876   |.  6A 00           PUSH    0                               /Timerproc = NULL  
   0041C878   |.  8B0D B0604300   MOV     ECX, \[4360B0\]                   |  
   0041C87E   |.  68 2C010000     PUSH    12C                             |Timeout = 300. ms  
   0041C883   |.  6A 00           PUSH    0                               |TimerID = 0  
   0041C885   |.  8B51 0C         MOV     EDX, \[ECX+C\]                    |  
   0041C888   |.  52              PUSH    EDX                             |hWnd = 7FFE0304  
   0041C889   |.  FF15 B0264500   CALL    \[<&USER32.SetTimer>\]            \\SetTimer  
   0041C88F   |>  A1 BC604300     MOV     EAX, \[4360BC\]  
  
   为简单起见，直接将代码 41C876 跳转到 41C88F 如下：  
   0041C876      /EB 17           JMP     SHORT sap.0041C88F  
   0041C878   |. |8B0D B0604300   MOV     ECX, \[4360B0\]                   |  
   0041C87E   |. |68 2C010000     PUSH    12C                             |Timeout = 300. ms  
   0041C883   |. |6A 00           PUSH    0                               |TimerID = 0  
   0041C885   |. |8B51 0C         MOV     EDX, \[ECX+C\]                    |  
   0041C888   |. |52              PUSH    EDX                             |hWnd = 7FFE0304  
   0041C889   |. |FF15 B0264500   CALL    \[<&USER32.SetTimer>\]            \\SetTimer  
   0041C88F   |> \\A1 BC604300     MOV     EAX, \[4360BC\]  
  
\========================================================================  

 4dd466d0e1e0cc123afbe504da2462ca