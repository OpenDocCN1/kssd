
### `seccomp_init(uint32_t def_action)`

可选的`def_action`有：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7 | `SCMP_ACT_ALLOW：即初始化为允许所有系统调用，过滤为黑名单模式；`  `SCMP_ACT_KILL：则为白名单模式过滤。`  `SCMP_ACT_KILL_PROCESS：整个进程将被内核终止`  `SCMP_ACT_TRAP:如果所有系统调用都不匹配，则给线程发送一个SIGSYS信号`  `SCMP_ACT_TRACE(uint16_t msg_num)：在使用ptrace根据进程时的相关选项`  `SCMP_ACT_ERRNO(uint16_t errno)：不匹配会收到errno的返回值`  `SCMP_ACT_LOG：不影响系统调用，但是会被记录；` |

- [`seccomp_rule_add`](https://bbs.kanxue.com/elink@cc5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0j5h3^5%4i4K6u0W2L8%4u0Y4i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6E0j5h3&6Q4x3X3c8H3j5h3N6W2M7#2)9J5c8X3#2S2L8U0y4Q4x3V1k6K6k6h3y4U0L8$3#2H3i4K6g2X3M7Y4g2D9k6g2)9#2k6X3q4V1k6q4)9J5k6e0y4Q4x3X3g2Z5N6r3#2D9) ：添加一条规则，函数原型为：

|  |  |
| --- | --- |
| 1 | `int` `seccomp_rule_add(scmp_filter_ctx ctx, uint32_t action,``int` `syscall, unsigned` `int` `arg_cnt, ...);` |

其中`arg_cnt`参数表明是否需要对对应系统调用的参数做出限制以及指示做出限制的个数，如果仅仅需要允许或者禁止所有某个系统调用，`arg_cnt`直接传入0即可，如 `seccomp_rule_add(ctx, SCMP_ACT_KILL, SCMP_SYS(execve), 0)` 即禁用`execve`，不管其参数如何。 如果`arg_cnt`的参数不为0， 那 `arg_cnt` 表示后面限制的参数的个数，也就是只有调用 `execve`，且参数满足要求时，才会拦截 `syscall` 。如果想要更细粒度的过滤系统调用，把参数也考虑进去,就要设置`arg_cnt`不为零，然后在利用宏做一些过滤。

举个例子，拦截 `write` 函数 参数大于 `0x10` 时的系统调用 ：

> `seccomp_write_limit.c`：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13 | `#include <unistd.h>`  `#include <seccomp.h>`  `#include <linux/seccomp.h>`    `int` `main(void){`  `scmp_filter_ctx ctx;`  `ctx` `=` `seccomp_init(SCMP_ACT_ALLOW);`  `seccomp_rule_add(ctx, SCMP_ACT_KILL, SCMP_SYS(write),``1``,SCMP_A2(SCMP_CMP_GT,``0x10``));``/``/``第``2``(从``0``)个参数大于``0x10`  `seccomp_load(ctx);`  `write(``1``,``"1234567812345678"``,``0x10``);``/``/``不被拦截`  `write(``1``,``"i will give you a shell\n"``,``24``);``/``/``会拦截`  `return` `0``;`  `}` |

编译执行

|  |  |
| --- | --- |
| 1  2  3 | `null@ubuntu:~``/``seccomp$ gcc` `-``g seccomp_write_limit.c` `-``o seccomp_write_limit` `-``lseccomp`  `null@ubuntu:~``/``seccomp$ .``/``seccomp_write_limit`  `1234567812345678Bad` `system call (core dumped)` |

其中`SCMP_A2`代表为第二个参数指定比较结构，`SCMP_CMP_GT`代表 大于(greater than)，详细内容如下。

> `libseccmop/include/[seccomp.h.in](https://bbs.kanxue.com/elink@b13K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6k6h3y4U0L8$3#2H3i4K6u0r3L8r3W2T1M7$3g2U0j5$3!0E0M7q4)9J5c8X3u0D9L8$3u0Q4x3V1j5K6k6U0m8W2y4o6N6X3k6e0t1%4x3e0N6T1y4K6y4U0j5$3g2X3y4U0S2U0j5e0p5^5k6U0W2X3y4K6t1&6y4$3g2W2y4K6y4W2j5X3t1J5i4K6u0r3K9h3&6U0L8s2g2V1k6g2)9J5c8Y4y4W2j5$3y4G2L8i4m8Q4x3X3g2Z5i4K6u0W2K9h3^5`.)：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49  50  51  52  53  54  55  56  57  58  59  60  61  62  63  64  65  66  67  68  69  70  71  72  73  74  75  76  77  78  79  80  81  82  83  84  85  86  87  88  89  90  91  92  93  94  95  96  97  98  99  100  101  102  103 | `...`  `...`    `/``*``*`  `*` `Comparison operators`  `*``/`  `enum scmp_compare {`  `_SCMP_CMP_MIN` `=` `0``,`  `SCMP_CMP_NE` `=` `1``,` `/``*``*``<` `not` `equal` `*``/`  `SCMP_CMP_LT` `=` `2``,` `/``*``*``< less than` `*``/`  `SCMP_CMP_LE` `=` `3``,` `/``*``*``< less than` `or` `equal` `*``/`  `SCMP_CMP_EQ` `=` `4``,` `/``*``*``< equal` `*``/`  `SCMP_CMP_GE` `=` `5``,` `/``*``*``< greater than` `or` `equal` `*``/`  `SCMP_CMP_GT` `=` `6``,` `/``*``*``< greater than` `*``/`  `SCMP_CMP_MASKED_EQ` `=` `7``,` `/``*``*``< masked equality` `*``/`  `_SCMP_CMP_MAX,`  `};`  `...`  `struct scmp_arg_cmp {`  `unsigned` `int` `arg;` `/``*``*``< argument number, starting at` `0` `*``/`  `enum scmp_compare op;` `/``*``*``< the comparison op, e.g. SCMP_CMP_``*` `*``/`  `scmp_datum_t datum_a;`  `scmp_datum_t datum_b;`  `};`  `....`  `/``*``*`  `*` `Specify a` `32``-``bit argument comparison struct` `for` `use` `in` `declaring rules`  `*` `@param arg the argument number, starting at` `0`  `*` `@param op the comparison operator, e.g. SCMP_CMP_``*`  `*` `@param datum_a dependent on comparison (``32``-``bits)`  `*` `@param datum_b dependent on comparison, optional (``32``-``bits)`  `*``/`  `#define SCMP_CMP32(x, y, ...) \`  `_SCMP_MACRO_DISPATCHER(_SCMP_CMP32_, __VA_ARGS__)(x, y, __VA_ARGS__)`    `/``*``*`  `*` `Specify a` `64``-``bit argument comparison struct` `for` `argument` `0`  `*``/`  `#define SCMP_A0_64(...)        SCMP_CMP64(0, __VA_ARGS__)`  `#define SCMP_A0            SCMP_A0_64`    `/``*``*`  `*` `Specify a` `32``-``bit argument comparison struct` `for` `argument` `0`  `*``/`  `#define SCMP_A0_32(x, ...)    SCMP_CMP32(0, x, __VA_ARGS__)`    `/``*``*`  `*` `Specify a` `64``-``bit argument comparison struct` `for` `argument` `1`  `*``/`  `#define SCMP_A1_64(...)        SCMP_CMP64(1, __VA_ARGS__)`  `#define SCMP_A1            SCMP_A1_64`    `/``*``*`  `*` `Specify a` `32``-``bit argument comparison struct` `for` `argument` `1`  `*``/`  `#define SCMP_A1_32(x, ...)    SCMP_CMP32(1, x, __VA_ARGS__)`    `/``*``*`  `*` `Specify a` `64``-``bit argument comparison struct` `for` `argument` `2`  `*``/`  `#define SCMP_A2_64(...)        SCMP_CMP64(2, __VA_ARGS__)`  `#define SCMP_A2            SCMP_A2_64`    `/``*``*`  `*` `Specify a` `32``-``bit argument comparison struct` `for` `argument` `2`  `*``/`  `#define SCMP_A2_32(x, ...)    SCMP_CMP32(2, x, __VA_ARGS__)`    `/``*``*`  `*` `Specify a` `64``-``bit argument comparison struct` `for` `argument` `3`  `*``/`  `#define SCMP_A3_64(...)        SCMP_CMP64(3, __VA_ARGS__)`  `#define SCMP_A3            SCMP_A3_64`    `/``*``*`  `*` `Specify a` `32``-``bit argument comparison struct` `for` `argument` `3`  `*``/`  `#define SCMP_A3_32(x, ...)    SCMP_CMP32(3, x, __VA_ARGS__)`    `/``*``*`  `*` `Specify a` `64``-``bit argument comparison struct` `for` `argument` `4`  `*``/`  `#define SCMP_A4_64(...)        SCMP_CMP64(4, __VA_ARGS__)`  `#define SCMP_A4            SCMP_A4_64`    `/``*``*`  `*` `Specify a` `32``-``bit argument comparison struct` `for` `argument` `4`  `*``/`  `#define SCMP_A4_32(x, ...)    SCMP_CMP32(4, x, __VA_ARGS__)`    `/``*``*`  `*` `Specify a` `64``-``bit argument comparison struct` `for` `argument` `5`  `*``/`  `#define SCMP_A5_64(...)        SCMP_CMP64(5, __VA_ARGS__)`  `#define SCMP_A5            SCMP_A5_64`    `/``*``*`  `*` `Specify a` `32``-``bit argument comparison struct` `for` `argument` `5`  `*``/`  `#define SCMP_A5_32(x, ...)    SCMP_CMP32(5, x, __VA_ARGS__)`    `...`  `...` |

