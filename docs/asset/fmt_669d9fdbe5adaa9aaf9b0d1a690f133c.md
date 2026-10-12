
# MBR 引导代码解析

## 头文件与宏定义

`MBR` 结构偏移以宏的方式定义在 `stage1.h` 中。

```c
#include <stage1.h>
/*重定位宏*/
#define ABS(x) (x-_start+0x7c00) 

/* 打印字符串宏定义 */
#define MSG(x)    movw $ABS(x), %si; call message 

#define    MOV_MEM_TO_AL(x)    .byte 0xa0;  .word x
```

## 段与符号定义

`.file op` 指定和目标文件关联的源文件名。

```assembly
    .file    "stage1.S"

/* .text 代码段声明; .code16 使用实模式 */
    .text
    .code16 

/* 代码段入口 .globl 全局符号定义 */
.globl _start; _start:
```

## MBR 引导跳转

`MBR` 开始跳转指令，跳转到真正的引导代码处。
后跟一个 `NOP` 指令，占 `MBR` 偏移的：`0x00-0x03`。
更确切的说，这应该是 `DBR`。

```assembly
    jmp    after_BPB
    nop     

/* .程序引用计数器，这里应该是对齐 */
    . = _start + 4 
```

## int13 扩展参数结构

以下定义为 `int13` 扩展使用的参数结构：
* 扇区号：`sectors`
* 磁头号：`heads`
* 柱面：`cylinders`
* 在 `int 13H` 调用时使用

```assembly
mode:
    .byte    0
disk_address_packet:     
sectors:
    .long    0
heads:
    .long    0
cylinders:
    .word    0
sector_start:
    .byte    0
head_start:
    .byte    0
cylinder_start:
    .word    0
```

## BPB 部分与版本定义

`MBR` 偏移 `0x0B` 开始是 `BPB` 部分，占用 `50` 或者 `80` 字节。
这里占用 `50` 字节，故此：
`#define STAGE1_BPBEND 0x3e`

```assembly
    . = _start + STAGE1_BPBEND 
```

`stage1` 版本定义，版本号在 `stage1` 中以宏的方式定义：
* `#define COMPAT_VERSION_MAJOR 3`
* `#define COMPAT_VERSION_MINOR 2`
* `#define COMPAT_VERSION ((COMPAT_VERSION_MINOR << 8) | COMPAT_VERSION_MAJOR)`
* `boot_drive` 用于指出 `stage1` 需要装载的代码在何处
* 后面是 `stage2` 的载入地址定义

```assembly
stage1_version:     
    .byte    COMPAT_VERSION_MAJOR, COMPAT_VERSION_MINOR
boot_drive:     
    .byte    0xFF     
force_lba:                   ；强制 LBA 模式
    .byte    0
stage2_address:             ；stage2 装入地址
    .word    0x8000
stage2_sector:             ；stage2 在磁盘上的扇区
    .long    1
stage2_segment:
    .word    0x800 
```

## 引导区代码

```assembly
after_BPB: 

    cli                    ；关中断     
/*
 * Drive Table
 * DL = 00h     1st floppy disk ( "drive A:" )
 * DL = 01h     2nd floppy disk ( "drive B:" )
 * DL = 80h     1st hard disk
 * DL = 81h     2nd hard disk     
 */ 

boot_drive_check:     
    jmp    1f
    testb    $0x80, %dl            ；BIOS 加载完启动代码会把%dl 设置成启动盘号(boot drive number):
    jnz    1f
    movb    $0x80, %dl
1:     

    ljmp    $0, $ABS(real_start)    ；跳转到真正的引导代码处，远跳转 

real_start:     

    xorw    %ax, %ax             
    movw    %ax, %ds
    movw    %ax, %ss            ;段寄存器初始化 

    movw    $STAGE1_STACKSEG, %sp       ;堆栈地址初始化， 宏定义：#define STAGE1_STACKSEG        0x2000 

    sti                ;关中断 

    MOV_MEM_TO_AL(ABS(boot_drive))     
    cmpb    $0xFF, %al            ；检查是否设置了启动磁盘,如果不是，dl 送 al，将 dx 入栈后在屏幕上显示“GRUB”
    je    1f            ；如果是，调整后一个 1：处
    movb    %al, %dl
1:
    pushw    %dx 

    MSG(notification_string) 

    testb    $STAGE1_BIOS_HD_FLAG, %dl   ；测试启动磁盘是否是硬盘，DL 内的值有 BIOS 设置
    jz    chs_mode            ；不是硬盘则使用 CHS 模式
    movb    $0x41, %ah
    movw    $0x55aa, %bx
    int    $0x13            ； int 13 0x41 扩展判断是否支持 LBA 模式 ， 

    popw    %dx            ；恢复 dl 值
    pushw    %dx            ；保存 dx 值 

    jc    chs_mode         
    cmpw    $0xaa55, %bx    ；是否支持 LBA 模式，不支持则启用 CHS 模式
    jne    chs_mode 

    MOV_MEM_TO_AL(ABS(force_lba))       ；相当于 movb ABS(force_lba), %al         
    testb    %al, %al            ；是否强制使用 LBA 模式
    jnz    lba_mode
    andw    $1, %cx
    jz    chs_mode
```

