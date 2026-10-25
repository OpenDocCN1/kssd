
# How to begin to write an Os（有兴趣的看看）

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-709.htm`](https://www.pediy.com/kssd/pediy07/pediy7-709.htm)

英文原文：

How to begin to write an Os

---------------------------

This tutorial is meant for those who are familiar with assembly languges and some knowledge of system internal. Writing an Os is a challenging project. The initial step of writing one is to code an efficient boot loader to load the kernel.

What happens when you boot a computer?

- The Bios is started first
- It read the boot sector and loads it into the memory

The boot sector is the first sector of the disk. A sector contain 512 bytes and that is the limitation to write our boot loader. So the boot loader must be at the most 512 bytes. So we must load the real program or the kernel using the loader. The boot sector is loaded at the physical address `0000:7c00`.

And how does the Bios knows whether the disk has a valid bootsector?

The final two bytes of the bootsector ie; bytes 511 and 512 should be `Aah` and `55h` respectively. In other words, the last word in the bootsector should be `AA55h`.

Tools required:

- `NASM` or any other good assembler
- Interrupt list like Ralf Brown's interrupt list
- PC Emulator like `Bochs` will help you and avoid frequent reboot of the system. But this is not a compulsory requirement.

Now we will move into the coding part. Let's start with a simple boot loader.

```
;** A simple bootloader-bl.asm
;****************************
org 0x7c00 ; Bios loads our loader at this point

mov si, msg ; Displays a message on the screen
call displaystring

mov ah, 0 ; Wait for key
int 016h

db 0EAh ; Machine reboot
dw 0000h
dw 0FFFFh

displaystring: ; Function to display string
lodsb
or al,al
jz short finish
mov ah,0x0E ; Echo a char
mov bx,0x0007
int 0x10
jmp displaystring

finish:
retn

msg db 'Press Any Key …', 13, 10, 0
times 510-($-$$) db 0 ; Fills the rest of the space except the last two bytes with zero
dw 0AA55h ; Magic word for a valid bootsector
;****************************

; The End
;****************************
```

Now we have to compile this to plain binary. And then write the `bl.bin` into the bootsector.

`NASM bl.asm -f bin -o bl.bin`

You can use any programs to write the binary file onto the bootsector. Even the good old `Debug.exe` can also be used for that.

```
c:\>debug bl.bin

-w 100 0 0 1 ; writes bl.bin into the floppy bootsector
-q

c:\>
```

Set the defualt boot to floppy and boot the machine.

If you are using a `Bochs` freedos emulator, you will have to do the following steps;

- Edit file `'bochsrc'` and change `boot` to `'a'` ie; `boot: a`
- Copy the binary file `'bl.bin'` into the `Bochs` directory and rename as `'a.img'`
- Run `'Bochs.exe'`

You will see the message `'Press Any Key …'`

He he….Did I hear you screaming in joy? Yes, you have written a successful bootloader.

This is not the end. The main purpose of a bootloader is not yet sattisfied. And what is it?

You will have to load the kernel into the memory because a 512 bytes of code will not run a machine with all its power. The next step is to load a simple kernel into the memory.

I have used many interrupt calls in the following example code. Each of them is properly commented.

First of all we have to allocate stack for our program. Then load the kernel into any know address. I have loaded the kernel into `7c00h + 512` for the simplicity of compiling the loader and kernel in the same program. Before loading we have to reset the disk ie; bringing the disk head to the track zero. This is same like seeking the file descriptor to beginning of the file. Then load the kernel into memory. And what mext? Ofcourse jump into the loaded kernel. So simple…isn't it? Try studying this example code given below.

```
;********************************
;** Boot Loader
;** A simple bootloader-bl.asm
;********************************
; Boot Loader by Fajib
; This is the boot sector….
; This will load the real os from the disk into the memory
org 0x7c00

start:
; setting the stack for loading the program
cli
mov ax, 0x9000
mov ss, ax
mov sp, 0xffff
sti

mov [bootdrv], dl ; dl contains the bootdisk name ie; dl = 0 if floppy a:\
call load ; function to load the kernel from disk into memory

mov si, msgloadsuccess
call putstr
jmp loadkernel ; I can directly jump into kernel becoz I load the kernel
; at 7c00 + 512
;hangs if kernel not loaded

oops:
mov si, msghang
call putstr
jmp oops
retf

; loader variables
bootdrv db 0
msgresetfail db 'Disk Reset Failure!', 13, 10, 0
msgresetsuccess db 'Disk reset success…', 13, 10, 0
msgkernelload db 'Loading kernel…', 13, 10, 0
msgloadsuccess db 'Kernel loaded successfully…', 13, 10, 0
msghang db '.', 0

