
# [翻译]二进制漏洞利用：如何使用GDB调试

本文讲解如何使用GDB调试二进制文件，方便大家更好理解二进制漏洞利用。原文链接:

b0aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2P5X3g2J5K9h3q4Q4x3X3c8D9j5h3u0K6i4K6u0W2j5$3!0E0i4K6u0r3k6r3g2T1N6h3N6Y4K9h3&6Y4i4K6u0V1N6$3W2@1K9q4)9J5k6r3N6V1j5W2)9J5k6r3W2F1N6s2u0G2k6s2g2U0N6r3W2G2L8W2)9J5c8R3`.

# DEBUGGING WITH GDB 用gdb调试

This is a very brief introduction into compiling ARM binaries and basic debugging with GDB. As you follow the tutorials, you might want to follow along and experiment with ARM assembly on your own. In that case, you would either need a spare ARM device, or you just set up your own Lab environment in a VM by following the steps in this short How-To.

这是一个非常简单的介绍，介绍如何如何将代码编译成ARM二进制文件和使用GDB进行基本的调试。学习教程时，您可能想一步步跟随教程，并在自己的设备上实践ARM 汇编，所以你要么需要一个备用的ARM设备，要么在how-To里搭建自己的实验环境。

You can use the following code from Part 7 – Stack and Functions, to get familiar with basic debugging with GDB.

可以用第七部分：栈和函数里的代码，熟悉GDB的基本调试

```
.section .text

.global _start



_start:

    push {r11, lr}    /* Start of the prologue. Saving Frame Pointer and LR onto the stack */

    add r11, sp, #0   /* Setting up the bottom of the stack frame */

    sub sp, sp, #16   /* End of the prologue. Allocating some buffer on the stack */

    mov r0, #1        /* setting up local variables (a=1). This also serves as setting up the first parameter for the max function */

    mov r1, #2        /* setting up local variables (b=2). This also serves as setting up the second parameter for the max function */

    bl max            /* Calling/branching to function max */

    sub sp, r11, #0   /* Start of the epilogue. Readjusting the Stack Pointer */

    pop {r11, pc}     /* End of the epilogue. Restoring Frame pointer from the stack, jumping to previously saved LR via direct load into PC */



max:

    push {r11}        /* Start of the prologue. Saving Frame Pointer onto the stack */

    add r11, sp, #0   /* Setting up the bottom of the stack frame */

    sub sp, sp, #12   /* End of the prologue. Allocating some buffer on the stack */

    cmp r0, r1        /* Implementation of if(a<b) */

    movlt r0, r1      /* if r0 was lower than r1, store r1 into r0 */

    add sp, r11, #0   /* Start of the epilogue. Readjusting the Stack Pointer */

    pop {r11}         /* restoring frame pointer */

    bx lr             /* End of the epilogue. Jumping back to main via LR register */
```

Personally, I prefer using GEF as a GDB extension. It gives me a better overview and useful features. You can try it out here: GEF – GDB Enhanced Features.

我个人更喜欢使用GDB的扩展`GEF`，它给我提供了更好的概览和有用的功能，你可以试试看：GEF-GDB增强功能

Save the code above in a file called `max.s` and compile it with the following commands:

把上面的代码保存到`max.s`文件后用下面命令编译

```
$ as max.s -o max.o

$ ld max.o -o max
```

The debugger is a powerful tool that can:

Load a memory dump after a crash (post-mortem debugging)

Attach to a running process (used for server processes)

Launch a program and debug it

Launch GDB against either a binary, a core file, or a Process ID:

Attach to a process: `$ gdb -pid $(pidof <process>)`

Debug a binary: `$ gdb ./file`

Inspect a core (crash) file: `$ gdb -c ./core.3243`

调试器是一个强大的工具可用来：

1. 崩溃后加载内存转储（事后调试）
2. 附加到正在运行的进程（用于服务器进程）
3. 启动程序并调试它

无论是二进制文件，核心文件或进程ID都可藉由GDB运行

1. 附加到进程：`gdb -pid $(进程的PID)`
2. 调试二进制文件：`$ gdb ./file`
3. 查看核心（崩溃）文件：`$ gdb -c ./core.3243`

```
$ gdb max
```

If you installed GEF, it drops you the `gef>` prompt.

若你安装了`GEF`会释放`gef>`提示符

This is how you get help:

`(gdb) h`

`(gdb) apropos <search-term>`

你可通过以下方式获得帮助

`(gdb) h`

`(gdb) apropos <search-term>`（译者注：下面将search-term指定为registesr看输出什么）

```
	gef> apropos registers

	collect -- Specify one or more data items to be collected at a tracepoint

	core-file -- Use FILE as core dump for examining memory and registers

	info all-registers -- List of all registers and their contents

	info r -- List of integer registers and their contents

	info registers -- List of integer registers and their contents

	maintenance print cooked-registers -- Print the internal register configuration including cooked values

	maintenance print raw-registers -- Print the internal register configuration including raw values

	maintenance print registers -- Print the internal register configuration

	maintenance print remote-registers -- Print the internal register configuration including each register's

	p -- Print value of expression EXP

	print -- Print value of expression EXP

	registers -- Display full details on one

	set may-write-registers -- Set permission to write into registers

	set observer -- Set whether gdb controls the inferior in observer mode

	show may-write-registers -- Show permission to write into registers

	show observer -- Show whether gdb controls the inferior in observer mode

	tui reg float -- Display only floating point registers

	tui reg general -- Display only general registers

	tui reg system -- Display only system registers
