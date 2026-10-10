

# IEEE754 浮点数的学习笔记

> 原文：[`www.pediy.com/kssd/pediy10/84132.html`](https://www.pediy.com/kssd/pediy10/84132.html)

IEEE754 浮点数的学习笔记  
  
格式       长度  符号位 指数位  尾数位  有效位数  指数偏移   尾数说明  
单精度     32     1       8       23      24        127       有一位隐含位  
双精度     64     1      11       52      53       1023       有一位隐含位  
扩展双精度 80     1      15       64      64      16383       没有隐含位  
  
注意：扩展双精度格式没有隐含位，因此它的有效位数与尾数位数一致，而单精度和双精度格式均有一位隐含位，因此它们的有效位数比尾数位数多 1。  
  
  
单精度格式浮点数表示范围  
  
             指数     尾数       数值(二进制)           数值(十进制)  
最大数       0xfe     0x7fffff   1.(23 个 1)  \* 2^(+127)  3,4028236692093846346337460743177e+38  
最小数       0x1      0x0        1.(23 个 0)  \* 2^(-126)  1,1754943508222875079687365372222e-38  
最小弱规范数 0x0      0x1        0.(22 个 0)1 \* 2^(-126)  1,4012984643248170709237295832899e-45  
  
注意：弱规范数的隐含位是 0。  
  
  
  
双精度格式浮点数表示范围  
  
             指数     尾数               数值(二进制)             数值(十进制)  
最大数       0x7fe    0xfffffffffffff    1.(52 个 1)  \* 2^(+1023)   1,797693134862315907729305190789e+308  
最小数       0x1      0x0                1.(52 个 0)  \* 2^(-1022)   2,2250738585072013830902327173324e-308  
最小弱规范数 0x0      0x1                0.(51 个 0)1 \* 2^(-1022)   4,9406564584124654417656879286822e-324  
  
注意：弱规范数的隐含位是 0。  
  
  
  
扩展双精度格式浮点数表示范围  
  
             指数     尾数                  数值(二进制)             数值(十进制)  
最大数       0x7ffe   0xffffffffffffffff    1.(63 个 1)  \* 2^(+16383)  1,189731495357231765085759326628e+4932  
最小数       0x1      0x8000000000000000    1.(63 个 0)  \* 2^(-16382)  3,3621031431120935062626778173218e-4932  
最小弱规范数 0x0      0x1                   0.(62 个 0)1 \* 2^(-16382)  3,6451995318824746025284059336194e-4951  
  
注意：扩展双精度格式数没有隐含位，第 63 位(尾数最高位)表示整数位。  
  
  
附：测试代码（vc 6.0 下通过）  

代码:

```
#include <stdio.h>

typedef struct FP\_SINGLE
{
  unsigned \_\_int32 fraction : 23;
  unsigned \_\_int32 exp      : 8;
  unsigned \_\_int32 sign     : 1;
} fp\_single;

typedef struct FP\_DOUBLE
{
  unsigned \_\_int64 fraction : 52;
  unsigned \_\_int64 exp      : 11;
  unsigned \_\_int64 sign     : 1;
} fp\_double;

typedef struct FP\_EX\_DOUBLE
{
  unsigned \_\_int64 fraction;
  unsigned \_\_int32 exp  : 15;
  unsigned \_\_int32 sign : 1;
} fp\_ex\_double;

int main()
{
  float x;
  fp\_single \* fp\_s = (fp\_single \*)&x;

  fp\_s->sign = 0;
  fp\_s->exp = 0xfe;
  fp\_s->fraction = 0x7fffff;
  printf ("float 最大数：      %le\\n",(double)x);

  fp\_s->sign = 0;
  fp\_s->exp = 0x1;
  fp\_s->fraction = 0x0;
  printf ("float 最小数：      %le\\n",(double)x);

  fp\_s->sign = 0;
  fp\_s->exp = 0;
  fp\_s->fraction = 0x1;
  printf ("float 最小弱规范数：%le\\n\\n",(double)x);


  double y;
  fp\_double \* fp\_d = (fp\_double \*)&y;

  fp\_d->sign = 0;
  fp\_d->exp = 0x7fe;
  fp\_d->fraction = 0xfffffffffffff;
  printf ("double 最大数：      %le\\n", y);

  fp\_d->sign = 0;
  fp\_d->exp = 0x1;
  fp\_d->fraction = 0x0;
  printf ("double 最小数：      %le\\n", y);

  fp\_d->sign = 0;
  fp\_d->exp = 0;
  fp\_d->fraction = 0x1;
  printf ("double 最小弱规范数：%le\\n\\n", y);


  char ch\[10\];

  fp\_ex\_double \* fp\_ex\_d = (fp\_ex\_double \*)ch;
  fp\_ex\_d->sign = 0;
  fp\_ex\_d->exp = 0x7ffe;
  fp\_ex\_d->fraction = 0xffffffffffffffff;  
                      
// 不知道扩展双精度浮点数如何输出，
// 不过可以用 od 跟踪，然后找到 ch\[0\]的地址，在数据窗口中选择 浮点 80 为长双精度，
// 就可以看到数值了。
  fp\_ex\_d->sign = 0;
  fp\_ex\_d->exp = 0x1;
  fp\_ex\_d->fraction = 0x8000000000000000;      
                     
  fp\_ex\_d->sign = 0;
  fp\_ex\_d->exp = 0;
  fp\_ex\_d->fraction = 0x1;      

  return 0;
}

```