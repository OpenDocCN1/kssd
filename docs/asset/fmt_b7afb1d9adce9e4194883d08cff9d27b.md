
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
