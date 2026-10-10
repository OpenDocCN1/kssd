

# OEP 定位

> 原文：[`www.pediy.com/kssd/pediy02/forum201.htm`](https://www.pediy.com/kssd/pediy02/forum201.htm)

 找 Entry Point 的又一方法 -----针对 Asprotect v1.0 加壳的程序   
  
【前言】   
这几天我在捣鼓对 Asprotect 加壳程序的脱壳，结果战况惨烈。花了 N 个钟头，所获甚少。急疯的时候，就差向 Asprotect 作者索要程序源代码了(当然也想过绑架他，可路费俺花不起)。从跟踪几个加壳程序的情况来看，发现它们的脱壳难度不同，虽然在用 FileInfo 检测的时候都报告 ASProtect + ASPack v1.084 加壳。（看来这种"侦察兵"工具也该加紧升级了！！！）   
由于 Asprotect 加壳的程序大多都有防跟踪的代码，已知的有检测 SICE,SIWVID,SIWDEBUG,BCHK。（想了解手动跳过防跟踪的方法请看国外大师的文章）这里我可不想每次都把时间花在这上面。我用 ICEPatch 修补了 SoftICE。   
  
以下以 Asprotect 加壳过的 ARPR v1.0 为例吧。（注：从我跟踪的 Azpr v3.10、AATool v4.0 的情况来看，都与此类似。你也可以用 Asprotect v1.0 加壳一个程序来试试）   
  
样例文件：    arpr.exe    （ARPR v1.0 的主程序文件）   
加壳方式：    ASProtect + ASPack v1.084   
检测工具：    FileInfo   
调试工具：    SoftICE v4.05,ICEPatch v2.0   
目标：        找到 Entry Point   
作者：        ljttt   
日期：        2000-08-03   
  
一、粗步确定 Entry Point 的所在的 Call 调用的方法   
首先，要找个比较好的入口进入 arpr.exe。由于修补了 SoftICE，所以就以这种防跟踪的部分为我们设断点进入的地方。   
1、设断点   
bpint 3   
  
（注：对于只检测 NTICE 等防跟踪的加壳程序，可以设断点 bpx CreateFileA，以后类似。不同的地方可能是以后要多按几个 F12 吧）   
  
这时，一般你可以看到类似如下的屏幕：   
015F:004BF23C  CC                  INT      3            <--你中断在此   
015F:004BF23D  EB02                JMP      004BF241   
015F:004BF23F  E9025DEB02          JMP      03374F46   
015F:004BF244  E9FF33C05A          JMP      5B0C2648   
015F:004BF249  59                  POP      ECX   
015F:004BF24A  59                  POP      ECX   
015F:004BF24B  648910              MOV      FS:\[EAX\],EDX   
015F:004BF24E  EB0A                JMP      004BF25A   
015F:004BF250  E91305FFFF          JMP      004AF768        <--注意这个跳转语句，int 3 过后就要到这里继续了。   
015F:004BF255  E8AA07FFFF          CALL      004AFA04   
015F:004BF25A  8A1518514C00        MOV      DL,\[004C5118\]   
015F:004BF260  8B45F8              MOV      EAX,\[EBP\-08\]   
  
2、把光标移到第五个 Jmp 语句以后，按 F7 键。好，我们过了第一道陷井。然后按三次 F12。然后开始按 F10 跟踪，   
直到出现如下类似屏幕   
  
015F:004BF842  EB13                JMP      004BF857        <--跳转了   
015F:004BF844  E91FFFFEFF          JMP      004AF768   
015F:004BF849  66B80E00            MOV      AX,000E   
015F:004BF84D  E8D2EDFFFF          CALL      004BE624   
015F:004BF852  E8AD01FFFF          CALL      004AFA04   
015F:004BF857  8B4508              MOV      EAX,\[EBP+08\]    <--跳到此处   
015F:004BF85A  8D4818              LEA      ECX,\[EAX+18\]   
015F:004BF85D  8B4508              MOV      EAX,\[EBP+08\]   
015F:004BF860  8B10                MOV      EDX,\[EAX\]   
015F:004BF862  8B4508              MOV      EAX,\[EBP+08\]   
015F:004BF865  8B401C              MOV      EAX,\[EAX+1C\]   
015F:004BF868  E887F6FFFF          CALL      004BEEF4        <---哈哈，目标。这就是 Entry Point 所在的 Call 调用。   
015F:004BF86D  5F                  POP      EDI   
015F:004BF86E  5E                  POP      ESI    
015F:004BF86F  5B                  POP      EBX   
015F:004BF870  5D                  POP      EBP   
015F:004BF871  C20400              RET      0004   
  
这里有个窍门，在按完三个 F12 后，在 SoftICE 代码窗口用 PageDown 向下翻页，找最早出现的 ret 的语句前的 Call 调用处，按 F7 键可以快速来到 Entry Point 所在的 Call 调用。这样不用按 F10 一步步地跟踪，不过如果要找 import table 时，就要一步步的来。   
  
二、找宝贝-------Entry Point   
  
【注】我遇到的 Asprotect 加壳的程序在跟踪到 Entry Point 入口的 Call 调用以后，内部跟踪时有很大不同，简单的，不远处就发现了。难的，其中还有陷井。Asprotect v1.0 加壳的程序就是这种情况。arpr.exe 也一样。而 Thumbnailer 就没有这样的危险。   
  
如果你继续跟踪，可能会在一个 Call 调用中发现这种情况。   
015F:004AF992  8BC0                MOV      EAX,EAX   
015F:004AF994  5A                  POP      EDX   
015F:004AF995  54                  PUSH      ESP   
015F:004AF996  55                  PUSH      EBP   
015F:004AF997  57                  PUSH      EDI   
015F:004AF998  56                  PUSH      ESI   
015F:004AF999  53                  PUSH      EBX   
015F:004AF99A  50                  PUSH      EAX   
015F:004AF99B  52                  PUSH      EDX   
015F:004AF99C  54                  PUSH      ESP   
015F:004AF99D  6A07                PUSH      07   
015F:004AF99F  6A01                PUSH      01   
015F:004AF9A1  68CEFAED0E          PUSH      0EEDFACE   
015F:004AF9A6  52                  PUSH      EDX   
015F:004AF9A7  E914E3FFFF          JMP      KERNEL32!RaiseException   
015F:004AF9AC  C3                  RET   
看到了吧，这是一个异常，Asprotect 留下的陷井。如果没有特别的方法，跟踪就此中断了，虽然离 Entry Point 不远了。（我为此头痛了好几天）经过几天的奋斗，我找到的以下方法可以继续跟踪。   
  
在 JMP KERNEL32!RaiseException 处（其实你也可以不用进入此 Call 调用中，只要在此 Call 调用前的 mov fs:\[eax\],esp 指令后就行了。）   
1、下指令   
db fs:0   
  
显示如下   
0EE7:00000000 D0 FD 5C 00 00 00 5D 00-00 F0 5B 00 F6 38 77 29  ..\\...\]...\[..8w)   
0EE7:00000010 00 00 00 00 00 00 00 00-3C A7 5D 81 01 00 FF FF  ........<.\].....   
0EE7:00000020 00 00 00 00 B1 5D 78 C1-8F 29 00 00 C4 A7 5D 81  .....\]x..)....\].   
0EE7:00000030 4C A5 5D 81 48 69 00 80-?? ?? ?? ?? ?? ?? ?? ??  L.\].Hi..........   
0EE7:00000040 ?? ?? ?? ?? ?? ?? ?? ??-?? ?? ?? ?? ?? ?? ?? ??  ................   
  
