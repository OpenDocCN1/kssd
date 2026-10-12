
# Delphi 逆向工程笔记\[2\] 

> 原文：[`www.pediy.com/kssd/pediy06/pediy6595.htm`](https://www.pediy.com/kssd/pediy06/pediy6595.htm)

自己写的破烂居然被加精了……这里做个说明，由于这个 `KeyGen` 的 `RE` 有点难度，目前手上完成了的也就一点点，够写 3 篇。以后的要等我慢慢分析了。不过我保证，一定会全部 `RE` 出来的，而且在 `PEDIY` 发全。

`PS`：本来是要发到 `CSDN` 的 `Blog` 上的，不过那个 `Blog` 有点烂，发不上去。于是就扔到 `PEDIY` 来了。以为不会有人注意，结果居然被最想躲开的人看见了……实际上某人的猜想是对的。他写了某篇文章给我看到了，于是我就拿了某个程序开刀，幸运的是那个程序没有用某些软件处理过，要不就没有这篇破烂了。莫怪莫怪。

`=================================================================================`

下面处理 `LicenseProc`，就是 `sub_404DD4`。为了便于查看，这里先把主框架 `rip` 出来。

```
0000:00404DD4  sub_404DD4      proc near  
0000:00404DD4  
0000:00404DD4  Paint           = PAINTSTRUCT ptr -48h  
0000:00404DD4  pt              = POINT ptr -8  
0000:00404DD4  hWnd            = dword ptr  8  
0000:00404DD4  Msg             = dword ptr  0Ch  
0000:00404DD4  wParam          = dword ptr  10h  
0000:00404DD4  lParam          = word ptr  14h  

//4 个参数，2 个局部变量  

0000:00404DD4  
0000:00404DD4                push    ebp  
0000:00404DD5                mov     ebp, esp  

//标准的 Delphi 调用框架  

0000:00404DD7                add     esp, 0FFFFFFB8h  

//这里相当于 sub esp,48h。注意上面的 Paint=PAINTSTRUCT ptr -48h  
//相当有趣。  

0000:00404DDA                push    ebx  
0000:00404DDB                xor     ebx, ebx  
0000:00404DDD                mov     eax, [ebp+Msg]  
0000:00404DE0                cmp     eax, 111h       ; WM_COMMAND  
0000:00404DE5                jg      short loc_404E0B  
0000:00404DE7                jz      loc_404EF8  

//我没有反过 VC，但这里显然和通常的思维不大一样。  
//WM_COMMAND 被当作分界线以减少判断。  

0000:00404DED                sub     eax, 0Fh        ; WM_PAINT  
//sub！居然不是 cmp！  

0000:00404DF0                jz      loc_404EBD  
0000:00404DF6                sub     eax, 1Ch        ; WM_DRAWITEM  
0000:00404DF9                jz      loc_404F3F  
0000:00404DFF                sub     eax, 0E5h       ; WM_INITDIALOG  
0000:00404E04                jz      short loc_404E2B  
0000:00404E06                jmp     loc_404F8D  

//这里就退出了  

0000:00404E0B loc_404E0B:  
0000:00404E0B                sub     eax, 136h       ; WM_CTLCOLORDLG  
0000:00404E10                jz      loc_404F4B  
0000:00404E16                sub     eax, 2          ; WM_CTLCOLORSTATIC  
0000:00404E19                jz      loc_404F6C  
0000:00404E1F                sub     eax, 0C9h       ; WM_LBUTTONDOWN  
0000:00404E24                jz      short loc_404E79  
0000:00404E26                jmp     loc_404F8D  

//退出  
...  
0000:00404F8D loc_404F8D:  
0000:00404F8D  
0000:00404F8D                xor     ebx, ebx  
//下面要把 ebx 作为返回值的。  
0000:00404F8F loc_404F8F:  
0000:00404F8F  
0000:00404F8F                mov     eax, ebx  
0000:00404F91                pop     ebx  
0000:00404F92                mov     esp, ebp  
0000:00404F94                pop     ebp  
0000:00404F95                retn    10h  
```

结合 `SDK` 程序的写法，得到主消息循环。

```
Function LicenseProc(hDlg:HWND;Msg,wParam,lParam:DWORD):LRESULT; stdcall;  
Var  
  sPaint:PAINTSTRUCT;  
Begin  
  Result:=0;  
  Case Msg of  
    WM_COMMAND:  
      Begin  
      End;  
    WM_PAINT:  
      Begin  
      End;  
    WM_DRAWITEM:  
      Begin  
      End;  
    WM_INITDIALOG:  
      Begin  
      End;  
    WM_CTLCOLORDLG:  
      Begin  
      End;  
    WM_CTLCOLORSTATIC:  
      Begin  
      End;  
    WM_LBUTTONDOWN:  
      Begin  
      End;  
  End;  
End;  
```

不寻常的事发生了。它用了两个不常见的消息：`WM_CTLCOLORDLG` 和 `WM_CTLCOLORSTATIC`。

先看 `WM_COMMAND`:

```
0000:00404EF8 loc_404EF8:  
0000:00404EF8                mov     eax, [ebp+wParam]  
0000:00404EFB                sub     eax, 3E9h  
//照例是 sub 的把戏，反正参数是传值的  
//结合 dump 出的 RC 可以看到这是一个按钮的 ID  
0000:00404F00                jz      short loc_404F0C  
0000:00404F02                dec     eax  
0000:00404F03                jz      short loc_404F23  
0000:00404F05                sub     eax, 2  
0000:00404F08                jz      short loc_404F30  
0000:00404F0A                jmp     short loc_404F3B  
0000:00404F0C loc_404F0C:  
0000:00404F0C                mov     ds:dword_4060B0, 0FFFFFFFFh  
//注意这里，这就是那个返回的 Flag！因为程序开头先把它清 0 了  
//所以这里只需要赋个值就可以了。注意，Delphi 里 True 是非 0 的！  
0000:00404F16                push    0               ; nResult  
0000:00404F18                mov     eax, [ebp+hWnd]  
0000:00404F1B                push    eax             ; hDlg  
0000:00404F1C                call    EndDialog  
0000:00404F21                jmp     short loc_404F3B  
0000:00404F23 loc_404F23:  
0000:00404F23                push    0               ; nResult  
0000:00404F25                mov     eax, [ebp+hWnd]  
0000:00404F28                push    eax             ; hDlg  
0000:00404F29                call    EndDialog  
0000:00404F2E                jmp     short loc_404F3B  
0000:00404F30 loc_404F30:  
0000:00404F30                push    0               ; nResult  
0000:00404F32                mov     eax, [ebp+hWnd]  
0000:00404F35                push    eax             ; hDlg  
0000:00404F36                call    EndDialog  
0000:00404F3B  
0000:00404F3B loc_404F3B:  
0000:00404F3B  
0000:00404F3B                xor     ebx, ebx  
0000:00404F3D                jmp     short loc_404F8F  
```

好的，现在 `WM_COMMAND` 的处理有了。

```
Case wParam of  
  LICENSE_YES:  
    Begin  
      Flag:=True;  
      EndDialog(hDlg,0);  
    End;  
  LICENSE_NO:  
    Begin  
      EndDialog(hDlg,0);  
    End;  
  LICENSE_CLOSE:  
    Begin  
      EndDialog(hDlg,0);  
    End;  
End;  
```

很清楚，不是吗？下面是 `WM_CTLCOLORDLG` 和 `WM_CTLCOLORSTATIC`。

```
0000:00404F4B loc_404F4B:  
0000:00404F50                mov     eax, [ebp+wParam]  
0000:00404F53                push    eax             ; HDC  
0000:00404F54                call    SetTextColor  
0000:00404F59                push    1               ; int  
0000:00404F5B                mov     eax, [ebp+wParam]  
0000:00404F5E                push    eax             ; HDC  
0000:00404F5F                call    SetBkMode  
0000:00404F64                mov     ebx, ds:hbr  

//这里有点费解，从 CreateSolidBrush 的说明可以知道，如果函数  
//失败，返回的是 null，就是 false。这里估计是利用了这点。  
//一旦 CreateSolidBrush 失败，这里就可以退出了。有点意思。  

0000:00404F6A                jmp     short loc_404F8F  
0000:00404F6C loc_404F6C:  
0000:00404F6C                push    0A0A0A0h        ; COLORREF  
0000:00404F71                mov     eax, [ebp+wParam]  
0000:00404F74                push    eax             ; HDC  
0000:00404F75                call    SetTextColor  
0000:00404F7A                push    1               ; int  
0000:00404F7C                mov     eax, [ebp+wParam]  
0000:00404F7F                push    eax             ; HDC  
0000:00404F80                call    SetBkMode  
0000:00404F85                mov     ebx, ds:hbr  
0000:00404F8B                jmp     short loc_404F8F  
```

给出相应的 `SRC`:

```
WM_CTLCOLORDLG:  
  Begin  
    SetTextColor(wParam,$A0A0A0);  
    SetBkMode(wParam,TRANSPARENT);  
    Result:=h_Brush;  
    //如果要清楚一点可以改为 Result:=h_Brush<>nil;  
    //当然函数返回类型要改。  
  End;  
WM_CTLCOLORSTATIC:  
  Begin  
    SetTextColor(wParam,$A0A0A0);  
    SetBkMode(wParam,TRANSPARENT);  
    Result:=h_Brush;  
  End;
```
