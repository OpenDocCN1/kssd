
```assembly
73D31393  JMP SHORT MFC42.73D313BD  

73D31395  /PUSH ESI  

73D31396  |CALL MFC42.#2867_?FromHandlePermanent@CWnd@@SGPAV>        ;取得 CWnd 指针值  

73D3139B  |TEST EAX,EAX                 ;CWnd * 值不为 NULL 时，则调用 CWnd::PreTranslateMessage()  

73D3139D  |JE SHORT MFC42.73D313AE  

73D3139F  |MOV EDX,DWORD PTR DS:[EAX]           

73D313A1  |PUSH EDI  

73D313A2  |MOV ECX,EAX  

73D313A4  |CALL DWORD PTR DS:[EDX+98]  ;<JMP.&MFC42.#5280_?PreTranslateMessage > ;通过虚函数方式调用  

73D313AA  |TEST EAX,EAX  

73D313AC  |JNZ SHORT MFC42.73D313C8        ;消息被处理了，返回 TRUE  

73D313AE  |CMP ESI,DWORD PTR SS:[ESP+C]  

73D313B2  |JE SHORT MFC42.73D313C1  

73D313B4  |PUSH ESI                             ; /hWnd  

73D313B5  |CALL DWORD PTR DS:[<&USER32.GetParent>]            ; \GetParent  

73D313BB  |MOV ESI,EAX  

73D313BD   TEST ESI,ESI  

73D313BF  \JNZ SHORT MFC42.73D31395  

73D313C1  XOR EAX,EAX                 ;返回 FALSE  

73D313C3  POP EDI  

73D313C4  POP ESI  

73D313C5  RETN 8  

73D313C8  XOR EAX,EAX  

73D313CA  INC EAX                 ;返回 TRUE  

73D313CB  JMP SHORT MFC42.73D313C3  
```

跟踪说明：

在上面一句 `73D313A4  CALL DWORD PTR DS:[EDX+98]` 设置按钮点击条件断点： `[[esp]]==002407B4 && [[esp]+4]==202`。

可以发现：当点击按钮后，按钮点击事件函数代码就会在这条语句后执行，当按钮事件函数代码执行完毕后，`CALL` 才会返回 `TRUE`。

### 4. CD1Dlg::PreTranslateMessage 函数

```cpp
BOOL CDialog::PreTranslateMessage(MSG* pMsg)
{    ...
 return CDialog::PreTranslateMessage(pMsg);
}
```

若接收消息的窗口类重载了 `PreTranslateMessage` 函数，则此时会调用它，否则就进入第 5 步。实际应用中，这里很有可能是消息流程的一个分水岭，可能走向两条不同的道路。这完全取决于应用程序新增的代码，若应用程序在这里返回 `TRUE`，消息流程就返回去了。否则，就会继续往下执行。

在跟踪按钮消息时，此处应作为一个注意点，而设置断点的最佳位置是在上一步 `WalkPreTranslateTree` 函数中所说的位置，跟踪下来，注意消息流程的走向。

### 5. CDialog::PreTranslateMessage 函数

```cpp
BOOL CDialog::PreTranslateMessage(MSG* pMsg)
{
 if (CWnd::PreTranslateMessage(pMsg)) return TRUE;
 CFrameWnd* pFrameWnd = GetTopLevelFrame();
 if (pFrameWnd != NULL && pFrameWnd->m_bHelpMode) return FALSE;
 if (pMsg->message == WM_KEYDOWN && (pMsg->wParam == VK_ESCAPE || pMsg->wParam == VK_CANCEL) &&
  (::GetWindowLong(pMsg->hwnd, GWL_STYLE) & ES_MULTILINE) &&_AfxCompareClassName(pMsg->hwnd, _T("Edit")))
 {
  HWND hItem = ::GetDlgItem(m_hWnd, IDCANCEL);
  if (hItem == NULL || ::IsWindowEnabled(hItem))
  {
   SendMessage(WM_COMMAND, IDCANCEL, 0);
   return TRUE;
  }
 }
 return PreTranslateInput(pMsg);      // 消息流入此处
}
```

`CDialog::PreTranslateMessage()` 反汇编代码如下：

