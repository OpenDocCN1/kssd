
```c
//一个文件pages被COW之后，MAP_PRIVATE vma可以在i_mmap与anon_vma之中
//MAP_SHARED vma只能在i_mmap树中
//匿名的MAP_PRIVATE，栈或者brk的vma只能在anon_vma链表中

struct list_head anon_vma_chain;
struct anon_vma *anon_vma;
const struct vm_operations_struct *vm_ops;//虚表指针
unsigned long vm_pgoff; //以page为单位的文件映射偏移量
struct file * vm_file; //映射了哪个文件
void * vm_private_data; /* was vm_pte (shared mem) */
......
} __randomize_layout;
```

### `find_vma()`

```c
//通过addr查找vma

struct vm_area_struct *find_vma(struct mm_struct *mm, unsigned long addr)
{
    struct rb_node *rb_node;
    struct vm_area_struct *vma;

    //首先从vma cache中寻找
    vma = vmacache_find(mm, addr);
    if (likely(vma))
        return vma;

    //红黑树跟节点
    rb_node = mm->mm_rb.rb_node;

    //遍历红黑树上的vma
    while (rb_node) {
        struct vm_area_struct *tmp;

        //当前节点
        tmp = rb_entry(rb_node, struct vm_area_struct, vm_rb);

        if (tmp->vm_end > addr) {
            vma = tmp;
            if (tmp->vm_start <= addr)
                break;
            rb_node = rb_node->rb_left;
        } else
            rb_node = rb_node->rb_right;
    }

    //如果成功找到，更新vma cache
    if (vma)
        vmacache_update(addr, vma);

    return vma;
}
```

### `vmacache_find()`

```c
struct vm_area_struct *vmacache_find(struct mm_struct *mm, unsigned long addr)
{
    // addr右移，定位idx
    int idx = VMACACHE_HASH(addr);
    int i;

    // 记录一次VMACACHE_FIND_CALLS事件
    count_vm_vmacache_event(VMACACHE_FIND_CALLS);

    //mm是否是当前进程的mm_struct
    if (!vmacache_valid(mm))
        return NULL;

    //从idx位置开始扫描查找vma
    for (i = 0; i < VMACACHE_SIZE; i++) {
        struct vm_area_struct *vma = current->vmacache.vmas[idx];

        if (vma) {
#ifdef CONFIG_DEBUG_VM_VMACACHE
            if (WARN_ON_ONCE(vma->vm_mm != mm))
                break;
#endif

            //判断是否查找成功，如果成功，记录事件VMACACHE_FIND_HITS
            if (vma->vm_start <= addr && vma->vm_end > addr) {
                count_vm_vmacache_event(VMACACHE_FIND_HITS);
                return vma;
            }
        }

        //如果已经扫描了[idx,VMACACHE_SIZE]，那么idx归0，从起点开始扫描
        if (++idx == VMACACHE_SIZE)
            idx = 0;
    }

    return NULL;
}
```

### `insert_vm_struct()`

```c
int insert_vm_struct(struct mm_struct *mm, struct vm_area_struct *vma)
{
    struct vm_area_struct *prev;
    struct rb_node **rb_link, *rb_parent;

    //定位vma的插入位置(双链表与红黑树)
    if (find_vma_links(mm, vma->vm_start, vma->vm_end,
                       &prev, &rb_link, &rb_parent))
        return -ENOMEM;

    if ((vma->vm_flags & VM_ACCOUNT) &&
        security_vm_enough_memory_mm(mm, vma_pages(vma)))
        return -ENOMEM;

    //如果是匿名的vma那么需要设置vm_pgoff
    if (vma_is_anonymous(vma)) {
        BUG_ON(vma->anon_vma);
        vma->vm_pgoff = vma->vm_start >> PAGE_SHIFT;
    }

    //vma插入红黑树与双链表中
    vma_link(mm, vma, prev, rb_link, rb_parent);

    return 0;
}
```

# Page Fault

