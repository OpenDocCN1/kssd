
进入 `/path/to/xxx` 目录，找到 `xxx.json`。

在表达式 `cvar = avar + bvar` 中，加法运算符（`+`）将 `avar` 与 `bvar` 相加，得到它们的和 `cvar`

之后我们这样调用 `cmd` 命令：`cmd arg0 arg1 arg2`。

```
if (condVar > someVal) {console.log("xxx")}
```

```
$ strace echo evilpan
execve("/usr/bin/echo", ["echo","evilpan"], 0x7fe55d5d18 /* 56 vars */) = 0
brk(NULL) = 0x57b1bd2000
faccessat(AT_FDCWD, "/etc/ld.so.preload", R_OK) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=19285, ...}) = 0
mmap(NULL, 19285, PROT_READ, MAP_PRIVATE, 3, 0) = 0x79aecf8000
close(3) = 0
openat(AT_FDCWD, "/lib/aarch64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0\267\0\1\0\0\0p\16\2\0\0\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0777, st_size=1439544, ...}) = 0
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x79aecf6000
mmap(NULL, 1511520, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x79aeb5e000
mprotect(0x79aecb7000, 61440, PROT_NONE) = 0
mmap(0x79aecc6000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x158000) = 0x79aecc6000
mmap(0x79aeccc000, 12384, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x79aeccc000
close(3) = 0
mprotect(0x79aecc6000, 16384, PROT_READ) = 0
mprotect(0x5787f5f000, 4096, PROT_READ) = 0
mprotect(0x79aecff000, 4096, PROT_READ) = 0
munmap(0x79aecf8000, 19285) = 0
brk(NULL) = 0x57b1bd2000
brk(0x57b1bf3000) = 0x57b1bf3000
fstat(1, {st_mode=S_IFCHR|0600, st_rdev=makedev(0x88,0x2), ...}) = 0
write(1, "evilpan\n", 8evilpan ) = 8
close(1) = 0
close(2) = 0
exit_group(0) = ?
+++ exited with 0 +++
```

对于需要监控系统调用的场景，`strace` 是个非常合适的工具，因为它基于 `PTRACE_SYSCALL` 去跟踪并基于中断的方式去接管所有系统调用，因此即便目标使用了不依赖 libc 的内联 svc 也可以被识别到。不过这个缺点也很明显，从名称也看出来，本质上该程序是基于 ptrace 对目标进行跟踪，因此如果对方代码中有反调试措施，那么就很有可能被检测到。

另外在 Android 系统中，APP 进程都是由 zygote fork 而出，因此使用 `strace` 比较不容易确定跟踪时机，而且由于许多应用有多个进程，就需要对输出结果进行额外的过滤和清洗。