## LBA 模式与 DAP 结构说明

`LBA` 模式使用下：

```c
struct DiskAddressPacket
    {
        BYTE PacketSize;     // 数据包尺寸:
                    //(固定值，恒等于 16，即 10H，指本结构所占用的存储空间)
        BYTE Reserved;       // ==0
        WORD BlockCount;     // 要传输的数据块个数(以扇区为单位)
        DWORD BufferAddr;    // 传输缓冲地址(segment:offset)
        QWORD BlockNum;      // 磁盘起始绝对块地址  8 个字节
    }; 
```

`PacketSize` 保存了 `DAP` 结构的尺寸, 以便将来对其进行扩充. 在
目前使用的扩展 `Int13H` 版本中 `PacketSize` 恒等于 `16`. 如果它小于
`16`, 扩展 `Int13H` 将返回错误码( `AH=01`, `CF=1` ).
`BlockCount` 对于输入来说是需要传输的数据块总数, 对于输出来说
是实际传输的数据块个数. `BlockCount` = `0` 表示不传输任何数据块.
`BufferAddr` 是传输数据缓冲区的 `32` 位地址 (段地址:偏移量). 数据
缓冲区必须位于常规内存以内(1M).
`BlockNum` 表示的是从磁盘开始算起的绝对块地址(以扇区为单位),
与分区无关. 第一个块地址为 `0`. 一般来说, `BlockNum` 与 `CHS` 地址的关系
是:
`BlockNum` =
(`cylinder` * `NumberOfHeads` + `head`) * `SectorsPerTrack` + `sector` - `1`; 

其中 `cylinder`, `head`, `sector` 是 `CHS` 地址, `NumberOfHeads` 是磁盘
的磁头数, `SectorsPerTrack` 是磁盘每磁道的扇区数.
也就是说 `BlockNum` 是沿着 扇区->磁道->柱面 的顺序记数的. 这一顺
序是由磁盘控制器虚拟的, 磁盘表面数据块的实际排列顺序可能与此不同
(如为了提高磁盘速度而设置的间隔因子将会打乱扇区的排列顺序). 

## LBA 模式读取实现

```assembly
lba_mode:                  ；使用 LBA 方式读入下一个扇区
    movl    0x10(%si), %ecx 

    movw    $ABS(disk_address_packet), %si  ；esi 保存 stDiskAddressPacket 结构体 

    movb    $1, -1(%si)
    movl    ABS(stage2_sector), %ebx 

    movw    $0x0010, (%si)      ；PacketSize 保存了 DAP 结构的尺寸,恒等于 0x10 

    movw    $1, 2(%si)          ；输入的数据块个数  [esi +2]
    movl    %ebx, 8(%si)        ；磁盘起始绝对块地址 [esi +8] 起 8 个字节为绝对块地址 

    movw    $STAGE1_BUFFERSEG, 6(%si)   ;读数据的缓冲区 [esi +6 ] 

    xorl    %eax, %eax            ;eax 清零
    movw    %ax, 4(%si)          ；[esi +4] == 0
    movl    %eax, 12(%si)        ; [esi +12] == 0
/*
 * BIOS call "INT 0x13 Function 0x42" to read sectors from disk into memory
 *    Call with    %ah = 0x42
 *            %dl = drive number
 *            %ds:%si = segment:offset of disk address packet
 *    Return:
 *            %al = 0x0 on success; err code on failure
 */ 

    movb    $0x42, %ah
    int    $0x13 

    jc    chs_mode         ；Jump if ECX is Zero 

    movw    $STAGE1_BUFFERSEG, %bx
    jmp    copy_buffer
```

## CHS 模式读取实现

