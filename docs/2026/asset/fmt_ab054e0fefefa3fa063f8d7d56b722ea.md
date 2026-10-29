

## Android Service 属性介绍

`android:isolatedProcess="true"` 和 `android:useAppZygote="true"` 都属于 Android 清单文件（Android Manifest）中 `<service>` 标签的属性，用于控制应用服务（Service）的运行方式。

### `android:isolatedProcess="true"`

`android:isolatedProcess` 属性控制服务是否在特殊的孤立进程（isolated process）中运行。

如果此属性设置为 `"true"`，服务将运行在一个没有任何权限的孤立进程中。这个进程是应用的一个私有进程，不与应用的其他部分共享同一 Linux 用户 ID 和权限。

由于孤立进程不具备任何权限，因此无法访问设备的大部分资源，除非通过 IPC 机制向其他进程请求。这种方式可以提高安全性，因为即使孤立进程被攻破，攻击者也无法直接访问应用的其他资源。

一般来说，如果服务需要处理敏感数据或执行敏感操作，可以考虑使用孤立进程。

### `android:useAppZygote="true"`

`android:useAppZygote` 属性指定应用的服务是否应该在专用的 AppZygote 进程中运行。

AppZygote 进程是一个预加载了应用共享资源和部分类的轻量级进程，可以更快地启动服务。

如果此属性设置为 `"true"`，服务将在一个 AppZygote 进程中运行。这种方式可以提高服务启动速度，但可能会导致内存消耗增加。

一般来说，如果服务需要频繁启动，而且启动速度很关键，可以考虑使用 AppZygote 进程。

请注意，这两个属性的具体行为可能取决于 Android 的版本和设备制造商的实现，可能需要在实际设备上进行测试，以确认它们的行为。

## Hunter 的检测思路

下面主要介绍一下 Hunter 的检测思路。

### 检测多余线程 PID

主要实现思路是检测 `/proc` 下面是否存在除主进程以外的其他 PID，因为正常启动时肯定只有一个主进程。

但是，沙箱会在启动之前启动其他进程，所以这部分可以进行 bypass。后文会统一说明应该如何对抗，包括如何绕过。

这里先介绍检测思路和检测原理。其中，`replaceSecInsns` 是自行封装的函数。由于担心 `opendir` 被 Hook，因此每次执行时都会将指令替换成本地文件中的指令，而不是执行内存中的指令。

```cpp
auto orig_opendir = reinterpret_cast<DIR *(*)(const char *)>(
    replaceSecInsns(getlibcPath(), "opendir")
);
DIR *pdr = orig_opendir("/proc");

if (pdr == nullptr) {
    return getItemData(
        env,
        "程序出错请放弃修改后重试",
        nullptr,
        true,
        RISK_LEAVE_DEADLY,
        TAG_SANDBOX
    );
}

auto orig_ator = reinterpret_cast<struct dirent *(*)(DIR *)>(
    replaceSecInsns(getlibcPath(), "readdir")
);
dirent *read_ptr;

// 在 app 启动之前检测当前 app 所有的进程，判断是否存在和 main 不一样的进程
while ((read_ptr = orig_ator(pdr)) != nullptr) {
    long procPid = strtol(read_ptr->d_name, nullptr, 10);

    // LOG(INFO) << "find /proc/ child dir  " << procPid;

    // 打开成功且发现一条不等于主进程 ID 的 PID
    if (procPid && procPid != getpid()) {
        auto title = string("检测到 APK 存在沙箱内部");
        char buff[200];

        getNameByPid((pid_t) procPid, buff);

        LOG(ERROR) << ">>>>>  FIND OTHER THREAD SANDBOX "
                   << procPid << "  " << buff;

        auto &data = string("当前线程主进程 PID(")
            .append(to_string(getpid()))
            .append(")")
            .append("\n")
            .append("异常 pid -> ")
            .append(to_string(procPid))
            .append("(")
            .append("pid name: ")
            .append(buff)
            .append(")");

        // 可能存在多个异常 PID 而非一个
        getItemData(
            env,
            title,
            data.c_str(),
            false,
            RISK_LEAVE_WARN,
            TAG_SANDBOX
        );
    }
}

closedir(pdr);
```

### 对抗

这部分想要绕过也很简单。`opendir` 底层其实就是 `getdents64`。如果 `getdents` 遇到想要隐藏的文件，直接对文件进行 bypass，指向下一个指针即可。

代码来自 proot。

