
# [原创]打造具有`EnableWindow`功能的SPYXX

SPYXX是Visual Studio附带的一个非常有用的工具，可以很方便的查看到窗口的句柄，类，进程，线程，窗口消息等信息。在很多时候，我们要对一些外部程序的控件进行控制，例如变灰，变为可用状态等等，通常可以使用SPYXX来获取到控件的句柄，然后自己写个程序调用`EnableWindow`等这些函数来让控件变为可用状态，但是窗口一重新出现，句柄就会改变，这个时候就会很麻烦，试想一下，如果在SPYXX上添加这样一些功能，那岂不是很方便吗？

下面我们就来自己实现这样一些功能。

在开始动手之前，我们有必要了解一下MFC的消息响应相关知识，或许，你很会使用Visual Studio为你的应用程序添加一个消息响应函数，但你并不知道VS在编译消息响应的时候做了什么事，虽然我们使用Class Wizard添加一段消息响应十分简单，但MFC在处理这段消息响应的时候是非常复杂的，例如，一个按钮被单击的时候，将会产生以下一连串的函数调用：

```
CWinThread::PumpMessage -> CWnd::PretranslateMessage -> CWnd::WWalkPreTranslateMessate -> CD1Dlg::PreTranslateMessage -> CDialog::PreTranslateMessage -> CWnd::PreTranslateInput  -> CWnd::IsDialogMessageA -> USER32内核 -> AfxWndProcBase -> AfxWndProc -> AfxCallWndProc -> CWnd::WindowProc -> CWnd::OnWndMsg -> CWnd::OnCommand -> CDialog::OnCmdMsg -> CCmdTarget::OnCmdMsg -> _AfxDispatchCmdMsg -> CD1Dlg::OnButton1()
```

当然，这很多调用都是对别的函数进行了一些简单的封装，在这里，我们要跟踪的2个比较重要的函数是`_AfxDispatchCmdMsg`和`CCmdTarget::OnCmdMsg`，`_AfxDispatchCmdMsg`的主要作用是调用我们自己编写的消息处理函数，而`CCmdTarget::OnCmdMsg`则是在当前的类以及父类里面查找消息的处理函数，如果找到处理函数，将调用`_AfxDispatchCmdMsg`来分发消息，如果没有找到，就返回，并在上级调用默认的消息处理函数。

以下是代码来自VS2005的MFC源代码：

