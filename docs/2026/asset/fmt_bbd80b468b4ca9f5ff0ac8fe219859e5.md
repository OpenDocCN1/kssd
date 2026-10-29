
# [翻译]编写Shellcode：寻找`EIP`/`RIP`

当学习<a href="elink@ca7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2G2k6X3k6W2L8Y4y4A6N6X3g2Q4x3X3c8K6k6h3y4#2M7X3W2@1P5g2)9J5k6h3y4G2L8g2)9J5c8X3y4@1M7q4)9J5k6r3!0K6j5$3g2Q4x3V1j5`.">主动安全认证专家（OSCE）</a>认证时，我花费了大量的时间用于研究如何编写自定义shellcode。本文是我计划要发布的一系列博文里的第一篇，这个系列详细介绍我在准备认证过程中所学习到的技术。本文的重点是描述如何找到`EIP`/`RIP`以及找到后做什么。OSCE专注于32位系统，作为继续学习的一部分，我会研究并记录下在64位系统上的方法。这超出了OSCE的需求，但这是我继续学习过程的一部分。

在本文中，我不会介绍到所有的可能找到`EIP`和`RIP`的方法。我会介绍一些我熟悉并已经研究过的方法。如果你想找到更多的方法，我鼓励你开始你自己的研究并开始做自己的学习总结。在这里你看到的是我的学习总结，我希望它会在某些方面对你有帮助。我知道这些主题多年来已经被多次提到，也没有新的东西，但重点不是提出新的东西，而是在学习总结中提升自己，并且还可以帮助到刚开始学习的人。

## 寻找`EIP`：32位汇编方法

从使用x86汇编（32位）指令定位`EIP`的方法开始。有两种方法可以用，根据你可能会遇到的情况和限制，两种方法可能都有用。两种方法都可以找到`EIP`。我要介绍的第一种方法比第二种方法小一个字节。两种方法都可以达到完全相同的目的，存储`EIP`的值到`EAX`寄存器中。两种方法都没有空（`0x00`）字节。之所以使用一种而不是另一种，是因为您可能会遇到大小或字符限制。

### 方法1：使用`FPU`指令

该方法最先由Aaron Adams在漏洞开发（Vulnerability Development）<a href="elink@f34K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3y4D9K9i4y4@1M7#2)9J5k6h3!0J5k6#2)9J5c8Y4k6#2L8r3&6Q4x3X3c8V1k6i4k6Q4x3V1j5J5x3o6l9K6i4K6u0r3e0X3!0$3i4K6u0r3y4o6b7`.">邮件列表</a>中提出.他使用的这种方法利用x87浮点单元（`FPU`）寄存器来获取`EIP`值。以下的基础汇编代码将`EIP`的值存储到`EAX`寄存器中。

```c
[SECTION .text]
BITS 32
global _start
_start:
fldz
fnstenv [esp-0x0C]
pop eax
add al, 0x07
```

执行`fldz`指令来激活`FPU`寄存器。该指令将常数值+0.0压入`FPU`寄存器栈（`ST(0)`）。在这个过程中，`FPU`寄存器被初始化，当前的`EIP`值被存储到`FPU`指令指针偏移（`FIP`）寄存器中。**图 1**描述了`FPU`寄存器的结构。该表拷贝自《<a href="elink@02aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2A6L8Y4c8W2L8q4)9J5k6h3y4G2L8g2)9J5c8X3y4G2L8Y4c8W2L8Y4c8Q4x3V1k6%4N6%4N6Q4x3V1k6#2M7#2)9J5c8X3g2F1i4K6u0r3j5i4u0U0K9r3W2@1k6h3y4@1N6i4u0W2i4K6u0V1j5h3&6V1i4K6u0V1N6r3g2U0K9r3&6G2L8r3!0Y4P5g2)9J5c8U0j5@1i4K6u0V1K9h3q4Q4x3X3b7K6x3W2)9J5k6r3q4J5j5$3S2A6N6r3g2U0N6s2g2J5k6i4y4Q4x3X3c8K6L8$3k6@1N6$3q4J5k6g2)9J5k6r3c8W2N6X3g2D9L8%4m8W2M7W2)9J5k6s2k6G2L8q4)9J5k6o6q4Q4x3X3c8E0j5h3&6#2j5h3I4Q4x3X3g2Z5N6r3#2D9">Intel(r) 64 和 IA-32 架构软件开发者手册，卷1：基础架构</a>PDF》.

