
# 导入表与函数列表

## kernel32.dll

| 索引 | 地址 | 模块 | 序号 | 函数名 |
| :--- | :--- | :--- | :--- | :--- |
| 1 | `0003E118` | `kernel32.dll` | `01FD` | `GlobalUnlock` |
| 1 | `0003E11C` | `kernel32.dll` | `02BD` | `ResetEvent` |
| 1 | `0003E120` | `kernel32.dll` | `0302` | `SetEvent` |
| 1 | `0003E124` | `kernel32.dll` | `034C` | `TlsAlloc` |
| 1 | `0003E128` | `kernel32.dll` | `034D` | `TlsFree` |
| 1 | `0003E12C` | `kernel32.dll` | `034E` | `TlsGetValue` |
| 1 | `0003E130` | `kernel32.dll` | `034F` | `TlsSetValue` |
| 1 | `0003E134` | `kernel32.dll` | `015F` | `GetFileType` |
| 1 | `0003E138` | `kernel32.dll` | `01AF` | `GetStdHandle` |
| 1 | `0003E13C` | `kernel32.dll` | `010A` | `GetCommandLineA` |
| 1 | `0003E140` | `kernel32.dll` | `0358` | `UnhandledExceptionFilter` |
| 1 | `0003E144` | `kernel32.dll` | `00CD` | `FindClose` |
| 1 | `0003E148` | `kernel32.dll` | `00D1` | `FindFirstFileA` |
| 1 | `0003E14C` | `kernel32.dll` | `021E` | `InterlockedIncrement` |
| 1 | `0003E150` | `kernel32.dll` | `0373` | `VirtualQuery` |
| 1 | `0003E154` | `kernel32.dll` | `0080` | `DeleteCriticalSection` |
| 1 | `0003E158` | `kernel32.dll` | `0241` | `LeaveCriticalSection` |
| 1 | `0003E15C` | `kernel32.dll` | `0097` | `EnterCriticalSection` |
| 1 | `0003E160` | `kernel32.dll` | `0216` | `InitializeCriticalSection` |
| 1 | `0003E164` | `kernel32.dll` | `024C` | `LocalFree` |
| 1 | `0003E168` | `kernel32.dll` | `0248` | `LocalAlloc` |
| 1 | `0003E16C` | `kernel32.dll` | `013F` | `GetCurrentThreadId` |
| 1 | `0003E170` | `kernel32.dll` | `021A` | `InterlockedDecrement` |
| 1 | `0003E174` | `kernel32.dll` | `03B0` | `lstrcpyn` |
| 1 | `0003E178` | `kernel32.dll` | `0243` | `LoadLibraryExA` |
| 1 | `0003E17C` | `kernel32.dll` | `037F` | `WideCharToMultiByte` |
| 1 | `0003E180` | `kernel32.dll` | `0265` | `MultiByteToWideChar` |
| 1 | `0003E184` | `kernel32.dll` | `03B3` | `lstrlen` |
| 1 | `0003E188` | `kernel32.dll` | `016C` | `GetLocaleInfoA` |
| 1 | `0003E18C` | `kernel32.dll` | `01CD` | `GetThreadLocale` |
| 1 | `0003E190` | `kernel32.dll` | `0174` | `GetModuleFileNameA` |
| 1 | `0003E194` | `kernel32.dll` | `0169` | `GetLastError` |

`FThunk: 0003E19C` `NbFunc: 00000019`

## oleaut32.dll

| 索引 | 地址 | 模块 | 序号 | 函数名 |
| :--- | :--- | :--- | :--- | :--- |
| 1 | `0003E19C` | `oleaut32.dll` | `0008` | `VariantInit` |
| 1 | `0003E1A0` | `oleaut32.dll` | `0009` | `VariantClear` |
| 1 | `0003E1A4` | `oleaut32.dll` | `007D` | `VarBoolFromStr` |
| 1 | `0003E1A8` | `oleaut32.dll` | `0071` | `VarBstrFromCy` |
| 1 | `0003E1AC` | `oleaut32.dll` | `000B` | `VariantCopyInd` |
| 1 | `0003E1B0` | `oleaut32.dll` | `0006` | `SysFreeString` |
| 1 | `0003E1B4` | `oleaut32.dll` | `000A` | `VariantCopy` |
| 1 | `0003E1B8` | `oleaut32.dll` | `0093` | `VariantChangeTypeEx` |
| 1 | `0003E1BC` | `oleaut32.dll` | `0072` | `VarBstrFromDate` |
| 1 | `0003E1C0` | `oleaut32.dll` | `00AD` | `VarNeg` |
| 1 | `0003E1C4` | `oleaut32.dll` | `00AE` | `VarNot` |
| 1 | `0003E1C8` | `oleaut32.dll` | `0040` | `VarI4FromStr` |
| 1 | `0003E1CC` | `oleaut32.dll` | `0054` | `VarR8FromStr` |
| 1 | `0003E1D0` | `oleaut32.dll` | `005E` | `VarDateFromStr` |
| 1 | `0003E1D4` | `oleaut32.dll` | `0068` | `VarCyFromStr` |
| 1 | `0003E1D8` | `oleaut32.dll` | `0005` | `SysReAllocStringLen` |
| 1 | `0003E1DC` | `oleaut32.dll` | `0014` | `SafeArrayGetLBound` |
| 1 | `0003E1E0` | `oleaut32.dll` | `0013` | `SafeArrayGetUBound` |
| 1 | `0003E1E4` | `oleaut32.dll` | `0074` | `VarBstrFromBool` |
| 1 | `0003E1E8` | `oleaut32.dll` | `000F` | `SafeArrayCreate` |
| 1 | `0003E1EC` | `oleaut32.dll` | `0028` | `SafeArrayRedim` |
| 1 | `0003E1F0` | `oleaut32.dll` | `0094` | `SafeArrayPtrOfIndex` |
| 1 | `0003E1F4` | `oleaut32.dll` | `0019` | `SafeArrayGetElement` |
| 1 | `0003E1F8` | `oleaut32.dll` | `001A` | `SafeArrayPutElement` |
| 1 | `0003E1FC` | `oleaut32.dll` | `0004` | `SysAllocStringLen` |

