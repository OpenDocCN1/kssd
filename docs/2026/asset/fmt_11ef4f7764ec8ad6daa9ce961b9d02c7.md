
是动态的，地址是随时可变的，也就是说做出的内存注册机不能通用，只能用一次。因此爆破点应选在`poppages.dll`文件中（`w32dasm`和`hiew`中的实际地址是`10050BD1`）。还需要指出的是在写注册机的时候，以上`01ED0B5D`、`01ED0B81`、`01ED0B94`中`0X7E000`、`0X780000`和`0X7F800000`中的0必须写完整，还有必须用`unsigned long`（无符号，长整数）定义变量，否则不能得出正确结果。另外在`01ED0BD3`至`01ED0BD9`处的循环可以不用考虑。这就是在【声明】中我提到的修改了的注册机算法。

**提供一组注册码**

- 注册项目：3DLINK
- CODE ENTRY：274925803
- COMPUTER ID：481920
- SN：8399413

---

## 算法注册机

```c
/* 3Dlink 1.6 sp2算法注册机 */
/* 在Turboc 2.0 下调试通过  */
/* 感谢Shooo提供算法思路       */
#include "stdio.h"

main()

{ unsigned long code,id;

  unsigned long i,j,k,l,m;

  printf("3DLINK 1.6 SP2 Register by jackily 2004-12-11\n");

  printf("Email:jackily_zhang@msn.com or jackily_zhang@yahoo.com.cn\n");

  printf("please input code entry:");

  scanf("%lu",&code);

  printf("\nplease input computer id:");

  scanf("%lu",&id);

   m=id;

  i=j=k=code;

  i&=0x7e000;i>>=0x0d;i+=0x7bc;i*=0x44;

  j&=0x1f00;j>>=0x08;j*=0xf3;

  i+=j;

  j=code;

  j&=0x780000;j>>=0x13;j*=0x108;

  l=i+j;

  i=code;

  i&=0x7f800000;i>>=0xf;

  k&=0xff;

  i+=k;

  i*=0x3;

  j=0x69964*0x7;

  i+=j;

  m*=2;

  i+=m;

  m=l;m*=0x1f;

  m+=i;

  l=m;

  l&=0x7fffffff;

  printf("\nserial number is %lu",l);

  }
```

---

## 版权声明

本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!2004-12-11
