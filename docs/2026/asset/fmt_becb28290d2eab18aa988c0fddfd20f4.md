

# 0x00 前言

本文不会介绍CPU特权级别、中断、MSR、段机制及页机制等相关前置知识，如果读者此前未接触过这些，建议阅读Intel SDM对应篇章或者[参阅链接](elink@1c9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2M7r3q4U0K9$3q4Y4k6h3y4D9L8%4g2V1i4K6u0W2K9h3!0Q4x3V1k6W2L8X3N6Q4x3V1j5J5x3o6p5$3i4K6u0r3x3o6c8Q4x3V1j5H3y4g2)9J5c8Y4c8Z5k6g2)9J5k6r3c8W2k6X3W2F1K9i4c8A6N6X3g2Q4x3X3c8Y4N6h3W2V1k6g2)9J5k6s2c8G2i4K6u0V1L8r3W2F1N6i4S2Q4x3X3c8K6P5i4y4@1k6h3#2Q4x3X3c8U0j5h3I4D9M7#2)9J5c8W2)9J5x3%4m8J5k6i4u0W2M7i4g2A6M7$3W2@1k6g2)9J5k6r3W2F1k6X3!0J5L8h3q4@1K9h3!0F1)<SUP>1</SUP>之后再继续下面篇幅。本文基于如下环境：

- CPU：Intel
- Kernel Version：4.15.0
- Debugging Env：Ubuntu 20.04.02 x64(Kernel Version—5.11.0)

# 0x01 INT `$0x80`

## a. 源码分析

首先从源码角度分析传统系统调用，即`int 0x80`。IDT(Interrupt Descriptor Table)建立位于`arch/x86/kernel/traps.c`中：

```
void __init trap_init(void)
{
/* Init cpu_entry_area before IST entries are set up */
setup_cpu_entry_areas();
idt_setup_traps();
/*
 * Set the IDT descriptor to a fixed read-only location, so that the
 * "sidt" instruction will not leak the location of the kernel, and
 * to defend the IDT against arbitrary memory write vulnerabilities.
 * It will be reloaded in cpu_init() */
cea_set_pte(CPU_ENTRY_AREA_RO_IDT_VADDR, __pa_symbol(idt_table),
            PAGE_KERNEL_RO);
idt_descr.address = CPU_ENTRY_AREA_RO_IDT;
/*
 * Should be a barrier for any external CPU state:
 */
cpu_init();
idt_setup_ist_traps();
x86_init.irqs.trap_init();
idt_setup_debugidt_traps();
}
```

`idt_setup_traps()`函数定义在`arch/x86/kernel/idt.c`中：

```
/**
 * idt_setup_traps - Initialize the idt table with default traps
 */
void __init idt_setup_traps(void)
{
    idt_setup_from_table(idt_table, def_idts, ARRAY_SIZE(def_idts), true);
}
```

其调用`idt_setup_from_table`函数同样位于该文件：

```
static void
idt_setup_from_table(gate_desc *idt, const struct idt_data *t, int size, bool sys)
{
    gate_desc desc;
    for (; size > 0; t++, size--) {
        idt_init_desc(&desc, t);
        write_idt_entry(idt, t->vector, &desc);
        if (sys)
            set_bit(t->vector, system_vectors);
    }
}
```

`def_idts`存储了IDT各项默认值，其定义如下：

