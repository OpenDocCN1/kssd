

# 密码学入门系列(四) 之 维吉尼亚密码(古典)

> 原文：[`www.pediy.com/kssd/pediy10/89667.html`](https://www.pediy.com/kssd/pediy10/89667.html)

【文章标题】: 密码学入门系列(四) 之 维吉尼亚密码(古典)  
【文章作者】: jackozoo  
【作者邮箱】: jackozoo@163.com  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
  **系列声明**: 见一.  
    
  维吉尼亚密码(Vige nere Cipher)简介:  
  大家熟知, 早前的单一的凯撒密码保密性能是十分的差的, 因此人们在单一的凯撒密码的基础上扩展出了多表密码, 称之为维吉尼亚密码. 它是由 16 世纪法国亨利三世王朝的不莱塞-维吉尼亚发明的. 关于维吉尼亚的密码的历史我这里就不做多的介绍了, 有兴趣的朋友可以在互联网上搜索相关文献查看.  
  维吉尼亚密码的特点是将 26 个凯撒密码表(也即我们常用的 Z26)合成一个表.  
    
  如下:  
    
  A B C D E F G H I J K L M N O P Q R S T U V W X Y Z   
  B C D E F G H I J K L M N O P Q R S T U V W X Y Z A   
  C D E F G H I J K L M N O P Q R S T U V W X Y Z A B   
  D E F G H I J K L M N O P Q R S T U V W X Y Z A B C   
  E F G H I J K L M N O P Q R S T U V W X Y Z A B C D   
  F G H I J K L M N O P Q R S T U V W X Y Z A B C D E   
  G H I J K L M N O P Q R S T U V W X Y Z A B C D E F   
  H I J K L M N O P Q R S T U V W X Y Z A B C D E F G   
  I J K L M N O P Q R S T U V W X Y Z A B C D E F G H   
  J K L M N O P Q R S T U V W X Y Z A B C D E F G H I   
  K L M N O P Q R S T U V W X Y Z A B C D E F G H I J   
  L M N O P Q R S T U V W X Y Z A B C D E F G H I J K   
  M N O P Q R S T U V W X Y Z A B C D E F G H I J K L   
  N O P Q R S T U V W X Y Z A B C D E F G H I J K L M   
  O P Q R S T U V W X Y Z A B C D E F G H I J K L M N   
  P Q R S T U V W X Y Z A B C D E F G H I J K L M N O   
  Q R S T U V W X Y Z A B C D E F G H I J K L M N O P   
  R S T U V W X Y Z A B C D E F G H I J K L M N O P Q   
  S T U V W X Y Z A B C D E F G H I J K L M N O P Q R   
  T U V W X Y Z A B C D E F G H I J K L M N O P Q R S   
  U V W X Y Z A B C D E F G H I J K L M N O P Q R S T   
  V W X Y Z A B C D E F G H I J K L M N O P Q R S T U   
  W X Y Z A B C D E F G H I J K L M N O P Q R S T U V   
  X Y Z A B C D E F G H I J K L M N O P Q R S T U V W   
  Y Z A B C D E F G H I J K L M N O P Q R S T U V W X   
  Z A B C D E F G H I J K L M N O P Q R S T U V W X Y   
    
    
  这样我们就从传统的单表替换扩展到了多表替换了.    
    
  引入一个概念:秘匙  
    
  在前面的凯撒移位密码和仿射密码中, 我们同样引入了秘匙的概念, 不过那里的秘匙只是一个或两个数字. 而在这里的秘匙是一个更为广义上的秘匙.  
  它是一个串. 不在局限于一个或两个整数.  
    
维吉尼亚密码的原理:  
    
  比如明文为 JACKOZOO, 秘匙为 LOVE, 则我们的密文是这样得到的:  
    
    
  J 对应的密文我们查秘匙为 L, 则在第 L 行中, 找到与第一行中的 J 对应的字母为 U.  
  A 对应的密文我们查秘匙为 O, 则在第 O 行中, 找到与第一行中的 A 对应的字母为 O.  
  C 对应的密文我们查秘匙为 V, 则在第 V 行中, 找到与第一行中的 C 对应的字母为 X.  
  K 对应的密文我们查秘匙为 E, 则在第 E 行中, 找到与第一行中的 K 对应的字母为 O.  
  O 对应的密文我们查秘匙为 L, 则在第 L 行中, 找到与第一行中的 O 对应的字母为 Z. (如果秘匙不够了,我们就循环使用秘匙, LOVELOVELO ... )  
  Z 对应的密文我们查秘匙为 O, 则在第 O 行中, 找到与第一行中的 Z 对应的字母为 N.  
  O 对应的密文我们查秘匙为 V, 则在第 V 行中, 找到与第一行中的 O 对应的字母为 J.  
  O 对应的密文我们查秘匙为 E, 则在第 E 行中, 找到与第一行中的 O 对应的字母为 S.  
    
  由此得到 JACKOZOO 在以 LOVE 作为秘匙的情况下, 其密文为: UOXOZNJS.  
    
    
  我们看到维吉尼亚密码也很好地隐藏了字频信息.  
  但是这里不得不提的是, 维吉尼亚密码(Vigenere Cipher)相对于希尔密码(Hill Cipher)来说, 其对字频信息的隐藏还不够彻底.  
  这也导致了在 19 世纪 50 年代, 英国人查尔斯-巴贝奇对其的破解. 其实其破解的基本思想如下:  
  比如在密文中, 经常出现了同一个子串(比如 UPK), 而且每个字串之间的距离都是 3 的整数倍. 那么解密者就很容易推测出秘匙的长度为 3. 其原因也是十分简单的:  
  当秘匙在重复了 N 次之后, 其还是用第一个字母去加密 UPK 相应的明文. 尤其是对 THE, YOU, WHAT 这类高频词汇当使用了弱秘匙的话,更容易遭受破解. 关于维吉  
  尼亚密码的具体破解工作, 不在本文范围之内, 有兴趣的朋友可以自行研究.  
    
    
  以上介绍的是维吉尼亚密码的加密, 解密的话, 直接找出相应行在第一行中对应的字母即可.  
    
    
  通过对维吉尼亚密码原理的学习, 我们不难发现用计算机语言写出维吉尼亚的算法也是很简单的一件事情.  
    
  **加密:**  
    
  

代码:

```

        nMLen = strlen(szM);
        nKLen = strlen(szK);
  
        for (i=0;i<nMLen;i++)
        {
          szL\[i\] = ((szM\[i\]-'A') + (szK\[i%nKLen\]-'A')) % 26 + 'A';                        
        }

```

  
  
**解密:**  
代码:

```

        nLLen = strlen(szL);
        nKLen = strlen(szK);
  
        for (i=0;i<nLLen;i++)
        {
          szM\[i\] = ((szL\[i\]-'A') + (szK\[i%nKLen\]-'A')) % 26 + 'A';
          if (szM\[i\] < 'A')
          {
            szM\[i\] += 26;
          }
        }
  

```

  
  附件中为维吉尼亚密码学习辅助软件. 希望大家用的上. ::)  
  
  
  
  
  
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2009 年 05 月 23 日 PM 04:39:14

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || Vigenere\_src.rar  
 |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || Vigenere\_bin.rar  
 |