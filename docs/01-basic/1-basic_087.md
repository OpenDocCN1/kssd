
# 386 保护模式【学习一、二、三、四】

> 原文：[`www.pediy.com/kssd/pediy10/62548.html`](https://www.pediy.com/kssd/pediy10/62548.html)

看雪的朋友门我来鸟。  

在编程区里:看给位大侠将的什么中断,内核，听得我直发毛。  

因为有很多专业词汇，内核知识。。从来没听过不知道大侠们在讲什么!!!  

那么本人不甘心。我就从头开始学吧  

..现在那点笔记出来跟大家分享一下!!!  

由于是笔记，也许记得有点糟糕！！  

呵呵,来看雪总想发点东西,!!请见谅啊!!!  

## 【保护模式编程、一】

### 【8086 模式编程】

如果想更深、更亲近的了解电脑软件。  
那么学习 cpu 是你的必选!!  

`386` 是 CPU 史的一大转折点,那 `386` 做基础课是最好不过了。那么我们将开始进行学习之旅！！！  

大家跟我一块学习吧,呵呵!!!  

### 一、准备工作

1、`NASM` 编译环境  

2、虚拟机  
`Virtual PC`(Windows 平台 ,执行比较快,即模拟又虚拟硬件)、  
`WMWarve`(WIndows 平台 虚拟硬件,)、  
`Bochs`(支持 Windows 平台、也支持在 Linux 平台上运行 有 RPM 版本的)  

我们这些生长在 Windows 这棵大树下的朋友们,还是用 `Virtual PC` 吧。。  

3、写虚拟启动镜像文件的程序  

:不知道我观察的对不对？  

用 `Nasm` 编译一个 `bin` 然后将它转换为 `img` 镜像文件的时候。  
只要文件大小符合软驱的标准就能启动。那么就代表 `a.bin` 与 `a.img` 文件的内容一模样就是文件大小不一样!  

我是不太了解镜像文件格式.我用的是 `Virtual PC`。  

### 二、开始接触引导程序

1、`Com` 文件  

`Com` 文件是纯二进制的文件，也是直接与 `Cpu` 交换的顺序指令文件。  

`Com` 文件的大小是有限制的，不能超过 `64KB`.  

因为 `8086` 时代的 `CPU` 地址线是 `20` 位的,20 位能表达的数值也就是 `ffffffh`(1MB )  
而寄存器最高也只是 `16` 位,无法用 `5` 个 `F` 的形式来表达地址，所以用 `CS`(段基地址)`*16:IP`(偏移地址)来寻址!  

`80386` 后通用寄存器都得到了 `32` 位扩展! 而 `Cpu` 地址线也得到了 `32` 位的扩展。  

引导程序前期是需要进入实模式的，因为这是硬件上的限制是 `IA32` 的限制。  

`386cpu` 只有两种模式: 实模式与保护模式!!!!,,  

2、引导程序  

引导程序也是有限制的,这个限制是靠 `Bios` 处理的,  
开机后 `Bios` 经过自检后，会从软驱或者硬盘的 `0` 面 `0` 磁道 `1` 扇区搜寻一个程序文件。  
该文件的数据必需是等于 `512Byte`,并且以 `aa55h` 结尾的(高高低低)。那么 `bios` 会认为它是引导程序,  
这个时候就会把该 `512byte` 装载到内存 `7c00` 开始处。然后将主控权交给程序的第一行代码。  
那么这个时候程序脱离 `Bios` 的控制。`Cpu` 将执行程序的代码.  

### 三、写一个引导程序

引导程序可以说是非常简单:  

1、`boot.asm`(`nasm` 的源文件如下)

```asm
;-----------------------欲编译,这里改成 100h 就是 com 程序 -------------------------------------------
%define \_BOOT\_DEBUG\_  ;做调试的时候用 100h
%ifdef \_BOOT\_DEBUG\_
org 0100h
%else
org 07c00h    ; 告诉编译器 以下代码段将从 07c00h 内存地址处开始
%endif
mov ax,cs             ;让数据段与附加段寄存器跟代码段一样，因为 COM 代码数据是混合.
mov ds,ax
mov es,ax
call \_HelloWorld        ;让程序显示一个 HelloWorld
jmp $                  ;$表示当前地址 无限循环
\_HelloWorld:
mov ax,strHello        ;取得字符串的地址
mov bp,ax              ;给堆栈基寄存器
mov cx,strLen
mov ax,1301H       ;ah 代表功能号
mov bx,000ch   ; 页号为 0(BH = 0) 黑底红字(BL = 0Ch,高亮
mov dx,0001h      ;显示的行与列
int 10h            ;bios 10h 显示中断
ret
strHello:     db "Hello World"
strLen     equ $ - strHello
times 510-($-$$) db 0 ; times 重复定义 510-($-$$)个 $$ 表示段的起始地址
dw 0xAA55
```

那么引导程序完成了,  

用 `nasm boot.asm -o a.com`  
就可以运行看效果.,  

如改成引导程序只需把`%define \_DEBUG\_BOOT\_` 注释 然后 `nasm boot.asm -o a.bin`  
然后用工具将 `a.bin` 转换成软驱大小的镜像文件 载入虚拟机启动就可以.  

## 【保护模式编程、二】

### 【80386 保护模式编程】

`8086` 到 `80386` 的跳转,`80386` 与 `8086` 在硬件上的区别在这就不说了!!  

那么 `80386` 与 `8086` 在软件逻辑上面的区别就是:  

`8086` 是实模式，而 `80386` 不仅包括实模式，而且还可以进入保护模式!!!  

保护模式不仅不受 `64KB` 内存寻址的限制,而且还拥有 `4GB` 的寻址空间。这是因为 `386` 扩展了 `20` 地址线，将它扩展成 `32` 位了(`32` 位能表达的字节数就是 `4GB`).  

此时的段寄存器不再是段基地了,而被叫做是选择子 ,存放的是一个段描述符的索引值.  

而我们的通用寄存器与 `EIP` 也是 `32` 位的，可以表达 `4GB` 地址!  

不过计算机开机后，CPU 默认是实模式。这就需要我们编程手动转换到 `386`.  

那么我们该怎么去做呢:  

### 一、【准备 GDT (全局描述符表)】

首先我们需要准备 `GDT` 结构体,它是 `386` 保护模式必须的东西。  

全局描述符寄存器 `GDTR` 指向的是所有段描述符表的信息.  

前面提到得段选择子索引，指得就是指向段描述符的索引.  

段描述符是 `8` 个字节的结构体、里面存放着段的段界限、段基址、段属性等信息.  

`LDTR` 寄存器是指向局部某一个段的描述符表。  

段描述符表结构体用一个宏来表示(注意段 1 2 表示同一个段描述内容被分开来放的):  

【段界限】`20` 位、表示段的总长度 这里并不是地址,而是段的字节长度。  
【段基址】`32` 位、表示物理地址  
【段属性】`12` 位. 系统、门、数据等属性  

```asm
%macro Descriptor 3     ; 有三个参数：【段界限】、【段基址】、【段属性】
dw %2 & 0FFFFh     ; 段界限 1     (2 字节)
dw %1 & 0FFFFh     ; 段基址 1     (2 字节)
db (%1 >> 16) & 0FFh    ; 段基址 1     (1 字节)
dw ((%2 >> 8) & 0F00h) | (%3 & 0F0FFh) ; 属性 1 + 段界限 2 + 属性 2   (2 字节)
db (%1 >> 24) & 0FFh    ; 段基址 2     (1 字节)
%endmacro ; 共 8 字节
```

看似很简单的结构体 理解起来可不是那么简单!  

【`Descriptor` 结构体】有 `8` 个字节。  

1、【第 `1`、`2` 字节】组合(`word`) 表示该段的`[段界限①]`, `dw %2 & 0FFFFh` ;引用第二个参数去掉高 `16` 位  

2、【第 `3`、`4`、`5` 字节】组合表示该段的`[段基址①]`, `dw %1 & 0FFFFh` ;先得到第一个参数(`段基址`)低 `WORD`.  

3、接着把第 `5` 个字节赋值,`db (%1 >> 16) & 0FFh` 去掉第 `3` 第 `4` 个字节的内容.再把剩下的字节赋值  

4、【第 `6` 个字节】是与【第 `7` 个字节】组合的内容可就更复杂了:  

【第 `6` 个字节】的内容:  

【`7(p) 6(DPL) 5(DPL) 4(S) 3(Type) 2(Type) 1(Type) 0(Type)`】  

`0-3` 位表示：`[段属性]`、说明存储段描述符所描述的存储段的具体属性。  
`4` 位表示：说明描述符的类型, 对于存储段描述符而言，`S=1` 表示是系统段描述符。  
`5-6` 位表示：`DPL` 该段的特权级别也就是 `Ring 0-3`;  
`7` 位表示：`P:` 存在(`Present`)位。  
`; P=1 表示描述符对地址转换是有效的，即描述的段在内存当中.`  
`; P=0 表示描述符对地址转换无效，即该段不存在。使用该描述符进行内存访问时会引起异常`  

【第 `7` 个字节】的内容:  

【`7(G) 6(D) 5(0 ) 4(AVL) 3(段界限) 2(段界限) 1(段界限) 0(段界限)`】  

`0-3` 位表示：`[段界限②]`  
`4` 位表示：软件可利用位。`80386` 对该位的使用未做规定，`Intel` 公司也保证今后开发生产的处理器只要与 `80386` 兼容，就不会对该位的使用做任何定义或规定。  
`5` 位表示：`0` ;`Intel` 资料也没表示  
`6` 位表示：是一个很特殊的位，在描述可执行段、向下扩展数据段或由 `SS` 寄存器寻址的段(`通常是堆栈段`)的三种描述符中的意义各不相同,通常置 `1`  
`7` 位表示： 段界限粒度(`Granularity`)位。  
`G=0` 表示界限粒度为字节；  
`G=1` 表示界限粒度为 `4K` 字节。  
注意，界限粒度只对段界限有效，对段基地址无效，段基地址总是以字节为单位。  

那么这段宏 `dw ((%2 >> 8) & 0F00h) | (%3 & 0F0FFh)`表示:  

取`[段界限]`参数除去低 `16` 位取 高 `4` 位,得到【段界限②】  
取`[段属性]`参数的低 `8` 位 `12-15` 位(`AVL` 属性等)  
属性 `1` + 段界限 `2` + 属性 `2`  

【第 `8` 个字节】的内容:  

`[段基址②]` 、`db (%1 >> 24) & 0FFh` 取基地址参数的最高 `8` 位  

那么一个 `Descriptor` 结构体就这样成形了.  

### 二、【编写程序跳转到保护模式】

```asm
%include "386.inc"         ;是 Descriptor 结构体宏
;%define \_DEBUG\_BOOT\_
%ifdef \_DEBUG\_BOOT\_
    org 0100h
%else
    org 07c00h
%endif
jmp LABEL\_BEGIN
[SECTION .gdt]          ;全局描述符数据段
                                     ; 段基址,      段界限     , 属性
LABEL\_GDT:     Descriptor        0,                0, 0         ;空描述符
LABEL\_DESC\_CODE32: Descriptor        0, SegCode32Len - 1, DA\_CR | DA\_32    ;代码段描述符  
LABEL\_DESC\_DATA:   Descriptor    0,SegDataLen - 1,DA\_DRW      ;数据段
LABEL\_DESC\_VIDEO:   Descriptor    0B8000h,0FFFFh,DA\_DRW      ;显示器内存段 由于 DOS 中断不能随意使用了，，只能输出到显示缓冲区
; GDT 结束
GdtLen equ       $ - 1
GdtPtr   dw GdtLen - 1     ;GDT 的段界限,
     dd   0      ;GDT 基地址
; GDT 选择子
SelectorCode32   equ LABEL\_DESC\_CODE32 - LABEL\_GDT     ;代码相对全局描述符起始地址的 EA 值
SelectorData   equ LABEL\_DESC\_DATA - LABEL\_GDT     ;数据段
SelectorVideo   equ LABEL\_DESC\_VIDEO - LABEL\_GDT     ;显示数据段
[SECTION .s16]     ;16 位代码段
[BITS 16]     ;BITS 指出处理器的模式 是 16 位
LABEL\_BEGIN:        
mov ax, cs
mov ds, ax
mov es, ax
mov ss, ax       ;初始化段寄存器
;初始化数据
mov eax,strHello
mov word[LABEL\_DESC\_DATA + 2],ax
mov byte[LABEL\_DESC\_DATA + 4],al
shr eax,16
mov byte[LABEL\_DESC\_DATA + 7],ah
; 初始化并把 32 位段代码的基地址分配给段描述符
mov eax, LABEL\_CODE32    
mov word [LABEL\_DESC\_CODE32 + 2], ax          ;ax  
shr eax, 16
mov byte [LABEL\_DESC\_CODE32 + 4], al
mov byte [LABEL\_DESC\_CODE32 + 7], ah
; 为加载 GDTR 作准备
mov eax, LABEL\_GDT  
mov dword [GdtPtr + 2], eax ;得到 GDT 基地址  
; 加载 全局描述符的信息结构体 到 GDTR  
lgdt [GdtPtr]
CA20    ;利用键盘端口打开 A20 地址线
; 将 CRO 的 PE 位 也就是 0 位 置 1 那么就进入 386 模式了
mov eax, cr0
or eax, 1
mov cr0, eax
jmp dword SelectorCode32:0 ;
;执行这一句会把 SelectorCode32 装入 cs, 并跳转到 Code32Selector:0 处
;这个描述符集合是以一个空描述符开始得,现在 LABEL\_DESC\_CODE32 描述符的索引值因该是 8，
;所以 SelectorCode32 的值应该就是 LABEL\_DESC\_CODE32 的索引值，Code32Selector:0 当中的 0 是指 LABEL\_DESC\_CODE32 的段基址+ 0
;那么在打开 cr0 的 PE 位后,这个 JMP 指令不再是直接跳到段地址去了;
;而是去 GDTR 全局描述符寄存器当中去找这个当前 CS 的索引,当前段基址+偏移 的内存地址了。
[SECTION .s32]; 32 位代码段. 由实模式跳入.
[BITS 32]
LABEL\_CODE32:
;保护模式的死循环
mov ax, SelectorVideo
mov gs, ax    ; 视频段选择子(目的)
mov edi, (80 * 10 + 9) * 2 ; 屏幕第 10 行, 第 0 列。
mov ah, 1Ch    ; 0000: 黑底    1100: 红字
mov esi,0
mov ds,SelectData
mov ecx,11
vi:
lodsb
mov [gs:edi], ax
inc edi
LOOPNZ vi
; 到此停止
jmp $
SegCode32Len equ $ - LABEL\_CODE32
[SECTION .data]    ;数据段
strHello:     db "Hello World"
SegDataLen   equ $- strHello
```

【总结】  

编写一个 `386` 程序主要用的步骤  

1、准备 `GDT` 描述符集合结构体  
2、用 `lgdt [gdtPtr]` 载入 `gdtPtr` 这 `6` 个字节的结构体，，低字是描述符集合的界限 也就是集合总长度，高双字是描述符集合的基地址.  
3、打开 `A20` 地址线  
有一种方法是向键盘端口 `IO`,  
4、置 `CR0` 的 `PE` 位 即 `0` 位为 `1`  
5、`JMP [段索引]:[段基址偏移]`  

呵呵 接下来继续学习啊!!!  

## 【继续 80386 编程】

在一、到二、我们了解 `386` 基本寻址机制，没错就是这么简单!!!  

接下来我们谈谈 对上一个 `386` 进行扩展:  

大家在第二节已经知道了进入 `386` 的基本步骤了,那么我们来具体设计吧.  

编程首先当然是【声明】与【定义】:  

### 一、【声明】：

在 `386.inc` 头文件里定义好需要的宏信息(好东西直接拿来用了呵呵)



;---------------------------------386.inc-----------------------------------------------------

`DA_32`    `EQU`   `4000h`   ;32 位段

`DA_DPL0`   `EQU`   `00h`  ; DPL = 0
`DA_DPL1`   `EQU`   `20h`  ; DPL = 1    (表示描述符特权级 Ring0-Ring3 级)
`DA_DPL2`   `EQU`   `40h`  ; DPL = 2
`DA_DPL3`   `EQU`   `60h`  ; DPL = 3

;----------------------------------------------------------------------------
; 存储段描述符类型值说明
;----------------------------------------------------------------------------

`DA_DR`    `EQU` `90h`  ; 存在的只读数据段类型值
`DA_DRW`   `EQU` `92h`  ; 存在的可读写数据段属性值
`DA_DRWA`  `EQU` `93h`  ; 存在的已访问可读写数据段类型值
`DA_C`     `EQU` `98h`  ; 存在的只执行代码段属性值
`DA_CR`    `EQU` `9Ah`  ; 存在的可执行可读代码段属性值
`DA_CCO`   `EQU` `9Ch`  ; 存在的只执行一致代码段属性值
`DA_CCOR`  `EQU` `9Eh`  ; 存在的可执行可读一致代码段属性值

;----------------------------------------------------------------------------
%macro `Descriptor` 3     ;3 表示宏的参数有 3 个 %1 表示是第一个参数的标识 >>右移位
dw `%2` & `0FFFFh`     ; 段界限 1     (2 字节)
dw `%1` & `0FFFFh`     ; 段基址 1     (2 字节)
db (`%1` >> 16) & `0FFh`    ; 段基址 2     (1 字节)
dw ((`%2` >> 8) & `0F00h`) | (`%3` & `0F0FFh`) ; 属性 1 + 段界限 2 + 属性 2   (2 字节)
db (`%1` >> 24) & `0FFh`    ; 段基址 3     (1 字节)
%endmacro ; 共 8 字节

%macro `CA20` 0   ; 打开地址线 A20
in al, 92h
or al, 00000010b
out 92h, al
%endmacro

%macro `DA20` 0        ;关地址线
in   al,92h
and al,11111101b
out 92h,al
%endmacro

## 二、【定义】

;在定义模块之前我们首先要有个概念，那就是整体的雏形。

可以简单划分出来也就是 2 个主要步骤：

### 1、定义 GDT 数据段
{
        1、定义 `Descriptor` 即段描述符： （通常是以一个全为零的 `Descriptor` 开始）。
       2、定义 `GdtPtr` 信息结构体（再加载 gdtr 时候要用到）。
       3、定义每个段描述符对应的索引位置（即定义选择子）。
}

### 2、定义如上描述的具体段
{
     1、实模式入口段 (这是个入口段：用于跳转到 386 的段，并不属于 386 段所以没有描述符）
    2、剩下的段就全是 386 模式的段。
}

## 三、【具体编码】

还是要拿实实在在的能运行的代码来讲：

先说一下代码的主要功能：

1、从 8086 跳到 386 模式(Protect Mode)
2、在 386 模式对大地址的寻址测试（超过 1MB）
3、测试完毕后回到 8086 模式（Real Mode）

具体细节上,就看代码吧!!!

在 `386.asm` 文件里实行具体模块的编写(代码比较多，刚开始阅读有点复杂，因为是汇编可读性不是很好
不过这是照上面的方法定义的，可以先从宏观入手!!!)：

```asm
;========================386.asm===============================

%include "386.inc"  ; 常量, 宏, 以及一些说明
%define   \_DEBUG\_B0OT\_
%ifdef   \_DEBUG\_B0OT\_
    org 0100h
%else
    org 07c00h
%endif

jmp LABEL_BEGIN

;===========================;GDT 全局描述符数据段==============================
[SECTION .gdt]
LABEL_GDT      Descriptor    0,0,0    ;以空开头
```

```asm
;这个段描述符描述的段有点特殊，因为在下面并没有实际的定义它。它的作用是从保护模式跳转到 8086 时要用到的,是用来初始化段寄存器的。保护模式与实模式段界限与段属性是不同的, 而这个段描述符的段基址是 0、段界限是 0FFFFH ,段属性是读加写，与 8086 的标准是一样,所以在回到 8086 模式之前，CPU 在对所有段寄存器进行实模式的转换就能正确安排界限与属性了,！！! ），

LABEL_DESC_NORMAL    Descriptor     0,0ffffh,DA_DRW         
LABEL_DESC_DATA    Descriptor    0,SegDataLen - 1 ,DA_DRW | DA_32    ;段属性是非一致的 32 位读写数据段
LABEL_DESC_STACK    Descriptor   0,TopOfStack - 1,DA_DRWA | DA_32    ;存在的已访问可读写的 32 位 stack 段 在保护模式下的 Call 命令需要堆栈
LABEL_DESC_CODE32    Descriptor   0,SegCode32Len -1 ,DA_C | DA_32    ;Protect mode 的 32 位代码区
```

```asm
;这个段是保护模式的段，但是它是以 16 位形式存放的，它是用来从保护模式跳回到 8086 模式。……因为直接用 32 位代码段跳转到 8086 模式是不行，必需从 16 位保护模式段跳转到 16 位 8086 模式。就像先前说的 Normal 描述符,它也是一个具有 8086 属性的描述符。，所以 CS 段寄存器的状态也需要先转换成与 8086 模式相同段界限与段属性。这样才能正确的转换到 8086 模式,由此可见:全部的段寄存器都需要对应 8086 模式的描述符状态。才能正常的进入 8086 模式。

LABEL_DESC_CODE16    Descriptor   0,0ffffh ,DA_C           
;以下两个段描述符是内存中的段不需要自己定义
LABEL_DESC_TEST    Descriptor   0500000h,0ffffh,DA_DRW        ;用于测试线性空间
LABEL_DESC_VIDEO    Descriptor   0B8000h,0ffffh,DA_DRW       ;显示器内存

GdtLen     equ   $ - LABEL_GDT
GdtPtr     dw   GdtLen - 1     ;段界限与段基地址  
       dd     0    
```

```asm
;----------------选择子---------------------
SelectorNormal   equ    LABEL_DESC_NORMAL - LABEL_GDT
SelectorData   equ    LABEL_DESC_DATA - LABEL_GDT
SelectorStack   equ    LABEL_DESC_STACK - LABEL_GDT
SelectorCode32   equ    LABEL_DESC_CODE32 - LABEL_GDT
SelectorCode16   equ    LABEL_DESC_CODE16 - LABEL_GDT
SelectorTest   equ    LABEL_DESC_TEST - LABEL_GDT
SelectorVideo   equ    LABEL_DESC_VIDEO - LABEL_GDT
```

```asm
;为了便于区分实模式代码与保护模式代码，
我就就把 16 位的实模式段先写前面:(一般的编程规范下 数据段是写前面的)

;=============================-8086 的 16 位实模式起始段============================
[SECTION .s16]     ;16 位代码段
[BITS 16]     ;BITS 指出处理器的模式 是 16 位
LABEL_BEGIN:               ;从 100h 处跳进来的
     mov   ax,cs
     mov ds,ax
     mov es,ax
     mov ss,ax
     mov sp,100h   ;在实模式下并没有用到 sp,这个 100h 只是形象表示需要保存实模式的 SP 值.
     mov [LABEL_GO_BACK_TO_REAL+3], ax   ; 请看[LABEL_GO...]+3 标号处的注释。是一个指令参数
     mov [SPValueInRealMode],sp            ;在这里保存实模式的 sp 值
     ;-----------全局数据段描述符初始化-----------
     xor eax,eax
     mov ax,ds
     shl eax,4                             ;ds * 16 代表这 DS 原来的基地址
     add eax,LABEL_DATA1       ;得到物理地址
     mov WORD   [LABEL_DESC_DATA + 2],ax
     shr eax,16
     mov BYTE [LABEL_DESC_DATA + 4],al
     mov BYTE [LABEL_DESC_DATA + 7],ah       ;此时数据段描述符已经有基址了也就是可以访问了
     ;-----------全局堆栈段描述符初始化-----------  
     xor eax,eax
     mov ax,ds
     shl eax,4
     add eax,LABEL_STACK
     mov WORD   [LABEL_DESC_STACK + 2],ax
     shr eax,16
     mov BYTE [LABEL_DESC_STACK + 4],al
     mov BYTE [LABEL_DESC_STACK + 7],ah;此时堆栈段描述符已经有基址了也就是可以访问了
     ;-----------32 位代码段描述符初始化-----------
     xor eax,eax
     mov ax,cs
     shl eax,4
     add eax,LABEL_SEG_CODE32
     mov WORD   [LABEL_DESC_CODE32 + 2],ax
     shr eax,16
     mov BYTE [LABEL_DESC_CODE32 + 4],al
     mov BYTE [LABEL_DESC_CODE32 + 7],ah     ;进入保护模式后开始执行的代码段
     ;-----------16 位代码段描述符初始化-----------
     xor eax,eax
     mov ax,cs
     shl eax,4
     add eax,LABEL_SEG_CODE16
     mov WORD   [LABEL_DESC_CODE16 + 2],ax
     shr eax,16
     mov BYTE [LABEL_DESC_CODE16 + 4],al
     mov BYTE [LABEL_DESC_CODE16 + 7],ah ;用来跳转到实模式的 386 代码段
     ;----------GDTR Ready -----------
     xor eax,eax
     mov ax,ds
     shl eax,4
     add eax,LABEL_GDT
     mov [GdtPtr + 2],eax
     lgdt [GdtPtr]   ;loader gdtr
      ;----------打开 A20--------
      CA20
       cli
      ;--------置 CR0 PE 位-----
      mov eax,cr0
      or   eax,1
      mov cr0,eax
      ;---------------跳到 386-----------
     jmp dword SelectorCode32:0             ;以上 3 个步骤无需多讲了
```

```asm
;这个是迎接保护模式跳回来的时候，执行的代码，，欢迎 386 回来啊！！
LABEL_REAL_ENTRY:
     mov ax,cs
     mov ds,ax
     mov es,ax
     mov ss,ax
     mov sp,[SPValueInRealMode]    ;恢复到 Real 原来的堆栈 注意这里要用[标号]
     DA20     ;关闭 20 地址线
     sti
     mov ax,4c00h
     int 21h
```

```asm
;===================================386 保护模式段==================================
;-------------------数据段--------------------------
[SECTION .data1]    
align 32
[BITS 32]
LABEL_DATA1:
   SPValueInRealMode     dw     0               ; 这个变量用来保存实模式跳入到保护模式前的 SP 值
   ;---------字符串-------------
   PMMessage:   db "welcome to Protect Mode ", 0 ; 进入保护模式后显示此字符串
   OffsetPMMessage   equ PMMessage - $$             ;保护模式寻址方式是按段偏移          
   StrTest:   db "This is 5MB", 0
   OffsetStrTest    equ StrTest - $$                        ;测试 5MB 空间所用的字符串
SegDataLen   equ   $ - LABEL_DATA1    ;数据段长
```

```asm
;-----------------------------------386 全局堆栈段------------------------------------
[SECTION .stack32]
align 32
[BITS 32]
LABEL_STACK:
times 512 db 0                                           ;堆栈大小是 512byte
TopOfStack equ $ - LABEL_STACK - 1                     ;栈顶的值
```

```asm
;-----------------------------------进入保护模式后的起始代码段------------------------------------
[SECTION .s32]; 32 位代码段. 由实模式跳入.
[BITS 32]
LABEL_SEG_CODE32:
     mov ax,SelectorData
     mov ds,ax
     mov ax,SelectorTest
     mov es,ax
     mov ax,SelectorVideo
     mov gs,ax                                      ;以上 3 个也不用多说了吧，段的选择子也就是 GDT 的索引
     ;---堆栈--
     mov ax,SelectorStack
     mov ss,ax
```



`mov esp,TopOfStack`                 ;当然堆栈段也是段的选择子咯

;-----------显示缓冲--------

`mov ah, 0Ch`    ; 0000: 黑底    1100: 红字
`mov esi,OffsetPMMessage`                          ;这个是字符串相对于它的段偏移值
`mov edi,(80*10+0)*2`     ;第 10 行
`cld`
.1:
`lodsb`
`test al,al`
`jz   .2`
`mov [gs:edi],ax`       ;要用 ax 做为参数传进缓冲区
`add edi,2`
`jmp .1`
.2:                                      ;OffsetPMMessage   字符串显示完成:
;------------------------测试 5MB 空间的读------------------

`call DispReturn`    ;显示回车，也就是改变 edi 的位置(edi 是显缓冲区段的偏移值)
`call ReadTest`
`call WriteTest`
`call ReadTest`     ;当在执行 Call 命令的时候     会将 eip + 1 压栈,然后跳转
`jmp SelectorCode16:0`      ;跳到最后那个 16 位代码段去了 前面有 16 位段描述符的讲解。。

;--------------显示回车---------------
DispReturn:
`push ebx`                         ;临时用 ebx eax 所以先保存一下
`push eax`
`mov bl,160`
`mov eax,edi`
`div bl`
`inc eax`   ;得到回车后的行数
`mov bl,160`
`mul bl`
`mov edi,eax` ;取得当前的位置
`pop eax`
`pop ebx`
`ret`      ;ret 指令会恢复 eip + 1

;--------------读取我们定义的大地址段---------------
ReadTest:
`xor  esi,esi`
`mov  ecx,11`
`mov  ah, 0Ch`      ; 0000: 黑底    1100: 红字
.loop:
`mov  al,[es:esi]`          ;这个是 SelectorTest 的段,也就是我们测试的大地址段
`call  DispAL`                ;显示从 test 段取出来的字符
`inc  esi`
`loop  .loop`               ;cx--
`call    DispReturn`
`ret`

;--------------写入我们定义的大地址段---------------
WriteTest:
`push esi`                         ;借这两个寄存器来传字符串
`push edi`
`xor esi, esi`
`xor edi, edi`
`mov esi, OffsetStrTest` ; 数据段的字符串
`cld`
.1:
`lodsb`
`test al, al`
`jz .2`
`mov [es:edi], al`    ;把数据段的字符串传给测试段
`inc edi`
`jmp .1`
.2:
`pop edi`
`pop esi`
`ret`

;--------------显示 AL 的内容---------------
DispAL:                         ;对传来的 Al 字符进行处理
`test al,al`
`jnz   .next`
`mov al,'0'`
.next:
`mov ah, 0Ch`    ; 0000: 黑底    1100: 红字
`mov [gs:edi],ax`
`add edi,2`
`ret`

`SegCode32Len equ $ - LABEL_SEG_CODE32`

;-----------------------------------准备 8086 模式的 16 位段------------------------------------
`[SECTION .code16]`
`align 32`
`[BITS 16]`

`LABEL_SEG_CODE16:`
`mov ax,SelectorNormal`       ; 保护 8086 标准段属性的描述符选择子
`mov ds, ax`
`mov es, ax`
`mov fs, ax`
`mov gs, ax`
`mov ss, ax`       ;使所有的段选择子都达到 8086 标准
`mov eax,cr0`
`and al,0feh`
`mov cr0,eax`    ;CR0 PE 位为 0 回到 Real mode
`LABEL_GO_BACK_TO_REAL:`
`jmp 0:LABEL_REAL_ENTRY`            ;还记得上面的[LABEL_GO..]+3 吗，指的就是 0 这个段值
`Code16Len equ $ - LABEL_SEG_CODE16`

呵呵、那么就完工了代码虽然比较繁杂，但是还不是特别复杂：

有几个要注意的地方:

1、堆栈(新加了堆栈这个段，其寻址方式也是选择子。)

在入口的时候是实模式,应该把当前的 sp 值保存下来,因为在返回系统时我们需要基本恢复原来的样子。

就在保护模式的全局数据段定义一个变量用来保存 SP。在返回 DOS 时候要记得恢复。

2、保护模式到实模式

实模式到保护模式 比较简单就几个固定步骤不需要考虑太多，而从保护模式到实模式就比较复杂：

首先需要定义两个 8086 标准属性的段描述符.它们分别是 Normal 数据段与 16 位代码段描述符。

在 16 位代码段中 cs 的属性跟 8086 模式的 CS 段界限与段属性一致，并且把 Normal 的段属性分别付给全部的数据段寄存器.这样就让所有保护模式的数据段寄存器全部跟 8086 段界限与段属性一致！！！

在 cr0 的 PE 位为 0 后 就可以转换成实模式寻址了!!!

接下来继续学习:

引用: 补充一下.com 要在纯 DOS 下运行，，，因为用到了很对特权指令!!!

Day Day Up

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || 5MB.rar  |

# 【LDT 描述符&&特权级&&门】

## 一、LDT（局部描述符）

GDT 是全局描述符，是整个系统的描述符，描述符着所有的段！！！

在前几章我们已经熟悉了 GDT 的一些基本功能，与运作机制。

对 GDT 描述符的定义与使用也就那么几项固定的步骤，接下来再了解 LDT.

LDT 是局部描述符。看字面 LDT 与 GDT 很相似.它们都是描述符。

只不过 GDT 是全局描述符、而 LDT 是局部描述符。

那么 LDT 该如何【定义】与【使用】呢？(与 GDT 非常类似，不过 LDT 是归属于 GDT 的)：

### @【定义】：

1、在 GDT 的描述集合中插入一条 `LABEL_DESC_LDT` Descriptor `0,LdtLen - 1,DA_LDT`
（这个段描述符描述的 LDT 数据段：是一个局部描述符段的集合,结构类似 GDT.）

2、在 GDT 选择子集合中照样也插入一条 `SelectorLdt equ ;LABEL_DESC_LDT - LABEL_GDT`
（看似跟普通的段描述符没啥两样哦,不过接下来就有点小区别了!）

3、建一个 LDT 数据段,这个数据段里的结构与 GDT 描述符段类似!

```
LABEL_LDT:         ;这里与 GDT 不同,LDT 的段描述符一开始就是实际的段
LABEL_DESC_LDT_CODEA:    Descriptor 0,LdtCodeALen - 1,DA_C | DA_32
LABEL_DESC_LDT_DATA:     Descriptor 0,LdtDataLen - 1,DA_DRW
LdtLen equ $ - LABEL_LDT;
;在加载 LDT 的时候并不是靠一个绝对结构体，而是通过 GDT 这个桥梁索引加载的。
;LDT 内部段的选择子
SelectorLdtCodeA   equ   LABEL_DESC_LDT_CODEA - LABEL_LDT | SA_TIL
SelectorLdtData   equ   LABEL_DESC_LDT_DATA - LABEL_LDT | SA_TIL
```

`;LDT 内部的选择子多了一个 SA_TIL 属性，他是段选择的 TI 位也就是第 2 位 如果是 1 代表当前选择子代表的段是 LDT 的内部段。 因为选择子的 0-2 位并不参与索引。所以选择子的索引都是 8 的倍数或 0，`

4、接着完成 LDT 里面段描述符描述的段。这里只定义了一个代码段: `LABEL_DESC_LDT_CODEA`
(这个跟普通的代码段一样，有执行、32 位等属性. 为了测试 LDT 的效果那么我们还是加一个 Data 段吧)

那么 LDT 的基本定义已经完成了！！！

### @【使用】：

1、LDT 描述符与其内部的局部段描述符信息、，我们在进入保护模式前初始化!!!

那么我们的进入保护模式之前的段在哪还知道吗？当然就是最最开始了！在 16 位段实模式入口那里!!!

我们在这里做完 GDT 描述符的初始化工作后：

`;--------------初始化 LDT 描述符---------`
`xor eax,eax`
`mov eax,ds`
`shl eax,4`
`add eax,LABEL_LDT`
`mov WORD [LABEL_DESC_LDT + 2],ax`
`shr eax,16`
`mov BYTE [LABEL_DESC_LDT + 4],al`
`mov BYTE [LABEL_DESC_LDT + 7],ah`

`;---------------初始化 LDT 内部的数据段描述符----------`
`xor eax,eax`
`mov eax,ds`
`shl eax,4`
`add eax,LABEL_LDT_DATA`
`mov WORD [LABEL_DESC_LDT_DATA + 2],ax`
`shr eax,16`
`mov BYTE [LABEL_DESC_LDT_DATA + 4],al`
`mov BYTE [LABEL_DESC_LDT_DATA + 7],ah`

`;---------------初始化 LDT 内部的代码段描述符----------`
`xor eax,eax`
`mov eax,ds`
`shl eax,4`
`add eax,LABEL_LDT_CODEA`
`mov WORD [LABEL_DESC_LDT_CODEA + 2],ax`
`shr eax,16`
`mov BYTE [LABEL_DESC_LDT_CODEA + 4],al`
`mov BYTE [LABEL_DESC_LDT_CODEA + 7],ah`

上面几个初始化段基地址的代码相信大家已经很熟悉了!!!!!!!!

2、 先前就说过了 LDT 跟 GDT 类似都是描述符集合，那么他们在使用起来也有点类似!!!

需要用 `lldt` 指令：它的参数并不是一个结构体的指针,因为 LDT 是属于 GDT 的。不能独立的加载。必需依靠 GDT 这个桥梁。那么这个参数也就是 LDT 在 GDT 当中的索引值.,

在进入保护模式后我们添加如下代码进行 LDT 的加载：

`mov ax,SelectorLdt`
`lldt ax`      ;lldt 加载时是从 GDT 当中的 ax 索引位置的段做 LDT 段加载到 LDTR 寄存器的.\\
`jmp SelectorLdtCodeA:0` ;首先检查选择子的 TI 位，当是 LDT 选择子的时候。马上去 LDTR 寄存器取 LDT 在 GDT 当中的索引，找到 LDT 的位置。最后根据 SelectorLdtCodeA 在 LDT 局部描述符集合中查找。

## 二、特权级：

我们在 WIindows 里使用 Ring3 程序也好、Ring0 函数也好，在执行他们的时候或者调用他们的时候是按一定规则执行的? 当然 Windows 自己包装的规则不在讨论之类,.我们现在需要了解 CPU 最铁最硬的规则!!!

Ring0 -Ring3 并不是属于 WINDOWS 的东西，而是属于 CPU 本身的!!! 那么当我们在使用程序的时候或者模块的时候 cpu 是如何鉴别程序特权级的呢?

1、`CPL`（Current Privilege Level）

`CPL` 是指当前执行的程序或者任务的特权级，它存储在 CS 和 SS 的 0 到 1 位。CS、SS 在这里都是段选择子了, (第 2 位是 TI 位代表 LDT），这 0-1 位合起来可以表示 Ring0-Ring3 个级别.

通常情况下 CPL 等于代码本身所在段的特权级(DPL), 当程序跳转到不同特权级的代码段时,处理器将改变 CPL, 以上指得是非一致代码段。 如果是一致代码段，一致代码段可以被同等级的或者低特权级访问，并且跳转过去后 CPL 也不会改变。

2、`DPL`(Descriptor Privilege Level)

`DPL` 表示段或者门的特权级. 门其实也是一个段描述符来描述的。所以他们的 DPL 值就是段描述符属性里面的 DPL 位。下面来列举一些访问这些段时所需要的 CPL 值。

×数据段 ：

一个代码段要想访问这个数据段，那么必需 `CPL`(代码段) <= `DPL`(数据段)。

这样代码段才能正常访问此数据段。

×非一致代码段 ：(不使用调用门的情况下)

访问非一致代码段的条件是 `CPL == DPL`(非一致代码段)

×调用门 ：

访问调用门的条件: 跟数据一样 `CPL`(代码段) <= `DPL`(调用门段）

×一致代码段和通过调用门访问的非一致代码段：

访问一致代码段的条件： 必需 `CPL >= DPL`

通过调用门访问非一致代码段条件: 必需 `CPL >= DPL`

×TSS(任务状态段)：

`Tss` 是一个 32 位的结构体，它包含了 100 个字段。其中偏移量为 4 -27 里有 3 个 ss 和 3 个 esp 信息。它们记录着不同特权级任务切换时的堆栈信息。 由于只有特权级低到高切换任务时，新堆栈才会从 TSS 中取得。所以 TSS 里没有 Ring3 的 SS ESP 信息。

访问 TSS 的条件： 跟数据一样 `CPL`(代码段) <= `DPL`(TSS）

3、`RPL`(Request Privilege Level)：

它存放在段选择子的 0-1 这两位上面。 在访问目标段时 CPU 会做一系列检查。

其中 RPL 是访问目标段的关键:如果当前代码段 CPL 为 0，要访问对方数据段的 DPL 值是 1,而 RPL 值却是 2。那么访问是不成功的。数据段的 RPL 值一定要<=DPL 才能正常访问。。



### 三、门(描述符 说得通俗点，门有点像函数指针)：

门描述符分位 4 种：×调用门、×中断门、×陷阱门、×任务门。

程序从一段代码转移到另外一段代码时，目标代码的选择子会被加载到 `CS` 中，而在加载之前 CPU 还要对目标段描述符、界限、类型、特权级等信息做检查。检查通过后目标段选择子被加载到 `CS`，然而 `EIP` 指向新偏移地址。 程序的转移可以由：`jmp`、`call`、`ret`、`sysenter`、`sysexit`、`int n`、`iret` 这些命令来控制。

×用 `jmp` 或者 `Call` 指令进行转移的时：

直接用 `Call` 指令转移时：

如果目标代码段是非一致代码段那么 `CPL` 必需等于目标 `DPL`,同时要求 `RPL<=DPL.`。

如果是一致代码段那么 `CPL` 必需>=目标 `DPL`, `RPL` 此时不做检查。转移过去后 `CPL` 不变。

由此看来 `call`、`jmp` 指令只适用于低特权向高特权转移。

那么怎样从高特权转移到低特权呢？继续看下面：

②、引用调用门：

门也是一种描述符,它跟 `Descriptor` 一样大小都是 8 个字节。不过这个 8 个字节代表的意义大有不同之处。

首先看看这个 `Gate` 宏：

``` 
%macro Gate 4            ;门宏有 4 个参数：选择子、偏移量、调用者堆栈元素数量、属性  
dw  (%2 & 0FFFFh)     ; 偏移 1     (2 字节)  
dw  %1      ; 选择子     (2 字节)  
dw  (%3 & 1Fh) | ((%4 << 8) & 0FF00h) ; 属性      (2 字节)  
dw  ((%2 >> 16) & 0FFFFh)    ; 偏移 2     (2 字节)  
%endmacro ; 共 8 字节
```

门描述符的字节意义说明：

第 0、1 个字节代表\[偏移 1\]。

第 2、3 个字节代表段选择子。

第 4、5 个字节，其中第 5 个字节跟段描述符的第 5 个字节一样, `TYPE`(属性)、`S`(存储段而言，`S=1` 代表不是系统段和门段。所以这里要变成 0)、`DPL`(段特权级)、`P`（存在位）这几个位。而第 4 个字节是代表.调用者堆栈元素数量,这是要根据 `TSS` 完成调用者的堆栈元素复制到目标堆栈里。

第 6、7 个字节代表\[偏移 2\]。与 0、1\[偏移 1\]按高高低低合并就可以得到，偏移值了。

调用门描述符的定义：

1、在 `GDT` 全局描述符里添加一个：

``` 
;门宏 、选择子                、偏移、ParamCount    、描述符属性  
LABEL_CALL_GATE_TEST:          Gate ,SelectorGateCode32,    0,    0   ,DA_386CGate
```

`SelectorGateCode32` 只是一个普通的代码段选择子，只不过是被调用门来使用而已。

2、同样门也需要自己在 `GDT` 中的位置也就是选择子：

``` 
SelectorGateTest equ LABEL_CALL_GATE_TEST - LABEL_GDT
```

调用门描述符的使用：

``` 
Call SelectorGateTest:0           ;这个指令，CPU 首先找到选择子在 GDT 的位置，然后检查选择子对应的描述符，是个什么描述符。 检查完毕以后就做对应的处理。
```

这里描述符是个门描述符所以 CPU 会进行门描述符对应的位功能检测,那么会得到:一个代码段的选择子，与偏移、等信息。 这就可以进行跳转了。将会跳转到 `SelectorGateCode32` 代码段选择子对应偏移为 0 的代码处。

### 四、让程序走进 Ring3：

调用门的过程实际分为两部分：

1、从低特权向高特权访问， 要 `Call` 调用门指令。
2、从高特权向低特权 用 `Ret` 指令

在 8086 时期的 `Call` 指令短转移(段内)时，CPU 会先将 `ip` + 1 压栈。 等待 `ret` 指令完成后才会弹出到新的 `ip`。

而在长转移时，CPU 会将 `cs` 与 `ip` 一起压栈.这样等待 `retf` 指令完成后 才会弹出到对应的 `ip` `cs`。

`JMP` 指令比较简单 只是目标操作数不同而已。

在 386 后不同特权级之间的堆栈是独立的，这样的话如果是远转移的话，那么就无法正确获得本身堆栈里的 `EIP` 信息了.因为都不是同一个堆栈了。那么 386 机制有个办法就是靠任务状态堆 `TSS` 来解决。在任务进行切换时，可以先从 `TSS` 里获得任务的重要信息。`TSS` 是一个 32 位的结构体，含有 100 个字段。其中 4 -27 字段就是 3 个特权级的堆栈信息。？？特权级有 4 种，为什么确只记录 3 个堆栈信息呢。因为只有从低特权级到高特权级转移时,才能从 `TSS` 里获取到新堆栈信息.所以没有 `Ring4` 就无法在 `TSS` 里获 `Ring3` 的信息,

通过调用门从一个代码段跳转到另外一个代码段的整个步骤：

1、根据目标代码的 `DPL` 对应的特权级从 `TSS` 中选择 `SS` 和 `ESP` （`TSS` 结构体第 4-27 记录着 3 个特权级堆栈信息）.如果目标代码 `DPL=1` 那么 就选 `ss1`、`esp1`;(12 -16 字段)。

2、第二步是检查所选择的 `ss1` `esp1` 或 `Tss` 是否有界限错误(`ERR #TS`)。

3、第三步对 `SS` 堆栈段选择子进行校验。

4、暂时性的保存当前 `SS` 和 `ESP` 的值,调用者自己的堆栈信息。

5、加载新的 `ss1` 和 `esp1` 到 `ss`、`esp`。

6、将刚才保存的当前 `ss`、`esp` 压栈。调用者自己的堆栈信息

7、从调用者的堆栈中将参数复制到新堆栈中，复制的数量由门描述符中的 `ParamCount` 指定。

8、将当前的 `cs` 和下一个 `eip` 压栈。

9、加载调用门指定的 `CS` `EIP`，开始执行新的代码段。此时的堆栈也是新的。

通过调用门方式转移时。只有低特权转高特权才能在 `TSS` 中获得新堆栈信息。如果不是低特权转高特权那么堆栈信息是不会改变的。用 `Call` 指令只能实现低特权到高特权。

通过调用门从一个代码段跳转到另外一个代码段后再返回到调用段的步骤：

1、检查保存在堆栈中的 `CS`(也就是调用者调用时的 `CS` 值(段选择子))中的 `RPL` ,以判断返回是否需要变换特权级，这个 `CS` 保存在新堆栈中弹出调用者 `eip` 后 `esp` 指向的地方。

2、加载新堆栈里的 `CS` 和 `EIP` （也就是调用者的 `CS:EIP`）,此时会对加载的 `CS:eip` 进行校验。

3、如果 `Ret` 指令含有参数,则跳过 `Ret` 指定的数量。然后 `esp` 指向调用者的 `ss:esp` 信息在新堆栈中的位置。

`Ret` 参数应该对应与门描述符的 `ParamCount`。

4、加载堆栈里面的 `SS`、`esp` 到 `ss:esp` 寄存器里，被调用者的 `ss` `esp` 被覆盖。进行 `ss:esp` 校验。

5、.如果先前的 `Ret` 有参数。也就是代表是从调用者堆栈复制过去的参数，现在调用过程完成了，不需要这些参数了，同样在调用堆栈里也清理掉。 `esp` +参数。

6、检查 `ds`、`es`、`fs`、`gs` 的值，因为 `cs`、`ss` 都已经检验过了。要对其它的段寄存器进行检验了，如果哪一个段寄存器的段描述符 `DPL` 小于 `CPL`(也就是段特权级大于了 `CPL.`那么 `CPL` 不能对它进行访问了)，那么一个空的描述符覆盖给此段寄存器。

`Ret` 转移的方式跟 `Cal` 恰恰 l 相反，所以它适合从高特权转移到低特权。不过在转移的时候会经过一系列检查与堆栈操作。

\*从 `Ring0` 到 `RIng3` 的方法：

1、使用 `ret` 指令(高特权到低特权)：

可在 32 位实模式入口段( 这是个逆操作,可以看作是被 `Ring3` Call 进来的.所以返回可以是 `retf`)：

``` 
push         SelectStack3  
push        TpoOfStack3  
push       SecletorR3Code32  
push         0  
retf                         ;Retf 的步骤上面有说明!
```

这就是从 `Ring0` 到 `RIng3` 的代码。

\*从 `Ring3` 到 `RIng0` 的方法

2、使用 `Call` 指令调用门(低特权到高特权)：

从低特权到高特权需要新的堆栈段那么需要 `TSS` 数据段:首先就定义一个 `TSS` 数据段吧.

``` 
;-----------------TSS 数据段-----------------  
[SECTION .tss]  
align 32     ;因为 TSS 本身是 32 位字段,那么就用 32 位对齐  
[BITS 32]  
LABEL_TSS:  
    dd   0         ;上一个任务链接  
    dd   TopOfStack0      ;0 级堆栈 esp  
    dd   SelectorStack0     ;0 级堆栈 ss   4-12 字段  
    dd   0         ;1 级堆栈  
    dd   0         ;1 级堆栈  
    dd   0         ;2 级堆栈  
    dd   0         ;2 级堆栈                     字段 r0 -r2  
   dd   0         ;cr3  
    dd   0         ;eip  
    dd   0         ;eflags  
    dd   0         ;eax  
    dd   0         ;ecx  
    dd   0         ;edx  
    dd   0         ;ebx  
    dd   0         ;esp  
    dd   0         ;ebp  
    dd   0         ;esi  
    dd   0         ;edi  
    dd   0         ;es  
    dd   0         ;cs  
    dd   0         ;ss  
    dd   0         ;ds  
    dd   0         ;fs  
    dd   0         ;gs  
    dd   0         ;LDT  
    dw   $ - LABEL_TSS + 2   ;I/O 位图基址  
    dw   0ffh        ;I/O 位图结束标志  
TssLen equ $ - LABEL_TSS
```

;--------加载 TSS------

``` 
     mov ax,SelectorTss   ;  
     ltr ax                         ;与加载 LDT 类似都是从 GDT 索引位置找到信息后,加载到对应的寄存器里.
```

下面来看看这个程序的来龙去脉吧：

1、入口段实模式：

进行 `GDT`、等数据初始化。

然后跳转到 `SelectorCode32(Ring0);`

2、`SelectorCode32(Ring0)`

首先显示一行字符串:"Hello Protect Mode".

;--------加载 TSS------

``` 
     mov ax,SelectorTss  
     ltr ax    
     ;-------这是个逆向,这里看作是被 Ring3 Call 进来的.所以返回可以是 retf  
     push SelectorStack3     ;ss  
     push TopOfStack3        ;esp  
     push SelectorR3Code32       ;cs  
     push 0         ;eip  
     retf                         ; Ring0 到 Ring3  
```

3、`SelectorR3Code32(Ring3)`

首先显示:字符"3"。

``` 
call SelectorGateTest:0      ;通过门进行低特权 Ring3 到高特权 Ring0 的转移
```

4、`SelectorGateCode32(Ring0)` 是调用门指向的代码段

首先显示：字符"0"。

``` 
mov ax,SelectorLdt  
     lldt ax  
     jmp SelectorLdtCode32:0       ;跳转到 LDT 代码区 ，
```

5、`SelectorLdtCode32(Ring0)`

首先显示：字符"L".

``` 
jmp       SelectorCode16:0 回到实模式了。
```

【总结】

这个程序的整个路线就是这样了,在这里的知识点比较多。本人在学习的时候也碰到了很多问题。不过想不通的先借过，等了解到新知识后再返回来学习，效果会更好!

这里有以下几个重要知识点：

一、LDT（局部描述符）

局部描述段跟 `GDT` 的定义很相似。因为都是存放数据或者代码段的描述符集合！

`LDT` 的加载是用 `Lldt ax` ;其中这个 `ax` 是 `LDT` 描述符在 `GDT` 中的选择子。

那么 `LDT` 段的寻址方式略有不同。 `jmp S:0` ;`S` 是 `LDT` 集合的一个选择子。其中 `TI` 位为 1 表示是 `LDT` 的选择子。这个时候 CPU 会从 `LDTR` 寄存器指向的 `LDT` 集合中去找对应的选择子。

二、特权级（局部描述符）

特权级用在对数据访问的控制和对代码转移的控制.

对数据的访问比较简单： `CPL` <=目标 `DPL` 并且 `RPL<=DPL.`

对代码的转移目前接触了:

1、直接用 `Call`、`jmp` 指令进行转移。

当目标是非一致代码的时候，必需 `CPL==`目标 `DPL,RPL<=DPL.`也就是说必需是相等特权转移。

当目标是一致代码的时候，必需 `CPL<=`目标 `DPL`，`RPL` 不做检查，转移后 `CPL` 不变。

所以直接用 `Call`、`Jmp` 指令转移只能是低特权到高特权。如果目标是非一致那还必需等于、

2、`CALL` 调用门：

`Call` 调用门 可以从低特权到高特权，并且不受非一致代码限制。 但是他要用到 `TSS.`

3、`Ret` 返回指令

`Ret` 是返回指令,`Call` 的逆向。了解 `Ret` 的运作机制就可以 正确从高特权向底特权转移了。

特权级的标志也就是这 3 个： `CPL`、`DPL`、`RPL`。

接下来继续学习......Day Day Up

上传的附件

`img/c8c74a380a3dea7fa2460187d80ee5e0.jpg` || `Ring3.rar`
