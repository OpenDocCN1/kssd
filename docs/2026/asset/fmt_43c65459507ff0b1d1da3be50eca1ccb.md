
```asm
ADRP Xd, val\_page\_addr ;@page Xd = addr + (PC - (PC & 0xfff))
add Xd, Xd, :lo12:val ;@pageoff
```

`ADRP`指令是以页为单位的大范围的地址读取指令，P为`page`, `pageoff`为`page offset`，这里填写的地址是最终地址的页基址，**汇编会自动把opcode转换成相对于pc页基址的地址。**

符号扩展一个21位的offset（`immhi+immlo`）。向左移动12位，将`PC`的值的低12位清零，然后把这两者相加，结果写入到`Xd`寄存器，用来得到一块含有label的4KB对齐内存区域的base地址（也就是说label所在的地址，一定落在这个4KB的内存区域里，指令助记符里Page也就是这个意思），可用来寻址 +/- 4GB的范围（2^33次幂）。

下面是几种指令的机器码比较：

```ASM
31 27 26 25 24 05 00
LDR32 0 0 0 1 1 0 0 0 imm19 (4倍されて, ±1MB) Rt(5bit)
LDR64 0 1 0 1 1 0 0 0 imm19 (4倍されて, ±1MB) Rt(5bit)
LDRSW 1 0 0 1 1 0 0 0 imm19 (4倍されて, ±1MB) Rt(5bit)
31 29 24 05 00
ADR 0 immL(2bit) 1 0 0 0 0 immHi (19bit) Rd(5bit)
ADRP 1 immL(2bit) 1 0 0 0 0 immHi (19bit) Rd(5bit)
```

下面代码说明了，`adrp`的机器码`immhi+immlo`存储的是相对于当前页面的页偏移0x22000，但是反汇编后显示的则是目标的`VA`，即`adrp x0, #0x25000`(当此指令在0x3000处时候，`cs.disasm(code, 0x3000)`)。

```python
from capstone import Cs, CS\_ARCH\_ARM64, CS\_MODE\_ARM
code = b'\x00\x01\x00\xd0'
cs = Cs(CS\_ARCH\_ARM64, CS\_MODE\_ARM)
i = next(cs.disasm(code, 0x3000))
print(i.mnemonic, i.op\_str) # adrp x0, #0x25000
code\_int = int.from\_bytes(code, 'little')
immHi = (code\_int&0x00ffffff)>>5
immL = code\_int>>29 & 0b11
imm =(immHi<<2|immL)<<12
print(f"code\_int=0b{code\_int:b}") # code\_int=0b11010000000000000000000100000000
print(f"immHi=0b{immHi:b}, immL=0b{immL:b}, imm=0x{imm:x}") # immHi=0b1000, immL=0b10, imm=0x22000
```

### (2) @page，@pageoff寻址

`adrp @page` 找到目标页地址，`add @pageoff`加上页内偏移。如果用ida的keypatch修改，需要去掉`@page`，`@pageoff`后缀，手动来计算目标页地址和页内偏移才行。ida和`objdump`的反汇编代码如下：

