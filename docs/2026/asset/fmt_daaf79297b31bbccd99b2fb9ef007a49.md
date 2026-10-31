
因此我们无法调用两次`kernel_open`后利用`kernel_release`的指针悬挂，来造成0x100 size的一个object的UAF。

但是由于`kernel_read`和`kernel_write`，`kernel_open`与`kernel_release`均未加锁，且`read`与`write`中含有类似于如下的`copy_to_user`操作：

```
if ( copy_to_user(a2, v4, v5) )
  return -2LL;
```

因此我们可以考虑使用`userfaultfd`卡住当前进程，在另外一个线程中调用`kernel_release`。这样同样可以达到一个0x100 size的UAF。

所以笔者是考虑用这个简单的洞来泄露内核基址，用的是0x100 size对应的`timerfd_ctx`结构体。

不过笔者用这种方法无论如何都无法泄露出内核基址，后来咨询`arttnba3`师傅后得知：

**内核调用`fput`对文件描述符进行释放，对于文件描述符的关闭会被delay，直到我们读取数据后才会关闭**。

![](https://bbs.kanxue.com/upload/attach/202402/959503_ZRMUQYUR4ME6MGB.jpg)

因此靠这种操作leak内核数据是不可行的。

### ②漏洞点

程序除了`module`自身的`open`，`release`，`read`，`write`操作。

提供了简单的菜单堆功能：

- add：两次add机会，size为0x400。

- delete：两次delete。

- edit：两次edit。

所有功能都未加锁，因此可以用`userfaultfd`在edit时将进程卡死，在另一个线程中free掉这个堆块后申请某些object到该地址上，实现0x400 size object的UAF（只能更改一次值）。

## 具体步骤

### ①leak内核基址

申请一个消息队列，上面只放一条消息，且size为0x400。同时申请一个`pipe_buffer`。

改大`msg_msg->m_ts`，用带有`MSG_COPY`位的`msgrcv`进行越界读，泄露出`pipe_buffer`上的函数表。

得到**内核基址**。

### ②提权 or 后续利用

leak内核地址分别消耗了一次add,delete,edit操作。因此我们还有一次UAF的机会。

笔者最先考虑的是用`pipe_buffer`提权，刚好满足0x400的size，因此我们利用UAF将`pipe_buffer->pipe_buf_operations->release`函数指针更改为某个栈迁移gadget即可。

但是我并没有找到可利用的gadget，其中有一条可能能达成的：

```
push_rsi_pop_rsp = 0xffffffff81934056; // push rsi; pop rsp; retf;
```

不过`retf`是按32位pop `eip`和`cs`，的，而32位根本不足以储存一个内核地址。

同时还有例如`mov esp,esi`类型的`gadget`，由于`intel x86&x64`的调用约定，当对32位寄存器进行赋值操作的时候，会将高32位寄存器值清零，因此也不可用。

同时注意到程序没有开启`CONFIG_STATIC_USERMODEHELPER`保护，因此笔者选择用`UAF`劫持0x400 size的`freelist`到`modprobe_path`附近，更改`modprobe_path`。

笔者是申请`msg_msg`结构体申请到`modprobe_path`附近，由于size太大，会将`modprobe_path`附近的所有内容全部清空。

直接进行接下来的提权or 读取flag操作会在成功前引起`kernel panic`，因此我们需要恢复`modprobe_path`附近的函数指针。

其中，`kmod`的函数指针恢复是必要的：

![](https://bbs.kanxue.com/upload/attach/202402/959503_8ENV57QFC2CGENB.jpg)

**因为`modprobe_path`是一个Linux程序，最初由Rusty Russell编写，用于在Linux内核中添加一个可加载的内核模块，或者从内核中移除一个可加载的内核模块，因此`modprobe`是安装某个内核模块，而`kmod`是一个用于控制linux内核模块的程序，因此在后续调用中需要用到**

最后直接利用`modprobe_path_hijack`更改flag权限后读取即可。

![](https://bbs.kanxue.com/upload/attach/202402/959503_RHKZXWFWCEMG6AN.jpg)

## poc
