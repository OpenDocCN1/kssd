
# 报恩计划前言

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-601.htm`](https://www.pediy.com/kssd/pediy09/pediy09-601.htm)

> `kflnig`

> 感谢我的信息老师，是他带我快速进入计算机的大门。我想现在我的技术还算过得去了。所以我想我得报恩。我要把更多的人带入计算机的殿堂。希望我的文章可以让你喜欢。我希望你可以懂点 `C++` 知识，不至于我的代码你看不懂。本系列文章主要关于：《全国信息学奥林匹克联赛，培训教程》，但我基本上都是用 `C++` 来编程了的。而且我用的是 `borland C++ 6.0` 可能会和 `VC` 等有些不同。

> 首先是前置知识：

> 未知循环层数的代码框架该怎么写。

```cpp
#include <iostream.h>

int k=0;

void dg(int n)
{
    if (n==0) exit;
    else {++k;dg(--n);}
}

int main(int argc, char* argv[])
{
    dg(10);
    cout<<k;
    getchar();//getchar()的作用是不让 cmd 自动关闭。没什么作用，只是为了方便我自己。
    return 0;
}
```

> 我想上面是一个很好的例子。也就是说，我们可以通过递归来控制循环层数。

> 再谈谈《全国信息学奥林匹克联赛，培训教程》中几点比较难懂的地方。

```pascal
for n1=1 to 1000 do
 for n2=1 to 2 do
 begin
 ……
 end;
```

> 这种写法不如

```pascal
for n2=1 to 2 do
 for n1=1 to 1000 do
 begin
 ……
 end;
```

> 开始我不了解，但是从汇编的角度，一看就知道了。

> 我们可以看看假如用汇编翻译一下，就是

```asm
xor eax,eax
l2:
inc eax
xor ecx,ecx
l1:
inc ecx
……
cmp ecx,2
jnz l1
cmp eax,1000
jnz l2
```

> 看到了吗？大量的 `cmp` 指令会大量执行，在外层循环中，就是这句：`cmp eax,1000`。所以如果按照较优的写法。

```asm
xor eax,eax
l2:
inc eax
xor ecx,ecx
l1:
inc ecx
……
cmp ecx,1000
jnz l1
cmp eax,2
jnz l2
```

> 那么 `cmp` 指令就执行的比较少了。

> 指针：

```cpp
struct  node
{
    int  val;
    struct  node  *next;
}
```

> 当年我看到这里我十分不明白。我想这样鸡生蛋，蛋生鸡，没完没了。现在懂了些汇编，才知道缘由。`struct node *next;`这一句，实际上，指针在 32bit 机器中，只分配一个 `DWORD`。所以这句话实际上是告诉编译器，要分配一个指针（大小为 32bit）。

> `struct node` 只是为了理论上的服从。说明这个指针将指向的是一个 `node` 结构体变量。

> 递归，我已经写过一篇关于递归的文章了。你最好自己在看雪论坛上找找。递归是一定得懂的。

> 我并不要求你看懂我这一系列中所有的文章，对不太懂数据结构的小鸟来说，可能这个有点难。我希望你可以从我的文档中找到一些，可以提高你自己的知识。

> 如果看得懂那么你可能比我厉害多了。因为我自己写代码风格不好。我厉害到什么程度呢？就是去年奥林匹克竞赛连初赛都没有进。

## 解题 1

```cpp
//---------------------------------------------------------------------------
题目：
设有 1g，2g，3g，5g，10g，20g 的砝码若干枚（其总重量＝<1000g），要求：
输入 a1，a2，a3，a4，a5，a6（表示 1g 的砝码有 a1 个，2g 的 a2 个，……，20g 的砝码 a6 个）
输出 total＝n（n 表示用这些砝码能够称出的不同重量的个数，但不包括一个砝码也不取用的情况）
//-------------------------------------------------------------------------
```

> 十分差的穷举我不写了，首先写一个可以上的了台面的穷举。

```cpp
#include <iostream.h>

