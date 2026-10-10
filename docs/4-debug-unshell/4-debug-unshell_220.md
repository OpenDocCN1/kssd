

# Armadillo for Dummies: Getright 5 手动脱壳和重建 IAT 

> 原文：[`www.pediy.com/kssd/pediy12/armadillo.html`](https://www.pediy.com/kssd/pediy12/armadillo.html)

 

 这是一篇 Armadillo 加壳软件 Getright 5.01 的脱壳译文，我是参照 Ricardo Narvaja 的“Getright 5 脱壳和重建 IAT”的文章以及 Bighead\[DFCG\]\[YCG\]的译文,一边实践一边再次翻译的。感谢 Ricardo Narvaja 和 Bighead\[DFCG\]\[YCG\]。   
  
Armadillo for Dummies: Getright 5 手动脱壳和重建 IAT (第一部分)   
因为 Armadillo 实在是太复杂了,以至于人们已经不得不沿用下面这个手动的方法去脱壳 armadillo,这不是我的过错,只是因为 armadillo 实在是太难脱壳了,然后我将会在这个文档中一步一步地去完成脱壳,同时配上很多图片,只是想让它变得尽量容易理解.我并不想写一个 10 兆大的参考教程,所以我把这个文档分成两部分.第一部分主要讲如何转储(dump)armadillo 保护的程序,然后我会在第二部分中提及一些以前在其它任何教程中都未曾见过的东西,那就是输入表(import table)的 redirection 和 rebuilding.   
  
在我们开始之前,我必须声明这篇参考教程只在 Windows XP 上测试.请不要在 windows98 或者 2000 中尝试.原因是只有 Windows xp 才拥有必要的 API,能够把子进程(child)从它的父进程 father 那儿脱钩下来.   
  
这次练习,我们找到的自愿者是 Getright 5 final. Crusader 曾经写过关于这个软件的某一个版本的参考教程,但是这篇参考教程在很多方面都不同于 crusader 写的那篇.   
  
crusader 写的那篇参考教程(原版)----附(1)   
  
crusader 写的那篇参考教程(FTBirthday 翻译版本)----附(2)   
  
你可以从以下连接下载 GetRight 5.不管怎样我在我的 ftp 中保存了一个副本,所以你可以从那儿下载,就不用去管它在网上被放在哪儿了.   
  
[`www.getright.com/`](http://www.getright.com/)   
  
抓起你的鼠标,安装好 Getright 然后准备作战.Armadillo 将会变得软弱无力.   
  
如何脱壳(UNPACK)以及转储(DUMP)Getright 5   
  
在 OLLYBG 中载入 GETRIGHT.exe 然后整装待发.我将按照以下的步骤,很容易记住的,你可以在其它任何 armadillo 保护软件中按部就班.   
  
第一步:重新设置检测 ISDEBUGGERPRESENT API 的字节   
  
我将要解释如何在任何电脑中去寻找这个字节.   
  
首先从这个连接下载命令行插件:   
  
[`dd.x-eye.net/file/cmdbar10802.zip`](http://dd.x-eye.net/file/cmdbar10802.zip)   
  
解压它然后复制 dll 到 OLLYDBG 文件夹.只要你运行 Ollydbg 你将会在调试窗口的左下角看到一个白色的输入框,在那儿你可以键入很多命令.让我们键入:   
Bp IsDebuggerPresent   
  
我们将要使用 BP 而不是 BPX,因为 BP 可以直接在 API 上下断点,那正是我们所需要的. 

我们必须准确地键入 Bp IsDebuggerPresent.记住我们必须保证大小写的顺序,因为如果我们键入了任何不同于以上形式的命令,那么一个错误提示(UNKNOWN COMMAND)将在命令行的右边产生.所以正确地键入 BP IsDebuggerPresent 然后按回车. 如果在按回车后没有错误提示出现,那就说明这个 BP 已经生效.   
  
现在按 F9 运行程序,它将会中断在 API IsDebuggerPresent 

![](img/6a4d495d75726715576edc121b8a1771.jpg)

在这张图中我们可以看到 OllyDBG 中断在 API IsDebuggerPresent(红色), 然后按 F7 步进到第三行(白色)   
  
77E52749 0FB640 02 MOVZX EAX,BYTE PTR DS:\[EAX+2\]   
  
一旦到达那儿,看一下 disassembly,在那儿我们可以看到如下图的内容: 

![](img/59f1a3bb97296f79714500e8c73a43ed.jpg)

这儿我们可以看到在我的机器中偏移是 7ffdf002,包含了值 01.这个地址在其它机器中可能会不同. 值 01 意味着调试器被检测到了,所以写下这个地址,在任何时候我们在 ollydbg 中运行程序,都必须把它修改为 00,从而避免被 packer 检测到.记住在 ollydbg 中运行或重运行程序的任何时候修改此值为 00.    
  
我们将以重启 ollydbg (ctrl+F2)来结束第一步,到转储窗口(DUMP)单击右键选择"前往 表达" 7ffdf002 或者其它任何先前你记下的地址,然后把它的值从 01 修改为 00. 

![](img/039f630fa4260dba03b0c787c6c482c6.jpg)

![](img/6a4694da14ef5e3da1b8941399608e2c.jpg)

把它改为 00. 

![](img/27e30662025914c29bb092528816edd3.jpg)

第二步:BP WAITFORDEBUGEVENT   
  
这一步并非必要,但却是一个好的选择,当它停止时在转储(DUMP)区看到 report.让我们看看该怎样做.   
  
首先我们需要在命令行输入 BP WaitForDebugEvent,和我们输入 IsDebuggerPresent 的方法是相同的. 

现在运行它,当它停下时我们可以在堆栈窗口(Stack window)看到关于这个 API 的所有参数信息. 

![](img/19d509f36fde9a0384cf3e18ef1eaa97.jpg)

看一下第二行.那儿我们可以看到它将在哪儿显示 report,所以到转储窗口(dump window), 右键选择 "前往 表达"= 12EFF8.注意这个值在一些机器上可能会不同,所以前往你的机器显示的值. 

![](img/ed3fdbee7a6ba6669d27364a0eddb33d.jpg)

我们在上面的图中看到的是我们将要称之为"API report"的东西.主程序(让我们把它称为 Father)通过那里得知 secondary program(让我们把它称为 son)发生了什么.    
  
现在我们可以很容易地通过在命令行输入 BC WaitForDebugEvent 来删除这个断点.   
  
所有的操作都是为了在转储窗口(dump window)看到 report,在这个 report 中我们可以看到入口点 OEP(entry point). 

第三步: BP WRITEPROCESSMEMORY 

![](img/b115643654dede4b5fca53038b809562.jpg)

像图中一样设置这个断点然后点击"运行".也许你会在一些例外(exceptions)处停下,但是你可以通过按 SHIFT + F9 很容易地走过.一旦我们停在了 WriteProcessMemory API 不要做任何事情,看下图:   
  
正如我们在下图中看到的, father 将要复制给 son 的第一个块(block)的所有信息,都可以在堆栈窗口(STACK window)看到. 

![](img/f1a22751c49701d85e02d999e467c1ce.jpg)

FTBirthday:由经验,我觉得很难走到这一步,但是我知道是在 005F949E 处 call 了 WriteProcessMemory 如图: 

![](img/5124cdd6c50a63dd501ad90e326311af.jpg)

我通过一遍遍试验,得出了最好的方法:等到前两次在别处 call 了 WriteProcessMemory 后,左手一直按住 shift, 右手连续按 F9 大约 31 次,就可以顺利到达.   
  
这些信息被保存在父进程的一个缓冲区从 3B8FB0 开始,然后它会复制给子进程一个从 538000 开始的 1000 bytes (BYTES TO WRITE)的块(block),所以如果我们到了这一步,我们可以很容易理解 child 的第一个 section 是完全空的,所以当它试图执行到 OEP 时,它报告一个 error 因为那儿没有任何东西,所以 father 得到了通知正如我们可以在 father 的 report 中看到的,所以它停止了运行,复制必要的数据块,然后继续运行直到下一个 error.被复制的数据块的大小是 1000 bytes,所以当程序试图执行任何超过这个大小的 block 外的指令时, 另一个 error 将会发生,然后这个 error 会被通知给 father, father 会复制另外 1000 bytes 的块 block 然后继续.    
  
顺便说一下,我们可以假设第一个 error 发生是因为 son (坏小子)call 了它的入口点 OEP,它的值应该就在 report 上.显然 OEP 的值必定在第一个 block,father 复制过去来解决 son 报告的 error.   
  
这个块从 538000 开始知道 538fff.OEP 值必定在这些值中. 让我们看一下 REPORT.看转储窗口(DUMP window) 我们先前已经知道了指向 report 的指针 12EFF8. 

![](img/9ec9df7dc7d0e3221c049a8c8795ab5a.jpg)

太好了!在那儿我们至少看到了三次这个值: 538540.数学课告诉我 538540 比 538000 大,比 538fff 小 (那些是 father 试图拷贝给 son 的那个块的起止值)所以我可以确信 538540 就是 child 的 OEP.   
  
让我们从 API WriteProcessMemory 来改变断点的性质.知道哪个块被拷贝不是很好吗.可以通过下面的方法做到.首先选择 bpx 所在的行,按 F2 移除 bpx. 

右击选择"断点/条件记录".现在在对话框中准确地填入如图所示的值.这样做的目的是使我们在记录窗口中看到所有被 father decrypts 并拷贝给 son 的 blocks. 

![](img/1277e7dec05e1cfa0fe84fe49a15e199.jpg)

第四步: NOP THE CRIPTER CALL   
  
在先前的参考教程 s 中我们知道,这儿我们是在 decripter call.Father 为 son decrypts 了一个 block.但是还有一个 cripter call 用来 encrypts 或者 destroys 已使用的 blocks 来避免被转储 dump.现在的任务是如何找到这个 call 并把它 nop 掉.    
  
我现在在 WriteProcessMemory API,所以我打开"呼叫堆栈窗口"(Alt+K)在那儿我可以看到: 

![](img/ac8b051e7ce74aedc2ef614521e74205.jpg)

在这个"呼叫堆栈窗口"越靠上面的是越最新被执行的.从 CALLED FROM 我们可以看到 5F949E 是 father calls API 的地方. 如果我们向下看,我们可以看到另一个 call.所以我们获得了 decripter call 的 offset.    
  
DECRIPTER CALL= 5F88D1   
  
通过"前往 表达" 5F88D1 跳到那儿 ,或者左键双击它. 

![](img/e1b11548af30b1c2f7b6eb05f996596b.jpg)

这是好的 CALL,它用来 decrypts.现在去找坏的那个,稍稍向下翻屏或者右击选择"查找参考/呼叫目标" 选择出现的两个参考中的另一个. 

![](img/00e6d846fe9bdb661e22921881e9b4e5.jpg)

它在这儿呢!   
  
重新加密的 call--ENCRIPTER CALL = 5F8A24 

![](img/4e710e0e3266b10a7a9bf197e1845d08.jpg)

现在我们把它 nop 掉.我们选择这个 call 按空格写入 nop. 

![](img/e85daace4db9834197e2b4b665f1f7f3.jpg)

第五步: 运行 API 并在 OEP 处设一个无限循环   
  
现在到了执行 WriteProcessMemory 一次了. 我们有两条路可走,可以选择执行到返回它将会执行直到 RET,然后按一次 F7. 第二个选择是去 stack 第一行在那儿设一个 BPX 最后运行它. 

在 stack 第一行提示程序会返回到 5F94A4, 我们到那儿 BPX (F2)然后运行它(F9). 

![](img/73ea134f6e797de8246ef11c24af712d.jpg)

现在该用 PUPE 了. PUPE 是一个西班牙工具.你可以在[www.google.com](http://www.google.com)搜索一下. 通过下面几行代码我们将要在 son 的 OEP 设置一个无限循环.这会使 son 休眠直到我们叫醒它.    
  
这是一张 PUPE 的图片.我们可以看到有两个进程, the son and the father.上面那个是 son. 在右边我们可以看到进程句柄. 

![](img/d626972382b5861d8232f96d1b00bcdb.jpg)

选择上面那个 getright.exe 然后右击选择"parchear". 

![](img/52902904af444988738c36d10a5728ce.jpg)

写下出现在对话框中的 original bytes 然后用无限循环代码 EB FE 代替,如下面的对话框中所示.   
  
最后一步就是"PARCHEAR"然后 INFINITE LOOP 就准备好了. 

![](img/7e590138360cfb4f1656708b66168a7e.jpg)

第六步: BP WAITFORDEBUGEVENT 和 NOP THE API   
  
不要清除先前断点 WriteProcessMemory,是时候写入新断点了.键入 BP WaitForDebugEvent 回车, 然后按 F9 运行 

![](img/5c09e58976bbbfc53c7ea76097d55088.jpg)

它停下时 

![](img/4dfca94c0d2f59e3e522ccb776d17ec7.jpg)

你必须记住 NEVER EVER RUN THIS API.在转储窗口"DUMP window"看一下 report, 然后去堆栈窗口(STACK window). 

![](img/f50139d1a6c2b60687c4182e7920408d.jpg)

正如我们看到的如果我们运行这个 API 我们应该去 5F751D,所以让我们通过"前往 表达" 5F751D 到主窗口. 

![](img/8be58c55da1d899ade1c7c31e208ab47.jpg)

再一次提醒不要 RUN THIS API.所以右击 5F751D 选择"新建起源".这是跳过这个 API 的方法. 

![](img/6f325a55c55281b37181d027c362c434.jpg)

现在我们必须 nop the call to the api 以及有关的 push. 

![](img/7d59e65b6de8b93b47d026f187959ddb.jpg)

所以在这些行上按空格键然后写入 nop.当你完成这项工作时,将会如下图所示. 这一步结束了. 

![](img/7cc3b3d18e3168c46721651f65dfe1e3.jpg)

第七步:如何打入补丁   
  
当做这一步时请注意.很多人不知道究竟该如何打补丁,所以努力试着理解我这里所作的, 然后学会在其它情况下运用.    
  
第一步是改变跳转, 

![](img/24917f37376dc9966ae6c9810f67bc13.jpg)

我们将要改变这个跳转为 JMP 401000 那是我们将要打补丁的地方, 父进程 offset 401000 . 

![](img/4635b99c8042154c2d3f08c260d838e8.jpg)

跳转设好了,现在去 401000 (ctrl+G)我们要开始写补丁了. 在 report 中我们可以看到三次 entry point value.也许 OEP 在其它情况下会出现更多次.但是我们只需改变这三个. 

![](img/6247c4763969dc6e0ad00c557fdaaf5f.jpg)

father unpacks 了在 report 中显示的 block.所以我们将要欺骗 father 则他会相信从第一个 section 开始的所有 blocks 都存在错误, 一个接一个所有的 blocks 都会被 unpacked.因为这一点我们需要知道第一个 section 的起止点.点击"查看/内存" 

![](img/f72cbd4836cdbe3e9b3458eb90ac524b.jpg)

这儿我们可以看到 getright 的 sections.忽略 Header section 看 text section,它从 401000 开始,结束在 589000-1=588fff.    
  
401000-588fff (第一个 section 的范围)   
  
我们将要在 report 中用 400000 代替 OEP 的值. 

![](img/25615e21c9be52248d78936fbde2a63b.jpg)

正如你在图中看到的我写入了 40000,然后 patch 会在每一个 loop 前加 1000,所以第一个被 unpack 的 block 将在 401000.   
  
现在我们将要在 401000 处写下如下补丁.   
  
    
  
00401000 8105 10F01200 0>picture1/add DWORD PTR DS:\[12F010\],1000   
  
0040100A 8105 1CF01200 0>picture1/add DWORD PTR DS:\[12F01C\],1000   
  
00401014 8105 20F01200 0>picture1/add DWORD PTR DS:\[12F020\],1000   
  
这些行将我们先前在转储窗口(dump window)写的值增加 1000.那些蓝色的值在不同的机器上可能会不同,所以修改为你在转储窗口(dump window)看到的实际值. 再次提醒,证实在转储窗口(dump window)写入了 00 00 40 .最容易犯的错误就是在补丁(蓝色的)中的 offset 和转储窗口(dump window)的 offset 不匹配.   
  
下一行该这样写:   
  
0040101E 813D 20F01200 0>CMP DWORD PTR DS:\[12F020\],getright.00589000   
  
比较是为了让我们知道何时所有的块都被 unpacked 了.    
  
然后我们必须写下:   
  
00401028 - 0F85 F6341F00 JNZ getright.005F7524   
  
如果比较为假,则返回 called loop 的地方.然后我们需要在最后写下 NOP,在那儿我们将要设下一个 BP. 当它完成转储时,它将会停在那儿. 

![](img/e63162f7a05bb8fae7df3d3e3cf0ac18.jpg)

这儿我们可以看到完整的补丁代码   
  
我们可以跟踪第一个循环,来检测 report 中的 each entry 是否每次都增加 1000.   
  
第八步: UNPACK    
  
按 F9 运行程序然后等待它停在我们先前在补丁中设下 nop 的 bpx 处.如果没有错误发生,当我们停止时,看 view-LOG 我们将要看到所有的 unpacked blocks.    
  
这是假设的所有 unpacked blocks 的 LOG.我没有完整的给出因为地方不够但是你可以在你的日志窗口中查看.   
  
所以现在从 401000 到 589000(包括 588fff)的所有 blocks 都被 unpacked 了,然后 dump 已经就绪.    
  
第九步:把 CHILD 从 FATHER 那儿脱钩   
  
一旦我们停在 NOP,让我们写下下面几行.   
PUSH (child 的 handle )   
  
Call DebugActiveProcessStop   
  
如果 handle 由字母开始,你需要在它前面键入一个 0,因为不这样做的话 olly 不会认出. 想知道 son 的 handle 只需看一下 pupe, 记住 child 的 handle 是在两个进程中的上面的那个.你一样可以通过 opening file attach 来获得 son 的 handle.那儿你可以看到两个名称相同的进程. 红的是 father,黑的是 son.在此情况下我得到的是 B78 所以我将要写下 0B78.记住 handle 在每次运行程序时都会改变. 

![](img/dea7556534d3f1407ebedc393ccc5eb6.jpg)

在图示的 nop 处下断点,然后运行程序.看 registers window,如果当程序停止时,EAX=1 那意味着 son 已经从 father 处脱钩了,我们可以关闭 OLLY.如果 EAX=0 那么你要检查那几行代码,因为你可能犯了写错误(也许是写错了 handle).   
  
  
现在关闭 OLLYDBG,然后再次打开他.不要载入任何东西.前往"文件/附加"然后找 son,附加它(我们搞定它的 father 了:X).   
  
一旦附加成功,程序将在 INFINITE LOOP 中 RUNNING ,需要按 F12 to pause the 去暂停程序,但是等一下,在做这个之前,我们先得把 EB FE 改回原始值(那些我叫你记下的值,你有记下吗?) 55 8B,然后我们将停在 OEP,已经准备好 dump 了. 

![](img/8d516f6a6ed7847987bb03be84a649ff.jpg)

![](img/3f53638a4ec922193020a80c8f938154.jpg)

我们已经准备好 dump 了,所以打开 LORD PE 然后找 Getright.exe 

![](img/fa28ee091488abf5617e63dacc2d1c0b.jpg)

选择 Getright.exe 进程然后选择 INTELLIDUMP 然后点击 DUMP FULL, 保存后启动 PEEDITOR (从 www.google.com 上搜索它)然后写入正确的 ENTRY POINT 值.   
在这儿不要填写 538540,而写入文件偏移,就像这样: 538540-400000 = 138540. 

![](img/11d7ff714c1516e4e226ff6ca5903f0b.jpg)

那就是 full dump.在本参考教程的第二部分我们会学习怎样去找 magic jump 从而完成 import table, 修复 IAT,使程序运行,因为它设下了一些陷阱来避免被 unpack.   
  
现在我们保存转储(dump)文件(我把它命名为 tute.exe),和原始的一样,只是 IAT 不同.   
  
第二部分再见.   
  
Ricardo Narvaja   
  
翻译(西班牙语翻译为英语)DEGETE 和 Tenshin,(英语翻译为中文)FTBirthday.自由传播,但是请保留作者的姓名.  
    
第二部分   
在本参考教程的第一部分我们学习了如何正确地转储(dump)Getright 5. 现在我们将要去找神奇跳转,这样 IAT 会被正确地转储下来,而不用手工修复了. 要完成这一点,我们需要打败程序中的一些陷阱, 并使它们即使在检测到被执行脱壳时也无所作为.   
  
让我们开始吧!   
  
第一步:如何找到 IAT 的起始点和终点   
  
在我们开始之前,先在安全的地方保存一份我们在先前第一部分教程中得到的转储文件的副本. 我将它命名为 TUTE.exe 记住这个名字.   
  
用 Ollydbg 载入 TUTE.exe. 我们将要去寻找 IAT.

![Image9.gif](img/968d5e3e8d4f1cfa6c348d0c009f4dec.jpg)

正如我们在上图中看到的,如果我们稍加跟踪,我们很容易发现一个   
  
CALL \[xxxxxx\] 或者 JMP \[xxxxxx\] 间接地带有它们即将跳往 IAT 中某处的内存地址值.在这张图中,我们可以看到 CALL \[5e9d94\]. 那意味着这个 call 将要前往这个内存地址,实际上就是 IAT 表的入口值.在它的右边我们可以看到"kernel32.Getversion" (黄色). 这就意味着这是一个可以在 IMPORT RECONTRUCTOR 上被找到的正确的入口值.   
  
让我们看一下转储(DUMP)窗口.在转储(DUMP)窗口右击,选择"前往 表达 5E9D94".

![Image10.gif](img/e6b70a89e5dac919edbb4bdd4ab87dfe.jpg)

这就是看上去很不错的 IAT. 正如你所看到的入口 5E9D94 指向 77E5D142,那个值在我的机器里是 API GetVersion.这个入口是正确的,此入口周围的其它一些入口也是指向类似 7xxxxxxx 的值.   
  
注意在其它的机器上值 7xxxxxx 可能会有所不同.但是这些值都是相互类似的. 现在我们稍微往上看一看,去找此表的起始点.我们到达这里: 

![Image11.gif](img/5ac5a144a47c635678e50fa8798ccc6f.jpg)

红线标记了此表的起始点.在此线的上面没有其它任何 API 值了.现在我们知道表从哪儿开始了   
  
TABLE STARTS= 5E99EC   
  
现在到右下角转储区去找 IAT 表的终止点.正如我们在图中看到的有两个可能的终止点. 如果我们吃不准最好选后面那个.但是我们有更好的方法.

![Image12.gif](img/bef35222b3d891defad8aac38b30835c.jpg)

选择任何一个不确认的值,然后将表向上翻页.停在 401000.我们必须去找一个 CALL \[xxxxx\]或者一个 JMP \[xxxxx\].右击鼠标并选择 SEARCH FOR BINARY STRING 

![Image13.gif](img/6978de47be579d29ece91b51e10af9e4.jpg)

我们将要去寻找那些不确认的入口中的一个,让我们试试 5EA25C.记住逆序写这个值如下图所示.

![Image14.gif](img/f96e442d30da51bf483ca7a9128c2674.jpg)

到了这里

![Image15.gif](img/88b280823d85bc50b424775db49ca185.jpg)

稍稍往上翻一下 

![Image16.gif](img/7439f9f814474b23a590cfec99c2e369.jpg)

我们看到那儿有一个使用此值的 CALL,所以那些不确认的值也都是属于 IAT 的.    
  
现在我们知道了   
  
END OF TABLE= 5EA2BC   
  
终点值是用来计算表的长度的.这是很简单的一步:    
  
LENGTH= END-START   
  
LENGTH=5EA2BC-5E99EC   
  
LENGTH= 8D0   
  
在纸上写下 OEP,表的起始点和长度,在使用 import reconstructor 时你将用到这些值.    
  
第二步:找出那些不指向任何 API 的错值.   
  
当 GETRIGHT.exe 正在进行转储时,这个问题变的很容易.然后选择 VIEW-MEMORY 你将看到哪些入口指向了 dlls 而哪些则指向了错误的地方.   
  
无论如何如果我们尝试用 revirgin 或 import reconstructor 则会有很多入口无法解决.这就是我们为什么请出哈里波特带来一些魔法.

![Image17.gif](img/396fce140abfbd31f521ea56d5be8ff8.jpg)

正如在表中看到的,还存在一些(黄色标记)没有解决的入口.通常对付这种情况的方法是从我开始转储起就跟踪,然后追踪每一个调用 API 的 call,写下名字最后用 Import Reconstructor.   
  
如果自由很少几处没有解决的入口这样做将非常简便,但是如果有很多的话,这项工作将令人难以忍受,所以让我们看看另一种方法.   
  
第三步:如何去找出 MAGIC JUMP    
  
这里说明一点：我们已经知道程序的父进程和他的子进程是相同文件，但要用不同的句柄装入两次，所以他们变成两个不同的进程。一方面父进程的 OEP 是 5F90B9,父进程从这里开始运行.

![Image18.gif](img/53ccc346dd24270a3e88f9f391ba3158.jpg)

另一方面我们知道当子进程被转储时其 OEP 是 534E90,但我也要告诉你，子进程也是以相同的父进程的入口点 5F80B9 开始运行.子进程运行并且解出自己的 IAT 然后跳到子进程引起错误的 OEP,我希望你记住这一点,我在第一部分曾经讲过.   
  
如果你相信以上所说,我告诉你,跟踪父进程直到父进程开始转变为两个进程,父进程和子进程,找到那个地方,那非常有用.我已经找到了那个地方,那是个条件跳转,至于如何找到的,我以后再告诉你.   
  
顺便一提它不是从父进程那复制 IAT 到它的子进程，却是从子进程那解出自己的 IAT。这应该是一大麻烦，因为我们不能在他从父进程脱离前用 OllyDbg 进入子进程。   
  
这是个烦恼,因为我不知道如何进入子进程去观察它如何解出自己的 IAT.   
  
首先我尝试着从转储程序 TUTE.exe 中找一个错误的 entry. 

![Image19.gif](img/3fa309a6885db94e5aad4fc8a4ccb1c4.jpg)

我选择了 5E9C34 但是你可以选择其它任何错误的 entry.正如你所看到的,它的值为 DF5070.   
  
按老规矩记住要将 IsDebuggerPresent 的值设为零，清除所有以前的 BPX、BP WaitForDebugEvent 然后单击运行.   
  
我们将停在此断点或者某个例外上.那时,打开 PUPE 选择进程中 2 个进程中的上面一个(因为他是子进程,是我们需要的).右击进入 parcheando 窗口选择 4 字节，如图在地址里填入错误的 entry offset 5E9C34.然后点击 BUSCAR. 你将会看到没什么变化,因为所有值都是零.

![Image20.gif](img/e9056d8bcf437aab4fd25244fa960a35.jpg)

然后不断的在 OD 中点击运行、在 PUPE 中点击 BUSCAR,并检查字节窗口中的值.   
  
第一次变化可能会很久才发生, 但是记住不断的在 OllyDbg 运行、在 PUPE 中点击 BUSCAR,并检查字节窗口中的值，直到字节窗口中的值从零开始变化.

![Image21.gif](img/62587172e7d55d77b84580f63a8e0ca4.jpg)

如图所示,那个值变化了. 现在再重复几次,你将会看到: 

![Image22.gif](img/19a9921fc0eb99b6f2501641e2343f6f.jpg)

 

子进程已经完全解出并赋值 IAT. 我们必须在那个值第一次变化之后子进程赋值之前进入子进程. 这是个问题,但我耍了一个把戏解决了.   
  
我们将要重复上述过程.关闭 Parcheando--PUPE 的窗口,因为当重启后进程的句柄会变.   
  
重复上述过程直到你在 PUPE 的窗口中观察到那个值的第一次变化.你应该看到类似这样的东西: 

![Image21.gif](img/ca75475d36f102f0c02c12b6ced607af.jpg)

 

很难找到一种好方法, 所以我决定在某个 API 上制造一个死循环.就选 GetProcAddress 吧.所以现在我们要去父进程中找到它.选择 VIEW-EJECUTABLE MODULE 查找 KERNEL32.dll 这是 GetProcAddress 所属的 dll. 

![Image23.gif](img/7feba489a4205b6c1dfc56f4ff4d6780.jpg)

右击 VIEW-NAMES 查找 api GetProcAddress 

![Image24.gif](img/ce007187ae240b5f0d1fddb0fb007c0d.jpg)

 

写下这个地址(红箭头).现在在 PUPE 中选择子进程,让它去找 search (BUSCAR)这个地址值. 不同的机器这个值会不同. 我的是 77E5B332.在字节窗口你会看到前两个字节是 55 8B,所以在纸上写下,并在 PUPE 中将其改为 EB FE 然后按下 PARCHEAR 

![Image25.gif](img/17f43a9f91a92f8e41a07fbd01906741.jpg)

在那儿现在子进程将会处于死循环中,我们可以说它已经睡着了. 我们现在必须把它从它的父进程那儿 unhook 下来.   
  
在父进程窗口任意处点击鼠标右键,选择"新建起源"然后写下下面的代码   
  
PUSH (son’shandle)   
  
Call DebugActiveProcessStop   
  
Nop (在这儿 BPX 并检查是否 EAX=1) 

![Image26.gif](img/c9661644ebdb2c1dd30ced0f56aaeba9.jpg)

在 0042F00A 下中断，然后运行一下停在 0042F00A 处。看看寄存器窗口的 EAX 值，如果这个值＝01 就表示子进程和父进程分离了。如果＝00 那么可能是子进程的句柄填错了, 你可以在下面重新写入代码再次运行，直到 EAX＝01 时就可以关闭 OllyDbg 了（杀掉父进程！）然后就可以进入子进程了。   
  
重新运行 Ollydbg（不要加载程序）附加上子进程.   
  
再次在 PUPE 中恢复原来的代码 55 8B 并按 "Parchear" 再看看程序中的代码又还原成了 77E5B332. 

![Image27.gif](img/8db328b241de62e99fe207ce3b19ae44.jpg)

程序将会中断,就像你在下图中看到的. 

![Image28.gif](img/b176e269ea07ad1f0e5edab1ac1b8ba8.jpg)

让我们看一下那个错误的 entry,右击转储 dump 窗口选择 GOTO EXPRESSION 5E9C34. 

![Image29.gif](img/c5f6dd0468a2f1d9894c0e4bce9b605d.jpg)

Bp GetProcAddress 然后按 RUN. 你可以看到当中断时错误的 entry 已经被重写了,但是表并不完整. 重新打开一个 ollydbg 载入 tute.exe.如果在 5e9c8c 下面还有一个错误的 entry 就看那儿.

![Image30.gif](img/67df60557e367b4f01eec85bcc8efbe1.jpg)

以 5E9c98 处的错值为例,因为那儿将会被写入 DF513C.

![Image31.gif](img/2b2faf2105f99de3c4fdd37758fdca8a.jpg)

回到 Getright.exe (第一个 ollydbg)看一下堆栈. 那儿的 API 被 Df4cB2 处的 called 调用并将会返回到 DF4cB8.    
  
在主窗口中选择 GOTO EXPRESSION 0Df4cb8.

![Image32.gif](img/36b8b33b71576afb80f2dabab883dfa4.jpg)

下图是我的机器中这个 call 了 API 的 call 的地址. 在 API 返回处 BPX(这个 call 的下一行).

![Image33.gif](img/931f457edb02c9311a44ac359fe136bc.jpg)

点击 RUN   
  
现在在错误 entry 5E9c98 上下 HARDWARE BPX ON WRITE 然后点击 RUN.当在那儿写入时将被断下.

![Image34.gif](img/3eb2d27a348a7b4a741665bdb673200c.jpg)

你可以看到在前面一行保存着错误值.后面指出这个错值从\[ebp-394\]处来.所以在转储窗口中选择 GOTO EXPRESSION EBP-394. 那儿你将会看到这个值.下 BREKPOINT HARDWARE ON ACCESS.

![Image35.gif](img/9f81cdb289574632a0c6a0e49102010b.jpg)

如果我按下运行键，我将看到无论值是好是坏都在这一行被保存.看附近有一个 call BPX 它. 

![Image36.gif](img/6307047a0b91ee66348b7c951f3c394b.jpg)

你可以看到这是关键 call.因为当它结束时 eax 带着稍后将写入的值. 我测试了这个 call 里的每一个跳转(只有 5 到 6 个,通常都是这样):   
  
00DF4B68 /75 03 JNZ SHORT 00DF4B6D   
  
00DF4B79 /75 07 JNZ SHORT 00DF4B82   
  
00DF4B8D /74 0C JE SHORT 00DF4B9B   
  
00DF4B92 /74 1B JE SHORT 00DF4BAF   
  
00DF4B99 ⁷⁵ F4 JNZ SHORT 00DF4B8F   
  
胜利者是 DF4B8D 处的跳转. 正如我先前所说的在所有的 armadillos 里都是一样的,所以当我们在脱其它 armadillo 时,通过跟踪这些代码我们可以很容易发现这个好跳转.    
  
00DF4B8D /74 0C JE SHORT 00DF4B9B   
  
它必须永远跳转,所以我们做如图修改:

![Image37.gif](img/9329361ae6bdd99e85366987be6e23f9.jpg)

清除 BPX 点击 run. (记住清除 hardware bpx). 当程序走到我们修改为死循环的指令那儿时输入表已经修复了,再也没有坏值了. 然后它在子进程的 OEP 处停下.在纸上写下 magic jump 的值(EB 0C),记住.   
  
我们将要重复那些步骤.当我们走到上次我们 unhook 子进程那一步时,我们将要在 magic jump 处设置一个 infinite loop 代替在 API 上设置.

![Image38.gif](img/e5828ec24539d6da88342c03b2a099cc.jpg)

在那儿从父进程处 unhook 子进程,然后它将在 magic jump 处死循环.   
  
附加子进程,按 RUN, 然后停止(F12). 它将停在 magic jump. 

![Image39.gif](img/8cbb40233402ce1208164e8cdeed39ec.jpg)

现在还没有任何改变,因为 magic jump 还没起作用.把它改回先前的值(EB 0C).

![Image40.gif](img/007e8535e3c8df1dd860b2a804f13713.jpg)

如果我们运行程序,它将停在 OEP,而且输入表也很完美和完整.现在我们可以利用这个进程用 Import Reconstructor 去修复在第一部分中转储的程序了.

![Image41.gif](img/4245536443dbfddc809584e52fd547b8.jpg)

所有的值都是好的.最后那几个是指向 dll 的. 

![Image42.gif](img/44aa20c76ce326e88c43aac8ef2196a8.jpg)

那些值指向 Exxxxx,如果我们看看 VIEW-MEMORY 我们可以得知那是 dll.

![Image43.gif](img/28a97a17789ff10a89ce17dd801fc8c0.jpg)

现在我们要用一下 Import Reconstructor.   
  
打开 Import Reconstructor 在"Attach to an Active Process"对话框选择这个进程.

![Image44.gif](img/ea8f7efcff6779414af51d2ca0ecabf7.jpg)

装载这些 dll 需要很长一段时间.当完成后写下 OEP 的值,表的起始地址值和表的长度,不要按 IAT AutoSearch.

![Image45.gif](img/e0f962b7433334ecc1a2029a796f187e.jpg)

如图在 IMPORT RECONSTRUCTOR 中那些值是这样计算的:你在纸上写下的值-Image Base. 比如: OEP=534E90, Image Base=400000, 534E90- 400000= 134E90. RVA 也一样 (表的起始地址值).   
  
现在按 GET IMPORTS. 所有的 entries 都修复了.

![Image46.gif](img/dcd5d885cd2dfb62c0fdf0409f98d47c.jpg)

现在按 FIX DUMP 载入转储文件.我把它叫做 TUTE.exe

![Image47.gif](img/974dcf82b6d95be0969a085e439fbc6d.jpg)

它将被保存为 TUTE\_.exe.原先的文件保持原样.   
  
到此 armadillo 保护已经被击败了.   
  
第四步:如何击败 BOSS   
  
如果我们尝试运行 TUTE\_.exe 它会无故终止.上面我刚说已经击败了 armadillo,但是脱壳文件不能正常运行.见鬼了?   
  
程序员(bad guys)不想让 getright 5 在我们的机器中自由运行、解压,然后他们做了手脚.   
  
在 OLLYDBG 中载入 tute\_.exe.    
  
运行,你会发现它突然终止.   
  
BPX GetEnvironmentVariableA 当它中断时我们需要改变条件跳转,当从 API 返回程序时.   
  
T 看下图.

![Image48.gif](img/9572979ec0dc35f5753be42793314e30.jpg)

这是中断点 BPX.现在返回程序用 EXECUTE TILL RETURN 然后按一下 F7.

![Image49.gif](img/ad4b841e65b9463feec90ebe314e3461.jpg)

这一流程将会重复多次,所以要熟练掌握.   
  
修改跳转.

![Image50.gif](img/0ac00d97cf3d7594019a294e1092ecfa.jpg)

依次类推. 

![Image51.gif](img/76238bcdc6d16ff9db79ecba8789da6c.jpg)

把 JNZ 改为 JMP 你可以找到所有 calls API 然后修改临近的跳转.我把那些跳转罗列出来. 

![](img/150e9c216cf8c5b83a4086222cd04dda.jpg)

把 JNZ 改为 JMP 你可以找到所有 calls API 然后修改临近的跳转.我把那些跳转罗列出来. 

![](img/014593a260ecfef66ba253f4997e5c52.jpg)

我想这个 armadillo 已经被搞定了. Ricardo Narvaja 英文翻译 Tenshin. 中文翻译:FTBirthday 完成了，好累哦……