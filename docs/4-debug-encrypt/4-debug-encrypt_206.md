

# SWF Browser

> 原文：[`www.pediy.com/kssd/pediy04/kanxue169.htm`](https://www.pediy.com/kssd/pediy04/kanxue169.htm)

SWF Browser   
      由于这个软件用了 BlowFish 算法，所以，详细的跟踪过程不好写出来。有兴趣的朋友可以去查看我   
写的《BlowFish 算法说明》（看雪的教学里头有）。这里只列出它的注册码算法：   
      1.把 name 的长度扩长为 8 个子节的倍数，并且 name 的长度要大于 3。   
        扩长的方法是在名字后头添加 8-(strlen(name)%8)   
      2.用 BlowFish 算法对 name 进行加密。key="1232hfbsdjdh2834121"   
      3.把输出级联，做一个字符变换得到 string1   
      4.把 string1 按照扩长名字同样的方法扩长到 8 个子节的倍数   
      5.用 BlowFish 算法对 string1 进行加密。key="ewrwk214134g7df2"   
      6.把输出级联，通过和 3 同样的字符变换得到 string2   
      7.把 string2 经过另一个变换，得到注册码。   
      这个软件算法已经算是比较复杂的了，不过可笑的是最后竟然出现明码对比。   
      所以如果你只想要它的注册码，是很 easy 的事情，可是，如果要编出注册机（非 crackcode 型的），   
没有一点密码学的功底，还是很不容易的。有兴趣的朋友可以试试看。  4dd466d0e1e0cc123afbe504da2462ca