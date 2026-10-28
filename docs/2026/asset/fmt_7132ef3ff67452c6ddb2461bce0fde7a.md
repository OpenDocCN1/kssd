
```asm
mrs x9, CurrentEL 

cmp x9, #(0b01 << 2)

//不等于 0 时，说明不是在异常级别 1，跳转到 notEL1 代码

bne .notEL1
```

### 循环遍历

```asm
str     xzr, [page_table1, tmp, lsl #3]

add     tmp, tmp, #1

cmp     tmp, #MMU_KERNEL_PAGE_TABLE_ENTRIES_TOP

bne     .Lclear_top_page_table_loop
```

等价于

```c
for(int tmp = 0;tmp < MMU_KERNEL_PAGE_TABLE_ENTRIES_TOP;tmp++) {
     page_table1[tmp] = 0;
}
```

# 启动过程概述

启动早期，即内核在进入 C++ 世界之前，主要分为以下几步

- 初始化各个 EL1 - EL3 下的异常配置
- 创建启动阶段页表
- 为打开 MMU 做准备
- 打开 MMU
- 配置栈准备进入 C 世界

# 启动时序与代码

在多核处理器架构中，很多初始化代码仅需要由 prime 处理器完成，其他处理器完成各自的配置即可。

<table>

 <tr>

 <td bgcolor=#00F5FF>prime 核心</td>

 <td bgcolor=#00F5FF>其他核心</td>

 </tr>

 <tr>

 <td>保存内核启动参数</td>

 <td>跳过</td>

 </tr>

 <tr>

 <td colspan="2">初始化 EL1 - EL3 的异常配置</td>

 </tr>

 <tr>

 <td colspan="2">初始化缓存</td>

 </tr>

 <tr>

 <td>修复 kernel base 地址 </td>

 <td>跳过</td>

 </tr>

 <tr>

 <td>检查并等待 .bss 段数据清除</td>

 <td>跳过</td>

 </tr>

 <tr>

 <td>创建启动阶段的页表</td>

 <td>自旋等待页表创建完成</td>

 </tr>

 <tr>

 <td colspan="2">打开 MMU 之前的准备工作</td>

 </tr>

 <tr>

 <td colspan="2">打开 MMU (以上代码运行在物理地址，以下代码运行在虚拟地址)</td>

 </tr>

 <tr>

 <td>重新配置内核栈指针</td>

 <td>配置其他 CPU 的栈指针</td>

 </tr>

 <tr>

 <td>跳转到 C 世界继续初始化</td>

 <td>休眠等待唤醒</td>

 </tr>

</table>

## 保存内核启动参数

`start.S` - `\_start`

```asm
    mrs     cpuid, mpidr_el1 //aff0 记录 cpu ID，aff1 记录是否支持超线程

    cbnz    cpuid, .Lno_save_bootinfo //如果不是 prim 核心(0 号核心)，则不需要启动内核，也就不需要准备内核启动参数，直接执行核心初始化工作

    /* save x0 in zbi_paddr */
    //prim 核心走这里，准备并保存内核启动参数
    //计算 zbi_paddr 段中数据的地址，保存在 x0 中，下同

    adrp    tmp, zbi_paddr

    str     x0, [tmp, #:lo12:zbi_paddr]

    /* save entry point physical address in kernel_entry_paddr */

    adrp    tmp, kernel_entry_paddr

    adr     tmp2, _start

    str     tmp2, [tmp, #:lo12:kernel_entry_paddr]

    adrp    tmp2, arch_boot_el

    mrs     x2, CurrentEL

    str     x2, [tmp2, #:lo12:arch_boot_el]

    //总之，x0 - x4 现在保存了核心初始化需要的参数，为跳转到 C 世界作准备。
```

## 初始化 EL1 - EL3

`asm.S` - `arm64\_elX\_to\_el1`

对各个 EL 的配置，需要 CPU 在对应的 EL 状态下才能配置

