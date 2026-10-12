
# 读书笔记 DOS 下可执行文件的加载

> 原文：[`www.pediy.com/kssd/pediy10/77282.html`](https://www.pediy.com/kssd/pediy10/77282.html)

关于我：[`hi.baidu.com/dp282074009/blog...70df4d248.html`](http://hi.baidu.com/dp282074009/blog/item/4b6b92828c715aa70df4d248.html)  

本文旨在介绍 DOS 的内存管理与可执行文件的加载的原理。  

在学习本篇文章之前，你有必要了解 DOS 下 `.EXE` 文件的结构。我的另一篇文章《读书笔记重定位》中有关于 DOS 下 `.EXE` 文件的介绍。

## 1．PC 微机的地址空间分布

学习内存管理，则不得不对 PC 微机的地址空间分布有所了解。在不考虑虚拟存储空间的情况下，内存地址空间的分布大致如图：

![](img/2f9dd21df953e7939c2b2087d3ceb151.jpg)

在实模式下，8086/8088 有 1021KB 的内存地址空间，80x86 有 1088KB 的内存空间。注意，这里提及的，都是地址空间，而并非物理地址。地址空间指处理器的地址线所能表示的寻址范围。而地址空间可及，并不代表物理地址可及，这得依赖于系统中安装内存的大小。

由于微机的设计与 CPU 类型的限制，实模式下并不是所有空间对于用户来说都是可用的。地址空间被划分为四个区域，见上图。DOS 用户程序只运行于低端 640KB 连续内存区，而且 DOS 本身以及各种驻留程序的占用，用户程序的空间将小于 640KB。其中最重要的三大区域是：常规内存、上位内存、延伸内存。不同的区域有不同的用途，管理策略也不尽相同。

### 1.1 常规内存

低端的 640KB RAM 称为常规内存（Conventional Memory），也叫基本内存（Base Memory）。中断向量表、系统数据、通信区、DOS 本身以及 `CONFIG.SYS` 和 `AUTOEXEC.BAT` 文件中列出的设备驱动程序、TSR 程序都常驻在此区中，剩余的空间才是用户真正可以使用的（600KB 左右）。

DOS 存储管理的系统功能用于管理常规内存，主要是管理用户程序空间。采用单一连续可变分区管理方式。

### 1.2 上位内存

从 640KB，到 1M 的地址空间被称为上位内存（Upper Memory）。这些空间是为接口卡的数据缓冲区和 ROM-BIOS 等使用的。BIOS 自检时，也只检查 `A0000h` 地址以下的区域。

上位内存的的划分情况如下：

A0000~AFFFFh(610~704KB)    EGA 或 VGA 的 ROM 和数据区  
B0000~B7FFFh(704~736KB)    MDA 数据区  
B8000~C3FFFh(736~812KB)    CGA 或 EGA 使用  
            (736~752KB)    CGA 数据区  
            (736~784KB)    EGA ROM 区  
C4000~DFFFFh(812~896KB)    未用  
E0000~EFFFFh(896~960KB)    其他 ROM 用  
F0000~FFFFFh(960~1024KB)   ROM-BOIS 使用

上位内存中总存在一些闲置的地址空间，被称为上位内存块 UMB（Upper Memory Blocks）。它随系统配置的不同而不同，而且这些地址空间不宜用 RAM 芯片填充，故也被称为空洞（hole）。DOS 的相信管理功能不针对此部分内存。在 DOS 6.0 的 `EMM386.EXE` 能检查上位内存的“空洞”，并使用分布技术对“空洞”进行“回填”。

### 1.3 延伸内存

高于 1MB 的内存区域被称为延伸内存（Extended Memory），也称扩展内存。DOS 下不能直接使用此区域。对于 80x86 CPU，实上高位内存是可以直接访问的。

简单地介绍了内存中的各个区域后，下一节将学习 DOS 的内存管理。

## 2．内存管理

MS-DOS 的内存管理是通过一组系统功能调用实现的。它们属于单一连续区管理方式，面向单道程序设计。

### 2.1 MCB

DOS 实现内存管理的重要数据结构是内存控制块 MCB（Memory Control Block）。通过 MCB 把已分配和空闲的内存按位置顺序连成链。DOS 的内存块大小以“节”为单位，1 节等于 16 字节。内存块的边界也是节对齐的。每个内存块的前面都有 1 节长的区域头（area header），即内存控制块 MCB，MCB 控制内存块的大小与其他相关信息。

MCB 的结构如下：

![](img/f2227e13f5528c9933764eb481ce0a8c.jpg)

各个域的解释如下：

标记为“Z”（5Ah）表示最后一个内存块，为“M”（4Dh）表示为非最后块；  
内存块拥有者 0000h,内存块空闲；非 0 值，拥有此内存块程序的程序段前缀（PSP）段址。  
内存块大小以节为单位的内存块大小，不包括此 MCB 的长度。

### 2.2 MCB 链

若某 MCB 地址为 `XXXX：0000h`,则其控制的内存块段址为 `XXXX+1`，MCB 段址加上其 3~4 字节的内存块大小则是下一 MCB 的段址。即：

`MCB 段址+内存块大小+1＝下一 MCB 的段址`

于是，由一个 MCB 提供的信息，便可找到其后续的 MCB 的位置，系统的全部 MCB 顺序组成一个 MCB 链。

下图为 DOS 3.30 启动后 MCB 链的最初状况：

![](img/85625efde0a881ee44d6a93ab696109c.jpg)

由图可知，`COMMAND.COM` 常驻部分占用三个内存块，除一块供内部使用外，两个内存块一个是环境块，一个是程序（加数据）块。这两个内存块构成一个进程实体。`COMMAND.COM` 常驻部分之后是暂驻程序 TPA（Transient Program Area），即用户程序空间。它是最后一个内存块，未分配，大小近似为 600KB。块后是以 `A000：0000h` 开始的上位内存。注意，`COMMAND.COM` 暂驻部分位于 TPA 的高端，且无 MCB，故未单独构成一个内存块，可被用户程序覆盖。

内存块的分配策略一般为：在 MCB 链上找一个大于或等于请求分配长度的空闲块，然后将此内存块的 MCB 的“拥有者”域由 `0000h` 改为“系统当前 PSP 段址”。

到此为止，我们应该对微机的内存结构及 DOS 下的内存管理有了一个大体的了解。接下来将切入主题：DOS 下可执行文件的加载。

## 3．用户程序空间

有关 DOS 下 `.exe` 文件的结构及重定位的知识，请参见我的另一篇文章：【原创】读书笔记重定位。可执行文件加载时，DOS 为装入模块在 TPA 区域中分配一块连续的存储区，装入之后一次性地，即静态地完成全部的地址重定位工作。也就是说，运行于实地址模式下的 DOS 及其连接编译工具软件不支持分段存储和动态重定位。DOS 的可执行文件可分为 `.exe` 和 `.com` 两种。其中，`.com` 文件不超过 64KB,无重定位信息，装入后不需要重定位。

程序加载时，DOS 实际为用户程序分配两个内存块：一个内存块为环境块；另一个才是程序本身。在程序之前还有 256 字节的程序段前缀 PSP.两个内存块都有各自的 MCB。

### 3.1 .EXE 文件映象的加载

经连接器连接之后产生的 `.EXE` 文件是一个可执行的浮动代码文件。该文件在磁盘中由头部信息块与浮动装入模块组成。头部信息块用于指导加载器对装入模块的装入。在确定了装入模块的实际装入段基址 `XXXX` 后，程序的代码段和堆栈段段值均可确定。

在装入 `.EXE` 文件前，系统会检查 TPA 区中可用内存空间的容量与相当装入模块的大小。装入模块的大小由其头部 04~05h 域的总扇区数、02~03h 域的最后扇区有效字节数和 08~09h 域的头部信息块长度计算得出。装入内存可分为低端和高端装入：

![](img/92ba7c7d99215971b0acc4031b152515.jpg)

#### 低端装入

经连接器连接时的未使用 `/H` 选项，生成的 EXE 文件则要求在可用内存的低地址端开始装入，这是一般情况。此时申请人分配块长度，除装入模块本身大小和 PSP 的 `10h` 节长度之外，还要考虑装入模块之后应保留的节数。格式化区字节 `0A~0Bh` 域给出应保留节数的最小值，字节 `0C~0Dh` 域中给出最大值。前者是强制性因素。若可用内存空间的容量小于装入模块、PSP、最小保留节数三者之各，则不允许装入。`4Bh` 号系统调用出错返回。

如果装入成功，则如上图（a）和（b）。装入模块紧邻 PSP 之后，装入起点的段址等于分配块段址加上 `10h`。

#### 高端装入

经连接器连接时选择 `/H` 选项，则生成 `.EXE` 文件的格式化区字节 `0C~0Dh` 域中值为 0，表示要求在可用内存空间的高地址端装入。

只要可用内存空间容量不小于装入模块本身长度与 PSP 长度（`10h` 节）之和，即可装入、此时 `1Bh` 号系统功能把全部可用内存空间都分配给此 `.EXE` 文件。装入模块的最高地址与可用空间的最高地址相重合。但装入模块的最低地址与 PSP 之间可能有一个示使用的空闲区，如上图（c）所示。装入起点的段址等于分配块段址加分配块长减去装入模块长。

### 3.2 .COM 文件映象的加载

`.COM` 文件无头部信息块，只有一个无需重定位的、长度不大于 64KB 的装入模块（实际上不能大于 65280 字节）。`4Bh` 号系统功能装入 `.COM` 文件时也要测试当前可用内存空间容量，并且要求其容量大于 PSP 长度（`100h` 字节）与文件实际长度（采用实读方式来判断）之和，否则装入出错。

债台高筑装入的情况下，`4Bh` 号系统功能将全部可用内存空间都分配给 `.COM` 文件。如果可用内存空间容量大于 64KB，则也只是以其低端的 64KB 作为装入空间。于是高端有一个虽已分配但未使用的区域。当然，对于可用内存空间小于或等于 64KB 的情况下，则全部可用内存空间都作为装入空间了。两种情况 `.COM` 文件的内存映象如下图：

![](img/a1a4288810194fcf1f80e19b4fd65d70.jpg)

`.COM` 文件的装入起始位置总在装入空间预留出 `100h` 字节给 PSP，而且约定它也是程序启动点位置，即装入后令 `CS：IP` 指向它。因此装入模块的起始位置应是存放一条指令而不应是数据，此位置的偏移量为 `100h`.`4Bh` 号系统功能还总在装入空间的最高端地址减 2 的位置上写入全 0 的一个字，并令 `SS：SP` 指向这里。

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

`.EXE` 文件和 `.COM` 文件装入后，系统对各寄存器的设置情况如下表：

![](img/8ad1311cd05ec0980330bd0a01ca457c.jpg)

注意，`4Bh` 号系统功能还可只装入文件但不执行文件，此时将不设置上述这些寄存器。如果是装入并执行的话，`4Bh` 号系统功能还要在启动程序之前，修改分配给程序的两个内存块的 MCB 中的“拥有者”域为分配块的段址（即 PSP 段址），使被加载的程序真正成为两个内存块的主人，并使此 PSP 段址为“系统当前 PSP 段址”。

附个代码。![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑")

代码:



```
.586P

LOADER_SIZE		EQU		1024
PADDING_100h		EQU		100h

code_seg	segment	 use16

	org		PADDING_100h
startup:
	jmp		main

include int21.inc

Error_Str	 db	 'Error!'
Error_Str_Len	 Equ	 $-Error_Str

PrintStr	Proc	near	C	USES	AX	BX	CX	ES	BP,\\
	Color:BYTE,Position:WORD,\\
	StrAddrOffset:WORD,StrLen:WORD

	mov	 cx,StrLen
	mov	 bl,Color
	mov	 dx,Position

	push	 bp
	push	 StrAddrOffset
	pop	 bp
	
	mov	 bh,0
	mov	 al,0
	mov	 ah,13h
	int	 10h
	pop	 bp
	ret
PrintStr	endp


WaitKey	Proc	near	C	USES	AX,KeyCode:BYTE
	.repeat
		mov	 ah,0
		int	 16h
	.until	 al == KeyCode
	ret
WaitKey	endp

Relocate_And_Exec	Proc	near	C
	Local	 Reloc_Table_Addr:WORD
	Local	 Reloc_Item_Num:WORD
	Local	 Exe_Header_Size:WORD
	Local	 Exe_Entry_Point:WORD
	Local	 Reloc_CS:WORD
	Local	 Init_SS:WORD
	Local	 Init_SP:WORD
	
	call	Current_Ip
Current_Ip:
	pop	 bx
	add	 bx,Exe_Start-Current_Ip
	
	.if	 word ptr[bx] !='ZM'	;find the exe file's magic number
	stc
	ret
	.endif
	mov	 word ptr[Reloc_CS],bx
	shr	 word ptr[Reloc_CS],4	;add reloc_cs with loader_size and psp size
	mov	 ax,cs
	add	 ax,word ptr[bx+8]	
	add	 word ptr[Reloc_CS],ax

	
	mov	 ax,word ptr[bx+18h]	;get the reloc_table address
	mov	 word ptr[Reloc_Table_Addr],ax
	
	mov	 ax,word  ptr[bx+6]	;get the reloc_item number
	mov	 word ptr[Reloc_Item_Num],ax

	mov	 ax,word ptr[bx+8]	;get the exe header_size/16
	shl	 ax,4		;X16
	mov	 word ptr[Exe_Header_Size],ax

	mov	 ax,word ptr[bx+0eh]
	add	 ax,word ptr[Reloc_CS]
	mov	 word ptr[Init_SS],ax

	mov	 ax,word ptr[bx+10h]
	mov	 word ptr[Init_SP],ax

	mov	 ax,word ptr[bx+14h]
	mov	 word ptr[Exe_Entry_Point],ax

	xor	 cx,cx
	.while	 cx < word ptr[Reloc_Item_Num]
	mov	 si,cx		
	shl	 si,2		;reloc_item size is 4 bytes
	add	 si,Reloc_Table_Addr
	mov	 di,word ptr[bx+si+2]	;cs
	shl	 di,4	
	add	 di,word ptr[bx+si]	;ip
	
	add	 di,bx
	add	 di,word ptr[Exe_Header_Size]
	
	mov	 ax,word ptr[Reloc_CS]
	add	 word ptr[di],ax
	inc	 cx
	.endw

	;mov	 si,0
	;mov	 di,LOADER_SIZE
	;add	 di,word ptr[Exe_Header_Size]
	;sub	 di,100h
	;mov	 cx,100h
	;cld
	;rep	 movsb	[di],[si]

	mov	 ax,word ptr[Reloc_CS]
	
	mov	 bx,word ptr[Exe_Entry_Point]
	mov	 cx,word ptr[Init_SS]
	mov	 dx,word ptr[Init_SP]
	
	
	mov	 ds,ax
	mov	 es,ax

	mov	 ss,cx
	mov	 sp,dx
	
	
	push	 ax	;cs
	push	 bx	;ip
	retf
Relocate_And_Exec	endp
main:
	mov	 ax,cs
	mov	 ds,ax
	mov	 es,ax
	mov	 ss,ax
	mov	 sp,0fffeh
	
	push	 ds

	push	 0
	pop	 ds
	
	mov	 word ptr ds:[21h*4],Int21_Handler
	mov	 word ptr ds:[21h*4+2],cs
	pop	 ds
	
	
	invoke	 Relocate_And_Exec
	.if	 Carry?
	invoke	 PrintStr,57h,0,offset Error_Str,Error_Str_Len
	int	 21h
	.endif
	invoke	 WaitKey,'n'
ret
;---------------------------
padding_size	 equ	$-startup
padding	 db	 LOADER_SIZE-padding_size dup(0)
Exe_Start:
code_seg ends
end startup
```

详细请看

[`hi.baidu.com/quick1/blog/item...4d539c916.html`](http://hi.baidu.com/quick1/blog/item/9eb0f9ea6aaad8d4d539c916.html)
