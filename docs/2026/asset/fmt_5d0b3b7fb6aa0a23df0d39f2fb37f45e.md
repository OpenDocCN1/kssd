
# 我来做个总结算法分析

1. 把我们输入的用户名转化为16进制数存起。
2. 再把16进制数倒过来，比如以前123456倒过来就是654321了。
3. 再分别取出固定字串 `flv67u986e` 中的前4位（`flv6`）和5-9位（`7u986`）。
4. 再把它们组成注册码 `flv6`－16进制数倒过来的前4位`7u986`－16进制数倒过来字串的5－8位，这就是注册码了。
5. 我用两个用户名测试了一下得到注册码：
   1. 第一个用户名是 `wangwei`，转化为16进制数是 `77616E67776569`，倒过来就是 `96567776E61677`，得到注册码就 `flv6－96567u986－7776`。
   2. 第二个用户名是 `sdrf5678lk`，转化为16进制数是 `73647266353637386C6B`，倒过来就是 `B6C68373635366274637`，得到注册码就 `flv6－B6C67u986－8373`。
6. 用写的一个算法注册机：

```c
#include <stdio.h>

int zifen(char *s,int *u)                    //------------->关键是zifen()这个函数它的作用是
{                                            //把我们的字符的ACSII值进行置换，如：i的ACSII值是             
   while (*s != '\0')                        //69通过zifen()就变为96了
   {
      *u=*s%0X10;
      u++;
      *u=*s/0X10;
      u++;
      s++;
   }
}                                            // <---------------

int main()
{
   int i=0, j=0, k=0;                        
   char a[30];                              //用来接收用户名字符串
   char b[3]={'0'};                         //存放用户名最后4个字符，而且是倒序好的；
   int  f[8]={'0'};                         //用来存放zifen()这个函数处理好的字符；
    printf("请输不小于4位的用户名： \n");
   scanf("%s", &a);                         //接收用户名字符串
    while ( a[i]!= '\0')                     //----------->计算用户名的位数
   {   
      i++;
      j++;
   }                                        //<---------------

    for ( i=(j-1);i>=(j-4 );i--)             //-------------->取用户名的最后4位并进行倒序
   {
      b[k] = a[i];
      k++;
   }                                        //<---------------  

   zifen(b,f);                               //处理倒序好的字符
   printf("你的注册码是：\n");               //------------打印出注册码
   printf("flv6-");                             
   for ( i=0; i<4;i++)
      printf("%X",f[i]);
   printf("7u986-");
   for ( i=4;i<=7;i++)
      printf("%X",f[i]);
   printf("\n");                             //-----------
}
```

![](https://bbs.kanxue.com/upload/attach/201104/315616_ci8tjqbdaaopl3k.jpg)

![](https://bbs.kanxue.com/upload/attach/201104/315616_i4ywvla6466uhxe.jpg)

![](https://bbs.kanxue.com/upload/attach/201104/315616_ack2ls8dkraxkgq.jpg)

![](https://bbs.kanxue.com/upload/attach/201104/315616_20vx361rkh1520f.jpg)

![](https://bbs.kanxue.com/upload/attach/201104/315616_c2jdufy0kz5m6ir.jpg)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [1.jpg](javascript:void(0);)（131.57kb，285次下载）
- [2.jpg](javascript:void(0);)（137.90kb，284次下载）
- [3.jpg](javascript:void(0);)（140.41kb，281次下载）
- [4.jpg](javascript:void(0);)（154.32kb，281次下载）
- [5.jpg](javascript:void(0);)（135.57kb，284次下载）
