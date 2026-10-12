
# Fuzzing（一个 off-by-one 漏洞）

> 原文：[`www.pediy.com/kssd/pediy12/130946.html`](https://www.pediy.com/kssd/pediy12/130946.html)

【题目】Fuzzing  
【作者】zelik  
【译者】后恋  
【时间】原文时间：2010 -11 月-22 日，译文时间：2011 年-3 月-17 日  
【原文】http://www.exploit-db.com/fuzzing-vs-reversing-round-1-fuzzing/

最近笔者一直在测试 Adobe Flash Player。在笔者自主研发的框架上对 Flash 的一个简单格式进行测试。这项测试逐步深入渐渐的覆盖到了更多的格式。一段时间后，笔者注意到在加载一个 `SWF` 格式的文件时会发生奇怪的崩溃，当用 `OD` 对它进行调试的时候程序会终止。如下图所示：

![](img/6f11701533890e6b88e9987b32844a6c.jpg)

*

**说明：调试环境为安装了 Adobe Flash 10.1.85.3 的 XP SP2 系统。在调试这个程序前，我们先调试下 IE 浏览器发现它被定位到了 `0x03281000h`.**

笔者在调试的时候发现在解析“`DefineFont2`”标签数据的函数中存在一个 Off-by-One 溢出漏洞。这个漏洞看上去并不是很明显，但是当解析“`DefineFont2`”的主函数在解析整个标签时，随着数据和我们定义的字体的提交这个漏洞就出现了。“`DefineFont2`”标签允许我们指定一个不超过 `0xFFh` 字节的字体名称。“`DefineFont2`”标签格式如下：

图

![](img/6dbabfc74c822da642e4751f314ffa8d.jpg)

在有关"DefintFont2"标签的程序初始化完成之后，`0x032ACB45h` 处函数"`CopyFontName`"会分配一个缓冲区用来存储函数所接收到的字体名称参数。这个缓冲区的分配是通过调用 `0x032CA7FEh` 处的"`AllocateBuffer`"函数进行分配的。

引用: 说明："`AllocateBuffer`"函数是封装在另一个函数中的，这个调用并不是直接调用，但是为了简洁，我们修改（modified）了如下的列表来表示直接调用。

代码:
```
.text:032ACB45  CopyFontName
...
.text:032ACB68      push    [esp+0Ch+FontName]
.text:032ACB6C      jge     short loc_32ACB79
.text:032ACB79  allocateFontName:
.text:032ACB79      lea     esi, [edi+10h]
.text:032ACB7C      mov     ecx, esi
.text:032ACB7E      call    AllocateBuffer  ; // AllocateBuffer(Buf, FontName);
.text:032ACB83      mov     ecx, esi
.text:032ACB85
.text:032ACB85  terminateFontName:
.text:032ACB85      push    80h
.text:032ACB8A      call    NullTerminate  ; // NullTerminate(Buf, 0x80);
.text:032ACB8F      mov     ecx, edi
.text:032ACB91      call    sub_32ACA47
.text:032ACB96      pop     edi
.text:032ACB97      pop     esi
.text:032ACB98      pop     ebx
.text:032ACB99      retn    8
```

"`AllocateBuffer`"分配的缓冲区其大小正好是接收的字符串的长度（这个缓冲区待会会被用来溢出那个存在 off-by-one 漏洞的栈区）

代码:
```
.text:032CA609  AllocateBuffer
…
.text:032CA61C      lea     edx, [eax+1]
.text:032CA61F
.text:032CA61F  loc_32CA61F:
.text:032CA61F      mov     cl, [eax]
.text:032CA621      inc     eax
.text:032CA622      test    cl, cl
.text:032CA624      jnz     short loc_32CA61F
.text:032CA626      sub     eax, edx
.text:032CA628      jmp     short loc_32CA62C
```

函数的这个部分计算了传递字符串的长度；等价于如下代码：

代码:
```
length = strlen(string); // The length is EAX
```

在计算完成字符串的长度之后，函数会分配一个合适大小的缓冲区并且会将字符串的内容复制到该缓冲区。当"`AllocateBuffer`"函数返回后，"`CopyFontName`"函数通过调用"`NullTerminate`"函数传递一个常参数 `0x80h` 来限制缓冲区的大小不超过 `0x80h`。

接下来，已分配的缓冲区会在"`OverWrite`"(`0x3426283h`)函数的执行期间被复制到一个栈缓冲区。这个函数拥有一个 `0x80h` 大小的栈缓冲区来复制字符串。问题出现在这里字符串的大小是 `0x80h+NULL` 结束符(实际上是 `0x81h`).函数通过如下编译器生成的代码进行字符串的复制：

代码:
```
.text:03426372  StringCopy:

.text:03426372      mov     cl, [eax]
.text:03426374      mov     [edx+eax], cl
.text:03426377      inc     eax
.text:03426378      test    cl, cl
.text:0342637A      jnz     short StringCopy
```

这些代码框架首先复制一个字符然后才检查一下字符是不是 null（像大部分字符串复制函数所做的那样）.结果一个典型的 off-by-one 漏洞出现了，它会导致栈缓存的一个字节被覆盖。这种情况在 windows 平台下不能够被利用但是在栈缓存测试失败的时候会引起浏览器崩溃。

（顺便说一下，据笔者所知这个漏洞到目前为止仍然没有修复）

## 关于作者

Tal Zeltzer（zelik）是一个来自以色列的安全研究员，专注于软件和硬件的逆向工程（译者注：硬件也能逆向？原来真能！看雪有个帖子：http://bbs.pediy.com/showthread.php?t=52957）。他大部分空闲时间都是来研发个人安全工具和 exploit 并以此为乐。