什么时候会发生 page falut：
- page table 中找不到对应的 `PTE`
- 无效地址（通过地址的 `addr` 来找 `vma`，如果没找到说明地址无效，段错误，内核 `panic()` 掉）
- 有效地址但是没有载入主存
- 首次访问，发生调页。
- 如果当前 page 的 `present=0`，说明不在主存中被 swap out 了，需要从外存调入主存。
- COW 时访问语义冲突，比如 `PTE` 不可写，但是做了写操作，会触发 COW 机制，在 copy page 中 write
- 对应虚拟地址的 `PTE` 拒绝访问

```c
/*
 * Page fault error code bits:
 *
 * bit 0 == 0: no page found 1: protection fault
 * bit 1 == 0: read access 1: write access
 * bit 2 == 0: kernel-mode access 1: user-mode access
 * bit 3 == 1: use of reserved bit detected
 * bit 4 == 1: fault was an instruction fetch
 * bit 5 == 1: protection keys block access
 * bit 15 == 1: SGX MMU page-fault
 */
enum x86_pf_error_code {
    X86_PF_PROT = 1 << 0,
    X86_PF_WRITE = 1 << 1,
    X86_PF_USER = 1 << 2,
    X86_PF_RSVD = 1 << 3,
    X86_PF_INSTR = 1 << 4,
    X86_PF_PK = 1 << 5,
    X86_PF_SGX = 1 << 15,
};
```

### `handle_page_fault()`

在 5.13 的内核中 `__do_page_fault()` 已经被移除（x86），取而代之的是 `handle_page_fault()`。

```c
DEFINE_IDTENTRY_RAW_ERRORCODE(exc_page_fault)
->handle_page_fault(regs, error_code, address)
```

```c
static __always_inline void
trace_page_fault_entries(struct pt_regs *regs, unsigned long error_code,
                         unsigned long address)
{
    if (!trace_pagefault_enabled())
        return;

    /*
     * (((regs)->msr & MSR_PR) != 0)
     */
    //通过检查MSR的对应位查看是否是用户态（比如用户态的越界访问）
    //DEFINE_PAGE_FAULT_EVENT记录相应的事件
    /*
     * DEFINE_PAGE_FAULT_EVENT(page_fault_user)
     * DEFINE_PAGE_FAULT_EVENT(page_fault_kernel)
     */
    if (user_mode(regs))
        trace_page_fault_user(address, regs, error_code);
    else
        trace_page_fault_kernel(address, regs, error_code);
}

static __always_inline void
handle_page_fault(struct pt_regs *regs, unsigned long error_code,
                  unsigned long address)
{
    //记录事件
    trace_page_fault_entries(regs, error_code, address);

    //跟踪判断是否是kmmio的falut，如果是调用kmmio_handler
    if (unlikely(kmmio_fault(regs, address)))
        return;

    //判断是发生在内核态还是用户态的fault
    //fault_in_kernel_space判断address是否处于vsyscall区域（大于TASK_SIZE_MAX），但是这片区域不被当作kernel sapce对待。
    if (unlikely(fault_in_kernel_space(address))) {
        do_kern_addr_fault(regs, error_code, address);
    } else {
        do_user_addr_fault(regs, error_code, address);
        /*
         * User address page fault handling might have reenabled
         * interrupts. Fixing up all potential exit points of
         * do_user_addr_fault() and its leaf functions is just not
         * doable w/o creating an unholy mess or turning the code
         * upside down.
         */
        local_irq_disable();
    }
}
```

### `do_kern_addr_fault()`

```c
static void
do_kern_addr_fault(struct pt_regs *regs, unsigned long hw_error_code,
                   unsigned long address)
{
    //X86_PF_PK只针对用户态的page起作用
    WARN_ON_ONCE(hw_error_code & X86_PF_PK);

    //检测是否由于特有的bug产生
    if (is_f00f_bug(regs, hw_error_code, address))
        return;

    /*
     * 确保故障不是由以下引起的：
     * 1.不是在一个有保留位设置的PTE上的故障
     * 2.不是由user mode访问kernel mem访问引起的故障
     * 3.不是由违反页级保护引起的故障（反问一个不存在的页，且它的X86_PF_PROT==0）
     */
    //32位下检测是否是由于vmalloc_fault导致
    if (!(hw_error_code & (X86_PF_RSVD | X86_PF_USER | X86_PF_PROT))) {
        if (vmalloc_fault(address) >= 0)
            return;

        /* 是否是由于TLB陈旧导致的，没有及时更新 */
        if (spurious_kernel_fault(hw_error_code, address))
            return;

        /* 是否是kprobe hook了缺页错误 */
        if (kprobe_page_fault(regs, X86_TRAP_PF))
            return;

        //非法地址访问导致的错误
        bad_area_nosemaphore(regs, hw_error_code, address);
    }
}
```

