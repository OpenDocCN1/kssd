
# Win32 asm 第二十七课：工具提示控件

> 原文：[`www.pediy.com/kssd/pediy10/60886.html`](https://www.pediy.com/kssd/pediy10/60886.html)

Tutorial 27: Tooltip Control
第二十七课：工具提示控件

We will learn about the tooltip control: What it is and how to create and use it.
我们将学习工具提示控件的有关内容：什么是工具提示控件以及如何创建并使用它。

Download the example.

### Theory

A tooltip is a small rectangular window that is displayed when the mouse pointer hovers over some specific area. A tooltip window contains some text that the programmer wants to be displayed. In this regard, a tooltip servers the same role as the status window but it disappears when the user clicks or moves the mouse pointer away from the designated area. You'll probably be familiar with the tooltips that are associated with toolbar buttons. Those "tooltips" are conveniencies provided by the toolbar control. If you want tooltips for other windows/controls, you need to create your own tooltip control.

工具提示控件是当鼠标指针在一些特定区域停留时被显示的一个小矩形窗口。一个工具提示窗口包含了一些程序员想显示的文本。在这一点上，工具提示服务程序和状态栏扮演着同样的角色，但是当用户的鼠标被点击或是将鼠标从指定区域移走时，工具提示窗口将消失不见。你可能很熟悉工具提示控件，它们和工具栏的按钮相关联。这些“提示”为工具栏控件提供了方便。如果你想在其它窗口／控件中使用工具提示，那么你需要创建你自己的工具提示控件。

Now that you know what a tooltip is, let's go on to how we can create and use it. The steps are outlined below:

现在，你知道了什么是工具提示控件，让我们学习如何创建和使用它。步骤略述如下：

1. Create a tooltip control with `CreateWindowEx`
   用 `CreateWindowEx` 创建工具提示控件。
2. Define a region that the tooltip control will monitor for mouse pointer movement.
   定义一个区域，工具提示控件将监控这个区域的鼠标移动事件。
3. Submit the region to the tooltip control
   传递这块区域给工具提示控件。
4. Relay mouse messages of the submitted region to the tooltip control (this step may occur earlier, depending on the method used to relay the messages)
   将已传递区域的鼠标消息转发给工具提示控件。（这步可能发生的更早，具体依赖于转发消息的方法。）

We will next examine each step in detail.
我们将详细的分析每一步：

### Tooltip Creation 工具提示控件的创建

A tooltip control is a common control. As such, you need to call `InitCommonControls` somewhere in your source code so that MASM implicitly links your program to `comctl32.dll`. You create a tooltip control with `CreateWindowEx`. The typical scenario would be like this:

工具提示控件是一个通用控件。同样的，你需要在源代码的某一处调用 `InitCommonControls` 函数，以便 MASM 将你的程序和 `comctl32.dll` 链接。然后，你可以用 `CreateWindowEx` 函数创建工具提示控件。典型的方案如下：

```asm
.data 
TooltipClassName db "Tooltips_class32",0 
.code 
..... 
invoke InitCommonControls 
invoke CreateWindowEx, NULL, addr TooltipClassName, NULL, TIS_ALWAYSTIP, CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT, NULL, hInstance, NULL
```

Note the window style: `TIS_ALWAYSTIP`. This style specifies that the tooltip will be shown when the mouse pointer is over the designated area regardless of the status of the window that contains the area. Put simply, if you use this flag, when the mouse pointer hovers over the area you register to the tooltip control, the tooltip window will appear even if the window under the mouse pointer is inactive.

注意窗口的样式：`TIS_ALWAYSTIP`。它指出了当鼠标指针在指定区域停留时，工具提示将被显示，而和包含这块区域的窗口的状态无关。简单的说，如果你用这个标志，当鼠标指针停留在你注册了的工具提示控件的那块区域上时，工具提示将显示，即使是鼠标指针下面的窗口并没被激活。

You don't have to include `WS_POPUP` and `WS_EX_TOOLWINDOW` styles in `CreateWindowEx` because the tooltip control's window procedure adds them automatically. You also don't need to specify the coordinate, the height and width of the tooltip window: the tooltip control will adjust them automatically to fit the tooltip text that will be displayed, thus we supply `CW_USEDEFAULT` in all four parameters. The remaining parameters are not remarkable.

你并不需要在 `CreateWindowEx` 函数包含 `WS_POPUP` 和 `WS_EX_TOOLWINDOW` 样式，因为工具提示控件的窗口过程会自动的添加它们。你也并不需要指定工具提示窗口的高和宽的坐标：工具提示控件会自动根据它将显示的文本调整合适的窗口大小。因此，我们提供 `CW_USEDEFAULT` 给这四个参数。剩下的参数是不值得注意的。

### Specifying the tool 指定工具

The tooltip control is created but it's not shown immediately. We want the tooltip window to show up when the mouse pointer hovers over some area. Now is the time to specify that area. We call such area "tool". A tool is a rectangular area on the client area of a window which the tooltip control will monitor for mouse pointer. If the mouse pointer hovers over the tool, the tooltip window will appear.

工具提示控件被创建，但是它并不会立即显示。我们希望的是，当鼠标指针在一些区域上停留时，工具提示窗口在露面。现在是时候指定这个区域了。我们叫这样的区域为“工具”。工具是在一个窗口客户区上的一矩形区域，工具提示控件将监视这个窗口的鼠标指针。如果鼠标指针停留在工具上，工具提示窗口将显示。

The rectangular area can cover the whole client area or only a part of it. So we can divided tool into two types: one that is implemented as a window and another that is implemented as a rectangular area in the client area of some window. Both has their uses. The tool that covers the whole client area of a window is most frequently used with controls such as buttons, edit controls and so on.

You don't need to specify the coordinate and the dimensions of the tool: it's assumed to be the whole client area of the window. The tool that is implemented as a rectangular area on the client area is useful when you want to divide the client area of a window into several regions without using child windows. With this type of tool, you need to specify the coordinate of the upper left corner and the width and height of the tool.

这个矩形区域可以覆盖整个客户区或者仅是客户区的一部分。所以我们能把工具分成两种类型：一种是作为整个窗口的工具，另一种是某窗口客户区中的一部分。两种各有所用。覆盖整个窗口客户区的工具常常被用在诸如按钮，编辑框等等一系列的控件上。你并不需要指定工具的大小和坐标：它被假定为某个窗口的整个客户区。覆盖窗口客户区一部分的"工具"在你想把窗口客户区分成几个部分但又不想使用子窗口时特别有用。用这种类型的工具，你需要指定“工具”的左上角坐标和“工具”的高度和宽度。

You specify the tool with the `TOOLINFO` structure which has the following definition:

你能用 `TOOLINFO` 结构体来指定“工具“，它的定义如下：

```asm
TOOLINFO STRUCT 
  cbSize          DWORD      ? 
  uFlags         DWORD      ? 
  hWnd           DWORD      ? 
  uId            DWORD      ? 
  rect           RECT       <> 
  hInst          DWORD      ? 
  lpszText       DWORD      ? 
  lParam         LPARAM     ? 
TOOLINFO ENDS 
```

**Field Name**
字段名
**Explanation**
说明

`cbSize`: The size of the `TOOLINFO` structure. You MUST fill this member. Windows will not flag error if this field is not filled properly but you will receive strange, unpredictable results.
`TOOLINFO` 结构的大小。你必须填充这个成员。如果填入不合适的值，Windows 并不会报错，但是你将得到奇怪的，不可预知的结果。

`uFlags`: The bit flags that specifies the characteristics of the tool. This value can be a combination of the following flags:
位标志指定了工具的特征。这个值能被下面的标志组合。

`TTF_IDISHWND`: "ID is hWnd". If you specify this flag, it means you want to use a tool that covers the whole client area of a window (the first type of tool above). If you use this flag, you must fill the `uId` member of this structure with the handle of the window you want to use. If you don't specify this flag, it means you want to use the second type of tool, the one that is implemented as the rectangular area on the client window. In that case, you need to fill the `rect` member with the dimension of the rectangle.
`TTF_IDISHWND` “ID is Hwnd “ 如果你指定了这标志，它意味着你想使用覆盖某个窗口整个客户区的“工具“（上面的第一种工具）。如果你使用这个标志，你必须用你想使用的窗口句柄填充这个结构的 `uId` 成员。如果你不指定这个标志，它意味你想使用第二种工具，即作为某个窗口客户区一部分的工具。在这种情况下，你需要填充 `rect` 成员作为这个矩形的大小。

`TTF_CENTERTIP`: Normally the tooltip window will appear to the right and below the mouse pointer. If you specify this flag, the tooltip window will always appear directly below the tool and is centered regardless of the position of the mouse pointer.
`TTF_CENTERTIP` 通常工具提示窗口将显示在鼠标指针的右下方。如果你指定这个标志，不论鼠标指针的位置如何，工具提示窗口将立即显示在鼠标的中下方。

`TTF_RTLREADING`: You can forget about this flag if your program is not designed specifically for Arabic or Hebrew systems. This flag displays the tooltip text with right-to-left reading order. Doesn't work under other systems.
`TTF_RTLREADING` 如果你的程序不是特别为阿拉伯人或是希伯来人的系统设计的，你能忽略这个标志。这个标志显示阅读顺序为从右到左的文本提示。在其他系统中无效。



# `TTF_SUBCLASS`

If you use this flag, it means you tell the tooltip control to subclass the window that the tool is on so that the tooltip control can intercept mouse messages that are sent to the window. This flag is very handy. If you don't use this flag, you have to do more work to relay the mouse messages to the tooltip control.

如果你使用这个标志，它意味着你告诉工具提示控件子类化工具所在的窗口，以便让工具提示控件能够窃听发送给窗口的鼠标消息。这个标志是非常方便的。如果你不使用这个标志，你必须做更多的工作来将鼠标消息转发给工具提示控件。

# `hWnd`

Handle to the window that contains the tool. If you specify `TTF_IDISHWND` flag, this field is ignored since Windows will use the value in `uId` member as the window handle. You need to fill this field if:

包含工具的窗口句柄。如果你指定 `TTF_IDISHWND` 标志，这个标志你可以忽略它，因为 `windows` 将使用在 `uId` 成员中的值作为窗口句柄。你需要填充这个字段，如果：

- 你没有使用 `TTF_IDISHWND` 标志。（换句话说，你使用矩形“工具“）
- 你在 `lpszText` 成员中指定了 `LPSTR_TEXTCALLBACK` 值。这个值告诉工具提示控件，当需要显示提示窗口时,必须向包含"工具"的窗口查询应该显示什么。这是一种动态快速的提示文本更新。如果你想动态的改变你的提示文本，你应该在 `lpszText` 中放入 `LPSTR_TEXTCALLBACK`。控件会将发送 `TTN_NEEDTEXT` 通知消息给由 `hWnd` 字段中的窗口句柄所指定的窗口。

# `uId`

The value in this field can have two meanings, depending on whether the `uFlags` member contains the flag `TTF_IDISHWND`.

在这个字段中的值可以有可以有两种含义，依赖于 `uFlags` 成员是否包含 `TTF_IDISHWND`。

- 如果 `TTF_IDISHWND` 标志没有被指定，就代表应用程序定义的工具 `ID` 值。因为这意味这你使用的是仅覆盖客户区一部分的工具，逻辑上来说，在同一个客户区中这种工具可以有很多个（但不存在交迭）。工具提示控件需要一种方式来区分它们。既然这样，在 `hwnd` 成员中的一个窗口句柄是不足够的，因为所有的工具都在同一个窗口中。因此，应用程序必须定义 `ID` 值来区分它们，这些 `ID` 可以是任何值，但要保证它们之间的唯一性。
- 如果 `TTF_IDISHWND` 标志被指定, 就表示整个客户区都作为焦点域的窗口句柄.你可能会奇怪为什么要用这个域来代替上面 `hwnd` 域来储存窗口句柄.答案是：如果在 `lpszText` 成员的值被 `LPSTR_TEXTCALLBACK` 指定, 那么这个 `hwnd` 可能已经被填充.或者为提示文本负责的窗口可能包含的并不是同一工具.( 你可以设计一个能充当两种角色的窗口程序,但这会受到很多限制,在这点上,微软给了你更多的自由,举杯欢呼吧!)

# `rect`

`RECT` structure that specifies the dimension of the tool. This structure defines a rectangle relative to the upper left corner of the client area of the window specified by the `hWnd` member. In short, you must fill this structure if you want to specify a tool that covers only a part of the client area. The tooltip control will ignore this field if you specify `TTF_IDISHWND` flag (you choose to use a tool that covers the whole client area).

`RECT` 结构指定工具的尺寸大小.这个结构相对于由 `hwnd` 成员指定的窗口客户区的左上角坐标来定义一个矩形.较而言之,如果你想指定一个仅覆盖客户区一部分的工具,你必须填充这个结构.如果你指定 `TTF_IDISHWND` 标志，工具提示控件将忽视这个字段中的值。（即，你选择使用一个覆盖整个客户区的工具）

# `hInst`

The handle of the instance that contains the string resource that will be used as the tooltip text if the value in the `lpszText` member specifies the string resource identifier. This may sound confusing. Read the explanation of the `lpszText` member first and you will understand what this field is used for. The tooltip control ignores this field if the `lpszText` field doesn't contain a string resource identifier.

包含字符串资源的实例句柄，如果在 `lpszText` 成员中指定了字符串资源标识符，那么这个字符串资源将被用作工具提示文本。听起来可能有点费劲。阅读一下 `lpszText` 的说明就可以明白这个字段是干什么用的了.若 `lpszText` 不包含字符串资源标识,工具控件会忽略这个字段.

# `lpszText`

This field can have several values:

这个字段可以有以下几个值：

- 如果指定为 `LPSTR_TEXTCALLBACK`, 工具提示控件就会向 `HWnd` 指定的窗口发送 `TTN_NEEDTEXT` 消息以获得将要显示的字符串.提示文本的动态更新方法:你能在每次提示窗口被显示时改变工具提示文本。
- 如果你在这个字段中指定了一字符串资源，当工具提示控件需要在提示窗口中显示提示文本时，它就会在由 `hinst` 成员指定的实例模块的字符串表中查找字符串。工具提示控件通过检查这个字段的高字节来识别一个字符串资源标识符。因为字符串资源标识是一个 16 位的值，这个字段的高字节将永远为 0 。如果你计划将程序移植到其它语言编写的程序中，这种方法是非常有用的。因为字符串资源定义在资源脚本中，你并不需要修改源代码。 你仅需要修改字符串资源表，工具提示文本就会改变而不用担心会在你的程序中引入 `BUG`.
- 如果在字段中的值不是 `LPSTR_TEXTCALLBACK` 而且高字节也不是 0 ，工具提示控件解释这个值为文本字串的指针，所指向的文本子串将被用作工具提示文本。这种方法是最容易被使用的，但也是最不灵活的。

总之，在将 `TOOLINFO` 结构传递给工具提示控件之前你应该填充好它。这个结构描述你期望得到的工具的特点。

## Register the tool with the tooltip control

After you fill the `TOOLINFO` structure, you must submit it to tooltip control. A tooltip control can service many tools so it is usually unnecessary to create more than one tooltip control for a window. To register a tool with a tooltip control, you send the `TTM_ADDTOOL` message to the tooltip control. The `wParam` is not used and the `lParam` must contain the address of the `TOOLINFO` structure you want to register.

在你填充好 `TOOLINFO` 结构之后，你必须传递它给工具提示控件。一个工具提示控件能服务于多个工具，所以通常你没必要为一个窗口创建多于一个的工具提示控件。为了向工具提示控件注册一个工具，你可以发送 `TTM_ADDTOOL` 消息给工具提示控件。`wParam` 参数不被使用而 `Lparam` 必须包含你想注册的 `TOOLINFO` 结构的地址。

```
.data?  

ti TOOLINFO <>  

.......  

.code  

.......  

<fill the TOOLINFO structure>  

.......  

invoke SendMessage, hwndTooltip, TTM_ADDTOOL, NULL, addr ti  
```

`SendMessage` for this message will return `TRUE` if the tool is successfully registered with the tooltip control, `FALSE` otherwise.

如果工具在工具控件中被成功注册，`SendMessage` 将返回 `TRUE`，否则返回 `FALSE`。

You can unregister the tool by sending `TTM_DELTOOL` message to the tooltip control.

通过发送 `TTM_DELTOOL` 消息给工具提示控件，你能取消工具的注册。

## Relaying Mouse Messages to the Tooltip Control

转发鼠标消息给工具提示控件。



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



```asm
mov eax,[esi].bottom
sub eax,[esi].top
shr eax,1
mov [edi].rect.bottom,eax

.elseif id==1
    mov eax,[esi].right
    sub eax,[esi].left
    shr eax,1
    inc eax
    mov [edi].rect.left,eax
    mov [edi].rect.top,0
    mov eax,[esi].right
    sub eax,[esi].left
    mov [edi].rect.right,eax
    mov eax,[esi].bottom
    sub eax,[esi].top
    mov [edi].rect.bottom,eax

.elseif id==2
    mov [edi].rect.left,0
    mov eax,[esi].bottom
    sub eax,[esi].top
    shr eax,1
    inc eax
    mov [edi].rect.top,eax
    mov eax,[esi].right
    sub eax,[esi].left
    shr eax,1
    mov [edi].rect.right,eax
    mov eax,[esi].bottom
    sub eax,[esi].top
    mov [edi].rect.bottom,eax

.else
    mov eax,[esi].right
    sub eax,[esi].left
    shr eax,1
    inc eax
    mov [edi].rect.left,eax
    mov eax,[esi].bottom
    sub eax,[esi].top
    shr eax,1
    inc eax
    mov [edi].rect.top,eax
    mov eax,[esi].right
    sub eax,[esi].left
    mov [edi].rect.right,eax
    mov eax,[esi].bottom
    sub eax,[esi].top
    mov [edi].rect.bottom,eax

.endif
    push lpText
    pop [edi].lpszText
    invoke SendMessage,hwndTool,TTM_ADDTOOL,NULL,lpti
    assume edi:nothing
    assume esi:nothing
    ret
SetDlgToolArea endp

end start
```

### 分析

主对话框被创建后，我们用`CreateWindowEx`函数创建工具提示控件。

```asm
invoke InitCommonControls
invoke CreateWindowEx,NULL,ADDR ToolTipsClassName,NULL, \
      TTS_ALWAYS TIP,CW_USEDEFAULT,\
      CW_USEDEFAULT,CW_USEDEFAULT,CW_USEDEFAULT,NULL,NULL,\
      hInstance,NULL
mov hwndTool,eax
```

在这之后，我们继续为每一个对话框的角落定义四个工具。

```asm
   mov id,0        ; used as the tool ID
    mov ti.cbSize,sizeof TOOLINFO
    mov ti.uFlags,TTF_SUBCLASS    ; tell the tooltip control to subclass the dialog window. 告诉工具提示控件子类化对话框窗口。
    push hDlg
    pop ti.hWnd    ; handle to the window that contains the tool 包含工具的窗口句柄。
    invoke GetWindowRect,hDlg,addr rect    ; obtain the dimension of the client area 获取客户区的尺寸大小。
    invoke SetDlgToolArea,hDlg,addr ti,addr MainDialogText1,id,addr rect
```

我们初始化`TOOLINFO`结构体的成员。注意，我们想把客户区划分为四个工具，所以我们需要知道客户区的大小。这就是我们调用`GetWindowRect`的原因。我们不想自己转发鼠标消息给工具提示控件，所以我们指定`TTF_SUBCLASS`标志。

`SetDlgToolArea`是一个计算每个工具的边界矩形并向工具提示控件注册工具的函数。我不详细解释计算过程，只需说明它将客户区划分为四个大小相同的区域。然后它向工具提示控件发送`TTM_ADDTOOL`消息，并在`lParam`参数中传递`TOOLINFO`结构的地址。

```asm
    invoke SendMessage,hwndTool,TTM_ADDTOOL,NULL,lpti
```

在注册完所有4个工具后，我们可以继续处理对话框上的按钮。我们可以按它们的`ID`逐个处理每个按钮，但这很繁琐。作为替代方案，我们将使用`EnumChildWindows` API 调用来枚举对话框上的所有控件，然后将它们注册到工具提示控件中。

#### `EnumChildWindows` 语法

`EnumChildWindows` 语法如下：

```asm
EnumChildWindows proto hWnd:DWORD, lpEnumFunc:DWORD, lParam:DWORD
```

`hWnd`是父窗口的句柄。`lpEnumFunc`是对每个被枚举的控件调用的`EnumChildProc`函数的地址。`lParam`是传递给`EnumChildProc`函数的应用程序自定义值。`EnumChildProc`函数的定义如下：

```asm
EnumChildProc proto hwndChild:DWORD, lParam:DWORD
```

`hwndChild`是被`EnumChildWindows`枚举的控件的句柄。`lParam`是与你传递给`EnumChildWindows`函数的`lParam`值相同的值。

在我们的例子中，我们像这样调用`EnumChildWindows`：

```asm
invoke EnumChildWindows,hDlg,addr EnumChild,addr ti
```

我们之所以将`TOOLINFO`结构的地址传递给`lParam`参数，是因为我们将在`EnumChild`函数中向工具提示控件注册每个子控件。如果不使用这种方法，我们需要将`ti`声明为全局变量，这可能会引入`BUG`。

当我们调用`EnumChildWindows`函数时，`Windows`会枚举我们对话框上的子控件，并为每个被枚举的控件调用一次`EnumChild`函数。因此，如果我们的对话框有两个控件，`EnumChild`将被调用两次。

`EnumChild`函数填充`TOOLINFO`结构的相关成员，然后将工具注册到工具提示控件中。

```asm
EnumChild proc uses edi hwndChild:DWORD,lParam:DWORD
    LOCAL buffer[256]:BYTE
    mov edi,lParam
    assume edi:ptr TOOLINFO
    push hwndChild
    pop [edi].uId    ; we use the whole client area of the control as the tool
    or [edi].uFlags,TTF_IDISHWND
    invoke GetWindowText,hwndChild,addr buffer,255
    lea eax,buffer    ; use the window text as the tooltip text
    mov [edi].lpszText,eax
    invoke SendMessage,hwndTool,TTM_ADDTOOL,NULL,edi
    assume edi:nothing
    ret
EnumChild endp
```

注意，在这种情况下，我们使用了一种不同类型的工具：覆盖窗口整个客户区的工具。因此，我们需要用包含工具的窗口句柄填充`uID`字段。此外，我们还必须在`uFlags`成员中指定`TTF_IDISHWND`标志。

_________________________________

本文源自 Iczelion 的汇编页面

风向改变 翻译于 2008-3-9