```assembly
chs_mode:     
    movb    $8, %ah
    int    $0x13           ；int 08h 中断读磁盘参数 ，读磁盘参数
    jnc    final_init          ；cx 不为 0 跳走，调用 int 13 2 号功能读取数据 

    testb    $STAGE1_BIOS_HD_FLAG, %dl
    jz    floppy_probe        ；是否是软盘 

    jmp    hd_probe_error       ；硬盘可能出错 

final_init:
    movw    $ABS(sectors), %si 

    movb    $0, -1(%si)
    xorl    %eax, %eax
    movb    %dh, %al
    incw    %ax
    movl    %eax, 4(%si) 

    xorw    %dx, %dx
    movb    %cl, %dl
    shlw    $2, %dx
    movb    %ch, %al
    movb    %dh, %ah 

    incw    %ax
    movw    %ax, 8(%si) 

    xorw    %ax, %ax
    movb    %dl, %al
    shrb    $2, %al 

    movl    %eax, (%si) 

setup_sectors:
    movl    ABS(stage2_sector), %eax 

    xorl    %edx, %edx 

    divl    (%si) 

    movb    %dl, 10(%si) 

    xorl    %edx, %edx     
    divl    4(%si)         

    movb    %dl, 11(%si) 

    movw    %ax, 12(%si) 

    cmpw    8(%si), %ax
    jge    geometry_error 

    movb    13(%si), %dl 

    shlb    $6, %dl         
    movb    10(%si), %cl     

    incb    %cl         
    orb    %dl, %cl     
    movb    12(%si), %ch     

    popw    %dx
    movb    11(%si), %dh 

/*
 * BIOS call "INT 0x13 Function 0x2" to read sectors from disk into memory
 *    Call with    %ah = 0x2
 *            %al = number of sectors
 *            %ch = cylinder
 *            %cl = sector (bits 6-7 are high bits of "cylinder")
 *            %dh = head
 *            %dl = drive (0x80 for hard disk, 0x0 for floppy disk)
 *            %es:%bx = segment:offset of buffer
 *    Return:
 *            %al = 0x0 on success; err code on failure
 */ 

    movw    $STAGE1_BUFFERSEG, %bx
    movw    %bx, %es     

    xorw    %bx, %bx     
    movw    $0x0201, %ax     
    int    $0x13 

    jc    read_error           ；cx 为 0，则读取错误 

    movw    %es, %bx
```

## 缓冲区复制与跳转

```assembly
copy_buffer:
    movw    ABS(stage2_segment), %es    ；stage 寄存器入 ES 段 

    pusha                 ；环境保存
    pushw    %ds             ；dl 保存
    movw    $0x100, %cx        ；因为使用的是 movsw，所以字 MBR 字节减半
    movw    %bx, %ds         ；读出的数据地址初始化 DS 段；
    xorw    %si, %si
    xorw    %di, %di
    cld                  ；设置方向标志，+
    rep
    movsw                 ；movsw es:\di},ds:\[si} 

    popw    %ds
    popa 

    jmp    *(stage2_address)       ;跳转执行 stage2 代码； 
```

## 错误处理

```assembly
/* 错误处理 */
geometry_error:
    MSG(geometry_error_string)
    jmp    general_error 

hd_probe_error:
    MSG(hd_probe_error_string)
    jmp    general_error 

read_error:
    MSG(read_error_string) 

general_error:
    MSG(general_error_string) 

stop:    jmp    stop                  ；死循环 

notification_string:    .string "GRUB "
geometry_error_string:    .string "Geom"
hd_probe_error_string:    .string "Hard Disk"
read_error_string:    .string "Read"
general_error_string:    .string " Error" 

/* BIOS int 10 号中断 0 号功能以打印机模式显示字符
 * AH = 功能号 0EH
 * AL = 打印的字符
 * BH = 页号
 * BL = 前景颜色
 * 16 位色彩编码表 (D7 D6 D5 D4 为背景色,D3 D2 D1 D0 为前景色)
 */ 

1:
    movw    $0x0001, %bx
    movb    $0xe, %ah
    int    $0x10         
message:
    lodsb
    cmpb    $0, %al
    jne    1b     
    ret 
```

## 软盘探测与尾部签名

```assembly
    . = _start + STAGE1_WINDOWS_NT_MAGIC
nt_magic:     
    .long 0
    .word 0 

part_start:     
    . = _start + STAGE1_PARTSTART 

probe_values:
    .byte    36, 18, 15, 9, 0 

floppy_probe: 

    movw    $ABS(probe_values-1), %si 

probe_loop:
    xorw    %ax, %ax
    int    $0x13 

    incw    %si
    movb    (%si), %cl 

    cmpb    $0, %cl
    jne    1f 

    MSG(fd_probe_error_string)
    jmp    general_error 

fd_probe_error_string:    .string "Floppy" 

1:     
    movw    $STAGE1_BUFFERSEG, %bx
    movw    $0x201, %ax
    movb    $0, %ch
    movb    $0, %dh
    int    $0x13
    jc    probe_loop     
    movb    $1, %dh
    movb    $79, %ch 

    jmp    final_init 

    . = _start + STAGE1_PARTEND
    .word    STAGE1_SIGNATURE
```
