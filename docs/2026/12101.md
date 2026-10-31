
# [原创][成果3.6]驱动和应用层的异步通信

这里来简单的讲解下驱动和应用层的异步通信，上次我写了驱动和应用层的三种基本通信方法，但是那三种方法都是通过同步的方法来实现的，就是说，在应用层向驱动层发送消息后，就堵死在那里等待驱动层的返回了，而异步的概念就是，应用层向驱动发送消息后，就马上返回了，而在驱动层的消息触发后，再将该消息反馈给应用层。

给个网上的例子：

同步就是你叫我去吃饭，我听到了就和你去吃饭；如果没有听到，你就不停的叫，直到我告诉你听到了，才一起去吃饭。

异步就是你叫我，然后自己去吃饭，我得到消息后可能立即走，也可能等到下班才去吃饭。

其实在明白了三种通信方式后，很容易使用异步方式来通信。

在调用`DeviceIoControl`时,不指定`FILE_FLAG_OVERLAPPED`标志,表示以同步方式调用,调用线程将被阻塞直到控制操作完成.

当指定`FILE_FLAG_OVERLAPPED`标志调用`DeviceIoControl`,表示以异步方式调用,调用线程不立即阻塞,直到调用线程需要结果时,调用者调用事件等待函数,等待驱动发出事件.

在异步调用时,得使用事件`event`来通信.

下面看应用层的代码：

```
// 初始化时创建设备
procedure TfrmMain.FormCreate(Sender: TObject);
begin
  //创建设备
  try
	  m_hCommDevice := CreateFile('\\.\Overlapp',	GENERIC_READ or GENERIC_WRITE,	FILE_SHARE_READ, nil,
		                	OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL or FILE_FLAG_OVERLAPPED, 0);
    //执行异步I/O的设备必须用FILE_FLAG_OVERLAPPED标志打开
    hEvent := CreateEvent( nil, False, False, nil);     //创建自动重置的事件
  except
    MessageBox(Handle, '创建设备失败', '驱动应用层通信', MB_OK + MB_ICONWARNING);
  end;
end;

//异步通信按钮，创建线程和驱动通信
procedure TfrmMain.btnBufferdClick(Sender: TObject);
var
  hThread:THandle;
  dwThreadID:DWORD;
begin
  hThread  := CreateThread(nil, 0, @WaitForNotify, self, 0, dwThreadID);
	CloseHandle(hThread);
end;

//线程的处理代码
//初始化一个OVERLAPPED 结构，然后用DeviceIoControl来通信
//在WaitForSingleObject等待返回
procedure WaitForNotify;
var
  dwReturn: DWORD;
  inData:array[0..1023] of char;
  outData:array[0..1023] of char;
  ol:OVERLAPPED ;
begin
  StrPCopy(inData, Trim(frmMain.edtBufferd_in.Text));
  //OVERLAPPED结构的Offset，OffsetHigh和hEvent成员必须被初始化。在这里初始化为0
  FillMemory(@ol, sizeof(OVERLAPPED), 0);
  ol.hEvent := frmMain.hEvent;
  if frmMain.m_hCommDevice <> 0 then
    DeviceIoControl(frmMain.m_hCommDevice, IOCTL_OVERLAP_BUFFERED_IO, @inData,	Length(inData), @outData, Length(outData), dwReturn, @ol);
  //调用WaitForSingleObject并传递设备内核对象的句柄。WaitForSingleObject会挂起调用线程直至内核对象变成有信号态。
  //设备驱动在它完成I/O后使内核对象有信号。在WaitForSingleObject返回后，I/O被完成 .
  while WaitForSingleObject(frmMain.hEvent,  100) = WAIT_TIMEOUT do
  begin
  end;
  GetOverlappedResult(frmMain.m_hCommDevice, ol, dwReturn, TRUE);
  frmMain.edtBufferd_out.Text := outData;
end;

//这里触发驱动将数据传输回来，异步消息得以完成
procedure TfrmMain.btnNotifyClick(Sender: TObject);
var
  dwReturn:DWORD;
begin
  if m_hCommDevice <> 0 then
    DeviceIoControl(m_hCommDevice, IOCL_OVERLAP_NOTIFY, nil, 0, nil, 0, dwReturn, nil);
end;

//退出时，关闭句柄，这时候如果irp还未处理，即btnNotifyClick这个函数没有触发
//则驱动中会触发取消irp的请求。
procedure TfrmMain.FormClose(Sender: TObject; var Action: TCloseAction);
begin
  if(m_hCommDevice <> 0) then
		CloseHandle(m_hCommDevice);
  if (hEvent <> 0) then
    CloseHandle(hEvent);
end;
```

