
```c
unsigned __int64 __fastcall challenge10(_BYTE *a1)
{
  int i; // [rsp+10h] [rbp-60h]
  int fd; // [rsp+14h] [rbp-5Ch]
  ssize_t v4; // [rsp+18h] [rbp-58h]
  char buf[72]; // [rsp+20h] [rbp-50h] BYREF
  unsigned __int64 v6; // [rsp+68h] [rbp-8h]

  v6 = __readfsqword(0x28u);
  fd = open("/proc/self/cmdline", 0);
  if ( fd != -1 )
  {
    v4 = read(fd, buf, 0x3FuLL);
    if ( v4 > 0 )
    {
      close(fd);
      for ( i = 0; v4 > i; ++i )
      {
        if ( !buf[i] )
          buf[i] = 32;
      }
      buf[v4] = 0;
      if ( !strcmp(buf, "qilinglab") )
        *a1 = 1;
    }
  }
  return __readfsqword(0x28u) ^ v6;
}
```

根据代码，我们需要让 `buf` 等于 `"qilinglab"`。

`fd`：`open()` 失败为 -1，`open()` 成功为 0。

`/proc/self/cmdline` 是一个在 Linux 系统中的特殊文件，它提供了当前进程（即访问 `/proc/self/cmdline` 的进程）的命令行参数信息。

假如我们使用命令 `./my_program arg1 arg2 arg3` 启动一个程序时，读取 `/proc/self/cmdline` 的过程在 `./my_program` 中，`/proc/self/cmdline` 文件的内容将是：

```
./my_program\0arg1\0arg2\0arg3\0
```

按照第三个挑战的方法，我们可以使用自定义文件对象来更改读取文件时的返回值：

## writeup 10

```python
class Fake_cmdline(QlFsMappedObject):
    def read(self, expected_len):
        return b'qilinglab'

    def close(self):
        return 0

def challenge10(ql):
    ql.add_fs_mapper('/proc/self/cmdline', Fake_cmdline())
```

在运行后并没有解题成功，好像是在 hook `/proc/self/cmdline` 这里存在问题，我写了个脚本单独调试以下 cmdline 读取程序：

**C 程序**：用来读取 cmdline，文件名：`test_cmdline`

```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    FILE *file;
    char ch;

    // 打开/proc/self/cmdline文件
    file = fopen("/proc/self/cmdline", "r");
    if (file == NULL) {
        printf("无法打开/proc/self/cmdline文件。\n");
        exit(EXIT_FAILURE);
    }
    printf("读取/proc/self/cmdline：\n");
    // 逐个字符读取文件内容并输出到终端
    while ((ch = fgetc(file)) != EOF) {
        // 将空字符替换为换行符以提高可读性
        if (ch == '\0') {
            putchar('\n');
        } else {
            putchar(ch);
        }
    }

    // 关闭文件
    fclose(file);

    return 0;
}
```

**Python 程序**：用来 hook `test_cmdline` 中的文件操作

```python
from qiling import *
from qiling.const import *
from qiling.os.mapper import QlFsMappedObject
from qiling.const import QL_INTERCEPT
from qiling.os.const import STRING

import struct
import os

class Fake_cmdline(QlFsMappedObject):
    def read(self, expected_len):
        return b'qilinglab'

    def close(self):
        return 0

def challenge10(ql):
    ql.add_fs_mapper('/proc/self/cmdline', Fake_cmdline())

if __name__ == '__main__':
    path = ["test_cmdline"]
    rootfs = "rootfs/x8664_linux"
    ql = Qiling(path, rootfs)
    challenge10(ql)
    ql.run()
```

运行结果如下，可以看到确实在读取操作的过程中存在问题。

```text
读取/proc/self/cmdline：
[=]     write(fd = 0x1, buf = 0x55555555b490, count = 0x1c) = 0x1c
[=]     fstat(fd = 0x3, buf_ptr = 0x80000000dc50) = -0x1 (EPERM)
[=]     read(fd = 0x3, buf = 0x55555555b8a0, length = 0x2000) = 0xd
test_cmdline
[=]     write(fd = 0x1, buf = 0x55555555b490, count = 0xd) = 0xd
[=]     read(fd = 0x3, buf = 0x55555555b8a0, length = 0x2000) = 0x0
[=]     close(fd = 0x3) = 0x0
```

切换到 Qilinglab 当时使用的版本看一下是否存在这个问题，Qilinglab 的文章是 21 年 7 月发的，对应 Qiling 版本应该是 1.2.4，使用 pip 指定版本安装方法：

```bash
pip install Qiling==1.2.4
```

运行后存在版本不兼容问题，因此我将从这个版本开始直到最新版的每个版本都进行了测试。结果如下：

**1.4.3 可以正确进行文件映射：**

```text
Challenge 10: SOLVED
[=]     write(fd = 0x1, buf = 0x55555575a260, count = 0x15) = 0x15
```

**1.4.4 无法正确进行文件映射：**

```text
Challenge 10: NOT SOLVED
[=]     write(fd = 0x1, buf = 0x55555575a260, count = 0x19) = 0x19
```

我们使用的最新版是 1.4.5，也就是说在 1.4.3 到 1.4.4 这个版本更新中可能存在问题，因此接下来进行代码审计。文件映射在源码对应的 `qiling/os/mapper.py` 中，我们对比一下文件改动：

