
# 如何选中其他程序 ListView 控件中的某个 Item

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-728.htm`](https://www.pediy.com/kssd/pediy07/pediy7-728.htm)

## 一：程序说明

题目是一位网友在我留言板上的留言，不知道大家看没看过我曾经写过的《如何向其他程序的 ListView 控件发送 LVM_GETITEMTEXT 消息》一文？在那篇拙文中，我的目的是得到某一 Item 的 TEXT。于是我将 LVITEM 结构插入到了目标进程中，才使得目标进程正确响应 LVM_GETITEMTEXT 消息。

要得到某一 Item 的 TEXT，需要发送 LVM_GETITEMTEXT 消息，而要选中某个 Item 则要发送 LVM_SETITEMSTATE 消息：

`LVM_GETITEMTEXT`
`wParam = (WPARAM) (int) iItem;`
`lParam = (LPARAM) (LV_ITEM FAR \*) pitem;`

`LVM_SETITEMSTATE`
`wParam = (WPARAM) (int) i;`
`lParam = (LPARAM) (LV_ITEM FAR \*) pitem;`

看到了吗？两个消息的参数一模一样！再来看看 LV_ITEM 结构的设置：只需将 state 和 stateMask 设置成 LVIS_SELECTED，并指定 iItem 即可。直接修改上篇拙文中的代码就能轻松搞定本篇要解决的问题。

作为演示，下面的这段程序将选中 TaskManager 中第 6 个项目。

## 二：具体实践

```cpp
//////////////////////////////////////////////////////////////////////////////////

/*
* Send LVM_SETITEMSTATE
* 版权所有 (C) 2005 天津 赵春生
* 2005.08.04
* http://timw.yeah.net
* http://timw.126.com
* 本程序适用于:Win2KP+SP4[Windows TaskManager(5.0.2195.6620)]
* WinXP+SP1[Windows TaskManager]
* 代码在 Win2000P+SP4 + VC6+SP6 测试通过
*/

#include<windows.h>

#include<commctrl.h>

int APIENTRY WinMain(HINSTANCE hInstance,
HINSTANCE hPrevInstance,
LPSTR lpCmdLine,
int nCmdShow)

{

HWND hwnd;

int iItem=0;

LVITEM lvitem, *plvitem;

DWORD PID;

HANDLE hProcess;

hwnd=FindWindow("#32770","Windows 任务管理器");

hwnd=FindWindowEx(hwnd,0,"#32770",0);

hwnd=FindWindowEx(hwnd,0,"SysListView32",0);

if (!hwnd)
    MessageBox(NULL,"[Windows 任务管理器] 尚未启动！","错误！",NULL);

else
{
    GetWindowThreadProcessId(hwnd, &PID);
    hProcess=OpenProcess(PROCESS_ALL_ACCESS,false,PID);

    if (!hProcess)
        MessageBox(NULL,"获取进程句柄操作失败！","错误！",NULL);
    else
    {
        plvitem=(LVITEM*)VirtualAllocEx(hProcess, NULL, sizeof(LVITEM), MEM_COMMIT, PAGE_READWRITE);

        if (!plvitem)
            MessageBox(NULL,"无法分配内存！","错误！",NULL);
        else
        {
            MessageBox(NULL,"本演示程序将选中 TaskManager 中的第 6 个项目。","提示",NULL);
            iItem=5;//5 在这里是第六个（从零开始）
            lvitem.state=LVIS_SELECTED;
            lvitem.stateMask=LVIS_SELECTED;

            WriteProcessMemory(hProcess, plvitem, &lvitem, sizeof(LVITEM), NULL);

            SendMessage(hwnd, LVM_SETITEMSTATE, (WPARAM)iItem, (LPARAM)plvitem);
        }
    }
}

//释放内存
CloseHandle(hwnd);
CloseHandle(hProcess);
VirtualFreeEx(hProcess, plvitem, 0, MEM_RELEASE);

return 0;

}

//////////////////////////////////////////////////////////////////////////////////
```

## 三：结论

以上代码在 Win2000P+SP4 + VC6+SP6 测试通过。

源码可从我的个人主页下载：

http://timw.yeah.net
http://timw.126.com

《如何向其他程序的 ListView 控件发送 LVM_GETITEMTEXT 消息》可通过搜索引擎获得。

18:53 2005-08-04

`http://timw_web.home4u.china.com/myzhuowen/sendlss/sendlss.zip`

`4dd466d0e1e0cc123afbe504da2462ca`
