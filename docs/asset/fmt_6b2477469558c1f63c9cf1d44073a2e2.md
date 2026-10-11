
# 关于在 Ring0 中读取/修改 PEB

> 原文：[`www.pediy.com/kssd/pediy10/71204.html`](https://www.pediy.com/kssd/pediy10/71204.html)

文章很菜，大牛飘过，如有纰漏，欢迎指正。`![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")` `![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")`

上次看到[`bbs.pediy.com/showthread.php?t=69730`](http://bbs.pediy.com/showthread.php?t=69730)中的一些回复，说是在驱动中修改 PEB 是不行的。

我寻思着`NtReadProcessMemory`/`NtWriteProcessMemory`读写用户空间也是`KeStackAttachProcess`到目标进程然后`RtlCopyMemory`的，所以在驱动中，`Attach`到目标进程然后读取/写入 PEB 应该是可行的，许多 Ring3 的程序不就这么做么？

另外对用户空间的操作应该使用`ProbeForRead`/`ProbeForWrite`来检查一下~

本程序省略此步骤。

经过实验，证明是可行的。

代码如下：

```
//===============================

NTKERNELAPI

NTSTATUS

PsLookupProcessByProcessId(
            HANDLE ProcessId,
            PEPROCESS *Process
            );

NTKERNELAPI

VOID

KeAttachProcess (
     PEPROCESS Process
     );

NTKERNELAPI

VOID

KeDetachProcess (
     VOID
     );

typedef struct _PEB_LDR_DATA {
  ULONG                    Length;
  BOOLEAN                  Initialized;
  PVOID                    SsHandle;
  LIST_ENTRY               InLoadOrderModuleList;
  LIST_ENTRY               InMemoryOrderModuleList;
  LIST_ENTRY               InInitializationOrderModuleList;
} PEB_LDR_DATA, *PPEB_LDR_DATA;

typedef struct _LDR_DATA_TABLE_ENTRY {
  LIST_ENTRY InLoadOrderLinks;
  LIST_ENTRY InMemoryOrderLinks;
  LIST_ENTRY InInitializationOrderLinks;
  PVOID DllBase;
  PVOID EntryPoint;
  ULONG SizeOfImage;
  UNICODE_STRING FullDllName;
  UNICODE_STRING BaseDllName;
  ULONG Flags;
  USHORT LoadCount;
  USHORT TlsIndex;
  union {
    LIST_ENTRY HashLinks;
    struct {
      PVOID SectionPointer;
      ULONG CheckSum;
    };
  };
  union {
    struct {
      ULONG TimeDateStamp;
    };
    struct {
      PVOID LoadedImports;
    };
  };
  struct _ACTIVATION_CONTEXT * EntryPointActivationContext;
  PVOID PatchInformation;
} LDR_DATA_TABLE_ENTRY, *PLDR_DATA_TABLE_ENTRY;

NTSTATUS ModifyModuleName()
{
  NTSTATUS status;
  PEPROCESS Process;
  ULONG ulPEB;
  PPEB_LDR_DATA pLdr;
  PLDR_DATA_TABLE_ENTRY pLdt = NULL;
  PLIST_ENTRY pList,pHead;
  wchar_t wszFakePath[] = L"0GiNr.com";

  status = PsLookupProcessByProcessId(
    (HANDLE)1356,
    &Process);//exporer.exe 的 pid

  if ( !NT_SUCCESS(status) ) {
    dprintf("PsLookupProcessByProcessId = 0x%08lX",status);
    return status;
  }

  ObDereferenceObject(Process);

  KeAttachProcess(Process);
  dprintf("Process : 0x%08lX. \n",Process);
  ulPEB = *(ULONG *)((ULONG)Process + 0x1B0);
  dprintf("PEB : 0x%08lX. \n",ulPEB);
  __try {
    pLdr = *(PPEB_LDR_DATA *)(ulPEB + 0x00C);
    dprintf("Ldr : 0x%08lX. \n",pLdr);
    pHead = pLdr->InLoadOrderModuleList.Flink;
    pList = pHead;
    do {
      pLdt = CONTAINING_RECORD(
        pList,
        LDR_DATA_TABLE_ENTRY,
        InLoadOrderLinks);
      if ( pLdt->EntryPoint && 
        pLdt->FullDllName.Buffer &&
        pLdt->BaseDllName.Buffer &&
        pLdt->DllBase )
      {
        dprintf("FullDllName : %S \n",pLdt->FullDllName.Buffer);
        dprintf("BaseDllName : %S \n",pLdt->BaseDllName.Buffer);
        dprintf("DllBase : 0x%08lX. \n",pLdt->DllBase);
        dprintf("DllSize : 0x%08lX. \n",pLdt->SizeOfImage);
        dprintf("--------------");
        //DllBase : 0x7C800000. 
        if ( (ULONG)pLdt->DllBase == 0x7C800000 ) {
          dprintf("Modify.");
          dprintf("--------------");
          wcscpy(pLdt->FullDllName.Buffer,wszFakePath);
          wcscpy(pLdt->BaseDllName.Buffer,wszFakePath);
                                                           //断链也是可以的。
          /*pLdt->InLoadOrderLinks.Blink->Flink =
            pLdt->InLoadOrderLinks.Flink;
          pLdt->InLoadOrderLinks.Flink->Blink =
            pLdt->InLoadOrderLinks.Blink;
          pLdt->InInitializationOrderLinks.Blink->Flink =
            pLdt->InInitializationOrderLinks.Flink;
          pLdt->InInitializationOrderLinks.Flink->Blink =
            pLdt->InInitializationOrderLinks.Blink;
          pLdt->InMemoryOrderLinks.Blink->Flink =
            pLdt->InMemoryOrderLinks.Flink;
          pLdt->InMemoryOrderLinks.Flink->Blink =
            pLdt->InMemoryOrderLinks.Blink;*/
        }
      }
      pList = pList->Flink;
    } while ( pList != pHead );
  } __except(EXCEPTION_EXECUTE_HANDLER) {
    status = GetExceptionCode();
    dprintf("GetExceptionCode() = 0x%08lX. \n",status );
  }

  KeDetachProcess();
  return status;
}
```

`![](img/0919a64d26abc7e8b5dfe4d60261ae3c.jpg)`
