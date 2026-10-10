

# 也谈 Armadillo 3.6 主程序 IAT 处理

> 原文：[`www.pediy.com/kssd/pediy06/pediy6464.htm`](https://www.pediy.com/kssd/pediy06/pediy6464.htm)

OS 2K   
od 1.1b   
od 设置：异常除 Memory access violation 外全部选中   
  
本文目的是为了澄清 armadillo 3.6 主程序对 iat 处理手法的认识。   
  
借用 fxyang 的一句话“前面都是定式了，不要我说…… ”   
当在 OpenMutexA 入口处中断后，ctrl+g 转到 401000，键入：   
pusha   
push edx   
push 0   
push 0   
call CreateMutexA   
popa   
jmp OpenMutexA   
光标置于 401000，Ctrl+Gray \*，再 f9，程序将在 mov \[eax\],eax 处发生异常，连续 shift+f9（共 13 次），堆栈   
中出现指向串"WSOCK32.DLL"的指针，下 bp 0DF18A7，再 shift+f9，来到处理 iat 代码区，如图，修改两个位置   
（0DF1A5B 和 0DF1A8F），去掉 0DF18A7 断点，再下 bp LoadLibraryA 以在 Arma 把 api 导入完毕时给我们机会，并   
把 od 选项中“Memory access violation”打勾，F9 执行程序...   
00DF18A7  PUSH 1   
00DF18A9  POP EAX                                     
00DF18AA  TEST EAX,EAX   
00DF18AC  JE 00DF1B71   
00DF18B2  AND WORD PTR SS:\[EBP\-1978\],0   
00DF18BA  AND DWORD PTR SS:\[EBP\-1980\],0   
00DF18C1  AND DWORD PTR SS:\[EBP\-197C\],0   
00DF18C8  MOV EAX,DWORD PTR SS:\[EBP\-137C\]   
00DF18CE  MOVSX EAX,BYTE PTR DS:\[EAX\]   
00DF18D1  TEST EAX,EAX   
00DF18D3  JNZ SHORT 00DF1919   
00DF18D5  LEA ECX,DWORD PTR SS:\[EBP\-13BC\]   
00DF18DB  CALL 00DD1040   
00DF18E0  MOVZX EAX,AL   
00DF18E3  CDQ   
00DF18E4  PUSH 14   
00DF18E6  POP ECX                                     
00DF18E7  IDIV ECX   
00DF18E9  MOV EAX,DWORD PTR SS:\[EBP\-13E0\]   
00DF18EF  MOV ECX,DWORD PTR SS:\[EBP+EDX\*4-155C\]   
00DF18F6  MOV DWORD PTR DS:\[EAX\],ECX   
00DF18F8  MOV EAX,DWORD PTR SS:\[EBP\-13E0\]   
00DF18FE  ADD EAX,4   
00DF1901  MOV DWORD PTR SS:\[EBP\-13E0\],EAX   
00DF1907  MOV EAX,DWORD PTR SS:\[EBP\-137C\]   
00DF190D  INC EAX   
00DF190E  MOV DWORD PTR SS:\[EBP\-137C\],EAX   
00DF1914  JMP 00DF1B71   
00DF1919  MOV EAX,DWORD PTR SS:\[EBP\-137C\]   
00DF191F  MOVZX EAX,BYTE PTR DS:\[EAX\]   
00DF1922  CMP EAX,0FF   
00DF1927  JNZ 00DF19B7   
00DF192D  MOV EAX,DWORD PTR SS:\[EBP\-137C\]   
00DF1933  INC EAX   
00DF1934  MOV DWORD PTR SS:\[EBP\-137C\],EAX   
00DF193A  MOV EAX,DWORD PTR SS:\[EBP\-137C\]   
00DF1940  MOV AX,WORD PTR DS:\[EAX\]   
00DF1943  MOV WORD PTR SS:\[EBP\-1978\],AX   
00DF194A  MOV EAX,DWORD PTR SS:\[EBP\-137C\]   
00DF1950  INC EAX   
00DF1951  INC EAX   
00DF1952  MOV DWORD PTR SS:\[EBP\-137C\],EAX   
00DF1958  CMP DWORD PTR SS:\[EBP\-1748\],0   
00DF195F  JE SHORT 00DF19B2   
00DF1961  MOV EAX,DWORD PTR SS:\[EBP\-1748\]   
00DF1967  MOV DWORD PTR SS:\[EBP\-1984\],EAX   
00DF196D  JMP SHORT 00DF197E   
00DF196F  MOV EAX,DWORD PTR SS:\[EBP\-1984\]   
00DF1975  ADD EAX,0C   
00DF1978  MOV DWORD PTR SS:\[EBP\-1984\],EAX   
00DF197E  MOV EAX,DWORD PTR SS:\[EBP\-1984\]   
00DF1984  CMP DWORD PTR DS:\[EAX+8\],0   
00DF1988  JE SHORT 00DF19B2   
00DF198A  MOVZX EAX,WORD PTR SS:\[EBP\-1978\]   
00DF1991  MOV ECX,DWORD PTR SS:\[EBP\-1984\]   
00DF1997  MOVZX ECX,WORD PTR DS:\[ECX+4\]   
00DF199B  CMP EAX,ECX   
00DF199D  JNZ SHORT 00DF19B0   
00DF199F  MOV EAX,DWORD PTR SS:\[EBP\-1984\]   
00DF19A5  MOV EAX,DWORD PTR DS:\[EAX+8\]   
00DF19A8  MOV DWORD PTR SS:\[EBP\-197C\],EAX   
00DF19AE  JMP SHORT 00DF19B2   
00DF19B0  JMP SHORT 00DF196F   
00DF19B2  JMP 00DF1A54   
00DF19B7  MOV EAX,DWORD PTR SS:\[EBP\-137C\]   
00DF19BD  MOV DWORD PTR SS:\[EBP\-1980\],EAX   
00DF19C3  PUSH 0   
00DF19C5  PUSH DWORD PTR SS:\[EBP\-137C\]   
00DF19CB  CALL DWORD PTR DS:\[DF82C8\]               ; MSVCRT.strchr   
00DF19D1  POP ECX                                     
00DF19D2  POP ECX                                     
00DF19D3  INC EAX   
00DF19D4  MOV DWORD PTR SS:\[EBP\-137C\],EAX   
00DF19DA  CMP DWORD PTR SS:\[EBP\-1748\],0   
00DF19E1  JE SHORT 00DF1A54   
00DF19E3  MOV EAX,DWORD PTR SS:\[EBP\-1748\]   
00DF19E9  MOV DWORD PTR SS:\[EBP\-1988\],EAX   
00DF19EF  JMP SHORT 00DF1A00   
00DF19F1  MOV EAX,DWORD PTR SS:\[EBP\-1988\]   
00DF19F7  ADD EAX,0C   
00DF19FA  MOV DWORD PTR SS:\[EBP\-1988\],EAX   
00DF1A00  MOV EAX,DWORD PTR SS:\[EBP\-1988\]   
00DF1A06  CMP DWORD PTR DS:\[EAX+8\],0   
00DF1A0A  JE SHORT 00DF1A54   
00DF1A0C  PUSH 100   
00DF1A11  LEA EAX,DWORD PTR SS:\[EBP\-1A88\]   
00DF1A17  PUSH EAX   
00DF1A18  MOV EAX,DWORD PTR SS:\[EBP\-1988\]   
00DF1A1E  PUSH DWORD PTR DS:\[EAX\]   
00DF1A20  CALL 00DD604D   
00DF1A25  ADD ESP,0C   
00DF1A28  LEA EAX,DWORD PTR SS:\[EBP\-1A88\]   
00DF1A2E  PUSH EAX   
00DF1A2F  PUSH DWORD PTR SS:\[EBP\-1980\]   
00DF1A35  CALL DWORD PTR DS:\[DF8334\]               ; MSVCRT.\_stricmp   
00DF1A3B  POP ECX                                     
00DF1A3C  POP ECX                                     
00DF1A3D  TEST EAX,EAX   
00DF1A3F  JNZ SHORT 00DF1A52   
00DF1A41  MOV EAX,DWORD PTR SS:\[EBP\-1988\]   
00DF1A47  MOV EAX,DWORD PTR DS:\[EAX+8\]   
00DF1A4A  MOV DWORD PTR SS:\[EBP\-197C\],EAX   
00DF1A50  JMP SHORT 00DF1A54   
00DF1A52  JMP SHORT 00DF19F1   
00DF1A54  CMP DWORD PTR SS:\[EBP\-197C\],0               
00DF1A5B  JNZ SHORT 00DF1A9C                       ;此处 nop 掉   
00DF1A5D  MOVZX EAX,WORD PTR SS:\[EBP\-1978\]   
00DF1A64  TEST EAX,EAX   
00DF1A66  JE SHORT 00DF1A77   
00DF1A68  MOVZX EAX,WORD PTR SS:\[EBP\-1978\]   
00DF1A6F  MOV DWORD PTR SS:\[EBP\-2E38\],EAX   
00DF1A75  JMP SHORT 00DF1A83   
00DF1A77  MOV EAX,DWORD PTR SS:\[EBP\-1980\]   
00DF1A7D  MOV DWORD PTR SS:\[EBP\-2E38\],EAX   
00DF1A83  PUSH DWORD PTR SS:\[EBP\-2E38\]   
00DF1A89  PUSH DWORD PTR SS:\[EBP\-1744\]                
00DF1A8F  CALL 00DD7EC6                            ;call GetProcAddress   
00DF1A94  POP ECX                                  ;nop   
00DF1A95  POP ECX                                  ;nop   
00DF1A96  MOV DWORD PTR SS:\[EBP\-197C\],EAX   
00DF1A9C  CMP DWORD PTR SS:\[EBP\-197C\],0   
00DF1AA3  JNZ 00DF1B41   
00DF1AA9  MOVZX EAX,WORD PTR SS:\[EBP\-1978\]   
00DF1AB0  TEST EAX,EAX   
00DF1AB2  JE SHORT 00DF1B08   
00DF1AB4  CALL DWORD PTR DS:\[DF80D4\]               ; KERNEL32.GetLastError   
00DF1ABA  CMP EAX,32   
00DF1ABD  JNZ SHORT 00DF1ACB   
00DF1ABF  MOV DWORD PTR SS:\[EBP\-197C\],0DD7EBB   
00DF1AC9  JMP SHORT 00DF1B06   
00DF1ACB  MOV EAX,DWORD PTR SS:\[EBP+8\]   
00DF1ACE  MOV EAX,DWORD PTR DS:\[EAX\]   
00DF1AD0  MOV DWORD PTR DS:\[EAX\],3   
00DF1AD6  CALL DWORD PTR DS:\[DF80D4\]               ; KERNEL32.GetLastError   
00DF1ADC  PUSH EAX   
00DF1ADD  MOVZX EAX,WORD PTR SS:\[EBP\-1978\]   
00DF1AE4  PUSH EAX   
00DF1AE5  PUSH DWORD PTR SS:\[EBP\-1860\]   
00DF1AEB  PUSH 0DFE510                             ; ASCII "File "%s", ordinal %d (error %d)"   
00DF1AF0  MOV EAX,DWORD PTR SS:\[EBP+8\]   
00DF1AF3  PUSH DWORD PTR DS:\[EAX+4\]   
00DF1AF6  CALL DWORD PTR DS:\[DF82C4\]               ; MSVCRT.sprintf   
00DF1AFC  ADD ESP,14   
00DF1AFF  XOR EAX,EAX   
00DF1B01  JMP 00DF2AEE   
00DF1B06  JMP SHORT 00DF1B41   
00DF1B08  MOV EAX,DWORD PTR SS:\[EBP+8\]   
00DF1B0B  MOV EAX,DWORD PTR DS:\[EAX\]   
00DF1B0D  MOV DWORD PTR DS:\[EAX\],3   
00DF1B13  CALL DWORD PTR DS:\[DF80D4\]               ; KERNEL32.GetLastError   
00DF1B19  PUSH EAX   
00DF1B1A  PUSH DWORD PTR SS:\[EBP\-1980\]   
00DF1B20  PUSH DWORD PTR SS:\[EBP\-1860\]   
00DF1B26  PUSH 0DFE4EC                             ; ASCII "File "%s", function "%s" (error %d)"   
00DF1B2B  MOV EAX,DWORD PTR SS:\[EBP+8\]   
00DF1B2E  PUSH DWORD PTR DS:\[EAX+4\]   
00DF1B31  CALL DWORD PTR DS:\[DF82C4\]               ; MSVCRT.sprintf   
00DF1B37  ADD ESP,14   
00DF1B3A  XOR EAX,EAX   
00DF1B3C  JMP 00DF2AEE   
00DF1B41  MOV EAX,DWORD PTR SS:\[EBP\-13E0\]   
00DF1B47  CMP EAX,DWORD PTR SS:\[EBP\-1394\]   
00DF1B4D  JNB SHORT 00DF1B6C   
00DF1B4F  MOV EAX,DWORD PTR SS:\[EBP\-13E0\]   
00DF1B55  MOV ECX,DWORD PTR SS:\[EBP\-197C\]   
00DF1B5B  MOV DWORD PTR DS:\[EAX\],ECX               ;在此处设一次断以找到 iat 的位置   
00DF1B5D  MOV EAX,DWORD PTR SS:\[EBP\-13E0\]   
00DF1B63  ADD EAX,4   
00DF1B66  MOV DWORD PTR SS:\[EBP\-13E0\],EAX   
00DF1B6C  JMP 00DF18A7   
00DF1B71  CMP DWORD PTR SS:\[EBP\-150C\],0   
00DF1B78  JNZ 00DF1C08   
00DF1B7E  MOVZX EAX,BYTE PTR SS:\[EBP\-1750\]   
00DF1B85  TEST EAX,EAX   
00DF1B87  JE SHORT 00DF1C08   
00DF1B89  PUSH 0   
00DF1B8B  MOV EAX,DWORD PTR SS:\[EBP\-174C\]   
00DF1B91  SHL EAX,2   
00DF1B94  PUSH EAX   
00DF1B95  MOV EAX,DWORD PTR SS:\[EBP\-14E4\]          ; Armadill.00400000   
00DF1B9B  ADD EAX,DWORD PTR SS:\[EBP\-1754\]   
00DF1BA1  PUSH EAX   
00DF1BA2  CALL 00DF34E5   
00DF1BA7  ADD ESP,0C   
00DF1BAA  MOV EAX,DWORD PTR SS:\[EBP\-174C\]   
00DF1BB0  SHL EAX,2   
00DF1BB3  PUSH EAX   
00DF1BB4  PUSH DWORD PTR SS:\[EBP\-138C\]   
00DF1BBA  MOV EAX,DWORD PTR SS:\[EBP\-14E4\]          ; Armadill.00400000   
00DF1BC0  ADD EAX,DWORD PTR SS:\[EBP\-1754\]   
00DF1BC6  PUSH EAX   
00DF1BC7  CALL 00DF7A54                            ; JMP to MSVCRT.memcpy   
00DF1BCC  ADD ESP,0C   
00DF1BCF  PUSH 1   
00DF1BD1  MOV EAX,DWORD PTR SS:\[EBP\-174C\]   
00DF1BD7  SHL EAX,2   
00DF1BDA  PUSH EAX   
00DF1BDB  MOV EAX,DWORD PTR SS:\[EBP\-14E4\]          ; Armadill.00400000   
00DF1BE1  ADD EAX,DWORD PTR SS:\[EBP\-1754\]   
00DF1BE7  PUSH EAX   
00DF1BE8  CALL 00DF34E5   
00DF1BED  ADD ESP,0C   
00DF1BF0  MOV EAX,DWORD PTR SS:\[EBP\-138C\]   
00DF1BF6  MOV DWORD PTR SS:\[EBP\-2C04\],EAX   
00DF1BFC  PUSH DWORD PTR SS:\[EBP\-2C04\]   
00DF1C02  CALL 00DF7A4E                            ; JMP to MSVCRT.??3@YAXPAX@Z   
00DF1C07  POP ECX                                     
00DF1C08  CMP DWORD PTR SS:\[EBP\-150C\],0   
00DF1C0F  JNZ SHORT 00DF1C3B   
00DF1C11  LEA EAX,DWORD PTR SS:\[EBP\-1758\]   
00DF1C17  PUSH EAX   
00DF1C18  PUSH DWORD PTR SS:\[EBP\-1758\]   
00DF1C1E  MOV EAX,DWORD PTR SS:\[EBP\-174C\]   
00DF1C24  SHL EAX,2   
00DF1C27  PUSH EAX   
00DF1C28  MOV EAX,DWORD PTR SS:\[EBP\-14E4\]          ; Armadill.00400000   
00DF1C2E  ADD EAX,DWORD PTR SS:\[EBP\-1754\]   
00DF1C34  PUSH EAX   
00DF1C35  CALL DWORD PTR DS:\[DF8134\]               ; KERNEL32.VirtualProtect   
00DF1C3B  JMP 00DF14BE   
  
当程序中断在 00DF1B5B 时去掉此处断点，由此时的 eax，可使我们找到 iat 的位置，我这里是   
12B1000 附近。不断 F9，直到 LoadLibraryA 要载入 shell32.dll，这时又对 00DF1B5B 下断，再   
F9，断在 00DF1B5B 后，F8 走过该语句，好了， api 导入完毕！   
用 ImportREC 处理 12B1008～12B1388（size=380）的数据，cut 掉该区块内的无效指针，我们得   
到有序排列的 iat，所有指针当然没有加密！   
FThunk: 00EB1008  NbFunc: 00000009   
1  00EB1008  advapi32.dll  018C  RegDeleteKeyA   
1  00EB100C  advapi32.dll  01A7  RegQueryValueA   
1  00EB1010  advapi32.dll  019E  RegOpenKeyExA   
1  00EB1014  advapi32.dll  01B3  RegSetValueExA   
1  00EB1018  advapi32.dll  018E  RegDeleteValueA   
1  00EB101C  advapi32.dll  0189  RegCreateKeyExA   
1  00EB1020  advapi32.dll  01A8  RegQueryValueExA   
1  00EB1024  advapi32.dll  0185  RegCloseKey   
1  00EB1028  advapi32.dll  0191  RegEnumKeyA   
  
FThunk: 00EB1030  NbFunc: 00000004   
1  00EB1030  comctl32.dll  003F  ImageList\_LoadImage   
1  00EB1034  comctl32.dll  002C  ImageList\_Create   
1  00EB1038  comctl32.dll  0046  ImageList\_ReplaceIcon   
1  00EB103C  comctl32.dll  0011  InitCommonControls   
  
FThunk: 00EB1044  NbFunc: 00000015   
1  00EB1044  gdi32.dll  01D0  SelectPalette   
1  00EB1048  gdi32.dll  002D  CreateDCA   
1  00EB104C  gdi32.dll  0046  CreatePen   
1  00EB1050  gdi32.dll  01B4  RealizePalette   
1  00EB1054  gdi32.dll  01D6  SetBkMode   
1  00EB1058  gdi32.dll  0032  CreateDIBitmap   
1  00EB105C  gdi32.dll  0052  DeleteDC   
1  00EB1060  gdi32.dll  002C  CreateCompatibleDC   
1  00EB1064  gdi32.dll  0013  BitBlt   
1  00EB1068  gdi32.dll  0168  GetStockObject   
1  00EB106C  gdi32.dll  004F  CreateSolidBrush   
1  00EB1070  gdi32.dll  01AE  Polygon   
1  00EB1074  gdi32.dll  0192  MoveToEx   
1  00EB1078  gdi32.dll  018E  LineTo   
1  00EB107C  gdi32.dll  0039  CreateFontIndirectA   
1  00EB1080  gdi32.dll  01CF  SelectObject   
1  00EB1084  gdi32.dll  01D5  SetBkColor   
1  00EB1088  gdi32.dll  01FB  SetTextColor   
1  00EB108C  gdi32.dll  0044  CreatePalette   
1  00EB1090  gdi32.dll  0055  DeleteObject   
1  00EB1094  gdi32.dll  017F  GetTextMetricsA   
  
FThunk: 00EB109C  NbFunc: 0000005A   
1  00EB109C  kernel32.dll  0303  WinExec   
1  00EB10A0  kernel32.dll  0165  GetShortPathNameA   
1  00EB10A4  kernel32.dll  025C  SearchPathA   
1  00EB10A8  kernel32.dll  005D  DeleteFileA   
1  00EB10AC  kernel32.dll  017F  GetTempPathA   
1  00EB10B0  kernel32.dll  014D  GetPrivateProfileSectionNamesA   
1  00EB10B4  kernel32.dll  0150  GetPrivateProfileStringA   
1  00EB10B8  kernel32.dll  0314  WritePrivateProfileStringA   
1  00EB10BC  kernel32.dll  0296  SetFilePointer   
1  00EB10C0  kernel32.dll  023E  ReadFile   
1  00EB10C4  kernel32.dll  0038  CreateFileA   
1  00EB10C8  kernel32.dll  00A0  FindClose   
1  00EB10CC  kernel32.dll  00A4  FindFirstFileA   
1  00EB10D0  kernel32.dll  002C  CopyFileA   
1  00EB10D4  kernel32.dll  030F  WriteFile   
1  00EB10D8  kernel32.dll  01FC  MoveFileA   
1  00EB10DC  kernel32.dll  0130  GetLocalTime   
1  00EB10E0  kernel32.dll  0126  GetFileSize   
1  00EB10E4  kernel32.dll  004E  CreateThread   
1  00EB10E8  kernel32.dll  013B  GetModuleHandleA   
1  00EB10EC  kernel32.dll  008E  ExitThread   
1  00EB10F0  kernel32.dll  0108  GetCurrentDirectoryA   
1  00EB10F4  kernel32.dll  02DF  UnmapViewOfFile   
1  00EB10F8  kernel32.dll  01F6  MapViewOfFile   
1  00EB10FC  kernel32.dll  0039  CreateFileMappingA   
1  00EB1100  kernel32.dll  01AF  GlobalUnlock   
1  00EB1104  kernel32.dll  01A8  GlobalLock   
1  00EB1108  kernel32.dll  0043  CreateMutexA   
1  00EB110C  kernel32.dll  028E  SetEnvironmentVariableA   
1  00EB1110  kernel32.dll  0187  GetTickCount   
1  00EB1114  kernel32.dll  008D  ExitProcess   
1  00EB1118  kernel32.dll  01DF  LeaveCriticalSection   
1  00EB111C  kernel32.dll  0070  EnterCriticalSection   
1  00EB1120  kernel32.dll  01C6  InitializeCriticalSection   
1  00EB1124  kernel32.dll  005B  DeleteCriticalSection   
1  00EB1128  kernel32.dll  01F3  LockResource   
1  00EB112C  kernel32.dll  01E5  LoadResource   
1  00EB1130  kernel32.dll  02C3  SizeofResource   
1  00EB1134  kernel32.dll  00B3  FindResourceA   
1  00EB1138  kernel32.dll  017D  GetTempFileNameA   
1  00EB113C  kernel32.dll  01DD  LCMapStringA   
1  00EB1140  kernel32.dll  0203  MultiByteToWideChar   
1  00EB1144  kernel32.dll  0302  WideCharToMultiByte   
1  00EB1148  kernel32.dll  01BF  HeapSize   
1  00EB114C  kernel32.dll  010A  GetCurrentProcess   
1  00EB1150  kernel32.dll  02CC  TerminateProcess   
1  00EB1154  kernel32.dll  01BE  HeapReAlloc   
1  00EB1158  kernel32.dll  01BB  HeapFree   
1  00EB115C  kernel32.dll  01B5  HeapAlloc   
1  00EB1160  kernel32.dll  018F  GetVersion   
1  00EB1164  kernel32.dll  00DB  GetCommandLineA   
1  00EB1168  kernel32.dll  0167  GetStartupInfoA   
1  00EB116C  kernel32.dll  0258  RtlUnwind   
1  00EB1170  kernel32.dll  0175  GetSystemTime   
1  00EB1174  kernel32.dll  018A  GetTimeZoneInformation   
1  00EB1178  kernel32.dll  0048  CreateProcessA   
1  00EB117C  kernel32.dll  012E  GetLastError   
1  00EB1180  kernel32.dll  0255  ResumeThread   
1  00EB1184  kernel32.dll  001F  CloseHandle   
1  00EB1188  kernel32.dll  02C4  Sleep   
1  00EB118C  kernel32.dll  011D  GetEnvironmentVariableA   
1  00EB1190  kernel32.dll  01E0  LoadLibraryA   
1  00EB1194  kernel32.dll  0154  GetProcAddress   
1  00EB1198  kernel32.dll  0139  GetModuleFileNameA   
1  00EB119C  kernel32.dll  012A  GetFullPathNameA   
1  00EB11A0  kernel32.dll  01DE  LCMapStringW   
1  00EB11A4  kernel32.dll  02DC  UnhandledExceptionFilter   
1  00EB11A8  kernel32.dll  00C2  FreeEnvironmentStringsA   
1  00EB11AC  kernel32.dll  00C3  FreeEnvironmentStringsW   
1  00EB11B0  kernel32.dll  011A  GetEnvironmentStrings   
1  00EB11B4  kernel32.dll  011C  GetEnvironmentStringsW   
1  00EB11B8  kernel32.dll  01F3  LockResource   
1  00EB11BC  kernel32.dll  0169  GetStdHandle   
1  00EB11C0  kernel32.dll  0129  GetFileType   
1  00EB11C4  kernel32.dll  0190  GetVersionExA   
1  00EB11C8  kernel32.dll  01B9  HeapDestroy   
1  00EB11CC  kernel32.dll  01B7  HeapCreate   
1  00EB11D0  kernel32.dll  02F2  VirtualFree   
1  00EB11D4  kernel32.dll  02EF  VirtualAlloc   
1  00EB11D8  kernel32.dll  016A  GetStringTypeA   
1  00EB11DC  kernel32.dll  016D  GetStringTypeW   
1  00EB11E0  kernel32.dll  02A9  SetStdHandle   
1  00EB11E4  kernel32.dll  00BA  FlushFileBuffers   
1  00EB11E8  kernel32.dll  00D0  GetCPInfo   
1  00EB11EC  kernel32.dll  00CA  GetACP   
1  00EB11F0  kernel32.dll  0147  GetOEMCP   
1  00EB11F4  kernel32.dll  028D  SetEndOfFile   
1  00EB11F8  kernel32.dll  0025  CompareStringA   
1  00EB11FC  kernel32.dll  010D  GetCurrentThreadId   
1  00EB1200  kernel32.dll  0026  CompareStringW   
  
FThunk: 00EB1208  NbFunc: 00000001   
1  00EB1208  shell32.dll  016F  ShellExecuteA   
  
FThunk: 00EB1210  NbFunc: 0000004C   
1  00EB1210  user32.dll  01D9  OpenClipboard   
1  00EB1214  user32.dll  018B  IsClipboardFormatAvailable   
1  00EB1218  user32.dll  014A  GetSystemMetrics   
1  00EB121C  user32.dll  019A  KillTimer   
1  00EB1220  user32.dll  01E2  PeekMessageA   
1  00EB1224  user32.dll  0261  SetWindowPos   
1  00EB1228  user32.dll  01BF  MapWindowPoints   
1  00EB122C  user32.dll  0161  GetWindowRect   
1  00EB1230  user32.dll  01CF  MoveWindow   
1  00EB1234  user32.dll  003C  ClientToScreen   
1  00EB1238  user32.dll  0051  CreateDialogParamA   
1  00EB123C  user32.dll  018C  IsDialogMessage   
1  00EB1240  user32.dll  018A  IsChild   
1  00EB1244  user32.dll  0116  GetKeyState   
1  00EB1248  user32.dll  010B  GetFocus   
1  00EB124C  user32.dll  0197  IsWindowVisible   
1  00EB1250  user32.dll  0157  GetWindow   
1  00EB1254  user32.dll  010C  GetForegroundWindow   
1  00EB1258  user32.dll  020F  ScreenToClient   
1  00EB125C  user32.dll  02B1  WindowFromPoint   
1  00EB1260  user32.dll  0100  GetCursorPos   
1  00EB1264  user32.dll  0208  ReleaseDC   
1  00EB1268  user32.dll  0101  GetDC   
1  00EB126C  user32.dll  015B  GetWindowLongA   
1  00EB1270  user32.dll  0194  IsWindow   
1  00EB1274  user32.dll  01C3  MessageBeep   
1  00EB1278  user32.dll  013E  GetPropA   
1  00EB127C  user32.dll  0139  GetParent   
1  00EB1280  user32.dll  00F6  GetClipboardData   
1  00EB1284  user32.dll  0017  CallWindowProcA   
1  00EB1288  user32.dll  0248  SetPropA   
1  00EB128C  user32.dll  025E  SetWindowLongA   
1  00EB1290  user32.dll  0195  IsWindowEnabled   
1  00EB1294  user32.dll  0163  GetWindowTextA   
1  00EB1298  user32.dll  00BA  EnableWindow   
1  00EB129C  user32.dll  0234  SetFocus   
1  00EB12A0  user32.dll  0096  DialogBoxParamA   
1  00EB12A4  user32.dll  00BC  EndDialog   
1  00EB12A8  user32.dll  00E7  GetAsyncKeyState   
1  00EB12AC  user32.dll  0106  GetDlgItem   
1  00EB12B0  user32.dll  0258  SetTimer   
1  00EB12B4  user32.dll  01A5  LoadImageA   
1  00EB12B8  user32.dll  0219  SendMessageA   
1  00EB12BC  user32.dll  0059  CreateMenu   
1  00EB12C0  user32.dll  0146  GetSubMenu   
1  00EB12C4  user32.dll  008A  DeleteMenu   
1  00EB12C8  user32.dll  0008  AppendMenuA   
1  00EB12CC  user32.dll  019D  LoadBitmapA   
1  00EB12D0  user32.dll  00F4  GetClientRect   
1  00EB12D4  user32.dll  00D7  FillRect   
1  00EB12D8  user32.dll  0147  GetSysColor   
1  00EB12DC  user32.dll  00B2  DrawTextA   
1  00EB12E0  user32.dll  00BE  EndPaint   
1  00EB12E4  user32.dll  0160  GetWindowPlacement   
1  00EB12E8  user32.dll  0198  IsZoomed   
1  00EB12EC  user32.dll  01E6  PostQuitMessage   
1  00EB12F0  user32.dll  0091  DestroyWindow   
1  00EB12F4  user32.dll  0087  DefWindowProcA   
1  00EB12F8  user32.dll  01E4  PostMessageA   
1  00EB12FC  user32.dll  003E  CloseClipboard   
1  00EB1300  user32.dll  01A3  LoadIconA   
1  00EB1304  user32.dll  019F  LoadCursorA   
1  00EB1308  user32.dll  01F7  RegisterClassA   
1  00EB130C  user32.dll  005B  CreateWindowExA   
1  00EB1310  user32.dll  0277  SystemParametersInfoA   
1  00EB1314  user32.dll  01C4  MessageBoxA   
1  00EB1318  user32.dll  0270  ShowWindow   
1  00EB131C  user32.dll  0297  UpdateWindow   
1  00EB1320  user32.dll  019B  LoadAcceleratorsA   
1  00EB1324  user32.dll  012E  GetMessageA   
1  00EB1328  user32.dll  0284  TranslateAccelerator   
1  00EB132C  user32.dll  0288  TranslateMessage   
1  00EB1330  user32.dll  0098  DispatchMessageA   
1  00EB1334  user32.dll  01F6  RedrawWindow   
1  00EB1338  user32.dll  0264  SetWindowTextA   
1  00EB133C  user32.dll  000D  BeginPaint   
  
FThunk: 00EB1344  NbFunc: 0000000E   
1  00EB1344  wsock32.dll  0074  WSACleanup   
1  00EB1348  wsock32.dll  0009  htons   
1  00EB134C  wsock32.dll  0034  gethostbyname   
1  00EB1350  wsock32.dll  0013  send   
1  00EB1354  wsock32.dll  0004  connect   
1  00EB1358  wsock32.dll  0016  shutdown   
1  00EB135C  wsock32.dll  0003  closesocket   
1  00EB1360  wsock32.dll  0008  htonl   
1  00EB1364  wsock32.dll  0073  WSAStartup   
1  00EB1368  wsock32.dll  0017  socket   
1  00EB136C  wsock32.dll  0002  bind   
1  00EB1370  wsock32.dll  0011  recvfrom   
1  00EB1374  wsock32.dll  0010  recv   
1  00EB1378  wsock32.dll  006F  WSAGetLastError   
  
FThunk: 00EB1380  NbFunc: 00000002   
1  00EB1380  comdlg32.dll  0070  GetSaveFileNameA   
1  00EB1384  comdlg32.dll  006E  GetOpenFileNameA   
  
  
  
下面我们再看看 arma 是怎样进一步处理这个表，使得我们因“ api 在 iat 中不是连续的...”   
而苦恼。这时你只要继续 F8，很快到达这里（此前 iat 中数据没有变化）：   
  
00DF1CE2  MOV EAX,DWORD PTR SS:\[EBP\-1A94\]   
00DF1CE8  INC EAX   
00DF1CE9  MOV DWORD PTR SS:\[EBP\-1A94\],EAX   
00DF1CEF  MOV EAX,DWORD PTR SS:\[EBP\-1A94\]   
00DF1CF5  CMP EAX,DWORD PTR SS:\[EBP\-1A8C\]   
00DF1CFB  JNB 00DF1D9B   
00DF1D01  PUSH 1DF5E0D   
00DF1D06  PUSH DWORD PTR SS:\[EBP\-1A90\]   
00DF1D0C  LEA ECX,DWORD PTR SS:\[EBP\-1A90\]   
00DF1D12  CALL 00DD1071                         ;被调函数见后面   
00DF1D17  INC EAX   
00DF1D18  XOR EDX,EDX   
00DF1D1A  MOV ECX,5F5E100   
00DF1D1F  DIV ECX   
00DF1D21  MOV DWORD PTR SS:\[EBP\-1A90\],EDX   
00DF1D27  MOV EAX,DWORD PTR SS:\[EBP\-1A90\]   
00DF1D2D  XOR EDX,EDX   
00DF1D2F  MOV ECX,2710   
00DF1D34  DIV ECX   
00DF1D36  IMUL EAX,DWORD PTR SS:\[EBP\-13B0\]   
00DF1D3D  XOR EDX,EDX   
00DF1D3F  MOV ECX,2710   
00DF1D44  DIV ECX   
00DF1D46  MOV DWORD PTR SS:\[EBP\-1A9C\],EAX   
00DF1D4C  MOV EAX,DWORD PTR SS:\[EBP\-150C\]   
00DF1D52  MOV EAX,DWORD PTR DS:\[EAX\]   
00DF1D54  MOV DWORD PTR SS:\[EBP\-1A98\],EAX   
00DF1D5A  MOV EAX,DWORD PTR SS:\[EBP\-1A9C\]   
00DF1D60  LEA EAX,DWORD PTR DS:\[EAX\*4+4\]   
00DF1D67  PUSH EAX   
00DF1D68  MOV EAX,DWORD PTR SS:\[EBP\-150C\]   
00DF1D6E  ADD EAX,4   
00DF1D71  PUSH EAX   
00DF1D72  PUSH DWORD PTR SS:\[EBP\-150C\]   
00DF1D78  CALL DWORD PTR DS:\[DF82D8\]               ; MSVCRT.memmove   
00DF1D7E  ADD ESP,0C   
00DF1D81  MOV EAX,DWORD PTR SS:\[EBP\-1A9C\]   
00DF1D87  MOV ECX,DWORD PTR SS:\[EBP\-150C\]   
00DF1D8D  MOV EDX,DWORD PTR SS:\[EBP\-1A98\]   
00DF1D93  MOV DWORD PTR DS:\[ECX+EAX\*4\],EDX   
00DF1D96  JMP 00DF1CE2                             ;循环   
00DF1D9B  PUSH DWORD PTR SS:\[EBP\-13AC\]             ; iat 变换完后从 0DF1CFB 跳到这里   
  
这就是 arma 对 iat“变换”的完整代码，这段代码中除 MSVCRT.memmove 外只在 0DF1D12 调用了下面   
这个函数，显然此时 arma 并未导入其它 api 函数。   
00DD1071  PUSH EBP   
00DD1072  MOV EBP,ESP   
00DD1074  PUSH ECX   
00DD1075  MOV EAX,DWORD PTR SS:\[EBP+8\]   
00DD1078  PUSH EBX   
00DD1079  MOV ECX,2710   
00DD107E  PUSH ESI   
00DD107F  CDQ   
00DD1080  MOV ESI,ECX   
00DD1082  PUSH EDI   
00DD1083  IDIV ESI   
00DD1085  MOV EAX,DWORD PTR SS:\[EBP+C\]   
00DD1088  MOV EDI,ECX   
00DD108A  MOV EBX,ECX   
00DD108C  MOV DWORD PTR SS:\[EBP\-4\],EDX   
00DD108F  CDQ   
00DD1090  IDIV ESI   
00DD1092  MOV EAX,DWORD PTR SS:\[EBP+8\]   
00DD1095  MOV ESI,EDX   
00DD1097  CDQ   
00DD1098  IDIV EDI   
00DD109A  MOV EDI,EAX   
00DD109C  MOV EAX,DWORD PTR SS:\[EBP+C\]   
00DD109F  CDQ   
00DD10A0  IMUL EDI,ESI   
00DD10A3  IDIV EBX   
00DD10A5  IMUL ESI,DWORD PTR SS:\[EBP\-4\]   
00DD10A9  XOR EDX,EDX   
00DD10AB  IMUL EAX,DWORD PTR SS:\[EBP\-4\]   
00DD10AF  ADD EAX,EDI   
00DD10B1  POP EDI   
00DD10B2  DIV ECX   
00DD10B4  MOV ECX,5F5E100   
00DD10B9  MOV EAX,EDX   
00DD10BB  XOR EDX,EDX   
00DD10BD  IMUL EAX,EAX,2710   
00DD10C3  ADD EAX,ESI   
00DD10C5  POP ESI   
00DD10C6  DIV ECX   
00DD10C8  POP EBX   
00DD10C9  MOV EAX,EDX   
00DD10CB  LEAVE   
00DD10CC  RETN 8   
  
当程序停在 0DF1D9B 时再次用 ImportREC 处理 12B1008 开始的数据，会看到 size 此时要设为 580，区块   
内出现了不少无效指针，仔细选中这些无效指针，cut 掉后，我们发现得到了与 tDasm 得到的数据   
完全一致，当然是“api 在 iat 中不是连续的...”，呵呵，这是 arma 精心设计的又一个加密 iat 而已！！ 

<tr\> | **jingulong 写道:** |tr> <tr\> | iat 修复不难，3.6 主程序中类似下面的语句比较麻烦：   
00401C8F   8B3D 94AC4400    MOV EDI,DWORD PTR DS:\[<&kernel32.GetProcAddress>\]  ; KERNEL32.GetProcAddress   
00401C95  -E9 04F15A02      JMP 029B0D9E   
00401C9A   66:87F9          XCHG CX,DI   
00401C9D   66:87F9          XCHG CX,DI   
00401CA0   FFD7             CALL EDI   
00401CA2  -E9 13F15A02      JMP 029B0DBA   
  
这里 jmp 到的地方只有程序运行起来才能得到，而这属于子进程！ |tr> 

   
  
这个很好解决，而且偶已经解决，把该段内存 sotlen code 代码全部 dump 出来，修复时再建立一内存映像到 2450000(偶 dump 时都 jmp 245xxxx 处执行)，把该段 stolen code 全部复制过去，OK!   
这是 dump 出来的部分代码：   
:00000000 53                      push ebx   
:00000001 52                      push edx   
:00000002 6687CB                  xchg bx, cx   
:00000005 93                      xchg eax,ebx   
:00000006 7A02                    jpe 0000000A   
:00000008 7A17                    jpe 00000021   
:0000000A 6687D3                  xchg bx, dx   
:0000000D 8BC9                    mov ecx, ecx   
:0000000F 6687D3                  xchg bx, dx   
:00000012 93                      xchg eax,ebx   
:00000013 6687CB                  xchg bx, cx   
:00000016 90                      nop   
:00000017 5A                      pop edx   
:00000018 5B                      pop ebx   
:00000019 56                      push esi   
:0000001A 6848100000              push 00001048   
:0000001F E9E10FFBFD              jmp FDFB1005    //回到程序，此处其实是 jmp 00401005,因为所处地址不一样变形了，下同。    
:00000024 FF742414                push \[esp+14\]   
:00000028 8BF0                    mov esi, eax   
:0000002A FF742414                push \[esp+14\]   
:0000002E F7D1                    not ecx   
:00000030 87D3                    xchg ebx, edx   
:00000032 87D3                    xchg ebx, edx   
:00000034 F7D1                    not ecx   
:00000036 FF742414                push \[esp+14\]   
:0000003A 56                      push esi   
:0000003B E9D00FFBFD              jmp FDFB1010   
:00000040 8B4508                  mov eax, dword ptr \[ebp+08\]   
:00000043 53                      push ebx   
:00000044 56                      push esi   
:00000045 57                      push edi   
:00000046 8D9800100000            lea ebx, dword ptr \[eax+00001000\]   
:0000004C 6A12                    push 00000012   
:0000004E 59                      pop ecx   
:0000004F 8D75AC                  lea esi, dword ptr \[ebp\-54\]   
:00000052 8BFB                    mov edi, ebx   
:00000054 C745AC886A3F24          mov \[ebp\-54\], 243F6A88   
:0000005B C745B0D308A385          mov \[ebp\-50\], 85A308D3   
:00000062 C745B42E8A1913          mov \[ebp\-4C\], 13198A2E   
:00000069 C745B844737003          mov \[ebp\-48\], 03707344   
:00000070 C745BC223809A4          mov \[ebp\-44\], A4093822   
:00000077 C745C0D0319F29          mov \[ebp\-40\], 299F31D0   
:0000007E C745C498FA2E08          mov \[ebp\-3C\], 082EFA98   
:00000085 C745C8896C4EEC          mov \[ebp\-38\], EC4E6C89   
:0000008C C745CCE6212845          mov \[ebp\-34\], 452821E6   
:00000093 C745D07713D038          mov \[ebp\-30\], 38D01377   
:0000009A C745D4CF6654BE          mov \[ebp\-2C\], BE5466CF   
:000000A1 C745D86C0CE934          mov \[ebp\-28\], 34E90C6C   
:000000A8 C745DCB729ACC0          mov \[ebp\-24\], C0AC29B7   
:000000AF C745E0DD507CC9          mov \[ebp\-20\], C97C50DD   
:000000B6 96                      xchg eax,esi   
:000000B7 96                      xchg eax,esi   
:000000B8 6687DB                  xchg bx, bx   
:000000BB C745E4B5D5843F          mov \[ebp\-1C\], 3F84D5B5   
:000000C2 C745E8170947B5          mov \[ebp\-18\], B5470917   
:000000C9 C745ECD9D51692          mov \[ebp\-14\], 9216D5D9   
:000000D0 C745F01BFB7989          mov \[ebp\-10\], 8979FB1B   
:000000D7 C745F804000000          mov \[ebp\-08\], 00000004   
:000000DE E94E0FFBFD              jmp FDFB1031   
:000000E3 8BD2                    mov edx, edx   
:000000E5 8B4D14                  mov ecx, dword ptr \[ebp+14\]   
:000000E8 894DF4                  mov dword ptr \[ebp\-0C\], ecx   
:000000EB E9DC0FFBFD              jmp FDFB10CC   
:000000F0 87D3                    xchg ebx, edx   
:000000F2 0FCF                    bswap edi   
:000000F4 7C00                    jl 000000F6   
:000000F6 7800                    js 000000F8   
:000000F8 0FCF                    bswap edi   
:000000FA 87D3                    xchg ebx, edx   
:000000FC 6687FF                  xchg di, di   
:000000FF BF00010000              mov edi, 00000100   
:00000104 E9CB0FFBFD              jmp FDFB10D4   
:00000109 96                      xchg eax,esi   
:0000010A F7D0                    not eax   
:0000010C 87D1                    xchg ecx, edx   
:0000010E 90                      nop   
:0000010F 87D1                    xchg ecx, edx   
:00000111 F7D0                    not eax   
:00000113 90                      nop   
:00000114 96                      xchg eax,esi   
:00000115 7200                    jb 00000117   
:00000117 6A09                    push 00000009   
:00000119 8DB304100000            lea esi, dword ptr \[ebx+00001004\]   
:0000011F 5F                      pop edi   
:00000120 E90110FBFD              jmp FDFB1126  
  
 4dd466d0e1e0cc123afbe504da2462ca