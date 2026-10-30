
# [原创]Linux内核[CVE-2017-5123] (waitid)原理分析

`CVE-2017-5123`是Linux内核中`waitid`有关的一个漏洞。

[TOC]

## waitid demo

类似于`wait`和`waitpid`。

```c
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <sys/wait.h>
#include <string.h>
int main(){
siginfo_t info;
id_t id;
int ret_val;
pid_t pid = fork();
if(pid < 0){
perror("fork failed");
exit(-1);
}
else if(pid == 0){
// in child process
printf("This is child process!\n");
exit(8);
}
else{
// in father process
memset(&info, 0 , sizeof(siginfo_t));
//P_ALL代表等待任何一个进程
//退出信息在info结构体中
//Wait for children that have terminated.
ret_val = waitid(P_ALL, id , &info , WEXITED);
if(ret_val < 0){
perror("waitid failed");exit(-2);
}
if(info.si_code == CLD_EXITED)
{
printf("si_code: _exit\n");
}
printf("si_status = %d\n" , info.si_status);
}
return 0;
}
```

```c
typedef struct siginfo {
int si_signo;
int si_errno;
int si_code;
union {
int _pad[SI_PAD_SIZE];
/* kill() */
struct {
__kernel_pid_t _pid; /* sender's pid */
__ARCH_SI_UID_T _uid; /* sender's uid */
} _kill;
/* POSIX.1b timers */
struct {
__kernel_timer_t _tid; /* timer id */
int _overrun; /* overrun count */
char _pad[sizeof( __ARCH_SI_UID_T) - sizeof(int)];
sigval_t _sigval; /* same as below */
int _sys_private; /* not to be passed to user */
} _timer;
/* POSIX.1b signals */
struct {
__kernel_pid_t _pid; /* sender's pid */
__ARCH_SI_UID_T _uid; /* sender's uid */
sigval_t _sigval;
} _rt;
/* SIGCHLD */
struct {
__kernel_pid_t _pid; /* which child */
__ARCH_SI_UID_T _uid; /* sender's uid */
int _status; /* exit code */
__ARCH_SI_CLOCK_T _utime;
__ARCH_SI_CLOCK_T _stime;
} _sigchld;
/* SIGILL, SIGFPE, SIGSEGV, SIGBUS */
struct {
void __user *_addr; /* faulting insn/memory ref. */
#ifdef __ARCH_SI_TRAPNO
int _trapno; /* TRAP # which caused the signal */
#endif
short _addr_lsb; /* LSB of the reported address */
union {
/* used when si_code=SEGV_BNDERR */
struct {
void __user *_lower;
void __user *_upper;
} _addr_bnd;
/* used when si_code=SEGV_PKUERR */
__u32 _pkey;
};
} _sigfault;
/* SIGPOLL */
struct {
__ARCH_SI_BAND_T _band; /* POLL_IN, POLL_OUT, POLL_MSG */
int _fd;
} _sigpoll;
/* SIGSYS */
struct {
void __user *_call_addr; /* calling user insn */
int _syscall; /* triggering system call number */
unsigned int _arch; /* AUDIT_ARCH_* of syscall */
} _sigsys;
} _sifields;
} __ARCH_SI_ATTRIBUTES siginfo_t;
```

## 漏洞源码

### waitid

<mark class="encrypted">c08K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8r3W2^5K9i4u0Q4x3X3g2T1L8$3!0@1L8r3W2F1i4K6u0W2j5$3!0E0i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6$3y4q4)9J5k6e0p5K6i4K6u0r3M7$3!0#2M7X3y4W2i4K6u0r3K9$3g2J5L8X3g2D9i4K6u0r3k6i4S2A6N6q4)9J5k6h3x3`.</mark>

```c
SYSCALL_DEFINE5(waitid, int, which, pid_t, upid, struct siginfo __user *,
infop, int, options, struct rusage __user *, ru)
{
struct rusage r;
struct waitid_info info = {.status = 0};
long err = kernel_waitid(which, upid, &info, options, ru ? &r : NULL);
int signo = 0;
if (err > 0) {
signo = SIGCHLD;
err = 0;
}
if (!err) {
if (ru && copy_to_user(ru, &r, sizeof(struct rusage)))
return -EFAULT;
}
if (!infop)
return err;
user_access_begin(); //暂时关闭smap
unsafe_put_user(signo, &infop->si_signo, Efault);
unsafe_put_user(0, &infop->si_errno, Efault);
unsafe_put_user((short)info.cause, &infop->si_code, Efault);
unsafe_put_user(info.pid, &infop->si_pid, Efault);
unsafe_put_user(info.uid, &infop->si_uid, Efault);
unsafe_put_user(info.status, &infop->si_status, Efault);
user_access_end(); //重新开启smap
return err;
Efault:
user_access_end();
return -EFAULT;
}
```

