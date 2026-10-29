
# [原创]X-Bogus vmp分析

## 第一步

1、简单看一下变量被压缩了，用v佬的`jstools`插件还原一下变量混淆，后续容易分析一点 看一下接口的堆栈

![图片描述](upload/attach/202304/921346_8M2U7XA6YDD3MZV.png)

2、先下`W`断点，发现断下的时候参数已经拼接上去了，`vmp`肯定在循环解释运行，直接先打断点步过执行。

![图片描述](upload/attach/202304/921346_8FB9NFPH3F2XMRY.png)

3、边执行边观察右侧的变量区域，有一个变量`O`一直在变，这个大概率就是整个流程存储的寄存器，看到`DFSz`就是这个`bogus`的前缀。

![图片描述](upload/attach/202304/921346_QGTSZETTC2PCHHU.png)

4、直接下日志，打印`O`，保存到本地分析，这只是外层日志。

![图片描述](upload/attach/202304/921346_BZE4RM9HSSMZB2T.png)

5、如果看详细的内容还得在`vmp`流程里再下一个，在这个加密流程里步入发现会进到主流程里，在这个下一个日志点方便分析，观察`O`的变化就完了。（最好加上一个详情接口的`xhr`断点，不然这个日志会很多）流程找对了，下手点就是把这个流程先切开，分析

![图片描述](upload/attach/202304/921346_86UHRBJR7W5JYXV.png)

6、这是已经出现了`DFSz`的第一条日志

