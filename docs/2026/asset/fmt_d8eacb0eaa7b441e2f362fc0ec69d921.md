
```c
int
os_init_common(int argc, char *argv[], char *envp[])
{
    initprogname(argc > 0 ? argv[0] : "sudo");
#ifdef STATIC_SUDOERS_PLUGIN
    preload_static_symbols();
#endif
    gc_init();
    return 0;
}
```

但是`initprogname`会优先使用`__progname`宏获取程序名，而不是传递进来的`argv[0]`。

```c
void
initprogname(const char *name)
{
# ifdef HAVE___PROGNAME
extern const char *__progname;
if (__progname != NULL && *__progname != '\0')
    progname = __progname;
else
# endif
if ((progname = strrchr(name, '/')) != NULL) {
    progname++;
} else {
    progname = name;
}
/* Check for libtool prefix and strip it if present. */
if (progname[0] == 'l' && progname[1] == 't' && progname[2] == '-' &&
    progname[3] != '\0')
    progname += 3;
}
```

因此，为了使被测程序通过`argv[0]`获取程序名，我们将`HAVE___PROGNAME`的部分删除。

```diff
--- ../origin_file/progname.c 2023-01-27 01:21:37.829958000 -0800
+++ progname.c 2023-01-27 01:23:42.824771537 -0800
@@ -59,13 +59,6 @@
 void
 initprogname(const char *name)
 {
-# ifdef HAVE___PROGNAME
- extern const char *__progname;
-
- if (__progname != NULL && *__progname != '\0')
- progname = __progname;
- else
-# endif
if ((progname = strrchr(name, '/')) != NULL) {
progname++;
} else {
```

# 编译

再完成上述四个补丁后，我们再次编译得到最终文件。

重新开一个容器，将最终代码放入`/src`，创建`/fuzz`，同时设置好主机跟踪目录，便于观察fuzz情况及其进展。

```shell
$ sudo docker run -it \
    -v path/to/all_change:/src \
    -v path/to/trace_paper_fuzz:/fuzz \
    aflplusplus/aflplusplus /bin/bash
```

在docker中添加uid为1000的普通用户

```shell
$ useradd -u 1000 aflfuzzer
```

使用`afl-clang-fast`进行插桩编译，使用`afl-gcc`编译得到的文件无法运行，相关分析我放到文末。

```shell
$ cd /src
$ make clean
$ CFLAGS="-g" LDFLAGS="-g" CC=afl-clang-fast ./configure --prefix=/fuzz/release --disable-shared
$ make
$ make install
```

编译成功后，再次验证poc能够引起崩溃。

```shell
echo -ne "sudoedit\x00-s\x00\x5c\x00aaaaaaaaaaaaaaaaaaaaaaaaaa\x00\x00" | ./sudo
# 为啥不用直接用\0，我觉得可以，但是我本机解析经常出问题，因此本文全篇都是用\xHH
```

![](upload/attach/202301/970576_G85MNWSGMJ45EWA.png)

# Fuzzing

经历总总曲折，总算可以开始测试了。

```shell
$ cd /fuzz
$ mkdir {input,output}
$ echo -ne "sudo\x00id\x00\x00" > input/payload1
$ echo -ne "sudoedit\x00id\x00\x00" > input/payload2
afl-fuzz -i input/ -o output/ -D -M Master /fuzz/release/bin/sudo
# 可以创建多个从fuzzer辅助测试
afl-fuzz -i input/ -o output/ -D -S slave1 /fuzz/release/bin/sudo
```

![](upload/attach/202301/970576_YDEZK2HARHP7AMA.png)

可以发现，很快就能获得一个崩溃。

![](upload/attach/202301/970576_KJS8DVXCWD93JX9.png)

查看崩溃，符合预期

![](upload/attach/202301/970576_AUYVPD6QVNPRF6M.png)

# afl-gcc失败原因

在进行插桩编译的过程中，笔者一开始使用的是`afl-gcc`编译，但是编译出来的文件运行会直接崩溃。查找相关资料推荐使用`llvm`模式编译。为了弄清楚`afl-gcc`编译的文件失败的原因，笔者对此做出必要探索。

使用`afl-gcc`编译

```shell
$ make clean
$ CFLAGS="-g" LDFLAGS="-g" CC=afl-gcc ./configure --prefix=/src/gcc_compile --disable-shared
$ make
$ make install
$ ./sudoedit
```

回顾一下插桩程序的运行过程

被测程序会在第一次执行`__afl_maybe_log`时进行初始化，第一次调用时共享内存指针`__afl_area_ptr`为空，进而调用`__afl_setup`初始化forkserver。

`__afl_setup`首先检查`__afl_setup_failure`是否为空，如果不为空代表已经初始化失败过，调用`__afl_return`返回，否则调用`__afl_setup_first`进行初始化。

`__afl_setup_first`会保存所有寄存器的值，然后调用`getenv`获取`SHM_ENV_VAR`（fuzz程序保存的共享内存id）

![](upload/attach/202301/970576_SDPNDHT9F4HGQPZ.png)

然而跟进`getenv`发现，`getenv`又调用了`__afl_maybe_log`，也就是说`getenv`也被插桩了。

![](upload/attach/202301/970576_CZ6YVKE7VPHQ3KR.png)

然后，本来是调用`__afl_maybe_log`进行初始化，但是初始化的过程又调用了`__afl_maybe_log`，而此时还未初始化完毕，于是又会进行初始化操作，就导致了程序执行流程的疯狂套娃。由于执行过程中，会保留寄存器到栈上，因此栈资源被疯狂使用，最终进程被操作系统杀掉。

为什么`getenv`会被插桩呢，`getenv`原本是c库函数，但是在sudo源代码中的`env_hook.c`中定义同名的`getenv`函数。

```c
__dso_public char *
getenv(const char *name)
{
    char *val = NULL;
    switch (process_hooks_getenv(name, &val)) {
    case SUDO_HOOK_RET_STOP:
        return val;
    case SUDO_HOOK_RET_ERROR:
        return NULL;
    default:
        return getenv_unhooked(name);
    }
}
```

而`afl-gcc`的插桩是通过解析编译过程中的`.s`汇编文件，在需要插桩的地方，添加插桩的汇编代码。

```c
afl-as.h
"\n"
"/* --- AFL TRAMPOLINE (64-BIT) --- */\n"
"\n"
".align 4\n"
"\n"
"leaq -(128+24)(%%rsp), %%rsp\n"
"movq %%rdx, 0(%%rsp)\n"
"movq %%rcx, 8(%%rsp)\n"
"movq %%rax, 16(%%rsp)\n"
"movq $0x%08x, %%rcx\n"
"call __afl_maybe_log\n"
"movq 16(%%rsp), %%rax\n"
"movq 8(%%rsp), %%rcx\n"
"movq 0(%%rsp), %%rdx\n"
"leaq (128+24)(%%rsp), %%rsp\n"
"\n"
"/* --- END --- */\n"
"\n";
```

因此`afl-gcc`在编译`env_hook.c`时，也无可避免地对`getenv`进行了插桩。而使用`CC=afl-clang-fast`编译，在`llvm`模式下对编译中间码IR进行插桩，就不会出现这个问题。

# 环境配置

下载源码，源代码下载连接：[105K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6N6h3c8G2i4K6u0W2N6%4y4Q4x3V1j5](https://bbs.kanxue.com/elink@b53K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6N6h3c8G2i4K6u0W2N6%4y4Q4x3V1j5)。
