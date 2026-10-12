
# 浅谈软件安全设计（一）

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-581.htm`](https://www.pediy.com/kssd/pediy09/pediy09-581.htm)

> {\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
>
> CrackMe #01
>
> code by 黑夜彩虹 & vxin with almost pure delphi
>
> --- 转载时请保留作者信息。
>
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*}

> 此 CM 的设计模式：
>
> 1. 插入一些花指令
> 2. 写了一些代码迷惑 Cracker
> 3. 有简单的 `Anti_DEDE` 和检测调试器
>
> 话不多说，请看以下代码：

```pascal
unit main;

interface

uses
  Windows, Messages, SysUtils, Variants, Classes, Graphics, Controls, Forms,
  Dialogs, StdCtrls, ExtCtrls, strutils;

  Const
  C1= 17856;
  C2= 23589;

type
  TForm1 = class(TForm)
    Image1: TImage;
    Edit1: TEdit;
    Label1: TLabel;
    Label2: TLabel;
    Edit2: TEdit;
    Button1: TButton;
    procedure FormCreate(Sender: TObject);
    procedure Button1Click(Sender: TObject);
  private
    { Private declarations }
  public
    { Public declarations }
  end;

var
  Form1: TForm1;

implementation

{$R *.dfm}

Procedure Anti_DeDe(); //检测 DEDE 反编译器
var
  DeDeHandle:THandle;
  i:integer;
begin
  DeDeHandle:=FindWindow(nil,chr($64)+chr($65)+chr($64)+chr($65));
  if DeDeHandle<>0 then
    begin
      For i:=1 to 4500 do
        SendMessage(DeDeHandle,WM_CLOSE,0,0);
    end;
end;

Function ABC42():Boolean; //检测调试器;
var
  YInt,NInt:Integer;
begin
  asm
    mov eax,fs:[30h]
    movzx eax,byte ptr[eax+2h]
    or al,al
    jz @No
    jnz @Yes
    @No:
      mov NInt,1
    @Yes:
      Mov YInt,1
  end;
  if YInt=1 then
    Result:=True;
  if NInt=1 then
    Result:=False;
end;

function EncryptModule(SourceStr:String;Key:Word;N:Integer):String;
var
  I:Integer;
begin
  SetLength(Result,Length(SourceStr));//利用 SetLength 函数指定密文长度
  //对每一个索引元素进行变换
  for I:=1 to Length(SourceStr) do
  begin
   Result[i]:=Char(byte(SourceStr[i]) xor (Key Shr N));
   Key:= (byte(Result[i]) + Key)*C1+C2;
  end;
end;

//==========以下是 549 的函数，据说没有暴破点，顺便试一试
//========函数作用：动态改变程序运行罗辑

function GetEIP: Integer;//自动生成 address 的方法
asm
  mov eax, [esp];
  sub eax, 5;      //call GetEIP 占用 5 字节
end;

function PatchOneItem(PatchItem: String): Boolean;
var
  PatchAddress: Integer;
  PatchLength: DWord;
  PatchData: Pointer;
  PatchDataStr: String;
  i: Integer;
  PatchByte: Byte;
  PID, PHandle: THandle;
  WriteCount: DWord;
begin
  Result := False;
  if Length(PatchItem) < 11 then Exit;
  PatchAddress := StrToInt('0x' + LeftStr(PatchItem, 8));
  for i := 1 to Length(PatchItem) do begin
    if PatchItem[i] <> ' ' then
      PatchDataStr := PatchDataStr + PatchItem[i];
  end;
  PatchLength := (Length(PatchDataStr) - 9) div 2;
  GetMem(PatchData, PatchLength);
  try
    for i := 0 to PatchLength - 1 do begin
      PatchByte := StrToInt('0x'+PatchDataStr[10 + i * 2] + PatchDataStr[10 + i * 2 + 1]);
      Byte(Pointer(Integer(PatchData) + i)^) := PatchByte;
    end;
    GetWindowThreadProcessId(Application.Handle, PID);
    PHandle := OpenProcess(PROCESS_ALL_ACCESS, False, PID);
    WriteProcessMemory(PHandle, Pointer(PatchAddress), PatchData, PatchLength, WriteCount);
    CloseHandle(PHandle);
  finally
    FreeMem(PatchData, PatchLength);
  end;
  Result := PatchLength = WriteCount;
end;

procedure Patch(PatchFile: String);
var
  PatchItems: TStrings;
  PatchIndex: Integer;
begin
  if not FileExists(PatchFile) then Exit;
  PatchItems := TStringList.Create;
  try
    PatchItems.LoadFromFile(PatchFile);
    for PatchIndex := 0 to PatchItems.Count - 1 do begin
      PatchOneItem(PatchItems[PatchIndex]);
    end;
  finally
    PatchItems.Free;
  end;
end;

procedure TForm1.FormCreate(Sender: TObject);
begin
     Anti_DeDe; //检测 DEDE，检测到关闭它。
     if ABC42 then  ExitProcess(0); //检测调试器，终止。
end;

procedure TForm1.Button1Click(Sender: TObject);  //注册按纽，开始检测
begin
  //========在这里插入一些花指令
  asm
   jz @Start
   jnz @Start
   db 0E8h, 24h, 0, 0  ;
   db 0, 8Bh, 44h,  24h
   db 4, 8Bh, 0, 3Dh
   db 4, 0, 0, 80h
   db 75h,  8, 8Bh,  64h
   db 24h,  8, 0EBh, 4
   db 58h,  0EBh, 0Ch, 0E9h
   db 64h,  8Fh, 5,  0
   db 0, 0, 0, 74h
   db 0F3h, 75h, 0F1h, 0EBh
   db 24h,  64h, 0FFh, 35h
   db 0, 0, 0, 0
   db 0EBh, 12h, 0FFh, 9Ch
   db 74h,  3, 75h,  1
   db 0E9h, 81h, 0Ch, 24h
   db 0, 1, 0, 0
   db 9Dh,  90h, 0EBh, 0F4h
   db 64h,  89h, 25h, 0
   db 0, 0, 0, 0EBh
   db 0E6h
   db 0EBh, 1, 0Fh, 31h  ;
   db 0F0h, 0EBh, 0Ch, 33h
   db 0C8h, 0EBh, 3, 0EBh
   db 9, 0Fh, 59h,  74h
   db 5, 75h, 0F8h, 51h
   db 0EBh, 0F1h
   db 0B9h, 4, 0, 0  ;
   @Start:
   end;
   if length(edit2.Text)>3 then  //比较大于 3 位
   begin
   //============再写一些骗 Cracker
  if edit2.Text=EncryptModule(Edit1.Text,12345,10) then
  begin
  showmessage('请重启本软件。');
  //=======还可以再写一些记号，这里我就不写了
  end;
  PatchOneItem(edit2.Text);   //真正的比较
  showmessage('ok');          //弹出 OK 对话框
  end;
end;

end.
```

顺便附上 CM，以便大家测试~~

`4dd466d0e1e0cc123afbe504da2462ca`
