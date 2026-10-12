
# 《精通 UNIX 下 C 语言编程及项目实践》学习笔记 04 进程通信篇

> 原文：[`www.pediy.com/kssd/pediy10/86832.html`](https://www.pediy.com/kssd/pediy10/86832.html)

欢迎转载，请保留作者信息

`bill@华中科技大学`

`http://billstone.cublog.cn`

《精通 UNIX 下 C 语言编程及项目实践》学习笔记目录

[`bbs.pediy.com/showthread.php?t=86392`](http://bbs.pediy.com/showthread.php?t=86392)

## 第四篇 进程通信篇

### 第十一章 管道

管道是 UNIX 中最古老的进程间通信工具，它提供了进程之间的一种单向通信的方法。

管道分为无名管道和有名管道(FIFO)两种，前者在父子进程中流行，后者由于可以独立成磁盘文件而存在，因而能够被无血缘关系的进程所共享。

#### 无名管道

无名管道占用两个文件描述符，不能被非血缘关系的进程所共享，一般应用在父子进程中。

在 UNIX 中，采用函数 `pipe` 创建无名管道，原型如下：

```
#include <unistd.h>
int pipe(int fildes[2]);
```

我们一般约定 `fildes[1]`描述管道的输入端，进程向此文件描述符中写入数据，`fildes[0]`描述管道的输出端，进程从此文件描述符中读取数据。

无名管道常用于父子进程中，可简单分为单向管道流模型和双向管道流模型。其中，单向管道流根据流向分为从父进程流向子进程的管道和从子进程流向父进程的管道。

下面设计一个实例，数据从父进程流向子进程：父进程向管道写入一行字符，子进程读取数据并打印到屏幕上。

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
                close(fildes[1]);                 // 子进程关闭管道输出
                memset(buf, 0, sizeof(buf));
                j = read(fildes[0], buf, sizeof(buf));
                fprintf(stderr, "[child] buf=[%s] len[%d]\n", buf, j);
                return;
        }
        close(fildes[0]);                      // 父进程关闭管道输入
        write(fildes[1], "hello!", strlen("hello!"));
        write(fildes[1], "world!", strlen("world!"));

        return 0;
}
[bill@billstone Unix_study]$ make pipe1
cc     pipe1.c   -o pipe1
[bill@billstone Unix_study]$ ./pipe1
[child] buf=[hello!world!] len[12]                   // 子进程一次就可以读出两次父进程写入的数据
[bill@billstone Unix_study]$
```

从上面可以看出，在父进程中关闭 `fildes[0]`，向 `fildes[1]`写入数据；在子进程中关闭 `filedes[1]`，从 `fildes[0]`中读取数据可实现从父进程流向子进程的管道。

在进程的通信中，我们无法判断每次通信中报文的字节数，即无法对数据流进行自行拆分，侧耳发生了上例中子进程一次性读取父进程两次通信的报文情况。

管道是进程之间的一种单向交流方法，要实现进程间的双向交流，就必须通过两个管道来完成。双向管道流的创立过程如下：

(1) 创建管道，返回两个无名管道文件描述符 `fildes1` 和 `fildes2`：

(2) 创建子进程，子进程中继承管道 `fildes1` 和 `fildes2`。

(3) 父进程关闭只读文件描述符 `fildes1[0]`，只写描述符 `fildes2[1]`

(4) 子进程关闭只写文件描述符 `fildes1[1]`，只读描述符 `fildes2[0]`

创建的结果如下：

父进程 --写--> `fildes1[1]` --管道--> `fildes1[0]` --读--> 子进程

父进程 <--读-- `fildes2[0]` <--管道-- `fildes2[1]` <--写-- 子进程

这里实现一个父子进程间双向通信的实例：父进程先向子进程发送两次数据，再接收子进程传送刚来的两次数据。为了正确拆分时间留从父进程流向子进程的管道采用'固定长度'方法传送数据；从子进程流向父进程的管道采用'显式长度'方法传回数据。

##### 固定长度方式

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

在此设计中，父子程序需要约定好每次发送数据的长度；且长度不能超过 255 个字符。

##### 显式长度方式

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

父子进程约定在发送消息前先指明消息的长度。

##### 主程序

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

执行结果如下：

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

#### popen 模型

从前面的程序可以看出，创建连接标准 I/O 的管道需要多个步骤，这需要使用大量的代码，UNIX 为了简化这个操作，它提供了一组函数实现之。原型如下：

```
#include <stdio.h>
FILE *popen(const char *command, char *type);
int pclose(FILE *stream);
```

函数 `popen` 调用成功时返回一个标准的 I/O 的 `FILE` 文件流，其读写属性由参数 `type` 决定。

这里看一个模拟 shell 命令'`ps -ef | grep init`'的实例。

```
[bill@billstone Unix_study]$ cat pipe3.c
#include <stdio.h>
#include <assert.h>

