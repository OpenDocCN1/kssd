
`/``/` `为文件创建EA`  
`NTSTATUS ZwSetEaFile(`  
`[``in``]  HANDLE           FileHandle,`  
`[out] PIO_STATUS_BLOCK IoStatusBlock,`  
`[``in``]  PVOID` `Buffer``,`  
`[``in``]  ULONG            Length`  
`);`  
`/``/` `查询文件EA`  
`NTSTATUS ZwQueryEaFile(`  
`[``in``]           HANDLE           FileHandle,` `/``/``文件句柄`  
`[out]          PIO_STATUS_BLOCK IoStatusBlock,`  
`[out]          PVOID` `Buffer``,` `/``/``扩展属性缓冲区（FILE_FULL_EA_INFORMATION结构）`  
`[``in``]           ULONG            Length,` `/``/``缓冲区大小`  
`[``in``]           BOOLEAN          ReturnSingleEntry,`  
`[``in``, optional] PVOID            EaList,` `/``/``指定需要查询的扩展属性`  
`[``in``]           ULONG            EaListLength,`  
`[``in``, optional] PULONG           EaIndex,` `/``/``指定需要查询的起始索引`  
`[``in``]           BOOLEAN          RestartScan`  
`);`

可以看到输出缓冲区`Buffer`以及该缓冲区的长度都是从用户空间传入的。这意味着我们根据缓冲区的大小控制内核空间的内存分配。

该漏洞对攻击者来说：
- 溢出拷贝时数据和大小均可控。
- 可以覆盖下一个内核池块
- 内核池分配时大小可控，并且可以进行堆布局。

## 触发漏洞

Windows10引入了新的方式进行堆块管理，称为`Segment Heap`，具体可看以下论文

[e21K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6M7%4c8A6j5#2)9J5k6h3!0J5k6#2)9J5c8X3#2W2k6r3W2S2i4K6u0r3f1#2y4f1d9f1x3J5x3o6t1H3i4K6u0r3f1#2y4f1d9f1y4Q4x3X3c8S2j5%4c8W2M7#2)9J5c8Y4m8G2L8$3I4Q4y4h3k6G2N6X3g2J5k6X3I4G2N6#2)9#2k6X3g2^5M7r3I4G2K9i4c8S2N6r3W2G2L8W2)9#2k6Y4y4A6L8X3y4W2i4K6g2X3N6$3W2F1k6r3!0%4M7#2)9#2k6U0p5H3i4K6g2X3x3e0W2Z5x3g2)9J5c8W2y4e0g2p5W2o6x3U0l9J5x3q4)9J5k6p5q4J5N6r3W2U0L8r3g2Q4x3X3c8H3L8$3!0D9i4K6g2X3L8%4k6W2M7X3k6D9L8%4N6Q4y4h3k6W2P5s2m8D9L8$3W2@1j5i4c8A6L8$3&6Q4y4h3k6K6K9h3&6U0k6g2)9#2k6Y4N6A6L8X3c8G2N6%4y4Q4y4h3j5I4x3q4)9#2k6U0p5&6K9o6q4Q4x3X3c8T1j5i4W2W2N6q4)9#2k6X3k6S2M7X3W2W2L8r3I4G2i4K6u0W2M7r3c8X3](https://bbs.kanxue.com/elink@900K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6M7%4c8A6j5#2)9J5k6h3!0J5k6#2)9J5c8X3#2W2k6r3W2S2i4K6u0r3f1#2y4f1d9f1x3J5x3o6t1H3i4K6u0r3f1#2y4f1d9f1y4Q4x3X3c8S2j5%4c8W2M7#2)9J5c8Y4m8G2L8$3I4Q4y4h3k6G2N6X3g2J5k6X3I4G2N6#2)9#2k6X3g2^5M7r3I4G2K9i4c8S2N6r3W2G2L8W2)9#2k6Y4y4A6L8X3y4W2i4K6g2X3N6$3W2F1k6r3!0%4M7#2)9#2k6U0p5H3i4K6g2X3x3e0W2Z5x3g2)9J5c8W2y4e0g2p5W2o6x3U0l9J5x3q4)9J5k6p5q4J5N6r3W2U0L8r3g2Q4x3X3c8H3L8$3!0D9i4K6g2X3L8%4k6W2M7X3k6D9L8%4N6Q4y4h3k6W2P5s2m8D9L8$3W2@1j5i4c8A6L8$3&6Q4y4h3k6K6K9h3&6U0k6g2)9#2k6Y4N6A6L8X3c8G2N6%4y4Q4y4h3j5I4x3q4)9#2k6U0p5&6K9o6q4Q4x3X3c8T1j5i4W2W2N6q4)9#2k6X3k6S2M7X3W2W2L8r3I4G2i4K6u0W2M7r3c8X3)

`EXP:[c80K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2j5i4A6Z5N6h3I4A6j5h3&6Y4i4K6u0r3b7#2k6q4i4K6u0V1x3U0l9J5x3g2)9J5k6o6x3I4z5e0f1$3i4K6u0V1c8g2S2b7](https://bbs.kanxue.com/elink@7c7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2j5i4A6Z5N6h3I4A6j5h3&6Y4i4K6u0r3b7#2k6q4i4K6u0V1x3U0l9J5x3g2)9J5k6o6x3I4z5e0f1$3i4K6u0V1c8g2S2b7)

仿照EXP改的可以触发溢出的POC

|  |  |

| --- | --- |
