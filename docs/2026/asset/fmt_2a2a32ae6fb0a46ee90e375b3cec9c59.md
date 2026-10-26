
# [原创]我写的模拟挂

发出来，一为抛砖引玉，论坛应该也有很多xd想写外挂，可以参考下；二来想出去找份工作，本人年近30岁，而且还没有编码的工作经验，没有信心，希望大家能给点意见。

驱动最初参考了<http://bbs.pediy.com/showthread.php?t=101653>中的代码，因为是根据特征码搜索，不爽，后面又根据寒江独钓中的代码作了修改(这里要澄清下，不是为寒江独钓打广告，反而我觉得那书写得不清不楚的)。

原理：通过直接调用`Kbdclass`的回调函数`KeyboardClassServiceCallback`直接给上层发送键盘驱动，就可以实现模拟键盘操作，鼠标类似。

通过`windbg`查看类设备下面的端口设备(`i8042prt`)或`usb`设备(`kbdhid`)，其设备对象中的`DeviceExtension`里面保存了设备对象与`KeyboardClassServiceCallback`回调函数，设备对象保存在回调函数前面一个地址中。

这个是驱动扩展结构，用来保存查找到的设备对象和回调函数，避免直接使用全局变量

```c
typedef struct _DEVICE_EXTENSION {
    PDEVICE_OBJECT       kbdDeviceObject;        //键盘类设备对象
    PDEVICE_OBJECT       mouDeviceObject;        //鼠标类设备对象
    MY_KEYBOARDCALLBACK  My_KbdCallback;         //KeyboardClassServiceCallback函数
    MY_MOUSECALLBACK     My_MouCallback;         //MouseClassServiceCallback函数
}DEVICE_EXTENSION, *PDEVICE_EXTENSION;
```

下面是查找`KeyboardClassServiceCallback`的关键函数，鼠标设备查找方法类似，我合成了一个函数

```c
NTSTATUS GetKmclassInfo(PDEVICE_OBJECT DeviceObject, USHORT Index)
{
    NTSTATUS           status;
    UNICODE_STRING     ObjectName;
    PCWSTR             kmhidName, kmclassName, kmName;
    PVOID              kmDriverStart;
    ULONG              kmDriverSize;
    PVOID*             TargetDeviceObject;
    PVOID*             TargetclassCallback;
    PDEVICE_EXTENSION  deviceExtension;
    PDRIVER_OBJECT     kmDriverObject       = NULL;
    PDRIVER_OBJECT     kmclassDriverObject  = NULL;
    deviceExtension = (PDEVICE_EXTENSION)DeviceObject->DeviceExtension;
    switch(Index)
    {
    case KEYBOARD_DEVICE:
        kmName              = L"kbd";
        kmhidName           = L"\\Driver\\kbdhid";
        kmclassName         = L"\\Driver\\kbdclass";
        TargetDeviceObject  = (PVOID*)&(deviceExtension->kbdDeviceObject);
        TargetclassCallback = (PVOID*)&(deviceExtension->My_KbdCallback);
        break;
    case MOUSE_DEVICE:
        kmName              = L"mou";
        kmhidName           = L"\\Driver\\mouhid";
        kmclassName         = L"\\Driver\\mouclass";
        TargetDeviceObject  = (PVOID*)&(deviceExtension->mouDeviceObject);
        TargetclassCallback = (PVOID*)&(deviceExtension->My_MouCallback);
        break;
    default:
        return STATUS_INVALID_PARAMETER;
    }
    // 通过USB类设备获取驱动对象
    RtlInitUnicodeString(&ObjectName, kmhidName);
    status = ObReferenceObjectByName(&ObjectName,
        OBJ_CASE_INSENSITIVE,
        NULL,
        FILE_READ_ACCESS,
        *IoDriverObjectType,
        KernelMode,
        NULL,
        (PVOID*)&kmDriverObject);
    if(!NT_SUCCESS(status))
    {
        // 通过i8042prt获取驱动对象
        RtlInitUnicodeString(&ObjectName, L"\\Driver\\i8042prt");
        status = ObReferenceObjectByName(&ObjectName,
            OBJ_CASE_INSENSITIVE,
            NULL,
            FILE_READ_ACCESS,
            *IoDriverObjectType,
            KernelMode,
            NULL,
            (PVOID*)&kmDriverObject);
        if(!NT_SUCCESS(status))
        {
            KdPrint(("Couldn't Get the i8042prt Driver Object\n"));
            return status;
        }
    }
    // 通过kmclass获取键盘鼠标类驱动对象
    RtlInitUnicodeString(&ObjectName, kmclassName);
    status = ObReferenceObjectByName(&ObjectName,
        OBJ_CASE_INSENSITIVE,
        NULL,
        FILE_READ_ACCESS,
        *IoDriverObjectType,
        KernelMode,
        NULL,
        (PVOID*)&kmclassDriverObject);
    if(!NT_SUCCESS(status))
    {
        KdPrint(("Couldn't Get the kmclass Driver Object\n"));
        return status;
    }
    else
    {
        kmDriverStart = kmclassDriverObject->DriverStart;
        kmDriverSize  = kmclassDriverObject->DriverSize;
    }
    ULONG             DeviceExtensionSize;
    PULONG            kmDeviceExtension;
    PDEVICE_OBJECT    kmTempDeviceObject;
    PDEVICE_OBJECT    kmclassDeviceObject;
    PDEVICE_OBJECT    kmDeviceObject = kmDriverObject->DeviceObject;
    while (kmDeviceObject)
    {
        kmTempDeviceObject = kmDeviceObject;
        while (kmTempDeviceObject)
        {
            kmDeviceExtension   = (PULONG)kmTempDeviceObject->DeviceExtension;
            kmclassDeviceObject = kmclassDriverObject->DeviceObject;
            DeviceExtensionSize = ((ULONG)kmTempDeviceObject->DeviceObjectExtension - (ULONG)kmTempDeviceObject->DeviceExtension) / 4;
            while (kmclassDeviceObject)
            {
                for (ULONG i = 0; i < DeviceExtensionSize; i++)
                {
                    if (kmDeviceExtension[i] == (ULONG)kmclassDeviceObject &&
                        kmDeviceExtension[i + 1] > (ULONG)kmDriverStart    &&
                        kmDeviceExtension[i + 1] < (ULONG)kmDriverStart + kmDriverSize)
                    {
                        // 将获取到的设备对象保存到自定义扩展设备结构
                        *TargetDeviceObject  = (PVOID)kmDeviceExtension[i];
                        *TargetclassCallback = (PVOID)kmDeviceExtension[i + 1];
                        KdPrint(("%SDeviceObject == 0x%x\n", kmName, kmDeviceExtension[i]));
                        KdPrint(("%SClassServiceCallback == 0x%x\n", kmName, kmDeviceExtension[i + 1]));
                        return STATUS_SUCCESS;
                    }
                }
                kmclassDeviceObject = kmclassDeviceObject->NextDevice;
            }
            kmTempDeviceObject = kmTempDeviceObject->AttachedDevice;
        }
        kmDeviceObject = kmDeviceObject->NextDevice;
    }
    return STATUS_UNSUCCESSFUL;
}
```

