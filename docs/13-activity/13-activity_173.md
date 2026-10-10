

# 第一题

> 原文：[`www.pediy.com/kssd/pediy10/77640.html`](https://www.pediy.com/kssd/pediy10/77640.html)

很多题看不明白，还是先从第一题开始吧  
我得计算过程是从 0 开始的到输入的数的统计，即\[0, ?x\]  
比如 输入 15  
输出  
\[0\]2  \[1\]8  \[2\]2  \[3\]2  \[4\]2  \[5\]2  \[6\]1  \[7\]1  \[8\]1  \[9\]1  
  
欢迎大家指正![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")  
  

代码:

```
// DigitCount.cpp : Defines the entry point for the console application.
//

//#include "stdafx.h"

#include <iostream>
#include <fstream>

using std::cout;
using std::cin;
using std::endl;
using std::ifstream;
using std::ostream;
using std::ofstream;


//static int digits\[10\] = {0};

void ShowResult(const int digits\[10\])
{
  for (int i = 0; i < 10; ++i)
  {
    cout<<'\['<<i<<'\]'<<digits\[i\]<<'\\t';
  }
  cout<<endl;

}



void Test(unsigned value, int digits\[10\])
{
  // init digits
  memset(digits, 0, sizeof(digits));
  digits\[0\] = 1;

  unsigned pos = 0, ratio = 1;
  unsigned curDigit = value % 10, head = value / 10, tail = 0;

  //do
  while (head > 0)
  {

    // calculate current digit and '0' digit
    if (curDigit == 0)
    {
      digits\[0\] += (head - 1) \* ratio + (tail + 1);
    }
    else
    {
      digits\[0\] += head \* ratio;
      digits\[curDigit\] += head \* ratio + (tail + 1);
    }

    // calculate y < curDigit
    for (unsigned i = 1; i < curDigit; ++i)
    {
      digits\[i\] += (head + 1) \* ratio;
    }

    // calculate y > curDigit
    for (unsigned i = curDigit + 1; i <= 9; ++i)
    {
      digits\[i\] += head \* ratio;
    }

    // update tail, ratio, curDigit, head
    tail += ratio \* curDigit;
    ratio \*= 10;
    curDigit = head  % 10;
    head /= 10;
  }

  // for curDigit == 0
  // do noting

  // for 0 < y < curDigit
  for (unsigned i = 1; i < curDigit; ++i)
  {
    digits\[i\] += ratio;
  }

  // for y = curDigit
  digits\[curDigit\] += tail + 1;

  // for y > curDigit
  // do nothing

}

void PrintDigits(int digits\[10\], ostream& os)
{
  for (int i = 0; i < 10; ++i)
  {
    os<<'\['<<i<<'\]'<<digits\[i\]<<'\\t';
  }

  os<<endl;

}

int main(int argc, char\* argv\[\])
{

  int digits\[10\] = {0};

  const char\* INPUTFILE = "in.txt";
  const char\* OUTPUTFILE = "out.txt";

  ifstream is(INPUTFILE);
  ofstream os(OUTPUTFILE);
  unsigned value = 0;
  int testCount = 0;
  is>>testCount;
  

  while (--testCount > 0)
  {
    is>>value;
    Test(value, digits);
    //Debug\_Test(value, digits);

    PrintDigits(digits, os);
    
    // print result in console
    //PrintDigits(digits, cout);

  }

  os.close();
  is.close();
  
  return 0;
}

```

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || countDigit.rar  
 |