
# AFL 源代码速通笔记

因为认识的师傅们都开始卷 fuzz 了，迫于生活压力，于是也开始看这方面的内容了。由于 `AFL` 作为一个现在仍然适用且比较经典的 fuzzer，因此笔者也打算从它开始。

> 本来，本篇博文叫做《AFL 源代码阅读笔记》，结果跟着大佬们的笔记去读（sakura 师傅的笔记确实是神中神，本文也有很多地方照搬了师傅的原文，因为说实话我觉得自己也写不到那么详细），囫囵吞枣般速通了，前前后后三天时间这样，但感觉自己尚且没有自己实现的能力，还是比较令人失望的（我怎么这么菜）

---

## afl-gcc 原理

首先，一般我们用 `afl` 去 fuzz 一些项目的时候都需要用 `afl-gcc` 去代替 `gcc` 进行编译。先说结论，这一步的目的其实是为了**向代码中插桩**，完成插桩后其实还是**调用原生的 `gcc` 进行编译**。

其实这个描述有些偏颇，插桩其实是 `afl-as` 负责的，不过在这里，笔者将 `afl-gcc` 和 `afl-as` 放到同一节，因此用了这样的表述，下文会具体分析 `afl-as` 的原理。

首先需要说明的是，`gcc` 对代码的编译流程是分层次的：

> 源代码 --> 预编译后的源代码 --> 汇编代码 --> 机器码 --> 链接后的二进制文件

其中，从源代码到汇编代码的步骤由 `gcc` 完成；而汇编代码到机器码的部分由 `as` 完成。

而 `afl-gcc` 的源代码如下：

```c
int main(int argc, char** argv) {
.......
find_as(argv[0]);
edit_params(argc, argv);
execvp(cc_params[0], (char**)cc_params);
FATAL("Oops, failed to execute '%s' - check your PATH", cc_params[0]);
return 0;
}
```

- `find_as`：查找 `as` 这个二进制程序，用 `afl-as` 替换它
- `edit_params`：修改参数
- `execvp`：调用原生 `gcc` 对代码进行编译