### EL1

EL1 不需要配置直接返回

```asm
    //读取现在的异常级别
    mrs x9, CurrentEL 
    cmp x9, #(0b01 << 2)
    //不等于 0 时，说明不是在异常级别 1，跳转到 notEL1 代码
    bne .notEL1 
    /* Already in EL1 */
    //EL1 直接返回
    ret
```

### EL2

EL2 状态下主要配置了：

- 配置 EL2 的异常向量表
- 配置时钟
- 清除 EL2 的转换表寄存器
- 配置 SPSR 和 ELR 寄存器，这两个看上面的寄存器介绍

实际上 EL2 在 Zircon 中还没有具体用处，所以此处初始化基本上就是设一些空值。

```asm
/* Setup the init vector table for EL2. */
    //计算EL2的异常向量表的基地址
    adr_global x9, arm64_el2_init_table
    //设定EL2的异常向量表的基地址
    msr vbar_el2, x9

    /* Ensure EL1 timers are properly configured, disable EL2 trapping of
        EL1 access to timer control registers.  Also clear virtual offset.
    */
    //检查并配置时钟
    mrs x9, cnthctl_el2
    orr x9, x9, #3
    msr cnthctl_el2, x9
    msr cntvoff_el2, xzr

    /* clear out stage 2 translations */
    //清除 vttbr_el2 寄存器，vttbr_el2 保存了转换表的基地址，负责在 EL2 下进行 EL0 -> EL1 的非安全存储器访问的转换
    msr vttbr_el2, xzr 

    //当系统发生了异常并进入EL2，SPSR_EL2，Saved Program Status Register (EL2)会保存处理器状态，ELR_EL2，Exception Link Register (EL2)会保存返回发生exception的现场的返回地址。
    //这里是设定SPSR_EL2和ELR_EL2的初始值。
    adr x9, .Ltarget
    msr elr_el2, x9
    //ELR 定义看上面
    mov x9, #((0b1111 << 6) | (0b0101)) /* EL1h runlevel */
    msr spsr_el2, x9
```

### EL3

EL3 状态的主要任务就是配置 EL0/EL1 的 Secure State/HVC/运行指令集，其他的也是上面 EL2 一样付空值。

- 设置 EL0/EL1 为 non-Secure State
- 开启 HVC 指令
- 使用 AARCH64 指令

```asm
   cmp x9, #(0b10 << 2)
    //当前为异常级别 2，跳转到 inEL2
    beq .inEL2
    //当不在 EL2 状态时，则为 EL3
    /* set EL2 to 64bit and enable HVC instruction */
    //scr_el3 控制EL0/EL1/EL2的异常路由  逻辑1允许
    //5f7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3W2F1k6X3!0U0k6h3&6@1k6i4u0Q4x3X3g2S2M7X3#2Q4x3X3g2U0L8$3#2Q4x3V1k6Z5k6h3I4H3i4K6u0r3K9h3&6V1k6i4S2Q4x3X3g2B7M7%4m8Q4x3@1k6@1L8%4m8A6j5#2)9K6c8q4)9J5c8X3y4G2L8g2)9J5k6h3q4J5L8g2)9J5k6h3c8G2j5#2)9J5k6e0p5H3x3o6b7H3x3#2)9#2k6U0l9J5x3o6m8Q4y4h3j5H3x3q4)9#2k6X3g2F1i4K6u0r3L8r3q4#2x3e0b7#2y4K6x3@1x3o6M7%4y4K6R3H3y4W2)9J5k6h3S2@1L8h3H3`.
    //若SCR_EL3.RW == 1，则决定 EL2/EL1 是使用AArch64，否则AArch32
    mrs x9, scr_el3
    //打开 EL0/EL1 的非安全状态，EL0/EL1 无法访问安全内存 
    orr x9, x9, #SCR_EL3_NS
    //开启 HVC 指令
    //关于 HVC，看 e66K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4N6G2N6$3!0@1k6h3y4Z5i4K6u0W2L8X3g2@1i4K6u0r3j5i4u0E0N6U0S2S2i4K6g2X3j5i4u0U0K9q4)9J5c8U0t1K6z5q4)9J5k6h3S2@1L8h3H3`.
    orr x9, x9, #SCR_EL3_HCE
    //设置 SCR_EL3.RW == 1，EL2/EL1 是使用AArch64
    orr x9, x9, #SCR_EL3_RW 
    msr scr_el3, x9

    //ELR 寄存器 Exception Link Register，用于保存异常进入ELX的异常地址，在返回异常现场的时候，可以使用 ELR_ELX(x = 1/2/3) 来恢复PC值, 异常迁移到哪一个exception level就使用哪一个ELR
    //同样的，由于不会有异常把系统状态迁移到EL0,因此也就不存在ELR_EL0了。
    adr x9, .Ltarget
    //这里异常进入地址为 Ltarget
    msr elr_el3, x9

    //设定 spsr_el3
    mov x9, #((0b1111 << 6) | (0b0101)) /* EL1h runlevel */
    msr spsr_el3, x9

    //配置 EL1 并准备进入 EL1 *
    b   .confEL1
