
# 菜鸟也写驱动程序

> 原文：[`www.pediy.com/kssd/pediy10/65193.html`](https://www.pediy.com/kssd/pediy10/65193.html)

## 1. 编译器

驱动说到底是由编译器产生的一个二进制文件，后缀`sys`，和普通的"console application"和"WINDOWS application"一样，也是`pe`结构，但是他们运行的子系统不同，`link`时`driver`程序链接的子系统为`NATIVE`，为编译器传递`/SUBSYSTEM:NATIVE`参数。一般不用手动命令行的方式打命令编译，可以写好`makefile`（内容固定不变）和`SOURCE`文件，`cd`到程序目录下，`build`即可，还可以用前辈制作的`vc`向导生成，或者用更简单的`quicksys`、`easysys`等工具编译。搜索一下全都有了-_- `quicksys`我只能用`0.2`版本的，最新的`0.4`版本用着有些问题..

## 2. 入口点，驱动的"main"函数

我们知道一个简单的`console`程序入口点通常为`main`，`windows`程序入口点为`WinMain`，驱动也有类似的东西，被称为`DriverEntry`例程，是每个驱动函数必须要有的，原型如下：

`NTSTATUS DriverEntry(PDRIVER_OBJECT pDriverObject, PUNICODE_STRING pRegistryPath);`

这个例程由`I/O`管理器在装载时调用。并在注册表里生成相关键值，也就是传递的第二个参数，`UNICODE_STRING`字符串指针，指向该键值。驱动里统一用`UNICODE_STRING`，它不是普通的数据，而是一个结构体，而且不是`NULL`结尾的，如果`NULL`结尾，将产生蓝屏。初始化要用特殊手段，调用`RtlInitUnicodeString`函数。

`DRIVER_OBJECT`是一个用来代表(represent)驱动的数据结构，用他来初始化驱动一些数据结构，为他们例程做准备工作。指向返回`NTSTATUS`系统状态。

## 3. DriverUnload 例程

参数为一个`DriverOjbect`，如其名，由`DriverOjbect`的`unload`域指定，做一些清理工作，比如删除设备对象，删除符号连接等工作，名称可以任意。

创建了链接名，设备才能为`win32`子系统可见，

## 4. DeviceOjbect， 设备对象

设备对象是代表`I/O`操作目标的虚拟/逻辑/物理设备，我的理解是驱动程序负责创建出来真正起实际作用的，是`Driver`程序操作的目标对象。由`IoCreateDevice`创建，`IoCreateDevice`在`DriverEntry`或`AddDevice`中调用，取决于是否为`WDM`模型，我们编写的一般都不是`WDM`模型，所以在`DriverEntry`中调用，下面是一段代码：

代码：

```
NTSTATUS 
DriverEntry(PDRIVER_OBJECT pDriverObj, PUNICODE_STRING pRegistryString)
{
  NTSTATUS status;
  ULONG ulDeviceNumber = 0;
  pDriverObj->DriverUnload = ExampleUnload;

  PDEVICE_OBJECT pDevObj;
  UNICODE_STRING usDeviceName,usDosDeviceName;
  RtlInitUnicodeString(&usDeviceName,L"\\Device\\Example");
  RtlInitUnicodeString(&usDosDeviceName,L"\\Dos\\DosDevices\\Example");
  status = IoCreateDevice(pDriverObj,0,
          &usDeviceName,FILE_DEVICE_UNKNOWN,
          FILE_DEVICE_SECURE_OPEN,
          FALSE,&pDevObj);

  IoCreateSymbolicLink(&usDosDeviceName,&usDeviceName);
  return status;   
}
void ExampleUnload(PDRIVER_OBJECT pDriverObj)
{
  // delete symboliclink
  UNICODE_STRING usDosDeviceName;
  RtlInitUnicodeString(&usDosDeviceName,L"\\Dos\\DosDevices\\Example");
  IoDeleteSymbolicLink(&usDosDeviceName);

  // delete the Device
  IoDeleteDevice(pDriverObj->DeviceObject);
}
```

## 5. 驱动 loader

这已经是一个驱动了，可以调用各类核心函数，前提是要把它加载到内存中去，程序如下：

代码：

```
int _cdecl main(void)
{
    HANDLE hSCManager;
    HANDLE hService;
    SERVICE_STATUS ss;

    hSCManager = OpenSCManager(NULL, NULL, SC_MANAGER_CREATE_SERVICE);
     
    printf("Load Driver\n");

    if(hSCManager)
    {
        printf("Create Service\n");

        hService = CreateService(hSCManager, "Example", 
                                 "Example Driver", 
                                 SERVICE_START | DELETE | SERVICE_STOP, 
                                 SERVICE_KERNEL_DRIVER,
                                 SERVICE_DEMAND_START, 
                                 SERVICE_ERROR_IGNORE, 
                                 "C:\\example.sys", 
                                 NULL, NULL, NULL, NULL, NULL);

        if(!hService)
        {
            hService = OpenService(hSCManager, "Example", 
                   SERVICE_START | DELETE | SERVICE_STOP);
        }

        if(hService)
        {
            printf("Start Service\n");

            StartService(hService, 0, NULL);
            printf("Press Enter to close service\r\n");
            getchar();
            ControlService(hService, SERVICE_CONTROL_STOP, &ss);

            DeleteService(hService);

            CloseServiceHandle(hService);
            
        }

        CloseServiceHandle(hSCManager);
    }
    
    return 0;
}
```

初学也可以用一些驱动加载用具，自动加载，启动，停止，卸载，比如说`coderui`所写的图形化工具

[`hi.baidu.com/coderui`](http://hi.baidu.com/coderui)

里面还可以打印出一些调试信息，加入`DbgPrint()`，用`Sysinternals DebugView`，[`www.sysinternals.com/`](http://www.sysinternals.com/)

## 6. Dispatch 例程

但他还不能响应`I/O`请求，比如说`read`、`write`等，要为他们编写`Dispatch`例程

`Dispatch`例程的作用就是充当`I/O`请求与驱动之间的桥梁，用来标识不同`I/O`操作类型的是`IRP_MJ_XXX`主功能码

`Dispatch`例程函数指针存放在驱动对象`MajorFunction`数组中，主功能码用于索引该数组，填入以激活各例程。

  `pDriverObj->MajorFunction[IRP_MJ_CREATE]=DispatchCreate;`

  `pDriverObj->MajorFunction[IRP_MJ_CLOSE]=DispatchClose;`

  `pDriverObj->MajorFunction[IRP_MJ_READ] =DispatchRead;`

  `pDriverObj->MajorFunction[IRP_MJ_WRITE] =DispatchWrite;`

`windows`中应用程序通过调用`win32`函数与驱动进行通信。用`CreateFile`打开设备获得句柄，`ReadFile`从驱动中读取数据，`WriteFile`写入数据，程序退出时，`CloseHandle`关闭设备等，一一对应关系图：

`CreateFile`   `IRP_MJ_CREATE`

`CloseHandle`   `IRP_MJ_CLEANUP`

`CloseHandle`   `IRP_MJ_CLOSE`

`ReadFile`    `IRP_MJ_READ`

`WriteFile`  `IRP_MJ_WRITE`

...

`WriteFile`分几种情况，有缓冲的和无缓冲的，比较复杂，不深入了因为我还没完全掌握，编写这些函数参看火影的这篇文章，他的那篇是`wdm`模型，

[`bbs.pediy.com/showthread.php?t=58251`](http://bbs.pediy.com/showthread.php?t=58251)

最后再推荐本书

`The Windows 2000 Device Driver Book(Second Edition)`其他书比较难看懂..

本文是半翻译了`codeproject`上`Introduction to Drivers part1`和其他驱动书写的菜鸟之作。
