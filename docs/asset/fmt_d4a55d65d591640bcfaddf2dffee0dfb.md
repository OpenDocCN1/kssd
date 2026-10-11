
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
