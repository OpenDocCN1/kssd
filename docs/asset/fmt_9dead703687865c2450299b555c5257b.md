
# FoxitReader 4.1 栈溢出调试分析

> 原文：[`www.pediy.com/kssd/pediy12/132181.html`](https://www.pediy.com/kssd/pediy12/132181.html)

发帖前在论坛搜索了一下，怕有重复就不好了。发现这篇（[Foxit 阅读器栈缓冲区溢出 egghunter 方法](http://bbs.pediy.com/showthread.php?t=131568&highlight=foxit)），没来得及细看，似乎是说的同一个漏洞，不过是翻译过来的介绍怎么 fuzz 测试出这个漏洞的，还好我可以断续。![](img/755d44573ea0045d4eb19f2164c82d4e.jpg "扮鬼脸")

最近得到一个样本(`bhk.py`)，这是样本中的描述信息：

```
#********************
# Exploit Title : Foxit 4.1.1
# Date          : 13/11/2010
# Author        : Sud0
# Bug found by  : dookie
# Original POC  : https://www.exploit-db.com/exploits/15514/
# Software Link : http://www.foxitsoftware.com/downloads/index.php
# Version       : 4.1.1
# OS            : Windows
# Tested on     : XP SP3 En (VirtualBox)
# Type of vuln  : EIP / SEH
# Thanks to my wife for her support
# Greetz to: Corelan Security Team
# Greetz to: dookie for finding the bug
# Thanks to offsec for the Exploit Week End challenge
# http://www.corelan.be:8800/index.php/security/corelan-team-members/
# writeup : http://www.corelan.be:8800/index.php/2010/11/13/offensive-security-exploit-weekend/
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

感谢 `dookie` 及 `Sudo` 的分享。我根据这个样本调试分析了一下该漏洞，跟踪出最初出问题的地方是 `lstrcpyw` 拷贝字符串时覆盖了函数返回地址，下面是分析过程，欢迎大家讨论。

## 1. 样本测试

首先阅读了一下样本代码，比较简单，就是定义了几个字符串，然后把它们全部写入文件成为样本 pdf，不过样本中值得注意的一点是注释中提及利用了 `SEH` 跳转到 `Shellcode`。

好的，运行脚本得到 `Foxit-exploit.pdf`，开虚拟机运行 `FoxitReader` 相应版本，`OD` 附加，注意设置 `OD` 调试选项，不忽略任何异常，同时如果用了 `StrongOD` 插件的话，也注意 `StrongOD` 的选项中有个“skip some exceptions”，不要勾选，我一开始调试时就是因为这个选项，直接到了利用成功，`OD` 也没有异常中断下来，让我还以为样本没有利用 `SEH` 来跳转…. 嗯，这也算帮我多熟悉了一点 `StrongOD` 吧。 然后打开样本 pdf，`FoxitReader` 就异常中断到 `OD` 了。

异常指令地址为 `0x00410048`，同时观察一下栈窗口，发现 `esp` 附近全是 `0x0041`，也就是 `A` 的 `Unicode` 码，回想起 `bhk.py` 中的”`junk = "A"*540`”，看来这是被覆盖了的栈区。观察一下栈区，没有什么明显的返回地址什么的，很长一段都是 `Unicode` 字串。看来样本覆盖了太多的栈区，现在想从栈中得到问题函数（拷贝数据到栈中导致栈被破坏的函数）被调用前最近的一些函数的线索（主要是返回地址）是不可能了，怎么办呢？？

先去异常处理函数 `0x006a0046` 看下吧，`pop pop ret`，没什么问题，典型的利用异常跳转到 `Shellcode` 的利用方式。如果要分析一下 `Shellcode`，那么 `bp 0x006a0046`、`shift+F9` 然后单步就一切了然，这不是这里要讨论的，其实对于这个样本也没有什么定位 `Shellcode` 的需要，利用程序(`bhk.py`)中不是都写明了 `Shellcode` 嘛。不过利用程序中的 `align`、`control` 所对应的汇编代码还是有一定参考价值的，演示了成功利用漏洞所应控制的细节方面。

![](img/e2a16de22e07a2ee34f1b0473715911e.jpg)

## 2. 变得有趣之前

这里我不是想要定位并分析 `Shellcode`, 而是想要了解到底是什么原因导致程序跳转去 `shellcode` 的，现在我们已经知道这是一个栈溢出了，什么地方导致了栈溢出呢？

从 `OD` 异常中断的地方入手吧，异常地址为 `0x00410048`，且异常时 `esp` 附近规规整整地排好了 `0x00410041`，嗯…推断一下前几步程序的执行，我们有理由相信一个函数返回到了 `0x00410041`，几步后，砰…异常了。真的是这样的。

好的，既然我们想在这个时候挖掘一点问题函数前面几个函数的信息，而保存这些信息的栈区被我们这个样本的数据覆盖了，我们要程序触发异常，却要保留这些信息，怎么办？好的，我这里是这样操作的…

我改短了用于覆盖栈区的数据的长度，用记事本打开 `Foxit-exploit.pdf`，在文件结尾处可以看见我们的覆盖数据，大量的`'A'`加大量的“无意义数据”，根据上面图中反映的情况，我们把覆盖数据后 10 个`'A'`和后面的“无意义数据”去掉吧，不一定是 10 个，反正大概算一下，使得数据既能覆盖返回地址（上图中的 `esp-4`）又少覆盖栈中数据即可，改后保存为文件 `crash.pdf`。

同样的调试步骤，这次打开文件 `crash.pdf`，异常中断到 `OD` 时栈区为这样：

![](img/f9762ac6c186014a18758a8a5dc02781.jpg)

怎么样，去 `0x96f6e6` 看看吧，

```
0096F6DE  |. 56            push    esi
0096F6DF  |. 8BCF          mov     ecx, edi
0096F6E1  |. E8 A7330000   call    00972A8D
0096F6E6  |. 8BC7          mov     eax, edi
0096F6E8  |. 5F            pop     edi
0096F6E9  |. 5E            pop     esi
0096F6EA  \. C2 0400       retn    4
```

`Ctrl+F2` 重新运行调试程序，`F9` 让程序运行，界面完全出现后，`bp 0x0096F6E1`… 唉，这里怎么说呢，反正有兴趣的朋友去实际试试吧，在 `0x00972A8D` 这个函数体内，很多地方尝试下断，程序都会一直断，可能我们真是走错地方了。

没关系，在数字迷宫中摸爬，我已习惯了头破血流，哈… 几次尝试后（几十次？it doesn't matter）我决定退一步看看，`0x00979905`：

```
009798F9  |> \8B10          |mov     edx, dword ptr [eax]
009798FB  |. 6A 01         |push    1
009798FD  |. 8BC8          |mov     ecx, eax
009798FF  |. FF92 E0000000 |call    dword ptr [edx+E0]
00979905  |. 43            |inc     ebx
00979906  |>  837D FC 00    |cmp     dword ptr [ebp-4], 0
0097990A  |.^ 75 B9         \jnz     short 009798C5
```

下断 `0x9798ff`，嗯，怎么动 `FoxitReader` 都没断下，现在打开 `crash.pdf` 试试，砰，断下了，事情似乎发展得很顺嘛。`F7` 去了 `0x00402e60`

```
00402E60   . 6A FF         push    -1
00402E62   . 68 701D9A00   push    009A1D70               ;  SE 处理程序安装
00402E67   . 64:A1 0000000>mov     eax, dword ptr fs:[0]
00402E6D   . 50            push    eax
00402E6E   . 64:8925 00000>mov     dword ptr fs:[0], esp
00402E75   . 83EC 18       sub     esp, 18
00402E78   . 8B4424 28     mov     eax, dword ptr [esp+28]
00402E7C   . 53            push    ebx
00402E7D   . 56            push    esi
00402E7E   . 57            push    edi
00402E7F   . 8BD9          mov     ebx, ecx
00402E81   . 50            push    eax
00402E82   . E8 04415800   call    00986F8B
00402E87   . E8 74110700   call    00474000
00402E8C   . 8BF8          mov     edi, eax
00402E8E   . 85FF          test    edi, edi
00402E90   . 897C24 14     mov     dword ptr [esp+14], edi
00402E94   . 0F84 6B010000 je      00403005
```

`00402E82` 处步过的时候程序异常了，重新来过，这次在这里 `F7`，进去 `0x00986F8B`，上下打量一翻，一个“可爱”的函数映入了眼帘：`lstrcpyW`，接下来的分析验证就简单了，经过确认，事故责任人就是它….

![](img/9d11c6a22f4537e6c240f8a3135027b7.jpg)

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || 样本及分析.rar(解压密码：`pediy`) |
