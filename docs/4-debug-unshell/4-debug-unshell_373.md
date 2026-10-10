

# 深入剖析 AsProtect 

> 原文：[`www.pediy.com/kssd/pediy06/pediy6865.htm`](https://www.pediy.com/kssd/pediy06/pediy6865.htm)

原文链接：[`bbs.pediy.com/showthread.php?s=&threadid=5354`](http://bbs.pediy.com/showthread.php?s=&threadid=5354)

译者废话：这是一篇重在“道”而不在“术”的好文章，这在文章的开头讲得很清楚，我就不多说了。小弟我初学脱壳、初学英语，不自量力地翻译这篇“巨著”（我们老师说了，学英语就得脸皮厚不怕丢人![](img/26ed9fa78f7d02ee5a28c57dc1e1e2a8.jpg)），很有力不从心的感觉，而且我也无法保证一定能够完成全文，所以请各位严格监督，对文中的错误之处猛烈批评。  
**  
AsProtected Notepad!  
   
深入剖析 AsProtect  
  
作者：E.Labir  
译者：RoBa\[TT\]  
**  
  
英文原文下载：[`www.codebreakers-journal.com/include/getdoc.php?id=60&article=34&mode=pdf`](http://www.codebreakers-journal.com/include/getdoc.php?id=60&article=34&mode=pdf)  
  
**摘要：**  
  
当我开始读关于脱壳的教程时，我发现只能找到像下面这样的东东：  
  
使用 TraceX N 次  
然后就来到了入口点(EP)....  
用 Procdump（或其它工具）来 dump  
用 XXXX 工具来重建输入表  
  
我从里面学到了什么呢？一点也没有！我不愿意只掌握一个针对某种过时的保护的偏方。这是不够的，我要学习！  
  
在这篇文章中，我试图展示那些反破解（Anti-Crack）高手们阻止脱壳或破解的通用方法。我们的基本方法就是调试被加壳的程序，寻找所有的反调试手段，去掉它们，然后（注意只有在做完上面那些以后）分析理解如何脱壳（还有如何准备脱壳）  
  
我将使用演示版的 AsProtect（我们可以给像 Notepad 或 Regedit 这样的常见软件加上保护）作为例子，但是我不会提供任何关于商业软件的信息（别浪费时间问我这个）。  
  
DISCLAIMER: what you do with this knowledge is ONLY your responsability.  
  
关键词：代码逆向工程，保护技术，分析评测软件保护系统的作用  
  
  
**目录**  
  
I          目标程序  
II         导论  
III        反静态分析  
IV         自解密  
V          向调试器抛出异常  
VI         校验和  
   VI-A        文件 NOTEPAD.EXE 的校验和  
   VI-B        内存映像的校验和  
VII        更多的反调试手段：IsDebuggerPresent,FS:\[18\],...  
VIII       钩子  
IX         如何找到入口点  
   IX-A        AsProtect 演示版  
X          准备 Dump  
XI         重建输入表  
   XI-A        如何保护你的输入表  
       XI-A.1      移除 OriginalFirstThunk  
       XI-A.2      移除 API 名称  
       XI-A.3      转移调用至外壳程序  
       XI-A.4      加入多态性  
       XI-A.5      混合手段  
       XI-A.6      完全取代一个 API  
       XI-A.7      Kernel32.dll 是不可写的  
       XI-A.8      移走整个过程  
   XI-B        发现这些反调试手段的技巧  
XII        灭掉所有的反调试  
XIII       对该保护的扩展  
XIV        最后注意事项  
XV         资源  
XVI        结论  
XVII       结束语  
  
**  
I.目标程序  
**  
我用演示版 ASPROTECT 保护的记事本程序来写这个教程。我在前面提到过，我不会提供任何商业软件的信息。这篇论文的目的是解释如何让你的软件被保护得更好，而不是破解某个实际的目标。你的第一个任务是从 ASPROTECT 的主页上下载 ASPROTECT 的演示版，并且阅读它的帮助文件，这会使你更能明白它如何工作。  
  
使用演示版时我选上所有的选项：  
  
Resource Protection = YES  
Use Max. compresion = YES  
Anti-debugger = YES  
Checksum = YES  
Trial info ... limited trial:  
Number of days = 30  
Number of executions = 10  
Reminder message = YES  
Expiration date = 2003, december the 31th  
  
在按“保护”按钮后你会看到类似下面的内容：  
  
" Use CRC check protection...  
Use anti-debugger protection...  
Use 30 trial days limitation...  
Use expiration date (31/12/03)...  
Use 10 executions limitation  
Use reminder  
Use built-in dialogs...  
Protection done ..."  
File size: ... compressed to ..., Ratio: ..%"  
**  
II.导论  
**  
ASPROTECT 有多种特性，比如：  
  
 + 你可以选择加密你的一部分程序代码  
 + 它保护输入表不被提取出来  
 + 加入大量的无效代码让调试变得令人厌烦  
 + 加入反调试，校验和，隐藏你的程序入口点  
 + 使用 WINDOWS 注册表存储时间限制和注册信息  
 + 让程序员使用一些专门的过程，例如 GetRegistrationInformation()  
  
在这篇论文中我不会讲述 RSA 保护，因为这是浪费时间。就算我能够在算法里面找到一处错误，这错误可以在几秒钟之内被修正。使用密码学算法的问题非常简单：  
如果你加密了程序的重要部分，没人能完全地测试这个演示版。  
如果你只是加密了“保存”功能，我可以用我自己的代码取代你的加密的代码。  
因此密码学不是至关重要的。  
  
你需要的工具：  
  
一个调试器，我使用 Olly  
一个十六进制编辑器，我使用 hiew  
Procdump（只为了从内存中提供数据）  
  
你不需要的工具：  
  
输入表重建工具  
  
关于如何进行下去：  
  
记住，当脱一个程序时，不要把你限制为只是“受害人”  
  
 + 写一个非常简单的程序并加壳，找到它的入口点（比如在入口点显示一个消息框）。你可以并且必须在你的代码中放置一些线索（可辨认的字符串）来引导你找到需要的东西。这在当你把调试器挂接在目标程序时非常有帮助。  
 + 对其他简单的程序（比如记事本）进行同样的操作，你的推论还成立吗？  
 + 尝试一些实际的软件并比较  
 + 加入/去除不同的保护（加入/去除时间限制，压缩/不压缩资源。。。）  
  
如果你按照从最简单到最复杂的顺序来做，就会非常容易。这种策略表明提供一个能对你的程序进行不完全保护的“演示版”的壳不是一个好主意，你不同意吗？令人吃惊的是，看上去没有人同意这种观点……  
**  
III 反静态分析  
**  
理解它的最好方法就是看一个实例，我们一起看看这个加壳的记事本的一些指令：  
  
01001000 $ 68 01300101 PUSH pnotepad.01013001  
01001005 . E8 01000000 CALL pnotepad.0100100B  
0100100A . C3 RETN  
0100100B $ C3 RETN  
  
可以看出，Asprotect 用一些 RET 来实现跳转。当你看见一个 RET 时，要查看堆栈来确定你到底要跳到哪去。  
  
01013001 60 PUSHAD ; 保存所有的寄存器  
01013002 E8 03000000 CALL pnotepad.0101300A ;  
01013007 -E9 EB045D45 JMP 465E34F7 ;  
0101300C 55 PUSH EBP ;  
  
首先要注意的是：步过(step over)这些 CALL 是个坏习惯。如果你步过了 CALL pnotepad.0101300A，你会发现你来到了第一个异常处。程序这时已经经过了大约 50 个不同的 API 调用，解开了一大批的字符串……然而你却对此一无所知。下次记住，一定要步进(step into)。再看一遍这个 CALL，0101300A 在反汇编器中 \*不是\* 一条指令的开始位置，因此你将会跳进一条指令的中间。  
  
PUSHAD ; 执行过这一句  
CALL pnotepad.0101300A ; 跟进  
  
现在，你进入这个 CALL 了，你停在 0101300A：  
  
0101300A 5D POP EBP ; pnotepad.01013007  
0101300B 45 INC EBP  
0101300C 55 PUSH EBP  
0101300D C3 RETN  
  
在这个跳转之后，你眼前的所有代码都完全改变了。这是通过下面的方法做到的：  
  
pushad  
call \_below  
db E9h ; 在指令中插入垃圾数据  
db EBh ;  
db 04h ;  
\_below: pop ebp  
inc ebp  
...  
  
反汇编器在执行跳转之前把 E9 当成一个跳转指令的开始，因此把它翻译成了一条跳转指令，从而使下面所有的代码都被错误的翻译了。对此我恐怕没有什么好方法，你常常不知道下一句将要执行的指令是什么，你必须对此感到习惯（你可以执行到下一条指令然后你会发现真正的代码，但这只能做有限几次因为这是很浪费时间的）。  
  
几点建议：  
  
 + 当看到 API 函数时，查看一下它的参数（在堆栈中）  
 + 所有关于 FS:\[XXh\]的指令都和 SEH 或反调试有关  
 + 所有类似“修改\[ESI\]”（或其它寄存器）的指令，如果 ESI 指向你的目标程序代码内部，这可能是计算校验和或是在解压数据  
 + 一旦确定一个 CALL 中的代码不含有用的信息，你可以跳过它  
 + 做一个可靠断点的列表——BPX 并不麻烦——像下面这样：  
     设置第一个 BPX  
     运行你的代码直到第一个 BPX  
     清除第一个断点  
     设置第二个  
     ……  
**  
IV 自解压  
**  
让我们感受一下 Asprotect 是如何工作的。  
  
果然不出所料，在开始的地方没有任何有用的串式参考。运行程序，你会来到第一个异常处：  
  
0084377C 3100 XOR DWORD PTR DS:\[EAX\],EAX ; eax = 0  
  
现在，我们来看一看当前的串式参考。我为你挑选出来了一些：  
  
"kernel32.dll",  
"GetProcAddress",  
"Protection Error",  
"regfile",  
"Software\\ASProtect\\SpecData"  
  
前 3 个和程序调用的 IsDebuggerPresent 过程有关，"regfile"一定是和 keyfile 有关了，最后一个是 Asprotect 保存试用信息的注册表键。  
  
我们看看调试器能找到的调用：  
  
USER32.MessageBoxA  
ADVAPI32.RegQueryValueExA  
KERNEL32.GetSystemTime  
KERNEL32.CreateFileA  
KERNEL32.GetProcAddress  
KERNEL32.CreateFileA  
...  
  
它们绝对不是明显出现的，但如果我们一旦发现程序调用 CALL 的地址是连续的，我们就可以转到那里检查。正如你想得那样，有一个 AsProtect 的“IAT”。Asprotect 建立了它自己的 IAT 并把它们放在一个间接跳转表中：  
  
008350C8 -FF25 B8818400 JMP DWORD PTR DS:\[8481B8\] ; ADVAPI32.RegSetValueA  
008350CE 8BC0           MOV EAX,EAX  
008350D0 -FF25 B4818400 JMP DWORD PTR DS:\[8481B4\] ; ADVAPI32.RegSetValueExA  
008350D6 8BC0           MOV EAX,EAX  
 ....  
  
还有一件有趣的事是，如果你检查 Asprotect 的输入表你只能找到 GetModuleHandleA 和 GetProcAddress，这两个函数足够定位我们需要的其他函数了。  
  
调试——像我们想要的那样做——需要能够走出所有 Asprotect 建立的“无限”循环，下面是试图解决这个问题的方法。  
  
如何识别并走出这些循环：  
  
这些循环通常都是一些固定的模式，很容易识别。我试过多个加壳后的程序都是这样，例如：  
  
1) Push/Pop 循环。类似下面的样子：  
  
     \_iterate:  
     mov ecx, \[ebx+edx\] ; edx 是计数器，初始化为 FFFFFFFF  
                   ; ebx 指向要解密的代码的开始部分  
     ....          ; 计算新值的代码  
     push ecx      ; 压入新值  
     ... ;  
     pop \[ebx+edx\] ; 新的指令！  
     sub edx, 4    ; 计数器递减  
     ;  
     cmp edx, -320 ; 将计数器与一个负数值比较  
     jnz \_iterate  ; 重复计算  
     jmp \_continue ; 离开循环（一个跳转指令或其它指令）  
  
想走出循环，只要简单在"jmp \_continue"这句上下断，要检查是否正确算出了 EDX 的最终结果，设置一个条件断点当 EDX==最终值时中断即可。有一些其它的保护壳会产生更加“精确的循环”，因此连“jmp \_continue”这句指令也是加密的，这时你不得不设置一个 EDX==-320 的条件断点了。（译注：原文此处为 EDX=320，疑为笔误）  
  
2) 下面也是一个从加壳的记事本里找出的例子，是一个更加复杂一些的双重 push/pop 的例子：  
  
       \_RealIterate:  ; 循环开始  
       push \[ebx+ecx\] ;  
       xor ax, 80BC   ;  
       pop edx        ; edx = \[ebx+ecx\]  
       call \_down     ; 压入一个双字然后跳到下面，仍然在循环内部  
       ;  
       ....           ; 不会被执行的代码，干扰静态分析  
       ;  
\_down: add esi, 71741184 ; 无意义  
       pop esi           ; esi = CALL 之后的偏移  
       ;  
       xor ecx, 55baa1f6 ; 计算新值  
       xor edx, 181419f7 ;  
       sub edx, 2c828064 ;  
       ;  
       ...           ; 无用的计算  
       ;  
       push edx      ; 压入新值  
       ... ;  
       pop \[ebx+ecx\] ; 设置新的指令  
       ... ;  
       cmp ebx, -674 ; 检查计数器  
       jnz \_iterate  ;  
                     ; 在这里放置你的“安全”断点  
       \_iterate:  
       ...           ; 垃圾代码  
       jmp \_RealIterate  
  
可见，成对的 push \[ebx+ecx\]和 pop \[ebx+ecx\]暴露了 Asprotect，像我上面提到的一样，你可以在"jnz \_itertae"后设置一个安全的 BPX。  
  
关于循环的一般注意事项：  
  
所有的循环都是从代码的结尾部分开始倒着解密直到“几乎”达到当前的循环（注意“几乎”）。从后面开始的原因是让你不能直接在循环的后面下断，至少你要确定这不会影响解密的过程。  
  
一些壳犯了下面的错误：  
  
jnz \_offset1  
jmp \_offset2  
  
当你来到这个跳转时，通常会在调试器看到“跳转已实现”这种字样，这清楚地意味着我们进入了一个循环中。在那里设置一个断点然后继续运行，等下次又来到这里时你可以完全确定你在一个循环中了。Bpx jmp \_offset2 就会走出来。  
  
下面这种循环对抗跟踪(tracers)非常有效。只要想想跟踪要比单纯的运行多花费 N 倍的时间……试试下面代码，不跟踪的话不会影响程序的速度：  
  
or eax, eax  
\_iterate:  
inc eax  
cmp eax, 05FFFFFh  
jne \_iterate  
  
BTW：如果你发现跟踪器几乎不再前进，把它停下来观察当前的状态，手动帮助它走出循环。  
  
BTW：有时你会看到这种“罕见”的条件跳转：  
jpe xxxxxxxxh ; 如果 1 的个数为偶就跳...  
jno xxxxxxxxh ; 如果没有溢出就跳..  
我们知道，这种条件跳转比通常的“jnz”难构造得多，所以你必须注意。如果这种跳转用来实现一个循环是非常不可思议的，当心。  
  
回到 AsProtect：  
  
毋庸置疑，Asprotect 产生了许多垃圾代码来隐藏循环，然而有趣的是它产生许多像“int 20”这种很明显不可能在程序中出现的代码。它的本意是让 CRACKER 无法区分真正的代码和垃圾代码，不是吗？为什么产生"int 20"呢？我将会在下面解释如何利用这个错误越过一些"prefetch queue"。  
  
定位 KERNEL32  
  
我们上面提到了在 Asprotect 的输入表中只有 GetModuleHandleA 和 GetProcAddress 函数，因此它需要定位许多 API。Asprotect 用一些标准算法来实现这些：  
  
现在，我们把注意力集中在第三个和第四个循环之间的部分，我们可以看到非常标准的 KERNEL32 定位算法，如下所示：  
  
mov eax, \[esp+24\]     ; 这是程序堆栈中的第一个双字  
                      ; 它包含的一个指向 KERNE32 内部的指针  
and eax, FFFF0000 ;  
add eax, 10000        ; 把 EAX 对齐，准备搜索  
\_next: sub eax, 10000 ; KERNEL32 的基址必定是 10000 的整倍数  
cmp eax, ’MZ’       ; 是 ms-dos stub 吗?  
jne \_next             ; 不是，继续搜索  
                      ; OK，KERNEL32 基址找到了  
  
（译者注：关于这一部分的详细解释可以参看一些病毒教程）  
  
在 KERNEL32 的定位之后你可以看到它找到 GetProcAddress 并且用它定位一些基本的 API。  
  
定位“基本”的 API  
  
基本 API 的定位紧接于 KERNEL32 映像基址的定位之后，算法使用预先计算好的用 CRC32 处理过的函数名来从 DLL 的输出表中定位 API。你会看到如果预先计算出的 CRC32 值和它从输出表中取出的字串用 CRC32 处理过后的值一致，那么它会保存起这个地址。我们可以把这个过程大致描述如下：  
  
esi = 指向 KERNEL32 输出表里的下一个函数名的字串  
call CRC32  
cmp eax, edi        ; eax = 返回值  
                    ; edi = 我们要查找的 API 的 CRC32 值   
jne \_continueSearch ; 取输出表里的下一个函数名测试  
je \_storeOffset  
  
它现在定位的 API 通常如下：GetModuleHandleA,LoadLibraryExA,VirtualAllocEx 和 VirtualFreeEx.这是它开始时必须的 API。  
  
（如果是加壳的 Regedit，现在的 EIP==42279B）  
  
例子：  
  
006E17B1 8B33          MOV ESI,DWORD PTR DS:\[EBX\]     ; 读取要搜索的 API 名称的 CRC 值  
006E17B3 89B5 6B030000 MOV DWORD PTR SS:\[EBP+36B\],ESI ; input for the next procedure  
006E17B9 E8 0B000000   CALL XXXXXX.006E17C9           ; 找到 API  
006E17BE AB            STOS DWORD PTR ES:\[EDI\]        ; 保存 API 的地址  
006E17BF 83C3 04       ADD EBX,4                      ; 下一个要定位的 API   
006E17C2 833B 00       CMP DWORD PTR DS:\[EBX\],0       ; 还有 API 没有定位吗  
006E17C5⁷⁵ EA         JNZ SHORT XXXXXX.006E17B1      ; 是的，继续搜索  
006E17C7 61            POPAD                          ; 没有了，恢复寄存器，继续执行  
006E17C8 C3            RETN ;  
  
这样，它从 KERNEL32 中定位了“基本”API（保护它们的地址以备后面使用）  
  
注意 CRC 算法的使用，这使得我们难以知道 Asprotect 寻找的 API 是什么。不幸的是，该算法有一个安全错误：某时某处，会用一个寄存器指向 API 的名字……然后你会看到更多的名字一次次出现……很明显它在定位 API 了。修复这个“安全”问题是很困难的，如果你试图通过序号引入函数你会遇到很多麻烦（似乎最好的方法是你自己写一段程序通过序号或名称来寻找 API）。  
**  
V 向调试器抛出异常  
**  
@DAEMON: You’re gonna see, man, that there’s still some ppl in the world who haven’t read your articles...  
  
对于这部分教程，你需要了解关于 SEH 的基本知识。你可以在 DAEMON 的网站[www.anticrack.de](http://www.anticrack.de)上找到全面的指导。我猜你应该知道当一个异常发生时 SEH 被调用，但这只是程序未被调试时。如果程序正在被调试，异常将被送至调试器，然后我们自己决定该怎样做。  
  
通过引发大量的异常来反调试是必须的，这样做的好处是：  
  
 + 调试变得更加浪费时间  
 + 大多数异常必须“手动”处理，因为调试器不能正确处理它们  
 + 一些异常是非常难以理解的  
  
演示版的 ASPR 只使用了两种异常手段，我们来看一下：  
  
手段 1：改变 EIP  
  
你可以在 ASPR 建立的第一个异常处发现它：  
  
0084377C 3100 XOR DWORD PTR DS:\[EAX\],EAX  
   
在异常处理程序处下断，回想一下，在 FS:\[0\]处设置 SEH（也许你可以在调试器中查看堆栈来找到当前的异常处理程序），然后把异常传递过去。  
  
ASPR 是这样做的：  
push \_return       ; SEH 的第一条指令，你的断点应该设在这句  
inc \[esp\]          ; 返回地址+1  
ret                ; 用 RET 做跳转，我们跳转到 1+\[esp\]  
\_return+1:  
mov eax, \[esp+0Ch\] ; eax = 指向 context 结构的指针  
add \[eax+b8\],2     ; context 结构中的 EIP 加 2（参考 context 结构的定义）  
xor eax, eax       ; 准备继续异常处理  
ret                ; 从异常处理程序中跳出，应该跳到 Except1+2,  
                   ;也就是引发异常的指令"xor \[eax\],eax"（长度为 2）的下一条指令  
  
注意，EIP 加 2 会使我们来到引发异常指令的下一条指令。还要注意，在这一处时当前线程的 context 结构会在继续执行前恢复（包括已经+2 的 EIP）  
  
结论：要躲过这种手段只需把引发异常的指令 NOP 掉，继续执行即可。   
  
在这个 ANTI-DEBUG 后 ASPR 移除了当前的异常处理程序，设置了另一个来处理下一处异常：  
  
pop fs:\[0\]  
pop eax     ; ESP 加 4，EAX 是无用的寄存器  
push value1  
...  
push value7  
ret         ; 用 RET 做跳转，我们跳到 value7  
  
手段 2：改变 EIP+调试寄存器  
  
按上面的方法继续，进入 SEH 你可以看到下面的指令：  
  
seh:  
mov eax, \[eax+C\] ; 指向 context  
add \[eax+b8\],2   ; EIP = 引发异常指令的下一条指令  
                 ; 直到这里还没有什么特别的  
push ecx         ; 保护 ECX  
xor ecx, ecx     ; 准备重写调试寄存器  
mov \[eax+4\], ecx ; DR0 = 0  
mov \[eax+8\], ecx ; DR1 = 0  
mov \[eax+C\], ecx ; DR2 = 0  
mov \[eax+10\], ecx; DR3 = 0  
mov \[eax+18\], 155; DR7 = 155  
pop ecx          ; 恢复 ECX，平衡堆栈  
xor eax, eax     ; 恢复 context，从异常处理返回  
                 ; 调试寄存器被设为新值(0,...,155)  
ret              ; 我们可以继续执行下条指令   
  
注意它没有重写 DR6，我想他应该设置 DR6=0。调试寄存器是我们的调试器使用的，因此我们不能让壳乱动它们。幸运的是，壳没有在从 SEH 返回后检查调试寄存器的值，因此我们可以安全地把那几句 NOP 掉。ASPR 产生了大约 25 个手段 1 和手段 2 这样的异常（太少也太简单了）（译注：真 BT![](img/26ed9fa78f7d02ee5a28c57dc1e1e2a8.jpg)）  
  
异常处理的扩展  
  
首先，总是用完全相同的指令产生异常，这样做太天真了。比如，我们可以这样：  
  
方法 1)  
db FFFFh ; 错误的操作码  
方法 2)  
int 3    ; 调试异常  
nop      ;  
方法 3)  
xor eax, eax ; 换一个非法的内存引用（不要总是 xor \[eax\],eax）  
pop \[eax\] ;  
  
