
```
long __get_user_pages(struct task_struct *tsk, struct mm_struct *mm,
        unsigned long start, unsigned long nr_pages,
        unsigned int gup_flags, struct page **pages,
        struct vm_area_struct **vmas, int *nonblocking){
    /* ... snip ... */

    do {
        /* ... snip ... */retry:
        cond_resched(); /* please rescheule me!!! */
        page = follow_page_mask(vma, start, foll_flags, &page_mask);
        if (!page) {
            int ret;
            ret = faultin_page(tsk, vma, start, &foll_flags,
                    nonblocking);
            switch (ret) {
            case 0:
                goto retry;
            case -EFAULT:
            case -ENOMEM:
            case -EHWPOISON:
                return i ? i : ret;
            case -EBUSY:
                return i;
            case -ENOENT:
                goto next_page;
            }
            BUG();
        } 
        if (pages) {
            pages[i] = page;
            flush_anon_page(vma, page, start);
            flush_dcache_page(page);
            page_mask = 0;
        }
        /* ... snip ... */
    }
    /* ... snip ... */}
```

代码首先定位远程进程中起始地址为`start`的内存页，而且`foll_flags`决定着当前的内存访问语义。如果该页面不可用`page == NULL`，即该页面不在内存中，需要进行页错误处理。之后`faultin_page`被调用，内部模拟一个内存空间的访问和触发页错误处理，以期待handler换进丢失的页。

通常有几个原因导致`follow_page_mask`返回空，如下是一个不完全的列表：
- 该地址没有关联的内存映射，比如访问空指针。
- 该内存映射已经被创建了，但是由于`demand-paging`，内容尚未被加载进来。
- 页已经被换出到原始的文件或者交换文件中。
- 访问语义`foll_flags`与页的权限配置不一致（比如，写一个只读的映射）。

最后一个原因就是我们调用`write(2)`写`/proc/self/mem`后发生的情况。通常的做法是页错误handler成功的处理错误，返回一个有效的页，之后再次重新访问。

注意那个`retry`标志。此刻还不清楚作用，之后我们会提到，它是另一个导致此次exploit的“帮凶”。

心里明白这点后，继续看`fault_page`的实现：

```
static int faultin_page(struct task_struct *tsk, struct vm_area_struct *vma,
        unsigned long address, unsigned int *flags, int *nonblocking){
    struct mm_struct *mm = vma->vm_mm;
    unsigned int fault_flags = 0;
    int ret;

    /* mlock all present pages, but do not fault in new pages */
    if ((*flags & (FOLL_POPULATE | FOLL_MLOCK)) == FOLL_MLOCK)
        return -ENOENT;
    /* For mm_populate(), just skip the stack guard page. */
    if ((*flags & FOLL_POPULATE) &&
            (stack_guard_page_start(vma, address) ||
             stack_guard_page_end(vma, address + PAGE_SIZE)))
        return -ENOENT;
    if (*flags & FOLL_WRITE)
        fault_flags |= FAULT_FLAG_WRITE;
    if (*flags & FOLL_REMOTE)
        fault_flags |= FAULT_FLAG_REMOTE;
    if (nonblocking)
        fault_flags |= FAULT_FLAG_ALLOW_RETRY;
    if (*flags & FOLL_NOWAIT)
        fault_flags |= FAULT_FLAG_ALLOW_RETRY | FAULT_FLAG_RETRY_NOWAIT;
    if (*flags & FOLL_TRIED) {
        VM_WARN_ON_ONCE(fault_flags & FAULT_FLAG_ALLOW_RETRY);
        fault_flags |= FAULT_FLAG_TRIED;
    }

    ret = handle_mm_fault(mm, vma, address, fault_flags);
    if (ret & VM_FAULT_ERROR) {
        if (ret & VM_FAULT_OOM)
            return -ENOMEM;
        if (ret & (VM_FAULT_HWPOISON | VM_FAULT_HWPOISON_LARGE))
            return *flags & FOLL_HWPOISON ? -EHWPOISON : -EFAULT;
        if (ret & (VM_FAULT_SIGBUS | VM_FAULT_SIGSEGV))
            return -EFAULT;
        BUG();
    }

    if (tsk) {
        if (ret & VM_FAULT_MAJOR)
            tsk->maj_flt++;
        else
            tsk->min_flt++;
    }

    if (ret & VM_FAULT_RETRY) {
        if (nonblocking)
            *nonblocking = 0;
        return -EBUSY;
    }

    /*
     * The VM_FAULT_WRITE bit tells us that do_wp_page has broken COW when
     * necessary, even if maybe_mkwrite decided not to set pte_write. We
     * can thus safely do subsequent page lookups as if they were reads.
     * But only do so when looping for pte_write is futile: in some cases
     * userspace may also be wanting to write to the gotten user page,
     * which a read fault here might prevent (a readonly page might get
     * reCOWed by userspace write).
     */
    if ((ret & VM_FAULT_WRITE) && !(vma->vm_flags & VM_WRITE))
        *flags &= ~FOLL_WRITE;
    return 0;}
```

函数的前半段解释`foll_flags`为对应的`fault_flags`，用以传递给`handle_mm_fault`，而该函数负责解析页错误，这样`__get_user_pages`可以继续执行。

在当前情况中，因为我们要修改的原始的内存映射为只读的，故`handle_mm_fault`将创建一个新的只读的`COW page` `do_wp_page`给我们想要写的地址，同时使它为变为私有的和dirty，因此称为`Dirty COW`。

真正创建COWed page的是嵌入在handle深处的`do_wp_page`，粗略的执行流程如下：

