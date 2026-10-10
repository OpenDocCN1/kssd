

# 流密码内嵌魔方阵于随机存档之研究-- Magic Square 之神奇篇

> 原文：[`www.pediy.com/kssd/pediy10/89799.html`](https://www.pediy.com/kssd/pediy10/89799.html)

这些附件是根据[【原创】流密码内嵌魔方阵于随机存档之研究](http://bbs.pediy.com/showthread.php?t=88968) 所设计出来的东西。  
  
然后接[【原创】流密码内嵌魔方阵于随机存档之研究-- Magic Square 篇](http://bbs.pediy.com/showthread.php?p=628133#post628133) 继续讨论。  
  
我在前几帖有介绍到 magic square 的应用，今天我更深入介绍 magic square 一些特性。  
  
Magic square 本身是一个 matrix，如果把它应用在 encryption/decryption，算是 block cipher 的一种，我想它被归类在 block cipher 应该不会有人有意见。  
  
截至目前为止，我们所谈论的 magic square 都是 normal type，什么是 normal type？简单的来讲，就是有多少格子，就有多少数字，而且每一个 column ，每一个 row ，及每一个 cross 的总和都相等。  
它的万用公式为  
![](img/7f264fa9a59b4bc815e22839154dabce.jpg)  
  
![](img/6dd609c28a2babd2f1687c80d9700fb3.jpg)  
譬如以 3 by 3 为例子，  
![](img/b18f267a46673bc2e5335b62f30c3ce3.jpg)，所以 3 by 3 的 magic square 的每一 line 的总和是 15。  
  
![](img/a4270c1e15f8f8f04b025438f4fc19dc.jpg)  
当 sender encrypt 一个东西之后，他最少要传 ciphertext 及 key 这两样东西给 receiver，receiver 根据 key (or password) 来把 ciphertext decrypt 成 plaintext。这是最基本的情况。  
  
如果 encrypt key 及 decrypt key 是一样的，那通常是 symmetric cryptosystem，如果不一样，通常是 asymmetric cryptosystem。  
如果要让 receiver 对 sender 做 authenticated 的动作时，sender 至少还要 plaintext, key 及 ciphertext 三种给对方，receiver 可以把 ciphertext decrypt 后，跟原来的 plaintext 比对看看是否一致。  
  
symmetric cryptosystem 与 asymmetric cryptosystem 各有它的特性，但却无法解决目前的一个问题，什么问题？  
Send 最少要传 plaintext 及 key 给 receiver。  
有没有什么方式是可以只传 ciphertext 而不用传 key 就可以达到前面所讲的那样？  
目前所知的 cryptosystem 技术是办不到的。连 quantum cryptosystem 也不例外。  
  
现在再回过头来讲 magic square。  
![](img/90dd877feaeb66504bbee337ca85815c.jpg)  
以 3 by 3 magic square 来看，两个对角的和分别是 (8+2)=10，(6+4)=10。  
因 3 \*3 =9，但 10 比 9 大 1。  
  
再来看 4 by 4 magic square。  
![](img/0a61527265bb3f26978d52be238c8a87.jpg)  
两个对角的和分别是 (16+1)=17，(13+4)=17。因 4 \* 4 =16，但 17 比 16 大 1。  
  
再来看 5 by 5 magic square。  
![](img/40809894de00dee349776bf24c6c9c34.jpg)  
两个对角的和分别是 (17+9)=26，(15+11)=26。因 5 \* 5 =26，但 26 比 25 大 1。  
  
接着看 6 by 6 magic square。  
![](img/e7036f466c3e327617f78248b12f488f.jpg)  
两个对角的和分别是 (32+7)=39，(21+14)=35。这两个对角合不一样，比较特别一点。  
先计算 (39+35)=74， 74/2=37，因 6 \* 6 =36，但 37 比 36 大 1。  
  
接着看 7 by 7 magic square。  
![](img/c8d9580d337d2e07b55e351d41dc9d4e.jpg)  
两个对角的和分别是 (30+20)=50，(28+22)=50。因 7 \* 7 =49，但 50 比 49 大 1。  
  
接着看 8 by 8 magic square。  
![](img/a923b16c6a0f9902dbf4ee6c2be04dd9.jpg)  
两个对角的和分别是 (64+1)=65，(57+8)=65。因 8 \* 8 =64，但 65 比 64 大 1。  
  
接着看 9 by 9 magic square。  
![](img/7660161e76e54585e3633ceb3982be2f.jpg)  
两个对角的和分别是 (47+35)=82，(45+27)=82。因 9 \* 9 =81，但 82 比 81 大 1。  
以上大家可以看出一种很奇妙的特性，就是两个各自对角的总和比格子数 (blocks number) 大 1。  
  
除了 6 by 6 比较特别。  
如果 4 个角落总和除以 2 来计算，还是符合大于 1 的特性。   
  
现在我也可以造出一个符合大于 1 特性的 6 by 6 magic square。  
看新的 6 by 6 magic square。  
![](img/3df0ca7d3e54fee728347388bc97dd74.jpg)  
两个对角的和分别是 (6+31)=37，(1+36)=37。因 6 \* 6 =36，但 37 比 36 大 1。  
  
![](img/dffeaae9afa8944a8ee29224b2ab4fac.jpg)  
现在再回到 communication environment ，如果 sender 在一般的 network 上传送 ciphertext 给 receiver，另外在 secure channel 或是 VPN 上传送 seeds 给 receiver，这个 seeds 就是 magic square 的四个 corner 及 size of base ，当 receiver 收到 sender 所传送的 ciphertext 及 seed 时，receiver 可以利用自行产生一个 magic square，此时，产生的这个 magic square 就是一个 key，利用这个 key 来把 ciphertext 做 decryption，就会得到 plaintext。  
  
For example：  
Sender 传送 seeds 为 (9,47,45,37,35) 及 cipertext 给 receiver 时，receiver 收到后，receiver 知道要产生 9 by 9 magic square，且四个 corner 一定是 (47,45,37,35)，若不是，receiver 就无法正确的 decrypt ciphertext 为 plaintext。  
  
在[【原创】流密码内嵌魔方阵于随机存档之研究-- Magic Square 篇](http://bbs.pediy.com/showthread.php?p=628133#post628133)，我所展示的只有用 9 by 9 magic square 来做 encrypt 及 decrypt，我是直接告诉您们我用的 是 9 by 9 的 magic square，这次我所介绍的技术与概念是 receiver 自行可以产生 key 来做 decryption，这是目前已知所有的 cryptosystem 所办不到的事。  
  
一个 3 by 3 normal magic square 只有 1 种，而 4 by 4 normal magic square 有 880 种，可是 5 by 5 normal magic square 却高达 275305224 种。6 by 6 normal magic square 已经是一个 very huge number，而 10 by 10 以上则是 googol number。    
这样排列与组合的证明已经有专家及学者证明出来，可以在 [How many magic squares are there?](http://www.trump.de/magic-squares/howmany.html) 里得到答案。  
  

代码:

```
#include <stdio.h>
#include <stdlib.h>
 main()
{ int i,j,x,y,n=2;/\* 宣告循环变量 i,j,及数组变量 x,y,及阶层变量 n \*/
  int \*\*magic;/\* 宣告数组的大小 \*/
 /\* clrscr();/\* 消除荧光幕 \*/       \*/
  while(n>19 || n%2==0){ /\* 判断条件是否成立 \*/
    printf("\\n 请输入奇数魔术方阵的大小(1-19):");
    scanf("%d",&n);
  }
  magic=(int \*)malloc(n\* sizeof(int));
  \*magic=(int \*)malloc(n\*sizeof(int));
  printf("======>%d\\n",magic);
  x=0;y=(n-1)/2; /\* 设定魔术方阵中的第一个位置 \*/
  for(i=0;i<n;i++) /\* 将所有的数组设定其值为 0 \*/
    for(j=0;j<n;j++)
      \*(\*(magic+i)+j)=0;
  \*(\*(magic+x)+y)=1; /\* 设定魔术方阵的第一个值 \*/
  for(i=2;i<=n\*n;i++)
  { x=(x+(n-1)) % n; /\* 计算方阵往右上的规则  \*/
    y=(y+1) % n;     /\* 及其 x ,y 值的变化现象  \*/
    if( (\*(\*magic+x)+y) != 0){ /\* 判断是否会重复 , 如果重复 \*/
      x=((x+2)) % n;       /\* 计算方阵往下移动到的规则  \*/
      y=(y+(n-1)) % n;     /\* 及其 x ,y 值的变化现象      \*/
    }
    \*(\*(magic+x)+y)=i; /\* 没有错误,将数字填入魔术方阵中 \*/
  }

  printf("--------------%2d 阶的魔术方阵------------------\\n",n);
  for(i=0;i<n;i++){
    for(j=0;j<n;j++)
      printf("%5d",\*((\*magic+i)+j));
    printf("\\n");
  }
  printf("--------------敬请批评指教--------------------\\n");
}
 

```