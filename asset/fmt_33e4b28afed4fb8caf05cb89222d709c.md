
由于 `OD` 这个函数，故采用这种方法来使用该函数，参考看雪论坛 7 中的记事本置顶中看雪的问题回复

未解决跨平台，导致写的比较乱,请见谅

```asm
00566380    6C              INS BYTE PTR ES:[EDI],DX                  ; I/O 命令  
00566381    73 74           JNB SHORT 复件_oll.005663F7  
00566383    72 6C           JB SHORT 复件_oll.005663F1  
00566385    65:6E           OUTS DX,BYTE PTR ES:[EDI]                ; I/O 命令  
00566387    41              INC ECX
```

```asm
0056638A    68 72635600     PUSH 复件_oll.00566372            ; ASCII "kernel32.dll"  
0056638F    E8 DE8CF4FF     CALL <JMP.&KERNEL32.GetModuleHandleA>  
00566394    68 80635600     PUSH 复件_oll.00566380             ; ASCII "lstrlenA"  
00566399    50              PUSH EAX  
0056639A    E8 F18CF4FF     CALL <JMP.&KERNEL32.GetProcAddress>  
0056639F    FFD0            CALL EAX  
005663A1  ^ E9 ACFEFFFF     JMP 复件_oll.00566252
```

这样就会 `OD` 插入了一个工具菜单，但菜单无法响应事件，我们继续

### 第五，让添加的工具菜单响应事件

为了能识别相对路径，我们采用 `CreateProcessA` 函数和完成

流程->取选择的工具名称->读取路径->执行读取的路径

#### 所有函数原型

```c
int GetMenuString(
  HMENU hMenu,      // handle to the menu
  UINT uIDItem,     // menu item identifier
  LPTSTR lpString,  // buffer for the string
  int nMaxCount,    // maximum length of string
  UINT uFlag        // options
);

VOID GetStartupInfo(
    LPSTARTUPINFO lpStartupInfo         // address of STARTUPINFO structure  
   );

BOOL CreateProcess(
    LPCTSTR lpApplicationName,        // pointer to name of executable module  
    LPTSTR lpCommandLine,        // pointer to command line string
    LPSECURITY_ATTRIBUTES lpProcessAttributes,        // pointer to process security attributes  
    LPSECURITY_ATTRIBUTES lpThreadAttributes,        // pointer to thread security attributes  
    BOOL bInheritHandles,        // handle inheritance flag  
    DWORD dwCreationFlags,        // creation flags
    LPVOID lpEnvironment,        // pointer to new environment block
    LPCTSTR lpCurrentDirectory,        // pointer to current directory name
    LPSTARTUPINFO lpStartupInfo,        // pointer to STARTUPINFO  
    LPPROCESS_INFORMATION lpProcessInformation         // pointer to PROCESS_INFORMATION   
   );
```

#### 找消息循环的位置

我找的位置

```asm
00433A1F  |> \81EA C7090000 SUB EDX,9C7
00433A25  |.  0F84 37070000 JE ollydbgb.00434162
```

改成

```asm
00433A1F   >- \E9 7C281300   JMP 复件_oll.005662A0
00433A24      90            NOP
005662A0    81FA F00A0000   CMP EDX,0AF0    如果这个根本不我们工具菜单 ID  返回
005662A6  ^ 7C E4           JL SHORT 复件_oll.0056628C
005662A8    81FA 0E0B0000   CMP EDX,0B0E     0B0E 是 2830,目录只支持 30 个，想要多的话，大家改大一下就行了
005662AE  ^ 7F DC           JG SHORT 复件_oll.0056628C
005662B0    6A 00           PUSH 0
005662B2    6A 30           PUSH 30
005662B4    68 00705600     PUSH 复件_oll.00567000   存放我们取到的工具名
005662B9    52              PUSH EDX
005662BA    8B15 7C3B4D00   MOV EDX,DWORD PTR DS:[4D3B7C]
005662C0    52              PUSH EDX
005662C1    E8 A291F4FF     CALL <JMP.&USER32.GetMenu>  
005662C6    50              PUSH EAX
005662C7    E8 AE91F4FF     CALL <JMP.&USER32.GetMenuStringA>
```

把取到的菜单工具名放到 `567000` 这个地址处

