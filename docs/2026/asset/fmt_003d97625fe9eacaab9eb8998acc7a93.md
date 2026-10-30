
# [原创]Pwn堆利用学习——Fastbin-House of Spirit——`LCTF_2016_pwn200`

House of Spirit（HOS）是一个组合型的漏洞利用，是变量覆盖和堆管理机制的组合利用。是构造一个 fake chunk，然后释放掉它，这样再次申请的时候就会申请它。具体来说，关键在于能够覆盖一个堆指针变量，使其指向可控的区域，只要构造好数据，释放后系统会错误的将该区域作为堆块放到相应的 fastbin 里面，最后再分配出来的时候，就有可能改写我们目标区域。

## 利用场景

1. 想要控制的目标区域的前段空间与后段空间都是可控的内存区域。

一般来说想要控制的目标区域多为返回地址或一个函数指针，正常情况下，该内存区域我们是无法通过输入数据来进行控制的，想要利用 HOS 攻击技术来改写该区域，首先需要我们可以控制那片目标区域的前面空间和后面空间。

```
----------------------------------------------------------------
             可控1
 ----------------------------------------------------------------
         目标区域（不可控，多为函数指针或返回地址等）
 ----------------------------------------------------------------
             可控2
 ----------------------------------------------------------------
```

2. 存在可将堆变量指针覆盖指向为可控区域，即上一步中的区域。

## `free()` 要绕过的检测

1. fake chunk 的 `ISMMAP` 位不能为 1，否则会调用 `munmap_chunk()` 函数去释放堆块。
2. fake chunk 的地址需要对齐 `MALLOC_ALIGN_MASK`。
3. fake chunk 的 size 大小需要满足对应的 fastbin 的需求。
4. `2 * SIZE_SZ < next chunk size < av->system_mem`。
5. fake chunk 对应的 fastbin 链表头部不能是该 fake chunk，即不能构成 double free 的情况。

## `LCTF_2016-pwn200`

### 步骤一：运行查看

