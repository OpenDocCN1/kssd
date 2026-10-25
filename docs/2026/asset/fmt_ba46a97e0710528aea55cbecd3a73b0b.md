
# [原创]Windows X64汇编入门(1)

tankaiha

最近断断续续接触了些64位汇编的知识，这里小结一下，一是阶段学习的回顾，二是希望对64位汇编新手有所帮助。我也是刚接触这方面知识，文中肯定有错误之处，大家多指正。

文章的标题包含了本文的四方面主要内容：

（1）Windows：本文是在 Windows 环境下的汇编程序设计，调试环境为 Windows Vista 64位版，调用的均为 Windows API。

（2）X64：本文讨论的是 x64 汇编，这里的 x64 表示 AMD64 和 Intel 的 EM64T，而不包括 IA64。至于三者间的区别，可自行搜索。

（3）汇编：顾名思义，本文讨论的编程语言是汇编，其它高级语言的64位编程均不属于讨论范畴。

（4）入门：既是入门，便不会很全。其一，文中有很多知识仅仅点到为止，更深入的学习留待日后努力。其二，便于类似我这样刚接触 x64 汇编的新手入门。

本文所有代码的调试环境：Windows Vista x64，Intel Core 2 Duo。

## 1. 建立开发环境

### 1.1 编译器的选择

对应于不同的 x64 汇编工具，开发环境也有所不同。最普遍的要算微软的 `MASM`，在 x64 环境中，相应的编译器已经更名为 `ml64.exe`，随 Visual Studio 2005 一起发布。因此，如果你是微软的忠实 fans，直接安装 VS2005 即可。运行时，只需打开相应的64位命令行窗口（图1），便可以用 `ml64` 进行编译了。

