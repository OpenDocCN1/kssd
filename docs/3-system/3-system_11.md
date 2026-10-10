

# Microsofts Rich Header

> 原文：[`www.pediy.com/kssd/pediy12/125447.html`](https://www.pediy.com/kssd/pediy12/125447.html)

题目：Microsofts Rich Header  
作者：Peter Kleissner  
译者：Cryin'([`hi.baidu.com/justear`](http://hi.baidu.com/justear))  
  
概述  
  
Rich Header 是 PE 文件 DOS Header 和 NT Header 之间的一个结构(在 PE Header 和 DOS stub 之间)。它包含链接库版本信息以及链接器版本。  
  
The Rich Header  
  
Rich Header 是由微软编译器创建，通常看起来像如下：  
  
WSNPOEM Decryption Code analyzed:  
  
00000080  54 62 EF 9B 10 03 81 C8 10 03 81 C8 10 03 81 C8  Tb??...è...è...è  
00000090  37 C5 EF C8 11 03 81 C8 37 C5 FC C8 12 03 81 C8  7??è...è7?üè...è  
000000A0  37 C5 FA C8 0B 03 81 C8 10 03 80 C8 C9 03 81 C8  7?úè...è..èé..è  
000000B0  37 C5 EC C8 33 03 81 C8 37 C5 FD C8 11 03 81 C8  7?ìè3..è7?yè...è  
000000C0  37 C5 F9 C8 11 03 81 C8 52 69 63 68 10 03 81 C8  7?ùè...èRich...è  
  
这是 notepad.exe 的 Rich Header 信息，起始偏移地址为 80h，紧跟在 DOS stub 之后。紧跟在 Rich Header 之后的是 PE Header。从上面的 Rich Header 信息你大概可以看出 Rich Header 是被加密过的。  
  
解密 Rich Header  
  
Rich Header 信息是被异或操作加密过的，观察上面 Rich Header 数据你会发现一个重复多次的 DWORD 值即"Rich"之后的 10 03 81 c8。将 Rich Header 信息的起始位置一直到关键字"Rich"的数据与这个 DWORD 值进行异或操作，操作后的 Rich Header 如下所示：  
  
00000080  44 61 6e 53 00 00 00 00 00 00 00 00 00 00 00 00  DanS............  
00000090  27 c6 6e 00 01 00 00 00 27 c6 7d 00 02 00 00 00  '?n.....'?}.....  
000000A0  27 c6 7b 00 1b 00 00 00 00 00 01 00 d9 00 00 00  '?{.........ù...  
000000B0  27 c6 6d 00 23 00 00 00 27 c6 7c 00 01 00 00 00  '?m.#...'?|.....  
000000C0  27 c6 78 00 01 00 00 00 52 69 63 68 10 03 81 c8  '?x.....Rich...è  
  
我们看到第一个 DWORD 值为"DanS"，这个好像正是 Dan Ruder 的开头部分，Dan Ruder 正是"Mechanics of Dynamic Linking" in 1993(MSDN Library Archive)的作者之一。  
  
格式解析  
  
从 lifewire 的文章详见参考\[6\]获知 Rich Header 结构的格式如下：  
  
'DanS'^b, b, b, b         -- 身份认证头  
compid^b, r^b           -- 从 0 开始  
..                     --      :  
compid^b, r^b          -- 直到 n  
  
'Rich', b              -- 结束位置  
  
后面是一些无用的信息  
  
  
上面所有的值都是 DWORD 型，b 是异或值，是一个校验和值。^是异或操作符。compid 是编译器 id(compiler id)。最小的 Rich Header 的大小为 8\*4(至少包含一个编译器 id)。可以通过关键字"Rich"和"DanS"验证 Rich Header 信息。  
  
编译器 id 是编译库文件所使用的连接器版本。Rich Header 包含一个所有被链接的库文件的链表，这个正是 compid 的值。compid 最后的值是链接文件的连接器版本号。comp.id 的值是按照 DWORD 型存储。LWORD 包含了组建号，HWORD 的低位(low 4 bits  )包含主版本号，高位(high 4 bits)包含子版本号。  
  
校验和与异或值 b 通过如下步骤计算得出：  
  
b=sizeof(dos\_stub)              // 一般情况均为 0x80  
  
for (int i = 0; i < sizeof(dos\_stub); i++)  
{  
    b += dos\_stub\[i\] ROL i;     // ROL 循环左移操作  
}  
  
for (int i = 0; i < n; i++)  
{  
    b += compid\[i\] ROL r\[i\];  
}  
  
  
第一个循环根据 DOS stub 生成一个校验和，第二个循环遍历所有库版本。具体算法介绍可参考\[6\]。  
  
@comp.id 值  
  
正如前面描述，compid 的值为连接器版本号。连接器包含所有静态链接库的 comp.id 值(例如 kernel32.lib)并存储在一个列表中。连接器将自己的版本信息添加到这个列表的末尾。你可以在一个 lib 文件中查找"@comp.id"，紧跟着的下一个 DWORD 就是连接器版本号(即编译器 id)。  
  
通过列表里面的最后一个 compid 值，你可以确定编译器版本、连接器版本以及编译应用程序所使用的开发环境。例如编译器 id 值为 0x0078C627 就意味着"Version 8.00.50727, Microsoft Visual Studio 2005"。这里要注意微软针对不同编译器版本可能不尽相同。你可以通过组建号来识别使用的是编译器(cl.exe)还是连接器(link.exe)版本(组建号相同，编译器和连接器的主版本和字版本号不同)。  
  
你可以创建并使用一个@comp.id 转换表(编译器/连接器版本号值的转换)来利用 Rich Header 的有效性。  
  
参考链接  
  
\[1\] http://www.woodmann.com/forum/archive/index.php/t-11367.html  
\[2\] http://www.donationcoder.com/Forums/bb/index.php?action=printpage;topic=11956.0  
\[3\] http://trendystephen.blogspot.com/2008/01/rich-header.html  
\[4\] http://www.ntcore.com/Files/richsign.htm  
\[5\] http://computer.forensikblog.de/en/2008/03/more\_about\_the\_rich\_header.html  
\[6\] http://www.woodmann.net/forum/attachment.php?attachmentid=1050  
  
关于翻译  
  
翻译的可能不是很清楚(Don't blame me)，如果想自己进行解密实验，可以使用我在[`bbs.pediy.com/showthread.php?t=125416`](http://bbs.pediy.com/showthread.php?t=125416)上传的工具中就有 DWORD 异或解密工具自己解密探个究竟。另外可以参见原文及参考文献进一步阅读。