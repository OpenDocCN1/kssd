
#### 脚本

```python
def modify\_arg(ql: Qiling):
ql.reg.edi = 0
def i\_slept\_i\_faked\_it(ql: Qiling):
# 我睡了，我装的
return
def hook\_nanosleep(ql: Qiling, \*args, \*\*kwargs):
# 注意参数列表
return
def challenge7(ql: Qiling):
# method 1
# ql.set\_api('sleep', modify\_arg, QL\_INTERCEPT.ENTER)
# method 2
# ql.set\_api('sleep', i\_slept\_i\_faked\_it)
# method 3
ql.set\_syscall('nanosleep', hook\_nanosleep)
```

## challenge 8: 解析结构体，往正确地址写入值

这里直接借用Joan Sivion整理的代码：

```c
void challenge8(char \*check) {
random\_struct \*s;
s = (random\_struct \*)malloc(24);
s->some\_string = (char \*)malloc(0x1E);
s->magic = 0x3DFCD6EA00000539;
strcpy(s->field\_0, "Random data");
s->check\_addr = check;
}
struct random\_struct {
char \*some\_string;
\_\_int64 magic;
char \*check\_addr;
};
```

方法有两种：

1. 最后一句的时候，获取这个结构体地址，可以看一下最后部分的汇编：

```纯文本
.text:0000564846E00FA9 48 8B 45 F8 mov rax, [rbp+stru] ; stru == -8
.text:0000564846E00FAD 48 8B 55 E8 mov rdx, [rbp+var\_18]
.text:0000564846E00FB1 48 89 50 10 mov [rax+10h], rdx
.text:0000564846E00FB5 90 nop
.text:0000564846E00FB6 C9 leave
.text:0000564846E00FB7 C3 retn
```

&ensp;&ensp;&ensp;&ensp;然后结构体偏移`+0x16`的地方就是保存了参数的位置。

2. 使用qiling提供的机制，用`ql.mem.search()`搜索内存。看一下代码可以发现，这个结构体保存了一个魔数`0x3DFCD6EA00000539`，这正好是个特征方便我们去搜索这块内存。当然，还在第一个字段保存了一个指针指向一个固定的字符串，如果那个魔数在内存中出现多次，那显然我们还需要进一步获取信息确定是不是我们要找的那个结构体：保存了参数`check`的结构体。

### 脚本

```python
def hook\_struct(ql: Qiling):
"""
0000564846E00FA9 48 8B 45 F8 mov rax, [rbp+str] ; rbp - 8
0000564846E00FAD 48 8B 55 E8 mov rdx, [rbp+var\_18]
0000564846E00FB1 48 89 50 10 mov [rax+10h], rdx
0000564846E00FB5 90 nop <<< hook here
0000564846E00FB6 C9 leave
0000564846E00FB7 C3 retn
"""
heap\_struct\_addr = ql.unpack64(ql.mem.read(ql.reg.rbp - 8, 8))
heap\_struct = ql.mem.read(heap\_struct\_addr, 24)
printHex(heap\_struct)
\_, \_, check\_addr = struct.unpack('QQQ', heap\_struct)
ql.mem.write(check\_addr, b'\x01')
def search\_mem\_to\_find\_struct(ql: Qiling):
MAGIC = ql.pack64(0x3DFCD6EA00000539)
candidate\_addrs = ql.mem.search(MAGIC)
for addr in candidate\_addrs:
# 有可能有多个地址，所以通过其他特征进一步确认
stru\_addr = addr - 8
stru = ql.mem.read(stru\_addr, 24)
string\_addr, \_, check\_addr = struct.unpack('QQQ', stru)
if ql.mem.string(string\_addr) == 'Random data':
ql.mem.write(check\_addr, b'\x01')
break
def challenge8(ql: Qiling):
base = ql.mem.get\_lib\_base(ql.path)
# method 1
# ql.hook\_address(hook\_struct, base + 0xFB5)
# method 2
ql.hook\_address(search\_mem\_to\_find\_struct, base + 0xFB5)
```

## challenge 9: 修改字符串函数

```c
unsigned \_\_int64 \_\_fastcall challenge9(bool \*a1)
{
char \*i; // [rsp+18h] [rbp-58h]
char dest[32]; // [rsp+20h] [rbp-50h] BYREF
char src[40]; // [rsp+40h] [rbp-30h] BYREF
unsigned \_\_int64 v5; // [rsp+68h] [rbp-8h]
v5 = \_\_readfsqword(0x28u);
strcpy(src, "aBcdeFghiJKlMnopqRstuVWxYz");
strcpy(dest, src);
for ( i = dest; \*i; ++i )
\*i = tolower(\*i);
\*a1 = strcmp(src, dest) == 0;
return \_\_readfsqword(0x28u) ^ v5;
}
```

要么让`tolower()`失效，要么让`strcmp()`失效。

### 脚本

```python
def fake\_tolower(ql: Qiling):
return
def challenge9(ql: Qiling):
ql.set\_api('tolower', fake\_tolower)
```

## challenge10: 劫持文件系统，返回指定命令行