看到这里首先调用`user_access_begin()`

```c
#define user_access_begin() __uaccess_begin()

#define user_access_end() __uaccess_end()

#define __uaccess_begin() stac()

#define __uaccess_end() clac()
```

实际上是调用了`stac()`还有`clac()`

```c
#ifdef CONFIG_X86_SMAP //是否开始smap?
static __always_inline void clac(void)
{
/* Note: a barrier is implicit in alternative() */
alternative("", __stringify(__ASM_CLAC), X86_FEATURE_SMAP);
}
static __always_inline void stac(void)
{
/* Note: a barrier is implicit in alternative() */
alternative("", __stringify(__ASM_STAC), X86_FEATURE_SMAP);
}
//其中 __ASM_CLAC 与 __ASM_STAC
/* "Raw" instruction opcodes */
#define __ASM_CLAC .byte 0x0f,0x01,0xca
#define __ASM_STAC .byte 0x0f,0x01,0xcb
```

简单说一下这个`CLAC`和`STAC`，由于众所周知的`SMAP`，内核不可以随意去读写用户空间的数据（这也导致了内核对于用户空间访问的低效）。但是当 **内核确实有正常的需求需要访问用户空间并进行读写时**，我们需要暂时**禁用SMAP**。

*However, the use of **SMAP** in an operating system may **lead to** a **larger kernel size and slower user-space memory accesses** from supervisor code, because **SMAP must be temporarily disabled** any time supervisor code intends to access user-space memory.*

而谁来disable SMAP呢？这就需要：`Extended Features CPUID leaf` 也就是 <a href="elink@34dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6r3e0p5q4s2f1#2)9#2k6Y4u0W2k6$3W2K6N6r3g2J5i4K6t1K6c8X3I4S2k6%4x3`.">EFLAGS</a>

**下面给出过程：**

1. 当 <a href="elink@b97K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6r3e0p5q4s2f1#2)9#2k6Y4u0W2k6$3W2K6N6r3g2J5i4K6t1K6c8X3I4S2k6%4x3`.">EFLAGS.AC</a> 被`stac`设置后，就可以暂时性的disable掉SMAP，然后内核便可以对用户数据进行读写。

2. 内核读写完毕用户数据，此时我们要恢复SMAP，于是调用`clac`清除`EFLAGS.AC`恢复SMAP。整个过程便完成了。

接下来调用了`unsafe_put_user`

### unsafe_put_user

```asm
/*
 * The "unsafe" user accesses aren't really "unsafe", but the naming
 * is a big fat warning: you have to not only do the access_ok()
 * checking before using them, but you have to surround them with the
 * user_access_begin/end() pair.
 */
#define user_access_begin() __uaccess_begin()
#define user_access_end() __uaccess_end()
#define unsafe_put_user(x, ptr, err_label) \
do { \
    int __pu_err; \
    __typeof__(*(ptr)) __pu_val = (x); \
    __put_user_size(__pu_val, (ptr), sizeof(*(ptr)), __pu_err, -EFAULT); \
    if (unlikely(__pu_err)) goto err_label; \
} while (0)
```

这里首先提醒我们，要配合`user_access_begin/end()`来使用`unsafe_put_user`

并且，要在这之前调用`access_ok()`检查！！！但是这里并没有`access_ok()`

### 缺失的access_ok()

```c
/**
 * access_ok: - Checks if a user space pointer is valid
 * @type: Type of access: %VERIFY_READ or %VERIFY_WRITE. Note that
 * %VERIFY_WRITE is a superset of %VERIFY_READ - if it is safe
 * to write to a block, it is always safe to read from it.
 * @addr: User space pointer to start of block to check
 * @size: Size of block to check
 *
 * Context: User context only. This function may sleep if pagefaults are
 * enabled.
 *
 * Checks if a pointer to a block of memory in user space is valid.
 *
 * Returns true (nonzero) if the memory block may be valid, false (zero)
 * if it is definitely invalid.
 *
 * Note that, depending on architecture, this function probably just
 * checks that the pointer is in the user space range - after calling
 * this function, memory access functions may still return -EFAULT.
 */
#define access_ok(type, addr, size) \
({ \
WARN_ON_IN_IRQ(); \
likely(!__range_not_ok(addr, size, user_addr_max())); \
})
```

这里做了如下事情：

1. `user_addr_max()`获得了`current->thread.addr_limit.seg`作为用户态地址的边界。
2. `__chk_user_ptr`检查我们的参数`addr`是否是指向**用户态**的

3. `__range_not_ok`检查`addr + size`和`limit`的大小关系，即`addr + size`是否也指向**用户态**

我们再找一个x86下对于`access_ok`的引用：
