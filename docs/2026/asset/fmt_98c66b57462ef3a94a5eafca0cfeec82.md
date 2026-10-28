
# [原创]通过PsSetLoadImageNotifyRoutine学习模块监控与反模块监控

## 一.实验环境

本文要实现的是对模块加载的监控，并卸载掉目标模块。实验是在WIN 7 X86系统上进行，需要使用的文件是`InjectDll.dll`和`TestDriver.sys`。这两个文件的内容非常简单，`InjectDll.dll`在加载进进程中以后只会弹出加载的对话框，而`TestDriver.sys`只是在驱动加载和卸载的时候打印字符提醒。

## 二.模块监控

### 1.实现原理

在内核中可以通过`PsSetLoadImageNotifyRoutine`来设置模块监控，监控系统中各个应用程序加载的DLL以及系统加载的驱动。在函数在中文档的定义如下

```
NTSTATUS

  PsSetLoadImageNotifyRoutine(

    IN PLOAD_IMAGE_NOTIFY_ROUTINE  NotifyRoutine

    );
```

其中`NotifyRoutine`是一个`LOAD_IMAGE_NOTIFY_ROUTINE`的函数指针，该函数在文档中的定义如下

```
typedef

VOID

(*PLOAD_IMAGE_NOTIFY_ROUTINE)(

    __in PUNICODE_STRING FullImageName,

    __in HANDLE ProcessId,                // pid into which image is being mapped

    __in PIMAGE_INFO ImageInfo

    );
```

|  |  |
| --- | --- |
| 参数 | 说明 |
| FullImageName | 指向模块的完整路径 |
| ProcessId | 加载模块的进程PID。如果是驱动文件，该值为0 |
| ImageInfo | 指向`IMAGE_INFO`结构体，包含了模块信息 |

`IMAGE_INFO`在文档中的定义如下

```
typedef struct _IMAGE_INFO {

    union {

        ULONG Properties;

        struct {

            ULONG ImageAddressingMode  : 8;  // Code addressing mode

            ULONG SystemModeImage      : 1;  // System mode image

            ULONG ImageMappedToAllPids : 1;  // Image mapped into all processes

            ULONG ExtendedInfoPresent  : 1;  // IMAGE_INFO_EX available

            ULONG Reserved             : 21;

        };

    };

    PVOID       ImageBase;

    ULONG       ImageSelector;

    SIZE_T      ImageSize;

    ULONG       ImageSectionNumber;

} IMAGE_INFO, *PIMAGE_INFO;
```

最关键的两个是`ImageBase`和`ImageSize`，分别代表了模块的基地址和模块的大小。

而要卸载模块监控，则需要使用`PsRemoveLoadImageNotifyRoutine`，该函数在文档中的定义如下，参数含义和设置模块监控一样

```
NTSTATUS

  PsRemoveLoadImageNotifyRoutine(

    IN PLOAD_IMAGE_NOTIFY_ROUTINE  NotifyRoutine 

    );
```

驱动的卸载和DLL的卸载是不一样的。

- 对于驱动的卸载，只需要找到驱动入口点，并且直接返回`STATUS_ACCESS_DENIED`(`0xC0000022`)错误码就好。这样，已经加载的驱动就会在执行的时候出错，导致驱动启动失败
- 对于DLL的卸载则需要使用未到出函数`MmUnmapViewOfSection`用来进行卸载。当由于加载进程模块的时候，系统会有一个内部锁，为了避免死锁。要通过创建线程进程延迟等待，等待DLL加载完成以后在调用函数进行卸载

具体代码如下

