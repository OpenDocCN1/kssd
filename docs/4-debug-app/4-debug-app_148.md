

# 金山词霸

> 原文：[`www.pediy.com/kssd/pediy03/forum534.htm`](https://www.pediy.com/kssd/pediy03/forum534.htm)

小弟用 Crack 软件同样的方法知道了 Acrobat Read 不让词霸在其 4.05 版本中取词的地方, 因此 patch 了它并专门做了一个网站供大家下载, 有兴趣的朋友可去看看:   
  
http://acropat.home.chinaren.com   
  
方法简单的不好意思说:   
1\. 在运行 Acrobat Reader 前下断 bpx CreateFileA do "d esp->4"   
直到显示出正在打开 xdict32.api.   
  
2\. 再下断:   
bpx ReadFile   
bpx ReadFileEx   
经几次 F12 后 发现有   
or eax, eax   
jnz xxxxx   
就在这儿改!   
  
其实 ICE 除了破解, 用它来发现并改变软件的行为是大有可为的. 我类似使用 ICE 的例子绝不只这一个.  4dd466d0e1e0cc123afbe504da2462ca