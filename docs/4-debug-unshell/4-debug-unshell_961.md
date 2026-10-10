

# 红色警报 2 脱壳

> 原文：[`www.pediy.com/kssd/pediy03/forum609.htm`](https://www.pediy.com/kssd/pediy03/forum609.htm)

红色警报 2(red Alert 2)繁体版脱壳解密说明   
1 概述 ：   
      红色警报 2 是 WestWood 的新版游戏，它在众多游戏中率先采用了 c-dilla 公司的光盘保护技术 safedic   
2 来对其起正版保护。要让游戏在虚拟光驱的环境下能够运行，首先必须模拟或者跳过对光盘保护特征区域   
的调用。根据实际的情况，在 red alert 2 中并没有把游戏的数据写入保护区域。这样就为我们通过脱去游   
戏的光盘识别部分的判别程序。重构一个不具备判断保护的程序提供了可能。以下的做法就是在以脱壳的基   
础上实现其在虚拟光驱下执行游戏的一种实现方法的操作说明。   
  
\--使用工具   
  - TRW V1.23 (safdisc2 的保护部分对 softice 的防护很周到，就是加补丁也搞死。好在还有 TRW)   
  - 天意 II0.46 测试版(也是一个很新的动态反编译工具，就是现在不太稳定。支持的命令也不太全)   
  - ProcDump 1.6.2  FINAL  VERSION (脱壳工具，用它可以方便的重内存中 dump 出映像)   
  - W32Dasm Version 8.93 (比较直观的静态反汇编工具)   
  - UltrEdit32 (或者其他的 2 进制编辑器用来修改程序)   
  - Vitual Driver 2000 v 6.0 虚拟光驱软件(farstone)      
\-----具体操作   
\---脱壳   
  首先用 TRW 1.23 装入该软件的可执行档 ra2.exe.程序载入后 ,首先停留在。stext371 这个 section 中代码   
  部分如下：   
016F:0041C1FD  55                  PUSH    EBP//装入停留在这里   
016F:0041C1FE  8BEC                MOV    EBP,ESP   
016F:0041C200  60                  PUSHAD   
016F:0041C201  B87BC24100          MOV    EAX,0041C27B   
016F:0041C206  2DFDC14100          SUB    EAX,0041C1FD   
016F:0041C20B  03057CC24100        ADD    EAX,\[0041C27C\]   
016F:0041C211  C705FDC14100E9000000 MOV    DWORD PTR \[0041C1FD\],E9   
016F:0041C21B  A3FEC14100          MOV    DOWRD PTR \[0041C1FE\],EAX   
016F:0041C220  68C9C04100          PUSH    0041C0C9   
016F:0041C225  68BBC04100          PUSH    0041C0BB   
016F:0041C22A  6809C04100          PUSH    0041C009   
016F:0041C22F  689BC04100          PUSH    0041C09B   
016F:0041C234  A021C04100          MOV    AL,\[0041C021\]   
016F:0041C239  3C01                CMP    AL,01   
016F:0041C23B  7407                JE      0041C244   
016F:0041C23D  B800000000          MOV    EAX,00   
016F:0041C242  EB03                JMP    0041C247   
016F:0041C244  8B4508              MOV    EAX,\[EBP+08\]   
016F:0041C247  50                  PUSH    EAX   
016F:0041C248  E833000000          CALL    0041C280//这里就是 safedisc2 解密代码的入口，按 F10 跳过   
016F:0041C24D  83C414              ADD    ESP,14   
016F:0041C250  83F800              CMP    EAX,00   
016F:0041C253  741C                JE      0041C271   
当程序过了 41c248 这个调用后就会弹出一个开始图象，在这个 call 41c280 中就是 safdisc2 的关键部分，但现   
在不用去研究它。当按 F10 走过这里时，程序再单步执行几步会跳出.stext371 这个 section,来到如下的部分   
这时看看 section 的情况，已经是在.text 的部分了。这就是说程序已经到了无壳时候的部分。该是游戏程序   
的真面目了。   
016F:0040787F  55                  PUSH    EBP//停在这里   
016F:00407880  8BEC                MOV    EBP,ESP   
016F:00407882  6AFF                PUSH    FF   
016F:00407884  6878234100          PUSH    00412378   
016F:00407889  68E4C54000          PUSH    0040C5E4   
016F:0040788E  64A100000000        MOV    EAX,BYTE PTR FS:\[00\]   
016F:00407894  50                  PUSH    EAX   
016F:00407895  64892500000000      MOV    FS:\[00\],ESP   
016F:0040789C  83EC58              SUB    ESP,58   
016F:0040789F  53                  PUSH    EBX   
016F:004078A0  56                  PUSH    ESI   
016F:004078A1  57                  PUSH    EDI   
016F:004078A2  8965E8              MOV    \[EBP+E8\],ESP   
016F:004078A5  FF15DC104100        CALL    \[004110DC\]   
016F:004078AB  33D2                XOR    EDX,EDX   
016F:004078AD  8AD4                MOV    DL,AH   
016F:004078AF  891518724100        MOV    \[00417218\],EDX   
016F:004078B5  8BC8                MOV    ECX,EAX   
016F:004078B7  81E1FF000000        ADN    ECX,FF   
016F:004078BD  890D14724100        MOV    \[00417214\],ECX   
016F:004078C3  C1E108              SHL    ECX,08   
016F:004078C6  03CA                ADD    ECX,EDX   
016F:004078C8  890D10724100        MOV    \[00417210\],ECX   
当程序在 40787f 处停下时在 TWR1.22 中下命令 suspend，挂起程序，回到 WINDOWS 界面，现在我们   
要脱掉程序上面的外壳了。在 windows 下调出 ProcDump 1.6.2  FINAL  VERSION 把内存中的影象 dump   
出来，打开 procdump 后再 task 对话框中可以找到一个 ra2.exe 的影象，用鼠标选中它，再点鼠标右键   
，在弹出框中选择 dump(full)项，然后选一个你指定的文件名保存，我用 abd.exe 保存，这样就把程   
序的主体部分给分离出来了。该是没 safedic2 调用了。    
  
\-----IMPORT\_TABLE 导入表的重构    
\------第一种情况（你要做 CALL \[\*\*\*\*\*\*\*\*\]= 转换=> Call Kernel32.dll(或 user32.dll)!具体函数）   
  然后执行一下该档碰碰运气，会不会就 OK 了。可惜程序可没这样简单，马上给报非法错误。。就在 4078a5   
处的 call\[004110dc\]非法错误，这个可是大麻烦处。以下的处理全都是为了解决这个问题而采取的。首先   
只有回到原来的程序重新调试到 4078a5,然后跟进去看看，马上就是调用一个外接的文件了，空间已经到了另   
一个程序应该是一个 dll 文件。看看它都做了些啥，下内存断点 bpm esp RW 执行几下后发现程序最后在   
kernel32.dll 的空间中停下，想想该是有 API 调用了。继续发现是对函数 Kernel32.dll!GetVersion()的调用，   
这只有一个可能加壳程序把原程序的导入表(IMPORT\_TABLE)中的 Kernel32 函数破坏了。它通过外接的 dll 用函   
数 GetProcAddress 来对每个原来的函数导入进行实现，而且把函数的执行也在其中代劳了。   
  那我就先在程序中找找导入表的位置把，用 procdump 打开，点中 PE Editor 按钮发现 IMPORT\_TABLE 的 RVA 是   
1f000 size 1b8,找到 1f000 的位置发现导入表的位置全不对！看来程序是把它给隐藏了。而且用 W32Dasm 反汇   
编也找不到。只有手工找找，根据 PE 文件的结构，我先在 abd.exe 中查找字符串"kernel32.dll",在 131a4 这个   
RVA 找到一个，然后再在程序中查找二进制 a4 31 01 也就是看 IMPORT\_TABLE 的 IMAGE\_IMPORT\_DESCRIPTOR 结构里   
有 Kernel32.dll 这个函数吗，还好，在 12ce4 处找到。 在我可以先假定 IMPROT\_TABLE 的 RVA 为 12cd8（根据结构   
IMAGE\_IMPORT\_DESCRIPTOR 找到其偏移首址）,先改改看。   
  用 procdump 打开 abd.exe，点中 PE Editor 按钮,把 IMPROT\_TABLE 的 RVA 改为 12cd8。然后再用 W32Dasm 反汇编   
看见 FUCTION 按钮的 IMPORT 已经出来了，可以看见 kernel32.dll,user32.dll,GDI32.dll,ADVAPI32.dll,   
Shell32.dll,COMCTL32.dll 已经出来了，但可以看见，kernel32.dll,和 user32.dll 的函数全是空白。这些函   
数看来要我去给找出来了。其他的 dll 调用都正常了。   
  下面就是找到程序的加密部分进行分析。把程序走过的 API 进行重构。还是针对第一个出错的地方，在跟进   
的 dll 空间可以看见以下代码   
  
016F:004078A5  FF15DC104100        CALL    \[004110DC\] 切入下面的代码中   
016F:004078AB  33D2                XOR    EDX,EDX   
/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/   
注：程序在下面用了间接 Ret 指令的方式代替 call 调用返回的方式，把调用下一条指令的 IP（004078AB)及   
    标志和所有的寄存器保存在堆栈单元中。然后再用改变堆栈的方法切回 4078AB,具体实现见下面代码   
    010c6CD7  PUSH DWORD BFEA13B4   
    010c6CDC  PUSHF//保存标志   
    010c6CdD  PUSHA//保存所有的寄存器   
    010c6CDE  PUSH ESP          
    010c6CdF  PUSH DOWRD 10C6D167   
    010c6CE4  CALL 100195F0  //进入~df394b.tmp，其实是个 dll 文件，F8 跟进可发现    
    010c6CE9  ADD ESP BYTE +8//不会到这条指令的，再上一条指令用 F10 的话就过了！！！！game  begin   
    010c6CEC  PUSH  BYTE 00   
    010c6CEE  POP EAX   
    010c6CEF  POPA   
    010c6CF0  POPF   
    010c6CF1  RET          
  
