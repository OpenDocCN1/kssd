
```
=------------------------=
| [0x4003b0] |
| main: | // Start in Main but is called by the libc constructor
| (fcn) sym.main 68 |
| addiu sp, sp, -0x28 |
| sw ra, 0x24(sp) | // Saved Return Address
| sw fp, 0x20(sp) |
| move fp, sp |
| sw zero, 0x18(fp) |
| addiu v0, zero, 1 |
| sw v0, 0x1c(fp) |
| lw a0, 0x18(fp) |
| lw a1, 0x1c(fp) |
| jal sym.call_one ;[a] | // Call (Jump and Link) the function call_one
| nop |
| move sp, fp |
| lw ra, 0x24(sp) |
| lw fp, 0x20(sp) |
| addiu sp, sp, 0x28 |
| jr ra |
| nop |
=------------------------=
|
|
V
=------------------------=
| [0x4003f4] |
| (fcn) sym.call_one 68 | // Function call_one
| addiu sp, sp, -0x20 |
| sw ra, 0x1c(sp) | // Saved Return Address so we can jump back to main()
| sw fp, 0x18(sp) |
| move fp, sp |
| sw a0, 0x20(fp) |
| sw a1, 0x24(fp) |
| jal sym.call_two ;[a] | // Call (Jump and Link) the function call_two
| nop |
| lw v1, 0x20(fp) |
| lw v0, 0x24(fp) |
| addu v0, v1, v0 |
| move sp, fp |
| lw ra, 0x1c(sp) |
| lw fp, 0x18(sp) |
| addiu sp, sp, 0x20 |
| jr ra |
| nop |
=------------------------=
|
|
V
=-----------------------=
| [0x400438] |
| (fcn) sym.call_two 36 | // Function call_two
| addiu sp, sp, -8 |
| sw fp, 4(sp) |
| move fp, sp |
| addiu v0, zero, 1 |
| move sp, fp |
| lw fp, 4(sp) |
| addiu sp, sp, 8 |
| jr ra | // Since call_two does not call any other functions there is no need to save the return address onto the stack.
| nop |
=-----------------------=
```

通过分析函数入口，我们可以预测函数是否会调用另一个函数。这在尝试查找位于堆栈中的内存破坏漏洞时非常有用。

# 分支条件

研究人员在分析新体系结构时需要了解的最重要的一件事是处理器如何处理分支。就像以前一样，我们将利用 C 和 Radare2 来分析这一点。

下面的应用程序将从 `argv[1]` 中获取输入，并将其作为 `int` 进行转换，并查看它是否小于 5。

```
#include <stdio.h>
// What if....
// This is to analyze how basic branching works on MIPS
// by b1ack0wl
int main(int argc, char **argv[]){
if (argc < 2 ){
printf("Usage: %s number", argv[0]);
return 1;
}
int a = atoi(argv[1]); // cast argv[1] as an integer
if (a < 5) {
printf("%i is less than 5", a);
}
else{
printf("%i is greater than 5", a);
}
return 0;
}
```

现在我们来看看编译器会产生什么来满足我们的条件。

