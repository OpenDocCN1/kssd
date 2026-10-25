
# 小试 anti vmware

> 原文：[`www.pediy.com/kssd/pediy11/110046.html`](https://www.pediy.com/kssd/pediy11/110046.html)

小试 anti vmware

今天偶然看到一款绿色版的硬盘专业工具，突然发现可以利用其中的一项功能来实现 anti vmware。

今日事今日毕，那就在今晚 12:00 之前把这个想法实现吧，let's go!

我的想法就是检测硬盘的 `modelnumber`，具体什么是 `modelnumber` 自己网上搜吧，反正不是硬盘序列号。难点就是在多种操作系统下都要能起到 anti vmware 的效果。程序在 `xp`、`2k`、`2003` 下都可以检测到 vmware 的运行。

直接贴代码了，如果看不懂也没关系，我也是逆了人家的代码写出来的。`Delphi` 也可以当汇编语言开发工具用，难道不是吗？

```delphi
unit Unit1;

interface

uses
  Windows, Messages, SysUtils, Variants, Classes, Graphics, Controls, Forms,
  Dialogs, StdCtrls, Buttons;

type
  TForm1 = class(TForm)
    BitBtn1: TBitBtn;
    procedure BitBtn1Click(Sender: TObject);
    procedure FormClose(Sender: TObject; var Action: TCloseAction);
  private
    { Private declarations }
  public
    { Public declarations }
  end;

var
  Form1: TForm1;
  hDeviceHandle: Thandle;

implementation

{$R *.dfm}

procedure TForm1.BitBtn1Click(Sender: TObject);
var
  InBuffer: array[0..$8f] of byte;
  cb: Cardinal;
  tmp: Pchar;
begin
  hDeviceHandle := CreateFile('\\.\PHYSICALDRIVE0', $C0000000, $3, nil, OPEN_EXISTING, $8000000, 0);
  ZeroMemory(@InBuffer, sizeof(InBuffer));
  asm
    pushad
    lea ebx, InBuffer
    xor ecx, ecx
    mov al, $2c
    MOV [ebx], al
    MOV EAX, $200c0000
    MOV [ebx+4], eax
    mov al, $01
    MOV [ebx+8], al
    mov al, $40
    MOV [ebx+$c], al
    MOV EAX, $0001a5E0
    MOV [ebx+$10], eax
    mov al, $30
    MOV [ebx+$18], al
    mov al, $12
    MOV [ebx+$1c], al
    mov al, $40
    MOV [ebx+$20], al
    add ecx, ebx
    add ecx, $50
    MOV [ebx+$14], ecx
    popad
  end;
  if DeviceIoControl(hDeviceHandle, $4D014, @InBuffer, $50, @InBuffer, $50, cb, nil) then
    begin
      asm
        pushad
        lea ebx, InBuffer
        add ebx, $58
        mov tmp, ebx
        popad
      end; //asm
      if ((pos('vmware', LowerCase(tmp)) > 0) or (pos('virtual', LowerCase(tmp)) > 0)) then
        showmessage('检测到 VMware Workstation!!!')
      else
        showmessage('请在 VMware 中测试!');
    end;
end;

procedure TForm1.FormClose(Sender: TObject; var Action: TCloseAction);
begin
  closehandle(hDeviceHandle);
end;
end.
```

代码很短，但是效果不错。截图几张，留作纪念!

![](img/17338397404e3cf011dc77312df4be9a.jpg)
![](img/2821ef6bd574eaa988de611511bf9cd8.jpg)
![](img/f065796f6ce305b519f78c0f80b7c2fb.jpg)

天易 love  
2010-04-01

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) | `tianyi.rar` |
| :--- | :--- |
