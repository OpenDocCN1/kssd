
# 《如何更改其他程序 ListView 控件中某个 Item 的内容》

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-781.htm`](https://www.pediy.com/kssd/pediy07/pediy7-781.htm)

## 一：程序说明

这次我将介绍如何更改其他程序 ListView 控件中某个 Item 的内容，关于类似的拙文我已经写了两篇，这是第三篇，本篇和第一篇《如何向其他程序的 ListView 控件发送 LVM_GETITEMTEXT 消息》类似，区别在于：

1. 发送的消息不同：前者是读取 `pszText` 的内容――发送 `LVM_GETITEMTEXT`；这次是设置 `pszText`，应该发送 `LVM_SETITEMTEXT`。
2. 字符串缓冲区的作用不同：前者 `pItem` 用来接收 `ITEMTEXT`，我们可以通过 `ReadProcessMemory` 函数来读取其内容；而在本篇中 `p_MyItemText` 则用来存放我们要设置的 `ITEMTEXT`，并用 `WriteProcessMemory` 函数将其写入到目标程序中。

作为演示，下面的这段程序将更改 TaskManager 中第 6 个项目中第 1 列的内容。程序运行后的效果图：

![](img/f1f40e9df3548f004aafd57e2aa82e64.jpg)

## 二：具体实践

```c
//////////////////////////////////////////////////////////////////////////////////

/*
 * Send LVM_SETITEMTEXT
 * 版权所有 (C) 2005 天津 赵春生
 * 2005.10.28
 * http://timw.yeah.net
 * http://timw.126.com
 * 本程序适用于:Win2KP+SP4[Windows TaskManager(5.0.2195.6620)]
 * WinXP+SP1[Windows TaskManager]
 * 代码在 Win2000P+SP4 + VC6+SP6 测试通过
 */

#include <windows.h>
#include <commctrl.h>

int APIENTRY WinMain(HINSTANCE hInstance,
                     HINSTANCE hPrevInstance,
                     LPSTR lpCmdLine,
                     int nCmdShow)
{
    HWND hwnd;
    int iItem = 0;
    LVITEM lvitem, *plvitem;
    DWORD PID;
    HANDLE hProcess;
    char *p_MyItemText;//目标程序中用来存放 TEXT 的地址

    //The pszText member is the pointer to a null-terminated
    // string containing the new text; it can also be NULL.
    //以上信息是从 API 手册中获得的，故在本例中字符串长度不能>=12，
    //以确保字符串后有 NULL。
    char str_MyItemText[12] = {0};

    strcpy(str_MyItemText, "天津 赵春生");
    hwnd = FindWindow("#32770", "Windows 任务管理器");
    hwnd = FindWindowEx(hwnd, 0, "#32770", 0);
    hwnd = FindWindowEx(hwnd, 0, "SysListView32", 0);

    if (!hwnd)
        MessageBox(NULL, "[Windows 任务管理器] 尚未启动！", "错误！", NULL);
    else
    {
        GetWindowThreadProcessId(hwnd, &PID);
        hProcess = OpenProcess(PROCESS_ALL_ACCESS, false, PID);

        if (!hProcess)
            MessageBox(NULL, "获取进程句柄操作失败！", "错误！", NULL);
        else
        {
            plvitem = (LVITEM *)VirtualAllocEx(hProcess, NULL, sizeof(LVITEM), MEM_COMMIT, PAGE_READWRITE);
            p_MyItemText = (char *)VirtualAllocEx(hProcess, NULL, 12, MEM_COMMIT, PAGE_READWRITE);

            if ((!plvitem) || (!p_MyItemText))
                MessageBox(NULL, "无法分配内存！", "错误！", NULL);
            else
            {
                MessageBox(NULL, "本演示程序将更改 TaskManager 中第 6 个项目中第 1 列的内容。", "提示", NULL);
                iItem = 5;//5 在这里是第六个（从零开始）
                lvitem.iSubItem = 0;//同上
                lvitem.pszText = p_MyItemText;

                WriteProcessMemory(hProcess, p_MyItemText, &str_MyItemText, 12, NULL);
                WriteProcessMemory(hProcess, plvitem, &lvitem, sizeof(LVITEM), NULL);

                //向目标程序发送 LVM_SETITEMTEXT 消息
                SendMessage(hwnd, LVM_SETITEMTEXT, (WPARAM)iItem, (LPARAM)plvitem);
            }
        }
    }

    //释放内存
    CloseHandle(hwnd);
    CloseHandle(hProcess);
    VirtualFreeEx(hProcess, plvitem, 0, MEM_RELEASE);
    VirtualFreeEx(hProcess, p_MyItemText, 0, MEM_RELEASE);

    return 0;
}
//////////////////////////////////////////////////////////////////////////////////
```

## 三：以上代码在 Win2000P+SP4 + VC6+SP6 测试通过

源码可从我的个人主页下载。

http://timw.yeah.net
http://timw.126.com

《如何向其他程序的 ListView 控件发送 LVM_GETITEMTEXT 消息》，
《如何选中其他程序 ListView 控件中的某个 Item》：

可通过搜索引擎获得。

23:06 2005-10-28

http://timw_web.home4u.china.com/myzhuowen/sendlst/sendlst.zip

4dd466d0e1e0cc123afbe504da2462ca
