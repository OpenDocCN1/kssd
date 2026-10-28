
```
unit GetData;



interface



uses

  Windows, WinSvc, Classes, common, ComCtrls, SysUtils, Dialogs;



type

  TGetData = class(TThread)

  private

    { Private declarations }

    ProcessData: PROCESS_DATA;

    procedure FillProcessInfo;

  protected

    constructor Create(CreateSuspended: Boolean);

    procedure Execute; override;

  end;



implementation



{ 

  Important: Methods and properties of objects in visual components can only be

  used in a method called using Synchronize, for example,



      Synchronize(UpdateCaption);  



  and UpdateCaption could look like,



    procedure TGetData.UpdateCaption;

    begin

      Form1.Caption := 'Updated in a thread';

    end; 

    

    or 

    

    Synchronize( 

      procedure 

      begin

        Form1.Caption := 'Updated in thread via an anonymous method' 

      end

      )

    );

    

  where an anonymous method is passed.

  

  Similarly, the developer can call the Queue method with similar parameters as 

  above, instead passing another TThread class as the first parameter, putting

  the calling thread in a queue with the other thread.

    

}



{ TGetData }

uses

  main;



constructor TGetData.Create(CreateSuspended: Boolean);

begin

  inherited Create(CreateSuspended);

  Priority := tpHighest;

end;



procedure TGetData.FillProcessInfo;

var

  buffer: array[0..1023] of AnsiChar;

  rtnVal: DWORD;

  pTmp: PAnsiChar;

  tlItems: TListItem;

  iItemCnt: Integer;

begin

  { The path can be it the short form. Convert it to long. }

	{ If no long path is found or path is in long form, GetLongPathName }

	{ simply returns the specified path. }

  FillChar(buffer, SizeOf(buffer), 0);

  rtnVal := GetLongPathName(@processData.szProcessName, @buffer, SizeOf(buffer));

  if (rtnVal = 0) or (rtnVal >= SizeOf(buffer)) then

  begin

    { 1024 bytes was not enough. Just display whatever we've got from the driver. }

		{ I want to keep the things simple. But you'd better to allocate more memory }

		{ and call GetLongPathName again and again until the buffer size will }

		{ satisfy the need. }

    pTmp := @processData.szProcessName;

  end else

    pTmp := @buffer;

  tlItems := Form1.lvProcessInfo.Items.Add;

  tlItems.Caption := string(pTmp);

  tlItems.SubItems.Add(Format('%8.8X', [processData.dwProcessId]));

  if ProcessData.bCreate <> 0 then

    tlItems.SubItems.Add('Created')

  else

    tlItems.SubItems.Add('Destroyed');

  iItemCnt := Form1.lvProcessInfo.Items.Count;

  Form1.lvProcessInfo.Items[iItemCnt - 1].MakeVisible(True);

end;



procedure TGetData.Execute;

var

  hThread: THandle;

  dwBytesReturned: DWORD;

begin

  { Place thread code here }

while True do

  begin

		if WaitForSingleObject(g_hEvent, INFINITE) <> WAIT_FAILED then

    begin

			if g_fbExitNow then

        Break;



			if DeviceIoControl(g_hDevice, IOCTL_GET_PROCESS_DATA, nil, 0,

						             @ProcessData, SizeOf(ProcessData),

                         dwBytesReturned, nil) then

      begin

				Synchronize(FillProcessInfo);

			end;

		end else

    begin

			ShowMessage('Wait for event failed. Thread now exits. Restart application.');

			Break;

		end;

    Sleep(1);

	end;

end;



end.
```

线程对象的作用就是等待驱动程序的通知，并在收到通知后从驱动程序获取信息并显示出来。

```
constructor TGetData.Create(CreateSuspended: Boolean);

begin

  inherited Create(CreateSuspended);

  Priority := tpHighest;

end;
```

一旦驱动程序里有最新的信息，我们就必须立即获取，否则信息就有可能被覆盖掉，因此，我们在构造函数中把线程的优先级设置成最高。当然在我们这个例子里是没必要这样做的，因为我们跟踪的进程的创建和销毁，这样的事件不会每时每刻都发生。

```
while True do

  begin

	if WaitForSingleObject(g_hEvent, INFINITE) <> WAIT_FAILED then

    begin

	  if g_fbExitNow then

        Break;



	  if DeviceIoControl(g_hDevice, IOCTL_GET_PROCESS_DATA, nil, 0,

					  @ProcessData, SizeOf(ProcessData),

                      dwBytesReturned, nil) then

      begin

		Synchronize(FillProcessInfo);

	  end;
```

接下来进入循环，等待驱动程序的通知，如果有通知到达，就调用`DeviceIoControl`向驱动发送`IOCTL_GET_PROCESS_DATA`控制码获取数据，数据返回到`ProcessData`中，然后调用`FillProcessInfo`将信息显示到主窗体的`ListView`控件中。这里我们使用的是VCL控件，由于可视的VCL控件不支持多线程，所以我们用`Synchronize`函数，这里不讨论`TThread`的用法，您可以参考Delphi的相关文档。

## 12.5 FillProcessInfo函数

有些时候，驱动程序返回的是短路径名（比如：`C: \ PROGRA ~ 1 \ WinZip \ WinZip32.EXE`），我也不明白为什么会这样。解决这个问题的方法也很简单，我们只需要调用windows提供的`GetLongPathName`函数。`GetLongPathName`的原型如下：

```
function GetLongPathName(lpszShortPath: PWideChar; lpszLongPath: PWideChar;

                       cchBuffer: DWORD): DWORD; stdcall;

◎	lpszShortPath是欲转换的短路径名字符串缓冲区

◎	lpszLongPath是转换成长路径名的接收缓冲区

◎	cchBuffer则是接收缓冲区的长度
```

函数成功返回转换成长路径名后的字符串的实际长度，如果接收缓冲区的长度太小不足以容纳转换后的长路径名字符串，那么函数会返回需要的缓冲区长度，这个长度包含空结束符，这个时候你就必须调整接收缓冲区的长度并重新调用`GetLongPathName`。这里我偷懒就把接收缓冲区设置成1024个字节，大多数时候是够用的。

```
Var

  buffer: array[0..1023] of AnsiChar;

  … …

FillChar(buffer, SizeOf(buffer), 0);

  rtnVal := GetLongPathName(@processData.szProcessName, @buffer, SizeOf(buffer));

  if (rtnVal = 0) or (rtnVal >= SizeOf(buffer)) then

  begin

    { 1024 bytes was not enough. Just display whatever we've got from the driver. }

		{ I want to keep the things simple. But you'd better to allocate more memory }

		{ and call GetLongPathName again and again until the buffer size will }

		{ satisfy the need. }

    pTmp := @processData.szProcessName;

  end else

pTmp := @buffer;
```

## 12.6 ProcessMon驱动程序源码