```c
unsigned \_\_int64 \_\_fastcall challenge10(\_BYTE \*a1)
{
int i; // [rsp+10h] [rbp-60h]
int fd; // [rsp+14h] [rbp-5Ch]
ssize\_t v4; // [rsp+18h] [rbp-58h]
char buf[72]; // [rsp+20h] [rbp-50h] BYREF
unsigned \_\_int64 v6; // [rsp+68h] [rbp-8h]
v6 = \_\_readfsqword(0x28u);
fd = open("/proc/self/cmdline", 0);
if ( fd != -1 )
{
v4 = read(fd, buf, 0x3FuLL);
if ( v4 > 0 )
{
close(fd);
for ( i = 0; v4 > i; ++i )
{
if ( !buf[i] )
buf[i] = 32;
}
buf[v4] = 0;
if ( !strcmp(buf, "qilinglab") )
\*a1 = 1;
}
}
return \_\_readfsqword(0x28u) ^ v6;
}
```

让我们能从`/proc/self/cmdline`读到`"qilinglab"`，故技重施即可，hook文件系统。

### 脚本

```python
class Fake\_cmdline(QlFsMappedObject):
def read(self, expected\_len):
return b'qilinglab'
def close(self):
return 0
def challenge10(ql: Qiling):
ql.add\_fs\_mapper('/proc/self/cmdline', Fake\_cmdline())
```

### 简便方法

可以直接替换指定文件成我们的文件，先创建我们需要的文件：

```bash
echo -n "qilinglab" > fake\_cmdline
```

然后代码就可以这样写了：

```python
def challenge10(ql):
ql.add\_fs\_mapper("/proc/self/cmdline", "./fake\_cmdline")
```

还一种方法，代码也不用写了，直接往qiling的rootfs里面写：

```bash
mkdir -p ./qiling/examples/rootfs/x8664\_linux/proc/self
echo -n "qilinglab" > ./qiling/examples/rootfs/x8664\_linux/proc/self/cmdline
```

## challenge 11: 指令hook

这里指的是用qiling的指令`hook\_code()`而不是`ql.hook\_insn()`，先看下目标

```c
unsigned \_\_int64 \_\_fastcall challenge11(\_BYTE \*a1)
{
int v7; // [rsp+1Ch] [rbp-34h]
int v8; // [rsp+24h] [rbp-2Ch]
char s[4]; // [rsp+2Bh] [rbp-25h] BYREF
char v10[4]; // [rsp+2Fh] [rbp-21h] BYREF
char v11[4]; // [rsp+33h] [rbp-1Dh] BYREF
unsigned \_\_int64 v12; // [rsp+38h] [rbp-18h]
v12 = \_\_readfsqword(0x28u);
\_RAX = 0x40000000LL;
\_\_asm { cpuid }
v7 = \_RCX;
v8 = \_RDX;
if ( \_\_PAIR64\_\_(\_RBX, \_RCX) == 0x696C6951614C676ELL && (\_DWORD)\_RDX == 538976354 )
\*a1 = 1;
// ...
}
```

看下汇编：

```NASM
cpuid
mov eax, edx
mov esi, ebx
mov [rbp+var\_30], esi
mov [rbp+var\_34], ecx
mov [rbp+var\_2C], eax
cmp [rbp+var\_30], 696C6951h
jnz short loc\_564846E011C0
cmp [rbp+var\_34], 614C676Eh
jnz short loc\_564846E011C0
cmp [rbp+var\_2C], 20202062h
jnz short loc\_564846E011C0
```

目标就很明确了。cpuid指令会填充几个寄存器，具体可以参考intel手册。

### 脚本

```python
def hook\_cpuid(ql: Qiling, address, size):
"""
0000564846E0118F 0F A2 cpuid
"""
if ql.mem.read(address, size) == b'\x0F\xA2':
ql.reg.ebx = 0x696C6951
ql.reg.ecx = 0x614C676E
ql.reg.edx = 0x20202062
ql.reg.rip += 2
def challenge11(ql: Qiling):
begin, end = 0, 0
for info in ql.mem.map\_info:
if info[2] == 5 and 'qilinglab-x86\_64' in info[3]:
begin, end = info[:2]
ql.hook\_code(hook\_cpuid, begin=begin, end=end)
```

说明：

`ql.mem.map\_info`也就是`ql.mem.show\_mapinfo()`的内容，5表示的是`r-x`属性，加这个判断也是为了缩小hook的范围，提高性能。

## 总结

好耶！

```NASM
Challenge 1: SOLVED
Challenge 2: SOLVED
Challenge 3: SOLVED
Challenge 4: SOLVED
Challenge 5: SOLVED
Challenge 6: SOLVED
Challenge 7: SOLVED
Challenge 8: SOLVED
Challenge 9: SOLVED
Challenge 10: SOLVED
Challenge 11: SOLVED
You solved 11/11 of the challenges
```

到此位置这个练习就全部结束了！感觉也没啥太多坑的点，有的话根据报错信息来解决就好。

---

更新后的脚本：
