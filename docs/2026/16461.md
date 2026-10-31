
# 中秋节了，发一个linux下的内存注册机源码，希望大家支持

看见论坛研究破解的不少，但是研究破解linux的很少。

在中秋佳节之季，发一个自己写的linux内存注册机，希望高手能将它改成通用的linux内存注册机吧.

此内存注册机的原理是，调用`ptrace`，在指定的地址处修改被调用进程的指令为`int3`，使其中断，然后查看各寄存器的值，得到自己需要的注册码.

作者：软件路由论坛 1d3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4u0G2N6i4c8W2M7X3y4D9N6h3u0Q4x3X3g2U0L8$3#2Q4c8e0y4Q4z5o6m8Q4z5o6m8Q4c8e0k6Q4z5o6y4Q4b7U0y4Q4c8e0g2Q4b7V1g2Q4z5e0N6Q4c8e0g2Q4b7e0c8Q4b7f1q4Q4c8e0N6Q4b7V1g2Q4z5p5g2Q4c8e0y4Q4z5o6m8Q4z5o6l9`.

一个linux程序，下面为在`ida`中反编译出来的代码，我们需要在`08049658`处设置断点，注册码保存在`[ebp+arg_4]`即`[ebp+12]`中.

```
.text:0804962D                 push    [ebp+var_C]     ; FILE *
.text:08049630                 call    _fclose
.text:08049635                 add     esp, 10h
.text:08049638                 mov     [ebp+var_18], 0
.text:0804963C                 sub     esp, 4
.text:0804963F                 sub     esp, 8
.text:08049642                 push    [ebp+arg_4]     ; char *
.text:08049645                 call    _strlen
.text:0804964A                 add     esp, 0Ch
.text:0804964D                 push    eax             ; size_t
.text:0804964E                 push    [ebp+arg_4]     ; char *  // 注册码的存放地址
.text:08049651                 lea     eax, [ebp+var_98]
.text:08049657                 push    eax             ; char *
.text:08049658                 call    _strncmp
.text:0804965D                 add     esp, 10h
.text:08049660                 test    eax, eax
.text:08049662                 jnz     short loc_8049670
.text:08049664                 mov     [ebp+var_9C], 1
.text:0804966E                 jmp     short loc_804967A
```

以下为内存注册机的源码，本注册机调用原程序，在原程序的`08049658`处设置断点，同时读取`[epb+12]`地址处的值，从而得到注册码：

```
#include <sys/ptrace.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>
#include <linux/user.h>
#include <sys/syscall.h>

unsigned char backup[4],backup1[4],bin[64];
const int long_size = sizeof(long);

void getdata(pid_t child, long addr, char *str, int len)
{   char *laddr;
    int i, j;
    union u {
            long val;
            char chars[long_size];
    }data;
    
    i = 0;
    j = len / long_size;
    laddr = str;
    while(i < j) {
        data.val = ptrace(PTRACE_PEEKTEXT, child, addr + i * 4, NULL);
        memcpy(laddr, data.chars, long_size);
        ++i;
        laddr += long_size;
    }
    
    j = len % long_size;
    if(j != 0) {
        data.val = ptrace(PTRACE_PEEKTEXT, child, addr + i * 4, NULL);
        memcpy(laddr, data.chars, j);
    }
    str[len] = '\0';       
}

void putdata(pid_t child, long addr, char *str, int len)
{   char *laddr;
    int i, j;
    union u {
            long val;
            char chars[long_size];
    }data;
    
    i = 0;
    j = len / long_size;
    laddr = str;
    while(i < j) {
        memcpy(data.chars, laddr, long_size);
        ptrace(PTRACE_POKETEXT, child, addr + i * 4, data.val);
        ++i;
        laddr += long_size;
    }
    
    j = len % long_size;
    if(j != 0) {
        memcpy(data.chars, laddr, j);
        ptrace(PTRACE_POKETEXT, child, addr + i * 4, data.val);
    }
}

int main()
{   pid_t child;
    const int long_size = sizeof(long);

    child = fork();
    if(child == 0) {
        ptrace(PTRACE_TRACEME, 0, NULL, NULL);
        execl("/home/gtr/crackme", "crackme", NULL);
    }
    else {
        int status;
        union u {
            long val;
            char chars[long_size];
        }data;
        struct user_regs_struct regs,regi;
        int start = 0;
        long ins,zz;
        char **a_eax;
        int i,jj=0;
        char code[] = {0xcd,0x80,0xcc,0};  //int3机器码
        unsigned char check[4];
        char str[6]={1,2,3,4,5,6};
        char str1[128]={0,0,0,0,0,0};

        while(1) {
            wait(&status);
            if(WIFEXITED(status))
                break; 
                 
            ptrace(PTRACE_GETREGS, child, NULL, &regs);
            ins = ptrace(PTRACE_PEEKTEXT, child, regs.eip, NULL);
            
            if (jj==0){
            getdata(child, 0x08049658, backup1, 3);
            putdata(child, 0x08049658, code, 3);　　　//修改程序的数据，设置断点
            jj=1;
          }

            if (regs.eip!=0x40000790) {
            getdata(child, regs.ebp+12, str,40);  //读取指定地址处的值
            getdata(child, *(long*)str, str1,128);  //读取指定地址处的值
            printf("EIP地址是: %lx 注册码是：%s\n", regs.eip,str1);
            }  
    }
    return 0;
}
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