走走....到下面........找了好久 在~df394b.tmp 中下面的指令中有花指令，静态汇编不可看的到的。   
  
    0100183c8  SETS \[EBP\-8\]   
    0100183cc  MOV EAX,\[EBP\-8\]   
    0100183cf  AND EAX,0x000000FF   
    0100183d4  TEST EAX,EAX   
    0100183d6  JNZ  0100183e5 //这里通常是要跳的   
    0100183d8  MOV ECX,\[0100500E8\]   
    0100183df  CALL  Kernel32.dll!SetEvent   
    0100183e5  JMP  SHORT  0100183ee//再 1 跳   
    0100183e7  MOV EBX,EBX   
    0100183e9  JO  0100183F1   
    0100183eb  NOP   
      0100183ec  JNO  SHORT 0100183f1//再 N 跳，花指令一直就在跳，这是最后一跳。。    
    0100183ee  JMP 0100183e7//再 2 跳   
    ...............................花指令   
    0100183f1  MOV ESP,\[EBP+0c\]//跳到这里水落石出，这里 EBP+0c 中的就是 IP（004078AB)那里压入    
                                //的环境数据   
    0100183f4  POPA//修改标志为 4078a5 的的状态   
    0100183f5  POPF//修改寄存器 4078a5 时的状态   
    0100183f6  RET //切入 Kernel32.dll 中的对应函数中，在这条指令下一条时，堆栈已经指向 IP  4078AB   
                    //当程序重 Kernel32 模块的对应函数返回时，就返回到 4078AB 回到主程序执行。    
