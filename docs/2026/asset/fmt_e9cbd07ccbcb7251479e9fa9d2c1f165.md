
`struct file {`  `union {`  `struct llist_node   fu_llist;`  `struct rcu_head     fu_rcuhead;`  `} f_u;`  `struct path     f_path;`  `struct inode        *f_inode;` `/* cached value */`  `const struct file_operations    *f_op;`    `/*`  `* Protects f_ep_links, f_flags.`  `* Must not be taken from IRQ context.`  `*/`  `spinlock_t      f_lock;`  `enum rw_hint        f_write_hint;`  `atomic_long_t       f_count;`  `unsigned int        f_flags;`  `fmode_t         f_mode;`  `struct mutex        f_pos_lock;`  `loff_t          f_pos;`  `struct fown_struct  f_owner;`  `const struct cred   *f_cred;`  `struct file_ra_state    f_ra;`    `u64         f_version;`  `#ifdef CONFIG_SECURITY`  `void            *f_security;`  `#endif`  `/* needed for tty driver, and maybe others */`  `void            *private_data;`    `#ifdef CONFIG_EPOLL`  `/* Used by fs/eventpoll.c to link all the hooks to this file */`  `struct list_head    f_ep_links;`  `struct list_head    f_tfile_llink;`  `#endif /* #ifdef CONFIG_EPOLL */`  `struct address_space    *f_mapping;`  `errseq_t        f_wb_err;`  `errseq_t        f_sb_err;` `/* for syncfs */`  `} __randomize_layout __attribute__((aligned(4)));` `/* lest something weird decides that 2 is OK */`

`struct file_handle {`  `__u32 handle_bytes;`  `int handle_type;`  `/* file identifier */`  `unsigned char f_handle[];`  `};`

### Slab 的两种内存缓存

众所周知，Linux 内核主要使用 `slab` 分配器来进行内存分配，`slab` 分配器中主要维护了两种内存缓存（即可以理解成两套作用不同的内存分配方式）：

- `dedicated cache`（专用缓存）：这里的内存是用于分配给内核中的常用对象，在该缓存中被分配的结构体将始终保持初始化状态，以便于提高分配速度
- `generic cache`（通用缓存）：通用缓存，大多数情况下其内存块的大小与 2 的幂次方对齐

这类 `cred` 和 `file` 结构体等 `credential` 对象都是在 `dedicated cache` 中分配，而大多数内存漏洞发生的地方都是在 `generic cache` 中

使用 `sudo cat /proc/slabinfo` 可以查看 `slab` 分配器的具体信息

- `generic cache`：在名称中带有 `kmalloc`
- `dedicated cache`：拥有特殊的名字

## 2. 漏洞与利用

### 漏洞点

- 将 `route4_filter` 对象从链表中删除和释放时的检查条件不一致
- 导致该对象被释放后仍存于链表中

### 大致步骤：

- 释放存在漏洞的非特权凭据
- 在释放的内存插槽中分配特权凭据
- 以特权用户身份操作

### 具体步骤：

本例是采用 `file` 对象完成利用，也可以采用 `cred` 对象

- 打开可写的文件 `/tmp/x`，就会分配可写的 `file` 对象，在通过写许可检查之后后，进行实际写操作之前暂停
- 利用漏洞释放该 `file` 对象
- 打开只读文件 `/etc/passwd`，就会分配新的 `file` 对象，占据旧的 `file` 对象，继续写入就能往只读文件写入内容（例如写入 `hacker:x:0:0:root:/:/bin/sh` 就能提权）

## 3. 需要解决的问题

1. 如何将内存破坏漏洞，转换为能够置换 `file object` 的原语
2. 如何延长文件的 **检查文件写权限 - 实际写入数据** 的竞争窗口
3. 如何创建高权限的 `file object`，来置换先前被释放的低权限 `file object`

## 4. 对应的解决措施

### 1. 置换内核凭证

一般就是如下三种方式：

- Out Of Bound Write：尝试越界写入下一个结构体的凭证字段，将其替换为高权限的凭证（例如：`request_key_auth->cred`）
- Use After Free：使用高权限的凭证来“占据”低权限的凭证
- Double Free：最终可以达到两个指针共同指向一个凭证的效果

### 2. 延长竞争窗口

由于 `credential` 的替换需要一些时间，因此如果能延长这个竞争窗口，那就能非常成功的进行漏洞利用，其中 `Userfaultfd` 和 `FUSE`，这两种机制都允许用户无限延长竞争窗口

- `Userfaultfd`：在多线程程序中，`userfaultfd` 允许一个线程管理其他线程所产生的 `Page Fault` 事件，当某个线程触发了 `Page Fault`，该线程将立即睡眠，而其他线程则可以通过 `userfaultfd` 来读取出这个 `Page Fault` 事件，并进行处理
- `FUSE`：一个用户层文件系统框架，允许用户实现自己的文件系统，用户可以在该框架中注册 `handler`，来指定应对文件操作请求（可以在实际操作文件之前，执行 `handler` 暂停内核执行，尽可能地延长窗口）
- `File Lock`：使用锁定暂停内核执行

### 3. 分配特权对象

由于 Dirty Cred 十分需要控制 `privilege credential` 对象的分配时机，控制该对象的分配成为了一个关键点

用户层中：

