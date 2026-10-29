
对 `X-Medusa` 来说，目前更直接有效的方式是：不追求完整解释器，而是 `lift` 实际执行路径和关键 `native` `helper`。

## 结语

这次 `X-Medusa` 的逆向过程，从最初只知道 VM 入口 `0x445b8`，到最终得到纯 Python 复现，大致经历了：

```text
定位 header 插入点
确定 VM 执行边界
分析 packet 外层结构
倒推 second_buffer
还原 reverse-xor
还原 d71bc byte-state-machine
确认 SM3 key material
解析 src_a protobuf
还原 f23/f24 动态字段
还原 bit-slice patch
还原 AES-like transform
组合端到端 pipeline
native 对照验证
```

最终结果：

```text
src_a_match=True
x_medusa_match=True
```

这也是本文最核心的判断标准：不是“猜到了算法像什么”，而是“同一次 native 输入下，Python 输出完全一致”。

# AI逆向某视频签名算法X-Medusa全过程

> 本文仅记录一次针对移动端 `native` 签名逻辑的逆向分析过程，用于安全研究、算法学习和逆向工程方法论交流。文中涉及的脚本、地址和结论均来自本地样本与模拟环境验证，不讨论任何绕过风控、批量请求或业务滥用场景。

## 前言

这次分析的目标是某视频 App `29.3` 版本中的一个 `native` 签名头：`X-Medusa`。

样本位于 Android `native` `so`：

```
libmetasec_ml.fully_deobf.so
```

外层调用函数是：

```
sub_D4F3C(url, headers_blob)
```

这个函数会一次性生成多个签名头：

```
X-Argus

X-Gorgon

X-Helios

X-Khronos

X-Ladon

X-Medusa
```

本文重点只讲 `X-Medusa`。

我最终将 `X-Medusa` 主路径还原成了纯 Python，可以在不启动 `native` VM 的情况下，只输入同一次运行的动态值，生成和 `native` 一致的 `X-Medusa`。

整个过程并不是一开始就直接进入算法还原。前半段我先用 Cursor 的 Opus 模型搭建和调通 `ExAndroidNativeEmu` 调用环境，它帮我把 `sub_D4F3C` 的 `native` 签名调用跑起来，也就是能从本地 `emu` 里拿到各个签名头。到这一步后，继续深入 `X-Medusa` 内部时，分析基本卡在 SM3 和周边混淆逻辑，无法继续稳定拆出后续 VM 路径。

后半段切到 Codex 后，分析方式变成了“动态 `trace` + 局部 Python `lift` + `native` 对照验证”。也就是本文后面记录的过程：不再只看静态伪代码，而是对每个 VM 片段抓输入、输出和实际内存读写变化，再把能证明的局部逻辑写成 Python，最后组合成完整 `pipeline`。

最终验证结果：

```
[rebuild] src_a_match=True x_medusa_match=True
```

也就是说，Python 重建出的明文 `src_a` 和最终 `X-Medusa` 都与 `native` 同一次运行完全一致。

## 分析环境

本次使用的是一个基于 `ExAndroidNativeEmu` 的本地模拟环境。目录中已有调用示例：

```
dy_sign_send.py
```

它负责：

```
1. 初始化 Android native emu

2. 加载 libmetasec_ml.fully_deobf.so

3. 调 JNI_OnLoad

4. 调用 sub_D4F3C

5. 解析返回的签名 header
```

分析过程中还遇到一个环境问题：系统里存在不匹配的 Unicorn `dylib`，会影响 `emu` 运行。后面所有 `native` 对照命令都统一这样跑：

```
env -u DYLD_LIBRARY_PATH python3 ...
```

避免 Python 加载错误的 Unicorn 动态库。

## 第一步：先定位 X-Medusa 在 sub_D4F3C 中的位置

一开始没有直接钻 VM，而是先观察 `sub_D4F3C` 的输出 `map`/`string` 插入位置。

最终确认各 `header` 的插入点：

```
0xca684 -> X-Gorgon

0xcaa5c -> X-Khronos

0xcb09c -> X-Argus

0xcb4dc -> X-Ladon

0xcba30 -> X-Medusa

0xcbe64 -> X-Helios

0x46a70 -> X-Neptune
```

其中 `X-Medusa` 的关键路径是：

```
sub_D4F3C

  -> native wrapper

  -> VM entry lib+0x445b8

  -> Medusa VM bytecode call lib+0xd8978

  -> bytecode entry base+0x18bdd0

  -> X-Medusa 插入点 lib+0xcba30
```

这里最重要的是确定 VM 执行边界：

