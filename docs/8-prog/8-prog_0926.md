
# 手写进程管理器

> 原文：[`www.pediy.com/kssd/pediy12/138701.html`](https://www.pediy.com/kssd/pediy12/138701.html)

## 最近在学 MFC

自己手工的进程管理器最近在学 MFC。以前习惯了向导，总感觉有哪里不对劲。现在知道了是对框架的不熟悉引起的。狠啃了好几天框架，艰深无比，看得脑大。写个程序练练手，算是应用吧。程序运行效果如下:

![](img/3c18758db2ca2578bb246ad19e9b23aa.jpg)

![](img/26854bdc6bf542fbf0e842efd9f32061.jpg)

## 创建工程与初始化

创建 `W32` 工程，添加资源脚本。使用通用控件要加载 `comctl32.dll` 库，`#pragma comment(lib,"comctl32.lib")`。使用前要初始化，调用 `InitCommonControlsEx()`。用 `ToolHelp` 函数列出当前正在运行的进程，要先声明 `#include<tlhelp32.h>`。首先用 `CreateToolHelp32Snapshot` 拍快照，再用 `Process32First`，`Process32Next` 遍历快照表。代码如下

```cpp
#include <windows.h>

#include <commctrl.h>

#include <tlhelp32.h>

#include "resource.h"

// 链接到 comctl32.lib 库

#pragma comment(lib,"comctl32.lib")

// 状态栏 ID 号

#define IDC_STATUS 101

BOOL __stdcall DlgProc(HWND, UINT, WPARAM, LPARAM);

void UpdateProcess(HWND hWndList);

int __stdcall WinMain(HINSTANCE hInstance, HINSTANCE, LPSTR, int)

{

  // 初始化 Comctl32.dll 库

  ::InitCommonControls();

  ::DialogBoxParam(hInstance, (LPCTSTR)IDD_DIALOG1, NULL, DlgProc, NULL);

  return 0;

}

BOOL __stdcall DlgProc(HWND hDlg, UINT message, WPARAM wParam, LPARAM lParam)

{

  switch(message)

  {

  case WM_INITDIALOG:

    {

      // 初始化列表视图控件

      HWND hWndList = ::GetDlgItem(hDlg, IDC_LIST2);

      // 设置它的扩展风格

      ::SendMessage(hWndList, LVM_SETEXTENDEDLISTVIEWSTYLE,

                    0, LVS_EX_FULLROWSELECT|LVS_EX_GRIDLINES);

      LVCOLUMN column;

      // 指定 LVCOLUMN 结构中的 pszText、fmt、cx 域有效

      column.mask = LVCF_TEXT|LVCF_FMT|LVCF_WIDTH;

      // 设置有效的域的属性

      column.fmt = LVCFMT_CENTER;  // 指定文本居中显示

      column.cx = 100;    // 指定此栏的宽度

      column.pszText = "映象名称";  // 指定此栏显示的文本

      // 添加一个新的专栏

      ::SendMessage(hWndList, LVM_INSERTCOLUMN, 0, (LPARAM)&column);

      // 再添加一个专栏

      column.pszText = "PID";

      column.cx = 50;

      ::SendMessage(hWndList, LVM_INSERTCOLUMN, 1, (LPARAM)&column);

      // 初始化状态栏

      // 创建状态栏

      HWND hWndStatus = ::CreateStatusWindow(WS_CHILD|WS_VISIBLE|SBS_SIZEGRIP,

                    NULL, hDlg, IDC_STATUS);

      // 设置背景色

      ::SendMessage(hWndStatus, SB_SETBKCOLOR, 0, RGB(0xa6, 0xca, 0xf0));

      // 给状态栏分栏

      int pInt[] = { 152, -1 };

      ::SendMessage(hWndStatus, SB_SETPARTS, 2, (long)pInt);

      // 设置各栏的文本

      ::SendMessage(hWndStatus, SB_SETTEXT, 0, (long)" 以上进程");

      ::SendMessage(hWndStatus, SB_SETTEXT, 1, (long)" 学习中！");

      // 刷新进程列表

      UpdateProcess(hWndList);

    }

    break;

  case WM_COMMAND:

    switch(LOWORD(wParam))

    {

    case IDC_BUT_EXIT:

      ::EndDialog(hDlg, IDOK);

      break;

    case IDCANCEL:

      ::EndDialog(hDlg, IDCANCEL);

      break;

    case IDC_BUT_XIN:

      UpdateProcess(::GetDlgItem(hDlg, IDC_LIST2));

      break;

    }

    break;

  case WM_NOTIFY:

    {

      if(wParam == IDC_LIST2)

      {

        NMHDR* pHeader = (NMHDR*)lParam;

        HWND hWndList = pHeader->hwndFrom;

        if(pHeader->code == NM_DBLCLK)  // 双击事件

        {

          NMLISTVIEW* pNMListView = (NMLISTVIEW*)pHeader;

          // 用户双击的项号

          int nIndex = pNMListView->iItem;

          // 取得进程 ID 号

          char szID[56];

          LVITEM lvi;

          memset(&lvi, 0, sizeof(LVITEM));

          lvi.iSubItem = 1;  // nIndex 项目中的第 1 个子项

          lvi.cchTextMax = 56;

          lvi.pszText = szID;

          ::SendMessage(hWndList, LVM_GETITEMTEXT, (WPARAM)nIndex, (long)&lvi);

          // 询问用户

          if(::MessageBox(hDlg, "确实要终止进程吗？",

            "07ComctlDemo", MB_OKCANCEL|MB_DEFBUTTON2) == IDCANCEL)

            return 0;

          // 试图打开目标进程，终止它

          HANDLE hProcess = ::OpenProcess(PROCESS_TERMINATE, FALSE, atoi(szID));

          if(hProcess != NULL)

          {

            HWND hWndStatus = ::GetDlgItem(hDlg, IDC_STATUS);

            if(::TerminateProcess(hProcess, 0))

            {

              ::SendMessage(hWndStatus, SB_SETTEXT, 0, (long)"你终止了我！");

              UpdateProcess(hWndList);

            }

            else

            {

              ::SendMessage(hWndStatus, SB_SETTEXT, 0, (long)"进程不让你终止！");

            }

          }

        }

      }

    }

    break;

  }

  return 0;

}

void UpdateProcess(HWND hWndList)

{

  // 删除所有的项

  ::SendMessage(hWndList, LVM_DELETEALLITEMS, 0, 0);

  int nItem = 0;  // 项计数

  PROCESSENTRY32 pe32 = { sizeof(PROCESSENTRY32) };

  HANDLE hProcessSnap = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);

  if(hProcessSnap == INVALID_HANDLE_VALUE)

    return;

  if(Process32First(hProcessSnap, &pe32))

  {

    do

    {

      // 取得进程 ID 号

      char szID[56];

      wsprintf(szID, "%u", pe32.th32ProcessID);

      // 插入一个项

      LVITEM item = { 0 };

      item.iItem = nItem;

      item.mask = LVIF_TEXT;      // 指定 pszText 域有效

      item.pszText = (LPTSTR)pe32.szExeFile;  // 设置文本

      ::SendMessage(hWndList, LVM_INSERTITEM, 0, (long)&item);

      // 设置新项的文本

      LVITEM lvi;

      lvi.iSubItem = 1;    // 指定要设置第 1 个专栏的文本

      lvi.pszText = (LPTSTR)szID;  // 要设置的文本

      ::SendMessage(hWndList, LVM_SETITEMTEXT, nItem, (LPARAM)&lvi);

      nItem++;

    }

    while(Process32Next(hProcessSnap, &pe32));

  }

  ::CloseHandle(hProcessSnap);

}
```