![图片描述](upload/attach/202001/703263_W8XTZUNSXFFTBSY.png)

*图 1：内存中保护模式x87 `FPU`状态图片(32位格式)*

下一条指令存储`FPU`寄存器在特殊的地址。因此使用`fnstenv`指令将`EIP`的值存储在偏移`0x0C`（12）的`FIP`寄存器中。`ESP-0x0C`的目标是固定的，所以`FIP`的值会存储在当前`ESP`地址处。接下来，该值会从栈中弹出，并存储到`EAX`寄存器中。由于在将`EIP`存储到`FPU`的`FIP`寄存器和弹到`EAX`的这段时间已经执行了几个字节的指令，所以需要调整该值来表示当前的`EIP`的值。用`AL`寄存器加`0x7`（7）完成此操作。`AL`寄存器被用来避免使用空字节。

#### 测试

要测试可以使用`nasm`来汇编代码：

```s
nasm method1.asm -o method1.asm
```

随后插入字节到`C`中：

```c
char code[] = "\xD9\xEE\xD9\x74\x24\xF4\x58\x04\x07";
int main(int argc, char **argv)
{
int (*func)();
func = (int (*)()) code;
(int)(*func)();
}
```

使用MingW来编译代码：

```shell
i686-w64-mingw32-gcc-win32 method1.c -o method1.exe -fno-stack-protector -no-pie -m32
```

运行最终的PE文件在你喜欢的调试器中，并看它是如何运行的。你需要将`.data`节标记为可执行。你可以使用调试器来实现或者使用如LordPE这样的工具。如果你不标记`.data`节为可执行，你会在执行你的shellcode时发生访问违例。我需要在PE文件中搜索`call eax`指令，并设置断点，第一个调用到`EAX`的会是shellcode。如果一切顺利，`EAX`会在执行`add al, 7`指令时指向`EIP`，如**图 2**。

![图片描述](upload/attach/202001/703263_NP57EZK22PBDK85.png)

*图 2：`EAX`指向`EIP`*

#### 方法1的替代——使用减法

感谢来自<a href="elink@50dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3g2r3S2W2b7$3!0D9L8$3&6A6j5h3H3`.">@TheColonial</a>的建议，对`EAX`寄存器使用减法而不是`add al`指令可以在一些情况下避免错误，我添加了该替代方法。他正确的指出的问题，是通过对`AL`进行加法运算，如果加法导致了进位，`EAX`就会指向错误的地址。例如，如果`AL`寄存器包含大于或等于`0xF9`的任何数值，加上`0x07`后，进位的1会被丢弃掉。例如，如果`EAX`是`0x001234F9`，我们把`0x07`加到`AL`寄存器（`0xF9`）后，结果会是`0x00123400`而不是我们需要的`0x00123500`。

为了避免这个问题，并且避免空字节，可以让`EAX`减去一个负值。基本的数学运算，减去一个负值等于加上它（的绝对值）。简单却有效。正确的代码如下，还避开了空字节，并且只比之前大了1字节：

```c
[SECTION .text]
BITS 32
global _start
_start:
fldz
fnstenv [esp-0x0C]
pop eax
sub eax, -0x07
```

在**图 9**中我们可以看出建议的代码正常运行并且会比原来的代码更加可靠。

![图片描述](upload/attach/202001/703263_GK8AYSCZHUB6GY6.png)

