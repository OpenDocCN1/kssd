
# 用 angr 解 Flare-on 2017 第 3 题 - greek_to_me.exe

## 0x00 说在前面

笔者 angr 入门，一开始目的是用通过 angr 复现原作者 [用 Angr 解决 Flareon4 题目 3](https://bbs.pediy.com/thread-223512.htm) 的 exp 来学习 angr 的，但是现在 angr 已经更新与到 python3 兼容 (windows 下在 python2 下面 angr 我没装成功，爆了一堆错)，因此笔者想根据原作者代码改出 python3 下 angr 的 exp，进而学习 angr 的使用情景及使用方法，因此 **除特殊说明，本文的 python 代码是 python3 的**

## 0x01 题解

### 一、题目分析

![](upload/attach/202106/914317_9VECC2AKK2566S4.jpg)

题目主函数如上图所示，一开始是个 socket 函数 `sub_401121()`，如下图，之后是一个对 `loc_40107C` (代码段) 的混淆，再之后是被混淆的代码段

![](upload/attach/202106/914317_46YUE9JFB2VD9PB.jpg)

可以看出是建立本地的 TCP 连接并接受四个字节的数据，由主函数可以看出，如果没收到这四个字节，则直接退出程序

好在我们可以通过 angr 的 `blank_entry` 指定程序入口点绕过检测，前面的混淆函数容易自己模拟。那从哪里设入口呢？这需要确定混淆代码段的起始位置，根据前面反复出现的 `loc_40107C` 可以看出开始位置，又根据提示成功的字符串位置，可以找到混淆结束点为 `004010F9`，我们还可以找到 `0x40105E` 有比较检查，必须匹配 `0xFB5E` 才能成功进入验证分支

![](upload/attach/202106/914317_JGS7EJDC4S4PPJA.jpg)

![](upload/attach/202106/914317_VFC96WV7WWP7K95.jpg)

![](upload/attach/202106/914317_4SHD9P6ZSGW5MEU.jpg)

总结一下收获:

- 找到了位于 `0x4010FE` 的成功提示字符串
- 比较检查在 `0x40105E` 处
- 必须匹配 `0xFB5E` 才能进入成功验证分支
- 已混淆代码起始于 `0x40107C`
- 已混淆代码的长度为 `0x79`
- 第 2 阶段的解混淆函数在 `0x401054` 处被调用
- 第 1 阶段的解混淆操作从 `0x401029` 开始到 `0x401045`
- 用户输入发生在 `0x401015` 处的函数调用
- 用户输入来自网络
- 输入缓冲区的长度是 `0x4`

### 二、题解

#### 1、生成二进制文件

将混淆的字节提取出来，使用 IDA Python (见补充知识)

```python
with open('greek_to_me_buffer.asm', 'wb') as f:
f.write(ida_bytes.get_bytes(0x40107C, 0x79))
```

#### 2、赋值给 buf 并暴力枚举

> 我们知道从网络接收的长度是 4 字节, 但是聪明的读者可能已经注意到代码中使用 `dl` 赋值给 `buf` 而非 `edx`, 也就导致实际值的范围是从 `0x0` 到 `0xff` (`dl` 只有 1 字节大小). 我们脚本的开始部分类似如下:
>
> ```python
> for buf in range(0x100):
> print("Using {0}".format(buf))
> ```

#### 3、解混淆并做合法性检查

第一阶段的混淆写一个模拟，第二阶段的混淆暴力枚举，注意把子串存起来才能跑(有点像多线程共用空间)(注意 `state` 方法在 `manager` 下面已经没有了)，我们使用 `capstone` 反汇编代码解混淆结果下图：

```
0x1000 mov bl, 0x65 None
0x1002 mov byte ptr [ebp - 0x2b], bl
0x1005 mov byte ptr [ebp - 0x2a], 0x74
0x1009 mov dl, 0x5f None
0x100b mov byte ptr [ebp - 0x29], dl
0x100e mov byte ptr [ebp - 0x28], 0x74
0x1012 mov byte ptr [ebp - 0x27], 0x75
0x1016 mov byte ptr [ebp - 0x26], dl
0x1019 mov byte ptr [ebp - 0x25], 0x62
0x101d mov byte ptr [ebp - 0x24], 0x72
0x1021 mov byte ptr [ebp - 0x23], 0x75
0x1025 mov byte ptr [ebp - 0x22], 0x74
0x1029 mov byte ptr [ebp - 0x21], bl
0x102c mov byte ptr [ebp - 0x20], dl
0x102f mov byte ptr [ebp - 0x1f], 0x66
0x1033 mov byte ptr [ebp - 0x1e], 0x6f
0x1037 mov byte ptr [ebp - 0x1d], 0x72
0x103b mov byte ptr [ebp - 0x1c], 0x63
0x103f mov byte ptr [ebp - 0x1b], bl
0x1042 mov byte ptr [ebp - 0x1a], 0x40
0x1046 mov byte ptr [ebp - 0x19], 0x66
0x104a mov byte ptr [ebp - 0x18], 0x6c
0x104e mov byte ptr [ebp - 0x17], 0x61
0x1052 mov byte ptr [ebp - 0x16], 0x72
0x1056 mov byte ptr [ebp - 0x15], bl
0x1059 mov byte ptr [ebp - 0x14], 0x2d
0x105d mov byte ptr [ebp - 0x13], 0x6f
0x1061 mov byte ptr [ebp - 0x12], 0x6e
0x1065 mov byte ptr [ebp - 0x11], 0x2e
0x1069 mov byte ptr [ebp - 0x10], 0x63
0x106d mov byte ptr [ebp - 0xf], 0x6f
0x1071 mov byte ptr [ebp - 0xe], 0x6d
0x1075 mov byte ptr [ebp - 0xd], 0
```

可以看到是向数组里面传一堆字符，写一个脚本，flag 就出来了

### exp(python3)

```python
#!/usr/bin/env python3
import angr
import sys
import binascii
p = angr.Project('greek_to_me.exe', load_options={'auto_load_libs': False})
f2 = None
# Interate through all of the possible byte values to find the correct "user" input to de-mask the flag
for buf in range(0x100):
print(("Trying buf = {0}".format(buf)))
# Variable to store the bits written to disk using IDA
asm = None
# Store the output from the first de-obfuscation routine
b2 = []
# Read in bytes written to file from IDA
with open('greek_to_me_buffer.asm', 'rb') as f:
asm = f.read()
# Re-implement loc_401039
dl = buf
for byte in asm:
#bl = ord(byte)
bl = byte
bl = bl ^ dl
bl = bl & 0xff
bl = bl + 0x22
bl = bl & 0xff
b2.append(bl)
# Set up angr to "run" sub_4011E6
s = p.factory.blank_state(addr=0x4011E6)
s.mem[s.regs.esp+4:].dword = 1 # Angr memory location to hold the xor'ed and add'ed bytes
s.mem[s.regs.esp+8:].dword = 0x79 # Length of ASM
# Copy bytes output from loc_401039 into address 0x1 so Angr can run it
#asm = ''.join(map(lambda x: chr(x), b2))
asm = bytes(b2)
s.memory.store(1, s.se.BVV(int.from_bytes(asm,byteorder='big',signed=False), 0x79 * 8 ))
# Create a simulation manager...
#import pdb; pdb.set_trace()
simgr = p.factory.simulation_manager(s)
# Tell Angr where to go, though there is only one way through this function,
# we just need to stop after ax is set
simgr.explore(find=0x401268)
# Once ax is set, check to see if the value in ax matches the comparison value
for found in simgr.found:
#import pdb; pdb.set_trace()
print((' ax = %s' % hex(found.solver.eval(found.regs.ax))))
# Comparison check
if hex(found.solver.eval(found.regs.ax)) == '0xfb5e':
# Upon success, dump the asm
code = ("%x" % found.solver.eval_upto(found.memory.load(1, 0x79), 1)[0])
print(('\n Winner is: {0}\n\n'.format(buf)))
print((' %s' % code))
bl = None
dl = None
flag = []
# Using capstone, interpret the ASM
from capstone import *
md = Cs(CS_ARCH_X86, CS_MODE_32)
code = binascii.unhexlify(code)
for i in md.disasm(code, 0x1000):
flag_char = None
#print(i.op_str)
# The if statements do the work of interpreting the ASCII codes to their value counterpart
try:
if i.op_str.split(',')[0].startswith("byte ptr"):
flag_char = chr(int(i.op_str.split(',')[1], 16))
if i.op_str.split(',')[0].startswith('bl'):
bl = chr(int(i.op_str.split(',')[1], 16))
if i.op_str.split(',')[0].startswith('dl'):
dl = chr(int(i.op_str.split(',')[1], 16))
except:
if i.op_str.split(',')[1].strip() == 'dl':
flag_char = dl
if i.op_str.split(',')[1].strip() == 'bl':
flag_char = bl
if (flag_char):
flag.append(flag_char.strip())
print((" 0x%x\t%s\t%s\t%s" %(i.address, i.mnemonic, i.op_str, flag_char)))
print('\n\n')
print((''.join(flag)))
sys.exit(0)
```

跑程序时间：秒表计时 2'11' (我的电脑配置不是很高)

运行结果：

![](upload/attach/202106/914317_Y5VGW3K4JNB4CE8.jpg)

## 0X02 补充知识

### 一、WIN socket 有关函数 (sub_401121 相关)

![](upload/attach/202106/914317_38QXFH2MYM55GQM.jpg)

#### 1、WSAStartup() (WSACleanup() 函数相反)

**WSAStartup** 函数用于初始化供进程调用的 Winsock 相关的 dll，如果调用成功，**WSAStartup** 函数返回 0。否则，将返回五种错误代码之一 `WSAVERNOTSUPPORTED`、`WSAVERNOTSUPPORTED`、`WSAEINPROGRESS`、`WSAEPROCLIM`、`WSAEFAULT`

这里用通俗的语言解释一下这个函数，就类似于 #include 一样，要添加链接库函数，要添加到附加依赖项，然后才能包含头文件进行各种函数的调用。socket 编程要调用各种 socket 函数需要 `Ws2_32.lib` 和头文件 `Winsock2.h`，这里的 **WSAStartup** 就是为了向操作系统说明，我们要用哪个库文件，让该库文件与当前的应用程序绑定，从而就可以调用该版本的 socket 的各种函数了。

> ## Syntax
>
> ```c++
> int WSAAPI WSAStartup(
> WORD wVersionRequested,
> LPWSADATA lpWSAData
> );
> ```
>
> ## Parameters
>
> ```
> wVersionRequested
> ```
>
> The highest version of Windows Sockets specification that the caller can use. The high-order byte specifies the minor version number; the low-order byte specifies the major version number.
>
> ```
> lpWSAData
> ```
>
> A pointer to the `WSADATA` data structure that is to receive details of the Windows Sockets implementation.

在 Windows 下，Socket 是以 DLL 的形式实现的。在 DLL 内部维持着一个计数器，只有第一次调用 **WSAStartup** 才真正装载 DLL，以后的 调用只是简单的增加计数器，而 **WSACleanup** 函数的功能则刚好相反，每调用一次使计数器减 1，当计数器减到 0 时，DLL 就从内存中被卸载！因此，你 调用了多少次 **WSAStartup**，就应相应的调用多少次的 **WSACleanup**.

#### 2、inet_addr()

`inet_addr` 将一个点分十进制的 IP 转换成一个长整数型数 (`u_long`)

#### 3、socket()

`socket` 函数对应于普通文件的打开操作。普通文件的打开操作返回一个文件描述字，而 **socket()** 用于创建一个 socket 描述符 (`socket descriptor`)，它唯一标识一个 socket。这个 socket 描述字跟文件描述字一样，后续的操作都有用到它，把它作为参数，通过它来进行一些读写操作。

正如可以给 `fopen` 的传入不同参数值，以打开不同的文件。创建 socket 的时候，也可以指定不同的参数创建不同的 socket 描述符，`socket` 函数的三个参数分别为：

- `domain`：即协议域，又称为协议族 (`family`)。常用的协议族有，`AF_INET`、`AF_INET6`、`AF_LOCAL` (或称 `AF_UNIX`，Unix 域 socket)、`AF_ROUTE` 等等。协议族决定了 socket 的地址类型，在通信中必须采用对应的地址，如 `AF_INET` 决定了要用 ipv4 地址 (32 位的) 与端口号 (16 位的) 的组合、`AF_UNIX` 决定了要用一个绝对路径名作为地址。

| Af | Meaning |