```assembly
73D468A4 >  PUSH ESI  
73D468A5    PUSH EDI  
73D468A6    MOV EDI,DWORD PTR SS:[ESP+C]  
73D468AA    MOV ESI,ECX  
73D468AC    PUSH EDI  
73D468AD    CALL MFC42.#5290_?PreTranslateMessage@CWnd@@UAEHPAUtagMSG@>  
73D468B2    TEST EAX,EAX  
73D468B4    JNZ MFC42.73D8D490  
73D468BA    MOV ECX,ESI  
73D468BC    CALL MFC42.#3813_?GetTopLevelFrame@CWnd@@QBEPAVCFrameWnd@@>  
73D468C1    TEST EAX,EAX  
73D468C3    JNZ MFC42.73D8D429  
73D468C9    CMP DWORD PTR DS:[EDI+4],100  
73D468D0    JE SHORT MFC42.73D468DF  
73D468D2    PUSH EDI  
73D468D3    MOV ECX,ESI  
73D468D5    CALL MFC42.#5278_?PreTranslateInput@CWnd@@QAEHPAUtagMSG@@@>     ;消息从流入此处  
73D468DA    POP EDI  
73D468DB    POP ESI  
73D468DC    RETN 4  
```

### 6. CWnd::PreTranslateInput 函数

```cpp
BOOL CWnd::PreTranslateInput(LPMSG lpMsg)
{
 if ((lpMsg->message < WM_KEYFIRST || lpMsg->message > WM_KEYLAST) &&
  (lpMsg->message < WM_MOUSEFIRST || lpMsg->message > WM_MOUSELAST))  // 过滤消息
  return FALSE;
 return IsDialogMessage(lpMsg);
}
```

从源码中可以看出，这个函数是对消息进行过滤，对于按键消息和鼠标消息，直接返回 `FALSE`，然后再返回到 `PumpMessge` 函数中，调用 `TranslageMessage()` 和 `DispatchMessage()` 函数，进行消息转换和分发，再进入 MFC。对于其它消息，则调用 `CWnd::IsDialogMessage()` 函数进行下一步处理。

`CWnd::PreTranslateInput()` 函数反汇编代码如下：

```assembly
73D34009 >  MOV EDX,DWORD PTR SS:[ESP+4]  
73D3400D    MOV EAX,DWORD PTR DS:[EDX+4]  
73D34010    CMP EAX,100  
73D34015    JNB SHORT MFC42.73D34023  
73D34017    CMP EAX,200  
73D3401C    JNB SHORT MFC42.73D34032  
73D3401E    XOR EAX,EAX  
73D34020    RETN 4  
73D34023    CMP EAX,108  
73D34028   ^ JA SHORT MFC42.73D34017  
73D3402A    PUSH EDX  
73D3402B    CALL MFC42.#4047_?IsDialogMessageA@CWnd@@QAEHPAUtagMSG@@@Z  
73D34030  ^ JMP SHORT MFC42.73D34020  
73D34032    CMP EAX,209  
73D34037  ^ JBE SHORT MFC42.73D3402A  
73D34039  ^ JMP SHORT MFC42.73D3401E  
```

### 7. CWnd::IsDialogMessageA 函数

```cpp
BOOL CWnd::IsDialogMessage(LPMSG lpMsg)
{
 if (m_nFlags & WF_OLECTLCONTAINER)
  return afxOccManager->IsDialogMessage(this, lpMsg);
 else
  return ::IsDialogMessage(m_hWnd, lpMsg);
}
```

这里会转进 `User32.IsDialogMessageA` 函数，从而转入系统内核，由 `Windows` 系统再来负责将消息的分发传送到各个目标窗口。

注：`User32.IsDialogMessage` 并不是象它的名字那样用来检查对话框消息的，而是用来解释或转换消息的。更贴切的名字应该是 `TranslateDialogMessage`。`CWnd::IsDialogMessage` 实际上是一个以 `LPMSG` 作为参数，再加上内部的 `m_hWnd` 参数来调用 `User32.IsDialogMessage` 的打包函数。这样，MFC 中每一个对话框都会解释自己的输入。所以，若同时运行五个对话框，每一个对话框的 `PreTranslateMessage` 都会自动调用 `User32.IsDialogMessage`，而且运转良好，完全可以不用我们编程处理，MFC 真是太牛了。

`CWnd::IsDialogMessageA` 函数反汇编代码：

```assembly
73D468F5 > PUSH ESI  
73D468F6   MOV ESI,ECX  
73D468F8   TEST BYTE PTR DS:[ESI+29],1  
73D468FC   JNZ MFC42.73D8E273  
73D46902   PUSH DWORD PTR SS:[ESP+8]  
73D46906   PUSH DWORD PTR DS:[ESI+20]  
73D46909   CALL DWORD PTR DS:[<&USER32.IsDialogMessageA>]       ;进入系统内核  
73D4690F   POP ESI  
73D46910   RETN 4  
```

