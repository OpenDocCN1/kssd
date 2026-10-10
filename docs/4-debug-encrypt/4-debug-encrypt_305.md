

# 唠一唠 mysql 的口令算法

> 原文：[`www.pediy.com/kssd/pediy11/108627.html`](https://www.pediy.com/kssd/pediy11/108627.html)

Author: kyo327  
email: humour327#hotmail.com  
Date: 2010-03-10  
  
大家都知道 mysql 口令的加密分两种。一种是古董级别的 mysql323 加密，另一种是蒙骗了大众的的 mysql sha1 加密。  
虽然现在遇到这样的加密可以用 cain 来破解，按说我不用在这里多此一举，但本文不是为了破解而写，只是科普一下，让大家更清楚的了解 mysql 口令的加密方法到底是怎样的。  
  
先说 mysql4.1.x 版本以后的：  

代码:

```
mysql> select password('kyo327');
+-------------------------------------------+
| password('kyo327') |
+-------------------------------------------+
| \*B19CB640DF626A73397BFBBB8111D11E2BEC11F1 |
+-------------------------------------------+
1 row in set (0.02 sec)

mysql> select sha1('kyo327');
+------------------------------------------+
| sha1('kyo327') |
+------------------------------------------+
| d81e2a1a72f5a4f570b0f05baaa0cffc5d2d6820 |
+------------------------------------------+
1 row in set (0.03 sec)
  

```

  
很明显不是标准的 sha1 加密。然而这个问题像是皇帝的新装一样，好像没有哪个人指出来，那我就在这里当一次无知的小孩吧。 再看下面  
代码:

```


mysql> select sha1(0xd81e2a1a72f5a4f570b0f05baaa0cffc5d2d6820);
+--------------------------------------------------+
| sha1(0xd81e2a1a72f5a4f570b0f05baaa0cffc5d2d6820) |
+--------------------------------------------------+
| b19cb640df626a73397bfbbb8111d11e2bec11f1 |
+--------------------------------------------------+
1 row in set (0.00 sec)
 

```

  
我想无需用过多言语，大家应该都很清楚 mysql4.1.x 以后版本的加密方式了吧。  
  
  
接下来说一说 mysql323 加密。   
  
这个我是直接从 mysql 源代码里抠出来的。代码如下：  
代码:

```


#include "stdio.h"
#include <string.h>

void hash\_password(long \*result, const char \*password, int password\_len)
{
register long nr=1345345333L, add=7, nr2=0x12345671L;
long tmp;
const char \*password\_end= password + password\_len;
for (; password < password\_end; password++)
{
if (\*password == ' ' || \*password == '\\t')
continue; /\* skip space in password \*/
tmp= (long) (char) \*password;
nr^= (((nr & 63)+add)\*tmp)+ (nr << 8);
nr2+=(nr2 << 8) ^ nr;
add+=tmp;
}
result\[0\]=nr & (((long) 1L << 31) -1L); /\* Don't use sign bit (str2int) \*/;
result\[1\]=nr2 & (((long) 1L << 31) -1L);
}

int main(int argc, char\* argv\[\])
{
const char \*password="kyo";
long hash\_res\[2\];
hash\_password(hash\_res, password, (int) strlen(password));
printf("%08lx%08lx\\n", hash\_res\[0\], hash\_res\[1\]);
return 0;
}
 

```

  
运行后  
7901b47128d1045d  
代码:

```
mysql> select old\_password('kyo');
+---------------------+
| old\_password('kyo') |
+---------------------+
| 7901b47128d1045d |
+---------------------+
1 row in set (0.00 sec)
 

```

  
  
over。  
想做 mysql 密码爆力破解工具的，根据本文应该很容易写出来了。