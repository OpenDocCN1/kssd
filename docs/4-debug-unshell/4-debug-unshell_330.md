

# aspr 脱壳总结

> 原文：[`www.pediy.com/kssd/pediy04/kanxue027.htm`](https://www.pediy.com/kssd/pediy04/kanxue027.htm)

aspr 脱壳总结(部分适用于其他壳保护)   
  
    首先，能用 caspr 脱的尽量用它吧，方便，比手动脱的要小，caspr 解决不了的那就自己来吧。   
    脱 aspr 壳基本上分 4 步，如下：   
  
1.寻找入口:   
  
(1)delphi:   
    快速寻找入口的方法:执行程序，用 prodump 选 dump(full)脱壳,存为 dump.exe。接着用 winhex 打开 dump.exe，选择搜索文本，填 runtime,执行搜索，搜到后,向前找到离 runtime 最近的机器码为 55 8B EC 的地方就是程序的 oep，所以 delphi 程序的 oep 最好找了。所以用 aspr 保护 delphi 程序的话会使 aspr 的保护能力减弱一半，入口太好找了。不过 delphi 程序的 crc 保护强悍，胜过 aspr 保护包括 winhex9.x，wincmd4.54，tag&rename1.9x，iptools1.xx,host-mon1.3x，其中后两个我还没有搞定，谁知道的话麻烦告诉我一声。   
  
脱壳详情请参考我最近写的   
  
注册 Tag&Rename v.1.9.6   
注册 Tag&Rename v2.0 beta 1   
注册 Advanced Ra-Renamer v.1.2   
脱 ASProtect1.2 的壳   
脱 ASPack 2.11 的壳   
  
(2)vb：   
    快速寻找入口的方法:执行程序，用 prodump 选 dump(full)脱壳,存为 dump.exe。接着用 winhex 打开 dump.exe，选择搜索文本，填程序的名字,执行搜索，搜到后,向前找到离名字最近的机器码为 68(后面就不一定了，一般很短，不超过 20h 字节)开头的地方就是程序的 oep，所以 vb 程序的 oep 也比较好找。只是用 asp express 压了一下看看，没做更多测试，不好多说啦。   
  
  
(3)vc:   
    无任何规律，慢慢跟吧:(。手动脱壳方法最好熟练掌握，对付任何程序都应该有效吧:)，熟悉以后就没问题了。   
  
脱壳详情请参考我刚写的写的   
  
脱壳 IglooFTP PRO v3.0 的详细过程   
  
(4)其他:   
    没试过，不知道了。   
  
2.脱壳：   
  
    最好要从入口处 dump，如果不是从入口 dump 的，一般不能用。再有用 prodump dump 的程序，需要手动把 op 改成正确的。用天意,trw 或 peditor dump 的程序稍大。其中天意在我的 winme 系统下用 suspend 死机，所以极少用，peditor dump 出的程序超大，trw 选在入口 dump 的可以免去手动改 ep 的麻烦，好像跟 prodump 脱出来的没区别，就是大了一点。喜欢用哪个看着办吧，caspr 脱出来的最小，还有一个叫 rad\_v06 的软件，没试过不太清楚了。再有其他的我就不知道了。   
  
3.重建 import table：   
  
    慢工，运气好的话 ImportREC 自己就能搞定，一般都有一部分需要自己找。需要的专业知识太多了，这方面是我的弱项呀。   
  
4.脱壳后正常运行：   
  
    具体情况具体分析，多数情况下不能正常运行，一般是地址错误引起的，需要和未脱壳的版比较，再修改。   
  
    这里回复一下大师们前面的回贴。hying 大哥的回贴，我在这次的"1.寻找入口:"已作了解释。blowfish 大虾的回贴，说实话看不明白，是由于我的相关知识太贫乏了吧，慢慢学不着急啦。   
    最后再谈一下 aspack 的壳，这个用 ti 找到入口，在入口处 dump，重建 import table 一般就可正常运行了。   
  
脱壳详情请参考我最近更新的的   
  
注册你的 Windows Commander 4.54   
  
    最后还要感谢 hying 大哥帮我脱 Tag&Rename v.1.9.6 的壳才使我下决心突击学习 aspr 脱壳!   
    感谢脱壳前辈们提供所有相关资料!   
    感谢各位 toye 论坛上的 cracker 们无私提供 crack 资料!   
    感谢 toye 院长将 crack 资料一一整理，提供如此方便的快捷的 crack 交流论坛及为 crack 事业做出贡献!   
  
全文完   
至此学习 aspr 脱壳告一段落，该休息一下了(休息是指换换别的东西学，要学的东西太多啦，真的受不了了呀~~~~~~~~~~~~~~~~\`)   
  
2001.9.14   
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