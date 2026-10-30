
此处可以看到，`accountContext.bitmapCurrencyId`已经被设置成了1，且再次调用`getAccount()`，可以通过代码知道`accountBalances[0]`的值已经被赋值，因为此处我们只enable了bitmap，没有deposit任何代币，所以此处`accountBalances[0]`为`(1, 0, 0, 0, 0)`是没有任何问题的。

![](https://bbs.kanxue.com/upload/attach/202202/811277_PYR7A7T2PBE67CD.png)

2. 此时在进行第二步操作，利用`depositUnderlyingToken()`向我们个人地址中deposit代币，此处用DAI作为例子，DAI的`currencyId`为2，这个可以直接通过代理合约调用接口查看。

```
[(1641427200, b'\x00', 0, 1, b'@\x02\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00'), [(1, 0, 0, 0, 0), (2, 18344299339310, 0, 0, 0), (0, 0, 0, 0, 0), (0, 0, 0, 0, 0), (0, 0, 0, 0, 0), (0, 0, 0, 0, 0), (0, 0, 0, 0, 0), (0, 0, 0, 0, 0), (0, 0, 0, 0, 0), (0, 0, 0, 0, 0)], []]
```

在充值完成后，查看账户信息，发现

`accountContext.activeCurrencies`被赋值`b'400200000000000000000000000000000000'`

`accountBalances[1]`被赋值为了`(2, 18344299339310, 0, 0, 0)`

`accountContext.activeCurrencies`变量的修改，来自于`depositUnderlyingToken()`中的`balanceState.finalize(account, accountContext, false)`，可以跟进看看

![](https://bbs.kanxue.com/upload/attach/202202/811277_RQJT5F8RCJC2QEN.png)

在`.finalize(account, accountContext, false)`中`accountContext.setActiveCurrency`会将`accountContext.activeCurrencies`修改。但这不是重点。

![](https://bbs.kanxue.com/upload/attach/202202/811277_W63ED7VJ7Y4W3X8.png)

![](https://bbs.kanxue.com/upload/attach/202202/811277_MSAK5JASN5QRDW4.png)

此处查看`getAccount()`方法，如果`accountContext.activeCurrencies`存在的话，会去从存储中读取该代币的值，`accountBalances = new AccountBalance[](10);`，`accountBalances`是一个长度为10数组，如果开始不是很清楚的话为啥是10的话，这个地方就能非常明白，在开启bitmap currency时，第一个数组是用来放账户中ETH的balance数据的，后面的9个数组是用来放其中支持的9种代币的balance数据的，`bytes18`即每2个字节代表一个代币。

![](https://bbs.kanxue.com/upload/attach/202202/811277_59CS84RB9W6G6Q2.png)

到此，程序都是正常运行的，我们enable了`currencyId == 1`的ETH，但是没有充值，所以ETH的balance数据为`(1, 0, 0, 0, 0)`，第二步我们充值了DAI，所以DAI的balance为`(2, 18344299339310, 0, 0, 0)`，这些都没有问题。

3. 接下来进行第三步，再次`enableBitmapCurrency()`，此时将DAI的`currencyId`作为参数。执行完成后，查看`Account`。此时发现`accountBalances`的前两个数组的值变成一样了，也就是说ETH所在的balance被DAI的balance覆盖了。

```
[(1641427200, b'\x00', 0, 2, b'@\x02\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00'), [(2, 18344299339310, 0, 0, 0), (2, 18344299339310, 0, 0, 0), (0, 0, 0, 0, 0), (0, 0, 0, 0, 0), (0, 0, 0, 0, 0), (0, 0, 0, 0, 0), (0, 0, 0, 0, 0), (0, 0, 0, 0, 0), (0, 0, 0, 0, 0), (0, 0, 0, 0, 0)], []]
```

同时查看：`free_collateral: [212752332, [18344299339310, 18344299339310, 0, 0, 0, 0, 0, 0, 0, 0]]`，发现价值翻倍了。

再来看看`getAccount()`中设置ETH balance的代码，如果`accountContext.isBitmapEnabled()`，则会以`bitmapCurrencyId`所代表的代币balance来赋值到`accountBalances[0]`。OK，问题就在这里，也就是说通过修改`bitmapCurrencyId`的值能覆盖ETH所代表的balance位的值，实现double counting。

![](https://bbs.kanxue.com/upload/attach/202202/811277_RXAPB4ZFEH87AU7.png)

## 漏洞复现

对于漏洞的复现，其实步骤很简单。

源码：

```
31bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6F1L8%4c8A6L8$3&6S2L8q4)9J5k6r3k6A6L8X3q4F1j5$3g2Q4x3V1k6U0L8$3&6@1M7X3q4U0N6s2y4Q4x3X3c8$3x3R3.
```

```
npx hardhat node --fork da0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6r3S2Q4x3X3c8E0j5h3W2F1L8X3g2@1i4K6u0W2j5h3I4U0K9r3g2E0P5h3q4H3K9g2)9J5k6h3W2G2i4K6u0r3N6U0u0Q4x3V1k6&6L8%4g2J5i4K6g2X3K9$3g2&6 --fork-block-number 13950000
```

1. `enableBitmapCurrency(1)` //启用bitmap，将ETH设置为bitmapCurrency

2. `depositUnderlyingToken(useraddr,2,amount)` //充值DAI

    此处如果没有DAI，需要先到swap购买DAI，然后`approve` notional的代理合约地址。

3. `enableBitmapCurrency(2)` //启用bitmap，将DAI设置为bitmapCurrency

此3步就能完全复现漏洞。

具体的POC，我是直接用python web3调用的，也可以自己构造或者用contract实现。

这是我临时用的py测试脚本，可以参考。
