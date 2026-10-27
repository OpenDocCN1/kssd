
# 反汇编分析

## 汇编代码段

```
004FCA93     8B4C24 10            mov ecx,dword ptr ss:[esp+10]

004FCA97     57                   push edi

004FCA98     51                   push ecx

004FCA99     E8 FAB9FFFF          call 004F8498                           ; 004F8498

004FCA9E     8906                 mov dword ptr ds:[esi],eax

004FCAA0     EB 1E                jmp short 004FCAC0                      ; 004FCAC0

004FCAA2     8B5424 10            mov edx,dword ptr ss:[esp+10]

004FCAA6     57                   push edi

004FCAA7     52                   push edx

004FCAA8     E8 EBB9FFFF          call 004F8498                           ; 004F8498

004FCAAD     8906                 mov dword ptr ds:[esi],eax

004FCAAF     8B4C24 14            mov ecx,dword ptr ss:[esp+14]

004FCAB3     85C9                 test ecx,ecx

004FCAB5     74 09                je short 004FCAC0                       ; 004FCAC0

004FCAB7     8901                 mov dword ptr ds:[ecx],eax

004FCAB9     83C1 04              add ecx,4

004FCABC     894C24 14            mov dword ptr ss:[esp+14],ecx

004FCAC0     33C0                 xor eax,eax

004FCAC2     66:8B03              mov ax,word ptr ds:[ebx]

004FCAC5     50                   push eax

004FCAC6     68 FF000000          push 0FF

004FCACB     57                   push edi

004FCACC     E8 5D8DFFFF          call 004F582E                           ; 004F582E

004FCAD1     66:C703 0000         mov word ptr ds:[ebx],0

004FCAD6     8B7C24 10            mov edi,dword ptr ss:[esp+10]

004FCADA     3BEE                 cmp ebp,esi

004FCADC     74 08                je short 004FCAE6                       ; 004FCAE6

004FCADE     6A 04                push 4

004FCAE0     55                   push ebp

004FCAE1     E8 B27FFFFF          call 004F4A98                           ; 004F4A98

004FCAE6     8B5C24 18            mov ebx,dword ptr ss:[esp+18]

004FCAEA     83C5 04              add ebp,4

004FCAED     83C6 04              add esi,4

004FCAF0   ^ E9 98F9FFFF          jmp 004FC48D ///循环处理函数。

004FCAF5     E8 8E7DFFFF          call 004F4888                           ; 004F4888

004FCAFA     8B7424 20            mov esi,dword ptr ss:[esp+20]           ; Project2.004840B4

004FCAFE     8B5424 3C            mov edx,dword ptr ss:[esp+3C]

004FCB02     8B4E 04              mov ecx,dword ptr ds:[esi+4]

004FCB05     51                   push ecx

004FCB06     68 FF000000          push 0FF

004FCB0B     52                   push edx

004FCB0C     E8 1D8DFFFF          call 004F582E                           ; 004F582E

004FCB11     6A 14                push 14

004FCB13     56                   push esi

004FCB14     E8 7F7FFFFF          call 004F4A98                           ; 004F4A98

004FCB19     83C6 14              add esi,14

004FCB1C     8B5C24 14            mov ebx,dword ptr ss:[esp+14]

004FCB20     8B6C24 34            mov ebp,dword ptr ss:[esp+34]

004FCB24     897424 20            mov dword ptr ss:[esp+20],esi

004FCB28   ^ E9 92F7FFFF          jmp 004FC2BF 　///循环处理DLL。

004FCB2D     8B8424 4C040000      mov eax,dword ptr ss:[esp+44C]

004FCB34     85C0                 test eax,eax
```

## 分析说明

从以上分析可以看出，程序对`kernel32.dll`、`user32.dll`、`gdi32.dll`、`advapi32.dll`、`shell32.dll`、`sdprotector.dll`等6个DLL的函数进行加密处理，同时对`MessageBoxA`、`MessageBoxW`两个函数进行加密处理。跳过这些加密处理就可以得到完整的IAT。

仍然用脚本跳过这些加密处理，得到的IAT全部有效。脚本运行结束后就可以用`ImportREC`修复IAT了，`OEP＝7EBB8`　`RVA＝84164`　`SIZE＝6F0`即可。

