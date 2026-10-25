
# 进位专家

> 原文：[`www.pediy.com/kssd/pediy06/pediy6564.htm`](https://www.pediy.com/kssd/pediy06/pediy6564.htm)

进位专家算法及注册机源码

官方主页：[`www.5dai.com`](http://www.5dai.com)

适用平台：`win98` / `win2000` / `winXP` / `winME`

注册价格：10 元/ID 终身

## 软件说明

进位专家是配合 `WPE` 使用的绝佳搭档，`WPE` 拦截的封包都是 16 进制的，看起来非常不方便，本程序可以直接翻译 16 进制到 10 进制，并且支持进制转换和直接 16 进制查字符。如果您要修改网络游戏，对于处理数据那是经常的事，所以，进位专家是必须具备的工具。科技极大地提高您的工作效率。

--------------------------------------------------------------------

小弟初学破解，我不使用 `WPE`，呵呵，好像在破解的时候也会用到进制的转换啊，像这样的工具，网上有不少免费的，为什么这个要注册？说是免费，可….

---------------------------------------------------------------------

用 `PEiD` 查无壳，`VB6.0`，呵呵，:p，启动的时候就要求输入注册码，要不就不能用。而且每次用的时候都要输入用户名和密码，真不方便啊^\_\*

用户名不支持中文，但支持空格 :P

-------------------------------------------------------------------

用 `SmartCheck` (我用 `6.03`) 加载, `Program->Start` 或直接按 `F5` `View->选择 Show All Events`

好了，下面我们输入用户名 `cnbragon`，这个时候我并没有填上注册码，而是直接点登陆，出现提示：您输入的用户名不是标准的用户名！

呵呵，我们看看 `SmartCheck`，展开

`_Click`，

```
Len(String:"cnbragon") returns LONG:8
MsgBox(VARIANT:String:" 您输入的用户名不是标准的用户名！…..省略)
```

看来用户名有长度的限制啊，那个 `Len()` `return` 8 就是得到我们用户名的长度，然后根据它来判断是不是合法的用户名

好，我们再试一个 `cnbragon1`，还是非法，看来 9 个也不行，好，那我们就输入 10 个看看吧，`cnbragon12`，好，这次出现另一个出错提示：您没有输入密码，如果您没有秘密（这个地方。。？？），。。。您可以免费获得密码。。（hoho~）说明用户名的长度必须大于 9 才行啊

这时我们再看看 `SmartCheck`

`_Click`，

```
Len(String:"cnbragon") returns LONG:10
MsgBox(VARIANT:String:" 您没有输入密码……..省略)
```

没有什么东东嘛，好，我们随便输入一些注册码试试，比如 `123456789`

然后点登陆，当然还是有出错提示:您的密码不能通过认证…，这个时候我们再返回看看 `SmartCheck` 给我们提供了哪些信息

`_Click`

```
Len(String:'cnbragon…") return Long:10
Long(10)->Integer(10)
Right(VARIANT:ByRef String:"cnbragon…",long:1)
Left(VARIANT:String:"2",long:1)
Asc(String:"2") returns interger:50
Interger(50)->String(50)
Right(VARIANT:ByRef String:"50",long:1)
………..省略 N 多
```

其实意思很简单就是说

得到用户名的长度 `leng`

循环 `leng` 次，第 `i` 次就取第 `i` 个字符，然后求出它的 `ascii` 码值,然后再取 `ascii` 码值的最后一位，这样就共有 `leng` 位，这就是注册码

注册机源代码：

```
#include <stdio.h>

void main()
{
 int leng,j;
 int i=0;
 int s[41];
 char n[40];
 printf("Please input Your RegName:");
 scanf("%s",n);
 while(n[i]!='\0') i++;
 leng=i;
 if(leng<10)
 {
  printf("Name must longer than 9(>9)");
  exit(0);
 }
 printf("Your RegCode is:\n\t");
 for(j=0;j<leng;j++)
 {
  s[j]=n[j]%10;
  printf("%d",s[j]);
 }
}
```

呵呵，你会试出来很多很多很有趣的密码：比如说你输入用户名为`"((((((((((("`，密码为 `00000000000`,

我狂晕啊，你输入 空格试试，你试试~~~，我晕倒了， 这个注册程序不支持空格字符，可进位专家这个软件支持啊，空格的 `ascii` 码是 32,所以它的注册码就是 `222222222222`...有多少个空格就有多少个 2，我狂晕啊，这也行！

`4dd466d0e1e0cc123afbe504da2462ca`
