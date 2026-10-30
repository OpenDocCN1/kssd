
```c
/* Do the last things and invoke protected mode */
go_to_protected_mode();
}
```

### 进入保护模式

```c
/* Actual transition to protected mode... */
setup_idt();
setup_gdt();
protected_mode_jump(boot_params.hdr.code32_start,
                    (u32)&boot_params + (ds() << 4));
```

1. 首先是`setup_idt();` 用来设置中断描述符表(IDT)。

```c
/*
 * Set up the IDT
 */
static void setup_idt(void)
{
    static const struct gdt_ptr null_idt = {0, 0};
    asm volatile("lidtl %0" : : "m" (null_idt));
}
```

`lidtl`将为空的`null_idt`载入寄存器`IDT`。

2. 接下来`setup_gdt();` 来设置全局描述符表即`GDT`表，其中定义了`boot_gdt[]`数组，这个数组中的内容就是我们要传给`GDTR`的段描述符的信息。

```c
static void setup_gdt(void)
{
    static const u64 boot_gdt[] __attribute__((aligned(16))) = {
        /* CS: code, read/execute, 4 GB, base 0 */
        [GDT_ENTRY_BOOT_CS] = GDT_ENTRY(0xc09b, 0, 0xfffff),
        /* DS: data, read/write, 4 GB, base 0 */
        [GDT_ENTRY_BOOT_DS] = GDT_ENTRY(0xc093, 0, 0xfffff),
        /* TSS: 32-bit tss, 104 bytes, base 4096 */
        /* We only have a TSS here to keep Intel VT happy;
           we don't actually use it for anything. */
        [GDT_ENTRY_BOOT_TSS] = GDT_ENTRY(0x0089, 4096, 103),
    };

    /* Xen HVM incorrectly stores a pointer to the gdt_ptr, instead
       of the gdt_ptr contents. Thus, make it static so it will
       stay in memory, at least long enough that we switch to the
       proper kernel GDT. */
    static struct gdt_ptr gdt;
    gdt.len = sizeof(boot_gdt)-1;
    gdt.ptr = (u32)&boot_gdt + (ds() << 4);
    asm volatile("lgdtl %0" : : "m" (gdt));
}
```

其中`GDT_ENTRY`是一个宏定义，传入三个参数（标志，基地址，段长度）：

```c
#define GDT_ENTRY(flags, base, limit) \
    ((((base) & _AC(0xff000000,ULL)) << (56-24)) | \
     (((flags) & _AC(0x0000f0ff,ULL)) << 40) | \
     (((limit) & _AC(0x000f0000,ULL)) << (48-16)) | \
     (((base) & _AC(0x00ffffff,ULL)) << 16) | \
     (((limit) & _AC(0x0000ffff,ULL))))
```

标志字段二进制展开后每一位都有自己的含义。之后获取`gdt`长度，以及指针，最后载入`GDTR`寄存器。

3. 最后调用`protected_mode_jump(boot_params.hdr.code32_start, (u32)&boot_params + (ds() << 4));` 完成从实模式到保护模式的跳转。

```c
/* pmjump.S */
void __attribute__((noreturn))
protected_mode_jump(u32 entrypoint, u32 bootparams);
```

接受两个参数：保护模式进入地址，`bootparams`结构的地址。

```asm
GLOBAL(protected_mode_jump)
    movl %edx, %esi # bootparams地址放入esi
    xorl %ebx, %ebx
    movw %cs, %bx # cs放入bx
    shll $4, %ebx
    addl %ebx, 2f
    jmp 1f # Short jump to serialize on 386/486
1:
    movw $__BOOT_DS, %cx
    movw $__BOOT_TSS, %di
    movl %cr0, %edx #
    orb $X86_CR0_PE, %dl # 设置 CR0 寄存器相应的位使 CPU 进入保护模式：
    movl %edx, %cr0
    # Transition to 32-bit mode
    .byte 0x66, 0xea # ljmpl opcode
2:  .long in_pm32 # offset
    .word __BOOT_CS # segment
    # 执行长跳转到代码段
ENDPROC(protected_mode_jump)
```

**跳转之后我们就在保护模式下执行以下代码了**

```asm
.code32
.section ".text32","ax"
GLOBAL(in_pm32)
    # 进入32位保护模式首先重置段寄存器
    movl %ecx, %ds
    movl %ecx, %es
    movl %ecx, %fs
    movl %ecx, %gs
    movl %ecx, %ss
    # The 32-bit code sets up its own stack, but this way we do have
    # a valid stack if some debugging hack wants to use it.
    addl %ebx, %esp
    # Set up TR to make Intel VT happy
    ltr %di
    # 清空通用寄存器
    # 32-bit boot protocol
    xorl %ecx, %ecx
    xorl %edx, %edx
    xorl %ebx, %ebx
    xorl %ebp, %ebp
    xorl %edi, %edi
    # Set up LDTR to make Intel VT happy
    lldt %cx
    jmpl *%eax # Jump to the 32-bit entrypoint
ENDPROC(in_pm32)
```

### 从32位保护模式向64位长模式切换

1. `jmpl *%eax # Jump to the 32-bit entrypoint`时`eax`存储的是32位的入口点。
2. 到达32位入口点，我们可以看到此时的目录：`arch/x86/boot/compressed/head_64.S`其中的compressed，因为`bzimage` 是由 `vmlinux` + 头文件 + 内核启动代码 被 `gzip` 压缩之后获得的。之前的属于内核启动代码。而`head_64.S`做内核解压前的准备。

