

# Themida v1.8.0.0 Demo 虚拟机分析

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-808-2.htm`](https://www.pediy.com/kssd/pediy08/pediy8-808-2.htm)

在开始讲述 handler 的执行过程之前,有必要了解 VM 中用到的 2 个重要数据结构,即 VM\_Context 和 PCODE 数据。在手工还原代码时,熟悉这些数据结构尤其重要。我修复代码基本没有用到调试器,是用 IDA 静态分析完成的。

 

执行进入 VM 的准备代码时,将 edi 置为 VM\_Context 的地址,并将 VM\_Context+0 的 DWORD 置为第 1 条指令对应的 PCODE 数据地址(来自 PUSH/JMP 下的数据)。进入 handler 后将该地址值赋给 esi,在 handler 执行过程中,esi/edi 始终不变,用来访问这 2 个结构(变形代码使用这 2 个寄存器会用 push/pop 加以保护)。

 

VM\_Context\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

 

有些 offset 对应的地址 handler 没有使用(我没看见?J),有的是为了对齐。下表只列出 handler 明确使用的 field。

 

| ***Offset*** || ***Size*** || ***Name*** || ***Description*** |
| --- | --- | --- | --- |
| ***00*** || ***dword*** || ***pcode\_data*** || ***pcode 数据地址*** |
| ***08*** || ***byte*** || ***Key3*** || ***解码 key3*** |
| ***0C*** || ***byte*** || ***next\_opcode1*** || ***下一条指令的 opcode1*** |
| ***10*** || ***byte*** || ***Key4*** || ***解码 key4*** |
| ***14*** || ***byte*** || ***next\_opcode2*** || ***下一条指令的 opcode2*** |
| ***18*** || ***byte*** || ***Key2*** || ***解码 key2*** |
| ***20*** || ***byte*** || ***Key1*** || ***解码 key1*** |
| ***24*** || ***byte*** || ***flag*** || ***影响 handler 执行方式的标记,根据 pcode 数据设置*** |
| ***28*** || ***byte*** || ***busy*** || ***VM 忙闲标记*** |
| ***30*** || ***dword*** || ***AddrOfRegister*** || ***VM\_Context 内某个通用寄存器的地址,调用 pfnGetRegister 的返回值,见+5C*** |
| ***34*** || ***dword*** || ***register*** || ***可以看作 VM 自己的唯一一个寄存器*** |
| ***38*** || ***dword*** || ***rotated\_bytes*** || *** VM\_Context 内通用寄存器环滚动字节数*** |
| ***3C*** || ***dword*** || ***addr\_of\_regs\_ring*** || ***通用寄存器环起始地址,这个地址等于 VM\_Context 基地址+74,即 VM\_Context 内 eax 的地址*** |
| ***40*** || ***dword*** || ***2-bytes-opcode-table*** || ***即前文的 2-Bytes-Opcode Table1 地址*** |
| ***44*** || ***dword*** || ***1-byte-opcode\_table*** || ***1-Byte-Opcode Table 地址*** |
| ***48*** || ***dword*** || ***Jmp\_flag*** || ***跳转指令是否执行跳转的标记*** |
| ***4C*** || ***dword*** || ***pcode\_beginning*** || ***VM 保护代码对应 pcode 数据起始地址,即 PUSH/JMP 下第 1 个 dword+DeltaOffset*** |
| ***50*** || ***dword*** || ***pcode\_endding*** || ***Pcode 数据结束地址,等于+4C 的值加 PUSH/JMP 下第 2 个 dword(pcode data size)*** |
| ***54*** || ***dword*** || ***DeltaOffset*** || ***J*** |
| ***58*** || ***dword*** || ***vm\_entry*** || ***指向 1 字节 opcode 的 handler 搜索代码(即使用 pcode+0 的低 7 位定位 handler 的代码),当执行跳转指令,控制转移到目的地址时也用这里的代码*** |
| ***5C*** || ***dword*** || ***pfnGetRegister*** || ***这里保存的是个函数地址,用于获取 VM\_Context 内某个通用寄存器的当前地址(这些寄存器随机滚动,Themida 自己也需要一个方便获取 register 地址的方式),参数为寄存器在 VM\_Context 内的 offset(如 74h->eax),返回值保存在+30 处*** |
| ***60*** || ***dword*** || ***null\_const\_1*** || ***这 2 个 dword 含义不明,在一定条件下用于调整解码出的 pcode 数据,由于这 2 个 dword 总为 0, 似乎并没有实际的作用*** |
| ***64*** || ***dword*** || ***null\_const\_2*** |
| ***68*** || ***dword*** || ***addr\_of\_pcode\_info*** || ***指向 PUSH/JMP 下 pcode 相关数据的起始地址*** |
| ***6C*** || ***dword*** || ***num\_of\_pcode\_items*** || ***PUSH/JMP 下数据项数(不含非模仿指令)*** |
| ***70*** || ***dword*** || ***eflag*** || ***从这里开始为原执行环境数据*** |
| ***74*** || ***dword*** || ***eax*** || ***这里(包括 cs 前的空间)即所谓的通用寄存器闭合环,在 handler 连续执行的过程中会随机滚动,调试时不容易看清代码究竟要访问哪一个寄存器。******这对解码程序不存在干扰,pcode 解码数据不包含滚动因素。当访问某个通用寄存器时,提供的参数为偏移量数据,该值减 F0000000 后代表闭合环内的 offset,如 F0000000 –>eax,F0000008->ebx*** |
| ***7C*** || ***dword*** || ***ebx*** |
| ***84*** || ***dword*** || ***ecx*** |
| ***8C*** || ***dword*** || ***edx*** |
| ***94*** || ***dword*** || ***esi*** |
| ***9C*** || ***dword*** || ***edi*** |
| ***A4*** || ***dword*** || ***ebp*** |
| ***AC*** || ***dword*** || ***esp*** |
| ***E4*** || ***word*** || ***cs*** ||  |
| ***E6*** || ***word*** || ***ds*** ||  |
| ***E8*** || ***word*** || ***es*** ||  |
| ***EA*** || ***word*** || ***ss*** ||  |

 

有些 field 虽然 size 为 byte,但常用 32 位方式访问,只用了最低字节,如几个 key。另外,对 key 的编号只是我的习惯,没什么特殊意义。这些 field 后面的空间是实现寄存器滚动用的 buffer。再后面即搜索 1 字节 opcode 对应 handler 的代码。 

 

VM\_Context 内各成员的 offset 和含义是固定的(除寄存器滚动部分)。

 

PCODE 数据\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

 

PCODE 数据的长度是固定的,为 14 字节(并没有用满)。每条指令都有自己的 PCODE 数据,所以也可以把 PCODE 数据的地址看作 VM 的 EIP。如果与 x86 指令类比,PCODE 数据相当于 opcode 以外的部分。

 

| 

# Name

 || 

# Size

 || 

# Decode Key

 || 

# Description

 |
| --- | --- | --- | --- |
| ***Init\_keys*** || ***1 byte*** ||  || ***这个 field 始终位于+0 处,若 b7 为 1,4 个 key 使用前先清 0。对于 1 字节 opcode,低 7 位即 opcode 值,对 2 字节 opcode,低 7 位未用*** |
| ***InitKey1\_and******\_NumOfRotate*** || ***1 byte*** ||  || ***b0,b1 决定如何变换 Key1:******0-> key1 ^= seed\_xor(加壳时生成的 imm8)******1-> Key1 += seed\_add******2-> key1 ^= pcode\[0\]******3-> key1 += pcode\[0\]******b2,b3,b4 3 位对应值代表 VM\_Context 内寄******存器环应如何滚动,为 0 不滚动,否则滚动字节数为该 3 位对应值 \* 2*** |
| ***Argument*** || ***4 bytes*** || ***Key1*** || ***handler 的参数,具体含义与 handler 相关******解码方式以伪码表示为:******for(int i = 0; i < 4; i++)******{******   \_asm ror Argument, 4 ****** if(Key1 > 0x80)******  Argument ^= Key1;******else******  Argument += Key1;******}******与 Key1 的运算用最低字节*** |
| ***Opcode detail*** || ***2-4******bytes*** || ***Key1*** || ***这个 field 我不知取什么名字好,暂且这样。不同类型的 handler 使用的字节数不同,位置也可以不连续,划到一起是因为其作用。每个字节的解码方式为:******data =( (data ^ Key1) + seed\_add) +******       Key1*** |
| ***Next\_pcode*** || ***1 byte*** || ***Key3*** || ***下条指令的 pcode 地址,解码方式为:******data = ((data ^ key3) ^ seed\_xor) – seed\_sub******key3 用 data 原值更新:******key3 ^= old\_data*** |
| ***Next\_opcode1*** || ***1 byte*** || ***Key4*** || ***下条指令的 opcode1,解码方式为:******data = ((data ^ key4) ^ seed\_xor) – seed\_sub******key4 用 data 原值更新:******key4 ^= old\_data*** |
| ***Next\_opcode2*** || ***1 byte*** || ***Key2*** || ***下条指令的 opcode2,解码方式为:******data = ((data ^ key2) ^ seed\_xor) – seed\_sub******key2 用 data 原值更新:******key2 ^= old\_data*** |

 

表内所有的 seed\_xxx,均为加壳时生成的随机字节值,各不相同。对 1 字节 opcode 表内的 handler,其使用的 pcode 数据内各 field 的 offset 及含义是固定的。2 字节 opcode 表内的 handler(与 1 字节 opcode 表内数据重叠的除外)各 field 在 PCODE 结构内的 offset 可变,所以这里没有列出各 field 的偏移量。还原代码时,熟悉 PCODE 的各 field 用哪一个 key 解码很重要,否则难以识别代码究竟在处理什么数据。

 4dd466d0e1e0cc123afbe504da2462ca