```
VM interpreter entry: lib+0x445b8

Medusa bytecode call: lib+0xd8978

VM bytecode entry:    base+0x18bdd0
```

有了这个边界，后续 `hook` 只在 Medusa VM 活跃期间记录，避免被 JNI 初始化、其它 `header` 或环境探测逻辑干扰。

## 第二步：确认 VM 解释器形态

最开始静态看 `0x445b8` 这个 VM 入口，会发现它很像 MIPS 风格解释器：

```
32 个虚拟寄存器

load/store

branch

R/I/J 类似的指令形态
```

但不能直接把它当标准 MIPS。

我写了一个 Python VM 模型和 `native` 状态对比脚本，核心思路是：

```
1. 在 native VM 初始化后抓 VM state

2. 在 VM fetch/decode 点抓每一步 PC 和寄存器

3. Python 模型模拟一条

4. 和 native 下一步状态对比
```

VM `fetch`/`decode` 点：

```
lib+0x4466c
```

对比后得到一个重要结论：

```
这个 VM 是 MIPS-like，但不是标准 MIPS。

r0 也不是硬编码 zero。
```

也就是说，不能做这样的假设：

```
r0 永远等于 0
```

部分 R-type 指令的目标寄存器编码也和标准 MIPS 有差异。所以后续还原关键逻辑时，我没有完全依赖静态反汇编，而是优先使用动态 `trace` 的输入、输出和实际内存读写变化。

## 第三步：从最终 X-Medusa 反推 packet 结构

接下来先看最终 `X-Medusa` 是什么。

通过跟踪 `base64` 调用链：

```
0xdc608 -> 0xee588 -> 0x10dd84
```

确认 `X-Medusa` 是标准 `base64`，使用普通字母表：

```
ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/
```

`base64` 解码后得到一个 `raw packet`。继续跟踪最终 `packet` 的 `copy` 序列，得到结构：

```
packet = fixed20 || var2 || 00 || 01 || body_first || body

fixed20     = c312f41237faabe817c5282916383c6297dd950a

var2        = 2 字节变量，后来确认等于 d71bc_seed32_le[:2]

00 01       = 固定 flags

body_first  = AES-like 输出的第 32 字节

body        = patch 后的 second_buffer
```

VM `copy` 点如下：

```
vm+0x191234: copy dst+0x00 len=20

vm+0x191264: copy dst+0x14 len=2

vm+0x1912a4: copy dst+0x16 len=1

vm+0x1912dc: copy dst+0x17 len=1

vm+0x191320: copy dst+0x18 len=1

vm+0x191360: copy dst+0x19 len=N
```

这个阶段先写出最外层 Python：

```
packet = fixed20 + var2 + b"\x00\x01" + bytes([body_first]) + body

X_Medusa = base64.b64encode(packet)
```

## 第四步：倒推 body 的来源

继续追 `packet body`，发现最终 `body` 来自一个中间 `buffer`，但不是直接复制出来的。

它的生成分成两步：

```
1. 先拼 second_buffer

2. 再 patch second_buffer 前 31 个 64-bit word 的稀疏 bit 位
```

`second_buffer` 布局：

```
second_buffer =

  byte[0]      = 0xa6

  byte[1:9]   = seed8

  byte[9:...] = first_intermediate

  tail         = tail2
```

其中：

```
seed8 = uint32_le(rand@vm+0x18ec30) || 013a0b00

tail2 = uint32_le(rand@vm+0x18e408)[:2]
```

关键 VM 写点：

```
vm+0x18eda4  写 first byte

vm+0x18edcc  copy 8-byte seed

vm+0x18edf4  copy first_intermediate

vm+0x18ee24  copy tail2

vm+0x1910b8  31 次 sparse word patch
```

这一步对应的 Python `lift`：

```
def medusa_second_buffer_layout(first_byte, seed8, first_intermediate, tail2):

    return bytearray(bytes([first_byte]) + seed8 + first_intermediate + tail2)
```

验证方式是同一次 `native` `run` 中抓取 `copy` 的 `source`/`destination`/`len`，然后和 Python 拼出来的 `buffer` 做 `byte-for-byte` 比较。

## 第五步：还原 reverse-xor

`first_intermediate` 继续往前追，来到 VM 片段：

```
vm+0x18eb4c..0x18eb98
```

静态看这里时很容易把 `source` 和 `destination` 看反。所以我对这个范围做了窄范围动态 `trace`，记录每一轮：

```
source pointer

destination pointer

source byte

key byte

written byte

loop index
```

最终确认真实逻辑：

