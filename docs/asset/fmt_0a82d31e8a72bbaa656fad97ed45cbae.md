
具体实现方式是在内核程序中设立全局队列，每当有新的进程创建/销毁消息时，便往队列中追加新的项目。当用户程序读取该队列时，则将队列清空。表示之前的进程监控情况已转达给用户程序。

同时为了保护内核安全，全局队列以 20 笔记录为限，如果记录了超过 20 笔记录仍没有用户程序读取该队列，则设置一个 `Dirty` 标志，表示记录溢出。并且停止消息的添加。在今后用户程序读取队列时，发现 `Dirty` 标志则强制刷新进程列表，防止有“影子”进程的存在。

经过实践证明，该方法不但实践简单，同时也具有很高的效率。每秒一次的轮询足够给用户以瞬时刷新进程列表的感觉。

## 参考文献

*   `Windows Internals,Fourth Edition`
*   `Undocumented Windows NT`
*   `Undocumented Windows.2000.SECRETS.A PROGRAMMERS.COOKBOOK`
*   `Playing with Windows /dev/(k)mem, By crazylord,`  
    `http://www.phrack.org/phrack/59/p59-0x10.txt`
*   `浅谈 NT 下 Ring3 无驱进入 Ring0 的方法, By hopy,`  
    `http://bbs.pediy.com/showthread.php?threadid=37946`
*   `必备绝技 hook 大法( 中 ), By Lvg,`  
    `http://bbs.pediy.com/showthread.php?threadid=42422`
*   `Do All In One EXE File Under Win32, By icelord,`  
    `www.xfocus.net/articles/200610/887.html`
*   `How PAE X86 Works.mht, Microsoft Tech Net,`  
    `http://technet2.microsoft.com/windowsserver/en/library/00284c8d-7a42-40f2-8a01-8de61dccd8c91033.mspx`
*   `内存与进程管理器, By greatdong,`  
    `https://www.xfocus.net/bbs/index.php?act=ST&f=2&t=58182`
*   `Ring0 下搜索内存枚举隐藏进程, By uty,`  
    `http://www.cnxhacker.net/Article/show/3412.html`
*   `Process Termination: Known Methods, By EP_X0FF,`  
    `http://forum.sysinternals.com/forum_posts.asp?TID=5865`
*   `Administrator 用户直接获取 SYSTEM 权限, By scz,`  
    `www.nsfocus.net/index.php?act=magazine&do=view&mid=1900`
*   `获得进程的 EPROCESS, By MustBE`  
    `www.xfocus.net/articles/200406/706.html`
*   `获取 Windows 系统的内核变量, By 于`  
    `www.nsfocus.net/index.php?act=magazine&do=view&mid=2248`
*   `一些常用内核变量的定位, By flyingkisser`  
    `http://forum.byr.cn/pc/index.php?id=flyingkisser&pno=7`
*   `Windows Template Library (WTL),`  
    `http://www.codeproject.com/wtl/`

`4dd466d0e1e0cc123afbe504da2462ca`
