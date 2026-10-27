
# [原创]IAT的填充过程与执行流程

我也是新手，有遗漏和疏忽之处希望大家及时指正。

IAT是什么？为什么它在脱壳修复中扮演着如此重要的地位？它在内存中是怎么样排列的，又是怎样获取的呢？

调试器：OllyDbg

环境：WinXP SP3 虚拟机

## （一）

什么是IAT呢。同一个函数，在A电脑里的首地址和在B电脑里的首地址是不一定相同的。系统版本不同，补丁版本不同，都会导致同一个函数的地址有所差异。如果找不到正确的系统API地址程序就没法正常调用函数，自然就没法运行。为了提升系统的兼容性，每次执行程序时软件都会重新获取一次基于当前的系统的API的地址，并把这些地址集中起来管理，组成一个表。我们把这个表叫输入函数地址表，简称输入表。即IAT(import address table)。

有些壳没有彻底破坏掉IAT结构。比如ASpck，我们以一个加Aspack的课件为例，我们首先使用ESP定律来到OEP，此时壳已经将IAT还原完毕了。（关于ESP定律的详细解释可以参考坛主大大的文章https://bbs.pediy.com/thread-20366.htm）

然后，来看一个调用函数的实例。

![](https://bbs.kanxue.com/upload/attach/201902/3_HPTKJM68W4TK83W.jpg)

`call <imp.&KERNAL32.GetmoduleHandle>`，点击这一行按下回车，来到跳转表

![](https://bbs.kanxue.com/upload/attach/201902/3_A7REC8KJ799U8MC.jpg)

接着按下空格键，看到未经修饰的原始代码为 `jmp [403238]`

![](https://bbs.kanxue.com/upload/attach/201902/3_2747V87FA4BUTP8.jpg)

`jmp [403238]` ,数据窗口跟随`403238`，看到 `[403238] = 752d582e`，如下图。所以 `jmp[403238]` 等价于`jmp 752d582e` ,而`752d582e` 是函数`GetModuleHandle`的首地址，从而达到了调用 `GetModuleHandle` 函数的目的。当然并不是所有程序都保留有跳转表结构，本例对IAT的保存比较完整和系统，有跳转表。

![](https://bbs.kanxue.com/upload/attach/201902/3_M4CZZ6UMP8KJ282.jpg)

那么能不能直接写 `jmp 752d582e`呢？不能，因为这个地址在每台电脑都不一样。必须用`[4031ac]`间接调用`77d507ea`。

本节我们初步了解了IAT长什么样，下一节我们接下来我们来了解IAT的原始结构及获取方法

## （二）IAT的原始结构及获取方法

根据上一节的预备知识可知，数据窗口跟随 `403238`，取出里面的内容，就是是`GetModuleHandle`函数的真实地址

![](https://bbs.kanxue.com/upload/attach/201902/3_KGSCWWEK2H5VV48.jpg)

选中 `752d1245` ，右键，点击“查看可执行文件”，如下图所示，最左上角的那里

![](https://bbs.kanxue.com/upload/attach/201902/3_QZN3NXTRNK85KSV.jpg)

原本的 `403238` 变成了`1038`，`752d1245`变成了`355E`。 这是为什么呢

我们把`1038` 叫文件偏移地址，这个地址加上`402200`就是`403228`。

`355e`是相对虚拟地址（RVA），它加上基址 `400000`（image base）得到一个虚拟内存地址`40355e`（VA），跟随地址`40355e`发现里面的内容是该函数的名称字符串如下图所示。

![](https://bbs.kanxue.com/upload/attach/201902/3_UMGJKYQQEMJHJ3P.jpg)

偏移地址的内容里存储着字符串，利用字符串， 接下来，程序调用`GetProcAddress`获取到函数的真实地址，并将该地址填充到`40355e`所在的区域。注意这里是存储偏移地址的地方，而不是存储字符串的地方。IAT就由这样的步骤重复执行而形成的。

我们使用十六进制工具`winhex`验证一下。载入课件，定位`1038`，里面也是`0000355e`，再来到`40355e`，里面是字符串`GetModuleHandle`

所以程序需要用获取哪些函数，该信息早就已经存在程序里了，在程序运行前用`GeProcAddress`获取这些函数的地址填充到IAT就能运行了。

字符串信息事先保存在程序中，但一旦IAT获取完毕，字符串往往会被删除。

## （三）导入表与输入表详解

在数据窗口右击-指定-PE头，然后转到`400000`（基址），查看PE文件，找到`import table address=3000`.这表示导入表指针偏移是3000，真实地址等于3000加上基址，即`403000`。

![](https://bbs.kanxue.com/upload/attach/201902/3_USQFV7GQF4NQMT4.jpg)

我们先来区分一下导入表（IT）和输入表（IAT）。导入表是IT（import table），输入表是IAT（import address table）.IT的描述符结构是每5个Dword为一个单元，这个单元叫IID，如图所示

![](https://bbs.kanxue.com/upload/attach/201902/3_W2PMVDCH858BJX7.jpg)

一个IID有五个字段，每个字段占一个Dword，但前三个字段一般没用，有用的是第四字段和第五字段。第四字段加基址`403290`，这里存放dll名称。我们定位到`403290`看一下

![](https://bbs.kanxue.com/upload/attach/201902/3_85NW95GTFFPC7QH.jpg)

如图所示，dll的名称是`user32.dll`

第五字段里的内容加基址，作为指针指向的内容是这个dll的第一个函数地址。

![](https://bbs.kanxue.com/upload/attach/201902/3_6DB9RGJ456VDPFE.jpg)

导入表虽然包含了输入表，但IAT不一定与IT所在的内存空间相邻，它可以位于任何一个具有写入权限的地方。即使他们分隔很远也可以，换言之，在内存空间中可以不连续。IID通过第五个双字里的函数地址与IAT链接起来，这相当于一个链表。

总结，先在PE文件里找到IT的位置，接着根据IT里第一个IID的第四字段定位到dll的名称，第五个字段定位dll对应的IAT项的起始位置。接着根据IAT里的字符串偏移，提取出函数的名称，接着利用`GetProcAddress`来获取函数的地址。最后将函数的地址填充进去。一直这样重复，当获取到的不再是字符串偏移地址而是`00000000`时，即表示当前dll里所有函数的IAT填充完成了。来到IT的下一个IID里重复刚才的过程。可以看到下一个dll是`kernal32.dll`，以此类推。

![](https://bbs.kanxue.com/upload/attach/201902/3_9JZ5QSYR4BXXSWF.jpg)

以上是填充IAT的完整过程。

认识了IAT，接下来我们就要着手修复了，以后如果还有时间，我会分享IAT修复的文章给大家。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2019-2-2 10:57 被 admin 编辑，原因：图片本地化。
