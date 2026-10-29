
7. [x86-64 Spec addition - SwapGS instruction](elink@d70K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6W2j5W2)9J5k6h3q4J5j5$3S2A6N6X3g2Q4x3X3g2G2M7X3N6Q4x3V1k6%4k6h3u0Q4x3V1j5J5x3o6p5$3x3o6M7J5z5e0p5I4x3U0f1@1x3q4)9J5c8X3S2@1N6s2m8Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2^5z5o6k6Q4x3X3b7$3y4q4)9J5k6h3!0J5k6#2)9J5c8Y4m8A6M7r3g2J5L8h3q4A6L8q4)9J5c8X3c8A6M7$3y4#2M7%4y4Q4x3V1j5J5x3o6l9H3i4K6u0V1e0$3y4@1L8$3u0W2M7W2)9J5c8U0l9H3x3e0l9H3z5g2)9J5k6h3S2@1L8h3H3`

8. [Linux系统调用过程分析](elink@3b6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0M7&6x3U0x3$3x3U0l9%4)

9. [Timers and time management in the Linux kernel. Part 7](elink@3c4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4j5i4u0D9K9h3y4K6M7r3q4U0k6g2)9J5k6h3y4G2L8g2)9J5c8U0t1H3x3U0m8Q4x3V1j5H3y4W2)9J5c8U0l9%4i4K6u0r3L8r3W2F1N6i4S2Q4x3U0g2q4y4g2)9J5y4e0R3$3i4K6t1#2z5o6g2Q4x3U0g2q4y4W2)9J5y4f1p5H3i4K6t1#2b7U0S2Q4x3U0g2q4y4q4)9J5y4f1t1^5i4K6t1#2b7f1c8Q4x3U0g2q4y4#2)9J5y4e0W2m8i4K6t1#2z5o6c8Q4x3U0g2q4y4g2)9J5y4f1q4q4i4K6t1#2z5f1q4Q4x3U0g2q4y4W2)9J5y4e0V1%4i4K6t1#2b7U0k6Q4x3U0g2q4y4g2)9J5y4e0V1&6i4K6t1#2b7e0S2Q4x3U0g2q4y4g2)9J5y4e0V1J5i4K6t1#2z5p5y4Q4x3U0g2q4y4W2)9J5y4e0V1%4i4K6t1#2b7U0k6Q4x3U0g2q4z5g2)9J5y4e0V1%4i4K6t1#2b7U0c8Q4x3U0g2q4y4#2)9J5y4f1q4q4i4K6t1#2b7e0q4Q4x3U0g2q4y4#2)9J5y4e0V1H3i4K6t1#2z5o6k6Q4x3X3c8H3j5i4u0@1i4K6u0V1y4#2)9J5c8R3`.`.)

# 0x00 前言

