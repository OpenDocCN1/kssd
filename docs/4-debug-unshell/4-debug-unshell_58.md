

# 修复输入表

> 原文：[`www.pediy.com/kssd/pediy03/forum852.htm`](https://www.pediy.com/kssd/pediy03/forum852.htm)

【标题：用 Import REConstructor v1.2 beta2 修复输入表   
        --- 之 HiClock Pro v2.2 及 S-Spline 2.04】   
  
\--------------------------------------------------------------------------------   
【下载处】   
1)Import REConstructor v1.2 beta2   
  http://www.digitalrice.com/kaparo/files/utilities/imprec.zip  (取自 protoools)   
  
以下两个程序皆用 Asprotect1.2 以后的版本加壳   
2)HiClock Pro version 2.2 build 126 - released on April 9, 2001   
  http://www.downme.com/download/6815hiclockpro.zip   
  http://www.kgsoft.com/  (官方网址)   
  
3)S-Spline 2.04   
  http://202.108.252.24/stcsr/rj/txtx/S-Spline%202.04.zip    (康乐园)   
  http://www.shortcut.nl/S-Spline/Shortcut\_S-SplineDemo.zip  (官方下载)   
  
\--------------------------------------------------------------------------------   
【前言】   
      本文只简述输入表的修复，没涉及软件的破解。   
      主角是 ImpREC1.2 beta2, 目标软件是 HiClockPro v2.2 及 S-Spline 2.04 。   
      其中， HiClockPro 2.2 可一次性修复完成，S-Spline 2.04 还需要手工修复/填补。   
      本文在 win98se 下完成，当然也适合在 windows2k 下做参考   
  
【工具】 1) ImpREC1.2beta2(必须是 beta2 或其最新版本)   
        2) SoftIce4.05build334 , icedump6.022      (《=必备!)   
        3) PE-Editor1.7   
        4) BW2000 v0.2 (for win9x only)   
  
      工具简述：ImpREC 的说明文件指出，使用 ImpREC 时系统中最好已加载 icedump。   
            这样建立的输入表比较少跨平台的问题。而运行 icedump 前，当然得先加   
            载 Softice。   
                Softice + icedump： 使用 Icedump 的/tracex 命令可以找到可能   
            是 OEiP 的地址。   
                PE-Editor 的 dumpfixer 可以使 dump 出来的文件的 RS=VS, RO=VO   
                BW2000 v0.2 是为了查找被加壳的程序的 OEiP (ImpREC 中需要此值)   
            若已经知道软件的 OEiP,可省略。   
\--------------------------------------------------------------------------------   
【准备工作】   
      加载 Softice 及 Icedump 。先追踪目标程序(使用/tracex 命令)，得到 OEiP，   
      并/dump 出一个脱壳版本。使用 PE-Editor 的 dumpfixer 修正脱壳后的文件.   
  
\==》注意 1：HiClockPro 2.2 的必须采取特别步骤脱壳，否则程序无法运行。   
          具体请参考以下两篇：   
        1) http://001.com.cn/forum/toye/14781.html   
          标 题:简述 ICEDUMP 中/tracex 命令在 HiClock 脱壳中的应用 (1 千字)   
          发信人:henryw   
          
        2) http://www.001.com.cn/forum/toye/14335.html   
          标 题:不是不愿写，我真的是只知其然而不知其所以然。   
                我那样改的原因如内。 (690 字)   
          发信人:hying   
  
\==》注意 2: S-Spline 必须是未注册的，否则脱壳后的程序将无法正常运行。   
            S-Spline 的破解可以参考下面这篇：   
            http://001.com.cn/forum/toye/14799.html    
            标 题:v2.04 的 (9 千字)   
            发信人:blowfish   
  
    附：这里有一篇 ImpRECT 1.2 beta1 (不是 beta2!!) 的使用方法。   
        http://001.com.cn/forum/toye/14358.html  (本文参考这篇)   
        标 题:这样试试...... (1011 字)   
        发信人:ljtt   
  
  
【修复 HiClockPro 2.2】   
\---------------------   
  0.运行 HiClockPro 2.2 及 ImpRECT   
  1.在 ImpRECT 下拉列表框中选择 HiClock.exe 进程；   
  2.在左下角的 OEP 添入 BF260  (即这个程序的 OEP)   
  3.按 IAT AutoSearch，让其自动检测 IAT 位置；   
    出现对话框 Found Something!=》   
    “Found address which may be in the Original IAT.Try 'Get Import'”   
    这表示我们输入的 OEP 发挥作用了。   
  4.按 Get Import，让其分析 IAT 结构得到基本信息；   
  
  5.按 Show Invalids 按钮。   
      在 Imported Function Found 栏中点击鼠标右键，选择"Trace Level1 (Disasm)"   
    再按 Show Invalids 按钮。如果成功，可以看到所有的 dll 都为 valid:YES 字样；   
  6. (很明显第 5 步时，这个程序的输入表已经完全修复好了)   
  
  7.现在我们要修复已脱壳的程序 HiClock.exe。选择 Add new section (缺省是选上的) 来   
    为 Dump 出来的文件加一个 Section(虽然文件比较大，但避免了许多不必要的麻烦)   
  8.按 Fix Dump，并选择你 Dump 出来的文件；   
  9.ImpRECT 会在该文件所在目录生成一个新的文件：HiClock\_.exe ,这就是修复后的文件。   
  
TIPS1: 在 Imported Function Found 栏中点击鼠标右键，可以选择"Expand all nodes" 及   
        "Collapse all nodes" 来打开/关闭所有节点。   
TIPS2: 若因操作上的错误需要重新按 GetImport 钮, 最好先按程序右边的 ClearImports 及   
        Clear Logs 按钮。否则，Imported Functions Found 栏可能会出现奇怪的现象。   
  
修复前   
\- ?Fthunk:000C317C NbFunc:2E (decimal:46) valid:NO   
    -rva:00000000 ptr:00505A4D   
    -rva:00000004 ptr:00000002   
    -rva:00000008 ptr:000F0004   
    -.....   
  
修复后   
\- KERNEL32.dll FThunk:000C317C NbFunc:2E (decimal:46) valid:YES  <===\[看这个 YES\]   
    -rva:000C317C mod:KERNEL32.dll ord:015D name:GetCurrentThreadId   
    -rva:000C3180 mod:KERNEL32.dll ord:00D6 name:DeleteCriticalSection   
    -rva:000c3184 mod:KERNEL32.dll ord:0228 name:LeaveCriticalSection   
    -.....   
  
  
【修复 S-Spline 2.04】   
\---------------------   
0.运行 S-Spline 及 ImpRECT   
1.在 ImpRECT 下拉列表框中选择 S-Spline.exe 进程；   
2.在左下角的 OEP 添入 7E910    
3.按 IAT AutoSearch，让其自动检测 IAT 位置；   
    出现对话框 Found Something!=》   
    “Found address which may be in the Original IAT.Try 'Get Import'”   
    这表示我们输入的 OEP 发挥作用了。   
4.按 Get Import，让其分析 IAT 结构得到基本信息；   
4.1.发现只有一个 dll 的 valid 为 NO   
  
5.按 Show Invalids 按钮。   
    在 Imported Function Found 栏中点击鼠标右键，选择"Trace Level1 (Disasm)"   
  再按 Show Invalids 按钮。如果成功，可以看到所有的 dll 都为 valid:YES 字样；   
5.1.看到长长的一堆函数名，我们知道至少有一个 dll 未修复。故使用   
  鼠标右键”Collapse all nodes"，发现只剩下一个 dll 未完成。   
  从 Log 栏中可以看到有一个地址修复失败。   
  
6.按 Show Invalids 按钮   
  在 Imported Function Found 栏中点击鼠标右键选择"Trace Level2 (Hook)"。   
  再按 Show Invalids 按钮。如果成功，可以看到所有的 dll 都为 valid:YES 字样；   
  若还不成功，目前只能手工修复了(Trace Level3 这个功能还没开发完成)。   
6.1.从 Log 栏可以看到”Tracer failed in 008927E4 “修复失败。   
    我们找到那个修复失败的是 rva:0009C2AC   
(到此，这个程序的输入表还没修复完成，我们必须手工修复这两个地址)   
  
6.2.在 Imported Function Found 栏,找 rva:0009C2AC ,并用鼠标左键双击它。   
      在 Module 中选 KERNEL32.dll   
      在 Function 中选 ord:032C name:WinExec   
      按 OK   
    (至于这个函数名如何获得？看看下面)   
6.3 按 Show Invalids 按钮, 都为 valid:YES 字样，输入表到此完成。   
  
7.现在我们要修复已脱壳的程序 S-Spline.exe。选择 Add new section (缺省是选上的) 来   
  为 Dump 出来的文件加一个 Section(虽然文件比较大，但避免了许多不必要的麻烦)   
