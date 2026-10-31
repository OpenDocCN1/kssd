
# [原创]让我们熟练掌握输入表,高手早已掌握

无论是在脱壳还是在PEDIY中，“输入表”这一概念总像阴魂一样缠绕在电脑屏幕前，令我等菜鸟彷徨叹息。遂于近日认真学习了一下PE格式，终于将此恶魔搞定。

将实例`PE.EXE`载入`LordPE`中，依次点击“数据目录”，“输入表”旁的小点点，弹出以下窗口。我们勾选下面的"总是查看`FirstThunk`".

![](https://bbs.kanxue.com/upload/attach/200811/117580_lnk3kr8uvsvtm8v.jpg)

![](https://bbs.kanxue.com/upload/attach/200811/117580_9ojcowra1ycw7bw.jpg)

只见有两个dll:`user32.dll`和`kernel32.dll`以及各dll引出的诸函数.

可见每个dll有五个属性.

我们暂且只记住第一个dll的`FirstThunk`值如图:`00002010`,窗口下半部表明该值指向了名为`DrawTextA`的API函数.

我们用`OD`载入实例.`bpx DrawTextA`, `F9`运行被断下，如图：

![](https://bbs.kanxue.com/upload/attach/200811/117580_pypxhp7erdbp059.jpg)

到达程序对`DrawTextA`的调用处，`F7`跟进

![](https://bbs.kanxue.com/upload/attach/200811/117580_hdjfwlajtn8a9wh.jpg)

双击一下，原来是直接跳转到`[402010]`的地方去了.

我们在数据窗口中`d 402010`看看`402010`到底是什么。

![](https://bbs.kanxue.com/upload/attach/200811/117580_1dojgtdgcrghhtr.jpg)

为方便查看，我们右键选择"长型","地址".

![](https://bbs.kanxue.com/upload/attach/200811/117580_hex7facogapp6dl.jpg)

原来,`402010`地址里面的值是`77D3C6CA`那么`jmp dword ptr [402010]`就是跳到`77D3C6CA`了.这个地址就是`DrawTextA`函数的入口地址。

现在我们大致知道输入表中这个`First Thunk`值的作用了，原来程序中对这些输入函数的调用都是`jmp 402010`实现的。

为进一步理解，我们在16进制编辑器(如`winhex`)中查看输入表。

![](https://bbs.kanxue.com/upload/attach/200811/117580_npbe74bbj9wzu44.jpg)

数据目录表第二项是输入表的`RVA`：`2050`.将其转换成文件偏移`650`.该处就是输入表的开始。

蓝色部分是2个`IID`数组。每个数组有五个属性，对应着一个`DLL`.这就是`lordpe`中输入表五个属性的由来，可与`lordpe`对照:

`OringinalFirstThunk` `TimeDateStamp` `ForwarderChain` `Name` `FirstThunk`

`9C200000` `00000000` `00000000` `CA210000` `10200000`

`8C200000` `00000000` `00000000` `0A220000` `00200000`

且看第一个`IID`。

他的`OriginalFirstThunk`值为`209C`这是`RVA`值转化为文件偏移`69C`.便是图中淡黄色部分.

`FirstThunk`值为`2010`转化为文件偏移`610`.便是图中深黄色部分.

明眼人已经看出来了，这两部分的数据是一模一样的。前者称为`INT`后者是`IAT`。

这部分数据也是指针,`22210000`转换后是`00002122`再转化成文件偏移`722`.看`772`处是什么？

原来是`DrawTextA`这个字符串.再看`2E210000`转化文件偏移`72e`.看`72e`处是什么？原来是`EndPaint`字符串....后面再不一一看了:)------------请注意到并不是刚好指向函数的字符串前面还多了两个字节称为`hint`值可为0,此处不详讨.

`772`,`77e`....是函数名的地址，这群地址组成的结构被称为`IMAGE_IMPORT_BY_NAME`.

为什么`INT`和`IAT`两个都指向这一结构呢？

系统在程序初始化时根据`OrignalFirstThunk`的值找到函数名,调用`GetProcAddress`函数(或类似功能的系统代码)且根据函数名取得函数的入口地址,

然后用函数入口地址取代`FirstThunk`指向的地址串的值`IAT`.

回顾本文开头时我们`OD`的一段跟踪，这段话便不难理解了.当我们在`OD`中查看`00402010`(`RVA 2010`加上基址`400000`)时，里面已经是`DrawTextA`函数的入口地址。

因为系统已经用真正的函数入口地址取代`IAT`了。

为加强理解，我们针对以上理论做以下实验.

1. 有时候程序不存在`OrignalFirstThunk`,我们将此程序的该值去掉以看效果.

将`650`处的`9C20`以`0`填充，保存文件.双击。。程序运行正常.可见,在没有`OrignalFisrtThunk`的情况下，系统直接以`IAT`中的的函数名得到函数入口地址并填充`IAT`。

2. `OrignalFirstThunk`值不变将`INT`以`0`填充以看效果.

按`CTRL+Z`数次恢复对文件的修改,把上图淡黄色部分用`0`填充,保存文件。双击.弹出程序运行错误窗口.

可见系统根据`OrignalFirstThunk`值只找到空的`INT`，程序便运行错误了.

3. 依照理解`IAT`反正会被根据`OrignalFirstThunk`得到函数入口地址所取代，似乎`IAT`原始值没有存在的必要.

按`CTRL+Z`数次恢复对文件的修改.这次将`610`处即上图深黄色部分用`0`填充,保存文件.双击.弹出程序运行错误窗口.

按`CTRL+Z`数次恢复对文件的修改,程序又能运行了.由此可见即使`OrignalFirst`值正常,`INT`也正常,`IAT`的值仍然必不可少,由此可见`IAT`的重要性!

相信至此读者已对输入表有了一定程度的认识,无论是`ImportREC`重建输入表还是手工重建输入表亦或是增加输入函数都已不再是难事.![](https://bbs.kanxue.com/view/img/face/41.gif)

[PE.zip](https://bbs.kanxue.com/attach-download-20802.htm)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [2008-11-24_163521.jpg](javascript:void(0);)
  （60.45kb，4935次下载）
- [2008-11-24_163530.jpg](javascript:void(0);)
  （46.36kb，4905次下载）
- [2008-11-24_163539.jpg](javascript:void(0);)
  （116.38kb，4921次下载）
- [2008-11-24_163555.jpg](javascript:void(0);)
  （134.95kb，4919次下载）
- [2008-11-24_163604.jpg](javascript:void(0);)
  （79.67kb，4884次下载）
- [2008-11-24_163611.jpg](javascript:void(0);)
  （52.41kb，4861次下载）
- [2008-11-24_164344.jpg](javascript:void(0);)
  （169.44kb，4895次下载）
- [PE.zip](javascript:void(0);)
  （0.98kb，553次下载）
