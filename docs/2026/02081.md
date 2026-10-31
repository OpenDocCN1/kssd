

# 翻译的第三篇，软件算法和 OllyDB 的用法（新的调试软件）（10千字）

原著：Hoof Arted  
译者：TAE![CCG]

唉，这篇翻译的实在是太烂了，大家就看看如何使用新的调试工具 OllyDB 吧。

## 教程信息

新手教程 by Hoof Arted（Hoof's Workshop）

教程：破解并且做 DLLShow by Software By Design 的注册机（奇怪，这篇好像没有注册机）

目标：DLLShow V4.6 - http://www.gregorybraun.com/

工具：OllyDBG.exe - http://www.suddendischarge.com

## 前言

这是我的第一篇教程，我希望有时间写更多的。我是一个破解新手，搞破解已经花了我太多的时间了，但我仍然有很长的路要走。我尽力使这篇文章通俗易懂，我认为只有在你熟悉目标软件的情况下，你才可以破解它。我将开始描述调我们这课需要的试产品。

我不是一个好的 SoftIce 迷，光是熟悉指令和安装就花了我许多时间。我也很喜欢使用 W32dasm，但我尽管使用它在许多场合有很多限制。我升级到 Windows2000 后，竟然运行不了 W32dasm 了，我想找一个和它相同的调试/反编译软件，如果要比 W32dasm 更强大的，那我就已经找到了，它就是 Ollydbg。作者是：Oleh Yuschuk，这是给像我一样已经等了很久的人的一个工具。会使用 W32dasm 的你，你也将会掌握它的。

一个伟大的工作即将开始。

干掉它，跟我来吧，做我所作的，并且我尽我所能的解释给你听。

