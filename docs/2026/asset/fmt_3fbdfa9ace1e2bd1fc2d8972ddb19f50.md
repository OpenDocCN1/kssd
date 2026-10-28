
```cpp
// MonitorKeyboard.cpp : 定义应用程序的入口点。

//

#include <Windows.h>

#include <cstdio>

#include "resource.h"



VOID ShowError(PCHAR msg);

BOOL InitDevice(HWND hwnd);	// 初始化原始输入设备

BOOL GetData(LPARAM lParam);		// 获取原始输入设备数据



BOOL CALLBACK DialogProc(HWND hwndDlg,		

						 UINT uMsg,     		

						 WPARAM wParam,		

						 LPARAM lParam)

{

	BOOL bRet = FALSE;



	switch (uMsg)

	{

		case WM_INPUT:

		{

			GetData(lParam);

			bRet = TRUE;

			break;

		}	

		case WM_CLOSE:

		{

			EndDialog(hwndDlg, 0);

			bRet = TRUE;

			break;

		}

		case WM_INITDIALOG:

		{

			if (!InitDevice(hwndDlg)) EndDialog(hwndDlg, 0);

			bRet = TRUE;

			break;

		}

	}



	return bRet;

}





int APIENTRY wWinMain(_In_ HINSTANCE hInstance,

                     _In_opt_ HINSTANCE hPrevInstance,

                     _In_ LPWSTR    lpCmdLine,

                     _In_ int       nCmdShow)

{

	DialogBox(hInstance, MAKEINTRESOURCE(IDD_DIALOG_MAIN), NULL, DialogProc);



    return 0;

}



BOOL GetData(LPARAM lParam)

{

	BOOL bRet = TRUE;

	RAWINPUT rawInputData = { 0 };

	UINT uSize = sizeof(rawInputData);

	CHAR szVKey[20] = { 0 };



	// 获取输入数据

	GetRawInputData((HRAWINPUT)lParam, RID_INPUT, &rawInputData, &uSize, sizeof(RAWINPUTHEADER));

	if (rawInputData.header.dwType == RIM_TYPEKEYBOARD)

	{

		if ((rawInputData.data.keyboard.Message == WM_KEYDOWN) ||

			(rawInputData.data.keyboard.Message == WM_SYSKEYDOWN))

		{

			sprintf(szVKey, "%d", rawInputData.data.keyboard.VKey);

			MessageBox(NULL, szVKey, TEXT("KeyDown"), MB_OK);

		}

	}



	return bRet;

}



BOOL InitDevice(HWND hwnd)

{

	BOOL bRet = TRUE;

	RAWINPUTDEVICE rawInputDevice = { 0 };



	// 初始化RAWINPUTDEVICE使其接收键盘消息

	rawInputDevice.hwndTarget = hwnd;

	rawInputDevice.usUsagePage = 0x01;

	rawInputDevice.usUsage = 0x06;

	rawInputDevice.dwFlags = RIDEV_INPUTSINK;



	// 注册原始输入设备

	if (!RegisterRawInputDevices(&rawInputDevice, 1, sizeof(rawInputDevice)))

	{

		ShowError("RegisterRawInputDevices");

		bRet = FALSE;

		goto exit;

	}



exit:

	return bRet;

}



VOID ShowError(PCHAR msg)

{

	CHAR szError[MAXBYTE] = { 0 };



	sprintf(szError, "%s Error %d", msg, GetLastError());

	MessageBox(NULL, szError, TEXT("Error"), MB_OK);

}
```

## 2.运行结果

最终可以看到按键操作被监控到了，并以虚拟键码的形式弹出来了

