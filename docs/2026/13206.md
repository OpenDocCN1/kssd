
# [分享]给`ZeroAdd`加入拖放文件的功能

`ZeroAdd`([3c4K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8W2k6r3W2&6i4K6u0W2j5$3!0E0i4K6u0r3N6r3!0G2L8s2y4Q4x3V1k6b7c8g2)9#2k6Y4c8G2L8$3I4K6i4K6u0r3M7$3g2U0N6r3W2G2L8W2)9J5c8W2A6W2M7X3!0Q4x3U0f1J5x3p5q4V1k6q4)9J5c8Y4A6W2M7X3!0S2k6r3c8Q4x3U0f1J5x3o6q4Q4x3X3f1H3i4K6u0W2P5X3W2H3](https://bbs.kanxue.com/elink@1b6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8W2k6r3W2&6i4K6u0W2j5$3!0E0i4K6u0r3N6r3!0G2L8s2y4Q4x3V1k6b7c8g2)9#2k6Y4c8G2L8$3I4K6i4K6u0r3M7$3g2U0N6r3W2G2L8W2)9J5c8W2A6W2M7X3!0Q4x3U0f1J5x3p5q4V1k6q4)9J5c8Y4A6W2M7X3!0S2k6r3c8Q4x3U0f1J5x3o6q4Q4x3X3f1H3i4K6u0W2P5X3W2H3))用来增加PE文件的区块很方便，但它不支持文件拖放，不爽。

经过修改使`ZeroAdd`实现了拖放操作PE文件的功能。

要使窗口支持拖放功能须在窗口创建后，先调用下面的函数：

```
VOID DragAcceptFiles(
    HWND hWnd,        // handle to the registering window
    BOOL fAccept        // acceptance option
   );
```

`hWnd`为窗口的句柄

`fAccept`为`TRUE`时表示允许拖放，`FALSE`表示禁止拖放

当窗口收到`WM_DROPFILES`消息时，通过`DragQueryFileA`可以得到拖放的文件名。

完了调用`DragFinish`结束拖放。

具体实现：

因为要用到的三个函数`DragAcceptFiles`、`DragQueryFileA`、`DragFinish`原程序中没有，所以先用IID King加入此三个API函数。此三个函数都在`shell32.dll`中。

IID King给程序增加了一个1kB的SECTION.我们增加的程序可以写在空白的地方。文件偏移`8200h`(`VA:40b100`)往后的地方都没有用到，我们的程序就写在这里。

修改`WM_INITDIALOG`的默认处理：

```
.text:0040101F                 push    ebp
.text:00401020                 mov     ebp, esp
.text:00401022                 cmp     [ebp+uMsg], 110h=WM_INITDIALOG
.text:00401029                 jnz     short fwd_401030
.text:0040102B                 jmp     InitDlg=40b100        ;修改此行跳转

InitDlg:
40b100:
        push        1
        push        d,[ebp][8]
        api call        DragAcceptFiles
        ;sinvoke DragAcceptFiles/[ebp+8]/1
        va jmp        40148c

.text:00401045                 cmp     [ebp+uMsg], WM_COMMAND
.text:0040104C                 jnz     Other_Msg=40b110        ;修改此行跳转
.text:00401052                 mov     eax, [ebp+wParam]
.text:00401055                 cmp     ax, 2
.text:00401059                 jnz     short loc_4010B3
.text:0040105B                 shr     eax, 10h
.text:0040105E                 or      ax, ax

Other_Msg:
40b110:
        cmp        d,[ebp+0ch],233h        ;233hWM_DROPFILES
        va jnz        401483h
        push        200h
        push        40337ch
        push        0
        push        d,[ebp+10h]
        api call        DragQueryFileA
        push        40337ch
        push        3
        push        d,[ebp+8]
        api call        SetDlgItemTextA
        push        d,[ebp+10h]
        api call        DragFinish
        ;sinvoke        DragQueryFileA,wParam,0,addr buffer,512
        ;sinvoke        SetDlgItemText,hWnd,IDCT_EDIT,addr buffer
        ;sinvoke        DragFinish,wParam
        va jmp        40148ch
```

推荐使用`MyHiew`修改，先在`40b100`处输入`InitDlg`和`Other_Msg`的代码，然后分别修改`0040102B`和`0040104C`处的跳转。

已修改过的`ZeroAdd`可通过如下地址下载：

[78cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8U0M7r3q4Y4k6i4y4Q4x3X3g2U0L8$3#2Q4x3V1k6H3K9i4y4U0k6i4y4Q4x3V1k6H3k6h3c8A6P5g2)9J5c8Y4m8S2N6r3y4Z5k6h3c8Q4y4h3k6\*7k6i4u0G2j5h3c8V1i4K6u0W2P5X3W2H3](https://bbs.kanxue.com/elink@4faK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8U0M7r3q4Y4k6i4y4Q4x3X3g2U0L8$3#2Q4x3V1k6H3K9i4y4U0k6i4y4Q4x3V1k6H3k6h3c8A6P5g2)9J5c8Y4m8S2N6r3y4Z5k6h3c8Q4y4h3k6*7k6i4u0G2j5h3c8V1i4K6u0W2P5X3W2H3)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
