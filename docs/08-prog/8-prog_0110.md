
# 《精通 UNIX 下 C 语言编程及项目实践》学习笔记 02 文件子系统

> 原文：[`www.pediy.com/kssd/pediy10/86640.html`](https://www.pediy.com/kssd/pediy10/86640.html)

《精通 UNIX 下 C 语言编程及项目实践》学习笔记目录

[`bbs.pediy.com/showthread.php?t=86392`](http://bbs.pediy.com/showthread.php?t=86392)

## 第二篇: 文件子系统

普天天下,莫非王土;率土之滨,莫非王臣. UNIX 之中,莫非文件.

### 四、文件系统结构

磁盘在使用前, 需要分区和格式化. 格式化操作将在磁盘分区中创建文件系统, 它们将确定文件的存储方式和索引方法, 确定磁盘空间分配和回收算法.

UNIX 文件系统的存储由`<目录-i 节点-数据块>`三级构成,其中目录存储了文件的层次结构, 数据块存储了文件的具体信息, i 节点是连接文件层次结构与其数据内容的桥梁.

UNIX 文件系统将磁盘空间划分为一系列大小相同的块, 划分为引导块、超级块、i 节点区和数据区四个部分.

文件系统通过 i 节点对文件进行控制和管理. 其中, 每个文件对应一个 i 节点, 每个 i 节点具有唯一的节点号, 记录了文件的属性和内容在磁盘上的存储位置. 但文件名并不记录在 i 节点里, 而是存储在目录文件中.

**磁盘文件如何存储?**

文件系统通过目录记载文件名及其对应的 i 节点编号, 通过 i 节点记录文件的信息和内容. 事实上, i 节点直接记录的只是文件的属性, 文件的具体内容存储在数据区的数据块中, i 节点中仅保留了一个`<磁盘地址表>`来记录文件内容存储的位置.

`<磁盘文件表>`由 13 个块号组成, 每个块号占用 4 个字节, 代表了数据区中的一个数据块编号.UNIX 文件系统采用三级索引结构存储文件,它把`<磁盘地址表>`分为直接索引地址, 一级索引地址, 二级索引地址和三级索引地址等四个部分. 其中前 10 项为直接索引地址, 直接指向文件数据所在磁盘快的块号. 第 11/12/13 项分别为一级/二级/三级索引地址. 一级间接索引的含义在于其存储的并非文件数据所在磁盘块的块号, 而是先指向一个`<磁盘块号表>`然后再指向具体磁盘块的块号. 同理, 二级/三级间接索引则是先间接指向了两次`<磁盘块号表>`才指向具体磁盘块的块号.

如果文件系统的数据块大小为 1kB, 每个`<磁盘块号表>`能够记录 256 个数据项. 那么, 直接索引能管辖 10 个数据块, 而一级索引能管辖 1\*256 个数据块, 二级索引能管辖 1\*256\*256(65536)个数据块, 三级索引能管辖 1\*256\*256\*256(16777216)个数据块.

例题: 大小为 56000K 的文件,占用多少索引块空间?

答: 因为(10+256) < 56000 < (10+256+65536), 故该文件具有二级间接索引. (56000-10-256)/256=217.7, 则文件需要二级间接索引块为 218 个,所以总索引块需要 1(一级间接索引块)+1(二级间接索引块)+218=220.

**磁盘文件读取示例(仿 ls 命令)**

通过 `stat` 结构中 `st_mode` 判断文件类型

代码:

```c
int GetFileType(mode_t st_mode, char *resp){
        if(resp == NULL)
                return 0;                 
        if(S_ISDIR(st_mode))    resp[0] = 'd';                   // 使用宏定义判断
        else if(S_ISCHR(st_mode))       resp[0] = 'c';
        else if(S_ISBLK(st_mode))       resp[0] = 'b';
        else if(S_ISREG(st_mode))       resp[0] = '-';
        else if(S_ISFIFO(st_mode))      resp[0] = 'p';
        else if(S_ISLNK(st_mode))       resp[0] = 'l';
        else resp[0] = ' ';
 
        return 1;
}
```

同样,通过 `st_mode` 判断文件访问权限

代码:

```c
int GetFileMode(mode_t st_mode, char *resp){
        if(resp == NULL)
                return 0;
        memset(resp, '-', 9);
        if(st_mode & S_IRUSR)   resp[0] = 'r';        // 使用各种宏定义与 st_mode 做与处理判断
        if(st_mode & S_IWUSR)   resp[1] = 'w';
        if(st_mode & S_IXUSR)   resp[2] = 'x';
        if(st_mode & S_IRGRP)   resp[3] = 'r';
        if(st_mode & S_IWGRP)   resp[4] = 'w';
        if(st_mode & S_IXGRP)   resp[5] = 'x';
        if(st_mode & S_IROTH)   resp[6] = 'r';
        if(st_mode & S_IWOTH)   resp[7] = 'w';
        if(st_mode & S_IXOTH)   resp[8] = 'x';
 
        return 9;
}
```

处理文件其他属性如下

代码:

```c
int GetFileOtherAttr(struct stat info, char *resp){
        struct tm *mtime;
 
        if(resp == NULL)
                return 0;
        mtime = localtime(&info.st_mtime);
                                            // 按 ls 命令显示顺序处理其他属性
        return(sprintf(resp, " %3d %6d %6d %11d %04d%02d%02d", info.st_nlink, info.st_uid, \\
                info.st_gid, info.st_size,mtime->tm_year+1900, mtime->tm_mon+1, mtime->tm_mday));
}
```

设计类似于 UNIX 命令`<ls -l>`的程序 `lsl`, 主程序如下

代码:

```c
[bill@billstone Unix_study]$ cat lsl.c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <time.h>
 
int GetFileType(mode_t st_mode, char *resp);
int GetFileMode(mode_t st_mode, char *resp);
int GetFileOtherAttr(struct stat info, char *resp);
 
int main(int argc, char **argv)
{
        struct stat info;
        char buf[100], *p = buf;
 
        if(argc != 2){
                printf("Usage: lsl filename\\n");
                return;
        }
        memset(buf, 0, sizeof(buf));
        if(lstat(argv[1], &info) == 0){
                p += GetFileType(info.st_mode, p);
                p += GetFileMode(info.st_mode, p);
                p += GetFileOtherAttr(info, p);
                printf("%s %s\\n", buf, argv[1]);
        }
        else
                printf("Open file failed!\\n");
 
        return 0;
} 
```

运行结果如下:

代码:

```bash
[bill@billstone Unix_study]$ make lsl
cc      lsl.c   -o lsl
[bill@billstone Unix_study]$ ./lsl
Usage: lsl filename
[bill@billstone Unix_study]$ ./lsl /etc/passwd
-rw-r--r--   1      0      0        1639 20090328 /etc/passwd
[bill@billstone Unix_study]$ ls -l /etc/passwd
-rw-r--r--    1 root     root         1639  3 月 28 16:38 /etc/passwd
```

### 五 标准文件编程库

在 UNIX 的应用中, 读写文件是最常见的任务. 标准文件编程库就是操作文件最简单的工具.

标准编程函数库对文件流的输入输出操作非常灵活, 我们既可以采用所见即所得的方式, 以无格式方式读写文件, 又可以对输入输出数据进行转化, 以有格式方式读写文件.

#### 文件的无格式读写

无格式读写分三类: 按字符读写, 按行读写和按块读写.

字符读写函数族:

代码:

```c
#include <stdio.h>
int getc(FILE *stream);
int fgetc(FILE *stream);
int putc(int c, FILE *stream);
int fputc(int c, FILE *stream);
```

函数 `fgetc` 的功能类似于 `getc`, 不同的是, 它的执行速度远低于 `getc`.

行读写函数族:

代码:

```c
#include <stdio.h>
char *gets(char *s);
char *fgets(char *s, int n, FILE *stream);
int puts(const char *s);
int fputs(const char *s, FILE *stream);
```

函数 `fgets` 中加入了放溢出控制, 应该优先选用. 注意函数 `fputs` 把字符串 `s`(不包括结束符`'\0'`)写入文件流 `stream` 中, 但不在输出换行符`'\n'`; 而函数 `puts` 则自动输出换行符.

块读写函数族:

代码:

```c
#include <stdio.h>
size_t fread(void *ptr, size_t size, size_t nitems, FILE *stream);
size_t fwrite(const void *ptr, size_t size, size_t nitems, FILE *stream);
```

函数 `fread` 和 `fwrite` 都不返回实际读写的字符个数, 而返回的是实际读写的数据项数.块读写函数常用于保存和恢复内存数据.

#### 文件的格式化读写

文件格式化读写时能够自动转换的数据格式有: 数据类型、精度、宽度、进制和标志等,而其一般格式为

`% [标志] [宽度] [.精度] 类型`

格式化输出函数族

代码:

```c
#include <stdio.h>
int printf(const char *format, /* [arg,] */ . . .);
int fprintf(FILE *stream, const char *format, /* [arg,] */ . . .);
int sprintf(char *s, const char *format, /* [arg,] */ . . .);
```

在做字符串处理时应该善用 `sprintf` 函数.

格式化输入函数族

代码:

```c
#include <stdio.h>
int scanf(const char format, /* [pointer,] */ . . .);
int fscanf(FILE *stream, const char format, /* [pointer,] */ . . .);
int sscanf(const char *s, const char format, /* [pointer,] */ . . .);
```

#### 二进制读写与文本读写

记得刚开始学习 C 语言的文件操作时, 这是一个最让我疑惑的问题. 我们都知道在调用 `fopen` 函数时需要指定操作类型, 比如说文本写`'r'`和二进制写`'rb'`.

那么它们究竟有何区别呢? 这要牵涉到两种存储方式: 以字符为单位存储的文本文件和以二进制数据为单位存储的二进制文件. 举个例子: 我们通常阅读的 `Readme.txt` 文件就是文本文件, 该类型文件存储的是一个一个的字符, 这些字符往往是可以打印的; 而我们的可执行程序(比如 `a.out`)则是二进制文件, 该文件是不可读的,需要解析才能识别.

那么在调用 `fopen` 函数时该如何选择呢? 如果你是先写入再从另外的地方读出, 那么两种方式都可以; 只要按写入时的方式读取就可以了. 但是, 比起文本方式, 二进制方式在保存信息时有着优势:

a) 加快了程序的执行速度, 提高了软件的执行效率. 内存中存储的都是二进制信息, 直接以二进制方式与文件交互, 可以免除二进制格式与文本格式之间的信息转换过程.

b) 节省了存储空间. 一般来讲, 二进制信息比文件信息占用更少的空间, 比如 8 位的整型数采用文本方式存储至少需要 8 字节, 而采用二进制存储只需一个整型即 4 个字节.

