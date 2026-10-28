
# [原创]《精通UNIX下C语言编程及项目实践》学习笔记04 进程通信篇

欢迎转载，请保留作者信息

bill@华中科技大学

f08K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0A6L8r3I4K6N6r3!0F1k6g2)9J5k6h3y4#2j5X3I4G2k6#2)9J5k6h3y4F1i4K6t1$3L8X3u0K6M7q4)9K6b7R3.`.`.

《精通UNIX下C语言编程及项目实践》学习笔记目录

<http://bbs.pediy.com/showthread.php?t=86392>

## 第四篇 进程通信篇

### 第十一章 管道

管道是UNIX中最古老的进程间通信工具, 它提供了进程之间的一种单向通信的方法.

管道分为无名管道和有名管道(FIFO)两种, 前者在父子进程中流行, 后者由于可以独立成磁盘文件而存在, 因而能够被无血缘关系的进程所共享.

#### 无名管道

无名管道占用两个文件描述符, 不能被非血缘关系的进程所共享, 一般应用在父子进程中.

在UNIX中, 采用函数`pipe()`创建无名管道, 原型如下:

```
#include <unistd.h>
int pipe(int fildes[2]);
```

我们一般约定`fildes[1]`描述管道的输入端, 进程向此文件描述符中写入数据, `fildes[0]`描述管道的输出端, 进程从此文件描述符中读取数据.

无名管道常用于父子进程中, 可简单分为单向管道流模型和双向管道流模型. 其中, 单向管道流根据流向分为从父进程流向子进程的管道和从子进程流向父进程的管道.

下面设计一个实例, 数据从父进程流向子进程:父进程向管道写入一行字符, 子进程读取数据并打印到屏幕上.

```
[bill@billstone Unix_study]$ cat pipe1.c
#include <unistd.h>
#include <stdio.h>
#include <sys/types.h>
#include <assert.h>

int main()
{
        int fildes[2];
        pid_t pid;
        int i,j;
        char buf[256];

        assert(pipe(fildes) == 0);             // 创建管道
        assert((pid = fork()) >= 0);           // 创建子进程
        if(pid == 0){                    // 子进程
                close(fildes[1]);                // 子进程关闭管道输出
                memset(buf, 0, sizeof(buf));
                j = read(fildes[0], buf, sizeof(buf));
                fprintf(stderr, "[child] buf=[%s] len[%d]\n", buf, j);
                return;
        }
        close(fildes[0]);                       // 父进程关闭管道输入
        write(fildes[1], "hello!", strlen("hello!"));
        write(fildes[1], "world!", strlen("world!"));

        return 0;
}
[bill@billstone Unix_study]$ make pipe1
cc     pipe1.c   -o pipe1
[bill@billstone Unix_study]$ ./pipe1
[child] buf=[hello!world!] len[12]               // 子进程一次就可以读出两次父进程写入的数据
[bill@billstone Unix_study]$
```

从上面可以看出, 在父进程中关闭`fildes[0]`, 向`fildes[1]`写入数据; 在子进程中关闭`filedes[1]`, 从`fildes[0]`中读取数据可实现从父进程流向子进程的管道.

在进程的通信中, 我们无法判断每次通信中报文的字节数, 即无法对数据流进行自行拆分, 侧耳发生了上例中子进程一次性读取父进程两次通信的报文情况.

管道是进程之间的一种单向交流方法, 要实现进程间的双向交流, 就必须通过两个管道来完成. 双向管道流的创立过程如下:

1. 创建管道, 返回两个无名管道文件描述符`fildes1`和`fildes2`:
2. 创建子进程, 子进程中继承管道`fildes1`和`fildes2`.
3. 父进程关闭只读文件描述符`fildes1[0]`, 只写描述符`fildes2[1]`
4. 子进程关闭只写文件描述符`fildes1[1]`, 只读描述符`fildes2[0]`