```
BOOL CCmdTarget::OnCmdMsg(UINT nID, int nCode, void* pExtra,
        AFX_CMDHANDLERINFO* pHandlerInfo)
{
#ifndef _AFX_NO_OCC_SUPPORT
        // OLE control events are a special case
        if (nCode == CN_EVENT)
        {
                ENSURE(afxOccManager != NULL);
                return afxOccManager->OnEvent(this, nID, (AFX_EVENT*)pExtra, pHandlerInfo);
        }
#endif // !_AFX_NO_OCC_SUPPORT

        // determine the message number and code (packed into nCode)
        const AFX_MSGMAP* pMessageMap;
        const AFX_MSGMAP_ENTRY* lpEntry;
        UINT nMsg = 0;

#ifndef _AFX_NO_DOCOBJECT_SUPPORT
        if (nCode == CN_OLECOMMAND)
        {
                BOOL bResult = FALSE;
                const AFX_OLECMDMAP* pOleCommandMap;
                const AFX_OLECMDMAP_ENTRY* pEntry;
                ENSURE_ARG(pExtra != NULL);
                COleCmdUI* pUI = (COleCmdUI*) pExtra;
                const GUID* pguidCmdGroup = pUI->m_pguidCmdGroup;

#ifdef _AFXDLL
                for (pOleCommandMap = GetCommandMap(); pOleCommandMap->pfnGetBaseMap != NULL && !bResult;
                        pOleCommandMap = pOleCommandMap->pfnGetBaseMap())
#else
                for (pOleCommandMap = GetCommandMap(); pOleCommandMap != NULL && !bResult;
                        pOleCommandMap = pOleCommandMap->pBaseMap)
#endif
                {
                        for (pEntry = pOleCommandMap->lpEntries;
                                pEntry->cmdID != 0 && pEntry->nID != 0 && !bResult;
                                pEntry++)
                        {
                                if (nID == pEntry->cmdID &&
                                        IsEqualNULLGuid(pguidCmdGroup, pEntry->pguid))
                                {
                                        pUI->m_nID = pEntry->nID;
                                        bResult = TRUE;
                                }
                        }
                }
                return bResult;
        }
#endif

        if (nCode != CN_UPDATE_COMMAND_UI)
        {
                nMsg = HIWORD(nCode);
                nCode = LOWORD(nCode);
        }

        // for backward compatibility HIWORD(nCode)==0 is WM_COMMAND
        if (nMsg == 0)
                nMsg = WM_COMMAND;

        // look through message map to see if it applies to us
        for (pMessageMap = GetMessageMap(); pMessageMap->pfnGetBaseMap != NULL;
          pMessageMap = (*pMessageMap->pfnGetBaseMap)())
        {
                // Note: catches BEGIN_MESSAGE_MAP(CMyClass, CMyClass)!
                ASSERT(pMessageMap != (*pMessageMap->pfnGetBaseMap)());
                lpEntry = AfxFindMessageEntry(pMessageMap->lpEntries, nMsg, nCode, nID);
                if (lpEntry != NULL)
                {
                        // found it
#ifdef _DEBUG
                        if (nCode == CN_COMMAND)
                                TRACE(traceCmdRouting, 1, "SENDING command id 0x%04X to %hs target.\n", nID,
                                        GetRuntimeClass()->m_lpszClassName);
                        else if (nCode > CN_COMMAND)
                                TRACE(traceCmdRouting, 1, "SENDING control notification %d from control id 0x%04X to %hs window.\n",
                                        nCode, nID, GetRuntimeClass()->m_lpszClassName);
#endif //_DEBUG

                        return _AfxDispatchCmdMsg(this, nID, nCode,
                                lpEntry->pfn, pExtra, lpEntry->nSig, pHandlerInfo);
                }
        }

        return FALSE;   // not handled
}
```

其中最下面的一个for循环最为关键，`pMessageMap = GetMessageMap();`这一句为获取到`AFX_MSGMAP`结构的基址，`AFX_MSGMAP`结构里面保存了消息处理映射入口的基址。

消息映射结构：

```
struct AFX_MSGMAP
{
#ifdef _AFXDLL
const AFX_MSGMAP* (PASCAL* pfnGetBaseMap)();
#else
const AFX_MSGMAP* pBaseMap;
#endif
const AFX_MSGMAP_ENTRY* lpEntries;
};
```

在这个结构中，我们只关心`lpEntries`这个值，因为这个值指向“消息处理映射入口”数组的首地址

消息处理映射入口结构：

```
AFX_MSGMAP_ENTRY
{
nMessage;
nCode   ;
nID  ;
nLastID ;
nSig    ;
lpEntry ;
}
```

在这个`AFX_MSGMAP_ENTRY`里面，`nID`保存了控件的ID，`lpEntry`则是控件的入口地址。

我们要自己为应用程序添加消息映射，关键的就是找到`pMessageMap`，然后在`lpEntries`的后面添加一个自己的消息响应结构。

对于按钮的消息入口则通常为：

```
AFX_MSGMAP_ENTRY
{
nMessage = 0x111 WM_COMMAND
nCode    = 0
nID   = (ID) 按钮控件的ID
nLastID  = (ID) 按钮控件的ID
nSig     = 0x38
lpEntry (消息响应函数入口)
}
```

在通常情况下，VC编译器编译好了的应用程序是不会留出很多的空位置来给我们添加自己的消息响应，因此，我们必须在一处新的位置来添加自己的消息映射，所以在下面我们分配了一个`MessageMap`的区段来专门存放消息响应。

