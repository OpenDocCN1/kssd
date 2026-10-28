
```python
import struct
def read(name):
with open(name) as f:
return f.read()
def u32(data):
return struct.unpack("I", data)[0]
def p32(num):
return struct.pack("I", num)
```

这里，我只添加了一些常用的在之后会用到的一些函数

`read`仅仅返回文件中的内容

`u32`将4字节的string（译注：python2中的string等同于bytes array）转换为integer，以小端序表示这个数据。

`p32`与`u32`相反，将一个数字转换为以小端序保存的4字节string.

如果你安装了pwntools，可以不创建这两个函数，直接`from pwn import *`就可以了。

接下来为x86-64架构初始化一下Unicorn引擎。

`mu = Uc(UC_ARCH_X86, UC_MODE_64)`

`Uc`函数需要一下参数：

- 第一个参数：架构类型。这些常量以`UC_ARCH_`为前缀
- 第二个参数：架构细节说明。这些常量以`UC_MODE_`为前缀

你可以在备忘录中找到这些常量。

正如我之前所写，使用Unicorn引擎之前需要手动初始化内存。针对这个二进制文件来说，我们需要将代码写入到某个地方，并且分配一些栈空间。

二进制文件的基址是`0x400000`。栈的话不妨从地址`0x0`开始，大小为`1024*1024`字节（译注：即1M）。或许并不需要这么大的栈空间，但是也没有多大的影响。

可以使用`mem_map`函数来映射内存。

使用如下代码：

```python
BASE = 0x400000
STACK_ADDR = 0x0
STACK_SIZE = 1024*1024
mu.mem_map(BASE, 1024*1024)
mu.mem_map(STACK_ADDR, STACK_SIZE)
```

此时，我们可以和加载器一样，加载二进制文件到准备好的基址上来了。然后需要设置`RSP`指向我们申请的栈空间底部。

```python
mu.mem_write(BASE, read("./fibonacci"))
mu.reg_write(UC_X86_REG_RSP, STACK_ADDR + STACK_SIZE - 1)
```

可以开始模拟和执行代码了，但是需要先知道从哪开始从哪结束。

我们可以从地址`0x00000000004004E0` 开始执行代码，这是`main`函数开始的地方。结尾的话可以是`0x0000000000400575`。这是`puts("\n")`函数所在，在Flag输出之后被调用。汇编代码如下：

```assembly
.text:0x400570 mov edi, 0Ah ; c
.text:0x400575 call __IO_putc
```

可以开始执行模拟了：

```assembly
mu.emu_start(0x00000000004004E0, 0x0000000000400575)
```

此时，可以执行这个脚本：

```python
a@x:~/Desktop/unicorn_engine_lessons$ python solve.py
Traceback (most recent call last):
File "solve.py", line 32, in <module>
mu.emu_start(0x00000000004004E0, 0x0000000000400575)
File "/usr/local/lib/python2.7/dist-packages/unicorn/unicorn.py", line 288, in emu_start
raise UcError(status)
unicorn.unicorn.UcError: Invalid memory read (UC_ERR_READ_UNMAPPED)
```

oooooops，发生了一些我们不知道的错误呀。在`mu.emu_start`之前我们可以加入：

```python
def hook_code(mu, address, size, user_data):
print('>>> Tracing instruction at 0x%x, instruction size = 0x%x' %(address, size))
mu.hook_add(UC_HOOK_CODE, hook_code)
```

这几行代码加入了一个hook。我们自定的函数`hook_code`在执行每一条代码模拟的时候都将被调用。参数如下：

- `Uc`实例句柄
- 指令的地址
- 执行的长度
- 用户自定义数据（我们可以在`hook_add`的可选参数中传递这个值）

此时，我们的脚本应该是这样子的 solve1.py

运行的时候就可以有以下的输出了：

```python
a@x:~/Desktop/unicorn_engine_lessons$ python solve.py
>>> Tracing instruction at 0x4004e0, instruction size = 0x1
>>> Tracing instruction at 0x4004e1, instruction size = 0x1
>>> Tracing instruction at 0x4004e2, instruction size = 0x2
>>> Tracing instruction at 0x4004e4, instruction size = 0x5
>>> Tracing instruction at 0x4004e9, instruction size = 0x2
>>> Tracing instruction at 0x4004eb, instruction size = 0x4
>>> Tracing instruction at 0x4004ef, instruction size = 0x7
Traceback (most recent call last):
File "solve.py", line 41, in <module>
mu.emu_start(0x00000000004004E0, 0x0000000000400575)
File "/usr/local/lib/python2.7/dist-packages/unicorn/unicorn.py", line 288, in emu_start
raise UcError(status)
unicorn.unicorn.UcError: Invalid memory read (UC_ERR_READ_UNMAPPED)
```

