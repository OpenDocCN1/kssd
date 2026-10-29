
```c
\* head.s contains the 32-bit startup code.
\*
\* NOTE!!! Startup happens at absolute address 0x00000000, which is also where
\* the page directory will exist. The startup code will be overwritten by
\* the page directory.
\*/
.text
.globl \_idt,\_gdt,\_pg\_dir,\_tmp\_floppy\_area
\_pg\_dir:
startup\_32:
movl $0x10,%eax
mov %ax,%ds
mov %ax,%es
mov %ax,%fs
mov %ax,%gs //上面是重新加载寄存器
lss \_stack\_start,%esp //lss \_stack\_start,%esp是将结构体 stact\_start 的值传送到ss:esp，即令 ss=0x10（段选择子）和 esp=& user\_stack [PAGE\_SIZE>>2]
call setup\_idt //设置idt和gdt
call setup\_gdt
movl $0x10,%eax # reload all the segment registers
mov %ax,%ds # after changing gdt. CS was already
mov %ax,%es # reloaded in 'setup\_gdt'
mov %ax,%fs
mov %ax,%gs
lss \_stack\_start,%esp
xorl %eax,%eax
1: incl %eax # check that A20 really IS enabled
movl %eax,0x000000 # loop forever if it isn't
cmpl %eax,0x100000
je 1b
/\*
\* NOTE! 486 should set bit 16, to check for write-protect in supervisor
\* mode. Then it would be unnecessary with the "verify\_area()"-calls.
\* 486 users probably want to set the NE (#5) bit also, so as to use
\* int 16 for math errors.
\*/
movl %cr0,%eax # check math chip
andl $0x80000011,%eax # Save PG,PE,ET
/\* "orl $0x10020,%eax" here for 486 might be good \*/
orl $2,%eax # set MP
movl %eax,%cr0
call check\_x87
jmp after\_page\_tables
/\*
\* We depend on ET to be correct. This checks for 287/387.
\*/
check\_x87:
fninit
fstsw %ax
cmpb $0,%al
je 1f /\* no coprocessor: have to set bits \*/
movl %cr0,%eax
xorl $6,%eax /\* reset MP, set EM \*/
movl %eax,%cr0
ret
.align 2
1: .byte 0xDB,0xE4 /\* fsetpm for 287, ignored by 387 \*/
ret
/\*
\* setup\_idt
\*
\* sets up a idt with 256 entries pointing to
\* ignore\_int, interrupt gates. It then loads
\* idt. Everything that wants to install itself
\* in the idt-table may do so themselves. Interrupts
\* are enabled elsewhere, when we can be relatively
\* sure everything is ok. This routine will be over-
\* written by the page tables.
\*/
setup\_idt:
lea ignore\_int,%edx //将ignore\_int的有效地址存到edx
movl $0x00080000,%eax //将0x8000放入eax的高16位
movw %dx,%ax /\* selector = 0x0008 = cs 将ignore\_int有效地址存到eax低16字节 \*/
movw $0x8E00,%dx /\* interrupt gate - dpl=0, present \*/
lea \_idt,%edi //
mov $256,%ecx
rp\_sidt:
movl %eax,(%edi)
movl %edx,4(%edi)
addl $8,%edi
dec %ecx
jne rp\_sidt
lidt idt\_descr
ret
/\*
\* setup\_gdt
\*
\* This routines sets up a new gdt and loads it.
\* Only two entries are currently built, the same
\* ones that were built in init.s. The routine
\* is VERY complicated at two whole lines, so this
\* rather long comment is certainly needed :-).
\* This routine will beoverwritten by the page tables.
\*/
setup\_gdt:
lgdt gdt\_descr
ret
/\*
\* I put the kernel page tables right after the page directory,
\* using 4 of them to span 16 Mb of physical memory. People with
\* more than 16MB will have to expand this.
\*/
.org 0x1000
pg0:
.org 0x2000
pg1:
.org 0x3000
pg2:
.org 0x4000
pg3:
.org 0x5000
/\*
\* tmp\_floppy\_area is used by the floppy-driver when DMA cannot
\* reach to a buffer-block. It needs to be aligned, so that it isn't
\* on a 64kB border.
\*/
\_tmp\_floppy\_area:
.fill 1024,1,0
after\_page\_tables:
pushl $0 # These are the parameters to main :-)
pushl $0
pushl $0
pushl $L6 # return address for main, if it decides to.
pushl $\_main
jmp setup\_paging
L6:
jmp L6 # main should never return here, but
# just in case, we know what happens.
/\* This is the default interrupt "handler" :-) \*/
int\_msg:
.asciz "Unknown interrupt\n\r"
.align 2
ignore\_int:
pushl %eax
pushl %ecx
pushl %edx
push %ds
push %es
push %fs
movl $0x10,%eax
mov %ax,%ds
mov %ax,%es
mov %ax,%fs
pushl $int\_msg
call \_printk
popl %eax
pop %fs
pop %es
pop %ds
popl %edx
popl %ecx
popl %eax
iret
/\*
\* Setup\_paging
\*
\* This routine sets up paging by setting the page bit
\* in cr0. The page tables are set up, identity-mapping
\* the first 16MB. The pager assumes that no illegal
\* addresses are produced (ie >4Mb on a 4Mb machine).
\*
\* NOTE! Although all physical memory should be identity
\* mapped by this routine, only the kernel page functions
\* use the >1Mb addresses directly. All "normal" functions
\* use just the lower 1Mb, or the local data space, which
\* will be mapped to some other place - mm keeps track of
\* that.
\*
\* For those with more memory than 16 Mb - tough luck. I've
\* not got it, why should you :-) The source is here. Change
\* it. (Seriously - it shouldn't be too difficult. Mostly
\* change some constants etc. I left it at 16Mb, as my machine
\* even cannot be extended past that (ok, but it was cheap :-)
\* I've tried to show which constants to change by having
\* some kind of marker at them (search for "16Mb"), but I
\* won't guarantee that's all :-( )
\*/
.align 2
setup\_paging:
movl $1024\*5,%ecx /\* 5 pages - pg\_dir+4 page tables \*/
xorl %eax,%eax
xorl %edi,%edi /\* pg\_dir is at 0x000 \*/
cld;rep;stosl
movl $pg0+7,\_pg\_dir /\* set present bit/user r/w \*/
movl $pg1+7,\_pg\_dir+4 /\* --------- " " --------- \*/
movl $pg2+7,\_pg\_dir+8 /\* --------- " " --------- \*/
movl $pg3+7,\_pg\_dir+12 /\* --------- " " --------- \*/
movl $pg3+4092,%edi
movl $0xfff007,%eax /\* 16Mb - 4096 + 7 (r/w user,p) \*/
std
1: stosl /\* fill pages backwards - more efficient :-) \*/
subl $0x1000,%eax
jge 1b
xorl %eax,%eax /\* pg\_dir is at 0x0000 \*/
movl %eax,%cr3 /\* cr3 - page directory start \*/
movl %cr0,%eax
orl $0x80000000,%eax
movl %eax,%cr0 /\* set paging (PG) bit \*/
ret /\* this also flushes prefetch-queue \*/
.align 2
.word 0
idt\_descr:
.word 256\*8-1 # idt contains 256 entries
.long \_idt
.align 2
.word 0
gdt\_descr:
.word 256\*8-1 # so does gdt (not that that's any
.long \_gdt # magic number, but it works for me :^)
.align 3
\_idt: .fill 256,8,0 # idt is uninitialized
\_gdt: .quad 0x0000000000000000 /\* NULL descriptor \*/
.quad 0x00c09a0000000fff /\* 16Mb \*/
.quad 0x00c0920000000fff /\* 16Mb \*/
.quad 0x0000000000000000 /\* TEMPORARY - don't use \*/
.fill 252,8,0
```

