
## 2. 获取DOS头和NT头，判断x64还是x32分别填充`IMAGE_NT_HEADERS`(32|64)结构体，保存DLL的Nt，入口点(`IMageBase`)，大小(`SizeOfImage`)，DataDirectory 地址。`Dll_RvaToAddr`转换导出表地址，保存至`dll->exports`，插入DLL链表。

![](https://bbs.kanxue.com/upload/attach/202012/819685_VWKRBHB8YXRKHV8.jpg)

####

### 6) `Syscall_Init()`

全局链表:

```
static LIST Syscall_List;

static SYSCALL_ENTRY **Syscall_Table = NULL;

static ULONG Syscall_MaxIndex = 0;

static UCHAR *Syscall_NtdllSavedCode = NULL;

static SYSCALL_ENTRY *Syscall_SetInformationThread = NULL;
```

![](https://bbs.kanxue.com/upload/attach/202012/819685_GYKXRZZVPAZJBWA.jpg)

`Syscall_Init_List()`

初始化全局`List`,`Syscall_List`.从全局`DLL_List`链表中获取`NTDll.dll`(Zw系列函数)，通过`Dll_GetNextProc`来获取导出表的 IAT-IOD-INT，保存序号表索引和函数名。

```
ULONG *names = Dll_RvaToAddr(dll, dll->exports->AddressOfNames);

ULONG *addrs = Dll_RvaToAddr(dll, dll->exports->AddressOfFunctions);

USHORT *ordis = Dll_RvaToAddr(dll, dll->exports->AddressOfNameOrdinals);
```

![](https://bbs.kanxue.com/upload/attach/202012/KS3FTDG3GS4AE7R.jpg)

过滤了特殊的函数如下：

![](https://bbs.kanxue.com/upload/attach/202012/EEF5RSRR4DUZDBJ.jpg)

分析每一个Zw导出找到索引，`SySCall_GetKernelAddr()`，关注点如何获取SSDT基地址，Sandboxie定位SSDT代码再`Syscall_GetServiceTable()`函数中：

```
            if (syscall_index != -1) {

                Syscall_GetKernelAddr(

                            syscall_index, &ntos_addr, ¶m_count);

            }
```

如果有导出函数可以使用，最为方便导出`KeServiceDscriptorTable`即可。

![](https://bbs.kanxue.com/upload/attach/202012/819685_X2ZHXBW8GZC2QQH.jpg)

Sandboxie中Windows10寻找SSDT方式如下：

1. 查询获取`MODULE_INFO`对象，获取`kernel_base`：

![](https://bbs.kanxue.com/upload/attach/202012/819685_8W8KYH2RFZ8WEZM.jpg)

2. 获取`KeAddSystemServiceTable`地址

![](https://bbs.kanxue.com/upload/attach/202012/819685_P6Y23EHW23GF7QX.jpg)![](https://bbs.kanxue.com/upload/attach/202012/819685_V6D7HWKX469854F.jpg)

3. 最后插入`Syscall_List`链表

`Syscall_Init_Table()`

### 7) `Driver_FindMissingServices()`

初始化session相关回调:

```
    Api_SetFunction(API_SESSION_LEADER,         Session_Api_Leader);

    Api_SetFunction(API_DISABLE_FORCE_PROCESS,  Session_Api_DisableForce);

    Api_SetFunction(API_MONITOR_CONTROL,        Session_Api_MonitorControl);

    Api_SetFunction(API_MONITOR_PUT,            Session_Api_MonitorPut);

    Api_SetFunction(API_MONITOR_PUT2,           Session_Api_MonitorPut2);

    Api_SetFunction(API_MONITOR_GET,            Session_Api_MonitorGet);
```

![](https://bbs.kanxue.com/upload/attach/202012/819685_9HD3CQRQY2CFNGV.jpg)

### 8) `Driver_FindMissingServices()`

获取`ZwSetInformationTOken`，`FIND_SERVICE`进行Hook，后面在对Hook进行分析

![](https://bbs.kanxue.com/upload/attach/202012/819685_63WDQFVA9JZGQW3.jpg)

### 9) `Process_Init()`

`PsSetCreateProcessNotifyRoutine`&`PsSetLoadImageNotifyRoutine`。

1. `Process_NotifyProcess`：

`PspCreateThread`中处理`Process`，Sandboxi回调初始化`LdrInitializeThunk`注入。具体代码可以看

`Process_NotifyImage`：是在`DbgKCreateThreaD`中处理，负责Sandboxie空间初始化，有一个概念，它属于某一个沙盘内的进程。

```
if (!proc->bHostInject)    {

        if (ok)

            ok = File_CreateBoxPath(proc);

 

        if (ok)

            ok = Ipc_CreateBoxPath(proc);

 

        if (ok)

            ok = Key_MountHive(proc);

 

        //

        // initialize the filtering components

        //

 

        if (ok)

            ok = File_InitProcess(proc);

 

        if (ok)

            ok = Key_InitProcess(proc);

 

        if (ok)

            ok = Ipc_InitProcess(proc);

 

        if (ok)

            ok = Gui_InitProcess(proc);

 

        if (ok)

            ok = Process_Low_InitConsole(proc);

 

        if (ok)

            ok = Token_ReplacePrimary(proc);

 

        if (ok)

            ok = Thread_InitProcess(proc);

}
```

`File_CreateBoxPath()`

初始化路径

```
        RtlInitUnicodeString(&objname, proc->box->file_path);

 

        status = ZwCreateFile(

            &handle,

            FILE_GENERIC_READ | FILE_WRITE_ATTRIBUTES,

            &objattrs,

            &IoStatusBlock,

            NULL,                   // AllocationSize

            0,                      // FileAttributes

            FILE_SHARE_VALID_FLAGS, // ShareAccess

            FILE_OPEN_IF,           // CreateDisposition

            FILE_DIRECTORY_FILE | FILE_SYNCHRONOUS_IO_NONALERT,

            NULL, 0);               // EaBuffer, EaLength
```

`Ipc_CreateBoxPath(proc)`

循环创建对象

```
    while (retries < 64) {

 

        ++retries;
        RtlInitUnicodeString(&objname, proc->box->ipc_path);

 

        status = ZwCreateDirectoryObject(

            &handle, DIRECTORY_ALL_ACCESS, &objattrs);

 

        if (status == STATUS_OBJECT_PATH_NOT_FOUND) {
```

`Key_Init()`

initialize the filtering components初始化过滤接口

`File_InitProcess()`

### 10) `Thread_Init`

### 11) `File_Init`

### 12) `Key_Init`

### 13) `Ipc_Init`

### 14) `Gui_Init`

### 15) `Api_Init`

### 16) `Dll_Unload`

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2020-12-23 17:01

被一半人生编辑，原因：
