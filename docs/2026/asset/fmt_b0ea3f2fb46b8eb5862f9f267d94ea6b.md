

```c
if (WIFEXITED(tracee_status)) {
    // WEXITSTATUS取得子进程exit（）返回的结束代码
    last_exit_status = WEXITSTATUS(tracee_status);
    // ALOGI("normal exit -> [%d] exit with status: %d", tracee->pid, tracee_status);
    // 被跟踪者进程正常执行结束，释放当前（被跟踪者）
    terminate_tracee(tracee);
} else if (WIFSIGNALED(tracee_status)) {
    int signNum = WTERMSIG(tracee_status);
    // 被跟踪进程因为信号退出
    ALOGE("[%d] process exit with signal: 终止信号 = %d 异常原因 = %s",
          tracee->pid, signNum, strsignal(signNum));
    terminate_tracee(tracee);
} else if (WIFSTOPPED(tracee_status)) {
    signal = (tracee_status & 0xfff00) >> 8;
    switch (signal) {
        // svc
        case SIGTRAP | 0x80:
            // 被调试线程调用svc，开始处理参数和返回值
            ...
    }
}
```

当参数或者返回值处理完毕以后，通过给**调试线程**调用`ptrace`，设置**被调试线程**的启动`PTRACE_SYSCALL`事件。

（当被调试进程执行了某些`SIGTRAP`事件，程序就会进入暂停。这个时候**调试线程**开始处理对应的逻辑。

常用的恢复暂停事件有两个，`PTRACE_SYSCALL`和`PTRACE_CONT`，可以理解成一个是调试的单步执行，一个是继续执行。

通过`PTRACE_SYSCALL`方式重新启动**被调试线程**以后，下次遇到`SVC`的`before`和`after`还会继续暂停。）

也就是当`svc`**执行之前（before）**和**执行之后（after）**，被调试线程都会暂停。

先把每个版本不同的寄存器进行匹配，用来区分`LR`、`SP`、`PC`等常用寄存器。

```c
#elif defined(ARCH_ARM_EABI)
static off_t reg_offset[] = {
    [SYSARG_NUM] = USER_REGS_OFFSET(uregs[7]),
    [SYSARG_1] = USER_REGS_OFFSET(uregs[0]),
    [SYSARG_2] = USER_REGS_OFFSET(uregs[1]),
    [SYSARG_3] = USER_REGS_OFFSET(uregs[2]),
    [SYSARG_4] = USER_REGS_OFFSET(uregs[3]),
    [SYSARG_5] = USER_REGS_OFFSET(uregs[4]),
    [SYSARG_6] = USER_REGS_OFFSET(uregs[5]),
    [SYSARG_RESULT] = USER_REGS_OFFSET(uregs[0]),
    [FRAME_POINTER] = USER_REGS_OFFSET(uregs[12]),
    [STACK_POINTER] = USER_REGS_OFFSET(uregs[13]),
    [LINK_REGISTER] = USER_REGS_OFFSET(uregs[14]),
    [INSTR_POINTER] = USER_REGS_OFFSET(uregs[15]),
    [USERARG_1] = USER_REGS_OFFSET(uregs[0]),
};

#elif defined(ARCH_ARM64)
#undef USER_REGS_OFFSET
#define USER_REGS_OFFSET(reg_name) offsetof(struct user_regs_struct, reg_name)

static off_t reg_offset[] = {
    [SYSARG_NUM] = USER_REGS_OFFSET(regs[8]),
    [SYSARG_1] = USER_REGS_OFFSET(regs[0]),
    [SYSARG_2] = USER_REGS_OFFSET(regs[1]),
    [SYSARG_3] = USER_REGS_OFFSET(regs[2]),
    [SYSARG_4] = USER_REGS_OFFSET(regs[3]),
    [SYSARG_5] = USER_REGS_OFFSET(regs[4]),
    [SYSARG_6] = USER_REGS_OFFSET(regs[5]),
    [SYSARG_RESULT] = USER_REGS_OFFSET(regs[0]),
    // https://zhuanlan.zhihu.com/p/42486116
    // http://blog.chinaunix.net/uid-25564582-id-5852920.html
    [FRAME_POINTER] = USER_REGS_OFFSET(regs[29]),
    // 64位30是LR寄存器
    [LINK_REGISTER] = USER_REGS_OFFSET(regs[30]),
    [STACK_POINTER] = USER_REGS_OFFSET(sp),
    [INSTR_POINTER] = USER_REGS_OFFSET(pc),
    [USERARG_1] = USER_REGS_OFFSET(regs[0]),
};
```

