
# [原创]看雪CTF2017第10题

## 1. 处理逻辑（大数运算用的gmp）

`sn`长度为70，前6位是`e`，后面的是`p`。

已知`n`、`d`，`p < q`，求`e`、`p`、`q`。

```
n: 6248BC3AB92A33B000FDB88568F19727F92F79EB68FF6AD73203EFD20A3E331BE941C7AA288095F33BC4B255FD983114D480EFFBEE2E313E6218A57F9CCC8189

d: 2476A7F02588913F228923E1F36F963F29708C07B117396817A6B94C336FC77FF7D381925EB40CFED8FBE894570155E41569B4EC69B26CB0320105A29651CB4B
```

## 2. 求解

因为`e < 0x1000000`，所以可以穷举`e`，得到`e: F552B3`。

有了`e`，因为`e`过小，可以直接得到`p`和`q`。

这里借用stackoverflow上的内容：

![](https://bbs.kanxue.com/upload/attach/201706/30337_juyjmipi2ow1llu.png)

## 3. 脚本

```
import itertools

from gmpy2 import *

#e=0xF552B3

n=0x6248BC3AB92A33B000FDB88568F19727F92F79EB68FF6AD73203EFD20A3E331BE941C7AA288095F33BC4B255FD983114D480EFFBEE2E313E6218A57F9CCC8189

d=0x2476A7F02588913F228923E1F36F963F29708C07B117396817A6B94C336FC77FF7D381925EB40CFED8FBE894570155E41569B4EC69B26CB0320105A29651CB4B

def get_e(n,d):

    for i in itertools.count(0xFFFFFF, -1):

        if i <= 2:

            return 0

        e=i

        if not is_prime(e, 500):

            continue

        m=0x12345678

        c=powmod(m,d,n)

        m2=powmod(c,e,n)

        if m == m2:

            return e

    return 0

def get_p_q(e,n,d):

        ed=mul(e,d)

        k1=div(ed, n)

        kk=[k1-1,k1,k1+1]

        for i in range(len(kk)):

                k=kk[i]

                (t, rem)=t_divmod(ed-1,k)

                if (rem != 0):

                    continue

                s = n + (1) - (t)

                r =isqrt(mul(s,s)-mul(4,n))

                p=div(s+r,2)

                q=div(s-r,2)

                if (p > q):

                    p=q

                print('sn: %X%X' % (e,p))

        return

e=get_e(n,d)

print('e: %X' % e)

get_p_q(e,n,d)
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
