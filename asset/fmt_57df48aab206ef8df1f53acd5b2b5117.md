
```c
    DbgPrint("IRP_MJ_CLOSE Received : Begin\r\n");  

    p_IO_STK = IoGetCurrentIrpStackLocation(Irp);  

    p_DVCEXT = DeviceObject->DeviceExtension;  

    IoReleaseRemoveLock(&p_DVCEXT->RemoveLock,  

        p_IO_STK->FileObject);  

    CompleteRequest(Irp, STATUS_SUCCESS, 0);  

    DbgPrint("IRP_MJ_CLOSE Received : Begin\r\n");  

    return STATUS_SUCCESS;  

}  

NTSTATUS  

  PsdoDispatchDeviceControl(  

    IN PDEVICE_OBJECT  DeviceObject,  

    IN PIRP  Irp  

    )  

{  

    ULONG code, cbin, cbout, info, pwrinf_size;  

    PIO_STACK_LOCATION p_IO_STK;  

    PDEVICE_EXTENSION p_DVCEXT;  

    PDEVICE_POWER_INFORMATION pValue;  

    ULONG IdxPwrState;  

    NTSTATUS status;  

    p_IO_STK = IoGetCurrentIrpStackLocation(Irp);  

    p_DVCEXT = DeviceObject->DeviceExtension;  

    //DeviceIoControl 访问码  

    code = p_IO_STK->Parameters.DeviceIoControl.IoControlCode;  

    //输入缓冲区长度  

    cbin = p_IO_STK->Parameters.DeviceIoControl.InputBufferLength;  

    //输出缓冲区长度  

    cbout = p_IO_STK->Parameters.DeviceIoControl.OutputBufferLength;  

    IoAcquireRemoveLock(&p_DVCEXT->RemoveLock, Irp);  

    switch(code)  

    {  

    case IOCTL_READ_DEVICE_INFO:  

        if (p_DVCEXT->Device_Description.Length > cbout)  

        {  

            cbout = p_DVCEXT->Device_Description.Length;  

            info = cbout;  

        } else {  

            info = p_DVCEXT->Device_Description.Length;  

        }  

        //将数据复制到系统缓冲区，I/O 管理器负责将数据复制到用户缓冲区内     

        RtlCopyMemory(Irp->AssociatedIrp.SystemBuffer,  

            p_DVCEXT->Device_Description.Buffer,  

            info);  

        status = STATUS_SUCCESS;  

        break;  

    //可跳过不看  

    case IOCTL_READ_POWER_INFO:  

        pwrinf_size = sizeof(DEVICE_POWER_INFORMATION);  

        if (pwrinf_size > cbout)  

        {  

            cbout = pwrinf_size;  

            info = cbout;  

        } else {  

            info = pwrinf_size;  

        }  

        //Display Related Device Power State  

        DbgPrint("Support Query Device Capability : %$r\n",  

            Global_PowerInfo_Ptr->SupportQueryCapability ? "Yes" : "No");  

        DbgPrint("DeviceD1 : %d\r\n", Global_PowerInfo_Ptr->DeviceD1);  

        DbgPrint("DeviceD2 : %d\r\n", Global_PowerInfo_Ptr->DeviceD2);  

        DbgPrint("WakeFromD0 : %d\r\n", Global_PowerInfo_Ptr->WakeFromD0);  

        DbgPrint("WakeFromD1 : %d\r\n", Global_PowerInfo_Ptr->WakeFromD1);  

        DbgPrint("WakeFromD2 : %d\r\n", Global_PowerInfo_Ptr->WakeFromD2);  

        DbgPrint("WakeFromD3 : %d\r\n", Global_PowerInfo_Ptr->WakeFromD3);  

        DbgPrint("SystemWake : %d\r\n", Global_PowerInfo_Ptr->SystemWake);  

        DbgPrint("DeviceWake : %d\r\n", Global_PowerInfo_Ptr->DeviceWake);  

        for (IdxPwrState = 0;  

            IdxPwrState < PowerSystemMaximum;  

            IdxPwrState++)  

        {  

            DbgPrint("DeviceState[%d] : %d\r\n",  

                IdxPwrState,  

                Global_PowerInfo_Ptr->DeviceState[IdxPwrState]);  

        }  

#ifdef _DEF_HANDLE_BY_POWER_INFO_STRUCTURE  

        pValue = (PDEVICE_POWER_INFORMATION)  

            Irp->AssociatedIrp.SystemBuffer;  

        pValue->SupportQueryCapability = Global_PowerInfo_Ptr->SupportQueryCapability;  

        pValue->DeviceD1 = Global_PowerInfo_Ptr->DeviceD1;  

        pValue->DeviceD2 = Global_PowerInfo_Ptr->DeviceD2;  

        pValue->DeviceWake = Global_PowerInfo_Ptr->DeviceWake;  

        pValue->SystemWake = Global_PowerInfo_Ptr->SystemWake;  

        pValue->WakeFromD0 = Global_PowerInfo_Ptr->WakeFromD0;  

        pValue->WakeFromD1 = Global_PowerInfo_Ptr->WakeFromD1;  

        pValue->WakeFromD2 = Global_PowerInfo_Ptr->WakeFromD2;  

        pValue->WakeFromD3 = Global_PowerInfo_Ptr->WakeFromD3;  

        for (IdxPwrState = 0;  

            IdxPwrState < PowerSystemMaximum;  

            IdxPwrState++)  

        {  

            pValue->DeviceState[IdxPwrState] =  

            Global_PowerInfo_Ptr->DeviceState[IdxPwrState];  

        }  

#else  

        RtlCopyMemory(Irp->AssociatedIrp.SystemBuffer,  

            Global_PowerInfo_Ptr,  

            info);  

#endif  

        status = STATUS_SUCCESS;  

        break;  

    default:  

        info = 0;  

        status = STATUS_INVALID_DEVICE_REQUEST;  

        break;  

    }  

    IoReleaseRemoveLock(&p_DVCEXT->RemoveLock, Irp);  

    CompleteRequest(Irp, STATUS_SUCCESS, info);  

    return status;  

}  

/*  
NTSTATUS CompleteRequest(  
         IN PIRP Irp,  
         IN NTSTATUS status,  
         IN ULONG_PTR info)  
{  
    Irp->IoStatus.Status = status;  
    Irp->IoStatus.Information = info;  
    //驱动程序已经完成对 IRP 的处理，并把 IRP 交给 I/O 管理器。  
    IoCompleteRequest(Irp, IO_NO_INCREMENT);  
    return status;  
}  

*/
```

