

# 动态规划用于加密初探（1）（2）

> 原文：[`www.pediy.com/kssd/pediy11/108173.html`](https://www.pediy.com/kssd/pediy11/108173.html)

1，在本文中，我试图将算法中经典的动态规划设计用于软件加密，可结合其他加密算法共同使用。  
2，以 C++实现，不进行 API 调用，主要关注算法在软件保护方面的灵活运用。  
3，欢迎各位进行破解尝试，源代码附有示例姓名与序列号。  
4，原创拙作，水平有限，欢迎各位给予指点。  
      Green Tsai  
  

代码:

```

/\*
  Name: 动态规划算法用于软件加密初探(1) 
  Copyright: @Green Tsai
  Author: Green Tsai
  Date: 04-03-10 12:11
  Description: 
    1，思路与想法：本程序试图将动态规划算法用于软件加密，并用动态规划中最经典的 LIS（Longest Increasing Subsequence）
  问题作一示例。 
    2，对软件进行非明码比较加密,避免一般的逆向破解。
    3，示例姓名：kanxue，序列号：xbayczhfea80。
\*/

#include <cstdlib>
#include <iostream>

using namespace std;

int Solve(char \*a,int n);
bool CheckName(char \*name,int len);
bool CheckSerial(char \*serial,int len);
bool CheckLen(int a,int b);

int main(int argc,char \*argv\[\])
{
  bool ok=false;  
  while(ok==false)
  {
    char name\[30\],serial\[30\];
    cout<<"Please input your name:"<<endl;
    cin>>name;
    cout<<"Please input your serial number:"<<endl;
    cin>>serial;
    int x=Solve(name,strlen(name));
      int y=Solve(serial,strlen(serial));
      
      // cout<<"strlen:"<<strlen(name)<<" "<<strlen(serial)<<endl;
      // cout<<x<<" "<<y<<endl;
      
      if(CheckName(name,strlen(name)) && CheckSerial(serial,strlen(serial)) && (2\*strlen(name)==strlen(serial)) && (x==y) && (x>=3))
    {
      ok=true;
      cout<<"Congratulations!"<<endl;
      cout<<endl;
    }
    else
    {
      cout<<"Sorry!"<<endl;
      cout<<endl;
    }  
  }
  
  system("PAUSE");
  return 0;   
}

int Solve(char \*a,int n)   
{   
  int dp\[30\];  
    dp\[0\]=1;   
    int ans=0;   
    for(int i=1;i<n;i++)   
    {   
        int max=0;   
        for(int j=0;j<i;j++)   
        {   
            if(a\[i\]>=a\[j\] && max<dp\[j\])   
                max=dp\[j\];   
        }   
        dp\[i\]=max+1;  
                       
        if(ans<dp\[i\])     
            ans=dp\[i\];      
    }   
    return ans;   
}  

bool CheckName(char \*name,int len)
{
  if(len>=3 && len<=10)
    return true;
  else
    return false;
}

bool CheckSerial(char \*serial,int len)
{
  if(len>=6 && len<=20)
    return true;
  else
    return false;
}

```

上传的附件
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || main.rar  
 |

2.使用 LIS 的算法太简单，O(n²)的 DP，哪怕是反汇编也一眼就看完了，建议可以改进成 O(nlogn)的 LIS 算法(有 DP 和贪心两种)。并且可以考虑把 LIS 计算过程中的方案也求出来，并用到加密上，就会强大许多。  
\----------------------这个学习了。----------------  
  
![](img/962324edb48d57effae0c0571850a66b.jpg "酷")您们搞破解，我来学加密。![](img/9b2aec7f3d4bb041b2d419a59832192f.jpg "恐怖!")  
  

代码:

```
/\*
  Name: 动态规划用于软件加密初探（2） 
  Copyright: @Green Tsai
  Author: Green Tsai
  Date: 04-03-10 13:09
  Description: 
    1，思路与思想：本程序试图将动态规划算法用于软件加密，主要涉及动态规划中的字符串编辑距离与相似度判定。
    2，本程序中的编辑代价函数与相似度判定函数均可调整来增加加密强度与逆向难度。 
    3，示例姓名：kanxue,序列号：90yxop。
  参考文档：《挑战编程》 Steven S.Skiena 著 刘汝佳 译 
\*/
#include <cstdlib>
#include <iostream>

using namespace std;

#define MAXLEN 35

#define MATCH  0    // enumerated type symbol for match
#define INSERT   1    
#define DELETE  2

typedef struct
{
  int cost;    // cost of reaching this cell
  int parent;    // parent cell
}cell;
cell m\[MAXLEN+1\]\[MAXLEN+1\];    // dynamic programming table

// 代价函数 
int match(char c,char d);
int indel(char c);
// 初始化与目标格 
void row\_init(int i);
void column\_init(int i); 
void goal\_cell(char \*s,char \*t,int \*i,int \*j);
// 编辑距离
int string\_compare(char \*s,char \*t);
// 相似度判定 
bool same(char \*s,char \*t);
 
int main()
{
  bool ok=false;    
  while(ok==false)
  {
    char name\[31\],serial\[31\];
    name\[0\]=serial\[0\]='!';    
    cout<<"Please input your name:"<<endl;
    cin>>&name\[1\];
    cout<<"Please input your serial number:"<<endl;
    cin>>&serial\[1\];    
          
        // cout<<strlen(name)<<" "<<strlen(serial)<<endl;
        // cout<<string\_compare(name,serial)<<endl;
      if( strlen(name)>=4 && ( strlen(name)==strlen(serial) ) && same(name,serial) && string\_compare(name,serial)==5 )
    {  
      ok=true;
      cout<<"Congratulations!"<<endl;
      cout<<endl;
    }
    else
    {
      cout<<"Sorry!"<<endl;
      cout<<endl;
    }  
  }
  
  system("PAUSE");
  return 0;   
  system("PAUSE");
  return 0;
}

int match(char c,char d)
{
  if(c==d)
    return 0;
  else 
    return 1;    
}
int indel(char c)
{
  return 1;
}

void row\_init(int i)
{
  m\[0\]\[i\].cost=i;
  if(i>0) m\[0\]\[i\].parent=INSERT;
  else m\[0\]\[i\].parent=-1;
}
void column\_init(int i)
{
  m\[i\]\[0\].cost=i;
  if(i>0) 
    m\[i\]\[0\].parent=DELETE;
  else 
    m\[0\]\[i\].parent=-1;
}
void goal\_cell(char \*s,char \*t,int \*i,int \*j)
{
  \*i=strlen(s)-1;
  \*j=strlen(t)-1;
}  
int string\_compare(char \*s,char \*t)
{
  int i,j,k;    // counter
  int opt\[3\];    // cost of the three options
  
  for(i=0;i<MAXLEN;i++)
  {
    row\_init(i);
    column\_init(i);
  }
  
  for(i=1;i<strlen(s);i++)
  {
    for(j=1;j<strlen(t);j++)
    {
      opt\[MATCH\]=m\[i-1\]\[j-1\].cost+match(s\[i\],t\[j\]);
      opt\[INSERT\]=m\[i\]\[j-1\].cost+indel(t\[j\]);
      opt\[DELETE\]=m\[i-1\]\[j\].cost+indel(s\[i\]);
      
      m\[i\]\[j\].cost=opt\[MATCH\];
      m\[i\]\[j\].parent=MATCH;
      for(k=INSERT;k<=DELETE;k++)
      {
        if(opt\[k\]<m\[i\]\[j\].cost)
        {
          m\[i\]\[j\].cost=opt\[k\];
          m\[i\]\[j\].parent=k;
        }
      }
    }
  }
  goal\_cell(s,t,&i,&j);
  return m\[i\]\[j\].cost;
}
bool same(char \*s,char \*t)
{
  // 相似度小于 50%,返回 true 
  int count=0;
  for(int i=1;i<strlen(s);i++)
  {
    if(s\[i\]==t\[i\])
      count++;
  }
  if(count<=strlen(s)/2)
    return true;
  else
    return false;
}

```

上传的附件
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || main.rar  
 |