
```c
 pFileInfo->NextEntryOffset = 0;  

 pFileInfo->Flags = 0;  

 pFileInfo->EaNameLength = TDI\_CONNECTION\_CONTEXT\_LENGTH;  

 memcpy( pFileInfo->EaName,  

  TdiConnectionContext,  

  pFileInfo->EaNameLength + 1 ); //includes NULL terminator  

 // CONNECTION\_CONTEXT is a user defined structure used to sort connections  

 // There is only one connection in this example, so CONNECTION\_CONTEXT is not used  

 pFileInfo->EaValueLength = sizeof(CONNECTION\_CONTEXT);  

 \*(CONNECTION\_CONTEXT\*)(pFileInfo->EaName+(pFileInfo->EaNameLength + 1)) =  

  (CONNECTION\_CONTEXT) connectionContext;  

 status = ZwCreateFile( &TdiEndpointHandle,  

  GENERIC\_READ | GENERIC\_WRITE | SYNCHRONIZE,  

  &TdiAttributes,  

  &IoStatusBlock,  

  0,  

  FILE\_ATTRIBUTE\_NORMAL,  

  FILE\_SHARE\_READ,  

  FILE\_OPEN,  

  0,  

  pFileInfo,  

   sizeof(eaBuffer) );  

 if( !NT\_SUCCESS( status ) )  

 {  

  DbgPrint("comint32: OpenTDIConnection() ZwCreate #2 failed, Status = %0x",  

status);  

  return STATUS\_UNSUCCESSFUL;  

 }  

 // get object handle  

 status = ObReferenceObjectByHandle( TdiEndpointHandle,  

  FILE\_ANY\_ACCESS,  

  0,  

  KernelMode,  

  (PVOID \*)&pFileObject,  

  NULL );  

 // Associate endpoint with address  

 pDeviceObject = IoGetRelatedDeviceObject( pAddressFileObject );  

 // Define a completion event  

 KeInitializeEvent( &irpCompleteEvent, NotificationEvent, FALSE );  

 // Build IO Request Packet  

 pIrp = TdiBuildInternalDeviceControlIrp( TDI\_ASSOCIATE\_ADDRESS,  

  pDeviceObject,  

  pFileObject,  

  &irpCompleteEvent,  

  &IoStatusBlock );  

 if( pIrp == NULL )  

 {  

  DbgPrint("comint32: No IRP for TDI\_ASSOCIATE\_ADDRESS");  

  return( STATUS\_INSUFFICIENT\_RESOURCES );  

 }  

  // Extend the IRP  

 TdiBuildAssociateAddress(pIrp,  

  pDeviceObject,  

  pFileObject,  

  NULL,  

  NULL,  

  TdiAddressHandle );  

 // set completion routine  

 IoSetCompletionRoutine( pIrp, TDICompletionRoutine, &irpCompleteEvent, TRUE, TRUE,  

TRUE);  

 // Send the packet  

 status = IoCallDriver( pDeviceObject, pIrp );  

 // Wait  

 if( status == STATUS\_PENDING )  

 {  

  DbgPrint("comint32: OpenTDIConnection() Waiting on IRP (associate)...");  

  KeWaitForSingleObject(&irpCompleteEvent, Executive, KernelMode, FALSE, 0);  

 }  

 if( ( status != STATUS\_SUCCESS) &&  

  ( status != STATUS\_PENDING ) )  

 {  

   DbgPrint("comint32: OpenTDIConnection() IoCallDriver #1 failed. Status = %0x",  

status);  

   return STATUS\_UNSUCCESSFUL;  

 }  

 // Connect to the remote controller  

 KeInitializeEvent(&connectionEvent, NotificationEvent, FALSE);  

 // build connection packet  

 pIrp = TdiBuildInternalDeviceControlIrp( TDI\_CONNECT,  

  pDeviceObject,  

  pFileObject,  

  &connectionEvent,  

  &IoStatusBlock );  

 if( pIrp == NULL )  

 {  

  DbgPrint("comint32: OpenTDIConnection() could not get an IRP for TDI\_CONNECT");  

  return( STATUS\_INSUFFICIENT\_RESOURCES );  

 }  

 // Initialize controller data  

 address1 = atoi(masterAddress1);  

 address2 = atoi(masterAddress2);  

 address3 = atoi(masterAddress3);  

 address4 = atoi(masterAddress4);  

 port = atoi(masterPort);  

 controllerPort = HTONS(port);  

 controllerIpAddress = INETADDR(address1,address2,address3,address4);  

 controllerTaIpAddress.TAAddressCount = 1;  

 controllerTaIpAddress.Address\[0\].AddressLength = TDI\_ADDRESS\_LENGTH\_IP;  

 controllerTaIpAddress.Address\[0\].AddressType = TDI\_ADDRESS\_TYPE\_IP;  

 controllerTaIpAddress.Address\[0\].Address\[0\].sin\_port = controllerPort;  

 controllerTaIpAddress.Address\[0\].Address\[0\].in\_addr = controllerIpAddress;  

 controllerConnection.UserDataLength = 0;  

 controllerConnection.UserData = 0;  

 controllerConnection.OptionsLength = 0;  

 controllerConnection.Options = 0;  

 controllerConnection.RemoteAddressLength = sizeof(controllerTaIpAddress);  

 controllerConnection.RemoteAddress = &controllerTaIpAddress;  

 // add controller data to the packet  

 TdiBuildConnect( pIrp,  

  pDeviceObject,  

  pFileObject,  

  NULL,  

  NULL,  

  NULL,  

  &controllerConnection,  

  0 );  

 // set completion routine  

 IoSetCompletionRoutine( pIrp, TDICompletionRoutine, &connectionEvent, TRUE, TRUE,  

TRUE);  

 // Send the packet  

 status = IoCallDriver( pDeviceObject, pIrp );  

 // wait  

 if( status == STATUS\_PENDING )  

 {  

  DbgPrint("comint32: OpenTDIConnection() waiting on IRP (connect)...");  

  KeWaitForSingleObject(&connectionEvent, Executive, KernelMode, FALSE, 0);  

 }  

 if( ( status != STATUS\_SUCCESS ) &&  

  ( status != STATUS\_PENDING ) )  

 {  

  DbgPrint("comint32: OpenTDIConnection() Connection failed. Status = %0x",  

status);  

  return( STATUS\_UNSUCCESSFUL );  

 }  

 // Start a Deferred Procedure Call  

 // Objects must be non paged  

 pKernelTimer = ExAllocatePool( NonPagedPool, sizeof( KTIMER ) );  

 pKernelDPC = ExAllocatePool( NonPagedPool, sizeof( KDPC ) );  

 timeout.QuadPart = -10;  

 KeInitializeTimer( pKernelTimer );  

 KeInitializeDpc( pKernelDPC, timerDPC, NULL );  

 if( KeSetTimerEx( pKernelTimer, timeout, 500, pKernelDPC ) ) // 1/2 second  

 {  

  DbgPrint("comint32: OpenTDIConnection() Timer was already set.");  

 }  

 return STATUS\_SUCCESS;  

}  

// Clean up  

void CloseTDIConnection()  

{  

 KeCancelTimer( pKernelTimer );  

 ExFreePool( pKernelTimer );  

 ExFreePool( pKernelDPC );  

 if( pFileInfo != NULL )  

  ExFreePool( pFileInfo );  

 if( pKernelTimer == NULL )  

  ExFreePool( pKernelTimer );  

 if( pKernelDPC == NULL )  

  ExFreePool( pKernelDPC );  

 if( pSendBuffer != NULL )  

  ExFreePool( pSendBuffer );  

 if( pSendMdl != NULL )  

  IoFreeMdl( pSendMdl );  

 if( pReceiveMdl != NULL )  

  IoFreeMdl( pReceiveMdl );  

}  

NTSTATUS SendToRemoteController( char\* buffer )  

{  

 NTSTATUS           status;  

 ULONG              bufferLength;  

 KEVENT             SendEvent;  

 PIRP               pIrp;  

 IO\_STATUS\_BLOCKIoStatusBlock;  

 KeInitializeEvent( &SendEvent, NotificationEvent, FALSE );  

 bufferLength = strlen( buffer );  

 if( pSendBuffer != NULL )  

  ExFreePool( pSendBuffer );  

 pSendBuffer = ExAllocatePool( NonPagedPool, bufferLength );  

 memcpy( pSendBuffer, buffer, bufferLength );  

 // build an IO Request Packet  

 pIrp = TdiBuildInternalDeviceControlIrp( TDI\_SEND,  

   pDeviceObject,  

   pFileObject,  

   &SendEvent,  

   &IoStatusBlock );  

 if( pIrp == NULL )  

 {  

  DbgPrint( "comint32: SendToRemoteController() could not get an IRP for TDI\_SEND"  

);  

  return( STATUS\_INSUFFICIENT\_RESOURCES );  

 }  

 if( pSendMdl != NULL )  

  IoFreeMdl( pSendMdl );  

 pSendMdl = IoAllocateMdl( pSendBuffer, bufferLength, FALSE, FALSE, pIrp );  

 if( pSendMdl == NULL )  

 {  

  DbgPrint("comint32: SendToRemoteController() could not get an MDL for TDI\_SEND");  

  return( STATUS\_INSUFFICIENT\_RESOURCES );  

 }  

 \_\_try  

 {  

  MmProbeAndLockPages(  pSendMdl,  

   KernelMode,  

   IoModifyAccess );  

 }  

 \_\_except( EXCEPTION\_EXECUTE\_HANDLER )  

 {  

  DbgPrint("comint32: SendToRemoteController() ProbeAndLock exception.");  

  return( STATUS\_UNSUCCESSFUL );  

  }  

 // Extend the packet  

 TdiBuildSend( pIrp,  

   pDeviceObject,  

   pFileObject,  

   NULL,  

   NULL,  

   pSendMdl,  

   0,  

   bufferLength );  

 // set completion routine  

 IoSetCompletionRoutine( pIrp, TDICompletionRoutine, &SendEvent, TRUE, TRUE, TRUE);  

 // Send the packet  

 status = IoCallDriver( pDeviceObject, pIrp );  

 // wait  

 if( status == STATUS\_PENDING )  

 {  

  DbgPrint("comint32: SendToRemoteController() waiting on IRP (send)...");  

  KeWaitForSingleObject( &SendEvent, Executive, KernelMode, FALSE, 0 );  

 }  

 if( ( status != STATUS\_SUCCESS ) &&  

  ( status != STATUS\_PENDING ) )  

 {  
```
