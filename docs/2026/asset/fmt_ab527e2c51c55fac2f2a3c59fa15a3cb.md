
An asynchronous procedure call (APC) is a function that executes asynchronously. APCs are similar to deferred procedure calls (DPCs), but unlike DPCs, APCs execute within the context of a particular thread. Drivers (other than file systems and file-system filter drivers) do not use APCs directly, but other parts of the operating system do, so you need to be aware of how APCs work.

The Windows operating system uses three kinds of APCs:

1. User APCs run strictly in user mode and only when the current thread is in an alertable wait state. The operating system uses user APCs to implement mechanisms such as overlapped I/O and the `QueueUserApc` Win32 routine.

2. Normal kernel APCs run in kernel mode at `IRQL = PASSIVE_LEVEL`. A normal kernel APC preempts all user-mode code, including user APCs. Normal kernel APCs are generally used by file systems and file-system filter drivers.

3. Special kernel APCs run in kernel mode at `IRQL = APC_LEVEL`. A special kernel APC preempts user-mode code and kernel-mode code that executes at `IRQL = PASSIVE_LEVEL`, including both user APCs and normal kernel APCs. The operating system uses special kernel APCs to handle operations such as I/O request completion.

其实总结一下很简单：一个用户模式的 APC，一个内核模式的 APC（分为 `NormalRoutine` 为 `NULL` 的 `SpecialKernelApc` 和不为 `NULL` 的 `NormalKernelApc`）。

区别在于：`SpecialKernelApc` 只执行 `KernelRoutine`，IRQL 为 `APC_LEVEL`；而 `NormalKernelApc` 不仅仅执行 `KernelRoutine`，还执行 `NormalRoutine`，在 `PASSIVE_LEVEL` 下执行 `NormalRoutine`。

但是 I/O API 也没有说明它内部用的是哪个类型的 Kernel APC，要稳一点就判断总开关 `SpecialApcDisable`。不过一般 MSDN 都会说明。

## 3. `KeAreApcsDisabled`/`KeAreAllApcsDisabled`

```
BOOLEAN

NTAPI

KeAreApcsDisabled(VOID)

{

    /* Return the Kernel APC State */

    return KeGetCurrentThread()->CombinedApcDisable ? TRUE : FALSE;

}



BOOLEAN

NTAPI

KeAreAllApcsDisabled(VOID)

{

    /* Return the Special APC State */

    return ((KeGetCurrentThread()->SpecialApcDisable) ||

            (KeGetCurrentIrql() >= APC_LEVEL)) ? TRUE : FALSE;

}



typedef struct _KTHREAD

{

......

    union

    {

        struct

        {

            SHORT KernelApcDisable;

            SHORT SpecialApcDisable;

        };

        ULONG CombinedApcDisable;

    };

......

};
```

主要看这两个 API 有什么区别，可能很多人看不出来什么区别……

我也理解了很久。`KeAreApcsDisabled` 是说只要当前在内核临界区内就是 `Disable` 状态，这个可以是子开关 `KernelApcDisable` 或者是总开关 `SpecialApcDisable` 至少一个有值：要是 `SpecialApcDisable`，则就是所有 APC 都是无效状态（与 `KeAreAllApcsDisabled` 判断相同）；要是 `KernelApcDisable`，就是 Normal KernelApc 失效；要是用的 `KeEnterCriticalRegion`，就只能用这个函数检查，一般用这个就可以了，当然最好 `if (KeAreApcsDisabled() || __readcr8() == APC_LEVEL)`。

而 `KeAreAllApcsDisabled` 则是真正意义上的所有 APC 都无效，但是对于 `KeEnterCriticalRegion` 的临界区，这个 API 是无法判断的。

## 4. 解决办法

当出现无法调用 I/O API 的时候，建议使用劳务线程，这个线程的执行环境还是比较稳定的，而且就算是做同步响应也不会耗时很久。例如：

