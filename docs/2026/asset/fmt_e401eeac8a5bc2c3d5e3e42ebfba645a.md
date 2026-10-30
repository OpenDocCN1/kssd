
> [seccomp_policy.c](https://bbs.kanxue.com/elink@11dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4y4@1i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2j5$3!0E0i4K6u0r3L8i4y4@1k6h3#2E0i4K6u0r3x3h3u0U0x3o6k6U0y4e0u0S2j5X3t1%4j5U0k6T1y4r3k6W2k6h3j5%4z5h3b7%4j5X3k6X3k6U0f1^5x3e0g2Q4x3U0y4X3K9h3I4W2i4K6u0V1M7$3g2U0j5$3!0E0M7q4)9#2k6Y4m8G2L8r3W2U0P5g2)9J5k6r3x3).

```c
#include <fcntl.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <assert.h>
#include <linux/seccomp.h>
#include <sys/prctl.h>
#include "seccomp-bpf.h"

void install_syscall_filter()
{
    struct sock_filter filter[] = {
        /* Validate architecture. */
        VALIDATE_ARCHITECTURE,
        /* Grab the system call number. */
        EXAMINE_SYSCALL,
        /* List allowed syscalls. We add open() to the set of
           allowed syscalls by the strict policy, but not close(). */
        ALLOW_SYSCALL(rt_sigreturn),
#ifdef __NR_sigreturn
        ALLOW_SYSCALL(sigreturn),
#endif
        ALLOW_SYSCALL(exit_group),
        ALLOW_SYSCALL(exit),
        ALLOW_SYSCALL(read),
        ALLOW_SYSCALL(write),
        ALLOW_SYSCALL(open),
        KILL_PROCESS,
    };
    struct sock_fprog prog = {
        .len = (unsigned short)(sizeof(filter) / sizeof(filter[0])),
        .filter = filter,
    };

    assert(prctl(PR_SET_NO_NEW_PRIVS, 1, 0, 0, 0) == 0);

    assert(prctl(PR_SET_SECCOMP, SECCOMP_MODE_FILTER, &prog) == 0);
}

int main(int argc, char **argv)
{
    int output = open("output.txt", O_WRONLY);
    const char *val = "test";

    printf("Calling prctl() to set seccomp with filter...\n");

    install_syscall_filter();

    printf("Writing to an already open file...\n");
    write(output, val, strlen(val) + 1);

    printf("Trying to open file for reading...\n");
    int input = open("output.txt", O_RDONLY);

    printf("Note that open() worked. However, close() will not\n");
    close(input);

    printf("You will not see this message--the process will be killed first\n");
}
```

**执行结果**

```bash
$ ./seccomp_policy
Calling prctl() to set seccomp with filter...
Writing to an already open file...
Trying to open file for reading...
Note that open() worked. However, close() will not
Bad system call
```

## seccomp-tools

一款用于分析 seccomp 的开源工具，项目地址：b68K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1j5i4k6A6k6o6V1@1x3X3A6Q4x3V1k6K6k6h3y4U0L8$3#2H3i4K6u0V1N6r3!0G2L8s2x3.

主要功能：

- Dump：从可执行文件中自动转储 seccomp BPF
- Disasm：将 seccomp BPF 转换为人类可读的格式
- Asm：使编写 seccomp 规则类似于编写代码
- Emu：模拟 seccomp 规则

安装

```bash
sudo apt install gcc ruby-dev
gem install seccomp-tools
```

使用

```bash
null@ubuntu:~/seccomp$ seccomp-tools dump ./simple_syscall_seccomp
line  CODE  JT   JF      K
=================================
0000: 0x20 0x00 0x00 0x00000004  A = arch
0001: 0x15 0x00 0x05 0xc000003e  if (A != ARCH_X86_64) goto 0007
0002: 0x20 0x00 0x00 0x00000000  A = sys_number
0003: 0x35 0x00 0x01 0x40000000  if (A < 0x40000000) goto 0005
0004: 0x15 0x00 0x02 0xffffffff  if (A != 0xffffffff) goto 0007
0005: 0x15 0x01 0x00 0x0000003b  if (A == execve) goto 0007
0006: 0x06 0x00 0x00 0x7fff0000  return ALLOW
0007: 0x06 0x00 0x00 0x00000000  return KILL
```

从输出中可知禁用了 `execve` 系统调用。

## 使用Seccomp保护Docker的安全

Seccomp 技术被用在很多应用程序上以保护系统的安全性，Docker 支持使用 seccomp 来限制容器的系统调用，不过需要启用内核中的 `CONFIG_SECCOMP`。

```bash
null@ubuntu:~$ grep CONFIG_SECCOMP= /boot/config-$(uname -r)
CONFIG_SECCOMP=y
```

当使用 `docker run` 启动一个容器时，Docker 会使用默认的 seccomp 配置文件来对容器施加限制策略，该默认文件是以 `json` 格式编写，在 300 多个系统调用中禁用了大约 44 个系统调用，可以在 Moby 项目中找到该[源码](https://bbs.kanxue.com/elink@286K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0L8$3u0&6i4K6u0r3L8h3!0T1P5g2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6H3M7X3!0X3K9h3I4W2M7#2)9J5c8Y4y4W2j5$3y4G2L8i4m8Q4x3V1k6V1k6h3k6S2N6h3I4@1i4K6u0W2K9Y4y4G2L8R3).

```bash
null@ubuntu:~$ sudo docker run --rm -it ubuntu /bin/bash
root@85e01c28bd2c:/# bash
root@85e01c28bd2c:/# ps
PID TTY          TIME CMD
1 pts/0    00:00:00 bash
10 pts/0    00:00:00 bash
13 pts/0    00:00:00 ps
root@85e01c28bd2c:/# grep -i seccomp /proc/1/status
Seccomp: 2
```

Docker 中默认的配置文件提供了最大限度的包容性，除了默认的选择之外，Docker 允许我们自定义该配置文件来灵活的对容器的系统调用进行限制。

示例：以白名单的形式允许特定的系统调用

> `example.json`

```json
{
  "defaultAction": "SCMP_ACT_ERRNO",
  "architectures": [
    "SCMP_ARCH_X86_64",
    "SCMP_ARCH_X86",
    "SCMP_ARCH_X32"
  ],
  "syscalls": [
    {
      "names": [
        "arch_prctl",
        "sched_yield",
        "futex",
        "write",
        "mmap",
        "exit_group",
        "madvise",
        "rt_sigprocmask",
        "getpid",
        "gettid",
        "tgkill",
        "rt_sigaction",
        "read",
        "getpgrp"
      ],
      "action": "SCMP_ACT_ALLOW",
      "args": [],
      "comment": "",
      "includes": {},
      "excludes": {}
    }
  ]
}
```

- `defaultAction`：指定默认的 seccomp 操作，具体的可选参数上面已经介绍过了，最常用的无非是 `SCMP_ACT_ALLOW`、`SCMP_ACT_ERRNO`，这里选择 `SCMP_ACT_ERRNO`，表示默认禁止全部系统调用，以白名单的形式在赋予可用的系统调用。
- `architectures`：系统架构，不同的系统架构系统调用可能不同。
- `syscalls`：指定系统调用以及对应的操作，`name` 定义系统调用名，`action` 对应的操作，这里表示允许 `name` 里边中的系统调用，`args` 对应系统调用参数，可以为空。

这样，在使用 `docker run` 运行容器时，就可以使用 `--security-opt` 选项指定该配置文件来对容器进行系统调用定制。

```bash
$ docker run --rm -it --security-opt seccomp=/path/to/seccomp/example.json hello-world
```

举例，禁止容器创建文件夹，就可以用黑名单的形式禁用 `mkdir` 系统调用

> `seccomp_mkdir.json`

```json
{
  "defaultAction": "SCMP_ACT_ALLOW",
  "syscalls": [
    {
      "name": "mkdir",
      "action": "SCMP_ACT_ERRNO",
      "args": []
    }
  ]
}
```

使用该策略启动容器，并在容器中创建文件夹时，就会收到禁止信息，不允许创建文件夹。

```bash
null@ubuntu:~/seccomp/docker$ sudo docker run --rm -it --security-opt seccomp=seccomp_mkdir.json busybox /bin/sh
/ # ls
bin   dev   etc   home  proc  root  sys   tmp   usr   var
/ # mkdir test
mkdir: can't create directory 'test': Operation not permitted
```

当然也可以不适用任何 seccomp 策略启动容器，只需要在启动选项中加上 `--security-opt seccomp=unconfined` 即可。

## zaz

`zaz seccomp` 是一个可以为容器自动生成 `json` 格式的 seccomp 文件的开源工具，项目地址：e35K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6H3K9X3u0Y4k6W2)9J5c8Y4A6S2P5W2!0q4x3#2)9^5x3q4)9^5x3R3.

主要用法为

```bash
zaz seccomp docker IMAGE COMMAND
```

它能够为特定的可执行文件定制系统调用，以只允许特定的操作，禁止其他操作

举个例子：为 alpine 中的 `ping` 命令生成 seccomp 配置文件
