
```c
#include <ntifs.h>

#include <intrin.h>



#define PROCESS_VM_READ           (0x0010)  // winnt

#define PROCESS_VM_WRITE          (0x0020)  // winnt





BOOLEAN isThreadWork = TRUE;



struct _HANDLE_TABLE_ENTRY_INFO

{

	ULONG AuditMask;                                                        //0x0

};



typedef struct _HANDLE_TABLE_ENTRY

{

	union

	{

		VOID* Object;                                                       //0x0

		ULONG ObAttributes;                                                 //0x0

		struct _HANDLE_TABLE_ENTRY_INFO* InfoTable;                         //0x0

		ULONG Value;                                                        //0x0

	};

	union

	{

		ULONG GrantedAccess;                                                //0x4

		struct

		{

			USHORT GrantedAccessIndex;                                      //0x4

			USHORT CreatorBackTraceIndex;                                   //0x6

		};

		ULONG NextFreeTableEntry;                                           //0x4

	};

}HANDLE_TABLE_ENTRY, *PHANDLE_TABLE_ENTRY;



typedef BOOLEAN(NTAPI *EX_ENUMERATE_HANDLE_ROUTINE)(

	IN PHANDLE_TABLE_ENTRY HandleTableEntry,

	IN HANDLE Handle,

	IN PVOID EnumParameter

	);



BOOLEAN ExEnumHandleTable(

	__in PVOID HandleTable,

	__in EX_ENUMERATE_HANDLE_ROUTINE EnumHandleProcedure,

	__in PVOID EnumParameter,

	__out_opt PHANDLE Handle

);



PVOID gObject = NULL;



BOOLEAN NTAPI enumRoutine(

	IN PHANDLE_TABLE_ENTRY HandleTableEntry,

	IN HANDLE Handle,

	IN PVOID EnumParameter

)

{

	if (HandleTableEntry)

	{

		

		ULONG_PTR object = (HandleTableEntry->Value & ~7) + 0x18;

		if (object == (ULONG_PTR)EnumParameter)

		{

			

			HandleTableEntry->Value = (ULONG_PTR)gObject | 1;

			HandleTableEntry->GrantedAccess |= PROCESS_ALL_ACCESS;



		}

	}



	return FALSE;

}



PEPROCESS FindProcessByName(PWCH name)

{



	PEPROCESS Process = NULL;

	PEPROCESS findProcess = NULL;

	for (int i = 8; i < 0x1000000; i += 4)

	{

		NTSTATUS status = PsLookupProcessByProcessId((HANDLE)i, &Process);

		if (!NT_SUCCESS(status))

		{

			continue;

		}



		PUNICODE_STRING ProcessName = NULL;

		status = SeLocateProcessImageName(Process, &ProcessName);



		if (!NT_SUCCESS(status))

		{

			ObDereferenceObject(Process);

			continue;

		}







		if (ProcessName->Length)

		{

			_wcsupr(ProcessName->Buffer);



			if (wcsstr(ProcessName->Buffer, name) != 0)

			{

				findProcess = Process;

				ExFreePoolWithTag(ProcessName, 0);

				break;

			}

		}



		ExFreePoolWithTag(ProcessName, 0);

		ObDereferenceObject(Process);

	}



	return findProcess;

}





ULONG GetProcessIdOffset()

{



	UNICODE_STRING unName = { 0 };

	RtlInitUnicodeString(&unName, L"PsGetProcessId");

	PUCHAR startFunc = MmGetSystemRoutineAddress(&unName);



	return *(PULONG)(startFunc + 8);

}



VOID DriverUnload(PDRIVER_OBJECT pDriver)

{



}



NTSTATUS DriverEntry(PDRIVER_OBJECT pDriver, PUNICODE_STRING pReg)

{

	gObject = ExAllocatePool(NonPagedPool, PAGE_SIZE);

	PEPROCESS srcProcess = (PUCHAR)gObject + 0x18;

	memset(gObject, 0, PAGE_SIZE);



	//复制进程

	PEPROCESS TargetProcess = FindProcessByName(L"DBGVIEW.EXE");

	memcpy(gObject, (PUCHAR)TargetProcess - 0x18, 0x300);



	//干掉PID

	*(PHANDLE)((PUCHAR)srcProcess + GetProcessIdOffset()) = 0;



	



	ULONG cr3 = *(PULONG)((PUCHAR)TargetProcess + 0x18);



	PHYSICAL_ADDRESS cr3Phy = {0};

	cr3Phy.QuadPart = cr3;

	ULONG msize = 0x20;  // 29912分页



	if ((cr3  % 0x1000 ) == 0)

	{

		//101012分页

		msize = PAGE_SIZE;

	}



	// WIN10 1803以上，不能映射页表

	//有弊端  如果是小页。在一次映射多页时候 不一定成功，如果是大页一次映射2M 或者4M 必须是页开头，跨页就有问题

	PVOID mem = MmMapIoSpace(cr3Phy, msize, MmNonCached);



	//复制CR3	

	PVOID srcCr3 = (PVOID)ExAllocatePool(NonPagedPool, PAGE_SIZE);

	memset(srcCr3, 0, PAGE_SIZE);

	memcpy(srcCr3, mem, msize);



	//如果不替换CR3 进程退出，或者在某些API的情况会卡死，蓝屏

	PHYSICAL_ADDRESS srcphyCr3 =  MmGetPhysicalAddress(srcCr3);

	//给它一个新的CR3

	*(PULONG)((PUCHAR)srcProcess + 0x18) = srcphyCr3.LowPart;



	PEPROCESS ceProcess = NULL;

	NTSTATUS status  = PsLookupProcessByProcessId(3088, &ceProcess);   //ce进程PID

	if (NT_SUCCESS(status))

	{

		ExEnumHandleTable(*(PULONG)((PUCHAR)ceProcess + 0xf4), enumRoutine, TargetProcess, NULL);

	}



	pDriver->DriverUnload = DriverUnload;

	return STATUS_SUCCESS;

}
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)
