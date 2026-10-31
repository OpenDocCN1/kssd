
```c++
int cap_capable(const struct cred *cred, struct user_namespace *targ_ns,
                int cap, unsigned int opts)
{
    struct user_namespace *ns = targ_ns;
    /* See if cred has the capability in the target user namespace
     * by examining the target user namespace and all of the target
     * user namespace's parents.
     */
    for (;;) {
        /* Do we have the necessary capabilities? */
        if (ns == cred->user_ns)
            return cap_raised(cred->cap_effective, cap) ? 0 : -EPERM;
        /*
         * If we're already at a lower level than we're looking for,
         * we're done searching.
         */
        if (ns->level <= cred->user_ns->level)
            return -EPERM;
        /*
         * The owner of the user namespace in the parent of the
         * user namespace has all caps.
         */
        if ((ns->parent == cred->user_ns) && uid_eq(ns->owner, cred->euid))
            return 0;
        /*
         * If you have a capability in a parent user ns, then you have
         * it over all children user namespaces as well.
         */
        ns = ns->parent;
    }
    /* We never get here */
}
```

#### 脚本

```c
void unshare_setup(void)
{
    char edit[0x100];
    int tmp_fd;

    unshare(CLONE_NEWNS | CLONE_NEWUSER | CLONE_NEWNET);

    tmp_fd = open("/proc/self/setgroups", O_WRONLY);
    write(tmp_fd, "deny", strlen("deny"));
    close(tmp_fd);

    tmp_fd = open("/proc/self/uid_map", O_WRONLY);
    snprintf(edit, sizeof(edit), "0 %d 1", getuid());
    write(tmp_fd, edit, strlen(edit));
    close(tmp_fd);

    tmp_fd = open("/proc/self/gid_map", O_WRONLY);
    snprintf(edit, sizeof(edit), "0 %d 1", getgid());
    write(tmp_fd, edit, strlen(edit));
    close(tmp_fd);
}
```

> USMA : USER SPEACE MAPPING ATTACK (用户空间映射攻击)
>
> 本文关于 Linux 源码的版本均为:6.12.32

## 参考文章:

- `c7dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9N6r3k6S2x3h3I4Q4x3X3g2@1L8%4m8Q4x3V1j5J5x3o6t1#2i4K6u0r3x3o6u0Q4x3V1j5J5y4g2)9J5c8Y4y4&6M7%4c8W2L8g2)9J5c8X3E0W2M7X3&6W2L8q4)9J5c8V1I4A6L8Y4g2^5i4K6g2X3K9$3g2J5L8X3g2D9z5q4)9#2k6W2g2e0e0f1q4Q4x3V1j5`.

## Pakcet Socket

> USMA 利用的是 `socket` 的 `pgv` 数组,所以我们在这里先深入了解一下 PacketSocket 的数据结构,在这里我们还是主要关注 `pgv` 的创建和映射,关于Socket其他板块的详细解析可以看我其他的文章。

### 数据结构

#### `packet_socket`

> 我们这里主要关注 `rx_ring`/`tx_ring`,这是我们后续利用的关键数据结构。
>
> 所以相当于除去 `struct socket` 以外其他均为 `packet_socket` 独有的结构.

- `sk`: 继承的通用 `socket *`
- `fanout`: `fanout` 组
- `rx_ring`/`tx_ring`: 接收/发送环形缓冲区（`mmap`）
- `prot_hook`: 协议钩子，用于接收数据包
- `ifindex`: 绑定的网络设备索引
- `tp_version`: TPACKET 版本（V1/V2/V3）

```c
struct packet_sock {
    /* struct sock has to be the first member of packet_sock */
    struct sock             sk;
    struct packet_fanout    *fanout;
    union tpacket_stats_u   stats;
    struct packet_ring_buffer       rx_ring;
    struct packet_ring_buffer       tx_ring;
    int                     copy_thresh;
    spinlock_t              bind_lock;
    struct mutex            pg_vec_lock;
    unsigned long           flags;
    int                     ifindex; /* bound device */
    u8                      vnet_hdr_sz;
    __be16                  num;
    struct packet_rollover  *rollover;
    struct packet_mclist    *mclist;
    atomic_long_t           mapped;
    enum tpacket_versions   tp_version;
    unsigned int            tp_hdrlen;
    unsigned int            tp_reserve;
    unsigned int            tp_tstamp;
    struct completion       skb_completion;
    struct net_device __rcu *cached_dev;
    struct packet_type      prot_hook ____cacheline_aligned_in_smp;
    atomic_t                tp_drops ____cacheline_aligned_in_smp;
};
```

#### `packet_ring_buffer` [核心]

