
```c
ENTRY(entry\_SYSCALL\_64)
/\*
\* Interrupts are off on entry.
\* We do not frame this tiny irq-off block with TRACE\_IRQS\_OFF/ON,
\* it is too small to ever cause noticeable irq latency.
\*/
SWAPGS\_UNSAFE\_STACK
// KPTI 进内核态需要切到内核页表
SWITCH\_KERNEL\_CR3\_NO\_STACK
/\*
\* A hypervisor implementation might want to use a label
\* after the swapgs, so that it can do the swapgs
\* for the guest and jump here on syscall.
\*/
GLOBAL(entry\_SYSCALL\_64\_after\_swapgs)
// 将用户栈偏移保存到 per-cpu 变量 rsp\_scratch 中
movq %rsp, PER\_CPU\_VAR(rsp\_scratch)
// 加载内核栈偏移
movq PER\_CPU\_VAR(cpu\_current\_top\_of\_stack), %rsp
TRACE\_IRQS\_OFF
/\* Construct struct pt\_regs on stack \*/
pushq $\_\_USER\_DS /\* pt\_regs->ss \*/
pushq PER\_CPU\_VAR(rsp\_scratch) /\* pt\_regs->sp \*/
pushq %r11 /\* pt\_regs->flags \*/
pushq $\_\_USER\_CS /\* pt\_regs->cs \*/
pushq %rcx /\* pt\_regs->ip \*/
pushq %rax /\* pt\_regs->orig\_ax \*/
pushq %rdi /\* pt\_regs->di \*/
pushq %rsi /\* pt\_regs->si \*/
pushq %rdx /\* pt\_regs->dx \*/
pushq %rcx /\* pt\_regs->cx \*/
pushq $-ENOSYS /\* pt\_regs->ax \*/
pushq %r8 /\* pt\_regs->r8 \*/
pushq %r9 /\* pt\_regs->r9 \*/
pushq %r10 /\* pt\_regs->r10 \*/
pushq %r11 /\* pt\_regs->r11 \*/
// 为r12-r15, rbp, rbx保留位置
sub $(6\*8), %rsp /\* pt\_regs->bp, bx, r12-15 not saved \*/
/\*
\* If we need to do entry work or if we guess we'll need to do
\* exit work, go straight to the slow path.
\*/
movq PER\_CPU\_VAR(current\_task), %r11
testl $\_TIF\_WORK\_SYSCALL\_ENTRY|\_TIF\_ALLWORK\_MASK, TASK\_TI\_flags(%r11)
jnz entry\_SYSCALL64\_slow\_path
entry\_SYSCALL\_64\_fastpath:
/\*
\* Easy case: enable interrupts and issue the syscall. If the syscall
\* needs pt\_regs, we'll call a stub that disables interrupts again
\* and jumps to the slow path.
\*/
TRACE\_IRQS\_ON
ENABLE\_INTERRUPTS(CLBR\_NONE)
#if \_\_SYSCALL\_MASK == ~0
// 确保系统调用号没超过最大值，超过了则跳转到后面的符号 1 处进行返回
cmpq $\_\_NR\_syscall\_max, %rax
#else
andl $\_\_SYSCALL\_MASK, %eax
cmpl $\_\_NR\_syscall\_max, %eax
#endif
ja 1f /\* return -ENOSYS (already in pt\_regs->ax) \*/
// 除系统调用外的其他调用都通过 rcx 来传第四个参数，因此将 r10 的内容设置到 rcx
movq %r10, %rcx
/\*
\* This call instruction is handled specially in stub\_ptregs\_64.
\* It might end up jumping to the slow path. If it jumps, RAX
\* and all argument registers are clobbered.
\*/
// 调用系统调用表中对应的函数
call \*sys\_call\_table(, %rax, 8)
.Lentry\_SYSCALL\_64\_after\_fastpath\_call:
// 将函数返回值压到栈中，返回时弹出
movq %rax, RAX(%rsp)
1:
/\*
\* If we get here, then we know that pt\_regs is clean for SYSRET64.
\* If we see that no exit work is required (which we are required
\* to check with IRQs off), then we can go straight to SYSRET64.
\*/
DISABLE\_INTERRUPTS(CLBR\_NONE)
TRACE\_IRQS\_OFF
movq PER\_CPU\_VAR(current\_task), %r11
testl $\_TIF\_ALLWORK\_MASK, TASK\_TI\_flags(%r11)
jnz 1f
LOCKDEP\_SYS\_EXIT // 宏的实现与 CONFIG\_DEBUG\_LOCK\_ALLOC 内核配置选项相关，该配置允许在退出系统调用时调试锁。
TRACE\_IRQS\_ON /\* user mode is traced as IRQs on \*/
movq RIP(%rsp), %rcx
movq EFLAGS(%rsp), %r11
RESTORE\_C\_REGS\_EXCEPT\_RCX\_R11
// 恢复除 rxc 和 r11 外所有通用寄存器, 因为 rcx 寄存器为调用系统调用的应用程序的返回地址， r11 寄存器为老的 flags register
/\*
\* This opens a window where we have a user CR3, but are
\* running in the kernel. This makes using the CS
\* register useless for telling whether or not we need to
\* switch CR3 in NMIs. Normal interrupts are OK because
\* they are off here.
\*/
SWITCH\_USER\_CR3 // KPTI 返回用户态需要切回用户页表
/\* 根据压栈的内容，恢复 rsp 为用户态的栈顶 \*/
movq RSP(%rsp), %rsp
USERGS\_SYSRET64
/\* 调用宏 USERGS\_SYSRET64 ，其扩展调用 swapgs 指令交换用户 GS 和内核GS， sysret 指令执行从系统调用处理退出 \*/
........
........
```
可以看出，在入口和结束的地方都加了`SWITCH_CR3`相关的宏定义，尝试着分析`SWITCH_KERNEL_CR3_NO_STACK`，里面汇编实现如下：
```python
mov rdi, cr3
nop
nop
nop
nop
nop
and rdi, 0xFFFFFFFFFFFFE7FF
mov cr3, rdi
```
拆分`FFFFFFFFFFFFE7FF`，它的第12和13位是零，这段代码目的就是将`CR3`的第12位与第13位置零（页表的第12位在`CR4`寄存器的`PCIDE`位未开启的情况下，都是保留给OS留做他用），我们只关心13位置零，就相当于`CR3-0x1000`，从用户态`PGD`转换成内核态`PGD`。
再看`SWITCH_USER_CR3`宏定义的汇编：
```python
mov rdi, cr3
or rdi, 1000h
mov cr3, rdi
```
同理，将`CR3`第13位置1，相当于`CR3+0x1000`，从内核态`PGD`切换成用户态`PGD`。
# 0x2 Bypass KPTI
在开启`KPTI`内核，提权返回到用户态（`iretq`/`sysret`）之前如果不设置`CR3`寄存器的值，就会导致进程找不到当前程序的正确页表，引发段错误，程序退出。
知道`KPTI`原理，在kernel提权返回用户态的时候绕过`kpti`的话就很简单了，利用内核映像中现有的`gadget`
```python
mov rdi, cr3
or rdi, 1000h
mov cr3, rdi
```
来设置`CR3`寄存器，并按照`iretq`/`sysret` 的需求构造内容，再返回就OK了。
有一种比较懒惰的方法就是利用`swapgs_restore_regs_and_return_to_usermode`这个函数返回：
`cat /proc/kallsyms| grep swapgs_restore_regs_and_return_to_usermode`
```c
arch/x86/entry/entry\_64.S
SYM\_INNER\_LABEL(swapgs\_restore\_regs\_and\_return\_to\_usermode, SYM\_L\_GLOBAL)
POP\_REGS pop\_rdi=0
/\*
\* The stack is now user RDI, orig\_ax, RIP, CS, EFLAGS, RSP, SS.
\* Save old stack pointer and switch to trampoline stack.
\*/
movq %rsp, %rdi
movq PER\_CPU\_VAR(cpu\_tss\_rw + TSS\_sp0), %rsp
/\* Copy the IRET frame to the trampoline stack. \*/
pushq 6\*8(%rdi) /\* SS \*/
pushq 5\*8(%rdi) /\* RSP \*/
pushq 4\*8(%rdi) /\* EFLAGS \*/
pushq 3\*8(%rdi) /\* CS \*/
pushq 2\*8(%rdi) /\* RIP \*/
/\* Push user RDI on the trampoline stack. \*/
pushq (%rdi)
/\*
\* We are on the trampoline stack. All regs except RDI are live.
\* We can do future final exit work right here.
\*/
STACKLEAK\_ERASE\_NOCLOBBER
SWITCH\_TO\_USER\_CR3\_STACK scratch\_reg=%rdi
/\* Restore RDI. \*/
popq %rdi
SWAPGS
INTERRUPT\_RETURN
```
纯汇编代码如下：
```python
swapgs\_restore\_regs\_and\_return\_to\_usermode
.text:FFFFFFFF81600A34 41 5F pop r15
.text:FFFFFFFF81600A36 41 5E pop r14
.text:FFFFFFFF81600A38 41 5D pop r13
.text:FFFFFFFF81600A3A 41 5C pop r12
.text:FFFFFFFF81600A3C 5D pop rbp
.text:FFFFFFFF81600A3D 5B pop rbx
.text:FFFFFFFF81600A3E 41 5B pop r11
.text:FFFFFFFF81600A40 41 5A pop r10
.text:FFFFFFFF81600A42 41 59 pop r9
.text:FFFFFFFF81600A44 41 58 pop r8
.text:FFFFFFFF81600A46 58 pop rax
.text:FFFFFFFF81600A47 59 pop rcx
.text:FFFFFFFF81600A48 5A pop rdx
.text:FFFFFFFF81600A49 5E pop rsi
.text:FFFFFFFF81600A4A 48 89 E7 mov rdi, rsp <<<<<<<<<<<<<<<<<<
.text:FFFFFFFF81600A4D 65 48 8B 24 25+ mov rsp, gs: 0x5004
.text:FFFFFFFF81600A56 FF 77 30 push qword ptr [rdi+30h]
.text:FFFFFFFF81600A59 FF 77 28 push qword ptr [rdi+28h]
.text:FFFFFFFF81600A5C FF 77 20 push qword ptr [rdi+20h]
.text:FFFFFFFF81600A5F FF 77 18 push qword ptr [rdi+18h]
.text:FFFFFFFF81600A62 FF 77 10 push qword ptr [rdi+10h]
.text:FFFFFFFF81600A65 FF 37 push qword ptr [rdi]
.text:FFFFFFFF81600A67 50 push rax
.text:FFFFFFFF81600A68 EB 43 nop
.text:FFFFFFFF81600A6A 0F 20 DF mov rdi, cr3
.text:FFFFFFFF81600A6D EB 34 jmp 0xFFFFFFFF81600AA3
.text:FFFFFFFF81600AA3 48 81 CF 00 10+ or rdi, 1000h
.text:FFFFFFFF81600AAA 0F 22 DF mov cr3, rdi
.text:FFFFFFFF81600AAD 58 pop rax
.text:FFFFFFFF81600AAE 5F pop rdi
.text:FFFFFFFF81600AAF FF 15 23 65 62+ call cs: SWAPGS
.text:FFFFFFFF81600AB5 FF 25 15 65 62+ jmp cs: INTERRUPT\_RETURN
\_SWAPGS
.text:FFFFFFFF8103EFC0 55 push rbp
.text:FFFFFFFF8103EFC1 48 89 E5 mov rbp, rsp
.text:FFFFFFFF8103EFC4 0F 01 F8 swapgs
.text:FFFFFFFF8103EFC7 5D pop rbp
.text:FFFFFFFF8103EFC8 C3 retn
\_INTERRUPT\_RETURN
.text:FFFFFFFF81600AE0 F6 44 24 20 04 test byte ptr [rsp+0x20], 4
.text:FFFFFFFF81600AE5 75 02 jnz native\_irq\_return\_ldt
.text:FFFFFFFF81600AE7 48 CF iretq
```
在`ROP`时，将程序流程控制到 `mov rdi, rsp` 指令，栈布局如下就行：
```python
rsp ----> mov_rdi_rsp
0
0
rip
cs
rflags
rsp
ss
```
# 0x3 Trick
当然改`modprobe_path`也是一个不错的方法，返回后当前进程`Segmentation fault`也不影响提权。
还可以使用`signal(SIGSEGV,shell)`捕获常规方式返回导致的`SIGSEGV`异常信号来`get root shell`。
# 0x4 Reference
<a href="elink@100K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6#2j5Y4g2F1N6s2g2I4j5g2)9J5k6h3y4G2L8g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1j5I4x3U0b7J5i4K6u0W2K9s2c8E0L8l9`.`.">如何检查我的Ubuntu上是否启用了KPTI？ </a>
