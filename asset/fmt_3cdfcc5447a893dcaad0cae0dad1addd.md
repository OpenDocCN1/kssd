
# 新查出的一个 BUG，和 MFC 注册窗口类管理相关

> 原文：[`www.pediy.com/kssd/pediy11/110020.html`](https://www.pediy.com/kssd/pediy11/110020.html)

最近我维护的一个项目出了问题，会莫名其妙地崩溃，分析崩溃现场，发现是内存被莫名其妙冲坏，但是对调查没有任何帮助，今天终于查到了原因，和大家分享，希望大家引以为戒：

MSDN 上说，在 `DLL` 中注册的窗口类在 `DLL` 卸载时不会自动反注册，我们程序必须保证在 `DLL` 卸载时，将所有在 `DLL` 中注册的窗口类反注册掉（原文：Windows NT/2000 or later: No window classes registered by a `.dll` are unregistered when the `.dll` is unloaded. A `.dll` must explicitly unregister its classes when it is unloaded. ）。

MFC 也是这样做的，只要我们的 `DLL` 中都使用 `AfxRegisterWndClass` 来注册窗口类，就不会出问题，MFC 框架能够保证在 `DLL` 卸载时将所有在 `DLL` 中注册的窗口类反注册掉，所以微软建议在 `DLL` 中注册窗口类使用 `AfxRegisterWndClass` 调用注册窗口类。

MFC 根据 `AfxRegisterWndClass` 提供的参数组装成一个窗口类名，并且使用该窗口类名注册一个窗口类，如过该窗口是在 `DLL` 中注册的，注册成功后会将该窗口类名记录到 MFC 的一个内部变量中，该变量是一个 4096 大小的字符数组，各个窗口类名之间使用 `"\n\0"` 分隔。

为了防止相同类型的窗口类被反复注册，`AfxRegisterWndClass` 会先检查根据参数组装的窗口类名的窗口类是否已经注册，如果已经注册，`AfxRegisterWndClass` 就会直接返回那个窗口类名。

MFC 记录下窗口类名的目的是为了在 `DLL` 卸载时，将所有在 `DLL` 中注册的窗口类都反注册掉，否则会有问题。注意，如果 `DLL` 中注册的窗口类超过了 MFC 设计容限，反注册列表会溢出，进而导致程序崩溃，没有任何警告，往往在运行中某一时刻爆发，这种缺陷是比较隐蔽的，再现性一般不确定，往往修正成本很高。

我们的项目的某些操作会无节制地注册大量的窗口类，会轻易超过了微软设计人员的设计界限，导致数组越界，进而导致程序崩溃。

下面附上 MFC 相关源代码，代码比较简单明了，很容易理解：

```cpp
LPCTSTR AFXAPI AfxRegisterWndClass(UINT nClassStyle,
  HCURSOR hCursor, HBRUSH hbrBackground, HICON hIcon)
{
  // Returns a temporary string name for the class
  //  Save in a CString if you want to use it for a long time
  LPTSTR lpszName = AfxGetThreadState()->m_szTempClassName;

  // generate a synthetic name for this class
  HINSTANCE hInst = AfxGetInstanceHandle();

  if (hCursor == NULL && hbrBackground == NULL && hIcon == NULL)
    wsprintf(lpszName, _T("Afx:%x:%x"), (UINT)hInst, nClassStyle);
  else
    wsprintf(lpszName, _T("Afx:%x:%x:%x:%x:%x"), (UINT)hInst, nClassStyle,
      (UINT)hCursor, (UINT)hbrBackground, (UINT)hIcon);

  // see if the class already exists
  WNDCLASS wndcls;
  if (::GetClassInfo(hInst, lpszName, &wndcls))
  {
    // already registered, assert everything is good
    ASSERT(wndcls.style == nClassStyle);

    // NOTE: We have to trust that the hIcon, hbrBackground, and the
    //  hCursor are semantically the same, because sometimes Windows does
    //  some internal translation or copying of those handles before
    //  storing them in the internal WNDCLASS retrieved by GetClassInfo.

    return lpszName;
  }

  // otherwise we need to register a new class
  wndcls.style = nClassStyle;
  wndcls.lpfnWndProc = DefWindowProc;
  wndcls.cbClsExtra = wndcls.cbWndExtra = 0;
  wndcls.hInstance = hInst;
  wndcls.hIcon = hIcon;
  wndcls.hCursor = hCursor;
  wndcls.hbrBackground = hbrBackground;
  wndcls.lpszMenuName = NULL;
  wndcls.lpszClassName = lpszName;

  if (!AfxRegisterClass(&wndcls))
    AfxThrowResourceException();

  // return thread-local pointer
  return lpszName;
}

// like RegisterClass, except will automatically call UnregisterClass
BOOL AFXAPI AfxRegisterClass(WNDCLASS* lpWndClass)
{
  WNDCLASS wndcls;
  if (GetClassInfo(lpWndClass->hInstance, lpWndClass->lpszClassName,
    &wndcls))
  {
    // class already registered
    return TRUE;
  }

  if (!::RegisterClass(lpWndClass))
  {
    TRACE1("Can't register window class named %s\n",
      lpWndClass->lpszClassName);
    return FALSE;
  }

  if (afxContextIsDLL)
  {
    AfxLockGlobals(CRIT_REGCLASSLIST);
    TRY
    {
      // class registered successfully, add to registered list
      AFX_MODULE_STATE* pModuleState = AfxGetModuleState();
      LPTSTR lpszUnregisterList = pModuleState->m_szUnregisterList;
      // the buffer is of fixed size -- ensure that it does not overflow
      ASSERT(lstrlen(lpszUnregisterList) + 1 +
        lstrlen(lpWndClass->lpszClassName) + 1 <
        _countof(pModuleState->m_szUnregisterList));

      // append classname + newline to m_szUnregisterList
      lstrcat(lpszUnregisterList, lpWndClass->lpszClassName);
      TCHAR szTemp[2];
      szTemp[0] = '\n';
      szTemp[1] = '\0';
      lstrcat(lpszUnregisterList, szTemp);
    }
    CATCH_ALL(e)
    {
      AfxUnlockGlobals(CRIT_REGCLASSLIST);
      THROW_LAST();
      // Note: DELETE_EXCEPTION not required.
    }
    END_CATCH_ALL
    AfxUnlockGlobals(CRIT_REGCLASSLIST);
  }

  return TRUE;
}

extern "C"
BOOL WINAPI DllMain(HINSTANCE hInstance, DWORD dwReason, LPVOID /*lpReserved*/)
{
  if (dwReason == DLL_PROCESS_ATTACH)
  {
    BOOL bResult = FALSE;

#ifdef _AFXDLL
    // wire up resources from core DLL
    AfxCoreInitModule();
#endif
    _AFX_THREAD_STATE* pState = AfxGetThreadState();
    AFX_MODULE_STATE* pPrevModState = pState->m_pPrevModuleState;

    // Initialize DLL's instance(/module) not the app's
    if (!AfxWinInit(hInstance, NULL, _T(""), 0))
    {
      AfxWinTerm();
      goto Cleanup;       // Init Failed
    }

    // initialize the single instance DLL
    CWinApp* pApp; pApp = AfxGetApp();
    if (pApp != NULL && !pApp->InitInstance())
    {
      pApp->ExitInstance();
      AfxWinTerm();
      goto Cleanup;       // Init Failed
    }

    pState->m_pPrevModuleState = pPrevModState;

#ifdef _AFXDLL
    // wire up this DLL into the resource chain
    VERIFY(AfxInitExtensionModule(controlDLL, hInstance));

    CDynLinkLibrary* pDLL; pDLL = new CDynLinkLibrary(controlDLL);
    ASSERT(pDLL != NULL);

#else
    AfxInitLocalData(hInstance);
#endif

    bResult = TRUE;

Cleanup:
    pState->m_pPrevModuleState = pPrevModState;

#ifdef _AFXDLL
    // restore previously-saved module state
    VERIFY(AfxSetModuleState(AfxGetThreadState()->m_pPrevModuleState) ==
      &afxModuleState);

    DEBUG_ONLY(AfxGetThreadState()->m_pPrevModuleState = NULL);
#endif

    return bResult;
  }

  else if (dwReason == DLL_PROCESS_DETACH)
  {

#ifdef _AFXDLL
    // set module state for cleanup
    ASSERT(AfxGetThreadState()->m_pPrevModuleState == NULL);

    AfxGetThreadState()->m_pPrevModuleState =
      AfxSetModuleState(&afxModuleState);
#endif

    CWinApp* pApp = AfxGetApp();
    if (pApp != NULL)
      pApp->ExitInstance();

#ifdef _DEBUG
    // check for missing AfxLockTempMap calls
    if (AfxGetModuleThreadState()->m_nTempMapLock != 0)
    {
      TRACE1("Warning: Temp map lock count non-zero (%ld).\n",
        AfxGetModuleThreadState()->m_nTempMapLock);
    }
#endif

    AfxLockTempMaps();
    AfxUnlockTempMaps(-1);

    // terminate the library before destructors are called
    AfxWinTerm();

#ifdef _AFXDLL
    AfxTermExtensionModule(controlDLL, TRUE);
#else
    AfxTermLocalData(hInstance, TRUE);
#endif
  }

  else if (dwReason == DLL_THREAD_DETACH)
  {
    AFX_MANAGE_STATE(&afxModuleState);

#ifdef _DEBUG
    // check for missing AfxLockTempMap calls
    if (AfxGetModuleThreadState()->m_nTempMapLock != 0)
    {
      TRACE1("Warning: Temp map lock count non-zero (%ld).\n",
        AfxGetModuleThreadState()->m_nTempMapLock);
    }
#endif

    AfxLockTempMaps();
    AfxUnlockTempMaps(-1);
    AfxTermThread(hInstance);
  }

  return TRUE;
}

void AFXAPI AfxWinTerm(void)
{
  // unregister Window classes
  AFX_MODULE_STATE* pModuleState = AfxGetModuleState();
  AfxLockGlobals(CRIT_REGCLASSLIST);
  LPTSTR lpsz = pModuleState->m_szUnregisterList;

  while (*lpsz != 0)
  {
    LPTSTR lpszEnd = _tcschr(lpsz, '\n');
    ASSERT(lpszEnd != NULL);
    *lpszEnd = 0;
    UnregisterClass(lpsz, AfxGetInstanceHandle());
    lpsz = lpszEnd + 1;
  }

  pModuleState->m_szUnregisterList[0] = 0;
  AfxUnlockGlobals(CRIT_REGCLASSLIST);

  // cleanup OLE if required
}
```