提示：

1. `OD` 中设断：`bp IsDialogMessageA MSG==202`，则当鼠标左键释放时，会中断在 `User32.IsDialogMessageA` 函数入口上。

2. 若已知按钮的句柄，且要求当点击该按钮时，程序中断在 `IsDialogMessageA` 上，则可以作如下设断：`bp IsDialogMessageA [[esp+8]]==00060350 && MSG==202.`

3. 中断后，可以通过堆栈返回到 `CWnd::IsDialogMessageA` 函数代码处。

`=========================================================================================================`

### 8. User32 内核处理，不分析

这里面的过程，我们就当作一个黑匣子吧，不管它，一般情况下，也无需管它。因为我们百分之百相信它。

`=========================================================================================================`

当消息到达此处时，又进入了 MFC 地界。第 8 步之前，可以说是经常峰回路转，山重水复。第 8 步之后，是柳暗花明，可以一路高歌，直奔目的地了。

欲知后事如何，且听下回分解！

上篇里嗦地说了一大堆，其实所说的消息都是 `PostMessage` 方式的。MFC 中还有另外一种很常见的消息发送方式，就是 `SendMessage` 函数。这个消息起始路径和上篇所讲的完全不一样。这种方式下，前面的 7 个站点均不执行，而是直接进入第 8 站点：`User32` 内核，从第 8 站点出来后，这两种消息方式走上了同一条道路，进入第 9 个站点或第 10 个站点了，真是殊道同归。

对于 MFC 窗口程序，所有窗口都使用同一窗口过程：`AfxWndProcBase`（第 9 个站点）或 `AfxWndProc`（第 10 个站点）。如果程序是动态链接到 MFC DLL（定义了 `_AFXDLL`），则 `AfxWndProcBase` 被用作窗口过程，否则 `AfxWndProc` 被用作窗口过程。而在 `AfxWndProcBase` 中，最终也是调用 `AfxWndProc` 函数。

所以，可以说，第 10 个站点：`AfxWndProc` 函数是 MFC 的所有消息必经之点。

可以作如下测试：在 `Button1` 事件代码中加入：`SendMessage(WM_COMMAND, IDC_BUTTON2, 0);` 这是往 `Button2` 发送点击消息，当点击 `Button1` 时，跟进 `Button1` 的事件代码流程，再跟进 `SendMessage` 函数的内部代码，可以发现，和上面所讲是完全一样的。

各位可能有疑问了，消息从 `User32` 内核出来之后，应该是由 `Windows` 系统自动发往各个窗口的消息处理函数，但这里怎么会全部进入了 `AfxWndProc()` 函数呢？这涉及到了钩子函数，有兴趣者，请看本文附录，正文不作多说。现在继续进入消息之旅：

请看以下源码：

### 9. AfxWndProcBase 函数

```cpp
LRESULT CALLBACK AfxWndProcBase(HWND hWnd, UINT nMsg, WPARAM wParam, LPARAM lParam)
{
 AFX_MANAGE_STATE(_afxBaseModuleState.GetData());
 return AfxWndProc(hWnd, nMsg, wParam, lParam);
}
```

`AfxWndProcBase` 首先使用宏 `AFX_MANAGE_STATE` 设置正确的模块状态，然后调用 `AfxWndProc`。

说明：如果程序是动态链接到 MFC DLL（定义了 `_AFXDLL`），则 `AfxWndProcBase` 被用作窗口过程，否则 `AfxWndProc` 被用作窗口过程。从源码可以知道，在 `AfxWndProcBase` 中，最终也是调用 `AfxWndProc` 函数。

`AfxWndProcBase` 反汇编代码：

```assembly
73D31B81 >  MOV EAX,MFC42.73DC2CFE  
73D31B86    CALL MFC42.__EH_prolog                                ; JMP 到 MSVCRT._EH_prolog  
73D31B8B    PUSH ECX  
73D31B8C    PUSH ECX  
73D31B8D    PUSH MFC42.#2188_?CreateObject@?$CProcessLocal@V_AFX_BASE_>  
73D31B92    MOV ECX,OFFSET MFC42._afxBaseModuleState  
73D31B97    CALL MFC42.#3028_?GetData@CProcessLocalObject@@QAEPAVCNoTr>  
73D31B9C    PUSH EAX  
```
