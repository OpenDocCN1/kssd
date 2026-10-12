
# EncryptPE 2003.5.18

> 原文：[`www.pediy.com/kssd/pediy06/pediy6673.htm`](https://www.pediy.com/kssd/pediy06/pediy6673.htm)

【脱文作者】 `simonzh2000[US]`

【使用工具】 `Peid0.92`, `Ollydbg1.10(反 Antidbg 版)`, `ImportREC1.60`, `LordPE`

【破解平台】 `Win2000SP4 English`

【软件名称】 `EncryptPE 2003.5.18`

【软件简介】 老王的壳 2003.5.18

【加壳方式】 自己

【作者声明】 本笔记只用于学习交流, 初学 Crack，只是感兴趣技术，没有其他目的, 如有不妥之处, 请谅解.

老王同志很大方, 免费给大家使用. 我也为大家抛快砖, 引点玉出来.

用 `IsDebug` 插件去掉 `OD` 的调试器标志。

忽略除了 “`INT3 异常`” 之外的其它异常, 添加“ 忽略 `0EEDFADE` ”异常。

```
004B7000 > 60             PUSHAD  //进入 OD 后停在这
004B7001  9C             PUSHFD
004B7002  64:FF35 0000000>PUSH DWORD PTR FS:[0]
004B7009  E8 79010000    CALL EncryptP.004B7187
```

`F9` 运行，程序会中断在 `INT3` 异常处，`Shift+F9` 通过异常, 有时会出来警告, 恭喜, 你中奖了, 重新来过.

经过几次 `INT3` 异常后, 程序会在 `0EEDFADE` 处长时间异常，所以上面忽略了这个指定异常。

上个 `WC` , 回来后, `OD` 已停下,

```
7119CF57 CC            int3          //异常, Shift+F9 过
7119CF58 90            nop
7119CF59 64:8F05 00000000 pop dword ptr fs:[0]
7119CF60 C3            retn

77F9FFE4 8B0424        MOV EAX,DWORD PTR SS:[ESP]  // C0000008 异常, 二次, shift+F9 过
77F9FFE7 8BE5          MOV ESP,EBP
77F9FFE9 5D            POP EBP
77F9FFEA C3            RETN

7119CF57 CC            int3          //异常, 停，看看堆栈
7119CF58 90            nop
7119CF59 64:8F05 00000000 pop dword ptr fs:[0]
7119CF60 C3            retn
```

```
// Stack
0012FF98 0012FFE0 Pointer to next SEH record
0012FF9C 7119CE8D SE handler             // 到 7119C8ED 下断, Shift+F9
```

```
7119CE8D  53         PUSH EBX
7119CE8E  52         PUSH EDX
7119CE8F  8B5C24 14  MOV EBX,DWORD PTR SS:[ESP+14]         // CONTEXT 结构的指针
7119CE93  8B93 C4000000 MOV EDX,DWORD PTR DS:[EBX+C4]
7119CE99  8B83 C0000000 MOV EAX,DWORD PTR DS:[EBX+C0]
7119CE9F  A3 38F61B71 MOV DWORD PTR DS:[711BF638],EAX
7119CEA4  E8 8F040000 CALL V1200351.7119D338               // F8
7119CEA9  9C         PUSHFD
7119CEAA  58         POP EAX
7119CEAB  A3 38F61B71 MOV DWORD PTR DS:[711BF638],EAX
7119CEB0  E8 83040000 CALL V1200351.7119D338               // F8
7119CEB5  8B83 B8000000 MOV EAX,DWORD PTR DS:[EBX+B8]
7119CEBB  40         INC EAX
7119CEBC  8983 B8000000 MOV DWORD PTR DS:[EBX+B8],EAX
7119CEC2  8B4424 0C  MOV EAX,DWORD PTR SS:[ESP+C]
7119CEC6  8B00       MOV EAX,DWORD PTR DS:[EAX]
7119CEC8  3D 03000080 CMP EAX,80000003
7119CECD  75 71      JNZ SHORT V1200351.7119CF40
7119CECF  803D 54F61B71 0>CMP BYTE PTR DS:[711BF654],1
7119CED6  74 4F      JE SHORT V1200351.7119CF27
7119CED8  8B42 0C    MOV EAX,DWORD PTR DS:[EDX+C]
7119CEDB  8983 9C000000 MOV DWORD PTR DS:[EBX+9C],EAX
7119CEE1  8B42 10    MOV EAX,DWORD PTR DS:[EDX+10]
7119CEE4  8983 A0000000 MOV DWORD PTR DS:[EBX+A0],EAX
7119CEEA  8B42 14    MOV EAX,DWORD PTR DS:[EDX+14]
7119CEED  8983 B4000000 MOV DWORD PTR DS:[EBX+B4],EAX
7119CEF3  8B42 1C    MOV EAX,DWORD PTR DS:[EDX+1C]
7119CEF6  8983 A4000000 MOV DWORD PTR DS:[EBX+A4],EAX
7119CEFC  8B42 20    MOV EAX,DWORD PTR DS:[EDX+20]
7119CEFF  8983 A8000000 MOV DWORD PTR DS:[EBX+A8],EAX
7119CF05  8B42 24    MOV EAX,DWORD PTR DS:[EDX+24]
7119CF08  8983 AC000000 MOV DWORD PTR DS:[EBX+AC],EAX
7119CF0E  8B42 28    MOV EAX,DWORD PTR DS:[EDX+28]
7119CF11  8983 B0000000 MOV DWORD PTR DS:[EBX+B0],EAX         // EAX = 499780, 异常处理完毕这里继续, OEP
7119CF17  8B02       MOV EAX,DWORD PTR DS:[EDX]
7119CF19  8942 24    MOV DWORD PTR DS:[EDX+24],EAX
7119CF1C  89D0       MOV EAX,EDX
7119CF1E  83C0 24    ADD EAX,24
7119CF21  8983 C4000000 MOV DWORD PTR DS:[EBX+C4],EAX
7119CF27  31C0       XOR EAX,EAX
7119CF29  8943 04    MOV DWORD PTR DS:[EBX+4],EAX
7119CF2C  8943 08    MOV DWORD PTR DS:[EBX+8],EAX
7119CF2F  8943 0C    MOV DWORD PTR DS:[EBX+C],EAX
7119CF32  8943 10    MOV DWORD PTR DS:[EBX+10],EAX
7119CF35  C743 18 5501000>MOV DWORD PTR DS:[EBX+18],155
7119CF3C  5A         POP EDX
7119CF3D  5B         POP EBX
7119CF3E  C3         RETN

00499780 55         PUSH EBP                   // 到这里下断, F9,  下面修复 IAT
00499781 8BEC       MOV EBP,ESP
00499783 83C4 F0    ADD ESP,-10
00499786 B8 98954900 MOV EAX,EncryptP.00499598
0049978B E8 D4D3F6FF CALL EncryptP.00406B64
```

// 往下找找, 发现程序里有很多这样的东西

```
00406AA0  90         NOP
00406AA1 - E9 8A70A000 JMP 00E0DB30
00406AA6  8BC0       MOV EAX,EAX
00406AA8  90         NOP
00406AA9 - E9 E26FA000 JMP 00E0DA90
00406AAE  8BC0       MOV EAX,EAX
00406AB0  90         NOP
00406AB1 - E9 4670A000 JMP 00E0DAFC
00406AB6  8BC0       MOV EAX,EAX
00406AB8  90         NOP
00406AB9 - E9 1270A000 JMP 00E0DAD0
00406ABE  8BC0       MOV EAX,EAX
```

// API

```
00406AA0  90         NOP
00406AA1 - E9 EE71A000 JMP 00E0DC94
...
```

```
7119D2FD  8B4424 0C   MOV EAX,DWORD PTR SS:[ESP+C]    // 如果 JMP E0DC94
7119D301  89C3        MOV EBX,EAX                    // 那么 EAX=E0DC99
7119D303  83C0 02     ADD EAX,2
7119D306  8B00        MOV EAX,DWORD PTR DS:[EAX]
7119D308  8B00        MOV EAX,DWORD PTR DS:[EAX]
7119D30A  31D8        XOR EAX,EBX
7119D30C  894424 0C   MOV DWORD PTR SS:[ESP+C],EAX    // EAX 即 API
7119D310  8B00        MOV EAX,DWORD PTR DS:[EAX]
7119D312  3C CC       CMP AL,0CC                     // 检查 1
7119D314  74 14       JE SHORT V1200351.7119D32A
7119D316  80FC CC     CMP AH,0CC                     // 检查 2
7119D319  74 0F       JE SHORT V1200351.7119D32A
7119D31B  C1E8 10     SHR EAX,10
7119D31E  3C CC       CMP AL,0CC                     // 检查 3
7119D320  74 08       JE SHORT V1200351.7119D32A
7119D322  80FC CC     CMP AH,0CC                     // 检查 4
7119D325  74 03       JE SHORT V1200351.7119D32A
7119D327  EB 08       JMP SHORT V1200351.7119D331
7119D329 - E9 C60554F6 JMP 676DD8F4
7119D32E  1B71 01     SBB ESI,DWORD PTR DS:[ECX+1]
7119D331  5B          POP EBX
7119D332  58          POP EAX
7119D333  9D          POPFD
7119D334  C3          RETN
```

// 上面就是壳解密 `API` 的过程

// 写一段恢复 `API` 的补丁程序, 放到 `7119CF60`

```
7119CF60   60          PUSHAD
7119CF61   B8 50124000  MOV EAX,401250                  ; // 搜索从 401250 开始
7119CF66   BA 00000101  MOV EDX,1010000                 ; // 把 API 放到 1010000 开始的空闲区域
7119CF6B   66:8138 90E9  CMP WORD PTR DS:[EAX],0E990             ; // 90 E9 = NOP, JMP XXXXXXX
7119CF70   0F85 2F000000 JNZ V1200351.7119CFA5
7119CF76   8BC8         MOV ECX,EAX                   ; // [EAX] is  90 E9
7119CF78   8B40 02      MOV EAX,DWORD PTR DS:[EAX+2]          ; // EAX = XXXXXXXX
7119CF7B   03C1         ADD EAX,ECX
7119CF7D   83C0 06      ADD EAX,6
7119CF80   3D 00000070  CMP EAX,70000000                ; // > 7000 0000 就是 API
7119CF85   0F87 0E000000 JA V1200351.7119CF99
7119CF8B   83C0 05      ADD EAX,5
7119CF8E   8BD8         MOV EBX,EAX
7119CF90   83C0 02      ADD EAX,2
7119CF93   8B00         MOV EAX,DWORD PTR DS:[EAX]
7119CF95   8B00         MOV EAX,DWORD PTR DS:[EAX]
```



## 汇编代码段

```
7119CF97 33C3               XOR EAX,EBX                     ; // < 7000 0000 的 API  
7119CF99 8902               MOV DWORD PTR DS:[EDX],EAX      ; // 保存 API Address  
7119CF9B 83C2 04            ADD EDX,4  
7119CF9E 8BC1               MOV EAX,ECX  
7119CFA0 90                 NOP  
7119CFA1 90                 NOP  
7119CFA2 90                 NOP  
7119CFA3 90                 NOP  
7119CFA4 90                 NOP  
7119CFA5 83C0 04            ADD EAX,4  
7119CFA8 3D 208C4300        CMP EAX,438C20                  ; // 搜索到 438C20 结束  
7119CFAD ^ 72 BC           JB SHORT V1200351.7119CF6B  
7119CFAF 61                 POPAD  
```

## 原始字节流

```
60 B8 50 12 40 00 BA 00 00 01 01 66 81 38 90 E9 0F 85 2F 00 00 00 8B C8 8B 40 02 03 C1 83 C0 06 3D 00 00 00  
70 0F 87 0E 00 00 00 83 C0 05 8B D8 83 C0 02 8B 00 8B 00 33 C3 89 02 83 C2 04 8B C1 90 90 90 90 90 83 C0 04  
3D 20 8C 43 00 72 BC 61
```

## 导入表信息

- `IMPortRec`, `VA=1010000`, `RVA = C10000`, `Size = 688`, Get Imports 得到 `IAT`
- `OEP: 00099780`, `IATRVA: 00C10000`, `IATSize: 00000688`
- `FThunk: 00C10000`, `NbFunc: 000001A2`

## 导入函数列表

```
1  00C10000  kernel32.dll  001F  CloseHandle
1  00C10004  kernel32.dll  0039  CreateFileA
1  00C10008  kernel32.dll  012D  GetFileType
1  00C1000C  kernel32.dll  012A  GetFileSize
1  00C10010  kernel32.dll  016D  GetStdHandle
1  00C10014  kernel32.dll  0237  RaiseException
1  00C10018  kernel32.dll  0244  ReadFile
1  00C1001C  kernel32.dll  025E  RtlUnwind
1  00C10020  kernel32.dll  0293  SetEndOfFile
1  00C10024  kernel32.dll  029C  SetFilePointer
1  00C10028  kernel32.dll  02E2  UnhandledExceptionFilter
1  00C1002C  kernel32.dll  0315  WriteFile
1  00C10030  user32.dll    0026  CharNextA
1  00C10034  kernel32.dll  0091  ExitProcess
1  00C10038  user32.dll    01C4  MessageBoxA
1  00C1003C  kernel32.dll  00A4  FindClose
1  00C10040  kernel32.dll  00A8  FindFirstFileA
1  00C10044  kernel32.dll  00C8  FreeLibrary
1  00C10048  kernel32.dll  00DF  GetCommandLineA
1  00C1004C  kernel32.dll  0132  GetLastError
1  00C10050  kernel32.dll  0135  GetLocaleInfoA
1  00C10054  kernel32.dll  013D  GetModuleFileNameA
1  00C10058  kernel32.dll  013F  GetModuleHandleA
1  00C1005C  kernel32.dll  0158  GetProcAddress
1  00C10060  kernel32.dll  016B  GetStartupInfoA
1  00C10064  kernel32.dll  0186  GetThreadLocale
1  00C10068  kernel32.dll  01E7  LoadLibraryExA
1  00C1006C  user32.dll    01B0  LoadStringA
1  00C10070  kernel32.dll  0338  lstrcpyn
1  00C10074  kernel32.dll  033B  lstrlen
1  00C10078  kernel32.dll  0209  MultiByteToWideChar
1  00C1007C  advapi32.dll  018C  RegCloseKey
1  00C10080  advapi32.dll  01A5  RegOpenKeyExA
1  00C10084  advapi32.dll  01AF  RegQueryValueExA
1  00C10088  kernel32.dll  0308  WideCharToMultiByte
1  00C1008C  kernel32.dll  02FD  VirtualQuery
1  00C10090  oleaut32.dll  0004  SysAllocStringLen
1  00C10094  oleaut32.dll  0005  SysReAllocStringLen
1  00C10098  oleaut32.dll  0006  SysFreeString
1  00C1009C  kernel32.dll  01D2  InterlockedIncrement
1  00C100A0  kernel32.dll  01CF  InterlockedDecrement
1  00C100A4  kernel32.dll  0111  GetCurrentThreadId
1  00C100A8  kernel32.dll  01EC  LocalAlloc
1  00C100AC  kernel32.dll  01F0  LocalFree
1  00C100B0  kernel32.dll  02F5  VirtualAlloc
1  00C100B4  kernel32.dll  02F8  VirtualFree
1  00C100B8  kernel32.dll  01CC  InitializeCriticalSection
1  00C100BC  kernel32.dll  0074  EnterCriticalSection
1  00C100C0  kernel32.dll  01E5  LeaveCriticalSection
1  00C100C4  kernel32.dll  005F  DeleteCriticalSection
1  00C100C8  kernel32.dll  0244  ReadFile
1  00C100CC  kernel32.dll  0315  WriteFile
1  00C100D0  user32.dll    011C  GetKeyboardType
1  00C100D4  kernel32.dll  013F  GetModuleHandleA
1  00C100D8  kernel32.dll  01EC  LocalAlloc
1  00C100DC  kernel32.dll  02D9  TlsGetValue
1  00C100E0  kernel32.dll  02DA  TlsSetValue
1  00C100E4  advapi32.dll  018C  RegCloseKey
1  00C100E8  advapi32.dll  01A5  RegOpenKeyExA
1  00C100EC  advapi32.dll  01AF  RegQueryValueExA
1  00C100F0  kernel32.dll  001F  CloseHandle
1  00C100F4  kernel32.dll  0025  CompareStringA
1  00C100F8  kernel32.dll  002C  CopyFileA
1  00C100FC  kernel32.dll  0035  CreateEventA
1  00C10100  kernel32.dll  0039  CreateFileA
1  00C10104  kernel32.dll  0052  CreateThread
1  00C10108  kernel32.dll  005F  DeleteCriticalSection
1  00C1010C  kernel32.dll  0061  DeleteFileA
1  00C10110  kernel32.dll  0074  EnterCriticalSection
1  00C10114  kernel32.dll  0075  EnumCalendarInfoA
1  00C10118  kernel32.dll  009C  FileTimeToDosDateTime
1  00C1011C  kernel32.dll  009D  FileTimeToLocalFileTime
1  00C10120  kernel32.dll  00A4  FindClose
1  00C10124  kernel32.dll  00A8  FindFirstFileA
1  00C10128  kernel32.dll  00B7  FindResourceA
1  00C1012C  kernel32.dll  00C3  FormatMessageA
1  00C10130  kernel32.dll  00C8  FreeLibrary
1  00C10134  kernel32.dll  00CA  FreeResource
1  00C10138  kernel32.dll  00CE  GetACP
1  00C1013C  kernel32.dll  00D4  GetCPInfo
1  00C10140  kernel32.dll  010F  GetCurrentProcessId
1  00C10144  kernel32.dll  0111  GetCurrentThreadId
1  00C10148  kernel32.dll  0112  GetDateFormatA
1  00C1014C  kernel32.dll  0118  GetDiskFreeSpaceA
1  00C10150  kernel32.dll  012C  GetFileTime
1  00C10154  kernel32.dll  0132  GetLastError
1  00C10158  kernel32.dll  0134  GetLocalTime
1  00C1015C  kernel32.dll  0135  GetLocaleInfoA
1  00C10160  kernel32.dll  013D  GetModuleFileNameA
1  00C10164  kernel32.dll  013F  GetModuleHandleA
1  00C10168  kernel32.dll  0158  GetProcAddress
1  00C1016C  kernel32.dll  0166  GetProfileStringA
1  00C10170  kernel32.dll  016D  GetStdHandle
1  00C10174  kernel32.dll  016F  GetStringTypeExA
1  00C10178  kernel32.dll  0175  GetSystemDirectoryA
1  00C1017C  kernel32.dll  0177  GetSystemInfo
1  00C10180  kernel32.dll  0186  GetThreadLocale
1  00C10184  kernel32.dll  018B  GetTickCount
1  00C10188  kernel32.dll  0193  GetVersion
1  00C1018C  kernel32.dll  0194  GetVersionExA
1  00C10190  kernel32.dll  019F  GlobalAddAtomA
1  00C10194  kernel32.dll  01A1  GlobalAlloc
1  00C10198  kernel32.dll  01A3  GlobalDeleteAtom
1  00C1019C  kernel32.dll  01A4  GlobalFindAtomA
1  00C101A0  kernel32.dll  01A8  GlobalFree
1  00C101A4  kernel32.dll  01AC  GlobalLock
1  00C101A8  kernel32.dll  01AB  GlobalHandle
1  00C101AC  kernel32.dll  01AF  GlobalReAlloc
1  00C101B0  kernel32.dll  01B3  GlobalUnlock
1  00C101B4  kernel32.dll  01CC  InitializeCriticalSection
1  00C101B8  kernel32.dll  01E5  LeaveCriticalSection
1  00C101BC  kernel32.dll  01E6  LoadLibraryA
1  00C101C0  kernel32.dll  01EB  LoadResource
1  00C101C4  kernel32.dll  01F9  LockResource
1  00C101C8  kernel32.dll  01FC  MapViewOfFile
1  00C101CC  kernel32.dll  0208  MulDiv
1  00C101D0  kernel32.dll  0212  OpenFileMappingA
1  00C101D4  kernel32.dll  0244  ReadFile
1  00C101D8  kernel32.dll  0259  ResetEvent
1  00C101DC  kernel32.dll  0293  SetEndOfFile
1  00C101E0  kernel32.dll  0296  SetErrorMode
1  00C101E4  kernel32.dll  0297  SetEvent
1  00C101E8  kernel32.dll  029C  SetFilePointer
1  00C101EC  kernel32.dll  029E  SetFileTime
1  00C101F0  kernel32.dll  02BA  SetThreadLocale
1  00C101F4  kernel32.dll  02C9  SizeofResource
1  00C101F8  kernel32.dll  02CA  Sleep
1  00C101FC  kernel32.dll  02E5  UnmapViewOfFile
1  00C10200  kernel32.dll  02F5  VirtualAlloc
1  00C10204  kernel32.dll  02FD  VirtualQuery
1  00C10208  kernel32.dll  0304  WaitForSingleObject
1  00C1020C  kernel32.dll  0315  WriteFile
1  00C10210  kernel32.dll  032F  lstrcmp
1  00C10214  kernel32.dll  0335  lstrcpy
```



## IAT 导出表

### `version.dll`
- `00C10218` 序号 `0001`：`GetFileVersionInfoA`
- `00C1021C` 序号 `0002`：`GetFileVersionInfoSizeA`
- `00C10220` 序号 `000B`：`VerQueryValueA`

### `gdi32.dll`
- `00C10224` 序号 `0013`：`BitBlt`
- `00C10228` 序号 `0022`：`CopyEnhMetaFileA`
- `00C1022C` 序号 `0026`：`CreateBitmap`
- `00C10230` 序号 `0028`：`CreateBrushIndirect`
- `00C10234` 序号 `002B`：`CreateCompatibleBitmap`
- `00C10238` 序号 `002C`：`CreateCompatibleDC`
- `00C1023C` 序号 `002D`：`CreateDCA`
- `00C10240` 序号 `0031`：`CreateDIBSection`
- `00C10244` 序号 `0032`：`CreateDIBitmap`
- `00C10248` 序号 `0039`：`CreateFontIndirectA`
- `00C1024C` 序号 `003E`：`CreateHalftonePalette`
- `00C10250` 序号 `0040`：`CreateICA`
- `00C10254` 序号 `0044`：`CreatePalette`
- `00C10258` 序号 `0047`：`CreatePenIndirect`
- `00C1025C` 序号 `004F`：`CreateSolidBrush`
- `00C10260` 序号 `0052`：`DeleteDC`
- `00C10264` 序号 `0053`：`DeleteEnhMetaFile`
- `00C10268` 序号 `0055`：`DeleteObject`
- `00C1026C` 序号 `005C`：`EndDoc`
- `00C10270` 序号 `005E`：`EndPage`
- `00C10274` 序号 `009D`：`ExcludeClipRect`
- `00C10278` 序号 `00A3`：`ExtTextOutA`
- `00C1027C` 序号 `00E0`：`GdiFlush`
- `00C10280` 序号 `010E`：`GetBitmapBits`
- `00C10284` 序号 `0113`：`GetBrushOrgEx`
- `00C10288` 序号 `0123`：`GetClipBox`
- `00C1028C` 序号 `0128`：`GetCurrentPositionEx`
- `00C10290` 序号 `012A`：`GetDCOrgEx`
- `00C10294` 序号 `012C`：`GetDIBColorTable`
- `00C10298` 序号 `012D`：`GetDIBits`
- `00C1029C` 序号 `012E`：`GetDeviceCaps`
- `00C102A0` 序号 `0134`：`GetEnhMetaFileBits`
- `00C102A4` 序号 `0137`：`GetEnhMetaFileHeader`
- `00C102A8` 序号 `0138`：`GetEnhMetaFilePaletteEntries`
- `00C102AC` 序号 `0158`：`GetObjectA`
- `00C102B0` 序号 `015D`：`GetPaletteEntries`
- `00C102B4` 序号 `015F`：`GetPixel`
- `00C102B8` 序号 `0168`：`GetStockObject`
- `00C102BC` 序号 `016C`：`GetSystemPaletteEntries`
- `00C102C0` 序号 `0177`：`GetTextExtentPoint32A`
- `00C102C4` 序号 `017F`：`GetTextMetricsA`
- `00C102C8` 序号 `0184`：`GetWinMetaFileBits`
- `00C102CC` 序号 `0186`：`GetWindowOrgEx`
- `00C102D0` 序号 `018A`：`IntersectClipRect`
- `00C102D4` 序号 `0190`：`LineTo`
- `00C102D8` 序号 `0191`：`MaskBlt`
- `00C102DC` 序号 `0194`：`MoveToEx`
- `00C102E0` 序号 `01A0`：`PatBlt`
- `00C102E4` 序号 `01A3`：`PlayEnhMetaFile`
- `00C102E8` 序号 `01B1`：`Polyline`
- `00C102EC` 序号 `01B6`：`RealizePalette`
- `00C102F0` 序号 `01B8`：`RectVisible`
- `00C102F4` 序号 `01B9`：`Rectangle`
- `00C102F8` 序号 `01C3`：`RestoreDC`
- `00C102FC` 序号 `01CA`：`SaveDC`
- `00C10300` 序号 `01D1`：`SelectObject`
- `00C10304` 序号 `01D2`：`SelectPalette`
- `00C10308` 序号 `01D3`：`SetAbortProc`
- `00C1030C` 序号 `01D7`：`SetBkColor`
- `00C10310` 序号 `01D8`：`SetBkMode`
- `00C10314` 序号 `01DA`：`SetBrushOrgEx`
- `00C10318` 序号 `01DF`：`SetDIBColorTable`
- `00C1031C` 序号 `01E3`：`SetEnhMetaFileBits`
- `00C10320` 序号 `01EC`：`SetMapMode`
- `00C10324` 序号 `01F2`：`SetPixel`
- `00C10328` 序号 `01F6`：`SetROP2`
- `00C1032C` 序号 `01F9`：`SetStretchBltMode`
- `00C10330` 序号 `01FD`：`SetTextColor`
- `00C10334` 序号 `0200`：`SetViewportOrgEx`
- `00C10338` 序号 `0202`：`SetWinMetaFileBits`
- `00C1033C` 序号 `0204`：`SetWindowOrgEx`
- `00C10340` 序号 `0206`：`StartDocA`
- `00C10344` 序号 `0209`：`StartPage`
- `00C10348` 序号 `020A`：`StretchBlt`
- `00C1034C` 序号 `0213`：`UnrealizeObject`

### `user32.dll`
- `00C10350` 序号 `0001`：`ActivateKeyboardLayout`
- `00C10354` 序号 `0003`：`AdjustWindowRectEx`
- `00C10358` 序号 `0022`：`CharLowerA`
- `00C1035C` 序号 `0008`：`AppendMenuA`
- `00C10360` 序号 `000D`：`BeginPaint`
- `00C10364` 序号 `0016`：`CallNextHookEx`
- `00C10368` 序号 `0017`：`CallWindowProcA`
- `00C1036C` 序号 `0023`：`CharLowerBuffA`
- `00C10370` 序号 `0026`：`CharNextA`
- `00C10374` 序号 `0031`：`CharUpperBuffA`
- `00C10378` 序号 `0035`：`CheckMenuItem`
- `00C1037C` 序号 `0038`：`ChildWindowFromPoint`
- `00C10380` 序号 `003C`：`ClientToScreen`
- `00C10384` 序号 `0053`：`CreateIcon`
- `00C10388` 序号 `0059`：`CreateMenu`
- `00C1038C` 序号 `005A`：`CreatePopupMenu`
- `00C10390` 序号 `005B`：`CreateWindowExA`
- `00C10394` 序号 `0083`：`DefFrameProcA`
- `00C10398` 序号 `0085`：`DefMDIChildProcA`
- `00C1039C` 序号 `0087`：`DefWindowProcA`
- `00C103A0` 序号 `008A`：`DeleteMenu`
- `00C103A4` 序号 `008E`：`DestroyCursor`
- `00C103A8` 序号 `008E`：`DestroyCursor`
- `00C103AC` 序号 `0090`：`DestroyMenu`
- `00C103B0` 序号 `0091`：`DestroyWindow`
- `00C103B4` 序号 `0098`：`DispatchMessageA`
- `00C103B8` 序号 `00A8`：`DrawEdge`
- `00C103BC` 序号 `00A9`：`DrawFocusRect`
- `00C103C0` 序号 `00AB`：`DrawFrameControl`
- `00C103C4` 序号 `00AC`：`DrawIcon`
- `00C103C8` 序号 `00AD`：`DrawIconEx`
- `00C103CC` 序号 `00AE`：`DrawMenuBar`
- `00C103D0` 序号 `00B2`：`DrawTextA`
- `00C103D4` 序号 `00B8`：`EnableMenuItem`
- `00C103D8` 序号 `00B9`：`EnableScrollBar`
- `00C103DC` 序号 `00BA`：`EnableWindow`
- `00C103E0` 序号 `00BE`：`EndPaint`
- `00C103E4` 序号 `00D0`：`EnumThreadWindows`
- `00C103E8` 序号 `00D3`：`EnumWindows`
- `00C103EC` 序号 `00D4`：`EqualRect`
- `00C103F0` 序号 `00D7`：`FillRect`
- `00C103F4` 序号 `00D8`：`FindWindowA`
- `00C103F8` 序号 `00DE`：`FrameRect`
- `00C103FC` 序号 `00E0`：`GetActiveWindow`
- `00C10400` 序号 `00E8`：`GetCapture`
- `00C10404` 序号 `00EB`：`GetClassInfoA`
- `00C10408` 序号 `00F1`：`GetClassNameA`
- `00C1040C` 序号 `00F4`：`GetClientRect`
- `00C10410` 序号 `00F6`：`GetClipboardData`
- `00C10414` 序号 `00FD`：`GetCursor`
- `00C10418` 序号 `0100`：`GetCursorPos`
- `00C1041C` 序号 `0101`：`GetDC`
- `00C10420` 序号 `0102`：`GetDCEx`
- `00C10424` 序号 `0103`：`GetDesktopWindow`
- `00C10428` 序号 `0106`：`GetDlgItem`
- `00C1042C` 序号 `010B`：`GetFocus`
- `00C10430` 序号 `010C`：`GetForegroundWindow`
- `00C10434` 序号 `010F`：`GetIconInfo`
- `00C10438` 序号 `0114`：`GetKeyNameTextA`
- `00C1043C` 序号 `0116`：`GetKeyState`
- `00C10440` 序号 `0117`：`GetKeyboardLayout`
- `00C10444` 序号 `0118`：`GetKeyboardLayoutList`
- `00C10448` 序号 `011B`：`GetKeyboardState`
- `00C1044C` 序号 `011D`：`GetLastActivePopup`
- `00C10450` 序号 `0120`：`GetMenu`
- `00C10454` 序号 `0126`：`GetMenuItemCount`
- `00C10458` 序号 `0127`：`GetMenuItemID`
- `00C1045C` 序号 `0128`：`GetMenuItemInfoA`
- `00C10460` 序号 `012B`：`GetMenuState`
- `00C10464` 序号 `012C`：`GetMenuStringA`
- `00C10468` 序号 `0130`：`GetMessagePos`
- `00C1046C` 序号 `0157`：`GetWindow`
- `00C10470` 序号 `0139`：`GetParent`
- `00C10474` 序号 `013E`：`GetPropA`
- `00C10478` 序号 `0142`：`GetScrollInfo`
- `00C1047C` 序号 `0143`：`GetScrollPos`
- `00C10480` 序号 `0144`：`GetScrollRange`
- `00C10484` 序号 `0146`：`GetSubMenu`
- `00C10488` 序号 `0147`：`GetSysColor`
- `00C1048C` 序号 `0149`：`GetSystemMenu`
- `00C10490` 序号 `014A`：`GetSystemMetrics`
- `00C10494` 序号 `0150`：`GetTopWindow`
- `00C10498` 序号 `0151`：`GetUpdateRect`
- `00C1049C` 序号 `0157`：`GetWindow`
- `00C104A0` 序号 `0159`：`GetWindowDC`
- `00C104A4` 序号 `015B`：`GetWindowLongA`
- `00C104A8` 序号 `0160`：`GetWindowPlacement`
- `00C104AC` 序号 `0161`：`GetWindowRect`
- `00C104B0` 序号 `0163`：`GetWindowTextA`
- `00C104B4` 序号 `0167`：`GetWindowThreadProcessId`



# 导入地址表 (IAT) 函数列表

## user32.dll
- `GetWindowThreadProcessId`
- `InflateRect`
- `InsertMenuA`
- `InsertMenuItemA`
- `IntersectRect`
- `InvalidateRect`
- `IsChild`
- `IsDialogMessage`
- `IsIconic`
- `IsRectEmpty`
- `IsWindow`
- `IsWindowEnabled`
- `IsWindowVisible`
- `IsZoomed`
- `KillTimer`
- `LoadBitmapA`
- `LoadCursorA`
- `LoadIconA`
- `LoadKeyboardLayoutA`
- `LoadStringA`
- `MapVirtualKeyA`
- `MapWindowPoints`
- `MessageBoxA`
- `OemToCharA`
- `OffsetRect`
- `PeekMessageA`
- `PostMessageA`
- `PostQuitMessage`
- `PtInRect`
- `RedrawWindow`
- `RegisterClassA`
- `RegisterClipboardFormatA`
- `ReleaseCapture`
- `ReleaseDC`
- `RemoveMenu`
- `RemovePropA`
- `ScreenToClient`
- `ScrollWindow`
- `SendMessageA`
- `SetActiveWindow`
- `SetCapture`
- `SetClassLongA`
- `SetCursor`
- `SetFocus`
- `SetForegroundWindow`
- `SetMenu`
- `SetMenuItemInfoA`
- `SetPropA`
- `SetRect`
- `SetScrollInfo`
- `SetScrollPos`
- `SetScrollRange`
- `SetTimer`
- `SetWindowLongA`
- `SetWindowPlacement`
- `SetWindowPos`
- `SetWindowTextA`
- `SetWindowsHookExA`
- `ShowCursor`
- `ShowOwnedPopups`
- `ShowScrollBar`
- `ShowWindow`
- `SystemParametersInfoA`
- `TrackPopupMenu`
- `TranslateMDISysAccel`
- `TranslateMessage`
- `UnhookWindowsHookEx`
- `UnregisterClassA`
- `UpdateWindow`
- `WaitMessage`
- `WinHelpA`
- `WindowFromPoint`

## kernel32.dll
- `Sleep`

## oleaut32.dll
- `VariantInit`
- `VariantClear`
- `VariantCopy`
- `VariantCopyInd`
- `VariantChangeType`
- `SafeArrayCreate`
- `SafeArrayRedim`
- `SafeArrayGetLBound`
- `SafeArrayGetUBound`
- `SafeArrayGetElement`
- `SafeArrayPutElement`
- `SafeArrayPtrOfIndex`

## ole32.dll
- `CoCreateGuid`

## comctl32.dll
- `InitCommonControls`
- `ImageList_Create`
- `ImageList_Destroy`
- `ImageList_GetImageCount`
- `ImageList_Add`
- `ImageList_ReplaceIcon`
- `ImageList_SetBkColor`
- `ImageList_GetBkColor`
- `ImageList_Draw`
- `ImageList_DrawEx`
- `ImageList_Remove`
- `ImageList_BeginDrag`
- `ImageList_EndDrag`
- `ImageList_DragEnter`
- `ImageList_DragLeave`
- `ImageList_DragMove`
- `ImageList_SetDragCursorImage`
- `ImageList_DragShowNolock`
- `ImageList_GetDragImage`
- `ImageList_Read`
- `ImageList_Write`
- `ImageList_GetIconSize`
- `ImageList_SetIconSize`

## winspool.drv
- `ClosePrinter`
- `DocumentPropertiesA`
- `EnumPrintersA`
- `OpenPrinterA`

## shell32.dll
- `ShellAboutA`

## comdlg32.dll
- `GetOpenFileNameA`

## 重新整理 IAT

所有函数都 OK 了，但顺序不对，需要重新整理。

找一个空闲空间 `4A8000-4AB000`，放整理好后的 IAT：

```
Kernel32     122    4A8000 - 4A81E4
User32       167    4A81EC - 4A8484
GDI32        75     4A848C - 4A85B4
comctl32     23     4A85BC - 4A8614
Oleaut32     15     4A861C - 4A8654
Advapi32     6      4A865C - 4A8670
version      3      4A8678 - 4A8680
Ole32        1      4A8688 - 4A8688
winspool     4      4A8690 - 4A869C
shell32      1      4A86A4 - 4A86A4
comdlg32     1      4A86AC - 4A86AC
```

## 数据拷贝

把下面数据拷贝到 `1011000`（用 Excel 花了我半小时）：

```
00 80 4A 00
04 80 4A 00
08 80 4A 00
0C 80 4A 00
10 80 4A 00
14 80 4A 00
18 80 4A 00
1C 80 4A 00
20 80 4A 00
24 80 4A 00
28 80 4A 00
2C 80 4A 00
EC 81 4A 00
30 80 4A 00
F0 81 4A 00
34 80 4A 00
38 80 4A 00
3C 80 4A 00
40 80 4A 00
44 80 4A 00
48 80 4A 00
4C 80 4A 00
50 80 4A 00
54 80 4A 00
58 80 4A 00
5C 80 4A 00
60 80 4A 00
F4 81 4A 00
64 80 4A 00
68 80 4A 00
6C 80 4A 00
5C 86 4A 00
60 86 4A 00
64 86 4A 00
70 80 4A 00
74 80 4A 00
1C 86 4A 00
20 86 4A 00
24 86 4A 00
78 80 4A 00
7C 80 4A 00
80 80 4A 00
84 80 4A 00
88 80 4A 00
8C 80 4A 00
90 80 4A 00
94 80 4A 00
98 80 4A 00
9C 80 4A 00
A0 80 4A 00
A4 80 4A 00
A8 80 4A 00
F8 81 4A 00
AC 80 4A 00
B0 80 4A 00
B4 80 4A 00
B8 80 4A 00
68 86 4A 00
6C 86 4A 00
70 86 4A 00
BC 80 4A 00
C0 80 4A 00
C4 80 4A 00
C8 80 4A 00
CC 80 4A 00
D0 80 4A 00
D4 80 4A 00
D8 80 4A 00
DC 80 4A 00
E0 80 4A 00
E4 80 4A 00
E8 80 4A 00
EC 80 4A 00
F0 80 4A 00
F4 80 4A 00
F8 80 4A 00
FC 80 4A 00
00 81 4A 00
04 81 4A 00
08 81 4A 00
0C 81 4A 00
10 81 4A 00
14 81 4A 00
18 81 4A 00
1C 81 4A 00
20 81 4A 00
24 81 4A 00
28 81 4A 00
2C 81 4A 00
30 81 4A 00
34 81 4A 00
38 81 4A 00
3C 81 4A 00
40 81 4A 00
44 81 4A 00
48 81 4A 00
4C 81 4A 00
50 81 4A 00
54 81 4A 00
58 81 4A 00
5C 81 4A 00
60 81 4A 00
64 81 4A 00
68 81 4A 00
6C 81 4A 00
70 81 4A 00
74 81 4A 00
78 81 4A 00
```



```
7C 81 4A 00
80 81 4A 00
84 81 4A 00
88 81 4A 00
8C 81 4A 00
90 81 4A 00
94 81 4A 00
98 81 4A 00
9C 81 4A 00
A0 81 4A 00
A4 81 4A 00
A8 81 4A 00
AC 81 4A 00
B0 81 4A 00
B4 81 4A 00
B8 81 4A 00
BC 81 4A 00
C0 81 4A 00
C4 81 4A 00
C8 81 4A 00
CC 81 4A 00
D0 81 4A 00
D4 81 4A 00
D8 81 4A 00
DC 81 4A 00
E0 81 4A 00
78 86 4A 00
7C 86 4A 00
80 86 4A 00
8C 84 4A 00
90 84 4A 00
94 84 4A 00
98 84 4A 00
9C 84 4A 00
A0 84 4A 00
A4 84 4A 00
A8 84 4A 00
AC 84 4A 00
B0 84 4A 00
B4 84 4A 00
B8 84 4A 00
BC 84 4A 00
C0 84 4A 00
C4 84 4A 00
C8 84 4A 00
CC 84 4A 00
D0 84 4A 00
D4 84 4A 00
D8 84 4A 00
DC 84 4A 00
E0 84 4A 00
E4 84 4A 00
E8 84 4A 00
EC 84 4A 00
F0 84 4A 00
F4 84 4A 00
F8 84 4A 00
FC 84 4A 00
00 85 4A 00
04 85 4A 00
08 85 4A 00
0C 85 4A 00
10 85 4A 00
14 85 4A 00
18 85 4A 00
1C 85 4A 00
20 85 4A 00
24 85 4A 00
28 85 4A 00
2C 85 4A 00
30 85 4A 00
34 85 4A 00
38 85 4A 00
3C 85 4A 00
40 85 4A 00
44 85 4A 00
48 85 4A 00
4C 85 4A 00
50 85 4A 00
54 85 4A 00
58 85 4A 00
5C 85 4A 00
60 85 4A 00
64 85 4A 00
68 85 4A 00
6C 85 4A 00
70 85 4A 00
74 85 4A 00
78 85 4A 00
7C 85 4A 00
80 85 4A 00
84 85 4A 00
88 85 4A 00
8C 85 4A 00
90 85 4A 00
94 85 4A 00
98 85 4A 00
9C 85 4A 00
A0 85 4A 00
A4 85 4A 00
A8 85 4A 00
AC 85 4A 00
B0 85 4A 00
B4 85 4A 00
FC 81 4A 00
00 82 4A 00
04 82 4A 00
08 82 4A 00
0C 82 4A 00
10 82 4A 00
14 82 4A 00
18 82 4A 00
1C 82 4A 00
20 82 4A 00
24 82 4A 00
28 82 4A 00
2C 82 4A 00
30 82 4A 00
34 82 4A 00
38 82 4A 00
3C 82 4A 00
40 82 4A 00
44 82 4A 00
48 82 4A 00
4C 82 4A 00
50 82 4A 00
54 82 4A 00
58 82 4A 00
5C 82 4A 00
60 82 4A 00
64 82 4A 00
68 82 4A 00
6C 82 4A 00
70 82 4A 00
74 82 4A 00
78 82 4A 00
7C 82 4A 00
80 82 4A 00
84 82 4A 00
88 82 4A 00
8C 82 4A 00
90 82 4A 00
94 82 4A 00
98 82 4A 00
9C 82 4A 00
A0 82 4A 00
A4 82 4A 00
A8 82 4A 00
AC 82 4A 00
B0 82 4A 00
B4 82 4A 00
B8 82 4A 00
BC 82 4A 00
C0 82 4A 00
C4 82 4A 00
C8 82 4A 00
CC 82 4A 00
D0 82 4A 00
D4 82 4A 00
D8 82 4A 00
DC 82 4A 00
E0 82 4A 00
E4 82 4A 00
E8 82 4A 00
EC 82 4A 00
F0 82 4A 00
F4 82 4A 00
F8 82 4A 00
FC 82 4A 00
00 83 4A 00
04 83 4A 00
08 83 4A 00
0C 83 4A 00
10 83 4A 00
14 83 4A 00
18 83 4A 00
1C 83 4A 00
20 83 4A 00
24 83 4A 00
28 83 4A 00
2C 83 4A 00
30 83 4A 00
34 83 4A 00
38 83 4A 00
3C 83 4A 00
40 83 4A 00
44 83 4A 00
48 83 4A 00
4C 83 4A 00
50 83 4A 00
54 83 4A 00
58 83 4A 00
5C 83 4A 00
60 83 4A 00
64 83 4A 00
68 83 4A 00
6C 83 4A 00
70 83 4A 00
74 83 4A 00
78 83 4A 00
7C 83 4A 00
80 83 4A 00
84 83 4A 00
88 83 4A 00
8C 83 4A 00
90 83 4A 00
94 83 4A 00
98 83 4A 00
9C 83 4A 00
A0 83 4A 00
A4 83 4A 00
A8 83 4A 00
AC 83 4A 00
B0 83 4A 00
B4 83 4A 00
B8 83 4A 00
BC 83 4A 00
C0 83 4A 00
C4 83 4A 00
C8 83 4A 00
CC 83 4A 00
D0 83 4A 00
D4 83 4A 00
D8 83 4A 00
DC 83 4A 00
E0 83 4A 00
E4 83 4A 00
E8 83 4A 00
EC 83 4A 00
F0 83 4A 00
F4 83 4A 00
F8 83 4A 00
FC 83 4A 00
00 84 4A 00
04 84 4A 00
08 84 4A 00
0C 84 4A 00
10 84 4A 00
14 84 4A 00
18 84 4A 00
1C 84 4A 00
20 84 4A 00
24 84 4A 00
28 84 4A 00
2C 84 4A 00
30 84 4A 00
34 84 4A 00
38 84 4A 00
3C 84 4A 00
40 84 4A 00
44 84 4A 00
48 84 4A 00
4C 84 4A 00
50 84 4A 00
54 84 4A 00
58 84 4A 00
5C 84 4A 00
60 84 4A 00
64 84 4A 00
68 84 4A 00
6C 84 4A 00
70 84 4A 00
74 84 4A 00
78 84 4A 00
7C 84 4A 00
80 84 4A 00
84 84 4A 00
E4 81 4A 00
28 86 4A 00
2C 86 4A 00
30 86 4A 00
34 86 4A 00
38 86 4A 00
3C 86 4A 00
40 86 4A 00
44 86 4A 00
48 86 4A 00
4C 86 4A 00
50 86 4A 00
54 86 4A 00
88 86 4A 00
BC 85 4A 00
C0 85 4A 00
C4 85 4A 00
C8 85 4A 00
CC 85 4A 00
D0 85 4A 00
D4 85 4A 00
D8 85 4A 00
DC 85 4A 00
E0 85 4A 00
E4 85 4A 00
E8 85 4A 00
EC 85 4A 00
F0 85 4A 00
F4 85 4A 00
F8 85 4A 00
FC 85 4A 00
00 86 4A 00
04 86 4A 00
08 86 4A 00
0C 86 4A 00
10 86 4A 00
14 86 4A 00
90 86 4A 00
94 86 4A 00
98 86 4A 00
9C 86 4A 00
A4 86 4A 00
AC 86 4A 00
```

再写一段程序, 修复

```
7119CF60    60               PUSHAD
7119CF61    B8 50124000      MOV EAX,401250                  ; // 搜索从 401250 开始
7119CF66    BA 00000101      MOV EDX,1010000                 ; // 从 1010000 开始的区域取 API
7119CF6B    66:8138 90E9    CMP WORD PTR DS:[EAX],0E990       ; // 90 E9 = NOP,
7119CF70    0F85 2F000000   JNZ V1200351.7119CFA5
7119CF76    8BC8             MOV ECX,EAX                       ; // EAX SAVE IN ECX
7119CF78    8B40 02          NOP
7119CF79    90               NOP
7119CF7A    90               NOP
7119CF7B    66:C700 FF25     MOV WORD PTR DS:[EAX],25FF        ; // 改成 JMP [XXXXXXX]
7119CF80    83C0 02          ADD EAX,2
7119CF83    8BDA             MOV EBX,EDX
7119CF85    81C3 00100000    ADD EBX,1000                      ; // 1011000 开始区域放 IAT 地址
7119CF8B    8B1B             MOV EBX,DWORD PTR DS:[EBX]         ; // 取出 XXXXXXX
7119CF8D    8918             MOV DWORD PTR DS:[EAX],EBX
7119CF8F    90               NOP
7119CF90    8B02             MOV EAX,DWORD PTR DS:[EDX]         ; // 取出 API
7119CF92    8903             MOV DWORD PTR DS:[EBX],EAX         ; // [XXXXXXX] = API
7119CF94    90               NOP
7119CF95    90               NOP
7119CF96    90               NOP
7119CF97    90               NOP
7119CF98    90               NOP
7119CF99    90               NOP
7119CF9A    90               NOP
7119CF9B    83C2 04          ADD EDX,4
7119CF9E    8BC1             MOV EAX,ECX
7119CFA0    90               NOP
7119CFA1    90               NOP
7119CFA2    90               NOP
7119CFA3    90               NOP
7119CFA4    90               NOP
7119CFA5    83C0 04          ADD EAX,4
7119CFA8    3D 208C4300      CMP EAX,438C20
7119CFAD    72 BC            JB SHORT V1200351.7119CF6B
7119CFAF    61               POPAD
```

```
60 B8 50 12 40 00 BA 00 00 01 01 66 81 38 90 E9 0F 85 2F 00 00 00 8B C8 90 90 90 66 C7 00 FF 25
83 C0 02 8B DA 81 C3 00 10 00 00 8B 1B 89 18 90 8B 02 89 03 90 90 90 90 90 90 90 83 C2 04 8B C1
90 90 90 90 90 83 C0 04 3D 20 8C 43 00 72 BC 61
```

回到 `499780` dump

`IMPortRec`, `VA=4A8000`, `RVA = A8000`, `Size = 6B0`, `Get Imports` 得到 `IAT`

`RVA = A9000`, `Fixdump`

`4dd466d0e1e0cc123afbe504da2462ca`
