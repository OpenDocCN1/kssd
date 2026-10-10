

# 初学者破解小技巧

> 原文：[`www.pediy.com/kssd/pediy06/pediy6212.htm`](https://www.pediy.com/kssd/pediy06/pediy6212.htm)

我可怜的小猫不知为何一命呜呼，害得我一个月没上网，只好在家里看以前下的 tKC-CRACK 教程。tKC 是 CRACK 界的老大哥，他写的 tKC's TuTors 共有一百多篇。最近看到这么一篇，觉得对我们菜鸟有点帮助，因此不自量力的翻译了一下，（课本上学的句式全都用不到，从初中开始学了六年英语不知学了些什么:( ），因为文章写得比较早，里面提到的工具之类有点过时，各位将就一下。我在里面加了自己的注释，标注 RB 的就是了。 

tKC's tutors #51  

Part 1  

题目：CrackTips For Newbies   
作者：LW2000  

\---  
英语不是我的母语，写得不好请原谅！（RB：英语也不是我的母语，译得不好请原谅！）  
看到这个题目 CrackTips For Newbies，高手莫笑！  
\---  
在 W32DASM 中你可以通过双击某一行，使那条绿线停在该行上。  
\---  
Hiew 可以设置默认打开方式为“代码模式”（decode mode）  
只要打开 hiew.ini，把'StartMode'键值从'Text'改为'Code'.  
（RB：其实最好改为'Hex'。不知各位是不是和我一样也喜欢用 HIEW，这样修改以后的确方便多了。）  
\---  
去除 NAG 窗口的简便方法：  
运行程序直到 NAG 出现，记下 NAG 窗口的标题和文本的前几个词。用一个十六进制编辑器打开这个 EXE 文件（有时是 DLL），查找你记下的词。如果找到了就向上翻页找十六进制字串'FF FF FF 80'，找到后改为'FF FF FF 90'。如果 NAG 窗口需要按一个钮才会消失，用这个方法通常不行。  
\---  
当你在 W32DASM 中得到一个偏移量(offset)，你会在状态栏里看到类似'Offset E346Ah'的字样，字符'h'的意思是十六进制，不要把它也算进去，偏移量应该是 E346A。  
\---  
下面这个问题我已经回答过十几遍了，所以请仔细地看，并且告诉你所有的朋友（如果你有朋友的话\*g\*）  
几乎在所有的 INI 或 CFG 文件中';'和'//'都表示这一行是无效的！这只是表示注释！在 winice.dat 中也是一样  
（RB：我孤陋寡闻，CFG 文件是什么东东？ConFiG？）  
我发誓，如果下一次还有人问我“我在 winice.dat 中加上了一行';EXP=c:\\windows\\system\\vbrun300.dll'为什么不起作用？”，他必须给我买台新电脑！  
\---  
试着把程序代码看懂！  
\---  
在破解过程中经常会用相同的假序列号，因此你应该知道它的十六进制形式是什么。我比较喜欢 123789，十六进制形式是 1E38D。应该避免使用像 123456789 这样的假码。  
\---  
学习汇编！学习汇编！学习汇编！学习汇编！  
（RB:不是我口吃，作者在这里重复了四遍，翻译要忠于原作 ）  
\---  
如果你跳过了一个 CALL，检查每个寄存器的值，你可能会发现序列号就在里面。  
（RB：序列号倒不一定有，但这个方法绝对太重要了！）  
\---  
不要急于求成（RB：原文是 Don't give up to fast），学习 CRACK 需要时间！  
\---  
在 W32DASM 中，如果你想要查找的“串式参考”出现了不止一次，你可以双击这个字串，然后 W32DASM 到了某个位置。再双击一次，就到了下一个出现该字串的地方，直到确定到了你要找的地方为止。  
\---  
大部分的程序员很懒，他们写一个过程或函数，当任务必须要完成时再调用它。通常一个程序会检查两次注册码：序列号刚输入时和程序启动时。So patch the function and save a lot of work. To Patch a funciton is often from advantage by CD-Checks, Nag Screens, Time or Date Checks,too.  
\---  
在 W32DASM 中只要右键双击 CALL 或 JUMP 所在的一行就可以跳转过去。想要返回按 F12。  
\---  
如果一个检测是否注册的函数返回值为-1，通常意味着没有注册。1（有时是 0）通常表示注册成功。  
\---  
阅读所有的 tKC 教程（RB：HEHE~~~给自己作广告 ）  
\---  
假如有一个让你使用 20 天的软件。记住：大部分的程序员都很懒！一个很懒的程序员可能会用这样的方法检查是否过期：  
 CMP DWORD PTR register, 14  <<-- 14 是 20 的十六进制  
 JLE/JGE address             <<-- 通常是这样一个条件跳转  
让我们看看代码。83 是 CMP，14 是 20 天，7E 是 JLE，7D 是 JGE。运行编辑器查找，你可能找到一处或多处。我们把找到的 JLE（7E 33）改为无条件跳转（EB 33）。通常程序就被破解了~~~~~（RB：这个~~~没这么简单吧 ）  
\---  
大部分的 CRACKER 说：如果你想变成真正的 CRACKER，应该阅读所有你能得到的教程！但是我认为：首先只读那些最好的教程（关于初学者的），其它的教程会把你搞糊涂。这样你可以节省很多时间。因此像我前面说的，阅读所有 TKC 的教程！其它很好的教程还有 BuLLeT,Tornado,HarvestR,The Saint Man,Acid Brun 所写的.（RB：在看雪论坛你可以找到最好的~~）  
\---  
永远不要——我说的是永远不要——使用 NOP，除非万不得已的时候，因为通常有更好的办法！例如你可以用 INC ECX，DEC ECX 或者其它类似的语句代替 2 个 NOP。  
\---  
现在你还在读这篇文章吗？好的！  
另一个技巧是，使用专门的工具。这可以节省很多时间，既然有这样的工具，为什么不用呢？  
下面是我比较喜欢的工具：  
W32DASM 8.93（有两个版本，一个含有 SDR Enabler for VB Apps,另一个不含有）  
SmartCheck 6.03 （用它破 VB 的程序很有趣！）  
Softice 4.0 （无需说明了）  
Hiew 6.16 （因为这是世界上最好的十六进制编辑器）（RB：同意）  
PE-Sniffer 1.06 （很好的检测壳的软件）（RB：我喜欢 FI）  
THE Customiser （玩窗口的工具）（RB：很有趣，有时也很有用）  
ProcView 3.1.1.2 （to trough apps perfect out of memory,有时你的破解成了&%$#!，它比\[ctrl\]+\[alt\]+\[del\]好得多）（RB：不好意思没用过）  
RegMon （监视注册表）  
FileMon （监视文件）  
GetType （检测文件）  
Opcodes （知识就是力量）  
ProcDump 1.5 （很酷的脱壳和 PE 文件工具）  
BlindRead （读取 CDROM 上的 crippeled files 的好工具）  
OpenList （看看该死的 WINDOWS 启动时运行了什么程序，对发现木马很有用）  
Snooper for Windows （get the text out of files）（RB：？？？）   
这些看上去很多，但大部分工具都是很小的。  

完成。有什么意见？Email: LW2000@gmx.net !!! （RB：这是 LW2000 的 EMAIL）  

  
 4dd466d0e1e0cc123afbe504da2462ca