`FThunk: 0003E204` `NbFunc: 00000004`

## shell32.dll

| 索引 | 地址 | 模块 | 序号 | 函数名 |
| :--- | :--- | :--- | :--- | :--- |
| 1 | `0003E204` | `shell32.dll` | `008A` | `DragAcceptFiles` |
| 1 | `0003E208` | `shell32.dll` | `008B` | `DragFinish` |
| 1 | `0003E20C` | `shell32.dll` | `008C` | `DragQueryFile` |
| 1 | `0003E210` | `shell32.dll` | `0167` | `ShellExecuteA` |

`FThunk: 0003E218` `NbFunc: 00000020`

## user32.dll

| 索引 | 地址 | 模块 | 序号 | 函数名 |
| :--- | :--- | :--- | :--- | :--- |
| 1 | `0003E218` | `user32.dll` | `01DC` | `MessageBeep` |
| 1 | `0003E21C` | `user32.dll` | `0254` | `SetDlgItemTextA` |
| 1 | `0003E220` | `user32.dll` | `0128` | `GetKeyboardType` |
| 1 | `0003E224` | `user32.dll` | `002B` | `CharNextA` |
| 1 | `0003E228` | `user32.dll` | `015E` | `GetSystemMetrics` |
| 1 | `0003E22C` | `user32.dll` | `0056` | `CreateDialogParamA` |
| 1 | `0003E230` | `user32.dll` | `0039` | `CheckDlgButton` |
| 1 | `0003E234` | `user32.dll` | `000E` | `BeginPaint` |
| 1 | `0003E238` | `user32.dll` | `02D9` | `wsprintfA` |
| 1 | `0003E23C` | `user32.dll` | `027B` | `SetTimer` |
| 1 | `0003E240` | `user32.dll` | `0284` | `SetWindowPos` |
| 1 | `0003E244` | `user32.dll` | `0287` | `SetWindowTextA` |
| 1 | `0003E248` | `user32.dll` | `0293` | `ShowWindow` |
| 1 | `0003E24C` | `user32.dll` | `01DD` | `MessageBoxA` |
| 1 | `0003E250` | `user32.dll` | `023C` | `SendMessageA` |
| 1 | `0003E254` | `user32.dll` | `01C9` | `LoadStringA` |
| 1 | `0003E258` | `user32.dll` | `01B6` | `LoadBitmapA` |
| 1 | `0003E25C` | `user32.dll` | `01B3` | `KillTimer` |
| 1 | `0003E260` | `user32.dll` | `0178` | `GetWindowTextA` |
| 1 | `0003E264` | `user32.dll` | `0112` | `GetDlgItem` |
| 1 | `0003E268` | `user32.dll` | `0111` | `GetDlgCtrlID` |
| 1 | `0003E26C` | `user32.dll` | `010D` | `GetDC` |
| 1 | `0003E270` | `user32.dll` | `00E3` | `FillRect` |
| 1 | `0003E274` | `user32.dll` | `00CC` | `EnumChildWindows` |
| 1 | `0003E278` | `user32.dll` | `00C9` | `EndPaint` |
| 1 | `0003E27C` | `user32.dll` | `00C5` | `EnableWindow` |
| 1 | `0003E280` | `user32.dll` | `009F` | `DialogBoxParamA` |
| 1 | `0003E284` | `user32.dll` | `0061` | `CreateWindowExA` |
| 1 | `0003E288` | `user32.dll` | `022B` | `ReleaseDC` |
| 1 | `0003E28C` | `user32.dll` | `0237` | `SendDlgItemMessageA` |
| 1 | `0003E290` | `user32.dll` | `0200` | `PostMessageA` |
| 1 | `0003E294` | `user32.dll` | `0202` | `PostQuitMessage` |

