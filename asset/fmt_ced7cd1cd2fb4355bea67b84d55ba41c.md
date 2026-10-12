
# 隐藏进程中的模块绕过 IceSword 的检测

> 原文：[`www.pediy.com/kssd/pediy10/66886.html`](https://www.pediy.com/kssd/pediy10/66886.html)

【文章标题】: 隐藏进程中的模块绕过 IceSword 的检测  

【文章作者】: 小伟的小伟\0GiNr\  

【作者主页】: [`http://www.0GiNr.com`](http://www.0GiNr.com) [`hi.baidu.com/zoo%5F`](http://hi.baidu.com/zoo%5F)  

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  

---

## 详细过程

最近对隐藏 DLL 来了点兴趣，首先是断了 `PEB` 里面的双向链，过了大部分程序。然后呢把 `MZ` 标志改掉了过了暴力搜索（感谢 aegisys）。

接下来就是对抗 IceSword 使用的 `NtQueryVirtualMemory` 的方法了。

我去 WRK 里面看了下 `NtQueryVirtualMemory` 的源码，找到了这里。![](img/3e9367e5826886294bccbb08463f3e34.jpg "酷") ![](img/3e9367e5826886294bccbb08463f3e34.jpg "酷")

以下是调试笔记。

```
lkd> dt_EPROCESS 0x81583B60

nt!_EPROCESS
  +0x000 Pcb             : _KPROCESS
  ;.............
  +0x11c VadRoot        : 0x816afb68

  ;.............

lkd> dt_MMVAD 0x816afb68 ;看到了吧。。。二叉树。。。。。。。。。。。

nt!_MMVAD
  +0x000 StartingVpn      : 0x30
  +0x004 EndingVpn        : 0x12f
  +0x008 Parent           : (null)
  +0x00c LeftChild        : 0x813d7748 _MMVAD
  +0x010 RightChild       : 0x816b21d8 _MMVAD  ;选择右节点来遍历
  +0x014 u                : __unnamed
  +0x018 ControlArea      : 0x0a050004 _CONTROL_AREA
  +0x01c FirstPrototypePte : 0x6d665346 _MMPTE
  +0x020 LastContiguousPte : 0x00000001 _MMPTE
  +0x024 u2               : __unnamed

lkd> dt_MMVAD 0x816b21d8

nt!_MMVAD
  +0x000 StartingVpn      : 0x400
  +0x004 EndingVpn        : 0x410
  +0x008 Parent           : 0x816afb68 _MMVAD
  +0x00c LeftChild        : 0x81672bb0 _MMVAD
  +0x010 RightChild       : 0x817b58d8 _MMVAD  ;再来一次
  +0x014 u                : __unnamed
  +0x018 ControlArea      : 0x8144a2f8 _CONTROL_AREA   ;这里是进程映像的地方。
  +0x01c FirstPrototypePte : 0xe24faaa0 _MMPTE
  +0x020 LastContiguousPte : 0xfffffffc _MMPTE
  +0x024 u2               : __unnamed

lkd> dt_MMVAD 0x817b58d8

nt!_MMVAD
  +0x000 StartingVpn      : 0x7c920
  +0x004 EndingVpn        : 0x7c9b3
  +0x008 Parent           : 0x816b21d8 _MMVAD
  +0x00c LeftChild        : 0x81731348 _MMVAD
  +0x010 RightChild       : 0x81629b70 _MMVAD
  +0x014 u                : __unnamed
  +0x018 ControlArea      : 0x81f05138 _CONTROL_AREA   ;从这里进去。。
  +0x01c FirstPrototypePte : 0xe1377040 _MMPTE
  +0x020 LastContiguousPte : 0xfffffffc _MMPTE
  +0x024 u2               : __unnamed

lkd> dt_CONTROL_AREA 0x81f05138

nt!_CONTROL_AREA
  +0x000 Segment          : 0xe1377008 _SEGMENT
  +0x004 DereferenceList  : _LIST_ENTRY [ 0x0 - 0x0 ]
  +0x00c NumberOfSectionReferences : 1
  +0x010 NumberOfPfnReferences : 0x6d
  +0x014 NumberOfMappedViews : 0x19
  +0x018 NumberOfSubsections : 5
  +0x01a FlushInProgressCount : 0
  +0x01c NumberOfUserReferences : 0x1a
  +0x020 u                : __unnamed
  +0x024 FilePointer      : 0x81e9d938 _FILE_OBJECT  ;这里可以得到模块信息
  +0x028 WaitingForDeletion : (null)
  +0x02c ModifiedWriteCount : 0
  +0x02e NumberOfSystemCacheViews : 0

lkd> dt_FILE_OBJECT 0x81e9d938 ;到这里 ntdll.dll 已经出来了。。。。

nt!_FILE_OBJECT
  +0x000 Type           : 5
  +0x002 Size           : 112
  +0x004 DeviceObject   : 0x81dce7b8 _DEVICE_OBJECT
  +0x008 Vpb            : 0x81e9f8e0 _VPB
  +0x00c FsContext      : 0xe13939e0
  +0x010 FsContext2     : 0xe1393b38
  +0x014 SectionObjectPointer : 0x81e9338c _SECTION_OBJECT_POINTERS
  +0x018 PrivateCacheMap  : (null)
  +0x01c FinalStatus      : 0
  +0x020 RelatedFileObject : (null)
  +0x024 LockOperation    : 0 ''
  +0x025 DeletePending    : 0 ''
  +0x026 ReadAccess       : 0x1 ''
  +0x027 WriteAccess      : 0 ''
  +0x028 DeleteAccess     : 0 ''
  +0x029 SharedRead       : 0x1 ''
  +0x02a SharedWrite      : 0 ''
  +0x02b SharedDelete     : 0 ''
  +0x02c Flags            : 0x44040
  +0x030 FileName         : _UNICODE_STRING "\\WINDOWS\\system32\\ntdll.dll"
  +0x038 CurrentByteOffset : _LARGE_INTEGER 0x0
  +0x040 Waiters          : 0
  +0x044 Busy             : 0
  +0x048 LastLock         : (null)
  +0x04c Lock             : _KEVENT
  +0x05c Event            : _KEVENT
  +0x06c CompletionContext : (null)

lkd> dt_FILE_OBJECT 0x81c08350 ;再往后是 kernel32.dll

nt!_FILE_OBJECT
  ;......................
  +0x030 FileName         : _UNICODE_STRING "\\WINDOWS\\system32\\kernel32.dll"
  ;......................
```

这里是源码，包含了大量硬编码，要编译的话自己搜索一下就可以。

```c
//////////////////////////////////////
//在 Ring0 下通过遍历 VAD 获取进程内的模块（WinXP SP2）
//By 小伟的小伟\0GiNr\
//http://www.0GiNr.com
//http://hi.baidu.com/zoo%5F
/////////////////////////////////////

VOID ShowModules()
{
  ULONG VAD;
  PEPROCESS TargetProcess;
  ////////////////////////////////////////
  PsLookupProcessByProcessId( (HANDLE)Pid,TargetProcess);
  if(!TargetProcess)
  {
    dprintf("[EnumModules] Error on Get EProcess By Pid.");
    return;
  }
  VAD = *(ULONG *)((ULONG)TargetProcess + Vad);
  //+0x11c VadRoot : Ptr32 Void
  dprintf("[EnumModules] EPROCESS : 0x%X , VAD : 0x%X",TargetProcess,VAD);
  PreOrderTraverse(VAD);  //遍历二叉树。。。
  dprintf("[EnumModules] Modules count : %d",nCount);
  //start.
}

VOID PreOrderTraverse(ULONG mmVad)
{
  if ( MmIsAddressValid( (ULONG *)mmVad ) )
  {
    ShowPath(mmVad);//读取地址
    PreOrderTraverse( *(ULONG *)(mmVad + LeftChild) );
    PreOrderTraverse( *(ULONG *)(mmVad + RightChild) );
  }
}

VOID ShowPath(ULONG mmVad)
{
  PUNICODE_STRING pPath;
  ULONG ca; //_CONTROL_AREA
  ULONG fp; //_FILE_OBJECT
  ca = *(ULONG *)(mmVad + ControlArea);
  if( !MmIsAddressValid( (ULONG *)ca ) )
  {
    //dprintf("[EnumModules] ControlArea is not available : 0x%X",ca);
    return;
  }
  fp = *(ULONG *)(ca + FilePointer);
  if( !MmIsAddressValid( (ULONG *)fp ) )
  {
    //dprintf("[EnumModules] FileObject is not available : 0x%X",fp);
    return;
  }
  pPath = (PUNICODE_STRING)(fp + FileName);
  dprintf("[EnumModules] The file name is %S",pPath->Buffer);
  //dprintf("[EnumModules] The MMVAD is 0x%X",mmVad);
  //dprintf("\n");
  nCount++;//计数
}
```

这样和用 `NtQueryVirtualMemory` 扫描出来的结果一样(`MemorySectionName`)，可以在 DebugView 里看到。输出是这样的。

```
[EnumModules] The file name is \\Project\\VisualC++\\HideModule\\Release\\HideModule.exe
[EnumModules] The file name is \\WINDOWS\\system32\\ntdll.dll
[EnumModules] The file name is \\WINDOWS\\system32\\kernel32.dll
[EnumModules] The file name is \\WINDOWS\\system32\\advapi32.dll
[EnumModules] The file name is \\WINDOWS\\system32\\rpcrt4.dll
[EnumModules] The file name is \\WINDOWS\\system32\\gdi32.dll
[EnumModules] The file name is \\WINDOWS\\system32\\user32.dll
[EnumModules] The file name is \\WINDOWS\\system32\\imm32.dll
[EnumModules] The file name is \\WINDOWS\\system32\\msvcrt.dll
[EnumModules] The file name is \\WINDOWS\\system32\\apphelp.dll
[EnumModules] The file name is \\WINDOWS\\system32\\version.dll
[EnumModules] The file name is \\WINDOWS\\system32\\lpk.dll
[EnumModules] The file name is \\WINDOWS\\system32\\usp10.dll
[EnumModules] The file name is \\WINDOWS\\system32\\unicode.nls
```
