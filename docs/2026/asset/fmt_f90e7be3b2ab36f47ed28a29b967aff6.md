
- `htype` 就是 Hook 的类型，`callback` 是 hook 回调用；
- `callback` 是 Hook 的处理 handler 指针。请注意！不同类型的 hook，handler 的参数定义也是不同的。
- `user_data` 附加参数，所有的 handler 都有一个 `user_data` 参数，由这里传值。
- `begin` hook 作用范围起始地址
- `end` hook 作用范围结束地址，默认则作用于所有代码。

## Hook callback

不同类型的 hook，对应的 `callback` 的参数也不相同，这里只给出 C 语言定义。

Python 编写 `callback` 的时候参考 C 语言即可（看参数）。

### `UC_HOOK_CODE` & `UC_HOOK_BLOCK` 的 callback 定义

```
typedef void (*uc_cb_hookcode_t)(uc_engine *uc, uint64_t address, uint32_t size, void *user_data);
```

- `address`: 当前执行的指令地址
- `size`: 当前指令的长度，如果长度未知，则为 0
- `user_data`: `hook_add` 设置的 `user_data` 参数

### READ, WRITE & FETCH 的 callback 定义

```
typedef void (*uc_cb_hookmem_t)(uc_engine *uc, uc_mem_type type,
        uint64_t address, int size, int64_t value, void *user_data);
```

- `type`: 内存操作类型 READ, or WRITE
- `address`: 当前指令地址
- `size`: 读或写的长度
- `value`: 写入的值（`type = read` 时无视）
- `user_data`: `hook_add` 设置的 `user_data` 参数

### invalid memory access events (UNMAPPED and PROT events) 的 callback 定义

```
typedef bool (*uc_cb_eventmem_t)(uc_engine *uc, uc_mem_type type,
      uint64_t address, int size, int64_t value, void *user_data);
```

- `type`: 内存操作类型 READ, or WRITE
- `address`: 当前指令地址
- `size`: 读或写的长度
- `value`: 写入的值（`type = read` 时无视）
- `user_data`: `hook_add` 设置的 `user_data` 参数

返回值

返回真，继续模拟执行

返回假，停止模拟执行

# 编译与安装 Unicorn

Unicorn 支持多种编译和安装方式，本课程以 Linux 作为学习环境，Python3 作为主要语言。Linux 下可以直接使用 pip 安装，方便快捷。

```
pip install unicorn
```

更多详细安装过程可以参考

