

# ASPack 2.11

> 原文：[`www.pediy.com/kssd/pediy04/kanxue029.htm`](https://www.pediy.com/kssd/pediy04/kanxue029.htm)

脱 ASPack 2.11 的壳   
  
  
unpack(2001.9.13)   
  
    还是利用高速寻找 aspr 入口的方法(适用于 delphi 写的程序)：   
执行 ASPack.exe，用 prodump 选 dump(full)脱壳,存为 dump.exe。接着用 winhex 打开 dump.exe，选择搜索文本，填 runtime,执行搜索，搜到后,向前找到离 runtime 最近的机器码为 55 8B EC 的地方就是程序的 oep，而在 ASPack 里这个位置在 offset:00041FEC 处，用 peditor 的 flc 功能将它转换为 virtual address 就是 004429EC----oep。   
  
    ok，下面要在入口处脱壳，因为直接 dump 的不管怎么折腾就是不让运行，所以老规矩打开 SuperBPM，点 erase，用 trw 载入 ASPack，下 g 458FD8，下 suspend。用 prodump 选 ASPack 进程 dump(full)，再打 ctrl+n，f5。   
  
    接着修复 import table   
  
1    000482D4    KERNEL32.dll    00C7    CreateProcessA   
  
最后选 add new section，然后点 fix dump。用 peditor 修改 ep 为 000429EC。   
  
    运行程序，报 110c54c 地址错误，不报错的还没有见过呢。load 吧，icedump+superbpm+s-ice 或 superbpm+trw 组合，看看在哪出的错，最后一步一步找到   
  
:00442A65 FF157C614400            call dword ptr \[0044617C\]   
  
用原版载入发现实际上是 call 4428f0，修改一下即可。   
再次运行报 110c4a8 地址错误，再用原版跟   
  
43f2c8 ff1574614400->90    call \[446174\] ->nop   
43f356 ff1518614400->90    call \[446174\] ->nop   
  
两个 call 的内容其实是一个 ret，无关紧要，nop 掉。   
再用 winhex 替换所有 ff1574614400 和 ff1518614400，总共 4 处。   
  
至此程序可正常运行，脱壳结束。   
  
脱壳后仍有注册限制，看来前面脱的 aspr1.2 也一样是未注册版了。   
  
  
  
感谢看完!   
  
2001.9.13   
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