这些输出表明，脚本执行以下的指令的时候发生了错误：

```ass
.text:0x4004EF mov rdi, cs:stdout ; stream
```

这条指令从地址`0x601038`处读取内存（你可以在IDA Pro中看）。这是`.bss`区段所在，然而我们并没有来分配这个区段。我的解决方案是跳过所有出现问题的指令。

接下来有一条指令：

```assembly
.text:0x4004F6 call _setbuf
```

我们没有办法来调用glibc里的函数，因为没有在被模拟的对象的内存中加载glibc。其实我们并不需要调用这个函数，所以直接跳过去就好了。

以下是直接跳过的指令：

```assembly
.text:0x4004EF mov rdi, cs:stdout ; stream
.text:0x4004F6 call _setbuf
.text:0x400502 call _printf
.text:0x40054F mov rsi, cs:stdout ; fp
```

可以通过修改`RIP`寄存器的方式来跳过这些指令。

```pytho
mu.reg_write(UC_X86_REG_RIP, address+size)
```

`hook_code`函数现在看起来是这样的：

```pyth
instructions_skip_list = [0x00000000004004EF, 0x00000000004004F6, 0x0000000000400502, 0x000000000040054F]
def hook_code(mu, address, size, user_data):
print('>>> Tracing instruction at 0x%x, instruction size = 0x%x' %(address, size))
if address in instructions_skip_list:
mu.reg_write(UC_X86_REG_RIP, address+size)
```

同样的针对一个字节一个字节输出Flag的代码也需要做一些处理：

```ass
.text:0x400558 movsx edi, dil ; c
.text:0x40055C add rbp, 1
.text:0x400560 call __IO_putc
```

`__IO_putc`将一个字节输出到`第一个参数`所在（`RDI`寄存器）

这里就可以直接读取`RDI`寄存器，然后输出，跳过模拟执行这部分代码。这里的`hook_code`函数如下：

```python
instructions_skip_list = [0x00000000004004EF, 0x00000000004004F6, 0x0000000000400502, 0x000000000040054F]
def hook_code(mu, address, size, user_data):
#print('>>> Tracing instruction at 0x%x, instruction size = 0x%x' %(address, size))
if address in instructions_skip_list:
mu.reg_write(UC_X86_REG_RIP, address+size)
elif address == 0x400560: #that instruction writes a byte of the flag
c = mu.reg_read(UC_X86_REG_RDI)
print(chr(c))
mu.reg_write(UC_X86_REG_RIP, address+size)
```

此时，整个代码看起来应该是这样的solve2.py.

可以运行以下看一下，尽管它依然很慢。

```python
a@x:~/Desktop/unicorn_engine_lessons$ python solve.py
h
x
```

### Part2：提升速度！

考虑以下速度提升，为什么这个程序跑这么慢？

观察一下反编译代码，我们可以看到`main`函数调用了`fibonacci`函数若干次，并且`fibonacci`函数是个递归函数。

看看这个函数，我们可以看到它有2个参数，返回2个值。第一个返回值存放在`RAX`寄存器中，第二个是第二个参数的指针。深入研究一下`main`和`fibonacci`函数可以发现，第二个参数只能取值为`0`或者`1`。如果发现不了的话，可以运行`gdb`然后在`fibonacci`函数的起始地址下个断点观察。

为了优化这个函数，我们可以使用动态编程来记住所给参数的返回值。因为第二个参数只有两种取值，只需记住只记住`2 * MAX_OF_FIRST_ARGUMENT`对就足够了。

可以在`RIP`指向`fibonacci`函数开始的时候获取函数的参数，值的返回在函数退出的时候。因为不能同时获取这两个值，所以我们需要一个栈来帮助我们在函数退出的时候获取这些值-在`fibonacci`入口我们需要将参数入栈，最后的时候出栈。可以使用`dict`来记住这些值。

如何保存这些成对的值？

- 在函数开始的时候，我们可以检查参数对应的值是否已经被dict记录
- 如果是，直接返回这个key-value就行，只需将返回值写入到`RAX`中，同时设置`RIP`为`RET`指令的值，退出这个函数。不能在`fabonacci`函数内直接跳转到`RET`，因为这条指令已经被HOOK了，所以我们跳转到`main`中的`ret`。
- 如果dict中没有出现参数和对应的值，将参数添加到dict中。
- 当退出函数的时候，保存返回值。可以从我们的栈结构中读取参数和返回值。

代码如下：