- 大量执行 `Set-UID` 程序（例如 `sudo`），或者频繁创建特权级守护进程（例如 `sshd`），从而创建 `privilege cred` 结构体
- 使用 ReadOnly 方式来打开诸如 `/etc/passwd` 等特权文件

内核层中，当内核创建新的 `kernel thread` 时，当前 `kernel thread` 将会被复制，这时其 `privileged cred` 结构体也会被拷贝一份

有两种方法可以做到这点：

1. 往 `kernel workqueue` 中填充大量任务，动态创建新的 `kernel thread` 来执行任务。
2. 调用 `usermode helper`（一种允许内核创建用户模式进程的机制），一种最常见的应用场所是加载内核模块至内核空间中。

## 5. 静态分析

- `route4_filter` 对象：（大小为“144”，属于 `kmalloc-192`）

```
struct route4_filter {
struct route4_filter __rcu  *next;
u32         id;
int         iif;
struct tcf_result   res;
struct tcf_exts     exts;
u32         handle;
struct route4_bucket    *bkt;
struct tcf_proto    *tp;
struct rcu_work     rwork;
};
```

- `tcf_exts` 对象的 `tc_action` 条目：（包含32个 `tc_action` 对象指针，属于 `kmalloc-256`）

```
struct tcf_exts {
#ifdef CONFIG_NET_CLS_ACT
__u32   type; /* for backward compat(TCA_OLD_COMPAT) */
int nr_actions;
struct tc_action **actions;
struct net *net;
#endif
/* Map to export classifier specific extension TLV types to the
* generic extensions API. Unsupported extensions must be set to 0.
*/
int action;
int police;
};
```

- 有漏洞的代码：

```
static int route4_change(struct net *net,
struct sk_buff *in_skb,
struct tcf_proto *tp, unsigned long base, u32 handle,
struct nlattr **tca,
void **arg,
bool ovr,
bool rtnl_held,
struct netlink_ext_ack *extack)
{
struct route4_head *head = rtnl_dereference(tp->root);
struct route4_filter __rcu **fp;
struct route4_filter *fold, *f1, *pfp, *f = NULL;
struct route4_bucket *b;
struct nlattr *opt = tca[TCA_OPTIONS];
struct nlattr *tb[TCA_ROUTE4_MAX + 1];
unsigned int h, th;
int err;
bool new = true;

if (opt == NULL)
return handle ? -EINVAL : 0;

err = nla_parse_nested_deprecated(tb, TCA_ROUTE4_MAX, opt,
route4_policy, NULL);
if (err < 0)
return err;

fold = *arg;
/* 现有的route4_filter对象 */
if (fold && handle && fold->handle != handle)
return -EINVAL;

err = -ENOBUFS;
f = kzalloc(sizeof(struct route4_filter), GFP_KERNEL);
/* 分配新的route4_filter对象 */
if (!f)
goto errout;

err = tcf_exts_init(&f->exts, net, TCA_ROUTE4_ACT, TCA_ROUTE4_POLICE);
/* 进行初始化,为route4_filter->exts.action分配256字节的空间 */
if (err < 0)
goto errout;

if (fold) {
/* 把旧的route4_filter对象中的数据填入新的route4_filter对象 */
f->id = fold->id;
f->iif = fold->iif;
f->res = fold->res;
f->handle = fold->handle;

f->tp = fold->tp;
f->bkt = fold->bkt;
new = false;
}

err = route4_set_parms(net, tp, base, f, handle, head, tb,
tca[TCA_RATE],
new, ovr, extack);
/* 初始化new filter */
if (err < 0)
goto errout;

/* 将new filter插入到list */
h = from_hash(f->handle >> 16);
fp = &f->bkt->ht[h];
for (pfp = rtnl_dereference(*fp);
(f1 = rtnl_dereference(*fp)) != NULL;
fp = &f1->next)
if (f->handle < f1->handle)
break;

tcf_block_netif_keep_dst(tp->chain->block);
rcu_assign_pointer(f->next, f1);
rcu_assign_pointer(*fp, f);

/* 若存在old filter,old handle不为"0",old new handle不同,则从list中移除 */
if (fold && fold->handle && f->handle != fold->handle) {
th = to_hash(fold->handle);
h = from_hash(fold->handle >> 16);
b = rtnl_dereference(head->table[th]);
if (b) {
fp = &b->ht[h];
/* ht存放的是route4_filter列表 */
for (pfp = rtnl_dereference(*fp); pfp;
fp = &pfp->next, pfp = rtnl_dereference(*fp)) {
if (pfp == fold) {
rcu_assign_pointer(*fp, fold->next);
/* 从链表中删除 */
break;
}
}
}
}

route4_reset_fastmap(head);
*arg = f;
if (fold) {
/* 若存在old filter,释放old filter */
tcf_unbind_filter(tp, &fold->res);
tcf_exts_get_net(&fold->exts);
tcf_queue_work(&fold->rwork, route4_delete_filter_work);
/* 启动内核任务,调用route4_delete_filter_work释放old filter */
}
return 0;

errout:
if (f)
tcf_exts_destroy(&f->exts);
kfree(f);
return err;
}
```

- 使用 `handle` 作为 ID 来区分不同的 `route4_filter`
- 如果存在某个 `handle` 之前已被初始化过（`fold` 变量非空），就会移除旧的 `filter`，添加新的 `filter`
