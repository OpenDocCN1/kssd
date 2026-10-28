
第一个红字的地方说明我们已经把`Import Table`的首址改到了`1a0250`长度是`190`了。第二个红字的地方说明新的`Import Table`的表项数据是可靠的。

好了，最关键的最激动人心的时候到了，让我们真正为这个新的`Import Table`添加表项，添加表项必须是要建立在你已经完全对`Import Table`结构了解的基础上，输入的过程就是输入一些16进制数和字符串。下面是输入好后的屏幕切图，我将结合该图为大家讲解。![](https://www.kanxue.com/patch_2_2.gif)

好了，大家已经看到了，这是添加好表项以后的`Import Table`，其中`1a0250`到`1a03b8`和以前的`Import Table`一模一样。以前的`Import Table`在`1a03b8`以后是20个字节的全零结束符。`1a03b9`到`1a03cc`是我们添加的表项，后面的20个字节是新的结束符。然后空了16个字节，接着是`DLL`信息。

让我把新表项的每一个字段都给大家讲解一下。

`1a03b9`开始，第一个`DWORD`是`1a0410`。地址`1a0410`是指向的`1a03f0`，`1a03f0`这是一个`DLL`接口的描述。`0100`（`0001`）是`\_GetAddress`函数在`DLL`中的序列号，`IPsearcher.dll`中只有一个函数接口，序号是1，然后是一个以0结尾的字符串`\_GetAddress`这是函数的名称。

表项的第二个第三个`DWORD`是`timestamp`和向后的指针链，不管。只有一个函数，所以指针链是0，表示结束。

第四个`DWORD`是`1a03e0`，它指向`DLL`文件的文件名，图例中是`ipsearcher.dll`，当然也需要以0结尾。

第五个`DWORD`是`1a0418`，这个地址是在图例中有数值，其实可以不管。因为这个地址是系统在装入`DLL`之后填写的该函数接口的具体地址。我们的补丁程序就是调用这个地址中的地址来调用`\_GetAddress`这个函数。具体怎么访问呢？程序装入地址是`40000h`，所以该地址在装入之后就是`40000h+1a0418h=5a0418h`。我们的补丁程序使用`call dword ptr [5a0418]`就可以调用`\_GetAddress`了。

呵呵，修改`Import Table`的工作已经结束。我使劲在讲，不知道大家明白了没有，可不要说我没照顾大家的感受！

对了，还有一个很重要的事情。由于在系统装入`DLL`的时候需要在`1a0418`中填写函数入口地址，所以，找个地址所在的段必须可写。关于怎么修改段表，使某一个段可写，请参阅我以前的文章。

最后当然要用我们的`tdump`查看一下我们修改`PE`文件的结果：

```
Imports from WININET.dll

(hint = 0075) InternetQueryOptionA

(hint = 0052) InternetCanonicalizeUrlA

(hint = 0056) InternetCloseHandle

(hint = 007E) InternetSetFilePointer

(hint = 0077) InternetReadFile

(hint = 0074) InternetQueryDataAvailable

(hint = 0088) InternetWriteFile

(hint = 0069) InternetGetLastResponseInfoA

(hint = 005C) InternetCrackUrlA

(hint = 0071) InternetOpenUrlA

(hint = 006F) InternetOpenA

(hint = 0083) InternetSetStatusCallback

Imports from ipsearcher.dll

(hint = 0001) \_GetAddress
```

看红字，呵呵，我们已经成功了。这样，被打补丁的程序在装入的时候会自动装入`ipsearcher.dll`，并且把`\_GetAddress`函数的入口地址填写到`1a0418`处。

以后该怎么办不是我们今天要探讨的内容，总之我们在被打补丁的程序中已经能够调用新的动态链接库中的函数接口了，这无疑为在补丁中添加实现复杂的功能提供了可能。

呵呵，很有用吧。我表达能力不太好，希望大家看懂了。不懂的自己先努力查资料自己专研，实在不懂的再问我吧。

辛苦大家看这么多，下次再见吧！

### 本篇总结

1. 本系列文章讨论的主要话题和范围。
2. `PE`文件是`Win32`平台的可执行文件。
3. 关于为程序添加动态连接库的问题。
4. 其他一些相关知识的介绍。
5. 需要参考我以前的文章

邹丹 于 2001年8月18
