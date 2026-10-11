
# 如何在 IDA 中找到 MFC 程序的消息处理函数

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-703.htm`](https://www.pediy.com/kssd/pediy07/pediy7-703.htm)

比起用 Win32SDK 写的程序，要分析 MFC 应用程序要麻烦不少。在前者，只要找到注册窗口类的地方就知道其 WinProc 的位置。那里是程序的控制中心，只要顺藤摸瓜就可以找到你感兴趣的地方。对于用 MFC 写的程序，这一切都变得复杂起来了。这时，所有的消息都是通过一套复杂的机制来完成分发的。他们是通过分发数据表来找到最终函数地址的。详细请参阅 MFC 的源代码。

常见的消息分发数据是由以下的宏来生成的：

`ON_WM_SIZE()`

`ON_NOTIFY(TCN_SELCHANGE, ID_TABBOARD, OnBoardSelchange)`

`ON_WM_LBUTTONDBLCLK()`

`ON_WM_LBUTTONDOWN()`

`ON_WM_RBUTTONDOWN()`

`ON_WM_TIMER()`

`ON_COMMAND(ID_REDRAW_ALL, OnRedrawAll)`

这里简单说一下如何找到二类消息的处理函数。一类是 `WM_XXX` 型消息，如 `WM_LBUTTONDOWN`，另一类是 `WM_COMMAND` 型消息。

对于第一类，它的调用栈是：

`CMyView::OnLButtonDown` <--最终目标

`CWnd::OnWndMsg` <--找到这个函数就接近最终目标了

`CWnd::WindowProc`

`AxfCallWndProc`

`AxfWndProc`

`AxfWndProcBase`

以 `WM_LBUTTONDOWN` 为例

```
#define ON_WM_LBUTTONDOWN() \
  { WM_LBUTTONDOWN, 0, 0, 0, AfxSig_vwp, \
    (AFX_PMSG)(AFX_PMSGW)(void (AFX_MSG_CALL CWnd::*)(UINT, CPoint))&OnLButtonDown },
```

`AfxSig_vwp` = `0x31`

对于 VC6.0 Release 版本，可搜索 `C0 24 F0 83 C0 2F 48 83 F8 30 0F 87 C6 02` 找到 `CWnd::OnWndMsg`。

进入 `CWnd::OnWndMsg` 后，找到 `case 0x30` (IDA 中的 `case 0x30` 其实是 `case 0x31`) 处的

`call ebx`

将进入你真正感兴趣的地方！

这里必须用条件断点 `Dword(ESP+0x0c) == 0x201` (注 `WM_LBUTTONDOWN` == `0x201`) 否则这个断点总会遇到。

找 `WM_COMMAND` 消息处理的地方

对于第二类，它的调用栈是：

`CMyDoc::OnCmdXXX` <--最终目标

`_AxfDispatchCmdMsg` <--找到这个函数就接近最终目标了

`CCmdTarget::OnCmdMsg`

`CDocument::OnCmdMsg`

`CView::OnCmdMsg`

`CFrameWnd::OnCmdMsg`

`CWnd::OnCommand`

`CFrameWnd::OnCommand`

`CWnd::OnWndMsg`

`CWnd::WindowProc`

`AxfCallWndProc`

`AxfWndProc`

`AxfWndProcBase`

`ON_COMMAND` 定义如下

```
#define ON_COMMAND(id, memberFxn) \
  { WM_COMMAND, CN_COMMAND, (WORD)id, (WORD)id, AfxSig_vv, (AFX_PMSG)&memberFxn },
```

其中的 `AfxSig_vv` = `12`

搜索 `71 74 5C 48 48 74 53 83 E8 0A 74 46`，将找到 `_AxfDispatchCmdMsg` 函数，在 `case 12` 的地方设断点并运行，当程序需要处理 `OnCmdXXX` 的时候，控制就会跑到这里，单步进入就可以了。

不必用 `ida`，用 `ultraedit` 就行。方法如下

注意到 `mfc` 的消息列表有

```
PAFX_MSGMAP_ENTRY = ^AFX_MSGMAP_ENTRY;

AFX_MSGMAP_ENTRY = packed record

nMessage: dword; // windows 中消息代号，定义在 WinUser.h 中

nCode: dword; // 如果 nMessage 是 0x0111(WM_Command)，nCode 代表扩展消息,例如 WM_Exchange 等.亦定义在 WinUser.h 中

nID: dword; // 控件 ID, 与资源部分中控件的 ID 一致

nLastID: dword; // = nID

nSig: dword; // 含义不清楚

AFX_PMSG: dword; // 指向消息处理的具体代码的开始位置

end;
```

所以我们只要知道消息的类型，和发出消息的控件 `ID`，就可以搜索找到这个数组，那么 `AFX_PMSG` 就是你需要下断的地址

基于类似的方法，`vb`，`delphi` 都可以这样来找到事件的处理代码的起始位置。

我想，大部分现代编译器编译的程序也应该适用这种方法。

扩展开去，这种方法应该属于伟大的 dead listing 技术之一。如果有兴趣，我们能进一步解析 `mfc`，`delphi`，`vb` 等程序的结构，从而了解更多。《黑客反汇编揭密》那本书讲的基本就是 dead listing 技术（虽然有人认为那本书很不怎么的）。

`4dd466d0e1e0cc123afbe504da2462ca`
