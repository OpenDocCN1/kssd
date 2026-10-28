
# 如何在IDA中找到MFC程序的消息处理函数

比起用Win32SDK写的程序，要分析MFC应用程序要麻烦不少。在前者，只要找到注册窗口类的地方就知道其`WinProc`的位置。那里是程序的控制中心，只要顺藤摸瓜就可以找到你感兴趣的地方。对于用MFC写的程序，这一切都变得复杂起来了。这时，所有的消息都是通过一套复杂的机制来完成分发的。他们是通过分发数据表来找到最终函数地址的。详细请参阅MFC的源代码。

常见的消息分发数据是由以下的宏来生成的：

```
ON_WM_SIZE()
ON_NOTIFY(TCN_SELCHANGE, ID_TABBOARD, OnBoardSelchange)
ON_WM_LBUTTONDBLCLK()
ON_WM_LBUTTONDOWN()
ON_WM_RBUTTONDOWN()
ON_WM_TIMER()
ON_COMMAND(ID_REDRAW_ALL, OnRedrawAll)
```

这里简单说一下如何找到二类消息的处理函数。一类是`WM_XXX`型消息，如`WM_LBUTTONDOWN`，另一类是`WM_COMMAND`型消息。

## 第Ⅰ类 `WM_XXX` 消息

对于第一类，它的调用栈是：

`CMyView::OnLButtonDown`        <--最终目标

`CWnd::OnWndMsg`                <--找到这个函数就接近最终目标了

`CWnd::WindowProc`

`AxfCallWndProc`

`AxfWndProc`

`AxfWndProcBase`

以`WM_LBUTTONDOWN`为例

```c
#define ON_WM_LBUTTONDOWN() \
        { WM_LBUTTONDOWN, 0, 0, 0, AfxSig_vwp, \
                (AFX_PMSG)(AFX_PMSGW)(void (AFX_MSG_CALL CWnd::*)(UINT, CPoint))&OnLButtonDown },
```

`AfxSig_vwp = 0x31`

对于`VC6.0 Release` 版本，可搜索 `C0 24 F0 83 C0 2F 48  83 F8 30 0F 87 C6 02` 找到`CWnd::OnWndMsg`。

进入`CWnd::OnWndMsg`后，找到 `case 0x30` `(IDA` 中的`case 0x30`其实是 `case 0x31)`处的

```asm
call    ebx
```

将进入你真正感兴趣的地方！

这里必须用条件断点`Dword(ESP+0x0c) == 0x201`, `(注WM_LBUTTONDOWN == 0x201)` 否则这个断点总会遇到.

## 第Ⅱ类 `WM_COMMAND` 消息

找`WM_COMMAND`消息处理的地方

对于第二类，它的调用栈是：

`CMyDoc::OnCmdXXX`        <--最终目标

`_AxfDispatchCmdMsg`        <--找到这个函数就接近最终目标了

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

`ON_COMMAND`定义如下

```c
#define ON_COMMAND(id, memberFxn) \
        { WM_COMMAND, CN_COMMAND, (WORD)id, (WORD)id, AfxSig_vv, (AFX_PMSG)&memberFxn },
```

其中的`AfxSig_vv = 12`

搜索 `71 74 5C 48  48 74 53 83 E8 0A 74 46`，将找到`_AfxDispatchCmdMsg`函数，在`case 12` 的地方设断点并运行，当程序需要处理`OnCmdXXX`的时候，控制就会跑到这里，单步进入就可以了。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
