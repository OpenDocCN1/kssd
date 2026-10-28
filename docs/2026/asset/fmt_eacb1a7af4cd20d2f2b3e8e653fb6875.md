
高亮部分的意思是：操作系统的其它部分和所有的应用程序按 `Scan Code Set 1` 里的值处理扫描码。所以我猜测微软官方文档里的 `keyToScanTbl[134]` 的值是按照 `Scan Code Set 1` 给出的。实在没精力调查的更清楚了，如果哪位小伙伴儿熟悉相关内容，请务必指点！我特意录制了使用自定义快捷键 `CTRL + ` + ` ` 触发蓝屏的过程。按一次 ` ` 是不行的，一定要按两次。

![](https://bbs.kanxue.com/upload/attach/202002/873494_7VDAQWEFSAN67UH.jpg)

## 5. 借助硬件

如果以上方法都行不通，还可以使用 PCI Express Dump Switch Card 使系统蓝屏，Windows 7, Windows Server 2008 R2 及更早的系统需要设置如下注册表项：

```
[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CrashControl]

"NMICrashDump"=dword:00000001
```

## 6. 内核调试器

如果已经用内核调试器附加到目标系统了，我们可以直接使用 `.dump` 命令保存转储文件，没必要使系统蓝屏再保存转储文件。当然，如果我们喜欢，也可以使用 `.crash` 命令使系统蓝屏，如果执行完该命令后，系统没蓝屏，请执行 `g` 命令。说明：如果当前正在执行 bug check 处理函数，不要使用 `.crash` 命令。

注意：

需要双机内核调试的时候才行。在本地内核调试的时候，不能使用 `.dump` 命令，也不能使用 `.crash` 命令！真是个坏消息！！！

## 7. 其它

- 利用 `NtRaiseHardError`，参考 github 上的 Invoke-BSOD 项目。
- 利用系统漏洞。这方面我不擅长，欢迎搞安全的小伙伴儿分享。

# 蓝屏转储一瞥

通过杀死系统关键进程导致的蓝屏，对应的 Bug Check Code 是 `0xF4`（`CRITICAL_OBJECT_TERMINATION`）。

通过 NMI 导致的蓝屏，对应的 Bug Check Code 是 `0x80`（`NMI_HARDWARE_FAILURE`），表示这个蓝屏是由于硬件错误触发的。

通过系统快捷键导致的蓝屏，对应的 Bug Check Code 是 `0xe2`（`MANUALLY_INITIATED_CRASH`），表示这个蓝屏是用户手动触发的。

我们可以用 `!analyze -show` 来查看转储文件中的 bug check code 信息。

![](https://bbs.kanxue.com/upload/attach/202002/873494_KCB6BCE7WJE5UTW.jpg)

从输出结果可知，这个蓝屏是用户手动触发蓝屏产生的。从调用栈可知，是 `i8042prt.sys` 处理了我们的按键，从而导致的蓝屏。

友情提示：

如果直接用 windbg 打开 `%SystemRoot%\MEMORY.DMP` 会提示权限不够。可以以管理员权限运行 windbg 后再打开。或者拷贝转储文件到其它目录后再打开即可。注意一定是拷贝，而不是剪切。

![](https://bbs.kanxue.com/upload/attach/202002/873494_TNEB8GC4ZNBRUW4.jpg)

# 总结

- 特殊情况下，我们可能会主动使系统蓝屏来保存一份转储文件，供我们事后分析。
- 我们可以通过本文介绍的几种方法来使系统蓝屏。我个人偏向使用 NotMyFault 或者蓝屏快捷键使系统蓝屏。
- 使用自定义的蓝屏快捷键的时候，需要关闭 `CrashOnCtrlScroll` 设置。
- `Dump1Keys` 的值可以是组合值，比如，`0x70` 表示同时按下左侧的 ALT + CTRL + SHIFT。
- `Dump2Key` 的值是对应按键的扫描码在数组 `keyToScanTbl[134]` 中的索引值，而且是 16 进制的。
- 需要连续按两次设置的按键才能蓝屏！比如，我们需要按 `CTRL + ` + ` `，而不是 `CTRL + ` `。
- 尽量不要设置已经被占用的快捷键。我尝试使用 `CTRL + A + A` 作为快捷键，但是不能触发蓝屏，猜测是因为 `CTRL + A` 被占用了。但是 `ALT + CTRL+ SHIFT + A + A` 是可以的。
- 在 windbg 中通过 `!analyze -show` 可以查看转储文件中的 bug check code 信息。
- 对于手动蓝屏得到的转储文件，使用 `!analyze -v` 不能得到更多有效的信息，我们需要分析转储文件中的其它信息。
- `%SystemRoot%\MEMORY.DMP` 需要管理员权限才能打开，可以拷贝（而不是剪切）一份到其它目录，就可以正常打开了。

# 参考资料

- 8abK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0K9r3q4F1L8X3g2D9z5g2)9J5k6h3#2K6k6r3&6Q4x3X3g2U0L8$3#2Q4x3V1k6e0K9r3!0%4M7#2)9J5c8V1c8W2k6Y4u0S2k6#2)9J5k6q4c8G2L8$3I4K6i4K6u0r3c8r3g2X3M7X3q4Y4i4K6u0V1g2r3!0G2L8s2y4Q4x3X3b7I4y4K6W2Q4x3X3c8y4j5h3&6#2j5h3I4D9P5g2)9J5k6p5k6G2M7X3y4A6L8X3N6Q4x3X3c8S2i4K6u0V1b7%4u0S2M7$3S2Q4x3X3c8p5N6h3#2H3
- `NMI_HARDWARE_FAILURE` error when an NMI is triggered on Windows 8 and Windows Server 2012
- de8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6U0L8r3W2W2L8Y4c8Q4x3X3c8E0j5h3&6S2k6$3g2E0k6h3&6@1i4K6u0r3k6$3g2F1k6i4u0S2N6r3g2Q4x3X3c8C8k6i4u0F1k6h3I4Q4x3X3c8G2M7W2)9J5k6r3y4G2L8i4m8D9k6i4c8W2i4K6u0V1j5%4u0S2M7$3S2Q4x3X3c8V1N6h3#2H3i4K6t1K6N6i4y4W2i4K6u0V1L8X3#2A6
- 60cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3X3c8Z5j5i4u0V1N6$3q4J5k6g2)9J5c8X3c8J5K9i4k6W2M7Y4y4Q4x3V1k6V1k6h3u0#2k6$3N6W2M7W2)9J5c8X3k6G2M7X3y4A6L8X3N6Q4x3X3c8S2i4K6u0V1M7%4W2K6N6r3g2E0i4K6u0V1j5%4u0S2M7$3S2Q4x3X3c8X3M7X3!0E0i4K6u0V1N6r3S2W2i4K6u0V1k6r3g2T1N6h3N6Y4k6i4t1`.
- b1bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6j5%4y4Q4x3X3g2K6N6r3q4F1k6X3!0J5k6q4)9J5k6h3g2V1N6g2)9J5c8U0p5H3N6$3W2Q4x3X3c8U0M7K6p5@1x3q4)9J5c8Y4m8A6L8Y4c8G2M7#2)9J5c8Y4y4H3k6h3y4K6i4K6u0r3K9$3u0V1i4K6u0r3M7$3y4S2L8X3y4G2k6r3g2K6i4K6u0V1z5g2)9J5k6h3S2@1L8h3H3`.
- 940K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3c8W2M7Y4c8%4i4K6u0W2j5$3!0E0i4K6u0r3i4K6t1#2c8e0c8Q4x3U0g2n7b7#2)9J5y4f1u0m8i4K6t1#2c8e0k6Q4x3U0f1&6b7#2)9J5y4e0S2p5i4K6t1#2c8e0g2Q4x3U0f1&6z5g2)9J5y4f1p5^5i4K6u0r3x3e0x3#2x3U0t1%4i4K6u0r3
- f77K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4V1N6X3c8T1k6#2)9J5k6h3!0J5k6#2)9J5c8X3k6G2M7Y4g2E0M7#2)9J5c8U0x3H3y4U0N6Q4x3V1k6b7M7X3W2F1N6q4m8G2M7%4c8Q4x3X3g2S2M7%4m8^5
- 9f7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6e0j5$3q4F1j5$3!0V1k6b7`.
- e28K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6w2k6i4W2T1L8$3q4J5k6q4)9#2k6X3I4S2P5h3!0#2N6l9`.
- 567K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8Z5K9h3I4A6M7s2y4@1L8%4u0J5i4K6u0W2K9h3c8Q4x3X3g2S2N6g2)9J5c8Y4m8U0j5X3!0G2K9#2)9J5c8X3u0G2L8$3D9K6i4K6u0r3M7$3y4S2L8X3y4G2k6r3g2Q4x3X3g2Z5N6r3@1`.
- e8cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6j5%4y4Q4x3X3g2K6N6r3q4F1k6X3!0J5k6q4)9J5k6h3g2V1N6g2)9J5c8U0p5H3N6$3W2Q4x3X3c8U0M7K6p5@1x3q4)9J5c8Y4m8A6L8Y4c8G2M7#2)9J5c8Y4y4H3k6h3y4K6i4K6u0r3K9$3u0V1i4K6u0r3M7$3y4S2L8X3y4G2k6r3g2K6i4K6u0V1z5g2)9J5k6h3S2@1L8h3H3`.
- e8fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3N6W2L8$3k6X3j5$3S2S2M7s2m8W2L8r3I4Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6s2g2V1K9h3g2K6i4K6u0r3N6$3W2F1k6r3!0%4M7#2)9J5c8Y4N6A6L8U0x3J5i4K6u0r3L8Y4c8V1L8r3I4Q4x3V1k6S2M7r3W2Q4x3V1k6J5N6r3I4Q4x3V1k6H3k6h3u0Q4x3V1k6K6k6i4c8H3M7X3!0U0k6i4y4K6K9i4y4U0M7X3W2@1K9h3y4S2L8q4)9J5k6h3S2@1L8b7`.

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2020-3-7 09:43

被编程难编辑，原因： 调整代码格式
