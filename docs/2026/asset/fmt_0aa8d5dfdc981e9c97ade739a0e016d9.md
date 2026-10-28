
```cpp
return hLastWin;

}

// 举例: HWND hLastWin = MyFindWindow();

HWND MyInputWindow()

{

    const int MyMaxParentWinCount = 9;

    // 父窗口类名数组
    char *A_szClassName[MyMaxParentWinCount] =
    {
        "TForm1",
        "TPageControl",
        "TTabSheet",
        "TPageControl",
        "TTabSheet",
        "TPageControl",
        "TTabSheet",
        "TGroupBox",
        "TEdit"
    };

    // 父窗口标题数组
    char *A_szWinName[MyMaxParentWinCount] =
    {
        "Keygener_Assistant v2.1.2 - Crypto Operations : Symmetric & Asymmetric",
        "",
        " Encryption ",
        "",
        " Symmetric ",
        "",
        "File Encryption",
        " Encrypt Data ",
        ""
    };

    // 首先求得顶级父窗口
    HWND hLastWin = FindWindow(A_szClassName[0], A_szWinName[0]);

    // 逐次用FindWindowEx函数求出各级子窗口
    for(int i=1; i<MyMaxParentWinCount; i++)
    {
        hLastWin = FindWindowEx(hLastWin, NULL, A_szClassName[i], A_szWinName[i]);
        switch (i)
        {
            case 1: SendMessage(hLastWin, WM_LBUTTONDOWN, 0, 0X000a0113); Sleep(100); SendMessage(hLastWin, WM_LBUTTONUP, 0, 0X000a0113); //找到这个窗口时模拟点击Hashing
                break;
            case 3: SendMessage(hLastWin, WM_LBUTTONDOWN, 0, 0X000b0026); Sleep(100); SendMessage(hLastWin, WM_LBUTTONUP, 0, 0X000b0026); //找到这个窗口时模拟点击Calc Hash
                break;
            case 5: SendMessage(hLastWin, WM_LBUTTONDOWN, 0, 0X000b009e); Sleep(100); SendMessage(hLastWin, WM_LBUTTONUP, 0, 0X000b009e); //找到这个窗口时模拟点击File Hash
                break;
        }
    }

    return hLastWin;
}
```

以上是老妖的SPY4WIN生成的查找窗口代码,特别好用。

附件为成品.放到EXE相同目录下即可.

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [`winspool.rar`](javascript:void(0);)

  （17.88kb，39次下载）
