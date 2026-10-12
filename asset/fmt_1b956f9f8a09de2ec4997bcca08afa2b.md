
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