创建的结果如下:

```
父进程 --写--> fildes1[1] --管道--> fildes1[0] --读--> 子进程
父进程 <--读-- fildes2[0] <--管道-- fildes2[1] <--写-- 子进程
```

这里实现一个父子进程间双向通信的实例: 父进程先向子进程发送两次数据, 再接收子进程传送刚来的两次数据. 为了正确拆分时间留从父进程流向子进程的管道采用'固定长度'方法传送数据; 从子进程流向父进程的管道采用'显式长度'方法传回数据.

#### 固定长度方式

```
char bufG[255];
void WriteG(int fd, char *str, int len){
        memset(bufG, 0, sizeof(bufG));
        sprintf(bufG, "%s", str);
        write(fd, bufG, len);
}

char *ReadG(int fd, int len){
        memset(bufG, 0, sizeof(bufG));
        read(fd, bufG, len);

        return(bufG);
}
```

在此设计中, 父子程序需要约定好每次发送数据的长度; 且长度不能超过255个字符.

#### 显式长度方式

```
char bufC[255];
void WriteC(int fd, char str[]){
        sprintf(bufC, "%04d%s", strlen(str), str);
        write(fd, bufC, strlen(bufC));
}

char *ReadC(int fd){
        int i, j;

        memset(bufC, 0, sizeof(bufC));
        j = read(fd, bufC, 4);
        i = atoi(bufC);
        j = read(fd, bufC, i);

        return(bufC);
}
```

父子进程约定在发送消息前先指明消息的长度.

#### 主程序

```
#include <unistd.h>
#include <stdio.h>
#include <assert.h>
#include <sys/types.h>

int main()
{
        int fildes1[2], fildes2[2];
        pid_t pid;
        char buf[255];

        assert(pipe(fildes1) == 0);
        assert(pipe(fildes2) == 0);
        assert((pid = fork()) >= 0);

        if(pid == 0){
                close(fildes1[1]);
                close(fildes2[0]);
                strcpy(buf, ReadG(fildes1[0], 10));
                fprintf(stderr, "[child] buf = [%s]\n", buf);
                WriteC(fildes2[1], buf);
                strcpy(buf, ReadG(fildes1[0], 10));
                fprintf(stderr, "[child] buf = [%s]\n", buf);
                WriteC(fildes2[1], buf);
                return(0);
        }

        close(fildes1[0]);
        close(fildes2[1]);
        WriteG(fildes1[1], "hello!", 10);
        WriteG(fildes1[1], "world!", 10);
        fprintf(stderr, "[father] buf = [%s] \n", ReadC(fildes2[0]));
        fprintf(stderr, "[father] buf = [%s] \n", ReadC(fildes2[0]));

        return 0;
}
```

执行结果如下:

```
[bill@billstone Unix_study]$ make pipe2
cc     pipe2.c   -o pipe2
[bill@billstone Unix_study]$ ./pipe2
[child] buf = [hello!]
[child] buf = [world!]
[father] buf = [hello!]
[father] buf = [world!]
[bill@billstone Unix_study]$
```

#### popen模型

从前面的程序可以看出, 创建连接标准I/O的管道需要多个步骤, 这需要使用大量的代码, UNIX为了简化这个操作, 它提供了一组函数实现之. 原型如下:

```
#include <stdio.h>
FILE *popen(const char *command, char *type);
int pclose(FILE *stream);
```

函数`popen()`调用成功时返回一个标准的I/O的`FILE`文件流, 其读写属性由参数`type`决定.

这里看一个模拟shell命令`ps -ef | grep init`的实例.

