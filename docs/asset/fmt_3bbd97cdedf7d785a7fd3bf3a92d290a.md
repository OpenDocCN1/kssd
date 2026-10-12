
# Tooltip Control Message Interception and Configuration

When the above step is completed, the tooltip control knows which area it should monitor for mouse messages and what text it should display in the tooltip window. The only thing it lacks is the `trigger` for that action. Think about it: the area specified by the tool is on the client area of the other window. How can the tooltip control intercept the mouse messages destined for that window? It needs to do so in order that it can measure the amount of time the mouse pointer hovers over a point in the tool so that when the specified amount of time elapses, the tooltip control shows the tooltip window. There are two methods of accomplishing this goal, one that requires the cooperation of the window that contains the tool and the other without the cooperation on the part of the window.

当完成上面的步骤后，工具提示控件应该知道了在那一块区域监听鼠标消息以及在工具提示窗口显示什么样的文本。唯一缺少的就是“触发”机制。想想看：被工具指定的区域如果在其它窗口的客户区上。工具提示控件又怎么截获发送给窗口的鼠标消息呢？它需要去做的就是测量鼠标指针在工具上停留的时间值，以至于当指定的时间值过去时，工具控件显示工具提示窗口。完成这个目标有两种方法：一种需要包含工具的窗口合作而另一种则不需要。

## Relaying Mouse Messages

The window that contains the tool must relay the mouse messages to the tooltip control by sending `TTM_RELAYEVENT` messages to the control. The `lParam` of this message must contain the address of a `MSG` structure that specifies the message to be relayed to the tooltip control. A tooltip control processes only the following mouse messages:

包含工具的的窗口必须通过发送 `TTM_RELAYEVENT` 消息给控件以便将鼠标消息转发给工具提示控件。消息的 `lParam` 参数必须包含指定发送给工具提示控件的消息结构体地址。工具提示控件只处理下面的鼠标消息：

- `WM_LBUTTONDOWN`
- `WM_MOUSEMOVE`
- `WM_LBUTTONUP`
- `WM_RBUTTONDOWN`
- `WM_MBUTTONDOWN`
- `WM_RBUTTONUP`
- `WM_MBUTTONUP`

All other messages are ignored. Thus in the window procedure of the window that contains the tool, there must be a switch that does something like this:

所有其它的消息都被忽略。因此在包含工具的窗口的窗口过程中，必须有如下的开关语句：

```assembly
WndProc proc hWnd:DWORD, uMsg:DWORD, wParam:DWORD, lParam:DWORD

.......

    if uMsg==WM_CREATE
        ............
    elseif uMsg==WM_LBUTTONDOWN || uMsg==WM_MOUSEMOVE || uMsg==WM_LBUTTONUP || uMsg==WM_RBUTTONDOWN || uMsg==WM_MBUTTONDOWN || uMsg==WM_RBUTTONUP || uMsg==WM_MBUTTONUP
        invoke SendMessage, hwndTooltip, TTM_RELAYEVENT, NULL, addr msg
        ..........
```

## Subclassing the Window

You can specify `TTF_SUBCLASS` flag in the `uFlags` member of the `TOOLINFO` structure. This flag tells the tooltip control to subclass the window that contains the tool so it can intercept the mouse messages without the cooperation of the window. This method is easier to use since it doesn't require more coding than specifying `TTF_SUBCLASS` flag and the tooltip control handles all the message interception itself.

你能在 `TOOLINFO` 结构的 `uFlags` 成员中指定 `TTF_SUBCLASS` 标志。这个标志告诉工具提示控件子类化包含工具的窗口，以便在没有窗口的协助下它也能截获鼠标消息。这个方法很容易使用，因为除了指定 `TTF_SUBCLASS` 标志和工具提示控件自己处理截获的所有消息外，它并不需要更多的编码。

## Tooltip-Related Messages

That's it. At this step, your tooltip control is fully functional. There are several useful tooltip-related messages you should know about.

就是这些了，到这一步，你的工具提示控件功能已经很完整了。这里还有几个和工具提示有关的消息你应该知道。

### `TTM_ACTIVATE`

`TTM_ACTIVATE`. If you want to disable/enable the tooltip control dynamically, this message is for you. If the `wParam` value is `TRUE`, the tooltip control is enabled. If the `wParam` value is `FALSE`, the tooltip control is disabled. A tooltip control is enabled when it first created so you don't need to send this message to activate it.

`TTM_ACTIVATE`. 如果你想动态的停用和启用工具提示控件，这个消息是为你准备的。如果 `wParam` 的值是 `TRUE`，那么工具提示控件被启用。如果 `wParam` 的值是 `FALSE`，工具提示控件被停用。一个工具提示控件在第一次被创建时是可用的，所以你并不需要发送这个消息去激活它。

### `TTM_GETTOOLINFO` and `TTM_SETTOOLINFO`

`TTM_GETTOOLINFO` and `TTM_SETTOOLINFO`. If you want to obtain/change the values in the `TOOLINFO` structure after it was submitted to the tooltip control, use these messages. You need to specify the tool you need to change with the correct `uId` and `hWnd` values. If you only want to change the `rect` member, use `TTM_NEWTOOLRECT` message. If you only want to change the tooltip text, use `TTM_UPDATETIPTEXT`.