```asm
005662CC    68 A4534D00     PUSH 复件_oll.004D53A4
005662D1    6A 70           PUSH 70
005662D3    68 40705600     PUSH 复件_oll.00567040
005662D8    68 F4615600     PUSH 复件_oll.005661F4
005662DD    68 00705600     PUSH 复件_oll.00567000
005662E2    68 E6615600     PUSH 复件_oll.005661E6                     ; ASCII "Tools"
005662E7    E8 9E8DF4FF     CALL <JMP.&KERNEL32.GetPrivateProfileStr>
```

根据取的工具名取出相应的路径

```asm
005662EC    68 20715600     PUSH 复件_oll.00567120
005662F1    E8 A68DF4FF     CALL <JMP.&KERNEL32.GetStartupInfoA>
005662F6    68 66715600     PUSH 复件_oll.00567166
005662FB    68 20715600     PUSH 复件_oll.00567120
00566300    6A 00           PUSH 0
00566302    6A 00           PUSH 0
00566304    6A 20           PUSH 20
00566306    6A 00           PUSH 0
00566308    6A 00           PUSH 0
0056630A    6A 00           PUSH 0
0056630C    68 40705600     PUSH 复件_oll.00567040
00566311    6A 00           PUSH 0
00566313    E8 B88CF4FF     CALL <JMP.&KERNEL32.CreateProcessA>
```

把路径放到 `CreateProcess` 中执行，让他打开就行了

```asm
00566318  - E9 F5DEECFF     JMP 复件_oll.00434212  跳回默认消息处
```

到此，工具菜单就添加到 `OD` 中了，为了能方便的配置 `OD`，写了一个简单配置程序，放到 `Tools` 下了.

用 `ResHacker` 给选项菜单加一个“工具菜单的配置”`ID` 为 `2305`

添加代码的方法和上面一样，找消息循环，执行程序,源码下面

```asm
0056600E    81FA 01090000   CMP EDX,901
00566014    74 1F           JE SHORT 复件_oll.00566035
00566016    81EA 61090000   SUB EDX,961
0056601C  - E9 BDD9ECFF     JMP 复件_oll.004339DE
00566021    90              NOP
```

字符串`"tools\\tools.exe"`

```asm
00566022    0000            ADD BYTE PTR DS:[EAX],AL
00566024    74 6F           JE SHORT 复件_oll.00566095
00566026    6F              OUTS DX,DWORD PTR ES:[EDI]               ; I/O 命令
00566027    6C              INS BYTE PTR ES:[EDI],DX                  ; I/O 命令
00566028    73 5C           JNB SHORT 复件_oll.00566086
0056602A    74 6F           JE SHORT 复件_oll.0056609B
0056602C    6F              OUTS DX,DWORD PTR ES:[EDI]               ; I/O 命令
0056602D    6C              INS BYTE PTR ES:[EDI],DX                  ; I/O 命令
0056602E    73 2E           JNB SHORT 复件_oll.0056605E
00566030    65:78 65        JS SHORT 复件_oll.00566098                  ; 多余的前缀
00566033    0000            ADD BYTE PTR DS:[EAX],AL
```

执行代码

```asm
00566035    68 B1605600     PUSH 复件_oll.005660B1
0056603A    E8 5D90F4FF     CALL <JMP.&KERNEL32.GetStartupInfoA>
0056603F    68 F5605600     PUSH 复件_oll.005660F5
00566044    68 B1605600     PUSH 复件_oll.005660B1
00566049    6A 00           PUSH 0
0056604B    6A 00           PUSH 0
0056604D    6A 20           PUSH 20
0056604F    6A 00           PUSH 0
00566051    6A 00           PUSH 0
00566053    6A 00           PUSH 0
00566055    68 24605600     PUSH 复件_oll.00566024                     ; ASCII "tools\\tools.exe"
0056605A    6A 00           PUSH 0
0056605C    E8 6F8FF4FF     CALL <JMP.&KERNEL32.CreateProcessA>
00566061  - E9 ACE1ECFF     JMP 复件_oll.00434212
```

好了，到些结束，修修补补，代码比较乱，慢慢看吧，第一次写这么多，可以有些东西没有说明白，回复中提问。

下载地址: http://www.fs2you.com/files/eea01547-be8f-11dc-9541-0014221b798a/