> V M198048:1 [null,[null,{},null,null,"\u0002ÿ-%.$ú{^1ì÷FË¢\u000f÷°Q","s2","="",{"s0":"ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=","s1":"Dkdpgh4ZKsQB80/Mfvw36XI1R25+WUAlEi7NLboqYTOPuzmFjJnryx9HVGcaStCe=","s2":"Dkdpgh4ZKsQB80/Mfvw36XI1R25-WUAlEi7NLboqYTOPuzmFjJnryx9HVGcaStCe="},"Dkdpgh4ZKsQB80/Mfvw36XI1R25-WUAlEi7NLboqYTOPuzmFjJnryx9HVGcaStCe=","DFSz",196397,4],null,"\u0002ÿ-%.$ú{^1ì÷FË¢\u000f÷°Q",[3],[null,{},null,null,"\u0002ÿ-%.$ú{^1ì÷FË¢\u000f÷°Q","s2","="",{"s0":"ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=","s1":"Dkdpgh4ZKsQB80/Mfvw36XI1R25+WUAlEi7NLboqYTOPuzmFjJnryx9HVGcaStCe=","s2":"Dkdpgh4ZKsQB80/Mfvw36XI1R25-WUAlEi7NLboqYTOPuzmFjJnryx9HVGcaStCe="},"Dkdpgh4ZKsQB80/Mfvw36XI1R25-WUAlEi7NLboqYTOPuzmFjJnryx9HVGcaStCe=","DFSz",196397,4],4,3]

7、这是上一条日志，我们先不管`bogus`怎么生成的，先往上分析，找到最开始的点再往下走

> [null,[null,{},null,null,"yso¨ÉHðr`\u000e1fã6(MÉXì\u001f\u0000N úg´f&o¿Ö\u000fÔº¤FWåà$\u001b3A`ÊàÿÙk\u001e<Ì93\u001a\u000fö\u0000\u0006a²9Ê\u0002ç?8\u0001;/·\u0010m\r-i'j|B\u000f+ÜÁ»\u0003ëô\u001e{dÇ­\u000fF^\u0004¥;Jä·q[","s0","="",{"s0":"ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=","s1":"Dkdpgh4ZKsQB80/Mfvw36XI1R25+WUAlEi7NLboqYTOPuzmFjJnryx9HVGcaStCe=","s2":"Dkdpgh4ZKsQB80/Mfvw36XI1R25-WUAlEi7NLboqYTOPuzmFjJnryx9HVGcaStCe="},"ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=","eXNvqMlI8HJgDjFm4zYoTclY7B8ATqD6Z7RmJm+ev9YPl9S6pEZX5Z3gJBszQWCayuD/2WsegTzMOTMagA/2AAZhsjnKAuc/OAE7L7eYiBBtDS1pJ2p8QoIPK9zBuwPr9B57ZMetjIkPRl4Ei6U7Spfkt3F",12022107,117],"eXNvqMlI8HJgDjFm4zYoTclY7B8ATqD6Z7RmJm+ev9YPl9S6pEZX5Z3gJBszQWCayuD/2WsegTzMOTMagA/2AAZhsjnKAuc/OAE7L7eYiBBtDS1pJ2p8QoIPK9zBuwPr9B57ZMetjIkPRl4Ei6U7Spfkt3F",null,"ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=",[27],63,117]

8、看到其中有一个值是`s0`，后面的`map`里面就有`s0`，就是一个`base64`的编码表，顺序也是正常的，这个流程块一看就是在做`base64`，直接略过不`trace`了，直接在控制台`btoa`一下那个字节

![图片描述](upload/attach/202304/921346_467P73AWJU87Z9P.png)

![图片描述](upload/attach/202304/921346_6QRHFEAQ9F2VZPG.png)

9、现在就看`O[1][4]`这个字节怎么来的了，看`log`的流程出现了`__reactEvents$t3n7f43w7o`那行下一条就出现了这个字节，所以说加密流程就在这两条中间，但这周我运行日志这条就没了，这个不重要，我们直接在`P.apply`下条件断点，让`O[2] == "execCommand"`，然后把条件断点换成普通断点，步过一下，看到详细日志有很多输出，大致看一下这就是`base64`那一段往上翻一下看一下这个字节的来源

![图片描述](upload/attach/202304/921346_RREBU2TBQBCD5ZZ.png)

10、搜索一下第一次出现的位置，看一下这个分支数是`136 0`，那就断上一个，上一个是`208 0`，但是下条件断点的时候要带上`O`里面的一些数据，反正没出现过的最好，因为`vmp`的流程循环光断分支数没什么鸟用

![图片描述](upload/attach/202304/921346_AVV8H4CSA8S7RMM.png)

11、去把那个`json`解一下，`O[4]`数组是`['\x00\x01\x0c', 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Safari/537.36']`，

所以断点条件

`E == 208 && P == 0 && O[4][1] == 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Safari/537.36'`

再把别的断点都取消重新执行到这，然后单步调试走到了`P.apply`

![图片描述](upload/attach/202304/921346_F78NMPWEUPB8T9R.png)

12、然后看一下`p`步进看一下，很明显的`rc4`，`a`是`key`，`b`是`data`，到这上面的流程就是`ua`的`rc4`转的`base64`

直接破案，但这个`a`上周我调试的时候发现跟这周不一样，他是会变的，所以根据刚才的日志在往上找一下`a`怎么来的

![图片描述](upload/attach/202304/921346_W76JV29FEP7S4X4.png)

找`key`的流程和上面是一样的就不过多叙述了，也是找`\x00\x01\x0c`第一次生成的地方，然后往上推，最后发现`O[4]`是`[1,12]`进到了一个函数里，最后出来的结果就是`key`。在找这两个数对应的是`ubcode`和`envcode`，看一下`ubcode`居然是鼠标轨迹进行运算出来的，很离谱唉，这样看来这玩意大概率不是很重要，先不管了看看这个`key`后续有没有用上，先给固定住吧。

## 第二步

1、接下来的`trace`就不贴流程了，流程跟上面是一样的

接下来就是分析`base64`之后这个字节数据怎么来的了，直接在外层设置条件断点，然后分析`vmp`内层日志

![图片描述](upload/attach/202304/921346_5YYSVPJ22RMUFFS.png)

2、因为每次都刷新一次流程，数据不对应很正常。`\u0002`开头的就是`bogus`最后的数据，上面的数组`[2,255,"-%.$Yu^\x8F1ì÷\x9Aja¥IñMÄ"]` `trace`后就是`bytes`了`2`和`255`作为填充加上后面的数据

![图片描述](upload/attach/202304/921346_4KKX9W9N74M5ZAF.png)

3、`trace` `"ÿ"`这一行，也是`rc4`了一下，`"ÿ"`先固定住，调试这么多次它都是一样的

![图片描述](upload/attach/202304/921346_Y4ZXQT4WHBW2AYZ.png)

4、然后继续`trace`，进入到一个函数里面的内容就是把红框的数组都填充到字节上

![图片描述](upload/attach/202304/921346_3D6VBMY99CPZTMR.png)

5、重点就是分析这个数组怎么来的，根据上面的图片的梯型一步一步`trace`，找到开始的流程打码的是`canvas hash` ![图片描述](upload/attach/202304/921346_WXJMXMYV4WXDXDS.png)

6、`python` 还原之后如下

`salt_payload` 是 `payload`的`md5` 转 十六进制字节 在`md5`在转十六进制字节

`salt_`是空字符串 同上

`salt_ua`是最开始分析的`base64`的`md5`转十六进制字节

这些`salt`都是取后两位做校验

时间戳和`canvas`都是取高低8位与255

![图片描述](upload/attach/202304/921346_344M85Y9MXWVAW7.png)

最后还有一个校验位，从第一个开始一次往下乘方运算

![图片描述](upload/attach/202304/921346_7H3WNMXZ2QW6862.png)

然后是最后取偶数位在前，奇数位在后组成的最后的数组

![图片描述](upload/attach/202304/921346_Z8823A6YPXREMMK.png)

![图片描述](upload/attach/202304/921346_8JTHHNG5BMMDF3J.png)

到现在为止`\u0002`开头的字节就还原出来了

7、剩下就是`bogus`流程了，这里的字符串变成了`s2`，说明最后会在`s2`里面取字符串拼接

![图片描述](upload/attach/202304/921346_CXXQFHG3EYRVHFN.png)

运算流程多出了个`2`，直接在上面那条日志下断

![图片描述](upload/attach/202304/921346_VZMVDSXFT68SEGD.png)

8、发现解释器出来一个`charcodeat`， 单步执行到最后就是 字节.`charcodeat`(索引)。

上图的索引不是运算时候需要的索引，只是解释器将要取值放置的，不要混淆，`[0]`才是真正运算的索引。

![图片描述](upload/attach/202304/921346_GF86CGVJ3UJ85H9.png)

索引`0`的结果就是`2`，下面的流程依然还是 `& 255 << 16`。

索引`1`就不一样了，他是 `&255 << 8`。

而且红框的最后一步变成了`196352`，这时候得下断点看一下，条件直接下`O[2] == 131072 && O[3] == 65280`，因为字节的最后一步就是填充`2`，`255`, 值是固定的。

单步调试最后发现`196352 = 131072 | 65280`，做了一个位运算

继续分析发现索引为 `2`的时候多了`16515072`，还是老办法`trace`

![图片描述](upload/attach/202304/921346_DN8REE66RR4FNKQ.png)

发现是`vmp`解释出来的，是固定的数，然后继续单步调试

![图片描述](upload/attach/202304/921346_CSADHF9XFXJABQQ.png)

`0 = 196397 & 16515072`，

走到这里发现第一个字符串`D`已经出来了，索引就是上面运行的值，断点走到这流程就是

`'Dkdpgh4ZKsQB80/Mfvw36XI1R25-WUAlEi7NLboqYTOPuzmFjJnryx9HVGcaStCe='.charAt(0)`

![图片描述](upload/attach/202304/921346_XFPCNT6FTANWZU5.png)

然后是第二个字符串`F`，这里多了`258048`，老办法条件断，发现也是`vmp`解释出来的

`192512 = 196397 & 258048`

但发现下面取索引的时候变成了`47`，原来是漏掉了一步，

`192512 >> 12`

那这样上面的不是直接等于`0`的是，`0 >> 18`，这个`18`和`12`都是解释器固定的值

到现在字符串`F`就出来了

![图片描述](upload/attach/202304/921346_S2AUEU5BSHXU8NS.png)

第三个字符串也是

`60 = （196397 & 4032）>> 6`

![图片描述](upload/attach/202304/921346_QKQQFGQUDH4528K.png)

第四个字符串

`45 = 196397 & 63`

发现些许不同，没有 `>>` 这一步了

![图片描述](upload/attach/202304/921346_8ZCMB9A9Q2HZ77X.png)

在看后续的日志内容跟这个是一样的，三个索引一循环，算法一致，后面直接还原就行

![图片描述](upload/attach/202304/921346_ANDFGDNTCT53PZY.png)

校验一下，跟网页结果一致，搞定收工

![图片描述](upload/attach/202304/921346_YPB4XMYCMSPPMQJ.png)

![图片描述](upload/attach/202304/921346_J6Z8WJHTN3RQXM7.png)

总结一下，`py`还原之后算法也就`100`行，`vmp`要找到技巧还原起来会事半功倍，其实大部分操作都是在做解释执行，有变量变化的时候才是真的在做真实操作。

## 第一步

1、简单看一下变量被压缩了，用v佬的`jstools`插件还原一下变量混淆，后续容易分析一点 看一下接口的堆栈

![图片描述](https://bbs.kanxue.com/upload/attach/202304/921346_8M2U7XA6YDD3MZV.png)

2、先下`W`断点，发现断下的时候参数已经拼接上去了，`vmp`肯定在循环解释运行，直接先打断点步过执行。

![图片描述](https://bbs.kanxue.com/upload/attach/202304/921346_8FB9NFPH3F2XMRY.png)

3、边执行边观察右侧的变量区域，有一个变量`O`一直在变，这个大概率就是整个流程存储的寄存器，看到`DFSz`就是这个`bogus`的前缀。

![图片描述](https://bbs.kanxue.com/upload/attach/202304/921346_QGTSZETTC2PCHHU.png)

4、直接下日志，打印`O`，保存到本地分析，这只是外层日志。

![图片描述](https://bbs.kanxue.com/upload/attach/202304/921346_BZE4RM9HSSMZB2T.png)

5、如果看详细的内容还得在`vmp`流程里再下一个，在这个加密流程里步入发现会进到主流程里，在这个下一个日志点方便分析，观察`O`的变化就完了。（最好加上一个详情接口的`xhr`断点，不然这个日志会很多）流程找对了，下手点就是把这个流程先切开，分析

![图片描述](https://bbs.kanxue.com/upload/attach/202304/921346_86UHRBJR7W5JYXV.png)

6、这是已经出现了`DFSz`的第一条日志