在下面，我们将要准备这些软件：Ollydbg v1.10，ResHacker 3.5（资源骇客），Zero Add，VS 2005，SPYXX

（1）使用ResHacker打开SPYXX，选择 对话框->24->2052，在这个窗口上添加5个按钮，按钮名称分别为“可用”、“变灰”、“显示”、“隐藏”、“关闭”，按钮ID分别为14550,14551,14552,14553,14554，然后保存文件。

（2）使用Zero Add工具为SPYXX增加2个区段，分别为`chCode`，1000字节，`MessageMap`，4000字节，然后保存文件。

（3）为了定位到`_AfxDispatchCmdMsg`，我们用VS2005自己建立一个MFC对话框工程，创建一个按钮响应，在响应内部设置断点，触发断点后查看堆栈，双击> `mfc80u.dll!CCmdTarget::OnCmdMsg(unsigned int nID=1, int nCode=0, void * pExtra=0x00000000, AFX_CMDHANDLERINFO * pHandlerInfo=0x00000000)` 行381 + 0x16 字节 C++

来到`mfc80u`的领空，转到反汇编，看到如下的反汇编代码（你的代码地址可能和我的不一样，请以自己的实际地址为准）：

```
7833A4C4 FF 70 10         push        dword ptr [eax+10h]
7833A4C7 8B 55 14         mov         edx,dword ptr [ebp+14h]
7833A4CA FF 75 08         push        dword ptr [ebp+8]
7833A4CD 8B 75 10         mov         esi,dword ptr [ebp+10h]
7833A4D0 8B 40 14         mov         eax,dword ptr [eax+14h]
7833A4D3 8B CF            mov         ecx,edi
7833A4D5 E8 F7 FD FF FF   call        _AfxDispatchCmdMsg (7833A2D1h)
7833A4DA EB E1            jmp         $LN48+0DAh (7833A4BDh)
```

可以看到`7833A4D5`就是调用`_AfxDispatchCmdMsg`的地址，向上我们可以看到：

```
7833A498 8B 07            mov         eax,dword ptr [edi]
7833A49A FF 50 30         call        dword ptr [eax+30h]
7833A49D EB 15            jmp         $LN48+0D1h (7833A4B4h)
```

这里的`call dword ptr [eax+30h]`就是调用`GetMessageMap()`的CALL，在这里获取到`AFX_MSGMAP`的地址。

（4）下面我们用OD载入SPYXX，运行程序，然后`bp 7833A49A`下断点，找到断点后设置条件`[ebp+8] == 0x38D6`（注：`ebp+8`为控件ID，`0x38D6`（十进制为14550）为“可用”按钮的ID），单击“可用”后被断下，F8单步，此时`eax = 0040A2D0`，选择在数据窗口中跟随，看到如下代码：

```
0040A2D0  00423B56  spyxx.00423B56
0040A2D4  0040A240  spyxx.0040A240（这个地址是关键，AFX_MSGMAP_ENTRY入口地址）
```

使用命令`dd 0040A240`后看到消息映射：

```
0040A240  00000111
0040A244  00000000
0040A248  00000649
0040A24C  00000649
0040A250  00000038
0040A254  00439B52  spyxx.00439B52
0040A258  00000111
0040A25C  00000000
0040A260  0000064F
0040A264  0000064F
0040A268  00000038
0040A26C  00439B83  spyxx.00439B83
0040A270  00000111
0040A274  00000000
0040A278  00000650
0040A27C  00000650
0040A280  00000038
0040A284  00439B83  spyxx.00439B83
0040A288  00000111
0040A28C  00000200
0040A290  00000651
0040A294  00000651
0040A298  00000038
0040A29C  004399DE  spyxx.004399DE
0040A2A0  0000001C
0040A2A4  00000000
0040A2A8  00000000
0040A2AC  00000000
```