int main()
{
        FILE *out, *in;
        char buf[255];

        assert((out = popen("grep init", "w")) != NULL);        //   创建写管道流
        assert((in = popen("ps -ef", "r")) != NULL);           // 创建读管道流

        while(fgets(buf, sizeof(buf), in))        // 读取 ps -ef 的结果
                fputs(buf, out);           // 转发到 grep init

        pclose(out);
        pclose(in);

        return 0;
}
[bill@billstone Unix_study]$ make pipe3
cc     pipe3.c   -o pipe3
[bill@billstone Unix_study]$ ./pipe3
root         1     0  0 Apr15 ?        00:00:04 init
bill     1392  1353  0 Apr15 ?        00:00:00 /usr/bin/ssh-agent /etc/X11/xinit/Xclients
bill     14204 14203  0 21:33 pts/0    00:00:00 grep init
[bill@billstone Unix_study]$ ps -ef | grep init
root         1     0  0 Apr15 ?        00:00:04 init
bill     1392  1353  0 Apr15 ?        00:00:00 /usr/bin/ssh-agent /etc/X11/xinit/Xclients
bill     14207  1441  0 21:35 pts/0    00:00:00 grep init
[bill@billstone Unix_study]$
```

读者可以从上面自行比较同 Shell 命令'`ps -ef | grep init`'的执行结果。

#### 有名管道 FIFO

FIFO 可以在整个系统中使用。

在 Shell 中可以使用 `mknod` 或者 `mkfifo` 命令创建管道；而在 C 程序中，可以使用 `mkfifo` 函数创建有名管道。

要使用有名管道，需要下面几个步骤：

(1) 创建管道文件

(2) 在某个进程中以只写方式打开管道文件，并写管道

(3) 在某个进程中以只读方式打开管道文件，并读管道

(4) 关闭管道文件。

低级文件编程库和标准文件编程库都可以操作管道。管道在执行读写操作之前，两端必须同时打开，否则执行打开管道某端操作的进程将一直阻塞到某个进程以相反方向打开管道为止。

下面是一个简单的实例。

首先是写进程：创建 FIFO 文件，再打开写端口，然后读取标准输入并将输入信息发送到管道中，当键盘输入'`exit`'或'`quit`'时程序退出。

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

然后是读进程：打开管道的读端口，从管道中读取信息(以行为单位)，并将此信息打印到屏幕上。当读取到'`exit`'或者'`quit`'时程序退出。



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

  在一个终端上执行 `fifo1`，而在另一个终端上执行 `fifo2`。

  我们先输入 `'hello'`、`'world'`，然后再输入 `'exit'` 退出:

  代码:

```
[bill@billstone Unix_study]$ ./fifo1
please input: hello
please input: world
please input: exit
[bill@billstone Unix_study]$
```

  我们可以看到读出结果如下:

  代码:

```
[bill@billstone Unix_study]$ ./fifo2
gets: [hello
]gets: [world]gets: [exit][bill@billstone Unix_study]$
```

  看到上面的输出结果，您可能认为是我写错了。其实不是的，读出结果正是如此。其实按照我们的本意，正确的输出结果应该是这样的:

  代码:

```
[bill@billstone Unix_study]$ ./fifo2
gets: [hello
]gets: [world
]gets: [exit
][bill@billstone Unix_study]$
```

  那么到底是什么原因导致了那样的输出呢？我现在也不知道。如果有知晓个中原因的朋友说一下。

《精通 UNIX 下 C 语言编程及项目实践》学习笔记目录

[`bbs.pediy.com/showthread.php?t=86392`](http://bbs.pediy.com/showthread.php?t=86392)

欢迎转载，请保留作者信息

bill@华中科技大学

[`billstone.cublog.cn`](http://billstone.cublog.cn)

### 十二 消息队列

消息队列是 `UNIX` 内核中的一个先进先出的链表结构。相对于管道，消息队列有明显的优势，原因在于:

1. 消息队列是一种先进先出的队列型数据结构，可以保证先送的货物先到达，后送的货物后到达，避免了插队现象。
2. 信息队列将输出的信息进行了打包处理，这样就可以保证以每个消息为单位进行接收了。
3. 消息队列还可以对信息进行分类处理，标记各种类别的信息，这样就可以根据信息类别分别出列。

`IPC` 就是进程间通信，狭义上讲，`IPC` 指消息队列、信号量和共享内存三种对象。通过 `shell` 命令 `ipcs` 可以查询当前系统的 `IPC` 对象信息:

代码:

```
[bill@billstone Unix_study]$ ipcs
 
------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status
0x00000000 196609     bill       777        393216     2          dest
0x00000000 491522     root       644        106496     2          dest
0x00000000 524291     root       644        110592     2          dest
0x00000000 557060     root       644        110592     2          dest
0x00000000 589829     root       644        110592     2          dest
 
------ Semaphore Arrays --------
key        semid      owner      perms      nsems
 
------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages
 
