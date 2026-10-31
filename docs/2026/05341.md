
# [原创]浅谈贝贝游戏数据加密解密方式

贝贝游戏 网址`fb1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3N6S2L8h3f1@1z5e0W2Q4x3X3g2U0L8$3@1`。这个游戏和其他的网狐的客服端不同，界面不同其次主要的不同是他们数据加密方法不同，传统的数据加密方式如下，因为手上有网狐的源码所以直接上源码

```cpp
//加密数据

WORD CTCPSocket::EncryptBuffer(BYTE pcbDataBuffer[], WORD wDataSize, WORD wBufferSize)

{
        //效验参数
        ASSERT(wDataSize>=sizeof(TCP_Head));
        ASSERT(wBufferSize>=(wDataSize+2*sizeof(DWORD)));
        ASSERT(wDataSize<=(sizeof(TCP_Head)+SOCKET_TCP_PACKET));

        //调整长度
        WORD wEncryptSize=wDataSize-sizeof(TCP_Command),wSnapCount=0;
        if ((wEncryptSize%sizeof(DWORD))!=0)
        {
                wSnapCount=sizeof(DWORD)-wEncryptSize%sizeof(DWORD);
                memset(pcbDataBuffer+sizeof(TCP_Info)+wEncryptSize,0,wSnapCount);
        }

        //效验码与字节映射
        BYTE cbCheckCode=0;
        for (WORD i=sizeof(TCP_Info);i<wDataSize;i++)
        {
                cbCheckCode+=pcbDataBuffer[i];
                pcbDataBuffer[i]=MapSendByte(pcbDataBuffer[i]);
        }

        //填写信息头
        TCP_Head * pHead=(TCP_Head *)pcbDataBuffer;
        pHead->TCPInfo.cbCheckCode=~cbCheckCode+1;
        pHead->TCPInfo.wPacketSize=wDataSize;
        pHead->TCPInfo.cbDataKind=SOCKET_VER;

        //创建密钥
        DWORD dwXorKey=m_dwSendXorKey;
        if (m_dwSendPacketCount==0)
        {
                //生成第一次随机种子
                GUID Guid;
                CoCreateGuid(&Guid);
                dwXorKey=GetTickCount()*GetTickCount();
                dwXorKey^=Guid.Data1;
                dwXorKey^=Guid.Data2;
                dwXorKey^=Guid.Data3;
                dwXorKey^=*((DWORD *)Guid.Data4);

                //随机映射种子
                dwXorKey=SeedRandMap((WORD)dwXorKey);
                dwXorKey|=((DWORD)SeedRandMap((WORD)(dwXorKey>>16)))<<16;
                dwXorKey^=g_dwPacketKey;
                m_dwSendXorKey=dwXorKey;
                m_dwRecvXorKey=dwXorKey;
        }

        //加密数据
        WORD * pwSeed=(WORD *)(pcbDataBuffer+sizeof(TCP_Info));
        DWORD * pdwXor=(DWORD *)(pcbDataBuffer+sizeof(TCP_Info));
        WORD wEncrypCount=(wEncryptSize+wSnapCount)/sizeof(DWORD);
        for (i=0;i<wEncrypCount;i++)
        {
                *pdwXor++^=dwXorKey;
                dwXorKey=SeedRandMap(*pwSeed++);
                dwXorKey|=((DWORD)SeedRandMap(*pwSeed++))<<16;
                dwXorKey^=g_dwPacketKey;
        }

        //插入密钥
        if (m_dwSendPacketCount==0)
        {
                MoveMemory(pcbDataBuffer+sizeof(TCP_Head)+sizeof(DWORD),pcbDataBuffer+sizeof(TCP_Head),wDataSize);
                *((DWORD *)(pcbDataBuffer+sizeof(TCP_Head)))=m_dwSendXorKey;
                pHead->TCPInfo.wPacketSize+=sizeof(DWORD);
                wDataSize+=sizeof(DWORD);
        }

        //设置变量
        m_dwSendPacketCount++;
        m_dwSendXorKey=dwXorKey;
        return wDataSize;
}
```