/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/   
说了半天，结果其实这里就是一个 API 调用，完全可以通过把函数在 004078A5  CALL    \[004110DC\]处的内容   
改为具体的 API 就可以了，这需要对 PE 文件的 IMPORT\_TABLE 的结构了解。你可以发现其实其他的 call 调用的区   
域跟这个一样，都是在 IMAGE\_IMPORT\_DESCRIPTOR 结构的 Kernel32.dll 或 user32.dll 的 FirstThunk 指象的区域   
你可以发现现在这里的全是通过上面的外接调用实现的，我们要改它其中的内容为内部直接调用，这就要用手   
工了，很麻烦。。首先在程序的.rdata section 的末端选择全是 00 的区域把该函数的 hint 和名字字符串写入，   
可能你还需要手工在 procdump 修改.rdata section 的 Vsize 和 Psize 为能放下你的所有函数 hint 和名字串的大   
小我的这个函数导入表的构造方法如下   
第一个断点在 4078a5 中，其真正的 CALL dword ptr \[4110dc\] ,110dc 地址的指针改为 1394d,在 1394d 中有第一   
个函数 GetVersion 的 hint 及标示字符（这里是自己加的哟）   
  RVA 110dc------value------>1394d =>  RVA 1394d----------value------->0800GetVersion   
                                                                          ^    ^   
                                                                      hint Characteris   
                                                                        