`FThunk: 0003E29C` `NbFunc: 00000001`

## winmm.dll

| 索引 | 地址 | 模块 | 序号 | 函数名 |
| :--- | :--- | :--- | :--- | :--- |
| 1 | `0003E29C` | `winmm.dll` | `0041` | `mciSendCommandA` |

# 3. 寻找 OEP

在第一个段下内存访问断点，会发现壳对原代码的 `JMP` 和 `CALL` 的偏移地址进行修正，继续运行，最后停在 OEP，代码如下：

```asm
00415675    C705 4B0E4>MOV DWORD PTR DS:[440E4B],0           ; OEP
0041567F    6A 00       PUSH 0
00415681    E8 8671000>CALL prot765.0041C80C                 ; 被 HOOK 的 GetModuleHandleA
00415686    A3 FC15440>MOV DWORD PTR DS:[4415FC],EAX
0041568B    E8 AE72000>CALL prot765.0041C93E                 ; JMP to comctl32.InitCommonControls
00415690    8D35 01024>LEA ESI,DWORD PTR DS:[440201]
00415696    8D3D 341B4>LEA EDI,DWORD PTR DS:[441B34]
0041569C    B9 1000000>MOV ECX,10
004156A1    F3:A4       REP MOVSB
004156A3    C705 181B4>MOV DWORD PTR DS:[441B18],-0C
004156AD    C705 281B4>MOV DWORD PTR DS:[441B28],109
004156B7    C605 2F1B4>MOV BYTE PTR DS:[441B2F],1
004156BE    6A 00       PUSH 0
004156C0    68 E156410>PUSH prot765.004156E1
004156C5    6A 00       PUSH 0
004156C7    68 1F00440>PUSH prot765.0044001F                 ; ASCII "IDD_DIALOG1"
004156CC    FF35 FC154>PUSH DWORD PTR DS:[4415FC]
004156D2    E8 D771000>CALL prot765.0041C8AE
004156D7    6A 00       PUSH 0
004156D9    E8 1671000>CALL prot765.0041C7F4
004156DE    33C0        XOR EAX,EAX
004156E0    C3          RET
```

# 4. 修复被 HOOK 的 API

进入 `0041C80C`，发现原本 `JMP DWORD PTR DS:[XXXXXXXX]` 的形式被 HOOK 成 `NOP; CALL 003B2B53`，代码如下：

```asm
<HookAPI>>  50         PUSH EAX
003B2B54    60         PUSHAD
003B2B55    E8 0000000>CALL 003B2B5A
003B2B5A    5D         POP EBP
003B2B5B    81ED 3B164>SUB EBP,41163B
003B2B61    8B7C24 24  MOV EDI,DWORD PTR SS:[ESP+24]
003B2B65    8BB5 A5414>MOV ESI,DWORD PTR SS:[EBP+4141A5]
003B2B6B    8B8D 034B4>MOV ECX,DWORD PTR SS:[EBP+414B03]
003B2B71    8B048E     MOV EAX,DWORD PTR DS:[ESI+ECX*4]     ; ESI 为被 HOOK 的地址表
003B2B74    33D2       XOR EDX,EDX
003B2B76    99         CDQ
003B2B77    25 FFFFFF7>AND EAX,7FFFFFFF
003B2B7C    0385 4D414>ADD EAX,DWORD PTR SS:[EBP+41414D]
003B2B82    3BC7       CMP EAX,EDI                            ; 根据返回地址查表
003B2B84    75 10      JNZ SHORT 003B2B96
003B2B86    8D348E     LEA ESI,DWORD PTR DS:[ESI+ECX*4]
003B2B89    0BD2       OR EDX,EDX
003B2B8B    75 04      JNZ SHORT 003B2B91
003B2B8D    EB 66      JMP SHORT 003B2BF5
003B2B8F    EB 05      JMP SHORT 003B2B96
003B2B91    E9 E101000>JMP 003B2D77                            ; 出口
003B2B96    8B85 A9414>MOV EAX,DWORD PTR SS:[EBP+4141A9]
003B2B9C    C1E8 02    SHR EAX,2
003B2B9F    83E8 02    SUB EAX,2
003B2BA2    894424 F8  MOV DWORD PTR SS:[ESP-8],EAX
003B2BA6    33C0       XOR EAX,EAX
003B2BA8    894424 FC  MOV DWORD PTR SS:[ESP-4],EAX
003B2BAC    8B4C24 FC  MOV ECX,DWORD PTR SS:[ESP-4]
003B2BB0    034C24 F8  ADD ECX,DWORD PTR SS:[ESP-8]
```
