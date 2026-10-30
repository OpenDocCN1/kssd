
# [原创]对Easy RM to MP3 Converter 2.7.3.700的栈溢出漏洞分析

说实话自己一直对如何定位漏洞点很头痛，看着别人下各种各样的断点，栈回溯等技巧，总是觉得似懂非懂。不同的软件，不同的漏洞类型有不同的技巧，这需要慢慢长期积累经验。

最近偶然看到一篇文章，《漏洞分析第一次--漏洞发现》，11年写的，作者当时还是新手，在大牛K\_K的指导下完成。分析的是一个普通的栈溢出，原文地址在这里：e27K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2A6i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6U0K9r3W2D9L8q4)9#2k6X3u0#2M7s2c8Q4x3V1k6A6N6r3g2E0i4K6u0r3k6o6f1@1x3r3j5@1x3K6V1@1j5U0j5&6z5e0g2V1j5K6x3&6x3X3k6X3j5e0m8U0i4@1f1K6i4K6R3H3i4K6R3J5i4@1f1%4i4K6W2o6i4K6S2n7i4@1f1#2i4@1q4q4i4K6S2o6i4@1f1#2i4K6V1H3i4K6S2q4i4@1f1^5i4K6R3%4i4@1q4m8i4@1f1#2i4@1t1%4i4@1t1I4i4@1f1@1i4@1t1&6i4K6W2r3i4@1f1#2i4K6S2r3i4K6V1%4i4@1f1#2i4K6R3^5i4@1t1H3i4@1f1@1i4@1t1^5i4K6R3H3i4@1f1@1i4@1u0m8i4K6W2n7i4@1f1#2i4K6V1H3i4@1q4r3i4@1f1#2i4K6S2r3i4K6V1I4i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1$3i4K6R3@1i4K6W2r3i4@1f1^5i4@1p5%4i4K6R3&6i4@1f1$3i4K6S2o6i4K6R3&6i4@1f1%4i4K6R3#2i4@1p5%4d9#2)9#2k6V1E0Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0k6Q4z5o6m8Q4z5f1c8Q4c8e0S2Q4b7U0N6Q4b7f1k6Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0g2Q4b7V1g2Q4z5o6S2Q4c8e0g2Q4b7e0c8Q4z5f1q4Q4c8e0k6Q4b7e0m8Q4z5o6S2Q4c8e0k6Q4b7V1q4Q4b7e0u0Q4c8e0g2Q4z5o6N6Q4b7V1q4Q4c8e0N6Q4b7U0q4Q4b7V1u0Q4c8e0g2Q4z5f1g2Q4z5p5u0Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0k6Q4b7V1y4Q4z5p5k6Q4c8e0k6Q4b7U0c8Q4z5f1g2Q4c8e0W2Q4z5o6y4Q4b7V1c8Q4c8e0g2Q4z5p5k6Q4b7f1k6Q4c8e0c8Q4b7V1u0Q4b7e0g2Q4c8e0N6Q4z5e0c8Q4b7e0S2Q4c8e0k6Q4b7f1c8Q4b7e0c8Q4c8e0k6Q4z5e0k6Q4b7U0W2Q4c8e0g2Q4b7V1y4Q4z5p5k6Q4c8e0k6Q4z5f1c8Q4b7e0g2Q4c8e0g2Q4z5o6S2Q4z5o6k6Q4c8e0k6Q4z5f1g2Q4z5e0m8Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0k6Q4z5o6W2Q4z5o6m8Q4c8e0c8Q4b7V1u0Q4b7e0g2Q4c8e0S2Q4z5o6N6Q4b7f1q4Q4c8e0g2Q4b7U0N6Q4b7U0q4Q4c8e0c8Q4b7U0W2Q4z5f1k6Q4c8e0S2Q4b7f1k6Q4z5e0g2Q4c8e0N6Q4z5f1c8Q4z5o6m8Q4c8e0g2Q4z5o6S2Q4z5o6k6Q4c8e0k6Q4z5f1g2Q4z5e0m8Q4c8e0c8Q4b7V1q4Q4z5o6k6Q4c8e0c8Q4b7U0S2Q4z5o6m8Q4c8e0c8Q4b7U0S2Q4b7f1q4Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0g2Q4b7U0m8Q4b7U0q4Q4c8e0k6Q4z5e0S2Q4b7f1k6W2P5s2m8D9L8$3W2@1i4@1f1%4i4@1u0o6i4K6V1$3i4@1f1#2i4K6R3$3i4K6V1&6i4@1f1%4i4@1t1K6i4@1u0n7i4@1f1#2i4K6R3^5i4K6V1%4i4@1f1$3i4K6V1#2i4K6V1&6i4@1f1%4i4@1p5^5i4K6S2n7i4@1f1%4i4@1q4o6i4@1q4o6i4@1f1@1i4@1t1^5i4K6R3H3i4@1f1%4i4@1q4r3i4K6R3%4i4@1f1&6i4K6R3%4i4K6S2o6i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1&6i4K6R3J5i4@1p5K6i4@1f1@1i4@1t1^5i4@1q4m8c8h3q4K6P5b7`.`. RM to MP3 Converter 2.7.3.700的栈溢出漏洞。我下载了原作者的exploit，发现在windows xp sp3简体中文版下已经不能运行（当时分析的环境是windows xp sp2 en），并且在调试中发现缓冲区大小也和原作者的分析有差异（他用的是metasploit的一个小工具计算的），于是自己从头到尾分析了这个漏洞，并写出弹计算器的exploit。

