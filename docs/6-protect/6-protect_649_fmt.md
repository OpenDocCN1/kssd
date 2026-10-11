
# Themida 虚拟机简单介绍

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-598.htm`](https://www.pediy.com/kssd/pediy08/pediy8-598.htm)

## 1. Themida v1.1.1.0 虚拟机代码的简单介绍

之所以写这篇东西，是我打算放弃跟 `v1.5.0.0` 的 VM 代码了，另外，有些东西已经忘了，所以下面的描述可能有错，也不完整。

`v1.1.1.0` 的 VM 代码由多个动态分配的内存块组成，除代码外，有一个全局的数据结构，可称之为 `VM_CONTEXT`，所有的虚拟机代码都要使用这个结构，`v1.5.0.0` 的数据结构没有改变。

结构中最重要的是 `+34` 处的 `dword`，代表 VM 的 `register`。

代码通过 `push/jmp` 指令进入 VM，格式为：

```
push xxxxxx
jmp  xxxxxx
push xxxxxx
jmp  xxxxxx
...
push xxxxxx
jmp  xxxxxx
dd   xxxxxx
dd   xxxxxx
...
dd   FFFFFFFF
```

最后的 `jmp` 下面为 VM 相关数据，第 1 个 `dword` 加 `delta offset` 为 `pcode` 数据起始地址，第 2 个 `dword` 为数据 `size`。下面每 3 个 `dword` 为 1 组，对应一块独立的 `pcode` 数据，也对应上面的 1 个 `push/jmp` 指令对。以 `FFFFFFFF` 结束。

原始代码按 `call` 分块，即原来代码内的 `call` 指令为 `push/jmp` 指令数 - 1。估计是因为 `call` 需要脱离 VM，执行完后重新进入 VM。

每句 `PCODE` 包含如下数据（解码用到 4 个 `key`）：

- `flag_zero_key`: 是否清零 4 个 `key`
- `flag_to_init_key1`: 取值 0 - 3，决定如何初始变换 `key1`
- `num_of_rotate`: `vm_context` 内保存有进入 VM 时的寄存器（执行过程中会改变），这些寄存器视为环状，会随机顺序滚动，这样同样地址的数据可能代表不同的 `reg32`。这个值决定滚动多少字节
- `opcode`: `opcode` 类型，用 `key1` 解码
- `opcode_flag`: `opcode` 的附加信息，用 `key1` 解码
- `argument`: `pcode` 的 `dword` 参数，用 `key1` 解码
- `next_pcode`: 下一条 `pcode` 数据地址，用 `key3` 解码
- `index1`: 用 `key4` 解码
- `index2`: 用 `key2` 解码，这 2 个 `index` 合起来用于查表，搜索下一条 `pcode` 解释函数地址

解释函数的动作包括：初始化各 `key`，解码上述数据，如果需要滚动 `context` 内寄存器，解释执行 `pcode`。计算下一条 `pcode` 地址（保存在全局数据结构 `+0` 处）及其 `handler` 地址，然后 `jmp esi` 执行下一句 `pcode`。

以类似下面的指令退出 VM：

```
push dword ptr ds:[edi+94]
push dword ptr ds:[edi+9C]
mov  dword ptr ds:[edi+28],0
popad
popfd
retn
```

用解码程序可以得到类似下面的 `pcode`，其中第 1 列为 `pcode` 地址，第 2 列为解释函数地址，`register` 为 `vm_context` 内的 `register`，即 VM 自己的 `register`。VM 使用自己的栈，`push` 都是压入 `vm` 栈。

```
00A186CE:011D80CA  mov  register,addr_of_context.ebp
00A186EA:011D0C5D  push dword ptr [register]
00A18706:011DA3DB  mov  register,context.esp
00A18722:011F0000  sub  register,00000004
00A186F8:011D3F6E  rep  movsb [register],[esp] (04 bytes)
00A186DC:011D18AF  push register
00A18714:011D744B  rep  movsb context.esp,[esp] (04 bytes)
00A18730:0123157F  mov  context.ebp,context.esp
00A1873E:01240B8E  mov  register,0D70178C       ; nop
00A1874C:012352F6  add  context.esp,FFFFFFE0
```

等价的真实代码：

```
push ebp
mov  ebp,esp
add  esp,FFFFFFE0
```

当然，也可以考虑自己定义一套 `pcode`，把得到的 `vm pcode` 转换为自己的 `pcode`（或直接跟 Themida 自己的编码），然后把上面的结果转换为真正的汇编码。我是自己读出来的。

加壳时，对 VM 代码做了随机化处理，`pcode` 数据的各个 `field` 摆放的顺序是变化的，对应的 `handler` 执行的操作顺序也在变化，虽然干的事情相同，但处理先后顺序可变。

这样，修复 VM 保护代码时，难以写出通用的解码程序，即使是用同一版本加壳。必须逐个分析 `handler`。从跟的结果看，很多 `handler` 是相同的，区别只是处理顺序。

## 2. Themida v1.5.0.0 虚拟机代码的变化

首先一个变化是混淆代码增强了，也许更早的版本就已经这样了。这是分析 VM 的主要障碍。对 `1110`，可以把清理混淆代码后的 VM 贴回 `dumped_.exe` 直接运行，现在就难以做到了。

基本的指令序列是类似的，但指令序列不连续，分隔指令序列的垃圾代码及其数量也在变化，难以识别或用 `NOP` 替代。

我最后是这样做的：跳着判断指令序列，如果匹配上，将中间的分隔指令全部作为 `junk` 清除。理由是，既然符合混淆指令序列，即由原来的正常指令 `expand` 而来，除匹配指令外，其余的应该是插入的垃圾。这样做的结果似乎还凑合，主要的问题是模式之间存在嵌套，对 1 个模式的处理破坏了别的代码。估计最好能判断出嵌套的模式，然后从内向外处理。

总之费了不少劲，但结果不好。

VM 这次在 1 个连续的内存块内。本身的结构并无多大变化，`pcode` 数据各 `field` 的含义，各 `key` 的使用都和原来一样。主要的区别有 2 处：

`pcode` 数据各 `field` 的 `offset` 固定了，这个不是很肯定，但看了 4 个 `handler` 都相同。各 `field` 在 `pcode` 数据内的偏移为：

```
0x0,    // 是否清零 keys,这个 byte 还参与 key1 的初始化
0xC,    // 初始化 key1
0xC,    // 本轮 rotate bytes,与上面使用 1 byte 不同位
0x1,    // opcode
0x4,    // opcode 附加信息
0x5,    // dword 参数
0xB,    // 下 1 条 pcode 地址
0x9,    // index1
0xA,    // index2
```

这是好事。坏消息则“相当”肯定。每个 `handler` 在解码时使用的常量都不同。如下面一段代码，是解码 `v1.1.1.0` 使用的：

```
void InitKeys(DWORD curr_pcode_data, int nIndex)
{
  // 初始化 4 个解码 key
  BYTE zero_key  = 0;
  BYTE init_key1  = 0;
  ReadProcessData(curr_pcode_data + data[nIndex].offset_zero_key,  &zero_key, 1);
  if(zero_key & 0x80)
  {
    key_1 = 0;
    key_2 = 0;
    key_3 = 0;
    key_4 = 0;
  }
  ReadProcessData(curr_pcode_data + data[nIndex].offset_how_to_init_key1, &init_key1, 1);
  switch(init_key1 & 0x3)  // 对 v1.5.0.0,使用的常数不同
  {
  case 0:
    key_1 ^= 0xD7;
    break;
  case 1:
    key_1 += 0x89;
    break;
  case 2:
    key_1 ^= zero_key;
    break;
  default:
    key_1 += zero_key;
  }
}
```

对于 `v1.1.1.0`，这个函数是固定的，对 `1.5.0.0`，下面 `case 0, case 1` 内的 `0xD7, 0x89` 是变化的，每个 `handler` 都不同。不只是 `key` 的初始化，`opcode`, `opcode_flag`, `dword` 参数，下句 `pcode` 地址及 `handler` 地址，2 个 `index`，所有数据的解码，不同 `handler` 都使用了不同的常数。看到这个我彻底失去了耐心。

`v1.5.0.0` 主程序用 VM 保护的代码有 40 余处。留给有坚强意志的朋友完成吧。

非常抱歉，1 楼的好消息是假的，看过约 20 解释函数后发现各数据 `field` 的 `offset` 仍然是变化的。

`v1.1.1.0` 的 `pcode` 指令条数大约为被保护代码的几倍，应该不会超出 10 倍。即，若用 VM 保护的汇编指令若为 10 行（从 `oep` 到 `call 401644` 就是 10 行），`pcode` 行数应该不会超出 100。

`v1.5.0.0` 有变化，如果在 `401644` 下断，在 `dump` 处 `trace into`，断下后用 `ultraedit` 统计 `run trace` 内的 `jmp esi` 个数为 767。我把原来的程序大概改了一下，先不考虑 `pcode` 的解释，仅跟踪 `pcode` 执行路径，还没有到处理完，已经近千条了。

大致看看，下面的列表，没有显示 `pcode` 的行是还没有做。为了清晰一点，稍微分行了一下。

```
0001  00BB3EDD:01501392  nop
0002  00BB3EEB:0148771E  mov  register,addr_of_context.ebp
0003  00BB3EF9:0150C5DD  push dword ptr [register]
0004  00BB3F07:0148771E  mov  register,context.esp
0005  00BB3F15:014C7A77  sub  register,00000004
0006  00BB3F23:014A1AEA  rep  movsb [register],[esp] (04 bytes)
0007  00BB3F31:0150C5DD  push register
0008  00BB3F3F:014A1AEA  rep  movsb context.esp,[esp] (04 bytes)  ; push ebp
0009  00BB3F4D:01450073  mov  register,0DD6C7B9
0010  00BB3F5B:0148771E  mov  register,addr_of_context.edi
0011  00BB3F69:0150C5DD  push dword ptr [register]
0012  00BB3F77:0148771E  mov  register,addr_of_context.edx
0013  00BB3F85:0150C5DD  push dword ptr [register]
0014  00BB3F93:014D5EF3  test [esp+4],[esp]         ; test edi,edx
0015  00BB3FA1:01450073  mov  register,01FDCEA1
0016  00BB3FAF:0148771E  mov  register,addr_of_context.eax
0017  00BB3FBD:0150C5DD  push dword ptr [register]
0018  00BB3FCB:0148771E  mov  register,context.esp
0019  00BB3FD9:014C7A77  sub  register,00000004
0020  00BB3FE7:014A1AEA  rep  movsb [register],[esp] (04 bytes)
0021  00BB3FF5:0150C5DD  push register
0022  00BB4003:014A1AEA  rep  movsb context.esp,[esp] (04 bytes)  ; push eax
0023  00BB4011:01450073  mov  register,0E111969
0024  00BB401F:0148771E  mov  register,addr_of_context.edx
0025  00BB402D:0150C5DD  push dword ptr [register]
0026  00BB403B:0150C5DD  push FFFFFFBA
0027  00BB4049:014D5EF3  test [esp+4],\esp\      ; test dl,BA
0028  00BB4057:01450073  mov  register,05C35893
0029  00BB4065:0148771E  mov  register,addr_of_context.ebp
0030  00BB4073:0150C5DD  push dword ptr [register]
0031  00BB4081:0150C5DD  push 00000005
0032  00BB408F:01446674  shr  [esp+4],\esp\
0033  00BB409D:014A1AEA  rep  movsb [register],[esp] (04 bytes)  ; shr ebp,5
0034  00BB40AB:01450073  mov  register,0A9C82A3
0035  00BB40B9:0148771E  mov  register,addr_of_context.eax
0036  00BB40C7:0150C5DD  push dword ptr [register]
0037  00BB40D5:0148771E  mov  register,addr_of_context.edi
0038  00BB40E3:0150C5DD  push dword ptr [register]
0039  00BB40F1:014FF38F            ; 未实现
0040  00BB40FF:0148771E  mov  register,addr_of_context.eax
0041  00BB410D:014A1AEA  rep  movsb [register],[esp] (04 bytes)
0042  00BB411B:01450073  mov  register,03CDA03D
0043  00BB4129:0148771E  mov  register,addr_of_context.ecx
0044  00BB4137:0150C5DD  push dword ptr [register]
0045  00BB4145:0148771E  mov  register,addr_of_context.eax
```



`0046 00BB4153:014A1AEA rep movsb [register],[esp] (01 bytes) ; mov eax,ecx`

`0047 00BB4161:0148771E mov register,context.esp`

`0048 00BB416F:0150C5DD push dword ptr [register]`

`0049 00BB417D:0148771E mov register,addr_of_context.eax`

`0050 00BB418B:014A1AEA rep movsb [register],[esp] (04 bytes) ; mov eax,[esp]`

`0051 00BB4199:0148771E mov register,context.esp`

`0052 00BB41A7:0152C47F ; 未实现,可以猜测是 add esp,4`

`0053 00BB41B5:0150C5DD push register`

`0054 00BB41C3:014A1AEA rep movsb context.esp,[esp] (04 bytes)`

`0055 00BB41D1:01450073 mov register,0BA18350`

`0056 00BB41DF:0148771E mov register,addr_of_context.ebp`

`0057 00BB41ED:0150C5DD push dword ptr [register]`

`0058 00BB41FB:0150C5DD push 00000005`

`0059 00BB4209:014F7A55 ; 未实现`

`0060 00BB4217:014A1AEA rep movsb [register],[esp] (04 bytes)`

`0061 00BB4225:01461D1F`

`0062 00BB4233:0148771E mov register,context.esp`

`0063 00BB4241:0150C5DD push dword ptr [register]`

`0064 00BB424F:0148771E mov register,addr_of_context.ebp`

`0065 00BB425D:014A1AEA rep movsb [register],[esp] (04 bytes)`

`0066 00BB426B:0148771E mov register,context.esp ; 可以看到 mov ebp,esp 在这里`

结论是，被转换为`pcode`方式运行的代码，事先被混淆过了。再用这种手工式的修复方法，已经不可行了。

1行真正的汇编指令，大约对应10行`pcode`代码，每条`pcode`执行时，对应的汇编指令一般超出2000行，真正有点疯狂呵呵。