```
faultin_page
  handle_mm_fault
    __handle_mm_fault
      handle_pte_fault
        FAULT_FLAG_WRITE && !pte_write
      do_wp_page        PageAnon() <- this is CoWed page already
        reuse_swap_page <- page is exclusively ours
        wp_page_reuse
          maybe_mkwrite <- dirty but RO again
          ret = VM_FAULT_WRITE
```

现在我们将视线转回`fault_page`的结束位置，在它返回之前，它做了如下一件事，使得该利用变为可能。

```
    /*
     * The VM_FAULT_WRITE bit tells us that do_wp_page has broken COW when
     * necessary, even if maybe_mkwrite decided not to set pte_write. We
     * can thus safely do subsequent page lookups as if they were reads.
     * But only do so when looping for pte_write is futile: in some cases
     * userspace may also be wanting to write to the gotten user page,
     * which a read fault here might prevent (a readonly page might get
     * reCOWed by userspace write).
     */
    if ((ret & VM_FAULT_WRITE) && !(vma->vm_flags & VM_WRITE))
        *flags &= ~FOLL_WRITE;
    return 0;
```

在它检测到一个写时复制发生后，`ret & VM_FAULT_WRITE == true`，它决定移除`FOLL_WRITE` flag。为什么要这样做？

还记得那个`retry` lable么？如果不移除`FOLL_WRITE`，则下一次retry，将执行同样的流程。新申请的COWed 页和原来的也有同样的访问权限。同样的访问权限，同样的`foll_flags`，同样的retry,会导致死循环。

为了打破这种无限的retry循环，一个聪明的想法是移除write flag。这样当下一次调用`follow_page_mask`时，将返回一个有效的页，指向起始地址。因为当前`FOLL_WRITE`不在了，`foll_flags`仅仅是一个普通的读权限，这对于新申请的COWed 只读页时允许的。

### 谎言

此处到了问题的关键。通过从`foll_flags`移除write标志， `follow_page_mask`在下一次retry时，该访问将被视为只读的，尽管我们的目标是要写。现在，假如我们在同一时刻，COWed page被抛弃了通过另一个线程调用`madvice(MADV_DONTNEED)`会怎样？当然什么灾难也不会发生。`follow_page_mask`将仍然失败由于定位COWed page时发生缺页。但是下一次在`faultin_page`发生的将非常有趣。因为这次`foll_flags`并不包含`FOLL_WRITE`，故不再创建一个dirty COW 页，`handle_mm_fault`将简单地将该页从page cache中移除！为什么这么直接，因为万能的kernel只是在处理请求read 权限（切记，FOLL_WRITE已经被移除了），为什么要费尽创建页的另一个拷贝，如果kernel已经约定不再修改它。

`faultin_page`返回不久之后，`__get_user_pages`将做另一次retry，来获取它请求了多次的页。多亏`follow_page_mask`在这次尝试中，最终返回给我们页。而且，它不再是普通的页，它是直接绑定特权文件的原始页。

Kernel帮助我们获得了打开特权城堡的钥匙。有这个页在手，通用的commonner non-root程序现在有能力修改root file了。

所有一切都是因为kernel在此撒谎了。在被告知dirty COW页已经ready之后的retry中，它只告诉了`follow_page_mask`和`handle_mm_fault`，只需要只读权限。这两个函数高兴的接受，最终返回一个当前任务最优的一个页。在这种情况下，它返回了一个如果我们修改它，它就将修改内容写回到原始特权文件的页。

在最终获得页之后，`__get_user_pages`可以最终跳过`faultin_page`调用，返回页给`__access_remote_vm`来进行更多的处理。

### 灾难

该页怎样被修改？如下是`access_remote_vm`的相关代码

```
    maddr = kmap(page);
    if (write) {
        copy_to_user_page(vma, page, addr,
                  maddr + offset, buf, bytes);
        set_page_dirty_lock(page);
    } else {
        /* ... snip ... */
    }
    kunmap(page);
```

上个代码片段中的`page`将直接映射我们之前提到的页。内核首先映射`kmap`这些页到内核地址空间中，之后调用`copy_to_user_page`快速地将`buf`中的用户数据写入到提到的页中，修改原始页的内容。

过一段时间以后，内核守护者线程 `kflushd`, `bdflush`, `kupdated`, `pdflush`线程等会将被修改的页将被会写回到位于磁盘的特权文件中，这样就完成了整个攻击。

你可以会问，是听起来不错，但是发生的概率是多大？利用的话，有多少成功率？所有这些是在内核空间中吧？内核拥有权力来决定什么时候一个线程运行吧？

不幸的是，你可能已经猜到了。概率很大，Dirty COW甚至在一个单核的机器上，利用都相当稳定，归功于`__get_user_ages`会显示请求任务机制来切换线程，通过调用`cond_resched`。

以下是两个线程如何相互竞争的：

![](https://leanote.com/api/file/getImage?fileId=5951d341ab644153e3000afc)

机敏的读者可能已经注意到了，如果我们直接访问一个基于文件的只读映射，一个段错误将会产生。但是，为什么我们使用`wirte`写`proc/self/mem`确返回了一个dirty COWed的页呢？

这个原因取决于当在一个进程内发生内存访问和当采用out-of-band `ptrace, /proc/{pid}/mem` 内存访问时，内核如何处理页错误的情况。这两种情况最终都会调用`handle_mm_fault`来处理页错误。但是后者使用`faultin_page`来模拟页错误，页错误直接导致触发MMU，将直接进入中断处理器，之后所有的路径都进入到平台独立的内核处理函数`__do_page_fault`中。而在直接写只读内存区域时，hanler将检测到访问违例在函数`access_error`中，同时在`handle_mm_fault`处理之前，直接触发信号`SIGEGV`在函数`bad_aea_access_error`中：