### 用户层部分

```c
#include "stdafx.h"  

#include <winioctl.h>  

/*  

Self-Defined I/O Control Code  

*/  

//I/O Control Code for Device Information retrieval  

//注意下面的缓冲模式与驱动程序中 adddevice 函数中初始的缓冲模式有些不同  

//解释如下：  

//那时我指出， 当读写请求到来时， 你必须记住在 AddDevice  

//中指出的访问用户模式缓冲区所使用的模式，是 buffered 模式还是 direct 模式(或者两者都不是)。控制请求也  

//利用这些寻址方式，但有一些差异。不是用设备对象中的标志来指定全局寻址方式，而是用 IOCTL 中的功能码  

//的低两位来为每个 IOCTL 指定寻址方式  

//此外，IOCTL 中指定的缓冲方式并不影响普通读写 IRP 的寻址缓冲区。  

//#define CTL_CODE( DeviceType, Function, Method, Access ) (                \  

    ((DeviceType) << 16) | ((Access) << 14) | ((Function) << 2) | (Method) )  

//#define METHOD_BUFFERED             0  

//#define METHOD_IN_DIRECT            1  

//#define METHOD_OUT_DIRECT           2  

//#define METHOD_NEITHER             3  

#define IOCTL_READ_DEVICE_INFO    \  

    CTL_CODE(                    \  

         FILE_DEVICE_UNKNOWN,    \  

         0x800,                  \  

         METHOD_BUFFERED,        \  

         FILE_ANY_ACCESS)  

int main(int argc, char* argv[])  

{  

    //倒数第二个参数：我们以不指定 FILE_FLAG_OVERLAPPED 标志的情况下打开设备句柄。因此，在这之后对  

    //DeviceIoControl 的调用将不返回，直到驱动程序对我们的请求做出回答。  

    //用来设置同步或异步  

    HANDLE hdevice = CreateFile("\\\\.\\PSEUDODEVICE", GENERIC_READ | GENERIC_WRITE, 0, NULL, OPEN_EXISTING, 0, NULL);  

    if (hdevice == INVALID_HANDLE_VALUE)  

    {  

        printf("Unable to open PSEUDODEVICE device - error %d\n", GetLastError());  

        return 1;  

    }  

    wchar_t answer[512] = {'\0'};  

    DWORD junk;  

    //只需要输出缓冲区，注意 answer 为 unicode  

    if (DeviceIoControl(hdevice, IOCTL_READ_DEVICE_INFO, NULL, 0, answer, sizeof(answer), &junk, NULL))  

    {  

        answer[junk] = 0;  

        MessageBoxW(NULL,answer,L"look me", 0);  

        wprintf(L"%s",answer);  

    }  

    else  

        printf("Error %d in call to DeviceIoControl\n", GetLastError());  

    CloseHandle(hdevice);  

    return 0;  

}
```

### 如何编译代码

进入 `ShareFiles` 文件夹，分别 Build `"Basic"`, `"PnP"`, `"PM"`, 然后 Build `"IOCTL_PW"`

### 如何安装驱动

这里使用“控制面板”中的“添加硬件”的方式，选中从“磁盘安装”，选择 `install` 文件夹内的 `inf` 文件进行安装即可（普通的动态加载驱动程序的方法好像不适用于 `WDM` 驱动，每次都蓝屏）

### 上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `PseudoDriver.rar`

|
