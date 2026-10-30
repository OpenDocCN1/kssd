
结合缓存未进行预初始化和准许复制多余字节两个情况，这就成了一个可以被利用的安全bug。在POC（32dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3N6K6i4K6u0W2j5$3S2J5L8$3#2A6N6h3#2Q4x3X3g2G2M7X3N6Q4x3V1k6H3i4K6u0r3M7s2u0G2K9X3g2U0N6q4)9J5k6s2A6W2M7X3!0Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3V1k6S2N6s2c8S2j5$3S2E0k6h3&6@1g2r3g2^5N6q4)9K6c8X3q4A6k6q4)9K6c8o6t1^5y4K6x3&6x3g2!0q4c8W2!0n7b7#2)9^5z5g2!0q4y4q4!0n7z5q4!0m8c8q4!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4W2)9^5z5q4)9&6x3g2!0q4y4q4!0n7b7W2!0m8b7#2!0q4y4q4!0n7c8q4!0n7c8W2!0q4y4#2)9&6y4q4!0m8z5q4!0q4y4q4!0n7z5q4)9^5x3q4!0q4y4q4!0n7z5q4!0m8b7g2!0q4y4W2)9&6b7#2!0m8b7g2!0q4y4W2)9&6y4W2)9^5y4#2!0q4y4W2!0m8x3g2!0m8x3#2!0q4y4g2)9^5b7#2)9&6y4W2!0q4y4q4!0n7c8W2!0m8x3g2!0q4y4W2)9^5x3g2!0m8c8W2!0q4y4#2!0n7x3g2!0n7b7W2!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4g2!0m8c8g2)9^5x3#2!0q4y4g2)9^5c8W2!0m8b7g2!0q4y4g2)9^5y4W2)9&6z5g2!0q4y4q4!0n7b7g2)9^5y4W2!0q4y4#2!0n7b7#2)9&6x3#2!0q4y4g2!0m8c8q4)9&6z5q4!0q4y4#2)9&6b7g2)9^5y4q4!0q4y4g2)9^5z5g2)9^5c8o6c8Q4c8e0g2Q4b7f1c8Q4z5e0N6Q4c8e0S2Q4z5p5q4Q4z5o6u0Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0N6Q4z5e0g2Q4z5e0W2Q4c8e0c8Q4b7U0S2Q4z5p5u0Q4c8e0g2Q4z5o6W2Q4b7e0W2Q4c8e0c8Q4b7U0S2Q4z5p5u0Q4c8e0N6Q4z5f1q4Q4z5o6b7^5z5q4!0q4y4g2!0m8c8q4)9&6y4#2!0q4z5q4)9^5b7g2)9^5x3W2!0q4y4g2)9^5y4#2)9^5y4W2!0q4y4g2!0m8y4q4)9^5y4#2!0q4y4W2!0n7x3#2)9^5y4q4!0q4z5g2)9&6b7#2!0n7x3W2!0q4y4#2!0n7b7W2)9&6z5g2!0q4y4W2)9&6y4q4!0n7b7W2!0q4y4g2)9^5y4#2!0n7b7W2!0q4z5q4)9^5x3q4)9^5y4g2!0q4x3#2)9^5x3q4)9^5x3R3.

# `win32k!NtGdiEngCreatePalette` (CVE-2017-8685)

这个例子中，脆弱性在Windows8中通过引入下图的`memset`被修复，但在Windows7中仍然存在：

