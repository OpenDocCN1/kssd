

# 【代码之美】题 1

> 原文：[`www.pediy.com/kssd/pediy10/73955.html`](https://www.pediy.com/kssd/pediy10/73955.html)

在《计算机算法设计与分析（第 3 版）》第一章有一个和该题非常类似的题。  
作者推荐的算法是：  
考察由 0-9 组成的 n 位数，从 n 个 0 到 n 个 9，0~9 出现的次数都是相同的，容易得知次数 f(n)为  

代码:

```
f(n)=10f(n-1) + 10^(n-1)  n > 1
f(n)=1                    n = 1
即 f(n)=n10^(n-1)

```

如对 000-999，各数字数目是相等的，有排列组合知。  
所以就有如下的解法：  
设数为 12345  
可分解为  
代码:

```
    0 --  9999 这里得减去一些多余的 0
10000 -- 10999
11000 -- 11999
12000 -- 12099
12100 -- 12199
12200 -- 12299
12300 -- 12309
12310 -- 12319
12320 -- 12329
12330 -- 12339
12340 -- 12345

```

分段求解，但是编写代码之后才发现性能很低。所以就不贴代码了。  
  
贴一段收藏的代码,不知道这算不算，按每位进行的计算，循环的次数是确定的，而上面的算法循环次数随数据影响很大  
代码:

```
#include<fstream>
using namespace std;

unsigned long countArray\[10\] = {0};

void Count( unsigned long num = 0 )
{  
  if( num<0 ) return;  
  
  char number\[11\] = {0};  
  sprintf( number, "%d", num );
  
  int i;
  for (i=0; i<10; ++i)
    countArray\[i\] = 0;
  
  int cursor = 0;
  int numberCount = 0;  
  while( number\[cursor\] != '\\0' )
  {
    // Add one digital at the end, the count of the number will be about 10 double. 
    // So the count of left digital will be approximately 10 times.    
    for(i=0; i<10; ++i )
      countArray\[i\] \*= 10;
    
    // But not all      
    // For 2=>23, '1' 10 double, '2' only 3+1,
    // For 2=>29, '2' will 10 dobule too.
    for( i=0; i<cursor; ++i )
      countArray\[ number\[i\]-'0' \] -= '9' - number\[cursor\];
    
    // There are some new digital for the last digit.
    // Add one for each digital.
    for( i=0; i<10; ++i )
      countArray\[i\] += numberCount;
    
    // Not all again.
    for(i=1; i<=( number\[ cursor \] - '0' ); ++i)
      ++countArray\[i\];
    
    // Get the count for previous part.
    numberCount \*= 10;
    numberCount += number\[ cursor \] - '0';
    
    ++cursor;
  }
  
  ++countArray\[0\];
}

void main(void)
{  
  ifstream fin("in.txt");
  ofstream fout("out.txt");
  
  int testCount;
  fin>>testCount;
  while(--testCount)
  {    
    unsigned long num;
    fin>>num;  
    Count(num);
    
    //Output
    fout<<countArray\[0\];
    for(int i=1; i<10; i++ )
      fout<<","<<countArray\[i\];
    fout<<endl;
  }  
}

```

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || 1.rar  
 |