## IAT 信息

最后完整的IAT：

```
; Syntax for each function in a thunk (the separator is a TAB)

; ------------------------------------------------------------

; Flag   RVA   ModuleName   Ordinal   Name

;

; Details for <Valid> parameter:

; ------------------------------

; Flag:  0 = valid: no  -> - Name contains the address of the redirected API (you can set

;                            it to zero if you edit it).

;                          - Ordinal is not considered but you should let '0000' as value.

;                          - ModuleName is not considered but you should let '?' as value.

;

;        1 = valid: yes -> All next parameters on the line will be considered.

;                          Function imported by ordinal must have no name (the 4th TAB must

;                                                                          be there though).

;

;        2 = Equivalent to 0 but it is for the loader.

;

;        3 = Equivalent to 1 but it is for the loader.

;

; And finally, edit this file as your own risk! :-)

Target: C:\Documents and Settings\csjwaman\桌面\Project2\Project2.exe

OEP: 0007EBB8        IATRVA: 00084164        IATSize: 000006F0

FThunk: 00084168        NbFunc: 00000022

1        00084168        kernel32.dll        007B        DeleteCriticalSection

1        0008416C        kernel32.dll        023A        LeaveCriticalSection

1        00084170        kernel32.dll        0090        EnterCriticalSection

1        00084174        kernel32.dll        020F        InitializeCriticalSection

1        00084178        kernel32.dll        0367        VirtualFree

1        0008417C        kernel32.dll        0364        VirtualAlloc

1        00084180        kernel32.dll        0245        LocalFree

1        00084184        kernel32.dll        0241        LocalAlloc

1        00084188        kernel32.dll        01D4        GetVersion

1        0008418C        kernel32.dll        0138        GetCurrentThreadId

1        00084190        kernel32.dll        0213        InterlockedDecrement

1        00084194        kernel32.dll        0217        InterlockedIncrement

1        00084198        kernel32.dll        036C        VirtualQuery

1        0008419C        kernel32.dll        0378        WideCharToMultiByte

1        000841A0        kernel32.dll        025E        MultiByteToWideChar

1        000841A4        kernel32.dll        03AC        lstrlen

1        000841A8        kernel32.dll        03A9        lstrcpyn

1        000841AC        kernel32.dll        023C        LoadLibraryExA

1        000841B0        kernel32.dll        01C6        GetThreadLocale

1        000841B4        kernel32.dll        01A6        GetStartupInfoA

1        000841B8        kernel32.dll        0191        GetProcAddress

1        000841BC        kernel32.dll        016F        GetModuleHandleA

1        000841C0        kernel32.dll        016D        GetModuleFileNameA

1        000841C4        kernel32.dll        0165        GetLocaleInfoA

1        000841C8        kernel32.dll        0103        GetCommandLineA

1        000841CC        kernel32.dll        00EA        FreeLibrary

1        000841D0        kernel32.dll        00CA        FindFirstFileA

1        000841D4        kernel32.dll        00C6        FindClose

1        000841D8        kernel32.dll        00B0        ExitProcess

1        000841DC        kernel32.dll        0385        WriteFile

1        000841E0        kernel32.dll        0351        UnhandledExceptionFilter

1        000841E4        kernel32.dll        02BE        RtlUnwind

1        000841E8        kernel32.dll        0290        RaiseException

1        000841EC        kernel32.dll        01A8        GetStdHandle

FThunk: 000841F4        NbFunc: 00000004

1        000841F4        user32.dll        0128        GetKeyboardType

1        000841F8        user32.dll        01C9        LoadStringA

1        000841FC        user32.dll        01DD        MessageBoxA

1        00084200        user32.dll        002B        CharNextA

FThunk: 00084208        NbFunc: 00000003

1        00084208        advapi32.dll        01EC        RegQueryValueExA

1        0008420C        advapi32.dll        01E2        RegOpenKeyExA

1        00084210        advapi32.dll        01C9        RegCloseKey

FThunk: 00084218        NbFunc: 00000003

1        00084218        oleaut32.dll        0006        SysFreeString

1        0008421C        oleaut32.dll        0005        SysReAllocStringLen

1        00084220        oleaut32.dll        0004        SysAllocStringLen

FThunk: 00084228        NbFunc: 00000004
```
