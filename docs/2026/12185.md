
# [原创]没有对应驱动文件的系统线程

```
/*
早在2014.10.22都已经写过一个失败的隐藏系统线程的代码.

现在看来,当时失败在函数的处理上,里面有很多无效/非法的指令.

这几天突然看到:

365K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6@1j5h3&6V1j5i4y4S2N6q4)9J5c8V1#2W2L8h3!0J5P5f1#2G2L8W2)9J5k6h3N6A6N6l9`.`.

371K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6o6M7U0c8K6K9q4)9J5c8V1c8J5N6V1S2A6k6r3g2Q4x3X3c8b7L8@1y4Q4x3X3g2Y4K9i4b7`.

来个灵感的想法,写个测试,所以有了此文.

此驱动在procexp.exe是查不到的,因为驱动加载失败.

保留搜索内存的PE特征也应该搜索不到,因为驱动加载失败.

但是procexp.exe查看system的进程的线程信息能看到,但是你找不到对应的驱动,因为驱动加载失败.

假如驱动支持卸载,让驱动启动成功,卸载函数不做任何处理,我想驱动卸载后自己的那个系统线程还会继续运行.

最后说下:

这是个小玩意,

也许对你有用,

请勿用于非法用途,

小心MemoryMon监控出.

其实线程回调也能监控出,并进程识别和阻断(自己编写杀系统线程的代码)的.

编译环境:VS2017+WDK10

测试环境:Windows 10 x64

made by correy

made at 10:38 2019/1/19

523K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8%4u0J5k6i4W2Q4x3X3g2%4k6h3u0K6i4K6u0W2j5$3!0E0

*/

#include <ntifs.h>
#include <windef.h>
#include <intrin.h>

#define TAG 'tset' //test

PVOID gThreadObj;

/*
这里有众多的注意事项要说明:

1.函数内不准有异常处理的函数,特别是X64.

2.编译选项要关闭/JMC.否者会出现一些怪异的函数.

3.编译选项要关闭/guard:cf

4.此函数可考虑用汇编写.

5.此函数也可考虑用ShellCode写.

6.此函数不可访问本驱动的全局变量.

7.此函数不可直接调用系统的API,可获取地址,因为导入表在本驱动.

*/

#pragma region Context code that cannot reference any exports directly

#pragma optimize("", off)

#pragma check_stack(off)

#pragma runtime_checks("", off)

KSTART_ROUTINE MyThreadStart;

VOID MyThreadStart(__in PVOID  StartContext)

/*
此函数的功能在于的你的想象.

StartContext传入的内容,你想想吧!但这必须是你申请的内存,不可使本驱动的(局部或全局)变量.

*/

{

//LARGE_INTEGER li;

UNREFERENCED_PARAMETER(StartContext);

__debugbreak();

for (;;)

{

/*

这里没有加退出条件。

*/

__debugbreak();

//li.QuadPart = -(10000 * 3 * 1000 * 60);

//KeDelayExecutionThread(KernelMode, FALSE, &li);

}

//PsTerminateSystemThread(STATUS_SUCCESS);

}

VOID MyThreadEnd(VOID)

{

NOTHING

}

#pragma runtime_checks("", restore)

#pragma check_stack()

#pragma optimize("", on)

#pragma endregion

DRIVER_INITIALIZE DriverEntry;

NTSTATUS DriverEntry(__in struct _DRIVER_OBJECT  * DriverObject, __in PUNICODE_STRING  RegistryPath)

{

NTSTATUS status = STATUS_SUCCESS;

HANDLE                hThread = 0;

OBJECT_ATTRIBUTES    oa;

PVOID buffer = NULL;

ULONG bufferLength = 0;

UNREFERENCED_PARAMETER(RegistryPath);

KdBreakPoint();

bufferLength = (ULONG)((SIZE_T)MyThreadStart - (SIZE_T)MyThreadEnd);

buffer = ExAllocatePoolWithTag(NonPagedPoolExecute, bufferLength, TAG);

if (buffer == NULL) {

return STATUS_INSUFFICIENT_RESOURCES;

}

RtlZeroMemory(buffer, bufferLength);

RtlCopyMemory(buffer, (PVOID)MyThreadStart, bufferLength);

/*

此时检查buffer函数中有没非法的指令,有没调用非法的函数,调用的函数中有没非法的指令.

可以与MyThreadStart函数做个对比.

还可以对buffer下个断点.

*/

InitializeObjectAttributes(&oa, NULL, OBJ_KERNEL_HANDLE, NULL, NULL);

status = PsCreateSystemThread(&hThread, THREAD_ALL_ACCESS, &oa, NULL, NULL, (PKSTART_ROUTINE)buffer, NULL);

if (!NT_SUCCESS(status))

{

KdPrint(("PsCreateSystemThread failed.\r\n"));

return status;

}

ObReferenceObjectByHandle(hThread, THREAD_ALL_ACCESS, NULL, KernelMode, &gThreadObj, NULL);

ZwClose(hThread);

return STATUS_UNSUCCESSFUL;//不返回STATUS_CANCELLED也照样能运行那个线程的代码.

}
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
