
`OBJECT_NAME_INFORMATION`的定义如下

```
typedef struct _OBJECT_NAME_INFORMATION {

    UNICODE_STRING Name;

} OBJECT_NAME_INFORMATION, *POBJECT_NAME_INFORMATION;
```

只有一个`UNICODE_STRING`的`Name`参数，保存了返回的名称。

想要实现对监控函数的删除，可以使用`CmUnRegisterCallback`函数来实现，该函数在文档中的定义如下。它只有一个参数，就是前面设置监控函数时候指定的`Cookie`的地址。

```
NTSTATUS

  CmUnRegisterCallback(

    IN LARGE_INTEGER  Cookie);
```

根据上面的内容就可以实现对注册表的监控来拒绝`AppInit_DLLs`的注入，具体实现代码如下

```
#include <ntifs.h>



VOID DriverUnload(IN PDRIVER_OBJECT driverObject);

// 未导出函数声明

PUCHAR PsGetProcessImageFileName(PEPROCESS pEProcess);	//根据EPROCESS获取进程名称

NTSTATUS ObQueryNameString(PVOID Object, POBJECT_NAME_INFORMATION ObjectNameInfo, ULONG Length, PULONG ReturnLength);	//根据对象获取名称

NTSTATUS RegistryCallback(__in PVOID  CallbackContext, __in_opt PVOID  Argument1, __in_opt PVOID  Argument2);	//回调函数

BOOLEAN GetRegisterPath(PUNICODE_STRING pRegPath, PVOID pRegObj);	//获取注册表的完整路径

//注册表回调使用的Cookie

LARGE_INTEGER g_liRegCookie;



NTSTATUS DriverEntry(IN PDRIVER_OBJECT driverObject, IN PUNICODE_STRING registryPath)

{

	NTSTATUS status = STATUS_SUCCESS;

	

	status = CmRegisterCallback(RegistryCallback, NULL, &g_liRegCookie);

	if (!NT_SUCCESS(status))

	{

		DbgPrint("回调函数设置失败 0x%X\r\n", status);

	}

	else

	{

		DbgPrint("回调函数设置成功\r\n");

	}

exit:

	driverObject->DriverUnload = DriverUnload;

	return STATUS_SUCCESS;

}



NTSTATUS RegistryCallback(__in PVOID  CallbackContext, __in_opt PVOID  Argument1, __in_opt PVOID  Argument2)

{

	NTSTATUS status = STATUS_SUCCESS;

	UNICODE_STRING uStrRegPath = { 0 };	//保存注册表完整路径

	// 保存操作码的类型

	REG_NOTIFY_CLASS uOpCode = (REG_NOTIFY_CLASS)Argument1;

	// 保存当前操作注册表的进程EPROCESS

	PEPROCESS pEProcess = NULL;

	PUCHAR pProcName = NULL;

	BOOLEAN bNeedProtected = FALSE;

	PWCHAR pValue = NULL;



	pEProcess = PsGetCurrentProcess();

	if (pEProcess != NULL)

	{

		pProcName = PsGetProcessImageFileName(pEProcess);

	}



	// 申请内存用来保存注册表路径

	uStrRegPath.Length = 0;

	uStrRegPath.MaximumLength = 1024 * sizeof(WCHAR);

	uStrRegPath.Buffer = (PWCHAR)ExAllocatePool(NonPagedPool, uStrRegPath.MaximumLength);



	if (uStrRegPath.Buffer == NULL)

	{

		DbgPrint("ExAllocatePool Error");

		goto exit;

	}



	RtlZeroMemory(uStrRegPath.Buffer, uStrRegPath.MaximumLength);



	switch (uOpCode)

	{

		// 创建注册表之前

		case RegNtPreCreateKey:

		{

			if (!GetRegisterPath(&uStrRegPath, ((PREG_CREATE_KEY_INFORMATION)Argument2)->RootObject))

			{

				DbgPrint("获取注册表路径失败\r\n");

			}

			// 显示

			// DbgPrint("[RegNtPreCreateKey][%wZ][%wZ]\n", &uStrRegPath, ((PREG_CREATE_KEY_INFORMATION)Argument2)->CompleteName);

			break;

		}

		// 打开注册表之前

		case RegNtPreOpenKey:

		{

			if (!GetRegisterPath(&uStrRegPath, ((PREG_CREATE_KEY_INFORMATION)Argument2)->RootObject))

			{

				DbgPrint("获取注册表路径失败\r\n");

			}

			// 显示

			// DbgPrint("[RegNtPreOpenKey][%wZ][%wZ]\n", &uStrRegPath, ((PREG_CREATE_KEY_INFORMATION)Argument2)->CompleteName);

			break;

		}

		// 修改键值之前

		case RegNtPreSetValueKey:

		{

			if (!GetRegisterPath(&uStrRegPath, ((PREG_SET_VALUE_KEY_INFORMATION)Argument2)->Object))

			{

				DbgPrint("获取注册表路径失败\r\n");

			}

			

			pValue = ((PREG_SET_VALUE_KEY_INFORMATION)Argument2)->ValueName->Buffer;

			//判断是否需要保护

			if (wcsstr(pValue, L"AppInit_DLLs") || wcsstr(pValue, L"LoadAppInit_DLLs")) 

			{

				DbgPrint("[RegNtPreSetValueKey][%wZ][%ws]\r\n", &uStrRegPath, pValue);

				if (pProcName) DbgPrint("进程:%s试图修改注册表,拦截成功\r\n", pProcName);

				status = STATUS_ACCESS_DENIED;	//对操作进行拦截

			}

			break;

		}

		default:

		{

			break;

		}

	}



exit:

	if (uStrRegPath.Buffer)

	{

		ExFreePool(uStrRegPath.Buffer);

		uStrRegPath.Buffer = NULL;

	}

	return status;

}



BOOLEAN GetRegisterPath(PUNICODE_STRING pRegPath, PVOID pRegObj)

{

	NTSTATUS status = STATUS_SUCCESS;

	BOOLEAN bRet = TRUE;

	ULONG uSize = 0, uRetLength = 0;

	PVOID pRegName = NULL;	



	if (!MmIsAddressValid(pRegObj) || pRegPath == NULL)

	{

		bRet = FALSE;

		goto exit;

	}



	uSize = 512;

	pRegName = ExAllocatePool(NonPagedPool, uSize);

	if (pRegName == NULL)

	{

		DbgPrint("ExAllocatePool Error\r\n");

		bRet = FALSE;

		goto exit;

	}



	//根据注册表对象获取注册表路径

	status = ObQueryNameString(pRegObj, (POBJECT_NAME_INFORMATION)pRegName, uSize, &uRetLength);

	if (!NT_SUCCESS(status))

	{

		if (pRegName) ExFreePool(pRegName);

		DbgPrint("ObQueryNameString Error 0x%X\r\n", status);

		bRet = FALSE;

		goto exit;

	}



	//将获得的路径拷贝到目标地址

	RtlCopyUnicodeString(pRegPath, (PUNICODE_STRING)pRegName);

	

exit:

	if (pRegName) ExFreePool(pRegName);

	return bRet;

}



VOID DriverUnload(IN PDRIVER_OBJECT driverObject)

{

	NTSTATUS status = STATUS_SUCCESS;



	if (g_liRegCookie.QuadPart > 0)

	{

		status = CmUnRegisterCallback(g_liRegCookie);

		if (!NT_SUCCESS(status))

		{

			DbgPrint("删除回调函数失败0x%X\r\n", status);

		}

		else

		{

			DbgPrint("删除回调函数成功\r\n");

		}

	}

	DbgPrint("驱动卸载完成\r\n");

}
```