#### 编写变长参数函数

文件的格式化输入输出函数都支持变长参数. 定义时, 变长参数列表通过省略号`'...'`表示, 因此函数定义格式为:

`type 函数名(参数 1, 参数 2, 参数 n, . . .);`

UNIX 的变长参数通过 `va_list` 对象实现, 定义在文件`'stdarg.h'`中, 变长参数的应用模板如下所示:

代码:

```c
#include <stdarg.h>
 
function(parmN, ...){
      va_list pvar;
      .............................
      va_start(pvar, parmN);
      while()
      {
              ..................
              f = va_arg(pvar, type);
              ..................
      }
      va_end(pvar);
}
```

`va_list` 数据类型变量 `pvar` 访问变长参数列表中的参数. 宏 `va_start` 初始化变长参数列表, 根据 `parmN` 判断参数列表的起始位置. `va_arg` 获取变长列表中参数的值, `type` 指示参数的类型, 也使宏 `va_arg` 返回数值的类型. 宏 `va_arg` 执行完毕后自动更新对象 `pvar`, 将其指向下一个参数. `va_end` 关闭对变长参数的访问.

下面给出一个实例 `mysum`, 计算输入参数的和并返回

代码:



```
[bill@billstone Unix_study]$ cat mysum.c
#include <stdarg.h>

int mysum(int i, ...){           // 参数列表中, 第一个参数指示累加数的个数
        int r = 0, j = 0;
        va_list pvar;

        va_start(pvar, i);
        for(j=0;j<i;j++){
                r += va_arg(pvar, int);
        }
        va_end(pvar);

        return(r);
}

int main()
{
        printf("sum(1,4) = %d\n", mysum(1,4));
        printf("sum(2,4,8) = %d\n", mysum(2,4,8));

        return 0;
}
[bill@billstone Unix_study]$ make mysum
cc      mysum.c   -o mysum
[bill@billstone Unix_study]$ ./mysum
sum(1,4) = 4
sum(2,4,8) = 12
[bill@billstone Unix_study]$
```