这个时候我们可以直接去获取寄存器内容，判断路径是否是我们需要修改的文件路径。

```c
/**
 * Return the cached value of the given @Tracers' @reg.
 * 返回给定@Tracers@reg的缓存值
 */
word_t peek_reg(const Tracer *Tracer, RegVersion version, Reg reg) {
    word_t result;

    assert(version < NB_REG_VERSION);

    result = REG(Tracer, version, reg);

    /*
     * Use only the 32 least significant bits (LSB) when running
     * 32-bit processes on a 64-bit kernel.
     */
    if (is_32on64_mode(Tracer))
        result &= 0xFFFFFFFF;

    return result;
}

/**
 * Set the cached value of the given @Tracers' @reg.
 *
 * 修改寄存器的内容方法，value标识的是指针
 */
void poke_reg(Tracer *Tracer, Reg reg, word_t value) {
    // 设置之前先判断是否相等
    if (peek_reg(Tracer, CURRENT, reg) == value)
        // 相等直接返回
        return;

    REG(Tracer, CURRENT, reg) = value;
    // 标识它已经被修改
    Tracer->_regs_were_changed = true;
}
```

将修改完毕的寄存器内容保存到数组里面，最后通过`ptrace`的`PTRACE_SETREGSET` action进行寄存器的设置。

```c
regs.iov_base = &current_sysnum;
regs.iov_len = sizeof(current_sysnum);

status = ptrace(PTRACE_SETREGSET, Tracer->pid, NT_ARM_SYSTEM_CALL, &regs);
if (status < 0) {
    // note(Tracer, WARNING, SYSTEM, "can't set the syscall number");
    return status;
}
```

修改被调试进程的寄存器内容，已达到修改`svc`参数和返回结果的目的。

这个思路确实可以实现`svc`参数和返回值的修改，但是**存在问题**。

**效率太低，调试线程和被调试线程本身是两条线程，主要通过线程间交互进行传递消息，而且被`attach`的进程会进行大量的暂停，甚至本身的`libc`去调用`svc`的时候也会进行暂停，导致程序卡顿。**

当时为了解决这个问题也花了很久，查了很多资料。

### Seccomp + ptrace

为了解决这个效率低的问题，看了很多开源框架，比如`Strace`也在使用`ptrace`进行`svc`的跟踪。

> 一个打印`Syscall`调用方法的插件，可以很清楚地打印全部的系统调用，比如文件相关类型函数、网络相关类型的函数等。
>
> 它同样也可以用在安卓上面，具体使用方式，国内资料比较多，可以去看一下。

`Strace`是怎么解决的？用的就是`Seccomp + ptrace`去做拦截，我们只需要关注我们需要进行拦截的函数即可。

比如常见的`IO`函数，`access`、`openat`、`open`、`fstat`等，而不需要关注别的系统调用。

`seccomp`初始化完毕以后，我们只需要在`ptrace PTRACE_SETOPTIONS`的时候加上`PTRACE_O_TRACESECCOMP`参数即可。

```c
const unsigned long default_ptrace_options = (
    PTRACE_O_TRACESYSGOOD |
    PTRACE_O_TRACEFORK |
    PTRACE_O_TRACEVFORK |
    PTRACE_O_TRACEVFORKDONE |
    PTRACE_O_TRACEEXEC |
    PTRACE_O_TRACECLONE |
    PTRACE_O_TRACEEXIT
);

// 尝试开启ptrace + seccomp
status = ptrace(
    PTRACE_SETOPTIONS,
    tracee->pid,
    NULL,
    default_ptrace_options | PTRACE_O_TRACESECCOMP
);
```