![](https://bbs.kanxue.com/upload/attach/202305/951654_GG7FKA3Z6B439WZ.png)

改动如下：

1. 新增 `QlFsMappedCallable`
2. 由 `inspect.isclass(real_dest)` 扩展到 `callable(real_dest)`
3. 由 `real_path` 更改为 `host_path`
4. 新增 `ql.fs_mapper.add_fs_mapping()` 方法，使用 `ql.add_fs_mapper()`（`QlFsMapper` 的实例）进行调用，能够卸载映射的文件

第 1 点：

在之前，Qiling 只支持字符串和对象做为文件系统映射。而 `QlFsMappedCallable` 允许用户传入一个类对象 `real_dest`，然后在调用时实例化这个类对象。

用于 `add_fs_mapping(self, ql_path: Union[os.PathLike, str], real_dest: Union[str, QlFsMappedObject, QlFsMappedCallable])`。

第 2 点：

- `isclass` 只检查类
- `callable` 可以检查函数和类

第 3 点：

- Guest 程序指的是在 Qiling 框架下被模拟执行的程序
- `real_path` 是 Guest 程序运行时所使用的路径
- `host_path` 是可以被 Guest 程序打开的 Host 系统中的一个路径

第 4 点：

- 属于新增能够卸载映射的文件的功能

我们替换 1.4.4 的 `mapper.py` 为 1.4.3 版本，

```bash
vim /home/ubuntu/anaconda3/envs/ql1-4-4/lib/python3.9/site-packages/qiling/os/mapper.py
```

更改后依然没变化，报错。接着分析是否是针对 `/proc/self/cmdline` 有新的处理方式，加载 1.4.4 的源码到 vscode 中，发现在 `linux.py` 文件中新增以下代码：

![](https://bbs.kanxue.com/upload/attach/202305/951654_U8YZAFD5JFF6EA5.png)

看一下更新对比：

![](https://bbs.kanxue.com/upload/attach/202305/951654_BDM7HFWN4XX8Q4Q.png)

1.4.4 版本的 `linux.py` 文件与 1.4.3 版本的 `linux.py` 文件相比，有以下主要改动：

1. 添加了 `setup_procfs()` 方法，用于设置 `/proc` 伪文件系统。这使得 Qiling 可以模拟 `/proc` 目录和相关文件，提供更真实的 Linux 环境。
2. `/proc/self` 目录下的几个文件（`auxv`、`cmdline`、`environ`、`exe`、`maps`）被映射到 `QlProcFS` 类的静态方法上，这使得 Qiling 可以模拟读取这几个文件。
3. `run()` 方法中，除非是处理驱动程序或 shellcode，否则都会调用 `setup_procfs()` 设置 `/proc` 文件系统。这确保除特殊情况外，Qiling 都会模拟 `/proc` 文件系统。
4. 添加了 `fs_mapper` 属性，用于管理文件系统映射。`setup_procfs()` 就是通过 `fs_mapper` 添加 `/proc` 文件系统映射的。

之前在 1.4.3 版本中模拟 `/proc` 目录的方式有两个：

1. 直接读取 `/proc` 文件内容
2. 通过 `QlFsMappedCallable` 映射 `/proc` 文件，返回自定义内容

这两种方式存在一定缺点：

1. 无法方便地重用 `/proc` 内容，需要多次读取
2. 映射时需要定义 `QlFsMappedCallable` 类，很麻烦，处理方式不统一

所以总的来说 1.4.4 版本对 `/proc` 处理做了封装，使得它更模块化、易用。但是在新增的几个模块中对 `/proc/self/auxv` 等文件做出了额外的限制，我们在映射这几个文件时可能存在问题。因此我怀疑 `cmdline` 的问题就是在这个地方。

接着看 `run(self)` 函数：

```python
def run(self):
    # do not set-up procfs for drivers and shellcode
    if not self.ql.code and not self.ql.loader.is_driver:
        self.setup_procfs()
```

这段代码的含义是：

只有在加载 ELF 文件或可执行程序的时候，才设置 `/proc` 文件系统。

1. 对于直接加载 Shellcode（`self.ql.code`）的情况，不设置 `/proc` 文件系统
2. 对于加载驱动程序（`self.ql.loader.is_driver`）的情况，也不设置 `/proc` 文件系统

```bash
vim /home/ubuntu/anaconda3/envs/ql1-4-4/lib/python3.9/site-packages/qiling/os/linux/linux.py
```

因此将此处判断代码注释后，就应该可以成功运行了。注释后运行：

```text
Challenge 8: SOLVED
[=]     write(fd = 0x1, buf = 0x55555575a260, count = 0x14) = 0x14
[=]     openat(fd = 0xffffff9c, path = 0x5555555556c1, flags = 0x0, mode = 0x0) = 0x3
[=]     read(fd = 0x3, buf = 0x80000000dcc0, length = 0x3f) = 0x9
[=]     close(fd = 0x3) = 0x0
Challenge 9: SOLVED
[=]     write(fd = 0x1, buf = 0x55555575a260, count = 0x14) = 0x14
Challenge 10: SOLVED
[=]     write(fd = 0x1, buf = 0x55555575a260, count = 0x15) = 0x15
```

运行成功！

这个问题应该在 1.4.4 和 1.4.5 中都存在，我们可以通过新增一个选项 `ql = Qiling(path, rootfs, enable_procfs=False)` 来控制是否设置 `/proc` 文件系统，当 `enable_procfs=False` 时关闭，默认值为 `True`。我们需要修改 `core.py` 文件：
