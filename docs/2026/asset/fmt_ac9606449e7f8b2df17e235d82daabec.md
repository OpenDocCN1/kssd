
当 CPU 运行的代码从用户进程 A 切换到用户进程 B 时，由于 A 和 B 的进程地址空间一般是不一样的，所以当前的`TLB`对于 B 进程完全没有意义，而且可能导致 B 错误的访问到 A 的进程空间。

所以，我们就需要将用户空间的`TLB`完全清空，而这样一来，当 CPU 再次切换到进程 A 时，`MMU`就需要重新从内存中查页表，内存压力将越来越大。。。

### `ASID`

幸好 ARM64 想到了这一点，还记得`VA`的高 16 位么？其中就承载了一个叫`ASID`的信息，每条`VA`都会携带上对应进程的`ASID`，这样的话`MMU`就有能力区别`TLB`中的`VA`是哪个进程的。

这样的话，进程切换将完全没有必要 Flush `TLB`，而是让`Cache`自然淘汰。

当然，当进程死亡的时候，还是可以主动 Flush `TLB`的，而且因为`ASID`，CPU 也有能力单独抹掉某个进程空间的`TLB`。

```
ttbr = ((uint64_t)aspace->asid_ << 48) | aspace->tt_phys_;

// 重写页表地址

ARM64_WRITE_SYSREG(ttbr0_el1, ttbr);
```

### Zircon 切换实现

```
// 切换内存地址空间，即切换页表

void ArmArchVmAspace::ContextSwitch(ArmArchVmAspace* old_aspace, ArmArchVmAspace* aspace) {

    if (TRACE_CONTEXT_SWITCH)

        TRACEF("aspace %p\n", aspace);

    uint64_t tcr;

    uint64_t ttbr;

    if (aspace) {

        aspace->canary_.Assert();

        DEBUG_ASSERT((aspace->flags_ & (ARCH_ASPACE_FLAG_KERNEL | ARCH_ASPACE_FLAG_GUEST)) == 0);

        tcr = MMU_TCR_FLAGS_USER;

        ttbr = ((uint64_t)aspace->asid_ << 48) | aspace->tt_phys_;

        // 重写页表地址

        ARM64_WRITE_SYSREG(ttbr0_el1, ttbr);

        if (TRACE_CONTEXT_SWITCH)

            TRACEF("ttbr %#" PRIx64 ", tcr %#" PRIx64 "\n", ttbr, tcr);

    } else {

        tcr = MMU_TCR_FLAGS_KERNEL;

        if (TRACE_CONTEXT_SWITCH)

            TRACEF("tcr %#" PRIx64 "\n", tcr);

    }

    ARM64_WRITE_SYSREG(tcr_el1, tcr);

}
```

## 销毁虚拟内存空间

- 通知物理内存管理器`PMM`释放对应的物理内存空间

- Flush 对应的`TLB`

  - 需要 Flush 所有 CPU 中对应的`TLB`

  - ARM64 可以用`tlbi`指令实现

  - X86 则需要使用核间中断通知每个 CPU 自行操作

```
zx_status_t ArmArchVmAspace::Destroy() {

    canary_.Assert();

    LTRACEF("aspace %p\n", this);

    fbl::AutoLock a(&lock_);

    DEBUG_ASSERT((flags_ & ARCH_ASPACE_FLAG_KERNEL) == 0);

    // XXX make sure it's not mapped

    vm_page_t* page = paddr_to_vm_page(tt_phys_);

    DEBUG_ASSERT(page);

    pmm_free_page(page);

    // 需要 flush 所有 CPU 的缓存中该 ASID 空间残余的 TLB

    // ARM64 可以使用 tlbi 指令实现这一操作

    // X86 是不支持的，需要使用核间中断通知每个 CPU 自行操作

    if (flags_ & ARCH_ASPACE_FLAG_GUEST) {

        paddr_t vttbr = arm64_vttbr(asid_, tt_phys_);

        __UNUSED zx_status_t status = arm64_el2_tlbi_vmid(vttbr);

        DEBUG_ASSERT(status == ZX_OK);

    } else {

        ARM64_TLBI(ASIDE1IS, asid_);

        asid.Free(asid_);

        asid_ = MMU_ARM64_UNUSED_ASID;

    }

    return ZX_OK;

}
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
