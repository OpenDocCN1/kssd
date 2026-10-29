
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48 | `po $rdi`  `<MQAIOChatViewController:` `0x7fca3a8def10``>`    `x``/``s $rsi`  `0x10fa1f89e``:` `"revokeMessages:"`  `c`  `Process` `35302` `resuming`  `Command` `#3 'c' continued the target.`  `po $rdi`  `<MQAIOChatViewController:` `0x7fca3a8def10``>`    `x``/``s $rsi`  `0x10fa07e45``:` `"topMsgListViewController"`  `c`  `Process` `35302` `resuming`  `Command` `#3 'c' continued the target.`  `po $rdi`  `<MQAIOChatViewController:` `0x7fca3a8def10``>`    `x``/``s $rsi`  `0x10fa07e45``:` `"topMsgListViewController"`  `c`  `Process` `35302` `resuming`  `Command` `#3 'c' continued the target.`  `po $rdi`  `<MQAIOChatViewController:` `0x7fca3a8def10``>`    `x``/``s $rsi`  `0x10fa1f973``:` `"scheduleRecognitionLogic"`  `c`  `Process` `35302` `resuming`  `Command` `#3 'c' continued the target.`  `po $rdi`  `<MQAIOChatViewController:` `0x7fca3a8def10``>`    `x``/``s $rsi`  `0x10fa07e45``:` `"topMsgListViewController"`  `c`  `Process` `35302` `resuming`  `Command` `#3 'c' continued the target.`  `po $rdi`  `<MQAIOChatViewController:` `0x7fca3a8def10``>`    `x``/``s $rsi`  `0x10f9e881c``:` `"isSimplestModel"`  `c`  `Process` `35302` `resuming`  `Command` `#3 'c' continued the target.` |

在x64汇编中，传递参数依次通过以下寄存器传递。

|  |  |

| --- | --- |

| 1 | `rdi, rsi, rdx, rcx, r8, r9` |

在很多动态分析的过程中可以借助`lldb`调试来帮助分析解决问题。

### MachO注入

为了避免每次从环境变量注入，还可以直接使用[`insert_dylib`](https://bbs.kanxue.com/elink@507K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6f1P5h3W2D9L8#2Q4x3V1k6m8L8r3!0F1k6f1#2G2L8X3E0W2P5g2)9J5c8X3W2F1M7$3g2J5N6q4)9#2k6X3c8&6L8r3W2T1)注入可执行文件。

|  |  |

| --- | --- |

| 1  2 | `.``/``insert_dylib` `-``-``weak` `/``path``/``to``/``dylib QQ QQ_patch`  `mv QQ_patch QQ` |

### 总结

总的来说，Mac上的逆向和iOS逆向差不太多，反而Mac上面操作更方便一些，不用连着手机，还有像`Dtrace`这样的神器。

这篇文章主要还是领大家走进Mac逆向的世界，其它靠自己慢慢去探索~

相关工具和代码尽在github

[1bbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6m8L8r3!0F1k6f1#2G2L8X3E0W2P5g2)9J5c8V1#2S2j5#2u0W2N6X3g2J5M7$3f1`.](https://bbs.kanxue.com/elink@791K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6m8L8r3!0F1k6f1#2G2L8X3E0W2P5g2)9J5c8V1#2S2j5#2u0W2N6X3g2J5M7$3f1`.)

几个月前的文章了~ （

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2023-4-7 10:18

被kanxue编辑

，原因：