; loader functions
load:
; we have to reset the disk before it move to the loaded program
; say our real program is stored in sector 2
; let's load it
push ds ; reset disk system
mov ax, 0 ; forces controller to recalibrate drive heads (seek to track 0)
mov dl, [bootdrv]
int 13h
pop ds
jc resetfail

mov si, msgresetsuccess
call putstr

mov ax,0 ; loads sector into memory
mov es,ax
mov ah,2
mov al,1 ; loading 1 sector-kernel will be loaded. Increase al for loading more
mov dx,0
mov cx,2 ; ch = cylinder number and cl = sector number 1-63
mov bx,7e00h ; 7e00h = 7c00h + 512 … loading my program here makes it easy for me
int 13h ; we can directly compile bootsector and kernel together in a single
; program
jc load ; if fail then try to load it again
mov si, msgkernelload
call putstr
retn

resetfail:
mov si, msgresetfail
call putstr
retn

putstr:
lodsb
or al,al
jz short putstrd
mov ah,0x0E
mov bx,0x0007
int 0x10
jmp putstr

putstrd:
retn

times 510-($-$$) db 0 ; Filling the remaining free space in the sector
dw 0AA55h

;********************************
;** End Boot Loader
;** A simple bootloader-bl.asm
;********************************

;********************************
;** Start Kernel
;** A simple bootloader-bl.asm
;********************************
;Kernel stored in Sector 2
loadkernel:

mov si, mymsg
call putstr
call Console ; Simple Console
db 0EAh ; machine reboot
dw 0000h
dw 0FFFFh

mymsg db "Message from Fajib's Simple Kernel…", 13, 10, 0

Console:
mov si, msgconsole
call putstr

mov si, prompt
call putstr

Waitkey:
mov ah, 00h ; Wait for key
int 016h
cmp al, 27 ; Check for ESC key
je Finish
cmp al, 13 ; Check for ENTER key
je NewLine
jmp EchoChar

NewLine:
mov si, prompt
call putstr
jmp Waitkey

EchoChar:
mov ah, 0x0E ; Display the char with out attribute
mov bx, 0x0007
int 10h
jmp Waitkey

Finish:
retn

prompt db 13, 10, 'C:\>', 0
msgconsole db 13, 10, 'Console: Press ESC to reboot', 13, 10, 0

;********************************
;** End Kernel
;** A simple bootloader-bl.asm
;********************************
```

Wow! You have now loaded a kernel successfully… But do not stop here… Improvise your kernel as good possible… Experimentation is the best learning method… Do inform me about your improvements and bugs in my code to me at;

Email: `fajib@mail.com`

Irc: I will be there in `#osdev` as `Devil_liveD`

中文翻译：



# 自己动手写操作系统

自由软件社区是一个充满自由和梦想的地方，在 10 余年的时间里它创造了一个又一个奇迹。然而，这些奇迹的创造者不只是 Stallman，也不只是 Linus Torvalds，而是活跃在世界各地的不计其数的开发人员。

在使用各种功能强大的自由软件时，我总会对其开发者充满崇敬之情，期盼有朝一日自己也能成为他们中的一员。很多对自由社区充满向往之情的人，虽然也想努力融身于其中，但又不知该怎么做。那么，就请与我们一起从编写一个简单的操作系统开始吧！

## 我们要做的事情

有人可能担心自己既没有学过计算机原理，也没有学过操作系统原理，更不懂汇编语言，对 C 语言也一知半解，能写操作系统吗？答案是没问题。我将带大家一步一步完成自己的操作系统。当然如果学一学上述内容再好不过。

首先要明确处理器(也就是 CPU)控制着计算机。对 PC 而言，启动的时候，CPU 都处在实模式状态，相当于只是一个 Intel 8086 处理器。也就是说，即使你现在拥有一个奔腾处理器，它的功能也只能是 8086 级别。从这一点上来讲，可以使用一些软件把处理器转换到著名的保护模式。只有这样，我们才可以充分利用处理器的强大功能。

编写操作系统开始是对 BIOS 控制，取出存储在 ROM 里的程序。BIOS 是用来执行 POST(Power On Self Test，自检)的。自检是检查计算机的完整性(比如外设是否工作正常、键盘是否连接等)。这一切完成以后，你就会听到 PC 喇叭发出一声清脆的响声。如果一切正常，BIOS 就会选择一个启动设备，并且读取该设备的第一扇区(即启动扇区)，然后控制过程就会转移到指定位置。启动设备可能是一个软盘、光盘、硬盘，或者其它所选择的设备。在此我们把软盘作为启动设备。如果我们已经在软盘的启动扇区里写了一些代码，这时它就被执行。因此，我们的目的很明确，就是往软盘的启动扇区写一些程序。