```
=-------------------------=
| [0x4003b0] |
| main: | // Main Function
| (fcn) sym.main 260 |
| addiu sp, sp, -0x28 |
| sw ra, 0x24(sp) |
| sw fp, 0x20(sp) |
| move fp, sp |
| lui gp, 0x42 |
| addiu gp, gp, 0x680 |
| sw gp, 0x10(sp) |
| sw a0, 0x28(fp) | // Store argc
| sw a1, 0x2c(fp) | // Store argv[]
| lw v0, 0x28(fp) | // Load argc into the $v0 register
| slti v0, v0, 2 | // Set $v0 to 1 if the value of $v0 is less than 2, otherwise set it to 0.
| beqz v0, 0x400418 ;[a] | // If $v0 is equal take the "true" branch. (Branch if Equal to Zero)
| nop |
=-------------------------=
t f
.---------------' '--------------------------------.
| |
| |
=-------------------------= =-----------------------=
| 0x400418 | | 0x4003e4 |
| lw v0, 0x2c(fp) | // load argv[] | lw v0, 0x2c(fp) |
| addiu v0, v0, 4 | // offset +4 | lw v1, (v0) |
| lw v0, (v0) | // load argv[1] | lui v0, 0x40 |
| move a0, v0 | | addiu a0, v0, 0x70f0 | // Load Format String
| lw v0, -0x7fd0(gp) | // $v0 = atoi | move a1, v1 | // Load argv[0]
| move t9, v0 | // $t9 = $v0 | lw v0, -0x7fd4(gp) | // Resolve printf pointer and load it into $v0
| jalr t9 ; atoi | // call atoi | move t9, v0 | // Move $v0 into $t9
| nop | | jalr t9 ; printf | // Call printf which is in $t9 (Jump and Link Register)
| lw gp, 0x10(fp) | | nop |
| sw v0, 0x18(fp) | // int a | lw gp, 0x10(fp) |
| lw v0, 0x18(fp) | // load int a | addiu v0, zero, 1 | // Return 1
| slti v0, v0, 5 | // if < 5 | j 0x40049c ;[d] | // Exit function
| beqz v0, 0x400478 ;[b] | // true if $v0=0 | nop |
| nop | =-----------------------=
=-------------------------= v
t f |
.-------------' '---------------------------------. '--.
| | |
| | |
=-----------------------= =-----------------------= |
| 0x400478 | | 0x400450 | |
| lui v0, 0x40 | | lui v0, 0x40 | | // Block 0x400450 does the same as 0x400478
| addiu a0, v0, 0x7118 | // load format string | addiu a0, v0, 0x7104 | | // Just needs to jump due to where it sits.
| lw a1, 0x18(fp) | // int a printf arg | lw a1, 0x18(fp) | |
| lw v0, -0x7fd4(gp) | // load printf in $v0 | lw v0, -0x7fd4(gp) | |
| move t9, v0 | // move $v0 to $t9 | move t9, v0 | |
| jalr t9 ; printf | // call $t9 (printf) | jalr t9 ; printf | |
| nop | | nop | |
| lw gp, 0x10(fp) | | lw gp, 0x10(fp) | |
=-----------------------= | j 0x400498 ;[c] | | // Jump to Return 0 due to memory position
v | nop | |
| =-----------------------= |
'-----------------. v |
.-----------------------------' |
| |
| |
=----------------= |
| 0x400498 | |
| move v0, zero | // Return 0 |
=----------------= |
v |
'-------------.-----------------------------------------'
|
|
=---------------------=
| 0x40049c |
| move sp, fp |
| lw ra, 0x24(sp) |
| lw fp, 0x20(sp) |
| addiu sp, sp, 0x28 |
| jr ra | // Function be gone!
=---------------------=
```

每当我们做比较时，我们就可以看到 `slti` 指令的用法。由于大量的比较运算符和类型，条件语句将成为学习新汇编语言中最耗时的部分。请参考 C 参考文献，以确保您分析了生成条件分支的所有不同方法。例如在 MIPS 中，有些条件语句使用可能被滥用的有符号或无符号立即数。

现在你已经看到了一些例子，只要你有一个编译器和反汇编器，你就应该有能力来学习任何处理器的架构和汇编。如果你不这样做，那么你将不得不以更加艰难的方式去做，阅读处理器的开发者手册，最终设计你自己的汇编器，模拟器和反汇编器。

# GPL

如果您正在审计使用开源软件的设备，该软件可能会根据通用公共许可证进行许可。如果是这样，那么简而言之，如果开发人员使用代码并编译它，则必须提供源代码！如果开发者拒绝开源，那么他们就违反了 GPL。

考虑到这一点，许多路由器或其他小型设备都使用 Linux（或 FreeRTOS）、Busybox 和其他利用 GPL 的开源软件。因此，在开始反汇编之前，尝试使用短语“$供应商源代码”或“$产品源代码”进行简单的 Google 搜索。以下是我只是通过 Google 搜索发现的一些示例源代码库。

