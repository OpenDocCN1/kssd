
```
地址       区段       类型    (  名称                                    注释
004040A0   .rdata     输入    (    USER32.DestroyWindow
004040B4   .rdata     输入    (    USER32.DialogBoxParamA
0040409C   .rdata     输入    (    USER32.EnableMenuItem
004040B0   .rdata     输入    (    USER32.EndDialog
00404034   .rdata     输入    (    KERNEL32.ExitProcess
00404048   .rdata     输入    (    KERNEL32.FreeEnvironmentStringsA
0040404C   .rdata     输入    (    KERNEL32.FreeEnvironmentStringsW
00404080   .rdata     输入    (    KERNEL32.GetACP
0040402C   .rdata     输入    (    KERNEL32.GetCommandLineA
0040407C   .rdata     输入    (    KERNEL32.GetCPInfo
0040403C   .rdata     输入    (    KERNEL32.GetCurrentProcess
004040A4   .rdata     输入    (    USER32.GetDlgItem
00404054   .rdata     输入    (    KERNEL32.GetEnvironmentStrings
00404004   .rdata     输入    (    KERNEL32.GetEnvironmentStringsW
00404060   .rdata     输入    (    KERNEL32.GetFileType
00404098   .rdata     输入    (    USER32.GetMenu
00404044   .rdata     输入    (    KERNEL32.GetModuleFileNameA
00404024   .rdata     输入    (    KERNEL32.GetModuleHandleA
00404084   .rdata     输入    (    KERNEL32.GetOEMCP
00404020   .rdata     输入    (    KERNEL32.GetProcAddress
00404028   .rdata     输入    (    KERNEL32.GetStartupInfoA
0040405C   .rdata     输入    (    KERNEL32.GetStdHandle
0040400C   .rdata     输入    (    KERNEL32.GetStringTypeA
00404008   .rdata     输入    (    KERNEL32.GetStringTypeW
00404030   .rdata     输入         KERNEL32.GetVersion
00404088   .rdata     输入    (    KERNEL32.HeapAlloc
00404068   .rdata     输入    (    KERNEL32.HeapCreate
00404064   .rdata     输入    (    KERNEL32.HeapDestroy
00404070   .rdata     输入    (    KERNEL32.HeapFree
00404058   .rdata     输入    (    KERNEL32.HeapReAlloc
00404014   .rdata     输入    (    KERNEL32.LCMapStringA
00404010   .rdata     输入    (    KERNEL32.LCMapStringW
00404094   .rdata     输入    (    USER32.LoadIconA
0040401C   .rdata     输入    (    KERNEL32.LoadLibraryA
00404018   .rdata     输入    (    KERNEL32.MultiByteToWideChar
004040A8   .rdata     输入    (    USER32.PostMessageA
00404074   .rdata     输入    (    KERNEL32.RtlUnwind
004040AC   .rdata     输入    (    USER32.SendMessageA
00404000   .rdata     输入    (    KERNEL32.SetHandleCount
00404038   .rdata     输入    (    KERNEL32.TerminateProcess
00404040   .rdata     输入    (    KERNEL32.UnhandledExceptionFilter
0040408C   .rdata     输入    (    KERNEL32.VirtualAlloc
0040406C   .rdata     输入    (    KERNEL32.VirtualFree
00404050   .rdata     输入    (    KERNEL32.WideCharToMultiByte
00404078   .rdata     输入    (    KERNEL32.WriteFile
00401210   .text      输出         <模块入口点>
```

选择 `USER32.EnableMenuItem`,右键--->在每个参考上下断点。

重新载入 `OD`

`F9` 运行，断下。

代码:

```
004011E3      6A 01         push    1
004011E5   .  68 459C0000   push    9C45                              ; |ItemID = 9C45 (40005.)
004011EA   .  50            push    eax                              ; |hMenu
004011EB   .  FF15 9C404000 call    dword ptr \<&USER32.EnableMenuIte>; \\EnableMenuItem       断在此处
```

