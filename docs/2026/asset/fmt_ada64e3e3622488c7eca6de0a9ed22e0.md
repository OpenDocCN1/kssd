
```
//系统服务表

typedef struct _KSYSTEM_SERVICE_TABLE

{
    PULONG ServiceTableBase;       //函数地址表的首地址
    PULONG ServiceCounterTableBase;//函数表中每个函数被调用的次数
    ULONG  NumberOfService;        //服务函数的个数
    ULONG  ParamTableBase;         //参数个数表首地址

}KSYSTEM_SERVICE_TABLE;
```

但实际上系统共有4个系统服务描述符，其中2个就是上述的2张表，另外2个没有被使用，可能是留着将来备用的。他们用如下结构体表示：

```
//服务描述符

typedef struct _KSERVICE_TABLE_DESCRIPTOR

{
    KSYSTEM_SERVICE_TABLE ntoskrnl;//ntoskrnl.exe的服务函数,即SSDT
    KSYSTEM_SERVICE_TABLE win32k;  //win32k.sys的服务函数,即ShadowSSDT
    KSYSTEM_SERVICE_TABLE notUsed1;//暂时没用1
    KSYSTEM_SERVICE_TABLE notUsed2;//暂时没用2

}KSERVICE_TABLE_DESCRIPTOR;
```

### 调用号

进入0环时调用号是`eax`传递的，但这个调用号并不只是一个普通的数字作为索引序号，系统会把他用32位数据表示，拆分成19：1：12的格式，如下：

