
因为这个`iovec_array`是堆喷的数据，它其实相当于`binder_thread.x2`的内容，已经由`0xBEEFDEAD`修改成`current_prt + 0x8`了。这时候`second_chunk`只剩下最后一个值`0xfffffffffffffffe`，然后继续执行`write(fd, current_prt + 0x8, 0xfffffffffffffffe)`，达到 patch `addr_limit` 的目的，从而实现任意内核写，拿到`root`。

这里需要注意的是：

```c
int recvmsg_result = recvmsg(socks[0], &msg, MSG_WAITALL);
```

`MSG_WAITALL`的标志起到了等待`write`调用完成的作用，也就是一直会等待下去。

对应 Android 9、Android 10 都运行拿到了`root`，以下为 Android 10 的运行日志。

![](https://bbs.kanxue.com/upload/attach/202101/726411_PXPVMNC35SD8C57.png)

# 有些手机会崩溃

1. 崩溃地方在`spin_lock_irqsave(&q->lock, flags)`。

   ![](https://bbs.kanxue.com/upload/attach/202101/726411_4TD4WB5JBPQ9QNH.png)

2. `readv`和`write`调用，会将所有的内容写入`dummy_page_4g_aligned`中，可能内核`read`和`write`实现的机制不同，但这部分还未分析。

# 其他知识点

## `printk`不可见的原因

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 11 12 13 | `diff --git a/lib/vsprintf.c b/lib/vsprintf.c`<br>`index 0a51559..279d5ff 100644`<br>`--- a/lib/vsprintf.c`<br>`+++ b/lib/vsprintf.c`<br>`@@ -1514,7 +1514,7 @@ char *pointer(const char *fmt, char *buf, char *end, void`<br>`case 3:`<br>`/* restrict all non-extensioned %p and %pK */`<br>`case 4:`<br>`/* restrict all non-extensioned %p, %pK, %pa*, %p[rR] */`<br>`default:`<br>`- ptr = NULL;`<br>`+ // ptr = NULL;`<br>`break;`<br>`}`<br>`break;` |

或者：

```bash
echo 0 > /prcoc/sys/kernel/kptr_restrict
```

```bash
dmsg
cat /proc/kmsg
cat /dev/kmsg
```

## 日志的插入位置

分别在`add_wait_queue`、`remove_wait_queue`和`binder_free_thread`函数插入前后的`log`，并以进程名字为过滤。指针有可能会被其他的值覆盖，所以最好不要用`%p`，否则内核会崩溃在自己写的`log`上。

# 总结

此次漏洞由`syzcaller`产生，主要在于设备实现了`binder_poll`函数。`binder_poll`函数内部使用了`binder_thread`结构成员，但未考虑`binder_thread`结构如果已经释放的情况下，`epoll`机制仍然使用其中的成员，导致`UAF`。

其`patch`在释放`binder_thread`结构前，会对`epoll`上的链表进行清理。从漏洞利用特点来看，这是`TOCTTOU`的升级利用，衍生出了某些条件下可以遇到`UAF`或者`heap overflow`这类漏洞，实现信息泄露和绕过`KASLR`的有效机制。

于 2019 年 11 月记录，分享出来与大家一起学习。

# 适配情况

A use-after-free in binder.c allows an elevation of privilege from an application to the Linux Kernel. Aimed at kernel 4.4

```text
991K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6C8j5h3&6Y4N6r3q4K6N6r3W2U0i4K6u0r3j5%4k6W2i4K6u0V1x3U0l9I4z5g2)9J5k6o6t1J5x3e0g2Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3j5%4k6W2i4K6u0V1x3U0l9I4z5g2)9J5k6o6t1J5x3e0g2Q4x3X3g2U0
```

A use-after-free in binder.c allows an elevation of privilege from an application to the Linux Kernel. Use one way and aimed at kernel 4.4

```text
2c2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6C8j5h3&6Y4N6r3q4K6N6r3W2U0i4K6u0r3j5%4k6W2i4K6u0V1x3U0l9I4z5g2)9J5k6o6t1J5x3e0g2Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3j5%4k6W2i4K6u0V1x3U0l9I4z5g2)9J5k6o6t1J5x3e0g2Q4x3X3g2U0
```

A use-after-free in binder.c allows an elevation of privilege from an application to the Linux Kernel. Use another way to root and aimed at kernel 3.18 and kernel 4

```text
d98K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5i4y4@1k6h3u0A6L8W2)9J5k6h3y4G2L8g2)9J5c8X3#2p5j5f1N6y4e0e0k6w2
```

# Patch

![](https://bbs.kanxue.com/upload/attach/202101/726411_D5CCME994TNZRK3.png)

增加了：

![](https://bbs.kanxue.com/upload/attach/202101/726411_U82KBEEYUAYVU9Y.png)

在`free binder_thread`的时候会对`wait_queue_head`进行处理，置`0`。

在`ep_poll_callback`中：

![](https://bbs.kanxue.com/upload/attach/202101/726411_N7M2RDA2TEV7952.png)

![](https://bbs.kanxue.com/upload/attach/202101/726411_NWUUAJAUTTGAM6T.png)

# References

## `crash`

```text
584K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3N6K6i4K6u0W2j5$3S2J5L8$3#2A6N6h3#2Q4x3X3g2G2M7X3N6Q4x3V1k6H3i4K6u0r3M7s2u0G2K9X3g2U0N6q4)9J5k6s2A6W2M7X3!0Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3V1k6S2N6s2c8S2j5$3S2E0k6h3&6@1g2r3g2^5N6q4)9K6c8X3q4A6k6q4)9K6c8o6b7I4y4o6l9J5z5l9`.
```

## `issue`

```text
3c6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3N6K6i4K6u0W2j5$3S2J5L8$3#2A6N6h3#2Q4x3X3g2G2M7X3N6Q4x3V1k6H3i4K6u0r3M7s2u0G2K9X3g2U0N6q4)9J5k6s2A6W2M7X3!0Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3V1k6V1k6i4c8S2K9h3I4Q4x3@1k6A6k6q4)9K6c8o6p5&6y4o6t1`.
```

## `poc`

```text
a4aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3N6K6i4K6u0W2j5$3S2J5L8$3#2A6N6h3#2Q4x3X3g2G2M7X3N6Q4x3V1k6H3i4K6u0r3M7s2u0G2K9X3g2U0N6q4)9J5k6s2A6W2M7X3!0Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3V1k6S2N6s2c8S2j5$3S2E0k6h3&6@1g2r3g2^5N6q4)9K6c8X3q4A6k6q4)9K6c8o6b7I4y4o6l9K6x3l9`.
```

```text
b6bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3N6K6i4K6u0W2j5$3S2J5L8$3#2A6N6h3#2Q4x3X3g2G2M7X3N6Q4x3V1k6H3i4K6u0r3M7s2u0G2K9X3g2U0N6q4)9J5k6s2A6W2M7X3!0Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3V1k6S2N6s2c8S2j5$3S2E0k6h3&6@1g2r3g2^5N6q4)9K6c8X3q4A6k6q4)9K6c8o6R3^5y4b7`.
```

## `patch`

```text
72dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8r3W2^5K9i4u0Q4x3X3g2T1L8$3!0@1L8r3W2F1i4K6u0W2j5$3!0E0i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6D9j5i4c8W2M7%4c8Q4x3V1k6A6k6r3g2F1N6q4)9J5c8W2m8a6e0p5I4r3f1V1g2q4
```

```text
bdfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3y4K6k6h3y4Q4x3X3g2B7M7q4)9J5c8Y4m8K6K9U0p5%4i4K6u0r3f1q4y4v1x3U0l9I4y4#2)9#2k6V1c8A6f1$3S2W2L8W2)9#2k6W2m8S2j5%4y4W2j5#2)9#2k6V1k6u0e0V1q4x3i4K6u0W2M7r3c8X3
```

```text
5d5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2P5s2c8W2M7X3&6S2L8r3W2K6N6q4)9J5c8X3g2^5M7r3I4G2K9i4c8Q4y4h3k6H3L8r3q4&6k6%4u0G2N6h3&6V1i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8V1y4h3c8g2)9J5k6o6t1H3x3e0k6Q4x3X3b7J5y4o6x3@1i4K6u0r3k6i4S2H3L8r3!0A6N6q4)9#2k6V1y4h3c8g2)9J5k6o6t1H3x3e0k6Q4x3X3b7J5y4o6x3@1i4K6g2X3j5$3!0E0L8h3g2F1N6r3g2V1i4K6u0W2j5H3`.
```

```text
713K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2Y4L8$3!0Y4L8r3g2Q4x3X3g2U0L8$3#2Q4x3X3g2Z5K9#2)9J5c8Y4g2J5L8q4)9K6c8Y4y4S2i4K6y4p5N6q4)9J5y4X3q4E0M7q4)9K6b7Y4u0U0N6q4)9K6c8r3A6Q4x3U0k6S2L8i4m8Q4x3@1u0I4i4K6y4p5i4K6t1$3j5h3#2H3i4K6y4n7k6i4y4J5j5#2)9K6c8s2y4Q4x3U0k6S2L8i4m8Q4x3@1u0K6L8%4g2J5j5$3g2Q4x3@1c8%4k6h3u0Q4x3U0k6S2L8i4m8Q4x3@1u0U0k6q4)9K6c8o6g2Q4x3U0k6S2L8i4m8Q4x3@1u0$3k6h3c8Q4x3@1b7J5j5h3S2g2d9@1g2%4K9i4b7K6k6X3t1J5P5V1I4t1L8p5q4Z5g2@1&6r3y4W2W2w2d9q4S2Q4y4h3k6p5b7K6q4g2f1f1k6B7b7f1g2W2k6#2q4u0b7h3S2m8b7W2)9J5y4X3q4E0M7q4)9K6b7Y4g2J5L8q4)9K6c8r3S2@1N6s2m8K6i4K6t1#2x3@1q4Q4x3U0f1J5c8W2)9J5y4e0u0r3M7$3g2U0N6i4u0A6N6s2W2S2k6X3k6S2K9i4u0K6i4K6u0W2j5$3!0Q4x3U0f1J5c8Y4N6G2M7X3c8H3M7X3g2K6M7#2)9J5y4e0u0r3z5e0t1$3x3K6y4Q4x3U0f1J5c8X3S2S2j5$3E0A6L8X3N6Q4x3U0f1J5c8X3y4$3k6g2)9J5k6o6t1H3x3e0W2Q4x3X3b7J5x3U0p5#2i4K6u0V1P5X3g2J5L8#2)9J5k6r3c8S2P5g2)9J5k6r3g2^5M7r3I4G2K9i4c8Q4x3X3g2Z5N6r3#2D9i4K6t1$3j5h3#2H3i4K6y4n7N6i4y4Y4i4K6y4p5b7f1!0$3g2X3q4%4x3V1W2@1K9@1j5%4L8X3N6%4c8$3V1^5P5U0k6e0k6V1&6@1d9r3Z5K6P5l9`.
```

## `epoll`的简单描述

```text
8f7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0M7s2m8X3j5h3&6K6i4K6u0W2L8%4u0Y4i4K6u0r3x3e0b7I4z5q4)9J5k6h3S2@1L8h3H3`.
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