8.按 Fix Dump，并选择你 Dump 出来的文件；   
9.ImpRECT 会在该文件所在目录生成一个新的文件：S-Spline\_.exe ,这就是修复后的文件。   
  
  
  《找 rva:0009C2AC 的函数名》   
  ---------------------------   
  以下方法只是一个参考，它不是万能的。另一方面，以后的 Asprotect 的小小改变足   
  以使它失效，诸位姑且看之。。。   
    
  此时 Softice 已经运行.   
  1.进入 S-Spline 领空(不知道这样说对不对 8-) ) :   
      我们只需 CTRL+D ,用 bpx getwindowtexta 设拦, F5 。   
      在 S-Spline 的 Serial Number 栏中添入任何资料。   
      程序就会被拦。此时已进入 S-Spline 领空^\_^   
    
  2.寻找函数名:   
      此时下指令 dd 49c2ac  (= 9c2ac + ImageBase) , 看见该数据区显示 8972B4 。   
      再 u 8972B4 , 看见代码区显示  PUSH EDX    
                                    JMP C0188DC2   
      再 u C00188DC2 , 看见代码区显示  MOV EDX,C0197490  (<==注意这里!!!)   
                                        JMP C0188DD   
      再 dd C0197490 , 看见该数据区显示 C0197327 。   
      最后 db C0197327 就可以看到我们的函数名了 ^\_^ (注：这里是用 db 不是 dd)   
      bc \* 再 F5 回到程序。        
  
\----作者：BestFont  ------------【全文完】------------------------------------ 

简述 ICEDUMP 中/tracex 命令在 HiClock 脱壳中的应用   
  
前两天 peterchen 提出脱壳 HiClock，结果 hying 兄成功完成。ljtt 兄的提示也使我对/Tracex 命令感兴趣了。   
  
昨日装上 softice 和 icedump，开始试验这个奇妙的命令。   
  
Asprotect 加壳的软件会使得断点无效，/tracex 可以解决这个问题。   
  
常用两种用法：   
  
1\. /tracex 地址   
  用 bpx 拦不住的地址它可以拦住，就是速度慢了些。   
  
2\. /tracex 地址下限 地址上限   
  比如在 HiClock 中就可以在 softice 中下/tracex 400000 500000，结果第一次断下的地方只是一个 ret，相比就是 Johnson 大侠说的只有 ret 的 Call 吧，用 f8 返回可以看到是调用它的是一个 call esi。再下一次/tracex 400000 500000，会在 4BD530 处拦住，用 F11 返回，会发现这里就是 hying 兄在脱壳文中所指的需要跳过的 call 所在处。当第三次下/tracex 400000 500000 以后，就会来到入口点 4BDB68(大概是这个值，由于删除了程序，有些记不清了)。   
  
  不过在使用这个命令前，load 程序后最好先跟一段，进入 11xxxxxx 以后再下这个命令。   
  
  虽然这种跟踪需要很长时间等待 Softice 窗口再次跳出，但是很值得，是吗？   
  
  btw: 用 icedump 后，如果我没有理解错说明文档的话，连 Frogice 和 icepatch 都可以不用了，爽！   
        记得在跟踪前先启动 icedump 喔！ 

在前面的帖子我说了，402770 处的 call 在入口点前已经被调用多次，而第一个调用点就在 11A04B1 里，改上面的跳转后下面原有的一些 402770 的调用也都被跳过了。   
0167:011A04A3 CALL 01192910   
0167:011A04A8 CMP DWORD \[011A3590\],BYTE +00   
0167:011A04AF JZ 011A04D0 〈---必须跳   
0167:011A04B1 CALL NEAR \[011A3590\]   
0167:011A04B7 LEA EDX,\[EBP-08\]   
0167:011A04BA CALL 0119A020   
henryw 翻译的文章里说有的程序跳过这些后就可以正常运行，然后我试了一下，真的可以了。   
我实在是不知道它的原因。 

那个股市风暴 4.0 的 antidebug 做得很好，反各种调试软件、反 W32DASM、反 filemon……，我能想到的它差不多都 anti，我也不知它是如何做的。 

你的网站我会经常去的，论坛我可是第二个留言的。 

\--------------------------------------------------------------------------------

S-Spline 2.04 的确可以只需一次修复即可完成而无需手工修复，kingtall 没说错。   
  
可能只是我的系统本身的问题吧！?? 或是 king 兄的系统太干净了^\_^   
若启动到 windows 的保护模式下,的确无需手工修复。   
  
我若正常启动 windows(无 Softice,且 msconfig 中已去掉所有开机会运行的程序)，用 prodump 的 kill   
“干”掉所有可以“干” 掉的进程，所得到的结果：还是得手工修复！？？？为什么？!!!   
  
给大家添了不少麻烦。。。实在。。。。不知诸位遇见的情况如何？  4dd466d0e1e0cc123afbe504da2462ca