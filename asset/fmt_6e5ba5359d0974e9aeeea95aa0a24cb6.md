
```
NTSTATUS handle_get_dbdata(PIRP Irp, PIO_STACK_LOCATION pIoStackIrp, ULONG* pdwDataWritten)

{
  NTSTATUS NtStatus = STATUS_SUCCESS;

  PCHAR pOutputBuffer = NULL;

  int dwwrite = 0;

  if(Irp->MdlAddress)
    {
        pOutputBuffer = MmGetSystemAddressForMdlSafe(Irp->MdlAddress, NormalPagePriority);

    }

  if(pOutputBuffer)
    {

//    DbgPrint("----get data");

//    while(!IsListEmpty(&hdbdata_list))

    if(!IsListEmpty(&hdbdata_list))
    {
      PLIST_ENTRY pEntry;

      pEntry = RemoveHeadList(&hdbdata_list);

    gp_dbdata = CONTAINING_RECORD(pEntry,
                       DB_DATA,
                       ListEntry);

    //if(gp_tmp_dbdata)

    //gp_tmp_dbdata = gp_dbdata;

    //KdPrint(("%d\n",pData->number));

    if(pIoStackIrp->Parameters.DeviceIoControl.OutputBufferLength >strlen(gp_dbdata->db_string))

    strcpy(pOutputBuffer,gp_dbdata->db_string);

    else strcpy(pOutputBuffer,"NULL");

    ExFreePool(gp_dbdata);

    dwwrite = strlen(gp_dbdata->db_string);

//    RemoveHeadList(&hdbdata_list);

//    KeSetEvent(pKEvent,IO_NO_INCREMENT, FALSE);

    //break;

    }

//    strcpy(pOutputBuffer,tmp_dbgprint_string);

//    RtlZeroMemory(tmp_dbgprint_string,256);

//    DbgPrint("----get data");

//    if(pKEvent)

//    KeSetEvent(pKEvent,IO_NO_INCREMENT, FALSE);

  }

  //*pdwDataWritten == 0 表示失败....否则返回真实的..用户程序需要根据这个返回值判断自己提供的缓冲区是否足够

  *pdwDataWritten = dwwrite;

  return NtStatus;

}
```

```
NTSTATUS handle_pass_uevent(PIRP Irp, PIO_STACK_LOCATION pIoStackIrp, ULONG* pdwDataWritten)

{

  NTSTATUS NtStatus = STATUS_SUCCESS;

    // 需要验证一下.

  HANDLE hUserEvent =  *(HANDLE*)Irp->AssociatedIrp.SystemBuffer;

  NtStatus = ObReferenceObjectByHandle(hUserEvent,EVENT_MODIFY_STATE,*ExEventObjectType,KernelMode,

                               (PVOID*)&pKEvent, NULL);

  if(NtStatus!=STATUS_SUCCESS)

  {

    DbgPrint("pass event wrong\n");

  }

  *pdwDataWritten = 0;

  return NtStatus;

}

///////////////////////////
```

```
NTSTATUS get_bpcontext(PIRP Irp, PIO_STACK_LOCATION pIoStackIrp, ULONG* pdwDataWritten)

{

  NTSTATUS NtStatus = STATUS_SUCCESS;

  PCHAR pOutputBuffer = NULL;

  int dwwrite = 0;

  if(Irp->MdlAddress)
    {
        pOutputBuffer = MmGetSystemAddressForMdlSafe(Irp->MdlAddress, NormalPagePriority);

    }

  if(pOutputBuffer)
    {
    if(!IsListEmpty(&hbpdata_list))
    {
      PLIST_ENTRY pEntry;

      pEntry = RemoveHeadList(&hbpdata_list);

    gp_bpdata = CONTAINING_RECORD(pEntry,
                       BP_DATA,
                       ListEntry);

    if(pIoStackIrp->Parameters.DeviceIoControl.OutputBufferLength >= sizeof(KINT_STACK_INFO))
    {
      RtlCopyMemory(pOutputBuffer,&gp_bpdata->bp_context,sizeof(KINT_STACK_INFO));

      dwwrite = sizeof(KINT_STACK_INFO);

    }

    else dwwrite = 0;

    ExFreePool(gp_bpdata);

    gp_bpdata = NULL;

    }

  }

  //*pdwDataWritten == 0 表示失败....否则返回真实的..用户程序需要根据这个返回值判断自己提供的缓冲区是否足够

  *pdwDataWritten = dwwrite;

  return NtStatus;

}
```

