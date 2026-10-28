
# [分享]利用邮箱实现bug提交和自动检查更新

看了下邮件的发送与接收，了解了这方面知识，写一个帖子。

## 1、设置捕获

```
int WINAPI _tWinMain(HINSTANCE hInst, HINSTANCE h0, LPTSTR lpCmdLine, int nCmdShow)

 {

     BOOL bRet;

     MSG msg;

     HWND hwndFrame;

     

  [B][COLOR="Red"]   g_previousFilter = SetUnhandledExceptionFilter(TopLevelExceptionFilter);[/COLOR][/B]

     //_CrtSetBreakAlloc(58);

     InitCommonControls();

     g_hInst = hInst;

     //hModUxTheme = LoadLibrary(L"uxtheme.dll");

     //注册框架窗口类

   //创建框架窗口   

   //消息泵



     //if (hModUxTheme)

     //FreeLibrary(hModUxTheme);

     _CrtDumpMemoryLeaks();

     return (int) bRet;

 } 

```

## 2、捕获处理

```
LONG WINAPI TopLevelExceptionFilter(struct _EXCEPTION_POINTERS *pExceptionInfo)

 {

     SYSTEMTIME time;

     LONG lRet = EXCEPTION_CONTINUE_SEARCH;

     //HRESULT hr ;

     int end;

     GetLocalTime(&time);

     char acBuf[MAX_PATH] = {0};

     // Build a string showing the date and time.

     //Returns the number of characters written to the buffer, excluding any terminating NULL characters. A negative value is returned if an error occurs.

     end = wnsprintfA(acBuf, MAX_PATH, "%d%02d%02d%02d%02d%02d.dmp",time.wYear,

         time.wMonth, time.wDay,time.wHour, time.wMinute,time.wSecond);

     

     if(end > 0)

     {

         HANDLE hFile;

         char szBugFile[MAX_PATH+1] = {0};

         GetModuleFileNameA(NULL,szBugFile,MAX_PATH + 1);

         PathRemoveFileSpecA(szBugFile);

         PathAppendA(szBugFile,acBuf);

         hFile = CreateFileA(szBugFile,GENERIC_WRITE,FILE_SHARE_WRITE,NULL,CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL,NULL); 



         if(hFile == INVALID_HANDLE_VALUE) 

         { 

             ;

             //WriteLog(_T("ExceptionInfo"),_T("CreateFile"),_T("-1"));

         }else{

             BOOL (__stdcall *fnMiniDumpWriteDump)(HANDLE,DWORD,HANDLE,MINIDUMP_TYPE,

                 PMINIDUMP_EXCEPTION_INFORMATION, PMINIDUMP_USER_STREAM_INFORMATION,PMINIDUMP_CALLBACK_INFORMATION); 

             //typedef BOOL (__stdcall *MiniDumpWriteDumpProc)(HANDLE,DWORD,HANDLE,MINIDUMP_TYPE,

             //PMINIDUMP_EXCEPTION_INFORMATION, PMINIDUMP_USER_STREAM_INFORMATION,PMINIDUMP_CALLBACK_INFORMATION);

             HINSTANCE hinstLib; 

             //BOOL fFreeResult, fRunTimeLinkSuccess = FALSE; 

             //MiniDumpWriteDumpProc fnMiniDumpWriteDump;

             hinstLib = LoadLibrary(TEXT("DbgHelp.dll")); 



             if(hinstLib)

             {

                 (FARPROC&)fnMiniDumpWriteDump = ::GetProcAddress(hinstLib, "MiniDumpWriteDump");

                 //fnMiniDumpWriteDump = ( MiniDumpWriteDumpProc)::GetProcAddress(hinstLib, _T("MiniDumpWriteDump"));

                 if(fnMiniDumpWriteDump )

                 {

                     MINIDUMP_EXCEPTION_INFORMATION info;

                     info.ThreadId = ::GetCurrentThreadId();

                     info.ExceptionPointers = pExceptionInfo;

                     info.ClientPointers = FALSE;

                     BOOL bOk = fnMiniDumpWriteDump(::GetCurrentProcess,::GetCurrentProcessId(),hFile,MiniDumpNormal,&info,NULL,NULL);

                     if(bOk == FALSE) 

                         WriteLog(_T("ExceptionInfo"),_T("MiniDumpWriteDump"),_T("error(0x%x)"),GetLastError());

                     HWND hBug = CreateDialogParam(g_hInst, MAKEINTRESOURCE(IDD_BUGDLG), NULL, (DLGPROC)BugDlgProc,(LPARAM)&info);

                     SetWindowLong(hBug,GWL_USERDATA,(LONG)szBugFile);

                     ShowWindow(hBug,SW_NORMAL);

                     BOOL bRet;

                     MSG msg;

                     while ((bRet = GetMessage(&msg, NULL, 0, 0)) != 0) 

                     { 

                         if (bRet == -1)

                         {

                             // Handle the error and possibly exit

                             break;

                         }

                         else if (!IsWindow(hBug) || !IsDialogMessage(hBug, &msg)) 

                         { 

                             TranslateMessage(&msg); 

                             DispatchMessage(&msg); 

                         } 

                     }

                     lRet = EXCEPTION_EXECUTE_HANDLER;

                 }else

                     WriteLog(_T("ExceptionInfo"),_T("MiniDumpWriteDump"),_T("Failed to get"));

                 FreeLibrary(hinstLib); 

             }else

                 WriteLog(_T("ExceptionInfo"),_T("DbgHelp.dll"),_T("Failed to load"));

         }

         CloseHandle(hFile);

         

         //SAFE_DELETE(g_pszBugFile);

     }

     //if(g_previousFilter)

     //    return g_previousFilter(pExceptionInfo);

     //else

         return lRet;

 }

```