# 三.逆向分析 `CmRegisterCallback`

接下来通过IDA分析`CmRegisterCallback`来看看是如何保存注册表的回调函数。IDA中对`CmRegisterCallback`的反汇编如下

```
PAGE:0069E3EE ; NTSTATUS __stdcall CmRegisterCallback(PEX_CALLBACK_FUNCTION Function, PVOID Context, PLARGE_INTEGER Cookie)

PAGE:0069E3EE                 public CmRegisterCallback

PAGE:0069E3EE CmRegisterCallback proc near

PAGE:0069E3EE

PAGE:0069E3EE Function        = dword ptr  8

PAGE:0069E3EE Context         = dword ptr  0Ch

PAGE:0069E3EE Cookie          = dword ptr  10h

PAGE:0069E3EE

PAGE:0069E3EE                 mov     edi, edi

PAGE:0069E3F0                 push    ebp

PAGE:0069E3F1                 mov     ebp, esp

PAGE:0069E3F3                 push    [ebp+Cookie]

PAGE:0069E3F6                 mov     eax, offset Init_Data ; 将Init_Data的地址赋给eax

PAGE:0069E3FB                 push    1

PAGE:0069E3FD                 push    [ebp+Context]

PAGE:0069E400                 push    [ebp+Function]

PAGE:0069E403                 call    CmCRegisterCallback

PAGE:0069E408                 pop     ebp

PAGE:0069E409                 retn    0Ch

PAGE:0069E409 CmRegisterCallback endp
```

这个函数做了两件事情，将`Init_Data`的赋值赋给`eax`，将需要的参数入栈以后调用`CmCRegisterCallback`函数。经过分析以后发现，这个`Init_Data`是用来赋值一些数据，不过在这里用处不大，入栈的`1`也是。

继续看`CmCRegisterCallback`的反汇编代码