int main(int argc, char* argv[])
{
    int a[6];
    bool qj[1001]={false};
    int n,n1,n2,n3,n4,n5,n6,tl=0;
    for (n=0;n<=5;n++)
        cin>>a[n];
    for (n1=0;n1<=a[0];n1++)
     for (n2=0;n2<=a[1];n2++)
      for (n3=0;n3<=a[2];n3++)
       for (n4=0;n4<=a[3];n4++)
        for (n5=0;n5<=a[4];n5++)
         for (n6=0;n6<=a[5];n6++)
            qj[n1+2*n2+3*n3+5*n4+10*n5+20*n6]=true;
            qj[0]=false;
    for (n=0;n<1001;n++)
    if (qj[n]==true)    ++tl;
    cout<<"total="<<tl;
    getchar();
    getchar();
    return 0;
}
```

> 写这个东西，是为了说明一点，就是引出本文乃至本系列中很重要的一点。数组下标利用。

> `bool qj[1001]` 设置了这么一个 `bool` 数组，就是为了利用其下标。我把解都放入了下标里。因为题目中有一点说：其总重量＝<1000g。所以一般情况来讲。不可能超出时间限制。

> 记住没有永远的算法，只有永恒的变化。

> 上面的穷举显然在某些方面不合适（如果题目改变的话）。因为穷举有时实效不是很高。所以我们需要优化。从思想上进行优化。

> 先给出代码：

```cpp
#include <iostream.h>

int main(int argc, char* argv[])
{
    int a[6],val[6];
    bool qj[1001]={false};
    int n,n1,n2,n3,n4,n5,n6,tl=0;
    for (n=0;n<=5;n++)
        cin>>a[n];
        qj[0]=true;
    val[0]=1;val[1]=2;val[2]=3;val[3]=5;val[4]=10;val[5]=20;
    n=a[0]+2*a[1]+3*a[2]+5*a[3]+10*a[4]+20*a[5];
    for (n3=0;n3<=5;++n3)
        for (n2=1;n2<=a[n3];++n2)
          for (n1=n;n1>=0;--n1)
                  if (qj[n1]) qj[n1+val[n3]]=true;
    qj[0]=false;
    for (n=0;n<1001;n++)
    if (qj[n]==true)    ++tl;
    cout<<"total="<<tl;
    getchar();
    getchar();
    return 0;
}
```

> 这种方法很不错。思想抓住了两点：

> 1、反正你的解就在 `qj[1001]` 中

> 2、砝码先加和后加的次序无关

> 懂得了思想剩下的就只有一些编程上的小问题了。

> 你知道为什么开始要

> `qj[0]=true;` 这个属于小技巧。为了让下面的 `for` 语句可以顺利运行。

> 三条 `for` 语句都在干什么？

> 第一条 `for` 语句是利用了思想 2。因为与砝码的次序无关，所以我们可以根据一个个砝码可以生成的值作文章。动态规划，差不多了。

### 马踏棋盘问题

```cpp
//---------------------------------------------------------------------------
/*本文的经典在后半部分。
首先是题目：马踏棋盘问题
设有一个棋盘（2<=n<=50，2<=m<=50），在棋盘上任一点有一个中国象棋“马”
给出马原来的坐标（1，1），问是否可以到达（ n，m）。
如果可以则求出最小步数。
不可以则输出 error！
规则：马走日字。
看到这个题目，大多数的人会觉得用回溯法是不二之选。可是，它和原来的马踏棋盘问题
又有一定的出入。因为经典的马踏棋盘问题还有一条规则。就是马只能向右走。
所以这道题目的处理量更加大了。在较短的时间内回溯法是绝对得不到解的。
我是个不懂算法的人。我勉强知道有一个算法叫回溯。就我个人先来谈谈如何解题。
因为这道题目比较简单我就直接给出算法了。为了可读性，我用数组时稍微浪费了一点。*/
#pragma hdrstop
#include <iostream.h>
//---------------------------------------------------------------------------
#pragma argsused
void search_(int val,int i,int j,int sz[51][51]);
int main(int argc, char* argv[])
{int sz[51][51]={0};
int i,j,n,m,num;
cin>>n>>m;
sz[1][1]=1;
for (num=1;num<=(n+m);num++)
        {
               for (i=1;i<=n;i++)
                for (j=1;j<=m;j++)
                {
                         search_(sz[i][j],i+1,j+2,sz);
                         search_(sz[i][j],i+1,j-2,sz);
                         search_(sz[i][j],i+2,j+1,sz);
                         search_(sz[i][j],i+2,j-1,sz);
                         search_(sz[i][j],i-2,j+1,sz);
                         search_(sz[i][j],i-2,j-1,sz);
                         search_(sz[i][j],i-1,j+2,sz);
                         search_(sz[i][j],i-1,j-2,sz);
                }
        }
      --sz[n][m];
      if (sz[n][m]!=-1) cout<<sz[n][m];
      else cout<<"error!";
      getchar();
      getchar();
      return 0;
}

