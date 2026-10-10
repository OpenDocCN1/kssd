

# Diffie-Hellman 协议以及安全研究

> 原文：[`www.pediy.com/kssd/pediy10/89467.html`](https://www.pediy.com/kssd/pediy10/89467.html)

一、Diffie-Hellman 协议以及安全研究  
Diffie-Hellman 密钥交换算法的有效性依赖于计算离散对数的难度。简言之，可以如下定义离散对数：首先定义一个素数 p 的原根，为其各次幂产生从 1 到 p-1 的所有整数根，也就是说，如果 a 是素数 p 的一个原根，那么数值  
  
            a mod p, a2 mod p, ..., ap-1 mod p  
  
是各不相同的整数，并且以某种排列方式组成了从 1 到 p-1 的所有整数。  
对于一个整数 b 和素数 p 的一个原根 a，可以找到惟一的指数 i，使得  
  
            b = ai mod p   其中 0 ≤ i ≤ (p-1)  
  
指数 i 称为 b 的以 a 为基数的模 p 的离散对数或者指数。该值被记为 inda ,p(b)。  
基于此背景知识，可以定义 Diffie-Hellman 密钥交换算法。该算法描述如下：  
1、有两个全局公开的参数，一个素数 q 和一个整数 a，a 是 q 的一个原根。  
2、假设用户 A 和 B 希望交换一个密钥，用户 A 选择一个作为私有密钥的随机数 XA<q，并计算公开密钥 YA=aXA mod q。A 对 XA 的值保密存放而使 YA 能被 B 公开获得。类似地，用户 B 选择一个私有的随机数 XB<q，并计算公开密钥 YB=aXB mod q。B 对 XB 的值保密存放而使 YB 能被 A 公开获得。  
3、用户 A 产生共享秘密密钥的计算方式是 K = (YB)XA mod q。同样，用户 B 产生共享秘密密钥的计算是 K = (YA)XB mod q。这两个计算产生相同的结果：  
        K = (YB)XA mod q  
          = (aXB mod q)XA mod q  
          = (aXB)XA mod q             （根据取模运算规则得到）  
          = aXBXA mod q  
          = (aXA)XB mod q  
          = (aXA mod q)XB mod q  
          = (YA)XB mod q  
因此相当于双方已经交换了一个相同的秘密密钥。  
  
4、因为 XA 和 XB 是保密的，一个敌对方可以利用的参数只有 q、a、YA 和 YB。因而敌对方被迫取离散对数来确定密钥。例如，要获取用户 B 的秘密密钥，敌对方必须先计算  
          XB = inda ,q(YB)  
然后再使用用户 B 采用的同样方法计算其秘密密钥 K。  
Diffie-Hellman 密钥交换算法的安全性依赖于这样一个事实：虽然计算以一个素数为模的指数相对容易，但计算离散对数却很困难。对于大的素数，计算出离散对数几乎是不可能的。  
下面给出例子。密钥交换基于素数 q = 97 和 97 的一个原根 a = 5。A 和 B 分别选择私有密钥 XA = 36 和 XB = 58。每人计算其公开密钥  
          YA = 536 = 50 mod 97  
          YB = 558 = 44 mod 97  
在他们相互获取了公开密钥之后，各自通过计算得到双方共享的秘密密钥如下：  
             K = (YB)^XA mod 97 = 44³⁶ = 75 mod 97  
            K = (YA)^XB mod 97 = 5058 = 75 mod 97  
从|50，44|出发，攻击者要计算出 75 很不容易。  
  
  
二、简单模拟实现  

代码:

```

/\*-------------------------------------------------------------------------
 \* 版权所有：jxccy，引用请注明出处
 \* 作者：jxccy
 \* QQ：524733264
 \* 完成时间： 2008 年 6 月
 -------------------------------------------------------------------------\*/


class User
{
public:
  string name;
  int a1;//用户随机选项的数
  int a2;//对方发送的数
  int k;//会话密钥
  User(string name);
};
User::User(std::string na)
{
  this->name=na;
}

//大数幂乘算法
int mul(int x,int r,int n)
{
  int a=x;
  int b=r;
  int c=1;
  while(b!=0)
  {
    if(b%2!=0)
    {
      b=b-1;
      c=(c\*a)%n;
    }
    else
    {
      b=b/2;
      a=(a\*a)%n;
    }
  }
  return c;
}
//判断数组里面元素都不相等(不相等为真)
bool IsEqualInArray(int \*a,int n)
{
  int flag=0;
  for(int i=0;i<n;i++)
  {
    for(int j=i+1;j<n-1;j++)
    {
      if(a\[i\]==a\[j\])
      {
        return false;
      }
    }
  }
  return true;
}
//求本原元
void BenYuan(int prime)
{
  int \*a=new int\[prime\];
  cout<<prime<<"的本原元为：";
  for(int i=1;i<=prime;i++)
  {
    
    for(int j=0;j<prime;j++)
    {
      a\[j\]=mul(i,j+1,prime);
    }
    if(IsEqualInArray(a,prime))
    {
      cout<<i<<"；";
    }
  }
  cout<<endl;
}

```

  
  
/\*-------------------------------------------------------------------------  
 \* 版权所有：jxccy，引用请注明出处  
 \* 作者：jxccy  
 \* QQ：524733264  
 \* 完成时间： 2008 年 6 月  
 -------------------------------------------------------------------------\*/

上传的附件
| ![](img/2674941de574f12fa6a3e90d2eb3468a.jpg) || Diffie-Hellman 协议以及安全研究.doc  
 |