
# Delphi 的注册机模板

> 原文：[`www.pediy.com/kssd/pediy05/pediy50360.htm`](https://www.pediy.com/kssd/pediy05/pediy50360.htm)

**标题**：一个 `Delphi` 的注册机模板，没有使用 `VCL`，类似 `C/ASM` 使用对话框模板，编译结果小，最方便的是可以使用 `Delphi` 的 `string` 类型

**内容**：

共 6 个文件，如下。只需添加注册算法到主程序文件的 `CalculateCode` 函数，运行 `Build.bat` 进行编译。（当然你必须安装了 `Delphi`, ;-)）

```c
// resource.h 
// 资源常量定义
// -----------------------------------------------------

#define IDD_MAIN          101
#define IDE_NAME          1011
#define IDE_CODE          1012
#define IDB_GENERATE      1013
#define IDB_ABOUT         1014
#define IDB_EXIT          1015
#define IDD_ABOUT         102
#define IDB_OK            1021
#define IDR_MAINFRAME     103
```

```pascal
// resource.pas 
// 资源常量定义的 Pascal 版本
// -----------------------------------------------------

unit resource;

interface

const
  IDD_MAIN       = 101;
  IDE_NAME       = 1011;
  IDE_CODE       = 1012;
  IDB_GENERATE   = 1013;
  IDB_ABOUT      = 1014;
  IDB_EXIT       = 1015;
  IDD_ABOUT      = 102;
  IDB_OK         = 1021;
  IDR_MAINFRAME  = 103;

implementation

end.
```

`KeyGen.ico` 图标文件

```rc
// KeyGen.rc 
// 资源脚本
// -----------------------------------------------------

#include "resource.h"

IDR_MAINFRAME         ICON    DISCARDABLE    "KeyGen.ico"

IDD_MAIN DIALOGEX 0, 0, 212, 76
STYLE DS_MODALFRAME | DS_CENTER | WS_POPUP | WS_VISIBLE | WS_CAPTION | WS_SYSMENU
EXSTYLE WS_EX_APPWINDOW
CAPTION "XXXX Key Generator"
FONT 8, "Verdana"

BEGIN
    EDITTEXT        IDE_NAME,38,12,163,12,ES_AUTOHSCROLL
    EDITTEXT        IDE_CODE,38,31,163,12,ES_AUTOHSCROLL | ES_READONLY
    PUSHBUTTON      "&Generate",IDB_GENERATE,5,56,50,14,0,WS_EX_STATICEDGE
    PUSHBUTTON      "&About",IDB_ABOUT,81,56,50,14,0,WS_EX_STATICEDGE
    PUSHBUTTON      "E&xit",IDB_EXIT,156,56,50,14,0,WS_EX_STATICEDGE
    LTEXT           "Name:",IDC_STATIC,10,14,24,8
    LTEXT           "Code:",IDC_STATIC,10,33,22,8
    GROUPBOX        "",IDC_STATIC,5,2,201,48

END

IDD_ABOUT DIALOGEX 0, 0, 233, 65
STYLE DS_MODALFRAME | DS_CENTER  | WS_POPUP | WS_CAPTION | WS_SYSMENU
CAPTION "About"
FONT 8, "Verdana"

BEGIN
    ICON            IDR_MAINFRAME,IDC_STATIC,7,7,20,20
    LTEXT           "KeyGen for XXXX",IDC_STATIC,34,7,136,8,SS_NOPREFIX
    LTEXT           "Programmed by Phidel",IDC_STATIC,34,21,119,8
    DEFPUSHBUTTON   "OK",IDB_OK,177,7,50,14,WS_GROUP,WS_EX_STATICEDGE
    LTEXT           "Compiled with Borland Delphi",IDC_STATIC,34,35,131,8
    LTEXT           "October, 25th, 2002",IDC_STATIC,34,49,100,8

END
```

```pascal
// KeyGen.dpr 
// 主程序文件
// -----------------------------------------------------

program KeyGen;

{$R KeyGen.res}

uses Windows, Messages, resource;

var
  WinClass: TWndClass;
  Inst: HINST;

function CalculateCode(Name: string): string;
begin
  Result := '';
  {Add Codes Here}
end;

procedure InitializeApp(hDlg: HWND; wParam: WPARAM; lParam: LPARAM);
begin
  SendMessage(hDlg, WM_SETICON, 1, LoadIcon(Inst, MAKEINTRESOURCE(IDR_MAINFRAME)));
end;

function AboutDialogFunc(hwndDlg: HWND; msg: UINT; wParam: WPARAM; lParam: LPARAM): Boolean; stdcall;
begin
  case msg of
    WM_COMMAND:
      begin
        case (LOWORD(wParam)) of
          IDB_OK:
            begin
              EndDialog(hwndDlg, 1);
              Result := True;
              Exit;
            end;
        end;
        Result := False;
        Exit;
      end;
    WM_CLOSE:
      begin
        EndDialog(hwndDlg, 0);
        Result := True;
        Exit;
      end;
  end;

  Result := False;
end;

function DialogFunc(hwndDlg: HWND; msg: UINT; wParam: WPARAM; lParam: LPARAM): Boolean; stdcall;
var
  Name: array[0..255] of Char;
begin
  case msg of
    WM_INITDIALOG:
      begin
        InitializeApp(hwndDlg, wParam, lParam);
        Result := True;
        Exit;
      end;
    WM_COMMAND:
      begin
        case (LOWORD(wParam)) of
          IDB_GENERATE:
            begin
              GetDlgItemText(hwndDlg, IDE_NAME, Name, 255);
              SetDlgItemText(hwndDlg, IDE_CODE, PChar(CalculateCode(string(Name))));
              Result := True;
              Exit;
            end;

          IDE_NAME:
            begin
              if HIWORD(wParam) = EN_CHANGE then
              begin
                SetDlgItemText(hwndDlg, IDE_CODE, '');
                Result := True;
              end
              else
                Result := False;
              Exit;
            end;

          IDB_ABOUT:
            begin
              DialogBox(Inst, MAKEINTRESOURCE(IDD_ABOUT), hwndDlg, @AboutDialogFunc);
              Result := True;
              Exit;
            end;

          IDB_EXIT:
            begin
              EndDialog(hwndDlg, 1);
              Result := True;
              Exit;
            end;
        end;
        Result := False;
        Exit;
      end;
    WM_CLOSE:
      begin
        EndDialog(hwndDlg, 0);
        Result := True;
        Exit;
      end;
  end;

  Result := False;
end;

begin
  Inst := hInstance;
  with WinClass do
  begin
    lpfnWndProc := @DefDlgProc;
    cbWndExtra := DLGWINDOWEXTRA;
    hInstance := Inst;
    hCursor := LoadCursor(0, IDC_ARROW);
    hbrBackground := COLOR_WINDOW + 1;
    lpszClassname := 'KeyGen';
  end;

  RegisterClass(WinClass);

  DialogBox(Inst, MAKEINTRESOURCE(IDD_MAIN), 0, @DialogFunc);
end.
```

```bat
// Build.bat 
// 编译批处理文件
// -----------------------------------------------------

brcc32 KeyGen.rc
dcc32 KeyGen.dpr
```
