

# Armadillo 3.6

> 原文：[`www.pediy.com/kssd/pediy06/pediy6444.htm`](https://www.pediy.com/kssd/pediy06/pediy6444.htm)

Armadillo 3.6 主程序脱壳   
  
工具：OllyDBG1.1 汉化版；LordPE   
操作系统：WIN2K   
目标程序：Armadillo.exe(3.6 主程序)   
　　最近大家对 Armadillo 3.6 脱的研究有所升温，有标准加壳的还有 CopyMemII+Debug 模式的加壳的等等。于是本人也凑个热闹：来个其主程序的脱文。较之他们的不同点是：一是主程序脱壳难度大点；二是不用其它辅助 dump 工具。（初步脱壳，没有修复）   
一、寻找 OEP 并 Dump 进程   
　　用 OllyDBG 加载后，停留在入口 004A2000 处，记下入口开始二字节(60E8)以便修复 IAT 用。用插件 IsDebuggerPresent 隐藏 ollydbg 并在调试设置里忽略掉所有异常(全钩上)。   
在命令窗口里下断点：bp WaitForDebugEvent，然后 F9 运行。断在 WaitForDebugEvent 入口(按 F2 取消断点)。查看堆栈窗口：   
0012DA98   00487F67  /CALL 到 WaitForDebugEvent 来自 Armadill.00487F61   
0012DA9C   0012EB5C  |pDebugEvent = 0012EB5C   
0012DAA0   000003E8  Timeout = 1000. ms   
0012DAA4   0012FF04   
0012DAA8   00000000   
0012DAAC   00497B99  Armadill.00497B99   
其中第 2 行 0012EB5C 就是发生调试事件时具体内容存放地址。到次行点击鼠标右键然后在菜单选择“转存中跟随”，以便在转存窗口随时观察 0012EB5C 开始一段地址中的数据变化。   
到 CPU 窗口，Ctrl-G:0048858A,然后在 0048858A 处设置硬件执行断点。此断点是偶经过无数次跟踪分析而找到的最佳切入点(本人独创，绝无 2 家)，主要是为了便于 Dump 进程，至于为什么大家看在此处断下后的代码分析片断。   
ok,在所设硬件断点处断下。察看 0012EB5C 开始一段地址中的数据：   
0012EB5C  01 00 00 00 58 06 00 00  ...X..   
0012EB64  30 04 00 00 01 00 00 80  0....€   
0012EB6C  00 00 00 00 00 00 00 00  ........   
0012EB74  B0 28 44 00 02 00 00 00  ?D....   
\[0012EB74\]=004428B0,这就是我们要找的 OEP！    
接着来看断点处的代码 (每行尾部//为说明)   
0048858A  CMP DWORD PTR SS:\[EBP\-A30\],0　//\[EBP-A30\]为 01000H 大小的块号(0-47),0 对应 00401000 块；1 对应 00402000 块；依次类推。    
00488591  JL Armadill.00488840   
00488597  MOV ECX,DWORD PTR SS:\[EBP\-A30\]   
0048859D  CMP ECX,DWORD PTR DS:\[4B85E4\]  //\[4B85E4\]为块的总个数，本程序=48h    
004885A3  JGE Armadill.00488840   
004885A9  MOV EDX,DWORD PTR SS:\[EBP\-9BC\]   
004885AF  AND EDX,0FF   
004885B5  TEST EDX,EDX   
004885B7  JE Armadill.0048866A   
004885BD  PUSH 0   
004885BF  MOV ESI,DWORD PTR SS:\[EBP\-A30\]   
004885C5  SHL ESI,4   
004885C8  MOV EAX,DWORD PTR SS:\[EBP\-A30\]   
004885CE  AND EAX,80000007   
004885D3  JNS SHORT Armadill.004885DA   
004885D5  DEC EAX   
004885D6  OR EAX,FFFFFFF8   
004885D9  INC EAX   
004885DA  XOR ECX,ECX   
004885DC  MOV CL,BYTE PTR DS:\[EAX+4B6A80\]   
004885E2  MOV EDX,DWORD PTR SS:\[EBP\-A30\]   
004885E8  AND EDX,80000007   
004885EE  JNS SHORT Armadill.004885F5   
004885F0  DEC EDX   
004885F1  OR EDX,FFFFFFF8   
004885F4  INC EDX   
004885F5  XOR EAX,EAX   
004885F7  MOV AL,BYTE PTR DS:\[EDX+4B6A81\]   
004885FD  MOV EDI,DWORD PTR DS:\[ECX\*4+4B2260\]   
00488604  XOR EDI,DWORD PTR DS:\[EAX\*4+4B2260\]   
0048860B  MOV ECX,DWORD PTR SS:\[EBP\-A30\]   
00488611  AND ECX,80000007   
00488617  JNS SHORT Armadill.0048861E   
00488619  DEC ECX   
0048861A  OR ECX,FFFFFFF8   
0048861D  INC ECX   
0048861E  XOR EDX,EDX   
00488620  MOV DL,BYTE PTR DS:\[ECX+4B6A82\]   
00488626  XOR EDI,DWORD PTR DS:\[EDX\*4+4B2260\]   
0048862D  MOV EAX,DWORD PTR SS:\[EBP\-A30\]   
00488633  CDQ   
00488634  MOV ECX,1C   
00488639  IDIV ECX   
0048863B  MOV ECX,EDX   
0048863D  SHR EDI,CL   
0048863F  AND EDI,0F   
00488642  ADD ESI,EDI   
00488644  MOV EDX,DWORD PTR DS:\[4B85D4\]   
0048864A  LEA EAX,DWORD PTR DS:\[EDX+ESI\*4\]   
0048864D  PUSH EAX   
0048864E  MOV ECX,DWORD PTR SS:\[EBP\-A30\]   
00488654  PUSH ECX   
00488655  CALL Armadill.0048A659   //解密数据并复制到对应的块。    
0048865A  ADD ESP,0C   
0048865D  AND EAX,0FF   
00488662  TEST EAX,EAX   
00488664  JE Armadill.00488840   
察看\[EBP\-A30\]的值如下：   
0012EB48  41 00 00 00 01 49 13 00  A...I.   
修改 0012eb48 处的值为 0,从第一块开始解密并复制数据到 00401000 处，大小 01000H.为了循环直到所有数据全部解密，偶修改了 0048865D 处的代码如下:   
0048865D:inc dword ptr \[0012eb48\]   
00488663:nop   
00488664:jmp 0048858A   
到数据转存窗口，Ctrl-G:004B85E4,用来观察 004B85E4 和 004B85E8 二处数据。其中 004B85E4 单元为总块数；004B85E8 单元为已经解密的块数，如果解密块数>=23H,则会对第 0 块即 00401000 处代码实现加密并改变该块读写权限不能进行任何操作(GUARD),所以当 004B85E8   
的值到了 20h 时就修改为 0(也可以在 21/22H 时改，但不能超过 23H).   
F9 继续运行，(中间记得修改 004B85E8 的值)在该硬件断点中断 48h 次后，运行 LordPE,选择第 2 个进程(有 2 个同名进程)，即可完全 dump.   
  
二、得到引入表   
    重头再来，用 OllyDBG 加载，设置 bp DebugActiveProcess。断下后察看堆栈窗口：   
0012DA9C   00487DDB  /CALL 到 DebugActiveProcess 来自 Armadill.00487DD5   
0012DAA0   0000057C  ProcessId = 57C   
0012DAA4   0012FF04   
0012DAA8   00000000   
子进程 ID 为 57C,另外打开一个 ollydbg 程序，附加到这个 57c 进程，OK。    
按 ALT+F9 来到入口处，修改入口指令 EB FE 为 60 E8(第一步记下的),结果：   
004A2000 >PUSHAD   
004A2001  CALL Armadill.004A2006   
004A2006  POP EBP   
004A2007  PUSH EAX   
004A2008  PUSH ECX   
004A2009  JMP SHORT Armadill.004A201A   
取消 DebugActiveProcess 断点，设置 bp OpenMutexA 断点，断下后取消。   
察看堆栈窗口：   
0012F574   004797F1  /CALL 到 OpenMutexA 来自 Armadill.004797EB   
0012F578   001F0001  |Access = 1F0001   
0012F57C   00000000  |Inheritable = FALSE   
0012F580   0012FBB4  MutexName = "57C::DAAD341ECC"   
0012F584   0012FF04   
在 00401000 空白处输入代码：   
00401000    60             PUSHAD   
00401001    68 B4FB1200    PUSH 12FBB4         
00401006    6A 00          PUSH 0   
00401008    6A 00          PUSH 0   
0040100A    E8 D08BA877    CALL KERNEL32.CreateMutexA   
0040100F    61             POPAD   
00401010   -E9 D48CA877    JMP KERNEL32.OpenMutexA   
在 00401000 地址按鼠标右键，然后在菜单中选择“此处新建 EIP”。   
设置 bp LoadLibraryA 断点,第 2 次断下后，再 F8 跟，就会发现 00E18000 开始就是原来未经任何修改的引入表。用 LordPe 把起始地址 00e18000,大小 03000h 全部 dump 出来，并编辑修改到第一步 dump 出来的文件的同一地址。 

Armadillo 3.6 主程序 IAT 处理   
  
    上次对 IAT 处理有误，近几天有空专门对其进行了分析，而且为此还安装了 WINXP。发现 Armadillo 3.6 和 3.7 处理 IAT 方法一样，但是操作系统不同 IAT 位置不一样，以 WIN2k 为最难。win2k 下 IAT 位置到 012c1000 处不在文件内存映像范围内，所以对其 IAT 修复还要处理位置问题。下面是分析笔记，不当之处请指点。   
用 OllyDBG 加载，在命令窗口输入 bp DebugActiveProcess。断下后察看堆栈窗口：   
0012DA9C   00487DDB  /CALL 到 DebugActiveProcess 来自 Armadill.00487DD5   
0012DAA0   0000057C  ProcessId = 57C   
0012DAA4   0012FF04   
0012DAA8   00000000   
进程 ID 为 57C.另外打开一个 ollydbg 程序，附加到 57C 这个进程，OK。    
按 ALT+F9 会来到入口处，修改入口指令 EB FE 为 60 E8(第一步记下的),结果：   
004A2000 >PUSHAD   
004A2001  CALL Armadill.004A2006   
004A2006  POP EBP   
004A2007  PUSH EAX   
004A2008  PUSH ECX   
004A2009  JMP SHORT Armadill.004A201A   
设置 bp OpenMutexA 断点，断下后取消。   
察看堆栈窗口：   
0012F574   004797F1  /CALL 到 OpenMutexA 来自 Armadill.004797EB   
0012F578   001F0001  |Access = 1F0001   
0012F57C   00000000  |Inheritable = FALSE   
0012F580   0012FBB4  MutexName = "57C::DAAD341ECC"   
0012F584   0012FF04   
其中 0012FBB4 指向 MutexName = "57C::DAAD341ECC"，根据此值在 00401000 空白处输入以下代码：(先建立再打开，对 OpenMutexA 这个 API 进行 Hook)   
00401000    60             PUSHAD   
00401001    68 B4FB1200    PUSH 12FBB4     ; ASCII "57C::DAAD341ECC"   
00401006    6A 00          PUSH 0   
00401008    6A 00          PUSH 0   
0040100A    E8 D08BA877    CALL KERNEL32.CreateMutexA   
0040100F    61             POPAD   
00401010   -E9 D48CA877    JMP KERNEL32.OpenMutexA   
在 00401000 地址按鼠标右键，然后在菜单中选择“此处新建 EIP”。   
设置 bp LoadLibraryA 断点,不断按 F9，当堆栈显示为：   
0012BEE4   00DF8B55  /CALL 到 LoadLibraryA 来自 00DF8B4F   
0012BEE8   012E0640  FileName = "WSOCK32.dll"   
此时说明开始处理 IAT 了。按 Ctrl-F9，然后 F7 到程序领空。   
然后一路 F8 到：   
00E11684  PUSH 0   
00E11686  CALL DWORD PTR DS:\[E180C4\]     ; KERNEL32.GetModuleHandleA   
00E1168C  CMP DWORD PTR SS:\[EBP\-1744\],EAX   
00E11692  JNZ SHORT 00E116A3   
再一路 F8 到：   
00E1174F  CMP DWORD PTR SS:\[EBP\-1744\],EAX       
00E11755  JNZ SHORT 00E11768   
此处要说明一下,目前大部分都是在这之后修改 jmp 使之不进行 IAT 加密。其实所有 Armadillo 壳处理 IAT 的原理是：先比较模块是不是 kernel32.dll/user32.dll/advapi32.dll,如果是再进行 API 比较(只对符合条件的 API 进行 hook 到壳中执行),如 API 符合就修改 IAT 中对应的项使其指向壳代码,都不满足则填入真实的 API 入口地址(即不加密)。   
00E1174F 处就是进行模块句柄比较,所以只要修改 00E11755 处的跳转或在此处下硬件执行断点手动转移，就可以免除所有 IAT 加密。   
ok,当所有 IAT 处理完毕后，就可以用 ImportREC.exe 得到所有正确 IAT 了。win2k 下 RVA=00EC1008,SIZA=600,然后即可 get imports.   
这是得到的所有函数清单：   
1  00EC1008  user32.dll  0261  SetWindowPos   
1  00EC100C  user32.dll  019D  LoadBitmapA   
1  00EC101C  user32.dll  01BF  MapWindowPoints   
1  00EC1020  gdi32.dll  0052  DeleteDC   
1  00EC1024  kernel32.dll  01FC  MapViewOfFile   
1  00EC102C  user32.dll  01C3  MessageBeep   
1  00EC1038  ws2\_32.dll  0034  gethostbyname   
1  00EC1040  gdi32.dll  01B4  RealizePalette   
1  00EC1048  ws2\_32.dll  0003  closesocket   
1  00EC1050  kernel32.dll  0026  CompareStringW   
1  00EC1058  user32.dll  019B  LoadAcceleratorsA   
1  00EC105C  kernel32.dll  0025  CompareStringA   
1  00EC1060  user32.dll  0100  GetCursorPos   
1  00EC1064  user32.dll  013E  GetPropA   
1  00EC106C  user32.dll  025E  SetWindowLongA   
1  00EC1074  user32.dll  0146  GetSubMenu   
1  00EC1078  kernel32.dll  0202  MoveFileA   
1  00EC107C  user32.dll  0087  DefWindowProcA   
1  00EC1080  kernel32.dll  018B  GetTickCount   
1  00EC1084  kernel32.dll  0309  WinExec   
1  00EC1088  gdi32.dll  0013  BitBlt   
1  00EC1090  ntdll.dll  0222  RtlLeaveCriticalSection   
1  00EC1094  user32.dll  0161  GetWindowRect   
1  00EC109C  comctl32.dll  003F  ImageList\_LoadImage   
1  00EC10A0  user32.dll  00D7  FillRect   
1  00EC10A4  user32.dll  015B  GetWindowLongA   
1  00EC10B4  user32.dll  00F6  GetClipboardData   
1  00EC10B8  kernel32.dll  0293  SetEndOfFile   
1  00EC10BC  user32.dll  0258  SetTimer   
1  00EC10C4  kernel32.dll  012A  GetFileSize   
1  00EC10CC  ntdll.dll  0199  RtlDeleteCriticalSection   
1  00EC10D0  gdi32.dll  0039  CreateFontIndirectA   
1  00EC10D4  advapi32.dll  0193  RegDeleteKeyA   
1  00EC10D8  user32.dll  0017  CallWindowProcA   
1  00EC10DC  gdi32.dll  0168  GetStockObject   
1  00EC10E0  kernel32.dll  01E6  LoadLibraryA   
1  00EC10E8  user32.dll  00B2  DrawTextA   
1  00EC10EC  kernel32.dll  0044  CreateMutexA   
1  00EC10F0  comdlg32.dll  006E  GetOpenFileNameA   
1  00EC10F8  ntdll.dll  0277  RtlSizeHeap   
1  00EC1104  kernel32.dll  0049  CreateProcessA   
1  00EC1108  gdi32.dll  0032  CreateDIBitmap   
1  00EC110C  user32.dll  018B  IsClipboardFormatAvailable   
1  00EC1110  kernel32.dll  02E5  UnmapViewOfFile   
1  00EC1114  kernel32.dll  029C  SetFilePointer   
1  00EC1118  user32.dll  0270  ShowWindow   
1  00EC1120  gdi32.dll  01CF  SelectObject   
1  00EC1124  gdi32.dll  0044  CreatePalette   
1  00EC1128  user32.dll  008A  DeleteMenu   
1  00EC112C  gdi32.dll  002C  CreateCompatibleDC   
1  00EC1130  user32.dll  0219  SendMessageA   
1  00EC113C  user32.dll  0288  TranslateMessage   
1  00EC114C  kernel32.dll  010C  GetCurrentDirectoryA   
1  00EC1154  user32.dll  02B1  WindowFromPoint   
1  00EC115C  user32.dll  0116  GetKeyState   
1  00EC1160  advapi32.dll  0198  RegEnumKeyA   
1  00EC1168  user32.dll  0008  AppendMenuA   
1  00EC1170  kernel32.dll  031A  WritePrivateProfileStringA   
1  00EC1174  kernel32.dll  00BE  FlushFileBuffers   
1  00EC1184  kernel32.dll  02AF  SetStdHandle   
1  00EC118C  user32.dll  0248  SetPropA   
1  00EC1198  kernel32.dll  011E  GetEnvironmentStrings   
1  00EC119C  user32.dll  0051  CreateDialogParamA   
1  00EC11A0  kernel32.dll  010E  GetCurrentProcess   
1  00EC11B0  kernel32.dll  0120  GetEnvironmentStringsW   
1  00EC11B8  ws2\_32.dll  0073  WSAStartup   
1  00EC11C0  user32.dll  0098  DispatchMessageA   
1  00EC11C4  user32.dll  0163  GetWindowTextA   
1  00EC11C8  user32.dll  0194  IsWindow   
1  00EC11D0  kernel32.dll  0194  GetVersionExA   
1  00EC11D4  gdi32.dll  01D6  SetBkMode   
1  00EC11D8  user32.dll  0284  TranslateAccelerator   
1  00EC11DC  kernel32.dll  0315  WriteFile   
1  00EC11E0  kernel32.dll  0151  GetPrivateProfileSectionNamesA   
1  00EC11E4  kernel32.dll  02C9  SizeofResource   
1  00EC11EC  user32.dll  01A3  LoadIconA   
1  00EC11F8  ws2\_32.dll  0016  shutdown   
1  00EC11FC  kernel32.dll  01AC  GlobalLock   
1  00EC1204  advapi32.dll  01BA  RegSetValueExA   
1  00EC1208  kernel32.dll  0039  CreateFileA   
1  00EC120C  kernel32.dll  016B  GetStartupInfoA   
1  00EC1210  user32.dll  0277  SystemParametersInfoA   
1  00EC1218  gdi32.dll  018E  LineTo   
1  00EC121C  ws2\_32.dll  006F  WSAGetLastError   
1  00EC1220  gdi32.dll  002D  CreateDCA   
1  00EC1224  user32.dll  0157  GetWindow   
1  00EC1228  comctl32.dll  0046  ImageList\_ReplaceIcon   
1  00EC1230  kernel32.dll  0193  GetVersion   
1  00EC1234  kernel32.dll  01BB  HeapCreate   
1  00EC123C  user32.dll  0106  GetDlgItem   
1  00EC1240  user32.dll  003C  ClientToScreen   
1  00EC1244  ws2\_32.dll  0013  send   
1  00EC1258  kernel32.dll  0262  SearchPathA   
1  00EC125C  user32.dll  010B  GetFocus   
1  00EC1264  user32.dll  0195  IsWindowEnabled   
1  00EC126C  comdlg32.dll  0070  GetSaveFileNameA   
1  00EC1270  kernel32.dll  01EB  LoadResource   
1  00EC1274  user32.dll  01F6  RedrawWindow   
1  00EC1278  kernel32.dll  0134  GetLocalTime   
1  00EC127C  user32.dll  00BE  EndPaint   
1  00EC1280  ntdll.dll  01B8  RtlEnterCriticalSection   
1  00EC1288  kernel32.dll  0169  GetShortPathNameA   
1  00EC1290  user32.dll  01C4  MessageBoxA   
1  00EC1298  user32.dll  020F  ScreenToClient   
1  00EC129C  user32.dll  0091  DestroyWindow   
1  00EC12A8  user32.dll  01E6  PostQuitMessage   
1  00EC12B8  user32.dll  00F4  GetClientRect   
1  00EC12C0  ws2\_32.dll  0074  WSACleanup   
1  00EC12C4  kernel32.dll  00D4  GetCPInfo   
1  00EC12C8  kernel32.dll  0121  GetEnvironmentVariableA   
1  00EC12CC  user32.dll  00E7  GetAsyncKeyState   
1  00EC12D0  kernel32.dll  0052  CreateThread   
1  00EC12D8  kernel32.dll  00B7  FindResourceA   
1  00EC12DC  user32.dll  00BA  EnableWindow   
1  00EC12F0  kernel32.dll  0209  MultiByteToWideChar   
1  00EC12F8  kernel32.dll  018E  GetTimeZoneInformation   
1  00EC12FC  kernel32.dll  01E4  LCMapStringW   
1  00EC1304  user32.dll  0234  SetFocus   
1  00EC1308  kernel32.dll  00A4  FindClose   
1  00EC130C  kernel32.dll  0294  SetEnvironmentVariableA   
1  00EC1314  kernel32.dll  0092  ExitThread   
1  00EC1318  user32.dll  0147  GetSysColor   
1  00EC131C  user32.dll  0160  GetWindowPlacement   
1  00EC1320  kernel32.dll  0308  WideCharToMultiByte   
1  00EC1324  user32.dll  01E2  PeekMessageA   
1  00EC1328  comctl32.dll  0011  InitCommonControls   
1  00EC132C  advapi32.dll  01AE  RegQueryValueA   
1  00EC1330  user32.dll  018C  IsDialogMessage   
1  00EC1334  kernel32.dll  0244  ReadFile   
1  00EC1338  gdi32.dll  01FB  SetTextColor   
1  00EC133C  kernel32.dll  02F8  VirtualFree   
1  00EC1340  user32.dll  0297  UpdateWindow   
1  00EC1344  kernel32.dll  016D  GetStdHandle   
1  00EC1348  gdi32.dll  004F  CreateSolidBrush   
1  00EC134C  kernel32.dll  01E3  LCMapStringA   
1  00EC135C  user32.dll  012E  GetMessageA   
1  00EC1360  ws2\_32.dll  0002  bind   
1  00EC1364  user32.dll  0197  IsWindowVisible   
1  00EC136C  ntdll.dll  029A  RtlUnwind   
1  00EC1378  gdi32.dll  01D0  SelectPalette   
1  00EC137C  kernel32.dll  0061  DeleteFileA   
1  00EC1380  kernel32.dll  0181  GetTempFileNameA   
1  00EC1384  user32.dll  0264  SetWindowTextA   
1  00EC1388  user32.dll  0139  GetParent   
1  00EC138C  kernel32.dll  0091  ExitProcess   
1  00EC1394  user32.dll  00BC  EndDialog   
1  00EC1398  advapi32.dll  0190  RegCreateKeyExA   
1  00EC139C  kernel32.dll  01BD  HeapDestroy   
1  00EC13A4  kernel32.dll  001F  CloseHandle   
1  00EC13AC  user32.dll  01E4  PostMessageA   
1  00EC13B0  wsock32.dll  0011  recvfrom   
1  00EC13C0  ntdll.dll  0150  RtlAllocateHeap   
1  00EC13C8  kernel32.dll  00C6  FreeEnvironmentStringsA   
1  00EC13CC  ws2\_32.dll  0004  connect   
1  00EC13D8  user32.dll  014A  GetSystemMetrics   
1  00EC13DC  kernel32.dll  014B  GetOEMCP   
1  00EC13E0  kernel32.dll  0154  GetPrivateProfileStringA   
1  00EC13E4  kernel32.dll  02D2  TerminateProcess   
1  00EC13EC  kernel32.dll  0183  GetTempPathA   
1  00EC13F4  kernel32.dll  00DF  GetCommandLineA   
1  00EC13F8  kernel32.dll  0179  GetSystemTime   
1  00EC13FC  ws2\_32.dll  0009  htons   
1  00EC1404  user32.dll  019F  LoadCursorA   
1  00EC1410  kernel32.dll  02E2  UnhandledExceptionFilter   
1  00EC1418  kernel32.dll  01F9  LockResource   
1  00EC1420  user32.dll  018A  IsChild   
1  00EC1434  wsock32.dll  0010  recv   
1  00EC143C  kernel32.dll  01F9  LockResource   
1  00EC1440  kernel32.dll  00A8  FindFirstFileA   
1  00EC1444  user32.dll  0059  CreateMenu   
1  00EC1450  kernel32.dll  025B  ResumeThread   
1  00EC145C  comctl32.dll  002C  ImageList\_Create   
1  00EC1460  kernel32.dll  00CE  GetACP   
1  00EC1488  kernel32.dll  012D  GetFileType   
1  00EC148C  user32.dll  019A  KillTimer   
1  00EC1490  kernel32.dll  0111  GetCurrentThreadId   
1  00EC1494  kernel32.dll  01B3  GlobalUnlock   
1  00EC149C  advapi32.dll  01A5  RegOpenKeyExA   
1  00EC14A0  ws2\_32.dll  0008  htonl   
1  00EC14A4  ntdll.dll  0251  RtlReAllocateHeap   
1  00EC14A8  advapi32.dll  018C  RegCloseKey   
1  00EC14B0  user32.dll  01CF  MoveWindow   
1  00EC14B4  advapi32.dll  0195  RegDeleteValueA   
1  00EC14BC  user32.dll  005B  CreateWindowExA   
1  00EC14C0  kernel32.dll  0171  GetStringTypeW   
1  00EC14C4  user32.dll  003E  CloseClipboard   
1  00EC14CC  advapi32.dll  01AF  RegQueryValueExA   
1  00EC14D4  user32.dll  01A5  LoadImageA   
1  00EC14D8  kernel32.dll  00C6  FreeEnvironmentStringsA   
1  00EC14E0  gdi32.dll  0046  CreatePen   
1  00EC14E4  kernel32.dll  0158  GetProcAddress   
1  00EC14E8  user32.dll  01D9  OpenClipboard   
1  00EC14F0  kernel32.dll  016E  GetStringTypeA   
1  00EC14F4  user32.dll  0096  DialogBoxParamA   
1  00EC1504  user32.dll  010C  GetForegroundWindow   
1  00EC150C  user32.dll  0198  IsZoomed   
1  00EC1510  kernel32.dll  01BF  HeapFree   
1  00EC1514  kernel32.dll  012E  GetFullPathNameA   
1  00EC1518  gdi32.dll  017F  GetTextMetricsA   
1  00EC1528  gdi32.dll  0055  DeleteObject   
1  00EC152C  kernel32.dll  003A  CreateFileMappingA   
1  00EC1530  user32.dll  01F7  RegisterClassA   
1  00EC1534  kernel32.dll  002C  CopyFileA   
1  00EC1538  user32.dll  0101  GetDC   
1  00EC153C  user32.dll  0208  ReleaseDC   
1  00EC1540  gdi32.dll  01AE  Polygon   
1  00EC1544  kernel32.dll  0132  GetLastError   
1  00EC154C  kernel32.dll  02F5  VirtualAlloc   
1  00EC1554  user32.dll  000D  BeginPaint   
1  00EC155C  ws2\_32.dll  0017  socket   
1  00EC1560  kernel32.dll  013F  GetModuleHandleA   
1  00EC1564  gdi32.dll  0192  MoveToEx   
1  00EC1568  kernel32.dll  01CC  InitializeCriticalSection   
1  00EC1578  kernel32.dll  02CA  Sleep   
1  00EC157C  kernel32.dll  013D  GetModuleFileNameA   
1  00EC1580  shell32.dll  0171  ShellExecuteA   
1  00EC1584  gdi32.dll  01D5  SetBkColor   
从上面可以看出，每个 dll 对应的 API 在 IAT 中都不是连续的，而是乱插。这样很难用 ImportREC 完整还原 IAT,会把需要的函数也 cut。因此必须在程序入口处自己写代码来专门处理 IAT.思路是：把上述 dll 名和对应的 API 名移植到程序尾部，创建一个内存映射到 012c1000 处，然后根据 dll 名和 API 名用 getprocaddress 得到所有函数入口地址，并写回其对应的地址。  
 4dd466d0e1e0cc123afbe504da2462ca