*图 9 `EAX`指向`EIP`*

### 方法2：使用跳转和调用

第二种方法仅比第一种方法长一个字节，也完成了将`EIP`的值存储到`EAX`中的目标。我从<a href="elink@018K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8Z5M7X3q4U0K9#2)9J5k6h3!0J5k6#2)9J5c8X3W2K6M7%4g2W2M7#2)9J5c8U0j5J5i4K6u0r3y4#2)9J5k6h3S2@1L8h3I4Q4x3U0y4S2M7Y4c8A6j5$3I4W2">Phrack issue 62,Phile 7</a>标题为*History and Advances in Windows Shellcode*中学到的这种方法。这篇文章由SK Chong所写。为了获得`EIP`这种方法使用了一系列的跳转和调用，最终`EIP`的值被存储在`EAX`中。SK Chong文章的原始文件已经找不到了，但是，我找到了其中的一些。在他的例子中，它使用了`db`项来硬编码了那些跳转和调用。我要提供的版本读起来更容易并且以及更容易使用`nasm`汇编。

```c
[SECTION .text]
BITS 32
global _start
_start:
jmp label2
label1:
jmp getEIP
label2:
call label1
getEIP:
pop eax
```

上面的代码执行了跳转到`label2`，一个`call`指令调用了`label1`。当`call`被执行时，返回地址，即下一条指令的地址，被压入栈中。执行，随后跳转到`getEIP`，会弹出返回地址到`EAX`。不要小题大作（No muss, no fuss.）。

#### 测试

你可以使用测试方法1的测试方法来测试方法2.当执行完，结果看起来如**图 3**。

![图片描述](upload/attach/202001/703263_Z2S2H3YQMHWSTWR.png)

*图 3：`EAX`指向`EIP`*

## 寻找`RIP`：64位汇编方法

查看是否32位架构中运行的方法在64位世界中是否运行是一个好的开始。如果只需做些微修改，为什么还要重新造轮子？

### 方法1：使用`FPU`指令

#### 尝试1

使用`FPU`指令可以运行，因为指令在64位下仍然生效。首先，找出未对32位模式运行下的指令进行修改或微小修改发生的情况。要做到这点，我使用`x64dbg`并使其执行`notepad.exe`，运行直到命中`EntryPoint`。使用`Assemble`指令，我手动替换了开始的一些指令为32-bit的指令。这样做的结果如**图 4**。这里有一些需要注意。首先，`67`：出现在`fnstenv`指令前。根据<a href="elink@472K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2A6L8Y4c8W2L8q4)9J5k6h3y4G2L8g2)9J5c8X3y4G2L8Y4c8W2L8Y4c8Q4x3V1k6V1j5h3#2Q4x3V1k6%4N6%4N6Q4x3V1k6H3N6h3u0D9K9h3y4Q4x3V1k6#2M7#2)9J5c8X3g2F1i4K6u0r3k6r3!0U0N6h3#2W2L8Y4c8K6i4K6u0r3L8h3q4F1N6h3q4D9M7#2)9J5c8U0j5@1i4K6u0V1K9h3q4Q4x3X3b7K6x3W2)9J5k6r3q4J5j5$3S2A6N6r3g2U0N6s2g2J5k6i4y4Q4x3X3c8K6L8$3k6@1N6$3q4J5k6g2)9J5k6r3c8W2N6X3g2D9L8%4m8W2M7W2)9J5k6r3W2F1M7%4c8J5N6h3y4@1K9h3!0F1i4K6u0V1M7$3g2@1i4K6u0V1M7X3g2X3k6i4u0W2L8X3y4W2i4K6u0V1L8h3q4F1N6h3q4D9i4K6u0V1x3K6t1#2x3K6R3K6i4K6u0W2M7r3c8X3">Intel® 64 and IA-32 Architectures Software Developer’s Manual Volume 2 (2A, 2B, 2C & 2D): Instruction Set Reference, A-Z </a>章节2.1.1，`67h`前缀是地址大小覆写。这一定会出现的，因为我之前使用的是`ESP`寄存器而不是`RSP`。第二，我不能使用`POP EAX`指令，我需要使用`POP RAX`。