`TTM_GETTOOLINFO` 和 `TTM_SETTOOLINFO`。如果你想在将 `TOOLINFO` 结构体传送给工具提示控件后，获取或是改变结构体中的值，用这两个消息。你需要用正确的 `uId` 和 `hWnd` 的值来指定你想改变的工具。如果你仅想改变 `rect` 成员，用 `TTM_NEWTOOLRECT` 消息。如果你仅想改变工具提示文本，用 `TTM_UPDATETIPTEXT` 消息。

### `TTM_SETDELAYTIME`

`TTM_SETDELAYTIME`. With this message, you can specify the time delay the tooltip control uses when it's displaying the tooltip text and much more.

`TTM_SETDELAYTIME`. 当工具提示控件正显示提示文本或做其它工作时，你可以用这个消息指定它的时间延迟。

## Example

The following example is a simple dialog box with two buttons. The client area of the dialog box is divided into 4 areas: upper left, upper right, lower left and lower right. Each area is specified as a tool with its own tooltip text. The two buttons also has their own tooltip texts.

下面的例子是一个简单的对话框和两个按钮。对话框的客户区被分成四个区域：左上，右上，左下，右下。每一块区域都被指定为有自己提示文本的工具。两个按钮也有它们自己的提示文本。

```assembly
.386

.model flat,stdcall

option casemap:none

include \masm32\include\windows.inc

include \masm32\include\kernel32.inc

include \masm32\include\user32.inc

include \masm32\include\comctl32.inc

includelib \masm32\lib\comctl32.lib

includelib \masm32\lib\user32.lib

includelib \masm32\lib\kernel32.lib

DlgProc proto :DWORD,:DWORD,:DWORD,:DWORD

EnumChild proto :DWORD,:DWORD

SetDlgToolArea proto :DWORD,:DWORD,:DWORD,:DWORD,:DWORD

.const

IDD_MAINDIALOG equ 101

.data

ToolTipsClassName db "Tooltips_class32",0

MainDialogText1 db "This is the upper left area of the dialog",0

MainDialogText2 db "This is the upper right area of the dialog",0

MainDialogText3 db "This is the lower left area of the dialog",0

MainDialogText4 db "This is the lower right area of the dialog",0

.data?

hwndTool dd ?

hInstance dd ?

.code

start:

    invoke GetModuleHandle,NULL

    mov hInstance,eax

    invoke DialogBoxParam,hInstance,IDD_MAINDIALOG,NULL,addr DlgProc,NULL

    invoke ExitProcess,eax

DlgProc proc hDlg:DWORD,uMsg:DWORD,wParam:DWORD,lParam:DWORD

    LOCAL ti:TOOLINFO

    LOCAL id:DWORD

    LOCAL rect:RECT

    .if uMsg==WM_INITDIALOG

        invoke InitCommonControls

        invoke CreateWindowEx,NULL,ADDR ToolTipsClassName,NULL,\

            TTS_ALWAYSTIP,CW_USEDEFAULT,\

            CW_USEDEFAULT,CW_USEDEFAULT,CW_USEDEFAULT,NULL,NULL,\

            hInstance,NULL

        mov hwndTool,eax

        mov id,0

        mov ti.cbSize,sizeof TOOLINFO

        mov ti.uFlags,TTF_SUBCLASS

        push hDlg

        pop ti.hWnd

        invoke GetWindowRect,hDlg,addr rect

        invoke SetDlgToolArea,hDlg,addr ti,addr MainDialogText1,id,addr rect

        inc id

        invoke SetDlgToolArea,hDlg,addr ti,addr MainDialogText2,id,addr rect

        inc id

        invoke SetDlgToolArea,hDlg,addr ti,addr MainDialogText3,id,addr rect

        inc id

        invoke SetDlgToolArea,hDlg,addr ti,addr MainDialogText4,id,addr rect

        invoke EnumChildWindows,hDlg,addr EnumChild,addr ti

    .elseif uMsg==WM_CLOSE

        invoke EndDialog,hDlg,NULL

    .else

        mov eax,FALSE

        ret

    .endif

    mov eax,TRUE

    ret

DlgProc endp

EnumChild proc uses edi hwndChild:DWORD,lParam:DWORD

    LOCAL buffer[256]:BYTE

    mov edi,lParam

    assume edi:ptr TOOLINFO

    push hwndChild

    pop [edi].uId

    or [edi].uFlags,TTF_IDISHWND

    invoke GetWindowText,hwndChild,addr buffer,255

    lea eax,buffer

    mov [edi].lpszText,eax

    invoke SendMessage,hwndTool,TTM_ADDTOOL,NULL,edi

    assume edi:nothing

    ret

EnumChild endp

SetDlgToolArea proc uses edi esi hDlg:DWORD,lpti:DWORD,lpText:DWORD,id:DWORD,lprect:DWORD

    mov edi,lpti

    mov esi,lprect

    assume esi:ptr RECT

    assume edi:ptr TOOLINFO

    .if id==0

        mov [edi].rect.left,0

        mov [edi].rect.top,0

        mov eax,[esi].right

        sub eax,[esi].left

        shr eax,1

        mov [edi].rect.right,eax
```