首先使用 8086 汇编来写一个小程序，然后将其拷贝至软盘的启动扇区。为了实现拷贝，要写一个 C 程序。最后，使用软盘启动计算机。

## 需要的工具

- `as86`：这是一个汇编程序，它负责把写的代码转换成目标文件。
- `ld86`：这是一个连接器，`as86` 产生的目标代码由它来转换成真正的机器语言。机器语言是 8086 能够解读的形式。
- `GCC`：著名的 C 编程器。因为我们需要写一个 C 程序将自己的 OS 转移到软盘中。
- 一张空软盘：它用于存储编写的操作系统，也是启动设备。
- 一台装有 Linux 的计算机：这台机器可以很旧，386、486 都可以。

在大部分标准 Linux 发行版中都会带有 `as86` 和 `ld86`。在我使用的 Red Hat 7.3 中就包含有这两个工具，并且在默认的情况下，它已经安装在机器里。如果使用的 Linux 没有这两个工具，可以从网上下载(`http://www.cix.co.uk/~mayday/`)，这两个工具都包含在一个名为 `bin86` 的软件包中。此外，有关的文档也可以在网上获得（`www.linux.org/docs/ldp/howto/Assembly-HOWTO/as86.html`)。

## 开始工作

使用一个你喜欢的编辑器输入以下内容：

```
entry start

start:

mov ax,#0xb800

mov es,ax

seg es

mov [0],#0x41

seg es

mov [1],#0x1f

loop1: jmp loop1
```

这是 `as86` 可以读懂的一段汇编程序。第一个句子指明了程序的入口点，声明整个过程从 `start` 处开始。第二行指明了 `start` 的位置，说明整个程序要从 `start` 处开始执行。`0xb800` 是显存的开始地址。`#`表明其后是一个立即数。执行语句：

`mov ax,#oxb800`

`ax` 寄存器的值就变为 `0xb800`，这就是显存的地址。下面再将这个值移至 `es` 寄存器，`es` 是附加段寄存器。请记住 8086 有一个分段的体系结构。它的各段寄存器为代码段、数据段、堆栈段和附加段，对应的寄存器名称分别为 `cs`、`ds`、`ss` 和 `es`。事实上，我们把显存地址送入了附加段，因此，任何送入附加段的东西都会被送到显存中。

要在屏幕上显示字符，就需要向显存中写两个字节。前一个是所要显示字符的 ASCⅡ值，第二个字节表示该字符的属性。属性包括字符的前景色、背景色及是否闪烁等等。`seg es` 指明下一个将要执行的指令是指向 `es` 段的。所以，我们把值 `0x41`(在 ASCⅡ中表示的字符是 `A`)送到显存的第一个字节中。接下来要把字符的属性送到下一个字节当中。在此输入的是 `0x1f`，该属性指的是在蓝色背景下显示白色的字符。因此，如果执行这个程序，就可以在屏幕上得到显示在蓝底上的一个白色的 `A`。接着是一个循环。因为在执行完显示字符的任务后，要么让程序结束，要么使用一个循环使其永远运行下去。把该文件命名为 `boot.s`，然后存盘。

此处显存的概念说得不是很清楚，有必要进一步解释一下。假设屏幕由 80 列×25 行组成，那么第一行就需要 160 字节，其中一个字节用于表示字符，另外一个字节用于表示字符的属性。如果要在第三行显示某一字符的话，就要跳过显存的第 0 和 1 字节(它们是用于显示第 1 列的)，第 2 和 3 字节(它们是用于显示第 2 列的)，然后把需要显示字符的 ASCⅡ码值入第 4 字节，把字符的属性写入第 5 字节。

## 把程序写至启动扇区

下面写一个 C 程序，把我的操作系统写入软盘第一扇区。程序内容如下：

```c
#include <sys/types.h> /* unistd.h 需要这个文件 */

#include <unistd.h> /* 包含有 read 和 write 函数 */

#include <fcntl.h>

int main()

{

char boot_buf[512];

int floppy_desc, file_desc;

file_desc = open("./boot", O_RDONLY);

read(file_desc, boot_buf, 510);

close(file_desc);

boot_buf[510] = 0x55;

boot_buf[511] = 0xaa;

floppy_desc = open("/dev/fd0", O_RDWR);

lseek(floppy_desc, 0, SEEK_CUR);

write(floppy_desc, boot_buf, 512);

close(floppy_desc);

}
```