```
/*
 * The default IDT entries which are set up in trap_init() before
 * cpu_init() is invoked. Interrupt stacks cannot be used at that point and
 * the traps which use them are reinitialized with IST after cpu_init() has
 * set up TSS.
 */
static const __initconst struct idt_data def_idts[] = {
    INTG(X86_TRAP_DE, divide_error),
    INTG(X86_TRAP_NMI, nmi),
    INTG(X86_TRAP_BR, bounds),
    INTG(X86_TRAP_UD, invalid_op),
    INTG(X86_TRAP_NM, device_not_available),
    INTG(X86_TRAP_OLD_MF, coprocessor_segment_overrun),
    INTG(X86_TRAP_TS, invalid_TSS),
    INTG(X86_TRAP_NP, segment_not_present),
    INTG(X86_TRAP_SS, stack_segment),
    INTG(X86_TRAP_GP, general_protection),
    INTG(X86_TRAP_SPURIOUS, spurious_interrupt_bug),
    INTG(X86_TRAP_MF, coprocessor_error),
    INTG(X86_TRAP_AC, alignment_check),
    INTG(X86_TRAP_XF, simd_coprocessor_error),
#ifdef CONFIG_X86_32
    TSKG(X86_TRAP_DF, GDT_ENTRY_DOUBLEFAULT_TSS),
#else
    INTG(X86_TRAP_DF, double_fault),
#endif
    INTG(X86_TRAP_DB, debug),
#ifdef CONFIG_X86_MCE
    INTG(X86_TRAP_MC, &machine_check),
#endif
    SYSG(X86_TRAP_OF, overflow),
#if defined(CONFIG_IA32_EMULATION)
    SYSG(IA32_SYSCALL_VECTOR, entry_INT80_compat),
#elif defined(CONFIG_X86_32)
    SYSG(IA32_SYSCALL_VECTOR, entry_INT80_32),
#endif
};
```

根据配置选项不同，`IA32_SYSCALL_VECTOR`项值不同——若启用`CONFIG_IA32_EMULATION`，则以64位兼容模式运行32位程序；否则是32位。`IA32_SYSCALL_VECTOR`定义如下：

```
#define IA32_SYSCALL_VECTOR 0x80
```

`INTG`与`SYSG`定义不同之处在于DPL：

```
/* Interrupt gate */
#define INTG(_vector, _addr) \
    G(_vector, _addr, DEFAULT_STACK, GATE_INTERRUPT, DPL0, __KERNEL_CS)
/* System interrupt gate */
#define SYSG(_vector, _addr) \
    G(_vector, _addr, DEFAULT_STACK, GATE_INTERRUPT, DPL3, __KERNEL_CS)
```

相关定义如下：

```
#define DPL0 0x0
#define DPL3 0x3
#define DEFAULT_STACK 0
#define G(_vector, _addr, _ist, _type, _dpl, _segment) \
{ \
    .vector = _vector, \
    .bits.ist = _ist, \
    .bits.type = _type, \
    .bits.dpl = _dpl, \
    .bits.p = 1, \
    .addr = _addr, \
    .segment = _segment, \
}
```

门描述符及类型定义如下(位于`/arch/x86/include/asm/desc_defs.h`)：

```
struct gate_struct {
    u16 offset_low;
    u16 segment;
    struct idt_bits bits;
    u16 offset_middle;
#ifdef CONFIG_X86_64
    u32 offset_high;
    u32 reserved;
#endif
} __attribute__((packed));

enum {
    GATE_INTERRUPT = 0xE,
    GATE_TRAP = 0xF,
    GATE_CALL = 0xC,
    GATE_TASK = 0x5,
};
```

对应于Intel SDM中：

![](upload/attach/202110/817966_44KYSQFYZR45WV9.jpg)

`idt_init_desc`函数定义如下：

```
static inline void idt_init_desc(gate_desc *gate, const struct idt_data *d)
{
    unsigned long addr = (unsigned long) d->addr;
    gate->offset_low = (u16) addr;
    gate->segment = (u16) d->segment;
    gate->bits = d->bits;
    gate->offset_middle = (u16) (addr >> 16);
#ifdef CONFIG_X86_64
    gate->offset_high = (u32) (addr >> 32);
    gate->reserved = 0;
#endif
}
```

`write_idt_entry`是`memcpy`函数的简单包装：