我选择这个目标的原因是它没有一个好用的卸载程序，许多程序员小贩都不把他们的软件做得很友好 :(

让我们开始吧！

## 开始破解

到我上面写的那个网站下载一个目标程序。解压缩并且安装它，运行 DLLShow 并且选择注册选项。你将看见软件需要输入名字，公司和注册码，我们现在已经可以大致确定这是一个根据姓名和公司名来生成注册码的软件，这也是破解过程中的一个进展。我们要找出密码，并且看它是如何产生的。

好的，现在我们要输入注册码但我们根本没有正确的注册码。打开 Ollydbg 并且用它打开软件（`DLLShow.exe`）。默认的，你将看见 4 段不同用处的调试窗口。左上角的用于显示反编译代码，右上角的用于显示寄存器数据，这是非常有用的。不像 SoftIce，你想查看寄存器中有疑问的地址，那么这个调试器显示地址的内容；如果程序是 ASCII 码值（姓名或注册码:），都将显示它并且你不需要转换。

左下角显示当前所载入的软件数据在内存中的位置，这对找出数据很有帮助，你待会儿将看见我是如何利用它的。右下角显示被各种函数所使用的内存，我也是初学者，也不知道这个窗口的所有用处，但是现在，我们并不需要知道。

那好，用调试器载入程序，我们需要运行它。选择 "Debug" 并且点击 "Run" 或按 F9。软件将像平时一样运行。去注册窗口并且输入名字，公司和注册码，我输入的是：

```
NAME : Hoof Arted
COMPANY : Hoof's Workshop
SERIAL : 77777777
```

你现在可以使用任何数字作注册码，我喜欢 7，而且保持在 8 个字符，这将是我干得快些（他们说 7 是幸运数字！）。

## 设置断点并捕获输入

我们知道如果我们点击 "OK" 按钮，软件将会弹出一个 "愚蠢破解者" 的窗口，我们需要知道在哪里设置断点。我喜欢首先试着用 `GetDlgItemTextA`，在 Ollydbg 中设置断点有一些不同，因为查找函数不像你希望的那样工作。右击左上角的窗口，并且选择 "Search for NAME (LABEL)"，这将出现一个带有许多软件所使用的函数的小窗口。向下拉动滑动条到了 `USER32.GetDlgItemTextA` 并且使它处于高亮状态，右击它选择 "Find references to import"；另一个窗口打开了，显示着软件调用函数的地址。通过按 F2 在每一个上面都设置一个断点，然后它的左边就会变成红色，如果有任何地方呼叫这里都会被调试器所暂停。

好的，现在单击目标软件的 "OK" 按钮。调试器将停在地址 `4159FB` 这里。我们不关心 `GetDlgItem` 函数是如何运行的，所以我们点击地址 `4159FB` 并且按 F2 来清除断点，然后选择下一行再按 F2 来恢复断点。按 F9 来运行软件以通过这个地方，它再次被停下来了，我们可以看见 `ESI` 寄存器（右上角）现在有一个 ASCII 码 "Hoof Arted"，真不错！它现在得到我们的名字了。如果我们再次按 F9，目标软件将再次停在相同的断点，但是这次，`ESI` 已经是我们的公司名了，是一个 ASCII。按 F9 第三次 `ESI` 将显示我们输入的注册码。好，现在我们知道程序已经得到我们所有的信息了。它将要用这些信息作什么呢？

## 跟踪注册算法

因为它已经得到了我们输入的所有信息了，我们需要从这儿将它带到软件中。我们不能再按 F9 运行，因为要找到我们想要得到的东西，所以我们按 F7 来一步步的运行，直到你到了以下代码处：

```
0040D717  |. 8D4C24 40      LEA    ECX,DWORD PTR SS:[ESP+40]      <<<<<<<<
0040D71B  |. 51            PUSH   ECX
0040D71C  |. E8 25990000    CALL   DLLShow.00417046              <<<<<<< Call (a)
0040D721  |. 56            PUSH   ESI
0040D722  |. 8BD8          MOV    EBX,EAX
0040D724  |. E8 37820000    CALL   DLLShow.00415960              <<<<<<< Call (b)
0040D729  |. 83C4 38        ADD    ESP,38
0040D72C  |. 3D 92A71901    CMP    EAX,119A792
0040D731  |.,75 18          JNZ    SHORT DLLShow.0040D74B
```

现在，如果你进入了第一个 Call（Call A），你将看见它对我们的注册码做了些什么，我们暂时不关心这个。设置断点在 `0040D721` 并按 F9。当我们停在下一行的时候，我们可以看看寄存器 `EAX`，我的显示为 `04A2CB71`。这个数的十进制是什么？用计算器算一下看看，这是我们输入的假注册码！`Call A` 是将我们的假注册码转换为 16 进制。Hmmmm...有趣，可能下一个 call 就是计算正确注册码的地方了。按 F7 进入 call 然后停一停，有些东西我需要解释给你听。

正如你所看到的，函数被显示在下面：

```
00415960  /$ 51            PUSH   ECX        ; DLLShow.00426006
00415961  |. 53            PUSH   EBX
00415962  |. 8B5C24 0C      MOV    EBX,DWORD PTR SS:[ESP+C]
00415966  |. 56            PUSH   ESI
00415967  |. 33F6          XOR    ESI,ESI
00415969  |. 53            PUSH   EBX        ; /Length Of Name
0041596A  |. 897424 0C      MOV    DWORD PTR SS:[ESP+C],ESI      ; |
0041596E  |. FF15 B4E04100  CALL   DWORD PTR DS:[<&KERNEL32.lstrlen>] ; \lstrlenA
00415974  |. 85DB          TEST   EBX,EBX
00415976  |.,74 4F          JE     SHORT DLLShow.004159C7
00415978  |. 85C0          TEST   EAX,EAX
0041597A  |.,74 4B          JE     SHORT DLLShow.004159C7
0041597C  |. 33D2          XOR    EDX,EDX
0041597E  |. 85C0          TEST   EAX,EAX
00415980  |.,7E 45          JLE    SHORT DLLShow.004159C7
00415982  |. 55            PUSH   EBP
00415983  |. 57            PUSH   EDI
00415984  |. BE 183E4200    MOV    ESI,DLLShow.00423E18 ; ASCII "|b!pz\*ls;rn|lf$vi^Axpe)rx5aic&9/2m5lsi4@0dmZw94cmqpfhw"
00415989  |. BF 01000000    MOV    EDI,1
0041598E  |. 2BF3          SUB    ESI,EBX
00415990  |. 8BCB          MOV    ECX,EBX
00415992  |. 2BFB          SUB    EDI,EBX
00415994  |> 0FBE1C0E      MOVSX  EBX,BYTE PTR DS:[ESI+ECX]      ; 将上面的 Ascii 依次移动到 EBX
00415998  |. 0FBEAC10 E03D4>MOVSX  EBP,BYTE PTR DS:[EAX+EDX+423DE0] ; chr(423DE0 + Len + Pos) 放入 EBP
004159A0  |. 0FAFDD        IMUL   EBX,EBP        ; 相乘
004159A3  |. 8D2C0F        LEA    EBP,DWORD PTR DS:[EDI+ECX]
004159A6  |. 0FAFDD        IMUL   EBX,EBP        ; 相乘结果再乘位数
004159A9  |. 0FBE29        MOVSX  EBP,BYTE PTR DS:[ECX]          ; 移动名字的字符到 EBP
004159AC  |. 0FAFDD        IMUL   EBX,EBP        ; 相乘结果再乘以名字字符
004159AF  |. 8B6C24 10      MOV    EBP,DWORD PTR SS:[ESP+10]
004159B3  |. 03EB          ADD    EBP,EBX        ; 累加
004159B5  |. 42            INC    EDX            ; 下一个字符
004159B6  |. 41            INC    ECX
004159B7  |. 3BD0          CMP    EDX,EAX        ; 名字结束了吗？
004159B9  |. 896C24 10      MOV    DWORD PTR SS:[ESP+10],EBP      ; 存入 [ESP+10]
004159BD  |.^7C D5          JL     SHORT DLLShow.00415994         ; 跳转，形成循环
004159BF  |. 8BC5          MOV    EAX,EBP
004159C1  |. 5F            POP    EDI
004159C2  |. 5D            POP    EBP
004159C3  |. 5E            POP    ESI
004159C4  |. 5B            POP    EBX
004159C5  |. 59            POP    ECX
004159C6  |. C3            RETN
```

Ollydbg 做的比较好的就是它可以给语句添加注释，你还可以随时更改这些注释，我还从没有看见其他的调试器可以做到这一点（译者：TRW 不就可以？）。你只需要对着语句点击右键然后选择 "Comment" 即可，那很简单。这样你就不需要把重要的地方记在纸上了。

好，我们看见那个函数检查了名字的个数（`0041596E`），然后它就移动下面的 ASCII：`"|b!pz\*ls;rn|lf$vi^Axpe)rx5aic&9/2m5lsi4@0dmZw94cmqpfhw"` 放进了内存。那么，这所有的动作说明了什么呢？好的，如果我们熟悉它的话，就会发现它移动我们上面所说的每个字符，一个一个的放入 `EBX`。`EBP` 中的值就是从地址 `423DE0 + (我们名字的个数) + (字符的位置)`。但是，`423DE0` 里面又是什么呢？让我们把它找出来，右击左下角的窗口并且选择要去的地址，输入 `423DE0` 然后单击 "OK"，你将看到以下片断：

```
00423DE0  23 73 65 72 42 26 6E 7A 7C 6D 66 4D 31 2F 35 28  #serB&nz|mfM1/5(
00423DF0  21 73 64 24 4D 71 2E 7B 73 5D 2B 73 46 6A 74 4B  !sd$Mq.{s]+sFjtK
00423E00  70 7A 53 64 74 7A 6F 58 71 6D 62 5E 41 6C 40 64  pzSdtzoXqmb^Al@d
00423E10  76 3A 73 3F 78 2F 00 00 7C 62 21 70 7A 2A 6C 73  v:s?x/..|b!pz\*ls
00423E20  3B 72 6E 7C 6C 66 24 76 69 5E 41 78 70 65 29 72  ;rn|lf$vi^Axpe)r
00423E30  78 35 61 69 63 26 39 2F 32 6D 35 6C 73 69 34 40  x5aic&9/2m5lsi4@
00423E40  30 64 6D 5A 77 39 34 63 6D 71 70 66 68 77  0dmZw94cmqpfhw
```

这是有用代码的第二段，下一行乘以两个值，我这里是 `66` 和 `7C`，它然后乘以总数，用名字的位数 1（H）、2（o）.... 它然后用名字的第一个字符和乘积，在我这儿是 `48`（H），然后所有的数都这样，我得总数是 `1E159AD`。继续按 F7 直到到达以下片断：

```
00415400  /$ 8B4424 04      MOV    EAX,DWORD PTR SS:[ESP+4]
00415404  |. 56            PUSH   ESI
00415405  |. 8B35 84C34200  MOV    ESI,DWORD PTR DS:[42C384] ; DB95DB95 = Hardcoded
0041540B  |. 50            PUSH   EAX
0041540C  |. 81CE 78030000  OR     ESI,378                    ; Or with 378 = DB95DBFD
00415412  |. E8 49050000    CALL   DLLShow.00415960
00415417  |. 8B4C24 10      MOV    ECX,DWORD PTR SS:[ESP+10]
0041541B  |. 03F0          ADD    ESI,EAX                    ; Add Name total and Co total to DB95DBFD
0041541D  |. 51            PUSH   ECX
0041541E  |. E8 3D050000    CALL   DLLShow.00415960
00415423  |. 83C4 08        ADD    ESP,8
00415426  |. 03C6          ADD    EAX,ESI                    ; Total = E2F7A8AE
00415428  |. 5E            POP    ESI
```

这是在干什么呢？它带着你的总数，并且加上它的 hardcoded，`(DB95DB95 OR 378) = DB95DBFD`。整个计算此数过程以公司名重复的，两个总和的地址指向 `E2F7A8AE` 这个数，做的所有事情就是转化成十进制数。这个数就是注册码了！

按 F9 直到得到错误讯息并且输入一个新的注册码，就是它!!! 你已经破解它了。

现在我们知道注册码是怎么产生的了，你可以做一个属于你自己的注册机了。我已经破解这个公司出的许多软件了，因为他们是很懒惰的。他们使用同一种算法，仅仅是换个 ID `DB95DB95`。

我希望你能从这个教程中学到一些关于 Ollydbg 的使用方法，这是一个新的调试软件。

Hoof Arted（Hoof's Workshop）"Go forth and KICK ass!"

h_arted@hotmail.com

