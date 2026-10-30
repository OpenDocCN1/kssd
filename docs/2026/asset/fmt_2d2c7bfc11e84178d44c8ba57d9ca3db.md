
# [原创]CVE-2024-0582 内核提权详细分析

## 漏洞简介

**漏洞编号**: `CVE-2024-0582`

**影响版本**：v6.4 < Linux Kernel < v[6.6.5](https://bbs.kanxue.com/elink@352K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0k6r3&6Q4x3X3g2C8k6i4u0F1k6h3I4Q4x3X3g2G2M7X3N6Q4x3V1k6H3N6h3u0Q4x3V1k6D9K9h3&6#2P5q4)9J5c8X3E0W2M7X3&6W2L8q4)9J5c8Y4j5$3i4K6u0W2P5q4)9J5c8V1y4Z5j5h3&6Y4k6f1I4G2k6#2)9J5k6o6k6Q4x3X3f1$3i4K6u0W2y4b7`.`.)

**漏洞产品**: linux kernel - `io_uring` & `io_unregister_pbuf_ring` & `uaf`

**利用效果**: 本地提权

![](https://bbs.kanxue.com/upload/attach/202507/1026022_FBN4MP4PAYKR5T7.jpg)

## 环境搭建

**复现环境**：qemu + linux kernel v6.5.3

**环境附件：** [mowenroot/Kernel](https://bbs.kanxue.com/elink@9ccK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0L8%4N6W2L8Y4u0G2L8%4c8Q4x3V1k6w2k6i4u0F1k6h3H3`.)

**复现流程**： 执行exp后，账号:`hacker`的root用户被添加。`su hacker`完成提权。

![](https://bbs.kanxue.com/upload/attach/202507/1026022_VRD232XRBQGDP2Y.jpg)

## 漏洞原理

漏洞本质是 `uaf`。从内核版本 **5.7** 开始，为了便于管理不同的缓冲区集，`io_uring` 允许应用程序注册由组 ID 标识的缓冲区池。通过 `io_uring_register` 的 `opcode->IORING_REGISTER_PBUF_RING` 调用 `io_register_pbuf_ring()` 来完成注册ID标识缓冲区。并从内核版本 **6.4** 开始，`io_uring` 还允许用户将提供的缓冲区环的分配委托给内核，由 `IOU_PBUF_RING_MMAP` 标识符即可生成。调用 `IOU_PBUF_RING_MMAP` 由内核完成分配空间后，然后使用 `mmap()` 标识符映射到用户的地址,但是这个操作不会修改**页面结构(pgae)的引用计数**，然后使用 `io_unregister_pbuf_ring()` 释放申请的空间的时候会调用 `put_page_testzero(page)`，对 `page` 引用`-1`并判断引用是否为 **0**，如果为 **0** 就会释放 `page` ，因为 `mmap` 映射的时候并不会**页面结构(pgae)的引用计数**，内核并不知道是否取消了内存的映射。所以就会出现映射未取消就释放 `page` 的情况，而导致用户虚拟地址对物理地址映射未取消的 `UAF` 。

---

## 漏洞分析

关于`io_uring`的一些基础知识之前的文章已经详细介绍过，如果师傅们感兴趣可以看看之前的文章。接下来只介绍漏洞相关的点。

**NVD描述**：A memory leak flaw was found in the Linux kernel’s `io_uring` functionality in how a user registers a buffer ring with `IORING_REGISTER_PBUF_RING`, `mmap()` it, and then frees it. This flaw allows a local user to crash or potentially escalate their privileges on the system.

`io_uring_register` 提供了接口 `io_register_pbuf_ring` 来完成注册 `ID` 标识缓冲区。旨在通过 `ID` 来对不同的缓冲区进行管理。下面分析的 `kernel` 源码版本为 `v6.5.3`。

![](https://bbs.kanxue.com/upload/attach/202507/1026022_JSYYKJA23XNJ5TK.jpg)

### `io_register_pbuf_ring`

「1」 首先把用户数据复制到内核的 `reg`。然后参数检查，判断 `entries` 是否为 `2` 次幂，并且限制 `entries` 最大不能超过 `65536`。需要注意一点:意味着条目最大为 `32768`。

「2」 如果 `ctx->io_bl` 为初始化时，会尝试初始化 `ctx->io_bl`，这里的 `io_bl`是一个 `64` 长度的数组。数组中存放 `io_buffer_list`。如果 `id` 大于等于 `64` 时就会通过 `xarray` 来管理。

「3」 调用 `io_buffer_get_list()` ,通过 `id` 获取对应 `bl (buf list)`，如果 `bl` 为空时，则会申请结构体 `io_buffer_list` 空间，标志为 `GFP_KERNEL`。

「4」 当使用 `IOU_PBUF_RING_MMAP` 时，会调用 `io_alloc_pbuf_ring()`,来申请 `buf_ring` 空间，否则内存页，最后将 bl加入到`ctx`中。

使用 `IOU_PBUF_RING_MMAP` 就会完成对 `buf_ring` 的空间分配，全程都由内核来完成，不同于 `io_sqe_buffers_register` 需要用户传入注册的地址。用户只需要指定对应注册的 `ID` 即可。

```
int io_register_pbuf_ring(struct io_ring_ctx *ctx, void __user *arg)
{
    struct io_uring_buf_reg reg;
    struct io_buffer_list *bl, *free_bl = NULL;
    int ret;

    if (copy_from_user(&reg, arg, sizeof(reg)))
        return -EFAULT;

    if (reg.resv[0] || reg.resv[1] || reg.resv[2])
        return -EINVAL;
    if (reg.flags & ~IOU_PBUF_RING_MMAP)
        return -EINVAL;
    if (!(reg.flags & IOU_PBUF_RING_MMAP)) {
        if (!reg.ring_addr)
            return -EFAULT;
        if (reg.ring_addr & ~PAGE_MASK)
            return -EINVAL;
    } else {
        if (reg.ring_addr)
            return -EINVAL;
    }
    if (!is_power_of_2(reg.ring_entries))
        return -EINVAL;
    if (reg.ring_entries >= 65536)
        return -EINVAL;
    if (unlikely(reg.bgid < BGID_ARRAY && !ctx->io_bl)) {
        int ret = io_init_bl_list(ctx);
        if (ret)
            return ret;
    }
    bl = io_buffer_get_list(ctx, reg.bgid);
    if (bl) {
        if (bl->is_mapped || !list_empty(&bl->buf_list))
            return -EEXIST;
    } else {
        free_bl = bl = kzalloc(sizeof(*bl), GFP_KERNEL);
        if (!bl)
            return -ENOMEM;
    }

    if (!(reg.flags & IOU_PBUF_RING_MMAP))
        ret = io_pin_pbuf_ring(&reg, bl);
    else
        ret = io_alloc_pbuf_ring(&reg, bl);

    if (!ret) {
        bl->nr_entries = reg.ring_entries;
        bl->mask = reg.ring_entries - 1;
        io_buffer_add_list(ctx, bl, reg.bgid);
        return 0;
    }

    kfree(free_bl);
    return ret;
}
```

#### `io_buffer_get_list`

「1」 返回 `ID` 对应的 `io_buffer_list(bl)`。 当 ID 符合限制的时候，会使用数组来管理 `io_buffer_list` ，这个时候直接返回 `ID`对应的 `bl` 即可。如果超过限制，则使用`xarray` 来管理。

```
static inline struct io_buffer_list *io_buffer_get_list(struct io_ring_ctx *ctx,
                            unsigned int bgid)
{
    if (ctx->io_bl && bgid < BGID_ARRAY)
        return &ctx->io_bl[bgid];
    return xa_load(&ctx->io_bl_xa, bgid);
}
```

#### `io_alloc_pbuf_ring`

「1」 为 `io_uring_buf_ring` 申请空间 。 `buf_ring` 环存放 `ring_entries` 个 `io_uring_buf_ring`,而 `io_uring_buf_ring` 中存放地址、长度等信息。值得注意的是：申请 `pages` 为复合页。

```
static int io_alloc_pbuf_ring(struct io_uring_buf_reg *reg,
                  struct io_buffer_list *bl)
{
    gfp_t gfp = GFP_KERNEL_ACCOUNT | __GFP_ZERO | __GFP_NOWARN | __GFP_COMP;
    size_t ring_size;
    void *ptr;
    ring_size = reg->ring_entries * sizeof(struct io_uring_buf_ring);
    ptr = (void *) __get_free_pages(gfp, get_order(ring_size));
    if (!ptr)
        return -ENOMEM;

    bl->buf_ring = ptr;
    bl->is_mapped = 1;
    bl->is_mmap = 1;
    return 0;
}
```

`io_uring_buf`、`io_uring_buf_ring` 结构体

```
struct io_uring_buf {
    __u64	addr;
    __u32	len;
    __u16	bid;
    __u16	resv;
};

struct io_uring_buf_ring {
    union {
        struct {
            __u64	resv1;
            __u32	resv2;
            __u16	resv3;
            __u16	tail;
        };
        __DECLARE_FLEX_ARRAY(struct io_uring_buf, bufs);
    };
};
```

### `io_unregister_pbuf_ring`

「1」 先根据 `id` 获取 `io_buffer_list`，如果 `id` 符合标志则调用 `__io_remove_buffers` 仅仅释放 `bl->buf_ring` 。而不符合标准的使用 `xarray` 来管理，会直接释放整个 `bl`。

```
int io_unregister_pbuf_ring(struct io_ring_ctx *ctx, void __user *arg)
{
    struct io_uring_buf_reg reg;
    struct io_buffer_list *bl;

    if (copy_from_user(&reg, arg, sizeof(reg)))
        return -EFAULT;
    if (reg.resv[0] || reg.resv[1] || reg.resv[2])
        return -EINVAL;
    if (reg.flags)
        return -EINVAL;
    bl = io_buffer_get_list(ctx, reg.bgid);
    if (!bl)
        return -ENOENT;
    if (!bl->is_mapped)
        return -EINVAL;
    __io_remove_buffers(ctx, bl, -1U);
    if (bl->bgid >= BGID_ARRAY) {
        xa_erase(&ctx->io_bl_xa, bl->bgid);
        kfree(bl);
    }
    return 0;
}
```

#### `__io_remove_buffers`

「1」 由 `IORING_REGISTER_PBUF_RING` 申请的空间，会先获取 `bl->buf_ring` 的虚拟地址，然后调用 `put_page_testzero()`，对 `page` 引用 `-1`，如果`当前引用==0`，则释放 `page`，但是使用 `mmap` 映射持有时，不会对 `page` 引用改变,内核无法知道此时映射是否取消。