![图片描述](upload/attach/202001/703263_FA3FH4G85S2YY2V.png)

*图 4：使用`FPU`寻找`RIP`，尝试1*

运行该序列会部分成功（见**图5**）。当加法执行时，`RAX`指向`RIP`之前的指令。这是必须使用`67h`地址大小前缀的原因。使用`0x08`替代`0x07`就能很容易的使之平衡。继续，自己尝试下，看看它是否有效。

![图片描述](upload/attach/202001/703263_UKYK6GX8A6EY5MY.png)

*图 5：使用`FPU`指令寻找`RIP`，尝试1的结果*

#### 编写汇编

既然已经证明了使用`FPU`寄存器来获取`RIP`的值是可行的。我的下一步是看看是否能够写一些汇编代码，可以在调试器中手工输入命令可以使之平衡。经过一些调整后，这是我想出的代码：

```c
[SECTION .text]
BITS 64
global _start
_start:
fldz
fnstenv [rsp-0x0C]
pop rax
add rax, 0x07
```

当使用`Nasm`汇编时，上面指令中的代码结果如**图 6**。为了测试那些指令，我对用`x64dbg`打开的64位进程的第一条指令做二进制粘贴。如你所看到的，在`add rax, 0x07`指令前有了新的`48h`前缀。根据<a href="elink@34aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8$3k6@1N6$3q4J5k6g2)9J5k6h3W2F1N6r3g2D9i4K6u0W2j5$3!0E0i4K6u0r3M7$3W2@1k6i4y4Q4x3V1k6V1k6h3k6S2N6h3I4@1i4K6u0r3k6X3W2D9k6i4y4Q4x3V1k6E0j5h3&6S2k6$3g2V1i4K6u0r3x3K6W2Q4x3V1k6U0y4g2)9J5c8U0x3J5y4e0b7$3x3W2)9J5k6s2y4V1L8g2)9J5k6s2k6G2L8q4)9J5k6o6q4Q4x3X3b7J5j5h3u0U0k6q4)9J5k6o6y4S2j5X3y4V1i4K6u0W2M7r3c8X3">Intel® 64 and IA-32 Architectures Software Developer’s Manual, Combined Volumes: 1, 2A, 2B, 2C, 2D, 3A, 3B, 3C, 3D and 4 </a>章节3.7.2.1，这是一个`REX`前缀，允许指令寻址8位寄存器。汇编器可能够以这种方式丢到空字节。而且，由于当使用`RSP`而不是`ESP`时，对于`fnstenv`指令，地址大小前缀是非必须的，所以，`RAX`加上`0x07`字节的结果是`RAX`包含了`RIP`的地址。

![图片描述](upload/attach/202001/703263_E2DG6K5YDSTT6VX.png)

*图 6：使用`FPU`指令寻找`RIP`，终结*

### 方法2：使用跳转和调用

在开始测试该方法前，我想做些微小的修改。我想有必要将`EAX`替换成`RAX`。这种跳转和调用和手工干扰到调试器中有一些不同，所以我选择先开始编写一些汇编代码。下面的代码就是我选择用来最开始尝试的：

```c
[SECTION .text]
BITS 64
global _start
_start:
jmp label2
label1:
jmp getEIP
label2:
call label1
getEIP:
pop rax
```

执行了使用汇编的代码覆盖的`notepad`的入口点的结果并可以在**图 7**中看到。这种方法依旧没有空字节，长度上仅仅10个字节。

![图片描述](upload/attach/202001/703263_MJ8NRQGC3BUWUHY.png)

*图 7：使用跳转和调用寻找`RIP`，终结*

### 方法3：加载`RIP`的有效地址
