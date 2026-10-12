
# 【半原创】贴点内核态中创建用户态进程的代码

> 原文：[`www.pediy.com/kssd/pediy10/80179.html`](https://www.pediy.com/kssd/pediy10/80179.html)

自己编写进线程、文件系统等相关函数，并在内核中实现虽然没什么应用价值，不过对内核的学习大有裨益。

刚想逆`CreateProcess`，发现`Gary Nebbett`大牛已经有了相关代码。那就用大牛的代码改进改进。

下面是驱动中创建进程的代码，不知道前人有发过没（论坛上有一份，我看了，那是插`apc`的取巧方式）

本代码大部分是搬`Gary Nebbett`的，不过为了移植到内核中我还是做了不少工作……

另外还有个驱动中创建用户态线程的函数，觉得太简单了，就不贴了![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")

（注意，代码中的写入用户环境部分我是直接把用户态下的`GetEnvironmentStringsW`得到的东西直接贴进去了，所以大家要用的时候自己改改吧。）

大牛就不用看了，自己写写抄抄的而已。![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")

另外还想请教大牛们，为什么有时候启动的时候会出现`STATUS_DLL_INIT_FAILED`错误？？

主函数：

```c
int MyCreateProcess(PUNICODE_STRING name, PWSTR param)
{
    //write by weolar(http://hi.baidu.com/weolar)
    PWSTR tmp;
    HANDLE hProcess, hThread, hSection, hFile;
    OBJECT_ATTRIBUTES oa;
    RtlZeroMemory(&oa, sizeof(OBJECT_ATTRIBUTES)) ;
    InitializeObjectAttributes( &oa,name, OBJ_CASE_INSENSITIVE, 0, NULL);
    IO_STATUS_BLOCK iosb;
    ZwOpenFile(&hFile,FILE_READ_DATA|FILE_EXECUTE|SYNCHRONIZE, &oa, &iosb,
               FILE_SHARE_READ, FILE_SYNCHRONOUS_IO_NONALERT);
    oa.ObjectName = 0;
    ZwCreateSection(&hSection, SECTION_ALL_ACCESS, 0, 0, PAGE_EXECUTE, SEC_IMAGE, hFile);
    ZwClose(hFile);
    ZwCreateProcess(&hProcess, PROCESS_ALL_ACCESS, 0,
                    (HANDLE)-1, TRUE, hSection, 0, 0);
    CloseHandle(hep);
    SECTION_IMAGE_INFORMATION sii;
    ZwQuerySection(hSection, SectionImageInformation, &sii, sizeof sii, 0);
    ZwClose(hSection);
    USER_STACK stack = {0};
    ULONG n = sii.StackReserve;
    ZwAllocateVirtualMemory(hProcess, &stack.ExpandableStackBottom, 0, &n,
                            MEM_RESERVE, PAGE_READWRITE);
    stack.ExpandableStackBase = PCHAR(stack.ExpandableStackBottom) + sii.StackReserve;
    stack.ExpandableStackLimit = PCHAR(stack.ExpandableStackBase) - sii.StackCommit;
    n = sii.StackCommit + PAGE_SIZE;
    PVOID p = PCHAR(stack.ExpandableStackBase) - n;
    ZwAllocateVirtualMemory(hProcess, &p, 0, &n, MEM_COMMIT, PAGE_READWRITE);
    ULONG x; n = PAGE_SIZE;
    ZwProtectVirtualMemory(hProcess, &p, &n, PAGE_READWRITE | PAGE_GUARD, &x);
    CONTEXT context = {CONTEXT_FULL};
    context.SegGs = 0;
    context.SegFs = 0x38;
    context.SegEs = 0x20;
    context.SegDs = 0x20;
    context.SegSs = 0x20;
    context.SegCs = 0x18;
    context.EFlags = 0x3000;
    context.Esp = ULONG(stack.ExpandableStackBase) - 4;
    context.Eip = ULONG(sii.EntryPoint);
    CLIENT_ID cid;
    ZwCreateThread(&hThread, THREAD_ALL_ACCESS, 0, hProcess, &cid, &context, &stack, TRUE);
    PROCESS_BASIC_INFORMATION pbi;
    ZwQueryInformationProcess(hProcess, ProcessBasicInformation, &pbi, sizeof pbi, 0);
    CreateProcessParameters(hProcess, pbi.PebBaseAddress, name);
    InformCsrss(hProcess, hThread, ULONG(cid.UniqueProcess), ULONG(cid.UniqueThread));
    ZwResumeThread(hThread, 0);
    ZwClose(hProcess);
    ZwClose(hThread);
    return int(cid.UniqueProcess);
}
```

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `MyCreateProcess.rar` |