```asm
// ida hello\_arm64
.text:00000000000009DC main ; DATA XREF: .got:main\_ptr↓o
.text:00000000000009DC FD 7B BF A9 STP X29, X30, [SP,#-0x10+var\_s0]!
.text:00000000000009E0 FD 03 00 91 MOV X29, SP
.text:00000000000009E4 00 00 00 90 ADRP X0, #aHelloWorld@PAGE ; "hello world",
.text:00000000000009E8 01 00 2D 91 ADD X1, X0, #aHelloWorld@PAGEOFF;0xb40
.text:00000000000009EC 80 00 00 90 ADRP X0, #\_ZSt4cout\_ptr@PAGE
.text:00000000000009F0 00 E4 47 F9 LDR X0, [X0,#\_ZSt4cout\_ptr@PAGEOFF]
.text:00000000000009F4 9F FF FF 97 BL .\_ZStlsISt11char\_traitsIcEERSt13basic\_ostreamIcT\_ES5\_PKc ; std::operator<<<std::char\_traits<char>>(std::ostream &,char const\*)
0000000009F8 E2 03 00 AA MOV X2, X0
.text:00000000000009FC 80 00 00 90 ADRP X0, #\_ZSt4endlIcSt11char\_traitsIcEERSt13basic\_ostreamIT\_T0\_ES6\_\_ptr@PAGE
.text:0000000000000A00 01 DC 47 F9 LDR X1, [X0,#\_ZSt4endlIcSt11char\_traitsIcEERSt13basic\_ostreamIT\_T0\_ES6\_\_ptr@PAGEOFF]
.text:0000000000000A04 E0 03 02 AA MOV X0, X2
.text:0000000000000A08 9E FF FF 97 BL .\_ZNSolsEPFRSoS\_E ; std::ostream::operator<<(std::ostream & (\*)(std::ostream &))
.text:0000000000000A0C 00 00 80 52 MOV W0, #0
.text:0000000000000A10 FD 7B C1 A8 LDP X29, X30, [SP+var\_s0],#0x10
.text:0000000000000A14 C0 03 5F D6 RET
// aarch64-linux-gnu-objdump -d hello\_arm64
00000000000009dc <main>:
9dc: a9bf7bfd stp x29, x30, [sp, #-16]!
9e0: 910003fd mov x29, sp
9e4: 90000000 adrp x0, 0 <\_init-0x810> ; aHelloWorld pageaddr
9e8: 912d0001 add x1, x0, #0xb40 ; #aHelloWorld@PAGEOFF
9ec: 90000080 adrp x0, 10000 <\_\_FRAME\_END\_\_+0xf340>
9f0: f947e400 ldr x0, [x0, #4040]
9f4: 97ffff9f bl 870 <\_ZStlsISt11char\_traitsIcEERSt13basic\_ostreamIcT\_ES5\_PKc@plt>
9f8: aa0003e2 mov x2, x0
9fc: 90000080 adrp x0, 10000 <\_\_FRAME\_END\_\_+0xf340>
a00: f947dc01 ldr x1, [x0, #4024]
a04: aa0203e0 mov x0, x2
a08: 97ffff9e bl 880 <\_ZNSolsEPFRSoS\_E@plt>
a0c: 52800000 mov w0, #0x0 // #0
a10: a8c17bfd ldp x29, x30, [sp], #16
a14: d65f03c0 ret
```

### (3) 编程修改ARM汇编字符串地址

下面的python脚本说明了如何用`capstone`进行反汇编，如何手动求`目标页地址`和`页内偏移`，以及之后用`keystone`进行汇编更新`adrp @page`和`add @pageoff`指令。

```python
import lief
from keystone import Ks, KS\_ARCH\_ARM64, KS\_MODE\_LITTLE\_ENDIAN
from capstone import Cs, CS\_ARCH\_ARM64, CS\_MODE\_ARM
# add section (as well as segment)
arm64elf = lief.parse("./build/hello\_arm64")
new\_sect = lief.ELF.Section(".new", lief.ELF.SECTION\_TYPES.PROGBITS)
new\_sect += lief.ELF.SECTION\_FLAGS.WRITE # .EXECINSTR
new\_sect.alignment = 4
new\_sect.content = list(bytes('xxhooked str hello world!\n', encoding='ansi'))
new\_sect = arm64elf.add(new\_sect)
#9E4 00 00 00 90 ADRP X0, #aHelloWorld@PAGE ; "hello world"
#9E8 01 00 2D 91 ADD X1, X0, #aHelloWorld@PAGEOFF ; "hello world"
code = b"\x00\x00\x00\x90\x01\x00\x2d\x91"
offset = 0x9e4
text\_sect = arm64elf.get\_section(".text")
addr = text\_sect.search(code) + text\_sect.virtual\_address
# disassembly adrp "hello sdr"
cs = Cs(CS\_ARCH\_ARM64, CS\_MODE\_ARM)
cs.detail = True
for i in cs.disasm(code, addr):
print(i.mnemonic, i.op\_str)
# change addrss to new sect
ks = Ks(KS\_ARCH\_ARM64, KS\_MODE\_LITTLE\_ENDIAN)
page = new\_sect.virtual\_address - (new\_sect.virtual\_address & 0xfff) # address page
pageoff = (new\_sect.virtual\_address+2) & 0xfff
asm\_str = "adrp x0, #0x{:x};add x1, x0, #0x{:x}".format(page, pageoff)
print("rebuild\_asm:", asm\_str)
code\_rebuild, \_ = ks.asm(asm\_str, addr)
print("rebuild\_code:", "".join([f"{i:02x}" for i in code\_rebuild]))
for i in cs.disasm(bytes(code\_rebuild), addr):
print(i.mnemonic, i.op\_str)
# patch file
arm64elf.patch\_address(addr, code\_rebuild)
arm64elf.write(r"./build/hello\_arm64\_rebuild")
```

运行patch脚本后，qemu测试成功！

![arm64\_hookstr](upload/attach/202108/617776\_8K3TQH3R5FDHCV8.jpg)

## 0x5 后记

这篇教程虽然难度不高，本来以为很快就能写完了，但是写了好久。很多东西叙述起来很麻烦，我尽量结合实例清晰和有条理性地讲述，不知不觉中写了好多东西。目前关于主机系列的汉化教程非常少，以往很多的教程也都随着论坛的关闭等无法访问了。同样，比起`x86`的汇编的修改，`arm64 elf`之类的分析也不多。我系统地做《galgame汉化中的逆向》系列是想总结一下汉化方法，结合我自身对一些问题的思考和理解，以使汉化逆向这门技术今后能更好地传承下去。

