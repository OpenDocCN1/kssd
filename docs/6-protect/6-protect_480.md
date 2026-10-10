

# 修改 regmon/filemon，跳过 themida 的检测

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-382.htm`](https://www.pediy.com/kssd/pediy08/pediy8-382.htm)

【文章标题】: 修改 regmon.filemon，跳过 themida 的检测  
【文章作者】: rockhard  
【作者邮箱】: wnh1@sohu.com  
【软件名称】: regmon/filemon  
【下载地址】: http://www.sysinternals.com/Utilities/Regmon.html  
【加壳方式】: 无  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
  
  
【详细过程】  
  在 themida 加壳选项中，有 anti-regmon 和 anti-filemon 选项，如果选择了，加过壳的程序正常情况下不能用 filemon 和 regmon 监视。本文就是尝试修改 regmon/filemon，以达到正常监视作用。  
    
  regmon/filemon 的实现大家可以参考源码，是利用驱动进入 ring0，然后挂接系统服务例程。themida 作者能用到的方法可以参考 http://bbs.pediy.com/showthread.php?threadid=19624 中所说.文章中列出了如下几点：  
    
  1 - 查找进程。  
  2 - 查找窗口(我在某些壳中也发现了这个)。  
  3 - 查找注册表中的驱动。  
  4 - 查找内存中的驱动。  
  5 - 查找驱动的对象表。  
  6 - 查找 SDT (只对 regmon)。  
    
  经过几次测试发现修改三个地方可以跳过 THEMIDA 的检测.下面是修改过程（这里以 regmon 目前最新版本 7.03 为例，filemon 同样办法），分别修改三样。1.标题 2.窗口类名 3.驱动的文件名  
    
  一　修改标题。  
    
  这是最简单的，运行 Regmon，可以看到窗口最上方标题显示 Registry Monitor - sysinternals:www.sysinternals.com.用 UltraEdit 打开 regmon，查找 ASCII 串："registry monitor -"　共有两处，把整个串改为 Registri Moniter - Sysexternals: www.sysinternals.com.或者其它的什么的.让 themida 从这个串的判断失效就行.  
    
  二　修改主窗口的类名.  
    
  用ＶＣ自带的不工具 SPY++,用 FIND WINDOW 功能，先看下主窗口的类名，看到为：18467-41.仿照上面的办法查找串，这次运气不太好.没找到.但所有的类名都是通过 api RegisterClass/RegisterClassEx 注册到系统的.用ＯＤ加载 Regmon，下断 bp RegisterClassA. F9 运行.程序断在  
    
  77D2148C >  8BFF            mov     edi, edi  
  77D2148E    55              push    ebp  
  77D2148F    8BEC            mov     ebp, esp  
  77D21491    83EC 30         sub     esp, 30  
  77D21494    8B45 08         mov     eax, \[ebp+8\]  
    
  Alt+F9 返回.并稍往上看看结构体 WNDCLASS 的成员，就能看到如下：  
    
  00408F32  |.  50            push    eax  
  00408F33  |.  68 EC594100   push    004159EC                         ;  ASCII "%d-%d"  
  00408F38  |.  56            push    esi  
  00408F39  |.  E8 FE0F0000   call    00409F3C  
  00408F3E  |.  83C4 10       add     esp, 10  
  00408F41  |.  8D4424 08     lea     eax, \[esp+8\]  
  00408F45  |.  897424 2C     mov     \[esp+2C\], esi  
  00408F49  |.  8B35 68224100 mov     esi, \[<&USER32.RegisterClassA>\]  ;  USER32.RegisterClassA  
  00408F4F  |.  50            push    eax                              ; /pWndClass  
  00408F50  |.  FFD6          call    esi                              ; \\RegisterClassA  
  00408F52  |.  66:85C0       test    ax, ax  
  00408F55  |.  75 08         jnz     short 00408F5F  
  00408F57  |.  5F            pop     edi  
    
    
  由窗口类名 18467-41　及ＯＤ 00408F33 的注释 ASCII "%d-%d"　.猜想就是由两个整数用类似 sprintf 函数生成类名的.　第一个想法是找到这两个整数在文件中的位置，并修改这个整数值.但很快的想到一个更简单的办法.就是修改＂%d-%d＂，还是用 UltraEdit 查找"%d-%d",就一处，肯定是这个地方了.我把它修改为%X-%08X.注意儿%X-%08X 比%d-%d 多两个字符.我是因为看到文件中%d-%d 后面有三个 0,否则还是为%X-%X 好，以免破坏程序不能运行.  
    
    
  三　修改由资源中释放出的驱动文件名.  
    
  首先确定驱动的文件名.重新载入 REGMON，下断 BP CreateFileA .F9 运行,看到堆栈出现：  
  0012F0E4   0040F0F7  /CALL 到 CreateFileA 来自 Regmon.0040F0F1  
  0012F0E8   0041829C  |FileName = "C:\\WINDOWS\\system32\\Drivers\\REGSYS701.SYS"  
  0012F0EC   40000000  |Access = GENERIC\_WRITE  
  0012F0F0   00000003  |ShareMode = FILE\_SHARE\_READ|FILE\_SHARE\_WRITE  
  0012F0F4   0012F110  |pSecurity = 0012F110  
  0012F0F8   00000002  |Mode = CREATE\_ALWAYS  
  0012F0FC   00000080  |Attributes = NORMAL  
  0012F100   00000000  \\hTemplateFile = NULL  
  0012F104   0041405E  Regmon.0041405E  
    
    
  上面的 FileName = "C:\\WINDOWS\\system32\\Drivers\\REGSYS701.SYS"　中 REGSYS701.SYS 就是从资源中释放出来的驱动名（如果先前运行过 regmon,OD 可能不能在此断下,因为没有再次创建这个文件,重新启动电脑再次调试）.  
  用 UltraEdit 查找 REGSYS701.SYS,替换为 AAAAAAAAA.SYS(共有三处)  
    
    
  至此,themida 本身即其加过壳的程序都可以用 regmon 监视了.FILEMON 通过同样的方法修改一样通过.  
  
  
【经验总结】  
  除了考试没写过文章,现在要改掉懒习惯.写的不对之处请大家谅解.  
    
  
  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢! 

 4dd466d0e1e0cc123afbe504da2462ca