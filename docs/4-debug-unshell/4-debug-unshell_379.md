

# ASProtect1.2

> 原文：[`www.pediy.com/kssd/pediy04/kanxue015.htm`](https://www.pediy.com/kssd/pediy04/kanxue015.htm)

脱 ASProtect1.2 的壳   
  
  
    这次的目标是 ASProtect1.2 本身。   
  
unpack(2001.9.11)   
  
    还是利用快速寻找 aspr 入口的方法：   
执行 ASProtect.exe，用 prodump 选 dump(full)脱壳,存为 dump.exe。接着用 winhex 打开 dump.exe，选择搜索文本，填 runtime,执行搜索，搜到后,向前找到离 runtime 最近的机器码为 55 8B EC 的地方就是程序的 oep，而在 ASProtect 里这个位置在 offset:000585D8 处，用 peditor 的 flc 功能将它转换为 virtual address 就是 458FD8----oep。   
  
    ok，下面要在入口处脱壳，因为直接 dump 的不管怎么折腾就是不让运行，所以老规矩打开 SuperBPM，点 erase，用 trw 载入 ASProtect，下 g 458FD8，下 suspend。用 prodump 选 ASProtect 进程 dump(full)，再打 ctrl+n，f5。   
  
    接着修复 import table   
  
为什么要用 ASProtect1.2 举例呢?因为 ASProtect1.2 是最好脱的 aspr 保护的东东呀。ImportREC 完全可以自动找出完整的 it，无需手动修改，原因么我想就是 ImportREC 的作者当初就是用的 ASProtect 本身为基础做 ImportREC 的(猜测而已)，怪不得除了这个其他的都要手动修复。只要在 oep 处填 00058FD8，剩下的就交给 ImportREC 搞掂。最后选 add new section，然后点 fix dump。   
  
    运行程序，报 112c6d8 地址错误，我倒~~~~~~~~。load 吧，三件套 icedump+superbpm+s-ice 或 trw，看看在哪出的错，最后一步一步找到   
45495f ff15ac4f4600->90    call \[464fac\]->nop   
而 464fac 指向的地址就是 112c6d8，下 dd 或 u 112c6d8(方式不一样，其实效果一样),在 112c6d8 处及周围全都是??，也就是说这块内存不归 aspr 管(不知道着么说是否合适)，再用原版从来一次看看 112c6d8 是什么，一个 ret,昏倒中~~~。直接 nop 掉好啦。再运行还报 112c6d8 地址错误。这次是   
454b3a ff15b04f4600->90 call \[464fb0\]->nop   
统统 nop 啦，不过理论上能补成其他的尽量不用 nop，为了快出结果，懒的想更多啦   
哦，这次正常运行了，点 about，报 112c6d8 地址错误，！·#￥%……—\*（）—+...........，一不做，二不休，掏出 winhex 替换所有 ff15ac4f4600 和 ff15b04f4600 为 nop，其实这个一共有 3 处，第 3 处报错是在压缩的时候。   
  
    至此脱壳完成可以正常运行了，本想汉化一下的，做了不到一半就放弃了，麻烦死我了，宏伟兄在吗?交给你啦~~~~~~~~~~~~~~~\`\`   
  
    补充点 aspr 注册的内容，aspr 的验证是在外壳部分进行的，我想脱掉后就应该是注册版了吧?把 unregistered version（这个可是手打的哦，看得太多了，打起来比打我的 id 还熟呢;-)）改成你的 id 就可以了。我对 aspr 研究不多，不确定它的可靠性，请知道的讲讲，多谢。   
  
另外我发现这个快速搜索 aspr 保护程序 oep 的方法对 azpr3.52 无效，看来这个方法还有一定的局限性，不过 azpr3.52 可以用 caspr 脱掉能省则省么;-)。azpr 的 ep 是 401000   
  
(2001.9.12)   
暴破：4172FF E9F8000000->E9F3000000 JMP 4173FC->JMP 4173F7   
  
完活   
  
感谢看完!   
  
2001.9.12   
zombieys\[CCG\]   
  
———————————————————————————————>   
                .-"      "-.      Cracked by zombieys\[CCG\]  >   
                /　　　　　　\\      qq:1789655                >   
              |              |    http://zombieys.yeah.net  >   
              |,  .-.  .-.  ,|    http://zombieys.126.com  >   
                |)(\_\_/  \\\_\_)(|                                >   
                |/    /\\    \\|      Thanks for your supports  >   
      (@\_@)    (\_    ^^    \_)                                >   
    \_  )\\\_\_\_\_\_\_\_\\\_\_|IIIIII|\_\_/\_\_\_\_\_                          >   
\_)@8@8{}<\_\_\_\_\_\_\_\_|-\\IIIIII/-|\_\_\_\_China Crack Group\_zombieys\_\_\_>  4dd466d0e1e0cc123afbe504da2462ca