> 正常 Linux Socket 的传输数据需要进行用户层到内核层的拷贝，然后发送后又需要内核层到用户层的拷贝，比较消耗性能。所以类似于 zerocopy 的设计思想，Linux 在 PacketSocket 中设计了共享环形结构，让用户态可以直接将数据写入环形结构避免了拷贝操作。当然如果设置了 `ring_buffer`，但是不使用 `mmap` 是没有使用到 zerocopy 的。

- `pg_vec` 是 `pgv` 数组，每个元素对应一个 block
- `pg_vec_len` 是 block 数量
- `pg_vec_pages` 是每个 block 的页数
- `pg_vec_order` 是页分配阶数

```c
struct packet_ring_buffer {
    struct pgv              *pg_vec;
    unsigned int            head;
    unsigned int            frames_per_block;
    unsigned int            frame_size;
    unsigned int            frame_max;
    unsigned int            pg_vec_order;
    unsigned int            pg_vec_pages;
    unsigned int            pg_vec_len;
    unsigned int __percpu   *pending_refcnt;
    union {
        unsigned long                   *rx_owner_map;
        struct tpacket_kbdq_core        prb_bdqc;
    };
};
```

#### `pg_vec` [漏洞目标结构]

![图片描述](https://bbs.kanxue.com/upload/attach/202512/994584_AZFPKVD3CVRGNRF.webp)

- `pgv`（Page Vector）是页向量，每个 `pgv` 指向一个内存块（block）的起始地址，一个 block 对应一个或者多个完整的 page。
- `buffer` 指向该 block 的实际内存，block 用于存储我们的网络请求等信息。

```c
struct pgv {
    char *buffer;
};
```

#### Socket 缓冲区对比

![图片描述](https://bbs.kanxue.com/upload/attach/202512/994584_GSVQ7ZQUTEK4MWN.webp)

### Socket 创建流程

#### 调用链

```text
用户空间: socket(AF_PACKET, SOCK_RAW, htons(ETH_P_ALL))
↓
sys_socket()
↓
__sock_create()
↓
pf->create() [packet_proto.create = packet_create]
↓
packet_create()
├─ sk_alloc() // 分配 sock 和 packet_sock
├─ packet_alloc_pending() // 分配 pending 结构
├─ sock_init_data() // 初始化 sock 数据
├─ mutex_init(&po->pg_vec_lock) // 初始化 pg_vec 锁
├─ po->prot_hook.func = packet_rcv // 设置接收函数
└─ __register_prot_hook() // 注册协议钩子
└─ dev_add_pack(&po->prot_hook) // 添加到协议栈
```

#### Setsockopt 设置 RingBuffer

> 用户通过 `setsockopt` 设置 `PACKET_RX_RING` 或 `PACKET_TX_RING`，最终调用 `packet_set_ring`，设置环形缓冲区。

#### 调用链

```text
用户空间: setsockopt(fd, SOL_PACKET, PACKET_RX_RING, &req, sizeof(req))
↓
sys_setsockopt()
↓
sock_setsockopt()
↓
packet_setsockopt()
↓
case PACKET_RX_RING:
packet_set_ring()
├─ 参数验证
├─ order = get_order(req->tp_block_size) // 计算页阶数
├─ alloc_pg_vec(req, order) // 分配 pg_vec 数组
│   ├─ kcalloc(block_nr, sizeof(struct pgv)) // 分配 pg_vec 数组
│   └─ for each block:
│       └─ alloc_one_pg_vec_page(order) // 分配每个 block 的内存
│           ├─ __get_free_pages() // 优先使用连续物理页
│           ├─ vzalloc() // 失败则使用虚拟连续内存
│           └─ __get_free_pages() // 最后重试（允许 swap）
├─ init_prb_bdqc() [V3 only] // 初始化 V3 块描述符
│   ├─ p1->pkbdq = pg_vec // 设置 pg_vec 指针
│   └─ prb_open_block() // 打开第一个 block
├─ 临时卸载协议钩子
├─ swap(rb->pg_vec, pg_vec) // 交换 pg_vec
├─ 设置 ring buffer 参数
└─ 重新注册协议钩子
└─ po->prot_hook.func = tpacket_rcv // 切换到 tpacket_rcv
```

#### `packet_set_ring`(入口)

- 参数校验（`block_size`、`frame_size` 等）
- 计算 `order`（页分配阶数）
- 调用 `alloc_pg_vec` 分配 `pg_vec`
- 根据版本初始化（V3 调用 `init_prb_bdqc`，V1/V2 分配 `rx_owner_map`）
- 临时卸载协议钩子，设置 `pg_vec`，再重新注册
- 若设置了 `rx_ring.pg_vec`，将接收函数切换为 `tpacket_rcv`