```cpp
//解密数据

WORD CTCPSocket::CrevasseBuffer(BYTE pcbDataBuffer[], WORD wDataSize)

{
        //效验参数
        ASSERT(m_dwSendPacketCount>0);
        ASSERT(wDataSize>=sizeof(TCP_Head));
        ASSERT(((TCP_Head *)pcbDataBuffer)->TCPInfo.wPacketSize==wDataSize);

        //调整长度
        WORD wSnapCount=0;
        if ((wDataSize%sizeof(DWORD))!=0)
        {
                wSnapCount=sizeof(DWORD)-wDataSize%sizeof(DWORD);
                memset(pcbDataBuffer+wDataSize,0,wSnapCount);
        }

        //解密数据
        DWORD dwXorKey=m_dwRecvXorKey;
        DWORD * pdwXor=(DWORD *)(pcbDataBuffer+sizeof(TCP_Info));
        WORD  * pwSeed=(WORD *)(pcbDataBuffer+sizeof(TCP_Info));
        WORD wEncrypCount=(wDataSize+wSnapCount-sizeof(TCP_Info))/4;
        for (WORD i=0;i<wEncrypCount;i++)
        {
                if ((i==(wEncrypCount-1))&&(wSnapCount>0))
                {
                        BYTE * pcbKey=((BYTE *)&m_dwRecvXorKey)+sizeof(DWORD)-wSnapCount;
                        CopyMemory(pcbDataBuffer+wDataSize,pcbKey,wSnapCount);
                }
                dwXorKey=SeedRandMap(*pwSeed++);
                dwXorKey|=((DWORD)SeedRandMap(*pwSeed++))<<16;
                dwXorKey^=g_dwPacketKey;
                *pdwXor++^=m_dwRecvXorKey;
                m_dwRecvXorKey=dwXorKey;
        }

        //效验码与字节映射
        TCP_Head * pHead=(TCP_Head *)pcbDataBuffer;
        BYTE cbCheckCode=pHead->TCPInfo.cbCheckCode;
        for (int i=sizeof(TCP_Info);i<wDataSize;i++)
        {
                pcbDataBuffer[i]=MapRecvByte(pcbDataBuffer[i]);
                cbCheckCode+=pcbDataBuffer[i];
        }
        if (cbCheckCode!=0) throw TEXT("数据包效验码错误");
        return wDataSize;
}
```

上面代码是从网狐的源码里面摘选出来的 他们默认的加密和解密秘钥是：`0X12345678` 这种加密方式和解密方式很简单安全性呢说呢如果数据被拦截 比如通过设备拦截数据是可以再中间加密解密出来的，就是说如果有人想从数据方面 至少是可以把数据给拦截下面并且知道里面具体是什么数据，在安全性方面并不能够达到特别安全‘

然而对于贝贝游戏，贝贝游戏源码应该是从之前998游戏手上购买的，他们的加密形式比较复杂，因为我手上并没有贝贝游戏的源码 我只能通过逆向简单说一下他们的加密方式

加密方式主要有AES加密

我找了一个AES 加密解密的DEMO 截图

![](https://bbs.kanxue.com/upload/attach/201510/383806_130b650f767dfd44.png)

还有一种是RSA加密

(RSA是目前最有影响力的公钥加密算法，它能够抵抗到目前为止已知的绝大多数密码攻击，已被ISO推荐为公钥数据加密标准。)

可见他们加密的严密程度是非常严的，可以断定的说即使你在中途把数据拦截下来你如果没有私钥你也没法破解出秘钥的，因为对于RSA私钥只会保存到游戏服务器端，其他地方均不会出现。

下面是百度RSA加密解密处理流程

![](https://bbs.kanxue.com/upload/attach/201510/383806_17a2d71d8a79354b.png)

大概理论知识说得差不多了，接下来我们主要给大家理清一下这个棋牌游戏他是如何和服务器通信的

大概步骤是这样的

1：第一步 客服端和服务器建立起TCP 链接，3次握手后发送数据给服务器端，第一个发送的数据是AES的加密密钥，这个密钥是随机生成的，以后在这个SOCK连接中一直是用这个秘钥进行通信的，客服端先随机生成一个秘钥然后通过用RSA的方法+RSA公钥加密 把加密数据发送到服务器端

服务器端通过RSA私钥把发送过来的数据解密出来，解密出来的即是AES的密钥，这个密钥是既作为在这个SOCK断开之前一直用的。到目前为止RSA加密就到此为止了，之后RSA即派不上用场了，所以RSA算法只是作为一个加密密钥的作用，那有人会问这样安全吗？说实话正是因为用RSA算法才让整个接下来的数据变得安全，因为一般的加密算法都是一个密钥，客服端要加密这个密钥肯定会在客服端出现，然后RSA机制是公钥在客服端，然后私钥只在服务端，即保证了在传送秘钥的过程中即使数据被拦截人家要解密不出来密钥，还有人会问既然这么安全为什么不一致都用RSA呢，其实很简单RSA加密算法安全其实是建立在他复杂上面，既然复杂他的处理流程是很慢的，加密解密速度是很慢的，尤其是作为游戏如果光一个数据加密解密就需要很长时间，那么游戏之前通信如此频繁游戏肯定会感觉很卡，因为数据不能及时被处理。所以他只是在建立SOCK的时候调用一下这个算法

2：第二步 既然客服端随机生成的秘钥已经通知到服务器端了，那么他们之后的数据都是用AES 这种算法速度很快，而且在不知道密钥的情况下也是很安全的，而这个密钥是随机的，所以你说安全吗

贝贝游戏AES加密算法调用处

![](https://bbs.kanxue.com/upload/attach/201510/383806_57d8a7177c796b84.png)

贝贝游戏AES解密算法调用处

![](https://bbs.kanxue.com/upload/attach/201510/383806_c0006785e40f8062.png)

上面截图是在贝贝游戏中AES加密解密的地方。

大概贝贝的数据加密解密就是通过上面2个步骤，所以说贝贝游戏在数据安全性方面还是很不错的，至少个人感觉很严密。

上面大致阐述了一下贝贝游戏的在通信方面的方式方法，里面可能有很多因为自己能力有限有错误的地方或者有限的地方请自动忽略，个人微信feilongqp  欢迎加我

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