首先，以只读模式打开 `boot` 文件，然后在打开文件时把文件描述符复制到 `file_desc` 变量中。从文件中读取 510 个字符，或者读取直到文件结束。在本例中由于文件很小，所以是读取至文件结束。然后关闭文件。

最后 4 行代码打开软盘驱动设备(一般来说是`/dev/fd0`)。使用 `lseek` 找到文件开始处，然后从缓冲中向软盘写 512 个字节。

在 `read`、`write`、`open` 和 `lseek` 的帮助页中，可以看到与函数所有有关的参数及其使用方法。程序中有两行比较难懂：

`boot_buf[510] = 0x55;`

`boot_buf[511] = 0xaa;`

该信息是用于 BIOS 的，如果它识别出该设备是一个可启动的设备，那么在第 510 和 511 的位置，该值就应该是 `0x55` 和 `0xaa`。程序会把文件 `boot` 读至名为 `boot_buf` 的缓冲中。它要求改变第 510 和第 511 字节，然后把 `boot_buf` 写至软盘之上。如果执行代码，软盘上的前 512 字节就包含了启动代码。最后，把文件存为 `write.c`。

## 编译运行

使用下面的命令把文件变为可执行文件：

```
as86 boot.s -o boot.o

ld86 -d boot.o -o boot

cc write.c -o write
```

首先将 `boot.s` 文件编译成目标文件 `boot.o`，然后将该文件连接成最终的 `boot` 文件。最后 C 程序编译成可执行的 `write` 文件。

插入一个空白软盘，运行以下程序：

```
./write
```

重新启动电脑，进行 BIOS 的界面设置，并且把软盘设为第一个启动的设备。然后插入软盘，电脑从软盘上启动。

启动完成后，在屏幕上可以看到一个字母 `A`(蓝底白字)，启动速度很快，几乎是在瞬间完成。这就意味着系统已经从我们制作的软盘上启动了，并且执行了刚才写入启动扇区的程序。现在，它正处在一个无限循环的状态。所以，如果想进入 Linux，必需拿掉软盘，并且重启机器。

至此，这个操作系统就算完成了，虽然它没有实现什么功能，但是它已经可以启动机器了。

## 上一期回顾

上一期，我讲述了如何在软盘的启动扇区写一些代码，然后再从软盘启动的过程。制作好一个启动扇区，在切换到保护模式之前，我们还应该知道如何使用 BIOS 中断。BIOS 中断是一些由 BIOS 提供的、为了使操作系统的创建更容易的低级程序。在本文中，我们将学习处理 BIOS 的中断。

### 为什么要用 BIOS

BIOS 会把启动扇区拷贝至 RAM 中，并且执行这些代码。除此之外，BIOS 还要做很多其它的事情。当一个操作系统刚开始启动时，系统中并没有显卡驱动、软盘驱动等任何驱动程序。因此，启动扇区中不可能包含任何一个驱动程序，我们要采取其它的途径。这个时候，BIOS 就可以帮助我们了。BIOS 中包含有各种可以使用的程序，包括检测安装的设备、控制打印机、计算内存大小等用于各种目的的程序。这些程序就是所说的 BIOS 中断。

### 如何调用 BIOS 中断

在一般的程序设计语言中，函数的调用是一件非常容易的事情。比如在 C 语言中，如果有一个名为 `display` 的程序，它带有两个参数，其中参数 `nofchar` 表示显示的字符数，参数 `attr` 表示显示字符的属性。那么要调用它，只需给出程序的名称即可。对于中断的调用，我们使用的是汇编语言中的 `int` 指令。

比如，在 C 语言中要显示一些东西时，使用的指令如下所示：

`display(nofchar，attr)；`

而使用 BIOS 时，要实现相同功能使用的指令如下：

`int 0x10`

### 如何传递参数

在调用 BIOS 中断之前，我们需要先往寄存器中送一些特定的值。假设要使用 BIOS 的中断 `13h`，该中断的功能是把数据从软盘传送至内存之中。在调用该中断之前，要先指定拷贝数据的段地址，指定驱动器号、磁道号、扇区号，以及要传送的扇区数等等。然后，就要往相应的寄存器送入相应的值。在进行下面的步骤前，读者有必要对这一点有比较明确地认识。

此外，一个比较重要的事实是同一个中断往往可以实现各种不同的功能。中断所实现的确切功能取决于所选择的功能号，功能号一般都存在 `ah` 寄存器之中。比如中断 `13h` 可以用于读磁盘、写磁盘等功能，如果把 `3` 送入 `ah` 寄存器中，那么中断选择的功能就是写磁盘；如果把 `2` 送入 `ah` 寄存器中，选择的功能则是读磁盘等。

## 我们要做的事情

