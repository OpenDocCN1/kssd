
# 一个小工具源码（Delphi 中文字符编码转换器）

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-691.htm`](https://www.pediy.com/kssd/pediy08/pediy8-691.htm)

{*  我相信用 `DEDE` 反汇编 `Delphi` 程序的朋友都遇到过这样的问题。如果遇有中文字符程序时，在 `DEDE` 里会显示成像`#12345#12345#12345`之类的编码，所以为了看懂这些编码的意思，写了一个可以将 `Delphi` 的编码字符转回中文字符的小工具。如果需要，这个工具还可以将 `Delphi` 的中文字符再转回编码形式。

  下面的代码只是随便写的，给兄弟们见笑了。![](img/9e41176b4900ac914e77ca870db1e3dd.jpg)  *}

{* 下面的源码需要一个窗口（废话）![](img/f5d9879e4703d3dcc963d0743af268d2.jpg) ，三个 `Button`，两个 `Edit` 就行了。![](img/d96007cf245c7669301e5df8beb41bf5.jpg)  *}

```
unit Ascii;

interface

uses
  Windows, Messages, SysUtils, Variants, Classes, Graphics, Controls, Forms,
  Dialogs, StdCtrls;

type
  TForm1 = class(TForm)
    Edit1: TEdit;
    Edit2: TEdit;
    Button1: TButton;
    Button2: TButton;
    Button3: TButton;
    Label1: TLabel;
    Label2: TLabel;
    procedure Button1Click(Sender: TObject);
    procedure Button2Click(Sender: TObject);
    procedure Button3Click(Sender: TObject);
  private
    { Private declarations }
  public
    { Public declarations }
  end;

var
  Form1: TForm1;

implementation

{$R *.dfm}

procedure TForm1.Button1Click(Sender: TObject);

var s1,s2:String;
    i:Integer;
    s3:WideChar;

begin
  s1 := Edit2.Text;
  Edit1.Text := '';
  s3 := #0;
  i := 1;
  if s1 = '' then
    Exit;
  while i <= length(s1) do
  begin
    s2 := '';
    if s1[i] = '#' then
    begin
      while i <= length(s1) do
      begin
        inc(i);
        if s1[i] <> '#' then
          s2 := s2 + s1[i]
        else
          break;
      end;
      s3 := WideChar(StrToInt(s2));
    end;
    Edit1.Text := Edit1.Text + s3;
    s3 := #0;
  end;
end;

procedure TForm1.Button2Click(Sender: TObject);

var s1:WideString;
    s2,s3:String;
    i:Integer;

begin
  s1 := Edit1.Text;
  s2 := '';
  s3 := '';
  if s1 = '' then
    Exit;
  for i := 1 to Length(s1) do
  begin
    s2 := '#' + IntToStr(Ord(s1[i]));
    s3 := s3 + s2;
  end;
  Edit2.Text := s3;
end;

procedure TForm1.Button3Click(Sender: TObject);

begin
  close;
end;

end.
```

附件下载：`widestringtoasscii.rar`

`4dd466d0e1e0cc123afbe504da2462ca`
