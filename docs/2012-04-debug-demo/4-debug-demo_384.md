
# diy 迅雷的开放引擎

> 原文：[`www.pediy.com/kssd/pediy12/144496.html`](https://www.pediy.com/kssd/pediy12/144496.html)

【文章标题】: diy 迅雷的开放引擎

【文章作者】: `Netsd`

【作者邮箱】: `yezhu_love@163.com`

【软件名称】: 迅雷开放引擎

【下载地址】: `http://thunderplatform.xunlei.com/`

【加壳方式】: 无

【保护方式】: 无

【编写语言】: `VC`

【使用工具】: `od` `vs2008`

【操作平台】: `win32`

【软件介绍】: 依托迅雷强大的下载加速技术，加速下载

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

## 【详细过程】

PS: 本文只是从技术角度进行分析，无意侵犯迅雷的版权，也没对迅雷的文件，进行任何的修改。

迅雷，开放的一个极其简单的下载引擎，不过看起来扭扭捏捏的，开放的不够彻底，每次用使用，还总自动来个托盘，提示安装迅雷可以加速，右键点下，还不能退出。左键单击 - -！那更是坑爹，直接开始下载迅雷了。更要命的是，这东西还搞共享的托盘。牢骚发了一堆，直接进入正题吧。下面就直接搞掉托盘和共享一个托盘的问题。

下面是下载的链接：`http://thunderplatform.xunlei.com/`

这次走的是 `IAT HOOK` 的路子，不 `patch` 文件，还有数字签名，看着安全。。。

开始的时候，直接断 `Shell_NotifyIconW`，能看到关键字符串。用 `IAT HOOK` 搞之，修改图标，发现能正常的显示，设置后的图标。后面发现在先开起其他实例的情况下，我自己的托盘就不出来了。也就是上面说的，共享了一个托盘。

下面就主要去 `patch` 那个东西。跨进程的首先考虑窗口和共享内存的东西

代码:

```
10039F0D   push XLDownlo.1003BBBC                     xl_open_downlaod_engine_mutex
10039F28   push XLDownlo.1003BBF8                     xl_open_downlaod_engine_share_memory
10039F43   push XLDownlo.1003BC48                     xl_open_downlaod_engine_task_tray_icon_wnd_name
10039F5E   push XLDownlo.1003BCA8                     xl_tasknumber_and_speed
10039FB3   push XLDownlo.1003C800                     DownloadServerNeedFileList.dat
10039FCE   push XLDownlo.1003C840                     xl_open_download_engine_temp_folder
```

直接就能看见几个关键的字符串了。我试了下，修改 `xl_open_downlaod_engine_share_memory` 和 `xl_open_downlaod_engine_task_tray_icon`

就可以跳过共享一个托盘的问题。自己拥有一个新的实例，不会被其他的实例干扰。

`xl_open_downlaod_engine_task_tray_icon` 是一个窗口的类名，所以需要 `hook` `Hook_RegisterClassExW`，当遇到这个字符串的时候

直接替换，然后记录下，注册之后的结果，代码如下

代码:

```
LPCTSTR lpNetsdFileMap = _T("Netsd_xl_open_downlaod_engine_share_memory");
 
ATOM ret = 0;
 
LPCWSTR lpXlTrayIconClassName  = L"xl_open_downlaod_engine_task_tray_icon";
LPCWSTR lpHookXlTrayClass = L"Netsd_TrayIcon";
 
ATOM WINAPI CTestProject2Dlg::Hook_RegisterClassExW(WNDCLASSEX *lpwcx)
{
    if(lpwcx && (StrCmpW(lpwcx->lpszClassName,lpXlTrayIconClassName) == 0))
    {
      lpwcx->lpszClassName = lpHookXlTrayClass;
      ret = RegisterClassExW(lpwcx);
      return ret;
    }
    else
    {
      return RegisterClassExW(lpwcx);
    }
 
}
```

还需要 `HOOK` `CreateWindowExW` 判断是不是需要处理的窗口类，然后顺便替换窗口名字

代码:

```
HWND WINAPI CTestProject2Dlg::Hook_CreateWindowExW(DWORD dwExStyle,
                  LPCWSTR lpClassName,
                  LPCWSTR lpWindowName,
                  DWORD dwStyle,
                  int X,
                  int Y,
                  int nWidth,
                  int nHeight,
                  HWND hWndParent,
                  HMENU hMenu,
                  HINSTANCE hInstance,
                  LPVOID lpParam)
{
 
 
    if ((ATOM)lpClassName == ret)
    {
        AtlTrace(lpWindowName);
        AtlTrace(_T("\\r\\n"));
      lpWindowName = _T("Netsd_TrayIcon_WindowName");
    }
    return CreateWindowExW(dwExStyle,lpClassName,lpWindowName,dwStyle,X,Y,nWidth,nHeight,hWndParent,hMenu,hInstance,lpParam);
}
```

剩下的就是 `HOOK` `CreateFileMappingW` 替换共享内存的名字

代码:

```
HANDLE WINAPI CTestProject2Dlg::Hook_CreateFileMappingW(
    __in      HANDLE hFile,
    __in_opt  LPSECURITY_ATTRIBUTES lpAttributes,
    __in      DWORD flProtect,
    __in      DWORD dwMaximumSizeHigh,
    __in      DWORD dwMaximumSizeLow,
    __in_opt  LPCWSTR lpName
    )
{
    AtlTrace(lpName);
    AtlTrace(_T("\\r\\n"));
    return CreateFileMappingW(hFile,lpAttributes,flProtect,dwMaximumSizeHigh,dwMaximumSizeLow,lpNetsdFileMap);
}
```

最后就在 `HOOK` `Shell_NotifyIconW` 让迅雷的注册托盘的成为睁眼瞎，然后我们获取下载速度等信息，发送到自己托盘上

代码:

```
BOOL WINAPI CTestProject2Dlg::Hook_Shell_NotifyIconW(DWORD dwMessage,PNOTIFYICONDATAW lpData)
{
 
    DWORD dwCbSize;
    WCHAR wszTemp[128];
    dwCbSize = 0;
    
    CTestProject2Dlg * m_this;
    LPCTSTR lpConvent;
 
    m_this = (CTestProject2Dlg *)(theApp.GetMainWnd());
    lpConvent = NULL;
    if(lpData && m_this)
    {
      dwCbSize = lpData->cbSize;
 
      if(IsBadWritePtr(lpData,dwCbSize) == FALSE)
      {
        if(dwMessage == NIM_ADD)
        {
 //          lpData->hIcon =  AfxGetApp()->LoadIcon(IDR_MAINFRAME);
 //          lpData->uID = IDR_TRAYMENU;
 //          lpData->hWnd = theApp.GetMainWnd()->GetSafeHwnd();
 //          lpData->uCallbackMessage = WM_MY_TRAY_NOTIFICATION;
        }
        else if(dwMessage == NIM_MODIFY)
        {
          if(lpData->uFlags == (NIF_MESSAGE|NIF_ICON|NIF_TIP))
          {
            ZeroMemory(wszTemp,sizeof(wszTemp));
            StringCbCopyW(wszTemp,sizeof(wszTemp),L"XXX 升级程序");
            if(lpData->szTip[8])
              StringCbCatW(wszTemp,sizeof(wszTemp),lpData->szTip+8);
            //StringCbCopyW(lpData->szTip,sizeof(lpData->szTip),wszTemp);
            //AtlTrace(lpData->szTip);
            //AtlTrace(L"\\r\\n");
            lpConvent = W2T(wszTemp);
            m_this->m_trayIcon.SetTooltipText(lpConvent);
          }else if(lpData->uFlags == (NIF_MESSAGE|NIF_ICON|NIF_TIP|NIF_INFO))
          {
            //StringCbCopyW(lpData->szInfoTitle,sizeof(lpData->szInfoTitle),L"xxxxx 下载");
            //StringCbCopyW(lpData->szInfo,sizeof(lpData->szInfo),L"xxxxxxxxx_Info");
            //AtlTrace(lpData->szInfo);
            lpConvent = W2T(lpData->szInfoTitle);
            m_this->m_trayIcon.ShowInfo(lpConvent,_T("XXX 升级程序"));
          }
        }
      }
    }
    return TRUE;
}
```

还有就是自己实现一个托盘了，详细的看代码吧。

`ThunderPlatform_SDK_1.1.0.rar`

--------------------------------------------------------------------------------

## 【经验总结】

主要还是注意寻找关键点，正常程序字符串应该都很容易看到。

--------------------------------------------------------------------------------

## 【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2011 年 12 月 20 日 22:29:15
