

# diy 迅雷的开放引擎

> 原文：[`www.pediy.com/kssd/pediy12/144496.html`](https://www.pediy.com/kssd/pediy12/144496.html)

【文章标题】: diy 迅雷的开放引擎  
【文章作者】: Netsd  
【作者邮箱】: yezhu\_love@163.com  
【软件名称】: 迅雷开放引擎  
【下载地址】: http://thunderplatform.xunlei.com/  
【加壳方式】: 无  
【保护方式】: 无  
【编写语言】: VC  
【使用工具】: od vs2008  
【操作平台】: win32  
【软件介绍】: 依托迅雷强大的下载加速技术，加速下载  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
    
     PS:本文只是从技术角度进行分析，无意侵犯迅雷的版权，也没对迅雷的文件，进行任何的修改。  
    
     迅雷，开放的一个极其简单的下载引擎，不过看起来扭扭捏捏的，开放的不够彻底，每次用使用，还总自动来个托盘，提示安装迅雷可以加速，右键点下，还不能退出。左键单击- -！那更是坑爹，直接开始下载迅雷了。更要命的是，这东西还搞共享的托盘。牢骚发了一堆，直接进入正题吧。下面就直接搞掉托盘和共享一个托盘的问题。  
    
  下面是下载的链接：http://thunderplatform.xunlei.com/  
    
    这次走的是 IAT HOOK 的路子，不 patch 文件，还有数字签名，看着安全。。。  
    
    开始的时候，直接断 Shell\_NotifyIconW，能看到关键字符串。用 IAT HOOK 搞之，修改图标，发现能正常的显示，设置后的图标。后面发现在先开起其他实例的情况下，我自己的托盘就不出来了。也就是上面说的，共享了一个托盘。  
    
    下面就主要去 patch 那个东西。跨进程的首先考虑窗口和共享内存的东西  
    

代码:

```
  10039F0D   push XLDownlo.1003BBBC                         xl\_open\_downlaod\_engine\_mutex
  10039F28   push XLDownlo.1003BBF8                          xl\_open\_downlaod\_engine\_share\_memory
  10039F43   push XLDownlo.1003BC48                          xl\_open\_downlaod\_engine\_task\_tray\_icon\_wnd\_name
  10039F5E   push XLDownlo.1003BCA8                          xl\_tasknumber\_and\_speed
  10039FB3   push XLDownlo.1003C800                          DownloadServerNeedFileList.dat
  10039FCE   push XLDownlo.1003C840                          xl\_open\_download\_engine\_temp\_folder

```

  
  直接就能看见几个关键的字符串了。我试了下，修改 xl\_open\_downlaod\_engine\_share\_memory 和 xl\_open\_downlaod\_engine\_task\_tray\_icon  
  就可以跳过共享一个托盘的问题。自己拥有一个新的实例，不会被其他的实例干扰。  
  
  xl\_open\_downlaod\_engine\_task\_tray\_icon 是一个窗口的类名，所以需要 hook Hook\_RegisterClassExW，当遇到这个字符串的时候  
  直接替换，然后记录下，注册之后的结果，代码如下  
代码:

```
  LPCTSTR lpNetsdFileMap = \_T("Netsd\_xl\_open\_downlaod\_engine\_share\_memory");
  
  ATOM ret = 0;
  
  LPCWSTR lpXlTrayIconClassName  = L"xl\_open\_downlaod\_engine\_task\_tray\_icon";
  LPCWSTR lpHookXlTrayClass = L"Netsd\_TrayIcon";
  
  ATOM WINAPI CTestProject2Dlg::Hook\_RegisterClassExW(WNDCLASSEX \*lpwcx)
  {
     if(lpwcx && (StrCmpW(lpwcx->lpszClassName,lpXlTrayIconClassName) == 0))
     {
      lpwcx->lpszClassName = lpHookXlTrayClass;
       ret = RegisterClassExW(lpwcx);
       return ret;
     }
     else
     {
      return RegisterClassExW(lpwcx);
     }
  
  }

```

  还需要 HOOK CreateWindowExW 判断是不是需要处理的窗口类，然后顺便替换窗口名字  
  
代码:

```
HWND WINAPI CTestProject2Dlg::Hook\_CreateWindowExW(DWORD dwExStyle,
                      LPCWSTR lpClassName,
                      LPCWSTR lpWindowName,
                      DWORD dwStyle,
                      int X,
                      int Y,
                      int nWidth,
                      int nHeight,
                      HWND hWndParent,
                      HMENU hMenu,
                      HINSTANCE hInstance,
                      LPVOID lpParam)
  {
  
  
    if ((ATOM)lpClassName == ret)
    {
        AtlTrace(lpWindowName);
        AtlTrace(\_T("\\r\\n"));
      lpWindowName = \_T("Netsd\_TrayIcon\_WindowName");
    }
    return CreateWindowExW(dwExStyle,lpClassName,lpWindowName,dwStyle,X,Y,nWidth,nHeight,hWndParent,hMenu,hInstance,lpParam);
  }
  

```

  剩下的就是 HOOK CreateFileMappingW 替换共享内存的名字  
  
代码:

```
  HANDLE WINAPI CTestProject2Dlg::Hook\_CreateFileMappingW(
    \_\_in      HANDLE hFile,
    \_\_in\_opt  LPSECURITY\_ATTRIBUTES lpAttributes,
    \_\_in      DWORD flProtect,
    \_\_in      DWORD dwMaximumSizeHigh,
    \_\_in      DWORD dwMaximumSizeLow,
    \_\_in\_opt  LPCWSTR lpName
    )
  {
    AtlTrace(lpName);
    AtlTrace(\_T("\\r\\n"));
    return CreateFileMappingW(hFile,lpAttributes,flProtect,dwMaximumSizeHigh,dwMaximumSizeLow,lpNetsdFileMap);
  }
  

```

  
  最后就在 HOOK Shell\_NotifyIconW 让迅雷的注册托盘的成为睁眼瞎，然后我们获取下载速度等信息，发送到自己托盘上  
代码:

```
 BOOL WINAPI CTestProject2Dlg::Hook\_Shell\_NotifyIconW(DWORD dwMessage,PNOTIFYICONDATAW lpData)
  {
  
    DWORD dwCbSize;
    WCHAR wszTemp\[128\];
    dwCbSize = 0;
    
    CTestProject2Dlg \* m\_this;
    LPCTSTR lpConvent;
  
    m\_this = (CTestProject2Dlg \*)(theApp.GetMainWnd());
    lpConvent = NULL;
    if(lpData && m\_this)
    {
      dwCbSize = lpData->cbSize;
  
      if(IsBadWritePtr(lpData,dwCbSize) == FALSE)
      {
        if(dwMessage == NIM\_ADD)
        {
  //         lpData->hIcon =  AfxGetApp()->LoadIcon(IDR\_MAINFRAME);
  //         lpData->uID = IDR\_TRAYMENU;
  //         lpData->hWnd = theApp.GetMainWnd()->GetSafeHwnd();
  //         lpData->uCallbackMessage = WM\_MY\_TRAY\_NOTIFICATION;
        }
        else if(dwMessage == NIM\_MODIFY)
        {
          if(lpData->uFlags == (NIF\_MESSAGE|NIF\_ICON|NIF\_TIP))
          {
             ZeroMemory(wszTemp,sizeof(wszTemp));
             StringCbCopyW(wszTemp,sizeof(wszTemp),L"XXX 升级程序");
             if(lpData->szTip\[8\])
               StringCbCatW(wszTemp,sizeof(wszTemp),lpData->szTip+8);
             //StringCbCopyW(lpData->szTip,sizeof(lpData->szTip),wszTemp);
            //AtlTrace(lpData->szTip);
            //AtlTrace(L"\\r\\n");
            lpConvent = W2T(wszTemp);
            m\_this->m\_trayIcon.SetTooltipText(lpConvent);
          }else if(lpData->uFlags == (NIF\_MESSAGE|NIF\_ICON|NIF\_TIP|NIF\_INFO))
          {
            //StringCbCopyW(lpData->szInfoTitle,sizeof(lpData->szInfoTitle),L"xxxxx 下载");
            //StringCbCopyW(lpData->szInfo,sizeof(lpData->szInfo),L"xxxxxxxxx\_Info");
            //AtlTrace(lpData->szInfo);
            lpConvent = W2T(lpData->szInfoTitle);
            m\_this->m\_trayIcon.ShowInfo(lpConvent,\_T("XXX 升级程序"));
          }
        }
      }
    }
    return TRUE;
  }

```

  
  还有就是自己实现一个托盘了，详细的看代码吧。  
  
ThunderPlatform\_SDK\_1.1.0.rar  
  
\--------------------------------------------------------------------------------  
【经验总结】  
  主要还是注意寻找关键点，正常程序字符串应该都很容易看到。  
  
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2011 年 12 月 20 日 22:29:15