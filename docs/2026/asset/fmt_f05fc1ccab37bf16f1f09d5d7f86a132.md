
# 一、通过 `PspTerminateProcess` 查找并调用
```
#include <ntifs.h>



typedef struct _LDR_DATA_TABLE_ENTRY

{

	LIST_ENTRY InLoadOrderLinks;

	LIST_ENTRY InMemoryOrderLinks;

	LIST_ENTRY InInitializationOrderLinks;

	PVOID DllBase;

	PVOID EntryPoint;

	UINT32 SizeOfImage;

	UNICODE_STRING FullDllName;

	UNICODE_STRING BaseDllName;

	UINT32 Flags;

	UINT16 LoadCount;

	UINT16 TlsIndex;

	LIST_ENTRY HashLinks;

	PVOID SectionPointer;

	UINT32 CheckSum;

	UINT32 TimeDateStamp;

	PVOID LoadedImports;

	PVOID EntryPointActivationContext;

	PVOID PatchInformation;

} LDR_DATA_TABLE_ENTRY, *PLDR_DATA_TABLE_ENTRY;
typedef NTSTATUS (*pfnPspTerminateProcess)(PEPROCESS pEprocess, NTSTATUS ExitCode);



VOID DriverUnload(IN PDRIVER_OBJECT driverObject);
PVOID SearchFunction(PUCHAR DllBase, UINT32 SizeOfImage);	// 查找模块中匹配特征码



ULONG g_uPID = 1700;	// 要关闭的进程PID
// 特征码
UCHAR g_szSpecialCode[] = {0x56, 0x64, 0xA1, 0x24, 0x01, 0x00, 0x00, 0x8B, 
	0x75, 0x08, 0x3B, 0x70, 0x44, 0x75, 0x07, 0xB8, 
	0x0D, 0x00, 0x00, 0xC0, 0xEB, 0x5A, 0x57 };
UINT32 g_uSpecialCodeLen = sizeof(g_szSpecialCode);



NTSTATUS DriverEntry(IN PDRIVER_OBJECT driverObject, IN PUNICODE_STRING registryPath)

{

	NTSTATUS status = STATUS_SUCCESS;

	PLDR_DATA_TABLE_ENTRY pLdrDataTableEntry = NULL;

	pfnPspTerminateProcess pPspTerminateProcess = NULL;

	PEPROCESS pEprocess = NULL;	

	

	DbgPrint("驱动加载完成\r\n");

	

	pLdrDataTableEntry = (PLDR_DATA_TABLE_ENTRY)driverObject->DriverSection;

	pLdrDataTableEntry = CONTAINING_RECORD(pLdrDataTableEntry, LDR_DATA_TABLE_ENTRY, InLoadOrderLinks);

	pLdrDataTableEntry = (PLDR_DATA_TABLE_ENTRY)pLdrDataTableEntry->InLoadOrderLinks.Flink;

	do 

	{

		pLdrDataTableEntry = CONTAINING_RECORD(pLdrDataTableEntry, LDR_DATA_TABLE_ENTRY, InLoadOrderLinks);

		if (pLdrDataTableEntry->DllBase)

		{

			pPspTerminateProcess = (pfnPspTerminateProcess)SearchFunction((PUCHAR)pLdrDataTableEntry->DllBase, pLdrDataTableEntry->SizeOfImage);

		}

		if (pPspTerminateProcess)	break;

		pLdrDataTableEntry = (PLDR_DATA_TABLE_ENTRY)pLdrDataTableEntry->InLoadOrderLinks.Flink;

	} while ((UINT32)pLdrDataTableEntry != (UINT32)driverObject->DriverSection);



	if (pPspTerminateProcess)

	{

		status = PsLookupProcessByProcessId((HANDLE)g_uPID, &pEprocess);

		if (NT_SUCCESS(status))

		{

			status = pPspTerminateProcess(pEprocess, 0);

			if (NT_SUCCESS(status))

			{

				DbgPrint("关闭进程成功, PID=%d\r\n", g_uPID);

			}

		}

		else

		{

			DbgPrint("PsLookupProcessByProcessId Error 0x%X\r\n", status);

		}

	}



	driverObject->DriverUnload = DriverUnload;

exit:
	return STATUS_SUCCESS;
}



VOID DriverUnload(IN PDRIVER_OBJECT driverObject)

{

	DbgPrint("驱动卸载完成\r\n");

}



PVOID SearchFunction(PUCHAR DllBase, UINT32 SizeOfImage)

{

	PVOID pFuncAddr = NULL;

	UINT32 uEnd = (UINT32)DllBase + SizeOfImage - g_uSpecialCodeLen, i = 0;

	BOOLEAN bOk = TRUE;

	

	while ((UINT32)DllBase <= uEnd)

	{

		bOk = TRUE;

		for (i = 0; i < g_uSpecialCodeLen; i++)

		{

			if (!MmIsAddressValid(&DllBase[i]) || DllBase[i] != g_szSpecialCode[i])

			{

				bOk = FALSE;

				break;

			}

		}



		if (bOk)

		{

			pFuncAddr = (PVOID)(DllBase - 5);	

			break;

		}

		DllBase++;

	}



	return pFuncAddr;
}
```

# 三.通过 `PspTerminateThreadByPointer` 函数来关闭进程

想要关闭进程，可以通过将进程中的所有线程全部关闭。因为当这个进程中的所有线程都被关闭的时候，这个进程就已经没有意义，等同于被关闭。Windows为程序员提供了一个导出函数 `PsTerminateSystemThread` 来关闭线程，但由于这个函数往往会被安全软件监控，所以这里选择更为底层的函数，也就是 `PspTerminateThreadByPointer` 函数来关闭进程。