![](https://bbs.kanxue.com/upload/attach/201904/847228_K4K7ZMEJ4WQPTBJ.jpg)

分析一下0-11这低12位组成一个真正的索引号，第12位表示服务表号，13-31位没有使用。

而进入内核后调用哪一张表，就由调用号中的第12位决定，为0则调用`SSDT`表，为1则调用`ShadowSSDT`表。

想要对`SSDT`表进行HOOK，首先要找到`SSDT`表，可以通过服务表找到它，服务表存在于`KTHREAD`结构体的偏移为`0xbc`的一个字段中，可双机调试使用`windbg`查看，输入`dt _kthread`，然后再找到偏移为`0xbc`的字段

![](https://bbs.kanxue.com/upload/attach/201904/847228_GBY8ZVU9YZ7DTX6.jpg)

![](https://bbs.kanxue.com/upload/attach/201904/847228_THUNNDPWGWBVA6E.jpg)

还是要说明一点`KTHREAD`这个结构体是未文档化的，所以里面的数据随着系统版本的不同各个字段偏移就有可能不同，所以使用前最好用`windbg`确认下所需要的字段偏移。

使用`PsGetCurrentThread()`函数可获取当前`KTHREAD`的首地址。

但是需要注意的是`SSDT`表所在的内存页属性是只读，没有写入的权限，所以需要把该地址设置为可写入，这样才能写入自己的函数。

我使用的是`CR0`寄存器关闭只读属性

简单介绍下`CR0`寄存器:

![](https://bbs.kanxue.com/upload/attach/201904/847228_SPDWRB9ESQKFEY9.jpg)

可以看到这里使用32位寄存器，而在`CR0`寄存器中，我们重点关注的是3个标志位：

- `PE` 是否启用保护模式，置1则启用。
- `PG` 是否使用分页模式，置1则开启分页模式，此标志置1时，`PE` 标志也必须置1，否则CPU报异常。
- `WP` 为1时，不能修改只读的内存页；`WP` 为0时，可以修改只读的内存页。

所以在进行HOOK时，只要把`CR0`寄存器中的`WP`位置为0，就能对内存进行写入操作。

操作代码如下

```
//关闭页只读保护

__asm

    {
        push eax;
        mov eax, cr0;
        and eax, ~0x10000;
        mov cr0, eax;
        pop eax;
        ret;
    }
```

当然使用完成后要把只读保护属性还回去，不然会引发不可预料的严重后果

```
//开启页只读保护

__asm

    {
        push eax;
        mov eax, cr0;
        or eax, 0x10000;
        mov cr0, eax;
        pop eax;
        ret;
    }
```

有了以上分析的基础那么现在就可以写安装内核钩子的代码了：

```
//安装钩子

void InstallHook()

{
    //1.获取KTHREAD

    PETHREAD pNowThread = PsGetCurrentThread();

    //2.获取ServiceTable表

    g_pServiceTable = (KSERVICE_TABLE_DESCRIPTOR*)

        (*(ULONG*)((ULONG)pNowThread + 0xbc));

    //3.保存hook的旧的函数的地址，0xbe为ZwOpenProcess的调用号

    g_OldZwOpenProcess = (FuZwOpenProcess)

        g_pServiceTable->ntoskrnl.ServiceTableBase[0xbe];

    //4.关闭页只读保护

    ShutPageProtect();

    //5.写入自己的函数到SSDT表内

    g_pServiceTable->ntoskrnl.ServiceTableBase[0xbe]

        = (ULONG)MyZwOpenProcess;

    //6.开启页只读保护

    OpenPageProtect();

}
```

看一下自己写的`MyZwOpenProcess`函数，通过对比`PID`找到要保护的进程，并且是以结束进程权限`PROCESS_TERMINATE`（`0x1`）访问时，则修改权限为0使其无法访问。就达到了保护进程的目的。

```
//自写的函数

NTSTATUS NTAPI MyZwOpenProcess(

    _Out_ PHANDLE ProcessHandle,

    _In_ ACCESS_MASK DesiredAccess,

    _In_ POBJECT_ATTRIBUTES ObjectAttributes,

    _In_opt_ PCLIENT_ID ClientId

)

{
    //当此进程为要保护的进程时，并且是以结束进程权限访问时

    if (ClientId->UniqueProcess == (HANDLE)g_Pid &&

        DesiredAccess == PROCESS_TERMINATE)

    {
        //设为拒绝访问

        DesiredAccess = 0;

    }

    //调用原函数

    return g_OldZwOpenProcess(

        ProcessHandle,

        DesiredAccess,

        ObjectAttributes,

        ClientId);

}
```

生成`.sys`文件后使用工具安装驱动服务，然后打开任务管理器，关闭被保护的进程，就可以看到拒绝访问，到此保护进程就成功了。在这主要描述了`SSDT`-HOOK，HOOK的功能就可以去任意发挥了。

![](https://bbs.kanxue.com/upload/attach/201904/847228_G3KH4HPFJRN83NX.jpg)

### 完整源码如下

```
#include <ntifs.h>



//内核之SSDT-HOOK

//系统服务表

typedef struct _KSYSTEM_SERVICE_TABLE

{
    PULONG ServiceTableBase;       //函数地址表的首地址
    PULONG ServiceCounterTableBase;//函数表中每个函数被调用的次数
    ULONG  NumberOfService;        //服务函数的个数
    ULONG ParamTableBase;          //参数个数表首地址

}KSYSTEM_SERVICE_TABLE;



//服务描述符

typedef struct _KSERVICE_TABLE_DESCRIPTOR

{
    KSYSTEM_SERVICE_TABLE ntoskrnl;//ntoskrnl.exe的服务函数,SSDT
    KSYSTEM_SERVICE_TABLE win32k;  //win32k.sys的服务函数,ShadowSSDT
    KSYSTEM_SERVICE_TABLE notUsed1;//暂时没用1
    KSYSTEM_SERVICE_TABLE notUsed2;//暂时没用2

}KSERVICE_TABLE_DESCRIPTOR;



//定义HOOK的函数的类型

typedef NTSTATUS (NTAPI*FuZwOpenProcess)(

    _Out_ PHANDLE ProcessHandle,

    _In_ ACCESS_MASK DesiredAccess,

    _In_ POBJECT_ATTRIBUTES ObjectAttributes,

    _In_opt_ PCLIENT_ID ClientId

);



//自写的函数声明

NTSTATUS NTAPI MyZwOpenProcess(

    _Out_ PHANDLE ProcessHandle,

    _In_ ACCESS_MASK DesiredAccess,

    _In_ POBJECT_ATTRIBUTES ObjectAttributes,

    _In_opt_ PCLIENT_ID ClientId

);



//记录系统的该函数

FuZwOpenProcess g_OldZwOpenProcess;

//服务描述符表指针

KSERVICE_TABLE_DESCRIPTOR* g_pServiceTable = NULL;

//要保护进程的ID

ULONG g_Pid = 9527;



//安装钩子

void InstallHook();

//卸载钩子

void UninstallHook();

//关闭页写入保护

void ShutPageProtect();

//开启页写入保护

void OpenPageProtect();



//卸载驱动

void OutLoad(DRIVER_OBJECT* obj);







////***驱动入口主函数***/

NTSTATUS DriverEntry(DRIVER_OBJECT* driver, UNICODE_STRING* path)

{
    path;

    KdPrint(("驱动启动成功！\n"));

    //DbgBreakPoint();



    //安装钩子

    InstallHook();



    driver->DriverUnload = OutLoad;

    return STATUS_SUCCESS;

}



//卸载驱动

void OutLoad(DRIVER_OBJECT* obj)

{
    obj;

    //卸载钩子

    UninstallHook();

}



//安装钩子

void InstallHook()

{
    //1.获取KTHREAD

    PETHREAD pNowThread = PsGetCurrentThread();

    //2.获取ServiceTable表

    g_pServiceTable = (KSERVICE_TABLE_DESCRIPTOR*)

        (*(ULONG*)((ULONG)pNowThread + 0xbc));

    //3.保存旧的函数

    g_OldZwOpenProcess = (FuZwOpenProcess)

        g_pServiceTable->ntoskrnl.ServiceTableBase[0xbe];

    //4.关闭页只读保护

    ShutPageProtect();

    //5.写入自己的函数到SSDT表内

    g_pServiceTable->ntoskrnl.ServiceTableBase[0xbe]

        = (ULONG)MyZwOpenProcess;

    //6.开启页只读保护

    OpenPageProtect();

}



//卸载钩子

void UninstallHook()

{
    //1.关闭页只读保护

    ShutPageProtect();

    //2.写入原来的函数到SSDT表内

    g_pServiceTable->ntoskrnl.ServiceTableBase[0xbe]

        = (ULONG)g_OldZwOpenProcess;

    //3.开启页只读保护

    OpenPageProtect();

}



//关闭页只读保护

void _declspec(naked) ShutPageProtect()

{
    __asm

    {
        push eax;
        mov eax, cr0;
        and eax, ~0x10000;
        mov cr0, eax;
        pop eax;
        ret;
    }

}



//开启页只读保护

void _declspec(naked) OpenPageProtect()

{
    __asm

    {
        push eax;
        mov eax, cr0;
        or eax, 0x10000;
        mov cr0, eax;
        pop eax;
        ret;
    }

}



//自写的函数

NTSTATUS NTAPI MyZwOpenProcess(

    _Out_ PHANDLE ProcessHandle,

    _In_ ACCESS_MASK DesiredAccess,

    _In_ POBJECT_ATTRIBUTES ObjectAttributes,

    _In_opt_ PCLIENT_ID ClientId

)

{
    //当此进程为要保护的进程时

    if (ClientId->UniqueProcess == (HANDLE)g_Pid &&

        DesiredAccess == PROCESS_TERMINATE)

    {
        //设为拒绝访问

        DesiredAccess = 0;

    }

    //调用原函数

    return g_OldZwOpenProcess(

        ProcessHandle,

        DesiredAccess,

        ObjectAttributes,

        ClientId);

}
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2019-4-17 11:04 被九阳道人编辑，原因：上传图片