[bill@billstone Unix_study]$
```

#### 消息队列简介

`UNIX` 内核使用结构 `msqid_ds` 来管理消息队列:

代码:

```
struct msqid_ds {
        struct ipc_perm msg_perm;
        struct msg *msg_first;          /* first message on queue, unused  */
        struct msg *msg_last;           /* last message in queue, unused */
        __kernel_time_t msg_stime;      /* last msgsnd time */
        __kernel_time_t msg_rtime;      /* last msgrcv time */
        __kernel_time_t msg_ctime;      /* last change time */
        unsigned long  msg_lcbytes;     /* Reuse junk fields for 32 bit */
        unsigned long  msg_lqbytes;     /* ditto */
        unsigned short msg_cbytes;      /* current number of bytes on queue */
        unsigned short msg_qnum;        /* number of messages in queue */
        unsigned short msg_qbytes;      /* max number of bytes on queue */
        __kernel_ipc_pid_t msg_lspid;   /* pid of last msgsnd */
        __kernel_ipc_pid_t msg_lrpid;   /* last receive pid */
};
```

其中 `msg` 结构的定义如下，我们在实际项目中几乎很少使用如下的结构，一般都需要我们根据实际的需求自行定义。

代码:

```
struct msg{
    struct msg* msg_next;
    long msg_type;
    long msg_ts;
    short msg_spot;
};
```

理论上可以通过结构 `msqid_ds` 的成员 `msg_first`、`msg_last` 和结构 `msg` 的成员 `msg_next` 遍历全部消息队列并完成管理和维护消息队列的功能，但实际上这三个成员是 `UNIX` 内核的直辖数据，用户无法引用。

消息队列中消息本身是由消息类型和消息数据组成，我们常常使用如下结构作为消息模板:

代码:

```
struct msgbuf {
        long mtype;         /* type of message */
        char mtext[1];      /* message text */
};
```

根据消息类型的不同，我们可以在同一个信息队列中定义不同功能的消息。

#### 使用消息队列

##### (1) 消息队列的创建

在 `UNIX` 中，采用函数 `msgget` 创建消息队列

代码:

```
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
extern int msgget (key_t __key, int __msg***) __THROW;
```

函数 `msgget` 创建一个新的消息队列，或者访问一个已经存在的消息队列。调用成功时返回消息队列的标志符，否则返回 `-1`。

##### (2) 消息队列的发送和接收

在 `UNIX` 中函数 `msgsnd` 向消息队列发送消息，原型如下:

代码:

```
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
extern int msgsnd (int __msqid, __const void *__msgp, size_t __msgsz, int __msg***) __THROW;
```

发送消息一般分五个步骤:

a) 根据自己的需要定义消息结构

代码:

```
struct msgbuf {
        long mtype;         /* type of message */
        char ctext[100];      /* message data */
};
```

b) 打开或创建消息队列

代码:

```
msgid = msgget(Key, 0666 | IPC_CREAT);
if(msgid < 0) 打开(或创建)队列失败
```

c) 组装消息

d) 发送消息

代码:

```
int ret;
ret = msgsnd(msgid, (void *)&buf, strlen(buf.ctext), IPC_NOWAIT);
```

e) 发送判断

代码:

```
If (ret == -1){
    if (errno == EINTR) 信号中断, 重新发送;
    else 系统错误
}
```

下面是一个发送消息的实例: 它循环读取键盘输入，将输入的字符串信息写入到消息队列(关键字为 `0x1234`)中。

代码:

```
[bill@billstone Unix_study]$ cat msg1.c
#include <sys/msg.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <stdio.h>
#include <sys/errno.h>
 
extern int errno;
 
struct mymsgbuf{
        long mtype;
        char ctext[100];
};
 
int main(void)
{
        struct mymsgbuf buf;
        int msgid;
 
        if((msgid = msgget(0x1234, 0666 | IPC_CREAT)) < 0){
                fprintf(stderr, "open msg %X failed\n", 0x1234);
                exit(1);
        }
        while(1){
                printf("Please input: ");
                memset(&buf, 0, sizeof(buf));
                fgets(buf.ctext, sizeof(buf.ctext), stdin);
                buf.mtype = getpid();
                while((msgsnd(msgid, (void *)&buf, strlen(buf.ctext), IPC_NOWAIT)) < 0){
                        if(errno == EINTR)
                                continue;
                        exit(2);
                }
                if(!strncmp(buf.ctext, "exit", 4))
                        break;
        }
 
        return 0;
}
[bill@billstone Unix_study]$
```

运行结果如下:

代码:

```
[bill@billstone Unix_study]$ make msg1
cc     msg1.c   -o msg1
[bill@billstone Unix_study]$ ipcs -q
 
------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages
 
[bill@billstone Unix_study]$ ./msg1
Please input: Hello world!
Please input: Nice to meet you!
Please input: bye!
Please input: exit
[bill@billstone Unix_study]$ ipcs -q
 
------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages
0x00001234 98304      bill       666        41           4
 
[bill@billstone Unix_study]$
```

在 `UNIX` 中函数 `msgrcv` 从消息队列中接收消息，原型如下:

代码:

```
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
extern int msgrcv (int __msqid, void *__msgp, size_t __msgsz, long int __msgtyp, int __msg***) __THROW;
```

与函数 `msgsnd` 不同，这里参数 `msgsz` 指的是缓冲区的最大容量，包括消息类型占用的部分。

这里配合上面的例子设计接收消息的实例: 以阻塞方式不断地从消息队列(关键字为 `0x1234`)中读取消息，并打印接收到的消息类型、长度和数据等信息，当接收到数据内容为 `'exit'` 时程序结束。

代码:



```shell
[bill@billstone Unix_study]$ cat msg2.c
#include <sys/msg.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <stdio.h>
#include <sys/errno.h>
 