```
for (i = 0; i < n; i++)

    dst[i] = src[n - 1 - i] ^ key4[i & 3];
```

也就是：

```
反向读取 source

正向写入 destination

使用 4 字节循环 xor key
```

`key` 的来源在前面：

```
vm+0x18eacc..0x18eb00
```

它调用一个短 VM `helper`，对 `tail2` 做 `hash`，取低 16 位组成 4 字节 `key`：

```
ret = block_189850_tail2_hash(tail2)

k = ret & 0xffff

key4 = bytes([k >> 8, k & 0xff, k >> 8, k & 0xff])
```

验证样例：

```
fe1e -> ret=fff80f18 -> key4=0f180f18

a057 -> ret=fffaff0d -> key4=ff0dff0d

c2c0 -> ret=fff9effb -> key4=effbeffb
```

## 第六步：找到 reverse-xor 的上游 d71bc

`reverse-xor` 的输入不是原始明文，而是：

```
reverse_source = 8 zero bytes || d71bc_output
```

继续追上游，发现：

```
vm+0x1897d0..0x1897f0  memset(dst, 0x20, len)

vm+0x1897f4..0x18982c  native helper call

native helper          lib+0xd71bc
```

进入 `lib+0xd71bc` 时参数：

```
x0 = dst

x1 = src_a

x2 = len(src_a)

x3 = key32

x4 = 0x20
```

这里一开始也容易误判为某种标准加密算法，但动态 `trace` 后发现它不是 AES/SM4 这种标准 `block cipher`，而是一个混淆过的字节状态机。

整体结构：

```
pass 1:

  正向读 src

  反向写 dst

  每个字节混入 key[(i*4)%32] 和 key[((i*4)|1)%32]



pass 2:

  使用 dst[i-1], dst[i-2], i 做前向反馈



tail:

  dst[n-1] ^= dst[n-2]

  dst[0] = (dst[0] ^ dst[1]) + sum(dst[1:])
```

把它 `lift` 成 Python：

```
block_d71bc_encode(src, key)
```

然后用 `native` `dump` 的 `src_a`/`key`/`dst` 验证：

```
block_d71bc_encode(src_a, key) == native dst
```

## 第七步：还原 d71bc 的 key

`d71bc` 的 `key32` 不是固定表，而是 SM3 结果。

调用链：

```
vm+0x18b6c0 -> wrapper lib+0xdc6bc -> native lib+0xd9bc0
```

对 `b"abc"` 做验证后确认 `lib+0xd9bc0` 是标准 SM3：

```
SM3("abc") =

66c7f0f462eeedd9d1f2d46bdc10e4e24167c4875cf2f7a2297da02b8f4ba8e0
```

`key material` 的构造：

```
sign_key_b64 = rBrarpWnr5SlEUqzs6l92ABQqgo5MUxAUoyuyVJWwow=

sign_key     = base64_decode(sign_key_b64)

seed32_le    = uint32_le(rand@vm+0x18e408)



material68 = sign_key || seed32_le || sign_key

key32      = SM3(material68)
```

所以：

```
d71bc_key = SM3(sign_key || uint32_le(d71bc_rand) || sign_key)
```

这也解释了前面 `tail2`：

```
tail2 = uint32_le(d71bc_rand)[:2]
```

同一个 `rand` 同时参与：

```
1. d71bc key

2. reverse-xor key 派生

3. packet var2/tail2
```

## 第八步：确认三个 rand 的作用

只在 Medusa VM 活跃期间 `trace rand wrapper`，确认共有三次：

```
vm+0x18c788 -> top.f3

vm+0x18e408 -> d71bc seed / tail2 / reverse_key4

vm+0x18ec30 -> second_buffer seed8
```

分别对应：

```
top_rand:

  top.f3 = zigzag32(top_rand)



d71bc_rand:

  seed32_le = uint32_le(d71bc_rand)

  tail2 = seed32_le[:2]

  d71bc_key = SM3(sign_key || seed32_le || sign_key)



seed8_rand:

  seed8 = uint32_le(seed8_rand) || 013a0b00
```

这里有一个坑：`--lock-time` 并不会固定这三个 `rand`。模拟器里的 `rand hook` 来自 Python `random.randint(0, 0xffffffff)`，所以想复现同一次签名，必须捕获这三个 `rand`，或者额外固定随机源。

## 第九步：还原 src_a 明文 protobuf

`d71bc` 的输入 `src_a` 是一个 `protobuf-like` 明文消息。

入口参数里可以直接拿到：

```
lib+0xd71bc:

  x1 = src_a

  x2 = len(src_a)
```

解析后字段如下：
