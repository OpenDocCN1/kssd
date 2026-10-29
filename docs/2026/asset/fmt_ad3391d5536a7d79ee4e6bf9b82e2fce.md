
终于，我们看到了`env`和`envkey`是这样赋值的，但这并没有结束，因为我们知道，它传入了一个`string`数组进去处理。所以即使我们知道了`env`和`envKey`的加密方法，但还需要知道传进去的字符串数组，也就是`hashedEnvFactors`是从何而赋值的。

## 0x05 env算法分析

我们暂且不管`hashedEnvFactors`从何而来，先看看`env`的算法

|  |  |
| --- | --- |
| 1 2 | `PassportEnvEncryptUtils.EncryptResult encrypt` `=` `PassportEnvEncryptUtils.encrypt(strArr);` `easyMap.easyPutOpt(com.xiaomi.verificationsdk.internal.Constants.e, encrypt.content);` |

我们看到，它是调用`PassportEnvEncryptUtils.encrypt(strArr);`直接返回的`encrypt`，并且直接取出`encrypt`里的`content`（对应`env`）和`encryptedKey`（对应`envKey`）得到的结果。所以直接跟进`encrypt`方法里面查看

第一层跟进

![](https://bbs.kanxue.com/upload/attach/202103/885524_GB2HERXNMSYQCR5.png)

第二层跟进

![](https://bbs.kanxue.com/upload/attach/202103/885524_ZPH4N2MZENAS6W6.png)

发现一个常量，去看下是什么值

![](https://bbs.kanxue.com/upload/attach/202103/885524_JUZ4X2AEHDXA8GM.png)

发现值：`"0102030405060708"`，继续返回跟进

![](https://bbs.kanxue.com/upload/attach/202103/885524_FV8GYW8MURP4MNP.png)

最终我们看到它是进行的一个`ASE`算法，整个结果其实也是相当简单的了，我们把代码抠出来`VSCODE`分析下。

![](https://bbs.kanxue.com/upload/attach/202103/885524_VCYXBAJ88DAWDRH.png)

首先参数1则是我们第一层看到它，他是把`hashedEnvFactors`数组用：分割连接起来了，所以我们写死他。

而参数2就是第三层看到的`DEFAULT_IV`，我们也写死

![](https://bbs.kanxue.com/upload/attach/202103/885524_PU6D5QYGV6EGB67.png)

所以它的传参应该是这样的；接着我们可以看到，它`new`了自己封装的一个`ASE`的`EncryptResultWithIv`类，然后赋值参数2给`iv`，估计就是加密的`KEY`。

|  |  |
| --- | --- |
| 1 | `SecretKey generateSymmetricKey` `=` `generateSymmetricKey();` |

关键在于它又调用了这行代码，`generateSymmetricKey()`，我们进去看一下~

![](https://bbs.kanxue.com/upload/attach/202103/885524_KWH7VWQXUVQEGJE.png)

好像也并不是很关键0.0，就是随机初始化了一个`ASE`然后返回它的`key`。那么我们继续阅读刚刚的代码，因为它连接的有一点长，我们把他格式处理一下。

![](https://bbs.kanxue.com/upload/attach/202103/885524_45QCBFEM3VP5EGD.png)

因为这里的计算感觉有点绕，前3行代码图片已经打出注释了，我们看下第接下来一行就是赋值跟`env`结果的关键代码，它又跑去执行了一个`aesEncrypt`方法，并且我们看到传入的正是`hashedEnvFactors`以及随机证书的`KEY`以及刚刚看到的`0102030405060708`的`IV`进行调用的，所以我们还需要跟进`aesEncrypt`查看一下都是怎么处理的。

![](https://bbs.kanxue.com/upload/attach/202103/885524_MR9YWQTR6AERF3X.png)

这里我们看到它以随机生成的证书作为构造类定义，然后在传入要加密的`hashedEnvFactors`以及`IV`，我们在进去看看。

![](https://bbs.kanxue.com/upload/attach/202103/885524_P2SJJKZUSEYMAWR.png)

这里跟进去后发现就是调用`encryptWithIv`进行`ASE`签名的，然后在转成`BASE64`完成`env`的加密！！

## 0x06 总结

至此我们已经完成了4项参数的算法~

`HASH` = 大写`MD5`(密码)

`_SIGN` = 来自服务器返回

`ENVKEY` = `BASE64`编码(`RSA`(一个随机证书,公共密钥))

`ENV` = `BASE64`编码(`ASE`(`hashedEnvFactors`,"`0102030405060708`"))

这些完全可以直接扣掉它自己定义的`ASE`类来放到`JAVA`独立调用的，就不再作演示，由于文章挺长了，下次在讲一讲`hashedEnvFactors`这10几个字符串的定义来源

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)
