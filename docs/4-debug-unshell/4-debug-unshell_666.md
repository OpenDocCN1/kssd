
# 简单分析一下 Execryptor One-touch 窗口的漏洞

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-564.htm`](https://www.pediy.com/kssd/pediy09/pediy09-564.htm)

先简单把 `Execryptor One-touch` 的窗口过程反一下

代码:

```c
#include <windows.h>
#pragma comment (linker, "/subsystem:windows")
#pragma comment (linker, "/entry:start")
#pragma comment (linker, "/filealign:0x200")

HFONT    hFont1;
HFONT    hFont2;
HWND     hEdit;
char     *szButtonName[5] = {"Evalute", "Order", "Order", "Enter Serial"};

LRESULT CALLBACK WndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam)
{
    int    i;
    HDC    hDC;
    RECT  rt;
    PAINTSTRUCT ps;

    switch (message)
    {
        case WM_CREATE:
            SendMessage(hWnd, WM_SETFONT, (WPARAM)hFont1, (LPARAM)TRUE);
            hEdit = CreateWindowEx(WS_EX_CLIENTEDGE,
                "EDIT",
                NULL,
                WS_CHILD|WS_TABSTOP|WS_VISIBLE|WS_EX_TOOLWINDOW,
                122,
                82,
                200,
                22,
                hWnd,
                (HMENU)100,
                GetModuleHandle(NULL),
                NULL
                );
            SendMessage(hEdit, WM_SETFONT, (WPARAM)hFont1, (LPARAM)TRUE);
            hEdit = CreateWindowEx(WS_EX_CLIENTEDGE,
                "EDIT",
                NULL,
                WS_CHILD|WS_TABSTOP|WS_VISIBLE|WS_EX_TOOLWINDOW|WS_EX_TOPMOST,
                122,
                112,
                200,
                22,
                hWnd,
                (HMENU)100,
                GetModuleHandle(NULL),
                NULL
                );
            SendMessage(hEdit, WM_SETFONT, (WPARAM)hFont1, (LPARAM)TRUE);
            SendMessage(hEdit, EM_LIMITTEXT, (WPARAM)20, (LPARAM)0);

            for (i=0; i<4; i++)
            {
                hEdit = CreateWindowEx(0,
                    "BUTTON",
                    szButtonName[i],
                    WS_CHILD|WS_TABSTOP|WS_VISIBLE,
                    8+i*80,
                    142,
                    75,
                    25,
                    hWnd,
                    (HMENU)(0x600+i),
                    GetModuleHandle(NULL),
                    NULL
                    );
                SendMessage(hEdit, WM_SETFONT, (WPARAM)hFont1, (LPARAM)TRUE);
            }
            break;

        case WM_DESTROY:
            PostQuitMessage(0);
            break;

        case WM_PAINT:
            hDC = BeginPaint(hWnd, &ps);
            SetBkColor(hDC, GetSysColor(COLOR_BTNFACE));
            SelectObject(hDC, hFont2);
            rt.left = 8;
            rt.top = 8;
            rt.right = 324;
            rt.bottom = 40;
            DrawText(hDC, "MyProduct 1.0", -1, &rt, DT_CENTER|DT_TOP|DT_SINGLELINE|DT_NOPREFIX);
            SelectObject(hDC, hFont1);
            rt.left = 8;
            rt.top = 36;
            rt.right = 328;
            rt.bottom = 54;
            DrawText(hDC, "Unregistered 0-days, 0-runs evaluation copy", -1, &rt, DT_CENTER|DT_TOP|DT_SINGLELINE|DT_NOPREFIX);
            rt.left = 8;
            rt.top = 58;
            rt.right = 324;
            rt.bottom = 80;
            DrawText(hDC, "You have 0 days and 0 runs left", -1, &rt, DT_CENTER|DT_TOP|DT_SINGLELINE|DT_NOPREFIX);
            rt.left = 8;
            rt.top = 86;
            rt.right = 150;
            rt.bottom = 104;
            DrawText(hDC, "Registration name:", -1, &rt, DT_LEFT|DT_TOP|DT_SINGLELINE|DT_NOPREFIX);
            rt.left = 8;
            rt.top = 116;
            rt.right = 150;
            rt.bottom = 134;
            DrawText(hDC, "Serial number:", -1, &rt, DT_LEFT|DT_TOP|DT_SINGLELINE|DT_NOPREFIX);
            EndPaint(hWnd, &ps);
            break;

        case WM_COMMAND:
            wParam -= 0x600;
            if (wParam == 0)  //点击 Evalute
            {
                SendMessage(hWnd, WM_CLOSE, 0, 0);
            }
            else if (wParam ==1 || wParam == 2) //点击 Order
            {
                ShellExecute(NULL, "open", "http://www.google.cn", NULL, NULL, SW_SHOWDEFAULT);
            }
            else if (wParam == 3) //点击 Enter Serial
            {
                MessageBox(hWnd, "For complete registration you must restart application.", "MyProduct 1.0", MB_OK|MB_ICONASTERISK|MB_TASKMODAL);
                ExitProcess(0);
            }
            break;

        default:
            return DefWindowProc(hWnd, message, wParam, lParam);
    }
    return 0;
}