# Galgame汉化中的逆向（特别篇一）：ArmArm64\_ELF中汉化字符串超长修改方法

by [devseed](https://bbs.kanxue.com/elink@e69K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6k6N6i4u0A6f1$3W2*7N6h3E0#2i4K6u0r3c8$3q4D9k6$3q4E0k6g2u0W2N6X3g2J5M7$3f1`.), 本贴论坛和[我的博客](https://bbs.kanxue.com/elink@62cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2M7$3y4Z5L8X3g2W2i4K6u0W2L8h3!0W2i4K6u0r3M7r3!0K6N6s2y4Q4x3V1k6s2j5h3I4Y4j5h3#2W2f1X3g2$3k6i4u0K6k6g2)9#2k6Y4y4H3k6h3y4A6j5h3H3I4)同时发布

## 0x0 前言

之前发的Galgame汉化中的逆向系列，大多是结合具体游戏来谈谈分析思路与汉化方法。大多数主机游戏可执行文件都是魔改的`ELF`文件（比如说psv，ps4系统内核就是由FreeBSD修改而来的），不同于以`x86`架构为主的pc平台，主机平台架构可谓是百花齐放：**psp，ps2是`mips`架构，ps3是`powerpc`架构，ps4是`x86`架构，psv是`arm`架构，switch是`arm64`架构**。ps4的`x86`架构修改方法和pc版类似，`powerpc`架构的汇编暂时还没研究，`mips`架构汇编相关的分析，今后有时间可能会更新教程。由于`arm`指令一般情况下都是4字节定长（还有2字节的`thumb`指令），`mov`，`ldw`等指令的立即数有范围限制，寻址方式多为相对于`PC`的偏移寻址，这与`x86`里立即数可以直接写VA地址（如`FF 15`系列长跳转指令、`b8`系列的`mov`）有很大区别。因此这篇作为《galgame汉化中的逆向》系列教程的补充，来聊聊**ELF结构、如何对ELF修改、Arm汇编寻址方法以及如何修改Arm汇编中字符串的指针**。

一般来说主机游戏剧本、字库等都在资源文件（封包文件）中，可执行文件中只有少量的系统文本。这部分通常也不需要汉化比原字符长（因为系统文本一般没有太多含义，且汉语本身就比较简洁），因此直接替换二进制字符就行了。但是对于一些特列，即字库或剧本整体封在了可执行文件中，就要增加区段和用汇编修改指针把字符串重定向到新增区段中了。如psv、switch版的Gnosia，和psv、ps3版的WhiteAlbum2，修改eboot方法详见[PSV版WA2汉化移植](https://bbs.kanxue.com/elink@995K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6%4k6i4c8G2M7W2)9J5c8W2m8e0g2W2)9#2k6W2N6t1d9g2c8q4i4K6g2X3b7f1I4n7g2f1@1J5i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8Y4m8K6N6W2)9#2k6Y4N6S2x3W2)9#2k6X3g2T1L8$3!0@1i4K6g2X3N6r3!0G2L8q4)9J5k6i4m8&6)。

为了更直观地说明原理和看到运行结果，本篇教程将不以具体游戏为例，而是用`arm`版helloword程序，讲解一下修改`arm`、`arm64` `ELF`的方法。理论上适用于psv、swich、android平台的可执行文件或动态库，根据不同平台可能需要再转换为其对应的魔改`ELF`格式。

## 0x1 工具准备

本篇教程需要的工具：

- Debain系Linux（WSL、虚拟机或实机）

- lief（elf解析与编辑），readelf（binutil里的elf查看器），hexdump（查看二进制）

- keystone（arm assembler），capstone（arm disassembler）

- qemu-arm-static，qemu-aarch64-static（qemu-user模式，用于运行arm elf）

- arm-linux-gnueabihf-g++，aarch64-linux-gnu-g++ （arm, arm64交叉编译）

- ida，ida64，arm-linux-gnueabihf-objdump，aarch64-linux-gnu-objdump （查看反汇编）

参考网站：

2aaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2E0P5Y4c8F1i4K6u0W2L8%4u0Y4i4K6u0r3k6s2u0S2k6$3!0F1i4K6u0r3j5i4u0E0y4U0b7H3y4i4y4@1M7W2)9J5k6h3S2@1L8h3H3`.

4f1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4J5L8h3y4G2L8Y4k6W2M7Y4c8W2M7W2)9J5k6h3y4G2L8g2)9J5c8R3`.

安装脚本如下：

|  |  |

| --- | --- |
