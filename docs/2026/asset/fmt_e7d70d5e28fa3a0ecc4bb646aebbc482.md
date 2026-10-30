
# [原创] CVE-2022-2588 Dirty Cred 漏洞分析与复现

# CVE-2022-2588 Dirty Cred

## 一、漏洞信息

### 1. 漏洞简述

- 漏洞名称：Dirty Cred
- 漏洞编号：CVE-2022-2588
- 漏洞类型：Double Free
- 漏洞影响：Linux 本地提权
- CVSS 评分：7.8
- 基础权限：需要

### 2. 组件概述

漏洞函数是`route4_change()`，用于初始化和替换`route4_filter`对象。使用`handle`作为 id 来区分不同的`route4_filter`。如果存在某个`handle`之前已被初始化过（`fold`变量非空），就会移除旧的`filter`，添加新的`filter`；否则直接添加新的`filter`。

### 3. 漏洞利用

由于将`route4_filter`对象从链表中删除和释放时的检查条件不一致，导致该对象被释放后仍存于链表中，后面可以触发 Double Free。本地攻击者利用该漏洞会导致系统崩溃，可能会造成本地特权升级问题。

### 4. 漏洞影响

影响 Linux Kernel 版本：

- Linux Kernel 版本 >= 2.6.12
- Linux Kernel 版本 <= 5.19.1

### 5. 解决方案

<mark class="encrypted">ed4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Q4x3X3g2C8k6i4u0F1k6h3I4Q4x3X3g2G2M7X3N6Q4x3V1k6H3N6h3u0Q4x3V1k6K6j5$3#2Q4x3V1k6D9K9h3&6#2P5q4)9J5c8X3E0W2M7X3&6W2L8q4)9J5c8X3N6A6N6q4)9J5c8Y4c8G2M7Y4k6S2L8r3c8K6i4K6u0r3L8r3W2F1N6i4S2Q4x3X3g2Y4K9i4c8Q4x3V1k6U0L8$3#2E0K9i4c8Q4x3V1k6Q4x3@1k6A6k6q4)9K6c8o6W2S2k6o6x3$3x3K6l9&6k6e0t1%4x3e0W2S2z5o6R3@1k6U0V1@1y4U0j5%4z5r3f1H3x3U0V1$3j5X3f1I4x3r3j5`.</mark>

## 二、环境搭建

### 安装 Kernel

```bash
wget <https://mirrors.tuna.tsinghua.edu.cn/kernel/v5.x/linux-5.19.1.tar.xz>
tar -xvf linux-5.19.1.tar.xz
make menuconfig
make x86_64_defconfig
make bzImage -j32
```

- 怕麻烦的可以在 bsauce 大佬的仓库去取：<mark class="encrypted">d75K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1M7$3q4#2j5$3g2Q4x3V1k6C8k6i4u0F1k6h3I4Q4x3X3c8W2P5s2m8D9L8$3W2@1i4K6u0V1k6X3q4U0N6r3!0J5P5g2)9J5c8Y4c8J5k6h3g2Q4x3V1k6E0j5h3W2F1i4K6u0r3b7#2k6q4i4K6u0V1x3U0l9J5x3W2)9J5k6o6t1#2z5o6R3`.</mark>

### 编译选项

- **`CONFIG_BINFMT_MISC=y`**（否则启动 VM 时会报错）
- **`CONFIG_USER_NS=y`**（触发漏洞需要 User Namespace）
- **`CONFIG_NET_CLS_ROUTE4=y`**（漏洞函数所在的模块）
- **`CONFIG_DUMMY=y`**、**`CONFIG_NET_SCH_QFQ=y`**（breezeO_o 提供的两个编译选项，触发 POC 需要用到）
- **`CONFIG_NET_CLS_ACT=y`** / **`CONFIG_NET_CLS_BASIC=y`**（默认已开启）
- **`CONFIG_NET_SCH_SFQ=y`**（exploit 中触发漏洞需用到 SFQ 随机公平队列）
- **`CONFIG_NET_EMATCH_META=y`**（exploit 中堆喷对象时需要用到）

## 三、漏洞分析

### 1. 前置知识

#### 内核凭证 Credential

