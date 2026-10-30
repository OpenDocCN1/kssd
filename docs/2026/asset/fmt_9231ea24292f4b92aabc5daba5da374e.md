
```c
...
...
/\*\*
\* Comparison operators
\*/
enum scmp\_compare {
\_SCMP\_CMP\_MIN = 0,
SCMP\_CMP\_NE = 1, /\*\*< not equal \*/
SCMP\_CMP\_LT = 2, /\*\*< less than \*/
SCMP\_CMP\_LE = 3, /\*\*< less than or equal \*/
SCMP\_CMP\_EQ = 4, /\*\*< equal \*/
SCMP\_CMP\_GE = 5, /\*\*< greater than or equal \*/
SCMP\_CMP\_GT = 6, /\*\*< greater than \*/
SCMP\_CMP\_MASKED\_EQ = 7, /\*\*< masked equality \*/
\_SCMP\_CMP\_MAX,
};
...
struct scmp\_arg\_cmp {
unsigned int arg; /\*\*< argument number, starting at 0 \*/
enum scmp\_compare op; /\*\*< the comparison op, e.g. SCMP\_CMP\_\* \*/
scmp\_datum\_t datum\_a;
scmp\_datum\_t datum\_b;
};
....
/\*\*
\* Specify a 32-bit argument comparison struct for use in declaring rules
\* @param arg the argument number, starting at 0
\* @param op the comparison operator, e.g. SCMP\_CMP\_\*
\* @param datum\_a dependent on comparison (32-bits)
\* @param datum\_b dependent on comparison, optional (32-bits)
\*/
#define SCMP\_CMP32(x, y, ...) \
\_SCMP\_MACRO\_DISPATCHER(\_SCMP\_CMP32\_, \_\_VA\_ARGS\_\_)(x, y, \_\_VA\_ARGS\_\_)
/\*\*
\* Specify a 64-bit argument comparison struct for argument 0
\*/
#define SCMP\_A0\_64(...) SCMP\_CMP64(0, \_\_VA\_ARGS\_\_)
#define SCMP\_A0 SCMP\_A0\_64
/\*\*
\* Specify a 32-bit argument comparison struct for argument 0
\*/
#define SCMP\_A0\_32(x, ...) SCMP\_CMP32(0, x, \_\_VA\_ARGS\_\_)
/\*\*
\* Specify a 64-bit argument comparison struct for argument 1
\*/
#define SCMP\_A1\_64(...) SCMP\_CMP64(1, \_\_VA\_ARGS\_\_)
#define SCMP\_A1 SCMP\_A1\_64
/\*\*
\* Specify a 32-bit argument comparison struct for argument 1
\*/
#define SCMP\_A1\_32(x, ...) SCMP\_CMP32(1, x, \_\_VA\_ARGS\_\_)
/\*\*
\* Specify a 64-bit argument comparison struct for argument 2
\*/
#define SCMP\_A2\_64(...) SCMP\_CMP64(2, \_\_VA\_ARGS\_\_)
#define SCMP\_A2 SCMP\_A2\_64
/\*\*
\* Specify a 32-bit argument comparison struct for argument 2
\*/
#define SCMP\_A2\_32(x, ...) SCMP\_CMP32(2, x, \_\_VA\_ARGS\_\_)
/\*\*
\* Specify a 64-bit argument comparison struct for argument 3
\*/
#define SCMP\_A3\_64(...) SCMP\_CMP64(3, \_\_VA\_ARGS\_\_)
#define SCMP\_A3 SCMP\_A3\_64
/\*\*
\* Specify a 32-bit argument comparison struct for argument 3
\*/
#define SCMP\_A3\_32(x, ...) SCMP\_CMP32(3, x, \_\_VA\_ARGS\_\_)
/\*\*
\* Specify a 64-bit argument comparison struct for argument 4
\*/
#define SCMP\_A4\_64(...) SCMP\_CMP64(4, \_\_VA\_ARGS\_\_)
#define SCMP\_A4 SCMP\_A4\_64
/\*\*
\* Specify a 32-bit argument comparison struct for argument 4
\*/
#define SCMP\_A4\_32(x, ...) SCMP\_CMP32(4, x, \_\_VA\_ARGS\_\_)
/\*\*
\* Specify a 64-bit argument comparison struct for argument 5
\*/
#define SCMP\_A5\_64(...) SCMP\_CMP64(5, \_\_VA\_ARGS\_\_)
#define SCMP\_A5 SCMP\_A5\_64
/\*\*
\* Specify a 32-bit argument comparison struct for argument 5
\*/
#define SCMP\_A5\_32(x, ...) SCMP\_CMP32(5, x, \_\_VA\_ARGS\_\_)
...
...
```

除了`seccomp_rule_add`之外，还有其他添加规则的函数，如：`seccomp_rule_add_array()`、`seccomp_rule_add_exact()`和`seccomp_rule_add_exact_array()`，详细信息可查看参考链接。

- `seccomp_load`： 将当前的 seccomp 过滤器加载到内核中 ，函数原型：

```c
int seccomp_load(scmp_filter_ctx ctx);
```

- `seccomp_reset` : 释放现有的过滤上下文重新初始化之前的状态，并且只能在成功调用`seccomp_init()` 之后才能使用。

```c
int seccomp_reset（scmp_filter_ctx ctx ，uint32_t def_action ）
```