![](https://bbs.kanxue.com/upload/attach/201805/805111_S8826KECW2YE4AK.png)

存在问题的系统调用负责建立由N个4字节颜色项组成的GDI调色板对象。同样的，实现中包含一个内存使用优化：如果N小于等于256（总大小1024字节），则通过`win32k!bSafeReadBits`将数据从用户模式读到内核栈缓存中；否则，直接调用`win32k!bSecureBits`锁定ring-3中的内存。和你猜的一样，添加额外`memset`的内存区域是用于临时存储用户定义的RGB颜色列表的本地缓存，它随后会被传给`win32k!EngCreatePalette`来建立调色板对象。现在的问题是，我们如何让缓存不初始化确仍然传输去创建一个非空的调色板？答案在`win32k!bSafeReadBits`的实现中：

![](https://bbs.kanxue.com/upload/attach/201805/805111_BUN6VDZVZHXH72Z.png)

从上面的反编译结果可以看到，如果源或目的指针为空，则函数未进行任何实际工作就结束。这里的源地址直接来自syscall的第三个参数，在传到该函数前未进行任何处理。这意味着，我们可以使得syscall认为它成功从用户模式获得了一个256个元素的列表，实际上栈缓存没有被写入东西。在我们的POC（2a0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3N6K6i4K6u0W2j5$3S2J5L8$3#2A6N6h3#2Q4x3X3g2G2M7X3N6Q4x3V1k6H3i4K6u0r3M7s2u0G2K9X3g2U0N6q4)9J5k6s2A6W2M7X3!0Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3V1k6S2N6s2c8S2j5$3S2E0k6h3&6@1g2r3g2^5N6q4)9K6c8X3q4A6k6q4)9K6c8o6t1^5y4K6x3&6y4#2!0q4c8W2!0n7b7#2)9^5z5g2!0q4y4q4!0n7z5q4!0m8c8q4!0q4c8W2!0n7b7#2)9^5b7#2!0q4z5q4!0n7c8W2)9&6z5g2!0q4z5g2)9^5x3q4)9&6b7g2!0q4z5q4!0n7c8W2)9^5y4#2!0q4y4g2!0m8y4W2)9^5x3W2!0q4y4q4!0n7z5q4)9^5b7W2!0q4y4#2)9&6b7g2)9^5y4q4!0q4y4#2!0n7x3#2!0n7b7W2!0q4y4#2!0n7b7W2)9&6c8W2!0q4z5q4!0n7x3q4)9^5x3#2!0q4y4#2)9&6y4q4!0m8z5q4!0q4y4g2!0m8c8g2)9&6c8g2!0q4y4#2)9^5c8g2!0n7x3q4!0q4c8W2!0n7b7#2)9&6b7b7.

```
HPALETTE hpal = (HPALETTE)SystemCall32(__NR_NtGdiEngCreatePalette, PAL_INDEXED, 256, NULL, 0.0f, 0.0f, 0.0f);
```

syscall返回后，我们收到一个调色板句柄，调色板内存着被泄露的栈内存。为了从句柄读出内存，需要调用`GetPaletteEntries`来实现。再次强调这个bug的重要性，它准许攻击者泄露整整1kB未初始化的内核栈内存，这将在攻击者发挥很大的作用。

在内存泄露本身之外，在附加代码区域还会发现其它有趣的东西。如果你看Windows8.1和Windows10中`win32k!NtGdiEngCreatePalette`有关的代码，你将发现它们之间的一个有趣的不同：在两个系统中栈数组都被完全清0了，但是却使用不同的方法实现。在Windows8.1中，函数先设置第一个`DWORD`为0，然后调用`memset`设置剩下0x3fc字节为0，Windows10直接使用`memset`设置全部0x400直接为0。存在这个不同的原因不清楚，尽管最后的结果一样，但这个不同又引起了一个想法，不仅比较是否存在不同数量的`memset`，还可以比较`memset`操作的内存尺寸。

![](https://bbs.kanxue.com/upload/attach/201805/805111_X4XWBCCDD74YD66.png)

在一个最近的报告中，`win32k!NtGdiEngCreatePalette`在利用内核漏洞时进行栈spraying也非常有用，它准许用户方便地写1024字节的可控数据到一个栈上的连续区域。虽然控制的缓存尺寸小于`nt!NtMapUserPhysicalPages`能提供的，但是控制的缓存位于顶层系统调用处理函数栈帧的高偏移处，这才某些情况下非常有用。

## 结论

本文目的是举例说明一个产品同时支持的不同分支版本的安全相关区别会被攻击者用于定位重要的漏洞。它不仅将某些客户暴露给攻击者，同时还清楚地说明了攻击向量是什么。这对于修复特征明显的bug尤其严重，如内核内存泄露和增加的`memset`。本文采用的二进制比较过程实际上是伪代码层的比较，不需要跟到系统底层的知识。非高级的攻击者就可以无需很多工作就可以发现这三个脆弱性（CVE-2017-8680，CVE-2017-8684，CVE-20178685）。我们期望这类很容易被发现与利用的问题是少数的，并且厂商要对他们的各个版本产品进行一致地修复。

## 引用

1. 3d6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0D9j5h3y4C8K9r3q4@1i4K6u0W2j5$3!0E0i4K6u0r3M7s2u0W2M7$3g2F1N6r3q4@1K9h3!0F1M7#2)9J5c8X3u0Z5i4K6u0V1N6i4y4S2i4K6u0V1x3o6W2Q4x3V1k6a6d9q4)9J5c8V1u0t1g2g2y4m8x3o6W2Q4x3X3c8a6K9q4)9J5k6p5c8A6k6X3k6A6L8X3N6n7K9h3&6S2M7X3W2W2M7#2)9J5k6q4y4x3d9f1c8q4f1#2)9J5k6i4m8V1k6R3.

2. 00aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1k6h3W2K6N6r3I4S2j5W2)9J5k6h3k6A6L8r3g2K6i4K6u0W2N6$3!0J5k6s2m8J5k6i4y4K6i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9I4x3W2)9J5c8U0p5H3i4K6u0r3K9i4y4W2j5#2)9#2k6U0t1H3x3e0u0Q4y4h3k6T1k6h3W2K6N6q4)9#2k6Y4y4D9K9h3c8W2M7#2)9J5k6i4m8V1k6R3.

3. 0a4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2J5M7$3q4U0L8$3&6X3k6i4u0W2L8X3y4W2i4K6u0W2j5$3!0E0i4K6u0r3N6%4u0A6N6r3q4T1L8r3g2Q4x3V1k6H3M7X3g2K6k6h3&6@1j5i4c8A6L8$3&6K6i4K6u0r3k6X3W2D9k6g2)9#2k6Y4g2H3L8r3!0S2k6q4)9J5c8X3S2@1i4K6u0V1N6o6p5H3i4K6u0V1j5Y4u0#2K9q4)9#2k6W2)9J5k6r3c8G2i4K6u0V1P5h3!0#2i4K6u0V1k6i4k6W2L8W2)9J5k6r3c8A6k6X3k6Q4x3X3c8V1K9h3k6X3K9h3&6Y4i4K6u0V1L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0V1M7r3q4@1j5$3S2W2M7#2)9J5k6s2c8G2i4K6u0V1k6X3W2F1k6q4)9J5k6s2k6#2L8r3&6W2M7X3q4T1K9h3I4A6N6r3W2W2M7#2)9J5k6i4m8V1k6R3.

