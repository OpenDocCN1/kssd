
```ql
/**
 * @kind path-problem
 */

import cpp
import semmle.code.cpp.dataflow.TaintTracking
import DataFlow::PathGraph

class MyExpr extends Expr {
  // 2.0 Todo
  MacroInvocation mi;

  MyExpr() {
    mi.getMacro().getName().regexpMatch("ntoh(l|ll|s)")
    and this = mi.getExpr()
  }
}

class Config extends TaintTracking::Configuration {
  Config() { this = "NetworkToMemFuncLength" }

  override predicate isSource(DataFlow::Node source) {
    // 2.1 Todo
    source.asExpr() instanceof MyExpr
  }

  override predicate isSink(DataFlow::Node sink) {
    // 2.1Todo
    // 存在memcpy的函数调用，并且该调用的第二个参数是sink.asExpr()
    exists(FunctionCall fc | fc.getTarget().hasName("memcpy") | sink.asExpr() = fc.getArgument(2))
  }
}

from Config cfg, DataFlow::PathNode source, DataFlow::PathNode sink
where cfg.hasFlowPath(source, sink)
select sink, source, sink, "ntoh flows to memcpy"
```

![image-20230610172948004](https://bbs.kanxue.com/upload/attach/202306/830671_SCAAHPEMUBGXWYC.jpg)

## 5 Step 3: Find additional vulnerabilities

## 根据Q2.1查询结果进行代码审计

**Question 3.0: There are 13 known vulnerabilities in U-Boot.**

The query you completed above probably found 9 of them. See if you can refine your query to find 1 or more additional vulnerabilities.

上面完成的查询可能找到13个漏洞中的9个。这里让我们尝试改进查询，以找到更多的漏洞。

在改进前，我们先根据前面的查询结果看看是不是已经能找到9个漏洞了。

### CVE-2019-14192、 CVE-2019-14193 和 CVE-2019-14199

结果【1】和【2】的`source`是在函数`net_process_received_packet`的末尾，此处很明显会发生integer underflow：

![image-20230610220449399](https://bbs.kanxue.com/upload/attach/202306/830671_65PKERC296HGCU5.jpg)

然后将underflow的整数传递进函数`nc_input_packet`，但是第149行检测了`len`的大小，这样我觉得即使传入一个很大的`len`，这里的`memcpy`也不会发生越界写。。然而我查了一下，[U-Boot NFS RCE Vulnerabilities (CVE-2019-14192)](https://bbs.kanxue.com/elink@925K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3y4#2M7X3W2@1P5h3I4S2j5W2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3y4G2L8g2)9J5c8Y4u0W2M7$3g2S2M7X3y4Z5i4K6u0r3N6h3u0G2L8%4c8Q4x3X3c8J5j5$3g2Q4x3X3c8F1k6Y4y4Q4x3X3c8$3N6h3I4F1k6i4u0S2j5X3W2D9K9i4c8&6i4K6u0r3)说这里是会越界写的。。。但是他就一句话，没懂。。。

![image-20230610221242234](https://bbs.kanxue.com/upload/attach/202306/830671_W7VAMQGAXHMMGPG.jpg)

然后找了一下[patch](https://bbs.kanxue.com/elink@84fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3X3g2V1k6h3&6^5i4K6u0W2k6r3g2Q4x3V1k6#2i4K6u0V1j5X3!0G2N6q4)9J5c8Y4g2Q4x3X3c8T1L8$3!0@1i4K6u0r3i4K6u0V1i4K6u0r3j5$3!0E0L8h3W2@1i4K6u0r3k6X3f1%4x3U0R3^5x3o6j5&6k6o6u0W2y4U0j5#2z5e0p5I4y4K6l9@1z5h3j5%4k6o6t1%4k6e0t1$3x3h3t1#2y4e0m8T1j5U0M7J5y4b7`.`.)，对应的编号有CVE-2019-14192、 CVE-2019-14193 和 CVE-2019-14199。三个？

![image-20230611024205527](https://bbs.kanxue.com/upload/attach/202306/830671_5U6QJVVNZZZPDST.jpg)

### CVE-2019-14197、CVE-2019-14200、 CVE-2019-14201、CVE-2019-14202、CVE-2019-14203 和 CVE-2019-14204

结果【1】【2】的`source`边上很明显还有一个整数回绕，将underflow的整数传递给函数指针`udp_packet_handler`，QL的查询结果里没有此处。

![image-20230611015647333](https://bbs.kanxue.com/upload/attach/202306/830671_H6GPB6KQMPRPD2M.jpg)

先审计一下是否存在漏洞，搜索`udp_packet_handler`，只有两处赋值的地方：

![image-20230611015913129](https://bbs.kanxue.com/upload/attach/202306/830671_Z7VRQ6VR24ADBXJ.jpg)

再搜索函数`net_set_udp_handler`，可以看到分别给nc、dhcp、bootp和nfs都设置了handler。

![image-20230611020024472](https://bbs.kanxue.com/upload/attach/202306/830671_RKRV8F82YV8H45F.jpg)

不想一个一个审计了，直接看文章[U-Boot NFS RCE Vulnerabilities (CVE-2019-14192)](https://bbs.kanxue.com/elink@d17K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3y4#2M7X3W2@1P5h3I4S2j5W2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3y4G2L8g2)9J5c8Y4u0W2M7$3g2S2M7X3y4Z5i4K6u0r3N6h3u0G2L8%4c8Q4x3X3c8J5j5$3g2Q4x3X3c8F1k6Y4y4Q4x3X3c8$3N6h3I4F1k6i4u0S2j5X3W2D9K9i4c8&6i4K6u0r3)知道漏洞都存在于`nfs_handler`。那就只审计它。可以看到，该函数仍然未校验`len`，直接在不同分支里将其分别传递给函数:

```c
static void nfs_handler(uchar *pkt, unsigned dest, struct in_addr sip, unsigned src, unsigned len)
{
    int rlen;
    int reply;

    debug("%s\n", __func__);

    if (dest != nfs_our_port)
        return;

    switch (nfs_state) {
    case STATE_PRCLOOKUP_PROG_MOUNT_REQ:
        if (rpc_lookup_reply(PROG_MOUNT, pkt, len) == -NFS_RPC_DROP)
            break;
        ......
        break;

    case STATE_PRCLOOKUP_PROG_NFS_REQ:
        if (rpc_lookup_reply(PROG_NFS, pkt, len) == -NFS_RPC_DROP)
            break;
        ......
        break;

    case STATE_MOUNT_REQ:
        reply = nfs_mount_reply(pkt, len);
        ......
        break;

    case STATE_UMOUNT_REQ:
        reply = nfs_umountall_reply(pkt, len);
        ......
        break;

    case STATE_LOOKUP_REQ:
        reply = nfs_lookup_reply(pkt, len);
        ......
        break;

    case STATE_READLINK_REQ:
        reply = nfs_readlink_reply(pkt, len);
        ......
        break;

    case STATE_READ_REQ:
        rlen = nfs_read_reply(pkt, len);
        ......
        break;
    }
}
```

- `rpc_lookup_reply` ->未检验`len`，直接`memcpy(&rpc_pkt.u.data[0], pkt, len);` -> 存在越界写漏洞
- `nfs_mount_reply`->未检验`len`，直接`memcpy(&rpc_pkt.u.data[0], pkt, len);` -> 存在越界写漏洞
- `nfs_umountall_reply`->未检验`len`，直接`memcpy(&rpc_pkt.u.data[0], pkt, len);` -> 存在越界写漏洞
- `nfs_lookup_reply`->未检验`len`，直接`memcpy(&rpc_pkt.u.data[0], pkt, len);` -> 存在越界写漏洞
- `nfs_readlink_reply`->未检验`len`，直接`memcpy((unsigned char *)&rpc_pkt, pkt, len);`-> 存在越界写漏洞
- `nfs_read_reply`->差点就错过了
  - `memcpy(&rpc_pkt.u.data[0], pkt, sizeof(rpc_pkt.u.reply));`
  - `pkt`的大小可能没有`sizeof(rpc_pkt.u.reply)`这么大，存在越界读

```c
struct rpc_t {
    union {
        uint8_t data[NFS_READ_SIZE + (6 + NFS_MAX_ATTRS) * sizeof(uint32_t)];
        struct {
            uint32_t id;
            uint32_t type;
            uint32_t rpcvers;
            uint32_t prog;
            uint32_t vers;
            uint32_t proc;
            uint32_t data[1];
        } call;
        struct {
            uint32_t id;
            uint32_t type;
            uint32_t rstatus;
            uint32_t verifier;
            uint32_t v2;
            uint32_t astatus;
            uint32_t data[NFS_READ_SIZE / sizeof(uint32_t) + NFS_MAX_ATTRS];
        } reply;
    } u;
} __attribute__((packed));
```

所以，这里有6个漏洞，经查询[patch](https://bbs.kanxue.com/elink@c78K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3X3g2V1k6h3&6^5i4K6u0W2k6r3g2Q4x3V1k6#2i4K6u0V1j5X3!0G2N6q4)9J5c8Y4g2Q4x3X3c8T1L8$3!0@1i4K6u0r3i4K6u0V1i4K6u0r3j5$3!0E0L8h3W2@1i4K6u0r3y4K6b7I4j5e0S2S2x3o6S2W2j5X3f1#2j5X3x3K6j5$3y4X3k6e0y4U0k6r3f1$3j5K6u0T1y4o6c8W2k6e0f1K6z5o6V1I4j5h3j5J5x3b7`.`.)，对应的CVE编号为：CVE-2019-14197、CVE-2019-14200、 CVE-2019-14201、CVE-2019-14202、CVE-2019-14203 和 CVE-2019-14204。

![image-20230611022343305](https://bbs.kanxue.com/upload/attach/202306/830671_EJ655MJU85Q2NSH.jpg)

### CVE-2019-14195

结果【4】【5】对应存在一个漏洞，从packet获取`rlen`之后，没有校验就直接传给`memcpy`。

![image-20230610174908752](https://bbs.kanxue.com/upload/attach/202306/830671_EJ7MPGZ2WKKJM4N.jpg)

经查询，CVE编号为：CVE-2019-14195，[patch](https://bbs.kanxue.com/elink@34fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3X3g2V1k6h3&6^5i4K6u0W2k6r3g2Q4x3V1k6#2i4K6u0V1j5X3!0G2N6q4)9J5c8Y4g2Q4x3X3c8T1L8$3!0@1i4K6u0r3i4K6u0V1i4K6u0r3j5$3!0E0L8h3W2@1i4K6u0r3j5$3j5K6j5e0c8X3x3h3f1^5y4X3g2U0k6r3b7J5y4r3j5^5y4$3t1$3x3e0f1H3y4e0q4T1y4o6W2V1z5r3f1I4z5e0j5^5j5K6t1K6x3l9`.`.)如下：

![image-20230610191127384](https://bbs.kanxue.com/upload/attach/202306/830671_9BU67ZBQCAPEC8F.jpg)

### CVE-2019-14196

【6】`nfs_lookup_reply()`，目标buffer：`filefh`的长度是64，传入的长度是负数的时候就会越界写。漏洞+1。

![image-20230610182803631](https://bbs.kanxue.com/upload/attach/202306/830671_EV33FF8Y88SBMSH.jpg)
