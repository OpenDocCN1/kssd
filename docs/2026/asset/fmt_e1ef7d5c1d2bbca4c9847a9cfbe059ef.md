
```
static int nfs_read_reply(uchar *pkt, unsigned len)
{
        [...]

    if (supported_nfs_versions & NFSV2_FLAG) {
        rlen = ntohl(rpc_pkt.u.reply.data[18]); // <-- rlen is attacker-controlled could be 0xFFFFFFFF
        data_ptr = (uchar *)&(rpc_pkt.u.reply.data[19]);
    } else {  /* NFSV3_FLAG */
        int nfsv3_data_offset =
            nfs3_get_attributes_offset(rpc_pkt.u.reply.data);

        /* count value */
        rlen = ntohl(rpc_pkt.u.reply.data[1 + nfsv3_data_offset]); // <-- rlen is attacker-controlled
        /* Skip unused values :
            EOF:        32 bits value,
            data_size:    32 bits value,
        */
        data_ptr = (uchar *)
            &(rpc_pkt.u.reply.data[4 + nfsv3_data_offset]);
    }

    if (store_block(data_ptr, nfs_offset, rlen)) // <-- We pass to store_block source and length controlled by the attacker
            return -9999;

    [...]
}
```

关注[`store_block`](https://bbs.kanxue.com/elink@e0aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6#2i4K6u0V1j5X3!0G2N6q4)9J5c8Y4g2Q4x3X3c8T1L8$3!0@1i4K6u0r3j5X3I4G2j5W2)9J5c8U0g2T1y4r3t1$3z5o6m8U0k6X3f1J5z5h3p5$3y4K6p5%4x3h3y4U0j5X3f1@1x3K6W2U0y4U0j5K6y4K6c8U0j5U0x3I4k6X3q4U0j5e0y4Q4x3V1k6F1k6i4c8Q4x3V1k6F1k6i4c8Q4x3X3g2U0i4K6t1K6e0o6R3I4)函数的物理内存部分，它尝试使用特定于arch的函数`map_physmem`保留一些内存，最终调用`phys_to_virt`。正如您在[x86实现](https://bbs.kanxue.com/elink@882K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6#2i4K6u0V1j5X3!0G2N6q4)9J5c8Y4g2Q4x3X3c8T1L8$3!0@1i4K6u0r3j5X3I4G2j5W2)9J5c8U0V1@1z5e0l9#2k6e0q4V1j5U0S2V1z5r3b7@1x3X3x3@1k6U0x3&6k6U0p5@1k6r3u0W2k6e0u0X3z5e0M7^5z5o6x3&6x3r3c8T1y4h3g2Q4x3V1k6A6L8X3y4D9N6h3c8W2i4K6u0r3j5i4y4E0i4K6u0V1k6$3g2F1k6i4u0A6j5#2)9J5c8X3W2G2i4K6u0W2K9q4)9J5x3@1H3K6x3l9`.`.)中所看到的那样，在保留物理内存时，它显然忽略了长度，并为您提供了原始指针，而没有检查周围区域是否为其他目的保留。

```
static inline void *phys_to_virt(phys_addr_t paddr)
{
        return (void *)(unsigned long)paddr;
}
```

其后在`store_block`中有一个攻击者控制的源和长度的`memcpy`缓冲区溢出。

```
static inline int store_block(uchar *src, unsigned offset, unsigned len)
{
    [...]

    void *ptr = map_sysmem(load_addr + offset, len); // <-- essentially this is ptr = load_addr + offset
    memcpy(ptr, src, len); // <-- unrestricted overflow happens here
    unmap_sysmem(ptr);

    [...]
}
```

`flash_write`代码路径中也可能存在类似的问题。

## 由于整数下溢，在解析UDP数据包时出现未限界memcpy

函数`net_process_received_packet`在[未经验证的情况下使用`ip->udp_len`](https://bbs.kanxue.com/elink@6c6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6#2i4K6u0V1j5X3!0G2N6q4)9J5c8Y4g2Q4x3X3c8T1L8$3!0@1i4K6u0r3j5X3I4G2j5W2)9J5c8U0g2T1y4r3t1$3z5o6m8U0k6X3f1J5z5h3p5$3y4K6p5%4x3h3y4U0j5X3f1@1x3K6W2U0y4U0j5K6y4K6c8U0j5U0x3I4k6X3q4U0j5e0y4Q4x3V1k6F1k6i4c8Q4x3V1k6F1k6i4c8Q4x3X3g2U0i4K6t1K6e0o6p5K6x3o6u0Q4x3X3c8x3x3e0x3I4y4l9`.`.)会导致整数下溢。其后，此字段将用于通过`net_set_udp_handler`（DNS，dhcp，...）设置的[`memcpy`at`nc_input_packet`](https://bbs.kanxue.com/elink@eadK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6#2i4K6u0V1j5X3!0G2N6q4)9J5c8Y4g2Q4x3X3c8T1L8$3!0@1i4K6u0r3j5X3I4G2j5W2)9J5c8U0V1@1z5e0l9#2k6e0q4V1j5U0S2V1z5r3b7@1x3X3x3@1k6U0x3&6k6U0p5@1k6r3u0W2k6e0u0X3z5e0M7^5z5o6x3&6x3r3c8T1y4h3g2Q4x3V1k6V1M7X3W2$3k6i4u0K6i4K6u0r3L8X3g2@1i4K6u0r3L8X3g2@1j5$3!0F1M7$3!0D9k6g2)9J5k6h3y4Q4x3U0y4x3x3e0x3@1)和所有udp数据包处理函数中。

```
#if defined(CONFIG_NETCONSOLE) && !defined(CONFIG_SPL_BUILD)
                nc_input_packet((uchar *)ip + IP_UDP_HDR_SIZE,
                                src_ip,
                                ntohs(ip->udp_dst),
                                ntohs(ip->udp_src),
                                ntohs(ip->udp_len) - UDP_HDR_SIZE); // <- integer underflow
#endif

                /*
                 * IP header OK.  Pass the packet to the current handler.
                 */
                (*udp_packet_handler)((uchar *)ip + IP_UDP_HDR_SIZE,
                                      ntohs(ip->udp_dst),
                                      src_ip,
                                      ntohs(ip->udp_src),
                                      ntohs(ip->udp_len) - UDP_HDR_SIZE); // <- integer underflow
```

请注意，我们并未审计为不同目的（DNS，DHCP等）设置的所有潜在udp处理程序。但是，我们确实对`nfs_handler`进行了审计，如下所述。

## 响应辅助函数`nfs_handler`中多个基于堆栈的缓冲区溢出

这是上述漏洞的代码审查变体。在此，当解析很大的`ip->udp_len`参数的udp数据包时会发生整数下溢，随后又调用`nfs_handler`。在此函数中，同样没有对长度的验证，我们将调用辅助函数，例如`nfs_readlink_reply`。该函数盲目使用长度而不进行验证，从而导致基于堆栈的缓冲区溢出。

```
static int nfs_readlink_reply(uchar *pkt, unsigned len)
{
    struct rpc_t rpc_pkt;

    [...]

    memcpy((unsigned char *)&rpc_pkt, pkt, len);
}
```

我们确定了5个不同的易受攻击的函数，它们遵循相同的代码模式，从而导致基于堆栈的缓冲区溢出。除了`nfs_readlink_reply`之外，还有：

- `rpc_lookup_reply`
- `nfs_mount_reply`
- `nfs_umountall_reply`
- `nfs_lookup_reply`

## `nfs_read_reply`中读取越界数据

这与以前的漏洞非常相似。开发人员试图在复制来自套接字的数据时执行大小检查，以保持谨慎。当他们检查以防止缓冲区溢出时，他们没有检查源缓冲区中是否有足够的数据，从而导致潜在的读取越界访问冲突。

```
static int nfs_read_reply(uchar *pkt, unsigned len)
{
    struct rpc_t rpc_pkt;

    [...]

    memcpy(&rpc_pkt.u.data[0], pkt, sizeof(rpc_pkt.u.reply));
}
```

攻击者可以向NFS数据包提供读取请求和发送给套接字的较小数据包请求。

## 有什么建议吗？

为了缓解这些漏洞，只有两种选择：

- 补丁发布后立即应用，或者
- 处于可攻击时，请勿通过NFS或任何U-Boot网络功能挂载文件系统

## 披露时间表

此漏洞报告受披露政策的约束，在此处可见[65dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6%4c8E0i4K6u0W2j5$3!0E0i4K6u0r3M7$3g2U0N6i4u0A6N6s2W2Q4x3V1k6Q4x3U0y4V1K9i4y4U0L8r3!0K6N6i4u0W2i4K6g2X3M7r3!0D9K9h3y4&6](https://bbs.kanxue.com/elink@ae5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6%4c8E0i4K6u0W2j5$3!0E0i4K6u0r3M7$3g2U0N6i4u0A6N6s2W2Q4x3V1k6Q4x3@1k6Q4y4h3k6Q4y4h3k6Z5M7%4c8U0i4K6y4p5y4K6l9J5x3U0f1%4y4o6y4Q4x3X3g2T1j5K6j5#2j5h3t1I4x3$3b7%4y4$3b7$3z5h3f1#2k6X3q4X3x3U0l9I4x3o6m8U0y4$3u0S2x3h3q4S2y4g2)9J5k6e0p5#2y4K6p5H3x3K6b7J5x3K6f1&6y4K6q4Q4x3X3f1I4y4e0M7I4x3o6x3@1x3U0x3#2z5e0M7I4i4K6u0W2x3e0f1%4x3e0l9K6y4o6t1K6y4e0V1%4x3g2)9J5k6e0q4Q4x3U0k6S2L8i4m8Q4x3@1u0Q4y4h3k6Q4y4h3k6Z5M7%4y4U0i4K6y4p5y4K6l9J5x3U0f1%4y4o6y4Q4x3X3f1I4i4K6u0W2x3e0f1%4x3e0l9K6y4o6t1K6y4e0V1%4x3g2)9J5y4X3q4E0M7q4)9K6b7W2)9#2k6W2)9#2k6X3S2K6k6Y4m8Q4x3@1b7^5y4U0f1K6x3e0R3%4x3U0c8Q4x3U0y4V1K9i4y4U0L8r3!0K6N6i4u0W2i4K6g2X3M7r3!0D9K9h3y4&6)。

- 2019年5月15日-Fermín Serna最初发现了两个漏洞，并编写了一个QL查询，以发现另外三个有问题的调用处。
- 2019年5月16日-Pavel Avgustinov带来了一些QL魔术，概括了查询并找到了更多解析ip和udp标头的函数。
- 2019年5月23日-Kevin Backhouse通过`nfs_handler`向Pavel和Fermín警告有关基于堆栈的缓冲区溢出的疏忽。
- 2019年5月23日-Semmle安全团队结束调查并通过电子邮件联系维护者。
- 2019年5月24日-Tom Rini（U-Boot的主要托管人）确认收到安全报告。
- 2019年7月19日-Tom Rini请求在其公共邮件列表u-boot@lists.denx.de上公开此报告。
- 2019年7月22日-为了避免周末披露，Fermin将该报告通过u-boot@lists.denx.de公开。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
