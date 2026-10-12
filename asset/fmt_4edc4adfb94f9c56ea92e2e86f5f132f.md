
```c
    ns = ControlDispatcher(pDeviceContext,  
                          pisl->Parameters.DeviceIoControl.IoControlCode,  
                          (BYTE *) pIrp->AssociatedIrp.SystemBuffer,  
                          pisl->Parameters.DeviceIoControl.InputBufferLength,  
                          (BYTE *) pIrp->AssociatedIrp.SystemBuffer,  
                          pisl->Parameters.DeviceIoControl.OutputBufferLength,  
                          &dwInfo);  
    MUTEX_RELEASE(DumpMutex);  
    break;  
   }  
  }  
  pIrp->IoStatus.Status = ns;  
  pIrp->IoStatus.Information = dwInfo;  
  IoCompleteRequest (pIrp, IO_NO_INCREMENT);  
  return ns;  
}  

NTSTATUS DriverDispatcher(PDEVICE_OBJECT pDeviceObject, PIRP pIrp)  
{  
   return (pDeviceObject == g_pDeviceObject ?  
      DeviceDispatcher(g_pDeviceContext, pIrp)  
      : STATUS_INVALID_PARAMETER_1);  
}  

VOID DriverUnload(PDRIVER_OBJECT pDriverObject)  
{  
   UNHOOK_SYSCALL(ZwCreateFile, HookedZwCreateFile, RealZwCreateFile);  
   UNHOOK_SYSCALL(ZwOpenKey, HookedZwOpenKey, RealZwOpenKey);  
   UnmapServiceTable(KeServiceTablePointers);  
    IoDeleteSymbolicLink(&usSymbolicLinkName);  
    IoDeleteDevice(pDriverObject->DeviceObject);  
}  

NTSTATUS DriverInitialize(PDRIVER_OBJECT pDriverObject,  
                PUNICODE_STRING pusRegistryPath)  
{  
    PDEVICE_OBJECT pDeviceObject = NULL;  
    NTSTATUS ns = STATUS_DEVICE_CONFIGURATION_ERROR;  
    RtlInitUnicodeString(&usDeviceName, DeviceName);  
    RtlInitUnicodeString(&usSymbolicLinkName, SymLinkName);  
    if ((ns = IoCreateDevice(pDriverObject, sizeof (DEVICE_CONTEXT),  
      &usDeviceName, FILE_DEVICE_ANTIMIDA, 0, FALSE,  
      &pDeviceObject)) == STATUS_SUCCESS)  
   {  
        if ((ns = IoCreateSymbolicLink(&usSymbolicLinkName,  
         &usDeviceName)) == STATUS_SUCCESS)  
      {  
            g_pDeviceObject  = pDeviceObject;  
            g_pDeviceContext = pDeviceObject->DeviceExtension;  
            g_pDeviceContext->pDriverObject = pDriverObject;  
            g_pDeviceContext->pDeviceObject = pDeviceObject;  
      }  
      else  
      {  
         IoDeleteDevice(pDeviceObject);  
      }  
   }  
   return ns;  
}  

NTSTATUS DriverEntry(PDRIVER_OBJECT pDriverObject,  
            PUNICODE_STRING pusRegistryPath)  
{  
   PDRIVER_DISPATCH *ppdd;  
    NTSTATUS ns = STATUS_DEVICE_CONFIGURATION_ERROR;  
    if ((ns = DriverInitialize(pDriverObject, pusRegistryPath)) == STATUS_SUCCESS)  
   {  
        ppdd = pDriverObject->MajorFunction;  
        ppdd[IRP_MJ_CREATE               ] =  
        ppdd[IRP_MJ_CREATE_NAMED_PIPE    ] =  
        ppdd[IRP_MJ_CLOSE                ] =  
        ppdd[IRP_MJ_READ                 ] =  
        ppdd[IRP_MJ_WRITE                ] =  
        ppdd[IRP_MJ_QUERY_INFORMATION    ] =  
        ppdd[IRP_MJ_SET_INFORMATION      ] =  
        ppdd[IRP_MJ_QUERY_EA             ] =  
        ppdd[IRP_MJ_SET_EA               ] =  
        ppdd[IRP_MJ_FLUSH_BUFFERS        ] =  
        ppdd[IRP_MJ_QUERY_VOLUME_INFORMATION] =  
        ppdd[IRP_MJ_SET_VOLUME_INFORMATION ] =  
        ppdd[IRP_MJ_DIRECTORY_CONTROL    ] =  
        ppdd[IRP_MJ_FILE_SYSTEM_CONTROL  ] =  
        ppdd[IRP_MJ_DEVICE_CONTROL       ] =  
        ppdd[IRP_MJ_INTERNAL_DEVICE_CONTROL ] =  
        ppdd[IRP_MJ_SHUTDOWN             ] =  
        ppdd[IRP_MJ_LOCK_CONTROL         ] =  
        ppdd[IRP_MJ_CLEANUP             ] =  
        ppdd[IRP_MJ_CREATE_MAILSLOT     ] =  
        ppdd[IRP_MJ_QUERY_SECURITY      ] =  
        ppdd[IRP_MJ_SET_SECURITY        ] =  
        ppdd[IRP_MJ_POWER               ] =  
        ppdd[IRP_MJ_SYSTEM_CONTROL      ] =  
        ppdd[IRP_MJ_DEVICE_CHANGE       ] =  
        ppdd[IRP_MJ_QUERY_QUOTA         ] =  
        ppdd[IRP_MJ_SET_QUOTA           ] =  
        ppdd[IRP_MJ_PNP                 ] = DriverDispatcher;  
        pDriverObject->DriverUnload      = DriverUnload;  
      MUTEX_INIT(DumpMutex);  
      KeServiceTablePointers = MapServiceTable(&ServiceIsHooked);  
      HOOK_SYSCALL(ZwCreateFile, HookedZwCreateFile, RealZwCreateFile);  
      HOOK_SYSCALL(ZwOpenKey, HookedZwOpenKey, RealZwOpenKey);  
   }  
   return ns;  
}
```

好了，我现在很累了。没有什么要解释的了，如果你有疑惑的话，google 在那里等着你。这项工程的继续取决于我得到的信息。我对逆向 `TheMida` 没有兴趣。

译者：我现在同样也是累得不行了。。。好长的程序代码，不过最后还是发现有词还是不会译。郁闷。。。

`4dd466d0e1e0cc123afbe504da2462ca`
