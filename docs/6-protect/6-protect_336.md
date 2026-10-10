

# \[翻译\]关于慢速感染与慢速多态变形的争论

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-753.htm`](https://www.pediy.com/kssd/pediy08/pediy8-753.htm)

注：文章比较简单，不用加精；附件为已排版的 PDF 文件。  
  
关于慢速感染与慢速多态变形的争论  
  
文档编号：  S001-F006  
原作者：  Rogue Warrior  
译者：  Arhat\[ptg\]  
审校：  NULL  
发布时间： 2006-06-19  
原文：  http://vx.netlux.org/lib/vrw01.html  
关键词：  慢速感染 慢速多态变形  
  
  
  
0 译者注  
1.关于传播速度有较多的争论，仁者见仁，Kris Kaspersky 的《Shellcoder's Programming Uncovered》中对蠕虫传播速度的讨论与此类似，有兴趣者可以参考。  
2.本文中所讨论的方法已经过时，但关于慢速感染及慢速多态变形的想法可以借鉴，希望读者能举一反三，而不是拘泥于本文的内容。  
  
1 慢速传播者  
许多人认为快速传播者比慢速传播者要好一些，但我并不这样认为。病毒的目标是尽可能传播更多的主机。你认为呢？  
对于慢速感染的病毒来说，它必须做：  
  
    1. 不引起注意（秘密行动）。  
    2. 感染当前主机上有可能外出的文件。  
  
OK，因此，我们写一个秘密的快速传播者，它能工作良好吗？错了…  
  
下面是一些理由，第一点可以分成两部分来看：  
  
    1.1 不能被检测（文件／磁盘 隐藏）。  
    1.2 不能被用户注意到（运行速度，内存占用，磁盘空间占用）。  
  
大部分编程者忽略了 1.2，从而使他们的病毒在主机系统上非常显眼――导致系统性能下降，有时候就像蜗牛爬一样。  
在病毒论坛里，我们看到过很多这样的帖子，“我注意到有些异常――因此，我做了一些调查研究，发现了 X 病毒――我怎么清除它呢？” － 我们的目标是首先避免用户产生这种猜疑。  
想想 AIDS／HIV？它在受感染的人体内潜伏超过 10 年，期间会不断感染与之接触的人－但只显示了很少的症状。  
这种道理同样可应用到病毒传播上，它们不能降低系统性能或引起不兼容（QEMM 异常／崩溃，等等），因为人们非常在意计算机运行状况可能出现的差异。  
这导致我们采用慢速感染技术，现在，如果目标是使病毒感染另外的系统，那我们需要做的是感染那些非常有可能被移到另外计算机系统上的文件，这些文件是：  
  
    1.  软盘上的文件。  
    2.  网络／远程设备上的文件。  
    3.  在通信程序内打开的文件。  
    4.  在压缩程序内打开的文件。  
    5.  在备份程序内打开的文件。  
  
所有这些文件都有机会离开系统。  
软盘：- 14-y/o 盗版者用它们交换游戏，是近乎完美的接触其它主机的方法。  
网络设备：- 如果你有超级用户访问权限的话这特别有用，你就有机会感染像 login.exe 之类的文件，那么你就有 250 台（或更多！）成为感染源的计算机。  
N.B., Novell Netware 允许把完整的可引导盘做为文件保存在服务器上。  因此，网络客户端可以从服务器上的文件启动。这些文件也是非常有用的目标，但是我缺乏这样的网络环境来生成／测试这样的病毒。  
通信程序：－上传 EXE／COM 文件的人都有他的理由（或许是发给某个人）。是另外完美的感染一台或多台主机的方法。  
压缩程序：－通常是在用通信程序上传／下载前用来压缩软件，于是，如果我们在压缩的过程中感染 EXE／COM 程序，那我们就有非常好的机会使外出见见世面。  
备份程序：－这可以预防从主机上移去我们的病毒－－如果用户通过 AV 程序发现了病毒，那我们还有机会重新感染它。  
  
    图示：  
                                      .-<<--backups-----.  
        .HOST SYSTEM------------------|---.             |  
        |.------------------..--------'--.|----------> floppies  
        ||ethernet/modem/fdd|| HARD DISK ||----------> network  
        |'------------------''--------.--'|----------> modem/comms  
        '-----------------------------|---'             |  
                                      '-compressed-->>--'  
  
感染硬盘文件用处不大，因此它们从来不会四处闲逛－不像感染软盘，网络和通信通道的文件那样。  
确保你的病毒在其它程序前被加载是个好主意，这样一来可以确保病毒活动秘密行动。  你应该自动感染\\COMMAND.COM 或把病毒分成多部分（感染 mbr/track 0）。  
  
1.1 这些方法的实现：  
    最好的实现方法是：  
  
        Floppy diskette and CDR (CD-ROM Writers) check:  
            Use AX=4408, DL=Logical Drive#, INT 21h  
  
        Network/Remote check:  
            Use AX=4409, DL=Logical Drive#, INT 21h  
            Use AX=440A, BX=File Handle, INT 21h  
  
        Comms/Compression/Backup check:  
            用 LUT（lookup table）禁止秘密活动，并在适当的时候激活快速传播者：  
  
            'CHKDSK'   - Stops CHKDSK errors (well known).  
            'SCANDISK' - Ditto.  
            'NDD'      - Ditto.  
            'PKLITE'   - Protect virus in PKLITE compression wrapper.  
            'DIET'     - Protect virus in DIET compression wrapper.  
            'LZ'       - Protect virus in LZEXE compression wrapper.  
            'TM'       - Telemate, to infect EXE/COM uploads.  
            'TE'       - Telix/Terminate, to infect EXE/COM uploads.  
            'BACKUP'   - To infect executable backups.  
            'MSBACKUP' - Ditto.  
            'CPBACKUP' - Ditto.  
  
            当压缩程序输出文件被打开时，也用下面的 LUT 来查看。  
  
                'ZIP' - PKZIP tmpfile extension opened.  
                'LZH' - LHA tmpfile extension opened.  
                'ARJ' - ARJ tmpfile extension opened.  
                'ARC' - ARC tmpfile extension opened.  
                'RAR' - RAR tmpfile extension opened.  
  
            当你截取 INT 21h AH=3C 和 INT 21h AH=3D 调用时，检查文件的后缀。  
  
            如果被打开的文件是上述的一种，那么激活快速传播者并禁止秘密活动。  
  
当它被关闭时（用 SFT 来检查 AX=1220/INT2F AX=1216/INT2F），重新启用慢速传播者并重新启用秘密活动。  
  
2 慢速多态变形  
慢速多态变形差不多使 AV 团体陷入了无尽的烦恼之中，关于这个问题没有太多的争议。  
这些人（AVer）每天都会收到许多病毒，他们不可能反汇编每一个病毒来看个详细，这对我们来说是个好消息，因为这意味着我们能使他们陷入浪费时间的烦恼之中。  
如果再和 Rajaat 阐述的 anti-goat 技术结合起来使用，将会使他们相当的烦恼。AV 团体为了下面的目的，将不得不花上至少一天的时间分析好的多态变形病毒：  
  
    a）使它感染他们的“替罪羊”文件  
    b）使它产生大量的样本（对于 SMEG 来说，我认为大概能生成 200,000 个样本）。  
  
BTW: 如果你决定使你的代码依赖 anti-goat 代码的完整性，那么你将发现研究员在喂养 200,000 个病毒样本时会有更多的问题：）  
  
2.1 实现  
多态变形过程基于随机数的生成，怎样选择一个慢速生成的随机数呢？我能想到的两个方法是：  
  
    1.  BIOS date.     -非常慢的多态变形，仅根据每个计算机而改变！  
    2.  当天的日期.   -非常慢。  
  
    -------------------------------------------------------------  
  
    1.  push    0FFFF  
        pop     ds  
        mov     si,0005             ;DS:SI -> FFFF:0005 (8 bytes).  
        xor     bx,bx               ;value=0  
        mov     cx,4                ;size=4 words.  
    L1: lodsw                       ;fetch word.  
        add     bx,ax               ;checksum.  
        loop    L1                  ;next 3 words.  
        mov     ds:rnd\_seed,bx      ;set seed.  
  
    -------------------------------------------------------------  
  
    2.  mov     ah,2A  
        int     21                  ;get date.  
        rol     dx,cl               ;random adjustment.  
        xor     dx,cx               ;place cx into eqn.  
        mov     ds:rnd\_seed,dx      ;set seed.  
  
\-------------------------------------------------------------------------------- 

 4dd466d0e1e0cc123afbe504da2462ca