
# 刘涛涛的面试题

> 原文：[`www.pediy.com/kssd/pediy11/119390.html`](https://www.pediy.com/kssd/pediy11/119390.html)

想起被`W`哥拒绝和被`S`哥无情地忽视，真是很伤心![](img/9b2aec7f3d4bb041b2d419a59832192f.jpg "恐怖!")

既学不成国际一流的做壳技术，也学不成国际一流的解密技术，世上还有什么事能让人快乐呢？![](img/b0e6af7069663aededb11e4e29a3e6d1.jpg "碌眼")从此之后，我便郁郁寡欢，整日在网上闲逛。

一日，我无意间访问到我国著名程序员`刘涛涛`的`blog`，逐一阅读其博文之后，深感佩服，让我印象最深的要数“扭曲变换加密”与“哈巴雪山－帐篷事件”，两篇文章让人感受到前辈技术和人格上的魅力。除此之外，我还在另外一篇文章中看到了他出的一道面试题目，引用原文片段如下

> 引用: 我出的一道测试题，在此公布，没有版权，随便转抄：有一个字符串，里面包含一些数字，写一个函数，把这些数字加起来。比如“我 30 你 40 他 50”结果就是 120。就是这么一道题，来面试程序员的本科毕业生，有超过 60%做不出来！

现在的大学生，真是不得了。就这水平也敢来应聘程序员。有时我就问他们，大学四年都干什么了，读书了？没有。学专业了？没有。泡女朋友了？没有。这就奇怪了，你大学这几年到底干了点什么呢？原来是打电脑游戏了！

看了`刘涛涛`前辈的话，我感到压力很大，在大学这几年，每天早上我都用父母给的钱在学校吃一根油条两个鸡蛋，现在我发现居然什么都没在大学里学到，书没读过几本，专业课一塌糊涂，女朋友也没谈过，最悲剧的是，电脑游戏我也很少玩![](img/b0e6af7069663aededb11e4e29a3e6d1.jpg "碌眼")

为了毕业以后仍然能每天早上吃上一根油条两个鸡蛋，我决定从解此题开始努力学习，在知识的海洋里奋勇遨游了十几个来回之后，我总算是做出了这道题目的基本解法![](img/6821a6175013046370981b6035d43ab4.jpg "使眼色")

代码:

```c
#include<stdio.h>
#include<string.h>
#include<stdlib.h>

void mystrrev(unsigned char* s);
void add(char *s1,const char *s2);

int main(int argc,char *argv[])
{
  
  const char* szStr = NULL;
  const char* locate = NULL;
  unsigned char szSum[101];
  unsigned char szTemp[101];
  if(argc != 2)
    return -1;
  szStr = argv[1];
  memset(szSum,0,101);
  while((szStr = strpbrk(szStr,"0123456789")) != NULL)
  {
    locate = szStr;
    while(*locate >= 0x30 && *locate <= 0x39)
      ++locate;
    memset(szTemp,0,101);
        strncpy(szTemp,szStr,locate - szStr);
    mystrrev(szSum);
    mystrrev(szTemp);
    add(szSum,szTemp);
    mystrrev(szSum);
    szStr = (const char*)(locate + 1);
  }
    printf("the sum is:%s\n",szSum);
  system("pause");
  return 0;
}

void mystrrev(unsigned char* s)
{
  char *p = NULL;
  char i = strlen(s);
  p = calloc(1,i + 1);
  if(p == NULL)
  {
    printf("strrev:fatal error!\n");
    return;
  }
  while(--i >= 0)
  {
    *(p++) = *(s + i);
  }
  p -= strlen(s);
  memset(s,0,strlen(s) + 1);
  strcpy(s,p);
  free(p);
}

void add(unsigned char *s1,const unsigned char *s2)
{
    while(*s1 != 0 && *s2 != 0)
  {
    *s1 = ( ((*s1) - 0x30) + ((*s2) - 0x30) );
    if(*s1 >= 10)
    {
      if(*(s1 + 1) != 0)
          *(s1 + 1) = ((* (s1 + 1) - 0x30) + 1) + 0x30;
      else
        *(s1 + 1) = 0x31;
      *s1 %= 10;
    }
    *s1 += 0x30;
    ++s1;
    ++s2;
  }
  while(*s2 != 0)
      *(s1++) = *(s2++);

}
```

用一个`bat`来启动程序进行测试

```bat
@echo off

test.exe 你 54839865486548650468054658847397577599 他 55423859977889993456666666666666666666699999999999999999999999 我 8888888888888888888888888888888888888
```

得到如下的结果

![](img/6d9366c6818ca27eec4a72a628bbfe4d.jpg)

和`Google`计算器对比一下

![](img/9c73d76a0049fc239ad890afe8989b75.jpg)

总结：

> 引用: 有一个字符串，里面包含一些数字，写一个函数，把这些数字加起来。

`刘涛涛`前辈的这个题目，看似很简单，实际不是很简单。他只说了包含一些数字，但并没有说正数还是负数，整数还是浮点数，我的答案只能够计算一定位数的正整数的情况，看来要找一份程序员的工作真不容易![](img/bedc2a7572d1054a9590e607b9635bdd.jpg "皱眉")

惟有用诗人屈原的话自勉：路漫漫其修远兮，吾将上下而求索！

代码:

```python
In [1]: s = '你 54839865486548650468054658847397577599 他 55423859977889993456666666666666666666999999999999999999999999 我 8888888888888888888888888888888888888'

In [2]: import re

In [3]: sum(map(int, re.findall('-*\d+', s)))
Out[3]: 554238599778899934566666730395421042104539356943547736286466486L
```