# 其他工具

## seccmop-bpf.h

`<a href="elink@66fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2K9s2g2H3L8%4N6W2M7X3c8F1M7#2)9J5c8Y4y4W2j5$3k6A6L8s2c8W2M7W2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6K6k6h3y4U0L8$3#2H3i4K6u0V1j5Y4m8X3i4K6u0W2K9l9`.`.">seccomp-bpf.h</a>`是由开发人员编写的一个十分便捷的头文件用于开发seccomp-bpf 。该头文件已经定义好了很多常见的宏，如验证系统架构、允许系统调用等功能，十分便捷，如下所示。

```c
...
define VALIDATE_ARCHITECTURE \
BPF_STMT(BPF_LD+BPF_W+BPF_ABS, arch_nr), \
BPF_JUMP(BPF_JMP+BPF_JEQ+BPF_K, ARCH_NR, 1, 0), \
BPF_STMT(BPF_RET+BPF_K, SECCOMP_RET_KILL)
define EXAMINE_SYSCALL \
BPF_STMT(BPF_LD+BPF_W+BPF_ABS, syscall_nr)
define ALLOW_SYSCALL(name) \
BPF_JUMP(BPF_JMP+BPF_JEQ+BPF_K, __NR_##name, 0, 1), \
BPF_STMT(BPF_RET+BPF_K, SECCOMP_RET_ALLOW)
define KILL_PROCESS \
BPF_STMT(BPF_RET+BPF_K, SECCOMP_RET_KILL)
...
```

应用示例：

> `<a href="elink@879K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4y4@1i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2j5$3!0E0i4K6u0r3L8i4y4@1k6h3#2E0i4K6u0r3x3h3u0U0x3o6k6U0y4e0u0S2j5X3t1%4j5U0k6T1y4r3k6W2k6h3j5%4z5h3b7%4j5X3k6X3k6U0f1^5x3e0g2Q4x3U0y4X3K9h3I4W2i4K6u0V1M7$3g2U0j5$3!0E0M7q4)9#2k6Y4m8G2L8r3W2U0P5g2)9J5k6r3x3`.">seccomp_policy.c </a>`

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
allowed syscalls by the strict policy, but not
close(). */
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
.len = (unsigned short)(sizeof(filter)/sizeof(filter[0])),
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
write(output, val, strlen(val)+1);
printf("Trying to open file for reading...\n");
int input = open("output.txt", O_RDONLY);
printf("Note that open() worked. However, close() will not\n");
close(input);
printf("You will not see this message--the process will be killed first\n");
}
```

执行结果

```bash
$ ./seccomp_policy
Calling prctl() to set seccomp with filter...
Writing to an already open file...
Trying to open file for reading...
Note that open() worked. However, close() will not
Bad system call
```

## seccomp-tools

一款用于分析seccomp的开源工具，项目地址：`<mark class="encrypted">d8dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1j5i4k6A6k6o6V1@1x3X3A6Q4x3V1k6K6k6h3y4U0L8$3#2H3i4K6u0V1N6r3!0G2L8s2x3`.</mark>`

主要功能：

- Dump： 从可执行文件中自动转储 seccomp BPF

- Disasm： 将 seccomp BPF 转换为人类可读的格式

- Asm：使编写seccomp规则类似于编写代码

- Emu： 模拟 seccomp 规则

安装

```bash
sudo apt install gcc ruby-dev
gem install seccomp-tools
```

使用

```bash
null@ubuntu:~/seccomp$ seccomp-tools dump ./simple_syscall_seccomp
line CODE JT JF K
=================================
0000: 0x20 0x00 0x00 0x00000004 A = arch
0001: 0x15 0x00 0x05 0xc000003e if (A != ARCH_X86_64) goto 0007
0002: 0x20 0x00 0x00 0x00000000 A = sys_number
0003: 0x35 0x00 0x01 0x40000000 if (A < 0x40000000) goto 0005
0004: 0x15 0x00 0x02 0xffffffff if (A != 0xffffffff) goto 0007
0005: 0x15 0x01 0x00 0x0000003b if (A == execve) goto 0007
0006: 0x06 0x00 0x00 0x7fff0000 return ALLOW
0007: 0x06 0x00 0x00 0x00000000 return KILL
```

从输出中可知禁用了`execve`系统调用。

# 使用Seccomp保护Docker的安全

Seccomp技术被用在很多应用程序上以保护系统的安全性，Docker支持使用seccomp来限制容器的系统调用，不过需要启用内核中的`CONFIG_SECCOMP`。

```bash
null@ubuntu:~$ grep CONFIG_SECCOMP= /boot/config-$(uname -r)
CONFIG_SECCOMP=y
```

当使用`docker run` 启动一个容器时，Docker会使用默认的seccomp配置文件来对容器施加限制策略，该默认文件是以`json`格式编写， 在 300 多个系统调用中禁用了大约 44 个系统调用，可以在Moby项目中找到该`<a href="elink@0d8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0L8$3u0&6i4K6u0r3L8h3!0T1P5g2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6H3M7X3!0X3K9h3I4W2M7#2)9J5c8Y4y4W2j5$3y4G2L8i4m8Q4x3V1k6V1k6h3k6S2N6h3I4@1i4K6u0W2K9Y4y4G2L8R3`.`.">源码</a>`。