extern int errno;
 
struct mymsgbuf{
        long mtype;
        char ctext[100];
};
 
int main(void)
{
        struct mymsgbuf buf;
        int msgid, ret;
 
        if((msgid = msgget(0x1234, 0666 | IPC_CREAT)) < 0){
                fprintf(stderr, "open msg %X failed\n", 0x1234);
                exit(1);
        }
        while(1){
                memset(&buf, 0, sizeof(buf));
                while((ret = msgrcv(msgid, (void *)&buf, sizeof(buf), 0, 0)) < 0){
                        if(errno == EINTR)
                                continue;
                        fprintf(stderr, "Error no: %d", errno);
                        exit(2);
                }
                fprintf(stderr, "Msg: Type=%d, Len=%d, Text:%s", buf.mtype, ret, buf.ctext);
                if(!strncmp(buf.ctext, "exit", 4))
                        break;
        }
 
        return 0;
}
[bill@billstone Unix_study]$
```

运行结果如下：

```shell
[bill@billstone Unix_study]$ make msg2
cc     msg2.c   -o msg2
[bill@billstone Unix_study]$ ipcs -q
 
------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages
0x00001234 98304      bill       666        41           4
 
[bill@billstone Unix_study]$ ./msg2
Msg: Type=15666, Len=13, Text:Hello world!
Msg: Type=15666, Len=18, Text:Nice to meet you!
Msg: Type=15666, Len=5, Text:bye!
Msg: Type=15666, Len=5, Text:exit
[bill@billstone Unix_study]$ ipcs -q
 
------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages
0x00001234 98304      bill       666        0            0
 
[bill@billstone Unix_study]$
```

从上面可以看到，采用消息队列通信比采用管道通信具有更多的灵活性。

系统调用 `msgctl` 对消息队列进行各种控制，包括查询消息队列数据结构，改变消息队列访问权限，改变消息队列属主信息和删除消息队列等，原型如下：

```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
int msgctl(int msqid, int cmd, struct msqid_ds *buf);
```

根据 `cmd` 参数对 `msqid` 消息队列操作：

a) `IPC_RMID`: 删除消息队列

b) `IPC_STAT`: 读取消息队列

c) `IPC_SET`: 重置消息队列结构 `msqid_ds` 中的成员 `uid`, `gid`, `mode` 及 `msg_qbytes`.

欢迎转载，请保留作者信息

bill@华中科技大学

http://billstone.cublog.cn

### 十三章 信号量

进程间的通信不仅仅包括数据交流，也包括过程控制。

信号量是一个可以用来控制进程存储共享资源的计数器，它可以是跟踪共享资源的生产和消费的计数器，也可以是协调资源的生产者和消费者之间的同步器，还可以是控制生产进程和消费进程的互斥开关。

#### 信号量简介

操作系统通过信号量和 `PV` 操作，可以完成同步和互斥操作。

信号量集合由一个或多个信号量集合组成，`IPC` 对象中的“信号量”通常指的是信号量集合，`UNIX` 的内核采用结构 `semid_ds` 来管理信号量，结构如下：

```c
struct semid_ds {
        struct ipc_perm sem_perm;                 /* permissions .. see ipc.h */
        __kernel_time_t sem_otime;                /* last semop time */
        __kernel_time_t sem_ctime;                /* last change time */
        struct sem      *sem_base;                /* ptr to first semaphore in array */
        struct sem_queue *sem_pending;            /* pending operations to be processed */
        struct sem_queue **sem_pending_last;      /* last pending operation */
        struct sem_undo *undo;                    /* undo requests on this array */
        unsigned short  sem_nsems;                /* no. of semaphores in array */
};
```

指针 `sem_base` 指向一个信号量数组，信号量由结构 `sem` 记载，如下所示：

```c
Struct sem{
    unsigned short semval;       // 信号量取值
    pid_t sempid;               // 最近访问进程 ID
    unsigned short semncnt;      // P 阻塞进程数
    unsigned short semzcnt;      // Z 阻塞进程数
}
```

在 `Shell` 中可以通过 `ipcs -a -s` 命令查询系统中信号量的基本信息。

##### (1) 信号量的创建

在 `UNIX` 中，函数 `semget` 用来创建信号量，原型如下：

```c
#include <sys/sem.h>
int semget(key_t key, int nsems, int sem***);
```

函数 `semget` 创建一个新的信号量，或者访问一个已经存在的信号量。

##### (2) 信号量的控制

系统调用 `semctl` 用来控制信号量，原型如下：

```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>
int semctl(int semid, int semnum, int cmd, ...);
```

函数 `semctl` 对标识号为 `semid` 的信号量集合中序号为 `semnum` 的信号量进行赋值，初始化，信息获取和删除等多相操作，参数 `cmd` 指定了操作的类型，参数 `arg` 指定了函数输入输出的缓冲区，定义如下：

```c
union semun {
        int val;                              /* value for SETVAL */
        struct semid_ds *buf;                 /* buffer for IPC_STAT & IPC_SET */
        unsigned short *array;                /* array for GETALL & SETALL */
        struct seminfo *__buf;                /* buffer for IPC_INFO */
        void **__pad;
};
```

函数 `semctl` 的第四个参数 `arg` 在本质上是一个 4 字节的缓冲区。调用失败时返回 -1 并置 `errno`。

本处设计一个类似于命令 `ipcs` 和命令 `ipcrm` 的程序 `ipcsem`，它从命令行参数中获取要执行的操作，包括创建信号量，读取信号量信息，读取信号量取值和删除信号量等，程序如下：

```c
#include <sys/sem.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/stat.h>
#include <stdio.h>
 
