
```
/*
 * 32-bit SYSENTER entry.
 *
 * 32-bit system calls through the vDSO's __kernel_vsyscall enter here
 * on 64-bit kernels running on Intel CPUs.
 *
 * The SYSENTER instruction, in principle, should *only* occur in the
 * vDSO.  In practice, a small number of Android devices were shipped
 * with a copy of Bionic that inlined a SYSENTER instruction.  This
 * never happened in any of Google's Bionic versions -- it only happened
 * in a narrow range of Intel-provided versions.
 *
 * SYSENTER loads SS, RSP, CS, and RIP from previously programmed MSRs.
 * IF and VM in RFLAGS are cleared (IOW: interrupts are off).
 * SYSENTER does not save anything on the stack,
 * and does not save old RIP (!!!), RSP, or RFLAGS.
 *
 * Arguments:
 * eax  system call number
 * ebx  arg1
 * ecx  arg2
 * edx  arg3
 * esi  arg4
 * edi  arg5
 * ebp  user stack
 * 0(%ebp) arg6
 */
ENTRY(entry_SYSENTER_compat)
/* Interrupts are off on entry. */
SWAPGS
/* We are about to clobber %rsp anyway, clobbering here is OK */
SWITCH_TO_KERNEL_CR3 scratch_reg=%rsp

movq    PER_CPU_VAR(cpu_current_top_of_stack), %rsp

/*
 * User tracing code (ptrace or signal handlers) might assume that
 * the saved RAX contains a 32-bit number when we're invoking a 32-bit
 * syscall.  Just in case the high bits are nonzero, zero-extend
 * the syscall number.  (This could almost certainly be deleted
 * with no ill effects.)
 */
movl    %eax, %eax

/* Construct struct pt_regs on stack */
pushq    $__USER32_DS /* pt_regs->ss */
pushq    %rbp /* pt_regs->sp (stashed in bp) */

/*
 * Push flags.  This is nasty.  First, interrupts are currently
 * off, but we need pt_regs->flags to have IF set.  Second, even
 * if TF was set when SYSENTER started, it's clear by now.  We fix
 * that later using TIF_SINGLESTEP.
 */
pushfq /* pt_regs->flags (except IF = 0) */
orl    $X86_EFLAGS_IF, (%rsp) /* Fix saved flags */
pushq    $__USER32_CS /* pt_regs->cs */
pushq    $0            /* pt_regs->ip = 0 (placeholder) */
pushq    %rax /* pt_regs->orig_ax */
pushq    %rdi /* pt_regs->di */
pushq    %rsi /* pt_regs->si */
pushq    %rdx /* pt_regs->dx */
pushq    %rcx /* pt_regs->cx */
pushq    $-ENOSYS /* pt_regs->ax */
pushq   $0            /* pt_regs->r8 = 0 */
pushq   $0            /* pt_regs->r9 = 0 */
pushq   $0            /* pt_regs->r10 = 0 */
pushq   $0            /* pt_regs->r11 = 0 */
pushq    %rbx /* pt_regs->rbx */
pushq    %rbp /* pt_regs->rbp (will be overwritten) */
pushq   $0            /* pt_regs->r12 = 0 */
pushq   $0            /* pt_regs->r13 = 0 */
pushq   $0            /* pt_regs->r14 = 0 */
pushq   $0            /* pt_regs->r15 = 0 */
cld

/*
 * SYSENTER doesn't filter flags, so we need to clear NT and AC
 * ourselves.  To save a few cycles, we can check whether
 * either was set instead of doing an unconditional popfq.
 * This needs to happen before enabling interrupts so that
 * we don't get preempted with NT set.
 *
 * If TF is set, we will single-step all the way to here -- do_debug
 * will ignore all the traps.  (Yes, this is slow, but so is
 * single-stepping in general.  This allows us to avoid having
 * a more complicated code to handle the case where a user program
 * forces us to single-step through the SYSENTER entry code.)
 *
 * NB.: .Lsysenter_fix_flags is a label with the code under it moved
 * out-of-line as an optimization: NT is unlikely to be set in the
 * majority of the cases and instead of polluting the I$ unnecessarily,
 * we're keeping that code behind a branch which will predict as
 * not-taken and therefore its instructions won't be fetched.
 */
testl    $X86_EFLAGS_NT|X86_EFLAGS_AC|X86_EFLAGS_TF, EFLAGS(%rsp)
jnz    .Lsysenter_fix_flags
.Lsysenter_flags_fixed:

/*
 * User mode is traced as though IRQs are on, and SYSENTER
 * turned them off.
 */
TRACE_IRQS_OFF

movq    %rsp, %rdi
call    do_fast_syscall_32
/* XEN PV guests always use IRET path */
ALTERNATIVE "testl %eax, %eax; jz .Lsyscall_32_done", \
    "jmp .Lsyscall_32_done", X86_FEATURE_XENPV
jmp    sysret32_from_system_call

.Lsysenter_fix_flags:
pushq    $X86_EFLAGS_FIXED
popfq
jmp    .Lsysenter_flags_fixed
GLOBAL(__end_entry_SYSENTER_compat)
ENDPROC(entry_SYSENTER_compat)
```

关于`SWAPGS`可阅读参阅链接<SUP>7</SUP>：

![](https://bbs.kanxue.com/upload/attach/202110/817966_SUSZ9EGWNZARH2X.jpg)

`do_fast_syscall_32`函数会调用`do_syscall_32_irqs_on`：

```
/*
 * Returns 0 to return using IRET or 1 to return using SYSEXIT/SYSRETL.
 */
__visible long do_fast_syscall_32(struct pt_regs *regs)
{
    ......
    /*
     * Now this is just like a normal syscall.
     */
    do_syscall_32_irqs_on(regs);
    ......
}
```

该函数其余代码部分见后文描述。

### b. 动态调试

使用如下代码作为示例(不建议这样去执行系统调用，下面的代码仅仅是作为展示)：

```
int main(int argc, char *argv[])
{
    unsigned long syscall_nr = 1;
    long exit_status = 44;

    asm ("movl %0, %%eax\n"
        "movl %1, %%ebx\n"
        "sysenter"
        : /* output parameters, we aren't outputting anything, no none */
          /* (none) */
        : /* input parameters mapped to %0 and %1, repsectively */
          "m" (syscall_nr), "m" (exit_status)
        : /* registers that we are "clobbering", unneeded since we are calling exit */
          "eax", "ebx");
}
```

于`entry_SYSENTER_compat`成功断下：

![](https://bbs.kanxue.com/upload/attach/202110/817966_6WV3PED7HKPBU7X.jpg)

将`regs`传递给`do_fast_syscall_32`：

![](https://bbs.kanxue.com/upload/attach/202110/817966_3K5HAZ9R8W9EK7F.jpg)

![](https://bbs.kanxue.com/upload/attach/202110/817966_4RPH686ZQA8546H.jpg)

可以看到其`orig_ax`成员偏移与之前相比发生了变化， 这是因为`regs`对应结构定义为：