```c
__HEAD
.code32
ENTRY(startup_32)
...
ENDPROC(startup_32)
```

其中`__HEAD`代表`#define __HEAD .section ".head.text","ax"`也就是说这个段是可执行的。

3. 首先调用`cld`清空`DF`，DF与串操作。
4. 通过`KEEP_SEGMENTS`标记来给段寄存器做正确的赋值。
5. 计算我们代码和编译运行之间的位置偏差。通过：定义一个标签并且跳转到它，然后把栈顶抛出到一个寄存器中。

```asm
leal (BP_scratch+4)(%esi), %esp
call 1f
1: popl %ebp
subl $1b, %ebp
```

`esi`指向`bootparams`结构体，把结构体中`scratch+4`的地址放进`esp`制造出一个4字节临时栈。然后通过`call 1f`。此时`1f`标签的地址在栈顶，再把他`pop`到`ebp`然后`ebp-1b`就得到了`startup_32`的加载地址。

6. 先找到`boot_stack_end`的实际地址，然后通过`call verify_cpu`之后`test eax`中的返回值，验证cpu是否支持长模式和SSE。如果不支持，就`hlt`掉。

7. 计算**内核解压缩地址**，此时`ebp`中的是`startup_32`的实际物理地址，当`CONFIG_RELOCATABLE`打开时，我们将`ebp`放在`ebx`中，然后做对齐（2M）然后与`$LOAD_PHYSICAL_ADDR`（对齐后内核加载位置的物理地址）比较，最后给`startup_32`加上偏移获得解压缩地址。结束后，`ebp`包含了加载时的地址，`ebx`包含了内核解压缩的目标地址。

```c
#ifdef CONFIG_RELOCATABLE
    movl %ebp, %ebx
    movl BP_kernel_alignment(%esi), %eax
    decl %eax
    addl %eax, %ebx
    notl %eax
    andl %eax, %ebx
    cmpl $LOAD_PHYSICAL_ADDR, %ebx
    jge 1f
#endif
    movl $LOAD_PHYSICAL_ADDR, %ebx
1:
    /* Target address to relocate to for decompression */
    addl $z_extract_offset, %ebx
```

8. 更新全局描述符表。全局描述符表保存在48位`GDTR`-全局描述符表寄存器中，由两个部分组成：
- `GDT`大小（16位）
- `GDT`基地址（32位）

```asm
/* Load new GDT with the 64bit segments using 32bit descriptor */
leal gdt(%ebp), %eax
movl %eax, gdt+2(%ebp)
lgdt gdt(%ebp)
```

`gdt`偏移量整体结构如下

```asm
.data /* 位于data段 */
gdt:
    .word gdt_end - gdt /* gdt整体长度 */
    .long gdt /* gdt基地址 */
    .word 0
    .quad 0x0000000000000000 /* NULL descriptor */
    .quad 0x00af9a000000ffff /* __KERNEL_CS */
    .quad 0x00cf92000000ffff /* __KERNEL_DS */
    .quad 0x0080890000000000 /* TS descriptor */
    .quad 0x0000000000000000 /* TS continued */
gdt_end:
```

9. 打开`PAE`模式。

```asm
/* Enable PAE mode */
movl %cr4, %eax
orl $X86_CR4_PAE, %eax
movl %eax, %cr4
```

10. **初期页表初始化，建立初期的4G启动页表**。linux内核使用4级页表，一般建立六个页表，每张表4k。

```c
/* Initialize Page tables to 0 */
leal pgtable(%ebx), %edi
xorl %eax, %eax
movl $((4096*6)/4), %ecx
rep stosl
```

其中`pgtable`定义如下，大小`24k`：

```asm
.section ".pgtable","a",@nobits
.balign 4096
pgtable:
.fill 6*4096, 1, 0
```

开始构建4级页表：

```asm
/* Build Level 4 */
leal pgtable + 0(%ebx), %edi
leal 0x1007 (%edi), %eax
movl %eax, 0(%edi)
```

其中`0x1007`是四级页表的大小`4096+7`（页表项标记`PRESENT+RW+USER`），最后我们把第一个`PDP`（页目录指针）项地址写入`PML4`。

11. 在`PDP`表（页目录指针表、三级页表）建立4个2级页表（Page Directory），他们都带有`PRESENT+RW+USE`标记。

```asm
/* Build Level 3 */
leal pgtable + 0x1000(%ebx), %edi
leal 0x1007(%edi), %eax /* 把第一个 2 级页目录指针表的首项的地址放到 eax 寄存器 */
movl $4, %ecx
1:  movl %eax, 0x00(%edi)
    addl $0x00001000, %eax
    addl $8, %edi /* 计算后面的几个页目录指针项的地址，每个占 8 字节 */
    decl %ecx
    jnz 1b
```

12. 建立2048个2M页表项。

```asm
/* Build Level 2 */
leal pgtable + 0x2000(%ebx), %edi
movl $0x00000183, %eax /* 标记位PRESENT + WRITE + MBZ */
movl $2048, %ecx
1:  movl %eax, 0(%edi)
    addl $0x00200000, %eax
    addl $8, %edi
    decl %ecx
    jnz 1b
```

最终我们拥有了一个4G表，映射了4G大小的内存。

13. 最后将`PML4`的地址放入`cr3`。

```asm
/* Enable the boot page tables */
leal pgtable(%ebx), %eax
movl %eax, %cr3
```
