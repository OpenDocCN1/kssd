
# WinZip Nag

> 原文：[`www.pediy.com/kssd/pediy04/kanxue031.htm`](https://www.pediy.com/kssd/pediy04/kanxue031.htm)

记得论坛上曾经有人说过一种破解 NAG 的方法：在 NAG 弹出来以后，给可以关闭它的 `BUTTON` 送一个 `WM_COMMAND` 消息。思路早就有了，可是具体实现的方法还没看见过有人写（不过我看过一篇 E 文的文章专门讲这个的，讲得很好。看来中国的 `CRACKER` 们写东西的干劲不太足呀！^\_^）。前天借同学买的《电脑爱好者》2001 年上半年合订本看的时候，看到有篇文章就是写这个的，文章题目叫：《Winzip 赖帐器》，有兴趣的朋友可以自己去找找看看，那篇文章是 `VC++` 写的，我们 `CRACKER` 们最喜欢的还是 `ASM` 吧？所以我写了一个 `ASM` 的版本，给大家参考参考（大虾们不要笑我不自量力哟！）。

首先分析一下程序的作用：假设我们程序的名称叫做 `loader.exe`。我们把它放在 `Winzip` 的目录中，然后在资源管理器中设定 `zip` 文件的打开方式为：`loader.exe`。当我们双击一个 `zip` 文件时，`loader.exe` 获取要解压的 `zip` 文件的文件名，并以此为命令行参数传递给 `winzip32.exe`，由于我们使用的 `winzip` 是未注册版本，所以 `winzip32.exe` 一执行，就会出来一个 NAG，让我们选择 "I Agree"，或者是 "Quit"，我们程序的任务就是向 "I Agree" 按钮发送一个 `WM_COMMAND` 消息，以此消除 NAG。

好了，程序的功用我们知道了，下面我们就要分析一下用什么函数可以达到我们的目的：

要发送一个 `WM_COMMAND` 消息给一个 `BUTTON`，我们必须知道两个值：按钮所在窗口的 `HWND` 和按钮的 `ID`，这样我们就可以用 `invoke SendMessage, HWND, WM_COMMAND, ID, 0` 达到给它发送消息的目的。按钮的 `ID` 很好办，执行 `WINZIP`，用 `SPY` 看一下就可以了。我得出的结果是：左边的按钮的 `ID` 是 2901，右边的按钮的 `ID` 是 2902，下面我们就要得到窗口的 `HWND` 了，取得 `HWND`，最常用的方法是 `FindWindow`，你只要知道窗口的 caption，用该函数就可以得到该窗口的 `HWND`，但是，`WINZIP` 执行时，除开开始的那个 NAG 外，还有一个不可见的也以 `WINZIP` 为 CAPTION 的窗口，所以，我们得到 `HWND` 后，还要用 `GetWindowLong` 获得该窗口的 `GWL_ID`，如果是 0，那么我们就算找到了我们要找的窗口了（用 `SPY` 可以得知 NAG 的 `ID` 是 0）。

好了，大致的流程定下来了，下面我们还有一个工作要做：判断到底是 2901 是我们要得 `BUTTON`，还是 2902 是我们要的 `BUTTON`。跟踪一下知道，按钮上的文字是 `DrawTextA` 画上去的，所以，我们还要写一段 gdi 函数。

问题都解决了，下面看看该程序的源代码吧：

```
----------------------------------------Cut From Here--------------------------------------

    .386
    .model flat, stdcall
    option casemap :none
include    windows.inc
include    gdi32.inc
include    kernel32.inc
include    user32.inc
includelib    gdi32.lib
includelib    kernel32.lib
includelib    user32.lib

    .data
szDirect    db    100 dup (?)
szCmdline    db    100 dup (?)
szExename    db    'WinZip32.exe'
szFile    db    50 dup (0)
szLoad    db    'loader.exe',0
szWinzip    db    'WinZip',0
szSubbuffer db    20 dup (0)
stStartup    STARTUPINFO   <?>
stProcinfo    PROCESS_INFORMATION   <?>
rcMyrect    RECT   <?>
szFlag    dd    ?
hWinzip    dd    ?
dcMydc    dd    ?

    .code
start:

    mov    dword ptr [stStartup],44h
    call    GetCommandLine
    mov    esi,eax
    xor    eax,eax
    invoke    lstrlen,esi
    mov    ecx,eax
    invoke    SearchPath,NULL,offset szLoad,NULL,100,offset szDirect,offset szFlag
    add    esi,eax
    mov    edi,offset szFile
    invoke    lstrlen,esi
    mov    ecx,eax
    rep    movsb
    invoke    CreateProcess,0,offset szExename,0,0,0,20h,0,0,offset stStartup,offset stProcinfo

find:
    invoke    FindWindow,NULL,offset szWinzip
    or    eax,eax
    jz    find
    mov    hWinzip,eax
    invoke    GetWindowLong,eax,GWL_ID
    cmp    eax,0
    jnz    find
    invoke    SetWindowPos,hWinzip,HWND_TOPMOST,0,0,0,0,SWP_NOSIZE OR SWP_NOMOVE
    invoke    GetWindowRect,hWinzip,offset rcMyrect
    invoke    GetDC,0
    mov    dcMydc,eax
    invoke    GetPixel,dcMydc,384,402
    .if    eax==0
        invoke    SendMessage,hWinzip,WM_COMMAND,2902,0
    .else
        invoke    SendMessage,hWinzip,WM_COMMAND,2901,0
    .endif
    invoke    ExitProcess,NULL
end start

---------------------------------------------Cut End-----------------------------------------
```

The End:

aNY qUESTION, eMAIL tO: luoyi.ly@yeah.net

Thanks For Your Reading 4dd466d0e1e0cc123afbe504da2462ca
