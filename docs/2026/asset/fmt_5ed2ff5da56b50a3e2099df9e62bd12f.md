
# [原创]SSDT HOOK及键盘过滤驱动

SSDT全称为`System Service Descriptor Table`，即系统服务描述符表。SSDT的作用就是把Ring3的WIN32 API函数和Ring0的内核API联系起来。R3下的一些API最终会对应`ntdll.dll`中的一个`Nt`开头的函数。将系统服务号放入`EAX`然后调用`KiSystemService`进入内核，根据系统服务号得到对应系统服务的内核地址。HIPS，杀毒软件往往会通过SSDT HOOK实现监控系统的目的。

32位系统中SSDT是由内核`Ntoskrnl.exe`导出的一张表，导出符号为`KeServiceDescriptorTable`。在64位系统上是不导出的。

以下程序实现将`ntdll.dll`文件映射到内核空间，并从导出表中获取导出函数地址，然后获取SSDT函数索引号。

具体操作流程如下：

1. 调用`InitializeObjectAttributes`宏初始化文件对象，它主要初始化文件路径，内核下的文件路径需要在路径前面加上`\??\`。

2. 调用`ZwCreateSection`函数生成一个内存映射对象得到句柄`hSection`，该对象建立在由`ZwOpenFile`函数创建的文件对象基础上。该内存映射对象对视图读写具有`SECTION_MAP_READ`和`SECTION_MAP_WRITE`访问权限，同时设置内存映射方式为`SEC_IMAGE`，以PE中的`SectionAlignment`大小对齐映射文件。

3. 调用`ZwMapViewOfSection`函数把文件映射到内存得到`lpMemory`。该指针直接读写内存。

4. 调用`ZwUnmapViewOfSection`解除文件映射，进程句柄、`lpMemory`作为参数。

5. 接下来根据PE文件结构获取导出函数地址，并从导出函数中获取SSDT函数索引号。根据`IMAGE_DOS_HEADER`和`IMAGE_NT_HEADERS`计算出`OptionalHeader`，获取数据目录中导出表的RVA地址，查找匹配的函数名称，从`AddressOfNamesOrdinal`中获取索引值。32位系统下导出函数偏移1字节后的四字节就是SSDT函数索引号，64位系统下需要偏移4字节。

```
#include "SSDTFunctionIndex.h"
// 从 ntdll.dll 中获取 SSDT 函数索引号
ULONG GetSSDTFunctionIndex(UNICODE_STRING ustrDllFileName, PCHAR pszFunctionName)
{
ULONG ulFunctionIndex = 0;
NTSTATUS status = STATUS_SUCCESS;
HANDLE hFile = NULL;
HANDLE hSection = NULL;
PVOID pBaseAddress = NULL;
// 内存映射文件
status = DllFileMap(ustrDllFileName, &hFile, &hSection, &pBaseAddress);
if (!NT_SUCCESS(status))
{
KdPrint(("DllFileMap Error!\n"));
return ulFunctionIndex;
}
// 根据导出表获取导出函数地址, 从而获取 SSDT 函数索引号
ulFunctionIndex = GetIndexFromExportTable(pBaseAddress, pszFunctionName);
// 释放
ZwUnmapViewOfSection(NtCurrentProcess(), pBaseAddress);
ZwClose(hSection);
ZwClose(hFile);
return ulFunctionIndex;
}
// 内存映射文件
NTSTATUS DllFileMap(UNICODE_STRING ustrDllFileName, HANDLE *phFile, HANDLE *phSection, PVOID *ppBaseAddress)
{
NTSTATUS status = STATUS_SUCCESS;
HANDLE hFile = NULL;
HANDLE hSection = NULL;
OBJECT_ATTRIBUTES objectAttributes = { 0 };
IO_STATUS_BLOCK iosb = { 0 };
PVOID pBaseAddress = NULL;
SIZE_T viewSize = 0;
// 打开 DLL 文件, 并获取文件句柄
InitializeObjectAttributes(&objectAttributes, &ustrDllFileName, OBJ_CASE_INSENSITIVE | OBJ_KERNEL_HANDLE, NULL, NULL);
status = ZwOpenFile(&hFile, GENERIC_READ, &objectAttributes, &iosb,
FILE_SHARE_READ, FILE_SYNCHRONOUS_IO_NONALERT);
if (!NT_SUCCESS(status))
{
KdPrint(("ZwOpenFile Error! [error code: 0x%X]", status));
return status;
}
// 创建一个节对象, 以 PE 结构中的 SectionALignment 大小对齐映射文件
status = ZwCreateSection(&hSection, SECTION_MAP_READ | SECTION_MAP_WRITE, NULL, 0, PAGE_READWRITE, 0x1000000, hFile);
if (!NT_SUCCESS(status))
{
ZwClose(hFile);
KdPrint(("ZwCreateSection Error! [error code: 0x%X]", status));
return status;
}
// 映射到内存
status = ZwMapViewOfSection(hSection, NtCurrentProcess(), &pBaseAddress, 0, 1024, 0, &viewSize, ViewShare, MEM_TOP_DOWN, PAGE_READWRITE);
if (!NT_SUCCESS(status))
{
ZwClose(hSection);
ZwClose(hFile);
KdPrint(("ZwMapViewOfSection Error! [error code: 0x%X]", status));
return status;
}
// 返回数据
*phFile = hFile;
*phSection = hSection;
*ppBaseAddress = pBaseAddress;
return status;
}
// 根据导出表获取导出函数地址, 从而获取 SSDT 函数索引号
ULONG GetIndexFromExportTable(PVOID pBaseAddress, PCHAR pszFunctionName)
{
ULONG ulFunctionIndex = 0;
// Dos Header
PIMAGE_DOS_HEADER pDosHeader = (PIMAGE_DOS_HEADER)pBaseAddress;
// NT Header
PIMAGE_NT_HEADERS pNtHeaders = (PIMAGE_NT_HEADERS)((PUCHAR)pDosHeader + pDosHeader->e_lfanew);
// Export Table
PIMAGE_EXPORT_DIRECTORY pExportTable = (PIMAGE_EXPORT_DIRECTORY)((PUCHAR)pDosHeader + pNtHeaders->OptionalHeader.DataDirectory[0].VirtualAddress);
// 有名称的导出函数个数
ULONG ulNumberOfNames = pExportTable->NumberOfNames;
// 导出函数名称地址表
PULONG lpNameArray = (PULONG)((PUCHAR)pDosHeader + pExportTable->AddressOfNames);
PCHAR lpName = NULL;
// 开始遍历导出表
for (ULONG i = 0; i < ulNumberOfNames; i++)
{
lpName = (PCHAR)((PUCHAR)pDosHeader + lpNameArray[i]);
// 判断是否查找的函数
if (0 == _strnicmp(pszFunctionName, lpName, strlen(pszFunctionName)))
{
// 获取导出函数地址
USHORT uHint = *(USHORT *)((PUCHAR)pDosHeader + pExportTable->AddressOfNameOrdinals + 2 * i);
ULONG ulFuncAddr = *(PULONG)((PUCHAR)pDosHeader + pExportTable->AddressOfFunctions + 4 * uHint);
PVOID lpFuncAddr = (PVOID)((PUCHAR)pDosHeader + ulFuncAddr);
// 获取 SSDT 函数 Index
#ifdef _WIN64
ulFunctionIndex = *(ULONG *)((PUCHAR)lpFuncAddr + 4);
#else
ulFunctionIndex = *(ULONG *)((PUCHAR)lpFuncAddr + 1);
#endif
break;
}
}
return ulFunctionIndex;
}
```

## 获取SSDT函数地址

32位系统上获取SSDT地址，直接获取`Ntoskrnl.exe`导出符号`KeServiceDescriptorTable`即可。
