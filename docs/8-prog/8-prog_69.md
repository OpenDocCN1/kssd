

# Win32 asm 第二十七课：工具提示控件

> 原文：[`www.pediy.com/kssd/pediy10/60886.html`](https://www.pediy.com/kssd/pediy10/60886.html)

Tutorial 27: Tooltip Control  
第二十七课：工具提示控件  
\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_  
We will learn about the tooltip control: What it is and how to create and use it.  
我们将学习工具提示控件的有关内容：什么是工具提示控件以及如何创建并使用它。  
 Download the example.   
Theory:  
A tooltip is a small rectangular window that is displayed when the mouse pointer hovers over some specific area. A tooltip window contains some text that the programmer wants to be displayed. In this regard, a tooltip servers the same role as the status window but it disappears when the user clicks or moves the mouse pointer away from the designated area. You'll probably be familiar with the tooltips that are associated with toolbar buttons. Those "tooltips" are conveniencies provided by the toolbar control. If you want tooltips for other windows/controls, you need to create your own tooltip control.   
工具提示控件是当鼠标指针在一些特定区域停留时被显示的一个小矩形窗口。一个工具提示窗口包含了一些程序员想显示的文本。在这一点上，工具提示服务程序和状态栏扮演着同样的角色，但是当用户的鼠标被点击或是将鼠标从指定区域移走时，工具提示窗口将消失不见。你可能很熟悉工具提示控件，它们和工具栏的按钮相关联。这些“提示”为工具栏控件提供了方便。如果你想在其它窗口／控件中使用工具提示，那么你需要创建你自己的工具提示控件。  
Now that you know what a tooltip is, let's go on to how we can create and use it. The steps are outlined below:   
现在，你知道了什么是工具提示控件，让我们学习如何创建和使用它。步骤略述如下：  
1.  Create a tooltip control with CreateWindowEx   
用 CreateWindowEx 创建工具提示控件。  
2.  Define a region that the tooltip control will monitor for mouse pointer movement.   
定义一个区域，工具提示控件将监控这个区域的鼠标移动事件。  
3.  Submit the region to the tooltip control   
传递这块区域给工具提示控件。  
4.  Relay mouse messages of the submitted region to the tooltip control (this step may occur earlier, depending on the method used to relay the messages)   
将已传递区域的鼠标消息转发给工具提示控件。（这步可能发生的更早，具体依赖于转发消息的方法。）  
We wll next examine each step in detail.  
我们将详细的分析每一步：   
Tooltip Creation　工具提示控件的创建。  
A tooltip control is a common control. As such, you need to call InitCommonControls somewhere in your source code so that MASM implicitly links your program to comctl32.dll. You create a tooltip control with CreateWindowEx. The typical scenario would be like this:   
工具提示控件是一个通用控件。同样的，你需要在源代码的某一处调用 InitCommonControls 函数，以便 MASM 将你的程序和 comctl32.dll 链接。 然后，你可以用 CreateWindowEx 函数创建工具提示控件。典型的方案如下：  
.data   
TooltipClassName db "Tooltips\_class32",0   
.code   
.....   
invoke InitCommonControls   
invoke CreateWindowEx, NULL, addr TooltipClassName, NULL, TIS\_ALWAYSTIP, CW\_USEDEFAULT, CW\_USEDEFAULT, CW\_USEDEFAULT, CW\_USEDEFAULT, NULL, NULL, hInstance, NULL  
Note the window style: TIS\_ALWAYSTIP. This style specifies that the tooltip will be shown when the mouse pointer is over the designated area regardless of the status of the window that contains the area. Put simply, if you use this flag, when the mouse pointer hovers over the area you register to the tooltip control, the tooltip window will appear even if the window under the mouse pointer is inactive.   
注意窗口的样式：TIS\_ALWAYSTIP。它指出了当鼠标指针在指定区域停留时，工具提示将被显示，而和包含这块区域的窗口的状态无关。简单的说，如果你用这个标志，当鼠标指针停留在你注册了的工具提示控件的那块区域上时，工具提示将显示，即使是鼠标指针下面的窗口并没被激活。  
You don't have to include WS\_POPUP and WS\_EX\_TOOLWINDOW styles in CreateWindowEx because the tooltip control's window procedure adds them automatically. You also don't need to specify the coordinate, the height and width of the tooltip window: the tooltip control will adjust them automatically to fit the tooltip text that will be displayed, thus we supply CW\_USEDEFAULT in all four parameters. The remaining parameters are not remarkable.   
你并不需要在 CreateWindowEx 函数包含 WS\_POPUP 和 WS\_EX\_TOOLWINDOW 样式，因为工具提示控件的窗口过程会自动的添加它们。你也并不需要指定工具提示窗口的高和宽的坐标：工具提示控件会自动根据它将显示的文本调整合适的窗口大小。因此，我们提供 CW\_USEDEFAULT 给这四个参数。剩下的参数是不值得注意的。  
Specifying the tool  指定工具  
The tooltip control is created but it's not shown immediately. We want the tooltip window to show up when the mouse pointer hovers over some area. Now is the time to specify that area. We call such area "tool". A tool is a rectangular area on the client area of a window which the tooltip control will monitor for mouse pointer. If the mouse pointer hovers over the tool, the tooltip window will appear.  
工具提示控件被创建，但是它并不会立即显示。我们希望的是，当鼠标指针在一些区域上停留时，工具提示窗口在露面。现在是时候指定这个区域了。我们叫这样的区域为“工具”。工具是在一个窗口客户区上的一矩形区域，工具提示控件将监视这个窗口的鼠标指针。如果鼠标指针停留在工具上，工具提示窗口将显示。  
 The rectangular area can cover the whole client area or only a part of it. So we can divided tool into two types: one that is implemented as a window and another that is implemented as a rectangular area in the client area of some window. Both has their uses. The tool that covers the whole client area of a window is most frequently used with controls such as buttons, edit controls and so on.  
 You don't need to specify the coordinate and the dimensions of the tool: it's assumed to be the whole client area of the window. The tool that is implemented as a rectangular area on the client area is useful when you want to divide the client area of a window into several regions without using child windows. With this type of tool, you need to specify the coordinate of the upper left corner and the width and height of the tool.   
这个矩形区域可以覆盖整个客户区或者仅是客户区的一部分。所以我们能把工具分成两种类型：一种是作为整个窗口的工具，另一种是某窗口客户区中的一部分。两种各有所用。覆盖整个窗口客户区的工具常常被用在诸如按钮，编辑框等等一系列的控件上。你并不需要指定工具的大小和坐标：它被假定为某个窗口的整个客户区。覆盖窗口客户区一部分的"工具"在你想把窗口客户区分成几个部分但又不想使用子窗口时特别有用。用这种类型的工具，你需要指定“工具”的左上角坐标和“工具”的高度和宽度。  
You specify the tool with the TOOLINFO structure which has the following definition:   
你能用 TOOLINFO 结构体来指定“工具“，它的定义如下：  
TOOLINFO STRUCT   
  cbSize             DWORD      ?   
  uFlags             DWORD      ?   
  hWnd               DWORD      ?   
  uId                DWORD      ?   
  rect               RECT      <>   
  hInst              DWORD      ?   
  lpszText           DWORD      ?   
  lParam             LPARAM     ?   
TOOLINFO ENDS   
Field Name  
字段名  Explanation  
说明　  
cbSize  The size of the TOOLINFO structure. You MUST fill this member. Windows will not flag error if this field is not filled properly but you will receive strange, unpredictable results.  
TOOLINFO 结构的大小。你必须填充这个成员。如果填入不合适的值，Windows 并不会报错，但是你将得到奇怪的，不可预知的结果。  
uFlags  The bit flags that specifies the characteristics of the tool. This value can be a combination of the following flags:   
位标志指定了工具的特征。这个值能被下面的标志组合。  
  TTF\_IDISHWND  "ID is hWnd". If you specify this flag, it means you want to use a tool that covers the whole client area of a window (the first type of tool above). If you use this flag, you must fill the uId member of this structure with the handle of the window you want to use. If you don't specify this flag, it means you want to use the second type of tool, the one that is implemented as the rectangular area on the client window. In that case, you need to fill the rect member with the dimension of the rectangle.   
TTF\_IDISHWND  “ID is Hwnd “ 如果你指定了这标志，它意味着你想使用覆盖某个窗口整个客户区的“工具“（上面的第一种工具）。如果你使用这个标志，你必须用你想使用的窗口句柄填充这个结构的 uId 成员。如果你不指定这个标志，它意味你想使用第二种工具，即作为某个窗口客户区一部分的工具。在这种情况下，你需要填充 Rect 成员作为这个矩形的大小。  
  TTF\_CENTERTIP  Normally the tooltip window will appear to the right and below the mouse pointer. If you specify this flag, the tooltip window will always appear directly below the tool and is centered regardless of the position of the mouse pointer.   
  TTF\_CENIERTIP 通常工具提示窗口将显示在鼠标指针的右下方。如果你指定这个标志，不论鼠标指针的位置如何，工具提示窗口将立即显示在鼠标的中下方。   
  TTF\_RTLREADING  You can forget about this flag if your program is not designed specifically for Arabic or Hebrew systems. This flag displays the tooltip text with right-to-left reading order. Doesn't work under other systems.   
  TTF\_RTLREADING  如果你的程序不是特别为阿拉伯人或是希伯来人的系统设计的，你能忽略这个标志。这个标志显示阅读顺序为从右到左的文本提示。在其他系统中无效。  
  TTF\_SUBCLASS  If you use this flag, it means you tell the tooltip control to subclass the window that the tool is on so that the tooltip control can intercept mouse messages that are sent to the window. This flag is very handy. If you don't use this flag, you have to do more work to relay the mouse messages to the tooltip control.  
  TTF\_SUBCLASS 如果你使用这个标志，它意味着你告诉工具提示控件子类化工具所在的窗口，以便让工具提示控件能够窃听发送给窗口的鼠标消息。这个标志是非常方便的。如果你不使用这个标志，你必须做更多的工作来将鼠标消息转发给工具提示控件。  
hWnd  Handle to the window that contains the tool. If you specify TTF\_IDISHWND flag, this field is ignored since Windows will use the value in uId member as the window handle. You need to fill this field if:   
包含工具的窗口句柄。如果你指定 TTF\_IDISHWND 标志，这个标志你可以忽略它，因为 windows 将使用在 uId 成员中的值作为窗口句柄。你需要填充这个字段，如果：   
  You don't use TTF\_IDISHWND flag (in other words, you use a rectangular tool)   
   你没有使用 TTF\_IDISHWND 标志。（换句话说，你使用矩形“工具“）  
  You specify the value LPSTR\_TEXTCALLBACK in lpszText member. This value tells the tooltip control that, when it needs to display the tooltip window, it must ask the window that contains the tool for the text to be displayed. This is a kind of dynamic realtime tooltip text update. If you want to change your tooltip text dynamically, you should specify LPSTR\_TEXTCALLBACK value in lpszText member. The tooltip control will send TTN\_NEEDTEXT notification message to the window identified by the handle in hWnd field.   
   你在 LpszText 成员中指定了 LPSTR\_TEXTCALLBACK 值。这个值告诉工具提示控件，当需要显示提示窗口时,必须向包含"工具"的窗口查询应该显示什么。这是一种动态快速的提示文本更新。如果你想动态的改变你的提示文本，你应该在 lpszText 中放入 LPSTR\_TEXTCALLBACK。控件会将发送 TTN\_NEEDTEXT 通知消息给由 Hwnd 字段中的窗口句柄所指定的窗口。  
uId  The value in this field can have two meanings, depending on whether the uFlags member contains the flag TTF\_IDISHWND.   
在这个字段中的值可以有可以有两种含义，依赖于 uFlags 成员是否包含 TTF\_IDISHWND。  
  Application-defined tool ID if the TTF\_IDISHWND flag is not specified. Since this means you use a tool which covers only a part of the client area, it's logical that you can have many such tools on the same client area (without overlap). The tooltip control needs a way to differentiate between them. In this case, the window handle in hWnd member is not enough since all tools are on the same window. The application-defined IDs are thus necessary. The IDs can be any value so long as they are unique among themselves.   
    如果 TTF\_IDISHWND 标志没有被指定，就代表应用程序定义的工具 ID 值。因为这意味这你使用的是仅覆盖客户区一部分的工具，逻辑上来说，在同一个客户区中这种工具可以有很多个（但不存在交迭）。工具提示控件需要一种方式来区分它们。既然这样，在 hwnd 成员中的一个窗口句柄是不足够的，因为所有的工具都在同一个窗口中。因此，应用程序必须定义 ID 值来区分它们，这些 ID 可以是任何值，但要保证它们之间的唯一性。  
  The handle to the window whose whole client area is used as the tool if the TTF\_IDISHWND flag is specified. You may wonder why this field is used to store the window handle instead of the hWnd field above. The answer is: the hWnd member may already be filled if the value LPSTR\_TEXTCALLBACK is specified in the lpszText member and the window that is responsible for supplying the tooltip text and the window that contains the tool may NOT be the same ( You can design your program so that a single window can serve both roles but this is too restrictive. In this case, Microsoft gives you more freedom. Cheers.)  
   如果 TTF\_IDISHWND 标志被指定, 就表示整个客户区都作为焦点域的窗口句柄.你可能会奇怪为什么要用这个域来代替上面 hwnd 域来储存窗口句柄.答案是：如果在 lpszText 成员的值被 LPSTR\_TEXTCALLBACK 指定, 那么这个 hwnd 可能已经被填充.或者为提示文本负责的窗口可能包含的并不是同一工具.( 你可以设计一个能充当两种角色的窗口程序,但这会受到很多限制,在这点上,微软给了你更多的自由,举杯欢呼吧!)  
rect  A RECT structure that specifies the dimension of the tool. This structure defines a rectangle relative to the upper left corner of the client area of the window specified by the hWnd member. In short, you must fill this structure if you want to specify a tool that covers only a part of the client area. The tooltip control will ignore this field if you specify TTF\_IDISHWND flag (you choose to use a tool that covers the whole client area)  
RECT 结构指定工具的尺寸大小.这个结构相对于由 hwnd 成员指定的窗口客户区的左上角坐标来定义一个矩形.较而言之,如果你想指定一个仅覆盖客户区一部分的工具,你必须填充这个结构.如果你指定 TTF\_IDISHWND 标志，工具提示控件将忽视这个字段中的值。（即，你选择使用一个覆盖整个客户区的工具）  
hInst  The handle of the instance that contains the string resource that will be used as the tooltip text if the value in the lpszText member specifies the string resource identifier. This may sound confusing. Read the explanation of the lpszText member first and you will understand what this field is used for. The tooltip control ignores this field if the lpszText field doesn't contain a string resource identifier.  
包含字符串资源的实例句柄，如果在 lpszText 成员中指定了字符串资源标识符，那么这个字符串资源将被用作工具提示文本。听起来可能有点费劲。阅读一下 lpszText 的说明就可以明白这个字段是干什么用的了.若 lpszText 不包含字符串资源标识,工具控件会忽略这个字段。.  
lpszText  This field can have several values:   
这个字段可以有以下几个值：  
  If you specify the value LPSTR\_TEXTCALLBACK in this field, the tooltip control will send TTN\_NEEDTEXT notification message to the window identified by the handle in hWnd field for the text string to be displayed in the tooltip window. This is the most dynamic method of tooltip text update: you can change the tooltip text each time the tooltip window is displayed.   
    如果指定为 LPSTR\_TEXTCALLBACK, 工具提示控件就会向 HWnd 指定的窗口发送 TTN\_NEEDTEXT 消息以获得将要显示的字符串.提示文本的动态更新方法:你能在每次提示窗口被显示时改变工具提示文本。，  
 If you specify a string resource identifier in this field, when the tooltip control needs to display the tooltip text in the tooltip window, it searches for the string in the string table of the instance specified by hInst member. The tooltip control identifies a string resource identifier by checking the high word of this field. Since a string resource identifier is a 16-bit value, the high word of this field will always be zero. This method is useful if you plan to port your program to other languages. Since the string resource is defined in a resource script, you don't need to modify the source code.You only have to modify the string table and the tooltip texts will change without the risk of introducing bugs into your program.   
如果你在这个字段中指定了一字符串资源，当工具提示控件需要在提示窗口中显示提示文本时，它就会在由 hinst 成员指定的实例模块的字符串表中查找字符串。工具提示控件通过检查这个字段的高字节来识别一个字符串资源标识符。因为字符串资源标识是一个 16 位的值，这个字段的高字节将永远为 0 。如果你计划将程序移植到其它语言编写的程序中，这种方法是非常有用的。因为字符串资源定义在资源脚本中，你并不需要修改源代码。 你仅需要修改字符串资源表，工具提示文本就会改变而不用担心会在你的程序中引入 BUG.  
  If the value in this field is not LPSTR\_TEXTCALLBACK and the high word is not zero, the tooltip control interprets the value as the pointer to a text string that will be used as the tooltip text. This method is the easiest to use but the least flexible.   
   如果在字段中的值不是 LPSTR\_TEXTCALLBACK 而且高字节也不是 0 ，工具提示控件解释这个值为文本字串的指针，所指向的文本子串将被用作工具提示文本。这种方法是最容易被使用的，但也是最不灵活的。  
To recapitulate, you need to fill the TOOLINFO structure prior to submitting it to the tooltip control. This structure describes the characteristics of the tool you desire.   
总之，在将 TOOLINFO 结构传递给工具提示控件之前你应该填充号它。这个结构描述你期望得到的工具的特点。  
Register the tool with the tooltip control  
After you fill the TOOLINFO structure, you must submit it to tooltip control. A tooltip control can service many tools so it is usually unnecessary to create more than one tooltip control for a window. To register a tool with a tooltip control, you send the TTM\_ADDTOOL message to the tooltip control. The wParam is not used and the lParam must contain the address of the TOOLINFO structure you want to register.   
在你填充好 TOOLINFO 结构之后，你必须传递它给工具提示控件。一个工具提示控件能服务于多个工具，所以通常你没必要为一个窗口创建多于一个的工具提示控件。为了向工具提示控件注册一个工具，你可以发送 TTM\_ADDTOOL 消息给工具提示控件。wParam 参数不被使用而 Lparam 必须包含你想注册的 TOOLINFO 结构的地址。  
.data?   
ti TOOLINFO <>   
.......   
.code   
.......   
<fill the TOOLINFO structure>   
.......   
invoke SendMessage, hwndTooltip, TTM\_ADDTOOL, NULL, addr ti  
SendMessage for this message will return TRUE if the tool is successfully registered with the tooltip control, FALSE otherwise.   
You can unregister the tool by sending TTM\_DELTOOL message to the tooltip control.   
如果工具在工具控件中被成功注册，SendMessage 将返回 TRUE，否则返回 FALSE。  
通过发送 TTM\_DELTOOL 消息给工具提示控件，你能取消工具的注册。  
Relaying Mouse Messages to the Tooltip Control  
转发鼠标消息给工具提示控件。  
When the above step is completed, the tooltip control knows which area it should monitor for mouse messages and what text it should display in the tooltip window. The only thing it lacks is the \*trigger\* for that action. Think about it: the area specified by the tool is on the client area of the other window. How can the tooltip control intercept the mouse messages destined for that window? It needs to do so in order that it can measure the amount of time the mouse pointer hovers over a point in the tool so that when the specified amount of time elapses, the tooltip control shows the tooltip window. There are two methods of accomplishing this goal, one that requires the cooperation of the window that contains the tool and the other without the cooperation on the part of the window.   
当完成上面的步骤后，工具提示控件应该知道了在那一块区域监听鼠标消息以及在工具提示窗口显示什么样的文本。唯一缺少的就是“触发”机制。想想看：被工具指定的区域如果在其它窗口的客户区上。工具提示控件又怎么截获发送给窗口的鼠标消息呢？它需要去做的就是测量鼠标指针在工具上停留的时间值，以至于当指定的时间值过去时，工具控件显示工具提示窗口。完成这个目标有两种方法：一种需要包含工具的窗口合作而另一种则不需要。  
  The window that contains the tool must relay the mouse messages to the tooltip control by sending TTM\_RELAYEVENT messages to the control. The lParam of this message must contain the address of a MSG structure that specifies the message to be relayed to the tooltip control. A tooltip control processes only the following mouse messages:   
包含工具的的窗口必须通过发送 TTM\_RELAYEVENT 消息给控件以便将鼠标消息转发给工具提示控件。消息的 Lparam 参数必须包含指定发送给工具提示控件的消息结构体地址。工具提示控件只处理下面的鼠标消息：  
o  WM\_LBUTTONDOWN   
o  WM\_MOUSEMOVE   
o  WM\_LBUTTONUP   
o  WM\_RBUTTONDOWN   
o  WM\_MBUTTONDOWN   
o  WM\_RBUTTONUP   
o  WM\_MBUTTONUP   
All other messages are ignored. Thus in the window procedure of the window that contains the tool, there must be a switch that does something like this:   
所有其它的消息都被忽略。因此在包含工具的窗口的窗口过程中，必须有如下的开关语句：   
WndProc proc hWnd:DWORD, uMsg:DWORD, wParam:DWORD, lParam:DWORD   
.......   
    if uMsg==WM\_CREATE   
        .............   
    elseif uMsg==WM\_LBUTTONDOWN || uMsg==WM\_MOUSEMOVE || uMsg==WM\_LBUTTONUP || uMsg==WM\_RBUTTONDOWN || uMsg==WM\_MBUTTONDOWN || uMsg==WM\_RBUTTONUP || uMsg==WM\_MBUTTONUP   
        invoke SendMessage, hwndTooltip, TTM\_RELAYEVENT, NULL, addr msg   
        ..........   
  You can specify TTF\_SUBCLASS flag in the uFlags member of the TOOLINFO structure. This flag tells the tooltip control to subclass the window that contains the tool so it can intercept the mouse messages without the cooperation of the window. This method is easier to use since it doesn't require more coding than specifying TTF\_SUBCLASS flag and the tooltip control handles all the message interception itself.   
你能在 TOOLINFO 结构的 uFlags 成员中指定 TTF\_SUBCLASS 标志。这个标志告诉工具提示控件子类化包含工具的窗口，以便在没有窗口的协助下它也能截获鼠标消息。这个方法很容易使用，因为除了指定 TTF\_SUBCLASS 标志和工具提示控件自己处理截获的所有消息外，它并不需要更多的编码。  
That's it. At this step, your tooltip control is fully functional. There are several useful tooltip-related messages you should know about.   
就是这些了，到这一步，你的工具提示控件功能已经很完整了。这里还有几个和工具提示有关的消息你应该知道。  
  TTM\_ACTIVATE.  If you want to disable/enable the tooltip control dynamically, this message is for you. If the wParam value is TRUE, the tooltip control is enabled. If the wParam value is FALSE, the tooltip control is disabled. A tooltip control is enabled when it first created so you don't need to send this message to activate it.   
TTM\_ACTIVATE 如果你想动态的停用和启用工具提示控件，这个消息是为你准备的。如果 wParam 的值是 TRUE，那么工具提示控件被启用。如果 wParam 的值是 FALSE，工具提示控件被停用。一个工具提示控件在第一次被创建时是可用的，所以你并不需要发送这个消息去激活它。。  
  TTM\_GETTOOLINFO and TTM\_SETTOOLINFO. If you want to obtain/change the values in the TOOLINFO structure after it was submitted to the tooltip control, use these messages. You need to specify the tool you need to change with the correct uId and hWnd values. If you only want to change the rect member, use TTM\_NEWTOOLRECT message. If you only want to change the tooltip text, use TTM\_UPDATETIPTEXT.   
TTM\_GETTOOLINFO 和 TTM\_SETTOOLINFO . 如果你想在将 TOOLINFO 结构体传送给工具提示控件后，获取或是改变结构体中的值，用这两个消息。你需要用正确的 uId 和 hWnd 的值来指定你想改变的工具。如果你仅想改变 rect 成员，用 TTM\_NEWTOOLRECT 消息。如果你仅想改变工具提示文本，用 TTM\_UPDATETIPTEXT 消息。  
  TTM\_SETDELAYTIME. With this message, you can specify the time delay the tooltip control uses when it's displaying the tooltip text and much more.   
TTM\_SETDELAYTIME 当工具提示控件正显示提示文本或做其它工作时，你可以用这个消息指定它的时间延迟。  
Example:  
The following example is a simple dialog box with two buttons. The client area of the dialog box is divided into 4 areas: upper left, upper right, lower left and lower right. Each area is specified as a tool with its own tooltip text. The two buttons also has their own tooltip texts.   
下面的例子是一个简单的对话框和两个按钮。对话框的客户区被分成四个区域：左上，右上，左下，右下。每一块区域都被指定为有自己提示文本的工具。两个按钮也有它们自己的提示文本。  
.386   
.model flat,stdcall   
option casemap:none   
include \\masm32\\include\\windows.inc   
include \\masm32\\include\\kernel32.inc   
include \\masm32\\include\\user32.inc   
include \\masm32\\include\\comctl32.inc   
includelib \\masm32\\lib\\comctl32.lib   
includelib \\masm32\\lib\\user32.lib   
includelib \\masm32\\lib\\kernel32.lib   
DlgProc proto :DWORD,:DWORD,:DWORD,:DWORD   
EnumChild proto :DWORD,:DWORD   
SetDlgToolArea proto :DWORD,:DWORD,:DWORD,:DWORD,:DWORD   
.const   
IDD\_MAINDIALOG equ 101   
.data   
ToolTipsClassName db "Tooltips\_class32",0   
MainDialogText1 db "This is the upper left area of the dialog",0   
MainDialogText2 db "This is the upper right area of the dialog",0   
MainDialogText3 db "This is the lower left area of the dialog",0   
MainDialogText4 db "This is the lower right area of the dialog",0   
.data?   
hwndTool dd ?   
hInstance dd ?   
.code   
start:   
    invoke GetModuleHandle,NULL   
    mov hInstance,eax   
    invoke DialogBoxParam,hInstance,IDD\_MAINDIALOG,NULL,addr DlgProc,NULL   
    invoke ExitProcess,eax DlgProc proc hDlg:DWORD,uMsg:DWORD,wParam:DWORD,lParam:DWORD   
    LOCAL ti:TOOLINFO   
    LOCAL id:DWORD   
    LOCAL rect:RECT   
    .if uMsg==WM\_INITDIALOG   
        invoke InitCommonControls   
        invoke CreateWindowEx,NULL,ADDR ToolTipsClassName,NULL,\\   
            TTS\_ALWAYSTIP,CW\_USEDEFAULT,\\   
            CW\_USEDEFAULT,CW\_USEDEFAULT,CW\_USEDEFAULT,NULL,NULL,\\   
            hInstance,NULL   
        mov hwndTool,eax   
        mov id,0   
        mov ti.cbSize,sizeof TOOLINFO   
        mov ti.uFlags,TTF\_SUBCLASS   
        push hDlg   
        pop ti.hWnd   
        invoke GetWindowRect,hDlg,addr rect   
        invoke SetDlgToolArea,hDlg,addr ti,addr MainDialogText1,id,addr rect   
        inc id   
        invoke SetDlgToolArea,hDlg,addr ti,addr MainDialogText2,id,addr rect   
        inc id   
        invoke SetDlgToolArea,hDlg,addr ti,addr MainDialogText3,id,addr rect   
        inc id   
        invoke SetDlgToolArea,hDlg,addr ti,addr MainDialogText4,id,addr rect   
        invoke EnumChildWindows,hDlg,addr EnumChild,addr ti   
    .elseif uMsg==WM\_CLOSE   
        invoke EndDialog,hDlg,NULL   
    .else   
        mov eax,FALSE   
        ret   
    .endif   
    mov eax,TRUE   
    ret   
DlgProc endp EnumChild proc uses edi hwndChild:DWORD,lParam:DWORD   
    LOCAL buffer\[256\]:BYTE   
    mov edi,lParam   
    assume edi:ptr TOOLINFO   
    push hwndChild   
    pop \[edi\].uId   
    or \[edi\].uFlags,TTF\_IDISHWND   
    invoke GetWindowText,hwndChild,addr buffer,255   
    lea eax,buffer   
    mov \[edi\].lpszText,eax   
    invoke SendMessage,hwndTool,TTM\_ADDTOOL,NULL,edi   
    assume edi:nothing   
    ret   
EnumChild endp SetDlgToolArea proc uses edi esi hDlg:DWORD,lpti:DWORD,lpText:DWORD,id:DWORD,lprect:DWORD   
    mov edi,lpti   
    mov esi,lprect   
    assume esi:ptr RECT   
    assume edi:ptr TOOLINFO   
    .if id==0   
        mov \[edi\].rect.left,0   
        mov \[edi\].rect.top,0   
        mov eax,\[esi\].right   
        sub eax,\[esi\].left   
        shr eax,1   
        mov \[edi\].rect.right,eax   
        mov eax,\[esi\].bottom   
        sub eax,\[esi\].top   
        shr eax,1   
        mov \[edi\].rect.bottom,eax   
    .elseif id==1   
        mov eax,\[esi\].right   
        sub eax,\[esi\].left   
        shr eax,1   
        inc eax   
        mov \[edi\].rect.left,eax   
        mov \[edi\].rect.top,0   
        mov eax,\[esi\].right   
        sub eax,\[esi\].left   
        mov \[edi\].rect.right,eax   
        mov eax,\[esi\].bottom   
        sub eax,\[esi\].top   
        mov \[edi\].rect.bottom,eax   
    .elseif id==2   
        mov \[edi\].rect.left,0   
        mov eax,\[esi\].bottom   
        sub eax,\[esi\].top   
        shr eax,1   
        inc eax   
        mov \[edi\].rect.top,eax   
        mov eax,\[esi\].right   
        sub eax,\[esi\].left   
        shr eax,1   
        mov \[edi\].rect.right,eax   
        mov eax,\[esi\].bottom   
        sub eax,\[esi\].top   
        mov \[edi\].rect.bottom,eax   
    .else   
        mov eax,\[esi\].right   
        sub eax,\[esi\].left   
        shr eax,1   
        inc eax   
        mov \[edi\].rect.left,eax   
        mov eax,\[esi\].bottom   
        sub eax,\[esi\].top   
        shr eax,1   
        inc eax   
        mov \[edi\].rect.top,eax   
        mov eax,\[esi\].right   
        sub eax,\[esi\].left   
        mov \[edi\].rect.right,eax   
        mov eax,\[esi\].bottom   
        sub eax,\[esi\].top   
        mov \[edi\].rect.bottom,eax   
    .endif   
    push lpText   
    pop \[edi\].lpszText   
    invoke SendMessage,hwndTool,TTM\_ADDTOOL,NULL,lpti   
    assume edi:nothing   
    assume esi:nothing   
    ret   
SetDlgToolArea endp   
end start   
  
Analysis:  
分析：  
After the main dialog window is created, we create the tooltip control with CreateWindowEx.   
在主对话框被创建后，我们用 CreateWindowEx 函数创建工具提示控件。  
  
invoke InitCommonControls   
invoke CreateWindowEx,NULL,ADDR ToolTipsClassName,NULL,\\   
       TTS\_ALWAYSTIP,CW\_USEDEFAULT,\\   
       CW\_USEDEFAULT,CW\_USEDEFAULT,CW\_USEDEFAULT,NULL,NULL,\\   
       hInstance,NULL   
mov hwndTool,eax   
After that, we proceed to define four tools for each corner of the dialog box.    
在这之后，我们继续为每一个对话框的角落定义四个工具。  
  
   mov id,0        ; used as the tool ID   
    mov ti.cbSize,sizeof TOOLINFO   
    mov ti.uFlags,TTF\_SUBCLASS    ; tell the tooltip control to subclass the dialog window.  告诉工具提示控件子类化对话框窗口。  
    push hDlg   
    pop ti.hWnd    ; handle to the window that contains the tool   
包含工具的窗口句柄。  
    invoke GetWindowRect,hDlg,addr rect    ; obtain the dimension of the client area            获取客户区的尺寸大小。  
    invoke SetDlgToolArea,hDlg,addr ti,addr MainDialogText1,id,addr rect   
We initialize the members of TOOLINFO structure. Note that we want to divide the client area into 4 tools so we need to know the dimension of the client area. That's why we call GetWindowRect. We don't want to relay mouse messages to the tooltip control ourselves so we specify TIF\_SUBCLASS flag.   
我们初始化 TOOLINFO 结构体的成员。注意，我们想把客户区划分为四个工具，所以我们需要知道客户区的大小。这就是我们调用 GetWindowRect 的原因。我们不想自己转发鼠标消息给工具提示控件，所以我们指定 TIF\_SUBCLASS 标志。  
  
SetDlgToolArea is a function that calculates the bounding rectangle of each tool and registers the tool to the tooltip control. I won't go into gory detail on the calculation, suffice to say that it divides the client area into 4 areas with the same sizes. Then it sends TTM\_ADDTOOL message to the tooltip control, passing the address of the TOOLINFO structure in the lParam parameter.   
SetDlgToolArea 是计算每一个工具的矩形大小范围并向工具提示控件注册工具的函数。我并不去详细的解释计算过程，只使说明它将客户区划分为四块大小相同的区域。然后发送 TTM\_ADDTOOL 消息给工具提示控件，并传递 TOOLINFO 结构的地址给 Lparam 参数。  
    invoke SendMessage,hwndTool,TTM\_ADDTOOL,NULL,lpti   
After all 4 tools are registered, we can go on to the buttons on the dialog box. We can handle each button by its ID but this is tedious. Instead, we will use EnumChildWindows API call to enumerate all controls on the dialog box and then registers them to the tooltip control.   
  
在所有的 4 个工具都被注册后，我们继续看看对话框上的按钮。我们能通过它们的 ID 来处理每个按钮，但这是比较单调的。作为替代，我们用 EnumChildWindow API 函数来列举在对话框上的所有控件，然后注册它们给工具提示控件。  
EnumChildWindows has the following syntax:   
EnumChildWindows 句法如下：　  
EnumChildWindows proto hWnd:DWORD, lpEnumFunc:DWORD, lParam:DWORD  
hWnd is the handle to the parent window. lpEnumFunc is the address of the EnumChildProc function that will be called for each control enumerated. lParam is the application-defined value that will be passed to the EnumChildProc function. The EnumChildProc function has the following definition:   
hWnd 是父窗口的句柄。LpEnumFunc 是即将被每一个被列举的控件调用的 EnumChildProc 函数的地址。Lparam 是应用程序自定义的值，它将被传递给 EnumChildProc 函数，EnumChildProc 定义如下：　  
  
EnumChildProc proto hwndChild:DWORD, lParam:DWORD  
  
hwndChild is the handle to a control enumerated by EnumChildWindows. lParam is the same lParam value you pass to EnumChildWindows.   
In our example, we call EnumChildWindows like this:   
  
hwndChild 是被 EnumChildWindows 函数枚举的控件句柄。Lparam 的值和你传递给 EnumChildWindows 函数的 Lparam 的值相同。在我们的例子中，EnumChildWindows 像这样：  
  
invoke EnumChildWindows,hDlg,addr EnumChild,addr ti   
  
We pass the address of the TOOLINFO structure in the lParam parameter because we will register each child control to the tooltip control in the EnumChild function. If we don't use this method, we need to declare ti as a global variable which can introduce bugs.   
我们传递 TOOLINFO 结构体的地址给 Lparam 参数，因为我们将在 EnumChild 函数中向工具提示控件注册每一个子控件。如果不用这个方法，我们需要声明 TI 作为一个全局变量。但这可能会引人 BUG。  
  
When we call EnumChildWindows, Windows will enumerate the child controls on our dialog box and call the EnumChild function once for each control enumerated. Thus if our dialog box has two controls, EnumChild will be called twice.   
当我们调用 EnumChildWindows 函数时，widnows 将枚举在我们对话框上的所有子控件，并且为每一个子控件调用一次 EnumChild 函数。因此，如果我们的对话框有两个控件，EnumChild 将被调用两次。  
The EnumChild function fills the relevant members of the TOOLINFO structure and then registers the tool with the tooltip control.   
这个 EnumChild 函数填充 TOOlINFO 结构的相关成员并将它们注册为工具提示控件的工具。  
  
EnumChild proc uses edi hwndChild:DWORD,lParam:DWORD   
    LOCAL buffer\[256\]:BYTE   
    mov edi,lParam   
    assume edi:ptr TOOLINFO   
    push hwndChild   
    pop \[edi\].uId    ; we use the whole client area of the control as the tool   
    or \[edi\].uFlags,TTF\_IDISHWND   
    invoke GetWindowText,hwndChild,addr buffer,255   
    lea eax,buffer    ; use the window text as the tooltip text   
    mov \[edi\].lpszText,eax   
    invoke SendMessage,hwndTool,TTM\_ADDTOOL,NULL,edi   
    assume edi:nothing   
    ret   
EnumChild endp   
  
Note that in this case, we use a different type of tool: one that covers the whole client area of the window. We thus need to fill the uID field with the handle to the window that contains the tool. Also we must specify TTF\_IDISHWND flag in the uFlags member.   
注意：在这个例子中，我们用了一种不同类型的工具：覆盖窗口整个客户区的工具。因此，我们需要用包含工具的窗口句柄填充 uID 字段。同时，我们必须在 uFlag 成员中指定 TTF\_IDISHWND 标志。  
\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_  
This article come from Iczelion's asm page   
风向改变翻译于 2008-3-9