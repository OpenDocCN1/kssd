
# 进程完整路径获取方式详解

> 原文：[`www.pediy.com/kssd/pediy12/129136.html`](https://www.pediy.com/kssd/pediy12/129136.html)

**标题**: 【原创】进程完整路径获取方式详解  
**作者**: `zyhfut`  
**时间**: 2011-2-10,20:28:57  
**链接**: `http://bbs.pediy.com/showthread.php?t=129136`

获取进程对应可执行文件的完整路径，在编程中经常遇到。下面做了下详细分析。欢迎指正！

## 获取进程完整路径方法小结

**By** `Angelkiss` &nbsp; 2011 年 2 月 9 日星期三

最近在做一个 `xx` 项目，少不了要拦截进程创建。拦截进程创建的方法在这里就不多说了，免得跑题（其实也就是各种 `Hook` 的运用）。拦截到进程创建后，需要获得进程对应的可执行文件完整路径，因为只有进一步分析可执行文件，才能获得更多信息来判断该进程的好坏。于是有了本文的产生。

很多人对于获取进程的完整路径可能不屑一顾。以前写过的代码中确实也遇到过，利用 `google` 几分钟内，你会得到几个 `Api`，例如：`GetModuleFileNameEx`，`GetProcessImageFileName` 等。但对于 `Ring3` 层提供的 `Api`，`Windows` 内核层做了什么，又有多少人很明确的知道？杀毒软件与病毒木马一直玩着猫和老鼠的游戏，病毒木马稍作手脚，`Ring3` 获取进程完整路径时就会失败。只有我们知道这些 `Api` 的本质，才能以不变应万变。下面将从 `Windows` 内核层分析这些 `Api` 的本质，并提供几种获取进程完整路径及防止别人获取进程完整路径的方法。

## 一. 进程内核数据结构中与路径信息相关位置

### 1. `_EPROCESS` -> `_SE_AUDIT_PROCESS_CREATION_INFO` -> `_OBJECT_NAME_INFORMATION`

![img/6824133b92ea719bdbfe6716a48581b2.jpg](img/6824133b92ea719bdbfe6716a48581b2.jpg)
![img/64b32815c0fe038efb16cabee2b1d760.jpg](img/64b32815c0fe038efb16cabee2b1d760.jpg)

从上面的截图可以看到，在进程内核对象 `EPROCESS` 偏移 `0x1f4` 处存放着 `_SE_AUDIT_PROCESS_CREATION_INFO` 结构的指针，该指针处又存放着 `_OBJECT_NAME_INFORMATION` 结构的指针，而该结构中存放着该进程的 `NT` 式文件路径。

### 2. `_EPROCESS` -> `_SECTION_OBJECT` -> `_SEGMENT_OBJECT` -> `_CONTROL_AREA` -> `_FILE_OBJECT`

![img/f95253630f5d2652a6c869097574fba4.jpg](img/f95253630f5d2652a6c869097574fba4.jpg)
![img/bb5b5fcc3f58903d2a033e5e5d83eb91.jpg](img/bb5b5fcc3f58903d2a033e5e5d83eb91.jpg)
![img/9517f6b59c5200d1540706c9e4157fba.jpg](img/9517f6b59c5200d1540706c9e4157fba.jpg)

从上面的截图可以看到进程内核对象偏移 `0x138` 处存放着指向 `_SECTION_OBJECT` 结构的指针，而该结构的 `0x14` 处存放着 `_SEGMENT_OBJECT` 结构指针，`_SEGMENT_OBJECT` 结构偏移 `0x0` 处是 `_CONTROL_AREA` 结构的指针，该结构偏移 `0x24` 处是 `_FILE_OBJECT` 结构的指针，利用该结构的 `_DEVICE_OBJECT`, `FileName` 两个成员和 `RtlVolumeDeviceToDosName` 函数可以获取进程文件的 `DOS` 完整路径。

### 3. `_PEB` -> `_PEB_LDR_DATA` -> `_LDR_DATA_TABLE_ENTRY`

![img/ef7f215313bfc65c482b38fa123a02d6.jpg](img/ef7f215313bfc65c482b38fa123a02d6.jpg)
![img/d07a51e287edcd891b60073686dcb41e.jpg](img/d07a51e287edcd891b60073686dcb41e.jpg)

可以看到 `_PEB` 偏移 `0xc` 处是 `_PEB_LDR_DATA` 数据结构，该结构偏移 `0xc` 处是 `_LDR_DATA_TABLE_ENTRY` 数据结构。但是需要注意的是，上面的成员获取必须在对应进程的上下文。

### 4. `_PEB` -> `_RTL_USER_PROCESS_PARAMETERS`

![img/a05d69e8dfcf701c1d167398b12c2996.jpg](img/a05d69e8dfcf701c1d167398b12c2996.jpg)
![img/9e7ae5052bd11ef0866f3ac6f0d33ec6.jpg](img/9e7ae5052bd11ef0866f3ac6f0d33ec6.jpg)

同样是 `_PEB` 结构，偏移 `0x10` 处是 `_RTL_USER_PROCESS_PARAMETERS` 结构。同样需要在目标进程的上下文空间。

上面就是进程内核对象中存放路径的相关位置，当然也许还有其他地方，麻烦知道的大牛告诉我，谢谢！

## 二. 微软提供的获取进程路径相关函数分析

### 1. `GetProcessImageFileName` 函数

```c
DWORD GetProcessImageFileName(
  HANDLE hProcess,//目标进程句柄
  LPTSTR lpImageFileName,//提供的空间,用于保存获取到的进程路径
  DWORD nSize//空间大小
);//返回实际路径长度
```

上面的函数获取的进程可执行文件 `NT` 式完整路径。从前面的分析中我们可以看到，`_EPROCESS`->`_SE_AUDIT_PROCESS_CREATION_INFO`->`_OBJECT_NAME_INFORMATION` 该处存放的就是 `NT` 式完整路径。我们来验证下：

```asm
0:000> uf PSAPI!GetProcessImageFileNameA

PSAPI!GetProcessImageFileNameA:
76bc3dbd 8bff            mov     edi,edi
76bc3dbf 55             push    ebp
76bc3dc0 8bec           mov     ebp,esp
76bc3dc2 53             push    ebx
76bc3dc3 8b5d10         mov     ebx,dword ptr [ebp+10h]
76bc3dc6 56             push    esi
76bc3dc7 57             push    edi
76bc3dc8 8d5c1b08       lea     ebx,[ebx+ebx+8]
76bc3dcc 53             push    ebx
76bc3dcd 33ff           xor     edi,edi
76bc3dcf 57             push    edi
76bc3dd0 ff155c10bc76   call    dword ptr [PSAPI!_imp__LocalAlloc (76bc105c)]
76bc3dd6 8bf0           mov     esi,eax
76bc3dd8 3bf7           cmp     esi,edi
76bc3dda 7504           jne     PSAPI!GetProcessImageFileNameA+0x23 (76bc3de0)

PSAPI!GetProcessImageFileNameA+0x1f:
76bc3ddc 33db           xor     ebx,ebx
76bc3dde eb53           jmp     PSAPI!GetProcessImageFileNameA+0x76 (76bc3e33)

PSAPI!GetProcessImageFileNameA+0x23:
76bc3de0 57             push    edi
76bc3de1 53             push    ebx
76bc3de2 56             push    esi
76bc3de3 6a1b           push    1Bh
76bc3de5 ff7508         push    dword ptr [ebp+8]
76bc3de8 ff15d810bc76   call    dword ptr [PSAPI!_imp__NtQueryInformationProcess (76bc10d8)]
76bc3dee 3d040000c0     cmp     eax,0C0000004h
76bc3df3 7503          jne     PSAPI!GetProcessImageFileNameA+0x3b (76bc3df8)

PSAPI!GetProcessImageFileNameA+0x38:
76bc3df5 83c01f        add     eax,1Fh

PSAPI!GetProcessImageFileNameA+0x3b:
76bc3df8 3bc7          cmp     eax,edi
76bc3dfa 7d12          jge     PSAPI!GetProcessImageFileNameA+0x51 (76bc3e0e)

PSAPI!GetProcessImageFileNameA+0x3f:
76bc3dfc 50             push    eax
76bc3dfd ff15e410bc76   call    dword ptr [PSAPI!_imp__RtlNtStatusToDosError (76bc10e4)]
76bc3e03 50             push    eax
76bc3e04 ff156010bc76   call    dword ptr [PSAPI!_imp__SetLastError (76bc1060)]
76bc3e0a 33db          xor     ebx,ebx
76bc3e0c eb1e          jmp     PSAPI!GetProcessImageFileNameA+0x6f (76bc3e2c)

PSAPI!GetProcessImageFileNameA+0x51:
76bc3e0e 0fb706        movzx   eax,word ptr [esi]
76bc3e11 57             push    edi
76bc3e12 57             push    edi
76bc3e13 ff7510         push    dword ptr [ebp+10h]
76bc3e16 ff750c         push    dword ptr [ebp+0Ch]
76bc3e19 50             push    eax
76bc3e1a ff7604         push    dword ptr [esi+4]
76bc3e1d 57             push    edi
76bc3e1e 57             push    edi
76bc3e1f ff156810bc76   call    dword ptr [PSAPI!_imp__WideCharToMultiByte (76bc1068)]
76bc3e25 8bd8          mov     ebx,eax
76bc3e27 3bdf          cmp     ebx,edi
76bc3e29 7401          je      PSAPI!GetProcessImageFileNameA+0x6f (76bc3e2c)

PSAPI!GetProcessImageFileNameA+0x6e:
76bc3e2b 4b             dec     ebx

PSAPI!GetProcessImageFileNameA+0x6f:
76bc3e2c 56             push    esi
76bc3e2d ff155810bc76   call    dword ptr [PSAPI!_imp__LocalFree (76bc1058)]

PSAPI!GetProcessImageFileNameA+0x76:
76bc3e33 5f             pop     edi
76bc3e34 5e             pop     esi
76bc3e35 8bc3          mov     eax,ebx
76bc3e37 5b             pop     ebx
76bc3e38 5d             pop     ebp
76bc3e39 c20c00         ret     0Ch
```

从上面的代码可以看到 `GetProcessImageFileName` 是调用 `NtQueryInformationProcess` 的 `0x1b` 号功能。我们再看下 `wrk` 中 `NtQueryInformationProcess` 函数中的 `0x1b` 号功能代码：

```c
NTSTATUS
NtQueryInformationProcess(
    __in HANDLE ProcessHandle,
    __in PROCESSINFOCLASS ProcessInformationClass,
    __out_bcount(ProcessInformationLength) PVOID ProcessInformation,
    __in ULONG ProcessInformationLength,
    __out_opt PULONG ReturnLength
    )

{
。。。。（省略）

  switch ( ProcessInformationClass ) {
    case ProcessImageFileName:(0x1b 号功能)
        {
            ULONG LengthNeeded = 0;
      //先根据进程句柄获取进程内核对象
            st = ObReferenceObjectByHandle (ProcessHandle,
                                            PROCESS_QUERY_INFORMATION,
                                            PsProcessType,
                                            PreviousMode,
                                            &Process,
                                            NULL);
            if (!NT_SUCCESS (st)) {
                return st;
            }

            //
            // SeLocateProcessImageName will allocate space for a UNICODE_STRING and point pTempNameInfo
            // at that string.  This memory will be freed later in the routine.
            //从进程内核对象中获取路径，然后复制到输出缓冲
            st = SeLocateProcessImageName (Process, &pTempNameInfo);
            if (!NT_SUCCESS(st)) {
                ObDereferenceObject(Process);
                return st;
            }
            LengthNeeded = sizeof(UNICODE_STRING) + pTempNameInfo->MaximumLength;
            //
            // Either of these may cause an access violation. The
            // exception handler will return access violation as
            // status code. No further cleanup needs to be done.
            //
            try {
                if (ARGUMENT_PRESENT(ReturnLength) ) {
                    *ReturnLength = LengthNeeded;
                }
```



```
        if (ProcessInformationLength >= LengthNeeded) {  
            RtlCopyMemory(  
                ProcessInformation,  
                pTempNameInfo,  
                sizeof(UNICODE_STRING) + pTempNameInfo->MaximumLength  
                );  
            ((PUNICODE_STRING) ProcessInformation)->Buffer = (PWSTR)((PUCHAR) ProcessInformation + sizeof(UNICODE_STRING));  
        } else {  
            st = STATUS_INFO_LENGTH_MISMATCH;  
        }  
    } except(EXCEPTION_EXECUTE_HANDLER) {  
        st = GetExceptionCode ();  
    }  
    ObDereferenceObject(Process);  
    ExFreePool( pTempNameInfo );  
    return st;  
}  
```

。。。。。。。。。。（省略）

`NtQueryInformationProcess` 函数先根据进程句柄获取进程内核对象，然后调用 `SeLocateProcessImageName` 函数，该函数是重点，其代码如下：

```
NTSTATUS  

SeLocateProcessImageName(  
    __in PEPROCESS Process,  
    __deref_out PUNICODE_STRING *pImageFileName  
    )  
/*++  

Routine Description  

    This routine returns the ImageFileName information from the process, if available.  This is a "lazy evaluation" wrapper  
    around SeInitializeProcessAuditName.  If the image file name information has already been computed, then this call simply  
    allocates and returns a UNICODE_STRING with this information.  Otherwise, the function determines the name, stores the name in the  
    EPROCESS structure, and then allocates and returns a UNICODE_STRING.  Caller must free the memory returned in pImageFileName.  

Arguments  

    Process - process for which to acquire the name  
    pImageFileName - output parameter to return name to caller  

Return Value  

    NTSTATUS.  

--*/  
{  
    NTSTATUS             Status            = STATUS_SUCCESS;  
    PVOID                FilePointer       = NULL;  
    PVOID                PreviousValue     = NULL;  
    POBJECT_NAME_INFORMATION pProcessImageName = NULL;  
    PUNICODE_STRING      pTempUS           = NULL;  
    ULONG                NameLength        = 0;  
    PAGED_CODE();  
    *pImageFileName = NULL;  
    if (NULL == Process->SeAuditProcessCreationInfo.ImageFileName) {  
        //如果进程内核对象中的文件名未被计算，则根据进程的文件对象自己计算  
        // The name has not been predetermined.  We must determine the process name.   First, reference the  
        // PFILE_OBJECT and lookup the name.  Then again check the process image name pointer against NULL.   
        // Finally, set the name.  
        //获取进程对象中的文件对象  
        Status = PsReferenceProcessFilePointer( Process, &FilePointer );  
        if (NT_SUCCESS(Status)) {  
            //  
            // Get the process name information.   
            //从文件对象中获取进程路径名  
            Status = SeInitializeProcessAuditName(  
                      FilePointer,  
                      TRUE, // skip audit policy  
                      &pProcessImageName // to be allocated in nonpaged pool  
                      );  
            if (NT_SUCCESS(Status)) {  
                //  
                // Only use the pProcessImageName if the field in the process is currently NULL.  
                //  
                PreviousValue = InterlockedCompareExchangePointer(  
                                    (PVOID *) &Process->SeAuditProcessCreationInfo.ImageFileName,  
                                    (PVOID) pProcessImageName,  
                                    (PVOID) NULL  
                                    );  
                if (NULL != PreviousValue) {  
                    ExFreePool(pProcessImageName); // free what we caused to be allocated.  
                }  
            }  
            ObDereferenceObject( FilePointer );  
        }  
    }  
//如果进程对象中的文件名已经计算，则直接从该处复制  
    if (NT_SUCCESS(Status)) {  
        //  
        // Allocate space for a buffer to contain the name for returning to the caller.  
        //  
        NameLength = sizeof(UNICODE_STRING) + Process->SeAuditProcessCreationInfo.ImageFileName->Name.MaximumLength;  
        pTempUS = ExAllocatePoolWithTag( NonPagedPool, NameLength, 'aPeS' );  
        if (NULL != pTempUS) {  
            RtlCopyMemory(  
                pTempUS,  
                &Process->SeAuditProcessCreationInfo.ImageFileName->Name,  
                NameLength  
                );  
            pTempUS->Buffer = (PWSTR)(((PUCHAR) pTempUS) + sizeof(UNICODE_STRING));  
            *pImageFileName = pTempUS;  
        } else {  
            Status = STATUS_NO_MEMORY;  
        }  
    }  
    return Status;  
}  
```

从上面的代码可以看出，函数首先检查 `_EPROCESS` 结构中的 `_SE_AUDIT_PROCESS_CREATION_INFO` 处是否为空，如果非空直接复制该处保存的进程路径信息；反之则需要根据进程对象的文件对象重新计算进程路径信息，先调用 `PsReferenceProcessFilePointer` 获取进程对象的文件对象，该函数代码如下：

```
NTSTATUS  

PsReferenceProcessFilePointer (  
    IN PEPROCESS Process,  
    OUT PVOID *OutFileObject  
    )  
/*++  

Routine Description:  
    This routine returns a referenced pointer to the FilePointer of Process.   
    This is a rundown protected wrapper around MmGetFileObjectForSection.  

Arguments:  
    Process - Supplies the process to query.  
    OutFileObject - Returns the file object backing the requested section if  
                    success is returned.  

Return Value:  
    NTSTATUS.  

Environment:  
    Kernel mode, PASSIVE_LEVEL.  

--*/  
{  
    PFILE_OBJECT FileObject;  
    PAGED_CODE();  
    if (!ExAcquireRundownProtection (&Process->RundownProtect)) {  
        return STATUS_UNSUCCESSFUL;  
    }  
    if (Process->SectionObject == NULL) {  
        ExReleaseRundownProtection (&Process->RundownProtect);  
        return STATUS_UNSUCCESSFUL;  
    }  
//从进程内核对象的 SectionObject 处获取文件对象  
    FileObject = MmGetFileObjectForSection ((PVOID)Process->SectionObject);  
    *OutFileObject = FileObject;  
    ObReferenceObject (FileObject);  
    ExReleaseRundownProtection (&Process->RundownProtect);  
    return STATUS_SUCCESS;  
}  
```

这个函数主要从 `_EPROCESS` 的 `_SECTION_OBJECT` 处获取文件对象，注意要防止进程退出。我们再深入看下 `MmGetFileObjectForSection` 这个函数：

```
PFILE_OBJECT  

MmGetFileObjectForSection (  
    IN PVOID Section  
    )  
/*++  

Routine Description:  
    This routine returns a pointer to the file object backing a section object.  

Arguments:  
    Section - Supplies the section to query.  

Return Value:  
    A pointer to the file object backing the argument section.  

Environment:  
    Kernel mode, PASSIVE_LEVEL.  
    The caller must ensure that the section is valid for the  
    duration of the call.  

--*/  
{  
    PFILE_OBJECT FileObject;  
    ASSERT (KeGetCurrentIrql() == PASSIVE_LEVEL);  
    ASSERT (Section != NULL);  
    FileObject = ((PSECTION)Section)->Segment->ControlArea->FilePointer;  
    return FileObject;  
}  
```

看到上面的代码很熟悉吧，上面我们已经分析过了。现在已经获取到进程文件对象了，我们来看下如何获取进程路径信息，

```
NTSTATUS  

SeInitializeProcessAuditName (  
    __in __typefix(PFILE_OBJECT) PVOID FileObject,  
    __in BOOLEAN bIgnoreAuditPolicy,  
    __deref_out POBJECT_NAME_INFORMATION *pAuditName  
    )  
/*++  

Routine Description:  
    This routine initializes the executable name for auditing purposes.  It allocates memory for the  
    image file name.  This memory is pointed to by pAuditName.   

Arguments:  
    FileObject - Supplies a pointer to a file object for the image being  
                 executed.  
    bIgnoreAuditPolicy - boolean that indicates that the call should proceed without  
        regard to the system's auditing policy.         
    pAuditName - Supplies a pointer to a pointer for the object name information.  

Return value:  
    NTSTATUS.  

Environment:  
    KeAttached to the target process so not all system services are available.  

--*/  
{  
    NTSTATUS Status;  
```



```c
    OBJECT_NAME_INFORMATION TempNameInfo;  

    ULONG ObjectNameInformationLength;  

    POBJECT_NAME_INFORMATION pInternalAuditName;  

    PFILE_OBJECT FilePointer;  

    PAGED_CODE();  

    ASSERT (pAuditName != NULL);  

    *pAuditName = NULL;  

    //  
    // Check if the caller would like to get the process name, even if auditing does not  
    // require it.  
    //  
    if (FALSE == bIgnoreAuditPolicy) {  
        //  
        // At the time of process creation, this routine should only proceed when Object Access or  
        // Detailed Tracking auditing is enabled.  In all other cases, the process name is acquired  
        // when it is requested.  
        //  
        if (!SepAdtAuditThisEventWithContext( AuditCategoryObjectAccess, TRUE, FALSE, NULL ) &&  
            !SepAdtAuditThisEventWithContext( AuditCategoryDetailedTracking, TRUE, FALSE, NULL )) {  
            return STATUS_SUCCESS;  
        }  
    }  

    FilePointer = (PFILE_OBJECT) FileObject;  
    //  
    // Compute full path for imagefile.  
    // This first call to ObQueryNameString is guaranteed to fail.  
    // The ObjectNameInformationLength contains only a  
    // UNICODE_STRING, so if this call succeeded it would indicate  
    // an imagefile name of length 0.  That is bad, so all return  
    // values except STATUS_BUFFER_OVERFLOW (from NTFS) and  
    // STATUS_BUFFER_TOO_SMALL (from DFS).  This call gives  
    // me the buffer size that I need to store the image name.  
    //  
    pInternalAuditName = &TempNameInfo;  
    ObjectNameInformationLength = sizeof(OBJECT_NAME_INFORMATION);  

    //  
    Status = ObQueryNameString (FilePointer,  
                                pInternalAuditName,  
                                ObjectNameInformationLength,  
                                &ObjectNameInformationLength);  

    if ((Status == STATUS_BUFFER_OVERFLOW) ||  
        (Status == STATUS_BUFFER_TOO_SMALL)) {  
        //  
        // Sanity check ObQueryNameString.  Different filesystems  
        // may be buggy, so make sure that the return length makes  
        // sense (that it has room for a non-NULL Buffer in the  
        // UNICODE_STRING).  
        //  
        if (ObjectNameInformationLength > sizeof(OBJECT_NAME_INFORMATION)) {  
            pInternalAuditName = ExAllocatePoolWithTag (NonPagedPool,  
                            ObjectNameInformationLength, 'aPeS');  
            if (pInternalAuditName != NULL) {  
                Status = ObQueryNameString (FilePointer,  
                                            pInternalAuditName,  
                                            ObjectNameInformationLength,  
                                            &ObjectNameInformationLength);  
                if (!NT_SUCCESS(Status)) {  
#if DBG  
                    DbgPrint("\n** ObqueryNameString failed with 0x%x.\n", Status);  
#endif //DBG  
                    //  
                    // If the second call to ObQueryNameString did not succeed, then  
                    // something is very wrong.  Set the image name to NULL string.  
                    //                                                  
                    // Free the memory that the first call to ObQueryNameString requested,  
                    // and allocate enough space to store an empty UNICODE_STRING.  
                    //  
                    ExFreePool (pInternalAuditName);  
                    ObjectNameInformationLength = sizeof(OBJECT_NAME_INFORMATION);  
                    pInternalAuditName = ExAllocatePoolWithTag (NonPagedPool,  
                                            ObjectNameInformationLength,  
                                            'aPeS');  
                    if (pInternalAuditName != NULL) {  
                        RtlZeroMemory(pInternalAuditName, ObjectNameInformationLength);  
                        //  
                        // Status = STATUS_SUCCESS to allow the process creation to continue.  
                        //  
                        Status = STATUS_SUCCESS;  
                    } else {  
                        Status = STATUS_NO_MEMORY;  
                    }  
                }  
            } else {  
                Status = STATUS_NO_MEMORY;  
            }  
        } else {  
            //  
            // If this happens, then ObQueryNameString is broken for the FS on which  
            // it was called.  
            //  
#if DBG  
            DbgPrint("\n** ObqueryNameString failed with 0x%x.\n", Status);  
#endif //DBG  
            ObjectNameInformationLength = sizeof(OBJECT_NAME_INFORMATION);  
            pInternalAuditName = ExAllocatePoolWithTag (NonPagedPool,  
                ObjectNameInformationLength, 'aPeS');  
            if (pInternalAuditName != NULL) {  
                RtlZeroMemory(pInternalAuditName, ObjectNameInformationLength);  
                //  
                // Status = STATUS_SUCCESS to allow the process creation to continue.  
                //  
                Status = STATUS_SUCCESS;  
            } else {  
                Status = STATUS_NO_MEMORY;  
            }  
        }  
    } else {  
        //  
        // If ObQueryNameString returns some other error code, we cannot  
        // be certain of which action to take, or whether it has properly  
        // set the ReturnLength.  For example, ObQueryNameString has slightly  
        // different semantics under DFS than NTFS.  Additionally, 3rd  
        // party file systems may also behave unpredictably.  For these reasons,  
        // in the case of an unexpected error code from ObQueryNameString  
        // we set AuditName to zero length unicode string and allow process  
        // creation to continue.  
        //  
#if DBG  
        DbgPrint("\n** ObqueryNameString failed with 0x%x.\n", Status);  
#endif //DBG  
        ObjectNameInformationLength = sizeof(OBJECT_NAME_INFORMATION);  
        pInternalAuditName = ExAllocatePoolWithTag(NonPagedPool, ObjectNameInformationLength, 'aPeS');  
        if (pInternalAuditName != NULL) {  
            RtlZeroMemory(pInternalAuditName, ObjectNameInformationLength);  
            //  
            // Status = STATUS_SUCCESS to allow the process creation to continue.  
            //  
            Status = STATUS_SUCCESS;  
        } else {  
            Status = STATUS_NO_MEMORY;  
        }  
    }  

    *pAuditName = pInternalAuditName;  
    return Status;  
}
```

从上面的代码可以看到，起始就是调用一个 `ObQueryNameString` 函数，如果失败则将进程路径信息设置成空，返回。关于 `ObQueryNameString` 的源码我就不详细分析了，参见博文 `http://blog.csdn.net/misterliwei/archive/2009/08/20/4467301.aspx`，主要是根据内核对象中的 `_OBJECT_HEADER` 和 `_OBJECT_HEADER_NAME_INFO` 结构获取。到此 `GetProcessImageFileName` 函数已经完全暴露在我们面前。

我们再简单回顾下：`GetProcessImageFileName` 需要传入一个进程句柄，然后调用 `NtQueryInformationProcess` 函数的 `0x1b` 号功能，首先获取进程内核对象，然后调用 `SeLocateProcessImageName`，该函数首先查看 `_EPROCESS` 结构 `0x1f4` 处的 `_SE_AUDIT_PROCESS_CREATION_INFO` 是否为空，如果非空直接从该处获取；反之需要自己获取，并填充到该处。首先会从 `_EPROCESS` 中获取 `_FILE_OBJECT` 对象，获取方法为：`_EPROCESS->_SECTION_OBJECT->_SEGMENT_OBJECT->_CONTROL_AREA->_FILE_OBJECT`，获取到文件对象后，利用 `SeInitializeProcessAuditName` 函数获取文件对象的路径，这个函数里边主要调用 `ObQueryNameString` 函数，该函数是未文档化的函数，简单申明后可以直接使用。该函数主要是遍历对象目录树，组建文件路径。

通过以上的分析，我有了以下简单的想法：

**防御：**

1. 该函数调用的前提条件是：进程的句柄。而应用层获取进程句柄的函数是 `OpenProcess`，那么我们只要在这个函数上做下过滤，不让别人将自己映射到它的句柄表即可。
2. 我们可以看到该函数其实调用的是 `NtQueryInformationProcess` 的 `0x1b` 号功能，那么我们只要在 `NtQueryInformationProcess` 上做过滤同样可以。更深层数的 hook，例如 `SeLocateProcessImageName`、`ObQueryNameString` 等同理。



### （3）信息源分析

从上面分析的过程可以看到，其实信息源是在`\_FILE\_OBJECT`对象的目录树中，`\_EPROCESS`中的`\_SE\_AUDIT\_PROCESS\_CREATION\_INFO`也只不过是个副本。因此我们可以尝试自己遍历`\_FILE\_OBJECT`对象目录树，然后将其到根目录的每个结点名删除，当然`\_SE\_ADUIT\_PROCESS\_CREATION\_INFO`处的必须先删除。这可以说从根本上解决了`GetProcessImageFileName`函数的调用。只是猜想，未实践。不知道这样的擦出，会不会对进程本身产生影响！知道的麻烦告知！谢谢。

#### 保护

（1）针对`OpenProcess`的过滤，我们可以在内核中通过`ObOpenObjetByPointer`、`KeStackAttachProcess`将目标对象映射到我们指定的进程空间，然后将句柄传到应用层，就可以直接操作了。当然既然已经到内核层，也就没必要到应用层了。

（2）对于`NtQueryInformationProcess`、`SeLocateProcessImageName`、`ObQueryNameString`等的Hook，我们既然知道其原理，就可以自己实现了。

（3）对于最后这种信息的擦除，只能看看别的地方是否曾经保存过进程路径信息了。如`\_PEB`中。

### 2. `GetModuleFileNameEx`函数

我们还是从源码看起，没有什么比源码更具说服力的了。

![](img/0d501e4618c949f6cb3aced822074dfb.jpg)

可以看到代码首先调用`PSAPI!FindModule`函数，然后调用`ReadProcessMemory`。我们一个个看。首先是`PSAPI!FindModule`函数：

![](img/fb1f8ff7b667452d781548bf736a29c3.jpg)

可以看到这次调用`NtQueryInformationProcess`的 0 号功能函数，获取的数据结构为：

```c
typedef struct _PROCESS_BASIC_INFORMATION {
    PVOID Reserved1;
    PPEB PebBaseAddress; //偏移 0x4 处
    PVOID Reserved2[2];
    ULONG_PTR UniqueProcessId;
    PVOID Reserved3;
} PROCESS_BASIC_INFORMATION;
```

可以看到偏移 0x4 处，也就是上面的`ebp-0x1c`处保存着`Peb`的指针，从`\_PEB`结构可以看到偏移 0xc 处恰好为`\_LDR`结构。

![](img/3ff28edfa64ad022cd238ade27e9f41e.jpg)

后面的就不看了。可以断定，该函数是从`\_PEB`的`\_PEB\_LDR\_DATA`中获取进程路径信息。同样我们怎么防护呢？原理与上面的基本相同。这个函数也需要进程句柄，所以可以过滤`OpenProcess`，同样可以利用更深层次的 Hook，以及擦除信息。

### 3. `ReadProcessMemory` + `\_RTL\_USER\_PROCESS\_PARAMETERS`

因为`\_RTL\_USER\_PROCESS\_PARAMETERS`中存放着进程的完整路径，结构如下所示：

![](img/21e593801006210aab03b37b3bb46d57.jpg)

这种方法的防御保护方法就不再重复了，同上。

### 4. `\_FILE\_OBJECT`结构

该种方法是在内核实现的。看下`\_FILE\_OBJECT`的结构。

![](img/cc5b164e71a23ec208aad9a13b4e4de2.jpg)

可以看到`\_FILE\_OBJECT`对象中的`FileName`中保存进程路径信息，再根据`DeviceObject`获取驱动器名，组合下就可以获取进程完整路径。

## 三. 后记

由前面的分析，我们可以看到：在内核态如果我们想要获取进程的 Dos 完整路径，我们可以先利用`ZwQueryInformationProcess`的 0x1b 号功能，获取到进程的 NT 路径，然后再利用`ZwOpenFile`打开文件，再利用`ObReferenceObjectByHandle`获取到文件对象，然后利用`RtlVolumeDeviceToDosName`获得驱动器名，再组合下即可。