- [Samsung Open Source Release Center](http://opensource.samsung.com/reception.do)
- [ASUS RT-AC68U Source Code](https://www.asus.com/us/Networking/RTAC68U/HelpDesk_Download/)
- [Belkin Open Source Code Center](http://www.belkin.com/us/support-article?articleNum=51238)
- [GPL Code Center - TP-LINK](http://www.tp-link.com/en/gpl-code.html)

现在您应该了解如何轻松利用 Google 或您最喜爱的搜索引擎来查找您正在审计设备的源代码。

# 利用

本节将假设读者具有利用基于内存损坏的漏洞的基本知识。如果没有，请参阅本页底部的 SmashtheStack 参考。SmashtheStack 是我个人开始进入 x86 开发之旅的地方。

如果您正在审计基于 MIPS 的 Linux 嵌入式设备，那么在分析您的目标二进制文件时，您已经看到以下几点：

```
# cat /proc/11554/maps
00400000-00402000 r-xp 00000000 1f:02 218 /pwnable/ShellCode_Required/socket_bof
00441000-00442000 rw-p 00001000 1f:02 218 /pwnable/ShellCode_Required/socket_bof
2aaa8000-2aaad000 r-xp 00000000 1f:02 440 /lib/ld-uClibc.so.0
2aaad000-2aaae000 rw-p 2aaad000 00:00 0
2aaec000-2aaed000 r--p 00004000 1f:02 440 /lib/ld-uClibc.so.0
2aaed000-2aaee000 rw-p 00005000 1f:02 440 /lib/ld-uClibc.so.0
2aaee000-2aafe000 r-xp 00000000 1f:02 445 /lib/libgcc_s.so.1
2aafe000-2ab3d000 ---p 2aafe000 00:00 0
2ab3d000-2ab3e000 rw-p 0000f000 1f:02 445 /lib/libgcc_s.so.1
2ab3e000-2ab79000 r-xp 00000000 1f:02 444 /lib/libc.so.0
2ab79000-2abb9000 ---p 2ab79000 00:00 0
2abb9000-2abba000 rw-p 0003b000 1f:02 444 /lib/libc.so.0
2abba000-2abbe000 rw-p 2abba000 00:00 0
7f81e000-7f833000 rwxp 7f81e000 00:00 0 [stack]
```

正如你所看到的堆栈和堆区域被标记为可执行的，所以 NX 不用担心。尽管堆栈是可执行的，但为了获得代码执行，还需要一些 ROP 操作。您还会发现 ASLR 并未在大多数设备上实现，因此我们不必首先找到信息泄露错误。

## 仿真

使用 Binwalk 提取固件，你就会想要模拟二进制文件来分析崩溃。我个人使用可在解压缩固件的 `chroot` 环境中启动的 QEMU 的静态内置版本。这使 exp 开发人员能够访问与嵌入式设备使用的相同 libc 库，因此唯一会改变的是 libc 地址。有时需要创建一个完整的 QEMU 系统，因为主机可能不支持程序正在使用的 IO 调用导致崩溃。

如果您使用的是基于 Debian 的 Linux Distro，那么您可以通过 `apt-get` 来安装 QEMU：`sudo apt-get install qemu-user-static qemu-system-*`

一旦你安装了 QEMU，你就需要将二进制文件复制到解压缩固件的根目录下。在这个例子中，我们将使用 `DVRF_v03` 的 MIPS Little Endian 模拟器：``cp `which qemu-mipsel-static` ./``

我们将使用 pwnable binary `/pwnable/Intro/stack_bof_01` 并为它制作一个小小的漏洞。然后，我们将把我们的有效载荷粘贴到实际设备上，以查看发生了什么。

二进制的源代码如下：

```
#include <string.h>
#include <stdio.h>
//Simple BoF by b1ack0wl for E1550
int main(int argc, char **argv[]){
char buf[200] ="\0";
if (argc < 2){
printf("Usage: stack_bof_01 <argument>\r\n-By b1ack0wl\r\n");
exit(1);
}
printf("Welcome to the first BoF exercise!\r\n\r\n");
strcpy(buf, argv[1]);
printf("You entered %s \r\n", buf);
printf("Try Again\r\n");
return 0x41; // Just so you can see what register is populated for return statements
}
void dat_shell(){
printf("Congrats! I will now execute /bin/sh\r\n- b1ack0wl\r\n");
system("/bin/sh -c");
exit(0);
}
```

所以我们有一个简单的基于栈的缓冲区溢出漏洞。目标是执行函数 `dat_shell`，但在分析 ELF 文件时，我们看到以下内容：

Entry point address: `0x00400630`