这次我们的源代码由两个汇编语言程序和一个 C 程序组成。第一个汇编文件是引导扇区的代码。在引导扇区中，我们写的代码是要把软盘中第二扇区拷贝至内存段的 `0x500` 处(地址是 `0x5000`，即偏移地址为 0)。这时我们需要使用 BIOS 的中断 `13h`。这时启动扇区的代码就会把控制权转移至 `0x500` 处。在第二个汇编文件中，代码会使用 BIOS 中断 `10h` 在屏幕上显示一个信息。C 程序实现的功能则是把可执行的文件 1 拷贝至启动扇区，把可执行的文件 2 拷贝至软盘的第二扇区。

### 启动扇区代码

使用中断 `13h`，启动扇区把软盘第二扇区里的内容加载至内存的 `0x5000` 处(段地址为 `0x500`)。下面的代码是用于实现这一目的的代码，将其保存至文件 `sbect.s` 中。

```
LOC1=0x500

entry start

start:

mov ax,#LOC1

mov es,ax

mov bx,#0 

mov dl,#0 

mov dh,#0 

mov ch,#0 

mov cl,#2 

mov al,#1 

mov ah,#2 

int 0x13

jmpi 0,#LOC1 
```

上面代码第一行类似于一个宏。接下去的两行则是把值 `0x500` 加载至 `es` 寄存器中，这是软盘上第二扇区代码将拷贝到的地方(第一扇区是启动扇区)。这时，把段内的偏移设为 0。

接下来把驱动器号送入 `dl` 寄存器中，其中磁头号送入 `dl` 寄存器中，磁道号送入 `ch` 寄存器中，扇区号送入 `cl` 寄存器中，扇区数送入 `al` 寄存器之中。我们想要实现的功能是把扇区 2、磁道号为 0、驱动器号为 0 的内容送至段地址 `0x500` 处。所有这些参数都和 1. 44MB 的软盘相对应。

把 2 送入 `ah` 寄存器中，是选择了由中断 `13h` 提供的相应功能，即实现从软驱转移数据的功能。

最后调用中断 `13h`，并且转至偏移为 0 的段地址 `0x500` 处。

### 第二个扇区的代码

第二个扇区中的代码如下所示(把这些代码保存至文件 `sbect2.s` 之中)：

```
entry start

start:

mov ah,#0x03 

xor bh,bh

int 0x10 

mov cx,#26 

mov bx,#0x0007 

mov bp,#mymsg

mov ax,#0x1301 

int 0x10 

loop1: jmp loop1

mymsg:

.byte 13,10

.ascii “Operating System is Loading……” 
```

上面代码将被加载至段地址为 `0x500` 处，并且被执行。在这段代码中，使用了中断 `10h` 来获取目前的光标位置，然后显示信息。

从第 3 行到第 5 行用于得到目前光标的位置，在此中断 `10h` 选用的是功能 3。然后，清除了 `bh` 寄存器的内容，并把字符串送至 `ch` 寄存器中。在 `bx` 中，我们送入了页码及显示的属性。此处，我们想要在黑背景上显示白色的字符。然后，把要显示字符的地址送到 `bp` 之中，信息由两个字节组成，其值分别为 13 的 10，它们分别对应回车和 LF(换行)的 ASCⅡ值。接下来是一个由 29 个字符组成的串；在下面实现的功能是输出字符串然后移动光标；最后是调用中断，然后进入循环。

### C 程序代码

C 程序的源代码如下所示，将其存储为 `write.c` 文件。

```c
#include <sys/types.h> /* unistd.h needs this */

#include <unistd.h> /* contains read/write */

#include <fcntl.h>

int main()

{

char boot_buf[512];

int floppy_desc, file_desc;

file_desc = open(“./bsect”, O_RDONLY);

read(file_desc, boot_buf, 510);

close(file_desc);

boot_buf[510] = 0x55;

boot_buf[511] = 0xaa;

floppy_desc = open(“/dev/fd0”, O_RDWR);

lseek(floppy_desc, 0, SEEK_SET);

write(floppy_desc, boot_buf, 512);

file_desc = open(“./sect2”, O_RDONLY);

read(file_desc, boot_buf, 512);

close(file_desc);

lseek(floppy_desc, 512, SEEK_SET);

write(floppy_desc, boot_buf, 512);

close(floppy_desc);

}
```

在上一期中，我曾经介绍过如何操作能启动的软盘。现在这一个过程稍微有点不同，首先把由 `bsect.s` 编译出来的可执行文件 `bsect` 拷贝至软盘的启动扇区。然后再把由 `sect2.s` 产生的可执行文件 `sect2` 拷贝至软盘的第二个扇区。

