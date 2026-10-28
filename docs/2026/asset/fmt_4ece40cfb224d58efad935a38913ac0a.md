
第二个设计上的考虑要麻烦一点。当FUTo从`PspCidTable`中删除一个对象的时候，句柄项被用`NULL`替代，这意味着这个进程“并不存在”。当进程被隐藏（在`PspCidTable`里面找不到）和关闭的时候问题就产生了。当系统想关闭进程的时候，它将找到`PspCidTable`并且得到一个`NULL`对象指针，这将导致蓝屏。解决方法很简单，但是并不优雅。首先，FUTo通过调用`PsSetCreateProcessNotifyRoutine`来安装一个进程通知例程。当有进程被创建时，这个回调函数将被调用，但更重要的是，当进程被删除时，它也会被调用。回调函数在进程被中止之前调用；也就是说，它在系统崩溃之前会被调用。当FUTo检测到对这种流氓进程对象的引用时，FUTo就把这个值保存下来以便以后使用。当进程被关闭时，FUTo会在进程结束之前恢复对象，这就允许系统解除对有效对象的引用。

## 结语

2005年的一句妙语是：“我们为rootkit检测增加了障碍。”希望读者能对顶尖rootkit检测程序是怎样检测隐藏进程的以及能够如何改进有更深入的了解。有些读者可能会问：“我该怎样做呢？”嗯，最简单的方法就是不要连到Internet，但是把Blacklight，IceSword和Rootkit Revealer联合起来使用将帮助你极大地减小中rootkit的几率。一个新的叫做RAIDE（Rootkit分析验证以及去除）的工具将在几个月之内在Blackhat Amsterdam揭开她的神秘面纱。这个新的工具将不再受本文提到的问题影响。

## 参考文献

[1]Blacklight Homepage. F-Secure Blacklight [161K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3k6Q4x3X3c8K6k6h3y4#2M7X3g2Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3q4U0K9$3I4A6k6$3S2@1i4K6u0r3](https://bbs.kanxue.com/elink@231K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3k6Q4x3X3c8K6k6h3y4#2M7X3g2Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3q4U0K9$3I4A6k6$3S2@1i4K6u0r3)

[2]FU Project Page. FU [e2eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4u0G2L8%4c8C8K9i4c8Q4x3X3g2U0L8$3#2Q4x3V1k6H3M7X3!0B7k6h3y4@1i4K6u0W2M7r3S2H3i4K6y4r3K9h3c8Q4x3@1b7I4x3R3`.`.

[3]IceSword Homepage. IceSword [e25K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4S2X3L8$3y4#2M7#2)9J5k6h3&6W2N6q4)9J5c8Y4c8G2L8$3I4K6i4K6u0r3x3U0l9H3y4e0l9#2i4K6u0r3x3e0l9K6x3W2)9J5k6h3S2@1L8h3H3`.

[4]LordPE Homepage. LordPE Info [f43K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2A6N6r3N6D9K9h3g2V1i4K6u0W2L8s2W2U0L8%4y4Q4x3X3g2V1k6g2)9J5c8Y4W2G2k6r3p5J5K9#2)9J5c8V1I4G2M7X3c8b7c8g2)9J5c8X3W2F1k6X3!0Q4x3X3g2Z5N6r3@1`.

[5]Opc0de. 2005. How to get some hidden kernel variables without scanning [261K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4u0G2L8%4c8C8K9i4c8Q4x3X3g2U0L8$3#2Q4x3V1k6F1k6i4N6K6M7X3g2S2k6q4)9J5k6i4m8Z5M7q4)9K6c8X3&6W2N6%4y4A6k6q4)9K6c8o6p5H3x3b7`.`.

[6]Rohitabs API Monitor. API Monitor - Spy on API calls [c30K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4u0G2K9r3W2@1j5h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7r3W2E0L8$3&6A6N6r3!0J5i4K6u0r3]

[7]Russinovich, Solomon. Microsoft Windows Internals Fourth Edition.

[8]Silberman. RAIDE:Rootkit Analysis Identification Elimination [192K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0D9j5h3y4C8K9r3q4@1i4K6u0W2j5$3!0E0i4K6u0r3K9s2c8E0L8q4)9J5c8X3u0Z5i4K6u0V1k6i4g2J5L8%4m8W2i4K6u0V1x3o6k6Q4x3V1k6T1K9q4)9J5k6r3g2#2i4K6u0V1x3o6k6Q4x3X3c8K6M7r3g2S2K9$3g2J5M7#2)9J5k6h3S2@1L8h3I4Q4x3U0y4e0K9h3I4T1k6i4u0E0j5h3^5`.

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