![image-20210126163312620](https://bbs.kanxue.com/upload/attach/202103/830671_3BSC8P7VYZGF9ED.jpg)

> LCTF 就是 XDCTF，由西电的 L-Team 主办。

### 步骤二：查看文件类型和保护机制

- 64 位程序
- 除了 `RELRO`，其他保护都关闭

```
$ file lctf_2016_pwn200
lctf_2016_pwn200: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.24, BuildID[sha1]=5a7b9f542c0bf79112b5be3f0198d706cce1bcad, stripped
$ checksec --file=lctf_2016_pwn200
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH Symbols  FORTIFY Fortified Fortifiable FILE
Partial RELRO   No canary found   NX disabled   No PIE          No RPATH   No RUNPATH   No Symbols   No 0  3  lctf_2016_pwn200
```

### 步骤三：IDA 反编译分析

#### a. `main()` 函数

有两个函数，第一个函数就是设置程序的缓冲区，重命名为 `setvbufs()`，不再赘述。第二个函数就是程序的主体了。这里重命名为 `welcome()`。

![image-20210307101225143](https://bbs.kanxue.com/upload/attach/202103/830671_K6M83XJ6GQ788AN.jpg)

#### b. `welcome()` 函数

1）首先打印 “who are you”，然后 `for` 循环以输入进行回答，接着打印输入。

2）这里的 `for` 循环存在 **off-by-one 漏洞**，利用后面的 `printf()` 能 **泄漏出 `welcome()` 栈帧中存储的 old `rbp`**，也就是能泄漏出栈里的某个地址。

3）再接着就是输入 `id` 和输入 `money` 以及菜单操作。

![image-20210307101452898](https://bbs.kanxue.com/upload/attach/202103/830671_BKCR2455K9GHJEZ.jpg)

![image-20210307102026866](https://bbs.kanxue.com/upload/attach/202103/830671_FXVB7FC224Y9UKC.jpg)

因为，这里能泄漏栈空间地址，所以解题和栈有关（学习的时候是上帝视角，知道这里用 HOS，肯定是和栈有关的 - -）。那么根据分析，可得到此时栈空间布局如下：

![image-20210307171003636](https://bbs.kanxue.com/upload/attach/202103/830671_52YBKGCAHV7BD7Z.jpg)

#### c. `input_number()` 函数

将输入转为 `int` 型，然后返回。该函数有返回值，但是上面的 `welcome()` 函数伪代码中却没有变量接收返回值。查看汇编，其实是有的，保存在 `[rbp-0x38]`，只不过这里是 IDA 反编译不准确。

![image-20210307101834173](https://bbs.kanxue.com/upload/attach/202103/830671_QD93RPZESAF4R6X.jpg)

![image-20210307102449586](https://bbs.kanxue.com/upload/attach/202103/830671_PHN435WQFBSSMJX.jpg)

到这的时候，栈空间布局如下：

![image-20210307171427371](https://bbs.kanxue.com/upload/attach/202103/830671_2SEMJMQSCDSJ8UY.jpg)

#### d. `input_money_and_menu()` 函数

1）`malloc()` 了一个 `0x40` 大小的堆空间，并将返回值赋值给指针 `dest`。

2）然后往 `0x38` 大小的栈缓存区 `buf` 输入 “money”，最大大小为 `0x40`，这里存在 **相邻变量覆盖**，覆盖掉的是 `dest`，即这里 **可以覆盖堆指针**。这可以从 `buf == [rbp-40h]` 和 `*dest = [rbp-8h]` 看出，也可以双击变量查看它们的栈结构，这里不再贴图。

3）接着将 `buf` 内容（输入的 `money`）拷贝到 `dest` 指向的堆块中。

4）将 `dest` 赋值给全局指针 `ptr`，即 `ptr` 也指向堆块，当然如果 `dest` 被覆盖了，`ptr` 也就会随之改变。

5）执行菜单操作。

![image-20210307102727628](https://bbs.kanxue.com/upload/attach/202103/830671_33ZEVAWB9XB2XCZ.jpg)

此时，栈空间布局如下：

![image-20210307171504187](https://bbs.kanxue.com/upload/attach/202103/830671_B87JVZ277XMHRTJ.jpg)

> 图中 `money_menu` 函数即 `input_money_and_menu()` 函数，缩写是为了图片的整齐。

#### e. `menu_options()` 函数

1）输入 `1`：`checkin()`

2）输入 `2`：`checkout()`

![image-20210307105032770](https://bbs.kanxue.com/upload/attach/202103/830671_9EH47U3KFS344X4.jpg)

#### f. `checkin()` 函数

根据 `ptr` 指针来判断用户是否登记：

1）如果登记，返回已登记字符串，退出函数；

2）如果没有登记，则输入一个数字 `nbytes`，然后新分配一个 `nbytes` 大小的堆空间，然后往里输入 “money”（`nbytes` 长度）。

![image-20210307113842075](https://bbs.kanxue.com/upload/attach/202103/830671_9KF9VCYZJ68GU3X.jpg)

#### g. `checkout()` 函数

根据 `ptr` 指针判断，如果用户已经登记，则 `free(ptr)`，并将 `ptr` 置为 `0`。

![image-20210307115111660](https://bbs.kanxue.com/upload/attach/202103/830671_6VMXFH46FUW5TNB.jpg)

#### h. 小结

1. `welcome()` 函数中首先回答 “who are you” 的 `for` 循环中存在 **off-by-one 漏洞**，能泄漏出栈的地址；
2. `input_money_and_menu()` 函数中首先出现 `malloc()` 操作，而且能通过输入 `money` 覆盖堆指针；
3. `checkin()` 函数：`malloc()` 一个指定大小的堆块，并往里输入；
4. `checkout()` 函数：`free()` 并置全局指针 `ptr` 为 `NULL`。

能泄漏栈的地址，能覆盖堆指针，能再次 `free()` 和 `malloc()`，满足 HOS 的利用条件。对应前面的利用场景，如下所示，那么可以在可控 2 区域输入 `shellcode`，可控 1 区域开始伪造 fake chunk，然后往 chunk 输入，覆盖 `input_money_and_menu()` 函数的返回地址为 `shellcode` 的地址，那么菜单第 3 选项退出的时候，`input_money_and_menu()` 也会 `return`。

```
------------------------------------------------------------------------------

            可控1 ： buf —— fake chunk

------------------------------------------------------------------------------

        目标区域（不可控，多为函数指针或返回地址等）：input_money_and_menu函数返回地址

------------------------------------------------------------------------------

            可控2 ： v2(name) —— shellcode

------------------------------------------------------------------------------
```

下面进行调试分析，验证一些分析。

### 步骤四：调试分析

#### a. 模板

```
from pwn import *
from LibcSearcher import LibcSearcher
from sys import argv

def ret2libc(leak, func, path=''):
    if path == '':
        libc = LibcSearcher(func, leak)
        base = leak - libc.dump(func)
        system = base + libc.dump('system')
        binsh = base + libc.dump('str_bin_sh')
    else:
        libc = ELF(path)
        base = leak - libc.sym[func]
        system = base + libc.sym['system']
        binsh = base + libc.search('/bin/sh').next()

    return (system, binsh)

s       = lambda data               : p.send(str(data))
sa      = lambda delim,data         : p.sendafter(delim, str(data))
sl      = lambda data               : p.sendline(str(data))
sla     = lambda delim,data         : p.sendlineafter(delim, str(data))
r       = lambda num=4096           : p.recv(num)
ru      = lambda delims, drop=True  : p.recvuntil(delims, drop)
uu64    = lambda data               : u64(data.ljust(8, '\0'))
leak    = lambda name,addr          : log.success('{} = {:#x}'.format(name, addr))

context.log_level = 'DEBUG'
binary = './lctf_2016_pwn200'
context.binary = binary
elf = ELF(binary, checksec=False)
p = remote('127.0.0.1', 0000) if argv[1] == 'r' else process(binary)
libc = ELF('/lib/x86_64-linux-gnu/libc.so.6', checksec=False)

def dbg():
    gdb.attach(p)
    pause()

#start
# end
p.interactive()
```

#### b. 分析程序的内存布局，并查看 `welcome()` 函数中的 off-by-one 漏洞效果

- 对于 `money`，如果输入 `0x38` 位，那么实际输入的会是 `0x39` 位，因为 `read()` 函数也会将 `\n` 输入到内存中，结果是会覆盖掉 `dest` 最低的一个字节，导致不能查看堆信息。
- 对于 `id`，也是同样如此，输入 4 位。

```
payload = 'a' * 48
ru('who are u?\n')
s(payload)

p.recvuntil(payload)

# leak main function rbp address
rbp_addr = u64(p.recvn(6).ljust(8, '\x00'))
print(hex(rbp_addr))

# input id
sla('give me your id ~~?', '111')

# input money
sla('give me money~', 0x37 * 'b')

# recvive menu
ru('your choice : ')

dbg()
```

![image-20210307195538391](https://bbs.kanxue.com/upload/attach/202103/830671_KG2YB5YQNWCCER8.jpg)

> `id = 0x6f = 111`

因此，纠正并完善前面静态分析得出的内存结构：

![image-20210308120537810](https://bbs.kanxue.com/upload/attach/202103/830671_UVS45DGBNHEAFG2.jpg)

#### c. 思路

1. 输入 `name` 的时候，将 `shellcode` 输入，并补齐 `0x30` 字节，以通过 **off-by-one 漏洞** 泄漏 `rbp` 地址；
2. 输入 `money` 的时候，在 `money` 里伪造 chunk，注意要绕过 `free()` 函数的检查。并通过 **相邻变量覆盖漏洞** 覆盖 `dest` 指针，将其覆盖为 fake chunk 的 `user data` 地址，那么输入完之后，`dest` 指针赋值给 `ptr`，`ptr` 也指向 fake chunk 的 `user data` 地址；
3. 调用 `checkout()`，将伪造 chunk 放入 fastbin 中，并置 `ptr` 指针为 `NULL`；
4. 调用 `checkin()`，因为 `ptr == NULL`，将重新 `malloc()` 指定大小的 chunk 并输入数据，此时，便可以 `malloc()` 可控 1 区域下方的不可控区域，覆盖 `input_money_and_menu()` 函数的返回地址为 `shellcode` 地址；
5. 退出时 `input_money_and_menu()` 函数也会 `return`，将跳转去执行 `shellcode`。

![image-20210308152821920](https://bbs.kanxue.com/upload/attach/202103/830671_Z98VN2BQTPRQDXT.jpg)

#### d. 输入 `name` 和 `money`