#define VerifyErr(a, b)  \\
        if (a) fprintf(stderr, "%s failed.\n", (b));  \\
        else fprintf(stderr, "%s success.\n", (b));
 
int main(int argc, char *argv[1])
{
       int semid, index, i;
        unsigned short array[100];
        struct semid_ds ds;
        
        if(argc != 4)
                 return 0;
        semid = atoi(argv[1]);
        index = atoi(argv[2]);
        if(argv[3][0] == 'c'){
                 VerifyErr(semget(semid, index, 0666|IPC_CREAT|IPC_EXCL) < 0, "Create sem");
        }
        else if(argv[3][0] == 'd'){
                 VerifyErr(semctl(semid, 0, IPC_RMID, NULL) < 0, "Delete sem");
        }
        else if(argv[3][0] == 'v'){
                 fprintf(stderr, "T    ID    INDEX    SEMVAL    SEMIPID    SEMNCNT    SEMZCNT\n");
                 fprintf(stderr, "s %6d %6d %10d %10d %10d %10d\n", semid, index,
                         semctl(semid, index, GETVAL), semctl(semid, index, GETPID),
                         semctl(semid, index, GETNCNT), semctl(semid, index, GETZCNT));
        }
        else if(argv[3][0] == 'a'){
                 ds.sem_nsems = 0;
                 VerifyErr(semctl(semid, 0, IPC_STAT, &ds) != 0, "Get Sem Stat");
                 VerifyErr(semctl(semid, 0, GETALL, array) != 0, "Get Sem All");
                 for(i=0;i<ds.sem_nsems;i++)
                         fprintf(stderr, "sem no [%d]: [%d]\n", i, array[i]);
        }
        else
                 VerifyErr(semctl(semid, index, SETVAL, atoi(argv[3])) != 0, "Set Sem Val");
 
        return 0;
}
```

执行结果如下：

```shell
[bill@billstone Unix_study]$ make ipcsem
cc     ipcsem.c   -o ipcsem
[bill@billstone Unix_study]$ ipcs -s
 
------ Semaphore Arrays --------
key        semid      owner      perms      nsems
0x000003e8 0          bill       666        10
 
[bill@billstone Unix_study]$ ./ipcsem 2000 2 c
Create sem success.
[bill@billstone Unix_study]$ ipcs -s
 
------ Semaphore Arrays --------
key        semid      owner      perms      nsems
0x000003e8 0          bill       666        10
0x000007d0 65537      bill       666        2
 
[bill@billstone Unix_study]$ ./ipcsem 65537 0 100
Set Sem Val success.
[bill@billstone Unix_study]$ ./ipcsem 65537 0 v
T    ID    INDEX    SEMVAL    SEMIPID    SEMNCNT    SEMZCNT
s  65537      0        100      23829          0          0
[bill@billstone Unix_study]$ ./ipcsem 65537 1 200
Set Sem Val success.
[bill@billstone Unix_study]$ ./ipcsem 65537 0 a
Get Sem Stat success.
Get Sem All success.
sem no [0]: [100]
sem no [1]: [200]
[bill@billstone Unix_study]$ ./ipcsem 65537 0 d
Delete sem success.
[bill@billstone Unix_study]$ ipcs -s
 
------ Semaphore Arrays --------
key        semid      owner      perms      nsems
0x000003e8 0          bill       666        10
 
[bill@billstone Unix_study]$
```

#### 操作信号量

信号量具有 `P`, `V` 和 `Z` 三种操作，在 `UNIX` 中，这些操作可以通过函数 `semop` 调用完成，函数 `semop` 可以一次性操作同一信号量集合中的多个信号量。原型如下：

```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>
int semop(int semid, struct sembuf *sops, unsigned nsops);
```

函数 `semop` 对标识号为 `semid` 的信号量集合中的一个或多个信号量执行信号数值的增加，减少或比较操作。参数 `sops` 指向一个 `sembuf` 结构的缓冲区，`nsops` 指定了缓冲区中存储的 `sembuf` 结构的个数。

```c



# 操作系统 IPC 机制与共享内存

## 信号量 (Semaphores)