除了`seccomp_rule_add`之外，还有其他添加规则的函数，如：`seccomp_rule_add_array ()`、` seccomp_rule_add_exact ()`和`seccomp_rule_add_exact_array ()`，详细信息可查看参考链接。

- `seccomp_load`：将当前的 `seccomp` 过滤器加载到内核中 ，函数原型：

|  |  |
| --- | --- |
| 1 | `int` `seccomp_load(scmp_filter_ctx ctx);` |

- `seccomp_reset` : 释放现有的过滤上下文重新初始化之前的状态，并且只能在成功调用`seccomp_init ()` 之后才能使用。

|  |  |
| --- | --- |
| 1 | `int` `seccomp_reset（scmp_filter_ctx ctx ，uint32_t def_action ）` |

# 其他工具

## seccmop-bpf.h

[`seccomp-bpf.h`](https://bbs.kanxue.com/elink@015K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2K9s2g2H3L8%4N6W2M7X3c8F1M7#2)9J5c8Y4y4W2j5$3k6A6L8s2c8W2M7W2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6K6k6h3y4U0L8$3#2H3i4K6u0V1j5Y4m8X3i4K6u0W2K9l9`.`.)是由开发人员编写的一个十分便捷的头文件用于开发`seccomp-bpf` 。该头文件已经定义好了很多常见的宏，如验证系统架构、允许系统调用等功能，十分便捷，如下所示。

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16 | `...`  `define VALIDATE_ARCHITECTURE \`  `BPF_STMT(BPF_LD``+``BPF_W``+``BPF_ABS, arch_nr), \`  `BPF_JUMP(BPF_JMP``+``BPF_JEQ``+``BPF_K, ARCH_NR,` `1``,` `0``), \`  `BPF_STMT(BPF_RET``+``BPF_K, SECCOMP_RET_KILL)`    `define EXAMINE_SYSCALL \`  `BPF_STMT(BPF_LD``+``BPF_W``+``BPF_ABS, syscall_nr)`    `define ALLOW_SYSCALL(name) \`  `BPF_JUMP(BPF_JMP``+``BPF_JEQ``+``BPF_K, __NR_``##name, 0, 1), \`  `BPF_STMT(BPF_RET``+``BPF_K, SECCOMP_RET_ALLOW)`    `define KILL_PROCESS \`  `BPF_STMT(BPF_RET``+``BPF_K, SECCOMP_RET_KILL)`  `...` |

应用示例：