等等等等……还有，SEH 也不只用于改变 EIP。我们知道，SEH 对 context 操作时给我们类似 ring0 的权限，为什么不用它把调试和堆栈寄存器一起改变呢？  
  
push ebp  
mov ebp, esp  
mov eax, \[ebp+10h\] ; 现在 EAX 指向 CONTEXT 结构  
                   ; 首先，重写调试寄存器  
mov \[eax+4\], ecx   ; DR0 = 0  
mov \[eax+8\], ecx   ; DR1 = 0  
mov \[eax+C\], ecx   ; DR2 = 0  
mov \[eax+10\], ecx  ; DR3 = 0  
mov \[eax+18\], 155  ; DR7 = 155  
                   ; 现在，改变 ESP, EBP, EIP  
mov edx, \[new\_esp\]  
mov dword ptr \[eax+0C4h\], edx ; 当返回时改变 ESP  
mov edx, \[new\_ebp\]  
mov dword ptr \[eax+0B4h\], edx ; 当返回时改变 EBP  
mov edx, \[new\_eip\]  
mov dword ptr \[eax+0B8h\], edx ; 当返回时改变 EIP  
xor eax, eax      ; 准备返回，继续执行  
pop ebp  
ret  
  
现在，我们来看看 CRACKER 遇到的麻烦：它不但把调试寄存器重写，还修改了 ESP、EBP 和 EIP 的值。因此要清除这个异常我们不能简单地把它交给调试器处理了（令人吃惊的是，Olly 如此简单地跳过就安全过关了），而是应该像下面这样：  
  