驱动层代码:

```
//接受到消息的处理函数
NTSTATUS Overlap_IoControl(IN PDEVICE_OBJECT DeviceObject, IN PIRP Irp)
{
    NTSTATUS status = STATUS_NOT_SUPPORTED;
    PIO_STACK_LOCATION irpStack = NULL;
    UINT sizeofWrite = 0;
    DbgPrint("Overlap_IoControl\r\n");
    irpStack = IoGetCurrentIrpStackLocation(Irp);    
    if(irpStack)
    {
        switch(irpStack->Parameters.DeviceIoControl.IoControlCode)
        {
            case IOCTL_OVERLAP_BUFFERED_IO:   //异步通信消息
				 status = Irp->IoStatus.Status = STATUS_PENDING; 
				 Irp->IoStatus.Information = 0;
				 IoMarkIrpPending(Irp); 
				 gUserMessageIrp = Irp;//保存请求的irp
				 IoSetCancelRoutine(Irp, DriverCancelIrp); //设置irp取消例程，在应用程序退出时，触发
				 return status; 
			case IOCL_OVERLAP_NOTIFY:                                //获取数据事件 
				 COMM_BufferedIo(gUserMessageIrp, irpStack);         //处理原来的irp，将传进来的数据传输出去
                 return status;
			default:
				return status;
        }
    }
	return status;
}

//当通知要获取数据时，获得异步的irp，然后传输数据
NTSTATUS COMM_BufferedIo(PIRP Irp, PIO_STACK_LOCATION pIoStackIrp)
{
    NTSTATUS status = STATUS_UNSUCCESSFUL;
    PVOID pInputBuffer, pOutputBuffer;
	ULONG  outputLength, inputLength;
    DbgPrint("COMM_BufferedIo\r\n");
	outputLength = pIoStackIrp->Parameters.DeviceIoControl.OutputBufferLength;
    inputLength  = pIoStackIrp->Parameters.DeviceIoControl.InputBufferLength;
    pOutputBuffer = Irp->AssociatedIrp.SystemBuffer;   //输出缓冲区
    pInputBuffer = Irp->AssociatedIrp.SystemBuffer;    //输入缓冲区
    if(pInputBuffer && pOutputBuffer)
    {              
		DbgPrint("COMM_BufferedIo UserModeMessage = '%s'", pInputBuffer);
        RtlCopyMemory(pOutputBuffer, pInputBuffer, outputLength);
		Irp->IoStatus.Status = STATUS_SUCCESS; 
		Irp->IoStatus.Information = sizeof(pOutputBuffer); 
		IoCompleteRequest(Irp,IO_NO_INCREMENT); //设置该irp已经完成
        status = STATUS_SUCCESS;
    }
    return status;
}

//取消irp的例程
VOID DriverCancelIrp(IN PDEVICE_OBJECT DeviceObject, IN PIRP Irp ) 
{ 
	//UNREFERENCED_PARAMETER这个宏用于去掉一个函数的参数未用或函数中定义了一个局部变量却从未用过的编译警告
    UNREFERENCED_PARAMETER(DeviceObject);
    KdPrint(("UserMessageCancelIrp....\n"));
    if ( Irp == gUserMessageIrp)
        gUserMessageIrp = NULL; 
    Irp->IoStatus.Status = STATUS_CANCELLED; 
    Irp->IoStatus.Information = 0; 
    IoCompleteRequest(Irp,IO_NO_INCREMENT); 
} 
```

流程如下：

1、
应用层：
通知异步消息
等待返回
驱动层：
收到消息
设置`irp`取消例程
保存该`irp`

2、
应用层：
通知获取数据，即完成异步消息
驱动层：
拷贝数据到输出缓冲区，完成该`irp`

在这里我是通过手动的方式来触发异步消息的完成，而不是由系统的消息触发的。主要是为了演示的方便。由于比较简单，所以也没有使用自旋锁之类的东西了。下面是代码了：

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [驱动和应用层异步通信.rar](javascript:void(0);)
  （210.11kb，551次下载）
