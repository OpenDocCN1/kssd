
# 突破《网络可疑心跳发现未知木马》理论继续隐藏

> 原文：[`www.pediy.com/kssd/pediy12/140376.html`](https://www.pediy.com/kssd/pediy12/140376.html)

## 前言

我在曾经发表过一片《网络可疑心跳发现未知木马》的文章，文章发表之后很多人在问我这个东西的详细算法问题，其实，这个算法我也只有一个模糊的模型，毕竟上线方式改变的木马太多了，最后有朋友说，你发表之后我们的 `GH0ST`，灰鸽子变种怎么活？为了小黑更快乐，于是我删了。。

一，变种网络心跳突破常规心跳包

二，模拟 `http`，突破 `IDS` 协议分析

三，图片读取 `ip`，突破动态域名上线

四，查看 `DNS` 活动，突破域名次数统计

五，一个 `IOCP` 模型的模型大家看看有没有什么不稳定的

六，总结

## 一，变种网络心跳突破常规心跳包

什么时常规网络心跳包呢？这里我就不抓图了。大家可以再网络上自己下一个木马用然后用那个什么抓包工具看看木马在正常工作的时候，是不是每隔几分或者几秒有一个固定字节通讯包？，我把这个包称为木马心跳包，很多 `IDS` 设备可以通过这个包的特征抓到这样的木马的再网络上存活的证据，但有一点只通过这个包证明内网是否被种马有很大的误报率。这就是我为什么提出《网络可疑心跳发现未知木马》这个理论，话说多了，我们还是看看如何突破那个《网络可疑心跳发现未知木马》这个理论吧。

`int` 类型 大小为 32 位

（8 位单片机 `int` 类型为 16 位，PC 机为 32 位，我还停留在 32 上，现在是不是 64 位呢？知道的 请告诉我一下）

首先我们对这 32 位数据大小进行功能的划分，例如：

*   10002000 之间的数据是上线标志
*   3000---4000 之间的数据时心跳标志

呵呵，，你现在应该快理解我的企图了吧，没有错！就是这样！

下面我给出实现随机心跳包代码

```cpp
// 实现随机心跳包.cpp : 定义控制台应用程序的入口点。

//

#include "stdafx.h"

#include "windows.h"

int GetCountInt(int i)

{

    int ce;

    SYSTEMTIME gtm;

looop:

    GetSystemTime(&gtm);

    srand((unsigned int)gtm.wMilliseconds);  //初始话随机种子

    ce=(int)(rand() % 10000);//得到 9999 以内的整数

    switch (i)

    {

    case 1:

        if ((1000<ce)&&(ce<2000))

            return ce;

            goto looop;            //单片机写多了，习惯性 goto....

            break;

    case 2:

        if ((2000<ce)&&(ce<3000))

            return ce;

        goto looop;         

        break;

    case 3:

        if ((3000<ce)&&(ce<4000))

            return ce;

        goto looop;              

        break;

    default:

        return false;

        break;

    }

}

int _tmain(int argc, _TCHAR* argv[])

{

    int c;

    c=GetCountInt(1);

    printf("%d\r\n",c);

    c=GetCountInt(2);

    printf("%d\r\n",c);

    c=GetCountInt(3);

    printf("%d\r\n",c);

    c=GetCountInt(4);

    printf("%d\r\n",c);

    return 0;

}
```

以上是我写的随机包测试代码，这代码可以实现我们所说的指定范围之内的 `int` 大小来实现指令信息的传递工作，但这并不能说明这已经完成了我们随机心跳所有工作（待续）