把异常 NOP 掉  
跳到 EIP  
把 ESP、EBP 设为新值  
  
通过这些你才可以清除这些手段。其实还有更狡猾的招数（参考 DAEMON 的老窝，其实他比我更适合来写这一段）。  
**  
VI 校验和  
**  
A 文件 NOTEPAD.EXE 的校验和  
  
After touching with procdump, open+save it is enough cos it changes the last access to the file, 被 ASPR 保护的程序会检测出文件己经被改变了，程序停止运行并弹出一个对话框“文件损坏，请进行病毒检查……”。  
  
现在，我们继续经过异常（避开所有的手段）直到出现一个对话框。记下我们经过的最后一个异常，重新加载然后跟进来定位，很快就可以找到下面：  
  
00843406 3100             XOR DWORD PTR DS:\[EAX\],EAX    ; 异常  
00843408 64:8F05 00000000 POP DWORD PTR FS:\[0\]          ; 移除 SEH  
0084340F 58               POP EAX                       ;  
00843410 A1 647E8400      MOV EAX,DWORD PTR DS:\[847E64\] ; eax = 预选计算好的校验值  
00843415 3B45 FC          CMP EAX,DWORD PTR SS:\[EBP-4\]  ; ok?  
00843418 74 44            JE SHORT 0084345E             ; 正确，跳到...  
  
