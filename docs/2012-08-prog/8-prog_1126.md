
# 查找隐藏驱动

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-113.htm`](https://www.pediy.com/kssd/pediy09/pediy09-113.htm)
>
> 最近写了个小工具查找隐藏驱动，可以查找断开 `PsLoadedModuleList` 链的驱动，比如 `np` 的驱动。
>
> 下面是关键代码：
>
> ```c
> PDIRECTORY_BASIC_INFORMATION    pDriverBuffer = NULL;
> pDriverBuffer = (PDIRECTORY_BASIC_INFORMATION)m_cSysInfo.QueryDirectoryObject(L"\\Driver", &uMemSize);
> ```
>
> 查找驱动对象目录下的所有对象，查找代码如下。
>
> ```c
> PVOID CNativeSysInfo::QueryDirectoryObject(PWSTR pwsDirPath, PULONG puMemSize)
> {
>     NTSTATUS            ntStatus;
>     UNICODE_STRING      usDirPath;
>     OBJECT_ATTRIBUTES   oa;
>     HANDLE              hDir = NULL;
>     PVOID               pBuffer = NULL;
>     ULONG               uLength = 0x800;
>     ULONG               uContext = 0;
>     ULONG               uResult = 0;
>
>     // 判断函数是否存在
>     if(m_lpRtlInitUnicodeString == NULL ||
>        m_lpZwOpenDirectoryObject == NULL ||
>        m_lpZwQueryDirectoryObject == NULL ||
>        m_lpZwClose == NULL)
>     {
>         return NULL;
>     }
>
>     // 打开目录对象
>     m_lpRtlInitUnicodeString(&usDirPath, pwsDirPath);
>     InitializeObjectAttributes(&oa, &usDirPath, OBJ_CASE_INSENSITIVE, NULL, NULL);
>     ntStatus = m_lpZwOpenDirectoryObject(&hDir, DIRECTORY_QUERY, &oa);
>     if(ntStatus != STATUS_SUCCESS)
>     {
>         TRACE(_T("ZwOpenDirectoryObject failed!"));
>         goto _exit;
>     }
>
>     // 查询目录对象
>     do
>     {
>         if(pBuffer)
>             VirtualFree(pBuffer, uLength, MEM_DECOMMIT);
>
>         uLength *= 2;
>
>         pBuffer = VirtualAlloc(NULL, uLength, MEM_COMMIT, PAGE_READWRITE);
>         if(pBuffer == NULL)
>             goto _exit;
>
>         ntStatus = m_lpZwQueryDirectoryObject(hDir, pBuffer, uLength, FALSE, TRUE, &uContext, &uResult);
>
>     } while(ntStatus == STATUS_MORE_ENTRIES || ntStatus == STATUS_BUFFER_TOO_SMALL);
>
>     // 判断查询是否成功完成
>     if(ntStatus == STATUS_SUCCESS)
>     {
>         if(puMemSize)
>             *puMemSize = uLength;
>     }
>     else
>     {
>         VirtualFree(pBuffer, uLength, MEM_DECOMMIT);
>         pBuffer = NULL;
>     }
>
> _exit:
>
>     if(hDir)
>     {
>         m_lpZwClose(hDir);
>         hDir = NULL;
>     }
>
>     return pBuffer;
> }
> ```
>
> 然后把得到的结果和常规结果比较就可以找到隐藏驱动。隐藏驱动的相关信息可以通过下面的方式得到。
>
> ```c
> PDRIVER_OBJECT          pDrvObject = NULL;
>
> RtlInitUnicodeString(&usDirPath, (PCWSTR)pvInBuf);
> ntStatus = ObReferenceObjectByName(&usDirPath,
>                                   OBJ_CASE_INSENSITIVE,
>                                   NULL,
>                                   0,
>                                   *IoDriverObjectType,
>                                   KernelMode,
>                                   NULL,
>                                   (PVOID*)&pDrvObject);
> ```
>
> 有了 `DRIVER_OBJECT` 结构就啥都有了！:)
>
> 下面是一个隐藏驱动的例子，用上面的办法可以发现。
>
> ```c
> void _EraseDrvFromModList(PDRIVER_OBJECT pDrvObject)
> {
>     PLDR_DATA_TABLE_ENTRY   pOwen;
>     PLDR_DATA_TABLE_ENTRY   pPrev;
>     PLDR_DATA_TABLE_ENTRY   pNext;
>
>     pOwen = (PLDR_DATA_TABLE_ENTRY)pDrvObject->DriverSection;
>     pPrev = (PLDR_DATA_TABLE_ENTRY)pOwen->InLoadOrderModuleList.Blink;
>     pNext = (PLDR_DATA_TABLE_ENTRY)pOwen->InLoadOrderModuleList.Flink;
>
>     pPrev->InLoadOrderModuleList.Flink = (PLIST_ENTRY)pNext;
>     pNext->InLoadOrderModuleList.Blink = (PLIST_ENTRY)pPrev;
>
>     pOwen->InLoadOrderModuleList.Flink = (PLIST_ENTRY)pOwen;
>     pOwen->InLoadOrderModuleList.Blink = (PLIST_ENTRY)pOwen;
> }
> ```

4dd466d0e1e0cc123afbe504da2462ca
