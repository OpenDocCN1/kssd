
# Holiday Lights V5.0

> 原文：[`www.pediy.com/kssd/pediy03/forum753.htm`](https://www.pediy.com/kssd/pediy03/forum753.htm)

Holiday Lights V5.0   
程式猎人

简介：`Holiday Lights` 在屏幕边缘放置圣诞树灯泡串，并播放节日背景音乐。

灯泡出现在桌面壁纸上或所有其他窗口之上，因此你在处理其他计算机任务时也能感受到节日气氛。你可以从许多不同的灯泡和歌曲中进行选择。

当计算机空闲时，`Holiday Lights` 还可以显示一个屏幕保护程序，在灯泡后面显示 falling snow 或其他动画。

追踪：`name`: `dahuilang`

`RN`: `01234567`

好长时间没有追踪软件了，昨天没事可做就开始追踪这个软件，发现这个软件真的不是一般的软件，想要追出来它的注册码还得需要一点工夫。这个可是对初学者有一定的帮助，可以启发初学者的思维，可以了解如何注册这个软件。现在就开始破解它了。

这个软件是使用当注册码正确后按钮才可按的注册方式，所以在这里先输入 `RN: 0123456` 后，设 `bpx hmemcpy`，输入 7 后拦下如下：

```
:0040AE41(C)

|

:0040ADD5 8D45C0                lea eax, dword ptr [ebp-40]
:0040ADD8 50                    push eax
:0040ADD9 E8927F0000            call 00412D70
:0040ADDE 84C0                  test al, al
:0040ADE0 59                    pop ecx
:0040ADE1 754C                  jne 0040AE2F
:0040ADE3 8D45C0                lea eax, dword ptr [ebp-40]
:0040ADE6 50                    push eax
:0040ADE7 E8C462FFFF            call 004010B0
:0040ADEC 84C0                  test al, al
:0040ADEE 59                    pop ecx
:0040ADEF 753E                  jne 0040AE2F
:0040ADF1 8D45C0                lea eax, dword ptr [ebp-40]
:0040ADF4 50                    push eax
:0040ADF5 E8F6FE0000            call 0041ACF0
:0040ADFA 84C0                  test al, al
:0040ADFC 59                    pop ecx
:0040ADFD 7530                  jne 0040AE2F
:0040ADFF 8D45C0                lea eax, dword ptr [ebp-40]
:0040AE02 50                    push eax
:0040AE03 E8E8750000            call 004123F0
:0040AE08 84C0                  test al, al
:0040AE0A 59                    pop ecx
:0040AE0B 7522                  jne 0040AE2F
:0040AE0D 8D45C0                lea eax, dword ptr [ebp-40]
:0040AE10 50                    push eax
:0040AE11 E8EA210000            call 0040D000
:0040AE16 84C0                  test al, al
:0040AE18 59                    pop ecx
:0040AE19 7514                  jne 0040AE2F
:0040AE1B 8D45C0                lea eax, dword ptr [ebp-40]
:0040AE1E 50                    push eax

* Reference To: USER32.TranslateMessage, Ord:0285h

                          |
:0040AE1F FF1510375400     Call dword ptr [00543710]
:0040AE25 8D45C0          lea eax, dword ptr [ebp-40]
:0040AE28 50              push eax

* Reference To: USER32.DispatchMessageA, Ord:0096h

                          |
:0040AE29 FF1514375400    Call dword ptr [00543714]

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0040ADD3(U), :0040ADE1(C), :0040ADEF(C), :0040ADFD(C), :0040AE0B(C)
|:0040AE19(C)
|
:0040AE2F 8D45C0          lea eax, dword ptr [ebp-40]
:0040AE32 6A00            push 00000000
:0040AE34 6A00            push 00000000
:0040AE36 6A00            push 00000000
:0040AE38 50              push eax

* Reference To: USER32.GetMessageA, Ord:012Ch

                          |
:0040AE39 FF1518375400    Call dword ptr [00543718]
:0040AE3F 85C0            test eax, eax
:0040AE41 7F92            jg 0040ADD5
:0040AE43 EB28            jmp 0040AE6D
```

大家看到上面有许多的比较地方，可是如果你要到达这里你将在这里一直在循环，无法从这里出来，所以在这里使用这个拦截将无法达到我们想要的结果，那么我们将如何对待它呢？

办法当然有了，可能不同的人会使用不同的办法，破解也需要机会的。我先考虑这个软件有可能会将注册的结果保存在注册表中，现在使用 `Regmon` 软件来监视软件对注册表有什么动作。经查找，终于发现这个监视软件的确是不负众望，查到了我想要得到的东西。如下：

```
HKEY_CURRENT_USER\Software\Tiger Technologies\Holiday Lights\serial number NTFF
HKEY_CURRENT_USER\Software\Tiger Technologies\Holiday Lights\user NTFF
```

现在打开注册表，查找是否有这两个键值，发现了没有？当然没有，因为 `Regmon` 软件已经提到此软件没有读到这两个值，所以我们就知道这点了。如果这个软件注册成功，那么它将注册信息写到这里，每次启动的时候从这里读取注册信息，如果成功就认为是注册用户，否则为试用用户。下面就是在注册表中创建这两个值了。如下：

```
serial number: 01234567
user: dahuilang
```

因为程序将读取注册表，所以我们使用 `TRW` 的 `load` 功能加载程序，后设 `bpx regqueryvalueexa`（这个就是从 `Regmon` 中看到的程序读取注册表的函数）。

你在第三次被拦下如下：

```
:0040EE00 55                push ebp
:0040EE01 89E5              mov ebp, esp
:0040EE03 53                push ebx
:0040EE04 56                push esi
:0040EE05 83EC08            sub esp, 00000008
:0040EE08 8B7508            mov esi, dword ptr [ebp+08]
:0040EE0B 8B5D0C            mov ebx, dword ptr [ebp+0C]
:0040EE0E C60600            mov byte ptr [esi], 00
:0040EE11 C60300            mov byte ptr [ebx], 00
:0040EE14 8D4DF0            lea ecx, dword ptr [ebp-10]
:0040EE17 6801000080        push 80000001
:0040EE1C E80FFBFFFF        call 0040E930
:0040EE21 8D4DF0            lea ecx, dword ptr [ebp-10]
:0040EE24 56                push esi
:0040EE25 6A40              push 00000040
:0040EE24 56                push esi
:0040EE25 6A40              push 00000040

* Possible StringData Ref from Data Obj ->"User"

                          |
:0040EE27 685EB55300        push 0053B55E
:0040EE2C E85FF9FFFF        call 0040E790
:0040EE31 8D4DF0            lea ecx, dword ptr [ebp-10]  <-出来
:0040EE34 53                push ebx
:0040EE35 6A10              push 00000010

* Possible StringData Ref from Data Obj ->"Serial Number"

                          |
:0040EE37 6863B55300        push 0053B563
:0040EE3C E84FF9FFFF        call 0040E790
:0040EE41 8D4DF0            lea ecx, dword ptr [ebp-10]  <-出来
:0040EE44 E827F9FFFF        call 0040E770
:0040EE49 803E00            cmp byte ptr [esi], 00
:0040EE4C 753A              jne 0040EE88
:0040EE4E 803B00            cmp byte ptr [ebx], 00
:0040EE51 7535              jne 0040EE88
:0040EE53 8D4DF4            lea ecx, dword ptr [ebp-0C]
:0040EE56 6802000080        push 80000002
:0040EE5B E8D0FAFFFF        call 0040E930
:0040EE60 8D4DF4            lea ecx, dword ptr [ebp-0C]
:0040EE63 56                push esi
:0040EE64 6A40              push 00000040

* Possible StringData Ref from Data Obj ->"User"

                          |
:0040EE66 685EB55300        push 0053B55E
:0040EE6B E820F9FFFF        call 0040E790
:0040EE70 8D4DF4            lea ecx, dword ptr [ebp-0C]
:0040EE73 53                push ebx
:0040EE74 6A10              push 00000010

* Possible StringData Ref from Data Obj ->"Serial Number"

                          |
:0040EE76 6863B55300        push 0053B563
:0040EE7B E810F9FFFF        call 0040E790
:0040EE80 8D4DF4            lea ecx, dword ptr [ebp-0C]
:0040EE83 E8E8F8FFFF        call 0040E770

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0040EE4C(C), :0040EE51(C)
|
:0040EE88 8D65F8            lea esp, dword ptr [ebp-08]
:0040EE8B 5E                pop esi
:0040EE8C 5B                pop ebx
:0040EE8D 5D                pop ebp
:0040EE8E C3                ret
```

上面是从 `W32dasm` 反汇编得到的结果。如果你使用 `TRW` 来动态追踪，一开始是看不到这些信息的，所以在上面将被拦下二次，分别读取 `user` 和 `serial number` 的值。现在程序可以追踪到了。我在内存中一看到有这两个值出现就知道离成功不远了。现在向下追踪。