```
#define write_idt_entry(dt, entry, g) native_write_idt_entry(dt, entry, g)
......
static inline void native_write_idt_entry(gate_desc *idt, int entry, const gate_desc *gate)
{
    memcpy(&idt[entry], gate, sizeof(*gate));
}
```

如此一来，便在IDT 0x80项写入了系统调用函数地址。上述函数调用关系为：

![](upload/attach/202110/817966_3PCHPPYBU8MXTVZ.jpg)

---

`entry_INT80_32`定义位于`arch/x86/entry/entry_32.S`文件中：

```
ENTRY(entry_INT80_32)
    ASM_CLAC
    pushl %eax /* pt_regs->orig_ax */
    SAVE_ALL pt_regs_ax=$-ENOSYS /* save rest */
    /*
     * User mode is traced as though IRQs are on, and the interrupt gate
     * turned them off.
     */
    TRACE_IRQS_OFF
    movl %esp, %eax
    call do_int80_syscall_32
.Lsyscall_32_done:
restore_all:
    TRACE_IRQS_IRET
.Lrestore_all_notrace:
#ifdef CONFIG_X86_ESPFIX32
    ALTERNATIVE "jmp .Lrestore_nocheck", "", X86_BUG_ESPFIX
    movl PT_EFLAGS(%esp), %eax # mix EFLAGS, SS and CS
    /*
     * Warning: PT_OLDSS(%esp) contains the wrong/random values if we
     * are returning to the kernel.
     * See comments in process.c:copy_thread() for details.
     */
    movb PT_OLDSS(%esp), %ah
    movb PT_CS(%esp), %al
    andl $(X86_EFLAGS_VM | (SEGMENT_TI_MASK << 8) | SEGMENT_RPL_MASK), %eax
    cmpl $((SEGMENT_LDT << 8) | USER_RPL), %eax
    je .Lldt_ss # returning to user-space with LDT SS
#endif
.Lrestore_nocheck:
    RESTORE_REGS 4 # skip orig_eax/error_code
.Lirq_return:
    INTERRUPT_RETURN
.section .fixup, "ax"
ENTRY(iret_exc )
    pushl $0 # no error code
    pushl $do_iret_error
    jmp common_exception
.previous
    _ASM_EXTABLE(.Lirq_return, iret_exc)
#ifdef CONFIG_X86_ESPFIX32
.Lldt_ss:
    /*
     * Setup and switch to ESPFIX stack
     *
     * We're returning to userspace with a 16 bit stack. The CPU will not
     * restore the high word of ESP for us on executing iret... This is an
     * "official" bug of all the x86-compatible CPUs, which we can work
     * around to make dosemu and wine happy. We do this by preloading the
     * high word of ESP with the high word of the userspace ESP while
     * compensating for the offset by changing to the ESPFIX segment with
     * a base address that matches for the difference.
     */
#define GDT_ESPFIX_SS PER_CPU_VAR(gdt_page) + (GDT_ENTRY_ESPFIX_SS * 8)
    mov %esp, %edx /* load kernel esp */
    mov PT_OLDESP(%esp), %eax /* load userspace esp */
    mov %dx, %ax /* eax: new kernel esp */
    sub %eax, %edx /* offset (low word is 0) */
    shr $16, %edx
    mov %dl, GDT_ESPFIX_SS + 4 /* bits 16..23 */
    mov %dh, GDT_ESPFIX_SS + 7 /* bits 24..31 */
    pushl $__ESPFIX_SS
    pushl %eax /* new kernel esp */
    /*
     * Disable interrupts, but do not irqtrace this section: we
     * will soon execute iret and the tracer was already set to
     * the irqstate after the IRET:
     */
    DISABLE_INTERRUPTS(CLBR_ANY)
    lss (%esp), %esp /* switch to espfix segment */
    jmp .Lrestore_nocheck
#endif
ENDPROC(entry_INT80_32)
```

执行系统调用的主要代码位于`do_int80_syscall_32`(`arch/x86/entry/common.c`)：