2、注意前四个字节，看到了吗，前四个字节为 D0、FD、5C、00。当成长整数，也就是 005CFDD0。好了，我们下指令   
dd 015F:005CFDD0   
（注：这里下指令 dd 005CFDD0 不行，我试过，必须要加你跟踪时的程序所在段选择符，就是 SoftICE 屏幕中代码部分最左边看到的 015F:xxxx 字样）   
  
显示如下   
015F:005CFDD0 005CFF68  004BEF31  005CFDF8  004ACAF0      h.\\.1.K...\\...J.   
015F:005CFDE0 00489AEC  004BF654  0045CC6F  00400000      ..H.T.K.o.E...@.   
015F:005CFDF0 004BE624  005CFE1C  005CFE0C  004BF86D      $.K...\\...\\.m.K.   
  
3、这里注意第四-八个字节，也就是 004BEF31。我们下指令   
u 004BEF31   
（注：这里可以省略段选择符）   
  
显示如下   
015F:004BEF31  E93208FFFF          JMP      004AF768        <--关键！   
015F:004BEF36  E8C90AFFFF          CALL      004AFA04   
015F:004BEF3B  33C0                XOR      EAX,EAX   
  
4、看到了吗 Jmp 004AF768，好就在此处下断点   
bpx 4AF768   
  
