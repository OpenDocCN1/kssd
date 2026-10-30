
![](https://bbs.kanxue.com/upload/attach/202110/921642_BK23G4UA3EPCSU8.jpg)

> **应用层面的编程**
>
> 1. 在应用程序中打开与关闭设备
> 2. 设备控制请求

```cpp
#include <iostream>
#include <Windows.h>
#include <winioctl.h>

//这个宏用于组装IRP控制码，其中用户自定义的控制码从0x800开始。
#define io_code1 CTL_CODE(FILE_DEVICE_UNKNOWN,0x800,METHOD_BUFFERED,FILE_ANY_ACCESS)
#define io_code2 CTL_CODE(FILE_DEVICE_UNKNOWN,0x900,METHOD_BUFFERED,FILE_ANY_ACCESS)
int main()
{
    CHAR* deviceName = (CHAR*)"\\\\.\\MyDevice1";
    //1.在应用程序中打开与关闭设备
    HANDLE deviceHandle = CreateFileA(deviceName,GENERIC_READ|GENERIC_WRITE, FILE_SHARE_READ| FILE_SHARE_WRITE,NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL,NULL);
    DWORD str = 100;
    DWORD back = 0;
    DWORD Length = 0; //实际返回的数据长度
    //2.设备控制请求
    DeviceIoControl(deviveHandle, io_code1, &str,0x4,&back,0x4,&length,NULL);
    printf("back = %d\r\n", Length);
    str = 200;
    DeviceIoControl(deviceHandle, io_code2, &str, 0x4, &back, 0x4, &Length, NULL);
    printf("back = %d\r\n", Length);
    getchar();
    CloseHandle(deviceHandle);
    return 0;
}
```

```cpp
BOOL DeviceIoControl (
    HANDLE hDevice, // 设备句柄
    DWORD dwIoControlCode, // IOCTL请求操作代码
    LPVOID lpInBuffer, // 输入缓冲区地址
    DWORD nInBufferSize, // 输入缓冲区大小
    LPVOID lpOutBuffer, // 输出缓冲区地址
    DWORD nOutBufferSize, // 输出缓冲区大小
    LPDWORD lpBytesReturned, // 存放返回字节数的指针
    LPOVERLAPPED lpOverlapped // 用于同步操作的Overlapped结构体指针
);
```

总结：通过用户层与内核层的通信，可以让用户程序在需要时调用驱动的特定功能.驱动通信类似于Windows消息机制，在内核中，消息被封装在IRP结构体中（I/O Request Packae ：IO请求包），设备对象接受IRP实现用户程序与驱动程序的通信。

**通过`DeviceIoControl`函数来使应用程序与驱动程序通信**

这种通信方式，就是驱动程序和应用程序自定义一种IO控制码，然后应用程序调用`DeviceIoControl`函数，`DeviceIoControl`函数会产生此 IRP `IRP_MJ_DEVICE_CONTROL`，系统就调用相应的处理`IRP_MJ_DEVICE_CONTROL`的分发函数，在分发函数中判断，是自定义的控制码你就进行相应的处理。

[编写 Hello World Windows 驱动程序 (KMDF) - Windows drivers | Microsoft Docs](https://bbs.kanxue.com/elink@937K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3P5X3S2Q4x3X3c8U0L8W2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3X3c8Z5j5i4u0V1N6$3q4J5k6g2)9J5c8X3c8J5K9i4k6W2M7Y4y4Q4x3V1k6Y4k6i4c8@1K9h3&6Y4M7%4c8S2M7Y4c8W2k6q4)9J5c8Y4N6J5K9i4c8A6L8X3N6Q4x3X3c8S2i4K6u0V1N6X3g2J5P5g2)9J5k6s2y4E0j5h3I4D9i4K6u0V1K9$3#2V1k6W2)9J5k6q4)9J5k6r3c8J5K9i4k6W2M7R3`.`.)

[Windows 驱动程序示例 - Windows drivers | Microsoft Docs](https://bbs.kanxue.com/elink@0a4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3P5X3S2Q4x3X3c8U0L8W2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3X3c8Z5j5i4u0V1N6$3q4J5k6g2)9J5c8X3c8J5K9i4k6W2M7Y4y4Q4x3V1k6K6j5h3#2H3L8r3g2K6i4K6u0r3)

（可以借助这个HEVD项目更加加深对驱动通信的理解）

## 三：漏洞点分析

漏洞原理：栈溢出漏洞，使用过程中使用危险的函数，没有对

（1）加载驱动

安装驱动程序，使用`kmdManager`加载驱动程序，`DebugView`检测内核，可以看到加载驱动程序成功。

![](https://bbs.kanxue.com/upload/attach/202110/921642_FK8RFEBE37G5WJC.jpg)

windbg:

![](https://bbs.kanxue.com/upload/attach/202110/921642_Z9NCW5U92YZQ35P.jpg)

```
lm  查看所有已加载模块
lm m H* 设置过滤，查找HEVD模块
lm m HEVD
```

![](https://bbs.kanxue.com/upload/attach/202110/921642_N9W8VAA4D6UVB5T.jpg)

（2）分析漏洞点

对`BufferOverflowStack.c`源码进行分析，漏洞函数`TriggerBufferOverflowStack`，存在明显的栈溢出漏洞，`RtlCopyMemory`函数没有对`KernelBuffer`大小进行验证，直接将`size`大小的`userbuffer`传入缓冲区，没有进行大小的限制，(例如安全版本的`sizeof(KernelBuffer)`),因此造成栈溢出，攻击返回地址，使之指向构造的payload。

```cpp
#define BUFFER_SIZE 512
// sizeof(KernelBuffer) 0x800

TriggerBufferOverflowStack(
    _In_ PVOID UserBuffer,
    _In_ SIZE_T Size
)
{
    NTSTATUS Status = STATUS_SUCCESS;
    ULONG KernelBuffer[BUFFER_SIZE] = { 0 };

    PAGED_CODE();

    __try
    {
        ProbeForRead(UserBuffer, sizeof(KernelBuffer), (ULONG)__alignof(UCHAR));
        DbgPrint("[+] UserBuffer: 0x%p\n", UserBuffer);
        DbgPrint("[+] UserBuffer Size: 0x%X\n", Size);
        DbgPrint("[+] KernelBuffer: 0x%p\n", &KernelBuffer);
        DbgPrint("[+] KernelBuffer Size: 0x%X\n", sizeof(KernelBuffer));

#ifdef SECURE
        RtlCopyMemory((PVOID)KernelBuffer, UserBuffer, sizeof(KernelBuffer)); // 安全版本
#else
        DbgPrint("[+] Triggering Buffer Overflow in Stack\n");
        RtlCopyMemory((PVOID)KernelBuffer, UserBuffer, Size); // 不安全版本：存在栈溢出漏洞，未对传进KernelBuffer大小进行限制。
#endif
    }
    __except (EXCEPTION_EXECUTE_HANDLER)
    {
        Status = GetExceptionCode();
        DbgPrint("[-] Exception Code: 0x%X\n", Status);
    }

    return Status;
}
```

## 四：漏洞利用

利用驱动与R3通信，能够直接读取R3地址，构造`exp`

1.获取栈中`KernelBuffer`到返回地址的偏移

`ida`中打开`HEVD.sys`

![](https://bbs.kanxue.com/upload/attach/202110/921642_QAX2C39MFBQDF7M.jpg)

`KernelBuffer`距离`ebp` `81Ch`，想覆盖到返回地址需要`81C+4+4 =824h`，也可以使用`windbg`结合断点，在`TriggerBufferOverflowStack`、`RtlCopyMemory`函数处下断点，判断偏移。

2.构造`exp`（官方`exp`）
