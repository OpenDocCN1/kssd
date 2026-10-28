
通过字符串 `"Using config section"` 定位到 `linker_config.cpp` 的 [`parse_config_file`](<https://bbs.kanxue.com/elink@62aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0M7#2)9J5k6h3q4F1k6s2u0G2K9h3c8Q4x3X3g2U0L8$3#2Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0r3M7r3I4S2N6r3k6G2M7X3#2Q4x3V1k6K6N6i4m8W2M7Y4m8J5L8$3A6W2j5%4c8Q4x3V1k6Q4x3V1u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3@1q4T1K9h3!0F1K9h3y4Q4x3V1k6D9K9h3&6C8k6i4u0Q4x3V1k6D9K9h3&6C8k6i4u0Q4y4h3k6U0L8$3&6X3K9h3N6Q4x3X3g2U0M7s2m8Q4x3@1u0T1M7s2k6Q4x3@1b7I4i4K6y4n7j5Y4m8@1i4K6y4p5x3g2)9K6b7X3I4Q4x3@1b7I4z5o6u0Q4x3@1k6Y4M7$3&6Q4x3@1c8H3j5i4u0K6k6g2)9#2k6X3y4G2L8X3k6A6k6#2)9#2k6X3k6A6L8r3g2Q4x3U0k6S2L8i4m8Q4x3@1u0Y4M7#2)9K6c8r3E0&6N6r3S2W2i4K6t1#2x3@1q4Q4x3U0f1J5c8W2)9J5y4e0u0r3j5h3&6V1M7X3!0A6k6q4)9J5k6h3N6G2L8$3N6D9k6i4y4G2N6i4u0U0k6g2)9J5k6h3y4G2L8g2)9J5y4e0u0r3M7r3I4S2N6r3k6G2M7X3#2Q4x3U0f1J5c8Y4y4#2M7r3g2J5M7s2u0G2K9X3g2U0N6q4)9J5y4e0y4r3L8r3q4F1k6#2)9J5y4e0y4p5j5#2)9J5y4e0t1#2x3V1u0Q4x3U0f1J5y4e0u0n7i4K6t1#2x3@1k6H3j5i4c8Z5i4K6t1#2x3@1c8T1K9h3!0F1K9h3y4Q4x3U0f1J5c8X3I4A6L8X3E0W2M7W2)9J5y4e0u0r3L8r3W2F1K9$3g2J5i4K6g2X3j5$3!0F1k6X3W2Y4i4K6u0W2j5%4m8H3i4K6t1#2x3U0x3I4b7g2)9J5k6q4W2W2y4o6N6F1K9X3!0r3N6i4k6E0f1@1k6F1y4p5#2y4b7h3I4h3f1%4x3^5c8Y4u0p5b7#2q4C8h3s2k6X3N6#2k6@1P5W2c8C8y4$3)9>) 函数，在阅读代码后，我们上面的尝试为什么会失败，就已经有了答案。

## 为什么设置 `/memfd:` 目录不行

[line242](<https://bbs.kanxue.com/elink@68eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0M7#2)9J5k6h3q4F1k6s2u0G2K9h3c8Q4x3X3g2U0L8$3#2Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0r3M7r3I4S2N6r3k6G2M7X3#2Q4x3V1k6K6N6i4m8W2M7Y4m8J5L8$3A6W2j5%4c8Q4x3V1k6Q4x3V1u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3@1q4T1K9h3!0F1K9h3y4Q4x3V1k6D9K9h3&6C8k6i4u0Q4x3V1k6D9K9h3&6C8k6i4u0Q4y4h3k6U0L8$3&6X3K9h3N6Q4x3X3g2U0M7s2m8Q4x3@1u0D9i4K6y4p5x3U0b7J5i4K6y4n7j5Y4m8$3i4K6y4p5x3q4)9K6b7X3u0H3N6q4)9K6c8o6p5>).

```
if (access(value.c_str(), R_OK) != 0) {
    if (errno == ENOENT) {
        // no need to test for non-existing path. skip.
        continue;
    }
    // If not accessible, don't call realpath as it will just cause
    // SELinux denial spam. Use the path unresolved.
    resolved_path = value;
}
```

因为它不是一个真实存在的目录，在代码中通过 `access` 函数判断该路径不存在，所以不行。

## 为什么设置根目录不行

[line227](<https://bbs.kanxue.com/elink@100K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0M7#2)9J5k6h3q4F1k6s2u0G2K9h3c8Q4x3X3g2U0L8$3#2Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0r3M7r3I4S2N6r3k6G2M7X3#2Q4x3V1k6K6N6i4m8W2M7Y4m8J5L8$3A6W2j5%4c8Q4x3V1k6Q4x3V1u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3@1q4T1K9h3!0F1K9h3y4Q4x3V1k6D9K9h3&6C8k6i4u0Q4x3V1k6D9K9h3&6C8k6i4u0Q4y4h3k6U0L8$3&6X3K9h3N6Q4x3X3g2U0M7s2m8Q4x3@1u0D9i4K6y4p5x3U0t1%4i4K6y4n7j5Y4m8$3i4K6y4p5x3q4)9K6b7X3u0H3N6q4)9K6c8o6p5>).

```
// remove trailing '/'
while (!value.empty() && value.back() == '/') {
    value.pop_back();
}

if (value.empty()) {
    DL_WARN("%s:%zd: warning: property value is empty (ignoring this line)",
            ld_config_file_path,
            cp.lineno());
    continue;
}
```

因为代码中删除了末尾所有的 `/` 符号，传入根目录之后字符串变成了空字符串，于是抛出了那个Warning。

## 相杀

修改无非两个思路：

1. 静态替换 `linker`
2. 动态 hook

虽然有 magisk，替换 system 分区下文件变得方便了起来，但是替换 `linker` 依然是一件麻烦事，动态 Hook 对于解决这个问题来说，我觉得更加方便。由于目标代码位置的特殊性，现有 inline hook, plt hook 轮子都不能取得比 `linker` 更早的执行时机。

> 也许是我孤陋寡闻，如果有请务必告诉我

最终我选择的方案是先 `fork` 生成一个子进程，对子进程进行 `ptrace`，修改逻辑后再 `detach` 掉。

对于逻辑的修改，主要有两步：

1. 给 `/linkerconfig/ld.config.txt` 添加 `"dir.unrestricted = /memfd:/\n"`
2. 让 `access` 这个目录返回 0

### 第一步

直接编辑该文件添加上述行即可。

### 第二步

```
bool mem_load(const std::string& image, char** argv, char** envp) {
    // 创建内存文件，设置这个参数会在exec后自动关闭这个文件
    int fd = memfd_create("/jit-cache", MFD_CLOEXEC);
    ftruncate(fd, (long)image.size()); // 设置文件长度
    void *mem = mmap(nullptr, image.size(), PROT_WRITE, MAP_SHARED, fd, 0);
    memcpy(mem, image.data(), image.size());
    munmap(mem, image.size());
    // 此时已经将ELF内容写入到内存文件里面
    int pid = fork();
    if (pid < 0) {
        printf("mem_load failed\n"); // fork 失败
        return false;
    } else if (pid == 0) {
        // 这里是子进程，使用 PTRACE_TRACEME 主动建立连接
        ptrace(PTRACE_TRACEME);
        fexecve(fd, argv, envp);
    }
    // 这里是父进程
    int status;
    struct user_regs_struct regs{};
    struct iovec iov{};
    iov.iov_base = &regs;
    iov.iov_len = sizeof(user_regs_struct);
    while (true) {
        wait(&status); // 等待子进程暂停
        if (WIFEXITED(status)) {
            break; // 子进程退出
        }
        // 读取通用寄存器，系统调用号
        ptrace(PTRACE_GETREGSET, pid, NT_PRSTATUS, &iov);
        if (regs.regs[8] == __NR_faccessat) { // access函数使用的系统调用号
            char path[] = "/memfd:";
            long word;
            // 注意，PTRACE_PEEKDATA 固定一次读取 sizeof(long) 字节长度的内容
            word = ptrace(PTRACE_PEEKDATA, pid, regs.regs[1], NULL);
            if (strcmp(path, (char*)&word) == 0) { // 判断是否是我们添加的目录
                ptrace(PTRACE_SINGLESTEP, pid, NULL, NULL); // 单步执行，让系统调用执行完
                wait(nullptr); // 等待系统调用执行完
                ptrace(PTRACE_GETREGSET, pid, NT_PRSTATUS, &iov); // 读取寄存器
                regs.regs[0] = 0; // 修改返回值为寄存器
                ptrace(PTRACE_SETREGSET, pid, NT_PRSTATUS, &iov); // 修改寄存器
                ptrace(PTRACE_DETACH, pid, NULL, NULL); // detach 进程
                return true;
            }
        }
        ptrace(PTRACE_SYSCALL, pid, NULL, NULL); // 执行到下一个系统调用时暂停
    }
    return false;
}
```
