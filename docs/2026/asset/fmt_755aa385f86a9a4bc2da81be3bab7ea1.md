

# [原创] AFL速通——流程及afl-fuzz.c源码简析

> 目录

- [source code fuzzing的基本流程](#source-code-fuzzing的基本流程)
- [Instrument](#instrument)
- [afl-gcc插桩分析](#afl-gcc插桩分析)
- [Fuzz target](#fuzz-target)
- [初始化](#初始化)
- [setup_shm()](#setup_shm)
- [Fork Server](#fork-server)
- [**关键执行函数fuzz_one()**](#关键执行函数fuzz_one)
- [**fuzz_one CALIBRATION**](#fuzz_one-calibration)
- [fuzz_one TRIMMING](#fuzz_one-trimming)
- [fuzz_one PERFORMANCE SCORE](#fuzz_one-performance-score)
- [fuzz_one变异](#fuzz_one变异)
- [**SIMPLE BITFLIP (+dictionary construction)阶段**](#simple-bitflip-dictionary-construction阶段)
- [bitflip 1/1](#bitflip-11)
- [arith 16/8](#arith-168)
- [arith 32/8](#arith-328)
- [**INTERESTING VALUES阶段**](#interesting-values阶段)
- [interest 8/8](#interest-88)
- [user extras (insert)](#user-extras-insert)
- [auto extras (over)](#auto-extras-over)
- [**RANDOM HAVOC阶段**](#random-havoc阶段)
- [**SPLICING阶段**](#splicing阶段)

**前言**

这是我读了部分AFL源码的产物，有些地方很可能表述或理解地并不准确，欢迎交流和指教。

# source code fuzzing的基本流程

![图片描述](upload/attach/202207/958400_FY9VV4H7GGPDVN6.png)

（图片引用自[AFL++文档](elink@8a0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6m8c8V1I4H3L8s2g2K6M7r3I4#2M7#2)9J5c8V1q4r3e0s2m8D9N6i4y4H3L8s2g2K6i4K6u0r3j5X3I4G2j5W2)9J5c8Y4y4@1j5h3u0D9k6g2)9J5c8X3c8G2j5%4y4Q4x3V1k6d9c8f1q4p5e0f1g2Q4x3X3g2E0k6l9`.`.)）

主要内容是Instrument target和Fuzz本体。

# Instrument

根据compiler的选择不同会影响后续fuzzing效率。

- LTO mode (afl-clang-lto/afl-clang-lto++)

LTO（Link Time Optimization）链接时优化是链接期间的程序优化，多个中间文件通过链接器合并在一起，并将它们组合为一个程序，缩减代码体积，因此链接时优化是对整个程序的分析和跨模块的优化。

需要llvm 11+，这是当前afl支持的效率最高的选择（理论上，实际情况会受未知因素影响，比如fuzzing libxml2的时候），也意味着编译要花更长时间。

- LLVM mode (afl-clang-fast/afl-clang-fast++)

依赖LLVM的optimizer，稳定性较高的编译器，用的比较多，可以跨平台（non-x86）编译。

实现了编译级插桩，效果比汇编级插桩更好。

- GCC_PLUGIN mode (afl-gcc-fast/afl-g++-fast)

效果和LLVM mode差不多，不过依赖的是GCC_plugin，也比较推荐。

- GCC mode (afl-gcc/afl-g++) (or afl-clang/afl-clang++ for clang)

相较其他编译器，没别的特色，基本用不到。

从编译的实现流程上理解插桩模式差异：

![图片描述](upload/attach/202207/958400_6K65KMKJT4C9KY6.png)

## afl-gcc插桩分析

考虑到afl的插桩方式随编译器的选择而变化，从最简单的`afl-gcc`开始入手。

先把一个简单程序用`afl-gcc`编译，[代码来源](elink@4bcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0P5h3E0@1k6i4u0Q4x3V1k6S2k6X3I4Q4x3X3c8@1M7X3q4A6L8X3W2F1k6H3`.`.)

```
#include <string.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#define INPUTSIZE 100
int process(char *input)
{
char *out;
char *rest;
int len;
if (strncmp(input, "u ", 2) == 0)
{ // upper case command
char *rest;
len = strtol(input + 2, &rest, 10); // how many characters of the string to upper-case
rest += 1; // skip the first char (should be a space)
out = malloc(len + strlen(input)); // could be shorter, but play it safe
if (len > (int)strlen(input))
{
printf("Specified length %d was larger than the input!\n", len);
return 1;
}
else if (out == NULL)
{
printf("Failed to allocate memory\n");
return 1;
}
for (int i = 0; i != len; i++)
{
char c = rest[i];
if (c > 96 && c < 123) // ascii a-z
{
c -= 32;
}
out[i] = c;
}
out[len] = 0;
strcat(out, rest + len); // append the remaining text
printf("%s", out);
free(out);
}
else if (strncmp(input, "head ", 5) == 0)
{ // head command
if (strlen(input) > 6)
{
len = strtol(input + 4, &rest, 10);
rest += 1; // skip the first char (should be a space)
rest[len] = '\0'; // truncate string at specified offset
printf("%s\n", rest);
}
else
{
fprintf(stderr, "head input was too small\n");
}
}
else if (strcmp(input, "surprise!\n") == 0)
{
// easter egg!
*(char *)1 = 2;
}
else
{
return 1;
}
return 0;
}
int main(int argc, char *argv[])
{
char *usage = "Usage: %s\n"
"Text utility - accepts commands and data on stdin and prints results to stdout.\n"
"\tInput | Output\n"
"\t------------------+-----------------------\n"
"\tu <N> <string> | Uppercased version of the first <N> bytes of <string>.\n"
"\thead <N> <string> | The first <N> bytes of <string>.\n";
char input[INPUTSIZE] = {0};
// Slurp input
if (read(STDIN_FILENO, input, INPUTSIZE) < 0)
{
fprintf(stderr, "Couldn't read stdin.\n");
}
int ret = process(input);
if (ret)
{
fprintf(stderr, usage, argv[0]);
};
return ret;
}
```

很显然，只要输出指定字符串，程序就会访问到非法内存，同时程序根据输入头部的不同产生多个分支，从而测试AFL输入样本的变异过程。

编译中程序显示对52处位置进行了插桩：

![图片描述](upload/attach/202207/958400_C55YBRGUGRTGETZ.png)

把编译得到的文件丢进IDA，可以发现编译生成的函数中有多个`__afl_maybe_log`，显然他们由`afl-gcc`的插桩产生。

![图片描述](upload/attach/202207/958400_ECD6PEEJ2TZJK3A.png)

当执行到这段代码，fuzzer知道这段代码被触发，从而统计每次输入样本的边缘覆盖率。

正常生成可执行文件过程为：

- 预处理：`.c`生成`.i`
- 编译：`.i`生成`.s`，到汇编语言
- 汇编：`.s`生成`.o`，汇编语言到机器语言
- 链接：由`.o`生成可执行文件

IR：高级语言到汇编的中间语言，可以解决平台间的差异。

llvm负责IR到汇编语言的转化，并在此过程中进行插桩。插桩的代码执行时与更新共享内存中的执行信息，从而对代码覆盖率进行统计。

使用`afl-clang-fast`编译，产生的函数`__sanitizer_cov_trace_pc_guard`，就是llvm插桩的经典例子。

![图片描述](upload/attach/202207/958400_G4E99A35AFHJAGJ.png)

ASAN(Address Sanitizer)：在数据前后添加禁止访问区域，访问到后报错。

- 存在一个判断标准，判断合法地址和非法地址
- 每个数据只能写在给他分配的位置上
- 会把原本相邻的变量隔离
- 需要更长的编译时间

# Fuzz target

源代码比较长，我就挑了几个重要函数的源码进行分析。

## 初始化

进入`main`函数，首先获取时间，循环读取参数：

```
gettimeofday(&tv, &tz);
srandom(tv.tv_sec ^ tv.tv_usec ^ getpid());
while ((opt = getopt(argc, argv, "+i:o:f:m:t:T:dnCB:S:M:x:Q")) > 0)
switch (opt) {
case 'i': /* input dir */
if (in_dir) FATAL("Multiple -i options not supported");
in_dir = optarg;
…………
```

下面接了一大堆目录处理和前期检查的函数。

## setup_shm()

forkserver、主进程、fork出的子进程间存在共享内存，这段共享内存由内核管理，其中存储数组，记录每次样本执行访问到的代码路径。

- 如果是forkserver，通过管道通信
- fork出的子进程通过共享内存传输结果

该函数用于配置共享内存和`virgin_bits`：

```
/*数组定义*/
EXP_ST u8 virgin_bits[MAP_SIZE], /* Regions yet untouched by fuzzing */
virgin_tmout[MAP_SIZE], /* Bits we haven't seen in tmouts */
virgin_crash[MAP_SIZE]; /* Bits we haven't seen in crashes */
EXP_ST void setup_shm(void) {
…………
if (!in_bitmap) memset(virgin_bits, 255, MAP_SIZE);
memset(virgin_tmout, 255, MAP_SIZE);
memset(virgin_crash, 255, MAP_SIZE);
```

将三个状态数组全部初始化为255（0~65535）。

- `virgin_bits`记录尚未覆盖的区域
- `virgin_tmout`记录timeout时的tuple信息
- `virgin_crash`记录crash时的tuple信息

```
shm_id = shmget(IPC_PRIVATE, MAP_SIZE, IPC_CREAT | IPC_EXCL | 0600);
…………
trace_bits = shmat(shm_id, NULL, 0);
```

`int shmget(key_t key, size_t size, int shmflg)`申请共享大小为65536的共享内存。

- 第一参数为`IPC_PRIVATE`，*使用`IPC_PRIVATE`创建的IPC对象, key值属性为0，和IPC对象的编号就没有了对应关系。这样毫无关系的进程，就不能通过key值来得到IPC对象的编号（因为这种方式创建的IPC对象的key值都是0）。因此，这种方式产生的IPC对象，和无名管道类似，不能用于毫无关系的进程间通信。但也不是一点用处都没有，仍然可以用于有亲缘关系的进程间通信。*

- 第二参数`MAP_SIZE`为65536，是这一段内存的大小。
- 第三参数`IPC_CREAT | IPC_EXCL | 0600`，代表这段内存的权限。
  - `0600`权限代表，只有创建者可以进行读写
  - `IPC_CREAT`如果共享内存不存在，则创建一个共享内存，否则打开操作。
  - `IPC_EXCL`只有在共享内存不存在的时候，新的共享内存才建立，否则就产生错误。

`void *shmat(int shm_id, const void *shm_addr, int shmflg)`访问共享内存。

- 第一参数指定这一段共享内存的id
- 第二参数为NULL一般，`shm_addr`指定共享内存连接到当前进程中的地址位置，通常为空，表示让系统来选择共享内存的地址。
- 第三参数`shm_flg`是一组标志位，通常为0。
- 返回一个指向共享内存起始位置的指针，存入`trace_bits`

## Fork Server

forkserver功能：

- 由主进程创建的子进程，负责fork出fuzz对象，使用pipe和主进程通信
- 管道只能在父子进程间通信，如果要实现祖孙进程通信，需要设置环境变量，孙进程通过环境变量获取文件描述符

调用链`perform_dry_run(use_argv)` -> `calibrate_case(*argv*, q, use_mem, 0, 1)` -> `init_forkserver(argv)`。

- `perform_dry_run()`：每个测试用例都执行一次，仅对初始输入执行一次测试，以确保程序按预期运行
- `calibrate_case()`：校准一个新的测试用例，只在处理输入目录和发现新路径是执行
- `init_forkserver()`：用于初始化forkserver

1. 初始参数中`st_pipe[2]`, `ctl_pipe[2]`分别为状态管道和控制管道：

```
EXP_ST void init_forkserver(char** argv) {
static struct itimerval it;
int st_pipe[2], ctl_pipe[2];
int status;
s32 rlen;
```

2. 接着fork出子进程forkserver并使其脱离主进程：

```
forksrv_pid = fork();//子进程为forkserver
if (forksrv_pid < 0) PFATAL("fork() failed"); //fork失败
if (!forksrv_pid) { //forkserver执行
…………
setsid(); //让子进程完全独立运行
```

3. 重定向forkserver的stdout、stderr到`dev_null_fd`。

视情况重定向stdin：

- 若定义了`out_file`，则把stdin重定向到`dev_null_fd`
- 否则关闭`out_fd`（间接关闭了stdin）

完成后对`FORKSRV_FD`和`FORKSRV_FD + 1`进行重定向。

[linux之dup和dup2函数解析](elink@de8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4N6W2K9i4S2A6L8W2)9#2k6U0x3H3y4K6j5@1x3o6b7#2i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5I4x3e0j5&6x3K6j5K6y4e0V1`.)