```
#include <ntifs.h>

#include <ntimage.h>



#define DRIVER_NAME L"TestDriver.sys"	//要拦截的驱动名

#define DLL_NAME L"InjectDll.dll"	//要拦截的DLL名



typedef struct _DLL_INFO

{

	HANDLE ProcessId;

	PVOID pImageBase;

}DLL_INFO, *PDLL_INFO;	//Dll的信息，用来作为线程的参数传递



VOID DriverUnload(IN PDRIVER_OBJECT driverObject);

VOID LoadImageNotifyRoutine(PUNICODE_STRING FullImageName, HANDLE ProcessId, PIMAGE_INFO ImageInfo);	//模块监控回调函数

BOOLEAN DenyLoadDriver(PVOID pLoadImageBase);	//对驱动的加载进行拦截

BOOLEAN DenyLoadDll(HANDLE ProcessId, PVOID pImageBase);	//对DLL的加载进行拦截

NTSTATUS MmUnmapViewOfSection(PEPROCESS Process, PVOID BaseAddr);	//未导出函数声明

VOID ThreadProc(PVOID StartContext);	//运行的线程函数



NTSTATUS DriverEntry(IN PDRIVER_OBJECT driverObject, IN PUNICODE_STRING registryPath)

{

	NTSTATUS status = STATUS_SUCCESS;



	DbgPrint("驱动加载完成\r\n");

	status = PsSetLoadImageNotifyRoutine(LoadImageNotifyRoutine);

	if (!NT_SUCCESS(status))

	{

		DbgPrint("模块监控设置失败 0x%X\r\n", status);

	}

	else

	{

		DbgPrint("模块监控设置成功\r\n");

	}

exit:

	driverObject->DriverUnload = DriverUnload;

	return STATUS_SUCCESS;

}



VOID LoadImageNotifyRoutine(PUNICODE_STRING FullImageName, HANDLE ProcessId, PIMAGE_INFO ImageInfo)

{

	PDLL_INFO pDllInfo = NULL;

	HANDLE hThread = NULL;



	/*

	DbgPrint("===========================================================================\r\n");

	DbgPrint("检测到新加载的模块,模块信息如下:\r\n");

	DbgPrint("加载该模块的进程ID:%d 模块完整名:%wZ 模块基址:0x%X 模块大小:0x%X", ProcessId, FullImageName, ImageInfo->ImageBase, ImageInfo->ImageSize);

	DbgPrint("===========================================================================\r\n");

	*/



	// 是否是exe或者dll文件

	if (ProcessId)

	{

		if (wcsstr(FullImageName->Buffer, DLL_NAME) != NULL)

		{

			pDllInfo = (PDLL_INFO)ExAllocatePool(NonPagedPool, sizeof(DLL_INFO));

			if (!pDllInfo)

			{

				DbgPrint("ExAllocatePool Error");

			}

			else

			{

				pDllInfo->ProcessId = ProcessId;

				pDllInfo->pImageBase = ImageInfo->ImageBase;

				PsCreateSystemThread(&hThread, 0, NULL, NtCurrentProcess(), NULL, ThreadProc, pDllInfo);

				if (hThread) ZwClose(hThread);

			}

		}

	}

	else

	{

		//加载的是驱动,判断是否是要拦截的驱动

		if (wcsstr(FullImageName->Buffer, DRIVER_NAME) != NULL )

		{

			if (DenyLoadDriver(ImageInfo->ImageBase))

			{

				DbgPrint("成功拦截驱动%wZ的加载\r\n", FullImageName);

			}

		}

	}

}



VOID ThreadProc(PVOID StartContext)

{

	PDLL_INFO pDllInfo = (PDLL_INFO)StartContext;

	LARGE_INTEGER liTime = { 0 };



	//延时5秒

	liTime.QuadPart = -50 * 1000 * 1000;

	KeDelayExecutionThread(KernelMode, FALSE, &liTime);

	//卸载DLL

	if (DenyLoadDll(pDllInfo->ProcessId, pDllInfo->pImageBase))

	{

		DbgPrint("Dll卸载完成\r\n");

	}



	if (pDllInfo) ExFreePool(pDllInfo);

}



BOOLEAN DenyLoadDll(HANDLE ProcessId, PVOID pImageBase)

{

	BOOLEAN  bRet = TRUE;

	NTSTATUS status = STATUS_SUCCESS;

	PEPROCESS pEprocess = NULL;	//保存加载DLL的进程的EPROCESS



	//根据进程PID获取EPROCESS

	status = PsLookupProcessByProcessId(ProcessId, &pEprocess);	

	if (!NT_SUCCESS(status))

	{

		DbgPrint("PsLookupProcessByProcessId Error 0x%X", status);

		bRet = FALSE;

		goto exit;

	}



	//卸载模块

	status = MmUnmapViewOfSection(pEprocess, pImageBase);

	if (!NT_SUCCESS(status))

	{

		DbgPrint("MmUnmapViewOfSection Error 0x%X\r\n", status);

		bRet = FALSE;

		goto exit;

	}

exit:

	return bRet;

}



BOOLEAN DenyLoadDriver(PVOID pLoadImageBase)

{

	BOOLEAN bRet = TRUE;

	NTSTATUS status = STATUS_SUCCESS;

	PVOID pVoid = NULL;

	PIMAGE_DOS_HEADER pDosHead = NULL;

	PIMAGE_NT_HEADERS pNtHeader = NULL;

	PVOID pDriverEntry = NULL;

	PMDL pMdl = NULL;

	// 要写入的ShellCode,硬编码的意思是

	// mov eax, 0xC0000022

	// ret

	UCHAR szShellCode[6] = { 0xB8, 0x22, 0x00, 0x00, 0xC0, 0xC3};

	ULONG uShellCodeLength = 6;



	pDosHead = (PIMAGE_DOS_HEADER)pLoadImageBase;

	pNtHeader = (PIMAGE_NT_HEADERS)((ULONG)pLoadImageBase + pDosHead->e_lfanew);

	pDriverEntry = (PVOID)((ULONG)pDosHead + pNtHeader->OptionalHeader.AddressOfEntryPoint);	//获取驱动入口点位置



	//创建MDL并为内存属性添加可写属性

	pMdl = MmCreateMdl(NULL, pDriverEntry, uShellCodeLength);

	if (pMdl == NULL)

	{

		bRet = FALSE;

		goto exit;

	}



	//建立内存页的MDL描述

	MmBuildMdlForNonPagedPool(pMdl);

	//改变MDL的标记为可写

	pMdl->MdlFlags |= MDL_MAPPED_TO_SYSTEM_VA;

	//映射MDL空间

	pVoid = MmMapLockedPages(pMdl, KernelMode);

	//将shellcode拷到目标地址

	RtlCopyMemory(pVoid, szShellCode, uShellCodeLength);

	//释放MDL

	MmUnmapLockedPages(pVoid, pMdl);

	IoFreeMdl(pMdl);

	pMdl = NULL;

exit:

	return bRet;

}



VOID DriverUnload(IN PDRIVER_OBJECT driverObject)

{

	NTSTATUS status = STATUS_SUCCESS;



	status = PsRemoveLoadImageNotifyRoutine(LoadImageNotifyRoutine);

	if (!NT_SUCCESS(status))

	{

		DbgPrint("模块监控删除失败 0x%X\r\n", status);

	}

	else

	{

		DbgPrint("模块监控删除成功\r\n");

	}

	DbgPrint("驱动卸载完成\r\n");

}
```

### 2.实验结果

对于DLL加载来说，当没有开起监控程序卸载的时候，在加载DLL文件以后可以在监控软件中查找到对应的DLL

![](https://bbs.kanxue.com/upload/attach/202110/835440_9ZHBGNMCR53JP4D.jpg)

![](https://bbs.kanxue.com/upload/attach/202110/835440_AKBX78WP5GVHB6W.jpg)

而当开起监控完成DLL卸载以后，就没办法找到相应的DLL了

![](https://bbs.kanxue.com/upload/attach/202110/835440_8P3XF4CBWPFN5AT.jpg)

![](https://bbs.kanxue.com/upload/attach/202110/835440_F3N87F42RPCD9C7.jpg)

对于驱动来说，当没有开起监控的时候，可以正常的加载与卸载

![](https://bbs.kanxue.com/upload/attach/202110/835440_ARYGFGUEBF5WVYE.jpg)

而开起监控以后，驱动的加载就会被拦截

![](https://bbs.kanxue.com/upload/attach/202110/835440_7X65BR7T7QJNZ4T.jpg)

## 三.反模块监控

### 1.逆向分析PsSetLoadImageNotifyRoutine

`PsSetLoadImageNotifyRoutine`在`IDA`中的反汇编代码如下
