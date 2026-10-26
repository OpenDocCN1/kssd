

```
00DF1BFC  PUSH DWORD PTR SS:[EBP-2C04]
00DF1C02  CALL 00DF7A4E                            ; JMP to MSVCRT.??3@YAXPAX@Z
00DF1C07  POP ECX
00DF1C08  CMP DWORD PTR SS:[EBP-150C],0
00DF1C0F  JNZ SHORT 00DF1C3B
00DF1C11  LEA EAX,DWORD PTR SS:[EBP-1758]
00DF1C17  PUSH EAX
00DF1C18  PUSH DWORD PTR SS:[EBP-1758]
00DF1C1E  MOV EAX,DWORD PTR SS:[EBP-174C]
00DF1C24  SHL EAX,2
00DF1C27  PUSH EAX
00DF1C28  MOV EAX,DWORD PTR SS:[EBP-14E4]          ; Armadill.00400000
00DF1C2E  ADD EAX,DWORD PTR SS:[EBP-1754]
00DF1C34  PUSH EAX
00DF1C35  CALL DWORD PTR DS:[DF8134]               ; KERNEL32.VirtualProtect
00DF1C3B  JMP 00DF14BE
```

当程序中断在`00DF1B5B`时去掉此处断点，由此时的`eax`，可使我们找到`iat`的位置，我这里是`12B1000`附近。不断`F9`，直到`LoadLibraryA`要载入`shell32.dll`，这时又对`00DF1B5B`下断，再`F9`，断在`00DF1B5B`后，`F8`走过该语句，好了，`api`导入完毕！

用`ImportREC`处理`12B1008`～`12B1388`（`size=380`）的数据，cut掉该区块内的无效指针，我们得到有序排列的`iat`，所有指针当然没有加密！

