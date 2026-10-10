

# 为没有源代码的软件改 Bug -- 聊天记录察看器除错手记（高手略过）

> 原文：[`www.pediy.com/kssd/pediy10/64942.html`](https://www.pediy.com/kssd/pediy10/64942.html)

我想，凡是来这儿的人都有一种潜意识，那就是软件的加密只是被软件作者用于保护他们的利益，和真正的软件的功能没有太大关系；我们解密只不过是让这些和我们作对的保护失去作用。当我们有了一套“有权”使用的软件后，如果它运行出错，我们通常要么将错误报告给作者，要么停止使用它。我们通常不会想到去改进软件，毕竟，我们没有源代码呀。  
  
其实，没有源代码一样也可以改进。我们不妨将软件的保护看成一种对作者有用的 Bug，我们可以去掉它，对于运行错误的 Bug，我们不是一样可以去掉吗？前不久，有朋友想看以前的聊天记录，可是他忘了 QQ 密码，怎么也登陆不上，他的唯一选择就是用聊天记录察看器来看了。看着看着，突然程序弹出一个严重错误对话框，说发生严重错误，访问了非法地址。反复试了几遍都是一样。他问我有没有什么办法。我说报告作者吧，他说，作者似乎停止了此软件的开发。我抱着试试看的想法，用 IDA 反汇编了这个小程序，然后在 IDA 中以调试的方式运行它，重复着朋友的操作。果然，错误再次出现，但是被 IDA 的调试器所捕获，它将我带到如下的函数和地址：  
  
CODE:004855D8 \_TQQMsgView\_ShowOneMsg proc near        ; CODE XREF: \_TQQMsgView\_ShowMsgPage+148p  
CODE:004855D8                                         ; DATA XREF: CODE:00483BC8o  
CODE:004855D8  
CODE:004855D8 var\_8A4         = dword ptr -8A4h  
CODE:004855D8 var\_8A0         = dword ptr -8A0h  
CODE:004855D8 var\_89C         = dword ptr -89Ch  
CODE:004855D8 var\_898         = dword ptr -898h  
CODE:004855D8 var\_894         = dword ptr -894h  
CODE:004855D8 var\_890         = dword ptr -890h  
CODE:004855D8 var\_886         = byte ptr -886h  
CODE:004855D8 var\_85          = byte ptr -85h  
CODE:004855D8 var\_20          = dword ptr -20h  
CODE:004855D8 var\_1C          = dword ptr -1Ch  
CODE:004855D8 var\_18          = dword ptr -18h  
CODE:004855D8 var\_14          = dword ptr -14h  
CODE:004855D8 var\_10          = dword ptr -10h  
CODE:004855D8 var\_C           = dword ptr -0Ch  
CODE:004855D8 var\_8           = dword ptr -8  
CODE:004855D8 var\_4           = dword ptr -4  
CODE:004855D8  
CODE:004855D8                 push    ebp  
CODE:004855D9                 mov     ebp, esp  
CODE:004855DB                 add     esp, -8A4h  
CODE:004855E1                 xor     ecx, ecx  
CODE:004855E3                 mov     \[ebp+var\_8A4\], ecx  
CODE:004855E9                 mov     \[ebp+var\_8A0\], ecx  
CODE:004855EF                 mov     \[ebp+var\_89C\], ecx  
CODE:004855F5                 mov     \[ebp+var\_898\], ecx  
CODE:004855FB                 mov     \[ebp+var\_894\], ecx  
CODE:00485601                 mov     \[ebp+var\_890\], ecx  
CODE:00485607                 mov     \[ebp+var\_20\], ecx  
CODE:0048560A                 mov     \[ebp+var\_C\], ecx  
CODE:0048560D                 mov     \[ebp+var\_10\], ecx  
CODE:00485610                 mov     \[ebp+var\_8\], edx  
CODE:00485613                 mov     \[ebp+var\_4\], eax  
CODE:00485616                 xor     eax, eax  
CODE:00485618                 push    ebp  
CODE:00485619                 push    offset loc\_485946  
CODE:0048561E                 push    dword ptr fs:\[eax\]  
CODE:00485621                 mov     fs:\[eax\], esp  
CODE:00485624                 lea     eax, \[ebp+var\_85\]  
CODE:0048562A                 xor     ecx, ecx  
CODE:0048562C                 mov     edx, 65h  
CODE:00485631                 call    @Windows@FillMemory$qqrpvuiuc ; Windows::FillMemory(void \*,uint,uchar)  
CODE:00485636                 lea     eax, \[ebp+var\_886\]  
CODE:0048563C                 xor     ecx, ecx  
CODE:0048563E                 mov     edx, 65h  
CODE:00485643                 call    @Windows@FillMemory$qqrpvuiuc ; Windows::FillMemory(void \*,uint,uchar)  
CODE:00485648                 lea     ecx, \[ebp+var\_C\]  
CODE:0048564B                 lea     edx, \[ebp+var\_10\]  
CODE:0048564E                 mov     eax, \[ebp+var\_8\]  
CODE:00485651                 mov     eax, \[eax\]  
CODE:00485653                 call    sub\_485150  
CODE:00485658                 cmp     ds:dword\_488850, 0  
CODE:0048565F                 jz      short loc\_485677  
CODE:00485661                 cmp     ds:dword\_488850, 1  
CODE:00485668                 jz      short loc\_485677  
CODE:0048566A                 cmp     ds:dword\_488850, 4  
CODE:00485671                 jnz     loc\_48574D  
CODE:00485677  
CODE:00485677 loc\_485677:                             ; CODE XREF: \_TQQMsgView\_ShowOneMsg+87j  
CODE:00485677                                         ; \_TQQMsgView\_ShowOneMsg+90j  
CODE:00485677                 mov     eax, \[ebp+var\_8\]  
CODE:0048567A                 add     eax, 5  
CODE:0048567D                 mov     \[ebp+var\_14\], eax  
CODE:00485680                 mov     eax, \[ebp+var\_14\]  
CODE:00485683                 mov     eax, \[eax\]  
CODE:00485685                 mov     \[ebp+var\_18\], eax  
CODE:00485688                 add     \[ebp+var\_14\], 4  
CODE:0048568C                 lea     eax, \[ebp+var\_85\]  
CODE:00485692                 mov     ecx, \[ebp+var\_18\]  
CODE:00485695                 mov     edx, \[ebp+var\_14\]  
CODE:00485698                 call    sub\_407160      <---其实是内存复制函数，将其改成 MemCopy  
CODE:0048569D                 mov     eax, \[ebp+var\_18\]  
CODE:004856A0                 add     \[ebp+var\_14\], eax  
CODE:004856A3                 mov     eax, \[ebp+var\_14\]  
CODE:004856A6                 mov     eax, \[eax\]  
CODE:004856A8                 mov     \[ebp+var\_18\], eax  
CODE:004856AB                 add     \[ebp+var\_14\], 4  
CODE:004856AF                 lea     eax, \[ebp+var\_886\]  
CODE:004856B5                 mov     ecx, \[ebp+var\_18\]  
CODE:004856B8                 mov     edx, \[ebp+var\_14\]  
CODE:004856BB                 call    sub\_407160       <---其实是内存复制函数，将其改成 MemCopy  
CODE:004856C0                 mov     eax, \[ebp+var\_4\]       <---在这里得到内存指针  
CODE:004856C3                 mov     eax, \[eax+318h\]        <---在这条指令处出错!!!  
  
程序在 4856C3 处出错。很显然，在 4856C0 得到的 eax 应该是一个内存指针，可是当我在\[ebp+var\_4\]双击时，IDA 将我带到的地方竟然是一堆聊天数据，极有可能是存放指针的地方被聊天数据覆盖了。但是如何证明这一点呢？往上看，有两个地方调用了同一个函数 sub\_407160，让我们进去看看，双击进去一看是这样的：  
CODE:00407160 sub\_407160      proc near               ; CODE XREF: sub\_4596DC+3Cp  
CODE:00407160                                         ; sub\_4596DC+59p ...  
CODE:00407160                 xchg    eax, edx  
CODE:00407161                 call    @System@Move$qqrpxvpvi ; System::Move(void \*,void \*,int)  
CODE:00407166                 retn  
CODE:00407166 sub\_407160      endp  
  
很明显，这是一个内存复制函数，IDA 的库函数识别可帮了我的大忙，这正是我一直坚持使用 IDA 作为调试器的原因之一。为了更直观，我将 sub\_407160 改成 MemCopy。（能将函数动态改名，多么酷的功能）。稍加分析，可以看出这个函数是以 ecx 作为要复制内存大小，而以 eax 作为复制目的地的。因此在 CODE:004856BB 处要将数据复制到 var\_886 这个堆栈变量地址，虽然这个变量的大小从堆栈布局可以看出是 0x801，2K 多一字节，会不会是这个变量还是嫌小呢？我们在即将出错前在 CODE:004856B8 设断点，监视 ecx 是否超过 0x801。果然，用不了几下，就发现 ecx 的值为 0x8c0。哇，错误找到了--用于接受聊天数据的堆栈缓冲区太小，接下来我们要做的就是如何修改程序从而避免这个错误。  
  
很容易想到，我们必须将 var\_886 这个变量扩大，但是如何做到这一点呢？  
在函数开始，有如下指令：  
CODE:004855DB                 add     esp, -8A4h  
它是用来为函数分配堆栈变量空间的，可以看出局部变量的总大小大约是 2K 多一点，只要我们在这里修改一下，改成 add esp, -28a4h，就可以将函数的堆栈变量大小扩大 8K，但仅仅这样改还是于事无补，所扩大的 8K 不会被函数使用到，var\_886 能用的空间还是原来那么大。于是我想到了下面的办法：  
在 CODE:004855DB 处将指令改成 add esp, -28a4h，从而将函数的堆栈变量大小扩大 8K 后，将 var\_886 变量地址也向上移 8K，这样做既可以保证 var\_886 还是在可使用的堆栈空间内，又保证了它大约有 7K 大小的空间，比原先的 2K 大多了。这样改后，函数中所有对 var\_886 的引用也要往上移动 8K。这看起来好像不好改，其实用以下的办法大约 1 到 2 分钟就可以改好。  
1. 用 UltraEdit 打开要改的可执行文件，从 IDA 中可以知道要改的指令在文件中的位置，先跳到那个位置。  
2. 到了要改的地方后，你往往可以看到指令中出现的立即数，如改堆栈变量大小时，你可以看到 5c f7 ff ff，这正是-8A4h。  
   在 IDA 中，按 Shift-?，弹出简易计算器，输入-0x28a4，可以知道它的 16 进制表示为 5c d7 ff ff，因此只要将那个 f7 改成 d7 就可以了。  
3.将所有对 var\_886 的引用向上挪动 8K。在 IDA 中找到对 var\_886 使用的地方，并在 UltraEditor 中跳到相应的地方，只要将指令处的立即数作类似的修改就可以了。所有这些改动是很快的。  
  
作完这些修改后，程序就正常了。这是我发现，原来有一条很长的聊天记录，长度超过 2K。  
  
这篇短文对程序的修改只是个个案，我觉得真正的意义在于说明，对于没有源代码的程序出现崩溃的情况，我们决不是只能望而兴叹，我们可以自己动手。