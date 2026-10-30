
速度很大程度上取决于变种和利用程序. 我看过的最高速度的报告是Brandon Falk的[MLPDS exploit](https://bbs.kanxue.com/elink@28bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4j5h3#2G2P5X3!0D9j5h3u0K6i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6E0k6i4c8J5L8$3I4G2k6%4W2Q4x3V1j5J5x3o6p5&6i4K6u0r3x3e0u0Q4x3V1j5K6x3q4)9J5c8X3I4G2j5h3c8Q4x3X3c8H3L8%4u0@1i4K6u0V1L8h3!0F1K9i4c8G2M7W2)9J5k6h3S2@1L8h3I4Q4x3U0y4S2L8W2)9J5k6r3#2D9M7r3c8K6i4K6u0V1k6i4S2H3L8r3!0A6N6l9`.`.), 其速度为228kB/s。 与之相比, 我机器上运行简单版的利用程序的速度仅有25kB/s.

所有变种共有的一个特性是, 它们在泄露的内容上具有概率性。然而[RIDL 论文](https://bbs.kanxue.com/elink@b34K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6s2y4S2N6s2c8S2j5$3E0K6i4K6u0W2j5$3!0E0i4K6u0r3k6X3W2D9k6i4y4Q4x3V1k6J5K9h3c8D9i4K6u0W2M7r3c8X3)描述了一些针对特定值的同步原语, 你通常需要触发对秘密的重复访问才能完全泄露它.

我最终使用不同的MDS变种编写了两个针对Chrome的利用程序, 其中一个针对于使用Xeon Gold 6154处理器的Linux构建, 另外一个是使用Core i7-7600U处理器的Windows。我将描述这两种方法, 因为它们在实践中会遇到不同的挑战.

## 微架构填充缓冲区数据采样(MFBDS)

我的第一个利用程序使用了MFBDS, 针对的是CPU的行填充缓冲区。PoC非常简单:

```
xbegin out            ; 启动TSX来捕获段错误

mov   rax, [0]        ; 读取分页0 => 泄露行填充缓冲区的值

; 其余指令只会推测执行

and   rax, 0xff       ; 掩盖一个字节

shl   rax, 0xc        ; 用作分页索引

add   rax, 0x13370000 ; 加上探测数组(probe array)的地址

prefetchnta [rax]     ; 访问探测数组

xend

out: nop

```

之后你需要定时访问探测数组来查看哪个索引被缓存了.

你可以在修改开头的`0`来控制你泄露的缓存行的偏移量。此外, 你还想对泄露值使用前缀或后缀过滤器, 如[论文](https://bbs.kanxue.com/elink@2baK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6s2y4S2N6s2c8S2j5$3E0K6i4K6u0W2j5$3!0E0i4K6u0r3k6X3W2D9k6i4y4Q4x3V1k6J5K9h3c8D9i4K6u0W2M7r3c8X3)中描述的那样。 要注意的是, 这里只会泄露不在L1缓存的值, 所以你希望有一种办法能在访问之间把秘密移出缓存。

对于我第一个泄露的目标, 我选择了有特权的[`URLLoaderFactory`](https://bbs.kanxue.com/elink@948K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0M7#2)9J5k6h3y4Z5M7X3!0E0K9i4g2E0i4K6u0W2L8%4u0Y4i4K6u0r3j5$3S2J5L8$3#2A6N6h3#2Q4x3V1k6K6M7X3y4Q4x3V1k6K6k6i4u0$3K9h3y4W2M7#2)9J5c8X3&6W2N6s2N6G2M7X3E0Q4x3V1k6H3N6h3u0D9K9h3y4Q4x3V1k6E0L8$3A6G2L8g2)9J5c8Y4g2J5L8q4)9#2k6X3I4G2j5h3c8W2M7W2)9#2k6X3k6S2j5%4c8G2M7Y4W2Q4x3X3g2E0L8$3A6G2L8g2)9K6c8X3I4Q4x3@1b7K6x3W2)9J5y4X3q4E0M7q4)9K6b7Y4u0U0L8q4)9K6c8o6l9J5z5e0k6U0k6o6R3@1x3r3p5H3j5$3x3K6z5e0R3K6k6r3t1^5k6e0c8V1y4$3x3H3y4e0M7%4k6h3f1%4z5o6b7#2j5e0p5@1z5e0x3`.). 正如前文所述, 渲染器使用`URLLoaderFactory`来获取网络资源。它会在渲染器上强制启用同源策略(实际上是同站)以确保你无法突破网站平台的限制。 然而, 浏览器进程也会将`URLLoaderFactory`用于其他的目的, 那些目的有格外的特权。除了忽略同源策略外, 它们也允许上传本地文件。 因此, 如果我们能泄露其中某个的端口名, 我们就能利用它来将`/etc/passwd`上传到[5f2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6X3W2D9i4K6u0W2N6$3g2T1M7$3W2@1k6b7`.`.](https://bbs.kanxue.com/elink@54dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6X3W2D9i4K6u0W2N6$3g2T1M7$3W2@1k6g2)9J5c8R3`.`.).

下一步是触发对特权加载程序的端口名的重复访问。让浏览器进程发出网络请求是一种选择, 但似乎开销太大。我决定改为在节点中查找端口。

```
class COMPONENT_EXPORT(MOJO_CORE_PORTS) Node {

  // [...]

  std::unordered_map<LocalPortName, scoped_refptr<Port>> ports_;

  // [...]

}

```

每个节点都有一个哈希map用于保存所有的[本地端口](https://bbs.kanxue.com/elink@723K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0M7#2)9J5k6h3y4Z5M7X3!0E0K9i4g2E0i4K6u0W2L8%4u0Y4i4K6u0r3j5$3S2J5L8$3#2A6N6h3#2Q4x3V1k6K6M7X3y4Q4x3V1k6E0L8$3A6G2i4K6u0r3j5$3!0J5k6g2)9J5c8Y4m8G2M7Y4c8K6i4K6u0r3L8X3!0V1k6g2)9J5k6h3S2Q4x3@1k6D9i4K6y4p5x3U0V1^5i4K6t1$3j5h3#2H3i4K6y4n7M7X3y4D9i4K6y4p5x3$3k6S2y4h3g2V1k6o6u0T1j5X3q4T1j5X3x3K6x3K6j5J5y4$3c8X3k6X3b7#2k6U0m8V1x3h3x3&6z5h3j5&6k6e0p5^5k6o6p5H3z5b7`.`.). 如果我们向一个不存在的端口发送消息, 目标节点会在map中搜索, 发现它并不存在, 然后丢弃消息。如果我们的端口名和其他的端口名在同一个哈希桶中, 它会读取未知端口的[完整哈希](https://bbs.kanxue.com/elink@b50K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0M7#2)9J5k6h3y4Z5M7X3!0E0K9i4g2E0i4K6u0W2L8%4u0Y4i4K6u0r3j5$3S2J5L8$3#2A6N6h3#2Q4x3V1k6K6M7X3y4Q4x3V1k6T1N6h3W2D9k6s2c8G2L8$3I4K6i4K6u0r3N6r3S2A6M7X3c8Q4y4h3k6H3j5i4u0@1P5g2)9J5c8X3I4A6j5X3y4Q4x3U0f1J5b7W2)9J5y4e0u0n7i4K6u0r3N6s2u0#2L8X3E0Q4x3V1k6A6L8X3y4D9N6h3c8W2i4K6u0r3i4K6g2X3i4K6g2X3K9r3q4K6K9q4)9#2k6Y4c8S2j5X3I4W2i4K6y4r3L8q4)9K6c8o6t1@1y4U0g2Q4x3U0k6S2L8i4m8Q4x3@1u0J5j5$3I4Q4x3@1b7%4z5r3b7$3j5e0M7%4y4U0N6W2k6o6f1%4j5U0f1H3x3e0t1J5j5e0p5$3x3h3t1&6x3h3j5#2z5h3j5I4z5h3x3&6j5X3b7H3k6o6p5&6)并进行比较。这也将端口名自身加载进缓存里, 因为它通常和哈希一样保存在同一个缓存行中。 MFBDS能让我们泄露整个缓存行, 即使其中的值无法直接访问.

在新的Chrome实例上该map的桶大小大概为700, 其会随着渲染器的数目而增长。 这会使得攻击无法进行, 因为这样我们必须得爆破桶索引和缓存行偏移量(多亏了对齐的原因, 有四分之一)。然而, 我注意到一个代码路径可以让你利用service workers创建大量的特权`URLLoaderFactories`。如果你启用[导航预加载](https://bbs.kanxue.com/elink@f16K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0K6i4K6u0W2k6$3!0G2k6$3I4W2i4K6u0W2j5$3!0E0i4K6u0r3N6$3g2T1i4K6u0r3N6i4m8V1j5i4c8W2M7#2)9J5c8U0t1H3x3e0N6Q4x3V1j5H3x3W2)9J5c8X3&6S2N6X3W2Y4j5i4c8A6L8$3&6Q4x3X3c8H3M7X3g2D9L8$3q4V1)创建service worker, 每个顶层的导航会[创建这样一个加载程序](https://bbs.kanxue.com/elink@938K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0M7#2)9J5k6h3y4Z5M7X3!0E0K9i4g2E0i4K6u0W2L8%4u0Y4i4K6u0r3j5$3S2J5L8$3#2A6N6h3#2Q4x3V1k6K6M7X3y4Q4x3V1k6U0L8$3&6@1k6h3&6@1i4K6u0r3j5Y4u0G2N6%4y4W2M7W2)9J5c8Y4y4W2M7Y4k6A6j5$3g2Q4y4h3k6%4L8%4u0C8k6i4u0Q4x3V1k6K6k6i4u0$3K9h3y4W2i4K6g2X3N6$3!0J5K9$3g2J5i4K6g2X3k6X3g2@1j5$3S2Q4y4h3k6V1K9i4y4H3j5i4c8U0K9r3g2J5i4K6u0W2j5$3y4Q4x3@1k6D9i4K6y4p5x3K6x3@1i4K6t1$3j5h3#2H3i4K6y4n7M7X3y4D9i4K6y4p5j5e0p5J5z5e0j5I4x3r3x3J5x3r3t1J5x3X3c8V1y4K6N6X3y4U0g2X3x3e0x3%4k6o6R3^5k6X3x3K6y4$3c8V1x3h3g2T1x3o6j5@1k6R3`.`.). 通过简单地创建多个iframe并在服务器端停止请求, 你可以同时使数千个加载程序保持活动状态, 并使暴力破解变得更加容易。

唯一漏掉的事是将L1缓存中的目标值赶出去。在实践中, 简单地用32KB数据填充我们的消息似乎可以解决问题, 因为我认为数据将被加载到受害者的L1缓存中并赶出其他所有数据。

总结一下完整的利用流程:

1. 破坏渲染器.

2. 在`$NUM_CPU-1` 个进程上以不同的缓存行偏移量运行RIDL利用程序.

3. 安装带有导航预加载(navigation preload)的service worker.

4. 创建许多iframes并暂停它们的请求.

5. 向网络进程的随机端口名发送消息.

6. 如果我们能和桶索引发生碰撞, 步骤2中的进程就能泄露端口名.

7. 向`URLLoaderFactory`发送欺骗消息, 上传本地文件到[d87K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6X3W2D9i4K6u0W2N6$3g2T1M7$3W2@1k6b7`.`.](https://bbs.kanxue.com/elink@900K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6X3W2D9i4K6u0W2N6$3g2T1M7$3W2@1k6g2)9J5c8R3`.`.)

### TSX异步中止(TAA)

在2019年11月, MDS攻击发布了新的变种, 看起来TAA PoC要比我的MFBDS利用快很多。我决定把它移植到Chrome利用上。除此之外, VUSec还发布了一个[针对store指令](https://bbs.kanxue.com/elink@afaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6$3N6i4y4W2j5#2)9J5c8Y4u0A6k6r3I4Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3M7r3!0U0M7#2)9J5c8Y4c8S2j5g2)9#2k6X3u0S2M7$3W2U0i4K6u0W2j5H3`.`.)的利用, 如果我们能将秘密写到内存的不同地址, 这应该能让我们避免了缓存刷新的要求。 如果我们能触发浏览器来向特权端口发送消息, 则应该能发生这种情况。 在这种情况下, 秘密端口名也将以节点名称作为前缀, 我们可以使用RIDL论文中的技术轻松对其进行过滤.

我也开始寻找有没有更好的原语, 然后我发现如果我能和[`NetworkService`](https://bbs.kanxue.com/elink@75cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0M7#2)9J5k6h3y4Z5M7X3!0E0K9i4g2E0i4K6u0W2L8%4u0Y4i4K6u0r3j5$3S2J5L8$3#2A6N6h3#2Q4x3V1k6K6M7X3y4Q4x3V1k6K6k6i4u0$3K9h3y4W2M7#2)9J5c8X3&6W2N6s2N6G2M7X3E0Q4x3V1k6H3N6h3u0D9K9h3y4Q4x3V1k6E0L8$3A6G2L8g2)9J5c8X3&6W2N6s2N6G2M7X3E0Q4y4h3k6K6k6i4u0$3K9h3y4W2i4K6u0W2L8h3!0B7L8$3#2Q4x3@1k6D9i4K6y4p5x3e0V1%4i4K6t1$3j5h3#2H3i4K6y4n7M7X3y4D9i4K6y4p5k6o6j5^5z5r3t1^5j5$3y4V1k6h3p5&6x3K6k6S2j5X3j5$3k6o6S2X3y4o6S2W2j5e0f1^5y4o6R3^5j5K6M7@1z5o6t1%4x3o6f1$3j5R3`.`.)通信, 它会允许我创建一个新的[`NetworkContext`](https://bbs.kanxue.com/elink@54bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0M7#2)9J5k6h3y4Z5M7X3!0E0K9i4g2E0i4K6u0W2L8%4u0Y4i4K6u0r3j5$3S2J5L8$3#2A6N6h3#2Q4x3V1k6K6M7X3y4Q4x3V1k6K6k6i4u0$3K9h3y4W2M7#2)9J5c8X3&6W2N6s2N6G2M7X3E0Q4x3V1k6H3N6h3u0D9K9h3y4Q4x3V1k6E0L8$3A6G2L8g2)9J5c8X3&6W2N6s2N6G2M7X3E0Q4y4h3k6U0L8$3&6@1k6i4S2@1i4K6u0W2L8h3!0B7L8$3#2Q4x3@1k6D9i4K6y4p5z5o6l9%4i4K6t1$3j5h3#2H3i4K6y4n7M7X3y4D9i4K6y4p5k6o6j5^5z5r3t1^5j5$3y4V1k6h3p5&6x3K6k6S2j5X3j5$3k6o6S2X3y4o6S2W2j5e0f1^5y4o6R3^5j5K6M7@1z5o6t1%4x3o6f1$3j5R3`.`.), 从而选择保存了cookies的sqlite3数据库的[文件路径](https://bbs.kanxue.com/elink@423K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0M7#2)9J5k6h3y4Z5M7X3!0E0K9i4g2E0i4K6u0W2L8%4u0Y4i4K6u0r3j5$3S2J5L8$3#2A6N6h3#2Q4x3V1k6K6M7X3y4Q4x3V1k6K6k6i4u0$3K9h3y4W2M7#2)9J5c8X3&6W2N6s2N6G2M7X3E0Q4x3V1k6H3N6h3u0D9K9h3y4Q4x3V1k6E0L8$3A6G2L8g2)9J5c8X3&6W2N6s2N6G2M7X3E0Q4y4h3k6U0L8$3&6@1k6i4S2@1i4K6u0W2L8h3!0B7L8$3#2Q4x3@1k6D9i4K6y4p5x3U0x3K6i4K6t1$3j5h3#2H3i4K6y4n7M7X3y4D9i4K6y4p5k6o6j5^5z5r3t1^5j5$3y4V1k6h3p5&6x3K6k6S2j5X3j5$3k6o6S2X3y4o6S2W2j5e0f1^5y4o6R3^5j5K6M7@1z5o6t1%4x3o6f1$3j5R3`.`.).