如果你在 00843415 这句时看一看寄存器的值，大概是下面的样子：  
EAX = D3F1E6B1, \[EBP-4\] = 989F8C34.  
  
猜出它在比较什么不是一件难事：既不是指向 DLL 的指针，也不是较小的用户变量……试着对同样的程序作不同的修改，然后比较结果（其中一个值不变——那个正确的值，另一个随机变化）。更重要的是，我们在屏幕上可以看到——就在下面几行——字符串“文件己损坏”。当然，要避开这种校验只需把 EAX 的值改为和\[EBP-4\]相同。  
  
这种校验和只是简单地计算被保护的文件 pnotepad.exe（BTW：我敢这么肯定是因为我钩挂了把文件映射到内存的 API，并且跟踪出校验和的值，你也可以自己试一下，不过这要花费很长时间）因此有人可能希望也能找到内存映像的校验值……是的，已经有人这样做了:-)  
  
BTW：D3+F1+E6+B1，98+9F+8C+34 做为校验和也是非常有效的（失败的概率为 1/100），并且这样很容易让人误以为这是一个循环的索引或是别的什么。如果你认为你需要更精确的检查那么使用其中的 4 个，毕竟 D3F1E6B1 和 989F8C34 太明显了。  
  
列举所有的检验和是非常冗长的，因此我们只简单地讲述与其相关的核心思想。  
  
B 内存映象的校验和  
  
我必须承认这一部分只是因为觉得有趣而写的，因为想上校验和的圈套是非常不容易的。（在用完 BPX 后去除它们，这就足够了）  
  
首先是一个简单的问题：在跳转到被保护的程序之前，你不认为我们应该检查它的内存映象吗？是的，看上去是一个好主意。演示版的 ASPR 的确这样做了。为了找到这处校验或其它的校验，我们改变内存映象并运行程序，在最后一个异常之前，我们来到了另一个异常处：“无法执行因为内存\[xxxxxxxxh\]不可读。”  
  
所以我们再次执行程序直到出错之前，调试进去……我们需要找到计算校验和或其它类似的地方，然后与旧的（己经存好的）校验值比较。ASPR 做的是计算好几个它的内存映象的校验和来比较，如果发现有改变，程序就不能从下面的指令中正常返回：  
  
xor \[esp\], eax ; 与计算出的校验和 XOR 运算  
ret            ; 如果改变就会 crash  
由此可见，因为校验和的值是随着你对内存映象的修改而随时改变的，所以当然会出现“随机”的错误。  
  
为了找到这些校验和在哪儿，我把修改后的程序和未修改的程序都进行调试，我建议你遇到这类问题时也这样做。正如上面所说，重要的是移除那些为了跳出循环之类而设置的断点，这样 ASPR 就不会发现任何改变。(neither you see the trick, that will remain hybernating for the next time).  
  