## 六、低级文件编程库

低级文件编程库常常用于访问终端、管道、设备和套接字等特殊文件, 一般不用于普通磁盘文件, 这是标准文件编程库的特长.

低级文件编程库听起来似乎低级, 其实它是 `UNIX` 中的 I/O 系统调用. 它们使用文件描述符, 直接读写各类文件.

低级文件编程库在输入输出上只有块读写的功能.

### **文件锁**

多用户多任务操作系统非常重要的一个内容就是文件锁. 用户在更新文件时, 期望可以使用某种机制, 防止两进程同时更新文件同一区域而造成写丢失, 或者防止文件内容在未更新完毕时被读取等并发引起的问题, 这种机制就是文件锁.

进程在操作文件期间, 可以使用文件锁, 锁定文件中的敏感信息, 防止其他进程越权操作该部分信息. 函数 `fcntl` 提供了对文件任意区域设置锁的能力, 既可以锁住全部文件, 又可以锁住文件的部分记录, 故文件锁又称为'记录锁'.

根据文件锁的访问方式, 可以区分为读锁和写锁两种. 文件记录在同一时刻, 可以设置多个读锁, 但仅能设置一个写锁, 并且读写锁不能同时存在.

当函数 `fcntl` 专用于锁操作时, 其原型为

```c
int fcntl(int fildes, int cmd, struct flock *arg);
```

