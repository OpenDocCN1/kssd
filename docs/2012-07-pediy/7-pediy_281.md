
# 给 ZeroAdd 加入拖放文件的功能

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-431.htm`](https://www.pediy.com/kssd/pediy07/pediy7-431.htm)

`ZeroAdd(http://www.pediy.com/tools/PEtools.htm)`用来增加 PE 文件的区块很方便，但它不支持文件拖放，不爽。  

经过修改使 `ZeroAdd` 实现了拖放操作 PE 文件的功能。  

要使窗口支持拖放功能须在窗口创建后，先调用下面的函数：  

```
VOID DragAcceptFiles(  
    HWND hWnd,  // handle to the registering window  
    BOOL fAccept  // acceptance option  
   );  
```

`hWnd` 为窗口的句柄  

`fAccept` 为 `TRUE` 时表示允许拖放，`FALSE` 表示禁止拖放  

当窗口收到 `WM\_DROPFILES` 消息时，通过 `DragQueryFileA` 可以得到拖放的文件名。  

完了调用 `DragFinish` 结束拖放。  

具体实现：  

因为要用到的三个函数 `DragAcceptFiles`,`DragQueryFileA`,`DragFinish` 原程序中没有，所以先用 `IID King` 加入此三个 API 函数。此三个函数都在 `shell32.dll` 中。  

`IID King` 给程序增加了一个 1kB 的 SECTION.我们增加的程序可以写在空白的地方。文件偏移 `8200h(VA:40b100)` 往后的地方都没有用到，我们的程序就写在这里。  

修改 `WM\_INITDIALOG` 的默认处理：  

```
.text:0040101F            push    ebp  
.text:00401020            mov     ebp, esp  
.text:00401022            cmp     [ebp+uMsg], 110h=WM_INITDIALOG  
.text:00401029            jnz     short fwd_401030  
.text:0040102B            jmp     InitDlg=40b100  ;修改此行跳转  
```

`InitDlg:`  

`40b100:`  

`push  1`  
`push  d,[ebp][8]`  
`api call  DragAcceptFiles`  
`;sinvoke DragAcceptFiles/[ebp+8]/1`  
`va jmp  40148c`  

```
.text:00401045            cmp     [ebp+uMsg], WM_COMMAND  
.text:0040104C            jnz     Other_Msg=40b110  ;修改此行跳转  
.text:00401052            mov     eax, [ebp+wParam]  
.text:00401055            cmp     ax, 2  
.text:00401059            jnz     short loc_4010B3  
.text:0040105B            shr     eax, 10h  
.text:0040105E            or      ax, ax  
```

`Other_Msg:`  

`40b110:`  

`cmp  d,[ebp+0ch],233h  ;233hWM_DROPFILES`  
`va jnz  401483h`  
`push  200h`  
`push  40337ch`  
`push  0`  
`push  d,[ebp+10h]`  
`api call  DragQueryFileA`  
`push  40337ch`  
`push  3`  
`push  d,[ebp+8]`  
`api call  SetDlgItemTextA`  
`push  d,[ebp+10h]`  
`api call  DragFinish`  
`;sinvoke  DragQueryFileA,wParam,0,addr buffer,512`  
`;sinvoke  SetDlgItemText,hWnd,IDCT_EDIT,addr buffer`  
`;sinvoke  DragFinish,wParam`  
`va jmp  40148ch`  

推荐使用 `MyHiew` 修改，先在 `40b100` 处输入 `InitDlg` 和 `Other_Msg` 的代码，然后分别修改 `0040102B` 和 `0040104C` 处的跳转。  

已修改过的 `ZeroAdd` 可通过如下地址下载：  

[`www.pcpages.com/pisces/pediy/patched\_zeroadd.zip`](http://www.pcpages.com/pisces/pediy/patched_zeroadd.zip)  
`4dd466d0e1e0cc123afbe504da2462ca`
