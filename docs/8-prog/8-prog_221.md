
# 山寨 Fsd Inline Hook

> 原文：[`www.pediy.com/kssd/pediy10/85020.html`](https://www.pediy.com/kssd/pediy10/85020.html)

代码大部分是炉子`0GiNr`提供的，在此感谢。增加了一段获取通过 NTFS 驱动对象获取分发函数地址的代码，如果已经被人 hook 过，可能造成蓝屏。最好的方法还是解析 NTFS 文件获取原始分发函数地址。Xp sp2 测试通过。

```c
extern POBJECT_TYPE *IoDriverObjectType;

NTKERNELAPI

NTSTATUS

NTAPI

ObReferenceObjectByName (
        IN PUNICODE_STRING ObjectName,
        IN ULONG Attributes,
        IN PACCESS_STATE PassedAccessState OPTIONAL,
        IN ACCESS_MASK DesiredAccess OPTIONAL,
        IN POBJECT_TYPE ObjectType OPTIONAL,
        IN KPROCESSOR_MODE AccessMode,
        IN OUT PVOID ParseContext OPTIONAL,
        OUT PVOID *Object
        );

PULONG NtfsMajorFunction = NULL;

ULONG NtfsDispatch = 0xfa8f3618; //Ntfs!NtfsFsdSetInformation

BOOLEAN GetNtfsMajorFunctionAddr()
{
  PDRIVER_OBJECT  pDriverObject;
  UNICODE_STRING  DeviceName;
  NTSTATUS    ntStaus;
  BOOLEAN      bRet = FALSE;
  RtlInitUnicodeString(&DeviceName, L"\\FileSystem\\Ntfs");
  ntStaus = ObReferenceObjectByName(&DeviceName, OBJ_CASE_INSENSITIVE, NULL, 0, *IoDriverObjectType, KernelMode, NULL, (PVOID*)&pDriverObject);
  if(!NT_SUCCESS(ntStaus))
    goto __end;
  NtfsMajorFunction = (PULONG)pDriverObject->MajorFunction;
  bRet = TRUE;
__end:
  return bRet;
}

NtfsDispatch = NtfsMajorFunction[IRP_MJ_SET_INFORMATION];
```

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `FsdInlineHook.rar` |
|---|---|
