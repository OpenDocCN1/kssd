

# Zero Popup 1.05 (2千字)

## 简介

这是一个防止 popup 的小程式，在 IE 启动时自动运行，可防止一些网站上的广告窗弹出。

## 软件信息

- **平台：** Windows 95/98/Me/NT/2000
- **分类：** Internet: World Wide Web: Browser Enhancements
- **作者：** Tooto Technologies, Inc.（http://popupkiller2002.tripod.com/）
- **类型：** Shareware

## 下载地址

下载地址如下，是 `ZeroPopup.msi` 方式，点击右键便可安装。

- http://fileforum.betanews.com/detail.php3?fid=997512878
- http://popupkiller2002.tripod.com/download.htm

## 暴力破解

暴力破解如下：

```
:100012B5 E8155B0000              call 10006DCF
      <-------- 做一些运算

:100012BA 668985DCFEFFFF          mov word ptr [ebp+FFFFFEDC], ax

:100012C1 8B95DCFEFFFF            mov edx, dword ptr [ebp+FFFFFEDC]

:100012C7 81E2FFFF0000            and edx, 0000FFFF

:100012CD 83FA07                  cmp edx, 00000007    <-------- 七天时间?

:100012D0 7C41                    jl 10001313        <-------- 改为 EB41
```

永远小於七天。

```
:100012D2 833D14D0011000          cmp dword ptr [1001D014], 00000000
:100012D9 7536                    jne 10001311
:100012DB C70518D0011001000000    mov dword ptr [1001D018], 00000001
:100012E5 6A00                    push 00000000
:100012E7 6800670010              push 10006700
:100012EC 6A00                    push 00000000
:100012EE 68CD000000              push 000000CD
```

改了上面 `100012D0` 的 `7C41` 为 `EB41` 後，过了七天後都没有窗口弹出了。

跟踪这个程式在 Win2000 下有点难，下了几个断点 `GetDlgitemTexta`、`GetWindowText`、`lstrcpya`、`lstrlena` 等都没反应。

它本身是 `dll` 形式出现（`HTMLEdit.dll`），是在 IE 启动时自动运行。没法在 `DLLMain` 下断，祗有下 `CreateFile` 这断点，在多次中断後停在 `HTMLEdit.dll` 里，但离开输入窗口还选。反汇编後又看不到有 `DialogBox` 这方面的断点。没辨法祗有放下键盘去洗澡了事!

不知过了多久，想起某人在论坛上问过关於 `Dialog` 的事，而且自己也告诉过他方法，就从资源档入手。

打开 Restorator 看看资源档，找到这窗口为 `0xCD`，再在 W32DASM 里找一下，在 `100012EE` 推这 ID 入 stack，向上便看到上面的机器码了。分析一下代码，把在 `100012D0` 的 `7C41` 改为 `EB41`，重启 IE，Bingo!! 七天後的 nag screen 没有了。

昨天把这改动在 mIRC 里告诉友人后，虽然解决了 nag 的问题，但好像有些事情还没完，再後看。

在下面的代码，经过分析後，结果如下：

```
:10006B20 E852000000              call 10006B77
:10006B25 83C404                  add esp, 00000004
:10006B28 8945F8                  mov dword ptr [ebp-08], eax
:10006B2B 837DF800                cmp dword ptr [ebp-08], 00000000
:10006B2F 7504                    jne 10006B35
:10006B31 32C0                    xor al, al
:10006B33 EB3E                    jmp 10006B73

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10006B2F(C)
|

:10006B35 8B45F8                  mov eax, dword ptr [ebp-08]  <------ 输入ID1
:10006B38 50                      push eax
:10006B39 8B4DF4                  mov ecx, dword ptr [ebp-0C]  <------ 输入ID2
:10006B3C 51                      push ecx
:10006B3D E8AE650000              call 1000D0F0
```

输入 `ID1` 为运算後的结果，是等於输入 `ID2`，注册便可成功。

我会试造注册机，如等不来可用下面注册码：

```
123456dill5k
WinDos2k
08.17.01
```