现在我们来看看这种保护是如何进行的，下面是我曾经提到的校验和中的一个（有 3 处调用这个过程）：  
  
0083AEF4 68 D76A3C93   PUSH 933C6AD7  
0083AEF9 68 DC150000   PUSH 15DC  
0083AEFE 68 14990000   PUSH 9914  
0083AF03 68 00A00100   PUSH 1A000  
0083AF08 FF35 D4748400 PUSH DWORD PTR DS:\[8474D4\]  
0083AF0E E8 F5E2FFFF   CALL 00839208  
  
你可以看到它的校验值，在下面这一句下断：  
  
0083923D 66:8B13 MOV DX,WORD PTR DS:\[EBX\]  
  
并且检查 EBX 的内容。因此，这个 CALL 在 EAX 中返回内存映象的校验和，然后像下面这样做：  
  
0083AF13 310424        XOR DWORD PTR SS:\[ESP\],EAX  
0083AF16 8B05 D4748400 MOV EAX,DWORD PTR DS:\[8474D4\]  
0083AF1C 010424        ADD DWORD PTR SS:\[ESP\],EAX  
0083AF1F C3            RETN  
  
如果校验和不正确，这时就会出现混乱。  
**  
VII 更多的反调试：IsDebuggerPresent, FS:\[18\]……  
**  
如果程序没有调用 IsDebuggerPresent，你难道不感觉有点失望吗？我也是……  
  
有时候，你会被提示一个对话框“检测到调试器……”。所以我们从最后的异常开始向前寻找我们在什么地方被骗了：  
  
008434B8 3100        XOR DWORD PTR DS:\[EAX\],EAX ; 出现对话框前的最后一个异常  
008434BA EB 01       JMP SHORT 008434BD  
008434BC 68 648F0500 PUSH 58F64  
  
如果你有点耐心的话，过一会你会来到这里：  
  
00840F1E 68 8C0F8400 PUSH 840F8C   ; ASCII "kernel32.dll"  
00840F23 E8 3842FFFF CALL 00835160 ; JMP to kernel32.GetModuleHandleA  
00840F28 8BD8        MOV EBX,EAX  
  
这是简单地得到 KERNEL32 的基址，下面：  
  
00840F2D B8 A40F8400 MOV EAX,840FA4 ; ASCII "HrFgavcc‘wVtbtmf}"  
00840F32 E8 E5FDFFFF CALL 00840D1C  
  
这是一个解密的过程，现在注意下面的事实：  
HrFgavcc'wVtbtmf} = 17 个字符   
IsDebuggerPresent = 17 个字符  
  
发现了什么？给 API 名字加上一些修饰是一个不错的主意，这样它的长度就不会引起我们的怀疑。  
  
00840F3F 50 PUSH EAX               ; API 名称字串的偏移  
00840F40 53 PUSH EBX               ; KERNEL32 的基址  
00840F41 E8 2242FFFF CALL 00835168 ; JMP to kernel32.GetProcAddress  
  
最后，它检查这个 API 是否被找到，如果找到了就调用它：  
  
00840F46 8BF8  MOV EDI,EAX         ; kernel32.IsDebuggerPresent  
00840F48 89FE  MOV ESI,EDI  
00840F4A 85FF  TEST EDI,EDI  
00840F4C 74 06 JE SHORT 00840F54  
00840F4E FFD6  CALL ESI            ; call IsDebuggerPresent  
  
躲过这个 IsDebuggerPresent 的调用是非常简单的，只要把返回值改为 0 或简单的给 KERNEL32 的这个 API 打补丁（在 WinXP 下这个 API 是这样做的）：  
  
MOV EAX,\[DWORD FS:18\]  
MOV EAX,\[DWORD DS:EAX+30\]  
MOV EAX, 0 ; 在这里打补丁  
RETN  
  
加入其它的反调试  
  
正如你所见，这个演示版的反调试并不是很强。你应该考虑加入新的反调试手段，但是应该像下面这样正确的加入：  
  
当分析一个实际目标程序时我遇到了这样的异常：  
  
CALL 010309E0  
MOV ECX,DWORD PTR DS:\[EBX+8\]  
ADD ECX,108  
MOV EAX,ECX  
MOV EDX,DWORD PTR DS:\[EAX\] ; 异常，EAX 不可读！  
  
因此我立刻想到了 SEH。因为我没有查看堆栈，我就去检查 FS:\[0\]，但是我非常吃惊地看到它是指向 KERNEL32 的。为什么？很明显，如果 ASPR 在这里设置了一个 SEH 反调试，那么 SEH 必定安装在某个地方；如果这不是一个 SEH 反调试手段，这条指令本来不应该被执行的（就算执行至少不是这个 EAX 值），因此我们肯定在什么地方被愚弄了。现在是返回去卷土重来（从上个异常开始）的时候了。  
  
mov eax, fs:\[30\]  
movzx eax, byte ptr \[eax+2\]  
or al, al  
je ...  
  
你必须承认这有点令人震惊，不是吗？要测试这是不是正确的反调试手段，最好的方法是自己写一个这样的程序来看当你在调试和不在调试时会有什么发生。因为这是第一次，我会给出这种反调试的汇编源码（TASM 编译）：  
  
.386  
.Model Flat ,StdCall  
  
extrn ExitProcess: PROC  
extrn MessageBoxA: PROC  
  
.Data  
  
szZeroe db ’zeroe’,0  
szNotZeroe db ’NOT zeroe’,0  
  
.Code  
  
Main:  
push 0    ; 为了消息框  
push 0  
mov eax, fs:\[30h\]  
movzx eax, byte ptr \[eax+2\]  
or al, al ; 检测返回值  
je \_zeroe  
push offset szNotZeroe  
jmp \_MessageBoxA  
\_zeroe:  
push offset szZeroe  
\_MessageBoxA:  
push 0  
call MessageBoxA  
push 0  
call ExitProcess  
End Main  ; 代码结束，Main 是入口点  
  
现在，运行这个程序你会看到 AL 满足：  
  
al = 0 未被调试 Not Debugged  
al = 1 正被调试 Debugged  
  
因此为了能继续调试下去，你必须清除 EAX 的值为 0。你还有其它的基于 TEB（或 PEB）的反调试手段——像通常一样，搜索 Deamon's cave——你可以把它插入你的代码，这是其中最简单的一个，和 IsDebuggerPresent 做的一样：  
  
mov eax, fs:\[18h\]  
mov eax, \[eax+30h\]  
movzx eax, byte ptr \[eax+2\]  
test eax, eax  
je ...  
  
像上面一样做，你可以发现：EAX=0 未被调试；EAX=1 正被调试  
  
一些建议：  
  
 + 当检测出你的程序正被调试时，没有必要立刻做出反应，先等一会儿（否则就意味着我可以精确地知道从哪里开始寻找）另外，不停地设置 SEH——为了迷惑 CRACKER——并且检查是好的（由你引发的）或是坏的  
 + 使用一个反静态分析方法来隐藏基于 FS:\[xx\]的反调试，当我第一次看到这种情况时我几乎犯了心脏病（没关系，尽管这样做，我还很年轻，死不了）  
 + 你应该在不同的地方放置同样的反调试手段……有时只是为了烦死你的 CRACKER……  
**  
VIII 钩子** （译注：对这一部分没有很理解，请高手指出错误的地方）  
  
在这一节我们将试着理解保护壳的内部，因为这要占用很大的篇幅，我们只限于分析对注册表的操作（和时间限制密切相关）。我们知道找出所有的反调试是非常麻烦的，其实只要观察目标程序的输入表——还有动态加载的 DLL——然后钩挂它们。在阅读这一节时，最好打开注册表编辑器，自己来检验一下。  
  
如何去做：  
  
比如你对时间限制有兴趣，你应该钩挂该壳引入的所有可以从读取文件和注册表的 API，以此来获取一些信息。下面是在这个演示版的 ASPR 中找到的东东：  
  
Advapi32:  
RegCloseKeyA  
RegEnumKeyExA  
RegOpenKeyExA  
RegQueryInfoKeyExA  
RegQueryValueExA  
RegSetValueA  
RegSetValueExA  
  
现在，你只要让程序运行，然后记下你看到的：  
  
调用 RegOpenKey，用 KEY\_ALL\_ACCESS 权限打开 HKEY\_CURRENT\_USER\\Software\\Borland\\Locales  
调用 RegOpenKey，用 KEY\_ALL\_ACCESS 权限打开 HKEY\_CURRENT\_USER\\Software\\Delphi\\Locales  
（如果它们不在你的系统中没有关系）  
  
调用 RegOpenKeyExA，用 KEY\_READ 权限打开 HKEY\_CURRENT\_USER\\Software\\Asprotect\\SpecData  
调用 RegQueryValueExA, 子键="A93471655372341".这个调用中 Buffer = NULL 并且 pBufferSize != NULL，这将会返回该键值的长度。注意"A93471655372341"已经被计算好了。  
调用 RegOpenKeyExA，用 KEY\_READ 权限打开 HKEY\_CURRENT\_USER\\Software\\Asprotect\\SpecData  
调用 RegQueryValueExA，子键="A93471655372341". 这一次这个 CALL 读出的键值。  
...  
调用 RegSetValueA,为 HKEY\_CURRENT\_USER\\Software\\Asprotect\\SpecData 设一个新值  
调用 RegSetValueA,把 HKEY\_CURRENT\_ROOT\\.key 的值设为空。为什么为空？只是因为想设置一个新值所以删除了它。   
调用 RegOpenKeyExA, 用 KEY\_SET\_VALUE 权限打开 HKEY\_CURRENT\_ROOT\\.key  
调用 RegSetValue,把 HKEY\_CURRENT\_ROOT\\.key 的值设为"regfile"  
...  
  
BTW：有时直接在系统 DLL 上设置钩子，你会得到类似下面的信息：0006F8D8 772AD1DA /CALL to RegOpenKeyExA from SHLWAPI.772AD1D4 很明显，你对这个不感兴趣，因为我们只想知道从保护壳中的调用  
  
我不想把这一节写得太长，但是，在对注册表的操作上设置钩子，你可以得到像下面这样的信息：ASPR 使用 HKEY\_CURRENT\_USER\\Software\\Asprotect\\SpecData 来保存注册信息，这个值每当目标程序运行时都会更新（ASPR 需要这样做）。这个键值是加密的，我们也可以发现它的保护很弱，比如：解除时间限制是非常简单的，只需要删除这个键然后钩挂 GetSystemTime 和 GetLocalTime 函数。我们也可以钩挂全部的注册表键。当然，如果你知道如何脱壳你完全不需要这些。![](img/620448a77d884bd6b2c4a7cfef437501.jpg)  
  
钩挂 GetModuleHandle 也是很有趣的：  
  
0006FF64 0084C07A /CALL to GetModuleHandleA from 0084C074 ; 寻找 kernel32 的基址  
0006FF68 0084C79C \\pModule = "kernel32.dll"  
...  
0006FF64 0084C4AC /CALL to GetModuleHandleA from 0084C4A6 ; 寻找 user32 的基址  
0006FF68 008484BC \\pModule = "user32.dll"  
...  
0006FE24 0084129F /CALL to GetModuleHandleA               ; 本程序的基址  
0006FE28 00000000 \\pModule = NULL  
  
从 GetModuleHandleA 返回后向下一点（我们在 ASPR 中）：  
  
0084129F 68 AA128400 PUSH 8412AA                   ;  
008412A4 E9 85000000 JMP 0084132E                  ;  
0084132E 5A          POP EDX                       ; 008412AA  
0084132F 5B          POP EBX                       ;  
00841330 68 37138400 PUSH 841337                   ;  
00841335 C3          RETN                          ; 用 RET 做跳转  
00841337 8943 F6     MOV DWORD PTR DS:\[EBX-A\],EAX  ; pnotepad.01000000  
  
\[EBX-A\]是在 ASPR 里的一个偏移量。稍后，如果我们在目标程序已经解压的代码里看见相同的值——通过进行查找——我们就知道这是一个对 GetModuleHandleA 的调用。更耐心一点的话，我们可以找到一个已经在解压后的目标程序里的 CALL，返回到：  
  
00841483 5D            POP EBP                ; restore ebp  
00841484 C2 0400       RETN 4                 ; jmp to NOTEPAD.01006C4E  
...  
01006C4C . FFD7        CALL EDI               ; we come from this call  
01006C4E . 50          PUSH EAX               ; |Arg1 = 01000000  
01006C4F . E8 ADBBFFFF CALL pnotepad.01002801 ; \\pnotepad.01002801  
  
现在，钩挂“CALL EDI”你会看见它实际上是“CALL 00841460”，我们找到那里：  
  
00841460 55                 PUSH EBP  
00841461 8BEC               MOV EBP,ESP  
00841463 8B45 08            MOV EAX,DWORD PTR SS:\[EBP+8\]  
00841466 85C0               TEST EAX,EAX  
00841468 75 13              JNZ SHORT 0084147D  
0084146A 813D 787A8400 0000 CMP DWORD PTR DS:\[847A78\],400000  
00841474 75 07              JNZ SHORT 0084147D  
00841476 A1 787A8400        MOV EAX,DWORD PTR DS:\[847A78\]  
0084147B EB 06              JMP SHORT 00841483  
0084147D 50                 PUSH EAX  
0084147E E8 DD3CFFFF        CALL 00835160 ; JMP to kernel32.GetModuleHandleA  
00841483 5D                 POP EBP  
00841484 C2 0400            RETN 4  
  
这是一个对 GetModuleHandle 的调用，还有两处检测 ASPR 是否还在。只要把这个调用换为 mov eax,\[预选计算好的模块句柄\]就可以搞定。  
**  
IX. 如何找到程序入口点  
**  
A. ASPROTECT 演示版  
  
到目前为止，你有了足够的信息来躲过所有的反调试：校验和，IsDebuggerPresent，当然还有异常。因此你应该数数到程序完全加载起来时异常的数量，从最后一个异常处重新开始调试，寻找远距离跳转。  
  
关于入口点的一般注意事项：  
  
这是非常明显的，但是我们作一些简单的观察。  
  
1) 许多程序在最开始处有下面的指令：  
  
   push ebp  
   mov ebp, esp  
   sub esp, ?? ; 在堆栈中为局部变量留出空间  
   push ebp    ; 保存寄存器，你可以在 DELPHI 程序中找到此类语句  
   push esi  
   push edi  
  
2) 在程序开始处你应该看到一些著名的 CALL（在按了调试器上的“分析”按钮这后），比如：  
  
   在 kernel32, user32...中  
  
     GetModuleHandleA  
     FindWindow  
     ShowWindow  
     GetVersion(Ex)  
     GetCommandLine  
     ...  
  
   在 MSVCRT (Microsoft Visual C++ Runtime Library) 中  
     \_\_set\_app\_type  
     \_\_p\_\_f\_mode  
     \_\_setusermatherr  
     ...  
  
   （请你自己补充完善）  
  
3) 我们要找到长距离跳转，这种跳转可以用很多方法来实现，比如：  
  
   mov eax, \[variable\] ; 预先算出的入口点  
   push eax            ; eax = 401038h  
   ret                 ; 压入返回地址，用 RET 来实现跳转  
  
   你还会在附近发现一些 POPAD，也许就紧挨在前面，这是为了在跳到真正的入口点时恢复所有的寄存器。  
  
4) 有可能在一个长距离跳转后，我们仍然没有来到入口点（试着用在 2)中提到的方法辨别一下）。在这种情况下，可能是壳放置了一段加密过的程序来迷惑你，在解密了一些东西后你会跳到真正的入口点。（这段解密程序可能在 PE 的空白处，也就是说，在区块头和代码开始处之间）  
  
5) 因为你已经准确地知道所有的反调试，你可以用调试器的“tracer”功能，这将会节省很多时间（如果不用的话可能要花 30 分钟找到入口点）。  
在 OLLY 中，可以像下面这样做：   
  
调试，设置条件，EIP 在某个范围内  
然后开始跟踪(Trace Into)  
  
案例一：查找注册表编辑器\[REGEDIT\]的入口点（ASPR 演示版，WIN98）  
  
当然，你不应该知道未保护的 REGEDIT 的信息，因为在实际情况中你不可能知道。好啦，像我说的那样，来到最后一个异常处并且 NOP 掉它。现在，我们开始调试直到找出正确的入口点\[EP\]。一些提示：  
  
  不要注意任何短跳转  
  不要注意任何近距离的 CALL  
  当你来到一个 RET 时，注意\[ESP\]  
  记着关于循环的提示  
  
这是最后一个异常（你在堆\[HEAP\]中）：  
  
00BC3033 3100             XOR DWORD PTR DS:\[EAX\],EAX  
00BC3035 64:8F05 00000000 POP DWORD PTR FS:\[0\]  
00BC303C 58               POP EAX  
00BC303D 833D 847EBC00 00 CMP DWORD PTR DS:\[XXXXXXXXh\],0  
00BC3044 74 14            JE SHORT 00BC305A  
  
重要提示：ASPR 的多态性并不很强，你可以在每个目标程序中找到同样的“开始”（包括 ASPR 的完全版）  
  
NOP 掉“xor \[eax\],eax”，开始调试。为了节省时间我们可以步过一些 CALL，比如：  
  
00BC3050 BA 04000000 MOV EDX,4  
00BC3055 E8 6EDAFFFF CALL 00BC0AC8 ; 步过  
...                                ; 步过显示“这是未注册版的……”消息框的 CALL  
00BD75FF 58 POP EAX  
00BD7600 E8 0A000000 CALL 00BD760F ; 步过  
  
这时，你会发现 REGEDIT 加载了，因此我们必须\*进入\*最后一个 CALL（用这种方法可以节省很多时间:-)）如果你进入这个 CALL，必定会发现下面这个 CALL：  
  
00BD61C8 81C6 8A7BA105 ADD ESI,5A17B8A  
00BD61CE E8 0E000000   CALL 00BD61E1 ; 也要进入  
  
现在，找出一个解密循环不是很困难了：  
  
00BD5B90 FF341A        PUSH DWORD PTR DS:\[EDX+EBX\]  
...  
00BD5BDE 893413        MOV DWORD PTR DS:\[EBX+EDX\],ESI  
...  
00BD5BE6 83EA 04       SUB EDX,4     ; 比较  
00BD5BE9 0FBFC8        MOVSX ECX,AX  
00BD5BEC 81FA 54EBFFFF CMP EDX,-14AC ; 标准循环  
00BD5BF2 0F85 19000000 JNZ 00BD5C11  
  
正如你所见，非常标准。然后，在这个解密循环之后来到了最后的跳转：  
  
00BD5C85 894424 1C MOV DWORD PTR SS:\[ESP+1C\],EAX ; AREGEDIT.0040B747  
00BD5C89 61        POPAD  
00BD5C8A 50        PUSH EAX  
00BD5C8B C3        RETN  
  
“mov \[esp+1c\],eax”是为了在 POPAD 后保护 EAX 的值。你现在在入口点了：  
  
0040B747 /. 55      PUSH EBP  
0040B748 |. 8BEC    MOV EBP,ESP  
0040B74A |. 83EC 1C SUB ESP,1C  
0040B74D |. 53      PUSH EBX  
0040B74E |. 56      PUSH ESI  
  
按“分析”，结果非常明显了。不过我警告你：ASPR 的完全版对入口点的保护强得多，我们待会儿会看到。  
  
案例二：查找记事本的入口点（ASPR 演示版，WINXP）  
  
像我们上面做的一样，来到最后一个异常处：  
  
00843033 3100             XOR DWORD PTR DS:\[EAX\],EAX  
00843035 64:8F05 00000000 POP DWORD PTR FS:\[0\]  
0084303C 58               POP EAX  
0084303D 833D 847E8400 00 CMP DWORD PTR DS:\[847E84\],0  
00843044 74 14            JE SHORT 0084305A  
...  
0084306B 6A 00            PUSH 0  
0084306D E8 5E21FFFF      CALL 008351D0                  ; JMP to USER32.MessageBoxA  
...  
0085A2A8 81C6 D20FB455    ADD ESI,55B40FD2  
0085A2AE E8 08000000      CALL 0085A2BB                  ; 进入  
...  
0085A2D6 8BC2             MOV EAX,EDX  
0085A2D8 FF343A           PUSH DWORD PTR DS:\[EDX+EDI\]    ; 开始解密  
...  
0085A309 890C17           MOV DWORD PTR DS:\[EDI+EDX\],ECX ; 结束解密  
...  
0085A316 81FA 54EBFFFF    CMP EDX,-14AC                  ; 循环的最终比较  
0085A31C 0F85 0F000000    JNZ 0085A331   
...  
0085A39A 894424 1C        MOV DWORD PTR SS:\[ESP+1C\],EAX  ; pnotepad.01006AE0  
0085A39E 61               POPAD                          ; 恢复所有的寄存器  
0085A39F 50               PUSH EAX   
0085A3A0 C3               RETN                           ; 跳转到记事本  
...   
  
01006AE0 . 6A 70       PUSH 70                ; 入口点！  
01006AE2 . 68 88180001 PUSH pnotepad.01001888 ;  
01006AE7 . E8 BC010000 CALL pnotepad.01006CA8 ;  
  
现在我们知道如何得到任一目标的入口点了。GREAT！想 DUMP 这个文件只需简单地把第一条指令改为一个无限循环，对于记事本来说改为"jmp 01006AE0"，然后运行它。进入 ProcDump 就可以 DUMP 了，做完以后记得把入口点处从无限循环改为原来的指令。  
  
当然，隐藏 EP 可以做得很好，我们看一个例子。首先要注意的是在这个例子中有一个和演示版 ASPR 相同的“退出”指令，但这只是用来迷惑你的……很好的心理战：  
  
0105E6D3 75 07     JNZ SHORT 0105E6DC        ; 跳转总会实现  
0105E6D5 894424 1C MOV \[DWORD SS:ESP+1C\],EAX ; 看到了吗？？  
0105E6D9 61        POPAD  
0105E6DA 50        PUSH EAX  
0105E6DB C3        RETN  
  
你可能天真地在 mov \[esp+1c\],eax 这句下断然后运行……但是失败了。其实，很明显这是一个骗局因为前面的条件跳转使这一句不可能达到，这是应该避免的。在同一个目标程序中还有一些非常有趣的变形代码来保护入口点，例如：  
  
0105E846 8D6424 4E LEA ESP,\[DWORD SS:ESP+4E\]  
0105E84A F3:       PREFIX REP:         ; Superfluous prefix  
0105E84B EB 02     JMP SHORT 0105E84F  
...  
01056BB6 F3:       PREFIX REP:         ; Superfluous prefix  
01056BB7 EB 02     JMP SHORT 01056BBB  
01056BB9 CD 20     INT 20  
  
当摆弄 ESP 时注意如果 ESP-X 不指向一个有效的地址，WIN9X 就会出错。我无法告诉你更多了，因为我现在在使用 WINXP，但是当然在 board.anticrack.de 上有一个关于此的非常有趣的讨论（查找 Merlin 和 Drizz，大约是十二月一号）。继续，你还会来一个有趣的地方：  
  
01056BF2 66:8135 FB6B0501 E1EF XOR WORD PTR DS:\[1056BFB\],0EFE1  
01056BFB 0AED                  OR CH,CH  
01056BFD CD 20                 INT 20  
  
你可以发现 1056FB 正是当前指令的\*下一句\*，当它被 XOR 运算后变成了：  
  
01056BFB EB 02 JMP SHORT 01056BFF  
  
我想你已经听说过 prefetch-queue。（如果没有，找相关的资料读一读）。这是吗？回答是否定的，这其实非常容易构造。假设指令是“按他们的样子”执行的，那么：  
  
XOR dword ptr \[\_next\],0EFE1 ; 因为在 CPU cache 里，不会被真正执行  
\_next: OR ch, ch            ; 被执行 executed  
INT 20                      ; 特权指令！在所有的 WinNT 里都会出错  
  
BTW：在奔腾系列 CPU 上正确地写出一个 Profetch queue 是很困难的，Profetch queue 和以前不同了，它不再精确地保存下一条指令。  
  
再看看 FS:\[0\]……没有设置 SEH，也没有更多的反调试（确定你是从最后一处异常开始的）。继续下去会来到下面：  
  
01056AF9 9D                  POPFD                              ; 仍然在壳里  
01056AFA 5F                  POP EDI  
01056AFB 59                  POP ECX  
01056AFC C3                  RETN                               ; 远距离跳到 004072DC  
...  
04072DC  -FF25 1C436200      JMP DWORD PTR DS:\[62431C\]          ; 跳到 01041C64  
...  
01041C64 55                  PUSH EBP                           ; 我们现在在壳里  
01041C65 8BEC                MOV EBP,ESP ;  
01041C67 8B45 08             MOV EAX,DWORD PTR SS:\[EBP+8\]       ; we move zeroe!!  
01041C6E 813D A47A0401 0000> CMP DWORD PTR DS:\[1047AA4\],400000  ; ASCII "MZP"  
01041C78 75 07               JNZ SHORT 01041C81                 ; 跳转不会实现  
01041C7A A1 A47A0401         MOV EAX,DWORD PTR DS:\[1047AA4\]     ; \[1047AA4\] = 400000  
01041C7F EB 06               JMP SHORT 01041C87  
01041C87 5D                  POP EBP  
01041C88 C2 0400             RETN 4                             ; 跳到 004073B1  
  
这是一个简单的骗局来迷惑我们，外壳只跳到我们的代码中一小会儿就跳了回去检查一些值。我们跟踪这个“RET”：  
  
004073B1 |. A3 68066200 MOV DWORD PTR DS:\[620668\],EAX ; target.00400000  
004073B6 |. A1 68066200 MOV EAX,DWORD PTR DS:\[620668\]  
004073BB |. A3 D0E06000 MOV DWORD PTR DS:\[60E0D0\],EAX  
004073C0 |. 33C0        XOR EAX,EAX  
004073C2 |. A3 D4E06000 MOV DWORD PTR DS:\[60E0D4\],EAX  
004073C7 |. 33C0        XOR EAX,EAX  
004073C9 |. A3 D8E06000 MOV DWORD PTR DS:\[60E0D8\],EAX  
004073CE |. E8 C1FFFFFF CALL target.00407394  
004073D3 |. BA CCE06000 MOV EDX, target.0060E0CC  
004073D8 |. 8BC3        MOV EAX,EBX  
004073DA |. E8 25D6FFFF CALL target.00404A04  
004073DF |. 5B          POP EBX  
004073E0 \\. C3          RETN  
  
现在对于我们是一个至关重要的时刻：我们必须确定哪里是入口点。为了达到目的，我们需要跟进——是的，再来一次——这些调用 CALL target.00407394 和 CALL target.00404A04 来确定是不是又一个骗局。另外，好奇心也会很有帮助：如果入口点是 004073B1，为什么它是在一个过程的中间呢？我们来来在 004073B1 上面有什么：  
  
004073A0 /$ 53          PUSH EBX  
004073A1 |. 8BD8        MOV EBX,EAX  
004073A3 |. 33C0        XOR EAX,EAX  
004073A5 |. A3 C4E06000 MOV DWORD PTR DS:\[60E0C4\],EAX  
004073AA |. 6A 00       PUSH 0  
004073AC |. E8 2BFFFFFF CALL target.004072DC  
  
现在有两个问题：004073A0 是从其他地方调用过来的吗？不是（如果这样我几乎不能相信这就是入口点了）。那么 CALL target.004072DC 呢？跟踪一下：  
  
004072DC $-FF25 1C436200    JMP DWORD PTR DS:\[62431C\] ; \[62431C\] = 01040C64  
  
01041C64 55                 PUSH EBP  
01041C65 8BEC               MOV EBP,ESP  
01041C67 8B45 08            MOV EAX,DWORD PTR SS:\[EBP+8\]  
01041C6A 85C0               TEST EAX,EAX  
01041C6C 75 13              JNZ SHORT 01041C81  
01041C6E 813D A47A0401 0000 CMP DWORD PTR DS:\[1047AA4\],400000 ; ASCII "MZP"  
01041C78 75 07              JNZ SHORT 01041C81  
01041C7A A1 A47A0401        MOV EAX,DWORD PTR DS:\[1047AA4\]  
01041C7F EB 06              JMP SHORT 01041C87  
01041C81 50                 PUSH EAX  
01041C82 E8 3135FFFF        CALL 010351B8 ; JMP to kernel32.GetModuleHandleA  
01041C87 5D                 POP EBP  
01041C88 C2 0400            RETN 4  
  
调用 GetModuleHandleA 得到目标程序的基址，因此，入口点是 004073A0 但是我们必须把 CALL target.004072DC 改为 CALL GetModuleHandleA。这是一个非常好的方法……  
  
通过继续调试你可以确定不会再跳回壳中，因此可以确定现在一直在程序代码里了。另外，像我前面说的，你应该能看到对常用 API 的调用，很可能所有的伪装 API 调用已经被壳加载了（使用 GetProcAddress 或其他方法）。  
  
  
（未完待续，持续更新，敬请关注）  

 4dd466d0e1e0cc123afbe504da2462ca