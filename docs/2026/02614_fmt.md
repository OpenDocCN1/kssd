
# KeyGen for Zero Popup 1.05 (5千字)

KeyGen for Zero Popup 1.05

Author: Tooto Technologies, Inc.  `http://popupkiller2002.tripod.com/`

Program Type: Shareware

在上次的教程里已经告诉大家如何暴力破解及给出一个注册码了,这次主要是讲如何写注册机.这个东东其实没多大用!有了它反而有些站连不上:-)

先讲注册原理,它的注册码分为两段,首6个字节它祗接受0 - 9 ,a - f为注册码,而後6个字节为首段注册码处理后的结果,总长度为12字节.

当我们输入12个字节时,它先把前6个字节跟据下面列表转换为另一码,再把对应位数加上特定值,再跟後6位相比,然後跳转.

```text
0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f    <--输入字符
5  d  7  9  f  1  a  2  b  3  6  c  8  e  0  4    <--转换之後的值
35 64 37 39 66 31 61 32 62 33 36 63 38 65 30 34    <--hex码
```

它所加的特定值是跟据转换後的字符值而加以选择.

假定转换之後的值=`ID`.

当转换之後的值为:

```text
0 - 4 ---> ID = ID + n        <--B
5 - 9 ---> ID = ID + n + 31   <--C
a - f ---> ID = ID + n * 2    <--A
```

下面为反汇出的机器码

```asm
:10006C76  0FBE4DF0      movsx ecx, byte ptr [ebp-10]    <--输入字
:10006C7A  83F961        cmp ecx, 00000061
:10006C7D  7C0F          jl 10006C8E
:10006C7F  0FBE55F0      movsx edx, byte ptr [ebp-10]
:10006C83  8B45FC        mov eax, dword ptr [ebp-04]     <--位数
:10006C86  8D0C42        lea ecx, dword ptr [edx+2*eax]  <--A
:10006C89  884DF0        mov byte ptr [ebp-10], cl
:10006C8C  EB23          jmp 10006CB1

; * Referenced by a (U)nconditional or (C)onditional Jump at Address:
;   :10006C7D(C)
:10006C8E  0FBE55F0      movsx edx, byte ptr [ebp-10]
:10006C92  83FA34        cmp edx, 00000034
:10006C95  7D0C          jge 10006CA3
:10006C97  0FBE45F0      movsx eax, byte ptr [ebp-10]
:10006C9B  0345FC        add eax, dword ptr [ebp-04]      <--B
:10006C9E  8845F0        mov byte ptr [ebp-10], al
:10006CA1  EB0E          jmp 10006CB1

; * Referenced by a (U)nconditional or (C)onditional Jump at Address:
;   :10006C95(C)
:10006CA3  0FBE4DF0      movsx ecx, byte ptr [ebp-10]
:10006CA7  8B55FC        mov edx, dword ptr [ebp-04]
:10006CAA  8D441131      lea eax, dword ptr [ecx+edx+31]  <--C
:10006CAE  8845F0        mov byte ptr [ebp-10], al
```

`[ebp-08]`为首6字节(处理後),`[ebp-0C]`为後6字节(我们输入的),跳转在`10006B47`.

```asm
:10006B35  8B45F8        mov eax, dword ptr [ebp-08]
:10006B38  50            push eax
:10006B39  8B4DF4        mov ecx, dword ptr [ebp-0C]
:10006B3C  51            push ecx
:10006B3D  E8AE650000    call 1000D0F0
:10006B42  83C408        add esp, 00000008
:10006B45  85C0          test eax, eax
:10006B47  7516          jne 10006B5F
```

编程方面,我是用VC的,下面是关键部份:

```c
for (j=0; j<6; j++)
{
    if (TmpCode[j] >= hex_a[10])
    {
        RegCode[j] = TmpCode[j] + j*2;
    }
    else if (TmpCode[j] < hex_a[4])
    {
        RegCode[j] = TmpCode[j] + j;
    }
    else
    {
        RegCode[j] = TmpCode[j] + j + 0x31;
    }
}
```