```c
struct sembuf {
        unsigned short  sem_num;        /* semaphore index in array */
        short           sem_op;         /* semaphore operation */
        short           sem_flag;       /* operation flags */
};
```

其中，第一个信号的序号是 `0`。`sem_op` 指定了操作的类型：

- 正数：V 操作。
- 负数：P 操作。
- 0：Z 操作。判断信号量数值是否等于 0。

而 `sem_flag` 取值有 `IPC_NOWAIT` 和 `SEM_UNDO` 等。

下面是一个用于临界资源的读写控制和并发进程的同步和互斥控制的实例：假设进程 A 是生产者，进程 B 是消费者，系统最多只能同时容纳 5 个产品，初始成品数为 0。当产品数不足 5 时允许进程 A 生产，当产品数超过 0 时允许进程 B 消费。

这里需要两个信号量模拟生产-消费过程。信号量 A 代表了当前生产的数目，它控制了生产者进程 A，信号量 n 代表当前尚有 n 个成品可以生产。

信号 B 代表了当前的产品数，他控制消费者进程 B，当信号量为 n 时剩余 n 个产品。

### 生产者进程

生产者进程 `sema.c` 如下：

```c
[ bill@billstone Unix_study]$ cat sema.c
#include <sys/sem.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <stdio.h>
#include <sys/stat.h>

#define VerifyErr(a,b) \
        if (a) { fprintf(stderr, "%s failed.\n", (b)); exit(1); } \
        else fprintf(stderr, "%s success.\n", (b));

int main(void)
{
        int semid;
        struct sembuf sb;

        VerifyErr((semid = semget(2000, 2, 0666)) < 0, "Open Sem 2000");
        sb.sem_num = 0;
        sb.sem_op = -1;
        sb.sem_flag &= ~IPC_NOWAIT;
        VerifyErr(semop(semid, &sb, 1) != 0, "P sem 2000:0");
        fprintf(stderr, "[%d] producing ... ...\n", getpid());
        sleep(1);
        fprintf(stderr, "[%d] produced\n", getpid());
        sb.sem_num = 1;
        sb.sem_op = 1;
        sb.sem_flag &= ~IPC_NOWAIT;
        VerifyErr(semop(semid, &sb, 1) != 0, "V sem 2000:1");

        return 0;
}
[ bill@billstone Unix_study]$
```

### 消费者进程

消费者进程 `semb.c` 如下：

```c
[ bill@billstone Unix_study]$ cat semb.c
#include <sys/sem.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <stdio.h>
#include <sys/stat.h>

#define VerifyErr(a,b) \
        if (a) { fprintf(stderr, "%s failed.\n", (b)); exit(1); } \
        else fprintf(stderr, "%s success.\n", (b));

int main(void)
{
        int semid;
        struct sembuf sb;

        VerifyErr((semid = semget(2000, 2, 0666)) < 0, "Open Sem 2000");
        sb.sem_num = 1;
        sb.sem_op = -1;
        sb.sem_flag &= ~IPC_NOWAIT;
        VerifyErr(semop(semid, &sb, 1) != 0, "P sem 2000:1");
        fprintf(stderr, "[%d] consuming ... ...\n", getpid());
        sleep(1);
        fprintf(stderr, "[%d] consumed\n", getpid());
        sb.sem_num = 0;
        sb.sem_op = 1;
        sb.sem_flag &= ~IPC_NOWAIT;
        VerifyErr(semop(semid, &sb, 1) != 0, "V sem 2000:1");

        return 0;
}
[ bill@billstone Unix_study]$
```

### 编译与测试

编译程序并使用之前的程序 `ipcsem` 创建信号量集合：

```bash
[ bill@billstone Unix_study]$ ./ipcsem 2000 2 c
Create sem success.
[ bill@billstone Unix_study]$ ipcs -s

------ Semaphore Arrays --------
key        semid     owner      perms      nsems
0x000003e8 0         bill       666        10
0x000007d0 98305     bill       666        2

[ bill@billstone Unix_study]$ ./ipcsem 98305 0 5
Set Sem Val success.
[ bill@billstone Unix_study]$ ./ipcsem 98305 1 0
Set Sem Val success.
[ bill@billstone Unix_study]$ ./ipcsem 98305 0 a
Get Sem Stat success.
Get Sem All success.
sem no [0]: [5]
sem no [1]: [0]
[ bill@billstone Unix_study]$
```

在一个终端上运行 `sema`：

```bash
[ bill@billstone Unix_study]$ ./ipcsem 98305 0 a
Get Sem Stat success.
Get Sem All success.
sem no [0]: [3]
sem no [1]: [2]
[ bill@billstone Unix_study]$ ./sema
Open Sem 2000 success.
P sem 2000:0 success.
[23940] producing ... ...
[23940] produced
V sem 2000:1 success.
[ bill@billstone Unix_study]$ ./ipcsem 98305 0 a
Get Sem Stat success.
Get Sem All success.
sem no [0]: [2]
sem no [1]: [3]
[ bill@billstone Unix_study]$
```

在另一个终端上执行 `semb`：

