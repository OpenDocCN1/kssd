

# Alchemy Gif Animator 2.3

> 原文：[`www.pediy.com/kssd/pediy04/kanxue026.htm`](https://www.pediy.com/kssd/pediy04/kanxue026.htm)

脱 Alchemy Gif Animator 2.3 的壳   
  
    本来说是不再写 aspr 脱壳的东西了，不过 fpc 老兄推荐的这个东西比较典型，所以有必要说明一下。作为报酬破解就交给 fpc 大哥了，破不出来的话，饶不了他;-)   
  
unpack(2001.9.16)   
  
    由于是 delphi 程序(我看如果是用 delphi 写的程序，还是放弃 aspr 保护吧，跟没加一样)，所以利用快速寻找 aspr 入口的方法:   
执行 AGif.exe，用 prodump 选 dump(full)脱壳,存为 dump.exe。接着用 winhex 打开 dump.exe，选择搜索文本，填 runtime,执行搜索，搜到后,向前找到离 runtime 最近的机器码为 55 8B EC 的地方就是程序的 oep，而在 AGif 里这个位置在 offset:000D4C14 处，用 peditor 的 flc 功能将它转换为 virtual address 就是 004D5614----oep。   
  
    ok，下面要在入口处脱壳，打开 SuperBPM，点 erase，用 trw 载入 AGif.exe，下 g 004D5614，下 suspend。用 prodump 选 AGif.exe 进程 dump(full)，再打 ctrl+n，f5。   
  
    接着修复 import table，ImportREC1.2final 无法提取完整的 it，要手动修复太多函数了，累死~~~，所以用 ImportREC1.2beta2 重建吧，hoho,good luck:)   
  
以下操作必须在 tag&rename 的地盘里探测   
  
(1)    000DB1D4    KERNEL32.dll    0151    GetCommandLineA   
  
(2)    000DB2EC    KERNEL32.dll    024c    LockResource   
  
(3)    000DB328    KERNEL32.dll    01E9    GetVersion   
  
(4)    000DB374    KERNEL32.dll    0163    GetCurrentProcessId   
  
(5)    000DB37C    KERNEL32.dll    024c    LockResource   
  
选 add new section，然后点 fix dump。最后把 dump\_.exe 的 ep 改成 000D5614。   
ImportREC 就这么有限几个函数认不出来，背都能背下来了~~~~\`   
  
  
    举这个例子就是要讲如何让脱壳的程序正常运行，这个比较有意思啦。   
    这样修复后程序能够正常执行，不过主窗口被锁死，不能动，与原版比较跟踪也没有发现任何不一样的地方。百思不得其解的时候，发现运行 dump\_.exe 的时候，同时会生成一个 dump\_.ini，再拿它同原来的 AGif.ini 比较，小了好多，所以将 AGif.ini 做成 dump\_.ini，再运行，正常，我晕倒~~~~~~~~~。   
  
    关于破解:各位的 crack 功力都比我强，这个用不着我动手，偶就等结果吧:)   
  
完活，希望我写的东西能够对大家有所帮助:)   
  
此注册码对 2.3 版无效   
51soft 放出的   
  
Advanced Gif Animator 2.2 中文注册版   
Serial: 0qT+7ks91OS6TwwuLvuwXOgnH9C1VS Eo9HIWngLCMBIuHEWyJzuGuNjjAVvs QIf8lo8MhU4c6QZfVgfvrKE7u6ZOsd sLI8UGMKEK3nDr1cRHcvufQ9LTGS3B z16BSbF7c7cd7gkAFbooFcHO/P8lPf FRTTAg+2Mnrz28OeYhZSIk=   
  
  
  
2001.9.16   
zombieys\[CCG\]   
  
  
———————————————————————————————>   
                .-"      "-.      Cracked by zombieys\[CCG\]  >   
                /　　　　　　\\      qq:1789655                >   
              |              |    http://zombieys.yeah.net  >   
              |,  .-.  .-.  ,|    http://zombieys.126.com  >   
                |)(\_\_/  \\\_\_)(|      zombieys.cn.hongnet.com  >   
                |/    /\\    \\|                                >   
      (@\_@)    (\_    ^^    \_)      Thanks for your supports  >   
    \_  )\\\_\_\_\_\_\_\_\\\_\_|IIIIII|\_\_/\_\_\_\_\_                          >   
\_)@8@8{}<\_\_\_\_\_\_\_\_|-\\IIIIII/-|\_\_\_\_China Crack Group\_zombieys\_\_\_>  4dd466d0e1e0cc123afbe504da2462ca