这样一来，当目标`App`调用了被我们拦截的系统调用时，就会走如下`case`：

```c
case SIGTRAP | PTRACE_EVENT_SECCOMP << 8:
```

我们直接在这个执行上面的流程设置参数，也是没问题的。但是这个时候又出现了一个问题：

**`Seccomp`只能处理`svc`的`before`，也就是当`svc`执行之前进入到这个`case`，不能处理`after`。**

因为修改`svc`的返回结果必须在`after`里面处理。有人可能会问了，为什么要处理返回结果呢？文件重定向只需要处理参数就行了。

#### 参数完全可以在 before 里面进行处理，为啥还要处理 after 呢？

答：做指纹 mock 时需要在`after`里面处理，比如`socket`常见的通信函数：`recv`、`recvfrom`、`recvmsg`。

它们都是在原始函数调用完毕以后把数据参数放到一个数组里面。如果在`before`处理这个数组，肯定是`NULL`。

只有在函数执行完毕以后，才会将数组的内容进行赋值。比如我之前讲的通过`netlinker`去获取设备指纹。

https://bbs.pediy.com/thread-271698.htm

这也是大厂的贯通套路，这种指纹想要去 mock 很难，特别是用内联`svc`的方式去获取。但是用了`ptrace`想去修改的话就很简单了。代码如下，先通过`peek_reg`寄存器把数据读到手，然后在把数据处理完毕以后通过`poke_reg`写回去。

```cpp
void NetlinkMacHandler::netlinkHandler_recv(Tracer *tracee) {
    ssize_t bytes_read =
        TEMP_FAILURE_RETRY(peek_reg(tracee, CURRENT, SYSARG_RESULT));

    if (bytes_read > 0) {
        word_t buff = peek_reg(tracee, CURRENT, SYSARG_2);
        // buff长度
        auto size = (size_t)peek_reg(tracee, CURRENT, SYSARG_3);
        char tempBuff[size];

        int readStr_ret = read_data(tracee, tempBuff, buff, size);
        if (readStr_ret != 0) {
            LOGE("svc netlink handler read_string error %s", strerror(errno));
            return;
        }

        auto *hdr = reinterpret_cast<nlmsghdr *>(tempBuff);
        // netlink数据包结构体
        NetlinkMacHandler::handler_mac_callback_svc(tracee, hdr, bytes_read);

        // 将数据写入覆盖掉原来的数据
        write_data(tracee, buff, tempBuff, size);
    }
}
```

为了解决`ptrace + seccomp`不能处理`after`的问题，想了好久，卡了我几个月之久。后来通过问 VA 作者发现一个很不错的开源项目，就是`proot`。

项目地址：`7f2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6H3M7X3!0G2N6q4)9J5k6r3#2W2i4K6u0r3M7s2u0G2L8%4b7`。

`proot`的解决方案也很简单，只需要一行即可。果然天才需要的是灵感。

```c
poke_reg(tracee, STACK_POINTER, peek_reg(tracee, ORIGINAL, STACK_POINTER));
```

**修改`SP`寄存器，让这个方法二次进入，一次修改参数，一次修改返回结果即可。**

简单介绍一下`proot`这个项目，它完全符合我们的需求，处理逻辑也类似。

在 Linux 里面有一个`chroot`函数，这个函数可以修改`root`用户根目录的位置，但是这个函数需要`root`权限才可以用。

比如我想把`/data/zhenxi/`路径变成 Linux 的根目录，而不是最原始的`/`，就可以使用这个`proot`。编译好以后，直接启动就可以在免`root`的环境下进行限制和使用。它的原理也是使用`ptrace + seccomp`进行限制。

对执行的文件路径进行替换和处理。**我上面发的代码也都是来自`proot`。**