```
typedef struct tag_FyWorkQueueItem

{

	WORK_QUEUE_ITEM WorkQueueItem;

	PVOID  lpParameter1;

	PVOID  lpParameter2;

	PVOID  lpParameter3;

	KEVENT CompleteEvent;

	BOOL   bStatus;

} FyWorkQueueItem, *PFyWorkQueueItem;



PUNICODE_STRING QueryProcessObjectName(IN HANDLE ProcessId)

{

	NTSTATUS Status = STATUS_SUCCESS;

	PEPROCESS EProcess = NULL;

	HANDLE hProcess = NULL;

	ULONG ulRealSize = 0;

	PUNICODE_STRING lpuniImageFileName = NULL;

	BOOL bSuccess = FALSE;



	if (KeGetCurrentIrql() <= APC_LEVEL)

	{

		Status = PsLookupProcessByProcessId(ProcessId, &EProcess);

		if (NT_SUCCESS(Status) && EProcess)

		{

			Status = ObOpenObjectByPointer((PVOID)EProcess, OBJ_KERNEL_HANDLE, NULL, 

				PROCESS_ALL_ACCESS, NULL, KernelMode, &hProcess);



			if (NT_SUCCESS(Status))

			{

				Status = ZwQueryInformationProcess(hProcess, ProcessImageFileName, NULL, 0, &ulRealSize);

				if (Status == STATUS_INFO_LENGTH_MISMATCH)

				{

					lpuniImageFileName = (PUNICODE_STRING)ExAllocatePoolWithTag(NonPagedPool, 

						ulRealSize + sizeof(UNICODE_STRING), 'hiti');



					if (lpuniImageFileName)

					{

						memset(lpuniImageFileName, 0, ulRealSize + sizeof(UNICODE_STRING));

						Status = ZwQueryInformationProcess(hProcess, ProcessImageFileName, 

							lpuniImageFileName, ulRealSize + sizeof(UNICODE_STRING), &ulRealSize);



						if (NT_SUCCESS(Status))

						{

							bSuccess = TRUE;

						}

					}

				}

				ZwClose(hProcess);

			}

			ObDereferenceObject(EProcess);

		}

	}



	if (!bSuccess)

	{

		ExFreePool(lpuniImageFileName);

		lpuniImageFileName = NULL;

	}



	return lpuniImageFileName;

}



BOOL GetProcessImageFileName(

	IN HANDLE ProcessId, 

	OUT WCHAR* lpwzImageFileName, 

	IN ULONG uMaxSize)

{

	NTSTATUS                   Status = STATUS_SUCCESS;

	HANDLE                     FileHandle = NULL;

	IO_STATUS_BLOCK            IoStatusBlock = { 0 };

	PUNICODE_STRING            lpuniProcessObjectName = NULL;

	OBJECT_ATTRIBUTES          oa = { 0 };

	PFILE_OBJECT               FileObject = NULL;

	POBJECT_NAME_INFORMATION   ObjectNameInformation = NULL;

	BOOL                       bStatus = FALSE;



	if (KeGetCurrentIrql() > PASSIVE_LEVEL) {

		return FALSE;

	}



	lpuniProcessObjectName = QueryProcessObjectName(ProcessId);

	if (!lpuniProcessObjectName) {

		return FALSE;

	}



	InitializeObjectAttributes(&oa, lpuniProcessObjectName, OBJ_KERNEL_HANDLE | OBJ_CASE_INSENSITIVE, NULL, NULL);

	Status = IoCreateFile(

		&FileHandle,

		FILE_READ_ATTRIBUTES,

		&oa,

		&IoStatusBlock,

		NULL,

		FILE_ATTRIBUTE_NORMAL,

		FILE_SHARE_READ | FILE_SHARE_WRITE,

		FILE_OPEN,

		FILE_NON_DIRECTORY_FILE,

		NULL,

		0,

		CreateFileTypeNone,

		NULL,

		IO_NO_PARAMETER_CHECKING);



	if (!NT_SUCCESS(Status))

	{

		ExFreePool(lpuniProcessObjectName);

		return FALSE;

	}



	Status = ObReferenceObjectByHandle(FileHandle, FILE_ANY_ACCESS, *IoFileObjectType, 

		KernelMode, (PVOID*)&FileObject, NULL);



	if (NT_SUCCESS(Status) && FileObject)

	{

		Status = IoQueryFileDosDeviceName(FileObject, &ObjectNameInformation);

		if (NT_SUCCESS(Status))

		{

			if (ObjectNameInformation)

			{

				if (ObjectNameInformation->Name.Length <= sizeof(WCHAR) * uMaxSize)

				{

					memset(lpwzImageFileName, 0, 2 * uMaxSize);

					memcpy(lpwzImageFileName, ObjectNameInformation->Name.Buffer, 

						ObjectNameInformation->Name.Length);



					bStatus = TRUE;

				}

				ExFreePool(ObjectNameInformation);

				ObjectNameInformation = NULL;

			}

		}

		ObDereferenceObject(FileObject);

	}



	ObCloseHandle(FileHandle, KernelMode);

	FileHandle = NULL;

	ExFreePool(lpuniProcessObjectName);



	return bStatus;

}



VOID QueryProcessFileNameWorkItem(IN PFyWorkQueueItem lpFyWorkQueueItem)

{

	lpFyWorkQueueItem->bStatus = GetProcessImageFileName(

		(HANDLE)lpFyWorkQueueItem->lpParameter1,

		(WCHAR*)lpFyWorkQueueItem->lpParameter2,

		(ULONG)lpFyWorkQueueItem->lpParameter3);



	KeSetEvent(&lpFyWorkQueueItem->CompleteEvent, IO_NO_INCREMENT, FALSE);

}



BOOL GetProcessImageFileNameSafeIrql(

	IN HANDLE ProcessId, 

	OUT WCHAR* lpwzImageFileName, 

	IN ULONG uMaxSize)

{

	BOOL            bStatus;

	FyWorkQueueItem WorkItem;



	if (KeGetCurrentIrql() <= APC_LEVEL)

	{

		if (KeAreApcsDisabled() || KeGetCurrentIrql() == APC_LEVEL)

		{

			memset(&WorkItem, 0, sizeof(WorkItem));

			KeInitializeEvent(&WorkItem.CompleteEvent, NotificationEvent, FALSE);



			WorkItem.bStatus = FALSE;

			WorkItem.WorkQueueItem.List.Flink = NULL;

			WorkItem.WorkQueueItem.WorkerRoutine = (PWORKER_THREAD_ROUTINE)QueryProcessFileNameWorkItem;

			WorkItem.lpParameter1 = (PVOID)ProcessId;

			WorkItem.lpParameter2 = (PVOID)lpwzImageFileName;

			WorkItem.lpParameter3 = (PVOID)uMaxSize;

			WorkItem.WorkQueueItem.Parameter = &WorkItem;

			ExQueueWorkItem(&WorkItem.WorkQueueItem, DelayedWorkQueue);



			KeWaitForSingleObject(&WorkItem.CompleteEvent, Executive, KernelMode, FALSE, NULL);

			bStatus = WorkItem.bStatus;

		}

		else

		{

			bStatus = GetProcessImageFileName(ProcessId, lpwzImageFileName, uMaxSize);

		}

	}

	else

	{

		bStatus = FALSE;

	}



	return bStatus;

}
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