Kernel 凭证是 kernel 文档中定义的 kernel 中携带特权信息的特征，表示权限和对应的能力，主要分为：

- task 凭证（`struct cred`）：其中存放了一个 task 的权限信息，例如 GID、UID 等等。如果能任意修改一个低权限进程的 cred 结构体，那么我们就可以将该进程提权至高权限（例如 root）。

```c
struct cred {
atomic_t usage;
#ifdef CONFIG_DEBUG_CREDENTIALS
atomic_t subscribers; /* number of processes subscribed */
void *put_addr;
unsigned magic;
#define CRED_MAGIC 0x43736564
#define CRED_MAGIC_DEAD 0x44656144
#endif
kuid_t uid; /* real UID of the task */
kgid_t gid; /* real GID of the task */
kuid_t suid; /* saved UID of the task */
kgid_t sgid; /* saved GID of the task */
kuid_t euid; /* effective UID of the task */
kgid_t egid; /* effective GID of the task */
kuid_t fsuid; /* UID for VFS ops */
kgid_t fsgid; /* GID for VFS ops */
unsigned securebits; /* SUID-less security management */
kernel_cap_t cap_inheritable; /* caps our children can inherit */
kernel_cap_t cap_permitted; /* caps we're permitted */
kernel_cap_t cap_effective; /* caps we can actually use */
kernel_cap_t cap_bset; /* capability bounding set */
kernel_cap_t cap_ambient; /* Ambient capability set */
#ifdef CONFIG_KEYS
unsigned char jit_keyring; /* default keyring to attach requested
* keys to */
struct key *session_keyring; /* keyring inherited over fork */
struct key *process_keyring; /* keyring private to this process */
struct key *thread_keyring; /* keyring private to this thread */
struct key *request_key_auth; /* assumed request_key authority */
#endif
#ifdef CONFIG_SECURITY
void *security; /* subjective LSM security */
#endif
struct user_struct *user; /* real user ID subscription */
struct user_namespace *user_ns; /* user_ns the caps and keyrings are relative to. */
struct group_info *group_info; /* supplementary groups for euid/fsgid */
/* RCU deletion */
union {
int non_rcu; /* Can we skip RCU deletion? */
struct rcu_head rcu; /* RCU deletion hook */
};
} __randomize_layout;
```

- open file 凭证（`struct file`）：存放一个文件的部分权限信息，例如 read & write 权限等。如果一个低权限用户可以任意修改高权限文件（例如`/etc/passwd`），那么同样也能造成提权的目的。

```c
struct file {
union {
struct llist_node fu_llist;
struct rcu_head fu_rcuhead;
} f_u;
struct path f_path;
struct inode *f_inode; /* cached value */
const struct file_operations *f_op;
/*
* Protects f_ep_links, f_flags.
* Must not be taken from IRQ context.
*/
spinlock_t f_lock;
enum rw_hint f_write_hint;
atomic_long_t f_count;
unsigned int f_flags;
fmode_t f_mode;
struct mutex f_pos_lock;
loff_t f_pos;
struct fown_struct f_owner;
const struct cred *f_cred;
struct file_ra_state f_ra;
u64 f_version;
#ifdef CONFIG_SECURITY
void *f_security;
#endif
/* needed for tty driver, and maybe others */
void *private_data;
#ifdef CONFIG_EPOLL
/* Used by fs/eventpoll.c to link all the hooks to this file */
struct list_head f_ep_links;
struct list_head f_tfile_llink;
#endif /* #ifdef CONFIG_EPOLL */
struct address_space *f_mapping;
errseq_t f_wb_err;
errseq_t f_sb_err; /* for syncfs */
} __randomize_layout
__attribute__((aligned(4))); /* lest something weird decides that 2 is OK */
struct file_handle {
__u32 handle_bytes;
int handle_type;
/* file identifier */
unsigned char f_handle[];
};
```

#### Slab 的两种内存缓存

众所周知，Linux 内核主要使用 slab 分配器来进行内存分配，slab 分配器中主要维护了两种内存缓存（即可以理解成两套作用不同的内存分配方式）：

- `dedicated cache（专用缓存）`：这里的内存是用于分配给内核中的常用对象，在该缓存中被分配的结构体将始终保持初始化状态，以便于提高分配速度。
- `generic cache（通用缓存）`：通用缓存，大多数情况下其内存块的大小与 2 的幂次方对齐。