如何确定这个菜单就是我们的目标呢？

用 `eXeScope` 查看，如下图。

![  看到了吗？Menu 子菜单的 ID 是 4005，而我们的 push    9C45;  |ItemID = 9C45 (40005.)不就是这个参数吗？  所以就是这里了，已经找到关键点。  好，下面把 004011E3 的 push 1 成为 push 0，即是激活菜单。  好，保存文件后，运行，果然 OK，如下图所示：  ![](img/8437369625de5eb8860ccbe05b6091e6.jpg)  

可以看到，menu 子菜单已经激活了，点击后，出现一个对话框，如下图：

![](img/1cb198f55d52f23b9726359cf071f6a5.jpg)

好了，菜单限制已经完了。

下面再看下这个全代码如何逆向：

我们在断点处向前找，代码如下：

代码:

```
004011B9   >  8B15 90544000 mov     edx, dword ptr \[405490\]           ;  EnableMe.00400000; Case 110 (WM_INITDIALOG) of switch 00401124
004011BF   .  56            push    esi
004011C0   .  6A 70         push    70                                ; /RsrcName = 112.
004011C2   .  52            push    edx                               ; |hInst => 00400000
004011C3   .  FF15 94404000 call    dword ptr \[<&USER32.LoadIconA>\]   ; \\LoadIconA
004011C9   .  50            push    eax                               ; /lParam
004011CA   .  6A 01         push    1                                 ; |wParam = 1
004011CC   .  8B7424 10     mov     esi, dword ptr \[esp+10\]           ; |
004011D0   .  68 80000000   push    80                                ; |Message = WM_SETICON
004011D5   .  56            push    esi                               ; |hWnd
004011D6   .  FF15 AC404000 call    dword ptr \[<&USER32.SendMessageA>>; \\SendMessageA
004011DC   .  56            push    esi                               ; /hWnd
004011DD   .  FF15 98404000 call    dword ptr \[<&USER32.GetMenu>\]     ; \\GetMenu
004011E3      6A 01         push    1
004011E5   .  68 459C0000   push    9C45                              ; |ItemID = 9C45 (40005.)
004011EA   .  50            push    eax                               ; |hMenu
004011EB   .  FF15 9C404000 call    dword ptr \[<&USER32.EnableMenuIte>; \\EnableMenuItem       断在此处 004011F1   .  5E            pop     esi
004011F2   .  B8 01000000   mov     eax, 1
004011F7   .  C2 1000       ret     10
```

其实上面就是 `WM_INITDIALOG` 消息的处理函数。

为什么这么说呢？ 因为我们可以查资料得知 `WM_INITDIALOG` 的常为 110h，所以我们来确定这个跳转是不是从 `CMP ESI,110` 后面过来的。

(其实 `OD` 已经帮我们分析好了，可以直观的看到注释，但我还是自己去分析下。)

我们看看第一行：

004011B9   >  8B15 90544000 mov     edx, dword ptr \[405490\]           ;  EnableMe.00400000; Case 110 (WM_INITDIALOG) of switch 00401124

它是从

代码:

```
00401120   .  8B4424 08     mov     eax, dword ptr \[esp+8\]
00401124   .  83E8 10       sub     eax, 10                           ;  Switch (cases 10..111)
00401127   .  0F84 CD000000 je      EnableMe.004011FA
0040112D   .  2D 00010000   sub     eax, 100
00401132   .  0F84 81000000 je      EnableMe.004011B9
```

跳过来的，`EAX` 先是减 10，再减 100，之后跳，不就是减了 110 吗？所以就是 `WM_INITDIALOG` 分支。

也即是说，用源代码是在 `WM_INITDIALOG` 中进行加载程序图标 和 将菜单设置为禁止的。

好了，其它就不分析了。

riusksk:感谢你的关注。

时间只要去挤，总会有的，只是每天加班回来都快 10 只有 1、2 个小时，我一定要坚持!!!

\---总是觉得从《加》书中的例子过一遍是很快的，但觉得学完例子后，自己的体会和总结才是自己的莫大收获，所以笔记的正文会很“嗦”。

### 5.5 KeyFile 保护

额，这个第一章进展好慢哪!!!要提速。

首先，查壳。

不得不说，`PEID` 比 `pe-scan` 要强大，`PEID` 查出来是 `MASM32 / TASM32` 汇编写的，无壳。

运行试下，如下图

![](img/6594321dc7e96cc3aa8002ad6352ac5e.jpg)

没有任何反应，特别之处：是这个 `CM` 没有任何供输入的地方。

　　虽然书中已经说了这个是文件保护的，但我在想，如果我面对一个新的 `CM`，我如何得知它是什么保护机制呢？

　　呵呵，这个问题先保留，继续搞下去，也许就分析明朗了。

\----------------------------------------------------------------------

好了，下面我们还推测是否是文件保护。

打开 `Filemon_fix.7.03.exe`(从看雪工具集中下载)

过滤器设置如下图：

![](img/229b5a74ce6ef5da3bcb8b075d99a147.jpg)

进入 `Filemon` 的界面后，说下我的使用心得：

注意 `Filemon` 会记录与 `PackMe` 相关的文件操作，我本以为不会很多，可是 360 等文件监控程序会不断地查看文件，所以会产生相关多的信息，有时会多到几百条，虽然 `Filemon` 的 `process` 列表会有程序的图标显示，我们可以看到是哪个程序在跑，但是我觉得还是进行一点技巧，让分析的结果少些(当然，重要的分析不能少)

　　本例的 `CM` 中是按下 `Check` 按钮后，才会进行文件保护机制的检查，所以我们可以先运行 `PackMe.exe`。

　　这时会产生一些分析结果，我们 `Ctrl+E` 停止分析，再 `Ctrl+X` 清空(因为这些都不是我们要的)

然后再 `Ctrl+E` 开始分析，然后迅速点 `Check` 按钮，半秒后我们就 `Ctrl+E` 停止，呵呵，此时该出来的都出来了，总分析数据也不是很大，如下图：

![](img/5b4871c3e9beb8c7b5cc41505f807f22.jpg)

我们一下就可以定位是哪个文件了。

0.00023467 `PackMe.exe:2192` `OPEN` `F:\\KwazyWeb.bit` `NOT FOUND` `Options:` `Open` `Access:` `Read`

上面这行是重要的信息，分析如下，以 `OPEN` 方式操作 `F:\\KwazyWeb.bit` 文件，操作结果是 `NOT FOUND`，权限是 `Read`

感觉就是这个 `KwazyWeb.bit` 文件了。

\------------------------  　以上是用 `Filemon.exe` 来查文件，其实，我现在遇到的好多软件已经相当有意识了，其本查不到个什么了。

下面 `OD` 载入。

`Ctrl+N`,对 `CreateFileA` 下参考断点，断下代码

```
004016D8 . 52 push edx ; |FileName => "KwazyWeb.bit"
004016D9 . E8 1C010000 call <jmp.&KERNEL32.CreateFileA> ; \\CreateFileA
004016DE . 83F8 FF cmp eax, -1 ; 断下返回到此
004016E1 . 74 64 je short PackMe.00401747
```

从 `004016D8 . 52 push edx ; |FileName => "KwazyWeb.bit"`可知，就是这个 `KwazyWeb,bit` 文件了。

用 `C32Asm` 创建一个 `KwazyWeb.bit` 文件，然后我们在 16 进制文件下写数据 123456789，这样的有明显顺序的数据可以方便我们进行分析和识别(当然，正确的注册码不可能是这种数据，一定有点复杂)。

![](img/a25d8a5d089fdd7c494d3cae5799a61e.jpg)

好，重新 `OD` 载入。

`F9` 运行，之后点 `Check` 按钮，因为 `Check` 之后，程序才会执行注册码的判断。

断在了 `Kernel.dll` 的领空，这里没我们的事，出去(`alt+F9`).`F8` 前进，

代码:
