

# 翻译九月专题\[023\]代码的逆向工程

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-775.htm`](https://www.pediy.com/kssd/pediy08/pediy8-775.htm)

原文地址 023.Reverse Engineering Hostile Code   http://www.mousearmy.net/tech/reverse\_engineering\_hostile\_code.pdf  
  
敌意代码的逆向工程  
作者 Joe Stewart  
最后更新 10.23，2002  
电脑罪犯总是准备好并等待时机去危机有安全漏洞的系统。当他们开始行动时，通常是在系统中留下程序来维持控制权。我们称这些程序为“特洛伊木马”根据古希腊的特洛伊木马的故事。通常这些程序是汇编格式的，并且不能广泛传播。因此，反病毒程序并不需要经常检测木马。这也就意味着关于任何特定的木马的操作通常是不清楚的，所以在对这些代码的一般分析，确定威胁的程度以及如何可能的话精确地指出攻击的来源是非常有必要的。  
这篇文章勾勒出恶意代码的逆向工程的整个过程轮廓。针对“恶意代码”，我们的定义是不经管理员的允许而运行在系统上的任何进程，例如 Trojans，病毒或者间谍软件。这篇文章不能规定为高水平的指南书，而是对所用的工具及进行的步骤的说明书。当具备了这些知识后，即使对汇编语言编程不熟悉的人也应当能恶意代码的内核并判定它所要进行的操作，至少在表面上的。  
需要的工具  
在进行许多类型的工程时，需要一些工具。我们提及工具本质上在 Unix 和 Windows 上都能运行。然而 Unix 是进行初始逆向工程操作的理想的平台，但你仍然可以在 Window 上进行这项工作，特别是如果你已安装例如 Cygwin，Unix 环境就能运行 Win32 平台。当 Cygwin 运行时，大部分命令在 Window 也能识别。然而，当你沿着 Window 路线进行反编译/反汇编/调试步骤时费用很大，然而利用 Unix 则是完全免费的。因此在选择进行逆向工程的平台时，必须权衡利用 Windows 的费用与在此平台的收益比。  
一些有用的命令：  
dd-从源设备一个一个字节拷贝。这对在经允许的系统上的硬件驱动执行分析是很有用的，因为它不影响入侵证据的完整性。  
file-基于内容对文件类型的识别。  
strings-输出可执行程序的可读的字符串  
hexedit-读并编辑二进制文件  
md5sum-生成文件比较的一个唯一的校验和  
diff-输出文件的不同之处  
lsof-显示所有打开的文件及进程的 Socket  
tcpdump-网络包的嗅探器  
grep-在一个文件中搜索字符串  
压缩的可执行体  
木马经常是利用可执行打包工具进行压缩的格式。这不仅使代码更精简，而且也使字串创命令及二进制编辑命令不能读取代码内部字符串。通常利用最多的就是 UPX 可执行压缩工具，这个软件可以用来压缩 Linux 或者 Windows 二进制文件。也有其他的打包工具，但只是典型的仅基于 Windows。幸运的是，UPX 少数几个提供手动解压功能恢复原始文件的打包工具之一。这也就使得我们没有必要利用高级技术来解压恢复原文件了。  
在普通的可执行文件中，执行“string”命令或者利用十六进制编辑器来检测木马就能展现许多在文件中可读和完全的字符串。如果你仅看见任意的二进制表示的字符或者大多数情况下，被删节的以及分散的片断文本，说明这个可执行文件很可能已经打包。利用 grep 和 hexedit 命令，如果文件利用 UPX 打包了，你就能在这个文件中的某个地方，找到“UPX”字符串。否则，你可能要处理许多其他的打包工具中一种。处理其他打包格式超出了本文的范围，但你仍能从本文中找到对处理这些文件的一些帮助。  
反编译  
在极其偶然的情况下，如果运气好的话，发现木马是利用解释或者半解释性语言如 Visual Basic，Java 或者甚至编译性的语言 Perl。总有一些可用的工具来对这些语言进行不同程度的反编译。  
Visual Basic-在互连网上流行的有关于 VB3 版本的反编译器。对新的版本的，还没有已知的反编译器，但你可以例如象 Compuware 的 SmartCheck 来跟踪程序中的调用。尽管它的输出不是源码列表，你仍能看到程序内部所作的任意操作。  
Java-有一款很出色的反编译器 jad，它可以完全反编译出源代码列表，但不影响源码重复的编译性。还有其他的已知的 java 反编译器。  
Perl-Perl 程序被编译成 Windows 可执行文件，可以利用 exe2perl 工具来将其精简为它们的核脚本。  
反汇编  
如果木马程序是利用完全编译的语言编写的话，你就不得不去啃这块硬石头，将这些代码反汇编为功能相同的汇编语言代码。对 Unix 可执行文件，objdump 就是沿着这样的思想进行工作的反汇编工具。对于 Windows 可执行文件，你就需要 IDA Pro 或者 W32dasm。也有一款与 IDA Pro 功能相媲美的免费版本的 IDA，但是它只有一个基于接口的控制台。这些软件可以反汇编去代码，然后再数据段中进行在程序中出现的字符匹配，并将子函数进行分割开来。它们也试图通过名称而不是偏移量提供 Windows API 调用的一些信息。这种输出就是所谓的静态代码列表，并为你提供这个程序内部执行的操作的概况。然而 GNU objdump 并没有提供这样有用的信息，但有一个基于 perl 的打包工具可使 objjump 调用 dasm，这样就可以为你提供与 Windows 反汇编工具所提供的同样的功能。  
调试器  
尽管静态代码列表非常有有用，然而你仍然需用调试器一步一步跟踪这个程序代码，特别是如果木马通过网线。这就为你提供了一种获得存储在程序中的临时变量以及内存数据的途径，其中包括所有它在网络交互中发送接受的数据。在 Unix 中，gdb 是首选的调试器。它在 Unix 系统中有很长的历史了，并有健全的帮助文件，且功能最好同时也是免费可得。在 Windows 系统下，就有了很广的选择的余地了，但许多在 Win32 下有关逆向工程的专题论文都是基于 SoftICE 调试器的。尽管代价不菲，但如果你能支付的起话，还是物有所值。  
准备调试  
即使在调试器下运行恶意代码时，你必须在做好防范措施。绝对不能在联网情况下，来调试木马。理想情况下，你应当建立一个试验网络，如图 1 所示。  
  
  
图一 ：一个典型的调试网络  
调试系统中 Trojan 有意向侵入的任何操作系统应当是全新安装的。在网络中存在第三方的系统可以使你仿效这些服务并能截取这些由 Trojan 产生的网络数据。与跟踪感染病毒的源头相比截取这些数据非常微不足道的。确信你的防火墙能阻挡所有与外界的联系，并只允许 Trojan 的控制连接。如果你不想让主控机，探测到你的试验网络在运行 Trojan，你可以建立起运行木马所需要的资源模拟服务器，例如 IRC 或者 FTP/TFTP 服务器。  
单步执行代码  
因为我们已经建立一个恰当隔离的试验环境，我们就可以开始调试代码了。利用代码清单，在程序中寻找关键函数，例如 Winsock 以及 I/O 调用。调试器允许我们基于偏移量在程序中设置断点，因此我们可以打断程序流并察看当前点的程序的存储以及 CPU 的寄存器。文章的剩余部分将介绍在 Linux 平台下如何来实现调试的一个例子。  
运行调试器  
 我们想知道木马如何与它的主控者如何联系的。经常地，监视网络传输就足够了。然而，许多新的木马则是以密文的形式来进行网络传输的，这就使得网络监视失去作用。然而，利用些小聪明，我们在信息编码前，就从内存中截取它们。通过在传输库调用“send”上设置断点，我们就能实现在数据包在传输前，打断代码执行。然后，通过跟踪堆栈指针，我们就可推测程序执行到哪里了。例如，木马的源程序也许如下：  
/\* encrypt output to master \*/  
elen = encrypt(crypted,buf,len);  
/\* write crypted output to socket \*/  
send(s, crypted, elen, 0);  
在 gdb 中检测被编译的木马也许可给我们下面的结果\[注意下面的粗体字陈述，代表作者对输出的评论\]  
\[test@debugger test\]$ gdb ./Trojan  
GNU gdb 5.2.1-2mdk (Mandrake Linux)  
Copyright 2002 Free Software Foundation, Inc.  
GDB is free software, covered by the GNU General Public License,  
and you are welcome to change it and/or distribute copies of  
it under certain conditions.  
Type "show copying" to see the conditions.  
There is absolutely no warranty for GDB. Type "show warranty"  
for details.  
This GDB was configured as "i586-mandrake-linux-gnu"...  
(no debugging symbols found)...  
(gdb) set disassembly-flavor intel \[Switch syntax output from AT&T\]  
(gdb) b send \[Set a breakpoint on the "send" library call\]  
Breakpoint 1 at 0x400f5c10  
(gdb) run  
Starting program: /home/test/Trojan  
Breakpoint 1, 0x400f5c10 in send () \[We hit a breakpoint\]  
(gdb) where \[Do a stack trace to see where we are at in the program\]  
#0 0x400f5c10 in send () from /lib/i686/libc.so.6  
#1 0x080487fa in socket ()  
#2 0x40040082 in \_\_libc\_start\_main () from /lib/i686/libc.so.6  
在 gdb 中显示偏移量每个子函数的执行后返回的结果就如上面所示。我们知道“send”调用正好是在编码调用之后，因此我们只需检测前面的子函数，它的返回值包含了偏移量为 0x080487fa 内存器的内容。我们感兴趣仅是这个偏移量前面的汇编代码。利用 gdb，我们可以在当前反汇编这些代码。  
(gdb) disas 0x080487d2 0x080487fa  
Dump of assembler code from 0x80487d2 to 0x80487fa:  
0x80487d2 <socket+622>: call 0x8048804 <socket+672>  
0x80487d7 <socket+627>: add esp,0x10  
0x80487da <socket+630>: mov DWORD PTR \[ebp-836\],eax  
0x80487e0 <socket+636>: push 0x0  
0x80487e2 <socket+638>: push DWORD PTR \[ebp-836\]  
0x80487e8 <socket+644>: lea eax,\[ebp-824\]  
0x80487ee <socket+650>: push eax  
0x80487ef <socket+651>: push DWORD PTR \[ebp-828\]  
0x80487f5 <socket+657>: call 0x8048534 <send>  
End of assembler dump.  
我们看到正好在调用“send”前，有一个对 0x8048804<socket+672>的调用。实际上这就是，我们需要找的编码子函数。当程序销毁它们的符号时，gdb 就不能辨别出子程序开始和结束的地方了，因此它持续利用它最后识别的所有子程序的名称，通常是前面的动态连接库调用。在这种情况下，它就有可能标错“socket”函数的开始部分。  
为了探测未编码的数据包的内容，我们仅需要知道这个调用函数是如何工作的。子函数的变量被压入堆栈段中，即存放临时数据和返回地址的内存。我们能通过在调用处设置断点来获得它所存储的变量内容，然后利用 CPU 的寄存器 ESP 知道堆栈指针的偏移量。ESP+4 指向第一个变量，ESP+8 指向第二个变量，ESP+12 指向第三个变量，依次类推。就这样搜索堆栈直止有用的信息出现。在这种情况下，有用的信息（文本数据）就是编码的变量。让我们在这个编码调用设置一个断点，并检查堆栈。  
(gdb) b \* 0x80487d2 \[Set a breakpoint on the "encrypt" call\]  
Breakpoint 2 at 0x80487d2  
(gdb) run  
The program being debugged has been started already.  
Start it from the beginning? (y or n) y  
Starting program: /home/test/Trojan  
(no debugging symbols found)...  
Breakpoint 2, 0x080487d2 in socket ()  
(gdb) x/x $esp+8 \[Get the offset of the second argument ESP+8\]  
0xbffff5e4: 0x0806fe20  
(gdb) x/fs 0x0806fe20 \[Examine the contents of the memory at 0x0806fe20\]  
0x806fe20: "root pts/0 Oct 11 14:22\\n"  
从这个输出我们能看到木马正在向登陆这个系统的主机汇报。当然，它能传输任何数据；网络数据包，键盘输入记录等等。幸运的是，我们使用已建立的网络改变数据传输的方向。  
结论  
上面的木马是虚假的。假如是实际中的木马，我们也许就要增加一些操作。通常木马将建立例如 IRC 这样的渠道来与主控者联系。我们能利用这个事实，来跟踪攻击的来源，如果木马编写者稍有疏忽，甚至获得木马主控的整个网络的控制权。如果木马利用 FTP 来更新自己，你就应当在 FTP 服务器上搜索一些其他的代码，来获得木马编写者的线索。  
尽管我们已经熟悉了逆向工程的基本步骤，你应当能汲取上面的信息并付诸实践。察看你的调试器的帮助文件；你就会发现它多么有用，可以学到很多有用的知识；即使你对汇编代码一点都不熟悉。如果它看上去很困难，但绝不要放弃希望。付出总会有回报的。在一个实际逆向工程工作中，作者发现了木马编写者的姓名，并不是有意嵌入在程序的源代码中（注意：不要在你的 NT 工作站在运行时，利用 VB 来编写木马程序）。利用搜索引擎 GOOGLE 来查找作者的 email，就会出现 VB 程序的论坛网站。然后就知道他是谁，以及他的家庭地址，以及电话号码。在巴西的某个地方，木马的编写者拍着自己的前额用葡萄牙语说 DOH。  
关于作者  
Joe Stewart 是 LURHQ 公司的一个高级信息安全分析师，住在南 Carrolina 的 Myrtle Beach 的安全服务提供者。  
参考文献  
Fravia's Pages of Reverse Engineering   
Detecting and Containing IRC-Controlled Trojans: When Firewalls, AV, and IDS Are   
Not Enough  
Reverse Engineering Malware 

 4dd466d0e1e0cc123afbe504da2462ca