

# SVKP1.32

> 原文：[`www.pediy.com/kssd/pediy06/pediy6813.htm`](https://www.pediy.com/kssd/pediy06/pediy6813.htm)

  
手工修复引入表是破解加壳软件最难的一道工序，一向是顶尖高手的专利，令我等菜鸟往往望而却步，而不敢奢望越雷池半步！那么有没有简单一些的方法呢？通过本人的实践，终于找到了这条捷径。我的做法可以分为以下几步：  
    1、确定程序的 OEP，并脱壳。  
    2、使未脱壳程序处于正常运行态，使用 ImportREC 或 LordPE 检查是否所有的 API 都可识别，并记录下不可识别 API 的列表及其函数入口地址。  
    3、重新启动未脱壳程序，并使之中断在 OEP，用前面记录到的资料在所有不可识别 API 的入口设执行断点，尽可能尝试软件所有的功能，通过跟踪查清每一个不可识别 API 的名称及实际入口地址，并加以记录。  
    4、再次启动未脱壳程序，并使之中断在 OEP，用上一步确定的实际 API 的入口地址逐一替换相应的未识别 API 入口地址，然后使程序处于正常运行态，使用 ImportREC 或 LordPE 进行检查，这时所有的 API 就都能识别了。API 识别完成后，对已脱壳的程序进行 FixDump，整个修复过程就完成了。  
  
   下面让我们一起来亲手搞定目前比较凶悍的加壳保护软件之一--SVKP1.32（当前版本，看雪工具有介绍）。  
    使用工具：（1）、SoftICE 2.6 根据[`www.chat001.com/forum/crackforum/251195.html 自行制作的修改版`](http://www.chat001.com/forum/crackforum/251195.html 自行制作的修改版)  
              （2）、ImportREC 1.6 （[`www.pediy.com/tools/PE\_tools/Rebuilder/Import%20REC/ucfir16f.zip）`](http://www.pediy.com/tools/PE_tools/Rebuilder/Import%20REC/ucfir16f.zip）)  
              （3）、LordPE DLX（[`www.pediy.com/tools/PE\_tools/Lordpe/LPE-DLX.ZIP）`](http://www.pediy.com/tools/PE_tools/Lordpe/LPE-DLX.ZIP）)  
              （4）、UltraEdit 10.10a（[`www.ttdown.com/SoftDown.asp?ID=6718）`](http://www.ttdown.com/SoftDown.asp?ID=6718）)  
    目标程序： SVK Protect 1.32 （[`www.anticracking.szm.sk/svkp\_setup.exe）`](http://www.anticracking.szm.sk/svkp_setup.exe）)  
  
    安装目标程序，装载 SoftICE。  
  
   1、下断点“bpint 1”，然后启动 SVKP.EXE，程序中断于  
001B:07BABC57  CD01                INT       01  
   下命令“bd 0”关闭断点，并使用命令“r eip 07BAEE11”修改指令指针，以便躲过 SEH 陷阱，再下命令“g 07D31FA5”，使程序中断于 07D31FA5 处（当然，你也可以使用命令“g 00401000”使程序直接停在 OEP 处），使用 F8 键单步跟踪，要不了几步就可以到达程序的 OEP：00401000 处了。记录下此值，并记录下从 OEP 开始的几行指令及相应十六进制编码，下汇编命令“a eip”修改入口指令为“JMP EIP”然后用命令“x”使程序退出 SoftICE 并在 OEP 处进入死循环。  
   2、下面需要从内存中 Dump 出程序。启动 LordPE，用鼠标指针点选 SVKP.EXE 进程，点鼠标右键，在弹出的菜单中点选“Correct Image Size”项前先修正映像尺寸。然后在相同的菜单中点选“Dump Full”以 Dump 出脱过壳的程序，命名为“SVKP\_Dump.exe”。然后还在相同菜单下点选“Burn Process”杀死在内存之中处于死循环状态的程序。此后用 LordPE 的 PE Editor 功能调入脱过壳的 SVKP\_Dump.exe，修改其 OEP 的 RVA 为 00001000，保存退出。  
  
  
3、下面需要复原 SVKP\_Dump.exe 在 OEP 处的指令。使用 UltraEdit 载入 SVKP\_Dump.exe，使用十六进制码搜索功能搜索以下字节串  
       eb fe f3 4b 00   
找到后修改为：  
       68 1d f3 4b 00  
改好后存盘退出。至此，脱壳程序 SVKP\_Dump.exe 修改完毕，剩下的工作就是修复引入表 IAT 了。  
   4、启动 ImpotREC，在进程列表中选取 SVKP.EXE，修改相应 OEP 的 RVA 为 00001000，点击“IAT Auto Search”按钮，再点击“Get Impots”按钮，逐一检查 API 的识别情况。最后发现，有 5 个 API 程序无法识别，对应的入口分别是：  
     (1) 07D37656  
     (2) 07D49E52  
     (3) 07D49E82  
     (4) 07D333DD  
     (5) 07D32070  
   5、重新启动加壳程序，使之停在 OEP 处，下断点：  
    （1）、“bpx 07D37656”  
    （2）、“bpx 07D49E52”  
    （3）、“bpx 07D49E82”  
    （4）、“bpx 07D333DD”  
    （5）、“bpx 07D32070”  
   然后让程序继续执行，分别从断点处开始跟踪，看看程序调用的究竟是哪个 API，然后再返回到调用处看看：  
   (1) 00401005 CALL \[004C521C\] -- 07D37656 <-- SVKP API  
  
   (2) 00402F04 CALL \[004C51A8\] -- 07D49E52 <-- KERNEL32!CopyFileA  
   (2) 0040300B CALL \[004C51A8\] -- 07D49E52 <-- KERNEL32!CopyFileA  
  
   (3) 00402F58 CALL \[004C51B0\] -- 07D49E82 <-- KERNEL32!CreateFileMappingA  
   (3) 00403694 CALL \[004C51B0\] -- 07D49E82 <-- KERNEL32!CreateFileMappingA  
  
   (4) 00401086 CALL \[004C51F8\] -- 07D333DD <-- KERNEL32!GetModuleHandleA  
  
   (5) 004011A8 CALL \[004C5200\] -- 07D32070 <-- KERNEL32!ExitProcess  
    现在情况就比较清楚了，第一个 API 是用于为传入的地址装载固定的文本内容，只在程序开始时调用，不是很重要，我们为它指定一个调用形式相当的 API--KERNEL32!GetVersionExA 作为替代，其它的 API 已经清楚了。  
    6、再次启动加壳程序，使之停在 OEP 处，用识别出来的 API 入口地址替换原来无法识别的 API 入口地址。在本人的系统下，这些识别出来的 API 的对应地址分别是：  
  （1）、KERNEL32!GetVersionExA      -- 77E6A9C3  
  （2）、KERNEL32!CopyFileA          -- 77E7DA46  
  （3）、KERNEL32!CreateFileMappingA -- 77E68492  
  （4）、KERNEL32!GetModuleHandleA   -- 77E66C42  
  （5）、KERNEL32!ExitProcess        -- 77E78F94  
   需要做的工作就是把这些新的入口地址填到相应的内存单元，就像这样：  
   序号    内存地址      修改前      修改后   
\===============================================  
   （1）、\[004C521C\]    07D37656    77E6A9C3  
   （2）、\[004C51A8\]    07D49E52    77E7DA46  
   （3）、\[004C51B0\]    07D49E82    77E68492  
   （4）、\[004C51F8\]    07D333DD    77E66C42  
   （5）、\[004C5200\]    07D32070    77E78F94  
\===============================================  
    为此，分别下命令：  
   （1）、    “e 004C521C C3 A9 E6 77”  
   （2）、    “e 004C51A8 46 DA E7 77”  
   （3）、    “e 004C51B0 92 84 E6 77”  
   （4）、    “e 004C51F8 42 6C E6 77”  
   （5）、    “e 004C5200 94 8F E7 77”  
    以完成此项修改。  
    完成此项工作后，下命令“r eip 0040102E”，把 EIP 从 00401000 更改为 0040102E，让程序继续执行。按照前面的第 4 项操作，你会发现，所有的 API 都可以正常识别出来了。这时，用鼠标点选“Fix Dump”按钮，在弹出的文件选择窗中，用 Browse 功能选取前面已经脱过壳的 SVKP\_Dump.EXE，点确定就 Ok 了。  
    7、由于 KERNEL32!GetVersionExA 是代用的，所以还需要考虑修改后给程序流向带来的影响。  
001B:00401000  681DF34B00          PUSH      004BF31D  
001B:00401005  FF151C524C00        CALL      \[004C521C\]  
001B:0040100B  85C0                TEST      EAX,EAX  
001B:0040100D  751F                JNZ       0040102E  
我们看到，修改前:  
001B:00401005  FF151C524C00        CALL      \[004C521C\]  
总是返回 EAX=1，而修改后又总是返回 EAX=0，这就改变了程序的实际流向，必须补救，措施就是把  
001B:0040100D  751F                JNZ       0040102E  
修改为  
001B:0040100D  EB1F                JMP       0040102E  
Ok，你都照做了吗？大功告成！试着启动一下程序看看，是不是很酷？