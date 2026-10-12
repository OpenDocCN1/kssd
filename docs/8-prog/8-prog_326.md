
# 利用 hook 编写通用破解程序

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-321.htm`](https://www.pediy.com/kssd/pediy07/pediy7-321.htm)

市面上很多软件运行后提示需要注册或者加密狗，如果无狗或不注册，则软件的主窗口自动关闭退出。针对这类加密的程序，可以使用 hook 编程拦截`wm_close`或`wm_quit`消息，阻止程序主窗口的关闭和程序的退出。当然，必须先确认程序在提示注册或提示无狗或自动退出之前已经加载所有必须的模块而且主窗体构造完成（不一定需要已经显示），否则你阻止了程序的退出，软件仍然会不能正常使用。

下面我以日本的某电路设计软件为例，破解思路和破解程序代码如下：

该软件需要加密狗，运行程序后会通过 timer 检测加密狗，如果没有发现加密狗或者加密狗被拔除，则产生`wm_close`消息，通知主窗体退出。

你可以选择拦截 timer 事件，阻止他检测加密狗，但是这样会导致软件其他地方需要 timer 事件触发的功能不正常。所以必须拦截加密狗检测完成后向主窗体发送的`wm_close`事件。

以下是我拦截事件的代码。

```pascal
library hdll;

uses
  SysUtils,
  dialogs,
  Classes,
  windows,
  messages,
  shellapi;

{$R \*.res}

var
  syshook: hhook;
  SavedProc: pointer;

function WinProc(hWnd: HWND; Msg: UINT; wParam: WPARAM; lParam: LPARAM): LRESULT; stdcall;
var
  msgstr: string;
begin
  msgstr := 'If you want close applicatin,please close dog simulator first!';

  case msg of
    WM_CLOSE: begin
      showmessage(msgstr);
      result := 1;
    end;
    else
      result := CallWindowProc(SavedProc, hwnd, msg, wparam, lparam);
  end;
end;

procedure GetAndSet(h: integer); stdcall;
begin
  if pointer(getwindowlong(h, GWL_WNDPROC)) <> @winProc then
    begin
      SavedProc := pointer(SetWindowLong(h, GWL_WNDPROC, cardinal(@Winproc)));
    end;
end;

function CallWndProc(
    nCode: integer;   // hook code
    wParam:    WPARAM;   // current-process flag
    lParam:    LPARAM    // address of structure with message data
    ): LRESULT; stdcall;
var
    h: longint;
begin
  h := FindWindow(pchar('DKMagic_CLASS'), nil);

  if tmsg(pointer(lparam)^).hwnd = h then
    begin
      GetAndSet(h);
    end;

  result := CallNextHookex(syshook, ncode, wparam, lparam);
end;

procedure RunStopHook(b: boolean); export; stdcall;
begin
  if b
    then SyShook := Setwindowshookex(WH_GETMESSAGE, @callWndProc, Hinstance, 0)
    else unhookwindowshookex(syshook);

  if (syshook = cardinal(-1)) or (syshook = cardinal(0)) then
    messagebox(0, 'hook 失败', '', mb_ok);
end;

exports RunStopHook;

begin
end.
```

这个破解方法对国内众多共享软件也测试通过。

2005/7/25 网网 QQ:108508999

4dd466d0e1e0cc123afbe504da2462ca
