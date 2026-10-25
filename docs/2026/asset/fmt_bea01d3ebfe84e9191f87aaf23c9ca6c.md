
# 破解 Doom 为了乐趣，血量和弹药

原文链接: [d54K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3&6K6k6i4m8G2M7%4c8Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3x3U0l9I4z5g2)9J5c8X3S2S2j5$3E0A6L8X3N6Q4x3X3c8V1L8$3!0E0i4K6u0V1k6X3!0J5i4K6u0V1k6Y4g2F1i4K6u0V1K9r3g2S2L8s2c8Z5i4K6u0V1j5h3&6V1i4K6u0V1j5h3#2E0L8#2)9J5c8R3`.`](<elink@2b0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3&6K6k6i4m8G2M7%4c8Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3x3U0l9I4z5g2)9J5c8X3S2S2j5$3E0A6L8X3N6Q4x3X3c8V1L8$3!0E0i4K6u0V1k6X3!0J5i4K6u0V1k6Y4g2F1i4K6u0V1K9r3g2S2L8s2c8Z5i4K6u0V1j5h3&6V1i4K6u0V1j5h3#2E0L8#2)9J5c8R3..>)

翻译: 看雪翻译小组 - Nxe

校对: lipss

------

还记得 `iddqd` 和 `idkfa`吗? 这两个字符串自我年幼的时候就深深刻入我的脑海, 那时的我特别喜欢玩共享版的毁灭战士(Doom).

在 SenseCon'19 上, 我和 Lauren 以及 Reino 三人逆向研究了 [chocolate-doom](<elink@eafK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0K9r3!0U0L8$3I4S2N6r3g2Q4x3X3c8V1L8$3!0E0i4K6u0W2L8%4u0Y4i4K6u0r3N6$3W2C8K9g2)9J5c8X3W2F1k6r3g2^5i4K6u0W2M7r3S2H3i4K6u0r3b7$3S2G2j5$3!0D9j5i4c8W2i4K6g2X3c8r3!0G2L8b7..>), 从而重现相似的作弊效果. 结果如何呢? 下面有一个展示视频. 我们设法做到了以下作弊功能:

- 开枪不会减少弹药而是会增加弹药

- 所有人的血量不会减少而是会增加. 是的, 你没看错, 所有人.

- 和经典的毁灭战士一样切换作弊

如果你想使用我们的作弊代码, 在[这里](<elink@b1bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6k6h3&6K6k6i4m8G2M7%4c8Q4x3V1k6X3M7X3W2V1j5g2)9J5k6s2N6A6L8X3c8G2N6%4y4Q4x3X3c8H3L8r3q4&6k6%4u0G2N6h3&6V1i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3c8G2L8$3#2Q4x3X3c8U0K9r3g2S2N6s2y4Q4x3X3g2B7M7H3..>)下载, 你也可以贡献新的代码 :)