本文不会介绍CPU特权级别，中断，MSR，段机制及页机制等相关前置知识，如果读者此前未接触过这些，建议阅读Intel SDM对应篇章或者[参阅链接](https://bbs.kanxue.com/elink@81bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2M7r3q4U0K9$3q4Y4k6h3y4D9L8%4g2V1i4K6u0W2K9h3!0Q4x3V1k6W2L8X3N6Q4x3V1j5J5x3o6p5$3i4K6u0r3x3o6c8Q4x3V1j5H3y4g2)9J5c8Y4c8Z5k6g2)9J5k6r3c8W2k6X3W2F1K9i4c8A6N6X3g2Q4x3X3c8Y4N6h3W2V1k6g2)9J5k6s2c8G2i4K6u0V1L8r3W2F1N6i4S2Q4x3X3c8K6P5i4y4@1k6h3#2Q4x3X3c8U0j5h3I4D9M7#2)9J5c8W2)9J5x3%4m8J5k6i4u0W2M7i4g2A6M7$3W2@1k6g2)9J5k6r3W2F1k6X3!0J5L8h3q4@1K9h3!0F1)<SUP>1</SUP>之后再继续下面篇幅。本文基于如下环境：

- CPU：Intel
- Kernel Version：4.15.0
- Debugging Env：Ubuntu 20.04.02 x64(Kernel Version—5.11.0)

# 0x01 INT $0x80

### a. 源码分析

首先从源码角度分析传统系统调用，即`int 0x80`。IDT(Interrupt Descriptor Table)建立位于`arch/x86/kernel/traps.c`中：

```void __init trap_init(void)
{
/* Init cpu_entry_area before IST entries are
set up */
setup_cpu_entry_areas();
  idt_setup_traps();
/*
* Set the IDT descriptor to a fixed read-only location, so that the
* "sidt" instruction will not leak the location of the kernel, and
* to defend the IDT against arbitrary memory write vulnerabilities.
* It will be reloaded in cpu_init()
*/
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

```/*
* idt_setup_traps - Initialize the idt table with default traps
*/
void __init idt_setup_traps(void)
{
idt_setup_from_table(idt_table, def_idts, ARRAY_SIZE(def_idts), true);
}
```

其调用`idt_setup_from_table`函数同样位于该文件：

```static void
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

```/*
* The default IDT entries which are set up in trap_init() before
* cpu_init() is invoked. Interrupt stacks cannot be used at that point
and
* the traps which use them are reinitialized with IST after cpu_init() has
* set up TSS.
*/
static const __initconst struct idt_data def_idts[] = {
INTG(X86_TRAP_DE,        divide_error),
INTG(X86_TRAP_NMI,        nmi),
INTG(X86_TRAP_BR,        bounds),
INTG(X86_TRAP_UD,        invalid_op),
INTG(X86_TRAP_NM,        device_not_available),
INTG(X86_TRAP_OLD_MF,        coprocessor_segment_overrun),
INTG(X86_TRAP_TS,        invalid_TSS),
INTG(X86_TRAP_NP,        segment_not_present),
INTG(X86_TRAP_SS,        stack_segment),
INTG(X86_TRAP_GP,        general_protection),
INTG(X86_TRAP_SPURIOUS,        spurious_interrupt_bug),
INTG(X86_TRAP_MF,        coprocessor_error),
INTG(X86_TRAP_AC,        alignment_check),
INTG(X86_TRAP_XF,        simd_coprocessor_error),
#ifdef CONFIG_X86_32
TSKG(X86_TRAP_DF,        GDT_ENTRY_DOUBLEFAULT_TSS),
#else
INTG(X86_TRAP_DF,        double_fault),
#endif
INTG(X86_TRAP_DB,        debug),
#ifdef CONFIG_X86_MCE
INTG(X86_TRAP_MC,        &machine_check),
#endif
SYSG(X86_TRAP_OF,        overflow),
#if defined(CONFIG_IA32_EMULATION)
SYSG(IA32_SYSCALL_VECTOR,    entry_INT80_compat),
#elif defined(CONFIG_X86_32)
SYSG(IA32_SYSCALL_VECTOR,    entry_INT80_32),
#endif
};
```

根据配置选项不同，`IA32_SYSCALL_VECTOR`项值不同——若启用`CONFIG_IA32_EMULATION`，则以64位兼容模式运行32位程序；否则是32位。`IA32_SYSCALL_VECTOR`定义如下：

```#define IA32_SYSCALL_VECTOR        0x80
```

`INTG`与`SYSG`定义不同之处在于DPL：

```/* Interrupt gate */
#define INTG(_vector, _addr)                \
G(_vector, _addr, DEFAULT_STACK, GATE_INTERRUPT, DPL0, __KERNEL_CS)
/* System interrupt gate */
#define SYSG(_vector, _addr)                \
G(_vector, _addr, DEFAULT_STACK, GATE_INTERRUPT, DPL3, __KERNEL_CS)
```

相关定义如下：

```#define DPL0        0x0
#define DPL3        0x3
#define DEFAULT_STACK    0
#define G(_vector, _addr, _ist, _type, _dpl, _segment)    \
{                        \
.vector = _vector,        \
.bits.ist = _ist,            \
.bits.type    = _type,        \
.bits.dpl = _dpl,            \
.bits.p = 1,            \
.addr = _addr,        \
.segment = _segment,        \
}
```

门描述符及类型定义如下(位于`/arch/x86/include/asm/desc_defs.h`)：

```struct gate_struct {
u16        offset_low;
u16        segment;
struct idt_bits    bits;
u16        offset_middle;
#ifdef CONFIG_X86_64
u32        offset_high;
u32        reserved;
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

![](https://bbs.kanxue.com/upload/attach/202110/817966_44KYSQFYZR45WV9.jpg)

`idt_init_desc`函数定义如下：

```static inline void idt_init_desc(gate_desc *gate, const struct idt_data *d)
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