```

### 从 EL3 返回 EL1

```asm
    /* disable EL2 coprocessor traps */
    mov x9, #0x33ff
    msr cptr_el2, x9

    /* set EL1 to 64bit */
    //设置 EL1 的异常处理为 AARCH64 指令，同上
    mov x9, #HCR_EL2_RW
    msr hcr_el2, x9

    /* disable EL1 FPU traps */
    mov x9, #(0b11<<20)
    msr cpacr_el1, x9

    /* set up the EL1 bounce interrupt */

    //配置 EL1 栈指针
    mov x9, sp   
    msr sp_el1, x9

    isb
    //模拟异常返回，执行该指令会使得CPU返回EL1状态
    eret
```

## 初始化缓存

```asm
    //使缓存失效 *
    bl      arch_invalidate_cache_all

    /* enable caches so atomics and spinlocks work */
    //启用缓存，使原子操作和自旋锁生效
    mrs     tmp, sctlr_el1
    //打开指令缓存
    orr     tmp, tmp, #(1<<12) /* Enable icache */
    //打开数据缓存
    orr     tmp, tmp, #(1<<2)  /* Enable dcache/ucache */
    msr     sctlr_el1, tmp
```

## 修复重定向的 Kernel Base 地址

此工作由 prime cpu 完成，其他 cpu 开始进入自旋等待

```asm
    //加载 kernel_relocated_base 段地址
    //内核重定向的基地址，即内核开始的虚拟地址
    adr_global  tmp, kernel_relocated_base
    //负值给 kernel_vaddr
    ldr     kernel_vaddr, [tmp]

    // Load the base of the translation tables.
    //貌似 Zircon 中 1GB 物理内存由一个 translation_table 维护，所以这里 tt_trampoline 相当于一级页表？
    adr_global page_table0, tt_trampoline
    //虚拟地址内存页地址转换表
    adr_global page_table1, arm64_kernel_translation_table

    // Send secondary cpus over to a waiting spot for the primary to finish.
    //如果不是 prim CPU 内核，则跳转到 Lmmu_enable_secondary 后等待 prim 内核运行完下面代码
    cbnz    cpuid, .Lmmu_enable_secondary
    //下面的代码只有 prim CPU 内核执行

    // The fixup code appears right after the kernel image (at __data_end in
    // our view).  Note this code overlaps with the kernel's bss!  It
    // expects x0 to contain the actual runtime address of __code_start.
    //将内核代码开始的虚拟地址保存到 x0 中
    mov     x0, kernel_vaddr
    //跳转到 __data_end *
    //__data_end 指向 image.S - apply_fixups 方法
    bl      __data_end
```
