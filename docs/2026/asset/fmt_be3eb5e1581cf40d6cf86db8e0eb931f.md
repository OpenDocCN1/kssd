
```
//自己发送IRP请求生成/打开文件(IRP_MJ_CREATE)

NTSTATUS IrpCreateFile(

	OUT PFILE_OBJECT *ppFileObject,

	OUT PDEVICE_OBJECT *ppDeviceObject, //如果成功，这里保存:(*ppFileObject)->Vpb->DeviceObject

	IN ACCESS_MASK DesiredAccess,

	IN PUNICODE_STRING punsFilePath, //必须以"盘符:\"开头, 比如: "C:\..."

	OUT PIO_STATUS_BLOCK pIoStatusBlock,

	IN PLARGE_INTEGER AllocationSize OPTIONAL,

	IN ULONG FileAttributes,

	IN ULONG ShareAccess,

	IN ULONG Disposition,

	IN ULONG CreateOptions,

	IN PVOID EaBuffer OPTIONAL,

	IN ULONG EaLength

)

{

	PAUX_ACCESS_DATA pAuxData = NULL;

	HANDLE hRoot = NULL;

	PFILE_OBJECT pRootObject = NULL, pFileObject = NULL;

	PDEVICE_OBJECT pDeviceObject = NULL, pRealDeviceObject = NULL;

	PIRP pIrp;

	PIO_STACK_LOCATION pIrpSp;

	NTSTATUS ntStatus = STATUS_UNSUCCESSFUL;

	ACCESS_STATE AccessState;

	KEVENT kEvent;

	IO_SECURITY_CONTEXT IoSecurityContext;

	OBJECT_ATTRIBUTES objAttrs;

	UNICODE_STRING unsRootPath;

	IO_STATUS_BLOCK userIosb;

	WCHAR wRootPath[8];  //"\??\C:\"

	

	//路径长度检查(最短: "C:\")

	if (punsFilePath->Length < 3 * sizeof(CHAR)) {

		return STATUS_INVALID_PARAMETER;

	}



	RtlZeroMemory(pIoStatusBlock, sizeof(IO_STATUS_BLOCK));



	//根目录符号链接

	ntStatus = RtlStringCbPrintfW(

		wRootPath,

		sizeof(wRootPath),

		L"\\??\\%c:\\",

		punsFilePath->Buffer[0]);

	ASSERT(NT_SUCCESS(ntStatus));

	RtlInitUnicodeString(&unsRootPath, (PCWSTR)wRootPath);



	do {

		//打开根目录

		InitializeObjectAttributes(&objAttrs,

			&unsRootPath,

			OBJ_KERNEL_HANDLE | OBJ_CASE_INSENSITIVE,

			NULL,

			NULL);

		ntStatus = IoCreateFile(&hRoot,

			FILE_READ_ATTRIBUTES | SYNCHRONIZE,

			&objAttrs,

			&userIosb,

			NULL,

			FILE_ATTRIBUTE_NORMAL,

			FILE_SHARE_READ | FILE_SHARE_WRITE | FILE_SHARE_DELETE,

			FILE_OPEN,

			FILE_DIRECTORY_FILE | FILE_SYNCHRONOUS_IO_NONALERT,

			NULL,

			0,

			CreateFileTypeNone,

			NULL,

			IO_NO_PARAMETER_CHECKING);

		*pIoStatusBlock = userIosb;

		if (!NT_SUCCESS(ntStatus)) {

			break;

		}



		//得到根目录文件对象

		ntStatus = ObReferenceObjectByHandle(hRoot,

			FILE_READ_ATTRIBUTES,

			*IoFileObjectType,

			KernelMode,

			(PVOID*)&pRootObject,

			NULL);

		if (!NT_SUCCESS(ntStatus)) {

			break;

		}

		if (pRootObject->Vpb == NULL || pRootObject->Vpb->DeviceObject == NULL || pRootObject->Vpb->RealDevice == NULL) {

			ntStatus = STATUS_INVALID_PARAMETER;

			break;

		}



		//得到卷设备

		pDeviceObject = pRootObject->Vpb->DeviceObject;  //文件系统卷设备

		pRealDeviceObject = pRootObject->Vpb->RealDevice; //卷管理器生成的卷设备(物理设备)

                ObDereferenceObject((PVOID)pRootObject);

		pRootObject = NULL;

		ZwClose(hRoot);

		hRoot = NULL;



		//创建文件对象

		InitializeObjectAttributes(&objAttrs,

			NULL,

			OBJ_CASE_INSENSITIVE,

			NULL,

			NULL);

		ntStatus = ObCreateObject(

			KernelMode,

			*IoFileObjectType,

			&objAttrs,

			KernelMode,

			NULL,

			sizeof(FILE_OBJECT),

			0,

			0,

			(PVOID*)&pFileObject);

		if (!NT_SUCCESS(ntStatus)) {

			break;

		}



		//填写文件对象内容

		RtlZeroMemory(pFileObject,

			sizeof(FILE_OBJECT)

		);

		pFileObject->Type = IO_TYPE_FILE;

		pFileObject->Size = sizeof(FILE_OBJECT);

		pFileObject->DeviceObject = pRealDeviceObject;

		pFileObject->Flags = FO_SYNCHRONOUS_IO;

		pFileObject->FileName.Buffer = &punsFilePath->Buffer[2];

		pFileObject->FileName.Length = punsFilePath->Length - 2 * sizeof(WCHAR);

		pFileObject->FileName.MaximumLength = punsFilePath->MaximumLength - 2 * sizeof(WCHAR);

		KeInitializeEvent(&pFileObject->Lock,

			SynchronizationEvent,

			FALSE);

		KeInitializeEvent(&pFileObject->Event,

			NotificationEvent,

			FALSE);



		//分配AUX_ACCESS_DATA缓冲区

		pAuxData = (PAUX_ACCESS_DATA)ExAllocatePoolWithTag(NonPagedPool,

			sizeof(AUX_ACCESS_DATA),

			MY_TAG);

		if (pAuxData == NULL) {

			ntStatus = STATUS_INSUFFICIENT_RESOURCES;

			break;

		}



		//设置安全状态

		ntStatus = SeCreateAccessState(&AccessState,

			pAuxData,

			DesiredAccess,

			IoGetFileObjectGenericMapping());

		if (!NT_SUCCESS(ntStatus)) {

			break;

		}



		//分配IRP

		pIrp = IoAllocateIrp(pDeviceObject->StackSize,

			FALSE);

		if (pIrp == NULL) {

			ntStatus = STATUS_INSUFFICIENT_RESOURCES;

			break;

		}



		//填写IRP

		pIrp->MdlAddress = NULL;

		pIrp->UserBuffer = NULL;

		pIrp->AssociatedIrp.SystemBuffer = EaBuffer;

		pIrp->Flags = IRP_CREATE_OPERATION | IRP_SYNCHRONOUS_API;

		pIrp->RequestorMode = KernelMode;

		pIrp->PendingReturned = FALSE;

		pIrp->Cancel = FALSE;

		pIrp->CancelRoutine = NULL;

		KeInitializeEvent(&kEvent,

			NotificationEvent,

			FALSE);

		pIrp->UserEvent = &kEvent;

		pIrp->UserIosb = &userIosb;

		pIrp->Overlay.AllocationSize.QuadPart = AllocationSize != NULL ? AllocationSize->QuadPart : 0;

		

		pIrp->Tail.Overlay.Thread = PsGetCurrentThread();

		pIrp->Tail.Overlay.OriginalFileObject = pFileObject;

		pIrp->Tail.Overlay.AuxiliaryBuffer = NULL;



		//得到下层栈空间

		pIrpSp = IoGetNextIrpStackLocation(pIrp);

		ASSERT(pIrpSp != NULL);



		//填写下层栈空间

		pIrpSp->MajorFunction = IRP_MJ_CREATE;

		IoSecurityContext.SecurityQos = NULL;

		IoSecurityContext.AccessState = &AccessState;

		IoSecurityContext.DesiredAccess = DesiredAccess;

		IoSecurityContext.FullCreateOptions = 0;

		pIrpSp->Parameters.Create.SecurityContext = &IoSecurityContext;

		pIrpSp->Parameters.Create.Options = (Disposition << 24) | CreateOptions;

		pIrpSp->Parameters.Create.FileAttributes = (USHORT)FileAttributes;

		pIrpSp->Parameters.Create.ShareAccess = (USHORT)ShareAccess;

		pIrpSp->Parameters.Create.EaLength = EaLength;

		pIrpSp->FileObject = pFileObject;

		pIrpSp->DeviceObject = pDeviceObject;



		//设置完成例程: IoSetCompletionRoutine(pIrp,FileOperationCompletion,NULL,TRUE,TRUE,TRUE);

		pIrpSp->CompletionRoutine = FileOperationCompletion;

		pIrpSp->Context = NULL;

		pIrpSp->Control = SL_INVOKE_ON_SUCCESS | SL_INVOKE_ON_ERROR | SL_INVOKE_ON_CANCEL;



		//下发IRP请求

		ntStatus = IoCallDriver(pDeviceObject,

			pIrp);



		//等候完成

		(VOID)KeWaitForSingleObject(&kEvent,

			Executive,

			KernelMode,

			FALSE,

			NULL);

		

                //得到完成状态

		*pIoStatusBlock = userIosb;

		ntStatus = pIoStatusBlock->Status;

    } while (FALSE);



	if (NT_SUCCESS(ntStatus)) {

		//成功后的处理

		ASSERT(pFileObject != NULL && pFileObject->DeviceObject != NULL);

		InterlockedIncrement(&pFileObject->DeviceObject->ReferenceCount);

		if (pFileObject->Vpb != NULL) {

			InterlockedIncrement((PLONG)&pFileObject->Vpb->ReferenceCount);

		}

		*ppFileObject = pFileObject;

		*ppDeviceObject = pDeviceObject;

	}else {

		//失败后的处理

		if (pFileObject != NULL) {

			pFileObject->DeviceObject = NULL;

			ObDereferenceObject(pFileObject);

		}

		pIoStatusBlock->Status = ntStatus;

	}

	

	//检查并释放相关资源

	if (pAuxData != NULL) {

		ExFreePoolWithTag((pAuxData),

			MY_TAG);

	}

    if (pRootObject != NULL) {

		ObDereferenceObject((PVOID)pRootObject);

	}

	if (hRoot != NULL) {

		ZwClose(hRoot);

	}



	return ntStatus;

}
```

3. 文件关闭的实现：分两步：1) 发送主功能号为`IRP_MJ_CLEANUP`的IRP向FSD表明：文件对象句柄数已为0 => 发送主功能号为`IRP_MJ_CLOSE`的IRP向FSD表明：文件对象引用计数已为0，由FSD负责销毁文件对象。

(1) 文件关闭1：`IRP_MJ_CLEANUP`