4. 5c4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2*7P5h3&6S2L8h3W2U0M7#2)9J5k6h3y4G2L8g2)9J5c8X3u0A6L8X3c8A6k6X3k6Q4x3X3g2Z5N6r3#2D9

5. 091K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3c8S2M7Y4g2F1k6%4u0A6L8g2)9J5k6h3!0J5k6#2)9J5c8R3.

6. b01K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6B7L8%4S2W2j5h3&6C8L8%4u0W2N6q4)9J5c8X3c8A6j5i4m8Z5L8%4u0S2

7. 41aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6N6i4m8H3L8%4u0@1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3S2W2L8s2m8Q4x3V1j5I4x3K6R3#2x3#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3X3c8D9K9h3k6W2j5%4W2U0L8r3g2Q4x3X3c8X3j5h3y4@1i4K6u0V1M7$3S2W2k6i4b7.

8. c10K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3N6K6i4K6u0W2M7%4c8S2N6r3y4G2N6h3&6@1k6i4u0Q4x3X3g2U0L8$3#2Q4x3V1k6G2M7#2)9J5k6s2k6W2M7Y4y4A6L8$3&6Q4x3X3c8E0j5i4u0C8k6i4c8Q4x3X3c8K6K9r3q4J5k6g2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6V1k6i4y4C8N6r3!0H3i4K6u0r3N6$3!0J5L8r3c8%4K9h3c8W2

9. 5feK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3Z5H3x3s2u0#2i4K6u0W2N6X3g2^5K9h3I4D9K9i4g2E0i4K6u0W2L8%4u0Y4i4K6u0r3M7$3I4A6k6r3g2K6i4K6u0r3x3U0l9I4y4#2)9J5c8Y4u0W2j5$3!0F1i4K6u0W2M7r3c8X3

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
