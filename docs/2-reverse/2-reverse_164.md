

# 给 OD 添加工具菜单

> 原文：[`www.pediy.com/kssd/pediy10/58098.html`](https://www.pediy.com/kssd/pediy10/58098.html)

给 OD 添加工具菜单  
大家好，第一次写这种文章，写的不好，请见谅，有什么不明白的地方，请提出，谢谢！  
OD 是我们常用的一个软件调试，在这里我就不做多的介绍了，进入正题  
  
参考文章：API 手册，打造自己喜欢的 Ollydbg、看雪论坛精华 7 中的文章  
  
修改流程->增加空间->添加函数->找插入菜单的位置->读取配置列表->插入菜单->处理工具菜单事件  
  
  
  
第一步，增加空间，肯定需要有空间了，OD 本身是不够的，我们用 ZeroAdd，为 OD 添加一个区段，大小为 2000（HEX），名字 patch，把我们所要写的代码写到这个区段中  
  
第二步，添加函数，我们这次用到是 lstrlenA，这个函数取数据，打开 LoaderPE->PE 编辑->修改的 OD->目录->单击“输入表”后面的"..."按钮，打开输入表对话，在 OLE32.dll 这行单击右键选择“添加导入表”，在打开的“添加导入函数”窗口中 DLL 处输入：Kernel32.dll,在 API 处，输入“lstrlenA”，点确定->关闭添加导入表对话框->保存->保存－>关闭 Loadpe，到时函数添加完成  
  
第三步，找 OD 启动时的消息流，让他根本我们的配置加载工具菜单。方法：大家都知道，OD 在启动后会加载插件，就让 OD 添加插件菜单后，添加我们的工具菜单吧。用另一个 OD 打开我们要修改的 OD，插入菜单的 API 是 InsertMenu，  
在 OD 命令行输入 bp InsertMenuA,F9 运行,OD 断下，Alt＋F9 返回,来到这里  
  
00496B06  |.  51            PUSH ECX                                 ; /pNewItem  
00496B07  |.  8B4424 0C     MOV EAX,DWORD PTR SS:\[ESP+C\]             ; |  
00496B0B  |.  50            PUSH EAX                                 ; |NewItemID  
00496B0C  |.  68 10040000   PUSH 410                                 ; |Flags = MF\_BYPOSITION|MF\_ENABLED|MF\_STRING|MF\_POPUP  
00496B11  |.  6A 03         PUSH 3                                   ; |ItemID = 3  
00496B13  |.  8B15 7C3B4D00 MOV EDX,DWORD PTR DS:\[4D3B7C\]            ; |  
00496B19  |.  52            PUSH EDX                                 ; |/hWnd => 0014056A (class='Aytcgjb')  
00496B1A  |.  E8 49890100   CALL <JMP.&USER32.GetMenu>               ; |\\GetMenu  
00496B1F  |.  50            PUSH EAX                                 ; |hMenu  
00496B20  |.  E8 9D890100   CALL <JMP.&USER32.InsertMenuA>           ; \\InsertMenuA  
00496B25  |.  8B0D 7C3B4D00 MOV ECX,DWORD PTR DS:\[4D3B7C\]  
00496B2B  |.  51            PUSH ECX                                 ; /hWnd => 0014056A (class='Aytcgjb')  
00496B2C  |.  E8 A1880100   CALL <JMP.&USER32.DrawMenuBar>           ; \\DrawMenuBar  
  
断续返回  
  
00438EE6   .  833D B8544D00>CMP DWORD PTR DS:\[4D54B8\],0  
00438EED   .  0F84 27010000 JE ollydbgb.0043901A  
00438EF3   .  C705 C8544D00>MOV DWORD PTR DS:\[4D54C8\],1  
00438EFD   .  F605 CC544D00>TEST BYTE PTR DS:\[4D54CC\],2  
00438F04   .  74 05         JE SHORT ollydbgb.00438F0B  
00438F06   .  E8 E5220200   CALL ollydbgb.\_Createlistwindow  
00438F0B   >  F605 CC544D00>TEST BYTE PTR DS:\[4D54CC\],4  
  
此时，OD 已经添加了插件菜单，该我们动手了，  
  
第四步，插入工具菜单  
  
566000－5667FFE 是我们添加的空间  
00438EE6  CMP DWORD PTR DS:\[4D54B8\],0 改成 JMP 005661FE   跳到我们的空间去  
  
字符串"Tools",ollydbg.ini 中的工具列表区节的区节名,OD 添加如下  
  
005661E6    54              PUSH ESP  
005661E7    6F              OUTS DX,DWORD PTR ES:\[EDI\]               ; I/O 命令  
005661E8    6F              OUTS DX,DWORD PTR ES:\[EDI\]               ; I/O 命令  
005661E9    6C              INS BYTE PTR ES:\[EDI\],DX                 ; I/O 命令  
005661EA    73 00           JNB SHORT 复件\_oll.005661EC  
  
字符串"工具(&T)",为 OD 添加的菜单名称  
  
005661D5    B9 A4BEDF28     MOV ECX,28DFBEA4  
005661DA    26:54           PUSH ESP                                 ; 多余的前缀  
005661DC    2900            SUB DWORD PTR DS:\[EAX\],EAX  
  
  
  
DIY 一定要注意的问题，一、堆栈平台问题，二、跨平台问题，本人首次修改，都遇到了  
流程－>读取工具列表名称－>添加到一个菜单中－》插入菜单  
用到函数：  
创建一个菜单,如果成功返回创建菜单的句柄，没有参数  
HMENU CreateMenu(VOID);  
  
取 INI 的配置文件,如果 lpKeyName 为 NULL，则读取这个区节中所有工具名称  
DWORD GetPrivateProfileString(  
  LPCTSTR lpAppName,        // section name  
  LPCTSTR lpKeyName,        // key name  
  LPCTSTR lpDefault,        // default string  
  LPTSTR lpReturnedString,  // destination buffer  
  DWORD nSize,              // size of destination buffer  
  LPCTSTR lpFileName        // initialization file name  
);  
  
把读取到的工具名称，追到了一个菜单中,我们要用他，把读取的工具名称添加到菜单句柄  
BOOL AppendMenu(  
  HMENU hMenu,         // handle to menu  
  UINT uFlags,         // menu-item options  
  UINT\_PTR uIDNewItem, // identifier, menu, or submenu  
  LPCTSTR lpNewItem    // menu-item content  
);  
把创建的菜单添加到 OD 的菜单中  
BOOL InsertMenu(  
  HMENU hMenu,          // handle to menu  
  UINT uPosition,       // item that new item precedes  
  UINT uFlags,          // options  
  UINT\_PTR uIDNewItem,  // identifier, menu, or submenu  
  LPCTSTR lpNewItem     // menu item content  
);  
重画菜单栏  
BOOL DrawMenuBar(  
  HWND hWnd  // handle to window  
);  
  
堆栈平衡问题：做完以后遇到堆栈平台问题，popad 出栈时，堆栈中参数少了一个，所以使用使用两次 PUSHAD，大家以后再 DIY 时，注意一下。  
我们开始了  
  
005661FE    9C              PUSHFD      寄存器之栈  
005661FF    60              PUSHAD        
00566200    60              PUSHAD  
00566201    68 A4534D00     PUSH 复件\_oll.004D53A4                     ; INI 的文件路径  
00566206    68 00020000     PUSH 200                                 ; 200 个字节的位置  
0056620B    68 00775600     PUSH 复件\_oll.00567700                     ; 存放取出字符串的地址  
00566210    68 F4615600     PUSH 复件\_oll.005661F4  
00566215    6A 00           PUSH 0  
00566217    68 E6615600     PUSH 复件\_oll.005661E6                     ; ASCII "Tools"  
0056621C    E8 698EF4FF     CALL <JMP.&KERNEL32.GetPrivateProfileStr>  
读取 ini 中的工具名列表  
  
00566221    BF 00775600     MOV EDI,复件\_oll.00567700  
00566226    803F 00         CMP BYTE PTR DS:\[EDI\],0  
00566229  ^ 0F84 49FFFFFF   JE 复件\_oll.00566178  
这里列表是否为空，如果为空，则直接跳回返回处     
  
0056622F    E8 5C91F4FF     CALL <JMP.&USER32.CreateMenu>  
00566234    8BD8            MOV EBX,EAX  
00566236    75 05           JNZ SHORT 复件\_oll.0056623D  
00566238  ^ E9 44FFFFFF     JMP 复件\_oll.00566181  
创建一个菜单句柄，如果失败，跳到返回处  
  
0056623D    BE F10A0000     MOV ESI,0AF1          0AF1 是第一个菜单的 ID＝2801  
00566242    57              PUSH EDI              EDI 的地址中存放要添加的工具名称  
00566243    56              PUSH ESI              菜单的 ID  
00566244    6A 00           PUSH 0                指定添加方式   
00566246    53              PUSH EBX              追到菜单的句柄  
00566247    E8 0291F4FF     CALL <JMP.&USER32.AppendMenuA>  
添加一个菜单  
  
0056624C    57              PUSH EDI  
0056624D    E9 38010000     JMP 复件\_oll.0056638A      lstrlenA，跨平台问题移动下面  
00566252    03F8            ADD EDI,EAX               地址＋长度+1＝下一个工具名的位置  
00566254    47              INC EDI                     
00566255    46              INC ESI          菜单 ID 也要加 1   
00566256    803F 00         CMP BYTE PTR DS:\[EDI\],0  
00566259  ^ 75 E7           JNZ SHORT 复件\_oll.00566242  
如果读完后，继续  
  
0056625B    68 D5615600     PUSH 复件\_oll.005661D5  
00566260    53              PUSH EBX  
00566261    68 10040000     PUSH 410  
00566266    6A 05           PUSH 5  
00566268    8B15 7C3B4D00   MOV EDX,DWORD PTR DS:\[4D3B7C\]  
0056626E    52              PUSH EDX  
0056626F    E8 F491F4FF     CALL <JMP.&USER32.GetMenu>  
00566274    50              PUSH EAX  
00566275    E8 4892F4FF     CALL <JMP.&USER32.InsertMenuA>  
插入菜单,参加 OD 插入插件菜单的方法  
  
0056627A    8B0D 7C3B4D00   MOV ECX,DWORD PTR DS:\[4D3B7C\]  
00566280    E8 4D91F4FF     CALL <JMP.&USER32.DrawMenuBar>  
重画菜单栏  
00566285  ^ E9 EFFEFFFF     JMP 复件\_oll.00566179  
  
跳到返回处  
  
  
返回处代码  
00566178    58              POP EAX  
00566179    58              POP EAX  
0056617A    58              POP EAX  
0056617B    58              POP EAX  
0056617C    58              POP EAX  
0056617D    58              POP EAX  
0056617E    58              POP EAX  
0056617F    58              POP EAX  
00566180    61              POPAD  
00566181    9D              POPFD  
00566182    833D B8544D00 0>CMP DWORD PTR DS:\[4D54B8\],0  
00566189  - E9 5F2DEDFF     JMP 复件\_oll.00438EED  
  
  
  
  
字符串"kernel32.dll"  
00566372    6B65 72 6E      IMUL ESP,DWORD PTR SS:\[EBP+72\],6E  
00566376    65:6C           INS BYTE PTR ES:\[EDI\],DX                 ; I/O 命令  
00566378    3332            XOR ESI,DWORD PTR DS:\[EDX\]  
0056637A    2E:             PREFIX CS:                               ; 多余的前缀  
0056637B    64:6C           INS BYTE PTR ES:\[EDI\],DX                 ; I/O 命令  
0056637D    6C              INS BYTE PTR ES:\[EDI\],DX                 ; I/O 命令  
  
字符串"lstrlenA"  
00566380    6C              INS BYTE PTR ES:\[EDI\],DX                 ; I/O 命令  
00566381    73 74           JNB SHORT 复件\_oll.005663F7  
00566383    72 6C           JB SHORT 复件\_oll.005663F1  
00566385    65:6E           OUTS DX,BYTE PTR ES:\[EDI\]                ; I/O 命令  
00566387    41              INC ECX  
  
由于 OD 这个函数，故采用这种方法来使用该函数，参考看雪论坛 7 中的记事本置顶中看雪的问题回复  
  
未解决跨平台，导致写的比较乱,请见谅  
  
0056638A    68 72635600     PUSH 复件\_oll.00566372            ; ASCII "kernel32.dll"  
0056638F    E8 DE8CF4FF     CALL <JMP.&KERNEL32.GetModuleHandleA>  
00566394    68 80635600     PUSH 复件\_oll.00566380             ; ASCII "lstrlenA"  
00566399    50              PUSH EAX  
0056639A    E8 F18CF4FF     CALL <JMP.&KERNEL32.GetProcAddress>  
0056639F    FFD0            CALL EAX  
005663A1  ^ E9 ACFEFFFF     JMP 复件\_oll.00566252  
  
这样就会 OD 插入了一个工具菜单，但菜单无法响应事件，我们继续  
  
第五，让添加的工具菜单响应事件，为了能识别相对路径，我们采用 CreateProcessA 函数和完成  
  
流程->取选择的工具名称->读取路径－>执行读取的路径  
  
所有函数原型  
  
int GetMenuString(  
  HMENU hMenu,      // handle to the menu  
  UINT uIDItem,     // menu item identifier  
  LPTSTR lpString,  // buffer for the string  
  int nMaxCount,    // maximum length of string  
  UINT uFlag        // options  
);  
  
VOID GetStartupInfo(  
  
    LPSTARTUPINFO lpStartupInfo         // address of STARTUPINFO structure   
   );  
     
  
BOOL CreateProcess(  
  
    LPCTSTR lpApplicationName,        // pointer to name of executable module   
    LPTSTR lpCommandLine,        // pointer to command line string  
    LPSECURITY\_ATTRIBUTES lpProcessAttributes,        // pointer to process security attributes   
    LPSECURITY\_ATTRIBUTES lpThreadAttributes,        // pointer to thread security attributes   
    BOOL bInheritHandles,        // handle inheritance flag   
    DWORD dwCreationFlags,        // creation flags   
    LPVOID lpEnvironment,        // pointer to new environment block   
    LPCTSTR lpCurrentDirectory,        // pointer to current directory name   
    LPSTARTUPINFO lpStartupInfo,        // pointer to STARTUPINFO   
    LPPROCESS\_INFORMATION lpProcessInformation         // pointer to PROCESS\_INFORMATION    
   );  
  
找消息循环的位置  
  
我找的位置  
00433A1F  |> \\81EA C7090000 SUB EDX,9C7  
00433A25  |.  0F84 37070000 JE ollydbgb.00434162  
改成  
  
00433A1F   >-\\E9 7C281300   JMP 复件\_oll.005662A0  
00433A24      90            NOP  
  
  
005662A0    81FA F00A0000   CMP EDX,0AF0    如果这个根本不我们工具菜单 ID  返回  
005662A6  ^ 7C E4           JL SHORT 复件\_oll.0056628C  
005662A8    81FA 0E0B0000   CMP EDX,0B0E     0B0E 是 2830,目录只支持 30 个，想要多的话，大家改大一下就行了  
大于这个不是我们的菜单 ID，返回    
005662AE  ^ 7F DC           JG SHORT 复件\_oll.0056628C  
  
  
005662B0    6A 00           PUSH 0  
005662B2    6A 30           PUSH 30  
005662B4    68 00705600     PUSH 复件\_oll.00567000   存放我们取到的工具名  
005662B9    52              PUSH EDX  
005662BA    8B15 7C3B4D00   MOV EDX,DWORD PTR DS:\[4D3B7C\]  
005662C0    52              PUSH EDX  
005662C1    E8 A291F4FF     CALL <JMP.&USER32.GetMenu>   
005662C6    50              PUSH EAX  
005662C7    E8 AE91F4FF     CALL <JMP.&USER32.GetMenuStringA>  
  
把取到的菜单工具名放到 567000 这个地址处  
  
005662CC    68 A4534D00     PUSH 复件\_oll.004D53A4  
005662D1    6A 70           PUSH 70  
005662D3    68 40705600     PUSH 复件\_oll.00567040  
005662D8    68 F4615600     PUSH 复件\_oll.005661F4  
005662DD    68 00705600     PUSH 复件\_oll.00567000  
005662E2    68 E6615600     PUSH 复件\_oll.005661E6                     ; ASCII "Tools"  
005662E7    E8 9E8DF4FF     CALL <JMP.&KERNEL32.GetPrivateProfileStr>  
根据取的工具名取出相应的路径  
  
  
005662EC    68 20715600     PUSH 复件\_oll.00567120  
005662F1    E8 A68DF4FF     CALL <JMP.&KERNEL32.GetStartupInfoA>  
005662F6    68 66715600     PUSH 复件\_oll.00567166  
005662FB    68 20715600     PUSH 复件\_oll.00567120  
00566300    6A 00           PUSH 0  
00566302    6A 00           PUSH 0  
00566304    6A 20           PUSH 20  
00566306    6A 00           PUSH 0  
00566308    6A 00           PUSH 0  
0056630A    6A 00           PUSH 0  
0056630C    68 40705600     PUSH 复件\_oll.00567040  
00566311    6A 00           PUSH 0  
00566313    E8 B88CF4FF     CALL <JMP.&KERNEL32.CreateProcessA>  
  
把路径放到 CreateProcess 中执行，让他打开就行了  
  
00566318  - E9 F5DEECFF     JMP 复件\_oll.00434212  跳回默认消息处  
  
  
到此，工具菜单就添加到 OD 中了，为了能方便的配置 OD，写了一个简单配置程序，放到 Tools 下了.  
  
用 ResHacker 给选项菜单加一个“工具菜单的配置”ID 为 2305  
  
添加代码的方法和上面一样，找消息循环，执行程序,源码下面  
  
  
  
0056600E    81FA 01090000   CMP EDX,901  
00566014    74 1F           JE SHORT 复件\_oll.00566035  
00566016    81EA 61090000   SUB EDX,961  
0056601C  - E9 BDD9ECFF     JMP 复件\_oll.004339DE  
00566021    90              NOP  
  
字符串"tools\\tools.exe"  
  
00566022    0000            ADD BYTE PTR DS:\[EAX\],AL  
00566024    74 6F           JE SHORT 复件\_oll.00566095  
00566026    6F              OUTS DX,DWORD PTR ES:\[EDI\]               ; I/O 命令  
00566027    6C              INS BYTE PTR ES:\[EDI\],DX                 ; I/O 命令  
00566028    73 5C           JNB SHORT 复件\_oll.00566086  
0056602A    74 6F           JE SHORT 复件\_oll.0056609B  
0056602C    6F              OUTS DX,DWORD PTR ES:\[EDI\]               ; I/O 命令  
0056602D    6C              INS BYTE PTR ES:\[EDI\],DX                 ; I/O 命令  
0056602E    73 2E           JNB SHORT 复件\_oll.0056605E  
00566030    65:78 65        JS SHORT 复件\_oll.00566098                 ; 多余的前缀  
00566033    0000            ADD BYTE PTR DS:\[EAX\],AL  
  
执行代码  
00566035    68 B1605600     PUSH 复件\_oll.005660B1  
0056603A    E8 5D90F4FF     CALL <JMP.&KERNEL32.GetStartupInfoA>  
0056603F    68 F5605600     PUSH 复件\_oll.005660F5  
00566044    68 B1605600     PUSH 复件\_oll.005660B1  
00566049    6A 00           PUSH 0  
0056604B    6A 00           PUSH 0  
0056604D    6A 20           PUSH 20  
0056604F    6A 00           PUSH 0  
00566051    6A 00           PUSH 0  
00566053    6A 00           PUSH 0  
00566055    68 24605600     PUSH 复件\_oll.00566024                     ; ASCII "tools\\tools.exe"  
0056605A    6A 00           PUSH 0  
0056605C    E8 6F8FF4FF     CALL <JMP.&KERNEL32.CreateProcessA>  
00566061  - E9 ACE1ECFF     JMP 复件\_oll.00434212  
  
  
  
好了，到些结束，修修补补，代码比较乱，慢慢看吧，第一次写这么多，可以有些东西没有说明白，回复中提问。  
  
下载地址:  http://www.fs2you.com/files/eea01547-be8f-11dc-9541-0014221b798a/