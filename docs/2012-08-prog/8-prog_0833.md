
# 基于 BIOS 的开机口令设计与实现【讨论】

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-587.htm`](https://www.pediy.com/kssd/pediy09/pediy09-587.htm)

> 基于 BIOS 的开机口令设计与实现  
>   
> 添加 BIOS 模块，提高系统安全性  

> 摘要：计算机病毒、后门程序层出不穷，系统安全让人防不胜防，可执行文件、数据、文档等等均成为攻击的目标，甚至 BIOS 也不能幸免成为黑客攻击的对象。本文将介绍 BIOS 的基本结构，并实际编写 BIOS 模块写入 BIOS 芯片以提高系统安全性。同时让读者了解 BIOS 存在的风险，提高防范意识。  

> 目录  
> 一、前言  
> 二、所用工具  
> 三、BIOS 插入技术介绍  
> 四、完整演示代码  
> 五、编译与写入  
> 六、总结  
>   
> ======================================================================

## 一、前言

一直想写这份文档，可是日复一日，冗事缠身，趁元旦休假空闲时间赶紧写出来。刷新 BIOS 可能大部分人是从系统功能、性能升级（尤其是 CPU 超频等）的角度考虑，因为工作的关系我更多的从安全的角度来考虑，而且从这个角度进行 DIY 你会得到更多的挑战与乐趣。

另外，本文是写给信息安全初学者参考的，高手见笑了。有不妥之处，尽请批评指正。联系 email 及 MSN: `tianzhengmail@hotmail.com`。

## 二、所用工具

工欲善其事，必先利其器。在编写、刷新 BIOS 的过程中，需要一些成熟的工具。目前 BIOS 主要由 AWARD 和 AMI 两家公司提供，本文主要以 AWARD 公司的 BIOS 为例进行介绍（包括使用的工具及操作过程，主要原因是本人所用的实验机器为 AWARD 的 BIOS），AMI 的 BIOS 刷新过程与此差别较大，有机会的话我会专门写一份文档，不过原理都大同小异。

下面是必需准备的工具（具备这些工具就足够了）：

1. BIOS 读取程序，比如 `Award_WinFlash`，可以在 windows 操作系统下读取、写入 AWARD 公司的 BIOS；
2. BIOS 分析、修改程序，比如 `cbrom` 等；
3. 汇编编译器，比如 `FASM`；
4. 计算校验和的工具，比如 `Hex Workshop`；
5. 二进制文件修改程序，比如 `UltraEdit`，用来修改编译成功的二进制文件。

以上工具均可在互联网上下载使用。

另外，如果您希望快点看到自己的劳动成果，可以直接看第五部分，待实际操作一番后再回过头来看前面的原理介绍。

## 三、BIOS 插入技术介绍

### 1、BIOS 简介

BIOS，即 Basic Input/Output System（基本输入/输出系统），是计算机主板上的固定部件，由软硬件两部分组成（通常以 flash 芯片的形式存在，其上写有程序代码，连接在南桥芯片组上，如图 1 所示），主要为主板部件和主要外围设备服务。计算机在启动的时候首先运行 BIOS 程序对系统硬件进行初始化操作。处理器是计算机的心脏，BIOS 无疑是它的灵魂。编写插入定制的 BIOS 模块可以极大的扩展 PC 的功能，比如这个模块可以解锁厂商隐藏的功能、超频 CPU、可以使计算机防范入侵、防止病毒发作，同样也可以制作病毒，等等。

图 1 BIOS 芯片在主板上的位置逻辑图

### 2、BIOS 程序的组成模块

在以前的 AT 计算机里 BIOS 固件位于地址空间的最末端（`FFFF0000h`-`FFFFFFFFh`），使用 32 位 X86 系列 CPU 的计算机加电以后第一条指令从 `FFFFFFF0` 开始执行，因此 BIOS 固件中位于地址总线 `FFFFFFF0` 处一般为短跳转指令，跳转至上方 64Kbytes 处执行，这个模块被称为 `boot block`,由其完成 CPU 等硬件的初始化，并加载所有以打包形式保存的模块。目前 BIOS 固件大小从 256K 至 2M 不等，为了保证向后兼容，设计者将 BIOS 分成若干模块。图 2 为使用“`cbrom oldbios.bin /d`”命令解包得到的目前 BIOS 版本中的模块信息，共有 0-8 九个模块。每个模块的具体作用这里就不多讲了（讲清楚这个可能比这份文档还要长-^- ）。

### 3、改写 BIOS 模块的可行性

BIOS 的模块化组成使编写、插入定制模块成为可能，每个模块都有自己固定的进入点。总之既然是模块化结构，我们就可以编写自己的功能模块找准切入点将其加入到 BIOS 文件中以实现我们想要的功能。

图 2 BIOS 组成模块示意图

## 四、完整演示代码

下面讲怎样编写定制的模块。

现在的主板大部分都不提供 ISA 控制器及其插槽了，但 BIOS 为兼容起见仍然保留了 ISA 模块的空间（也可能是编写 BIOS 程序的程序员不想修改程序以避免头痛的调试、测试，呵呵）。

ISA 模块以单独的二进制文件形式存在，大小是 `200h` 的整数倍；它以标准的 `55 AA` 两个字节开始，最后的字节是校验和，校验和字节的目的是使所有字节的校验和为 0。

下面是一个简单的 ISA 模块代码，使用 `FASM` 编译。系统启动时，这个模块显示一句欢迎词“Do not Crack！”，然后等待用户输入密码，按<Enter>键表示输入完成。这个定制的模块为系统提供了增强的密码保护，除非更换 BIOS 芯片，否则没人能破解这个保护措施（手工暴力尝试除外，呵呵）。

;代码清单
; ISAOEM.ASM
use16      ; ISA module operates in the 16-bit segment
DB  55h, 0AAh  ; ISA 模块开始的标志
DB  01h      ;此模块占用 200hBytes * 1 = 200hBytes
jmp  x_code    ;
x_code:
 ; Preparing registers
 ; --------------------
 MOV DX, 101Dh  ; Where to output (DH - Y, DL - X)
 MOV SI, text  ; What to output
 XOR BX,BX  ; Initial color - 1
 MOV CX,1  ; Output one character at a time

 ; Output a colored string
 ; --------------------
print_string:
;// ah=2 bh=display page dh=row dl=col
 MOV AH, 02h  ; Function for controlling the cursor
 INT 10h    ; Position the cursor
 INC DL    ; Move to the next position

 LODSB    ; Load the next character to AL
 TEST AL,AL  ; End of line?
 JZ myinput0         ; Exit if yes
;//ah=9 al=character bh=page number bl=attribute cx=number of character
;//to write
 MOV AH, 09h  ; Function for character output
 INC BL    ; Use all colors sequentially
 INT 10h    ; Pring the character
 JMP print_string     ; Loop

;//myPasswordCheck
;提供五位口令的保护
myinput0:
 xor  dx,dx  ;记录校验值，不为 0 时口令不正确 ；
 xor  bx,bx  ;统计输入密码的位数
myinput1:   
 xor  ax,ax
 int  16h
 cmp  al,0dh  ;是否输入回车
 jz  check
 inc  bx
 cmp  al,'m'
 jz  myinput2
 inc  dx
myinput2:
 xor  ax,ax
 int  16h
 cmp  al,0dh
 jz  check
 inc  bx
 cmp  al,'a'
 jz  myinput3
 inc  dx
myinput3:   
 xor  ax,ax
 int  16h
 cmp  al,0dh
 jz  check
 inc  bx
 cmp  al,'r'
 jz  myinput4
 inc  dx
myinput4:   
 xor  ax,ax
 int  16h
 cmp  al,0dh
 jz  check
 inc  bx
 cmp  al,'t'
 jz  myinput5
 inc  dx
myinput5:
 xor  ax,ax
 int  16h
 cmp  al,0dh
 jz  check
 inc  bx
 cmp  al,'i'
 jz  myinput6
 inc  dx
myinput6:
 xor  ax,ax
 int  16h
 cmp  al,0dh
 jz  check
 inc  bx
 cmp  al,'x'
 jz  myinput7
 inc  dx
myinput7:
 xor  ax,ax
 int  16h
 cmp  al,0dh  ;是否输入回车
 jnz  myinput0
   


check:   
 cmp  bx,06h
 jnz  myinput0
 test  dx,dx
 jnz  myinput0  ;重新开始
 retf
   
text db "Do not Crack!",0
;代码结束
---------------------------------------------------------------------

## 五、编译与写入

迫不及待了吧，下面我们就来介绍实际的操作过程，让你得到前面描述的额外的密码保护功能。

### 1、读取 BIOS 内容

利用 `Award_WinFlash` 读取 BIOS 内容，并另存为 `oldbios.bin`，如图 3 所示(注意文件名长度需在 8 个字符以内，否则进行第 2 步的操作时可能会报错)。如果凑巧你所使用主板上的 BIOS 不是 AWORD 的，程序会提示；你最好找一台使用 AWARD 公司 BIOS 的计算机来继续下面的操作。

图 3 使用 WinFlash 读取现有 BIOS

### 2、分析读取的 BIOS 内容,选择插入点

利用 `CBROM` 解包上一步得到的 BIOS 文件包 `oldbios.bin`,在命令行方式下进入你的 `cbrom` 所在目录，使用命令“`cbrom.exe oldbios.bin /d`”命令得到图 2 的结果（注意选择适合你的 BIOS 版本的 `cbrom` 工具，`cbrom` 主要有 2.07 和 6.06 两个版本，一个不行你可以尝试使用另一个）；从图 2 中可以看到目前的 BIOS 中没有 ISA 模块，呵呵，那我们就给他插入一个 ISA 模块吧。现在大部分主板上都不会有 ISA 插槽，因此你可以在 BIOS 中放心的使用 ISA 模块。

### 3、编译插入的程序

按照第四部分的介绍编写 ISA 模块的 BIOS 代码，实现需要的功能（最简单的方法是什么都不要改动，直接使用上面介绍的例子程序，当然你也可以更换一个你自己喜欢的提示界面和只有自己知道的口令，但这样你在进行下面第 4 步操作时就要小心了），并使用 `FASM` 编译器编译生成 `isaoem.bin` 文件，所用命令为“`fasm.exe isaoem.asm  isaoem.bin`”。

### 4、在旧 BIOS 文件中插入 isaoem.bin

现在到了最关键的时刻了，注意下面的操作：得到 `isaoem.bin` 以后，还不能马上进行插入操作，BIOS 文件对各个模块都有大小和校验的要求，我们插入的 ISA 模块大小应为 `200h` 字节（0.5KB）,校验和必须为 0;因此我们先利用二进制文件编辑工具（比如 `UltraEdit`）编辑 `isaoem.bin`，在其尾部添加若干“00”，直到其文件总长为 `200h` 字节，然后利用 `Hex Workshop` 工具生成校验和，第四部分给出的代码的校验和为 `86H`，因此最后一个字节应该是 `100h - 86h = 7Ah`，把它写入 `isaoem.bin` 的 `1ffh` 偏移处。嗯，差不多了，下面是最后一步，将 `isaoem.bin` 模块插入到 BIOS 中，利用如下命令进行：“`cbrom oldbios.bin /isa  isaoem.bin`”,将得到的文件改名为 `newbios.bin`。

### 5、检查插入结果，利用写入工具刷新 BIOS

好，现在回过头来仔细检查你的每一步操作，确保没有任何差错，否则不要进行下面的冒险。

确定自己没有犯任何错误以后，该是享受成果的时候了。输入“`cbrom newbios.bin  /d`”查看新得到的 BIOS 模块信息，如图 4 所示，显示了我们新插入的 ISA 模块。

现在利用 `WinFlash` 工具将 `newbios.bin` 刷入计算机 BIOS 芯片中。

图 4 添加 ISA 模块后的 BIOS 文件

### 6、重新启动机器，检验结果

根据提示重新启动计算机，如果前面没有犯任何错误的话，那么幸运的你现在应该能在屏幕中间看到类似于图 6 所示的界面，输入口令，然后一切正常。

## 六、总结

其实，通过编写定制的 BIOS 模块还可以实现很多功能，比如可以编写传说很久的 BIOS 病毒或 RootKit，同样可以编写对抗他们的模块，这全都掌握在你的手里，BIOS 里还有很多有趣的内容值得你去研究，enjoy it：）

## 参考文献

1. 《shellcoder programming uncovered》2005 年版 Kris Kaspersky 著
2. [`www.biosmods.com`](http://www.biosmods.com)   专门讲修改 BIOS 的网站 

