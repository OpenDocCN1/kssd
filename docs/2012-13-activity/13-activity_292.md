
# 第三阶段简单分析

> 原文：[`www.pediy.com/kssd/pediy10/75886.html`](https://www.pediy.com/kssd/pediy10/75886.html)

## 1. 简单分析下

木马释放一个驱动，一个 `SYSTEM.EXE`，一个 `HBQQXX.dll`，其中驱动起到保护这几个文件的作用，`HBQQXX.dll` 主要起到木马的主要功能，在此不做解释

## 2.

木马首先查找相关进程，并试图关掉游戏，包含在内的游戏有

```
00403000  6D 79 2E 65 78 65 00 43 6C 69 65 6E 74 2E 65 78  my.exe.Client.ex
00403010  65 00 77 6F 6F 6F 6C 2E 64 61 74 00 77 6F 6F 6F  e.woool.dat.wooo
00403020  6C 38 38 2E 64 61 74 00 78 79 32 2E 65 78 65 00  l88.dat.xy2.exe.
00403030  67 61 6D 65 2E 65 78 65 00 53 4F 32 47 61 6D 65  game.exe.SO2Game
00403040  2E 65 78 65 00 53 4F 32 47 61 6D 65 46 72 65 65  .exe.SO2GameFree
00403050  2E 65 78 65 00 46 53 4F 6E 6C 69 6E 65 32 2E 65  .exe.FSOnline2.e
00403060  78 65 00 67 61 6D 65 63 6C 69 65 6E 74 2E 65 78  xe.gameclient.ex
00403070  65 00 65 6C 65 6D 65 6E 74 63 6C 69 65 6E 74 2E  e.elementclient.
00403080  65 78 65 00 61 73 6B 74 61 6F 2E 6D 6F 64 00 57  exe.asktao.mod.W
00403090  6F 77 2E 65 78 65 00 5A 65 72 6F 4F 6E 6C 69 6E  ow.exe.ZeroOnlin
004030A0  65 2E 65 78 65 00 42 6F 2E 65 78 65 00 43 6F 6E  e.exe.Bo.exe.Con
004030B0  71 75 65 72 2E 65 78 65 00 73 6F 75 6C 2E 65 78  quer.exe.soul.ex
004030C0  65 00 54 68 65 57 61 72 6C 6F 72 64 73 2E 65 78  e.TheWarlords.ex
004030D0  65 00 63 68 69 6E 61 5F 6C 6F 67 69 6E 2E 6D 70  e.china_login.mp
004030E0  72 00 62 6C 75 65 73 6B 79 63 6C 69 65 6E 74 5F  r.blueskyclient_
004030F0  72 2E 65 78 65 00 78 79 33 2E 65 78 65 00 51 51  r.exe.xy3.exe.QQ
00403100  4C 6F 67 69 6E 2E 65 78 65 00 44 4E 46 2E 65 78  Login.exe.DNF.ex
00403110  65 00 67 63 31 32 2E 65 78 65 00 68 75 67 65 6D  e.gc12.exe.hugem
00403120  61 6E 63 6C 69 65 6E 74 2E 65 78 65 00 48 58 32  anclient.exe.HX2
00403130  47 61 6D 65 2E 65 78 65 00 51 51 68 78 67 61 6D  Game.exe.QQhxgam
00403140  65 2E 65 78 65 00 74 77 32 2E 65 78 65 00 51 51  e.exe.tw2.exe.QQ
00403150  53 47 2E 65 78 65 00 51 51 46 46 4F 2E 65 78 65  SG.exe.QQFFO.exe
00403160  00 7A 68 65 6E 67 74 75 2E 64 61 74 00 6D 69 72  .zhengtu.dat.mir
00403170  31 2E 64 61 74 00 6D 69 72 32 2E 64 61 74 00 74  1.dat.mir2.dat.t
00403180  74 79 33 64 2E 65 78 65 00 6D 65 74 69 6E 32 2E  ty3d.exe.metin2.
00403190  62 69 6E 00 41 43 6C 69 65 6E 74 2E 65 78 65 00  bin.AClient.exe.
004031A0  67 61 6D 65 66 72 65 65 2E 65 78 65              gamefree.exe
```

然后去掉了 360 在注册表中的一些键值，使 360 失效，然后释放 `SYSTEM.EXE`，`HBKernel32.sys`，`HBQQXX.dll`，最后就是加载 `HBKernel32.sys`，运行 `SYSTEM.EXE` 并写入启动项，最后就是自删除了，

## 3.

`SYSTEM.exe` 创建个窗口，并设置个时钟不停写 `APPINIT_DLLS`

```
00403000  48 42 6D 68 6C 79 2E 64 6C 6C 00 48 42 31 30 30  HBmhly.dll.HB100
00403010  30 59 2E 64 6C 6C 00 48 42 57 4F 4F 4F 4C 2E 64  0Y.dll.HBWOOOL.d
00403020  6C 6C 00 48 42 58 59 32 2E 64 6C 6C 00 48 42 4A  ll.HBXY2.dll.HBJ
00403030  58 53 4A 2E 64 6C 6C 00 48 42 53 4F 32 2E 64 6C  XSJ.dll.HBSO2.dl
00403040  6C 00 48 42 46 53 32 2E 64 6C 6C 00 48 42 58 59  l.HBFS2.dll.HBXY
00403050  33 2E 64 6C 6C 00 48 42 53 48 51 2E 64 6C 6C 00  3.dll.HBSHQ.dll.
00403060  48 42 46 59 2E 64 6C 6C 00 48 42 57 55 4C 49 4E  HBFY.dll.HBWULIN
00403070  32 2E 64 6C 6C 00 48 42 57 32 49 2E 64 6C 6C 00  2.dll.HBW2I.dll.
00403080  48 42 4B 44 58 59 2E 64 6C 6C 00 48 42 57 4F 52  HBKDXY.dll.HBWOR
00403090  4C 44 32 2E 64 6C 6C 00 48 42 41 53 4B 54 41 4F  LD2.dll.HBASKTAO
004030A0  2E 64 6C 6C 00 48 42 5A 48 55 58 49 41 4E 2E 64  .dll.HBZHUXIAN.d
004030B0  6C 6C 00 48 42 57 4F 57 2E 64 6C 6C 00 48 42 5A  ll.HBWOW.dll.HBZ
004030C0  45 52 4F 2E 64 6C 6C 00 48 42 42 4F 2E 64 6C 6C  ERO.dll.HBBO.dll
004030D0  00 48 42 43 4F 4E 51 55 45 52 2E 64 6C 6C 00 48  .HBCONQUER.dll.H
004030E0  42 53 4F 55 4C 2E 64 6C 6C 00 48 42 43 48 49 42  BSOUL.dll.HBCHIB
004030F0  49 2E 64 6C 6C 00 48 42 44 4E 46 2E 64 6C 6C 00  I.dll.HBDNF.dll.
00403100  48 42 57 41 52 4C 4F 52 44 53 2E 64 6C 6C 00 48  HBWARLORDS.dll.H
00403110  42 54 4C 2E 64 6C 6C 00 48 42 50 49 43 4B 43 48  BTL.dll.HBPICKCH
00403120  49 4E 41 2E 64 6C 6C 00 48 42 43 54 2E 64 6C 6C  INA.dll.HBCT.dll
00403130  00 48 42 47 43 2E 64 6C 6C 00 48 42 48 4D 2E 64  .HBGC.dll.HBHM.d
00403140  6C 6C 00 48 42 48 58 32 2E 64 6C 6C 00 48 42 51  ll.HBHX2.dll.HBQ
00403150  51 48 58 2E 64 6C 6C 00 48 42 54 57 32 2E 64 6C  QHX.dll.HBTW2.dl
00403160  6C 00 48 42 51 51 53 47 2E 64 6C 6C 00 48 42 51  l.HBQQSG.dll.HBQ
00403170  51 46 46 4F 2E 64 6C 6C 00 48 42 5A 54 2E 64 6C  QFFO.dll.HBZT.dl
00403180  6C 00 48 42 4D 49 52 32 2E 64 6C 6C 00 48 42 52  l.HBMIR2.dll.HBR
00403190  58 4A 48 2E 64 6C 6C 00 48 42 59 59 2E 64 6C 6C  XJH.dll.HBYY.dll
004031A0  00 48 42 4D 58 44 2E 64 6C 6C 00 48 42 53 51 2E  .HBMXD.dll.HBSQ.
004031B0  64 6C 6C 00 48 42 54 4A 2E 64 6C 6C 00 48 42 46  dll.HBTJ.dll.HBF
004031C0  48 5A 4C 2E 64 6C 6C 00 48 42 57 4C 51 58 2E 64  HZL.dll.HBWLQX.d
004031D0  6C 6C 00 48 42 4C 59 46 58 2E 64 6C 6C 00 48 42  ll.HBLYFX.dll.HB
004031E0  52 32 2E 64 6C 6C 00 48 42 43 48 44 2E 64 6C 6C  R2.dll.HBCHD.dll
004031F0  00 48 42 54 5A 2E 64 6C 6C 00 48 42 51 51 58 58  .HBTZ.dll.HBQQXX
00403200  2E 64 6C 6C 00 48 42 57 44 2E 64 6C 6C 00 48 42  .dll.HBWD.dll.HB
00403210  5A 47 2E 64 6C 6C 00 48 42 50 50 42 4C 2E 64 6C  ZG.dll.HBPPBL.dl
00403220  6C 00 48 42 58 4D 4A 2E 64 6C 6C 00 48 42 4A 54  l.HBXMJ.dll.HBJT
00403230  4C 51 2E 64 6C 6C 00 48 42 51 4A 53 4A 2E 64 6C  LQ.dll.HBQJSJ.dl
00403240  6C 00 00 00 00 00 00 00 00 00 00 00 00        l.............
```

然后设置鼠标钩子加载 `HBQQXX.dll`，最后设置 360 的相关注册表键值，使 360 失效

并吧自身的 `Pid` 发给驱动，让驱动给保护起来

## 4.

驱动挂钩了 `NtOpenProcess`，`NtSetFileInformation`，`NtCreateThread`，`NtSetValueKey`，来过滤一些操作，文件路径中包含 `HBKernel32.sys`，`SYSTEM.EXE`，`HBmhly.dll` 的文件，对他们操作返回数据结构错误的错误码，而 `HBkernel32.sys` 还被占用，这个是在 `HBkernel32.SYS` 创建的线程里完成的，而这个线程不段的挂钩上面这几个函数，还不停将驱动，`SYSTEM.EXE`，`HBmhly.dll` 写入注册表

先看下进程保护，它通过过滤 `Pid` 来保护进程，然而 `SYSTEM.EXE` 创建了个窗口，在 `WINDOWS` 强大的消息机制面前，这个保护显得苍白无力：

```
VOID KILLSYSTEM.EXE()
{
    SendMessageA(FindWindow("HBInject32Class",NULL),WM_CLOSE,0,0);
}
```

看看 IOCODE 的处理

```
int __stdcall handleIoCode(int a1, int a2, void *inputbuffer, int inputbufferlength, int userbuffer, int outputbufferlength, int iocode, int iostatus, int a9)
{
  int v9;  // esi@1
  v9 = iostatus;
  *(_DWORD *)iostatus = STATUS_INVALID_DEVICE_REQUEST;
  *(_DWORD *)(iostatus + 4) = 0;
  switch ( iocode )
  {
    case 0x22E007:
      *(_DWORD *)iostatus = RecoverALLServiceAddress();
      break;
    case 0x22E00B:
      if ( (unsigned int)inputbufferlength >= 4 && (unsigned int)outputbufferlength >= 4 )
      {
        ProbeForRead(inputbuffer, 4u, 4u);
        ProbeForWrite((PVOID)userbuffer, 4u, 4u);
        *(_DWORD *)userbuffer = ProtectPid;
        ProtectPid = *(_DWORD *)inputbuffer;
        *(_DWORD *)v9 = STATUS_SUCCESS;
        *(_DWORD *)(v9 + 4) = 4;
      }
      else
      {
        *(_DWORD *)iostatus = STATUS_BUFFER_TOO_SMALL;
      }
      break;
    case 0x22E00F:
      if ( (unsigned int)inputbufferlength >= 4 )
      {
        ProbeForRead(inputbuffer, 4u, 4u);
        if ( *(_DWORD *)inputbuffer )
          IsProtectFile = 0;
        else
          IsProtectFile = 1;
        *(_DWORD *)v9 = 0;
      }
      else
      {
        *(_DWORD *)iostatus = STATUS_BUFFER_TOO_SMALL;
      }
      break;
  }
  return *(_DWORD *)v9;
}
```

从这里可以看出，它对文件和进程的保护，我们只要自己发个 `IOCODE` 就可以解决掉+

```
DWORD input=0;
BOOL issuccess;
HANDLE handle=CreateFileA("\\\\.\\slHBKernel32",GENERIC_ALL,0,NULL,OPEN_EXISTING,FILE_ATTRIBUTE_NORMAL,NULL);
  issuccess=DeviceIoControl(handle,0x22e00f,&input,4,&input,4,&ret,NULL);
  issuccess=DeviceIoControl(handle,0x22e00b,&input,4,&input,4,&ret,NULL);
CloseHandle(handle);
```

`NtSetValueKey` 过滤了对 `APPINIT_DLL` 的键值操作，但没有对键名过滤，改名后直接可以删除，但还有线程重复写这个地方

现在 `SYSTEM.EXE` 等文件可以删去了，但驱动还是无法删掉，因为它一直都是被占用，看看那个线程就知道

D



## StartRoutine 反汇编

```
.text:00012D6D loc_12D6D:                              ; CODE XREF: StartRoutine+25 j
.text:00012D6D                                         ; StartRoutine+428 j
.text:00012D6D         cmp     IsTerminate, 0
.text:00012D74         jz      loc_12958
.text:00012D7A         cmp     [ebp+PspCreateThreadInNtCreateThread], 0
.text:00012D81         jz      short loc_12D8E
.text:00012D83         push    [ebp+PspCreateThreadInNtCreateThread]
.text:00012D89         call    UnInlinehook
.text:00012D8E

.text:00012D8E loc_12D8E:                              ; CODE XREF: StartRoutine+453 j
.text:00012D8E         cmp     [ebp+ObReferenceObjectByHandleInNtSetInformationFile], 0
.text:00012D95         jz      short loc_12DA2
.text:00012D97         push    [ebp+ObReferenceObjectByHandleInNtSetInformationFile]
.text:00012D9D         call    UnInlinehook
.text:00012DA2

.text:00012DA2 loc_12DA2:                              ; CODE XREF: StartRoutine+467 j
.text:00012DA2         cmp     [ebp+PsLookupProcessByProcessIdInNtOpenProcess], 0
.text:00012DA9         jz      short loc_12DB6
.text:00012DAB         push    [ebp+PsLookupProcessByProcessIdInNtOpenProcess]
.text:00012DB1         call    UnInlinehook
.text:00012DB6

.text:00012DB6 loc_12DB6:                              ; CODE XREF: StartRoutine+47B j
.text:00012DB6         cmp     [ebp+indexofNtSetValueKey], 0
.text:00012DBD         jz      short loc_12DCA
.text:00012DBF         push    [ebp+indexofNtSetValueKey]
.text:00012DC5         call    RecoverService
.text:00012DCA

.text:00012DCA loc_12DCA:                              ; CODE XREF: StartRoutine+48F j
.text:00012DCA         cmp     [ebp+Handle], 0
.text:00012DD1         jz      short loc_12DE5
.text:00012DD3         push    [ebp+Handle]    ; Handle
.text:00012DD9         call    ZwClose
.text:00012DDE         and     [ebp+Handle], 0
.text:00012DE5

.text:00012DE5 loc_12DE5:                              ; CODE XREF: StartRoutine+4A3 j
.text:00012DE5         push    0               ; ExitStatus
.text:00012DE7         call    PsTerminateSystemThread
.text:00012DEC         pop     edi
.text:00012DED         leave
.text:00012DEE         retn    4
.text:00012DEE StartRoutine    endp
```

只要 `00012D74` 不跳就可以了，但 `IsTerminate` 这个地方除了开始初始化为 `0` 外，没有地方对他进行写，

如果用驱动的话很简单，不用驱动解决的话我知道一个函数可以操作内核内存 `NtSystemDebugControl`

### `NtSystemDebugControl` 定义

```c
NTSYSAPI
NTSTATUS
NTAPI
NtSystemDebugControl (
    IN SYSDBG_COMMAND Command,
    IN PVOID InputBuffer,
    IN ULONG InputBufferLength,
    OUT PVOID OutputBuffer,
    IN ULONG OutputBufferLength,
    OUT PULONG ReturnLength
);
```

```c
typedef enum _SYSDBG_COMMAND {
    SysDbgGetTraceInformation = 1,
    SysDbgSetInternalBreakpoint = 2,
    SysDbgSetSpecialCall = 3,
    SysDbgClearSpecialCalls = 4,
    SysDbgQuerySpecialCalls = 5,
    SysDbgDbgBreakPointWithStatus = 6,
    SysDbgSysGetVersion = 7,
    // 从内核空间拷贝到用户空间，或者从用户空间拷贝到用户空间
    // 但是不能从用户空间拷贝到内核空间
    SysDbgCopyMemoryChunks_0 = 8,
  // SysDbgReadVirtualMemory = 8,
    // 从用户空间拷贝到内核空间，或者从用户空间拷贝到用户空间
    // 但是不能从内核空间拷贝到用户空间
    SysDbgCopyMemoryChunks_1 = 9,
  // SysDbgWriteVirtualMemory = 9,
    // 从物理地址拷贝到用户空间，不能写到内核空间
    SysDbgCopyMemoryChunks_2 = 10,
  // SysDbgReadVirtualMemory = 10,
    // 从用户空间拷贝到物理地址，不能读取内核空间
    SysDbgCopyMemoryChunks_3 = 11,
  // SysDbgWriteVirtualMemory = 11,
    // 读写处理器相关控制块
    SysDbgSysReadControlSpace = 12,
    SysDbgSysWriteControlSpace = 13,
    // 读写端口
    SysDbgSysReadIoSpace = 14,
    SysDbgSysWriteIoSpace = 15,
    // 分别调用 RDMSR@4 和 _WRMSR@12
    SysDbgSysReadMsr = 16,
    SysDbgSysWriteMsr = 17,
    // 读写总线数据
    SysDbgSysReadBusData = 18,
    SysDbgSysWriteBusData = 19,
    SysDbgSysCheckLowMemory = 20,
    // 以下是 NT 5.2 新增的
    // 分别调用 _KdEnableDebugger@0 和 _KdDisableDebugger@0
    SysDbgEnableDebugger = 21,
    SysDbgDisableDebugger = 22,
    // 获取和设置一些调试相关的变量
    SysDbgGetAutoEnableOnEvent = 23,
    SysDbgSetAutoEnableOnEvent = 24,
    SysDbgGetPitchDebugger = 25,
    SysDbgSetDbgPrintBufferSize = 26,
    SysDbgGetIgnoreUmExceptions = 27,
    SysDbgSetIgnoreUmExceptions = 28
} SYSDBG_COMMAND, *PSYSDBG_COMMAND;
```

这个函数很强大，能达到我们的目的，具体代码我就不写了，至于用驱动的话，大家都是在一个水平上，至于怎么用那就是八仙过海，各显神通了

附件是驱动 IDB

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || HBKernel32.rar  |