其中, 结构 `flock` 用于描述文件锁的信息, 定义于头文件 `fcntl.h` 中, 如下所示

```c
struct flock {
    short l_type;    // 锁类型, 取值为 F_RDLCK, F_WRLCK 或 F_UNLCK 之一
    short l_whence;  // 锁区域开始地址的相对位置, 取值为 SEEK_SET, SEEK_CUR 或 SEEK_END 之一
    off_t l_start;    // 锁区域开始地址偏移量
    off_t l_len;     // 锁区域的长度, 0 表示锁至文件末
    pid_t l_pid;    // 拥有锁的进程 ID 号
};
```

函数 `fcntl` 在专用于锁操作时, 参数 `cmd` 有三种取值:

(a) `F_GETLK`. 获取文件描述符 `fildes` 对应文件指定区域的文件锁信息.

(b) `F_SETLK`. 在文件描述符 `fildes` 对应的文件中指定区域设置锁信息.

(c) `F_SETLKW`. 该命令是 `F_SETLK` 命令的阻塞版本.

文件锁最典型应用于两个方面: 一是锁定文件中的临界数据, 比如并发投票时文件记录的投票数; 二是利用具有互斥性质的写锁, 实现进程的并发控制.

在锁机制的使用中,最常见的操作有锁的请求, 释放和测试等, 下面一一说明.

(a) 测试锁. 设计函数 `SeeLock`, 查询文件描述符 `fd` 对应文件的锁信息.

```c
void SeeLock(int fd, int start, int len)
{                                   // 查询描述符 fd 对应文件从 start 处开始的 len 字节中的锁信息
        struct flock arg;

        arg.l_type = F_WRLCK;
        arg.l_whence = SEEK_SET;
        arg.l_start = start;
        arg.l_len = len;

        if(fcntl(fd, F_GETLK, &arg) == -1)
                fprintf(stderr, "See Lock failed.\n");
        else if(arg.l_type == F_UNLCK)
                fprintf(stderr, "No Lock From %d to %d\n", start, start+len);
        else if(arg.l_type == F_WRLCK)
                fprintf(stderr, "Write Lock From %d to %d, id = %d\n", start, start+len, arg.l_pid);
        else if(arg.l_type == F_RDLCK)
                fprintf(stderr, "Read Lock From %d to %d, id = %d\n", start, start+len, arg.l_pid);
}
```