```c
static void edit_params(u32 argc, char** argv) {
......
#else
if (!strcmp(name, "afl-g++")) {
u8* alt_cxx = getenv("AFL_CXX");
cc_params[0] = alt_cxx ? alt_cxx : (u8*)"g++";
} else if (!strcmp(name, "afl-gcj")) {
u8* alt_cc = getenv("AFL_GCJ");
cc_params[0] = alt_cc ? alt_cc : (u8*)"gcj";
} else {
u8* alt_cc = getenv("AFL_CC");
cc_params[0] = alt_cc ? alt_cc : (u8*)"gcc";
}
#endif /* __APPLE__ */
}
while (--argc) {
u8* cur = *(++argv);
if (!strncmp(cur, "-B", 2)) {
if (!be_quiet) WARNF("-B is already set, overriding");
if (!cur[2] && argc > 1) { argc--; argv++; }
continue;
}
if (!strcmp(cur, "-integrated-as")) continue;
if (!strcmp(cur, "-pipe")) continue;
#if defined(__FreeBSD__) && defined(__x86_64__)
if (!strcmp(cur, "-m32")) m32_set = 1;
#endif
if (!strcmp(cur, "-fsanitize=address") ||
!strcmp(cur, "-fsanitize=memory")) asan_set = 1;
if (strstr(cur, "FORTIFY_SOURCE")) fortify_set = 1;
cc_params[cc_par_cnt++] = cur;
}
cc_params[cc_par_cnt++] = "-B";
cc_params[cc_par_cnt++] = as_path;
if (clang_mode)
cc_params[cc_par_cnt++] = "-no-integrated-as";
if (getenv("AFL_HARDEN")) {
cc_params[cc_par_cnt++] = "-fstack-protector-all";
if (!fortify_set)
cc_params[cc_par_cnt++] = "-D_FORTIFY_SOURCE=2";
}
if (asan_set) {
/* Pass this on to afl-as to adjust map density. */
setenv("AFL_USE_ASAN", "1", 1);
} else if (getenv("AFL_USE_ASAN")) {
if (getenv("AFL_USE_MSAN"))
FATAL("ASAN and MSAN are mutually exclusive");
if (getenv("AFL_HARDEN"))
FATAL("ASAN and AFL_HARDEN are mutually exclusive");
cc_params[cc_par_cnt++] = "-U_FORTIFY_SOURCE";
cc_params[cc_par_cnt++] = "-fsanitize=address";
} else if (getenv("AFL_USE_MSAN")) {
if (getenv("AFL_USE_ASAN"))
FATAL("ASAN and MSAN are mutually exclusive");
if (getenv("AFL_HARDEN"))
FATAL("MSAN and AFL_HARDEN are mutually exclusive");
cc_params[cc_par_cnt++] = "-U_FORTIFY_SOURCE";
cc_params[cc_par_cnt++] = "-fsanitize=memory";
}
if (!getenv("AFL_DONT_OPTIMIZE")) {
#if defined(__FreeBSD__) && defined(__x86_64__)
/* On 64-bit FreeBSD systems, clang -g -m32 is broken, but -m32 itself
works OK. This has nothing to do with us, but let's avoid triggering
that bug. */
if (!clang_mode || !m32_set)
cc_params[cc_par_cnt++] = "-g";
#else
cc_params[cc_par_cnt++] = "-g";
#endif
cc_params[cc_par_cnt++] = "-O3";
cc_params[cc_par_cnt++] = "-funroll-loops";
/* Two indicators that you're building for fuzzing; one of them is
AFL-specific, the other is shared with libfuzzer. */
cc_params[cc_par_cnt++] = "-D__AFL_COMPILER=1";
cc_params[cc_par_cnt++] = "-DFUZZING_BUILD_MODE_UNSAFE_FOR_PRODUCTION=1";
}
if (getenv("AFL_NO_BUILTIN")) {
cc_params[cc_par_cnt++] = "-fno-builtin-strcmp";
cc_params[cc_par_cnt++] = "-fno-builtin-strncmp";
cc_params[cc_par_cnt++] = "-fno-builtin-strcasecmp";
cc_params[cc_par_cnt++] = "-fno-builtin-strncasecmp";
cc_params[cc_par_cnt++] = "-fno-builtin-memcmp";
cc_params[cc_par_cnt++] = "-fno-builtin-strstr";
cc_params[cc_par_cnt++] = "-fno-builtin-strcasestr";
}
cc_params[cc_par_cnt] = NULL;
}
```

挺长的，不过逻辑基本上都是重复的，主要做两件事：

- 给 `gcc` 添加一些额外的参数
- 根据参数设置一些 flag

在完成了汇编以后，接下来会使用 `afl-as` 对生成的汇编代码进行插桩：

```c
int main(int argc, char** argv) {
......
gettimeofday(&tv, &tz);
rand_seed = tv.tv_sec ^ tv.tv_usec ^ getpid();
srandom(rand_seed);
edit_params(argc, argv);
......
if (!just_version) add_instrumentation();
if (!(pid = fork())) {
execvp(as_params[0], (char**)as_params);
FATAL("Oops, failed to execute '%s' - check your PATH", as_params[0]);
}
if (pid < 0) PFATAL("fork() failed");
if (waitpid(pid, &status, 0) <= 0) PFATAL("waitpid() failed");
if (!getenv("AFL_KEEP_ASSEMBLY")) unlink(modified_file);
exit(WEXITSTATUS(status));
}
```

在 `afl-as` 中，仍然使用 `edit_params` 编辑和修改参数，并使用 `add_instrumentation` 来对生成的汇编代码进行插桩。完成插桩后，用 `fork` 生成子进程，并调用原生的 `as` 进行编译。

插桩逻辑也很朴素：