## 5、`main.c`(WORKING)

```c
void main(void) /\* This really IS void, no error here. \*/
{ /\* The startup routine assumes (well, ...) this \*/
/\*
\* Interrupts are still disabled. Do necessary setups, then
\* enable them
\*/
ROOT\_DEV = ORIG\_ROOT\_DEV;
drive\_info = DRIVE\_INFO;
memory\_end = (1<<20) + (EXT\_MEM\_K<<10);
memory\_end &= 0xfffff000;
if (memory\_end > 16\*1024\*1024)
memory\_end = 16\*1024\*1024;
if (memory\_end > 12\*1024\*1024)
buffer\_memory\_end = 4\*1024\*1024;
else if (memory\_end > 6\*1024\*1024)
buffer\_memory\_end = 2\*1024\*1024;
else
buffer\_memory\_end = 1\*1024\*1024;
main\_memory\_start = buffer\_memory\_end;
#ifdef RAMDISK
main\_memory\_start += rd\_init(main\_memory\_start, RAMDISK\*1024);
#endif
mem\_init(main\_memory\_start,memory\_end);
trap\_init();
blk\_dev\_init();
chr\_dev\_init();
tty\_init();
time\_init();
sched\_init();
buffer\_init(buffer\_memory\_end);
hd\_init();
floppy\_init();
sti();
move\_to\_user\_mode();
if (!fork()) { /\* we count on this going ok \*/
init(); //init函数在三.1有分析
}
/\*
\* NOTE!! For any other task 'pause()' would mean we have to get a
\* signal to awaken, but task0 is the sole exception (see 'schedule()')
\* as task 0 gets activated at every idle moment (when no other tasks
\* can run). For task0 'pause()' just means we go check if some other
\* task can run, and if not we return here.
\*/
for(;;) pause();
}
```

# 五：信号概述

内核的信号量是很重要的，关于信号的定义在`/include/signal.h`文件内，比如运行一个`elf`文件可能会出现段错误(`SIGSEGV`)，玩pwn的同学应该很熟悉。在`system_call.s`中存在`call do_signal`，那么`do_signal`在`/kernel/signal.c`内定义。

**硬件来源**：信号由硬件驱动产生

**软件来源**：系统提供了些API，例如`kill`命令

当进程收到信号时，会有三种场景；

**忽略**：忽略信号

**执行**：执行每个信号所对应的操作

**执行自定操作**：用户自定义的操作

1. 在系统中什么是信号，都有什么信号？
2. 在系统接收到信号后，是如何进行处理的？
3. 信号作用

## 1.`signal.h`