> 更多关于 strace 的实现原理可以参考: [How does strace work?](https://bbs.kanxue.com/elink@e05K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2M7r3q4U0K9$3q4Y4k6h3y4D9L8%4g2V1i4K6u0W2K9h3!0Q4x3V1k6W2L8X3N6Q4x3V1j5J5x3o6p5$3i4K6u0r3x3o6u0Q4x3V1j5J5z5g2)9J5c8X3S2G2N6#2)9J5k6r3c8G2k6i4y4Q4x3X3c8K6N6s2u0S2j5$3g2Q4x3X3c8%4L8%4u0C8i4K6u0r3)

## jtrace

在早期 `strace` 程序还不支持 arm64，因此 *Jonathan Levin* 在编写 **Android Internal** 一书时就写了 `jtrace` 这个工具，旨在用于对 Android 应用的跟踪。虽然现在 Google 也在 AOSP 中支持了 `strace`，但 `jtrace` 仍然有其独特的优点:

- 支持系统属性的访问监控 (`setprop`/`getprop`)
- 支持输入事件的监控 (`InputReader`)
- 支持 Binder 信息的解析
- 支持 AIDL 的解析
- ……

虽然 `jtrace` 是闭源的，但提供了独特的插件功能，用户可以根据其提供的接口去编写一个插件(动态库)，并使用 `--plugin` 参数或者 `JTRACE_EXT_PATH` 环境变量指定的路径加载插件，从而实现自定义的系统调用参数解析处理。

虽然优点比 `strace` 多了不少，但其缺点并没有解决，`jtrace` 本身依然是基于 `PTRACE_SYSCALL` 进行系统调用跟踪的，因此还是很容易被应用的反调试检测到。

> 详见: 35eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3&6W2N6$3q4F1k6s2u0G2K9h3c8T1L8$3!0C8i4K6u0W2j5$3!0E0i4K6u0r3N6r3!0G2L8s2y4Q4x3V1k6B7N6s2u0S2j5$3g2Q4x3X3g2Z5N6r3#2D9

## Frida

[frida](https://bbs.kanxue.com/elink@c5cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3M7X3W2V1j5g2)9J5c8X3k6J5K9h3c8S2) 是目前全球最为知名的动态跟踪工具集 (Instrumentation)，支持使用 js 脚本来对目标应用程序进行动态跟踪。相信读者对于 frida 已经不陌生，这里也就不再过多介绍。其功能之丰富毋庸置疑，但也有一些硬伤，比如:

- `frida-gum` 基于 inline-hook 对目标跟踪代码进行实时重编译 (JIT)，对于应用本身有较大的侵入性；
- `frida-inject` 需要依赖 `ptrace` 对目标应用进行第一次注入并加载 agent，有一个较短的注入窗口可能会被反调试应用检测到；
- `frida` 目前尚不支持系统调用事件级别的追踪，虽然 `frida-stalker` 可以做到汇编级别，但是开销过大；
- `frida` 太过知名，以至于有很多针对 `frida` 的特征检测；
- ……

类似的 Instrumentation 工具还有 [QDBI](https://bbs.kanxue.com/elink@c1eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6I4j5X3c8A6i4K6u0W2M7i4g2S2M7X3E0K6L8r3q4T1i4K6u0W2j5$3!0E0i4K6u0r3)，[hookzz](https://bbs.kanxue.com/elink@b8cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6B7L8i4m8W2N6%4y4Q4x3V1k6p5L8$3u0T1P5b7`.`.) 等等。

## 其他

除了上面提到的这些工具，还有很多其他工具可以进行动态监控，比如 `ltrace`、`gdb` 等但这些工具都不能完美实现我的需求。既要马儿跑得快(开销小)，又要马儿不吃草(无侵入)，那我们就只有把眼光放向内核了。

# Kernel Tracing 101

如果目标是为了实现系统调用监控，以及部分系统调用参数的修改(例如 IO 重定向)，那么一个直观的想法是修改内核源码，在我们感兴趣的系统调用入口插入自己的代码实现具体功能。但是这样非常低效，一来我们要在不同的系统调用相关函数中增加代码，引入过多修改后会导致更新内核合并上游提交变得困难；二来我们每次修改后都需要重新编译内核以及对应的 AOSP 代码(因为内核在 boot.img 中，详见后文)，再烧写到手机或模拟器中，流程过于复杂。

另外一个想法是通过在内核代码中引入一次性的 trampoline，然后在后续增加或者减少系统调用监控入口时通过内核模块的方式去进行修改。这样似乎稍微合理一些，但其实内核中已经有了许多类似的监控方案，这样做纯属重复造轮子，效率低不说还可能随时引入 kernel panic。

## 大局观

那么，内核中都有哪些监控方案？这其实不是一个容易回答的问题，我们在日常运维时听说过 `kprobe`、`jprobe`、`uprobe`、`eBPF`、`tracefs`、`systemtab`、`perf`，……到底他们之间的的关系是什么，分别都有什么用呢？

这里推荐一篇文章: [Linux tracing systems & how they fit together](https://bbs.kanxue.com/elink@146K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6B7N6X3&6K6i4K6u0W2j5$3q4Q4x3V1k6T1L8r3!0Y4i4K6u0r3x3U0l9I4y4#2)9J5c8U0l9%4i4K6u0r3x3o6g2Q4x3V1k6D9K9h3&6#2P5q4)9J5k6s2c8J5j5h3y4A6L8X3N6Q4x3X3c8K6P5i4y4@1k6h3#2K6i4K6u0r3)，根据其中的介绍，这些内核监控方案/工具可以分为三类:

1. 数据: 根据监控数据的来源划分
2. 采集: 根据内核提供给用户态的原始事件回调接口进行划分
3. 前端: 获取和解析监控事件数据的用户工具

![trace](https://bbs.kanxue.com/upload/attach/202201/844554_CKAA7CDSGJSC6N6.jpg)

后面对这些监控方案分别进行简要的介绍。

## kprobe

简单来说，`kprobe` 可以实现动态内核的注入，基于中断的方法在任意指令中插入追踪代码，并且通过 `pre_handler`/`post_handler`/`fault_handler` 去接收回调。

### 使用

参考 Linux 源码中的 **samples/kprobes/kprobe_example.c**，一个简单的 `kprobe` 内核模块实现如下:

```
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/kprobes.h>

#define MAX_SYMBOL_LEN  64
static char symbol[MAX_SYMBOL_LEN] = "_do_fork";
module_param_string(symbol, symbol, sizeof(symbol), 0644);

/*
For each probe you need to allocate a kprobe structure
*/
static struct kprobe kp = {
    .symbol_name = symbol,
};

/*
kprobe pre_handler: called just before the probed instruction
is executed
*/
static int handler_pre(struct kprobe *p, struct pt_regs *regs)
{
    pr_info("<%s> pre_handler: p->addr = 0x%p, pc = 0x%lx\n", p->symbol_name, p->addr, (long)regs->pc);
    /*
    A dump_stack() here will give a stack backtrace
    */
    return 0;
}

/*
kprobe post_handler: called after the probed instruction
is executed
*/
static void handler_post(struct kprobe *p, struct pt_regs *regs, unsigned long flags)
{
    pr_info("<%s> post_handler: p->addr = 0x%p\n", p->symbol_name, p->addr);
}

/*
* fault_handler: this is called if an exception is generated for any
* instruction within the pre- or post-handler, or when Kprobes
* single-steps the probed instruction.
*/
static int handler_fault(struct kprobe *p, struct pt_regs *regs, int trapnr)
{
    pr_info("fault_handler: p->addr = 0x%p, trap #%d\n", p->addr, trapnr);
    /*
    Return 0 because we don't handle the fault.
    */
    return 0;
}

static int __init kprobe_init(void)
{
    int ret;
    kp.pre_handler = handler_pre;
    kp.post_handler = handler_post;
    kp.fault_handler = handler_fault;

    ret = register_kprobe(&kp);
    if (ret < 0) {
        pr_err("register_kprobe failed, returned %d\n", ret);
        return ret;
    }
    pr_info("Planted kprobe at %p\n", kp.addr);
    return 0;
}

static void __exit kprobe_exit(void)
{
    unregister_kprobe(&kp);
    pr_info("kprobe at %p unregistered\n", kp.addr);
}

module_init(kprobe_init)
module_exit(kprobe_exit)
MODULE_LICENSE("GPL");
```