```
[bill@billstone Unix_study]$ cat pipe3.c
#include <stdio.h>
#include <assert.h>

int main()
{
        FILE *out, *in;
        char buf[255];

        assert((out = popen("grep init", "w")) != NULL);        //  创建写管道流
        assert((in = popen("ps -ef", "r")) != NULL);           // 创建读管道流

        while(fgets(buf, sizeof(buf), in))        // 读取ps -ef的结果
                fputs(buf, out);              // 转发到grep init

        pclose(out);
        pclose(in);

        return 0;
}
[bill@billstone Unix_study]$ make pipe3
cc     pipe3.c   -o pipe3
[bill@billstone Unix_study]$ ./pipe3
root         1     0  0 Apr15 ?        00:00:04 init
bill      1392  1353  0 Apr15 ?        00:00:00 /usr/bin/ssh-agent /etc/X11/xinit/Xclients
bill     14204 14203  0 21:33 pts/0    00:00:00 grep init
[bill@billstone Unix_study]$ ps -ef | grep init
root         1     0  0 Apr15 ?        00:00:04 init
bill      1392  1353  0 Apr15 ?        00:00:00 /usr/bin/ssh-agent /etc/X11/xinit/Xclients
bill     14207  1441  0 21:35 pts/0    00:00:00 grep init
[bill@billstone Unix_study]$
```

读者可以从上面自行比较同Shell命令`ps -ef | grep init`的执行结果.

#### 有名管道FIFO

FIFO可以在整个系统中使用.

在Shell中可以使用`mknod`或者`mkfifo`命令创建管道; 而在C程序中, 可以使用`mkfifo()`函数创建有名管道.

要使用有名管道, 需要下面几个步骤:

1. 创建管道文件
2. 在某个进程中以只写方式打开管道文件, 并写管道
3. 在某个进程中以只读方式打开管道文件, 并读管道
4. 关闭管道文件.

低级文件编程库和标准文件编程库都可以操作管道. 管道在执行读写操作之前, 两端必须同时打开, 否则执行打开管道某端操作的进程将一直阻塞到某个进程以相反方向打开管道为止.

下面是一个简单的实例.

首先是写进程: 创建FIFO文件, 再打开写端口, 然后读取标准输入并将输入信息发送到管道中, 当键盘输入`exit`或`quit`时程序退出.

```
[bill@billstone Unix_study]$ cat fifo1.c
#include <stdio.h>
#include <assert.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <sys/errno.h>
extern int errno;

int main()
{
        FILE *fp;
        char buf[255];

        assert((mkfifo("myfifo", S_IFIFO|0666) > 0) || (errno == EEXIST));

        while(1){
                assert((fp = fopen("myfifo", "w")) != NULL);
                printf("please input: ");
                fgets(buf, sizeof(buf), stdin);
                fputs(buf, fp);
                fclose(fp);
                if(strncmp(buf, "quit", 4) == 0 || strncmp(buf, "exit", 4) == 0)
                        break;
        }

        return 0;
}
[bill@billstone Unix_study]$ make fifo1
cc     fifo1.c   -o fifo1
[bill@billstone Unix_study]$
```

然后是读进程: 打开管道的读端口, 从管道中读取信息(以行为单位), 并将此信息打印到屏幕上. 当读取到`exit`或者`quit`时程序退出.

```
[bill@billstone Unix_study]$ cat fifo2.c
#include <stdio.h>
#include <assert.h>
#include <sys/types.h>
#include <sys/stat.h>

int main()
{
        FILE *fp;
        char buf[255];

        while(1){
                assert((fp = fopen("myfifo", "r")) != NULL);
                fgets(buf, strlen(buf), fp);     // 三楼解答
                printf("gets: [%s]", buf);
                fclose(fp);
                if(strncmp(buf, "quit", 4) == 0 || strncmp(buf, "exit", 4) == 0)
                        break;
        }

        return 0;
}
[bill@billstone Unix_study]$ make fifo2
cc     fifo2.c   -o fifo2
[bill@billstone Unix_study]$
```

在一个终端上执行`fifo1`, 而在另一个终端上执行`fifo2`.

我们先输入`hello`, `world`, 然后再输入`exit`退出:
