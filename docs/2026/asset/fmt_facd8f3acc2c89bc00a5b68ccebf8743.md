
# [原创]某6789安全卫士核心功能之InfinityHook （抄，我疯狂的抄）

InfinityHook 核心原理 替换`WmipLoggerContext[CLCK_INDEX]->GetCpuClock`

原作者见 `600K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2N6X3g2J5k6r3!0^5i4K6u0r3d9h3&6X3K9h3&6A6N6s2W2t1L8$3!0C8`

from `6789base.sys` version 5.12.0.4874

为节约版面以下只放关键内容

## `InitializeInfinityHook()`

`DriverEntry->InitializeInfinityHook()` 启动两条线程，一条初始化`infhook`，一条保护`infhook`。

```__int64 InitializeInfinityHook()

{

  HANDLE ThreadHandle; // [rsp+40h] [rbp-58h]

  HANDLE Handle; // [rsp+48h] [rbp-50h]

  OBJECT_ATTRIBUTES ObjectAttributes; // [rsp+50h] [rbp-48h]

  unsigned int v4; // [rsp+80h] [rbp-18h]



  v4 = -1073741823;

  ThreadHandle = 0i64;

  ObjectAttributes.Length = 0;

  memset(&ObjectAttributes.RootDirectory, 0, 0x28ui64);

  if ( byte_3648C )

    return 3221225473i64;

  stru_365D0.Count = 1;

  stru_365D0.Owner = 0i64;

  stru_365D0.Contention = 0;

  KeInitializeEvent(&stru_365D0.Event, SynchronizationEvent, 0);

  sub_22980(&unk_36608);

  KeInitializeEvent(&Event, NotificationEvent, 0);

  byte_36938 = 1;

  stru_36568.Count = 1;

  stru_36568.Owner = 0i64;

  stru_36568.Contention = 0;

  KeInitializeEvent(&stru_36568.Event, SynchronizationEvent, 0);

  ObjectAttributes.Length = 48;

  ObjectAttributes.RootDirectory = 0i64;

  ObjectAttributes.Attributes = 512;

  ObjectAttributes.ObjectName = 0i64;

  ObjectAttributes.SecurityDescriptor = 0i64;

  ObjectAttributes.SecurityQualityOfService = 0i64;

  v4 = PsCreateSystemThread(

         &ThreadHandle,

         0x1FFFFFu,

         &ObjectAttributes,

         0i64,

         0i64,

         (PKSTART_ROUTINE)InfinityHookInitThread,

         0i64);

  if ( !v4 )

    ZwClose(ThreadHandle);

  ObjectAttributes.Length = 48;

  ObjectAttributes.RootDirectory = 0i64;

  ObjectAttributes.Attributes = 512;

  ObjectAttributes.ObjectName = 0i64;

  ObjectAttributes.SecurityDescriptor = 0i64;

  ObjectAttributes.SecurityQualityOfService = 0i64;

  v4 = PsCreateSystemThread(

         &Handle,

         0x1FFFFFu,

         &ObjectAttributes,

         0i64,

         0i64,

         (PKSTART_ROUTINE)InitializeInfinityHookProtectThread,

         0i64);

  if ( !v4 )

  {

    ObReferenceObjectByHandle(Handle, 0x1FFFFFu, PsThreadType, 0, &g_ThreadObject, 0i64);

    ZwClose(Handle);

  }

  return v4;

}
```

## `InfinityHookInitThread()`

```void __fastcall InfinityHookInitThread(PVOID StartContext)

{

  __int64 _RAX; // rax

  LARGE_INTEGER (*_RCX)(); // rcx

  __int64 _RCX; // rcx

  LARGE_INTEGER Interval; // [rsp+20h] [rbp-28h]

  _QWORD *getCpuClock; // [rsp+28h] [rbp-20h]

  int v6; // [rsp+30h] [rbp-18h]



  v6 = -1073741823;

  getCpuClock = 0i64;

  Interval.QuadPart = -30000000i64;

  while ( !LoadNtdll() )

    KeDelayExecutionThread(0, 0, &Interval);

  if ( (int)InitializeInfhookCore() >= 0 )

  {

    v6 = EnableEtwLogger();

    if ( v6 >= 0 )

    {

      v6 = GetCkcl(2u, (_QWORD *)infhookCore + 90115);

      if ( v6 >= 0 && *((_QWORD *)infhookCore + 90115) )

      {

        getCpuClock = (_QWORD *)(*((_QWORD *)infhookCore + 90115)

                               + (unsigned int)OFFSET_WMI_LOGGER_CONTEXT_CPU_CYCLE_CLOCK);

        if ( (unsigned __int8)MmIsAddressValid(getCpuClock)

          && (unsigned __int8)MmIsAddressValid((char *)getCpuClock + 7) )

        {

          if ( *getCpuClock >= MmSystemRangeStart )

          {

            _RAX = *((_QWORD *)infhookCore + 90115) + (unsigned int)OFFSET_WMI_LOGGER_CONTEXT_CPU_CYCLE_CLOCK;

            _RCX = InternalGetCpuClock;

            __asm { xchg    rcx, [rax] }

            *((_QWORD *)infhookCore + 90114) = _RCX;

            v6 = 0;

          }

          else

          {

            v6 = 0xC0000001;

          }

        }

        else

        {

          v6 = 0xC0000001;

        }

      }

      else

      {

        v6 = 0xC0000001;

      }

    }

    else

    {

      v6 = 0xC0000001;

    }

  }

  else

  {

    v6 = 0xC0000001;

  }

  PsTerminateSystemThread(0);

}
```

## `EnableEtwLogger()`

