
# [原创]WIN7-X64内核模式下枚举导出表

内核模式下枚举查看导出表需要注意的事项比在用户模式下要考虑更加周详；

X64环境倒不是变化太多，指针定是64位的，数据数据及指针指向的数据类型需要注意；

不费话，直接上源码，`VS2010`+`WDK7600`下编译通过：

```
/// 
/// @file aux_list.c
/// @author cntom
/// @date2015-09-24
/// 

#include <ntddk.h>
#include <WinDef.h>
#include <aux_klib.h>

#pragma comment(lib,"aux_klib.lib")	

#define TAG		'test'
#define PtrFromRva(xbase,rva)	(((PBYTE)xbase)+rva)

// 提供一个Unload函数只是为了
VOID DriverUnload(PDRIVER_OBJECT driver)
{
	// 但是实际上我们什么都不做，只打印一句话:
    // #if DBG
    //    DbgBreakPoint();
    // #endif
	KdPrint(("first: Our driver is unloading…\r\n"));
}

NTSTATUS ExpList(PVOID imagebase)
{
	//PVOID LoadAddress=(PVOID)*(PVOID*)imagebase;
	NTSTATUS status=0;
	PIMAGE_EXPORT_DIRECTORY ExportDirectory;
	PIMAGE_DATA_DIRECTORY ExportDataDirectory;
	PIMAGE_NT_HEADERS NtHeader;
	PIMAGE_DOS_HEADER DosHeader=(PIMAGE_DOS_HEADER)imagebase;
	
	ULONG Index;
	PULONG FuncRvaArray;
	PUSHORT OrdArray;
	PULONG NameArray;

	if (IMAGE_DOS_SIGNATURE!=DosHeader->e_magic)
	{
		KdPrint(("No Dos Header!\n"));
		return status;
	}
	NtHeader=(PIMAGE_NT_HEADERS)PtrFromRva(DosHeader,DosHeader->e_lfanew);
	if (IMAGE_NT_SIGNATURE!=NtHeader->Signature)
	{
		KdPrint(("Not Nt File!\n"));
		return status;
	}
	
	ExportDataDirectory=&NtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT];
	ExportDirectory=(PIMAGE_EXPORT_DIRECTORY)PtrFromRva(imagebase,ExportDataDirectory->VirtualAddress);
	//这上下两个可以对比
	ExportDirectory=AuxKlibGetImageExportDirectory(imagebase);
	
	if (!MmIsAddressValid(ExportDirectory) || ExportDirectory->NumberOfFunctions==0 || ExportDirectory->AddressOfFunctions==0 || ExportDirectory->AddressOfNameOrdinals==0 || ExportDirectory->AddressOfNames==0 )
	{
		KdPrint(("no export....\n"));
		return status;
	}
	
	FuncRvaArray=(PULONG)PtrFromRva(imagebase,ExportDirectory->AddressOfFunctions);
	OrdArray=(PUSHORT)PtrFromRva(imagebase,ExportDirectory->AddressOfNameOrdinals);
	NameArray=(PULONG)PtrFromRva(imagebase,ExportDirectory->AddressOfNames);
	for (Index=0;Index<ExportDirectory->NumberOfNames;Index++)
	{
		USHORT Ordinal=(USHORT)OrdArray[Index]+(USHORT)ExportDirectory->Base;
		ULONG FunRva=FuncRvaArray[Ordinal-ExportDirectory->Base];
		if (FunRva>=ExportDataDirectory->VirtualAddress && FunRva<ExportDataDirectory->VirtualAddress+ExportDataDirectory->Size)
		{
			KdPrint(("It's a forwarders\n"));	
			continue;
		}
		else
		{
			KdPrint(("%4x   %s\n",FunRva,PtrFromRva(imagebase,NameArray[Index])));
		}
	}
	return status;
}

NTSTATUS Aux_Test()
{
	NTSTATUS status;
	ULONG modsize;
	ULONG numofmod;
	AUX_MODULE_EXTENDED_INFO *mod;
	ULONG i;
	
	PVOID imagebase;

	KdPrint(("Aux_Klib testing ......\n\n"));
	status=AuxKlibInitialize();
	if (!NT_SUCCESS(status))
	{
		KdPrint(("AuxKlibInitialize failed!\n"));
		return status;
	}
	status=AuxKlibQueryModuleInformation(&modsize,sizeof(AUX_MODULE_EXTENDED_INFO),NULL);
	if (!NT_SUCCESS(status) || modsize==0)
	{
		KdPrint(("first query failed!\n"));
		return status;
	}
	numofmod=modsize / sizeof(AUX_MODULE_EXTENDED_INFO);
	mod=(AUX_MODULE_EXTENDED_INFO*)ExAllocatePoolWithTag(PagedPool,modsize,TAG);
	if(mod==NULL)
	{
		status=STATUS_INSUFFICIENT_RESOURCES;
		KdPrint(("insufficient resouces!\n"));
		return status;
	}
	RtlZeroMemory(mod,modsize);
	status=AuxKlibQueryModuleInformation(&modsize,sizeof(AUX_MODULE_EXTENDED_INFO),mod);
	if (!NT_SUCCESS(status))
	{
		ExFreePoolWithTag(mod,TAG);
		KdPrint(("Query Failed!\n"));
		return status;
	}
	for (i=0;i<numofmod;i++)
	{
		KdPrint(("Load order:%d\n",i));
		KdPrint(("FileName:%s\n",mod[i].FullPathName+mod[i].FileNameOffset));
		KdPrint(("ImageBase:%p\n",mod[i].BasicInfo.ImageBase));
		KdPrint(("ImageSize:0x%08X\n",mod[i].ImageSize));
		KdPrint(("\n\n"));
		imagebase=mod[i].BasicInfo.ImageBase;	

		/*if (i==128 || i==3)
			DbgBreakPoint();*/

		if (!MmIsAddressValid(imagebase))
			continue;
		ExpList(imagebase);
	}
	ExFreePoolWithTag(mod,TAG);
	return status;
}

// DriverEntry，入口函数。相当于main。
NTSTATUS DriverEntry(PDRIVER_OBJECT driver, PUNICODE_STRING reg_path)
{
	// 设置一个卸载函数便于这个函数能退出。
	driver->DriverUnload = DriverUnload;

     #if DBG
        DbgBreakPoint();
     #endif
	// 这是我们的内核模块的入口，可以在这里写入我们想写的东西。
	// 我在这里打印一句话。因为”Hello,world” 常常被高手耻笑，所以
	// 我们打印一点别的。
	KdPrint(("first: Hello, my salary!\r\n"));
	Aux_Test();

	return STATUS_SUCCESS;
}
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