## 应用层模拟键盘操作函数

```c
BOOL KeyboardButton(USHORT VirtualKey, USHORT Flags)
{
    KEYBOARD_INPUT_DATA  kid ;
    DWORD dwOutput;
    HANDLE hDevice = CreateFile(KEYMOUSE_WIN32_DEVICE_NAME, GENERIC_READ | GENERIC_WRITE,
        0, NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL);
    if (hDevice == INVALID_HANDLE_VALUE)
        return FALSE;
    memset(&kid, 0, sizeof(KEYBOARD_INPUT_DATA));
    kid.Flags    = Flags;
    kid.MakeCode = (USHORT)MapVirtualKey(VirtualKey, 0);
    BOOL bRet = DeviceIoControl(hDevice, IOCTL_KEYBOARD, &kid, sizeof(KEYBOARD_INPUT_DATA), NULL, 0, &dwOutput, NULL);
    if (!bRet)
        TRACE(_T("Error! please open the simulate kmclass driver!\n"));
    CloseHandle(hDevice);
    return bRet;
}
```

## 模拟鼠标的函数

```c
BOOL MouseMove(LONG dx, LONG dy, USHORT Flags)
{
    MOUSE_INPUT_DATA  mid ;
    DWORD dwOutput;
    HANDLE hDevice = CreateFile(KEYMOUSE_WIN32_DEVICE_NAME, GENERIC_READ | GENERIC_WRITE,
        0, NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL);
    if (hDevice == INVALID_HANDLE_VALUE)
        return FALSE;
    memset(&mid, 0, sizeof(MOUSE_INPUT_DATA));
    mid.Flags = Flags;
    switch (mid.Flags)
    {
    case MOUSE_MOVE_RELATIVE:
        mid.LastX = dx;
        mid.LastY = dy;
        break;
    case MOUSE_MOVE_ABSOLUTE:
        mid.LastX = dx * 0xffff / GetSystemMetrics(SM_CXSCREEN);
        mid.LastY = dy * 0xffff / GetSystemMetrics(SM_CYSCREEN);
        break;
    default:
        TRACE(_T("Flags: Parameter error!\n"));
        return FALSE;
    }
    BOOL bRet = DeviceIoControl(hDevice, IOCTL_MOUSE, &mid, sizeof(MOUSE_INPUT_DATA), NULL, 0, &dwOutput, NULL);
    if (!bRet)
        TRACE(_T("Error! please start the kmclass driver!\n"));
    CloseHandle(hDevice);
    return bRet;
}
```

## 前台窗口找图实现

`bmp`类定义

```c
class Cbm {
private:
        BITMAPFILEHEADER   bmfh;                 // 位图文件头
        BITMAPINFOHEADER   bmih;                 // 位图信息头
        PBYTE              pBits;                // 位图像素位指针
        int                cBits;                // 位图每行所用字节总数
        int                cxDib;                // 位图水平像素宽度
        int                cyDib;                // 位图垂直像素高度
        void SetcBits() {cBits = ((cxDib * bmih.biBitCount + 31) & ~31) >> 3;}
        void SetcxDib() {cxDib = bmih.biWidth;}
        void SetcyDib() {cyDib = bmih.biHeight;}
....
}
```