```cpp
case SC_getdents64:
case SC_getdents: {
    /*
     * get the result of the syscall, which is the number of bytes read by getdents
     * 获取系统调用的结果，即 getdents 读取的字节数
     */
    unsigned int res = peek_reg(tracee, CURRENT, SYSARG_RESULT);
    if (res <= 0) {
        break;
    }

    /* get the system call arguments */
    word_t orig_start = peek_reg(tracee, CURRENT, SYSARG_2);
    unsigned int count = peek_reg(tracee, CURRENT, SYSARG_3);
    char orig[count];

    char path[PATH_MAX];
    int status = readlink_proc_pid_fd(
        tracee->pid,
        (int) peek_reg(tracee, ORIGINAL, SYSARG_1),
        path
    );

    if (status < 0) {
        break;
    }

    // 不属于绑定路径则不处理，这块应该加个判断。
    // 如果这个 path 没有处理的路径应该返回。
    // 目前需要处理的只有 proc，用于隐藏调试线程。
    //
    // if (!belongs_to_guestfs(tracee, path))
    //     return 0;

    if (!StringUtils::containsInsensitive(path, "proc")) {
        break;
    }

    /*
     * retrieve the data from getdents
     * 从 getdents 检索数据
     */
    status = read_data(tracee, orig, orig_start, res);
    if (status < 0) {
        break;
    }

    /*
     * allocate a space for the copy of the data we want
     * 为我们想要的数据的副本分配一个空间
     */
    char copy[count];

    /*
     * curr will hold the current struct we're examining
     * curr 将保存我们正在检查的当前结构
     */
    struct linux_dirent64 *curr64;
    struct linux_dirent *curr32;

    /*
     * pos keeps track of where in memory the copy is
     * pos 跟踪副本在内存中的位置
     */
    char *pos = copy;

    /*
     * ptr keeps track of where in memory the original is
     * ptr 跟踪原始文件在内存中的位置
     */
    char *ptr = orig;

    /*
     * nleft keeps track of how many bytes we've saved
     * nleft 跟踪我们保存了多少字节
     */
    unsigned int nleft = 0;

    /*
     * while we're still within the memory allowed
     * 当我们还在允许的 memory 范围内时
     */
    if (get_sysnum(tracee, ORIGINAL) == SC_getdents64) {
        while (ptr < orig + res) {
            /*
             * get the current struct
             * 获取当前结构
             */
            curr64 = (struct linux_dirent64 *) ptr;

            /*
             * if the name does not match a given prefix
             * 如果名称与给定前缀不匹配
             *
             * 如果这个目录项的名称不以 HIDDEN_PREFIX 开始，
             * 也就是 hasprefix(HIDDEN_PREFIX, curr64->d_name) 返回 false，
             * 那么这个目录项会被保留在结果中，否则，这个目录项会被忽略，也就是隐藏。
             */
            //
            // if (!hasprefix(HIDDEN_PREFIX, curr64->d_name)) {
            if (!isRuntimeHideDir(
                    tracee->pid,
                    curr64->d_name,
                    tracer_pc(tracee),
                    tracer_lr(tracee)
                )) {
                /*
                 * copy the information
                 * 复制信息
                 */
                mybcopy(ptr, pos, curr64->d_reclen);

                /* move the pos and nleft */
                pos += curr64->d_reclen;
                nleft += curr64->d_reclen;
            }

            /*
             * move to the next linux_dirent
             * 隐藏这个目录，指向下一个文件
             */
            ptr += curr64->d_reclen;
        }
    } else {
        while (ptr < orig + res) {
            /*
             * get the current struct
             * 获取当前结构
             */
            curr32 = (struct linux_dirent *) ptr;

            /*
             * if the name does not match a given prefix
             */
            //
            // if (!hasprefix(HIDDEN_PREFIX, curr64->d_name)) {
            if (!isRuntimeHideDir(
                    tracee->pid,
                    curr32->d_name,
                    tracer_pc(tracee),
                    tracer_lr(tracee)
                )) {
                /*
                 * copy the information
                 * 复制信息
                 */
                mybcopy(ptr, pos, curr32->d_reclen);

                /* move the pos and nleft */
                pos += curr32->d_reclen;
                nleft += curr32->d_reclen;
            }

            /*
             * move to the next linux_dirent
             */
            ptr += curr32->d_reclen;
        }
    }

    /*
     * If there is nothing left
     * 这个文件夹里面本身没有东西，暂不处理
     */
    if (!nleft) {
        /*
         * call getdents again
         */
        //
        // if (get_sysnum(tracee, ORIGINAL) == PR_getdents64)
        //     register_chained_syscall(
        //         tracee,
        //         PR_getdents64,
        //         peek_reg(tracee, ORIGINAL, SYSARG_1),
        //         orig_start,
        //         count,
        //         0,
        //         0,
        //         0
        //     );
        // else
        //     register_chained_syscall(
        //         tracee,
        //         PR_getdents,
        //         peek_reg(tracee, ORIGINAL, SYSARG_1),
        //         orig_start,
        //         count,
        //         0,
        //         0,
        //         0
        //     );
    } else {
        /*
         * copy the data back into the register
         * 将数据复制回寄存器
         */
        status = write_data(tracee, orig_start, copy, nleft);
        if (status < 0) {
            break;
        }

        /*
         * update the return value to match the data
         * 更新返回值，使其与数据匹配
         */
        poke_reg(tracee, SYSARG_RESULT, nleft);
    }

    break;
}
```

## 执行 `ps` 命令

这部分的思路是通过执行 `ps -ef` 命令，检测是否存在其他进程。

