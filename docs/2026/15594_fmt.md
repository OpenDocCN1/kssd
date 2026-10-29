
# [原创]记一次基于Soket通信的app的分析

## 1.App介绍

该App是一个股票数据查看软件，可以查看股票的交易数据，K线

![数据](https://bbs.kanxue.com/upload/attach/202004/821869_3FQMQA7CC2GZWTQ.jpg)

![分时线](https://bbs.kanxue.com/upload/attach/202004/821869_PJ9NY2B7CQHB4FY.jpg)

![K线](https://bbs.kanxue.com/upload/attach/202004/821869_VKJ6F4VXDX2MVQF.jpg)

## 2.APP分析

**由于APP逻辑复杂故只列出有用部分**

### 2.1.`MyApp->onCreate`

APP经过加壳，脱壳后使用`MT`绕过签名效验重打包成功

![壳](https://bbs.kanxue.com/upload/attach/202004/821869_DUSB29WAHJSRE5B.jpg)

在自定义的`Application`的`onCreate`方法中进行一些数据的初始化

![图片.png](https://bbs.kanxue.com/upload/attach/202004/821869_RZ64TBQ6J3Y4X35.jpg)

读取配置文件中的配置

![图片.png](https://bbs.kanxue.com/upload/attach/202004/821869_UMNJHAK5MBRAPGR.jpg)

解析`assets`文件夹中的`MbUserMarket.xml`文件存入`mPBMarketArray`和`mHomeMenuMarkets`

![图片.png](https://bbs.kanxue.com/upload/attach/202004/821869_44WZCGCW7FV4HE3.jpg)

解析文件`pbtransclnt.xml`存入`CPDDataDecode`的静态数组`g_vDataPackage`设置so库的`DesKey`

![图片.png](https://bbs.kanxue.com/upload/attach/202004/821869_YBYTCEDF8NXEBXE.jpg)

读取`main.cfg`和`hq_addr.ini`文件中的服务端地址

![图片.png](https://bbs.kanxue.com/upload/attach/202004/821869_UGG7MRF3TTWFF27.jpg)

### 2.2.`SplashActivity->onCreate`

直接发送`handlerMessage`

![图片.png](https://bbs.kanxue.com/upload/attach/202004/821869_KGAD42MW4MNYB8M.jpg)

对应操作

![图片.png](https://bbs.kanxue.com/upload/attach/202004/821869_DWWY37HXVWSC3UB.jpg)

在`initCodeTable()`中尝试从文件中加载数据

![图片.png](https://bbs.kanxue.com/upload/attach/202004/821869_XKRUBT7EAK7QHW5.jpg)

`doLogin()`进行登录请求

![图片.png](https://bbs.kanxue.com/upload/attach/202004/821869_H8P9XQ54GQ9UPWP.jpg)

然后将参数编码加密后调用`addSendData`方法

![图片.png](https://bbs.kanxue.com/upload/attach/202004/821869_N9EDZWSA2AJS5EQ.jpg)

编码方式为:将参数用`|`分隔然后在转换为字节数组

![图片.png](https://bbs.kanxue.com/upload/attach/202004/821869_VVSH5BEFGQHG3AD.jpg)

![图片.png](https://bbs.kanxue.com/upload/attach/202004/821869_BQJC9MJ3XSJXEBR.jpg)

加密方式根据参数确定是否`DES`加密，`Zlib`压缩，然后将发送的数据的信息存入前12位

![图片.png](https://bbs.kanxue.com/upload/attach/202004/821869_VT54F3UGEG96NWC.jpg)

完成后将数据加入到`GlobalNetConnect`的`NetSendThread`

如果是`mHQPushNet`则使用第二个端口

![图片.png](https://bbs.kanxue.com/upload/attach/202004/821869_YEGRADHAE5UJNNT.jpg)

然后启动`NetSendThread`线程（`run`方法代码较长只列出关键部分）

首先读取在`mApp`中加载的服务器地址读取后直接尝试连接，删除连接成功外的所有地址然后检查欲发送的数据是否合法

![图片.png](https://bbs.kanxue.com/upload/attach/202004/821869_7DWQU9WTBPVRESS.jpg)

上图 连接成功后将成功的地址记录

![图片.png](https://bbs.kanxue.com/upload/attach/202004/821869_YME9WU6HD2JES4G.jpg)

然后将欲发送数据写入`SendByteBuffer`

![图片.png](https://bbs.kanxue.com/upload/attach/202004/821869_FVEG95P3QBZMAS2.jpg)

然后写入`Socket`

![](https://bbs.kanxue.com/upload/attach/202004/821869_5UGA3V9P4C3RGSN.jpg)

再从`Socket`中读取回复

![图片.png](https://bbs.kanxue.com/upload/attach/202004/821869_JD9BM3EAB3HD89W.jpg)

回复的数据解密（也就是刚才分析的加密的逆过程）

解析完成后如果设置了接受消息的`Handler`，则会发送消息

![图片.png](https://bbs.kanxue.com/upload/attach/202004/821869_VRJSC64FMKS2KCG.jpg)

登录成功后 请求`MarketInfo`，并根据第一次登录返回，用另一网络线程登录

（请求过程与登录过程类似 故不再诉）

![图片.png](https://bbs.kanxue.com/upload/attach/202004/821869_J5JNSCFBYE7VQQ5.jpg)

请求`MarketInfo`成功后 解析`MarketInfo`并请求`CodleTable`

![图片.png](https://bbs.kanxue.com/upload/attach/202004/821869_FURBR4NWPG6TAY6.jpg)

解析成功后记录数据 并保存为文件

![图片.png](https://bbs.kanxue.com/upload/attach/202004/821869_YN6GYB44HAKFYH9.jpg)

### 2.3.其它

其它的数据请求与上述过程类似，不再重复

## 3.模拟客户端获取数据

既然已经了解客户端与服务端的通信过程，则可以模拟客户端对服务器收发数据

![图片.png](https://bbs.kanxue.com/upload/attach/202004/821869_MVCQ2D7JH5GZXZJ.jpg)

首先模仿App的编码加密过程将预发送的数据编码加密

![图片.png](https://bbs.kanxue.com/upload/attach/202004/821869_PHPZA2QUMK62AUP.jpg)

然后开启新线程将数据发送到服务端

![image.png](https://bbs.kanxue.com/upload/attach/202004/821869_93SR96UE3ACNJKB.jpg)

接收返回的数据并解析

![image.png](https://bbs.kanxue.com/upload/attach/202004/821869_WMR9QVXUZQHEPUJ.jpg)

解析返回的数据并打印出来

![image.png](https://bbs.kanxue.com/upload/attach/202004/821869_2QDTAEV3SYDT64M.jpg)

即可获取数据

![image.png](https://bbs.kanxue.com/upload/attach/202004/821869_ZTEJJPWA5GHTVYU.jpg)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