把上述文件置于同一目录之下，然后分别对其进行编译，方法如下所示：

```
as86 bsect.s -o bsect.o

ld86 -d bsect.o -o bsect
```

对 `sect2.s` 文件重复以上的操作，得出可执行文件 `sect2`。编译 `write.c`，插入软盘后执行 `write` 文件，命令如下所示：

```
cc write.c -o write

./write
```

## 下一步我们要做的事情

从软盘启动以后，可以看到显示出来的字符串。这是使用了 BIOS 中断来完成的。下一期要做的事情是在这个操作系统中实现实模式向保护模式的转换。

现在，这个操作系统已经越来越接近当年 Linus Torvalds 的那个具有“历史意义”的 Linux 内核了。因此，要马上把这个系统切换到保护模式之下。

## 什么是保护模式



自从 1969 年推出第一个微处理器以来，Intel 处理器就在不断地更新换代，从 `8086`、`8088`、`80286`，到 `80386`、`80486`、奔腾、奔腾Ⅱ、奔腾 4 等，其体系结构也在不断变化。`80386` 以后，提供了一些新的功能，弥补了 `8086` 的一些缺陷。这其中包括内存保护、多任务及使用 640KB 以上的内存等，并仍然保持和 `8086` 家族的兼容性。也就是说 `80386` 仍然具备了 `8086` 和 `80286` 的所有功能，但是在功能上有了很大的增强。早期的处理器是工作在实模式之下的，`80286` 以后引入了保护模式，而在 `80386` 以后保护模式又进行了很大的改进。在 `80386` 中，保护模式为程序员提供了更好的保护，提供了更多的内存。事实上，保护模式的目的不是为了保护程序，而是要保护程序以外的所有程序（包括操作系统）。

简言之，保护模式是处理器的一种最自然的模式。在这种模式下，处理器的所有指令及体系结构的所有特色都是可用的，并且能够达到最高的性能。

### 保护模式和实模式

从表面上看，保护模式和实模式并没有太大的区别，二者都使用了内存段、中断和设备驱动来处理硬件，但二者有很多不同之处。我们知道，在实模式中内存被划分成段，每个段的大小为 64KB，而这样的段地址可以用 16 位来表示。内存段的处理是通过和段寄存器相关联的内部机制来处理的，这些段寄存器（`CS`、`DS`、`SS` 和 `ES`）的内容形成了物理地址的一部分。具体来说，最终的物理地址是由 16 位的段地址和 16 位的段内偏移地址组成的。用公式表示为：

物理地址=左移 4 位的段地址+偏移地址。

在保护模式下，段是通过一系列被称之为“描述符表”的表所定义的。段寄存器存储的是指向这些表的指针。用于定义内存段的表有两种：全局描述符表(`GDT`)和局部描述符表(`LDT`)。`GDT` 是一个段描述符数组，其中包含所有应用程序都可以使用的基本描述符。在实模式中，段长是固定的(为 64KB)，而在保护模式中，段长是可变的，其最大可达 4GB。`LDT` 也是段描述符的一个数组。与 `GDT` 不同，`LDT` 是一个段，其中存放的是局部的、不需要全局共享的段描述符。每一个操作系统都必须定义一个 `GDT`，而每一个正在运行的任务都会有一个相应的 `LDT`。每一个描述符的长度是 8 个字节，格式如图 3 所示。当段寄存器被加载的时候，段基地址就会从相应的表入口获得。描述符的内容会被存储在一个程序员不可见的影像寄存器(`shadow register`)之中，以便下一次同一个段可以使用该信息而不用每次都到表中提取。物理地址由 16 位或者 32 位的偏移加上影像寄存器中的基址组成。实模式和保护模式的不同可以从图 1 和图 2 中很清楚地看出来。

图 1 实模式的寻址

图 2 保护模式下的寻址

图 3 段描述符的格式

此外，还有一个中断描述符表(`IDT`)。这些中断描述符会告诉处理器到那里可以找到中断处理程序。和实模式一样，每一个中断都有一个入口，但是这些入口的格式却完全不同。因为在切换到保护模式的过程中没有使用到 `IDT`，所以在此就不多做介绍了。

### 进入保护模式

`80386` 有 4 个 32 位控制寄存器，名字分别为 `CR0`、`CR1`、`CR2` 和 `CR3`。`CR1` 是保留在未来处理器中使用的，在 `80386` 中没有定义。`CR0` 包含系统的控制标志，用于控制处理器的操作模式和状态。`CR2` 和 `CR3` 是用于控制分页机制的。在此，我们关注的是 `CR0` 寄存器的 `PE` 位控制，它负责实模式和保护模式之间的切换。当 `PE=1` 时，说明处理器运行于保护模式之下，其采用的段机制和前面所述的相应内容对应。如果 `PE=0`，那么处理器就工作在实模式之下。

