
```c
DriverObject->MajorFunction[IRP\_MJ\_DEVICE\_CONTROL] = JhxxsDispatchDeviceControl;

DriverObject->DriverUnload                         = JhxxsUnload;

if (NT\_SUCCESS(status))

        return status;

__failed:

    if (fSymbolicLink)

            IoDeleteSymbolicLink(&dosDeviceName);

    if (deviceObject)

            IoDeleteDevice(deviceObject);

    return status;

}

NTSTATUS

JhxxsDispatchCreate(

        IN PDEVICE\_OBJECT                DeviceObject,

        IN PIRP                                        Irp

        )

{

        NTSTATUS status = STATUS\_SUCCESS;

        Irp->IoStatus.Information = 0;

    Irp->IoStatus.Status = status;

    IoCompleteRequest(Irp, IO\_NO\_INCREMENT);

    return status;

}

NTSTATUS

JhxxsDispatchClose(

        IN PDEVICE\_OBJECT                DeviceObject,

        IN PIRP                                        Irp

        )

{

        NTSTATUS status = STATUS\_SUCCESS;

    Irp->IoStatus.Information = 0;

        Irp->IoStatus.Status = status;

    IoCompleteRequest(Irp, IO\_NO\_INCREMENT);

    return status;

}

NTSTATUS

JhxxsDispatchDeviceControl(

        IN PDEVICE\_OBJECT                DeviceObject,

        IN PIRP                                        Irp

        )

{

        NTSTATUS                        status = STATUS\_SUCCESS;

    PIO\_STACK\_LOCATION        irpStack;

    PDEVICE\_EXTENSION        deviceExtension;

        PVOID                                ioBuf;

    ULONG                                inBufLength, outBufLength;

        ULONG                                ioControlCode;

        UCHAR               \*buff =0;

        ULONG               OutByteCount =0;

        HANDLE              Writehandel;

        PVOID               WriteDstAddr;

        PVOID               WriteSrcAddr;

        ULONG               WriteSize;

        NTSTATUS            WriteReturn;

        HANDLE              Readhandel;

        PVOID               ReadBaseAddr;

        PVOID               ReadBuffer;

        ULONG               ReadSize;

        NTSTATUS            ReadReturn;

        ULONG               OpenPid;

        PHANDLE             PProcessHandle;

        NTSTATUS            OpenReturn;

    irpStack = IoGetCurrentIrpStackLocation(Irp);

    deviceExtension = (PDEVICE\_EXTENSION)DeviceObject->DeviceExtension;

        Irp->IoStatus.Information = 0;

        ioBuf = Irp->AssociatedIrp.SystemBuffer;

        inBufLength = irpStack->Parameters.DeviceIoControl.InputBufferLength;

    outBufLength = irpStack->Parameters.DeviceIoControl.OutputBufferLength;

    ioControlCode = irpStack->Parameters.DeviceIoControl.IoControlCode;

    switch (ioControlCode)

        {

        case  0X0022E004:

                {

                 buff=(UCHAR \*)Irp->AssociatedIrp.SystemBuffer ;

                 memmove(&Writehandel,&buff[0],4);

                 memmove(&WriteDstAddr,&buff[4],4);

                 memmove(&WriteSrcAddr,&buff[8],4);

                 memmove(&WriteSize,&buff[12],4);

                 WriteReturn=MyWriteMemory(Writehandel,WriteDstAddr,WriteSrcAddr,WriteSize);

                 memmove(Irp->AssociatedIrp.SystemBuffer,&WriteReturn,4);

                 OutByteCount=4;

                 break;

                }

        case  0X0022E008:

                {

                 buff=(UCHAR \*)Irp->AssociatedIrp.SystemBuffer ;

                 memmove(&Readhandel,&buff[0],4);

                 memmove(&ReadBaseAddr,&buff[4],4);

                 memmove(&ReadBuffer,&buff[8],4);

                 memmove(&ReadSize,&buff[12],4);

                 ReadReturn=MyReadMemory(Readhandel,ReadBaseAddr,ReadBuffer,ReadSize);

                 memmove(&buff[0],&ReadReturn,4);

                 OutByteCount=4;

                 break;

                }

        case  0X0022E000:

                {

                 OpenReturn = MyOpenProcess(\*(PULONG)ioBuf,ioBuf);

                 buff=(UCHAR \*)Irp->AssociatedIrp.SystemBuffer ;

                 memmove(&buff[4],&OpenReturn,4);

                 OutByteCount=8;

                 break;

                }

        case IOCTL\_JHXXS\_HELLO:

                {

            break;

                }

    default:

        status = STATUS\_INVALID\_PARAMETER;

                break;

        }

        Irp->IoStatus.Status = status;

        Irp->IoStatus.Information = OutByteCount;

        IoCompleteRequest(Irp, IO\_NO\_INCREMENT);

    return status;

}

VOID

JhxxsUnload(

        IN PDRIVER\_OBJECT                DriverObject

        )

{

    UNICODE\_STRING dosDeviceName;

        RtlInitUnicodeString(&dosDeviceName, JHXXS\_DOS\_DEVICE\_NAME\_W);

    IoDeleteSymbolicLink(&dosDeviceName);

        IoDeleteDevice(DriverObject->DeviceObject);

}
```

完整可编译代码如下[驱动由 BC++编写 WINDDK编译]

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [jhxxs.rar](javascript:void(0);)

  （10.63kb，459次下载）
