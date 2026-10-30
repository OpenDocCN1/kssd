
# [翻译]利用 QuickZip 4.60 的缓冲区溢出漏洞在 Win7 X64 系统上弹出计算器

最近我在准备[OSCE](https://bbs.kanxue.com/elink@7c4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2G2k6X3k6W2L8Y4y4A6N6X3g2Q4x3X3c8K6k6h3y4#2M7X3W2@1P5g2)9J5k6h3y4G2L8g2)9J5c8X3W2F1k6X3!0J5L8h3q4@1K9h3!0F1i4K6u0V1M7$3g2U0N6i4u0A6N6s2W2Q4x3X3c8U0k6i4u0@1K9h3k6A6j5$3q4@1K9h3!0F1M7#2)9J5c8X3!0K6j5$3g2Q4x3X3c8G2k6X3k6W2L8Y4y4A6N6X3g2Q4x3X3c8K6k6h3y4#2M7X3W2@1P5g2)9J5k6r3y4W2M7Y4c8A6k6X3W2W2k6q4)9J5k6r3g2^5M7r3g2J5N6q4)9J5c8R3`.`.) exam, 所以一直想找一些有趣的漏洞和POC代码来练习一下，顺便在漏洞方面学习一些新的东西。

在寻觅了一会后，发现了 [QuickZip v4.60 Buffer Overflow exploit](https://bbs.kanxue.com/elink@cc6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2P5s2m8D9L8$3W2@1i4K6u0V1k6r3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2P5s2m8D9L8$3W2@1M7#2)9J5c8U0p5I4y4U0f1$3i4K6u0r3), 是 [corelanc0d3r](https://bbs.kanxue.com/elink@e35K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3j5$3!0J5k6h3I4S2L8X3x3H3k6o6y4J5)在[这篇博客](https://bbs.kanxue.com/elink@a33K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8%4u0W2L8r3q4F1i4K6u0W2j5X3g2Q4x3V1k6A6L8X3c8W2P5q4)9J5k6i4m8Z5M7q4)9J5c8U0t1H3x3e0m8Q4x3V1j5H3x3#2)9J5c8U0t1%4i4K6u0r3M7i4g2A6j5$3E0*7K9i4m8Q4x3X3c8K6N6r3q4U0K9#2)9J5k6r3u0G2k6W2)9J5k6o6m8V1j5i4W2Q4x3X3c8S2i4K6u0V1j5X3!0^5i4K6u0V1L8$3k6Q4x3X3c8U0K9r3!0U0L8$3I4S2N6r3g2K6i4K6u0r3)里记录的漏洞。

因为这个漏洞是2010年的，是为了32 位的`window-xp` 设计的。我试下能否在64位的`windows 7` 操作系统上重现它，这会是一个有趣的挑战！

## PoC

起初，我先获取了 [QuickZip v4.60 Windows XP exploit from exploit-db](https://bbs.kanxue.com/elink@d78K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2P5s2m8D9L8$3W2@1i4K6u0V1k6r3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2P5s2m8D9L8$3W2@1M7#2)9J5c8U0p5I4y4U0f1$3i4K6u0r3), 然后分成几部分来写一个简单的POC，从而触发崩溃。

上面的代码创建了一个名为`4064A;`的压缩文件，后缀名为 `txt`; `Header_1`, `header_2` 和 `header_3` 是`zip` 文件结构要求的文件头，我不会深入探讨这个问题。你想了解更多的话，看 [这里](https://bbs.kanxue.com/elink@572K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6K9K9i4m8Q4y4h3k6Q4x3U0S2X3K9h3I4W2i4K6g2X3k6X3!0J5L8h3q4@1)。

如果你在`QuickZip` 中打开创建的`ZIP`文件，尝试提取其内容（只是双击文件名），`QuickZip`就会崩溃。

## 理解崩溃过程

好的，让我们来运行一下POC，看看究竟会发生什么。

用上面的`python`代码创建`zip`文件，用`QuickZip`打开它，打开 `ImmunityDebugger`, 附加到`QuickZip` 进程上，在`QuickZip` 中，双击文件名来引发崩溃。 **注意:** 我们会反复不厌其烦地重复这个过程，所以习惯它吧！

![](https://bbs.kanxue.com/upload/attach/201705/726271_mr3lig8ko9u6r9c.png)

非常好，我们像预期那样引发了崩溃。同时，我们得到了一个异常 - 看屏幕的底部 *; 写入[`00190000`];*时访问冲突。这说明我们向无效的内存地址写入时引发了一个异常。

让我们来看看`SHE`（结构化异常处理）链。

![](https://bbs.kanxue.com/upload/attach/201705/726271_3bbjl16bzxrd9n5.png)

非常棒，上图表明我们能够控制`nSHE` 指针! 看起来很有希望，让我们来试试找出偏移量。

## 偏移量

和往常一样，我还是使用`mona` ([55eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8%4u0W2L8r3q4F1i4K6u0r3L8h3!0F1j5b7`.`.](https://bbs.kanxue.com/elink@8acK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8%4u0W2L8r3q4F1i4K6u0r3L8h3!0F1j5b7`.`.)) 来解决这里的问题。

首先，先生成一个有**4064** 个特殊字符的样例，把它放在POC 利用的`payload`上:

再次引发崩溃，看看这次发生了什么。

![](https://bbs.kanxue.com/upload/attach/201705/726271_jw3pudtnjp0cpaa.png)

这次与上次不太一样了。 `LEAVE` 指令尝试从堆栈跳转回一段无效的内存地址`0EEDFADE`。

但，我们确不再能够控制`SHE`了。

![](https://bbs.kanxue.com/upload/attach/201705/726271_7gu0apotdi9m54x.png)

然而，注意到我们实际是在内核模块(看下`Immunity` 窗口的名称-`*CPU 主线程，KERNELBA 模块*`)。通过 `SHIFT + F9` 返回程序执行的上一步，看看在`QuickZip` 模块中能否引发异常情况。

![](https://bbs.kanxue.com/upload/attach/201705/726271_mvzxb5tqx3wvova.png)

![](https://bbs.kanxue.com/upload/attach/201705/726271_wza8k1fok56o51q.png)

很好，看来我们又回来了！

用下面的命令让`mona`计算偏移量:

![](https://bbs.kanxue.com/upload/attach/201705/726271_hny156jki0ky127.png)

到这以后，一个很有意思的事情是 `nSEH字段：偏移量 292`。

让我们用偏移信息更新下POC，再次尝试触发崩溃。

![](https://bbs.kanxue.com/upload/attach/201705/726271_9ye7arwilfvk4zx.png)

太好了，我们控制了`SEH`！让我们跳过异常程序(`SHIFT + F9`) 并进一步看下发生了什么。

![](https://bbs.kanxue.com/upload/attach/201705/726271_ctaq09h94ccbog1.png)

当然了，另一个异常程序也触发了，因为`43434343`对于这个程序来说是一段无效的内存地址，但是我们来看看在栈上发生了什么--通常为`SEH`溢出，我们需要调用一组`*POP-POP-RET*`指令来返回到我们的缓冲区。

很容易用`mona`找到这样的指令,但是首先我们必须知道哪些指令是被允许使用的。而这才是问题的关键。

## 坏字符

大体上来说，大部分是这样的。为什么？因为我们在文件名参数和文件名上面的溢出是被严格限定的-------通常只有可打印的`ASCII`字符。

因为实际上如果手工通过`mona` 找到所有坏的字符需要很长时间，我只是假设使用了整张`ASCII` 表（字符最多为127个）除了`0x00`, `0x0a` 和 `0x0d`(`NULL`字节，换行回车符)。

这个假设可能会使问题变难（因为我没有使用那些本来应该没有问题的字符），或者会导致更多后续的问题，如果我的假设的字符范围是不对的话。

我不喜欢做出这样的假设，但是出于练习的原因，这次姑且做个假设吧。

我只需要记住要小心，如果有什么问题的话，再次检查下坏的字符。有点冒险，但没关系，来吧！

## POP-POP-RET

让我们用`mona` 找到一个易于使用的`*POP-POP-RET*` 指令地址：

![](https://bbs.kanxue.com/upload/attach/201705/726271_j3j5eeqyiyajtsu.png)

发现了很多东西（7909个！），但是突出显示的看起来更让人兴奋------全部由数字字母这样的字符组成，出现在`QuickZip.exe`二进制文件中，希望这能够使它更加地跨平台，而我们就不必依赖于特定操作系统的`DLL`库了。

这里只有一个问题，就是 `0x00` 字节。又因为程序地址全部以`0x00`开始 我们来试试看这是否会破坏我们的漏洞。

更新下`PoC`漏洞，用 `\x33\x28\x42\x00`替换目前代表`SEH`的`CCCC` 再次触发崩溃，调查下`SEH`链。

![](https://bbs.kanxue.com/upload/attach/201705/726271_iai5vdw3kxh8e8f.png)

很好，看来我们的地址没有乱码，正如我们所预料的那样。设置断点 (`F2`) ，按 `SHIFT + F9`将控件传给程序。

![](https://bbs.kanxue.com/upload/attach/201705/726271_bbbeckeity9x4gv.png)

如你所见，我们被重定向到了`*POP-POP-RET*` 指令，通过`F8` 单步步过， 然后在 `RETN 4` 指令后停下。

![](https://bbs.kanxue.com/upload/attach/201705/726271_pv2powxhgiu2sn1.png)

太好了，我们又重新地找到了我们的有效数据。但是有个问题，由于存在`NULL`字节，每个`SEH`链后面都被切断了，所以我们只有很少的空间来完成任务了。

## shellcode 哪里去了？！

好的，我们来分析下目前处于哪一阶段。

程序崩溃，然后控制了`SEH`，太棒了！问题是我们被限制在一个非常有限的字符集里，当使用`payload`时，因为必须使用`NULL` 字节的地址来调用`*POP-POP-RET*`指令 ，我们`payload` 中的重要部分被截断了，我们的`shellcode`剩余的空间一点也不大。

但是到底有多大呢？记住我们还有`payload`开始时使用的填充为了获得`SEH`：

![](https://bbs.kanxue.com/upload/attach/201705/726271_6meulf9xkcn47p4.png)

所以我们有多少空间呢？正好是**292**个字节。不幸的是，对于 任何有用的`shellcode`--只包含可打印的 `ASCII`码字符并且需要加密，都是 不够的。

这听起来可以用`egghunter`来解决!

**Egghunter** 是一堆在程序内存空间中查找一个特定的，已知的字节序列的指令，一旦找到的话，将重定向到该区域。这样的话，我们不用担心`shellcode`在哪里结束了，只需要调用`egghunter`例程，它会为我们找到的！

听起来不错，接下来的问题是，`payload` 中被截断的部分在内存的哪里呢?我们来找找看。

让我们生成**3764**个唯一字符的模式（在`NULL`字节后填充`payload`），并用它替换现有的`A`。

触发崩溃，当第一次出现 异常时，不要将异常传给程序，而是调用下面的命令在内存中搜索以前生成的模式：

![](https://bbs.kanxue.com/upload/attach/201705/726271_ut1vq9qmx4z6a89.png)

太好了！`payload`中被截断的部分仍然在内存中，所以我们能够成功地用`egghunter`来获取到`shellcode`。

## Egghunter

所以现在我们知道我们应该能够使用`egghunter`来获取我们的`shellcode`，但是我们只有**292**个字节可供我们使用。实际上，292字节做很多事儿了，但是，我们得记住只能使用非常有限的字符集。

我们试着用`metasploit`的`x86/alpha_mixed`编码器对`egghunter`进行编码，看看在这之后剩下多少空间。

首先，让我们生成`egghunter` 的有效载荷。牢记现在是64位的操作系统，所以我们需要使用正确的`egghunter` 例程 (更多细节可以在[59dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8%4u0W2L8r3q4F1i4K6u0W2j5X3g2Q4x3V1k6A6L8X3c8W2P5q4)9J5k6i4m8Z5M7q4)9J5c8U0t1H3x3e0q4Q4x3V1j5I4x3g2)9J5c8U0p5^5i4K6u0r3N6$3!0%4y4U0c8Q4x3X3c8W2k6$3N6Z5N6h3&6@1k6i4u0Q4x3V1k6Q4x3U0V1`.](https://bbs.kanxue.com/elink@cafK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8%4u0W2L8r3q4F1i4K6u0W2j5X3g2Q4x3V1k6A6L8X3c8W2P5q4)9J5k6i4m8Z5M7q4)9J5c8U0t1H3x3e0q4Q4x3V1j5I4x3g2)9J5c8U0p5^5i4K6u0r3N6$3!0%4y4U0c8Q4x3X3c8W2k6$3N6Z5N6h3&6@1k6i4u0Q4x3V1j5`.)找到：

把生成的字节拷贝到一个文本文件中，用`xxd`把它转换为二进制文件：

现在，我们需要用编码器来确保只用了可打印的`ASCII`码 字符。

*注意：* 我已经用了 `bufferedregister=eax` 选项。原因在于需要知道编码器在内存中的位置，以便于能对有效载荷解码。最初，负责这个的例程不在 可打印的`ASCII`码字符集中，因此这会毁掉我们的`payload`。

指定 `bufferregister`选项只是告诉编码器不用担心找不到它在内存中的位置，因为已经提前把它的地址放在了`EAX` 寄存器中。这样的话，我们的编码器`egghunter`就只含有`ASCII`码字符了（更多关于生成只包含字母和数字的`shellcode`可以在 [这里](https://bbs.kanxue.com/elink@947K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2G2k6X3k6W2L8Y4y4A6N6X3g2Q4x3X3c8K6k6h3y4#2M7X3W2@1P5g2)9J5k6h3y4G2L8g2)9J5c8X3#2W2N6r3q4K6M7r3I4G2K9i4c8Q4x3X3c8#2L8X3I4W2j5i4y4Z5k6h3c8Q4x3V1k6S2L8s2m8Z5j5h3&6#2L8h3g2J5K9h3y4Q4x3X3c8K6K9r3g2D9L8r3y4G2k6r3g2Q4x3V1j5`.)找到)。

更新下`POC`漏洞，看看目前为止做到哪了。

让我们触发崩溃，通过执行程序执行`POP-POP-RET` 指令。在这之后，在`CPU`窗口中向上滚动，尝试找到`egghunter` 的有效载荷和长指令集`INC ECX`指令的结束（代表`A`字符）。

![](https://bbs.kanxue.com/upload/attach/201705/726271_mi3gsed4i1at9t7.png)

非常好，看起来就是这里，都是和预期想的一样，没有错误-也没有坏字符出现！

**跳回**