(b) 申请读锁. 以阻塞方式设计共享读锁申请函数 `GetReadLock`.

```c
void GetReadLock(int fd, int start, int len)
{                      // 以阻塞方式在描述符 fd 对应文件中从 start 处的 len 字节上申请共享读锁
        struct flock arg;

        arg.l_type = F_RDLCK;
        arg.l_whence = SEEK_SET;
        arg.l_start = start;
        arg.l_len = len;

        if(fcntl(fd, F_SETLKW, &arg) == -1)
                fprintf(stderr, "[%d] See Read Lock failed.\n", getpid());
        else
                fprintf(stderr, "[%d] Set Read Lock From %d to %d\n", getpid(), start, start+len);
}
```

(c) 申请写锁. 以阻塞方式设计互斥写锁申请函数 `GetWriteLock`.

```c
void GetWriteLock(int fd, int start, int len)
{                    // 以阻塞方式在描述符 fd 对应文件中从 start 处的 len 字节上申请互斥写锁
        struct flock arg;

        arg.l_type = F_WRLCK;
        arg.l_whence = SEEK_SET;
        arg.l_start = start;
        arg.l_len = len;

        if(fcntl(fd, F_SETLKW, &arg) == -1)
                fprintf(stderr, "[%d] See Write Lock failed.\n", getpid());
        else
                fprintf(stderr, "[%d] Set Write Lock From %d to %d\n", getpid(), start, start+len);
}
```

(d) 释放锁. 设计文件锁释放函数 `ReleaseLock`.

```c
void ReleaseLock(int fd, int start, int len)
{                    // 在描述符 fd 对应文件中释放从 start 处的 len 字节上的锁
        struct flock arg;

        arg.l_type = F_UNLCK;
        arg.l_whence = SEEK_SET;
        arg.l_start = start;
        arg.l_len = len;

        if(fcntl(fd, F_SETLKW, &arg) == -1)
                fprintf(stderr, "[%d] UnLock failed.\n", getpid());
        else
                fprintf(stderr, "[%d] UnLock From %d to %d\n", getpid(), start, start+len);
}
```

下面设计一个文件锁控制进程的实例 `lock1`. 为了观察阻塞方式下的锁申请, 在释放锁前休眠 30 秒.

```c
#include <stdio.h>
#include <fcntl.h>

int main()
{
        int fd;
        struct flock arg;

        if((fd = open("/tmp/tlockl", O_RDWR | O_CREAT, 0755)) < 0){
                fprintf(stderr, "open file failed.\n");
                exit(1);
        }

        SeeLock(fd, 0, 10);
        GetReadLock(fd, 0, 10);
        SeeLock(fd, 11, 20);
        GetWriteLock(fd, 11, 20);
        sleep(30);
        ReleaseLock(fd, 0, 10);
        ReleaseLock(fd, 11, 20);

        return 0;
}
```

下面是执行情况:

```
[bill@billstone Unix_study]$ make lockl
cc      lockl.c   -o lockl
[bill@billstone Unix_study]$ ./lockl &              // 先在后台执行
[2] 12725
No Lock From 0 to 10
[12725] Set Read Lock From 0 to 10
No Lock From 11 to 31
[12725] Set Write Lock From 11 to 31        // 此后休眠 30 秒
[bill@billstone Unix_study]$ ./lockl                  // 再次执行
Read Lock From 0 to 10, id = 12725
[12726] Set Read Lock From 0 to 10           // 可在同一区域申请多个共享读锁
Write Lock From 11 to 31, id = 12725
[12725] UnLock From 0 to 10
[12725] UnLock From 11 to 31
[12726] Set Write Lock From 11 to 31        // 在同一区域只能申请一个互斥写锁
[12726] UnLock From 0 to 10
[12726] UnLock From 11 to 31
[2]+  Done                    ./lockl
[bill@billstone Unix_study]$
```

## 七 目录文件编程库

`UNIX` 专门给出了一组用于目录操作的函数, 可以方便地获取目录项的确切含义.