```
FThunk: 00EB1008  NbFunc: 00000009
1  00EB1008  advapi32.dll  018C  RegDeleteKeyA
1  00EB100C  advapi32.dll  01A7  RegQueryValueA
1  00EB1010  advapi32.dll  019E  RegOpenKeyExA
1  00EB1014  advapi32.dll  01B3  RegSetValueExA
1  00EB1018  advapi32.dll  018E  RegDeleteValueA
1  00EB101C  advapi32.dll  0189  RegCreateKeyExA
1  00EB1020  advapi32.dll  01A8  RegQueryValueExA
1  00EB1024  advapi32.dll  0185  RegCloseKey
1  00EB1028  advapi32.dll  0191  RegEnumKeyA
FThunk: 00EB1030  NbFunc: 00000004
1  00EB1030  comctl32.dll  003F  ImageList_LoadImage
1  00EB1034  comctl32.dll  002C  ImageList_Create
1  00EB1038  comctl32.dll  0046  ImageList_ReplaceIcon
1  00EB103C  comctl32.dll  0011  InitCommonControls
FThunk: 00EB1044  NbFunc: 00000015
1  00EB1044  gdi32.dll  01D0  SelectPalette
1  00EB1048  gdi32.dll  002D  CreateDCA
1  00EB104C  gdi32.dll  0046  CreatePen
1  00EB1050  gdi32.dll  01B4  RealizePalette
1  00EB1054  gdi32.dll  01D6  SetBkMode
1  00EB1058  gdi32.dll  0032  CreateDIBitmap
1  00EB105C  gdi32.dll  0052  DeleteDC
1  00EB1060  gdi32.dll  002C  CreateCompatibleDC
1  00EB1064  gdi32.dll  0013  BitBlt
1  00EB1068  gdi32.dll  0168  GetStockObject
1  00EB106C  gdi32.dll  004F  CreateSolidBrush
1  00EB1070  gdi32.dll  01AE  Polygon
1  00EB1074  gdi32.dll  0192  MoveToEx
1  00EB1078  gdi32.dll  018E  LineTo
1  00EB107C  gdi32.dll  0039  CreateFontIndirectA
1  00EB1080  gdi32.dll  01CF  SelectObject
1  00EB1084  gdi32.dll  01D5  SetBkColor
1  00EB1088  gdi32.dll  01FB  SetTextColor
1  00EB108C  gdi32.dll  0044  CreatePalette
1  00EB1090  gdi32.dll  0055  DeleteObject
1  00EB1094  gdi32.dll  017F  GetTextMetricsA
FThunk: 00EB109C  NbFunc: 0000005A
1  00EB109C  kernel32.dll  0303  WinExec
1  00EB10A0  kernel32.dll  0165  GetShortPathNameA
1  00EB10A4  kernel32.dll  025C  SearchPathA
1  00EB10A8  kernel32.dll  005D  DeleteFileA
1  00EB10AC  kernel32.dll  017F  GetTempPathA
1  00EB10B0  kernel32.dll  014D  GetPrivateProfileSectionNamesA
1  00EB10B4  kernel32.dll  0150  GetPrivateProfileStringA
1  00EB10B8  kernel32.dll  0314  WritePrivateProfileStringA
1  00EB10BC  kernel32.dll  0296  SetFilePointer
1  00EB10C0  kernel32.dll  023E  ReadFile
1  00EB10C4  kernel32.dll  0038  CreateFileA
1  00EB10C8  kernel32.dll  00A0  FindClose
1  00EB10CC  kernel32.dll  00A4  FindFirstFileA
1  00EB10D0  kernel32.dll  002C  CopyFileA
1  00EB10D4  kernel32.dll  030F  WriteFile
1  00EB10D8  kernel32.dll  01FC  MoveFileA
1  00EB10DC  kernel32.dll  0130  GetLocalTime
1  00EB10E0  kernel32.dll  0126  GetFileSize
1  00EB10E4  kernel32.dll  004E  CreateThread
1  00EB10E8  kernel32.dll  013B  GetModuleHandleA
1  00EB10EC  kernel32.dll  008E  ExitThread
1  00EB10F0  kernel32.dll  0108  GetCurrentDirectoryA
1  00EB10F4  kernel32.dll  02DF  UnmapViewOfFile
1  00EB10F8  kernel32.dll  01F6  MapViewOfFile
1  00EB10FC  kernel32.dll  0039  CreateFileMappingA
1  00EB1100  kernel32.dll  01AF  GlobalUnlock
1  00EB1104  kernel32.dll  01A8  GlobalLock
1  00EB1108  kernel32.dll  0043  CreateMutexA
1  00EB110C  kernel32.dll  028E  SetEnvironmentVariableA
1  00EB1110  kernel32.dll  0187  GetTickCount
1  00EB1114  kernel32.dll  008D  ExitProcess
1  00EB1118  kernel32.dll  01DF  LeaveCriticalSection
1  00EB111C  kernel32.dll  0070  EnterCriticalSection
1  00EB1120  kernel32.dll  01C6  InitializeCriticalSection
1  00EB1124  kernel32.dll  005B  DeleteCriticalSection
1  00EB1128  kernel32.dll  01F3  LockResource
1  00EB112C  kernel32.dll  01E5  LoadResource
1  00EB1130  kernel32.dll  02C3  SizeofResource
1  00EB1134  kernel32.dll  00B3  FindResourceA
1  00EB1138  kernel32.dll  017D  GetTempFileNameA
1  00EB113C  kernel32.dll  01DD  LCMapStringA
1  00EB1140  kernel32.dll  0203  MultiByteToWideChar
1  00EB1144  kernel32.dll  0302  WideCharToMultiByte
1  00EB1148  kernel32.dll  01BF  HeapSize
1  00EB114C  kernel32.dll  010A  GetCurrentProcess
1  00EB1150  kernel32.dll  02CC  TerminateProcess
1  00EB1154  kernel32.dll  01BE  HeapReAlloc
1  00EB1158  kernel32.dll  01BB  HeapFree
1  00EB115C  kernel32.dll  01B5  HeapAlloc
1  00EB1160  kernel32.dll  018F  GetVersion
1  00EB1164  kernel32.dll  00DB  GetCommandLineA
1  00EB1168  kernel32.dll  0167  GetStartupInfoA
1  00EB116C  kernel32.dll  0258  RtlUnwind
1  00EB1170  kernel32.dll  0175  GetSystemTime
1  00EB1174  kernel32.dll  018A  GetTimeZoneInformation
1  00EB1178  kernel32.dll  0048  CreateProcessA
1  00EB117C  kernel32.dll  012E  GetLastError
1  00EB1180  kernel32.dll  0255  ResumeThread
1  00EB1184  kernel32.dll  001F  CloseHandle
1  00EB1188  kernel32.dll  02C4  Sleep
1  00EB118C  kernel32.dll  011D  GetEnvironmentVariableA
1  00EB1190  kernel32.dll  01E0  LoadLibraryA
1  00EB1194  kernel32.dll  0154  GetProcAddress
1  00EB1198  kernel32.dll  0139  GetModuleFileNameA
1  00EB119C  kernel32.dll  012A  GetFullPathNameA
1  00EB11A0  kernel32.dll  01DE  LCMapStringW
1  00EB11A4  kernel32.dll  02DC  UnhandledExceptionFilter
1  00EB11A8  kernel32.dll  00C2  FreeEnvironmentStringsA
1  00EB11AC  kernel32.dll  00C3  FreeEnvironmentStringsW
1  00EB11B0  kernel32.dll  011A  GetEnvironmentStrings
1  00EB11B4  kernel32.dll  011C  GetEnvironmentStringsW
1  00EB11B8  kernel32.dll  01F3  LockResource
1  00EB11BC  kernel32.dll  0169  GetStdHandle
1  00EB11C0  kernel32.dll  0129  GetFileType
1  00EB11C4  kernel32.dll  0190  GetVersionExA
1  00EB11C8  kernel32.dll  01B9  HeapDestroy
1  00EB11CC  kernel32.dll  01B7  HeapCreate
1  00EB11D0  kernel32.dll  02F2  VirtualFree
1  00EB11D4  kernel32.dll  02EF  VirtualAlloc
1  00EB11D8  kernel32.dll  016A  GetStringTypeA
1  00EB11DC  kernel32.dll  016D  GetStringTypeW
1  00EB11E0  kernel32.dll  02A9  SetStdHandle
1  00EB11E4  kernel32.dll  00BA  FlushFileBuffers
1  00EB11E8  kernel32.dll  00D0  GetCPInfo
1  00EB11EC  kernel32.dll  00CA  GetACP
1  00EB11F0  kernel32.dll  0147  GetOEMCP
1  00EB11F4  kernel32.dll  028D  SetEndOfFile
1  00EB11F8  kernel32.dll  0025  CompareStringA
1  00EB11FC  kernel32.dll  010D  GetCurrentThreadId
1  00EB1200  kernel32.dll  0026  CompareStringW
FThunk: 00EB1208  NbFunc: 00000001
1  00EB1208  shell32.dll  016F  ShellExecuteA
FThunk: 00EB1210  NbFunc: 0000004C
1  00EB1210  user32.dll  01D9  OpenClipboard
1  00EB1214  user32.dll  018B  IsClipboardFormatAvailable
1  00EB1218  user32.dll  014A  GetSystemMetrics
1  00EB121C  user32.dll  019A  KillTimer
1  00EB1220  user32.dll  01E2  PeekMessageA
1  00EB1224  user32.dll  0261  SetWindowPos
1  00EB1228  user32.dll  01BF  MapWindowPoints
1  00EB122C  user32.dll  0161  GetWindowRect
1  00EB1230  user32.dll  01CF  MoveWindow
1  00EB1234  user32.dll  003C  ClientToScreen
1  00EB1238  user32.dll  0051  CreateDialogParamA
1  00EB123C  user32.dll  018C  IsDialogMessage
1  00EB1240  user32.dll  018A  IsChild
1  00EB1244  user32.dll  0116  GetKeyState
1  00EB1248  user32.dll  010B  GetFocus
1  00EB124C  user32.dll  0197  IsWindowVisible
1  00EB1250  user32.dll  0157  GetWindow
1  00EB1254  user32.dll  010C  GetForegroundWindow
```