[官方安装教程](https://bbs.kanxue.com/elink@ffbK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4g2F1K9h3y4G2M7X3&6Q4x3X3c8W2L8X3N6A6L8X3g2Q4x3X3g2G2M7X3N6Q4x3V1k6V1L8$3y4K6i4K6u0r3)

# Unicorn 的 Hello World

## 在 python 中导入 Unicorn 库

```
from unicorn import *
```

## 导入处理器相关的常量

Unicorn 支持多种不同的 CPU 指令集，每一种指令集都有自己独立的寄存器, Unicorn 使用统一 API 管理多种不同的 CPU 指令集，并将寄存器名字映射成数字常量。

```
from unicorn.arm_const  import *
from unicorn.arm64_const import *
from unicorn.m68k_const import *
from unicorn.mips_const import *
from unicorn.sparc_const import *
from unicorn.x86_const import *
```

寄存器常量命名规则

`UC_` + 指令集 + `REG` + 大写寄存器名

`UC_ARMREG` + 大写寄存器名（`UC_ARM_REG_R0`）

`UC_X86REG` + 大写寄存器名（`UC_X86_REG_EAX`）

本课程以 python3 + arm 指令集为例子，导入 arm 的常量

```
from unicorn import *
from unicorn.arm_const import *
```

## 模拟执行的代码

为了简单起见，我们直接将要执行代码的数据硬编码。

```
ARM_CODE   = b"\x37\x00\xa0\xe3\x03\x10\x42\xe0" 
# mov r0, #0x37; 
# sub r1, r2, r3
```

## 创建一个 Uc 对象并设置异常处理

```
# Test ARM
def test_arm():
    print("Emulate ARM code")
    try:
        # Initialize emulator in ARM mode
        mu = Uc(UC_ARCH_ARM, UC_MODE_ARM)
        # 其它代码添加到此处
    except UcError as e:
        print("ERROR: %s" % e)
```

`Uc` 是 unicorn 的主类，`Uc` 对象则代表了一个独立的虚拟机实例，它有独立的寄存器和内存等资源，不同 `Uc` 对象之间的数据是独立的。`Uc` 的构造函数有两个参数 `arch` 和 `mode`，用来指定模拟执行的指令集和对应的位数或模式。

`arch` 常量参数一般以 `UC_ARCH_` 开头，`MODE` 常量以 `UC_MODE_` 开头。

同一种指令集可以有多种模式，比如 x86 可以同时运行 32 位和 16 位的汇编，arm 也有 arm 模式和 Thumb 模式，它们是向下兼容的，并可以通过特殊指令来切换 CPU 运行模式。调用构造函数时的模式（mode）以第一条执行指令的模式为准。

## 映射内存

想用 Unicorn 模拟执行代码，是不能将代码字节流直接以参数形式传递给 Unicorn，而是将要执行的代码写入到 Unicorn 的虚拟内存中。`Uc` 虚拟机实例初始内存是没有任何映射的，在读写内存之前使用 `uc_mem_map` 函数映射一段内存。

```
# map 2MB memory for this emulation
ADDRESS = 0x10000
mu.mem_map(ADDRESS, 2 * 0x10000)
```

这段代码在内存地址 `0x10000` 处映射了一段大小为 2M 的内存。`mem_map` 函数特别娇气，要求 `address` 和 `size` 参数都与 `0x1000` 对齐,否则会报 `UC_ERR_ARG` 异常。

## 写入代码

我们要执行代码，就需要将欲执行代码的字节数据写入到虚拟机内存中。

```
mu.mem_write(ADDRESS, ARM_CODE)
```

`mem_write` 的第二个参数也很娇气，只支持 python 的 byte 数组，不能是 String 或者 bytearray。

## 给寄存器赋值

```
mu.reg_write(UC_ARM_REG_R0, 0x1234)
mu.reg_write(UC_ARM_REG_R2, 0x6789)
mu.reg_write(UC_ARM_REG_R3, 0x3333)
```

## 添加指令级的 Hook

这个有点像单步调试的感觉。

```
mu.hook_add(UC_HOOK_CODE, hook_code, begin=ADDRESS, end=ADDRESS)
```

在 `begin...end` 范围内的每一条指令被执行前都会调用 `callback`。

让我们来看看 `hook_code` 的实现吧

```
# callback for tracing instructions
def hook_code(uc, address, size, user_data):
    print(">>> Tracing instruction at 0x%x, instruction size = 0x%x" %(address, size))
```

这段代码仅打印指令执行的地址和长度信息。实际应用中可配合 capstone 反汇编引擎玩一些更骚的操作。

`UC_HOOK_CODE` 的 `callback` 中可以修改 PC 或 EIP 等寄存器力来改变程序运行流程。实际上，unicorn 调试器的单步调试就是以这个为基础实现的。

## 开机

原谅我用开机这个词汇吧！我们已经映射内存并将数据写入到内存，并设置好执行 Hook 以监视指令是否正常执行，但是虚拟机还没有启动！

```
# emulate machine code in infinite time
mu.emu_start(ADDRESS, ADDRESS + len(ARM_CODE))
```

`emu_start` 可以通过 `timeout` 参数设置最长执行时长，防止线程死在虚拟机里面。原型如下

```
def emu_start(self, begin, until, timeout=0, count=0):
    pass
```

`emu_start` 执行完成后，可以通过读取内存或寄存器的方式来获取执行结果。

## 获取结果

```
r0 = mu.reg_read(UC_ARM_REG_R0)
r1 = mu.reg_read(UC_ARM_REG_R1)
print(">>> R0 = 0x%x" % r0)
print(">>> R1 = 0x%x" % r1)
```

## 完整代码

```
from unicorn import *
from unicorn.arm_const import *
ARM_CODE   = b"\x37\x00\xa0\xe3\x03\x10\x42\xe0" 
# mov r0, #0x37; 
# sub r1, r2, r3
# Test ARM
# callback for tracing instructions
def hook_code(uc, address, size, user_data):
    print(">>> Tracing instruction at 0x%x, instruction size = 0x%x" %(address, size))
def test_arm():
    print("Emulate ARM code")
    try:
        # Initialize emulator in ARM mode
        mu = Uc(UC_ARCH_ARM, UC_MODE_THUMB)
        # map 2MB memory for this emulation
        ADDRESS = 0x10000
        mu.mem_map(ADDRESS, 2 * 0x10000)
        mu.mem_write(ADDRESS, ARM_CODE)
        mu.reg_write(UC_ARM_REG_R0, 0x1234)
        mu.reg_write(UC_ARM_REG_R2, 0x6789)
        mu.reg_write(UC_ARM_REG_R3, 0x3333)
        mu.hook_add(UC_HOOK_CODE, hook_code, begin=ADDRESS, end=ADDRESS)
        # emulate machine code in infinite time
        mu.emu_start(ADDRESS, ADDRESS + len(ARM_CODE))
        r0 = mu.reg_read(UC_ARM_REG_R0)
        r1 = mu.reg_read(UC_ARM_REG_R1)
        print(">>> R0 = 0x%x" % r0)
        print(">>> R1 = 0x%x" % r1)
    except UcError as e:
        print("ERROR: %s" % e)
```

## 小结

Unicorn 版的 Hello world 很好展示了 Unicorn 的使用过程。美中不足的是，它过于简陋，没有涉及到栈操作、系统调用、API 调用等，要知道，现在任何一段代码、一个函数都会至少涉及到其中的一项。

# Unicorn 调试器模块编写

## 起始

前一篇文章中，我们学习了如何使用 Unicorn 来模拟执行一段二进制代码。然而目前 Unicorn 对于我们来说，几乎就等同于黑盒子。我们只知输入输出，对于代码中间执行过程的失败与否全然不知，如果出现 BUG 将难以调试。为了解决这种难以调试的状况，我决定写一个 Unicorn 调试器，用来调试 Unicorn 模拟执行的二进制代码。

## 功能 & 特性

- 硬件断点
- 读/写 寄存器信息
- 反汇编
- dump 内存
- 单步调试（步入/步过）
- 快速接入各种不同的 Unicorn 项目

## 如何实现

Unicorn 提供了强大的指令级 Hook（`UC_HOOK_CODE`），使得每一条指令执行前，我们都有机会处理。

`UC_HOOK_CODE` 的 `callback` 原型定义如下

```
typedef void (*uc_cb_hookcode_t)(uc_engine *uc, uint64_t address, uint32_t size, void *user_data);
```

- `address`: 当前执行的指令地址
- `size`: 当前指令的长度，如果长度未知，则为 0
- `user_data`: `hook_add` 设置的 `user_data` 参数

调用 `hook_add` 函数可以为指定的代码范围添加 hook 的 `callback`。

python 包中的 `hook_add` 函数原型如下

```
def hook_add(self, htype, callback, user_data=None, begin=1, end=0, arg1=0):
    pass
```

`UC_HOOK_CODE` 的功能是每条指令执行前调用 `callback`。

`callback` 中，我们可以通过参数得知指令执行地址、指令执行长度、虚拟机指针。

有了虚拟机指针，我们可以很方便的访问各种寄存器、内存等资源。在 `UC_HOOK_CODE` 的 `callback` 中，也可以直接修改 PC 寄存器来改变流程。

调试器要支持的调试指令如下

```
def show_help(self):
    help_info = """
    # commands
    # set reg <regname> <value>
    # set bpt <addr>
    # n[ext]
    # s[etp]
    # r[un]
    # dump <addr> <size>
    # list bpt
    # del bpt <addr>
    # stop
    # f show ins flow
    """
    print (help_info)
```

本文将编写一个 `UnicornDebugger` 类，调试器的各种功能均在该类中实现，调用该类的构造函数即可附加到一个 `Uc` 虚拟机对象上。

类的定义

```
class UnicornDebugger:
    def __init__(self, mu, mode = UDBG_MODE_ALL):
        pass
```

附加调试器

```
mu = Uc(UC_ARCH_ARM, UC_MODE_ARM)
dbg = udbg.UnicornDebugger(mu, udbg.UDBG_MODE_ALL)
dbg.add_bpt(0x1112233)
mu.emu_start(......)
```

可见，这段代码在 `0x1112233` 处添加了一个断点，当程序执行到 `0x1112233` 处的时候就会停下来，陷入调试器的处理，等待调试者的指令。

## 反汇编

Unicorn 并没有反汇编功能，虽然它的内部一定有与反汇编相关的代码。我们只能自己想办法反汇编。Unicorn 有一个兄弟，它叫 Capstone。Capstone 是一款支持多种处理器和开发语言的反汇编框架。[Capstone 官方地址](https://bbs.kanxue.com/elink@36bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4S2M7s2y4@1L8$3&6W2i4K6u0V1k6h3&6Y4K9h3&6W2i4K6u0W2L8%4u0Y4i4K6u0r3) 我将使用 Capstone 作为调试模块的反汇编器。

### 安装 Capstone

Capstone 对 python 的支持特别好，我们的开发语言是 python3，所以直接使用 pip 安装 capstone 即可。

```
pip install capstone
```

### 快速入门 Capstone

Capstone 很强大，也可以很简单，下面一段代码就是 Capstone 的入门例子。
