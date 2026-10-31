
# [原创]Linux内核攻击 USMA 解析

> USMA : USER SPEACE MAPPING ATTACK (用户空间映射攻击)
>
> 本文关于 Linux 源码的版本均为:6.12.32

## 参考文章:

- <mark class="encrypted">e3aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9N6r3k6S2x3h3I4Q4x3X3g2@1L8%4m8Q4x3V1j5J5x3o6t1#2i4K6u0r3x3o6u0Q4x3V1j5J5y4g2)9J5c8Y4y4&6M7%4c8W2L8g2)9J5c8X3E0W2M7X3&6W2L8q4)9J5c8V1I4A6L8Y4g2^5i4K6g2X3K9$3g2J5L8X3g2D9z5q4)9#2k6W2g2e0e0f1q4Q4x3V1j5`.</mark>

## Pakcet Socket

> USMA 利用的是 `socket` 的 `pgv` 数组,所以我们在这里先深入了解一下 PacketSocket 的数据结构,在这里我们还是主要关注 `pgv` 的创建和映射,关于 Socket 其他板块的详细解析可以看我其他的文章。

### 数据结构

#### `packet_socket`

> 我们这里主要关注 `rx_ring/tx_ing`,这是我们后续利用的关键数据结构。
>
> 所以相当于除去 `struct socket` 以外其他均为 `packetsocket` 独有的结构.

- `sk`: 继承的通用 `socket *`
- `fanout`: `fanout` 组
- `rx_ring/tx_ring`: 接收/发送环形缓冲区（`mmap`）
- `prot_hook`: 协议钩子，用于接收数据包
- `ifindex`: 绑定的网络设备索引
- `tp_version`: `TPACKET` 版本（V1/V2/V3）

```C++
struct packet\_sock {
...
};
```

#### `packet_ring_buffer` [核心]

> 正常 Linux Socket 的传输数据需要进行用户层到内核层的拷贝，然后发送后又需要内核层到用户层的拷贝，比较消耗性能。所以类似于 zerocopy 的设计思想，Linux 在 PacketSocket 中设计了共享环形结构，让用户态可以直接将数据写入环形结构避免了拷贝操作。当然如果设置了 `ring_buffer`，但是不使用 `mmap` 是没有使用到 zerocopy 的。
>
> `ring_buffer` 是 `packet_socket` 独有的数据结构。当没有设置 `PACKET_RX_RING` 或 `PACKET_TX_RING` 时，`Packet_socket` 和正常的 `socket` 一样都使用标准的 `sk_receive_queue/sk_write_queue`。

- `pg_vec` 是 `pgv` 数组，每个元素对应一个 `block`
- `pg_vec_len` 是 `block` 数量
- `pg_vec_pages` 是每个 `block` 的页数
- `pg_vec_order` 是页分配阶数

```C++
struct packet\_ring\_buffer {
...
};
```

#### `Pg_vec` [漏洞目标结构]

![图片描述](upload/attach/202512/994584\_AZFPKVD3CVRGNRF.webp)

- `pgv`（Page Vector）是页向量，每个 `pgv` 指向一个内存块（`block`）的起始地址，一个 `block` 对应一个或者多个完整的 `page`。
- `buffer` 指向该 `block` 的实际内存，`Block` 用于存储我们的网络请求等信息。

```C
struct pgv {
char \*buffer;
};
```

#### Socket 缓冲区对比

![图片描述](upload/attach/202512/994584\_GSVQ7ZQUTEK4MWN.webp)

### Socket 创建流程

#### 调用链

```C
用户空间: socket(AF\_PACKET, SOCK\_RAW, htons(ETH\_P\_ALL))
↓
sys\_socket()
↓
\_\_sock\_create()
↓
pf->create() [packet\_proto.create = packet\_create]
↓
packet\_create()
├─ sk\_alloc() // 分配 sock 和 packet\_sock
├─ packet\_alloc\_pending() // 分配 pending 结构
├─ sock\_init\_data() // 初始化 sock 数据
├─ mutex\_init(&po->pg\_vec\_lock) // 初始化 pg\_vec 锁
├─ po->prot\_hook.func = packet\_rcv // 设置接收函数
└─ \_\_register\_prot\_hook() // 注册协议钩子
└─ dev\_add\_pack(&po->prot\_hook) // 添加到协议栈
```

### `Setsockopt` 设置 RingBuffer

> 用户通过 `setsockopt` 设置 `PACKET_RX_RING` 或 `PACKET_TX_RING`，最终调用 `packet_set_ring`，设置环形缓冲区。

#### 调用链

```C
用户空间: setsockopt(fd, SOL\_PACKET, PACKET\_RX\_RING, &req, sizeof(req))
↓
sys\_setsockopt()
↓
sock\_setsockopt()
↓
packet\_setsockopt()
↓
case PACKET\_RX\_RING:
packet\_set\_ring()
├─ 参数验证
├─ order = get\_order(req->tp\_block\_size) // 计算页阶数
├─ alloc\_pg\_vec(req, order) // 分配 pg\_vec 数组
│ ├─ kcalloc(block\_nr, sizeof(struct pgv)) // 分配 pg\_vec 数组
│ └─ for each block:
│ └─ alloc\_one\_pg\_vec\_page(order) // 分配每个 block 的内存
│ ├─ \_\_get\_free\_pages() // 优先使用连续物理页
│ ├─ vzalloc() // 失败则使用虚拟连续内存
│ └─ \_\_get\_free\_pages() // 最后重试（允许 swap）
├─ init\_prb\_bdqc() [V3 only] // 初始化 V3 块描述符
│ ├─ p1->pkbdq = pg\_vec // 设置 pg\_vec 指针
│ └─ prb\_open\_block() // 打开第一个 block
├─ 临时卸载协议钩子
├─ swap(rb->pg\_vec, pg\_vec) // 交换 pg\_vec
├─ 设置 ring buffer 参数
└─ 重新注册协议钩子
└─ po->prot\_hook.func = tpacket\_rcv // 切换到 tpacket\_rcv
```

#### `packet_set_ring`(入口)

- 参数校验（`block_size`、`frame_size` 等）
- 计算 `order`（页分配阶数）
- 调用 `alloc_pg_vec` 分配 `pg_vec`
- 根据版本初始化（V3 调用 `init_prb_bdqc`，V1/V2 分配 `rx_owner_map`）
- 临时卸载协议钩子，设置 `pg_vec`，再重新注册
- 若设置了 `rx_ring.pg_vec`，将接收函数切换为 `tpacket_rcv`
