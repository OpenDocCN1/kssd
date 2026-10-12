
# 原创-《向其他程序的 ListView 控件发送 LVM\_GETITEMTEXT》

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-727.htm`](https://www.pediy.com/kssd/pediy07/pediy7-727.htm)

## 一：问题的由来与解决方法

首先给大家介绍一个我编写的小程序——`CallFI`(Call FileInfo)，它与 2K/XP 中的 `taskmgr.exe`(Windows TaskManager)程序配合使用。它能获取指定进程的路径并依此调用 `FileInfo`(文件类型分析工具)程序，这样你就能更加全面得了解这个进程了。

当你选中一个进程并运行 `CallFI` 后，`CallFI` 首先向 `taskmgr.exe` 中的 ListView 控件发送 `LVM_GETNEXTITEM` 消息来确定你所选定的是哪个 `index`，然后再发送 `LVM_GETITEMTEXT` 消息获取相应的值（`PID`），以便根据该 `PID` 进行后面的一系列操作。`index` 很顺利得就得到了，但当我直接向 ListView 控件发送 `LVM_GETITEMTEXT` 消息时却无法得到我想要的返回值！这就是本文所要解决的问题。

在 GOOGLE 上我找到了 `Cory Nelson` 写的《Stealing Program's Memory》一文，问题的关键在于：利用 `WriteProcessMemory()`函数将 `LVITEM` 结构插入到目标进程中……

作为演示，下面的这段程序将显示在 `TaskManager` 中指定进程的进程名。

## 二：具体实践

```cpp
//////////////////////////////////////////////////////////////////////////////////
/*
* Send_LGT
* 版权所有 (C) 2004 赵春生
* 2004.08.08
* http://timw.yeah.net
* http://timw.126.com
* 本程序适用于:Win2KP+SP4[Windows TaskManager(5.0.2195.6620)]
*               WinXP+SP1[Windows TaskManager]
* 代码在 Win2000P+SP4 + VC6+SP5 测试通过
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
char ItemBuf[512],*pItem;
DWORD PID;
HANDLE hProcess;

hwnd=FindWindow("#32770","Windows 任务管理器");
hwnd=FindWindowEx(hwnd,0,"#32770",0);
hwnd=FindWindowEx(hwnd,0,"SysListView32",0);

if (!hwnd)
MessageBox(NULL,"[Windows 任务管理器] 尚未启动！","错误！",NULL);
else
{
iItem=SendMessage(hwnd,LVM_GETNEXTITEM,-1,LVNI_SELECTED);

if (iItem==-1)
MessageBox(NULL,"没有指定目标进程 ！","错误！",NULL);
else
{
//itoa(iItem,temp,10);
//MessageBox(NULL,temp,"iItem",NULL);

GetWindowThreadProcessId(hwnd, &PID);
hProcess=OpenProcess(PROCESS_ALL_ACCESS,false,PID);

if (!hProcess)
MessageBox(NULL,"获取进程句柄操作失败！","错误！",NULL);
else
{
plvitem=(LVITEM*)VirtualAllocEx(hProcess, NULL, sizeof(LVITEM), MEM_COMMIT, PAGE_READWRITE);
pItem=(char*)VirtualAllocEx(hProcess, NULL, 512, MEM_COMMIT, PAGE_READWRITE);

if ((!plvitem)||(!pItem))
MessageBox(NULL,"无法分配内存！","错误！",NULL);
else
{
lvitem.cchTextMax=512;
//lvitem.iSubItem=1;//PID
lvitem.iSubItem=0; //ProcessName
lvitem.pszText=pItem;

WriteProcessMemory(hProcess, plvitem, &lvitem, sizeof(LVITEM), NULL);
SendMessage(hwnd, LVM_GETITEMTEXT, (WPARAM)iItem, (LPARAM)plvitem);
ReadProcessMemory(hProcess, pItem, ItemBuf, 512, NULL);
MessageBox(NULL,ItemBuf,"ItemBuf",NULL);
}
}
}
}

//释放内存
CloseHandle(hwnd);
CloseHandle(hProcess);
VirtualFreeEx(hProcess, plvitem, 0, MEM_RELEASE);
VirtualFreeEx(hProcess, pItem, 0, MEM_RELEASE);

return 0;
}
//////////////////////////////////////////////////////////////////////////////////
```

## 三：以上代码在 Win2000P+SP4 + VC6+SP5 测试通过

源码可从我的个人主页下载。

`http://timw.yeah.net`

`http://timw.126.com`

`Cory Nelson` 写的《Stealing Program's Memory》：

`http://www.codeproject.com/threads/int64_memsteal.asp`

20:09 2004-08-08

`http://timw_web.home4u.china.com/myzhuowen/sendlgt/sendlgt.zip`

`4dd466d0e1e0cc123afbe504da2462ca`
