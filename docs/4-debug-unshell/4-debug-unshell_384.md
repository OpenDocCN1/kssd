

# PicturesToExe v3.60

> 原文：[`www.pediy.com/kssd/pediy04/kanxue019.htm`](https://www.pediy.com/kssd/pediy04/kanxue019.htm)

脱 PicturesToExe v3.60 的壳   
  
    有人问这个呀，这是最后一次复习了，简写一下。   
  
unpack(2001.9.15)   
  
    由于是 delphi 程序利用快速寻找 aspr 入口的方法:   
执行 apr.exe，用 prodump 选 dump(full)脱壳,存为 dump.exe。接着用 winhex 打开 dump.exe，选择搜索文本，填 runtime,执行搜索，搜到后,向前找到离 runtime 最近的机器码为 55 8B EC 的地方就是程序的 oep，而在 apr 里这个位置在 offset:0011EC34 处，用 peditor 的 flc 功能将它转换为 virtual address 就是 0051EC34----oep。   
  
    ok，下面要在入口处脱壳，打开 SuperBPM，点 erase，用 trw 载入 apr.exe，下 g 0051EC34，下 suspend。用 prodump 选 apr 进程 dump(full)，再打 ctrl+n，f5。   
  
    接着用 ImportREC1.2final 修复 import table   
  
1    001241F8    KERNEL32.dll    0151    GetCommandLineA   
1    001242EC    KERNEL32.dll    033C    WinExec   
1    00124338    KERNEL32.dll    024C    LockResource   
1    0012437C    KERNEL32.dll    01E9    GetVersion   
1    001243D4    KERNEL32.dll    0163    GetCurrentProcessId   
1    001243DC    KERNEL32.dll    013D    FreeResource   
1    0012441C    KERNEL32.dll    00C7    CreateProcessA   
  
选 add new section，然后点 fix dump。   
  
    最后把 dump\_.exe 的 ep 改成 0011EC34。这个我以前疏忽了，ImportREC 修完程序后会在后面加\_,所以以前的都是改 dump\_.exe。   
  
    这是我唯一见过的一个脱壳后能正常运行的软件了。   
  
    我写的够多啦，不是特别难，。   
  
    crack 我可不管了，谁有时间就来试试吧。   
  
2001.9.15   
zombieys\[CCG\]   
  
———————————————————————————————>   
                .-"      "-.      unpacked by zombieys\[CCG\] >   
                /　      　　\\      qq:1789655                >   
              |      ★　　  |    http://zombieys.yeah.net  >   
              |,  .-.  .-.  ,|    http://zombieys.126.com  >   
                |)(\_\_/  \\\_\_)(|      zombieys.cn.hongnet.com  >   
                |/    /\\    \\|                                >   
      (@\_@)    (\_    ^^    \_)      Thanks for your supports  >   
    \_  )\\\_\_\_\_\_\_\_\\\_\_|IIIIII|\_\_/\_\_\_\_\_                          >   
\_)@8@8{}<\_\_\_\_\_\_\_\_|-\\IIIIII/-|\_\_\_\_China Crack Group\_zombieys\_\_\_>  4dd466d0e1e0cc123afbe504da2462ca