void search_(int val,int i,int j,int sz[51][51])
{
 if ((val!=0)&&(i<=50) && (i>=1) &&(j<=50)&&(j>=1))
 if ((sz[i][j]>val)|| (sz[i][j]==0)) sz[i][j]=val+1;
}
//---------------------------------------------------------------------------
```

> 上面的讲起来比较麻烦。但就是利用数组下标。表示棋盘。用数值记到达此处的最小步数。

> `for (num=1;num<=(n+m);num++)` 这条东西，比较变态。因为下面两条 `for` 语句算出来的

> 不一定是最优解。至少我无法在理论上证明是最优的。我觉得这条 `for` 语句可以保证最优。

> 大家不妨比较一下回溯和我的这个方法的时间效率。

> 有读者云：回溯可以找到路径，你能吗？

> 精彩部分现在开始。自己编造了一种颇不错的数据结构。

> 数组＋指针网。我将在以后专门写文章，来说明这种解类似题目，近乎完美的结构。

## BFS

代码:

```c
#include <stdio.h>   #include <string.h>   #include <stdlib.h>      const int dir[][2] = {{-2,-1},{-2,1},{2,-1},{2,1},{-1,-2},{-1,2},{1,-2},{1,2}};   int q[51*51][3];   int step[51][51];      void output(int k)   {    if (q[k][2] == -1) {printf("(1,1)"); return;}    output(q[k][2]);    printf("->(%d,%d)",q[k][0],q[k][1]);   }      int main()   {    int head, tail, m, n, tx, ty, d;    memset(step, -1, sizeof(step));    scanf("%d%d",&n,&m);    q[0][0] = q[0][1] = 1; q[0][2] = -1;    step[1][1] = 0; head = 0; tail = 1;    while (head < tail) {     if (q[head][0] == n && q[head][1] == m) break;     for (d = 0 ; d < 8 ; d++) {      tx = q[head][0] + dir[d][0];      ty = q[head][1] + dir[d][1];      if (tx >= 1 && tx <= 50 && ty >= 1 && ty <= 50 && step[tx][ty] == -1) {        step[tx][ty] = step[q[head][0]][q[head][1]] + 1;        q[tail][0] = tx;        q[tail][1] = ty;        q[tail++][2] = head;      }     }     ++head;    }    if (step[m][n] == -1) printf("error!\n");    else {     printf("STEP:%d\n",step[m][n]);     output(head);     printf("\n");    }    system("pause");    return 0;   }
```

引用:

*最初由 **kflnig** 发布* (帖子 304508)

```cpp
//---------------------------------------------------------------------------
/*本文的经典在后半部分。
首先是题目：马踏棋盘问题
设有一个棋盘（2<=n<=50，2<=m<=50），在棋盘上...
```

你的程序对于 `(1,3)`, `(1,5)` 等不能给出正确结果