```c
case IDTEST:
    GetDlgItemText(hDlg,119,szBuff0,40);
    .
    .
    .
    SetDlgItemText(hDlg,116,szBuff0);
```

在响应`case IDTEST:`时,先用`GetDlgItemText(hDlg,119,szBuff0,40);`取得输入ID,用上面方法处理後,再用`SetDlgItemText(hDlg,116,szBuff0);`放到另一窗口显示.

整个原理就是这样简单!

下面讲一下其它情况

在处理输入字符时,我花了一些时间在字符与内码间之转换,最後用这句 `wsprintf(lpBuf, "%c", szBuff0[i]);` 才把问题解决,其中用SICE跟踪多次才发现,再用下面转为hex.

```c
if (lpBuf[0] > 0x39)
{
    k = lpBuf[0] - 0x57;
}
else
{
    k = lpBuf[0] - 0x30;
}
lpBuf[0] = hex_d[k];        <--这是对照转换的值
lstrcat(TmpCode, lpBuf);
```

这是最笨的方法,如有高手,请多指正!

当运行这注册机时,"Test"按键是被置为不可选用(disable),可选用(enable)条件为字节长度非零! 有点象密码输入框 !!

```c
case WM_COMMAND:
    Dlg_OnCommand(hDlg, wParam);
    StrLen = GetDlgItemText(hDlg,119,szBuff0,40);   <--取字节长度
    if (StrLen > 0)
    {
        EnableWindow(hwndb,TRUE);    <--这enable按键
    }
    break;
```

你会发现它祗接受0 - 9,a - f这16个字符,其它没法显示,那是因为用了subclassig的功能.

```c
case WM_INITDIALOG:
    Dlg_OnInit(hDlg);
    g_oldEditProc = (WNDPROC)SetWindowLong(
        GetDlgItem(hDlg, 119), GWL_WNDPROC, (long) EditProc );    <--这是做subclass
    if( message == WM_CHAR )
    {
        chCharCode = (TCHAR) wparam;
        if(chCharCode >= 0x30 && chCharCode <= 0x39 || chCharCode >= 0x61 && chCharCode <= 0x66)  <--这是0 - 9,a - f的选定
        {
            return( CallWindowProc( g_oldEditProc, hDlg, message, wparam, lparam ) );
        }
    }
```

最後是E-mail按键,先多谢Arbiter的提示,这按键才有颜色!

```c
case WM_CTLCOLORSTATIC:
    hDC = (HDC) wParam;
    hwnd = (HWND) lParam;
    if(hwnd == GetDlgItem(hDlg,124))
    {
        SetTextColor(hDC, RGB(0,0,255));   <--字符颜色
        SetBkMode(hDC, TRANSPARENT);       <--背景颜色
        return((BOOL)GetStockObject(NULL_BRUSH));
    }
```

Underline是下面造的

```c
void MakeCtrlUseUnderlinedFont(HWND hCtl)
{
    LOGFONT lf;
    GetObject(GetWindowFont(hCtl), sizeof(lf), &lf);
    lf.lfUnderline = TRUE;
    SetWindowFont(hCtl, CreateFontIndirect(&lf), TRUE);
}
```

那小手是从`winhlp32.exe`里第106个icon取出,这功能是自造的,Windows没这功能!

```c
GetWindowsDirectory(strWndDir, MAX_PATH);  // Get the windows directory
lstrcat(strWndDir, "\\winhlp32.exe");
hModule = LoadLibrary(strWndDir);
SetCursor(LoadCursor(hModule, MAKEINTRESOURCE(106)));
```

至於那动作,是用

```c
case IDMAIL:
ShellExecute(0, "open", "mailto:windos2k@163.com", 0, 0, 0);
```

可把这`mailto:windos2k@163.com`改为自巳的URL页.

小手的转换是用定时器(timer),这方法很不好,但我没别的!!

源码及可运行档在 `http://briefcase.yahoo.com/windos2k`

WinDos2K

09.14.01