软件我上传到了附件中。

还记得当时的场景吗？构造一个恶意的m3u文件，填了20000个A，没崩，填了30000个，崩掉了，eip被改为41414141。我就拿这个填了30000个A的m3u来分析。

## 环境

环境：`windows xp sp3` professional 简体中文版（虚拟机下）

调试器：`Immunity Debugger 1.85`

用调试器打开Easy RM to MP3 Converter，加载m3u文件，崩掉，eip被改为41414141。

很显然崩溃是因为返回这个问题函数的上层函数的时候发生的。虽然返回失败，但这个问题函数还会调用其他函数，只要调用了其他函数，那么从这个问题函数的栈帧开始向低地址搜索，肯定会发现其他函数的返回地址，肯定返回到了这个问题函数中。然后，就好办了。

在栈帧中向低地址找，直到没有那一堆A。

![](https://bbs.kanxue.com/upload/attach/201405/589828_lt9s36xudht8kxz.png)

再往上是一些参数，然后`esp-8920`，`esp-8924`是现在`ebx`的值，`esp-8928`是现在`ebp`的值。再往上`msvcrt`的那个返回地址不用管它，`esp-8930`是现在`edi`的值，然后就是返回地址了。

在`Immunity Debugger`中转到`0041E9D8`，当单步`RETN 4`后，程序崩掉。

![](https://bbs.kanxue.com/upload/attach/201405/589828_uwqy4rdbravjew3.png)

现在上`IDA`，打开Easy RM to MP3 Converter，转到`0041E9D8`，来到这里：

![](https://bbs.kanxue.com/upload/attach/201405/589828_3s901n9hsw46nbr.png)

按空格键，转换为视图模式：

![](https://bbs.kanxue.com/upload/attach/201405/589828_tc98tzbjvp6ejui.png)

可以看出这个函数的名字是：`sub_41E2B0`，也就是返回失败前最后调用的那个函数，也就是上文提到的问题函数。双击它，来到这个函数开始的地方。

![](https://bbs.kanxue.com/upload/attach/201405/589828_5misxe2k288q20c.png)

顺着跳转往下找，很快发现了复制字符串，计算字符串长度的东西：

![](https://bbs.kanxue.com/upload/attach/201405/589828_uabgijixiq0fdj1.png)

在到达这个问题函数的末尾前，还有一些这样跟复制有关的，那怎么确定到底是在哪里出现问题，发生了溢出？看来只有调试一下了。

先来看看栈上的情况，当最后崩掉时，`esp=000FFD38`，那个问题函数最后返回时是`RETN 4`，所以，存放返回地址的地方是`000FFD38 - 8 =000FFD30`

要复制东西，应该是把`esi`指向的东西复制到`edi`指向的东西。在这个问题函数开始的地方设一个断点，在栈中来到`000FFD30`（此时存放的还是返回地址），把栈的位置锁定，然后单步，同时观察`ESI`,`EDI`，注意`REP MOVS`之类的指令，如果执行完某条指令后`000FFD30`处变成一堆A，那就说明上一条指令发生了溢出。

![](https://bbs.kanxue.com/upload/attach/201405/589828_keqahgswn3idofc.png)

到`0041E3F6`时，`edi`的值指向一堆A。那么溢出应该发生在上一条指令：

```
0041E3F0  |. FF93 72640000  CALL DWORD PTR DS:[EBX+6472]
```

重新加载，在这条指令下断点：

```
0041E3F0  |. FF93 72640000  CALL DWORD PTR DS:[EBX+6472]
```

仔细观察栈上`000FFD30`处的数据：

执行前，`000FFD30`处的数据还是返回地址，执行后，`000FFD30`处的数据就成一堆A了。

应该就是它了。

重新加载，`F7`单步步入，来到`10008D40`，module `MSRMfilt`。`Alt+E`查看加载的模块，实际上对应的是安装目录下的`MSRMfilter03.dll`这个文件。

用`IDA`打开这个`dll`文件，来到`10008D40`，切换到视图模式，这个函数的逻辑很简单，很快找到复制字符串的地方：

![](https://bbs.kanxue.com/upload/attach/201405/589828_tma9268idcorrr8.png)

当然，也可以用调试器单步调试一下：

```
10008D40 > 68 C0000000      PUSH 0C0
10008D45   68 68940310      PUSH MSRMfilt.10039468                            ; ASCII "D:\Mpf2.0\MplayerMod\dll_interface\PlayListInterface.c"
10008D4A   68 48950310      PUSH MSRMfilt.10039548                            ; ASCII "Debug: Playlist_FindNextItem enter. %s(%u)"
10008D4F   6A 05            PUSH 5
10008D51   E8 8A000000      CALL MSRMfilt.10008DE0
10008D56   A1 00D60410      MOV EAX,DWORD PTR DS:[1004D600]
10008D5B   6A 01            PUSH 1
10008D5D   50               PUSH EAX
10008D5E   E8 EDDAFFFF      CALL MSRMfilt.10006850
10008D63   83C4 18          ADD ESP,18
10008D66   85C0             TEST EAX,EAX
10008D68   74 44            JE SHORT MSRMfilt.10008DAE
10008D6A   56               PUSH ESI                                          ; 001046AF
10008D6B   57               PUSH EDI                                          ; 000FFD30（栈上存放返回地址的位置）
10008D6C   8BF8             MOV EDI,EAX
10008D6E   83C9 FF          OR ECX,FFFFFFFF
10008D71   33C0             XOR EAX,EAX
10008D73   68 CD000000      PUSH 0CD
10008D78   F2:AE            REPNE SCAS BYTE PTR ES:[EDI]                      ; EDI指向C:\Documents and Settings\Administrator\桌面\AAAAAAAAAAA
10008D7A   F7D1             NOT ECX
10008D7C   2BF9             SUB EDI,ECX
10008D7E   68 68940310      PUSH MSRMfilt.10039468                            ; ASCII "D:\Mpf2.0\MplayerMod\dll_interface\PlayListInterface.c"
10008D83   8BD1             MOV EDX,ECX
10008D85   8BF7             MOV ESI,EDI                                       ; ESI指向C:\Documents and Settings\Administrator\桌面\AAAAAAAAAAA
10008D87   8B7C24 14        MOV EDI,DWORD PTR SS:[ESP+14]                     ; EDI指向OOOF9730，一开始那里是一堆0
10008D8B   68 20950310      PUSH MSRMfilt.10039520                            ; ASCII "Debug: Playlist_FindNextItem ok. %s(%u)"
10008D90   C1E9 02          SHR ECX,2
10008D93   F3:A5            REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]    ; 在这里发生溢出，覆盖返回地址！
```

找到溢出位置后，下一步是弹计算器。

返回地址变成一堆A后，马上查看栈上那一堆A的最低位置：`000F975D`

![](https://bbs.kanxue.com/upload/attach/201405/589828_intali7bui8vp0b.png)

现在相对距离可以确定了，返回地址`000FFD30` - 最低位置`000F975D` = `65D3`，转换为十进制为26067。

别忘了，问题函数最后一条指令是`RETN 4`，所以返回地址后面还要再加上4字节填充，然后就是`esp`了。

最后请教论坛里的朋友们一个问题，我找`jmp esp` 时，软件自带的dll里只有一处：

Found commands (All modules), item 6

Address=`01EBF23A`

Disassembly=`JMP ESP`

Module Name=`C:\Program Files\Easy RM to MP3 Converter\MSRMCcodec02.dll`

但用这个`01EBF23A`时却失败了，里面应该没有包含敏感字符吧，我也没想明白，所以在这里请教论坛里的朋友们。

最后我用了`kernel32.dll`里的一个地址：`7C86467B`

用`python`写的exploit，`shellcode`用的是这位同学863K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8#2M7Y4m8D9k6i4u0G2j5#2)9J5k6h3y4G2L8g2)9J5c8X3S2@1L8h3I4Q4x3V1j5&6x3K6f1I4y4K6S2Q4x3X3g2Z5N6r3#2D9i4@1f1#2i4K6R3$3i4K6V1&6i4@1f1%4i4K6W2m8i4K6R3@1N6$3W2F1k6r3!0%4M7H3`.`. xp sp3下的弹计算器shellcode（谁能告诉我win7动态定位的怎么写）。
