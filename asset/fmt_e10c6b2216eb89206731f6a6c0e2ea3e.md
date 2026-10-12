
# 外挂科普

> 原文：[`www.pediy.com/kssd/pediy12/142086.html`](https://www.pediy.com/kssd/pediy12/142086.html)

咳咳 好久都没发帖 闲来无事科普下关于外挂的那点事

话说现在的游戏越来越难 无论二三线 一般都采用从复杂的数据结构上 anti-WG 例如动态结构 结构分散化

什么叫动态结构，即链式结构。说起链式结构 我想大家都不陌生把 链表 二叉树等 在游戏中经常会出一些多重链表，N 叉书等，我相信这里很多新人都会犯傻了 卧槽那么复杂的结构 果断删游戏 放弃走人 其实不管他怎么写 多重链表也罢 N 叉树也罢 但最终还是链式结构存储

还剩下一部分有点基础和耐心的就会想链表嘛 简单得到表头遍历就行了 经过 N 天分析终于得到表头然后遍历 结果发现又走错路了。 PS:游戏那么大，数据结构少则几百多个则上千 在说了遍历完 也得不到结构数据- -

那么游戏又怎么获得结构 `address` 呢 一般是通过 `ID` 或者常量得到指定的结构

```
DWORD getstruncrbase(int id) char *name 函数原型参考
{
    return structbase
}
```

俗话说 外挂考的要么是逆向 要么是算法 我这人有点懒 我懒得去分析 `getstruncrbae`

记录下结构特征码 和地址范围 写个简单的搜索算法

```
DWORD getlinebase()
{
       DWORD addrlist[256]={0};
       byte addressbyte[4];
   DWORD addressline=NULL;
   int addrcount=0;
   DWORD startaddr=0x32f0000;
   for (int i=1;i<=48771;i++)
   {
     ReadProcessMemory(GetCurrentProcess(),(LPVOID)startaddr,&addressbyte,1,NULL);
     if (addressbyte[0]==0xA9)
     {    
       addrlist[addrcount]=startaddr;
       addrcount++;
     }
     startaddr=startaddr+0x10;
   }
   int addresscount=checkarrgy(addrlist);
   for (int ii=0;ii<=addresscount-1;ii++)
   {
     ReadProcessMemory(GetCurrentProcess(),(LPVOID)addrlist[ii],&addressbyte,4,NULL);
     if (addressbyte[0]==0xA9&&addressbyte[1]==0x8B&&addressbyte[2]==0x8c&&addressbyte[3]==0x00)
     {
       addressline=addrlist[ii];
       return addressline;
       break;
     }
   }
}
```

结构分散化就不说了 就与字面意思一样 比如一个人物类 打散 对拖延时间还是有点效率的

一些人会说怎么不带例子，我之能说有基础的自然能看懂

最近真是身心疲惫 有点累 数据包加解密相关 以及 `BUG` 挖掘利用方面就不说了 对游戏安全有兴趣也可以加我 `Q2290380093`

一部分新人想着靠 `WG` 发财 其实没那么容易的 竞争力也是很大的 人家 3 天~7 天基本都可以完成

有些人会说 3 天 这什么速度。。。 其实还满正常的 毕竟有些人对游戏真的很了解 一看就知道怎么做了 只要简单的 `test` 下 就知道游戏的存储结构了 然后 3 台机器 一台 `test` 一台记录数据 一台开发

关于 `VM` 我很少见到有游戏上 `VM` 的 上 `VM` 很麻烦毕竟要以牺牲游戏稳定性为前提 我想没几个游戏敢这么干把
