

# Delphi 的注册机模板

> 原文：[`www.pediy.com/kssd/pediy05/pediy50360.htm`](https://www.pediy.com/kssd/pediy05/pediy50360.htm)

标题: 一个 Delphi 的注册机模板,没有使用 VCL,类似 C/ASM 使用对话框模板,编译结果小,最方便的是可以使用 Delphi 的 string 类型   
  
内容:   
共 6 个文件,如下. 只需添加注册算法到主程序文件的 CalculateCode 函数,运行 Build.bat 进行编译.(当然你必须安装了 Delphi, ;-)   
  
\-----------------------------------------------------   
resource.h    资源常量定义   
\-----------------------------------------------------   
#define IDD\_MAIN                        101   
#define IDE\_NAME                        1011   
#define IDE\_CODE                        1012   
#define IDB\_GENERATE                    1013   
#define IDB\_ABOUT                      1014   
#define IDB\_EXIT                        1015   
#define IDD\_ABOUT                      102   
#define IDB\_OK                          1021   
#define IDR\_MAINFRAME                  103   
  
  
\-----------------------------------------------------   
resource.pas  资源常量定义的 Pascal 版本   
\-----------------------------------------------------   
unit resource;   
  
interface   
  
const   
  IDD\_MAIN      =  101;   
  IDE\_NAME      =  1011;   
  IDE\_CODE      =  1012;   
  IDB\_GENERATE  =  1013;   
  IDB\_ABOUT    =  1014;   
  IDB\_EXIT      =  1015;   
  IDD\_ABOUT    =  102;   
  IDB\_OK        =  1021;   
  IDR\_MAINFRAME =  103;   
  
implementation   
  
end.   
  
  
\-----------------------------------------------------   
KeyGen.ico  图标文件   
\-----------------------------------------------------   
  
  
\-----------------------------------------------------   
KeyGen.rc  资源脚本   
\-----------------------------------------------------   
#include "resource.h"   
  
IDR\_MAINFRAME          ICON    DISCARDABLE    "KeyGen.ico"   
  
IDD\_MAIN DIALOGEX 0, 0, 212, 76   
STYLE DS\_MODALFRAME | DS\_CENTER | WS\_POPUP | WS\_VISIBLE | WS\_CAPTION | WS\_SYSMENU   
EXSTYLE WS\_EX\_APPWINDOW   
CAPTION "XXXX Key Generator"   
FONT 8, "Verdana"   
BEGIN   
    EDITTEXT        IDE\_NAME,38,12,163,12,ES\_AUTOHSCROLL   
    EDITTEXT        IDE\_CODE,38,31,163,12,ES\_AUTOHSCROLL | ES\_READONLY   
    PUSHBUTTON      "&Generate",IDB\_GENERATE,5,56,50,14,0,WS\_EX\_STATICEDGE   
    PUSHBUTTON      "&About",IDB\_ABOUT,81,56,50,14,0,WS\_EX\_STATICEDGE   
    PUSHBUTTON      "E&xit",IDB\_EXIT,156,56,50,14,0,WS\_EX\_STATICEDGE   
    LTEXT          "Name:",IDC\_STATIC,10,14,24,8   
    LTEXT          "Code:",IDC\_STATIC,10,33,22,8   
    GROUPBOX        "",IDC\_STATIC,5,2,201,48   
END   
  
IDD\_ABOUT DIALOGEX 0, 0, 233, 65   
STYLE DS\_MODALFRAME | DS\_CENTER  | WS\_POPUP | WS\_CAPTION | WS\_SYSMENU   
CAPTION "About"   
FONT 8, "Verdana"   
BEGIN   
    ICON            IDR\_MAINFRAME,IDC\_STATIC,7,7,20,20   
    LTEXT          "KeyGen for XXXX",IDC\_STATIC,34,7,136,8,SS\_NOPREFIX   
    LTEXT          "Programmed by Phidel",IDC\_STATIC,34,21,119,8   
    DEFPUSHBUTTON  "OK",IDB\_OK,177,7,50,14,WS\_GROUP,WS\_EX\_STATICEDGE   
    LTEXT          "Compiled with Borland Delphi",IDC\_STATIC,34,35,131,8   
    LTEXT          "October, 25th, 2002",IDC\_STATIC,34,49,100,8   
END   
  
  
\-----------------------------------------------------   
KeyGen.dpr 主程序文件  
\-----------------------------------------------------   
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
  SendMessage(hDlg, WM\_SETICON, 1, LoadIcon(Inst, MAKEINTRESOURCE(IDR\_MAINFRAME)));   
end;   
  
function AboutDialogFunc(hwndDlg: HWND; msg: UINT; wParam: WPARAM; lParam: LPARAM): Boolean; stdcall;   
begin   
  case msg of   
    WM\_COMMAND:   
      begin   
        case (LOWORD(wParam)) of   
          IDB\_OK:   
            begin   
              EndDialog(hwndDlg, 1);   
              Result := True;   
              Exit;   
            end;   
        end;   
        Result := False;   
        Exit;   
      end;   
    WM\_CLOSE:   
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
  Name: array\[0..255\] of Char;   
begin   
  case msg of   
    WM\_INITDIALOG:   
      begin   
        InitializeApp(hwndDlg, wParam, lParam);   
        Result := True;   
        Exit;   
      end;   
    WM\_COMMAND:   
      begin   
        case (LOWORD(wParam)) of   
          IDB\_GENERATE:   
            begin   
              GetDlgItemText(hwndDlg, IDE\_NAME, Name, 255);   
              SetDlgItemText(hwndDlg, IDE\_CODE, PChar(CalculateCode(string(Name))));   
              Result := True;   
              Exit;   
            end;   
          IDE\_NAME:   
            begin   
              if HIWORD(wParam) = EN\_CHANGE then   
              begin   
                SetDlgItemText(hwndDlg, IDE\_CODE, '');   
                Result := True;   
              end   
              else   
                Result := False;   
              Exit;   
            end;   
          IDB\_ABOUT:   
            begin   
              DialogBox(Inst, MAKEINTRESOURCE(IDD\_ABOUT), hwndDlg, @AboutDialogFunc);   
              Result := True;   
              Exit;   
            end;   
          IDB\_EXIT:   
            begin   
              EndDialog(hwndDlg, 1);   
              Result := True;   
              Exit;   
            end;   
        end;   
        Result := False;   
        Exit;   
      end;   
    WM\_CLOSE:   
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
    hCursor := LoadCursor(0, IDC\_ARROW);   
    hbrBackground := COLOR\_WINDOW + 1;   
    lpszClassname := 'KeyGen';   
  end;   
  
  RegisterClass(WinClass);   
  
  DialogBox(Inst, MAKEINTRESOURCE(IDD\_MAIN), 0, @DialogFunc);   
end.   
  
  
\-----------------------------------------------------   
Build.bat  编译批处理文件   
\-----------------------------------------------------   
brcc32 KeyGen.rc   
dcc32 KeyGen.dpr  4dd466d0e1e0cc123afbe504da2462ca