切换到保护模式，实际就是把 `PE` 位置为 1。为了把系统切换到保护模式，还要做一些其它的事情。程序必须要对系统的段寄存器和控制寄存器进行初始化。把 `PE` 位置 1 后，还要执行跳转指令。过程简述如下：

1. 创建 `GDT` 表;
2. 通过置 `PE` 位为 1 进入保护模式;
3. 执行跳转以清除在实模式下读取的任何指令。

下面使用代码来实现这个切换过程。

需要的东西：

- 一张空白软盘
- `NASM` 编译器

下面是整个程序的源代码：

```nasm
org 0x07c00; 起始地址是 0000:7c00 

jmp short begin_boot ; 跳过其它的数据，跳转到引导程序的开始处

bootmesg db "Our OS boot sector loading ……"

pm_mesg db "Switching to protected mode …."

dw 512 ; 每一扇区的字节数

db 1 ; 每一簇的扇区数

dw 1 ; 保留的扇区号

db 2

dw 0x00e0 

dw 0x0b40 

db 0x0f0 

dw 9 

dw 18 

dw 2 ; 读写扇区号

dw 0 ; 隐藏扇区号

print_mesg :

mov ah,0x13 ; 使用中断 10h 的功能 13，在屏幕上写一个字符串

mov al,0x00 ; 决定调用函数后光标所处的位置

mov bx,0x0007 ; 设置显示属性

mov cx,0x20 ; 在此字符串长度为 32 

mov dx,0x0000 ; 光标的起始行和列

int 0x10 ; 调用 BIOS 的中断 10h 

ret ; 返回调用程序

get_key :

mov ah,0x00 

int 0x16 ; Get_key 使用中断 16h 的功能 0，读取下一个字符 

ret

clrscr :

mov ax,0x0600 ; 使用中断 10h 的功能 6，实现卷屏，如果 al=0 则清屏

mov cx,0x0000 ; 清屏

mov dx,0x174f ; 卷屏至 23，79

mov bh,0 ; 使用颜色 0 来填充

int 0x10 ; 调用 10h 中断 

ret

begin_boot :

call clrscr ; 先清屏

mov bp,bootmesg ; 提供串地址

call print_mesg ; 输出信息

call get_key ; 等待用户按下任一键

bits 16

call clrscr ; 清屏

mov ax,0xb800 ; 使 gs 指向显示内存

mov gs,ax ; 在实模式下显示一个棕色的 A

mov word [gs:0],0x641 ; 显示

call get_key ; 调用 Get_key 等待用户按下任一键

mov bp,pm_mesg ; 设置串指针

call print_mesg ; 调用 print_mesg 子程序

call get_key ; 等待按键

call clrscr ; 清屏

cli ; 关中断

lgdt[gdtr] ; 加载 GDT 

mov eax,cr0 

or al,0x01 ; 设置保护模式位

mov cr0,eax ; 将更改后的字送至控制寄存器中

jmp codesel:go_pm

bits 32

go_pm : 

mov ax,datasel 

mov ds,ax ; 初始化 ds 和 es，使其指向数据段

mov es,ax 

mov ax,videosel ; 初始化 gs，使其指向显示内存

mov gs,ax 

mov word [gs:0],0x741 ; 在保护模式下显示一个白色的字符 A

spin : jmp spin ; 循环

bits 16

gdtr :

dw gdt_end-gdt-1 ; gdt 的长度 

dd gdt ; gdt 的物理地址 

gdt

nullsel equ $-gdt ; $指向当前位置，所以 nullsel = 0h 

gdt0 ; 空描述符

dd 0 

dd 0 ; 所有的段描述符都是 64 位的 

codesel equ $-gdt ; 这是 8h 也就是 gdt 的第二个描述符 

code_gdt 

dw 0x0ffff ; 段描述符的界限是 4Gb 

dw 0x0000 

db 0x00 

db 0x09a 

db 0x0cf 

db 0x00 

datasel equ $-gdt 

data_gdt 

dw 0x0ffff 

dw 0x0000 

db 0x00 

db 0x092

db 0x0cf

db 0x00

videosel equ $-gdt 

dw 3999 

dw 0x8000 ; 基址是 0xb8000

db 0x0b

db 0x92 

db 0x00 

db 0x00

gdt_end

times 510-($-$$) db 0 

dw 0x0aa55 
```