![](https://bbs.kanxue.com/upload/attach/202111/835440_B9TB8VNZARSNS82.jpg)

# 四.过滤驱动

## 1.实现原理

相对于用户层的键盘监控，驱动层的监控更加底层，它可以绕过绝大多数的保护。

按键操作IRP在驱动层会经过分层驱动设备栈，对于同一设备栈中，IRP会从栈顶一直传递到栈底，且新添加的设备总是附加在设备栈的顶部，也就是说新添加的设备可以更先获取IRP。

过滤驱动的实现便是基于此：

> - 分层驱动中在加一层而不影响它的上下层，以过滤它们之间的数据，对数据或行为进行安全的控制。

> - 过滤是通过设备绑定实现的。

键盘过滤驱动是工作在异步模式下的，为了得到一个按键操作，它首先需要发送`IRP_MJ_READ`到驱动的设备栈，驱动会立刻完成这个IRP，并将刚按下的键的相关数据作为该IRP的返回值返回。

> 键盘过滤驱动的按键记录就是基于这个原理。驱动程序创建一个键盘设备，将它附加在键盘类`KbdClass`设备栈上，此时该设备就是设备栈的栈顶。当有键盘按下IRP消息的时候，该设备最先接收到IRP消息，此时就可以设置完成回调函数，向下传递按键信息，获取按键信息。该键盘驱动设备就是一个过滤驱动设备，附加在键盘类驱动设备栈之上。

具体的创建过滤驱动设备的实现流程如下：

1. 调用`IoCreateDevice`创建`FILE_DEVICE_KEYBOARD`键盘设备，并调用`IoCreateSymbolicLink`函数为该设备创建一个符号链接

2. 调用`IoGetDeviceObjectPointer`函数获取`KeyboardClass0`驱动设备对象

3. 调用`IoAttachDeviceToDeviceStack`函数将创建的键盘设备附加到`KeyboardClass0`设备对象所在的设备栈顶之上

`IoGetDeviceObjectPointer`函数的定义如下：

```cpp
NTSTATUS 

  IoGetDeviceObjectPointer(

    IN PUNICODE_STRING  ObjectName,

    IN ACCESS_MASK  DesiredAccess,

    OUT PFILE_OBJECT  *FileObject,

    OUT PDEVICE_OBJECT  *DeviceObject

    );
```

| 参数 | 含义 |
| --- | --- |
| `ObjectName` | 指向包含作为设备对象名称的Unicode字符串的缓冲区的指针 |
| `DesiredAccess` | 指定表示所需访问的访问掩码值。通常是`FILE_READ_DATA`，极少会由`FILE_WRITE_DATA`或`FILE_ALL_ACCESS` |
| `FileObject` | 指向文件对象的指针，如果调用成功，该文件对象表示与用户模式代码对应的设备对象 |
| `DeviceObject` | 如果调用成功，则指向表示命名逻辑、虚拟或物理设备的设备对象的指针 |

`IoAttachDeviceToDeviceStack`函数的定义如下：

```cpp
PDEVICE_OBJECT 

  IoAttachDeviceToDeviceStack(

    IN PDEVICE_OBJECT  SourceDevice,

    IN PDEVICE_OBJECT  TargetDevice);
```

| 参数 | 含义 |
| --- | --- |
| `SourceDevice` | 指向调用方创建的设备对象的指针 |
| `TargetDevice` | 指向另一个驱动程序的设备对象的指针，如前一次调用`IoGetDeviceObjectPointer`返回的指针 |

> 该函数的返回值是原来设备栈上的栈顶设备，也就是当前设备栈上附加设备的下一个设备

> 想要截获键盘输入，重点是截获操作系统发给键盘的IRP读请求。当键盘收到IRP读请求时，等待用户输入，若用户输入，则把用户输入的数据填充到IRP读请求中，在把这个IRP读请求发送给操作系统。自定义的键盘过滤驱动设备最先捕获到这个IRP读请求，但是里面并没有按键数据。解决方法是设置一个回调函数，然后，将IRP读请求继续往下传递。等到键盘的IRP读请求处理完毕之后，再去执行先前设置的回调函数，从中获取键盘信息。

在IRP读请求中具体的处理流程如下：

1. 调用`IoCopyCurrentIrpStackLocationToNext`函数将当前设备中的IRP赋值到下一个设备中

2. 调用`IoSetCompletionRoutine`函数设置完成回调函数，将在下一层驱动完成由IRP指定的操作请求时调用这个函数

3. 调用`IoCallDriver`函数，将IRP发送到下一个设备

`IoCopyCurrentIrpStackLocationToNext`函数定义如下：

```cpp
VOID 

  IoCopyCurrentIrpStackLocationToNext(

    IN PIRP  Irp

    );
```

`IoSetCompletionRoutine`函数定义如下：

```cpp
VOID 

  IoSetCompletionRoutine(

    IN PIRP  Irp,

    IN PIO_COMPLETION_ROUTINE  CompletionRoutine  OPTIONAL,

    IN PVOID  Context  OPTIONAL,

    IN BOOLEAN  InvokeOnSuccess,

    IN BOOLEAN  InvokeOnError,

    IN BOOLEAN  InvokeOnCancel

    );
```

| 参数 | 含义 |
| --- | --- |
| `Irp` | 指向驱动程序正在处理的IRP的指针 |
| `CompletionRoutine` | 指定驱动程序提供的`IoCompletion`例程的入口点，该例程在下一个较低的驱动程序完成数据包时调用 |
| `Context` | 指向驱动程序确定的要传递给`IoCompletion`例程的上下文的指针。上下文信息必须存储在非分页内存中，因为`IoCompletion`例程是在IRQL<=`DISPATCH_LEVEL`级别调用的 |
| `InvokeOnSuccess` | 指定在IRP的`IO_STATUS_BLOCK`结构体中使用成功状态值完成IRP时，是否根据`NT_SUCCESS`宏的结果调用完成例程（请参见使用`NTSTATUS`值） |
| `InvokeOnSuccess` | 指定如果IRP在IRP的`IO_STATUS_BLOCK`结构体中使用非成功状态值完成，是否调用完成例程 |
| `InvokeOnSuccess` | 指定如果驱动程序或内核调用`IoCancelIrp`以取消IRP，是否调用完成例程 |

其中的完成回调函数`CompletionRoutine`定义如下：

```cpp
IO_COMPLETION_ROUTINE IoCompletion;



NTSTATUS

  IoCompletion(

    __in PDEVICE_OBJECT  DeviceObject,

    __in PIRP  Irp,

    __in PVOID  Context

    )

  {...}
```

| 参数 | 含义 |
| --- | --- |
| `DeviceObject` | 调用者提供的指向`DEVICE_OBJECT`结构体的指针。这是目标设备的设备对象，以前由驱动程序的`AddDevice`例程创建 |
| `Irp` | 调用方提供的指向描述I/O操作的IRP结构的指针 |
| `Context` | 调用方提供的指向特定于驱动程序的上下文信息的指针，以前在调用`IoSetCompletionRoutineEx`或`IoSetCompletionRoutineEx`时提供。上下文信息必须存储在非分页内存中，因为可以在`DISPATCH_LEVEL`级别调用`IoCompletion`例程 |

设置完回调函数以后，驱动就可以在回调函数的IRP中的`AssociatedIrp.SystemBuffer`获取包含按键数据信息`PKEYBOARD_INPUT_DATA`，该结构体的定义如下：

```cpp
typedef struct _KEYBOARD_INPUT_DATA {



    //

    // Unit number.  E.g., for \Device\KeyboardPort0 the unit is '0',

    // for \Device\KeyboardPort1 the unit is '1', and so on.

    //



    USHORT UnitId;



    //

    // The "make" scan code (key depression).

    //



    USHORT MakeCode;



    //

    // The flags field indicates a "break" (key release) and other

    // miscellaneous scan code information defined below.

    //



    USHORT Flags;



    USHORT Reserved;



    //

    // Device-specific additional information for the event.

    //



    ULONG ExtraInformation;



} KEYBOARD_INPUT_DATA, *PKEYBOARD_INPUT_DATA;
```

其中`Flags`若为0，则表示按键按下，若为1则表示按键弹起，而`MakeCode`中保存的则是按键的扫描码。

这里还需要解决的问题是，由于要向下一个设备发送IRP以及任何时候设备栈底都会有一个键盘的`IRP_MJ_READ`请求处于挂起未确定状态，这意味着只有该IRP完成返回，并且新的IRP请求还未送到时，设备才会有一个很短暂的时间处于非挂起状态。如果按照一般的方式动态卸载键盘过滤驱动，那么基本都有`IRP_MJ_READ`请求处于挂起未确定状态。这时，驱动程序若已经执行了卸载驱动的操作，那么之后的按键要处理这个IRP的时候会因为找不到驱动而蓝屏。要解决这两个问题，只需要在创建设备的时候增加`DeviceExtension`，在这个`DeviceExtension`将信息记录下来以供程序正常运行。

具体代码如下：
