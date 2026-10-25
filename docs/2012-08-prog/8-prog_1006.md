
# 《Windows 核心编程》第四版 `HandShake` 程序的 BUG

> 原文：[`www.pediy.com/kssd/pediy11/108954.html`](https://www.pediy.com/kssd/pediy11/108954.html)

这几天看《Windows 核心编程》书上 `HandShake` 的代码，发现了一个问题，猜想程序可能会因此陷入死锁状态。

后经实际测试，发现这确实是一个 BUG。再后来，修改了部分代码，BUG 消失了。![](img/45dd044a2743704cbb333715eae4a5f6.jpg "狂笑")

特发出来与大家分享。有什么不对的地方，还希望大家能够指出。![](img/cda69129882b40377a87144c782680e1.jpg "微笑")

## `Windows` 核心编程（第 4 版）`HandShake` 程序有一个 BUG

如果在 `Request` 编辑框里输入字符串 `Server Shutdown`，点击提交按钮，服务器线程会终止。

然后在编辑框里输入任意字符串，再次点击提交按钮时，由于服务器线程已经终止，所以事件 `g_hevtResultReturned` 不会被设置为通知状态，主线程陷入死锁。

本程序对其进行了改动。改动如下：

（1）主线程在提交请求之前，检查服务器线程是否已经终止。如果已经终止，则不进入等待函数。否则，提交请求，并进入等待函数。

（2）为了让主线程能够检查服务器线程是否已经终止，将服务器线程句柄设置为全局变量。

（3）`DialogBox` 返回之后，主线程仍然可能陷入死锁。原因同上。为此，添加检测服务器线程是否已经终止的代码。如果服务器线程已经终止，则直接进入清理句柄步骤。否则，发送字符串 `Server Shutdown`，令其终止，待终止后再进入清理句柄步骤。

另一种修改方法是，只等待服务器线程终止，而不等待事件 `g_hevtResultReturned` 被设置为通知状态。

修改后的代码如下：

```c
#include <windows.h>
#include <tchar.h>
#include <windowsX.h>

#include "Resource.h"

// This event is signaled when the client has a request for the server
HANDLE g_hevtRequestSubmitted;

// This event is signaled when the server has a result for the client
HANDLE g_hevtResultReturned;

// The buffer shared between the client and server threads
TCHAR  g_szSharedRequestAndResultBuffer[1024];

// The special value sent from the client that causes the 
// server thread to terminate cleanly.
TCHAR  g_szServerShutdown[] = _T("Server Shutdown");

// 服务器线程句柄
HANDLE g_hThreadServer;

///////////////////////////////////////////////////////////////////////////////


// This is the code executed by the server thread
DWORD WINAPI ServerThread(PVOID pvParam)
{
  
  // Assume that the server thread is to run forever
  BOOL fShutdown = FALSE;
  
  while ( !fShutdown )
  {
    
    // Wait for the client to submit a request
    WaitForSingleObject( g_hevtRequestSubmitted, INFINITE );
    
    // Check to see if the client wants the server to terminate
    fShutdown = 
      (lstrcmpi( g_szSharedRequestAndResultBuffer, g_szServerShutdown ) == 0);
    
    if (!fShutdown)
    {
      // Process the client's request (reverse the string)
      _tcsrev( g_szSharedRequestAndResultBuffer );
    }
    
    // Let the client process the request's result
    SetEvent( g_hevtResultReturned );
  }
  
  // The client wants us to shutdown, exit
  return 0;
}

///////////////////////////////////////////////////////////////////////////////


BOOL Dlg_OnInitDialog(HWND hwnd, HWND hwndFocus, LPARAM lParam)
{
  SendMessage( hwnd, WM_SETICON, TRUE,  (LPARAM) 
    LoadIcon( (HINSTANCE)hwnd, MAKEINTRESOURCE(IDI_HANDSHAKE) ) );   
  
  // Initialize the edit control with some test data request
  Edit_SetText( GetDlgItem( hwnd, IDC_REQUEST ), _T("Some test data") );
  
  return(TRUE);
}


///////////////////////////////////////////////////////////////////////////////


void Dlg_OnCommand(HWND hwnd, int id, HWND hwndCtl, UINT codeNotify)
{
  
  switch (id)
  {
    
  case IDCANCEL:
    EndDialog( hwnd, id );
    break;
    
  case IDC_SUBMIT:  // Submit a request to the server thread
    
    //检查服务器线程是否已经终止
    DWORD ExitCode;
    GetExitCodeThread( g_hThreadServer, &ExitCode );
    if ( ExitCode!=STILL_ACTIVE )
    {
      break;
    }

    // Copy the request string into the shared data buffer
    Edit_GetText( GetDlgItem(hwnd, IDC_REQUEST), 
      g_szSharedRequestAndResultBuffer, 
            sizeof(g_szSharedRequestAndResultBuffer) );

    // Let the server thread know that a request is ready in the buffer
    SetEvent( g_hevtRequestSubmitted );
    
    // Wait for the server to process the request and give us the result
    WaitForSingleObject( g_hevtResultReturned, INFINITE );/*如果用户已经发送过终止字符串，则 g_hevtResultReturned 不会变为通知状态*/
    
    // Let the user know the result
    Edit_SetText( GetDlgItem(hwnd, IDC_RESULT), 
            g_szSharedRequestAndResultBuffer );
    
    break;
  }
}


///////////////////////////////////////////////////////////////////////////////



int WINAPI Dlg_Proc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
  
  switch (uMsg)
  {
    HANDLE_MSG(hwnd, WM_INITDIALOG, Dlg_OnInitDialog);
    HANDLE_MSG(hwnd, WM_COMMAND,    Dlg_OnCommand);
  }
  
  return(FALSE);
}

int WINAPI _tWinMain( HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nShowCmd )
{
  // Create & initialize the 2 nonsignaled, auto-reset events
  g_hevtRequestSubmitted = CreateEvent( NULL, FALSE, FALSE, NULL );
  g_hevtResultReturned   = CreateEvent( NULL, FALSE, FALSE, NULL );
  
  // Spawn the server thread
  DWORD dwThreadID;
  g_hThreadServer = CreateThread( NULL, 0, ServerThread, NULL, 0, &dwThreadID );
  
  // Execute the client thread's user-interface
  DialogBox( hInstance, MAKEINTRESOURCE(IDD_HANDSHAKE), NULL, Dlg_Proc );
  
  // 检查服务器线程是否已经终止
  DWORD ExitCode;
  GetExitCodeThread( g_hThreadServer, &ExitCode );
  if ( ExitCode!=STILL_ACTIVE )
  {
    goto _clean_up_everything;
  }

  // The client's UI is closing, have the server thread shutdown
  lstrcpy( g_szSharedRequestAndResultBuffer, g_szServerShutdown );
  SetEvent( g_hevtRequestSubmitted );
  
  // Wait for the server thread to acknowledge the shutdown AND
  // wait for the server thread to fully terminate
  HANDLE h[2];
  h[0] = g_hevtResultReturned;
  h[1] = g_hThreadServer;
  WaitForMultipleObjects( 2, h, TRUE, INFINITE );/*如果用户已经发送过终止字符串，则 g_hevtResultReturned 不会变为通知状态*/

  //添加一个标签
_clean_up_everything:   
  // Properly clean up everything
  CloseHandle( g_hThreadServer );       
  CloseHandle( g_hevtRequestSubmitted );       
  CloseHandle( g_hevtResultReturned );       
  
    // The client thread terminates with the whole process

  return 0;
}
```

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) | `HandShake_修改版.rar` |
| :--- | :--- |
