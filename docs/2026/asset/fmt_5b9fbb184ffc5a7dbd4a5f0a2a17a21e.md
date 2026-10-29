
## System V共享内存

```
/***** writer.c *******/
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>

int main(int argc, char** argv)
{
    int shm_id, i;
    key_t key;
    char buff[0x20];
    char* p_map;
    char* name = "./test_shm";

    setbuf(stdin, 0);
    setbuf(stdout, 0);

    key = ftok(name, 0);
    if (key == -1)
        perror("ftok error");

    shm_id = shmget(key, 4096, IPC_CREAT);
    if (shm_id == -1)
    {
        perror("shmget error");
        return;
    }
    p_map = (people*)shmat(shm_id, NULL, 0);

    printf("[+] shared memory in writer's addr: %p\n", p_map);

    printf("[+] input: ");
    read(0, buff, 0x20);
    memcpy(p_map, buff, strlen(buff));
    if (shmdt(p_map) == -1)
        perror(" detach error ");
}
```

`reader`：

```
/********** reader.c **********/
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/types.h>
#include <unistd.h>

int main(int argc, char** argv)
{
    int shm_id, i;
    key_t key;
    char* p_map;
    char* name = "./test_shm";

    key = ftok(name, 0);
    if (key == -1) {
        perror("ftok error");
        return -1;
    }
    shm_id = shmget(key, 4096, IPC_CREAT);
    if (shm_id == -1)
    {
        perror("shmget error");
        return -1;
    }
    p_map = (char *)shmat(shm_id, NULL, 0);
    printf("%s\n", p_map);
    if (shmdt(p_map) == -1) {
        perror(" detach error ");
        return -1;
    }
}
```

我试了试，需要用`root`权限运行。

运行结果：