```
////////////////////

NTSTATUS get_call_info(PIRP Irp, PIO_STACK_LOCATION pIoStackIrp, ULONG* pdwDataWritten)

{

  NTSTATUS NtStatus = STATUS_SUCCESS;

  PCHAR pOutputBuffer = NULL;

  int dwwrite = 0;

  if(Irp->MdlAddress)
    {
        pOutputBuffer = MmGetSystemAddressForMdlSafe(Irp->MdlAddress, NormalPagePriority);

    }

  if(pOutputBuffer)

    {
    if(!IsListEmpty(&hcalldata_list)&&(can_get_callinfo == TRUE))  // 因为结构被多次用..除非慢满了..否则有数据也不取的..
    {
      PLIST_ENTRY pEntry;

      pEntry = RemoveHeadList(&hcalldata_list);

      gp_callinfo = CONTAINING_RECORD(pEntry,CALL_INFO,ListEntry);

      if(pIoStackIrp->Parameters.DeviceIoControl.OutputBufferLength >= sizeof(CALL_INFO))
      {
        RtlCopyMemory(pOutputBuffer,gp_callinfo,sizeof(CALL_INFO));

        dwwrite = sizeof(CALL_INFO);

        //can_get_callinfo = FALSE;

      }

      else dwwrite = 0;

      ExFreePool(gp_callinfo);

    }

  }

  //*pdwDataWritten == 0 表示失败....否则返回真实的..用户程序需要根据这个返回值判断自己提供的缓冲区是否足够

  *pdwDataWritten = dwwrite;

  return NtStatus;

}
```

```
/////////////////////

NTSTATUS get_eip_info(PIRP Irp, PIO_STACK_LOCATION pIoStackIrp, ULONG* pdwDataWritten)

{

  NTSTATUS NtStatus = STATUS_SUCCESS;

  PCHAR pOutputBuffer = NULL;

  int dwwrite = 0;

  if(Irp->MdlAddress)
    {
        pOutputBuffer = MmGetSystemAddressForMdlSafe(Irp->MdlAddress, NormalPagePriority);

    }

  //DbgPrint("------------------get data 0000000 --------------------");

  if(pOutputBuffer)

    {
    if(!IsListEmpty(&heipinfo_list)&&(can_get_callinfo == TRUE))  // 因为结构被多次用..除非慢满了..否则有数据也不取的..
    {
      PLIST_ENTRY pEntry;

      //DbgPrint("------------------get data --------------------");

      pEntry = RemoveHeadList(&heipinfo_list);

      gp_eipinfo = CONTAINING_RECORD(pEntry,EIP_INFO,ListEntry);

      if(pIoStackIrp->Parameters.DeviceIoControl.OutputBufferLength >= sizeof(EIP_INFO))
      {
        RtlCopyMemory(pOutputBuffer,gp_eipinfo,sizeof(EIP_INFO));

        dwwrite = sizeof(EIP_INFO);

        //can_get_callinfo = FALSE;

      }

      else dwwrite = 0;

      ExFreePool(gp_eipinfo);

    }

  }

  //*pdwDataWritten == 0 表示失败....否则返回真实的..用户程序需要根据这个返回值判断自己提供的缓冲区是否足够

  *pdwDataWritten = dwwrite;

  return NtStatus;

}
```

```
NTSTATUS SysIoControl(PDEVICE_OBJECT DeviceObject, PIRP Irp)

{

    NTSTATUS NtStatus = STATUS_SUCCESS;

    PIO_STACK_LOCATION pIoStackIrp = NULL;

    ULONG dwDataWritten = 0;

  pIoStackIrp = IoGetCurrentIrpStackLocation(Irp);

  if(pIoStackIrp) /* Should Never Be NULL! */
    {
    switch(pIoStackIrp->Parameters.DeviceIoControl.IoControlCode)
    {
    case IOCTL_GET_DBGPRINT_DATA:
      handle_get_dbdata(Irp, pIoStackIrp, &dwDataWritten);
      break;

    case IOCTL_PASS_USER_EVENT:
      handle_pass_uevent(Irp, pIoStackIrp, &dwDataWritten);

      InitializeListHead(&hdbdata_list);

      install_idt_hook(0x2d, (PVOID)my_int_2d);

      break;

    case IOCTL_UNPASS_USER_EVENT:
      if(pKEvent)
      {
        ObDereferenceObject(pKEvent);

        pKEvent = NULL;

      }
      break;

    case IOCTL_INSTALL_INT3:
      InitializeListHead(&hbpdata_list);

      install_idt_hook(0x3, (PVOID)my_interrupt_hook);

      break;

    case IOCTL_UNINSTALL_INT3:
      unistall_idt_hook(0x3);

      break;

    case IOCTL_INSERT_BREAKPOINT:
      // 验证一下..

      insert_breakpoint((PVOID)(*(ULONG*)Irp->AssociatedIrp.SystemBuffer));

      break;

    case IOCTL_GET_BREAKPOINT_CONTEXT:
      get_bpcontext(Irp, pIoStackIrp, &dwDataWritten);

      break;

    case IOCTL_SET_STEPING:
      b_steping = TRUE;

      break;

    case IOCTL_SET_STEPING_COUNT:
      tmp_count = *(int*)Irp->AssociatedIrp.SystemBuffer;

      break;

    case IOCTL_UNSET_STEPING:
      b_steping = FALSE;

      break;

    case IOCTL_INSTALL_INT1:
      InitializeListHead(&hcalldata_list);

      InitializeListHead(&heipinfo_list);

      install_idt_hook(0x1, (PVOID)my_int_1);

      break;

    case IOCTL_UNINSTALL_INT1:
      unistall_idt_hook(0x1);

      break;

    case IOCTL_GET_CALL_DATA:
      get_call_info(Irp, pIoStackIrp, &dwDataWritten);

      break;

    case IOCTL_SET_GET_EIPINFO:
      call_or_address = TRUE;  //// 是否只用一个命令就可以,...
      can_get_callinfo = FALSE; //////////////////////////////////
```
