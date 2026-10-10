

# PE-ShiELD v0.2b2

> 原文：[`www.pediy.com/kssd/pediy03/forum397.htm`](https://www.pediy.com/kssd/pediy03/forum397.htm)

 标题: 小丁丁 和 WinDos2000 过来看看(多谢帮助 ^\_^),哪位 1212 也过来看看吧  ----对 PE-ShiELD v0.2b2 加壳程序的脱壳的一点分析   
  
内容:   
对 PE-ShiELD v0.2b2 加壳程序的脱壳的一点分析   
  
【声明】   
我写文章以交流为主，希望大家在转载时能保持文章的完整性。   
  
【前言】   
脱 PE-ShiELD v0.2b2 的壳很花功夫，不过为了感谢丁丁虾和 WinDos2000 的帮助，还是赶写了此文，选这个壳的原因，并不是它很简单，只因为正好碰上一位 1212 想了解这个壳，而我又有过这个壳的脱壳经验。所以就选它了。不过现在这个版本据说已经有脱壳机了，而我当时脱壳时还没有。所以，如果你只为了脱壳，就没有必要看此文了。找脱壳机就是了。   
  
这个程序的手动脱壳很花功夫，这里写这篇文章主要侧重一点它的保护机制。在我跟踪此程序的过程中发现他反跟踪的地方相当多，不过幸亏没有多少故意让你死机的代码。列举一下他反跟踪的种类：   
1、检测 \\\\.\\SICE 来防范 SoftICE 的跟踪   
2、检测 \\\\.\\TRW 来防范 TRW2000 的跟踪   
3、在对 UnhandledExceptionFilter 及其要引入的 API 函数在调用之前检查入口代码是否为 0xCC。防范在 API 函数上设断点跟踪。   
4、在对要引入的 API 函数在调用时，并不从入口处执行，而是进入其中的代码执行。防范在入口几处的代码上设断点跟踪。   
4、检测 EntryPoint 和 NumberOfSections 是否被修改，来防范程序被修改，或者是 SMC 的方法进行破解。   
5、把文件头部的所有数据进行累加和 CRC 比较是否被修改，来防范程序被修改或者破解。   
6、使用"花指令"和 SMC 的方法来分段还原代码，来防范静态分析或者增加动态跟踪的难度。   
7、修改了代码段中的 JMP \[xxxxxxxx\]使之指向指向内存中动态申请的地址，防范用一般方法脱壳。   
8、运行时破坏了 PE 结构中的所有数据，来防范用 ProcDump 等工具脱壳或者 DUMP 内存映象。   
9、修改了 WINDOWS 加载环境中保留的 PE 结构，来防范 ProcDump 等工具脱壳或者 DUMP 内存映象。   
还有一些，不足列举。   
  
样例文件：    znzdatII.exe    （指南针数据转换程序）   
下载地址：    http://202.102.230.15/tanfu/soft/mdat/znzdatII.exe   
加壳方式：    PE-ShiELD v0.2b2   
检测工具：    FileInfo v2.30   
目标：        手动脱壳   
脱壳工具：    SoftICE 4.05，ICEDUMP 6.0168，PEditor 1.6，HEX WorkShop 2.10，HIEW 5.91   
作者：        ljttt   
写作日期：    2000-10-28   
  
1、启动 ICEDUMP。   
  
2、Ctrl-D 进入 SoftICE，设下断点   
bpint 3   
  
3、用 PEditor 1.6 打开 znzdatII.exe，记下其中的关键，然后单击 break'n'enter 按钮，单击 Run 按钮。将中断进入 SoftICE。   
Entry Point:        AA000   
Image Base:            400000   
Image Size:            AD000   
Size of Headers:    600   
  
4、现在开始了我们的艰苦历程。由于显示的都是"花指令"，所以无法正确显示出正确的反汇编结果。列举代码也没有意义。不过跟踪了一次后，心里有数以后，可以在某些自己认为比较关键的代码处设下断点，以便下次跟踪时可以快速按 F5 到此。   
  
5、我们来看看 UnhandleExceptionFilter 函数入口代码的比较代码：   
015F:004AA2F9  EB01                JMP      004AA2FC                <--反汇编正确的代码   
015F:004AA2FB  E98D9DB511          JMP      1200408D                <--反汇编不正确的代码   
015F:004AA300  0000                ADD      \[EAX\],AL                            
015F:004AA302  EB02                JMP      004AA306   
015F:004AA304  CD20                INT      20 VXDJmp 000D,4DE8   
015F:004AA30A  00EB                ADD      BL,CH   
015F:004AA30C  01EA                ADD      EDX,EBP   
015F:004AA30E  8038CC              CMP      BYTE PTR \[EAX\],CC        <--这句是关键！   
015F:004AA311  EB01                JMP      004AA314   
015F:004AA313  E90F84AF03          JMP      03FA2727   
015F:004AA318  0000                ADD      \[EAX\],AL   
可以看到许多代码反汇编的结果不正确。这是其中一处在主程序中反跟踪代码，还有许多在子程序中的反跟踪代码就不一一列举了。   
我们可以记住 CMP BYTE BTR \[EAX\],CC 的代码地址 004AA30E ，那么设下断点后下次跟踪时就方便了。其他类似。   
  
6、我们再来看看 检测 TRW2000 的地方。（记住代码地址 004AA5C3 ）   
015F:004AA59C  6800008000          PUSH      00800000   
015F:004AA5A1  EB01                JMP      004AA5A4                <--反汇编正确的代码   
015F:004AA5A3  C78D8560100000EB02EAMOV      DWORD PTR \[EBP+00106085\],EA02EB00    <--反汇编不正确的代码   
015F:004AA5AD  0450                ADD      AL,50   
015F:004AA5AF  EB01                JMP      004AA5B2   
015F:004AA5B1  C78B858D110000EB01EAMOV      DWORD PTR \[EBX+00118D85\],EA01EB00   
015F:004AA5BB  E821070000          CALL      004AACE1                <--反跟踪，检测 \\\\.\\TRW.VXD 来防范 TRW2000 的跟踪   
015F:004AA5C0  EB01                JMP      004AA5C3   
015F:004AA5C2  C783F8FFEB01EA0F85E3MOV      DWORD PTR \[EBX+01EBFFF8\],E3850FEA   
015F:004AA5C3  83F8FF              CMP      EAX,-01                <--比较的地方   
015F:004AA5C6  EB01                JMP      004AA5C9   
015F:004AA5C8  EA0F85E3000000      JMP      0000:00E3850F   
015F:004AA5CF  8D8585100000        LEA      EAX,\[EBP+00001085\]   
015F:004AA5D5  EB01                JMP      004AA5D8   
015F:004AA5D7  EA8D9D92100000      JMP      0000:10929D8D   
015F:004AA5DE  EB02                JMP      004AA5E2   
  
7、好了，现在快到了关键的地方了。我们来看看：   
015F:004AA638  6A00                PUSH      00   
015F:004AA63A  EB01                JMP      004AA63D                <--反汇编正确的代码   
015F:004AA63C  E96A00EB01          JMP      0235A6AB                <--反汇编不正确的代码   
015F:004AA641  C78B8592100000EB01EAMOV      DWORD PTR \[EBX+00109285\],EA01EB00   
015F:004AA64B  E891060000          CALL      004AACE1                <--好象是 CreateThread 调用   
015F:004AA650  EB01                JMP      004AA653   
015F:004AA652  E9FF95A510          JMP      10F03C56   
015F:004AA653  FF95A5100000        CALL      \[EBP+000010A5\]            <--这里是 ExitThread 调用   
015F:004AA659  8B859D110000        MOV      EAX,\[EBP+0000119D\]        <--注意要先在此处设下断点   
015F:004AA65F  E97D060000          JMP      004AACE1   
015F:004AA664  8B85E0110000        MOV      EAX,\[EBP+000011E0\]   
015F:004AA66A  E972060000          JMP      004AACE1   
015F:004AA66F  8B85E4110000        MOV      EAX,\[EBP+000011E4\]   
在 CALL \[EBP+000010A5\] 处我们停下来，在其后的代码 MOV EAX,\[EBP+0000119D\] 处设下断点   
bpx 004AA659   
然后按 F10 继续，否则程序将直接执行。那么这一次的跟踪就结束了。如果对比在 CALL \[EBP+0000119D\] 执行前后的 401000 代码处的数据就可以发现执行完后代码加载到内存中了。这时的代码是还原后的代码。不过还有些没有还原。这也是 PE-SHiELD 防范脱壳的技巧吧。其中一处是代码段中 JMP \[xxxxxxxxx\] 语句没有完全还原，另外就是 .idata Section 所在的 Import Table 部分没有还原了。   
  
8、我们继续跟踪，现在我们来看看如何得到完整的 .idata Section 。当我们到如下代码处时：   
015F:004AA94B  0FB6C9              MOVZX    ECX,CL   
015F:004AA94E  EB01                JMP      004AA951   
015F:004AA950  EA8B06EB01C889      JMP      89C8:01EB068B   
015F:004AA957  16                  PUSH      SS   
015F:004AA958  EB01                JMP      004AA95B   
015F:004AA95A  6833D3EB01          PUSH      01EBD333   
015F:004AA95F  CA8907              RETF      0789   
015F:004AA962  EB01                JMP      004AA965                            
015F:004AA964  C783C604EB01E283C704MOV      DWORD PTR \[EBX+01EB04C6\],04C783E2   
015F:004AA96E  EB01                JMP      004AA971   
015F:004AA970  684B75DAEB          PUSH      EBDA754B   
015F:004AA975  02EA                ADD      CH,DL   
015F:004AA977  04F3                ADD      AL,F3   
015F:004AA979  A4                  MOVSB   
015F:004AA97A  8B9549110000        MOV      EDX,\[EBP+00001149\]    <--从这里开始的代码将没有"花指令"了，我们终于走出了沼泽地   
015F:004AA980  2B9541110000        SUB      EDX,\[EBP+00001141\]   
015F:004AA986  0395C9110000        ADD      EDX,\[EBP+000011C9\]   
015F:004AA98C  89954D110000        MOV      \[EBP+0000114D\],EDX   
015F:004AA992  8BB589110000        MOV      ESI,\[EBP+00001189\]   
015F:004AA998  0BF6                OR        ESI,ESI   
015F:004AA99A  7433                JZ        004AA9CF   
015F:004AA99C  03B5C9110000        ADD      ESI,\[EBP+000011C9\]   
015F:004AA9A2  83C610              ADD      ESI,10   
015F:004AA9A5  8B9549110000        MOV      EDX,\[EBP+00001149\]   
015F:004AA9AB  2B95C5110000        SUB      EDX,\[EBP+000011C5\]   
015F:004AA9B1  0BD2                OR        EDX,EDX   
015F:004AA9B3  7405                JZ        004AA9BA   
015F:004AA9B5  E854040000          CALL      004AAE0E   
015F:004AA9BA  8B95C9110000        MOV      EDX,\[EBP+000011C9\]   
015F:004AA9C0  2B95D1110000        SUB      EDX,\[EBP+000011D1\]   
015F:004AA9C6  0BD2                OR        EDX,EDX   
015F:004AA9C8  7405                JZ        004AA9CF   
015F:004AA9CA  E83F040000          CALL      004AAE0E   
015F:004AA9CF  80BD3C11000000      CMP      BYTE PTR \[EBP+0000113C\],00   
015F:004AA9D6  7424                JZ        004AA9FC   
015F:004AA9D8  6A04                PUSH      04   
015F:004AA9DA  6800100000          PUSH      00001000   
015F:004AA9DF  FFB5AD110000        PUSH      DWORD PTR \[EBP+000011AD\]   
015F:004AA9E5  6A00                PUSH      00   
015F:004AA9E7  E86DFCFFFF          CALL      004AA659   
015F:004AA9EC  0BC0                OR        EAX,EAX   
015F:004AA9EE  0F8469FDFFFF        JZ        004AA75D   
015F:004AA9F4  8BF0                MOV      ESI,EAX   
015F:004AA9F6  8985A1110000        MOV      \[EBP+000011A1\],EAX   
015F:004AA9FC  8B8DC9110000        MOV      ECX,\[EBP+000011C9\]   
015F:004AAA02  8B9549110000        MOV      EDX,\[EBP+00001149\]   
  
015F:004AAA08  8B420C              MOV      EAX,\[EDX+0C\]                <--大循环体开始处   
015F:004AAA0B  0BC0                OR        EAX,EAX   
015F:004AAA0D  0F84F3000000        JZ        004AAB06   
015F:004AAA13  894A0C              MOV      \[EDX+0C\],ECX                <--注意！   
015F:004AAA16  03854D110000        ADD      EAX,\[EBP+0000114D\]   
015F:004AAA1C  52                  PUSH      EDX   
015F:004AAA1D  51                  PUSH      ECX   
015F:004AAA1E  50                  PUSH      EAX   
015F:004AAA1F  50                  PUSH      EAX   
015F:004AAA20  C6853F11000000      MOV      BYTE PTR \[EBP+0000113F\],00   
015F:004AAA27  8B18                MOV      EBX,\[EAX\]   
015F:004AAA29  81E3DFDFDF00        AND      EBX,00DFDFDF   
015F:004AAA2F  81FB4D464300        CMP      EBX,0043464D   
015F:004AAA35  7518                JNZ      004AAA4F   
015F:004AAA37  8B5805              MOV      EBX,\[EAX+05\]   
015F:004AAA3A  81E3FFDFDFDF        AND      EBX,DFDFDFFF   
015F:004AAA40  81FB2E444C4C        CMP      EBX,4C4C442E   
015F:004AAA46  7507                JNZ      004AAA4F   
015F:004AAA48  C6853F11000001      MOV      BYTE PTR \[EBP+0000113F\],01   
015F:004AAA4F  8BD8                MOV      EBX,EAX   
015F:004AAA51  E80EFCFFFF          CALL      004AA664                    <--记得好象是把 DLL 链接库名称还原   
015F:004AAA56  5B                  POP      EBX   
015F:004AAA57  59                  POP      ECX   
015F:004AAA58  5A                  POP      EDX   
015F:004AAA59  0BC0                OR        EAX,EAX   
015F:004AAA5B  7512                JNZ      004AAA6F   
015F:004AAA5D  52                  PUSH      EDX   
015F:004AAA5E  51                  PUSH      ECX   
015F:004AAA5F  53                  PUSH      EBX   
015F:004AAA60  E80AFCFFFF          CALL      004AA66F                    <--记得好象是加载 DLL 链接库吧   
015F:004AAA65  0BC0                OR        EAX,EAX   
015F:004AAA67  0F8473FCFFFF        JZ        004AA6E0   
015F:004AAA6D  59                  POP      ECX   
015F:004AAA6E  5A                  POP      EDX   
015F:004AAA6F  E8EF000000          CALL      004AAB63                    <--注意！把 DLL 链接库名称 K 掉   
015F:004AAA74  8985D40A0000        MOV      \[EBP+00000AD4\],EAX   
015F:004AAA7A  8B32                MOV      ESI,\[EDX\]   
015F:004AAA7C  890A                MOV      \[EDX\],ECX                    <--注意！   
015F:004AAA7E  8B7A10              MOV      EDI,\[EDX+10\]   
015F:004AAA81  894A10              MOV      \[EDX+10\],ECX                <--注意！   
015F:004AAA84  0BF6                OR        ESI,ESI   
015F:004AAA86  7502                JNZ      004AAA8A   
015F:004AAA88  8BF7                MOV      ESI,EDI   
015F:004AAA8A  03B54D110000        ADD      ESI,\[EBP+0000114D\]   
015F:004AAA90  03BD4D110000        ADD      EDI,\[EBP+0000114D\]   
  
015F:004AAA96  8B06                MOV      EAX,\[ESI\]                    <--小循环体开始处   
015F:004AAA98  0BC0                OR        EAX,EAX   
015F:004AAA9A  7462                JZ        004AAAFE   
015F:004AAA9C  890E                MOV      \[ESI\],ECX                    <--注意！   
015F:004AAA9E  7905                JNS      004AAAA5   
015F:004AAAA0  0FB7C0              MOVZX    EAX,AX   
015F:004AAAA3  EB26                JMP      004AAACB   
015F:004AAAA5  03854D110000        ADD      EAX,\[EBP+0000114D\]   
015F:004AAAAB  66C7000000          MOV      WORD PTR \[EAX\],0000        <--注意！   
015F:004AAAB0  40                  INC      EAX   
015F:004AAAB1  40                  INC      EAX   
015F:004AAAB2  53                  PUSH      EBX   
015F:004AAAB3  56                  PUSH      ESI   
015F:004AAAB4  50                  PUSH      EAX   
015F:004AAAB5  8B9DA9110000        MOV      EBX,\[EBP+000011A9\]   
015F:004AAABB  8BF0                MOV      ESI,EAX   
015F:004AAABD  E8A8030000          CALL      004AAE6A                    <--记得好象是还原引入函数名称   
015F:004AAAC2  899DA9110000        MOV      \[EBP+000011A9\],EBX   
015F:004AAAC8  58                  POP      EAX   
015F:004AAAC9  5E                  POP      ESI   
015F:004AAACA  5B                  POP      EBX   
015F:004AAACB  50                  PUSH      EAX   
015F:004AAACC  52                  PUSH      EDX   
015F:004AAACD  56                  PUSH      ESI   
015F:004AAACE  57                  PUSH      EDI   
015F:004AAACF  51                  PUSH      ECX   
015F:004AAAD0  53                  PUSH      EBX   
015F:004AAAD1  50                  PUSH      EAX   
015F:004AAAD2  50                  PUSH      EAX   
015F:004AAAD3  6878563412          PUSH      12345678   
015F:004AAAD8  E89DFBFFFF          CALL      004AA67A                    <--加载引入函数获得函数地址   
015F:004AAADD  5B                  POP      EBX   
015F:004AAADE  0BC0                OR        EAX,EAX   
015F:004AAAE0  0F8443FCFFFF        JZ        004AA729   
015F:004AAAE6  E878000000          CALL      004AAB63                    <--注意！把引入函数名称 K 掉   
015F:004AAAEB  5B                  POP      EBX   
015F:004AAAEC  59                  POP      ECX   
015F:004AAAED  5F                  POP      EDI   
015F:004AAAEE  E887000000          CALL      004AAB7A                    <--注意！把 IMPORT\_THUNK\_DATA 区 K 掉   
015F:004AAAF3  5E                  POP      ESI   
015F:004AAAF4  5A                  POP      EDX   
015F:004AAAF5  5B                  POP      EBX   
015F:004AAAF6  83C604              ADD      ESI,04   
015F:004AAAF9  83C704              ADD      EDI,04   
015F:004AAAFC  EB98                JMP      004AAA96                 <--小循环体结束处   
  
015F:004AAAFE  83C214              ADD      EDX,14   
015F:004AAB01  E902FFFFFF          JMP      004AAA08                    <--大循环体结束处   
  
015F:004AAB06  EB01                JMP      004AAB09                    <--大循环退出后将到此   
  
以上列出的所有代码所做的工作主要获得函数地址，但是其中有许多代码来防止你跟踪后 DUMP 得到完整的 Import Table。以下说明一下：   
首先程序把加密的 idata Section 复制到申请的内存中（内存地址保存在 EDI 中）。把加密的 DLL 名称还原，然后 LoadLibrary 加载 DLL 动态链接库，完成后又把 DLL 名称用'K'掉。然后把该 DLL 加密的引入函数名称还原，用 GetProcAddress 得到函数地址。完成后又把函数名称用'K'掉。而且其中还有多处代码对 IMPORT\_THUNK\_DATA 区的数据进行了处理。这样来防止你得到完整的 Import Table。因此如果不做处理，内存中始终不会存在一个完整的 Import Table。这也是加壳程序用到的防脱壳的技巧吧。分析了其中的代码后，我们仍然可以想办法得到完整的 Import Table。   
  
其中我标记为"注意！"的代码共有 8 处，列举如下：   
1、015F:004AAA13  894A0C              MOV      \[EDX+0C\],ECX                <--注意！   
2、015F:004AAA6F  E8EF000000          CALL      004AAB63                    <--注意！把 DLL 链接库名称 K 掉   
3、015F:004AAA7C  890A                MOV      \[EDX\],ECX                    <--注意！   
4、015F:004AAA81  894A10              MOV      \[EDX+10\],ECX                <--注意！   
5、015F:004AAA9C  890E                MOV      \[ESI\],ECX                    <--注意！   
6、015F:004AAAAB  66C7000000          MOV      WORD PTR \[EAX\],0000            <--注意！   
7、015F:004AAAE6  E878000000          CALL      004AAB63                    <--注意！把引入函数名称 K 掉   
8、015F:004AAAEE  E887000000          CALL      004AAB7A                    <--注意！把 IMPORT\_THUNK\_DATA 区 K 掉   
好！我们把这些地址的代码全部用 NOP 代替。（注意代码长度不同，NOP 掉的字节数也不一样）这样在循环结束时我们就有了一个完整的 Import Table 了。   
  
9、由于以上代码是个循环，我们并不需要详细跟踪他，所以可以在 JMP 004AAB09 处设下一个断点   
bpx 004AAB06   
然后按 F5 键，这时，程序中断在此处，这时，我们需要 DUMP 出这个完整的 idata Section 了（也就是 Import Table）。下指令   
/dump 5D0000 3000 c:\\temp\\idata.bin   
（其中 5D0000 中申请的内存地址的首地址， 3000 为 idata Section 的大小，此命令用法为 ICEDUMP 6.0168，低版本不同）   
  
10、然后，我们继续跟踪，不远处就会到如下：   
015F:004AAB53  B95B0B0000          MOV      ECX,00000B5B   
015F:004AAB58  EB01                JMP      004AAB5B   
015F:004AAB5A  EAF3AA61EB01EA      JMP      EA01:EB61AAF3   
015F:004AAB61  FFE0  

 10、然后，我们继续跟踪，不远处就会到如下：   
015F:004AAB53  B95B0B0000          MOV      ECX,00000B5B   
015F:004AAB58  EB01                JMP      004AAB5B   
015F:004AAB5A  EAF3AA61EB01EA      JMP      EA01:EB61AAF3   
015F:004AAB61  FFE0                JMP      EAX                                <--在此处停下，此时 EAX=OEP=401000   
015F:004AAB63  0000                ADD      \[EAX\],AL   
015F:004AAB65  0000                ADD      \[EAX\],AL   
也许你看到的一部分代码和我的不一样，为什么？因为程序到此处时会打扫战场。把前后的代码全部清零掉了。只有简单的 JMP EAX 跳转到程序真正的入口处了。好了！说了这么多，到 JMP EAX 处我们停下，下指令   
/dump 400000 AD000 c:\\temp\\dump.exe   
当然，这以后要下指令   
a eip /jmp eip   
然后，用 PEEditor 把这个进程中止掉。（注意哦，不要让他继续运行，因为为了得到完整的.idata Section 我们是对代码做了修改的）   
  
11、前期工作我们完成，但这仅仅是个开始。开始我们繁重的修补工作。   
用 HEX WorkShop 打开 znzdatII.exe 和 dump.exe。把 znzdatII.exe 开始的 600 个字节替换到 dump.exe 头部的相同个数的字节。然后保存退出。   
这是由于我们得到的 dump.exe 头部的 PE 结构完全被破坏了。所以如果你直接用 PEditor 或者 ProcDump 无法得到正确的文件的原因之一。   
  
12、用 PEditor 打开 dump.exe 修改 Entry Point = 000001000，（由前面跟踪时得知）。进入 directory 修改其中的 Import Table 为 94000。（这个由跟踪得知）。进入 sections ，右键执行其中的菜单命令 dumpfixer 使各个 Section 的 RS=VS，R0=V0。   
  
13、最后还要启动我们的 HIEW 来修改其中的 JMP \[xxxxxxxx\] 指令其中的偏移。   
查找 FF,25，当定位的地址反汇编时显示为许多一连串的 JMP \[xxxxxxxxx\] 指令时说明我们找到了地方。可以看到类似如下的代码：   
000835AF: 5D                        pop ebp   
000835B0: C20400                    ret 0004   
000835B3: 90                        nop   
000835B4: FF25DC005D00              jmp dword ptr \[005D00DC\]   
000835BA: FF25E0005D00              jmp dword ptr \[005D00E0\]   
000835C0: FF25E4005D00              jmp dword ptr \[005D00E4\]   
000835C6: FF25E8005D00              jmp dword ptr \[005D00E8\]   
000835CC: FF25A0025D00              jmp dword ptr \[005D02A0\]   
000835D2: FF25A4025D00              jmp dword ptr \[005D02A4\]   
000835D8: FF25A8025D00              jmp dword ptr \[005D02A8\]   
  
可以看到偏移地址定位到跟踪时我们发现的申请的内存地址。（这也是许多加壳程序来防范脱壳的一种办法）。而这并不是程序 Import Table 应该指向的真实地址。我们对比一下该如何修改   
内存空间首地址        005D0000   
Import Table 首地址    00494000    <-- Image Base+Import Table RVA=400000+94000=494000   
所以 005D00DC 对应地址为 004940DC ，所以我们只需要修改每个偏移地址中的两个字节 5D00 = 4940，以此类推。由于这些 JMP 指令有许多   
（我发现的情况它们都在文件偏移 835B4 - 83F12 空间内）。因此修改是个很繁重的工作。批处理修改时必须小心！以前我就是因为无意中错误地修改了不相关的一个字节，结果让我走了很多弯路，以为自己的这种脱壳方法无效。结果第二次重新测试成功。不过这已是一个月后的事了。现在写这篇文章想起当时来真是.......）   
  
【后记】   
由于 PE-SHiELD 0.2b2 版本并不是最新版本，所以可能大家都没有注意。但是我当时并没有发现脱壳机。而且 ProcDump 中 Script 对此无效。所以只好自己手动了。一来了解一下这个壳的特点。二来掌握一下脱壳方法，以便以后再遇到这类壳时也好有个防范。呵呵.......。新版的 PE-SHiELD 0.25 已经有了脱壳机，就不用我 cao 心了。(这论坛"看清楚"了，是我 cao 心了。不是我 cao！！！还让我重发。:-()   
  
\[闲言闲语\]   
小丁丁，就别找 Sun Bird\[CCG\]大哥来扁我一顿了。这篇文章作为向他老人家献上的"保护费"如何？   4dd466d0e1e0cc123afbe504da2462ca