```bash
[ bill@billstone Unix_study]$ ./ipcsem 98305 0 a
Get Sem Stat success.
Get Sem All success.
sem no [0]: [2]
sem no [1]: [3]
[ bill@billstone Unix_study]$ ./semb
Open Sem 2000 success.
P sem 2000:1 success.
[23942] consuming ... ...
[23942] consumed
V sem 2000:1 success.
[ bill@billstone Unix_study]$ ./ipcsem 98305 0 a
Get Sem Stat success.
Get Sem All success.
sem no [0]: [3]
sem no [1]: [2]
[ bill@billstone Unix_study]$
```

读者可以试着连续执行 `sema` 几次，观察进程的 P 阻塞状态。

---

*欢迎转载，请保留作者信息*
*bill@华中科技大学*
*http://billstone.cublog.cn*

## 共享内存 (Shared Memory)

管道、消息队列和信号量都需要借助第三方对象进行通信；而共享内存正好弥补了这些缺陷，它是最快的 IPC 对象。在本质上，共享内存是一端物理内存。

### 共享内存简介

共享内存中最重要的属性是内存大小和内存地址，进程在访问共享内存前必须先将共享内存映射到进程空间的一个虚拟地址中，然后任何对该虚拟地址的数据操作都将直接作用到物理内存上。

共享内存由进程创建，但是进程结束时共享内存仍然保留，除非该共享内存被显式地删除或者重启操作系统。

UNIX 的内核采用结构 `shmid_ds` 来管理消息队列，它的数据成员与命令 `ipcs -a -m` 的结果一一对应：

```c
struct shmid_ds {
        struct ipc_perm        shm_perm;       /* operation perms */
        int                    shm_segsz;      /* size of segment (bytes) */
        __kernel_time_t        shm_atime;      /* last attach time */
        __kernel_time_t        shm_dtime;      /* last detach time */
        __kernel_time_t        shm_ctime;      /* last change time */
        __kernel_ipc_pid_t     shm_cpid;       /* pid of creator */
        __kernel_ipc_pid_t     shm_lpid;       /* pid of last operator */
        unsigned short         shm_nattch;     /* no. of current attaches */
        unsigned short         shm_unused;     /* compatibility */
        void                   *shm_unused2;   /* ditto - used by DIPC */
        void                   *shm_unused3;   /* unused */
};
```

命令 `ipcs -m` 查询系统中共享内存的基本信息：

```bash
[ bill@billstone bill]$ ipcs -m

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status

[ bill@billstone bill]$
```

### 使用共享内存

1. **共享内存的创建**

在 UNIX 中，函数 `shmget` 用来创建或获取共享内存，原型如下：

```c
#include <sys/ipc.h>
#include <sys/shm.h>
int shmget(key_t key, int size, int shmflg);
```

同样的，参数 `shmflg` 有 `IPC_CREAT` 和 `IPC_EXCL` 两种取值。

2. **共享内存的映射**

与消息队列和信号量不同，共享内存在获取标志号后，仍需要调用 `shmat` 函数将共享内存映射到进程的地址空间后才能访问，原型如下：

```c
#include <sys/types.h>
#include <sys/shm.h>
void *shmat(int shmid, const void *shmaddr, int shmflg);
```

3. **共享内存的释放**

当进程不再需要共享内存时，可以使用函数 `shmdt` 释放共享内存内存映射，原型如下：

```c
#include <sys/types.h>
#include <sys/shm.h>
int shmdt(const void *shmaddr);
```

4. **共享内存的控制**

与消息队列和信号量一样，共享内存也有自给的控制函数 `shmctl`，原型如下：

```c
#include <sys/ipc.h>
#include <sys/shm.h>
int shmctl(int shmid, int cmd, struct shmid_ds *buf);
```

这里设计一个类似于命令 `ipcs` 和命令 `ipcrm` 的程序 `ipcshm`：它从命令行参数中获取要执行的操作，包括创建共享内存，读取共享内存信息和删除共享内存等，主体代码如下：

```c
#include <sys/shm.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <stdio.h>
#include <sys/stat.h>

#define VerifyErr(a, b) \
        if (a) fprintf(stderr, "%s failed.\n", (b)); \
        else fprintf(stderr, "%s success.\n", (b));

int main(int argc, char *argv[])
{
        int shmid, size;

        if(argc != 3)
                 exit(1);
        shmid = atoi(argv[1]);
        if(strcmp(argv[2], "v") == 0){
                 StatShm(shmid);
        }
        else if(strcmp(argv[2], "d") == 0){
                 VerifyErr(shmctl(shmid, IPC_RMID, NULL) < 0, "Delete Shm");
        }
        else{
                 size = atoi(argv[2]);
                 VerifyErr(shmget(shmid, size, 0666|IPC_CREAT|IPC_EXCL) < 0, "Create Shm");
        }

        return 0;
}
```

其中用到了两个函数：



