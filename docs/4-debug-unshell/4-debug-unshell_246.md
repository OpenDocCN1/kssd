

# Konvertor V3.30 

> 原文：[`www.pediy.com/kssd/pediy06/pediy6954.htm`](https://www.pediy.com/kssd/pediy06/pediy6954.htm)

FTBirthday 的 Armadillo 全程教程--第一章．标准加壳方式  
  
第一章．标准加壳方式  
  
   
  
\[实例解析三\] Armadillo v3.01 标准加壳方式的脱壳--Konvertor V3.30   
  
   
  
※软件简介※: Konvertor V3.30 是一个图片、音乐、文字及影音格式文件的转换程序，它总共可以转换 135 种文件格式，包含 25 种音乐格式、4 种影音格式、105 种图片格式及 1 种文字格式。  
  
软件大小：6439 KB  
  
软件语言：英文  
  
软件类别：国外软件 / 共享版 / 图像转换  
  
运行环境：Win98/NT/2000/XP/  
  
软件添加：乱马  
  
下载次数：1356  
  
开 发 商：http://www.logipole.com/download\_konv\_us.htm  
  
下载页面:http://www.downxp.com/soft/2638.htm  
  
   
※软件限制※:The shareware version of this software can be freely used for 30 days.  
  
You will have to register next.  
  
  
   
※破解工具※:Ollydbg1.09、FI 3.01、DriverSuite2.7 中的 softice4.2.7   
  
   
※※※※※※※※※※※※※※※※开始吧※※※※※※※※※※※※※※※※※※  
  
FI 3.01 侦测知 Armadillo v3.01 加壳  
  
  
   
记得 jwh51 在 Armadillo 标准加壳的程序的脱壳和引入表修复方案中说:"用过 ARMADILLO 的人都知道,用它加壳有两种方式,一是使用 COPYMEMII,一是标准加壳.用 COPYMEMII 的一般可用 DILLODUMP 脱(当然也有时脱不了),而用标准加壳用 DILLODUMP 是脱不了的."  
  
   
  
Konvertor V3.30 采用了 Armadillo 标准加壳。有 30 天试用期。并且在程序退出时出现提示框。所以,如果脱了壳,上述现象都将没有了.现在开始脱壳历程.!!   
  
   
**如何 DUMP(转储)Konvertor v3.30**  
  
   
第一步，扫除障碍  
  
   
  
\[障碍一\]挫败 anti-debug 代码,   
  
用 OLLYDBG 载入程序.在调试选项中忽略所有异常,这样可以少按 SHIFT+F9.  
  
\[方法一\]  
  
用 IsDebug 1.4 插件去掉 Ollydbg 的调试器标志。  
  
\[方法二\] 重新设置检测 IsDebuggerPresent API 的那个字节!  
  
Bp IsDebuggerPresent  
  
![](img/72c1c4e49812f39d63e669751995a112.jpg)  
  
现在按 F9 运行程序,当遇到异常,一直按 SHIFT+F7 或者 F8,直到中断于  
  
![](img/cbcf7e070343abd0de48688a5f9d5214.jpg)  
  
在这张图中我们可以看到 OllyDBG 中断在 API IsDebuggerPresent(红色), 然后按 F7 步进到第三行(白色) 77E52749 0FB640 02 MOVZX EAX,BYTE PTR DS:\[EAX+2\]  
  
一旦我到达那儿,看一下 disassembly,在那儿我们可以看到的内容如下图:  
  
![](img/fe694e29a25b792e53f22104bc83af35.jpg)  
  
这儿我们可以看到在我的机器中偏移是 7ffdf002,包含了值 01.这个地址在其它机器中可能会不同. 值 01 意味着调试器被检测到了,所以写下这个地址,在任何时候我们在 ollydbg 中运行程序,都必须改变它为 00,从而避免被 packer 检测到.记住改变此值为 00 在任何时候在 ollydbg 中运行或重运行程序. 记下这个地址，以后经常会用到的。  
  
我们将以重启 ollydbg (ctrl+F2)来结束第一步,到转储窗口(DUMP)单击右键选择"前往 表达" 7ffdf002 或者其它任何先前你记下的地址,然后改变它的值从 01 变为 00.  
  
\[障碍二\]击败检测系统时间的代码  
  
如果系统时间在安装 SoundEdit Pro 后修改过,程序会检测到,并警告修改系统时间而退出程序. 在我安装完 Konvertor V3.30 后,我改了一下系统时间,在启动时 Konvertor V3.30 就提示发现时间不对,需要从作者那儿得到帮助.  
  
  
如下方法可挫败它的这个保护。打开 softice4.2.7,ctrl+D 调出 softice  
  
bpx getlocaltime  
  
g 回来点击 ok 按键,中断两次后,先别忙 g 退出.  
  
经过测试内存地址 00CB4677 处的 test al,al 是关键,所以 bpx 00CB4677 再 g.  
  
((((   
  
00CB4672   E8 08060000      CALL 00CB4C7F   
  
00CB4677   84C0             TEST AL,AL      ====>走到这里将 AL 改为 00000000 即可    
  
00CB4679   74 1C            JE SHORT 00CB4697   
  
00CB467B   E8 56050000      CALL 00CB4BD6   ====>这个 call 警告修改系统时间  
  
))))  
  
中断后,R EAX 00000000 修改寄存器 EAX 的值为 0,然后 BC \*清除所有断点.  
  
先别忙退出,继续 BPX VirtualProtect.至此击败它了.  
  
   
  
第二步: BP WaitForDebugEvent 寻找 API report  
  
这一步并非必要，但却是一个好的选择，当它停止时在 dump 区看到 API report。  
  
首先我们需要在命令行输入 BP WaitForDebugEvent，现在运行它，当它停下时我们可以在 Stack 窗口看到关于这个 API 的所有参数信息。   
  
   
  
0012DAC4   00984B44  /CALL to WaitForDebugEvent  
  
0012DAC8   0012EB74  |pDebugEvent = 0012EB74  
  
0012DACC   000003E8  \\Timeout = 1000. ms  
  
   
  
看一下第二行。那儿我们可以看到它将在哪儿显示 report，所以到 dump 窗口， 右键选择 "前往 表达"= 12EB74。注意这个值在一些机器上可能会不同，所以前往你的机器显示的值。  
  
   
  
0012EB74  9C EB 12 00 C0 EB 12 00  滊.离.12  
  
0012EB7C  00 00 00 00 90 FE 12 00  ....慆.0  
  
0012EB84  9C 00 00 00 89 46 99 00  ?..塅? 00  
  
0012EB8C  00 00 00 00 D8 EC 12 00  ....仂.0  
  
0012EB94  F7 C6 E5 77 D0 EC 12 00  髌鍂徐. E5  
  
0012EB9C  00 00 00 00 00 00 00 00  ........  
  
   
  
我们在上面的图中看到的是我们将要称之为"API report"的东西。父进程通过那里得知子进程发生了什么。现在我们可以很容易地通过在命令行输入 BC WaitForDebugEvent 来删除这个断点。所有的操作都是为了在 dump 窗口看到 report，在这个 report 中我们可以看到入口点 OEP。   
  
   
  
**第三步: 寻找子进程入口点**  
  
\[方法一\]，BP WRITEPROCESSMEMORY（使用 OLLYDBG）  
  
Bp WriteProcessMemory 停下时在 STACK 窗口看到：  
  
0012D964   0098867F  /CALL to WriteProcessMemory from Konverto.00988679  
  
0012D968   00000050  |hProcess = 00000050  
  
0012D96C   004BA000  |Address = 4BA000  
  
0012D970   003B51D0  |Buffer = 003B51D0  
  
0012D974   00001000  |BytesToWrite = 1000 (4096.)  
  
0012D978   0012DA80  \\pBytesWritten = 0012DA80  
  
这些信息被保存在父进程的一个缓冲区从 3B51D0 开始，然后它会复制给子进程一个从 4BA000 开始的 1000 bytes 的块，所以如果我们到了这一步，我们可以很容易理解子进程的第一个 section 是完全空的，所以当它试图执行到 OEP 时，它报告一个错误因为那儿没有任何东西，所以 父进程得到了通知正如我们可以在父进程的 report 中看到的，所以它停止了运行，复制必要的数据块，然后继续运行直到下一个错误。被复制的数据块的大小是 1000 bytes，所以当程序试图执行任何超过这个大小的块外的指令时， 另一个错误将会发生，然后这个错误会被通知给父进程， 父进程会复制另外 1000 bytes 的块然后继续。   
  
顺便说一下，我们可以假设第一个错误发生是因为子进程 call 了它的入口点 OEP，它的值应该就在 API report 上。显然 OEP 的值必定在第一个块，父进程复制过去来解决子进程报告的错误。  
  
这个块从 4BA000 开始直到 4BAFFF。OEP 值必定在这些值中。 让我们看一下 API report。看 dump 窗口我们先前已经知道了指向 report 的指针 12EB74。  
  
0012EB74  01 00 00 00 98 03 00 00  ...?..0  
  
0012EB7C  D4 05 00 00 01 00 00 80  ?....€0  
  
0012EB84  00 00 00 00 00 00 00 00  ........  
  
0012EB8C  97 A6 4B 00 02 00 00 00  棪 K.\[1\]...B  
  
0012EB94  00 00 00 00 97 A6 4B 00  ....棪 K.0  
  
0012EB9C  97 A6 4B 00 00 00 00 00  棪 K.....B  
  
0012EBA4  01 85 54 E1 10 01 00 00  匱?..54  
  
太好了!在那儿我们至少看到了三次这个值: 4BA697。数学课告诉我 4BA697 比 4BA000 大，比 4BAFFF 小 (那些是父进程试图拷贝给子进程的那个块的起止值)所以我可以确信 4BA697 就是子进程的 OEP。  
  
 \[方法二\]，bpx SetProcessWorkingSetSize（使用 DriverSuite2.7 中的 softice4.2.7）  
  
标准的流程，下 bpx SetProcessWorkingSetSize 去找 OEP 一旦 softice 中断了，检查 softice 是否在 Get Right 的领空，如果不在则按 F5 直到 softice 在 Get Right 的领空中断。然后按 F12 辅之以 F10 跟踪过去一些指令，直到你看到一个"call edi"指令，停在那儿，然后写下 edi 的值。那就是 OEP。  
  
按三下 F12 返回  
  
……  
  
001B:00981739  PUSH      EAX  
  
001B:0098173A  CALL      \[009B1EA8\]     ；这个 CALL 进入  
  
001B:00981740  MOV       EAX,\[EBP-1C\]  
  
……  
  
……  
  
001B:0109EEE2  XOR       ECX,\[EAX+6C\]  
  
001B:0109EEE5  XOR       ECX,\[EAX\]  
  
001B:0109EEE7  SUB       EDI,ECX  
  
001B:0109EEE9  CALL      EDI             ；到了这里，EDI 的值就是 OEP=004BA697  
  
001B:0109EEEB  MOV       EBX,EAX  
  
001B:0109EEED  POP       EDI  
  
001B:0109EEEE  MOV       EAX,EBX  
  
001B:0109EEF0  POP       ESI  
  
001B:0109EEF1  POP       EBX  
  
001B:0109EEF2  RET  
  
注意：copy-mem II 加壳方式找 OEP 有三种方法，标准加壳只有两种，标准加壳不能用 CreateProcessA 方法。因为只有 copy-mem II 加壳方式的父进程通过 CreateProcessA 产生另外一个进程即子进程。  
  
**第四步,DUMP 出程序**  
  
标准的流程如下, 下 bpx VirtualProtect 去找 OEP 一旦 softice 中断了,检查 softice 是否在  
  
Konvertor 的领空,如果不在则按 F5 直到 softice 在 Konvertor 的领空中断.然后按 F12 辅之以 F10 跟踪过去一些指令,直到你看到一个"call EDI"指令,停在那儿,然后写下 EDI 的值.那就是 OEP.  
  
具体如下:  
  
运行 softice4.2.7, ctrl+D 调出 softice, BPX VirtualProtect.到中断 124 次时程序运行了,重新来一遍,当在 123 次出现中断在 VirtualProtect 时停下(好累啊，也好晕啊),BC \*清除所有断点,然后按 F10 单步返回到主程序. 再耐心地用 F10 单步走,因为有一堆的 JMP,JNZ,JO 等,用不了多久的就可来到这:  
  
001B:009815EA  C705B01E9B0006000000MOV       DWORD PTR \[009B1EB0\],00000006  
  
001B:009815F4  FF1568C19A00        CALL      \[009AC168\]    ,这个 CALL 用 F8 跟进  
  
在 009815F4 这个 CALL 用 F8 跟进,然后继续 F10,几次后就来到这:  
  
001B:0109EEDF  8B487C              MOV       ECX,\[EAX+7C\]  
  
001B:0109EEE2  33486C              XOR       ECX,\[EAX+6C\]  
  
001B:0109EEE5  3308                XOR       ECX,\[EAX\]  
  
001B:0109EEE7  2BF9                SUB       EDI,ECX  
  
001B:0109EEE9  FFD7                CALL      EDI  =====>EDI=004BA697  
  
dump 取内存中己脱壳的文件  
  
0109EEE9  FFD7                CALL      EDI  
  
现在这一行,键入以下命令:   
  
a eip (然后按回车)  
  
jmp eip (然后按回车)   
  
按下 F5   
  
这样将改变 001B:0109EEE9 行的代码. 你会注意到在键入"jmp eip"并按下回车后, 0109EEE9 的指令现在是一个 jmp.这将有效地使程序"暂停"（有点类似 TRW2000 的 suspend 命令）. 按下 F5 使你回到 window XP。运行 LordPE,在进程列表中选择" Konvertor.exe"然后点击鼠标  
  
右键选中"Dump (Full)",给脱壳的程序起名 dump 存盘.  
  
知道了 OEP 那就 bpm 004BA697   
  
001B:0098737D  6A00                PUSH      00  
  
001B:0098737F  8B4D0C              MOV       ECX,\[EBP+0C\]  
  
001B:00987382  51                  PUSH      ECX  
  
001B:00987383  8B5508              MOV       EDX,\[EBP+08\]  
  
001B:00987386  52                  PUSH      EDX  
  
001B:00987387  E847030000          CALL      009876D3  
  
001B:0098738C  83C40C              ADD       ESP,0C  
  
001B:0098738F  25FF000000          AND       EAX,000000FF  
  
001B:00987394  85C0                TEST      EAX,EAX  
  
001B:00987396  7507                JNZ       0098739F  
  
**第五步, 修复输入表**  
  
一般步骤如下：  
  
（1）找被脱壳的入口点（OEP）；  
  
（2）完全 Dump 目标文件；  
  
（3）运行 Import REConstructor 和需要脱壳的应用程序；  
  
（4）在 Import REConstructor 下拉列表框中选择应用程序进程；   
  
（5）在左下角填上应用程序的真正入口点偏移（OEP）；  
  
（6）按"IAT AutoSearch"按钮，让其自动检测 IAT 位置，出现"Found address which may be in the Original IAT.Try 'Get Import'"对话框，这表示输入的 OEP 发挥作用了。  
  
（7）按"Get Import"按钮，让其分析 IAT 结构得到基本信息；  
  
（8）如发现某个 DLL 显示"valid :NO" ，按"Show Invalids"按钮将分析所有的无效信息，在 Imported Function Found 栏中点击鼠标右键，选择"Trace Level1 (Disasm)"，再按"Show Invalids"按钮。如果成功，可以看到所有的 DLL 都为"valid:YES"字样；   
  
（9）再次刷新"Show Invalids"按钮查看结果，如仍有无效的地址，继续手动用右键的 Level 2 或 3 修复；  
  
（10）如还是出错，可以利用"Invalidate function(s)"、"Delete thunk(s)"、编辑 Import 表（双击函数）等功能手动修复。  
  
（11）开始修复已脱壳的程序。选择 Add new section (缺省是选上的) 来为 Dump 出来的文件加一个 Section(虽然文件比较大，但避免了许多不必要的麻烦) 。  
  
（12）按"Fix Dump"按钮，并选择刚在（2）步 Dump 出来的文件，在此不必要备份。如修复的文件名是"Dump.exe"，它将创建一个"Dump\_.exe"，此外 OEP 也被修正。  
  
（13）生成的文件可以跨平台运行。  
  
在 Import REConstructor v1.4.2+ 的 Attach to an Active Process 窗口中选取加壳程序的进程,有 2 个进程选上面的一个,然后在下方的 oep 处填入 000BA697,点 IAI AutoSearch,再点 Get Imports,然后点 Auto Trace,然后点看还有多少地址是 NO,剩下的事就是手动找这些函数了.    
  
hying 大哥说过  CreateEventW   GetStringTypeA  如果发现这两个函数连续在一起的话，上面一个该改为 GetStringTypeW  手工修复修入表后，程序应能运行了，好像也就不用注册了，因为注册部分在壳中，壳脱了注册也就不用了.这程序的输入表我是手工找到一部分，然后参照 hying 大哥给我的 Konvertor 3.12A 的输入表修复的.先用 importREC 试试,输入我们找到的正确的 OEP: 004BA697,注意输入 004BA697-00400000=000BA697 有很多指针无效。  
  
这里现不要 bpx GetModuleHandleA，否则会被发现的，我想可能在 unpack 前程序会检测，所以要是能在程序开始 unpack 后再设断多好，怎么做呢，这样，先 bpx SetProcessWorkingSetSize  
  
断下来之后再 bpx GetModuleHandleA 按 F12 没几次就到了如下  
  
001B:01085B86  FF15AC800A01        CALL      \[KERNEL32!GetModuleHandleA\]  
  
001B:01085B8C  394508              CMP       \[EBP+08\],EAX  
  
001B:01085B8F  7507                JNZ       01085B98  
  
001B:01085B91  B968B40A01          MOV       ECX,010AB468  
  
001B:01085B96  EB52                JMP       01085BEA  
  
001B:01085B98  393D68BA0A01        CMP       \[010ABA68\],EDI  
  
001B:01085B9E  B968BA0A01          MOV       ECX,010ABA68  
  
001B:01085BA3  0F8491000000        JZ        01085C3A  ======>magic jump!  
  
001B:01085BA9  8B35581C0B01        MOV       ESI,\[010B1C58\]  
  
001B:01085BAF  A1085D0B01          MOV       EAX,\[010B5D08\]  
  
001B:01085BB4  F6410801            TEST      BYTE PTR \[ECX+08\],01  
  
001B:01085BB8  740E                JZ        01085BC8  
  
001B:01085BBA  8B507C              MOV       EDX,\[EAX+7C\]  
  
001B:01085BBD  335070              XOR       EDX,\[EAX+70\]  
  
001B:01085BC0  33501C              XOR       EDX,\[EAX+1C\]  
  
001B:01085BC3  F6C280              TEST      DL,80  
  
001B:01085BC6  7513                JNZ       01085BDB  
  
001B:01085BC8  8B507C              MOV       EDX,\[EAX+7C\]  
  
001B:01085BCB  335078              XOR       EDX,\[EAX+78\]  
  
001B:01085BCE  335068              XOR       EDX,\[EAX+68\]  
  
001B:01085BD1  335004              XOR       EDX,\[EAX+04\]  
  
001B:01085BD4  3316                XOR       EDX,\[ESI\]  
  
001B:01085BD6  395508              CMP       \[EBP+08\],EDX  
  
001B:01085BD9  740C                JZ        01085BE7  
  
001B:01085BDB  83C10C              ADD       ECX,0C  
  
001B:01085BDE  83C604              ADD       ESI,04  
  
001B:01085BE1  3939                CMP       \[ECX\],EDI  
  
001B:01085BE3  75CF                JNZ       01085BB4  
  
这里面有几个跳转,其中最后一个 01085BB4 是往上跳的.我们看一个就可发现:     
  
001B:01085BA3  0F8491000000  JZ 01085C3A,这个跳正好可跳出这个循环.只要改这个跳,就不会破坏引入表了. 再来, 注意刚回到那段代码时，就下 a 01085BA3 将 JZ 01085C3A 改成 JMP  01085C3A,清除所有断点,按 F5 运行,程序运行后打开 importREC,选中程序(注意选择上面那个进程),入口填入 OEP: 004BA697 ,-->RVA005AC000 SIZE00000214-->获得输入信息"。  
  
如果还有无效的大可放心的 cut,因为它们本来就不是有用指针了.然后修复脱壳后的文件.OK,大功告成.脱壳后的程序,运行正常,圆满成功.  
 4dd466d0e1e0cc123afbe504da2462ca