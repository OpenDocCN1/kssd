

# Armadillo v3.x

> 原文：[`www.pediy.com/kssd/pediy06/pediy6326.htm`](https://www.pediy.com/kssd/pediy06/pediy6326.htm)

Armadillo v3.x for copymem-II 脱壳完全篇――上篇   
                    ――查找 OEP 和代码修复   
  
  这是一篇脱 Armadillo 加壳软件的个人的经历过程，本人特别声明：这个过程是参照了 Bighead\[DFCG\]\[YCG\]翻译的 Ricardo Narvaja 的“Getright 5 脱壳和重建”文章操作的，可以说是个照葫芦画瓢做的。感谢 Bighead\[DFCG\]\[YCG\]和相关的翻译者，感谢 Ricardo Narvaja 的文章。   
   这次拿来练习的软件是 FCG 的 peterdocter 老大提供的软件，下载地址：http://peter.88vip.com/armadillo3.rar   
工具：   
Ollydbg v1.09d 中文版   
PuPe 2002 版 感谢 yesky1 兄提供   
LordPE Deluxe-1.4 by yoda   
Import Reconstructor 16f   
   准备好笔和纸（不能少喔！），一杯茶和好是心情^\_^.让我们开始一个艰苦和漫长的脱壳过程吧（因为我菜！）。Armadillo 是当今猛壳之一啦。其 CopyMem-II+Debug-Blocker 的加壳方式是非常强劲的，好在有牛人在前面带路，虽艰难倒也不迷失方向。   
  （根据 Ricardo Narvaja 的文章：必须说明一下，此教程仅在 Windows XP 调试。不要尝试在 Windows 98 或者 Windows 2000 下进行。事实上是因为只有 Windows XP 已经与必要的 API 脱钩了。我是 windows2003 应该符合要求了。）   
  开始吧！～   
SETUP 1  BP IsDebuggerPresent   
  用 OllyDbg 装入需要脱壳的软件 armadillo3.exe（其实是个 crackme），开始我们的第一道关，反 anti\_debug 这是每个 Armadillo 加壳软件都要做的第一件工作。在 OD 的命令行窗口中键入 BP IsDebuggerPresent。记住，我们应保留大小写字母正常状态，因为如果我们键入任何命令有异，我们可以在命令栏右侧的显示来反映出错误信息（未知的命令）。所以要正确写入 BP IsDebuggerPresent 然后按回车键。如果在按回车键后没有信息显示，这是表明此 BP 运行良好。（最好把 OD 的异常选择项全部勾上）F9 运行一下就会来到 IsDebuggerPresent 函数的入口地址：   
77E2AC39 >MOV EAX,DWORD PTR FS:\[18\]                ; <--中断在这里   
77E2AC3F  MOV EAX,DWORD PTR DS:\[EAX+30\]   
77E2AC42  MOVZX EAX,BYTE PTR DS:\[EAX+2\]            ; <--运行到这里   
77E2AC46  RETN   
在 77E2AC42 地址处停下，看看 DS:\[EAX+2\]指向的地址 7FFDF002 的指是 01（这个地址可能和你的不同），修改这个值为 00 因为 Armadillo 通过 IsDebuggerPresent 函数来检查是不是被调试器调试，如果有调试器就会赋 EAX 值为 01 没有就赋 EAX 值为 00。修改这个值就是告诉他没有调试器，自然不会出错拉！记下这个地址，以后经常会用到的。   
SETUP 2  BP WaitForDebugEvent   
  下面该干什么呢？查找 OEP 的内存出现的地方，如果你用 Peid 来查看软件的 OEP 会得到一个错误的值。因为 Armadillo 会修改 OEP 地址。来看看怎么查找呢？在命令行中输入 BP WaitForDebugEvent 然后回车。运行一下就会来到函数的入口：   
77E605B6 >PUSH EBP    -停在入口   
77E605B7  MOV EBP,ESP   
77E605B9  SUB ESP,68   
77E605BC  PUSH ESI   
在堆栈(Stack)窗口我们可以见到在此 API 函数有关参数的全部信息   
0012E220   0043776E  /CALL 到 WaitForDebugEvent   
0012E224   0012EFF4  |pDebugEvent = 0012EFF4  //注意这个地址   
0012E228   000003E8  Timeout = 1000. ms   
这个函数的作用我不知道，但我知道 0012EFF4 地址就是 OEP 将会出现的地方。来到转存窗口 G 0012EFF4 ：   
0012EFF4  01 00 00 00 10 00 00 00   ... ...   
0012EFFC  00 00 00 00 A8 F0 12 00  .... .   
0012F004  00 00 50 00 64 EF 12 00  ..P.d?.   
0012F00C  5C F0 12 00 00 00 00 00  ?.....   
0012F014  5C F1 12 00 00 00 00 01  ?....    
0012F01C  58 EF 12 00 AC F0 12 00  X?. .   
0012F024  AC F2 12 00 34 5A F3 77   .4Z 體   
0012F02C  58 BB F7 77 FF FF FF FF  X 击 w����   
0012F034  E7 DF F3 77 E1 26 F4 77  邕體?魒   
这一步的作用就是通过函数找到这个内存地址，因为他会出现真正的 OEP 入口。为什么？我也不知道。   
SETUP 3 Bp WriteProcessMemory   
命令栏写入 Bp WriteProcessMemory 断点然后点击 RUN。可能会在某些异常停下，那你可以通过按下 Shift + F9 顺利地绕过。这个软件还会出现未注册的 Armadillo 加壳提示，越过他。只要我们在 WriteProcessMemory API 停下了，千万不要触动任何键，来看看能有什么收获――   
堆栈窗口可以看到第一个块的全部信息：   
0012E0C0   0043AFC2  /CALL 到 WriteProcessMemory 来自 armadill.0043AFBC   
0012E0C4   00000044  |hProcess = 00000044 (window)   
0012E0C8   00425000  |Address = 425000   
0012E0CC   003A2630  |Buffer = 003A2630   
0012E0D0   00001000  |BytesToWrite = 1000 (4096.)   
0012E0D4   0012E1DC  pBytesWritten = 0012E1DC   
先来看看 Ricardo Narvaja 的文章里说的：   
是父进程将要复制到它的子进程的指引。此信息是保存在一个缓存之内，父进程来自 003A2630，并且它将会从 00425000 开始以每 1000 字节块方式复制到子进程（字节写入），所以如果我们找到此方案，我们就可以很容易理解到子进程第一个块是完全空的，然后当子进程尝试执行 OEP 时将它返回一个错误信息，因为在这个点并没有什么数据，因此父进程获得报告，同样我们可以在父进程看到有关报表，所以它停止运行，在下一个错误前复制必要的数据块然后继续运行。该数据副本的数值大小是 1000 字节，所以当程序试图执行任何超过此块时，各种错误将会发生，然后错误将会向父进程报告，然后父进程将会复制另外的 1000 字节的块，诸如此类。   
  这个块开始在 425000 一直到 425FFF。OEP 必定在其值之内。让我们来留意刚才的那个转存窗口：   
0012EFF4  01 00 00 00 B0 0E 00 00   ...?..   
0012EFFC  B8 0E 00 00 01 00 00 80  ?.. ..€   
0012F004  00 00 00 00 00 00 00 00  ........   
0012F00C  D0 51 42 00 02 00 00 00  蠶 B. ...   
0012F014  00 00 00 00 D0 51 42 00  ....蠶 B.   
0012F01C  D0 51 42 00 00 00 00 00  蠶 B.....   
0012F024  00 00 00 00 00 00 00 00  ........   
0012F02C  13 00 00 00 94 10 00 C0   ...?.   
注意窗口中蓝色的字节 004251D0（应该倒过来），这个值是在 425000 到 425FFF 之间的值。那么软件的 OEP 应该的 004251D0    
呵呵，这段我先怎么也不明白。现在让我来解释一下：（我编程不行，所以说的不一定正确）   
WriteProcessMemory 函数是内存复制函数，大家知道 Armadillo 的 copymem-II 方式会在内存中生成同名的 2 个进程，其中一个是父进程，一个是子进程。那么子进程是怎么生成的呢，就是从父进程中用 WriteProcessMemory 函数复制来的。（这二个进程的窗口句柄不同，并且互相掌握着对方的 OEP）SETUP2 就是得到了内存复制信息的内存地址，因为复制是按 1000 的块进行的，所以第一个复制的块中必定包含了子进程的 OEP 地址。这个地址的值在第一块的值之间。所以上面的转存窗口中的蓝色那个地址就是真正的 OEP 了。Ricardo Narvaja 真牛呀！   
  OEP 找到了，是不是就能 dump 出来呢？试试 dump 出来的代码又是错误的。可以肯定的是有一个函数在解密后又破坏了复制的数据，这就引出了下面来查找破坏数据的函数的方法。接着来――   
SETUP 4 NOP 填充 Cripter?Call (不让程序破坏解密的代码)   
父进程会为他的子进程解密一个块。但是仍有 Cripter Call 那些加密或破坏旧块来避免被转储。现在就以实际操作来查找这样的 Call 然后用 NOP 填充替换它。这是个艰苦和复杂的工作.   
首先在 cpu 窗口中右击鼠标出现功能菜单，选择中断的〔条件记录〕窗口。在条件表达式中填 〔ESP+U〕然后选择〔永远记录条件表达式〕。这个用来检查复制的数据块。   
怎么查找 Cripter Call 呢？   
在 WriteProcessMemory 函数的入口处停下。Alt+K 打开调用堆栈窗口：   
呼叫堆栈    
地址       堆栈       例程 / 参数                         调用来自                          Frame   
0012E0C0   0043AFC2   ? kernel32.WriteProcessMemory       armadill.0043AFBC   
0012E0C4   00000044     hProcess = 00000044 (window)   
0012E0C8   00425000     Address = 425000   
0012E0CC   003A2630     Buffer = 003A2630   
0012E0D0   00001000     BytesToWrite = 1000 (4096.)   
0012E0D4   0012E1DC     pBytesWritten = 0012E1DC   
0012E1E8   00439D34   ? armadill.0043A07B                 armadill.00439D2F   
0012E21C   00437CA8   armadill.004398F5                   armadill.00437CA3                 0012E218   
0012F5BC   00434384   armadill.00436099                   armadill.0043437F                 0012F5B8   
0012FD20   00434BDA   armadill.00433CF4                   armadill.00434BD5                 0012FD1C   
0012FF38   0043CF87   armadill.00434940                   armadill.<ModuleEntryPoint>+0C9   0012FF34   
0012FF3C   00400000     Arg1 = 00400000 ASCII "MZP"   
0012FF40   00000000     Arg2 = 00000000   
0012FF44   00141EFB     Arg3 = 00141EFB   
0012FF48   0000000A     Arg4 = 0000000A   
  
看看这里：   
0012E0C0   0043AFC2   ? kernel32.WriteProcessMemory       armadill.0043AFBC   
现在父进程的 0043AFC2 地址处调用了这个函数，向下看看：   
0012E1E8   00439D34   ? armadill.0043A07B                 armadill.00439D2F   
通过这个我们知道程序在 00439D2F 处也调用了这个函数。双击来到：   
00439D27  MOV ECX,DWORD PTR SS:\[EBP+C\]   
00439D2A  PUSH ECX   
00439D2B  MOV EDX,DWORD PTR SS:\[EBP+8\]   
00439D2E  PUSH EDX   
00439D2F  CALL armadill.0043A07B  //这里调用了上面的函数   
00439D34  ADD ESP,0C   
00439D37  AND EAX,0FF   
00439D3C  TEST EAX,EAX   
00439D3E  JNZ SHORT armadill.00439D47   
00439D40  XOR AL,AL   
00439D42  JMP armadill.0043A074   
00439D2F  CALL armadill.0043A07B 这个 Call 是个解密的函数，那么肯定有一个破坏解密的函数。查找另外一个同样的 Call 的地方就是我们要找的地方，在 OD 中的：   
00439D2F  CALL armadill.0043A07B 处右击功能菜单，选择〔查找参考〕……〔调用目标〕选项就会打开全部的调用什么 Call 的地方：   
参考位于 armadill:.text 到 0043A07B   
地址       反汇编                                    注释   
00439D2F   CALL armadill.0043A07B                    (初始 CPU 选择)   
00439FEA   CALL armadill.0043A07B        //原来这里有一个^\_^   
双击 00439FEA   CALL armadill.0043A07B 就会来到这个梦中的地方：   
00439FDA  MOV ECX,DWORD PTR DS:\[462AAC\]   
00439FE0  MOV EDX,DWORD PTR DS:\[462AB0\]   
00439FE6  MOV EAX,DWORD PTR DS:\[EDX+ECX\*4\]   
00439FE9  PUSH EAX   
00439FEA  CALL armadill.0043A07B  //这里 这里 这里   
00439FEF  ADD ESP,0C   
00439FF2  PUSHFD   
00439FF3  PUSHAD   
00439FF4  JMP SHORT armadill.0043A021   
没有别的说了，nop 他。   
  
SETUP 5 运行该 API 然后在 OEP 中生成一个死循环   
现在是时侯要执行 WriteProcessMemory 一次了。做法是我们可以在两个选项之间作出选择。一是要转到菜单 Debug|Execute till return 然后它将会运行直到 RET，然后按一次 F7。二是要转到堆栈第一行然后在那里设置 BPX 然后就 F9 运行它。 这段我就复制了 Ricardo Narvaja 的原文。我没得说了。反正会到 API 的调用处就可以了：   
0043AFB3  PUSH EDX                                          ; |Address   
0043AFB4  MOV EAX,DWORD PTR DS:\[462A9C\]                     ; |   
0043AFB9  MOV ECX,DWORD PTR DS:\[EAX\]                        ; |   
0043AFBB  PUSH ECX                                          ; |hProcess   
0043AFBC  CALL NEAR DWORD PTR DS:\[<&KERNEL32.WriteProcessMe>; WriteProcessMemory \*\*\*\*\*\*\*\*   
0043AFC2  TEST EAX,EAX   
0043AFC4  JNZ SHORT armadill.0043B001   
0043AFC6  PUSHFD   
0043AFC7  PUSHAD   
0043AFC8  JMP SHORT armadill.0043AFF5   
现在是 PUPE 运作的时候了，PUPE 是一个西班牙语的工具。你可以到 www.google.com 网站搜索它，在下一行我们将要在子进程的 OEP 内生成一个死循环。此时将会静止子进程直到我们能够叫醒他。不过我没有找到，还是 yesky1 兄提供了这个工具。再次感谢！   
打开 PUPE 选择进程中二个 armadillo3.exe 的上面的一个，位于上面的子进程，下面的父进程。我们选择的是子进程。然后在它上面按右键并选择 "Parchear"。打开了 Parchear 窗口。填入参数，No de bytes = 2 和 Introducir la direccion = 4251D0（子进程的 OEP），然后点击 Buscar)记下在对话框见到的原始字节然后用死循环 EB FE 来替换原始字节，点击 " Parchear" 然后 INFINITE LOOP (死循环)便会就绪。   
呵呵，第一次使用还真不知道这是干什么呢，其实就是用 PUPE 修改入口的代码，让他变成跳转到 EIP 地址的代码。这样程序在入口点就会停下。跳向自己的地址，程序会向下执行吗？   
  
SETUP 6 BP WaitForDebugEvent 和 NOP 填充 API   
 在命令行中键入 BP WaitForDebugEvent 然后按回车键，最后点击 RUN   
（为什么要下这个 API 请参照 leo\_cyl1 大虾的文章）   
当停止后，会来到这里：   
77E605B6 >PUSH EBP  //停在这里   
77E605B7  MOV EBP,ESP   
你要切记：在任何时候都绝对不要运行这个 API！留意一下在 (转储)窗口中的报表，然后转到 STACK (堆栈)窗口。这段我也只能抄，我太菜了。现在来看看堆栈窗口中的信息：   
0012E220   0043776E  /CALL 到 WaitForDebugEvent   //调用信息   
0012E224   0012EFF4  |pDebugEvent = 0012EFF4   
0012E228   000003E8  Timeout = 1000. ms   
可以看到，如果我们运行这个 API，我们将要转到 0043776E 所以在主窗口用 Go to|Expression =0043776E 能到达那里。   
0043776E  TEST EAX,EAX     
  
再一次提醒你：不要运行这个 API！因而在 0043776E 内按右键然后选择\[新建起源\]。这是跳过该 API 的正确做法。现在我们必须用 NOP 来填充调用到该 API 及其有关 Push。   
把下面的代码用 nop 修改掉：   
00437759  MOV AL,BYTE PTR DS:\[E8686133\]   
0043775E  ADD EAX,DWORD PTR DS:\[EAX\]   
00437760  ADD BYTE PTR DS:\[EBX+FFFA3895\],CL   
00437766  CALL NEAR DWORD PTR DS:\[EDX-1\]   
00437769  ADC EAX,<&KERNEL32.WaitForDebugEvent>   
（不知道为什么这段代码是动态的）   
改为：   
00437759  NOP   
0043775A  NOP   
0043775B  NOP   
0043775C  NOP   
0043775D  NOP   
0043775E  NOP   
0043775F  NOP   
00437760  NOP   
00437761  NOP   
00437762  NOP   
00437763  NOP   
00437764  NOP   
00437765  NOP   
00437766  NOP   
00437767  NOP   
00437768  NOP   
00437769  NOP   
0043776A  NOP   
0043776B  NOP   
0043776C  NOP   
0043776D  NOP   
0043776E  TEST EAX,EAX   
  
SETUP 7 打补丁   
  
“当处理这个步骤时要特别小心! 多数人并不知道究竟是如何打补钉的正确方法，所以要尽力领会我在这里是怎样做，并且你还需要在其它情况下会完成。” 我是试了几次才成功的！   
第一步是更改此跳转：   
0043776E  TEST EAX,EAX   
00437770  JE armadill.004398A5  //把这里修改为 Jmp 00401000   
Why？ 因为父进程的偏移是 00401000 我们需要在那里打补丁。   
现在 go 00401000 准备打补丁了：   
00401000  ADD BYTE PTR DS:\[EAX\],AL   
00401002  ADD BYTE PTR DS:\[EAX\],AL   
00401004  ADD BYTE PTR DS:\[EAX\],AL   
00401006  ADD BYTE PTR DS:\[EAX\],AL   
00401008  ADD BYTE PTR DS:\[EAX\],AL   
来看看内存镜象中的数据，Alt+M 打开内存镜象窗口：   
00400000   00001000   armadill              PE header     Imag   R         RWE   
00401000   00025000   armadill   CODE                     Imag   R         RWE   
00426000   00001000   armadill   DATA                     Imag   R         RWE   
00427000   00001000   armadill   BSS                      Imag   R         RWE   
00428000   00002000   armadill   .idata                   Imag   R         RWE   
0042A000   00001000   armadill   .tls                     Imag   R         RWE   
0042B000   00001000   armadill   .rdata                   Imag   R         RWE   
0042C000   00003000   armadill   .reloc                   Imag   R         RWE   
0042F000   00020000   armadill   .text      code          Imag   R         RWE   
0044F000   00010000   armadill   .adata                   Imag   R         RWE   
0045F000   00010000   armadill   .data      data,imports  Imag   R         RWE   
0046F000   00010000   armadill   .reloc1    relocations   Imag   R         RWE   
0047F000   00040000   armadill   .pdata                   Imag   R         RWE   
004BF000   00011000   armadill   .rsrc      resources     Imag   R         RWE   
  
看看程序的代码段开始于 00401000 结束于 00425FFF   
在转存窗口中把 OEP 的入口修改为 400000，修改后是这样的：   
0012EFF4  01 00 00 00 B0 0E 00 00   ...?..   
0012EFFC  B8 0E 00 00 01 00 00 80  ?.. ..€   
0012F004  00 00 00 00 00 00 00 00  ........   
0012F00C  00 00 40 00 02 00 00 00  ..@. ... //\*\*\*\*\*\*\*   
0012F014  00 00 00 00 00 00 40 00  ......@. //\*\*\*\*\*\*\*   
0012F01C  00 00 40 00 00 00 00 00  ..@..... //\*\*\*\*\*\*\*   
因为每个块循环以 1000 作增量补钉，所以要解压的第一块必须是 401000。   
  
现在我们必须在主窗口下以 401000 作开始行，并写入   
  
00401000    8105 0CF01200 00100000   ADD DWORD PTR DS:\[12F00C\],1000   
0040100A    8105 18F01200 00100000   ADD DWORD PTR DS:\[12F018\],1000   
00401014    8105 1CF01200 00100000   ADD DWORD PTR DS:\[12F01C\],1000   
这里补丁要和转存窗口中的地址一致。   
下一行就要这样写∶   
  
0040101E  CMP DWORD PTR DS:\[12F01C\],armadill.00426000   
//测试代码段结束了吗？   
  
要知道何时我们已经解压了全部块。    
则必须写入∶   
00401028 - 0F85 F6341F00 JNZ 00437775  //没有完成就继续   
如果比较结果不是 True，那这个循环返回到的位置紧挨着哪里是调用该循环。然后我们必须写入下一行写 NOP ，我们将放置一个 BP，是要在完成转储操作时来停止它。   
下面是打好补丁的全部代码：   
00401000  ADD DWORD PTR DS:\[12F00C\],1000   
0040100A  ADD DWORD PTR DS:\[12F018\],1000   
00401014  ADD DWORD PTR DS:\[12F01C\],1000   
0040101E  CMP DWORD PTR DS:\[12F01C\],armadill.00426000   
00401028  JNZ armadill.00437775   
0040102E  NOP   //这里下个中断，如果完成了就断在这里。   
0040102F  NOP   
  
好了，最关键的部分完成了，下面就可以脱壳了。   
  
SETUP 8 脱壳了   
检查一下前面的几步，保证他的正确后就运行这个程序了。F9 一下，哈哈中断在：   
0040102E  NOP   //这里下个中断，如果完成了就断在这里    
好像成功了耶！   
看看记录里有些什么，Alt+L 打开记录窗口：   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
77E65A12   COND: 未知的标示符   
0040102E   中断在 armadill.0040102E   
我不知道为什么成了未知的标示符，可能是我的条件记录没有设置好，不过那就是复制的代码块。有了这个信息就说明代码被完全的复制过来了。   
  
SETUP 9 由父进程解出子进程   
  
先用 OD 的附加功能查看子进程的句柄，打开附加窗口，看到没有变色的那个 armadillo3.exe 就是子进程，他的句柄是 OBDC （这个值每次加载后就会变）   
在程序中写入下面的代码：   
0040102E  PUSH 0BDC  //PUSH (子进程句柄)   
00401033  CALL kernel32.DebugActiveProcessStop   
00401038  NOP     // 这里下个中断   
运行程序。看看寄存器窗口，如果当 程序 在 EAX = 1 停下时，可以确定子进程与他的父进程已经分离，然后我们可以关闭 OllyDbg。而如果 EAX = 0，那是因为你写入有点不对劲（可能是句柄），则你必须要从头到尾核对那些行。你可以重新写入代码，再来一次。   
现在关闭 OllyDbg 然后再次打开它。不要装入任何东西。转到菜单 文件|附加 然后寻找子进程并且 附加上 它(我们已经杀死他的父进程 :X)。   
  
只要成功附加上，运行一下，程序就会在死循环运行，因此按 F12 来暂停程序，然后在 PUPE 中把 OEP 的代码还原成 55 8B ，再“Parchear”一下就会还原成原来的代码了。   
打开 LordPE 然后搜索 armadillo3.exe 这个进程。选择这进程以及选择〔active dump engine〕| 〔IntelliDump〕|〔select〕，然后点击 Dump full ，一旦保存完成，立即运行 PEditor (，然后再写入这个有效的入口点( ENTRY POINT)。计算一下是 4251D0-400000＝251D0 修复即可。   
  
第一部分已完，写了一个晚上。太冷了，第二部分关于如何查找这奇妙跳转来完成引入表、修复 IAT 和生成程序运行 还是留给明天吧。   
  
                                      fxyang\[OCN\]\[BCG\]\[FCG\]   
                                            2003.11.26 夜 

* * *

Armadillo v3.x 脱壳完全篇――下篇   
                    ――完成引入表、修复 IAT   
  这是一篇脱 Armadillo 加壳软件的个人的经历过程，本人特别声明：这个过程是参照了 Bighead\[DFCG\]\[YCG\]翻译的 Ricardo Narvaja 的“Getright 5 脱壳和重建”文章操作的，可以说是个照葫芦画瓢做的。感谢 Bighead\[DFCG\]\[YCG\]和相关的翻译者，感谢 Ricardo Narvaja 的文章。   
   这次拿来练习的软件是 FCG 的 peterdocter 老大提供的软件，下载地址：http://peter.88vip.com/armadillo3.rar   
工具：   
Ollydbg v1.09d 中文版   
PuPe 2002 版 感谢 yesky1 兄提供   
LordPE Deluxe-1.4 by yoda   
Import Reconstructor 16f   
   准备好笔和纸（不能少喔！），一杯茶和好是心情^\_^.让我们开始一个艰苦和漫长的脱壳过程吧（因为我菜！）。Armadillo 是当今猛壳之一啦。其 CopyMem-II+Debug-Blocker 的加壳方式是非常强劲的，好在有牛人在前面带路，虽艰难倒也不迷失方向。   
在上篇中完成了查找 OEP 和代码修复的工作，但是用 LordPE dump 下的程序不能运行。这是因为 Armadillo 的壳破坏了引入表，下面我们将要设法查找 magic 跳转，消除破坏引入表的代码。   
开始吧！～   
用 Ollydbg 加载 dump 下来的程序 dump.exe 用 F7 运行他，在下面的代码处就会出错了，因为程序需要的函数地址指向了一个不正确的地方：   
0040128C  - FF25 D0804200   JMP NEAR DWORD PTR DS:\[4280D0\]     
DS:\[4280D0\] 的内存地址指向了 00BF919B,这个地址在程序的代码外，所以运行到这里后就会出现异常问题。原来的程序在这里肯定指向一个有效的地址才能运行下去。现在来到 OD 的转存窗口中。用 Ctrl+G 命令输入 4280D0 到窗口中来到：   
004280D0  9B 91 BF 00 A5 5B F3 77  洃?體  //看这里   
004280D8  9F 9C BF 00 9C 76 BF 00  煖?渧?   
004280E0  CC 76 BF 00 B2 E3 E2 77  蘶?层鈝   
004280E8  5F 8E F5 77 66 74 BF 00  \_ 庻 wft?   
004280F0  8B 49 E3 77 D7 E7 E1 77  婭鉾诅醱   
这是程序的引入表的一部分，来看看他开始于什么地方，向上找到这里：   
004280A0  B7 8B F3 77 C6 20 F3 77  穻體?體   
004280A8  00 20 F3 77 03 2D E1 77  . 體 -醱   
004280B0  66 63 E1 77 49 25 E1 77  fc 醱 I%醱   
因为再向上没有了这个结构的地方，所以可以认定引入表的开始地址是 004280A0，再向下看看什么地方是表的结束地址：   
004284D0  AF 9B BC 70 00 00 00 00  瘺紁....   
004284D8  6B 65 72 6E 65 6C 33 32  kernel32   
004284E0  2E 64 6C 6C 00 00 00 00  .dll....   
004284E8  00 00 00 00 00 00 00 00  ........   
后面再也没有了。所以可以认定表的结束地址是 004284E4 ，看看表的长度＝004284E4-004280A0＝444 用纸记下这些信息。以后用 ImportREC 修复 IAT 时需要手工填入这些信息。   
Ricardo Narvaja 的原文有这样的一段话：   
《这里说明一点：我们已经知道程序的父进程和他的子进程是相同文件，但要用不同的句柄装入两次，所以他们变成两个不同的进程。由其中一个进程掌管着父进程的 OEP 从这里开始运行。   
 我们知道当它被转储时，另一进程是掌管着子进程的 OEP，但我也要告诉你，并不是所有子进程都以相同父进程的入口点来开始运行的。顺便一提：我是设法告诉你，它不是从父进程那复制 IAT 到它的子进程，却是从子进程那解出自己的 IAT。这应该是一大麻烦，因为我们不能在他从父进程脱离前用 OllyDbg 进入子进程。》   
现在的问题是怎么用 OD 进入子进程中，让我跟着 Ricardo Narvaja 向前走。   
现在用 OD 加载没有脱壳的程序 armadillo3.exe 按老规矩记住要将 IsDebuggerPresent 的值设为零，清除所有以前的 BPX、BP WaitForDebugEvent 然后单击运行。等 armadillo 的注册提示框出来后，打开 PUPE 选择进程中 2 个 armadillo3.exe 中的上面一个（因为他是子进程，是我们需要的）。右击进入 parcheando 窗口选择 4 字节，在地址里填 4280D0 （这是第一个不正确的地址）Buscar 一下信息窗口中就出现了 4 个字节的值 00 00 00 00  然后不断的在 OD 中运行、在 PUPE 中检查字节窗口中的值，直到字节窗口中的值变成 9B 91 BF 00 ，这个值就是程序写入到内存地址 004280D0 处的。那么反过来想想，如果能在写入这个值前进入到子进程中并跟踪什么地方的代码写入了这个值就能跟踪到那个 magic 的跳转。那么出现 9B 91 BF 00 这个值前出现过 CA 85 02 00 这个值。重新做上面的一切直到字节窗口中出现 CA 85 02 00 这个值时停下。现在是进入子进程的时候了：   
在 OD 中打开模块窗口选择一个 API 生成死循环，然后进入到子进程中。我模仿 Ricardo Narvaja 选择了 KERNEL32.dll 中的 GetProcAddress 这个 API。在名称窗口中查找这个函数的地址是 77E12DFB（这个地址可能与你的不同）。又是用到 PUPE 这个工具的时候了，在字节数中选择 2 个字节。地址中填 77E12DFB 再"Buscar"一下，就会在字节信息窗口中看到 55 8B 这个值，修改为 EB FE 并按 "Parchear"，看看字节信息窗口中的值变成了 EB FE ，子进程就会在这个 API 中停下了。或者说是休眠在这个 API 中直到你来唤醒他。   
会到 OD 中选择一段代码，比如：   
0042F000      55            PUSH EBP   
0042F001      8BEC          MOV EBP,ESP   
0042F003      83EC 0C       SUB ESP,0C   
0042F006      8B45 10       MOV EAX,DWORD PTR SS:\[EBP+10\]   
0042F009      50            PUSH EAX   
0042F00A      E8 18CE0000   CALL armadill.0043BE27   
用鼠标在 0042F000 行右击选择〔新建起源〕（呵呵，程序会到这里运行我们写的代码），打开附加窗口查看那个没有变色的主程序的句柄是 0BAC(这个值每次加载会不同，注意了)这个就是子进程的句柄。回到代码中把上面的代码修改为：   
0042F000      68 AC0B0000   PUSH 0BAC //子进程句柄   
0042F005      E8 AD16A377   CALL kernel32.DebugActiveProcessStop   
0042F00A      90            NOP   //在这里下中断   
在 0042F00A  下中断，然后运行一下停在 0042F00A 处。看看寄存器窗口的 EAX 值，如果这个值＝01 就表示子进程和父进程分离了。如果＝00 那么可能是子进程的句柄填错了，你可以在下面重新写入代码再次运行，直到 EAX＝01 时就可以关闭 OD 了（杀掉父进程！）然后就可以进入子进程了。   
重新运行 Ollydbg（不要加载程序）附加上进程中的那个唯一的 armadillo3.exe （附加上了吗？成功了吗？反正我是成功了）F9 运行一下然后 F12 暂停程序就会来到休眠的地方：   
77E12DFB >- EB FE           JMP SHORT kernel32.GetProcAddress //看在循环呢   
77E12DFD    EC              IN AL,DX   
77E12DFE    51              PUSH ECX   
再次在 PUPE 中恢复原来的代码 55 8B 并按 "Parchear" 再看看程序中的代码又还原成：   
77E12DFB >  55              PUSH EBP   
77E12DFC    8BEC            MOV EBP,ESP   
77E12DFE    51              PUSH ECX   
这个程序重新被我唤醒了。   
先来到出错的地址看看，在转存窗口中 G 004280D0 ：   
004280D0  CA 85 02 00 DE 85 02 00  蕝 .迏 .   
004280D8  EE 85 02 00 00 86 02 00  顓 ..?.   
004280E0  0C 86 02 00 1E 86 02 00  .?. ?.   
004280E8  2E 86 02 00 3A 86 02 00  .?.:?.   
看看引入表还没有写入呢，说明我们停在解密引入表的代码前面，上面说过这个引入表是子进程自己解密的，那么只要找到解密的地方。然后把破坏的代码修改就可以得到完整的引入表了。   
好了，第一个目的达到了－进入到子进程中并且停在解密引入表的前面。成功了一半了！下面要做的就是找那个 magic 了。   
Ctrl+F9 再 F7 一次就来到：   
00BF545E  PUSH DWORD PTR SS:\[EBP+C\]   
00BF5461  PUSH ECX   
00BF5462  CALL NEAR DWORD PTR DS:\[C130C4\]          ; kernel32.GetProcAddress   
00BF5468  JMP SHORT 00BF54BE  //返回的地方。   
我们知道程序的第一个出错的地方就是：004280D0 地址指向的代码 00BF919B 现在在转存窗口中的 004280D0 开始的 4 个字节上下内存写入中断－跟踪是什么地方写入了 00BF919B 这个值，运行被 OD 中断在：   
00C0942F  MOV EAX,DWORD PTR SS:\[EBP-4A4\]   
00C09435  MOV ECX,DWORD PTR SS:\[EBP-6B8\]   
00C0943B  MOV DWORD PTR DS:\[EAX\],ECX                ; ECX=00BF919B   
00C0943D  MOV EAX,DWORD PTR SS:\[EBP-4A4\]   
00C09443  ADD EAX,4   
00C09446  MOV DWORD PTR SS:\[EBP-4A4\],EAX   
00C0944C  JMP 00C092C9   
可以看出 00BF919B 这个值是 EBP-6B8 这个地址里的。   
在转存窗口中 G EBP-6B8 来到这里：   
0012ED60  9B 91 BF 00 B3 73 F3 77  ....硈體   
0012ED68  78 01 14 00 00 00 00 00  x  .....   
在上面的 9B 91 BF 00 这 4 个字节上下内存访问断点然后 Run 一下就来到：   
00C093D1    8985 F4FDFFF>MOV DWORD PTR SS:\[EBP-20C\],EAX   
00C093D7    FFB5 40F9FFF>PUSH DWORD PTR SS:\[EBP-6C0\]   
00C093DD    FFB5 70FBFFF>PUSH DWORD PTR SS:\[EBP-490\]   
00C093E3    E8 F8C0FEFF  CALL 00BF54E0   
00C093E8    8985 48F9FFF>MOV DWORD PTR SS:\[EBP-6B8\],EAX    //停在这里               ; ntdll.RtlGetLastWin32Error   
00C093EE    83BD 48F9FFF>CMP DWORD PTR SS:\[EBP-6B8\],0   
00C093F5    75 38        JNZ SHORT 00C0942F   
看看停下的地方，发现是 EAX 赋了这个内存地址的值。而这个 EAX 是上面的 CALL 00BF54E0 函数计算的值，到这个函数中看看：   
00BF54E0  PUSH EBP         //在这里下中断   
00BF54E1  MOV EBP,ESP   
00BF54E3  PUSH EBX   
00BF54E4  PUSH ESI   
00BF54E5  PUSH EDI   
00BF54E6  XOR EDI,EDI   
00BF54E8  XOR EBX,EBX   
00BF54EA  TEST WORD PTR SS:\[EBP+E\],0FFFF   
00BF54F0  JNZ SHORT 00BF54F5          // 第一个跳   
00BF54F2  MOV EBX,DWORD PTR SS:\[EBP+C\]   
00BF54F5  PUSH EDI   
00BF54F6  CALL NEAR DWORD PTR DS:\[C130A4\]          ; kernel32.GetModuleHandleA   
00BF54FC  CMP DWORD PTR SS:\[EBP+8\],EAX   
00BF54FF  JNZ SHORT 00BF5508    //第二个跳   
00BF5501  MOV ESI,0C153C0   
00BF5506  JMP SHORT 00BF5568   
00BF5508  CMP DWORD PTR DS:\[C15998\],EDI   
00BF550E  MOV ECX,0C15998   
00BF5513  JE SHORT 00BF5551      // 第三个跳 改为 JMP（magic 跳转）   
00BF5515  MOV ESI,DWORD PTR DS:\[C1B1B8\]   
00BF551B  MOV EAX,DWORD PTR DS:\[C1EECC\]   
00BF5520  TEST BYTE PTR DS:\[ECX+8\],1   
00BF5524  JE SHORT 00BF5534       //第 4 个跳   
00BF5526  MOV EDX,DWORD PTR DS:\[EAX+70\]   
00BF5529  XOR EDX,DWORD PTR DS:\[EAX+60\]   
00BF552C  XOR EDX,DWORD PTR DS:\[EAX+3C\]   
00BF552F  TEST DL,80   
00BF5532  JNZ SHORT 00BF5547        //第 5 个跳   
00BF5534  MOV EDX,DWORD PTR DS:\[EAX+70\]   
00BF5537  XOR EDX,DWORD PTR DS:\[EAX+64\]   
00BF553A  XOR EDX,DWORD PTR DS:\[EAX+58\]   
00BF553D  XOR EDX,DWORD PTR DS:\[EAX+20\]   
00BF5540  XOR EDX,DWORD PTR DS:\[ESI\]   
00BF5542  CMP DWORD PTR SS:\[EBP+8\],EDX   
00BF5545  JE SHORT 00BF5565         //第 6 个跳   
00BF5547  ADD ECX,0C   
00BF554A  ADD ESI,4   
00BF554D  CMP DWORD PTR DS:\[ECX\],EDI   
00BF554F  JNZ SHORT 00BF5520       //第 7 个跳   
00BF5551  PUSH DWORD PTR SS:\[EBP+C\]   
00BF5554  PUSH DWORD PTR SS:\[EBP+8\]   
00BF5557  CALL 00BF5397   
00BF555C  POP ECX   
00BF555D  POP ECX   
00BF555E  POP EDI   
00BF555F  POP ESI   
00BF5560  POP EBX   
00BF5561  POP EBP   
00BF5562  RETN 8   
  
上面有 7 个跳转，跟踪看看是那个跳转跳到破坏引入表的地方。跟踪发现第 3 个跳转如果不跳就会到破坏引入表的代码处。现在只要把这个跳转改成 JMP 就可以避开破坏引入表的代码。这个就是 magic 跳转了。好像在 Armadillos 是相同的。   
好了找到了这个关键的 magic 跳转就能修复全部的引入表了。   
重新做上面的工作，用 magic 的地址做休眠的地址，附加上程序后改 magic 的跳转然后运行就会停在子进程的 OEP 处，用 Import REConstructor 修复 dump 的程序，注意要点：   
在 Import REConstructor 中填：   
OEP＝入口地址－400000   
RVA＝引入表的开始地址（上面说过了）－400000   
SIZE＝引入表的结束地址－开始地址   
比如我的信息：   
OEP＝000251D0   
RVA＝000280A0   
SIZE＝00000444   
注意！注意！注意！――不要按 IAT Autosearch 直接按 Get Import 获得引入表，如果有错误的就直接 Cut 他 Fix dump 修复 dump 下的程序，运行一下是不是还正常了。   
由于时间关系，我写得很仓促，可能存在许多的错误和表述不正确的地方。请大虾们指正了！   
虽是依葫芦画的瓢，且画的不好。但也是我的心得，我想学习其实也是不断的在画别人已画好的瓢，如果画的好了就是自己的了。   
文章虽粗糙，转载请保证他的完整性。   
感谢二点兄弟收集的资料！   
  
                               fxyang                                     
                             2003.11.27 晚 

  
 4dd466d0e1e0cc123afbe504da2462ca