### `vmalloc_fault()`

内核使用 `vmalloc()` 来分配在**虚拟内存中连续但在物理内存中不一定连续**的内存。

处理 `vmalloc` 或模块映射区的故障。之所以需要这样做，是因为在 `vmalloc` 映射代码更新 `PMD` 到它与系统中其他页表同步这一更新的时间点之间存在着一个竞争条件。在这个竞争窗口中，另一个线程/CPU 可以在同一个 `PMD` 上映射一个区域，发现它已经存在，并且还没有与系统的其他部分同步。因此，`vmzalloc` 可能会返回未被系统中的每个页表映射的区域，当这些区域被访问时，会引起未处理的页错误。

主要的修复过程就是把 `init` 进程的页表项（全局）复制到当前进程的页表项中，这样就可以实现所有进程的内核内存地址空间同步。

```c
static noinline int vmalloc_fault(unsigned long address)
{
    unsigned long pgd_paddr;
    pmd_t *pmd_k;
    pte_t *pte_k;

    /* 确保我们在系统的vmalloc区域 */
    if (!(address >= VMALLOC_START && address < VMALLOC_END))
        return -1;

    //首先拿到进程内核的PGD的物理地址（进程内核页表）以及全局内核页表，然后复制更新进程内核页表，完成同步
    pgd_paddr = read_cr3_pa();
    pmd_k = vmalloc_sync_one(__va(pgd_paddr), address);
    if (!pmd_k)
        return -1;

    //是否是大页（PSE）
    if (pmd_large(*pmd_k))
        return 0;

    //在pmd中找到pte的对应指针
    pte_k = pte_offset_kernel(pmd_k, address);

    //检查PTE是否存在于内存中，如果不存在则缺页
    if (!pte_present(*pte_k))
        return -1;

    return 0;
}
```

### `spurious_kernel_fault()`

本函数用于处理由于 `TLB` entry 没有及时更新导致的虚假错误。
可能发生的原因：`TLB` entry 对应的 permission 比 page table entry 的少。
可能导致的原因：
1. 向 ring0 做 write 操作。
2. 对一块 NX 区域 fetch。

```c
static noinline int
spurious_kernel_fault(unsigned long error_code, unsigned long address)
{
    pgd_t *pgd;
    p4d_t *p4d;
    pud_t *pud;
    pmd_t *pmd;
    pte_t *pte;
    int ret;

    //只可能由如下标志导致X86_PF_WRITE / X86_PF_INSTR
    if (error_code != (X86_PF_WRITE | X86_PF_PROT) &&
        error_code != (X86_PF_INSTR | X86_PF_PROT))
        return 0;

    //定位内核页表
    pgd = init_mm.pgd + pgd_index(address);

    //判断pgd是否在内存中
    if (!pgd_present(*pgd))
        return 0;

    //通过偏移获得p4d entry
    p4d = p4d_offset(pgd, address);

    if (!p4d_present(*p4d))
        return 0;

    //是否是大页？
    if (p4d_large(*p4d))
        //如果p4d开启了PSE，那么调用spurious_kernel_fault_check，此时p4d作为pte被检测
        return spurious_kernel_fault_check(error_code, (pte_t *) p4d);

    pud = pud_offset(p4d, address);

    if (!pud_present(*pud))
        return 0;

    if (pud_large(*pud))
        return spurious_kernel_fault_check(error_code, (pte_t *) pud);

    pmd = pmd_offset(pud, address);

    if (!pmd_present(*pmd))
        return 0;

    if (pmd_large(*pmd))
```