### **工作目录**

进程在搜索文件相对路径时都会有一个起始点, 这个起始点称为'当前工作目录'. 在 `UNIX` 中对工作目录的操作可分为读取工作目录和更改工作目录两种.

(1) 读取工作目录. 函数 `getcwd` 和 `getwd` 都返回工作目录的绝对路径

```c
#include <unistd.h>
char *getcwd(char *buf, size_t size);
char *getwd(char *pathname);
```

(2) 更改工作目录.

```c
#include <unistd.h>
int chdir(const char *path);
int fchdir(int fildes);
```

下面是一个读取和更改当前工作目录的例子

```
[bill@billstone Unix_study]$ cat dirl.c
#include <unistd.h>
#include <stdio.h>

int main()
{
        char buf[255];

        fprintf(stderr, "pwd = [%s] \n", getcwd(buf, sizeof(buf)));
        chdir("../");                  // 更改工作目录为上一级目录
        fprintf(stderr, "pwd = [%s] \n", getcwd(buf, sizeof(buf)));

        return 0;
}
[bill@billstone Unix_study]$ make dirl
cc      dirl.c   -o dirl
[bill@billstone Unix_study]$ pwd
/home/bill/Unix_study
[bill@billstone Unix_study]$ ./dirl
pwd = [/home/bill/Unix_study]
pwd = [/home/bill]                        ; 更改成功
[bill@billstone Unix_study]$ pwd
/home/bill/Unix_study                    ; 不影响当前 Shell 的工作目录
[bill@billstone Unix_study]$
```

### **读取目录**

'目录文件编程库'不提倡直接更改目录文件内容, 它仅仅执行读取操作

```c
#include <dirent.h>
DIR *opendir(const char *dirname);
struct dirent *readdir(DIR *dirp);
int closedir(DIR *dirp);
```

函数 `opendir` 打开目录文件 `dirname`, 并返回一个目录流, 存储为 `DIR` 结构.

函数 `readdir` 读取当前目录项内容存入参数 `dirp` 指向的结构 `dirent` 中, 并移动目录文件指针到下一目录项. 目录中每个目录项采用结构 `dirent` 描述.



```
struct dirent {
    long            d_ino;         // 文件对应 i 节点编号
    __kernel_off_t  d_off;
    unsigned short  d_reclen;
    char            d_name[256];   // 文件名称
};
```

下面是一个简单的读取目录程序`ls2`，它列举了目录下的全部文件及其对应的 i 节点编号。

代码：

```
[bill@billstone Unix_study]$ cat ls2.c
#include <stdio.h>
#include <dirent.h>

int main(int argc, char **argv)
{
        DIR *pdir;
        struct dirent *pent;

        if(argc != 2){
                fprintf(stderr, "Usage: ls2 <directory>\n");
                return 0;
        }
        if((pdir = opendir(argv[1])) == NULL){
                fprintf(stderr, "open dir failed.\n");
                exit(1);
        }
        while(1){
                if((pent = readdir(pdir)) == NULL)
                        break;
                fprintf(stderr, "%5d %s\n", pent->d_ino, pent->d_name);
        }
        closedir(pdir);

        return 0;
}
```

执行结果如下：

代码：

```
[bill@billstone Unix_study]$ make ls2
cc     ls2.c   -o ls2
[bill@billstone Unix_study]$ ./ls2 /home/bill/Doc
134706 .
   29 ..
134708 学习笔记.doc
[bill@billstone Unix_study]$ ls -ai /home/bill/Doc
 134706 .       29 ..   134708 学习笔记.doc
```

### 八 设备文件

对于 UNIX 程序员来说，操作设备只是一件非常简单的事，因为 UNIX 中的所有设备文件都是文件，称为设备文件。

UNIX 中的设备分为块设备和字符设备。块设备主要应用于随机访问中；而字符设备常应用于顺序访问中。

对设备文件的操作一般分为打开、设置、读写和关闭几部分。

**《精通 UNIX 下 C 语言编程及项目实践》学习笔记目录**

[`bbs.pediy.com/showthread.php?t=86392`](http://bbs.pediy.com/showthread.php?t=86392)
