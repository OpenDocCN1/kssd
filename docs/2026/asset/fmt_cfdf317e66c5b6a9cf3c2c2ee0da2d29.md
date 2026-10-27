
```assembly
/*预处理包含定义

 *其中STAGE1_5被自GRUB2后不使用

 *用到的share.h被其它一些源文件使用，

 *此文件使用到的宏不多，用到时给与说明

 ******************************/

#define ASM_FILE
#include <shared.h>

#ifndef STAGE1_5
#include <stage2_size.h>
#endif

/*
 *  defines for the code go here
 */

	/* Absolute addresses
	   This makes the assembler generate the address without support
	   from the linker. (ELF can't relocate 16-bit addresses!) */

/*重定位，并根据是否定义STAGE1_5选择装入地址*/

#ifdef STAGE1_5
# define ABS(x) (x-_start+0x2000)
#else
# define ABS(x) (x-_start+0x8000)
#endif /* STAGE1_5 */

	/* 调用int 10h中断以打印机模式在屏幕上显示一字符串*/

#define MSG(x)	movw $ABS(x), %si; call message

/*****************************************
   .file op 指定和目标文件关联的源文件名
 *****************************************/

	.file	"start.S"

/*****************************************
    .text代码段声明; .code16使用实模式
 *****************************************/

	.text

	/* 告诉GAS产生16位指令，以便此代码能在实模式下运行 */
	.code16

	.globl	start, _start
start:
_start:	
	/*start由stage1载入到0x8000处，si仍然用于DiskAddressPacket结构
	 *
	/* 首先保存设备类型号，stage1由bios初始化在dl中! */

	pushw	%dx

	/* 在屏幕上打印字符串：loading stage2 or loading stage1_5 
	 *由于在输出字符的时候使用了si寄存器，故此保存原si地址
	 *字符串输出完毕后恢复原字符串*/

	pushw	%si
	MSG(notification_string)
	popw	%si

	/*** #define BOOTSEC_LISTSIZE		8"  firstlist为此段末尾偏移地址*/

	movw	$ABS(firstlist - BOOTSEC_LISTSIZE), %di

	/* LBA以0开始，故此，stage1占1扇区，start占一扇区，故stage2从2开始，并存 倒数第8个字节处。 */
	movl	(%di), %ebp

        /*由于firstlist -08h处保存着需要载入的扇区号和扇区的数目，故此，
         * bootloop 子功能，循环将对应的扇区号中的数据载入相应内存中并执行，
	 *可能有人会问，为什么不将stage1和start融合在一起呢，这是因为一般的
	 * 硬盘低格后，默认的扇区大小为512个字节，也就是分开也是不得于而为之
	 */

bootloop:

	/* [edi]+4处为	.word (STAGE2_SIZE + 511) >> 9，这里不考虑stage1_5的情况
	 * STAGE2_SIZE为grub编译后的大小，>>9相当于除以512，
	 *4（%di）处存的是stage2占的扇区数目/
	 */

	cmpw	$0, 4(%di)

	/* 如果为0说明使用stage1_5直接跳至bootit处执行 */
	je	bootit

setup_sectors:	
	/* stage1中，如果检测支持LBA模式，此语句被执行：movb	$1, -1(%si)
	 * 避免了相同功能的重复调用/
	 */

	cmpb	$0, -1(%si)

	/* 结果为0，说明不支持LBA模式 */
	je	chs_mode

lba_mode:	
	/* 装载逻辑扇区号入ebx */
	movl	(%di), %ebx

/* the maximum is limited to 0x7f because of Phoenix EDD  
  看一下int 43的调用的参数说明：
  Format   of   disk   address   packet:   
  Offset   Size   Description   
  00h   BYTE   10h   (size   of   packet)   
  01h   BYTE   reserved   (0)   
  02h   WORD   number   of   blocks   to   transfer   (max   007Fh   for   Phoenix   EDD)   
  04h   DWORD   ->   transfer   buffer   
  08h   QWORD   starting   absolute   block   number   
  (for   non-LBA   devices,   compute   as   
  (Cylinder*NumHeads   +   SelectedHead)   *   SectorPerTrack   +   
  SelectedSector   -   1   
  --------N-134257DX1234-----------------------   */

	xorl	%eax, %eax
	movb	$0x7f, %al

	/* 比较应该读的数据数否超过最大传输参数 */

	cmpw	%ax, 4(%di)	/* compare against total number of sectors */

	/* 如果大于，跳至本句后一个标号 1:处，否则将值付给ax寄存器 */

	jg	1f

	/* if less than, set to total */
	movw	4(%di), %ax

1:	
	/* 用stage2的总大小减去ax中的值，ax的值为上面的判断后的设置值 */

	subw	%ax, 4(%di)

	/* 将ax的值加上起始扇区号，因为下面的代码会将ax中保存的扇区数目循环读出
	 * 这样，即使要读的数据超出7f也不用在定义其它的数据来保存剩下的扇区数
	 * /

	addl	%eax, (%di)

	/* 设置磁盘地址数据包结构体 */

	/* the size and the reserved byte */
	movw	$0x0010, (%si)

	/* 需要读出的扇区数目，见到ax的用处了吧！ */
	movw	%ax, 2(%si)

	/* 绝对扇区地址，(low 32 bits) */
	movl	%ebx, 8(%si)

	/* 缓冲区段地址设置，读出的数据保存于此处   #define BUFFERSEG   RAW_SEG (0x7000)*/

	movw	$BUFFERSEG, 6(%si)

	/* 保存ax的值 */
	pushw	%ax

	/* zero %eax */
	xorl	%eax, %eax

	/* 缓冲区偏移地址设置 == 0 */
	movw	%ax, 4(%si)

	/* 高32位绝对地址 (high 32 bits) */
	movl	%eax, 12(%si)

/*
 * BIOS call "INT 0x13 Function 0x42" to read sectors from disk into memory
 *	Call with	%ah = 0x42
 *			%dl = drive number
 *			%ds:%si = segment:offset of disk address packet
 *	Return:
 *			%al = 0x0 on success; err code on failure
 */

	movb	$0x42, %ah
	int	$0x13

	jc	read_error

	movw	$BUFFERSEG, %bx      /*缓冲区地址保存*/
	jmp	copy_buffer          /*拷贝读出的数据*/
			
chs_mode:	
	/* lBA扇区号送eax寄存器 ，需要将LBA转换为CHS地址在读写*/
	movl	(%di), %eax

	/* edx填零*/
	xorl	%edx, %edx

	/* divide by number of sectors */
	divl	(%si)

	/* save sector start */
	movb	%dl, 10(%si)

	xorl	%edx, %edx	/* zero %edx */
	divl	4(%si)		/* divide by number of heads */

	/* save head start */
	movb	%dl, 11(%si)

	/* save cylinder start */
	movw	%ax, 12(%si)

	/* do we need too many cylinders? */
	cmpw	8(%si), %ax
	jge	geometry_error

	/* determine the maximum sector length of this read */
	movw	(%si), %ax	/* get number of sectors per track/head */

	/* subtract sector start */
	subb	10(%si), %al

	/* how many do we really want to read? */
	cmpw	%ax, 4(%di)	/* compare against total number of sectors */

	/* which is greater? */
	jg	2f

	/* if less than, set to total */
	movw	4(%di), %ax

2:	
	/* subtract from total */
	subw	%ax, 4(%di)

	/* add into logical sector start */
	addl	%eax, (%di)

/*
 *  This is the loop for taking care of BIOS geometry translation (ugh!)
 */

	/* get high bits of cylinder */
	movb	13(%si), %dl

	shlb	$6, %dl		/* shift left by 6 bits */
	movb	10(%si), %cl	/* get sector */

	incb	%cl		/* normalize sector (sectors go
					from 1-N, not 0-(N-1) ) */
	orb	%dl, %cl	/* composite together */
	movb	12(%si), %ch	/* sector+hcyl in cl, cylinder in ch */

	/* restore %dx */
	popw	%dx
	pushw	%dx

	/* head number */
	movb	11(%si), %dh

	pushw	%ax	/* save %ax from destruction! */

/*
 * BIOS call "INT 0x13 Function 0x2" to read sectors from disk into memory
 *	Call with	%ah = 0x2
 *			%al = number of sectors
 *			%ch = cylinder
 *			%cl = sector (bits 6-7 are high bits of "cylinder")
 *			%dh = head
 *			%dl = drive (0x80 for hard disk, 0x0 for floppy disk)
 *			%es:%bx = segment:offset of buffer
 *	Return:
 *			%al = 0x0 on success; err code on failure
 */

	movw	$BUFFERSEG, %bx
	movw	%bx, %es	/* load %es segment with disk buffer */

	xorw	%bx, %bx	/* %bx = 0, put it at 0 in the segment */
	movb	$0x2, %ah	/* function 2 */
	int	$0x13

	jc	read_error

	/* save source segment */
	movw	%es, %bx

copy_buffer:	

	/* load addresses for copy from disk buffer to destination */

	/*附加段数据初始化。。
	 * blocklist_default_seg:
	 *	#ifdef STAGE1_5
	 *	.word 0x220
	 *	#else
	 *	.word 0x820   段的开始地址，*16后为 8200，其中8000 - 8200被start占用
	 ******************************************************************** */
      
	movw	6(%di), %es	/* load destination segment */

	/* 读了几个扇区，吐出来。。 */
	popw	%ax

	/* determine the next possible destination address (presuming
		512 byte sectors!) */

	//逻辑左移，末尾补零，移1位相当于算术*2，这里以16进制算出下一次拷贝的起始地址；
	shlw	$5, %ax		/* shift %ax five bits to the left */
	addw	%ax, 6(%di)	/* add the corrected value to the destination
				   address for next time */
        
	/* 环境保存，因为MSG调用int10中断后可能会改变寄存器的值 */
	pusha
	pushw	%ds

	/* 由于开始用>>9 ，故此，两次shl 移动的位数为9，从而得到读出的字节数 */
	shlw	$4, %ax
	movw	%ax, %cx

	xorw	%di, %di	/* zero offset of destination addresses */
	xorw	%si, %si	/* zero offset of source addresses */
	movw	%bx, %ds	/* 读出后保存于此movw	$BUFFERSEG, %bx    缓冲区地址保存
	                         * 这里得分开，BUFFERSEG为0x7000 ，保存到16位的bx中取的是0x70，注意AT&T和MASM的字节序/ 

	cld		/* 自增方向标志*/

	/* 重复拷贝，每次一个字节，拷贝到0x8200处 */
	rep		/* sets a repeat */
	movsb		/* this runs the actual copy */

	/* restore addressing regs and print a dot with correct DS 
	   (MSG modifies SI, which is saved, and unused AX and BX) */
	popw	%ds
	MSG(notification_step)
	popa     /*环境恢复*/

	/*检查数据是否已经读完，如果没有，继续读取并拷贝 */
	cmpw	$0, 4(%di)
	jne	setup_sectors

	/* update position to load from */
	subw	$BOOTSEC_LISTSIZE, %di

	/* jump to bootloop */
	jmp	bootloop

/* END OF MAIN LOOP */

bootit:
	/* 打印\r\n后，恢复寄存器dx的值，根据是否定义了STAGE1_5跳到相应的起始地址处执行装入的stage2的内容 */

	MSG(notification_done)
	popw	%dx	/* 保证dl寄存器的内容为boot设置的初始值 */

#ifdef STAGE1_5
	ljmp	$0, $0x2200
#else /* ! STAGE1_5 */
	ljmp	$0, $0x8200
#endif /* ! STAGE1_5 */

/*
 * BIOS Geometry translation error (past the end of the disk geometry!).
 */

 /*错误处理代码，出错后执行，stop ： jmp stop 死循环；*/
geometry_error:
	MSG(geometry_error_string)
	jmp	general_error

/*
 * Read error on the disk.
 */
read_error:
	MSG(read_error_string)

general_error:
	MSG(general_error_string)

/* go here when you need to stop the machine hard after an error condition */
stop:	jmp	stop

#ifdef STAGE1_5
notification_string:	.string "Loading stage1.5"
#else
notification_string:	.string "Loading stage2"
#endif

notification_step:	.string "."
notification_done:	.string "\r\n"

geometry_error_string:	.string "Geom"
read_error_string:	.string "Read"
general_error_string:	.string " Error"

/*
 * message: write the string pointed to by %si
 *
 *   WARNING: trashes %si, %ax, and %bx
 */

	/*
	 * Use BIOS "int 10H Function 0Eh" to write character in teletype mode
	 *	%ah = 0xe	%al = character
	 *	%bh = page	%bl = foreground color (graphics modes)
	 */
1:
	movw	$0x0001, %bx
	movb	$0xe, %ah
	int	$0x10		/* display a byte */

	incw	%si
message:
	movb	(%si), %al
	cmpb	$0, %al
	jne	1b	/* if not end of string, jmp to display */
	ret
lastlist:

/*
 *  This area is an empty space between the main body of code below which
 *  grows up (fixed after compilation, but between releases it may change
 *  in size easily), and the lists of sectors to read, which grows down
 *  from a fixed top location.
 */

	.word 0
	.word 0

	. = _start + 0x200 - BOOTSEC_LISTSIZE

        /* fill the first data listing with the default */
blocklist_default_start:
	.long 2		/* this is the sector start parameter, in logical
			   sectors from the start of the disk, sector 0 */
blocklist_default_len:
			/* this is the number of sectors to read */
#ifdef STAGE1_5
	.word 0		/* the command "install" will fill this up */
#else
	.word (STAGE2_SIZE + 511) >> 9
#endif
blocklist_default_seg:
#ifdef STAGE1_5
	.word 0x220
#else
	.word 0x820	/* this is the segment of the starting address
			   to load the data into */
#endif

firstlist:	/* this label has to be after the list data!!! */
```