```

（译者注：看倒数第8行）

## Breakpoint commands

`break` (or just `b`) `<function-name>`

`break <line-number>`

`break filename:function`

`break filename:line-number`

`break *<address>`

`break +<offset>`

`break –<offset>`

`tbreak` (set a temporary breakpoint)

`del <number>` (delete breakpoint number x)

`delete` (delete all breakpoints)

`delete <range>` (delete breakpoint ranges)

`disable/enable <breakpoint-number-or-range>` (does not delete breakpoints, just enables/disables them)

`continue` (or just `c`) – (continue executing until next breakpoint)

`continue <number>` (continue but ignore current breakpoint number times. Useful for breakpoints within a loop.)

`finish` (continue to end of function)

断点命令：

1. `break`(或只是`b`) + 函数名
2. `break` 行号
3. `break` 文件名:函数名
4. `breake` 文件名:行号
5. `break *地址`
6. `break +偏移`
7. `break –偏移`
8. `tbreak`（设置临时断点）
9. `del` 号码（删除断点号码）
10. `delete` （删除所有的断点）
11. `delete` 地址范围 （删除范围内的断点）
12. `disable/enable` 断点号码或者范围 （不删除断点，只是使能/去使能这些断点）
13. `continue`（或只是`c`） （继续执行直到下一个断点）
14. `finish` （继续执行直到末尾）

```
gef> break _start

gef> info break

Num Type Disp Enb Address What

1 breakpoint keep y 0x00008054 <_start>

 breakpoint already hit 1 time

gef> del 1

gef> break *0x0000805c

Breakpoint 2 at 0x805c

gef> break _start
```

This deletes the first breakpoint and sets a breakpoint at the specified memory address. When you run the program, it will break at this exact location. If you would not delete the first breakpoint and just set a new one and run, it would break at the first breakpoint.

以上代码删除了第一个断点，然后在特定的内存地址下断，当你运行程序会断在准确的地址处，要是你不删第一个断点又下了一个新断点，然后运行，程序会断在第一断点处。

## Start and Stop

Start program execution from beginning of the program

`run`

`r`

`run <command-line-argument>`

Stop program execution

`kill`

Exit GDB debugger

`quit`

`q`

开始和停止：

从程序的开始启动执行：

1. `run`
2. `r`
3. `run` 命令行参数

停止程序执行

1. `kill`

退出GDB调试器

1. `quit`
2. `q`

```
gef> run
```

![](https://bbs.kanxue.com/upload/attach/201908/803510_47XKXAJTKS8QZYD.jpg)

Now that our program broke exactly where we wanted, it’s time to examine the memory. The command `x` displays memory contents in various formats.

既然程序断在我们希望的地方，现在有时间检查内存，用命令`x`以各种格式显示内存的内容

![](https://bbs.kanxue.com/upload/attach/201908/803510_A9YDFWNTQYHXM4E.jpg)

```
gef> x/10i $pc

=> 0x8054 <_start>: push {r11, lr}

 0x8058 <_start+4>: add r11, sp, #0

 0x805c <_start+8>: sub sp, sp, #16

 0x8060 <_start+12>: mov r0, #1

 0x8064 <_start+16>: mov r1, #2

 0x8068 <_start+20>: bl 0x8074 <max>

 0x806c <_start+24>: sub sp, r11, #0

 0x8070 <_start+28>: pop {r11, pc}

 0x8074 <max>: push {r11}

 0x8078 <max+4>: add r11, sp, #0

gef> x/16xw $pc

0x8068 <_start+20>: 0xeb000001  0xe24bd000  0xe8bd8800  0xe92d0800

0x8078 <max+4>:     0xe28db000  0xe24dd00c  0xe1500001  0xb1a00001

0x8088 <max+20>:    0xe28bd000  0xe8bd0800  0xe12fff1e  0x00001741

0x8098:             0x61656100  0x01006962  0x0000000d  0x01080206
```