这样就构造好了第一个函数 Kernel32.dll!GetVersion()   
后面的这种调用如法泡制，可能要重构 30 几个这样的函数。但可能还是有规律的，要是能找到规律或完整的   
导入表，就不用这样辛苦了。   
  
  
  
  
  
  
  
  
\------第二钟情况（你要做 JMP.stxt774 的入口地址==转换=>Call Call Kernel32.dll(或 user32.dll)!具体函数)   
    第二种情况通过远跳指令跳到.stxt774 的相应入口中。   
第一条花指令切入是函数 GetVersionExA,位置在   
409450  jmp 41b12a（跳到。stxt 段）  ；这里实际上是  call Kernel32.dll!GetVersionExA，原程序的加   
                            ；密处理可以见后继 code 程序下面的是花指令，静态反汇编会看不见真正代码   
  
它在.stxt774 section 中的代码如下   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00409450(U)   
|   
:0041B12A 53                      push ebx   
:0041B12B E800000000              call 0041B130 //这个 call 的目的只是把 IP 压栈保存   
  
\* Referenced by a CALL at Address:   
|:0041B12B    
|   
:0041B130 870424                  xchg dword ptr \[esp\], eax//   
:0041B133 9C                      pushfd  //标志保存   
:0041B134 05D0FEFFFF              add eax, FFFFFED0   
:0041B139 8B18                    mov ebx, dword ptr \[eax\]   
:0041B13B 6BDB0A                  imul ebx, 0000000A //计算切入 dll 的位置   
:0041B13E 035804                  add ebx, dword ptr \[eax+04\]   
:0041B141 9D                      popfd //恢复状态   
:0041B142 58                      pop eax   
:0041B143 871C24                  xchg dword ptr \[esp\], ebx//切入~df394b.tmp 的位置   
:0041B146 C3                      ret//切入   
  
    
    
  然后可以看见下面的类似代码中   
  
    010c6CD7  PUSH \*\*\*\*\*\*\*\*\*\*\*//409450 的下一条指令，可是花指令哟，这里处理很毒的，不看这里的    
                                //话你是不能看见 409450 的真正的下条指令的情况的。   
    010c6CDC  PUSHF//保存标志   
    010c6CdD  PUSHA//保存所有的寄存器   
    010c6CDE  PUSH ESP          
    010c6CdF  PUSH DOWRD 10C6D167   
    010c6CE4  CALL 100195F0  //进入~df394b.tmp，其实是个 dll 文件，F8 跟进可发现    
    010c6CE9  ADD ESP BYTE +8//不会到这条指令的，再上一条指令用 F10 的话就过了！！！！game  begin   
    010c6CEC  PUSH  BYTE 00   
    010c6CEE  POP EAX   
    010c6CEF  POPA   
    010c6CF0  POPF   
    010c6CF1  RET   
  
  
  
  
  
  
这种情况的修改方法，在程序的相对偏移地址 13bd9 中加入 3300GetVersionExA   
                                                    ^    ^   
                                                    hint Characteris   
然后在 IMPORT\_TABLE 区域的空余空间相对偏移地址 11184 中写入 Hint 和 Characteris 的地址首址 13bd9 即   
Address 411184------value------>13bd9 =>  Address 13bd9----------value------->3300GetVersionExA   
                                                                                ^    ^   
                                                                                hint Characteris   
然后再动态执行原程序下断点 bpx 409450,用 a 指令动态修改这里的汇编代码   
a   
\*\*\*\*\*:409450  call near \[411184\]   
\*\*\*\*\*:409456  \*\*\*\*\*\*\*\*\*\*\*        ;这样原来的指令就还原了，可以看见真正的指令   
记录下程序在该处的 16 进至数据为 FF1584114100,然后用编辑器在程序中的该位置修改为这几个数据，完成   
对 GetVersionExA 的重构 。。。。    
  
  
  用上面的方法一个一个跟踪就可以得到一个完全无壳的可执行文件， 最后在一切搞定后执行的时候一定要把   
原程序的执行目录里的 ra2.lcf 复制为 abd.lcf 这样才行。也可以让程序执行了，但过程真的好麻烦要是能够生成   
这些东西就好了。在制作虚拟光碟时用 farstone 的制作虚拟光碟制作 vcd 档选定智慧算法读取，不要用穷读法，否则过不了 safe   
disc2 的不可读保护，制作完毕后装入就可以不用光盘第顽 red  alert 2 游戏了。  4dd466d0e1e0cc123afbe504da2462ca