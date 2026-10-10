

# 金蝶 2000XP 标准版

> 原文：[`www.pediy.com/kssd/pediy05/pediy50205.htm`](https://www.pediy.com/kssd/pediy05/pediy50205.htm)

NetSentinel-C 网络狗破解（金蝶 2000XP 标准版）   
金蝶财务软件 2000XP 标准版网络狗破解   
  
软件名：金蝶财务软件 2000XP 标准版   
加密狗类型：彩虹圣天诺 NetSentinel-C 网络狗   
加密方法：API 方式。   
解密方法：无狗解狗。   
使用工具：TRW2000，W32DASM，Delphi5.0   
  
简要说明：   
金蝶财务软件 2000xp 标准版是一个单用户版和小规模网络版的复合体，整个系统用 VB 语言开发。   
系统启动时，先进行单用户狗的检测，检测不到单用户狗数据时就转向检测网络狗。   
单用户版使用的狗与行政事业版相同，都是圣天诺公司的 Superpro 软件锁。   
网络版的狗就有了区别，所用的加密狗改成了同样是圣天诺公司出品的 NetSentinel-C 网络狗。   
NetSentinel-C 和 Superpro 做为同一个公司的产品，虽然有许多的不同之处，但其相同的地方也是显而易见的。   
  
破解基本思路：   
上次破解 Superpro 时，我从 Superpro 狗的动态库 sx32w.dll 入手，这次破解网络版，sx32w.dll 就没了用处。我们这次的突破口改成了这里——Nslms324.dll。   
你可别小看 Nslms324.dll 这个动态链接库，这个东东就是 NetSentinel 狗的客户端通讯驱动。它就是软件客户端与 NetSentinel 网络狗服务器端的通讯桥梁。   
  
破解过程：   
1、用 W32DASM 反汇编 Nslms324.dll,查看其内部的过程。   
2、屏蔽掉 sx32w.dll(为什么屏蔽？？不好意思了，单机版原来的 sx32w.dll 已经不知道哪里去了，现在笨笨机器里的这个已经是笨笨破解过的了。呵呵~~)   
3、用 TRW 载入 kdwin.exe，跟踪至 Nslms324.dll 处细看，发现金蝶 Nslms324.dll 中过程的调用次序如下：   
  
软件载入时启动以下过程：   
a) set\_rnbo\_lib\_parameters (多次) -->出错时提示退出   
b) open\_first\_key -->出错时提示退出   
c) get\_rnbo\_lib\_parameters (多次) -->出错时提示退出   
d) query\_key (多次执行) -->出错时无提示，继续执行   
e)验证 query\_key 的返回值，   
正确时：正常功能版本，Pass!!!   
出错时：提示软件为教学演示版。   
  
软件退出时启动以下过程：   
f) close\_key   
g) detach\_from\_dll   
  
  
3、分析以上流程，发现:   
A. set\_rnbo\_lib\_parameters 是一个复合的多分支函数,传入值采用动态定义方式，第一个传入值是功能号。   
软件先后以不同形式的参量使用了 set\_rnbo\_lib\_parameters 的第 3,第 5 功能，   
其目的分别为设置服务器，设置通讯协议，返回值可能是出错号，设为 0 时通过。   
  
B. open\_first\_key 大致相当于 Superpro 狗的 SproFindFirstUnit 功能，传入狗类型和查询口令等，应该是用于定位狗，返回值同上。   
  
C. get\_rnbo\_lib\_parameters 也是一个复合的多分支函数,传入值同 set\_rnbo\_lib\_parameters 一样采用动态定义方式，第一个传入值也是功能号。应该是用于对 client 和 server 进行检测(大致包括 client 端及 server 端的驱动版本号、句柄、种子点、用户数及许可证等等)，返回值同前。   
  
D. query\_key 大致相当于 Superpro 狗的 SproRead 功能,传入地址用于采集数据。   
  
E. closekey 看名字就知道了，一定是程序退出时从 server 端释放服务器句柄及许可证用的，在这里我们用不到。   
  
F. detach\_from\_dll 应该不是一个函数，而是一个子程序，可能是在结束驱动调用，没有传入值。   
  
4、经过以上分析，用 delphi 重新构造了一个 Nslms324.dll,调试通过后用其替换原来的 Nslms324.dll。运行软件，提示狗数据出错。(当然会出错，因为笨笨没有网络狗，采不到数据嘛~~。query 那里的返回值笨笨我全部给设成了 3838 这样子，为了方便跟踪调试，笨笨还多写了一个 messagebox 进去，^\_^)。   
5、现在容易了，再花了一点点时间跟一下吧。   
用 trw2000 载入，断点当然是下在 messagebox 那里，执行到 query\_key 函数以后的数据验证部分，发现正确的返回值应为，XXXX(因为是行业软件，这里省略掉，大家见谅)......   
6、将正确的返回值写回我们的 Nslms324.dll 里，运行，通过!!   
  
也不知道前面的几篇各位看不看的上眼,行是不行 peterchen 又不讲话，趁上午有点时间，多贴一篇上来吧。  4dd466d0e1e0cc123afbe504da2462ca