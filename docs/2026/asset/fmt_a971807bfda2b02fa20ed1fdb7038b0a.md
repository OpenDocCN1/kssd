
```c
    *IsShadowCopy = FALSE;

#if WINVER >= 0x0501

    if (IS_WINDOWS2000())
        {
#endif

                UNREFERENCED_PARAMETER( StorageStackDeviceObject );
                return STATUS_SUCCESS;

#if WINVER >= 0x0501

        }

        if (IS_WINDOWSXP())
        {

                UNICODE_STRING volSnapDriverName;
                WCHAR buffer[MAX_DEVNAME_LENGTH];
                PUNICODE_STRING storageDriverName;
                ULONG returnedLength;
                NTSTATUS status;

                if (FILE_DEVICE_DISK != StorageStackDeviceObject->DeviceType)
                {
                        return STATUS_SUCCESS;
                }

                storageDriverName = (PUNICODE_STRING) buffer;
                RtlInitEmptyUnicodeString( storageDriverName, Add2Ptr( storageDriverName, sizeof( UNICODE_STRING ) ), sizeof( buffer ) - sizeof( UNICODE_STRING ) );
                status = ObQueryNameString( StorageStackDeviceObject, (POBJECT_NAME_INFORMATION)storageDriverName, storageDriverName->MaximumLength, &returnedLength );
                if (!NT_SUCCESS( status ))
                {
                        return status;
                }

                RtlInitUnicodeString( &volSnapDriverName, L"\\Driver\\VolSnap" );
                if (RtlEqualUnicodeString( storageDriverName, &volSnapDriverName, TRUE ))
                {
                        *IsShadowCopy = TRUE;
                }
                else
                {
                        NOTHING;
                }

                return STATUS_SUCCESS;

        }

        else
        {

                PIRP irp;
                KEVENT event;
                IO_STATUS_BLOCK iosb;
                NTSTATUS status;

                if (FILE_DEVICE_VIRTUAL_DISK != StorageStackDeviceObject->DeviceType)
                {
                        return STATUS_SUCCESS;
                }

                KeInitializeEvent( &event, NotificationEvent, FALSE );

           /*
                *Microsoft WDK官方文档对 IOCTL_DISK_IS_WRITABLE是这样解释的：
                *Determines whether a disk is writable.
                *The Status field can be set to STATUS_SUCCESS, or possibly to STATUS_INSUFFICIENT_RESOURCES, STATUS_IO_DEVICE_ERROR, or STATUS_MEDIA_WRITE_PROTECTED.
                *
                *IOCTL_DISK_IS_WRITABLE是没有输入也没有输出的。
                */

                irp = IoBuildDeviceIoControlRequest( IOCTL_DISK_IS_WRITABLE,
                                             StorageStackDeviceObject,
                                             NULL,
                                             0,
                                             NULL,
                                             0,
                                             FALSE,
                                             &event,
                                             &iosb );

                if (irp == NULL)
                {
                        return STATUS_INSUFFICIENT_RESOURCES;
                }

                status = IoCallDriver( StorageStackDeviceObject, irp );

                if (status == STATUS_PENDING)
                {
                        (VOID)KeWaitForSingleObject( &event,
                                         Executive,
                                         KernelMode,
                                         FALSE,
                                         NULL );
                        status = iosb.Status;
                }

                if (STATUS_MEDIA_WRITE_PROTECTED == status)
                {
                        *IsShadowCopy = TRUE;
                        status = STATUS_SUCCESS;
                }

                return status;

        }

#endif

}
```

上面的代码可以全部复制下来，并编译通过。

我是在`WDK7600.16385.1`环境下顺利编译通过的。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