5、好了，准备工作做完，就再不怕 Asprotect 给我们下的异常陷井了。按 F5 继续，三次以后（因为我跟踪的情况只有三次异常陷井），还是中断在我们设好的断点处。   
以后跟踪的时候，就没有什么难点了，OK，Entry Point 离你近在咫尺了。   
  
（注：如果你有了经验，可以还可以进一步简化跟踪步骤。找到它要到的地方，直接按 F7，就不用那么多步骤。这也是我前面直接按 F7 的原因。当然那是我 N \*  N 次战斗后的一点收获。这里我想让大家了解一下原因，就写得详细一点。如果想了解以上方法的原理，到绿盟处下载第十期的月刊。看看 WebCrazy 写的《 Windows NT/2000 内部数据结构探究》一文，如果还不懂，问问 WebCrazy 吧，抱歉！我只懂这一点儿。如果没有 WebCrazy 大虾的文章，恐怕我还在 Asprotect 的陷井中兜圈子。唉，Asprotect 连 CIH 病毒中的一些技术都用进去了。不知道以后还有什么变化。不会把二维变形病毒中的技术也用进去吧，写个什么变形引擎之类的，ou，我这不是没事找抽吗？）   
  
  
【后记】   
由于手头上收集的 Asprotect 加壳程序不多，所以没法更多的验证。dr0 老兄要我说说，就写了此文。   
对 Asprotect 加壳程序的脱壳花了我很长时间。就有些问题向 RuFeng 兄写信问过，（不过他现在正在开发调试器，打扰他实在抱歉）也没有下文。cloudnumber9 兄这些天也没碰上。没办法，自力更生处理了一部分问题。不过我还有许多问题要向各位请教？（老毛病了）看在我写文章劳心劳力劳神的份上，知道的就请告诉我。欢迎交流！！！   
  
1、Asprotect v1.0 加壳的程序的 import table 怎样重建，我用 ProcDump 得到的不完整，无法使用？手动脱壳也不成功。cloudnumber9 兄说国外都已经出了修正工具？那儿有下载？   
  
2、怎样写 UnAspack 之类的工具？那儿有相关文章？   
  
  
〖声明〗   
说一下我写文章的目的，我希望大家把它当作是一种技术交流，不要当作教学一类。因为许多观点还只是我一家之言，我不想误导初学者。  

 MSDN 上面有介绍 SEH, <<Advanced Windows>>上面详细讲述了如何在 C 中使用 SEH, win32asm.cjb.net 也有讲用汇编来实现 SEH 的. 我记得似乎 VBOX 的某个版本也用了 RaiseException( ).   4dd466d0e1e0cc123afbe504da2462ca