![图1](https://bbs.kanxue.com/upload/attach/200705/11545_slyos6ez2ngoibr.jpg)

第二个推荐的编译器是 `GoASM`，共包含三个文件：`GoASM` 编译器、`GoLINK` 链接器和 `GoRC` 资源编译器，且自带了 `Include` 目录。它的最大好处是小，不用为了学习64位汇编安装几个G的VS。因此，本文的代码就在 `GoASM` 下编译。

第三个 `Yasm`，因为不熟，所以不再赘述，感兴趣的朋友自行测试吧。

不同的编译器，语法会有一定差别，这在下面再说。

### 1.2 IDE的选择

搜遍了Internet也没有找到支持 asm64 的IDE，甚至连个 Editor 都没有。因此，最简单的方法是自行修改 EditPlus 的 `MASM` 语法文件，这也是我采用的方法，至少可以得到语法高亮。当然，如果你懒得动手，那就用 `notepad` 吧。

没有IDE，每次编译时都要手动输入不少参数和选项，做个批处理就行了。

### 1.3 硬件与操作系统

硬件要求就是64位的CPU。操作系统也必须是64位的，如果在64位的CPU上安装了32位的操作系统，就算编译成功也无法运行程序。

## 2. 寄存器的改变

汇编是直接与寄存器打交道的语言，因此硬件对语言影响很大。先来看看 x64 与 x32 相比在硬件上多了什么，变了什么（图2）。

![图2](https://bbs.kanxue.com/upload/attach/200705/11545_p7x76lxku3lwnpq.jpg)

X64多了8个通用寄存器：`R8`、`R9`、`R10`、`R11`、`R12`、`R13`、`R14`、`R15`，当然，它们都是64位的。另外还增加了8个128位XMM寄存器，不过通常用不着。

X32中原有的寄存器在X64中均扩展为64位，且名称的第一个字母从`E`改为`R`。不过我们还是可以在64位程序中调用32位的寄存器，如`RAX`（64位）、`EAX`（低32）、`AX`（低16位）、`AL`（低8位）、`AH`（8到15位），相应的有`R8`、`R8D`、`R8W`和`R8B`。不过不要在程序中使用如`AH`之类的寄存器，因为在AMD的CPU上这种用法会与某些指令产生冲突。

## 3. 第一个x64汇编程序

本节，我们开始编写自己的第一个x64汇编程序。在这之前，先讲一下calling convention的改变。

### 3.1 API调用方式

把Calling convention放在第一个讲，代表它的重要性。在32位汇编中，我们调用一个API时，采用的是`stdcall`，它有两个特点：一是所有参数入栈，通过堆栈传递；二是被调用的API负责栈指针（`ESP`）的恢复，我们在调用`MessageBox`后不用`add esp,14h`，因为`MessageBox`已经恢复过了。

而在x64汇编中，两方面都发生了变化。一是前四个参数分别通过四个寄存器传递：`RCX`、`RDX`、`R8`、`R9`，如果还有更多的参数，才通过堆栈传递。二是调用者负责堆栈空间的分配与回收。

下面给出一段代码，功能是显示一个简单的`MessageBox`，注意对`RSP`的操作：

```asm
;示例代码1.asm
;语法：GoASM

DATA SECTION

text     db 'Hello x64!', 0
caption  db 'My First x64 Application', 0

CODE SECTION

START:
sub rsp,28h
xor r9d,r9d
lea r8, caption
lea rdx, text
xor rcx,rcx
call MessageBoxA
add rsp,28h
ret
```

这段代码是在`GoASM`中编译，指令部分`GoASM`与`ML64`差不多，关键是一些宏的定义有差别。比如`MASM`中的`.code`，在这里就成了`CODE SECTION`。下面再说区别，先编译。`GoASM`中编译分两步：

（1）编译：`goasm /x64 1.asm`
（2）链接：`golink 1.obj user32.dll`

如果一切正常，命令行中应显示图3的内容。

![图3](https://bbs.kanxue.com/upload/attach/200705/11545_qm8fj0gr3oqsru7.jpg)

运行一下，我们的第一个64位Windows程序就运行了。

![图4](https://bbs.kanxue.com/upload/attach/200705/11545_d5v0dpwqwmtcdil.jpg)

`GoASM`还有一个特点是支持宏：`ARG`和`INVOKE`，使用这两个宏可以免除程序员自己对堆栈进行操作。不过初学吗，还是从基础掌握比较好。下面的一段代码是相同功能的`MASM`代码，注意看看区别。`ML64`至今仍不支持宏，所以每一步工作都要自己做。

```asm
;示例代码2.asm
;语法：ML64
extrn MessageBoxA: proc

.data

text     db 'Hello x64!', 0
caption  db 'My First x64 Application', 0

.code

Main proc
sub rsp,28h
xor r9d,r9d
lea r8, caption
lea rdx, text
xor rcx,rcx
call MessageBoxA
add rsp,28h
ret

Main ENDP
end
```

编译这段代码的命令行是：`ml64 2.asm /link /subsystem:windows /entry:Main user32.lib`。如果正常，应该如图5显示那样。

![图5](https://bbs.kanxue.com/upload/attach/200705/11545_wgj2ns6vcsjwe1q.jpg)

很有意思吧，在64位系统下，我们仍然调用`user32`的API。可能是名称用习惯了，微软自己都懒得改了吧。

### 3.2 64位的堆栈

代码中还有一处值得注意，那就是`sub rsp,28h`和`add rsp,28h`。`28h`这个数值是怎么来的呢？

首先，x64中堆栈被扩展为64位；其次，我们在调用`MessageBoxA`时，要给四个参数外加一个返回地址留空间，因此8（位）×5=40=`28h`。

另外一些小问题要注意，AMD64不支持`push`32位寄存器的指令，最好的方法就是`push`和`pop`都用64位寄存器。EM64T如何？看了下Intel的开发手册，各个指令都分三种情况：纯32位、纯64位和32与64位混合。下面是手册的片段：

```text
Opcode*      Instruction        64-Bit Mode       Compat/Leg Mode      Description
FF /6         PUSH r/m16         Valid                 Valid            Push r/m16.
FF /6         PUSH r/m32         N.E.                  Valid            Push r/m32.
FF /6         PUSH r/m64         Valid                  N.E.           Push r/m64.

Default operand size 64-bits.
```

没别的好方法，使用中多注意，尽量在64位程序中使用64位寄存器。

## 4. 一些参考资料

写完了第一个hello world，本文就此打住。本还想写一些内容，但掌握不深，留待下回吧。感觉有些资料不得不在第一篇文章中放出来，因为它们是现有学习x64汇编的最好教材了，文中很多代码和知识点也来自于这些资料。

1. 《Moving to Windows x64》，出自：f85K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3&6@1j5$3!0J5k6g2)9J5k6h3y4G2L8g2)9J5c8V1k6A6L8r3g2K6i4K6u0r3N6X3W2K6N6r3q4Q4y4h3k6^5y4U0c8Q4x3X3g2Z5N6r3@1。
2. `GoASM`的帮助文档，目前最好的64位汇编教程。出自：6e7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3A6G2M7X3N6G2L8W2)9J5k6h3k6J5k6h3g2K6k6i4u0$3k6g2)9J5k6h3y4G2i4K6u0W2N6h3D9。
3. 《开始进行 64 位 Windows 系统编程之前需要了解的所有信息》，出自：8e7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3y4Z5K9h3&6S2i4K6u0r3e0g2y4p5e0W2)9J5c8X3I4A6j5Y4u0S2M7Y4W2Q4x3V1k6i4K9h3&6V1k6i4k6Q4x3V1j5$3y4r3u0A6N6q4)9J5c8X3W2K6M7%4g2W2M7%4R3$3y4q4)9J5k6h3#2K6M7s2R3。
4. 来自CodeGurus的两篇文章：
   - 《Assembler & Win64》，出自：34aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4G2k6r3g2Y4N6i4u0#2M7#2)9J5k6h3u0W2i4K6u0r3j5$3!0V1k6h3N6#2M7Y4g2K6i4K6u0r3f1s2u0G2k6%4u0S2L8h3#2A6L8X3N6Q4x3V1k6S2M7%4y4W2L8h3u0D9k6i4u0Q4x3U0k6%4K9h3^5$3y4q4)9#2k6X3g2F1i4K6u0W2K9s2c8E0
   - 《bout RIP relative addressing》，出自：ac2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4G2k6r3g2Y4N6i4u0#2M7#2)9J5k6h3u0W2i4K6u0r3j5$3!0V1k6h3N6#2M7Y4g2K6i4K6u0r3f1s2u0G2k6%4u0S2L8h3#2A6L8X3N6Q4x3V1k6J5K9i4m8J5k6h3I4S2N6r3W2$3k6h3q4V1k6s2u0W2M7%4y4A6L8X3N6Q4y4h3k6W2L8W2)9J5k6h3S2@1L8b7。
5. AMD开发手册
6. Intel开发手册，注意是新的《Intel® 64 and IA-32 Architectures software Developer’s Manual》

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [1.jpg](javascript:void(0);)（55.61kb，2349次下载）
- [2.jpg](javascript:void(0);)（63.77kb，2475次下载）
- [3.jpg](javascript:void(0);)（29.36kb，2274次下载）
- [4.jpg](javascript:void(0);)（6.28kb，2249次下载）
- [5.jpg](javascript:void(0);)（37.78kb，2291次下载）
