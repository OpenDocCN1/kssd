
# 为答谢看雪论坛,放出 QQ2008 的自动聊天精灵 delphi 源码

> 原文：[`www.pediy.com/kssd/pediy10/60655.html`](https://www.pediy.com/kssd/pediy10/60655.html)

在 `xpsp2+delphi7` 下正式通过编译,为了防止有朋友搞破坏,故没有没出无窗口 `SDK` 代码,免杀和 `7` 种自启动还有远程自动控制和特定人特制视频也已成功,但暂不放出,主要针对 `QQ2008`.同时兼容 `QQ07`,希望看雪论坛能越办越好.

//俏狐出品 QQ:86074731 最后修改于 2008 年 3 月

```pascal
unit Unit1;

interface

uses
Windows, Messages, SysUtils, Variants, Classes, Graphics, Controls, Forms,
Dialogs, StdCtrls, Buttons, ExtCtrls,Registry, ExtDlgs, bsSkinShellCtrls,
  BusinessSkinForm, bsSkinBoxCtrls, bsSkinCtrls;

type
TTform1 = class(TForm)
    GroupBox1: TGroupBox;
    Bevel1: TBevel;
    Label2: TLabel;
    Bevel2: TBevel;
    Bevel3: TBevel;
    Bevel4: TBevel;
    FindBtn: TSpeedButton;
    Image1: TImage;
    SendBtn: TSpeedButton;
    LoadBtn: TSpeedButton;
    loaddialog1: TOpenDialog;
    ListBox1: TListBox;
    bsBusinessSkinForm1: TbsBusinessSkinForm;
    bsSkinOpenDialog1: TbsSkinOpenDialog;
    AoutBtn: TSpeedButton;
    procedure FormCreate(Sender: TObject);
    procedure FindBtnClick(Sender: TObject);
    procedure LoadBtnClick(Sender: TObject);
    procedure SendBtnClick(Sender: TObject);
    procedure AoutBtnClick(Sender: TObject);

private
{ Private declarations }

public
{ Public declarations }

end;

var
Tform1: TTform1;

implementation

{$R \*.dfm}

//定义一组全程变量

const
   WinCaption07:string='聊天中';
   WinCaption08:string='交谈中';

var
  x:integer;
  TextBoxNum:shortint; //QQ 输入框是第几个对话框
  SendButtonNum:shortint; //发送按钮是第几个按钮
  QQInputBoxHandle,SendButtonHandle:HWND;//发送按钮和输入框句柄
  StopSend:boolean;

//=====================延时时程序===================

procedure Delay(msecs:integer);

var
FirstTickCount:longint;

begin
FirstTickCount:=GetTickCount;
repeat
    if STopSend then  exit ;
    Application.ProcessMessages;
until ((GetTickCount-FirstTickCount) >= Longint(msecs));
end;

//=====================得到窗口内容===================

function GetWindowStr(Wnd: HWND): String;

var
Len: Integer;

begin
Len := SendMessage(Wnd, WM_GETTEXTLENGTH, 0, 0);
SetLength(Result, Len + 1);
SendMessage(Wnd, WM_GETTEXT, Length(Result), Longint(@Result[1]));
end;

//=====================得到所属类===================

function GetWindowClass(Wnd: HWND): String;

begin
SetLength(Result, 65);
GetClassName(Wnd, @Result[1], 65);
end;

//=====================查找子控件===================

function EnumChildProc(Wnd: HWND; lParam: LPARAM): Boolean; stdcall;

var
S, C: String;

begin
  S := GetWindowStr(Wnd);
  C := GetWindowClass(Wnd);

     X:=X+1;
    if  Pos('RichEdit', C) =1  then
      begin
        TextBoxNum:=TextBoxNum+1;
        if  TextBoxNum =3 then QQInputBoxHandle :=Wnd;
      end;
    if (pos('发送',S) =1) and (Pos('Button', C) =1) then
      begin
        if  SendButtonNum=2 then  SendButtonHandle:=wnd;
        SendButtonNum:= SendButtonNum+1;
      end;

Result := True;
end;

//=====================定义一个回调函数===================

function EnumWindowsProc(Wnd: HWND; lParam: LPARAM): Boolean; stdcall;

var
S, C: String;

begin
S := GetWindowStr(Wnd);
C := GetWindowClass(Wnd);

//看是 07 和 08 版 QQ 的标题吗?
 if (Pos(WinCaption07, S) >0) or (Pos(WinCaption08, S) >0) then
  begin  //如果找到 QQ 窗体则找出所有控件
    if not EnumChildWindows(Wnd, @EnumChildProc, lParam) then ;
    Result := False;
  end;

Result := True;
end;

//=====================主表单初始化===================

procedure TTform1.FormCreate(Sender: TObject);

begin
  //初始化表单和列表框颜色
  Tform1.color:=tcolor(rgb(236,233,216));
  ListBox1.color:=Tcolor(rgb(96,96,97));
end;

//=====================查找 QQ 主窗体===================

procedure TTform1.FindBtnClick(Sender: TObject);

begin
  X:=0;
  TextBoxNum:=1;
  SendButtonNum:=1;
  try
  if not EnumWindows(@EnumWindowsProc, Integer(Pointer(ListBox1))) then ;
  finally
    if X = 0 then messagebox( Tform1.Handle,'不能找到ＱＱ发送窗口！','错误',MB_OK+MB_DEFBUTTON1 +MB_ICONHAND);  end;
  listbox1.ItemIndex:=0;
  if (QQInputBoxHandle<>0) and (SendButtonHandle <>0) then SendBtn.Enabled :=True;
end;

//=====================装入聊天记录===================

procedure TTform1.LoadBtnClick(Sender: TObject);

begin
 if bsSkinOpenDialog1.execute then
   begin
     ListBox1.Clear;
     ListBox1.Items.LoadFromFile(bsSkinOpenDialog1.filename);
   end;
end;

//=====================可中断的连续发送================

procedure TTform1.SendBtnClick(Sender: TObject);

var
  SendTxt:string;

begin
  StopSend := False; //把是否安暂停设为不停
  if SendBtn.Caption='发 送' then
    begin
      SendBtn.Caption :='暂 停';
    end
  else
    begin //如果是暂停按钮按下
      SendBtn.Caption:='发 送';
      StopSend:=True;
    end;
  while (listbox1.ItemIndex<ListBox1.Items.Count-1) and (not StopSend)  do
    begin
       listbox1.ItemIndex:=listbox1.ItemIndex+1;
       //如果导入的文本文件里有空行,则跳过空行
       while ListBox1.Items.strings[listbox1.ItemIndex]='' do listbox1.ItemIndex:=listbox1.ItemIndex+1;
       if STopSend then   exit; //如果暂停键按下
       SendTxt :=ListBox1.Items.strings[listbox1.ItemIndex];
       SendMessage(QQInputBoxHandle,EM_REPLACESEL,180,Integer(Pchar(SendTxt)));
       delay(300);
       SendMessage(SendButtonHandle,BM_CLICK,0,0);
    end;
end;

procedure TTform1.AoutBtnClick(Sender: TObject);

begin
  messagebox( Tform1.Handle,'为了梦想的天空而努力！','关于',MB_OK+MB_DEFBUTTON1 +MB_ICONQUESTION );
end;

end.
```