这类`cred`和`file`结构体等`credential`对象都是在`dedicated cache`中分配，而大多数内存漏洞发生的地方都是在`generic cache`中。

使用`sudo cat /proc/slabinfo`可以查看 slab 分配器的具体信息：

- `generic cache`：在名称中带有`kmalloc`
- `dedicated cache`：拥有特殊的名字

### 2. 漏洞与利用

#### 漏洞点

- 将`route4_filter`对象从链表中删除和释放时的检查条件不一致。
- 导致该对象被释放后仍存于链表中。

#### 大致步骤

- 释放存在漏洞的非特权凭据。
- 在释放的内存插槽中分配特权凭据。
- 以特权用户身份操作。

#### 具体步骤

本例是采用`file`对象完成利用，也可以采用`cred`对象。

- 打开可写的文件`/tmp/x`，就会分配可写的`file`对象，在通过写许可检查之后、进行实际写操作之前暂停。
- 利用漏洞释放该`file`对象。
- 打开只读文件`/etc/passwd`，就会分配新的`file`对象，占据旧的`file`对象，继续写入就能往只读文件写入内容，例如写入`hacker:x:0:0:root:/:/bin/sh`就能提权。

### 3. 需要解决的问题

1. 如何将内存破坏漏洞转换为能够置换`file object`的原语。
2. 如何延长文件的**检查文件写权限 - 实际写入数据**的竞争窗口。
3. 如何创建高权限的`file object`，来置换先前被释放的低权限`file object`。

### 4. 对应的解决措施

#### 1. 置换内核凭证

一般就是如下三种方式：

- Out Of Bound Write：尝试越界写入下一个结构体的凭证字段，将其替换为高权限的凭证（例如：`request_key_auth->cred`）。
- Use After Free：使用高权限的凭证来“占据”低权限的凭证。
- Double Free：最终可以达到两个指针共同指向一个凭证的效果。

#### 2. 延长竞争窗口

由于`credential`的替换需要一些时间，因此如果能延长这个竞争窗口，那就能非常成功地进行漏洞利用，其中`Userfaultfd`和`FUSE`这两种机制都**允许用户无限延长竞争窗口**。

- Userfaultfd：在多线程程序中，`userfaultfd`允许一个线程管理其他线程所产生的`Page Fault`事件。当某个线程触发了`Page Fault`，该线程将立即睡眠，而其他线程则可以通过`userfaultfd`来读取出这个`Page Fault`事件，并进行处理。
- FUSE：一个用户层文件系统框架，允许用户实现自己的文件系统，用户可以在该框架中注册`handler`，来指定应对文件操作请求（可以在实际操作文件之前，执行`handler`暂停内核执行，尽可能地延长窗口）。
- File Lock：使用锁定暂停内核执行。

#### 3. 分配特权对象

由于 Dirty Cred 十分需要控制`privilege credential`对象的分配时机，控制该对象的分配成为了一个关键点。

用户层中：

- 大量执行`Set-UID`程序（例如`sudo`），或者频繁创建特权级守护进程（例如`sshd`），从而创建`privilege cred`结构体。
- 使用 ReadOnly 方式来打开诸如`/etc/passwd`等特权文件。

内核层中，当内核创建新的`kernel thread`时，当前`kernel thread`将会被复制，这时其`privileged cred`结构体也会被拷贝一份。

有两种方法可以做到这点：

1. 往`kernel workqueue`中填充大量任务，动态创建新的`kernel thread`来执行任务。
2. 调用`usermode helper`（一种允许内核创建用户模式进程的机制），一种最常见的应用场所是加载内核模块至内核空间中。

### 5. 静态分析

- `route4_filter`对象：（大小为“144”，属于`kmalloc-192`）

```c
struct route4_filter {
struct route4_filter __rcu *next;
u32 id;
int iif;
struct tcf_result res;
struct tcf_exts exts;
u32 handle;
struct route4_bucket *bkt;
struct tcf_proto *tp;
struct rcu_work rwork;
};
```

- `tcf_exts`对象的`tc_action`条目：（包含 32 个`tc_action`对象指针，属于`kmalloc-256`）
