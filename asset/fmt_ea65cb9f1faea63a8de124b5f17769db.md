
# Themida v1.8.0.0 Demo 虚拟机分析 – Part8

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-808-8.htm`](https://www.pediy.com/kssd/pediy08/pediy8-808-8.htm)

## 关于解码器

实际上，从前面得到的细节已经可以写解码程序了。问题是写出的解码器只能针对这一个程序。由于加壳时引入的随机因素（`PCODE` 各 `field` 具有可变的 `offset`，解码 `PCODE` 数据时使用可变的字节值以及动态的 2 字节 `opcode`），即使完全看懂了 VM 的运作机理，也不见得能干净利落地将其击破。

要写出通用的解码器，需要解决以下问题：

识别 `handler` 的行为以解析 `opcode` 的确切含义，正确提取 `PCODE` 数据的各 `field` 及所用的解码字节常量。当然，还有个最大的障碍，必须尽可能清理变形代码。这些我目前做不到。

需要说明的是，前面所有的分析都来自对 dump 出的 VM 的逆向，我只看了硬币的反面，从来没有跟过加壳过程（这很愚蠢！J）。有兴趣的朋友不妨跟一下加壳时 VM 代码的生成过程，一定会有新的发现。

最后，我们以 `Themida1800 Demo` 内的一段 VM 保护代码为例，讲解手工还原代码。

![](img/74bc37967990fa8bcf047545a67105c9.jpg)

这段 VM 保护代码内包含 `Themida` 的 anti-debug，包括所谓的 `HeapMagic`（这个名字似乎也是来源于 `shoooo`）。在列出解码数据前，先啰嗦几句。

由于 VM 自己的寄存器及 `VM_Context` 内保存的原程序寄存器实际都只是 `VM_Context` 结构内的成员变量，解码时使用的命名方式有点别扭，如 `context.eax` 表示 `eax` 的值，`addr_context.eax` 表示 `VM_Context.eax` 的地址。也许你有更好的方式 J。

VM 内的大部分操作都使用 VM 自己的栈（堆栈机？）。举 2 个例子。

![](img/c1b093cb9789acc27759923298da8523.jpg)

对 `push eax` 的模仿。

![](img/7319e692eeb97806386eae38309a0434.jpg)

对 `lea ebx,[ebp+035D4CFA]` 的模仿。注意第 3 行压栈的是 `ebp + 035D4CFA` 的结果值，而不是 `[ebp + 035D4CFA]` 所指向的值，所以这里应还原为 `lea` 指令。若为 `push [context.register]`，则还原代码为 `mov ebx,[ebp+035D4CFA]`。

另外，从以前修复 `CVDemo` 的情况看，的确有所谓的 `Virtual Machine Anti-Dumper` 存在，隐藏在解码数据的前面约 1000 行内。但这些代码有明显的特征，即每个 `handler` 执行时都将 4 个 `key` 清 0 了。很容易看出，真正要还原的代码正是从 `keys` 被连续使用时开始的。

## 解码数据

其中第 1 列为 `counter`，第 2,3 列为 `opcode`，第 4 列为 `pcode` 数据地址。我已经分了行，每组对应 1 条还原的指令。

```
0001 02     00B2DE1A  mov      [addr_context.register],addr_context.eax
0002 2A 00  00B2DE28  push     [context.register]
0003 1F 01  00B2DE36  mov      [addr_context.register],addr_context.eax
0004 14 00  00B2DE44  pop      [context.register] (4 bytes)
0005 1F 01  00B2DE52  mov      [addr_context.register],context.ebp
0006 0E 00  00B2DE6E  add      [addr_context.register],034F2C5D
0007 2D 02  00B2DE8A  push     [context.register]
0008 19 01  00B2DE60  push     00000000
0009 27 00  00B2DE7C  cmp      [esp+4],[esp] (4 bytes)
0010 30 01  00B2DE98  jnz      00B2DF08
0011 2C 01  00B2DEA6  mov      [addr_context.register],context.ebp
0012 0E 00  00B2DEB4  add      [addr_context.register],034F0C4D
0013 16 00  00B2DEDE  push     [context.register]
0014 2D 02  00B2DED0  push     00000000
0015 27 00  00B2DEC2  cmp      [esp+4],[esp] (4 bytes)
0016 09 01  00B2DEEC  load     context.register,02AC3398
0017 11 01  00B2DEFA  jz       00B2E242
0018 2C 01  00B2DF08  mov      [addr_context.register],addr_context.eax
0019 2B 02  00B2DF24  push     [context.register]
0020 33 02  00B2DF16  mov      [addr_context.register],context.esp
0021 2D 00  00B2DF32  sub      [addr_context.register],00000004
0022 14 00  00B2DF5C  pop      [context.register] (4 bytes)
0023 16 00  00B2DF40  push     [addr_context.register]
0024 14 00  00B2DF4E  pop      [addr_context.esp] (4 bytes)
0025 1F 01  00B2DF6A  mov      [addr_context.register],addr_context.ebx
0026 16 00  00B2DFB0  push     [context.register]
0027 1F 01  00B2DFBE  mov      [addr_context.register],context.esp
0028 36 00  00B2DF86  sub      [addr_context.register],00000004
0029 01 01  00B2DFCC  pop      [context.register] (4 bytes)
0030 19 01  00B2DF78  push     [addr_context.register]
0031 14 00  00B2DF94  pop      [addr_context.esp] (4 bytes)
0032 23 02  00B2DFA2  load     context.register,083B1BB9
0033 23 01  00B2DFDA  mov_reg  [addr_context.eax],context.eax
0034 23 02  00B2DFE8  load     context.register,08169770
0035 2D 02  00B2DFF6  push     0000047B
0036 1B 02  00B2E012  mov      [addr_context.register],addr_context.eax
0037 14 00  00B2E004  pop      [context.register] (4 bytes)
0038 13 00  00B2E020  load     context.register,064E0B3F
0039 33 02  00B2E02E  mov      [addr_context.register],addr_context.eax
0040 2D 02  00B2E074  push     [context.register]
0041 12 00  00B2E04A  mov      [addr_context.register],context.ebp
0042 0E 00  00B2E066  add      [addr_context.register],034F1B41
0043 32 02  00B2E03C  pop      [context.register] (4 bytes)
0044 12 01  00B2E058  load     context.register,0753383F
0045 1B 02  00B2E082  mov      [addr_context.register],context.ebp
0046 1E 00  00B2E09E  add      [addr_context.register],035D4CFA
0047 19 01  00B2E0AC  push     [addr_context.register]
0048 12 00  00B2E0BA  mov      [addr_context.register],addr_context.ebx
0049 01 01  00B2E090  pop      [context.register] (4 bytes)
0050 1B 02  00B2E0C8  mov      [addr_context.register],context.esp
0051 11 02  00B2E0F2  sub      [addr_context.register],00000004
0052 2D 02  00B2E0D6  push     [addr_context.register]
0053 32 02  00B2E10E  pop      [addr_context.esp] (4 bytes)
0054 2A 00  00B2E11C  push     00B2E146
0055 33 02  00B2E12A  mov      [addr_context.register],context.esp
0056 01 01  00B2E138  pop      [context.register] (4 bytes)
0057 12 00  00B2E0E4  mov      [addr_context.register],addr_context.ebx
0058 33 01  00B2E146  call     [context.register]
0059 25 02  00B2E100  load     context.register,0021A180
0060 2C 01  00B2E154  mov      [addr_context.register],context.esp
0061 2B 02  00B2E162  push     [context.register]
0062 12 00  00B2E19A  mov      [addr_context.register],addr_context.ebx
0063 32 02  00B2E17E  pop      [context.register] (4 bytes)
0064 1B 02  00B2E170  mov      [addr_context.register],context.esp
0065 0E 01  00B2E1B6  add      [addr_context.register],00000004
0066 2D 02  00B2E18C  push     [addr_context.register]
0067 14 00  00B2E1A8  pop      [addr_context.esp] (4 bytes)
0068 33 02  00B2E1C4  mov      [addr_context.register],context.esp
0069 16 00  00B2E226  push     [context.register]
0070 1F 01  00B2E218  mov      [addr_context.register],addr_context.eax
0071 14 00  00B2E1E0  pop      [context.register] (4 bytes)
0072 1B 02  00B2E1D2  mov      [addr_context.register],context.esp
0073 37 02  00B2E1EE  add      [addr_context.register],00000004
0074 2A 00  00B2E234  push     [addr_context.register]
0075 01 01  00B2E1FC  pop      [addr_context.esp] (4 bytes)
0076 13 00  00B2E20A  load     context.register,06F15901
0077 2C 01  00B2E242  mov      [addr_context.register],context.ebp
0078 26 00  00B2E26C  add      [addr_context.register],034F0C4D
0079 2B 02  00B2E27A  push     [context.register]
0080 2B 02  00B2E250  push     00000000
0081 27 00  00B2E25E  cmp      [esp+4],[esp] (4 bytes)
0082 16 01  00B2E288  jz       00B2E58A
0083 2C 01  00B2E296  mov      [addr_context.register],addr_context.eax
0084 2B 02  00B2E2A4  push     [context.register]
0085 1F 01  00B2E2DC  mov      [addr_context.register],context.esp
0086 2D 00  00B2E2CE  sub      [addr_context.register],00000004
0087 32 02  00B2E2EA  pop      [context.register] (4 bytes)
0088 19 01  00B2E2B2  push     [addr_context.register]
0089 14 00  00B2E2C0  pop      [addr_context.esp] (4 bytes)
0090 33 02  00B2E2F8  mov      [addr_context.register],addr_context.ebx
0091 16 00  00B2E34C  push     [context.register]
0092 12 00  00B2E314  mov      [addr_context.register],context.esp
0093 2D 00  00B2E306  sub      [addr_context.register],00000004
0094 32 02  00B2E322  pop      [context.register] (4 bytes)
0095 16 00  00B2E33E  push     [addr_context.register]
0096 32 02  00B2E330  pop      [addr_context.esp] (4 bytes)
0097 2B 02  00B2E35A  push     0000047B
0098 1B 02  00B2E368  mov      [addr_context.register],addr_context.eax
0099 14 00  00B2E384  pop      [context.register] (4 bytes)
0100 13 00  00B2E376  load     context.register,0C7E36F0
0101 1F 01  00B2E392  mov      [addr_context.register],addr_context.eax
0102 2B 02  00B2E3A0  push     [context.register]
```
