

# RegSnap2.80

> 原文：[`www.pediy.com/kssd/pediy03/forum890.htm`](https://www.pediy.com/kssd/pediy03/forum890.htm)

RegSnap2.80 注册表比较工具。   
该工具功能非常不错，本身也属于 Crack 软件,但它要注册并有三十天的时间   
限制，真令人讨厌。   
  
    网上已有不少 Crack 高手，破解了它，但我不知他们有没有试过这软件，   
它还隐含了一个陷阱，破解过它的人都知道，首先是把文件位置 0x41F9 处 7409   
改为 EB09，即原有的条件跳转语句改为无条件跳转语句，以及 0x421b 处 74DE   
改为 9090，同时把程序检查完整性的地方改掉。   
  
  但不知这些高手们是否测试过程序，或者说他们也太忙了，经过这样改过的   
程序，RegSnap 会在做比较时始终会说 Broken Source file.然后就停止工作。   
  
  我曾试过找到此处也进行修改，但发现又有问题，于是我干脆对程序不作修改，   
而采用 ppatcher 来处理，我把 ppatcher.ppc 文件进行一下配置：   
  
#Process Patcher Configuration File   
Version=3.93   
  
DisplayName=My Test Program   
Filename=regsnap.exe   
Filesize=299008   
Arguments=/quiet   
Address=0x4041f9:0x74:0xEB   
Address=0x40421b:0x74:0x90   
Address=0x40421c:0xde:0x90   
  
  
#End of Configuration File   
  
然后把 ppatcher.exe 和 ppatcher.ppc 拷贝到 Regsnap 执行目录下，然后运行   
ppatcher.exe，^\_^，一切正常，运行比较功能，也正常。   
  
当然这时显示的注册信息好恐怖哦。。。。   
  
有时明明可以用另外一种办法避免的麻烦事，何不去试试呢。   
  
我要偷懒，我要偷懒。。。。。  4dd466d0e1e0cc123afbe504da2462ca