![writer](https://bbs.kanxue.com/upload/attach/202211/968207_S7WJ4DTKTCHYZQC.jpg)

![reader](https://bbs.kanxue.com/upload/attach/202211/968207_AKGMMB39C87ZWY9.jpg)

## POSIX `mmap`文件映射实现共享内存

`POSIX`表示可移植操作系统接口（`Portable Operating System Interface`，缩写为`POSIX`），`POSIX`标准定义了操作系统应该为应用程序提供的接口标准，是`IEEE`为要在各种`UNIX`操作系统上运行的软件而定义的一系列`API`标准的总称，其正式称呼为`IEEE 1003`，而国际标准名称为`ISO/IEC 9945`。

`POSIX`提供了两种在无亲缘关系进程间共享内存区的方法：

- 内存映射文件，由`open`函数打开，由`mmap`函数把所得到的描述符映射到当前进程空间地址中的一个文件。
- 共享内存区对象（shared-memory object），由`shm_open`函数打开一个`POSIX IPC`名字，所返回的描述符由`mmap`函数映射到当前进程的地址空间。

这两种共享内存区的区别在于共享的数据的载体（底层支撑对象）不一样：内存映射文件的数据载体是物理文件；共享内存区对象，也就是共享的数据载体是物理内存。共享内存，一般是指共享内存区对象，也就是共享物理内存。

posix的共享内存机制实际上在库过程中以及用户空间的其他部分被展示为完全的文件系统的调用过程，在调用完`shm_open`之后，需要调用`mmap`来将`tmpfs`的文件映射到地址空间，接着就可以操作这个文件了，需要注意的是，别的进程也可以操作这个文件，因此这个文件其实就是共享内存。

### 相关API

```
#include <sys/types.h>
#include <sys/stat.h>        /* For mode constants */
#include <sys/mman.h>
#include <fcntl.h>           /* For O_* constants */
#include <unistd.h>

int shm_open(const char *name, int oflag, mode_t mode);
// 创建并打开一个新的共享内存对象或者打开一个既存的共享内存对象， 与函数open的用法是类似的；函数返回值是一个文件描述符，会被下面的API使用。
int shm_unlink(const char *name);
// 删除一个共享内存对象名字。
int ftruncate(int fildes, off_t length);
// 设置共享内存对象的大小，新创建的共享内存对象大小为0。
void *mmap(void *addr, size_t len, int prot, int flags, int fildes, off_t off);
// 将共享内存对象映射到调用进程的虚拟地址空间。
int munmap(void *addr, size_t len);
// 取消共享内存对象到调用进程的虚拟地址空间的映射。
int close(int fildes);
// 当shm_open函数返回的文件描述符不再使用时，使用close函数关闭它。
int fstat(int fildes, struct stat *buf);
// 获得共享内存对象属性的stat结构体。结构体中会包含共享内存对象的大小(st_size)，权限(st_mode)，所有者(st_uid)，归属组 (st_gid)。
int fchown(int fildes, uid_t owner, gid_t group);
// 改变一个共享内存对象的所有权。
int fchmod(int fildes, mode_t mode);
// 改变一个共享内存对象的权限。
```

### 示例

`writer.c`

```
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/mman.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>

#define MAXSIZE 1024*4   /*共享内存的大小，建议设置成内存页的整数倍*/
#define FILENAME "shm.test"

int main()
{
    /* 创建共享对象，可以查看/dev/shm目录 */
    int fd = shm_open(FILENAME, O_CREAT | O_TRUNC | O_RDWR, 0777);
    if (fd == -1) {
        perror("open failed:");
        exit(1);
    }

    /* 调整大小 */
    if (ftruncate(fd, MAXSIZE) == -1) {
        perror("ftruncate failed:");
        exit(1);
    }

    /* 获取属性 */
    struct stat buf;
    if (fstat(fd, &buf) == -1) {
        perror("fstat failed:");
        exit(1);
    }
    printf("the shm object size is %ld\n", buf.st_size);

    /* 建立映射关系 */
    char *ptr = (char *)mmap(NULL, MAXSIZE, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    if (ptr == MAP_FAILED) {
        perror("mmap failed:");
        exit(1);
    }
    printf("mmap %s success\n", FILENAME);
    close(fd); /* 关闭套接字 */

    /* 写入数据 */
    char *content = "hello world";
    strncpy(ptr, content, strlen(content));

    sleep(30);

    return 0;
}
```

`reader.c`

```
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/mman.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>

#define FILENAME "shm.test"

int main()
{
    /* 创建共享对象，可以查看/dev/shm目录 */
    int fd = shm_open(FILENAME, O_RDONLY, 0);
    if (fd == -1) {
        perror("open failed:");
        exit(1);
    }

    /* 获取属性 */
    struct stat buf;
    if (fstat(fd, &buf) == -1) {
        perror("fstat failed:");
        exit(1);
    }
    printf("the shm object size is %ld\n", buf.st_size);

    /* 建立映射关系 */
    char *ptr = (char *)mmap(NULL, buf.st_size, PROT_READ, MAP_SHARED, fd, 0);
    if (ptr == MAP_FAILED) {
        perror("mmap failed:");
        exit(1);
    }
    printf("mmap %s success\n", FILENAME);
    close(fd); /* 关闭套接字 */

    printf("the read msg is: %s\n", ptr);

    sleep(30);

    return 0;
}
```

要添加`-lrt`进行编译：

```
gcc writer.c -lrt -o writer
gcc reader.c -lrt -o reader
```

运行结果：

![shm_writer](https://bbs.kanxue.com/upload/attach/202211/968207_9GKESXM2WHQTNAD.jpg)

![shm_reader](https://bbs.kanxue.com/upload/attach/202211/968207_VTTPATUF5AD37CX.jpg)

## `memfd_create`和`fd`跨进程共享实现共享内存

第三种是内存`fd`，通过`memfd_create`创建基于`tmpfs`的匿名文件（返回文件描述符），再通过`mmap`建立内存映射实现内存共享。

`memfd_create`会创建一个匿名文件，并返回文件描述符。这个文件像普通文件一样，可以执行修改、截取、映射等操作。区别在于这个文件是存放在`RAM`当中，在`tmpfs`文件系统中创建。

该方法的共享内存中，存在一个问题，即如何让另外一个进程获得这个文件？因为`memfd_create`创建的是匿名文件，无法在文件系统中找到相应文件（`fd`），不能像其他共享内存机制一样约定好文件名进行共享，这让文件共享变得困难。

一种可行的方案是通过`proc`来传递文件，进程创建的匿名文件在`proc`系统中是有记录的，路径为`/proc/<pid>/fd/<fd>`，创建文件的进程需要将这个路径传递给需要共享的进程，打开该路径就实现了文件共享。

### 相关API
