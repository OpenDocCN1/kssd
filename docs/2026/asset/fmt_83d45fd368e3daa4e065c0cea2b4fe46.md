
x86/x64 CPU 支持 32/64 Bits 寄存器寻址（4 GBytes ~ 16 EBytes），前者（x86）不会有任何问题，因为线性地址范围最小支持 32 Bits（当然也兼容更古老的 CPU）。然而，后者（x64）却引发了问题：CPU 仅支持 48 Bits 线性地址，因此，厂商规定线性地址的高 16 Bits 被作为符号扩展（`0x0000...` 或 `0xFFFF...`），这样的线性地址称为 canonical 地址；反之视为 non-canonical 地址（如高 16 Bits 的 `0x0001...` 或 `0xFFF0...` 等等），使用 non-canonical 地址将引发异常（Intel 手册描述）。

### CPU 线性地址布局：

- ``0x00000000`00000000`` ~ ``0x00007FFF`FFFFFFFF``：canonical 地址共 128 TBytes。
- ``0xFFFF8000`00000000`` ~ ``0xFFFFFFFF`FFFFFFFF``：canonical 地址共 128 TBytes。
- ``0x00008000`00000000`` ~ ``0xFFFF7FFF`FFFFFFFF``：non-canonical 地址共 16,776,960 TBytes。

> 合法线性地址的 Bit 47 必须和 Bit 48 ~ 63 一致，否则为非法地址。若以后需要更大的空间，只需将符号位上移。

从 x64 Windows 8.1 及其之后的版本线性地址支持 256 TBytes，这与 CPU 描述的线性地址吻合。而之前的版本仅支持 16 TBytes 线性地址（微软描述，实测为 248 TBytes）。

```c
SYSTEM_INFO info = {};
GetSystemInfo(&info);
/*
x64 Win7:
info.lpMinimumApplicationAddress = 0x00000000`00010000
info.lpMaximumApplicationAddress = 0x000007FF`FFFEFFFF
x64 Win10:
info.lpMinimumApplicationAddress = 0x00000000`00010000
info.lpMaximumApplicationAddress = 0x00007FFF`FFFEFFFF
*/
```

可以通过 `windbg` 的 `!address` 命令枚举线性地址布局，还可以使用 `!pte` 命令查看线性地址是否 canonical：

#### x64 Win7：

```text
kd>!address
....
BaseAddress      EndAddress+1        RegionSize         VaType
----------------------------------------------------------------
0`00000000        0`00010000        0`00010000         UserRange
....
7FF`FFFF0000      800`00000000      0`00010000         UserProbeArea
800`00000000      8000`00000000     7800`00000000      <unknown>
8000`00000000     FFFF8000`00000000 FFFF0000`00000000  NonAddressable
FFFF0800`00000000 FFFF8000`00000000 7800`00000000      SystemRange
FFFF8000`00000000 FFFFF680`00000000 7680`00000000      SystemRange
FFFFF680`00000000 FFFFF700`00000000 80`00000000        PageTables
....
FFFFFFFF`FFC00000 FFFFFFFF`FFFFFFFF 0`00400000         HAL
```

用户模式线性地址空间是固定大小的：``0x800`00000000``（因从 0 计数，有效地址要 - 1） = 8 TBytes。未知区域（`unknown`）大小 = ``0x7800`00000000`` = 120 TBytes。

内核模式从高到低对地址进行布局：`HAL - PageTables + 1` = ``0xFFFFFFFF`FFFFFFFF - 0xFFFFF680`00000000 + 1`` = ``0x980`00000000`` = 9.5 TBytes。

从上向下数第一块 `SystemRange` 的 ``0xFFFF0800`00000000`` 和 ``0xFFFF8000`00000000 - 1`` 这两个地址都是 non-canonical，但如果将这两个地址的高 16 Bits 清零，则与 `unknown` 吻合（应该是微软将用户模式没有使用的 120 TBytes 归为内核了）。

第二块 `SystemRange` 的大小 = 118.5 TBytes（128 - 9.5）。综上内核模式共占 248 TBytes（``0xFFFFFFFF`FFFFFFFF - 0xFFFF0800`00000000 + 1``）。

#### x64 Win10：

```text
kd>!address
....
BaseAddress      EndAddress+1        RegionSize         VaType
----------------------------------------------------------------
0`00000000        0`7ffe0000        0`7ffe0000         UserRange
....
7fff`ffff0000     ffff8000`00000000 ffff0000`00010000  UserProbeArea
8000`00000000     ffff8000`00000000 ffff0000`00000000  NonAddressable
ffff8000`00000000 ffffa908`ac201000 2908`ac201000      SystemRange
....
ffffffff`ffc00000 ffffffff`ffffffff 0`00400000         HAL
```

标准的 canonical 地址，用户与内核模式各占 128 TBytes。（`SystemRange` 块的 `EndAddress` 的值是随机的……不明所以）。

> 实模式，或保护模式分段下，线性地址就是物理地址；保护模式分页下，线性地址转物理地址涉及 32/48 Bits 的两种分页模式，论坛中有其他文章介绍，因此，就不赘言了。

# 逻辑地址

言不如表；表不如图。下图截取自 Intel 手册，图中绘出了逻辑地址到线性地址的转换过程。从图中大致可以将逻辑地址归纳为 'Segment:Offset' 形式，通常 Segment 是隐式的。而现今的编程环境基本皆为 Flat Model，也就是说逻辑地址已经被削弱，取而代之的是线性地址，至少 c/c++ 中指针使用线性地址。

![](https://bbs.kanxue.com/upload/attach/202106/709052_Y658CW23T2JUAR7.png)

# 参考资料

[Memory Limits for Windows and Windows Server Releases](<https://bbs.kanxue.com/elink@44fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3#2W2L8h3!0J5P5g2)9J5c8X3#2W2L8h3!0J5P5g2)9J5k6r3I4A6L8h3W2@1M7#2)9J5k6r3k6G2M7W2)9J5k6s2N6A6L8X3c8G2N6%4y4Q4x3X3c8J5k6h3I4W2j5i4y4W2M7H3`.`.)>)

[Kernel Virtual Address Layout](<https://bbs.kanxue.com/elink@3bcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3c8W2L8h3q4U0K9r3W2F1k6g2)9J5k6h3y4G2L8g2)9J5c8X3q4J5N6r3W2U0L8r3g2K6i4K6u0r3P5o6j5@1i4K6g2X3K9$3g2J5L8X3g2D9i4K6g2X3N6X3W2J5N6s2g2S2L8q4)9#2k6X3q4V1k6s2u0W2M7%4y4Q4y4h3k6K6M7r3q4U0k6g2)9#2k6X3I4S2P5h3!0#2N6q4)9J5k6h3S2@1L8h3H3`.)>)

[I/O Ports](<https://bbs.kanxue.com/elink@0cdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4K9h3E0A6i4K6u0W2L8%4y4V1k6i4k6Q4x3X3g2G2M7X3N6Q4x3V1k6u0i4K6u0r3e0#2)9#2k6W2m8G2M7Y4c8K6)>)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2021-6-26 17:51 被 khristian 编辑，原因：

[#基础知识](https://bbs.kanxue.com/forum-41-1-130.htm)