```c
char * GetFileMode(mode_t st_mode, char *resp){
        if(resp == NULL)
                return 0;
        memset(resp, '-', 9);
        if(st_mode & S_IRUSR)   resp[0] = 'r';
        if(st_mode & S_IWUSR)   resp[1] = 'w';
        if(st_mode & S_IXUSR)   resp[2] = 'x';
        if(st_mode & S_IRGRP)   resp[3] = 'r';
        if(st_mode & S_IWGRP)   resp[4] = 'w';
        if(st_mode & S_IXGRP)   resp[5] = 'x';
        if(st_mode & S_IROTH)   resp[6] = 'r';
        if(st_mode & S_IWOTH)   resp[7] = 'w';
        if(st_mode & S_IXOTH)   resp[8] = 'x';
                                                    
        return resp;
}
 
int StatShm(int shmid){
        char resp[10];
        struct shmid_ds buf;
 
        memset(&buf, 0, sizeof(buf));
        memset(resp, 0, sizeof(resp));
        shmctl(shmid, IPC_STAT, &buf);
        fprintf(stderr, "T    ID    KEY    MODE    OWNER    GROUP    NATTCH    SEGSZ    CPID    LPID\n");
    fprintf(stderr, "m %6d %#6x %s %6d %6d %6d %10d %10d %10d\n", shmid, buf.shm_perm.__key,
             GetFileMode(buf.shm_perm.mode, resp), buf.shm_perm.uid, buf.shm_perm.gid,
             buf.shm_nattch, buf.shm_segsz, buf.shm_cpid, buf.shm_lpid);
 
        return 0;
}
```

执行情况如下:

代码:

```bash
[bill@billstone Unix_study]$ gcc -o ipcshm ipcshm.c
[bill@billstone Unix_study]$ ./ipcshm 2000 100
Create Shm success.
[bill@billstone Unix_study]$ ipcs -m
 
------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status
0x000007d0 229377     bill       666        100        0
 
[bill@billstone Unix_study]$ ./ipcshm 229377 v
T    ID    KEY    MODE    OWNER    GROUP    NATTCH    SEGSZ    CPID    LPID
m 229377  0x7d0 rw-rw-rw-    500    500      0        100       1796          0
[bill@billstone Unix_study]$ ./ipcshm 229377 d
Delete Shm success.
[bill@billstone Unix_study]$ ipcs -m
 
------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status
 
[bill@billstone Unix_study]$
```

(5) 共享内存实例

共享内存的应用可以分为打开(创建)共享内存, 映射共享内存, 读写共享内存和释放共享内存映射等四个步骤.

程序`shm1`向共享内存中指定位置写入数据.

代码:

```c
#include <sys/shm.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <stdio.h>
#include <sys/stat.h>
 
#define VerifyErr(a, b) \
        if (a) { fprintf(stderr, "%s failed.\n", (b)); return; } \
        else fprintf(stderr, "%s success.\n", (b));
 
int main(void)
{
        int shmid, no;
        char *pmat = NULL, buf[1024];
 
        VerifyErr((shmid = shmget(0x1234, 10*1024, 0666|IPC_CREAT)) == -1, "Open(Create) Shm");
        VerifyErr((pmat = (char *)shmat(shmid, 0, 0)) == 0, "Link Shm");
        printf("Please input NO.(0-9): ");
        scanf("%d", &no);
        VerifyErr(no<0 || no>9, "Input No.");
        printf("Please input data: ");
        memset(buf, 0, sizeof(buf));
        getchar();                // 读入'\n'回车符
        fgets(buf, sizeof(buf), stdin);
        memcpy(pmat+no*1024, buf, 1024);
        shmdt(pmat);
 
        return 0;
}
```

程序`shm2`从共享内存指定位置读取数据

代码:

```c
#include <sys/shm.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <stdio.h>
#include <sys/stat.h>
 
#define VerifyErr(a, b) \
        if (a) { fprintf(stderr, "%s failed.\n", (b)); return; } \
        else fprintf(stderr, "%s success.\n", (b));
 
int main(void)
{
        int shmid, no;
        char *pmat = NULL, buf[1024];
 
        VerifyErr((shmid = shmget(0x1234, 10*1024, 0666|IPC_CREAT)) == -1, "Open(Create) Shm");
        VerifyErr((pmat = (char *)shmat(shmid, 0, 0)) == 0, "Link Shm");
        printf("Please input NO.(0-9): ");
        scanf("%d", &no);
        VerifyErr(no<0 || no>9, "Input No.");
        memcpy(buf, pmat+no*1024, 1024);
        printf("Data: [%s]\n", buf);
        shmdt(pmat);
 
        return 0;
}
```

运行结果如下

代码:

```bash
[bill@billstone Unix_study]$ make shm1
cc      shm1.c   -o shm1
[bill@billstone Unix_study]$ make shm2
cc      shm2.c   -o shm2
[bill@billstone Unix_study]$ ./shm1
Open(Create) Shm success.
Link Shm success.
Please input NO.(0-9): 1
Input No. success.
Please input data: this is a test!
[bill@billstone Unix_study]$ ./shm2
Open(Create) Shm success.
Link Shm success.
Please input NO.(0-9): 1
Input No. success.
Data: [this is a test!
]
[bill@billstone Unix_study]$
```
