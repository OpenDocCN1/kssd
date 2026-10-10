

# DLL 脱壳

> 原文：[`www.pediy.com/kssd/pediy02/forum255.htm`](https://www.pediy.com/kssd/pediy02/forum255.htm)

 对 PECompact 加壳的 DLL 脱壳的一点分析   
  
【声明】   
我写文章以交流为主，希望大家在转载时能保持文章的完整性。   
  
【前言】   
目前，我对 DLL 的脱壳的了解也不多，相信有些地方会和对 EXE 的脱壳大致相同。现在我知道的主要不同是必须要在 DLL 的空间开始跟踪。否则从 EXE 开始跟踪，那还不把人累死。另外 ProcDump 和 PEditor 之类的工具好象还无法自动修复 DLL 的 import table。唉，又是手动，什么时候有个 DLL 的脱壳机出现呢？ ou，别看我！我编程的水平三流，写不出那些好东东。   
  
样例文件：    dlcsp32.dll    （DynaDoc Reader v3.01 所带动态链接库文件，这个程序就是看.wdl 电子图书文件的那个）   
加壳方式：    PECompact v1.41b1 加壳   
检测工具：    和尚头上的虱子-----明摆着的嘛   
调试工具：    SoftICE v4.05,PEditor v1.5,Hex WorkShop 2.10   
目标：        脱壳   
作者：        ljttt   
写作日期：    2000-08-16   
  
1、首先当然要分析基本信息了。用 PEditor 打开动态链接库文件，得到如下信息   
Entry Point:    00024000   
Image Base:    10000000   
Size of Image:    00029000   
  
Section        Virtual Size    Virtual Offset   
pec1        00020000    00001000   
pec2        00003000    00021000   
.pec        00004000    00024000   
.rsrc        00001000    00028000   
  
再来看看 import table 和 export table 的情况，export table 没有改变。问我怎么知道？有未加壳的 DLL 嘛。西西。   
  
2、现在我们要想办法在动态链接库的入口点 10024000 处中断，当然方法很多了，这里介绍两种办法。   
  
①、第一种方法   
一、首先，用 PEditor 打开 DLL 文件，然后单击 FLC 按钮（这个功能是帮你计算 Virtual Address 转换为 Offset 的）   
输入 10024000 ，单击 DO! 按钮，得到 Offset\[hex\] 为 B000。   
二、然后，用 Hex WorkShop 打开 DLL 文件，定位到 B000，记下此处字节的值 EB。   
三、然后，用 PEditor 打开 Dlview32.EXE（主程序文件）。单击 break'n enter 按钮（这个功能是帮你在程序的某个地址空间设下 int 3 中断），在 Virtual Address 中输入 10024000。   
四、Ctrl-D 中断进入 SoftICE，设断点 bpint 3。   
五、F5 回到 Windows，单击 break'n enter 窗口中 RUN 按钮。这样我们将在动态链接库的第一条指令处中断。   
  
②、第二种方法   
一、同第一种方法   
二、同第一种方法   
三、然后，用 Hex WorkShop 把 B000 处的字节 EB 改为 CC。   
四、同第一种方法   
五、运行程序。这样我们也可以在动态链接库的第一条指令处中断。   
  
这里，我用第一种方法。我们现在中断在第一条指令处。但是还不能继续跟踪。要把 CC （即 int 3 指令）改为原来程序的代码。   
  
（简称说明： EP: Entry Point,    OEP: Orginal Entry Point,    RVA: Relative Virtual Address ）   
  
代码窗口显示如下：   
015F:10024000  CC                  INT      3            <---中断在此，这就是我们要改回的字节   
015F:10024001  06                  PUSH      ES   
015F:10024002  6810DE0000          PUSH      0000DE10        <---有点奇怪哦？ DE10 不是我们要找的 OEP 吗 ，怎么在这里就出现了？难道不用找了？   
015F:10024007  C3                  RET   
015F:10024008  9C                  PUSHFD   
015F:10024009  60                  PUSHAD   
015F:1002400A  E802000000          CALL      10024011   
015F:1002400F  33C0                XOR      EAX,EAX   
015F:10024011  8BC4                MOV      EAX,ESP   
015F:10024013  83C004              ADD      EAX,04   
  
下指令   
eb eip EB        （修改当然 IP 所在地址的字节为 EB）   
  
好，代码显示成原来的模样了。   
015F:10024000  EB06                JMP      10024008        <---代码复原后   
015F:10024002  6810DE0000          PUSH      0000DE10        <---奇怪？   
015F:10024007  C3                  RET   
\==> 10024008  9C                  PUSHFD   
015F:10024009  60                  PUSHAD   
015F:1002400A  E802000000          CALL      10024011   
015F:1002400F  33C0                XOR      EAX,EAX   
015F:10024011  8BC4                MOV      EAX,ESP   
015F:10024013  83C004              ADD      EAX,04   
  
3、这样代码就复原了。不过你发现一点问题没有？ DLL 的 OEP 怎么会在这里就出现了？ 好奇怪？！先不管它，设个断点留着看看。设断点   
bpx 10024002   
  
4、设断点   
bpx loadlibrarya do "dd esp\->4"        （老一套了）   
  
5、按 F5 继续，又中断在我们新设的断点。我们来看一下数据窗口   
  
显示如下：   
015F:1002041A 4E52454B  32334C45  6C6C642E  00000000      KERNEL32.dll....   
015F:1002042A 656C6552  44657361  00000043  44746547      ReleaseDC...GetD   
015F:1002043A 53550043  32335245  6C6C642E  00000000      C.USER32.dll....   
015F:1002044A 656C6544  624F6574  7463656A  00000000      DeleteObject....   
  
哦，这里看来就是我们要找的 import table 的一部分了。   
  
6、继续搜索，下指令   
s 30:10000000 l ffffffff 1A,04,02,00    （这里就不多说了，和《脱壳----对用 Petite2.2 加壳的程序进行手动脱壳的一点分析》一文中所说的情况相同，所以搜索方法也一样了。以后几步同理）   
搜索结果显示如下：   
Pattern found at 0030:1002000C (0002000C)      
  
7、下指令   
dd 1002000C-C   
  
数据窗口显示如下：   
0030:10020000 000200DC  00000000  00000000  0002041A      ................   
0030:10020010 00020270  000201D8  00000000  00000000      p...............   
0030:10020020 0002043C  0002036C  00020050  00000000      <...l...P.......   
  
8、这样到了 import table 的起始处了。现在开始保存 import table。   
/dump 10020000 1000 c:\\temp\\dump.bin    （1000 的长度是根据 00 字节的位置来确定的）   
  
9、下指令 BD 2 禁止 bpx loadlibrarya 中断，继续跟踪。到如下   
  
015F:10025335  51                  PUSH      ECX   
015F:10025336  53                  PUSH      EBX   
015F:10025337  6A00                PUSH      00   
015F:10025339  FFD0                CALL      EAX   
015F:1002533B  FFA55E854000        JMP      \[EBP+0040855E\]   
015F:10025341  8BB54E854000        MOV      ESI,\[EBP+0040854E\]   
015F:10025347  8BBD52854000        MOV      EDI,\[EBP+00408552\]   
015F:1002534D  E82E0C0000          CALL      10025F80   
015F:10025352  61                  POPAD   
015F:10025353  9D                  POPFD   
015F:10025354  50                  PUSH      EAX   
015F:10025355  6810DE0010          PUSH      1000DE10            <--- 1000DE10 - 10000000 = DE10 就是 OEP 了   
015F:1002535A  C20400              RET      0004            <--- 这里就要到回到真正的入口处了，在此停下。保存映象   
  
10、记下 OEP 为 DE10。下指令保存整个 DLL 内存映象。   
/dump 10000000 29000 c:\\temp\\dump.dll   
  
11、按 F5 继续，没有中断在我们在步骤 3 处设下中断。关闭主程序时，中断。正好在此处中断。   
显示如下：   
015F:10024000  EB06                JMP      10024008   
015F:10024002  6810DE0000          PUSH      1000DE10        <---开始中断时的 0000DE10 这时已经变成 1000DE10 了。   
015F:10024007  C3                  RET   
\==> 10024008  9C                  PUSHFD   
015F:10024009  60                  PUSHAD   
015F:1002400A  E802000000          CALL      10024011   
015F:1002400F  33C0                XOR      EAX,EAX   
015F:10024011  8BC4                MOV      EAX,ESP   
015F:10024013  83C004              ADD      EAX,04   
  
看来此处的几条指令的用处 PECompact 是别有用意安排的，可能是为了方便去卸载动态链接库吧。   
  
12、开始修补工作。用 PEditor 打开 dump.dll 文件，选择 sections，右键打开菜单，选择 dumpfixer。OK!一次完成所有 Section 的 RVA、Size 的转换工作。再修改 Entry Point 为 DE10，单击 apply changes 保存，选择 directory，修改其中的 Import Table 的 RVA 为 20000，Size 为 1000。单击保存。   
  
13、用 Hex WorkShop 打开 dump.dll 和 dump.bin，定位 dump.dll 位置到 20000，选择 1000 个字节。删除。   
选择 dump.bin 的 1000 个字节，复制到 dump.dll 中。保存。   
  
14、测试。OK，收工。   
  
【后记】   
本来我想用 UPX 加壳 DLL 来实验的。当然是想先找个"软柿子"捏。没想到，发现它加壳的 DLL 在 import table 做了手脚。这些天我正窝火，还有一个 Asprotect 加壳的 Awave Audio v7.0 的 import table 始终还原不出来。想不到又碰上这种问题，让我头晕眼花了好一阵子，（现在我看见 import table 的问题就有这毛病......烙下病根了）于是我一脚把 UPX 踢开，改为分析 PECompact 加壳 DLL 来实验，西西，这回简单。怎么样，我写的东东价钱公道，水分又足，咬一口新鲜水灵，大家快来买呀...........不好，打假别动队来了，赶紧溜 .......   4dd466d0e1e0cc123afbe504da2462ca