该函数的定义如下
```
NTSTATUS 
PspTerminateThreadByPointer(PETHREAD pEThread,
                            NTSTATUS ntExitCode,
                            BOOLEAN bDirectTerminate);
```

这个函数在32位和64位中的区别仅仅是调用约定的不同
```
// 64 位
typedef NTSTATUS(__fastcall *pfnPspTerminateThreadByPointer) (PETHREAD pEThread, NTSTATUS ntExitCode, BOOLEAN bDirectTerminate);

// 32 位
typedef NTSTATUS(*pfnPspTerminateThreadByPointer) (PETHREAD pEThread, NTSTATUS ntExitCode, BOOLEAN bDirectTerminate);
```

要找到这个函数，这里就通过找到导出函数 `PsTerminateSystemThread`，并通过特征码的方式找到目标函数。`PsTerminateSystemThread` 函数的实现如下
```
kd> u PsTerminateSystemThread L10
nt!PsTerminateSystemThread:
805d3594 8bff            mov     edi,edi
805d3596 55              push    ebp
805d3597 8bec            mov     ebp,esp
805d3599 64a124010000    mov     eax,dword ptr fs:[00000124h]
805d359f f6804802000010  test    byte ptr [eax+248h],10h
805d35a6 7507            jne     nt!PsTerminateSystemThread+0x1b (805d35af)
805d35a8 b80d0000c0      mov     eax,0C000000Dh
805d35ad eb09            jmp     nt!PsTerminateSystemThread+0x24 (805d35b8)
805d35af ff7508          push    dword ptr [ebp+8]
805d35b2 50              push    eax
805d35b3 e828fcffff      call    nt!PspTerminateThreadByPointer (805d31e0)
805d35b8 5d              pop     ebp
805d35b9 c20400          ret     4
```

由上可知，选择 `0xE8` 作为特征码就可以找到目标函数。有了这个函数，就可以关闭线程。接下来只需要遍历所有的线程判断该线程是不是属于要关闭的进程，如果是的话就把它关掉。

具体代码如下
```
#include <ntifs.h>



typedef NTSTATUS(*pfnPspTerminateThreadByPointer)(PETHREAD pEThread, NTSTATUS ntExitCode, BOOLEAN bDirectTerminate);



VOID DriverUnload(IN PDRIVER_OBJECT driverObject);
PVOID FindTargetFunc();	// 寻找函数地址
PEPROCESS PsGetThreadProcess(PETHREAD pEThraed);	// 根据EPROCESS得到EPROCESS



ULONG g_uPID = 176;	// 要关闭的进程PID



NTSTATUS DriverEntry(IN PDRIVER_OBJECT driverObject, IN PUNICODE_STRING registryPath)

{

	NTSTATUS status = STATUS_SUCCESS;

	PETHREAD pEThread = NULL;

	PEPROCESS pEProcess = NULL, pThreadProcess = NULL;

	ULONG uThreadId = 0;

	pfnPspTerminateThreadByPointer PspTerminateThreadByPointer = NULL;

	PspTerminateThreadByPointer = (pfnPspTerminateThreadByPointer)FindTargetFunc();



	if (PspTerminateThreadByPointer == NULL)

	{

		goto exit;

	}



	//获取要关闭的进程的EPROCESS
	status = PsLookupProcessByProcessId((HANDLE)g_uPID, &pEProcess);

	if (!NT_SUCCESS(status))

	{

		DbgPrint("PsLookupProcessByProcessId Error 0x%X\r\n", status);

		goto exit;

	}



	// 遍历所有线程
	for (uThreadId = 4; uThreadId < 0x8000; uThreadId += 4)

	{

		status = PsLookupThreadByThreadId((HANDLE)uThreadId, &pEThread);

		if (NT_SUCCESS(status))

		{

			// 获取线程对应的进程结构对象
			pThreadProcess = PsGetThreadProcess(pEThread);

			if (pThreadProcess == pEProcess)

			{

				PspTerminateThreadByPointer(pEThread, 0, 1);

				DbgPrint("成功关闭线程\r\n");

			}

			ObDereferenceObject(pEThread);

		}

	}

	ObDereferenceObject(pEProcess);

	driverObject->DriverUnload = DriverUnload;

exit:
	return STATUS_SUCCESS;
}



VOID DriverUnload(IN PDRIVER_OBJECT driverObject)

{

	DbgPrint("驱动卸载完成\r\n");

}



PVOID FindTargetFunc()

{

	PVOID pFunAddr = NULL;

	PUCHAR pPsTerminateSystemThreadAddr = NULL;

	UNICODE_STRING uStrFuncName = RTL_CONSTANT_STRING(L"PsTerminateSystemThread");



	pPsTerminateSystemThreadAddr = (PUCHAR)MmGetSystemRoutineAddress(&uStrFuncName);

	while (MmIsAddressValid(pPsTerminateSystemThreadAddr) && *pPsTerminateSystemThreadAddr != 0xC2)

	{

		if (*pPsTerminateSystemThreadAddr == 0xE8)

		{

			pFunAddr = (PVOID)((ULONG)pPsTerminateSystemThreadAddr + 5 + *(PULONG)(pPsTerminateSystemThreadAddr + 1));

			break;

		}

		pPsTerminateSystemThreadAddr++;

	}

	return pFunAddr;
}
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2021-11-30 10:03

被1900编辑，原因：

[#驱动开发](https://bbs.kanxue.com/forum-41-1-132.htm)