```NTSTATUS __stdcall EnableEtwLogger()

{

  NTSTATUS result; // eax

  NTSTATUS v1; // [rsp+20h] [rbp-18h]

  NTSTATUS v2; // [rsp+20h] [rbp-18h]



  v1 = IfhpModifyTraceSettings(1);

  if ( v1 >= 0 )

    goto LABEL_9;

  v2 = IfhpModifyTraceSettings(0);

  if ( v2 < 0 )

    return v2;

  v1 = IfhpModifyTraceSettings(1);

  if ( v1 >= 0 )

LABEL_9:

    result = v1;

  else

    result = v1;

  return result;

}
```

## `IfhModifyTraceSettings`

`IfhModifyTraceSettings`复制粘贴过于直球这里就不贴代码了，放个对比图：

![](https://bbs.kanxue.com/upload/attach/202003/619065_6S6FZQQUPPVH4BH.jpg)

![](https://bbs.kanxue.com/upload/attach/202003/619065_KEAN87XZ6F4QYBV.jpg)

## `InitializeInfhookCore()`

主要流程：

- 初始化`infhookCore`结构（应该是一个很大的结构体，具体成员就不逆了）
- `sub_2D210`判断两个不明物体（无关紧要懒得逆）
- 寻找`explorer.exe`的PID
- 加载`L"\\SystemRoot\\system32\\ntdll.dll"`并搜索导出表，把nt函数全部保存进`infhookcore`里的一个表

```NTSTATUS __stdcall InitializeInfhookCore()

{

  LARGE_INTEGER Interval; // [rsp+20h] [rbp-28h]

  NTSTATUS v2; // [rsp+28h] [rbp-20h]

  UNICODE_STRING DestinationString; // [rsp+30h] [rbp-18h]



  v2 = -1073741823;

  Interval.QuadPart = -30000000i64;

  if ( infhookCore && isZwTraceControlAvailable )

    return 0;

  while ( 1 )

  {

    if ( !infhookCore )

    {

      infhookCore = ExAllocatePoolWithTag(NonPagedPool, 720928ui64, 'hchk');

      if ( !infhookCore )

      {

        v2 = -1073741670;

        goto LABEL_26;

      }

      memset(infhookCore, 0, 0xB0020ui64);

    }

    if ( !*((_QWORD *)infhookCore + 90112) && !*((_QWORD *)infhookCore + 90113) )

    {

      v2 = sub_2D210((char *)infhookCore + 720896, (char *)infhookCore + 720904);

      if ( v2 < 0 || !*((_QWORD *)infhookCore + 90112) && !*((_QWORD *)infhookCore + 90113) )

      {

        v2 = -1073741823;

        goto LABEL_26;

      }

    }

    RtlInitUnicodeString(&DestinationString, L"explorer.exe");

    v2 = FindProcessByProcessName(&DestinationString, &g_ExplorerPID);

    if ( v2 >= 0 )

    {

      if ( g_ExplorerPID )

        break;

    }

    KeDelayExecutionThread(0, 0, &Interval);

  }

  v2 = LoadSSDTFunctionEntry();

  if ( v2 >= 0 )

  {

    v2 = LoadOffsetsVars();

    if ( v2 >= 0 )

    {

      v2 = LoadSSSDTFunctionEntry();

      if ( v2 >= 0 )

      {

        byte_3648D = 1;

        sub_2A640();

        ZwTraceControl = (__int64 (__fastcall *)(_QWORD, _QWORD, _QWORD, _QWORD, _DWORD, _QWORD))MakeZwTraceControl();

        if ( ZwTraceControl )

        {

          isZwTraceControlAvailable = 1;

          v2 = 0;

        }

        else

        {

          v2 = -1073741823;

        }

      }

      else

      {

        v2 = -1073741823;

      }

    }

    else

    {

      v2 = -1073741823;

    }

  }

  else

  {

    v2 = -1073741823;

  }

LABEL_26:

  if ( v2 )

  {

    if ( infhookCore )

    {

      ExFreePoolWithTag(infhookCore, 0x6863686Bu);

      infhookCore = 0i64;

    }

  }

  return v2;

}
```

## `LoadOffsetsVars()`

根据系统版本加载一些偏移

```NTSTATUS __stdcall LoadOffsetsVars()

{

  NTSTATUS v1; // [rsp+20h] [rbp-18h]

  unsigned int v2; // [rsp+24h] [rbp-14h]



  v1 = 0;

  v2 = GetBuildNumber();

  if ( v2 > 14393 )

  {

    if ( v2 != 15063 && v2 != 16299 && v2 != 17134 && v2 != 17763 && v2 != 18362 )

      goto LABEL_18;

LABEL_17:

    LODWORD(OFFSET_WMI_LOGGER_CONTEXT_CPU_CYCLE_CLOCK) = 40;

    offset_SystemCallNumber = 0x80i64;

    PerfGlobalGroupMask = GetPerfGlobalGroupMask();

    goto LABEL_18;

  }

  switch ( v2 )

  {

    case 14393u:

      goto LABEL_17;

    case 610u:

      LODWORD(OFFSET_WMI_LOGGER_CONTEXT_CPU_CYCLE_CLOCK) = 24;

      offset_SystemCallNumber = 0x1F8i64;

      PerfGlobalGroupMask = GetPerfGlobalGroupMask2();

      break;

    case 620u:

      LODWORD(OFFSET_WMI_LOGGER_CONTEXT_CPU_CYCLE_CLOCK) = 40;

      offset_SystemCallNumber = 0x80i64;

      PerfGlobalGroupMask = GetPerfGlobalGroupMask3();

      break;

    case 630u:

    case 10240u:

    case 10586u:

      goto LABEL_17;

  }

LABEL_18:

  if ( v2 > 18362 )

    v1 = FindOffsetsFor18383();

  return v1;

}
```
