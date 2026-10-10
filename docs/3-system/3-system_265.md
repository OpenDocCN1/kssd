

# 笑解 API 函数 －－ API 绝密档案系列之一

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-309.htm`](https://www.pediy.com/kssd/pediy08/pediy8-309.htm)

看到 qduwg 老师凭"【翻译】3 个脱壳相关的重要函数介绍" 也"骗"到一个精，不觉心痒，也弄几个函数来介绍一下，不过我不是翻译，而是著作。我们就从 qduwg 老师介绍的几个函数入手。希望上面的话没有伤到 qduwg 老师的自尊心。我看到你对别人评价的反应，知道你很在意这些。  
  
1. HMODULE GetModuleHandle(  
  LPCTSTR lpModuleName   // address of module name to return handle   
                         // for  
);  
  
  GetModuleHandle 实际上分为两个函数  
    
  GetModuleHandleA－－处理 Ansi 字符串  
  GetModuleHandleW－－处理 Unicode 字符串  
    
其实真正干活的函数是，GetModuleHandleW (指 NT 以后的系统)，GetModuleHandleA 只不过将用户输入的 Ansi 字符串转成 Unicode 字符串然后就直接调用 GetModuleHandleW，所以在 OD 中如果下断拦截这个函数，直接拦截 GetModuleHandleW 就可以了，保险一个都跑不掉。我看一些新手写 Hook 程序时往往不厌其烦的将每个函数都“沟”一下，其实大可不必，随着我的文章深入，你会发现一个基本原则－－千条江河归大海，大部分的调用其实最后都是进入最底层的 ntoskrnl.exe、Hal.dll、和其他几个内核程序。而我们平常打交道最多的 Kernel32.dll 和 ntdll.dll 其实只是对底层函数进行了包装(当然对于某些 ring3 层的特定功能还是自己完成的)，相当于一个接口，起一个中转和隔离的作用，将用户的函数调用进行预处理，比如检测用户参数的合法性，对用户参数进行重新排序(底层函数的参数排列顺序有时和用户层不同，这样可以防止你轻易的弄明白底层在干什么)，将用户的调用分离拆解为若干个更为细小的调用，因为底层函数分类往往更细，这样比较灵活。  
使用 GetModuleHandle 函数，在高级语言中可以直接写成:  
hMod = GetModuleHandle(lpModuleName);   
编译器会根据参数的性质自行决定调用那一个函数，但在汇编中却不能这样使用，你必须明确的指明你要使用那一个函数。另外在 OD 中下断点，你也必须明确的指明对那一个函数下断，例如在 OD 中，进入 CPU 窗口，按 Ctrl-G，然后输入 GetModuleHandle，OD 会告诉你 “未识的标识符”，你必须清楚的告诉 OD 你到底要在那一个函数下断，是 A 还是 W。按前面的说法其实你只要下 GetModuleHandleW 就可以了(当然有时候在 A 下断，比较容易知道程序是从那里调用的，如果程序使用的是 A 调用)。  
  
俗话说，口说无凭。你怎么知道那两个还是最后变成了一个？  
看雪老大也在 qduwg 的文章后说：  

引用:    
不过这三个函数的定义只需要 Google 就能找到相关文档，因为太常用，不光是脱壳，编程是经常用的。  
现在的强壳都不直接调用这些函数了，都自己实现。

  
  
其后跟贴更有"牛 B 王"--nbw 惊人的评论：  

引用:    
可以参考以下 MSDN。写壳都是自己实现。  
  
以前自己能写个 GetProcAddress 就觉得很不错了，后来看高手写虚拟机，半小时模拟出来 LoadLibrary，那个让偶惊啊。。。。

  
  
我没有他们那样的水平，自己不会写，人都说半瓶醋晃荡，醋瓶满了就不响了，我这个瓶子还没有满，所以你们有幸能看到这些垃圾文章(当然也是为了骗一点"加精"，风光风光)，等有一天我的瓶子也满了（最好能够做到），也就只会“啊...”了。  
  
既然如此，那就只好看看老盖是怎么做的，这个函数在 Kernel32.dll 中，你可能会问，你怎么知道在 Kernel32.dll 中的？qduwg 老师在翻译文中可惜漏掉了可能他认为不重要的信息：  
  
QuickInfo  
  Windows NT: Requires version 3.1 or later.  
  Windows: Requires Windows 95 or later.  
  Windows CE: Unsupported.  
  Header: Declared in winbase.h.  
  Import Library: Use **kernel32.lib.**    //这里指明了函数的归宿  
  Unicode: Implemented as Unicode and ANSI versions on Windows NT.  
  
See Also  
Dynamic-Link Libraries Overview, Dynamic-Link Library Functions, FreeLibrary, GetModuleFileName, GetProcAddress, LoadLibrary,LoadResource   
  
现在我们来看看 GetModuleHandleA 的具体操作：  

> ```
> 代码:
> 
> * * *
> 
> 77E80B1A ; HMODULE \_\_stdcall GetModuleHandleA(LPCSTR lpModuleName)
> 77E80B1A           public GetModuleHandleA
> 77E80B1A GetModuleHandleA proc near              ; CODE XREF: sub\_77E684C2+2p
> 77E80B1A                                         ; CreateRemoteThread+FBp ...
> 77E80B1A
> 77E80B1A lpModuleName= dword ptr  8
> 77E80B1A
> 77E80B1A           push  ebp
> 77E80B1B           mov   ebp, esp
> 77E80B1D           cmp   \[ebp+lpModuleName\], 0
> 77E80B21           jnz   short loc\_77E80B31      ; lpModuleName 是有效的则转移
> 77E80B23           mov   eax, large fs:18h       ; TEB.NT\_TIB.Self
> 77E80B29           mov   eax, \[eax+30h\]          ; TEB.Peb
> 77E80B2C           mov   eax, \[eax+8\]            ; PEB.ImageBaseAddress
> 77E80B2F           jmp   short loc\_77E80B45
> 77E80B31
> 77E80B31 loc\_77E80B31:                           ; CODE XREF: GetModuleHandleA+7j
> 77E80B31           push  \[ebp+lpModuleName\]
> 77E80B34           call  @AnsiStrToUnicodeStr    ; 将 Ansi 字符串转成 Unicode 字符串
> 77E80B39           test  eax, eax
> 77E80B3B           jz    short loc\_77E80B45
> 77E80B3D           push  dword ptr \[eax+4\]       ; lpModuleName
> 77E80B40           call  GetModuleHandleW        ; 这里开始同流合污了。
> 77E80B45
> 77E80B45 loc\_77E80B45:                           ; CODE XREF: GetModuleHandleA+15j
> 77E80B45                                         ; GetModuleHandleA+21j
> 77E80B45           pop   ebp
> 77E80B46           retn  4
> 77E80B46 GetModuleHandleA endp
> 
> * * *
> 
> 
> ```
> 
>     

  
这些结果是在 IDA 中分析的，在使用 IDA 分析这些内核模块时请注意，一定要将那个符号文件.PDB 和调试符号文件.DBG 从老盖的网站上下载回来（注意要下载最新的版本），和你要分析的文件放在一个目录下，这样，IDA 就会自动加载这些符号文件，分析出的结果可读性大为提高。另外值得注意的是 windows 的这些系统模块和符号文件的版本一定要相同，大多数的情况下，符号文件的版本会相对低一点，这样你可以到系统目录下去寻找过去版本的系统模块，windows 在 update 时会将这些旧的系统模块保存在其他目录，具体的版本对于可执行模块来讲，比较简单，就是 PE 文件头中的 TimeDateStamp，使用任何工具查看都行。  
IMAGE\_FILE\_HEADER STRUCT  
  Machine               WORD    ?  
  NumberOfSections      WORD    ?  
  TimeDateStamp         DWORD   ?  //就是这里，不明白的话请看 qduwg 的翻译文章  
  PointerToSymbolTable  DWORD   ?  
  NumberOfSymbols       DWORD   ?  
  SizeOfOptionalHeader  WORD    ?  
  Characteristics       WORD    ?  
IMAGE\_FILE\_HEADER ENDS  
  
对于 PDB 文件比较麻烦，我没有现成的工具来查看这个 TimeDateStamp ，用任何十六进制编辑器打开这个.PDB 文件在偏移 0x4404 的地方就是这个 TimeDateStamp 了，对于.DBG 文件，在偏移 0x8 的地方，这两个 TimeDateStamp 一定要一样，否则宁愿不要使用。  
  
下面对 fs 段加一些详细的说明，很多人可能不太熟悉这个段。  
  
fs 段在用户模式（Ring3）和系统模式（Ring0）分别指向两个最重要的系统结构：  
Ring3:  
fs --> TEB (Thread Environment Block)结构表 --> 7FFDE000  
即“线程环境块”。  
  
Ring0:  
fs --> KPCR (Kernel Processor Control Region)结构表 --> FFFDF000  
即“内核处理器控制域”。  
  
在 NT 和 2K 和 XP 和 2k+3 中系统这个地址是固定的，随着版本的增大或 Update，TEB 会有新的内容增加，在将来的 windows 系统中，老盖为了防止那些坏人（我可不是坏人，否则说不定也有能力做点坏事）太容易利用这个特性搞破坏，有可能将这个地址变成随机的，也就是每次启动后都会在不同的位置。  
  
这些结构都是老盖没有公布的，所以如果你想使用这些结构，要特别小心，老盖为了防止小人搞破坏，每次 Update 都有可能修改这些结构而使的你程序崩溃。让你吃免费蓝玻璃。  
  
因为是所谓 UnDocument 的东西，所以最好的办法就是通过老盖的 WinDBG 或他的兄弟 KD 来获得这些结构，由于上面说的原因，这个结构以你自己看到的为准，至于如何使用 WinDBG 和 KD ，我想最好请那个叫 Themida flea 老兄来写（这可不是我创出的名字，是这位 Themida 老兄自己的图标，Themida 可是我非常欣赏的强壳，虽然不太稳定） 在本论坛的精品 “【原创】用 WinDbg 动态脱 Reflector” 中，我说我学不会 WinDBG ，这位 Themida 老兄答到：  

引用:    
一群笨蛋，笑翻。。。  
  
windbg 的 GUI 只是个 DEMO，各位可以编写自己的 windbgGUI  

  
由此可知，flea 老兄，高人也。flea 老兄不但可以教你们如何使用 WinDBG 和 KD，甚至可以教你们写一个带有自己特色的 debug，例如输入你自己开发的命令 allstruct，哇! 所有 undocument 的结构都展现在你眼前，想想，那是什么境界。  
  
下面这些是在我的机器上显示的结构，遗憾的是看不到 TEB 的完整结构，可以到 4F 提供的 w2kundoc.inc 中看到这个 TEB 的完整结构，不过我可不能保证是正确的。但对于我们所要讨论的问题，使用 WinDBG 提供的信息就足够了。使用 WinDBG 的内核模式或直接使用 KD：（双机模式或虚拟机）输入下面的命令:  

> ```
> 代码:
> 
> * * *
> 
> kd> !teb                 //老盖的 Debug 命令一般都需要这个"哇"符号做前缀，读作"哇!!添一杯"
> TEB at 7FFDE000
>     ExceptionList:    12f830                \\   0x0
>     Stack Base:       130000                |   0x4
>     Stack Limit:      126000                |   0x8
>     SubSystemTib:     0                     |   0xC     这里其实是 NT\_TIB 的一个结构
>     FiberData:        1e00                  |   0x10
>     ArbitraryUser:    0                     |   0x14
>     Self:             7ffde000              /   0x18
>     EnvironmentPtr:   0                         0x1C
>     ClientId:         4cc.4c8                   0x20
>     Real ClientId:    4cc.4c8                   0x24
>     RpcHandle:        0                         0x28
>     Tls Storage:      0                         0x2C
>     PEB Address:      7ffdf000                  0x30  这里指向 PEB 表，即进程环境块
>     LastErrorValue:   0
>     LastStatusValue:  8000001a
>     Count Owned Locks:0
>     HardErrorsMode:   0
> 
> * * *
> 
> 
> ```
> 
>     

  
这里只是 TEB 表的部分，但已经够使用的了，如果想在开发程序中使用这些结构，如前所说，可以到 4F 的 inc 中去寻找，也可以自己到网上去 google 一把，但不一定有美满的结果。  

> ```
> 代码:
> 
> * * *
> 
> kd> !kdex2x86.strct peb  //如果 kdex2x86 已经加载，可以直接输入 !strct peb
> struct   \_PEB (sizeof=488)
> +000 byte     InheritedAddressSpace
> +001 byte     ReadImageFileExecOptions
> +002 byte     BeingDebugged                   //Debug 运行标志
> +003 byte     SpareBool
> +004 void     \*Mutant
> +008 void     \*ImageBaseAddress               //这里就是程序加载的基地址了
> +00c struct   \_PEB\_LDR\_DATA \*Ldr
> +010 struct   \_RTL\_USER\_PROCESS\_PARAMETERS \*ProcessParameters
> +014 void     \*SubSystemData
> +018 void     \*ProcessHeap
> +01c void     \*FastPebLock
> +020 void     \*FastPebLockRoutine
> +024 void     \*FastPebUnlockRoutine
> +028 uint32   EnvironmentUpdateCount
> +02c void     \*KernelCallbackTable
> +030 uint32   SystemReserved\[2\]
> +038 struct   \_PEB\_FREE\_BLOCK \*FreeList
> +03c uint32   TlsExpansionCounter
> +040 void     \*TlsBitmap
> +044 uint32   TlsBitmapBits\[2\]
> +04c void     \*ReadOnlySharedMemoryBase
> +050 void     \*ReadOnlySharedMemoryHeap
> +054 void     \*\*ReadOnlyStaticServerData
> +058 void     \*AnsiCodePageData
> +05c void     \*OemCodePageData
> +060 void     \*UnicodeCaseTableData
> +064 uint32   NumberOfProcessors
> +068 uint32   NtGlobalFlag
> +070 union    \_LARGE\_INTEGER CriticalSectionTimeout
> +070    uint32   LowPart
> +074    int32    HighPart
> +070    struct   \_\_unnamed3 u
> +070       uint32   LowPart
> +074       int32    HighPart
> +070    int64    QuadPart
> +078 uint32   HeapSegmentReserve
> +07c uint32   HeapSegmentCommit
> +080 uint32   HeapDeCommitTotalFreeThreshold
> +084 uint32   HeapDeCommitFreeBlockThreshold
> +088 uint32   NumberOfHeaps
> +08c uint32   MaximumNumberOfHeaps
> +090 void     \*\*ProcessHeaps
> +094 void     \*GdiSharedHandleTable
> +098 void     \*ProcessStarterHelper
> +09c uint32   GdiDCAttributeList
> +0a0 void     \*LoaderLock
> +0a4 uint32   OSMajorVersion
> +0a8 uint32   OSMinorVersion
> +0ac uint16   OSBuildNumber
> +0ae uint16   OSCSDVersion
> +0b0 uint32   OSPlatformId
> +0b4 uint32   ImageSubsystem
> +0b8 uint32   ImageSubsystemMajorVersion
> +0bc uint32   ImageSubsystemMinorVersion
> +0c0 uint32   ImageProcessAffinityMask
> +0c4 uint32   GdiHandleBuffer\[34\]
> +14c function \*PostProcessInitRoutine
> +150 void     \*TlsExpansionBitmap
> +154 uint32   TlsExpansionBitmapBits\[32\]
> +1d4 uint32   SessionId
> +1d8 void     \*AppCompatInfo
> +1dc struct   \_UNICODE\_STRING CSDVersion
> +1dc    uint16   Length
> +1de    uint16   MaximumLength
> +1e0    uint16   \*Buffer
> 
> * * *
> 
> 
> ```
> 
>     

  
注意这些是 WinDBG 的扩展命令，由外部模块 kdex2x86.dll 提供（可以到老盖的网站去下载），先觉条件是这个模块必须被装载。  
至于那个 Ring0 的 KPCR 结构如果我的兴趣还在，而且那个瓶子还没有满，再详细讨论，KPCR 的结构巨大无比也复杂无比。前些日子 qduwg 老师发了题为 “For 菜鸟文章：PE 文件格式, qduwg 翻译”，我没有留心说漏嘴，大概意思是 PE 结构拿过来使用不就完了，用得着去学吗。其实此话来源就是因为底层的这些结构，和底层的这些结构相比， PE 结构那真是小巫见大巫了（PE 文件头其实就是一个非常简单的结构），如果你一个 PE 文件头都要花那么大的力气去学，而且到处都可以找到详细的资料，那么底层的这些结构怎么办??? 其一是这些结构远比 PE 结构更加庞大，更加复杂；其二是根本就没有任何资料和解释，全部要靠你自己去努力去理解，去破解。而且这些巨大的结构有无数多，怎么办，以我看，如果真是那样，不如回家种红薯。没有想到这段无心的话引起了两国战争，伊万大叔立刻就对我发动了战争，伊万大叔在俄国，我在加拿大，你想，加拿大全国警察加军队也不过才几万人，怎么能打的过伊万大叔，于是我只好狼狈逃跑。哈哈，由于我的当机立断，结果世界大战没有爆发，为了消除坏影响，伊万大叔买通看雪老大，将这场争论给完全删了，毁灭罪证，实在可惜，哈哈。伊万大叔在随后的跟贴中可不要对我口诛笔伐。  
这些是玩笑话，当不得真，如果不写一些这样的垃圾，我的创作热情恐怕立刻就消失了，所以请看官们忍忍。  
  
这里插一句题外话，大家在使用 OD 脱壳时，往往为了隐藏 OD，会加载 IsDebug V1.4.dll 来隐藏 OD，其实完全不需要加载这个东西，在 OD 中的数据窗口， 按 Ctrl-G ，在打开的小窗中输入 7FFDF000（见上面的 PEB 表）然后在偏移量 02（PEB.BeingDebugged 标志）的地方或者干脆输入 7FFDF002, 将鼠标放在这个地址上，Ctrl-E，将 1 改为零就 IsDebug 了，如果已经是 0 就是你已经使用什么 dll 将 Debug 隐藏了。   
  
对这两张表有了一定的认识后，我们再来看上面的那段代码的头部：  

> ```
> 代码:
> 
> * * *
> 
> 77E80B1A           push  ebp
> 77E80B1B           mov   ebp, esp
> 77E80B1D           cmp   \[ebp+lpModuleName\], 0
> 77E80B21           jnz   short loc\_77E80B31      ; lpModuleName 是有效的则转移
> 77E80B23           mov   eax, large fs:18h       ; TEB.NT\_TIB.Self
> 77E80B29           mov   eax, \[eax+30h\]          ; TEB.Peb
> 77E80B2C           mov   eax, \[eax+8\]            ; PEB.ImageBaseAddress
> 77E80B2F           jmp   short loc\_77E80B45
> 
> * * *
> 
> 
> ```
> 
>     

  
其中：  

> ```
> 代码:
> 
> * * *
> 
> 77E80B23           mov   eax, large fs:18h       ; TEB.NT\_TIB.Self
> 
> * * *
> 
> 
> ```
> 
>     

  
就是将那个 7FFDE000 的地址送到 eax 中去，现在我们知道那是 TEB 表。  

> ```
> 代码:
> 
> * * *
> 
> 77E80B29           mov   eax, \[eax+30h\]          ; TEB.Peb
> 
> * * *
> 
> 
> ```
> 
>     

  
根据上面的 TEB 表，我们知道 7FFDE000 偏移 30h 的地方是一个指针，指向 PEB 表。  

> ```
> 代码:
> 
> * * *
> 
> 77E80B2C           mov   eax, \[eax+8\]            ; PEB.ImageBaseAddress
> 
> * * *
> 
> 
> ```
> 
>     

  
根据上面的 PEB 表，我们得到最后结果，原来这几句代码的目的是获得程序启动后被装载的基地址。就这么简单。  

> ```
> 代码:
> 
> * * *
> 
> 77E80B1D           cmp   \[ebp+lpModuleName\], 0
> 
> * * *
> 
> 
> ```
> 
>     

  
这句是检测用户给出的 lpModuleName 是否为零，如果为零，用户发出的请求是获取自己当前进程的 hModule，这就是为什么要获取程序启动后被装载的基地址的原因了。再往下面的东西不用我解释了把，你可以亲眼看到他将那个 Ansi 字符串转换成 Unicode 后就调用 GetModuleHandleW 最后跑到一起去了。  
至于 GetModuleHandleW 干了一些什么事，且听下回分解，哈哈这样我可以多骗几个“加精”，qduwg 老师用三个 API 换一个“加精”，我准备用一个 API 调用换 3 到 5 个“加精”，哈哈，那当然还要看雪老大肯给我才行。  
  
后话：过些日子我就要回中国了，回到我可爱的祖国，如果很忙，可能就不能连续的写了，这点请大家谅解，如果成了海带，那就继续制造垃圾。多给我一点鼓励，使我能多写几篇，否则热情一旦消失或瓶子不幸装满了，就没有这些垃圾文章了，哈哈。  
  
另外伊万大叔花重金从专门出产那种体型苗条、曲线流畅的、并会飞的蟑螂的国家－－德国（现在中国许多城市的蟑螂多为这个德国种，本土的蟑螂太笨，不会飞，所以蔓延比较慢，危害也比较小，有灭亡的危险）买了一条游艇，还说会开到珠江口，在船上请我喝那个“可爱的水”－－vodka，根据伊万大叔提供的资料，vodka 没有任何的味道，最大的好处就是你喝过以后，不影响你和你那一半做某些亲热的动作（不过可没有规定必须是那一半，也许是小密），因为嘴中没有异味。不过我还是认为我们中国的酒好，你看那茅台、五粮液，酒瓶一开满屋香，多来劲，另外中国酒还喝的放心，谁放心？当然是老婆啦，你想，满嘴的酒气，那种可能对小密产生的专用动作不就失效了，想不到中国酒还有这种妙用吧！！不过到时候说不定又要爆发新的战争，为酒而战，不过这次是中俄之战了，哈哈，请看官们不用担心，我依然会毫不犹豫的跳海逃走，幸亏我的游泳技术不错，尤其擅长长距离游泳。  
  
注：修改了一些错笔字，另外为了赢得更高的点击率，将标题也换了，最好那位仁兄大才，给弄一个不吓死人不罢休的标题，那我就感激不尽了。伊万大叔就给了一个主意，说标题上一定要有个“密”字，我觉得有道理，这不，马上就成了绝密了。 

引用:  *最初由** RegKiller **发布*  
为什么我这没有这个地址？ CTRL + G 无法到达？ 

  
1、在数据窗口 Ctrl + G  
2、输入 fs:\[18\] + 30，显示如下  

> ```
> 代码:
> 
> * * *
> 
> 7FFDE030  **00 F0 FD 7F 7E** 00 00 00 00 00 00 00 00 00 00 00
> 
> * * *
> 
> 
> ```
> 
>     

  
3、将头上四个字节选上(黑体字)(变为黄色代表选中)  
4、鼠标右键弹出式菜单中选择 \[在数据窗口中跟随 DWORD\]，显示如下，看偏移 2 处那个黑体字 1 既是 begindebug 标志了。  

> ```
> 代码:
> 
> * * *
> 
> 7FFDF000  00 00 **01** 00 FF FF FF FF 00 00 40 00 A0 1E 13 00
> 
> * * *
> 
> 
> ```
> 
>