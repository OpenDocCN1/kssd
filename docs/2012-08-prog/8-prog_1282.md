
# 发一块代码段（删除正在运行的程序文件）

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-156.htm`](https://www.pediy.com/kssd/pediy09/pediy09-156.htm)

> 先声明一下，本文的代码其实也是从别人的工具里 A 出来的（A＝逆向＋还原），所以也不敢称是原创，有同好的朋友自然知道是哪个工具里的：），另外，由于本人不太会写文章，看不明白的地方还请大家多多谅解。

> 回到正题，通过分析得知，删除正在运行的程序文件的关键在于 hook `MmFlushImageSection` 这个函数，该函数原型 `BOOLEAN MmFlushImageSection(PSECTION_OBJECT_POINTERS SectionObjectPointer, MMFLUSH_TYPE FlushType)`，其流程大概如下：

1. 打开要删除的文件，如：调用 `IoCreateFile`
2. 把要删除的文件属性设为 `Normal`
3. Hook `ntfs` 和 `fastfat` 内的引入函数 `MmFlushImageSection`
4. 发送删除 `IRP`

> 大概代码如下（比较简单的一个 Demo）：

```c
void ForceDeleteFile(PVOID ThreadContext)  
{  
  int krResult = FALSE;  
  int krRetCode = FALSE;  
  //NTSTATUS rc;  
  HANDLE hFile = NULL, hFileSystem = NULL, SectionHandle = NULL;  
  CHAR *FileName = (CHAR*)ThreadContext;  
  CHAR szFileSystem[MAX_PATH_LEN];  
  PVOID OrgFunc = NULL, OrgFunc_Ptr = NULL;  
  KFile ntfs, fastfat, del;  
  //UNICODE_STRING usFileName;  
  //PFILE_OBJECT pFileObject = NULL;  
  //OBJECT_ATTRIBUTES ob;  
  DEBUG_BREAK;  
  krRetCode = del.OpenFile(FileName, GENERIC_READ);  
  PROCESS_ERROR(krRetCode);  
  dprintf("Open %s successful.\n", FileName);  
  krResult = del.SetFileAttributeToNormal();  
  PROCESS_ERROR(krResult);  
  strcpy(szFileSystem, DRIVERS_PATH);  
  strcat(szFileSystem, "ntfs.sys");  
  krRetCode = ntfs.OpenFile(szFileSystem, GENERIC_READ);  
  PROCESS_ERROR(krRetCode);  
  g_Deleted = TRUE;  
  krRetCode = ntfs.MapFile(&pMap, &uMapSize);  
  PROCESS_ERROR(krRetCode);  
  krRetCode = HookIAT(  
    "MmFlushImageSection",  
    "ntfs.sys",  
    pMap,  
    uMapSize,  
    (PVOID)MmFlushImageSection_New,  
    (PVOID*)&_MmFlushImageSection,  
    &OrgFunc_Ptr  
  );  
  PROCESS_ERROR(krRetCode);  
  krRetCode = ntfs.UnMapFile();  
  PROCESS_ERROR(krRetCode);  
  strcpy(szFileSystem, DRIVERS_PATH);  
  strcat(szFileSystem, "fastfat.sys");  
  krRetCode = fastfat.OpenFile(szFileSystem, GENERIC_READ);  
  PROCESS_ERROR(hFileSystem);  
  krRetCode = fastfat.MapFile(&pMap, &uMapSize);  
  PROCESS_ERROR(krRetCode);  
  krRetCode = HookIAT(  
    "MmFlushImageSection",  
    "fastfat.sys",  
    pMap,  
    uMapSize,  
    (PVOID)MmFlushImageSection_New,  
    (PVOID*)&_MmFlushImageSection,  
    &OrgFunc_Ptr  
  );  
  PROCESS_ERROR(krRetCode);  
  krRetCode = ntfs.UnMapFile();  
  PROCESS_ERROR(krRetCode);  
  krResult = Rd_DeleteFile(del.GetFileHandle());  
  PROCESS_ERROR(krResult);  
  krResult = TRUE;  
Exit0:  
  g_Deleted = FALSE;  
  if (OrgFunc && OrgFunc_Ptr)  
  {  
    UnHookIAT(_MmFlushImageSection, OrgFunc_Ptr);  
  }  
  ntfs.Close();  
  fastfat.Close();  
  del.Close();  
  return;  
}  
```

> `Rd_DeleteFile` 代码如下：

```c
int Rd_DeleteFile(HANDLE hFile)  
{  
  int krResult = FALSE;  
  DEVICE_OBJECT *pRealObject = NULL;  
  DEVICE_OBJECT *pDeviceObject = NULL;  
  PIRP irp;  
  NTSTATUS rc;  
  KEVENT event;  
  PIO_STACK_LOCATION irpSp;  
  IO_STATUS_BLOCK localIoStatus;  
  PFILE_OBJECT pFileObject = NULL;  
  FILE_DISPOSITION_INFORMATION disp;  
  ULONG Length = sizeof(FILE_DISPOSITION_INFORMATION);  
  disp.DeleteFile = TRUE;  
  rc = ObReferenceObjectByHandle(  
    hFile,  
    0,  
    *IoFileObjectType,  
    0,  
    (PVOID*)&pFileObject,  
    0);  
  PROCESS_DDK_ERROR(rc);  
  pDeviceObject = GetRealDeviceObject(pFileObject, &pRealObject);  
  PROCESS_ERROR(pDeviceObject);  
  KeInitializeEvent(&event, SynchronizationEvent, FALSE);  
  irp = IoAllocateIrp(pDeviceObject->StackSize, 0);  
  if (!irp)  
  {  
    goto Exit0;  
  }  
  irp->UserEvent = &event;  
  irp->UserIosb = &localIoStatus;  
  irp->Overlay.AsynchronousParameters.UserApcRoutine= (PIO_APC_ROUTINE) NULL;  
  irp->RequestorMode = 0;  
  irp->Tail.Overlay.Thread = PsGetCurrentThread();  
  irp->Tail.Overlay.OriginalFileObject = pFileObject;  
  irp->Flags = IRP_BUFFERED_IO;  
  irpSp = IoGetNextIrpStackLocation(irp);  
  irpSp->MajorFunction = IRP_MJ_SET_INFORMATION;  
  irpSp->FileObject = pFileObject;  
  irpSp->DeviceObject = pDeviceObject;  
  irp->UserBuffer = NULL;  
  irp->AssociatedIrp.SystemBuffer = NULL;  
  irp->MdlAddress = (PMDL) NULL;  
  __try {  
    irp->AssociatedIrp.SystemBuffer = ExAllocatePoolWithQuota( NonPagedPool,  
                                                            Length );  
    RtlCopyMemory( irp->AssociatedIrp.SystemBuffer,  
                  &disp,  
                  Length );  
  }  
  __except(EXCEPTION_EXECUTE_HANDLER)  
  {  
    //  
    // An exception was incurred while allocating the intermediary  
    // system buffer or while copying the caller's data into the  
    // buffer. Cleanup and return an appropriate error status code.  
    //  
    IopExceptionCleanup( pFileObject,  
                        irp,  
                        (PKEVENT) NULL,  
                        &event );  
    return GetExceptionCode();  
  }  
  irp->Flags |= IRP_BUFFERED_IO | IRP_DEALLOCATE_BUFFER | IRP_DEFER_IO_COMPLETION;  
  irpSp->Parameters.SetFile.DeleteHandle = hFile;  
  irpSp->Parameters.SetFile.Length = Length;  
  irpSp->Parameters.SetFile.FileInformationClass = FileDispositionInformation;  
  irpSp->Control = 0xE0;  
  irpSp->CompletionRoutine = (PIO_COMPLETION_ROUTINE)Io_Completion_Routine;  
  if (rc == STATUS_PENDING)  
  {  
    rc = KeWaitForSingleObject(  
      &event,  
      Executive,  
      0,  
      1,  
      0);  
  }  
  krResult = TRUE;  
Exit0:  
  return krResult;  
}  
```

> 原理：

> 该 `IRP` 最后会调用 `Ntfs` 内的 `NtfsSetDispositionInfo` 函数或 `Fastfat` 内的 `FatSetPositionInfo` 函数（这区别于你的硬盘分区是什么文件系统），而这两个函数都会调用 `Ntoskrnl` 内的 `MmFlushImageSection` 函数，由于我们 hook 了这两驱动内的 `MmFlushImageSection` 调用，所以当它自己发送 `IRP` 时，`Hook_MmFlushImageSection` 就能截获请求，然后在该函数内判断 `SectionObjectPointer` 参数是否等于需要删除文件的 `SectionObjectPointer`，如果相等则调用原来的 `MmFlushImageSection` 并返回 `True`（返回 `True` 表示正在被操作的 `FileObject` 的 `Section` 不存在或没有被 `Map` 进内存），如果不相等，则返回原来的 `MmFlushImageSection` 的调用结果。

> 后记：

> 自己构造 `IRP` 直接与 `FSD` 通信已经是老生常谈的话题了，有兴趣的朋友可以网上找找。

4dd466d0e1e0cc123afbe504da2462ca