把上面的代码存在一个名为 `abc.asm` 的文件之中，使用命令 `nasm abc.asm`，将得出一个名为 `abc` 的文件。然后插入软盘，输入命令：`dd if=abc of=/dev/fd0`。该命令将把文件 `abc` 写入到软盘的第一扇区之中。然后重新启动系统，就会看到如下的信息：

- `*Our os booting…………….`
- `* A (棕色)`
- `* Switching to protected mode….`
- `* A (白色)`

### 对代码的解释

上面给出了所有的代码，下面我对上述代码做一些解释。

#### 使用的函数

下面是代码中一些函数的说明：

`print_mesg` 该子程序使用了 BIOS 中断 10h 的功能 13h，即向屏幕写一字符串。属性控制是通过向一些寄存器中送入不同的值来实现的。中断 10h 是用于各种字符串操作，我们把子功能号 13h 送到 `ah` 中，用于指明要打印一个字符串。`al` 寄存器中的 0 说明了光标返回的起始位置，0 表示调用函数后光标返回到下一行的行首。如果 `al` 为 1 则表示光标位于最后一个字符处。

显存被分成了几页，在同一时刻只能显示其中的一页。`bh` 指明的是页号；`bl` 则指明要显示字符的颜色；`cx` 指明要显示字符串的长度；`dx` 指明光标的位置(即起始的行和列)。所有相关寄存器初始化完成以后，就可以调用 BIOS 中断 10h 了。

`get_key` 使用中断 16h 的子功能 00h，从屏幕得到下一个字符。

`clrscr` 该函数使用了中断 10h 的另外一个子功能 06h，用于输出开始前清屏。初始化时给 `al` 中送入 0。寄存器 `cx` 和 `dx` 指明要清屏的屏幕范围，在本例中是整个屏幕。寄存器 `bh` 指明屏幕填充的颜色，在本例中是黑色。

#### 其它内容

程序一开始是一条短跳转指令，跳到 `begin_boot` 处。在实模式下，在此打印一个棕色的“A”，并且设置一个 `GDT`。切换到保护模式，并且打印一个白色的“A”。这两种模式使用的都是自己的寻址方法。

在实模式下，使用段寄存器 `gs` 指示显存位置，我们使用的是 `CGA` 显卡(默认基址是 0xb8000)。在代码中是不是漏了一个 0 呢？没有，因为实模式下会提供一个附加的 0。这种方式也被 `80386` 继承下来了。`A` 的 ASCII 是 0x41，0x06 指明了需要一个棕色的字符。该显示会一直持续直至按下任意键。下面要在屏幕上显示一句话，告诉使用者下面马上要进入保护模式了。

启动到保护模式，在进行切换时不希望此时有中断的影响，故要关闭所有的中断(使用 `cli` 来实现)。然后对 `GDT` 初始化。在整个切换过程中，对 4 个描述符进行了初始化。这些描述符对代码段(`code_gdt`)、数据和堆栈段(`data_gdt`)，以及为了访问显存而对显示段进行初始化。此外，还会对一个空描述符进行初始化。

`GDT` 的基址要加载至 `GDTR` 系统寄存器之中。`gdtr` 段的第一个字加载的是 `GDT` 的大小，在下一个双字中则加载的是基址。然后，`lgdt` 指令把把 `gdt` 段加载至 `GDTR` 寄存器中。现在已经做好了切换到保护模式前的所有准备。最后一件事情就是把 `CR0` 寄存器的 `PE` 位置 1。不过，即使这样还没有处于保护模式状态之下。

设置了 `PE` 位以后，还需要通过执行 `JMP` 指令来清除处理器指令预取队列。在 `80386` 中，使用指令前总是先将其从内存中取出，并且进行解码和寻址。然而，当进入保护模式以后，预取指令信息(它还处于实地址模式)就无效了。使用 `JMP` 指令的目的就是强迫处理器放弃无效的信息。

现在，已经在保护模式下了。那么，如何检测是在保护模式状态之下呢？让我们来看一看屏幕上这个白色的字母 `A`。在这里，使用了数据段选择符(`datasel`)对数据段和附加段进行了初始化，使用显示段选择符(`videosel`)对 `gs` 进行了初始化。显示的字符“A”其 ASCII 值和属性位于`[gs:0000]`处，也就是 `b8000:0000` 处。循环语句使得该字符一直在屏幕上显示，直至重新启动系统。

### 下一步要做的事

现在，这个操作系统已经工作在保护模式下了，但是实际上它并不实现什么具体的功能。你可以在这个基础上为它增加各种操作系统所具有的功能。我们自己动手写操作系统到此也就告一段落。

4dd466d0e1e0cc123afbe504da2462ca