## 3、捕获后用户窗口界面消息处理

```
INT_PTR CALLBACK BugDlgProc(HWND hWnd, UINT uMsg, WPARAM wParam, LPARAM lParam)

 {

     switch(uMsg)

     {

     case CTLCOLOR_STATIC:

         {

             if(IDC_LINK == GetDlgCtrlID((HWND)lParam))

             {

                 HBRUSH br = CreateSolidBrush( RGB(255,255,255) );

                 return (INT_PTR)br;

             }

         }

         break;

     case WM_COMMAND:

         switch(LOWORD(wParam))

         {



         case IDC_BTNATTACH:

                 OnBugAddFiles(hWnd);

                 return 0;        

         case IDOK:

             {

                 if(BST_CHECKED == IsDlgButtonChecked(hWnd,IDC_SUBMIT))

                 {

                     BugSendMail(hWnd);

                 }

                 if(BST_CHECKED == IsDlgButtonChecked(hWnd,IDC_REBOOT))

                 {

                     TCHAR pszAppPath[MAX_PATH + 1];

                     GetModuleFileName(NULL,pszAppPath,MAX_PATH + 1);

                     ShellExecute(NULL, NULL,pszAppPath, NULL, NULL, SW_SHOWNORMAL);

                 }

             }

             break;

         case IDCANCEL:

                 DestroyWindow(hWnd);



         }

         break;

     case WM_INITDIALOG:

         {

             CenterWindow(hWnd,NULL);

             MINIDUMP_EXCEPTION_INFORMATION *pInfo = (MINIDUMP_EXCEPTION_INFORMATION*)lParam;

  



         }

         return TRUE;

     case WM_DESTROY:

         //SAFE_DELETE(g_pszAppPath);

         PostQuitMessage(0); 

         break;

     }

     return FALSE;

 } 



 //这里直接2个附件，多个附件需要改下Smtp类



void BugSendMail(HWND hWnd)

 {

     char acBugFile[MAX_PATH + 1] = {0};

     GetDlgItemTextA(hWnd,IDC_ATTACHFILE,acBugFile,MAX_PATH + 1);

     char* pMoreFile = (char*)GetWindowLong(hWnd,GWL_USERDATA);



     SetCursor( LoadCursor(NULL, IDC_WAIT) );

     Smtp smtp;

     char acount[] = "";

     DWORD dwSize;

     char  sName[MAX_COMPUTERNAME_LENGTH + 1];

     dwSize = MAX_COMPUTERNAME_LENGTH + 1;

     GetComputerNameA(sName, &dwSize);



     if(smtp.Connect("smtp.163.com") && smtp.Login(acount,"...")\

         && smtp.Send(acount,"...@qq.com",sName,"f","...",pData,acBugFile,pMoreFile,0))

         ;

     else MessageBox(NULL,_T("send failed."),_T("bug"),MB_OK|MB_ICONERROR);

     SetCursor( LoadCursor(NULL, IDC_ARROW) );



 }



 void CenterWindow(HWND hWnd,HWND hParent)

 {

     if(hParent == NULL)

         hParent = GetDesktopWindow(); 

     RECT rcParent,rc;

     GetWindowRect(hParent, &rcParent); 

     GetWindowRect(hWnd, &rc); 

     int w = rc.right-rc.left,

         h = rc.bottom-rc.top,

         x = rcParent.right - rcParent.left -w,

         y=rcParent.bottom - rcParent.top -h;

     x = rcParent.left + (long)(x/2.0 + 0.5);

     y = rcParent.top + (long)(y/2.0+0.5);

     SetWindowPos(hWnd, HWND_TOP, x, y, 0, 0,SWP_NOSIZE); 

 } 
```

## 5、检查更新