[![Youtube演示视频](upload/attach/202001/820961\_VJ6GF4Y2GSD9QNH.jpg)](<elink@f84K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2&6L8%4g2@1N6h3u0W2i4K6u0W2j5$3!0E0i4K6u0r3k6h3#2T1k6h3c8Q4x3V1k6A6L8q4c8d9y4q4y4S2x3@1u0V1z5q4)9J5z5b7..>)

## 安装

原本的毁灭战士发布于[1993](<elink@e5aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6p5L8$3!0E0i4K6g2X3i4K6t1^5k6Y4u0S2L8X3y4Z5K9i4y4W2>)#Main_series), 运行系统和我们现在用的有些不同. [chocolate-doom](<elink@88bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0K9r3!0U0L8$3I4S2N6r3g2Q4x3X3c8V1L8$3!0E0i4K6u0r3j5$3S2G2j5$3!0D9j5i4c8W2i4K6u0V1k6r3!0G2L8b7..>)项目旨在我们现代的操作系统上尽可能贴近原版地正常运行游戏. 完美怀旧. 我们从[这里](<elink@bb9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0K9r3!0U0L8$3I4S2N6r3g2Q4x3X3c8V1L8$3!0E0i4K6u0W2L8%4u0Y4i4K6u0r3k6r3!0%4L8X3I4G2j5h3c8K6i4K6u0r3x3#2)9J5k6e0m8Q4x3X3f1H3i4K6u0r3j5$3S2G2j5$3!0D9j5i4c8W2i4K6u0V1k6r3!0G2L8g2)9J5k6o6y4Q4x3X3f1H3i4K6u0W2x3q4)9J5k6s2N6A6L8U0x3J5i4K6u0W2P5X3W2H3>)下载 chocolate-doom, 解压并获取一个[共享的 WAD 文件](<elink@326K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3!0E0N6$3W2C8K9g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6p5e0@1!0y4x3g2)9J5k6g2N6m8c8l9..>)来使用.

我们也给我们的项目订了些规矩. chocolate-doom 是开源的, 然而我们一点也不想参考. 解压之后, `chocolate-doom.exe` 是一个 stripped PE32 可执行文件. 这意味着逆向工程可能会更难一些, 但这正是我们想要的学习和挑战中的一部分. 使用诸如 [IDA 免费版](<elink@69aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2Z5k6i4S2Q4x3X3c8J5j5i4W2K6i4K6u0W2j5$3!0E0i4K6u0r3M7s2u0G2k6s2g2U0N6s2y4Q4x3V1k6A6k6r3q4Q4x3V1k6K6N6i4m8H3L8%4u0@1i4K6u0r3k6r3!0%4L8X3I4G2j5h3c8Q4y4h3k6X3M7X3g2W2N6$3q4J5k6g2)9J5k6i4y4Z5N6r3#2D9>), [CheatEngine](<elink@44dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0K9r3g2S2N6r3g2F1k6$3W2F1k6g2)9J5k6h3!0J5k6#2)9J5c8R3..>) 和 [WindDBG](<elink@05dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3X3c8Z5j5i4u0V1N6$3q4J5k6g2)9J5c8X3c8J5K9i4k6W2M7Y4y4Q4x3V1k6V1k6h3u0#2k6$3N6W2M7W2)9J5c8X3c8W2j5Y4g2Y4k6$3g2J5i4K6u0V1k6r3!0%4L8X3I4G2j5h3c8Q4x3X3c8@1L8$3!0D9M7H3..>) 一类的工具完成这个挑战应该算是公平. 不过任何补丁或者二进制修改是使用 Frida 实现的, 而不是手动给 `chocolate-doom.exe` 打补丁.

## 寻找切入点 - Windows

万事开头难, 我们打算从使用 CheatEngine 开始, 根据游戏的界面找到一些有趣的代码位置.

首先是找出负责弹药数的代码. CheatEngine 是一款特别适合这个任务的内存扫描器和调试器. 你可以将 CheatEngine 附加到某个运行的进程上, 然后扫描这个进程的内存, 从而发现特定数值的所有实例. 如果我们目前的弹药数是49，我们可以在内存中搜索值是49的所有实例。不过进程内存中有可能扫出许多跟数值相关的实例 - 一次扫描通常会返回许多个. 不过不是所有实例都与弹药数有关。

![](upload/attach/202001/820961\_MMHYJHXZDS6WBJ5.jpg)

搜索值 49 返回了许多内存位置

为了找出正确的位置, 我们可以对该值稍作变更, 然后使用 CheatEngine 重新扫描, 查看之前找到的实例的值是否有随之发生相同变化的. 我们可以在游戏中用手枪射击几次, 注意弹药量的变化. 然后在 CheatEngine 使用 "下一次搜索(Next Scan)" 功能和 "减少数值(Decreased value by)" 扫描类型选项来找到发生相同变化的值. 如此操作之后, 可能负责弹药量的位置减少到仅 3 处了.

![](upload/attach/202001/820961\_CKPEHA8PW8T6Y84.jpg)

仅有3处可能负责弹药量的位置

此时所有可能的实例可能是弹药量的原始值, 或是该原始值的拷贝. 我们可以观察这些内存位置, 来确定哪些指令对这些地址进行了写操作, 从而尝试找出负责减少弹药量的代码. 为了用 CheatEngine 做到这点, 你只需右键点击观察的内存位置, 然后选择 "找出何指令写入该地址(Find out what writes to this address)".

![](upload/attach/202001/820961\_R2QQY3N4TTFDRSD.jpg)

观察对内存位置的更新

当观察我们找到的前两个位置时, 我们发现即使在游戏暂停时, 这些指令也会快速地写入目标指针. 而且这些指令也没有减去弹药量, 这意味着这些指令可能不是我们所寻找的.

![](upload/attach/202001/820961\_YB4F2PUN44FAHQX.jpg)

观察的内存位置上不断快速运行的指令

第三个匹配仅在手枪射击时进行写入. 该指令进行减一操作, 因此它可能是我们感兴趣的进行减少弹药量的指令. CheatEngine 可以反编译发现的指令周围的代码. 利用这个功能, 我们得到了在手枪射击时负责减少弹药量的指令的位置 `0x0430A2c`.

![](upload/attach/202001/820961\_JE666ECPD4FS23M.jpg)

减少1发手枪弹药的指令

## 寻找切入点 - Linux

尽管不是我们项目的主要目标, 我们也尝试了对 `chocolate-doom` ELF 文件打补丁. 我们使用了类似于上文提到的工具, 然而 CheatEngine 在 Linux 上用不了. 所以我们使用了类似 CheatEngine 功能的[scanmem](<elink@915K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6j5$3q4F1L8h3g2E0i4K6u0r3M7$3y4S2L8X3#2W2L8b7..>)来代替它来找到有趣的代码位置, 但是 `scanmem` 只能给出某个值在内存中的地址, 无法给出修改某个值的指令. 在运行 `scanmem` 并多次输入改变的弹药量的值后, 它也筛选出了3个可能的弹药量地址. 因为 ASLR 机制, 每次运行 `chocolate-doom` 时这些地址可能会有所改变.

![](upload/attach/202001/820961\_TG6EY4XC8HCW4TR.jpg)

Scanmem 显示保存现在弹药量的值的地址

为了找出哪个指令向找到的指针有写入操作, 我们使用了 `gdb` 并在 `scanmem` 找到了每个地址处都设置了断点.

![](upload/attach/202001/820961\_BTURMYMGRYKQ8KT.jpg)

设置的弹药量减少的 Watchpoints

我们之后继续游戏, 用手枪射击一次来触发 watchpoint. 下图中你可以看到之前和现在的弹药量的值.

![](upload/attach/202001/820961\_RAMWEN6JM6G5PP6.jpg)

在手枪射击后触发的 Watchpoint

下一步是检查指令指针在哪里, 然后检查它周围的指令, 看能否找到使每次射击后弹药量值减一的确切指令.

![](upload/attach/202001/820961\_HCVCEFXAU2D7HEZ.jpg)

`rip` 和周围的指令

你可以看到, 在当前所停的指令指针的前一个位置正好是一个 `sub` 指令. 为了在 IDA 中搜索确切指令, 我们查看 `gdb` 中该指令的16进制值.

![](upload/attach/202001/820961\_H9AVE7TCPNRB4G9.jpg)

以16进制查看 `sub` 指令

因为下面的值是小端序的, 我们在 IDA 中搜索操作码`83 ac 83 a8`然后得到了负责减少弹药量的指令位置的偏移量, 也就是`0x49F28`.

## Frida 中查看函数

有了一些目标的偏移量以后, 我们可以使用 Frida 进行实时的观察. Frida 有 [Interceptor API](<elink@c06K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3M7X3W2V1j5g2)9J5k6i4u0W2i4K6u0r3k6r3!0U0M7#2)9J5c8X3A6S2N6X3q4K6j5%4u0A6M7s2c8Q4x3X3c8S2M7r3W2Q4x3V1k6Q4x3U0y4A6L8Y4c8W2M7X3y4W2M7s2c8G2M7R3..>), 允许你"拦截"函数调用以及像函数前导指令(prolog)和后导指令(epilog)一样运行代码. 使用 `拦截器(Interceptor)` 可以访问到参数和返回值, 使得能够记录这些数值, 需要的话还可以进行修改.

因为我们知道哪个指令正在写入包含手枪弹药值的内存区域, 我们使用 IDA 分析这个该指令周围的函数来找到入口点. 这样我们就能够知道搭配拦截器使用的内存地址.

![](upload/attach/202001/820961\_DRP7RWJJTGDKAKA.jpg)

影响手枪弹药的函数入口点

在上面的截图中可以看到, 函数起始于`0x4309f0`. 基地址`0x400000`意味着该函数偏移量为`0x309f0`(这个值一会很重要). 知道了偏移量, 我们的第一个 Frida 脚本就可以开工了. 这个脚本仅用来找到 `chocolate-doom` 模块的基地址, 计算我们感兴趣的函数的偏移量并附加拦截器.

![](upload/attach/202001/820961\_9GMG9VXUXZSM572.jpg)

附加在手枪射击函数的入口点的拦截器

附加到 `chocolate-doom` 以后, 我们可以在下面视频中看到该脚本的运行情况.

[![Youtube演示视频](upload/attach/202001/820961\_S8ATPRRVNGFGFY3.jpg)](<elink@298K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2&6L8%4g2@1N6h3u0W2i4K6u0W2j5$3!0E0i4K6u0r3k6h3#2T1k6h3c8Q4x3V1k6#2N6g2u0t1k6#2g2B7z5p5I4h3M7#2)9J5z5b7..>)

当我们用手枪射击时触发了函数, 证实了我们没走错路

## 第一个作弊

我们想先写一个简单的修补开始, 它仅将弹药量减1的指令替换为 `NOP` 指令. Frida 的 [Code writer](<elink@e51K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3M7X3W2V1j5g2)9J5k6i4u0W2i4K6u0r3k6r3!0U0M7#2)9J5c8X3A6S2N6X3q4K6j5%4u0A6M7s2c8Q4x3X3c8S2M7r3W2Q4x3V1k6Q4x3U0y4^5z5o6k6%4M7X3W2@1k6i4t1..>) 模块可以帮我们做到这一点. 原本的递减指令 `sub dword ptr [ebx+edx*4+0A4h], 1` 以十六进制来看由8个操作码组成.

![](upload/attach/202001/820961\_GKEKQVG58D7S4H2.jpg)

递减手枪弹药量指令的16进制操作码

可以使用 code writer 实例中的 `putNopPadding(8)` 模式在`0x430a2c`位置填充8个 `NOP` 指令.

![](upload/attach/202001/820961\_AC35PQ54MPKZYAU.jpg)

Frida code writer 用来将弹药递减指令替换为8个 NOP.

应用这个修补意味着我们再也用不完手枪的弹药了.