void start()
{
    char    szClassName[13];
    int     i;
    WNDCLASSEX  wndclass;
    HWND    hWnd;
    MSG     msg;

    hFont1 = CreateFont(-12,
        0,
        0,
        0,
        FW_DONTCARE,
        FALSE,
        FALSE,
        FALSE,
        DEFAULT_CHARSET,
        OUT_DEFAULT_PRECIS,
        CLIP_DEFAULT_PRECIS,
        DEFAULT_QUALITY,
        VARIABLE_PITCH | FF_DONTCARE,
        "Arial"
        );

    hFont2 = CreateFont(-18,
        0,
        0,
        0,
        FW_DONTCARE,
        FALSE,
        FALSE,
        FALSE,
        DEFAULT_CHARSET,
        OUT_DEFAULT_PRECIS,
        CLIP_DEFAULT_PRECIS,
        DEFAULT_QUALITY,
        VARIABLE_PITCH | FF_DONTCARE,
        "Times New Roman"
        );

    srand(GetTickCount());
    for (i=0; i<12; i++)
    {
        szClassName[i] = 'a' + rand()%27;
    }
    szClassName[12] = 0;

    wndclass.cbSize = sizeof(WNDCLASSEX);
    wndclass.style = CS_NOCLOSE | CS_VREDRAW | CS_HREDRAW;
    wndclass.lpfnWndProc = WndProc;
    wndclass.cbClsExtra = 0;
    wndclass.cbWndExtra = 0;
    wndclass.hInstance = GetModuleHandle(NULL);
    wndclass.hIcon = NULL;
    wndclass.hCursor = LoadCursor(NULL, IDC_ARROW);
    wndclass.hbrBackground = HBRUSH(COLOR_BTNSHADOW);
    wndclass.lpszMenuName = NULL;
    wndclass.lpszClassName = szClassName;
    wndclass.hIconSm = NULL;
    if (RegisterClassEx(&wndclass) == 0)
    {
        ExitProcess(0);
    }

    hWnd = CreateWindowEx(WS_EX_DLGMODALFRAME | WS_EX_CONTROLPARENT,
        szClassName,
        NULL,
        WS_POPUP | WS_MAXIMIZEBOX | WS_VISIBLE,
        (GetSystemMetrics(SM_CXSCREEN)-336)/2,
        (GetSystemMetrics(SM_CYSCREEN)-178)/2,
        336,
        178,
        NULL,
        NULL,
        GetModuleHandle(NULL),
        NULL
        );
    if (hWnd == 0)
    {
        ExitProcess(0);
    }
    while (GetMessage(&msg, NULL, 0, 0))
    {
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }

    DeleteObject(hFont1);
    DeleteObject(hFont2);

    MessageBox(0, "Evalute Run", "Evalute", 0);

    ExitProcess(0);
}
```

---

试用期内窗口上一共有三种 Button

- 当按下 `Evalute` 时, `SendMessage(hWnd, WM_CLOSE, 0, 0)` 关闭窗口
- 当按下 `Order` 时, `ShellExecute` 弹出网页
- 当按下 `Enter Serial` 时, 将 `Name` 和 `Serial` 存入注册表并退出

过了试用期后 窗口上只有 `Order` 和 `Enter Serial` 两种 Button

因此我们只要人为的给窗口发送 `WM_CLOSE` 消息

代码:

```c
SendMessage(hWnd, WM_CLOSE, 0, 0)
```

---

或者

代码:

```c
SendMessage(hWnd, WM_SYSCOMMAND, SC_CLOSE, 0)
```

---

就相当于手工点了 `Evalute` 的 Button

这个漏洞在最新的 2.4.1 版本仍然存在

作者真是偷懒

下在说说没意义的 Patch 方法

不使用 `Evalute`, 也就是加壳时不让试用, 同时找到相关的 `stub` (在 `Execrypt.exe` 里, 不难找)

把最后的 `DeleteObject(hFont1)` 改成 `ExitProcess(hFont1)` 就可以了

其实最好还是由作者自己修正, 只要在处理 `Evalute` 的地方多加点东西,

使得 `GetMessage` 的循环出来后有东西不一样就可以了

本文没有技术含量
