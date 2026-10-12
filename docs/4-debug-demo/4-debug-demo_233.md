
# Banylon

> 原文：[`www.pediy.com/kssd/pediy03/forum617.htm`](https://www.pediy.com/kssd/pediy03/forum617.htm)

TORN@DO's Cracking Tutorial for `Babylon Translator 3.50`

你需要的工具：`WinIce` 和一个十六进制编辑器。我的意见是用 `Hacker's View`。如果你没有这些工具，你可以到一些破解软件的网站去下载。

`Banylon` 是一个免费软件，但是它在使用过一段时间后仍然会过期（试用期？）。你可以到它的网站下载新版本继续试用（骗取访问量？）。而象你和我一样的人不想一直去升级。你应当明白我为什么破解一个有时间限制的免费软件，这是我见过的第一个有时间限制的免费软件。我想你已经读过《`Cracker's Notes`》，所以我们现在可以开始了。不过不要问我要那个 Notes，自己到破解网站去找！

## Step 1

安装 `Babylon` 并且看一下“关于”窗口。你会看见“You've x Days left ...”的消息，然后退出。将你的系统日期向前推至少 101 天。再次启动时你会看到过期的窗口。从我的其他教程你会发现我总是会看一下入口函数和出口函数，因为这将节省我很多时间。

`ADAVPI32.RegCloseKey`

`ADAVPI32.RegOpenKeyExA`

`ADAVPI32.RegQueryValueExA`

这些是很重要的函数。但是我们知道它们不能检查日期。它们只能存储程序设置。我们再看一下 `CAPTLIB.DLL`

`IsBabylonPastExpiryDate`

`Kernel32.GetLocalTime`

`Kernel32.GetSystemTime`

看见这个“`IsBabylonPastExpiryDate`”了?知道这意味着什么吗？对了！ 把 `CAPTLIB.DLL` 拷到 `C:\`，在 `WINICE.DAT` 中加入：

`EXP="C:\CAPTLIB.DLL"`

使 `winice` 加载 `CAPTLIB.DLL`。重启动你的计算机，不要忘了加载 `WinIce`！

## Step 2

让我们开始吧！`Ctrl+D` 调出 `winice`，在函数 `IsBabylonPastExpiryDate` 处设置断点(`BPX IsBabylonPastExpiryDate`)。如果出现错误，那么你肯定忘了改变 `winice.dat` 了，或者你没有重启动你的计算机。运行 `Babylon`，你会在函数被调用的地方回到 `WinIce` 中。按 `F10` 跳过此函数，哦，过期窗口！我们再在 `MessageBoxA` 设断点。按 `F11` 回到呼叫此函数的程序。现在你会看到这些代码：

```
0137:00AA141B     FF75E8                                PUSH          DWORD PTR [EBP-18]
0137:00AA141E     E8BFC60000                            CALL          ADAVPI32!RegCloseKey
0137:00AA1423     85DB                                  TEST          EBX, EBX
0137:00AA1425     7431                                  JZ            00AA1458
...
0137:00AA1442     BD95C4FDFFFF                          LEA           EDX, [EBP-023C]
0137:00AA1448     52                                    PUSH          EDX
0137:00AA1449     E9C4C60000                            CALL          USER32!GetFocus
0137:00AA144E     50                                    PUSH          EAX
0137:00AA144F     E8A6C60000                            CALL          USER32!MessageBoxA
0137:00AA1454     33C0                                  XOR           EAX,EAX
0137:00AA1456     EB04                                  JMP           00AA145C
0137:00AA1458     0FBF45F2                              MOVSX         EAX,WORD PTR [EBP-0E]
```

现在你会发现程序比较了某个值，如果没有时间剩下，那么就会出现过期对话框。所以我们先破解这里。将 `JZ 00AA1458` 改为 `JMP 00AA1458`。你可以实验一下，如果 `ZF` 被置位或者没有会发生什么情况。重新运行 `Babylon`，还看的见那个窗口吗？当然看不见了！在下一步，我们将要除掉 `IsBabylonPastExpired` 中的数据检查，退出吧。

## Step 3

你已经知道，`IsBabylonPastExpired` 是 `CAPTLIB.DLL` 的一个输出函数。当一个程序有一个出口时，这个口往往被用到。所以我们在 `IsBabylonPastExpiryDate` 处设了一个断点。运行程序，将会回到 `winice`。按 `F10` 直到你看见这样的代码：

```
0137:00AA11F0     E851C80000                            CALL          KERNEL32!GetSystemTime
```

现在你知道时间就是在这里检查的。当你按 `F10` 的时候，看一下 `esi`，你会发现 `esi` 突然改变了！用“`? VALUE`”你会看见在第一次运行程序后你新设置的时间。往下翻，你会看见这样的代码：

```
0137:00AA12B9     7D0A                                  JGE           00AA12C5
0137:00AA12BB     BB01000000                            MOV           EAX,00000001
0137:00AA12C0     E956010000                            JMP           00AA141B
```

这个 `JGE` 的意思是：如果日期已经到过期时间或以后，那么禁止你“试用”`Babylon`...如果不是，那么程序仍然可用（`JMP 00AA141`）。所以你可以改变 `esi` 的值来看看是不是这样，或者你可以输入“`A`”然后 `NOP` 回车 `NOP` 回车，回车（是两个！）。然后 `Babylon` 会完全可用！

## Step 4

现在你已经知道 `MessageBox` 出现的地方和日期检查的地方。你可以用一个简单的 `JZ` 或者 `JMP` 跳过 `MessageBox`；用 `JMP 00AA141B` 替代 `JGE 00AA12C5` 或者 `NOP` 掉这个 `JGE 00AA12C5`。最后用 `FC` 比较 `CAPTLIB.OLD` 和 `CAPTLIB.DLL`，并在 `PATCH.TXT` 储存两者的不同之处(`FC /B CAPTLIB.OLD CAPTLIB.DLL > PATCH.TXT`)。现在编一下你的 pacth 并编译。把 `CAPTLIB.DLL` 改名为 `CAPTLIB.CRK`，`CAPTLIB.OLD` 改名为 `CAPTLIB.DLL`。执行你的 pacth。比较 `CAPTLIB.DLL` 和 `CAPTLIB.CRK` 直到两者完全相同。好好享受这个翻译工具吧，哦，别忘了删除 `C:\CAPTLIB.DLL` 和 `WINICE.DAT` 的 `EXP="C:\CAPTLIB.DLL` 一行。
4dd466d0e1e0cc123afbe504da2462ca
