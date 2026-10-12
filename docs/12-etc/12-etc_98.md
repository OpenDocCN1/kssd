
# \[WM 系统开发\] 获取用户拨打电话记录

> 原文：[`www.pediy.com/kssd/pediy10/79745.html`](https://www.pediy.com/kssd/pediy10/79745.html)

## 意义
病毒在手机中可以查看用户的最近拨号记录，知道用户给那些人拨打了电话,电话从什么时间开始到什么时间结束等。

## 实现方法
`PhoneOpenCallLog()`生成呼叫日志句柄。
`PhoneGetCallLogEntry()`获取呼叫日志中的单个条目，并移动条目指针到下一条。
此函数获取的电话通话信息非常丰富。可以深入研究。
`PhoneCloseCallLog()`关闭呼叫日志句柄。

## 示例代码如下

```c
#include "stdafx.h"
#include "CallLog01.h"
#include <windows.h>
#include <commctrl.h>
#include <phone.h>
#pragma comment(lib,"phone.lib")
#include "E:\\MFC 程序\\XHProgram\\MGSEI_ERROR.h"
#include "E:\\MFC 程序\\XHProgram\\XHLIB.h"
int WINAPI WinMain(HINSTANCE hInstance,
           HINSTANCE hPrevInstance,
           LPTSTR    lpCmdLine,
           int       nCmdShow)
{
  HANDLE       hCallLog = NULL;
  CALLLOGENTRY entry;
  int          nret = MGSEI_NO_ERROR;
  WCHAR        Number[256];
  SYSTEMTIME   st;
  //打开呼叫日志获取句柄。
        if( S_OK != PhoneOpenCallLog(&hCallLog) )
  {
            return MGSEI_API_CALL_ERROR;
  }
        //清空缓冲区准备获取日志信息
  ZeroMemory(&entry,sizeof(CALLLOGENTRY));
  entry.cbSize = sizeof(CALLLOGENTRY);
  entry.pszNumber = Number;
  //枚举获取呼叫日志
  while( S_OK == PhoneGetCallLogEntry(hCallLog,&entry) )
  {
    ShowText(entry.pszNumber);
    FileTimeToLocalFileTime(&entry.ftStartTime,&entry.ftStartTime);
    FileTimeToSystemTime(&entry.ftStartTime,&st);
    ShowST(st);
    FileTimeToLocalFileTime(&entry.ftEndTime,&entry.ftEndTime);
    FileTimeToSystemTime(&entry.ftEndTime,&st);
    ShowST(st);
  }
  //如果返回 ERROR_NO_MORE_ITEMS 说明已经没有条目了
  if(ERROR_NO_MORE_ITEMS == GetLastError())
  {
    ShowText(L"No More Items");      
  }
  //关闭呼叫日志句柄
  if( S_OK != PhoneCloseCallLog(hCallLog) )
  {
    return MGSEI_API_CALL_ERROR;
  }
  return 0;
}
```

构建好的程序见附件。通过弹出对话框显示用户所拨号码，开始时间，结束时间等信息。

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || CallLog01.rar |

