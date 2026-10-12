
# 旧文新贴，关于一个古典密码破解的实例

> 原文：[`www.pediy.com/kssd/pediy10/91232.html`](https://www.pediy.com/kssd/pediy10/91232.html)

几年前写的东西了，原来贴在西西河里地址是：`http://www.ccthere.com/article/1678844`

为祝贺密码学版块开张转一下![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")

要破解的这个密文是来自`mala`的网站的，这位兄弟的网站很怪，他既不要你钱，也不要你命（呃，说漏嘴了，要命的网站谁敢上啊？！）。不过要看内容，必须要证明一下你的实力

怎么证明呢？当然就是找到并破解一段密文罗。密文是放在`html`文件的注释里的，这个找起来很方便，不过对于非计算机的同学来说专业了一点，先略过不表。

找到之后密文就是这个样子的：

`MAL TIRRUEZF CR MAL RKZYIOL EX MAL OIY UAE RICF "MAL ACWALRM DYEUPLFWL CR ME DYEU MAIM UL IZL RKZZEKYFLF GH OHRMLZH"`

几年前实际上我就把这个密文破掉了，但是这是人家网站上的东西，一直不敢公布答案，不过现在这个网站貌似已经关掉了，所以就有恃无恐的发了上来:)

这位爷说了，网站都关掉了谁知道你是不是在吹牛呢？你可以从这个密文中抽出几个字来网上`google`一下，到现在还有些老外当时在四处问这个密文的解是什么的帖子呢，呵呵。

好了闲话少说，开始破解

要破解的密文是：

`MAL TIRRUEZF CR MAL RKZYIOL EX MAL OIY UAE RICF "MAL ACWALRM DYEUPLFWL CR ME DYEU MAIM UL IZL RKZZEKYFLF GH OHRMLZH"`

我们注意到这段密文中各个字母的词频出现频率是不一样的。所以应该是个替换密码，从哪里开始入手破解呢？

我们注意到这段密文中`MAL`出现频率很高，一共出现了 4 次！加上`mala`的网站是个 E 文网站……所以我猜它就是 E 文中出现频率很高的单词`the`

好了，我们把`the`这个结果填到解密的结果中去，注意大写字母表示密文，小写字母表示明文，这似乎已经是密码学界的规矩了，我也不好打破这个规矩，呵呵

`MAL TIRRUEZF CR MAL RKZYIOL EX MAL OIY UAE RICF "MAL ACWALRM DYEUPLFWL CR ME DYEU MAIM UL IZL RKZZEKYFLF GH OHRMLZH"`

`the ___________ __ the __________ __ the ___ ___ ___ "the ________ ___________ __ __ _________ ___ _________________ __ _______________"`

另外我们在下面这张替换表中也应该已经可以填入一些东西了

`ABCDEFGHIJKLMNOPQRSTUVWXYZ`

`h   et`

接着我们就可以把`ALM`这三个字母对应的`het`三个字母解到下面的明文结果中去，以便进一步去分析。

`MAL TIRRUEZF CR MAL RKZYIOL EX MAL OIY UAE RICF "MAL ACWALRM DYEUPLFWL CR ME DYEU MAIM UL IZL RKZZEKYFLF GH OHRMLZH"`

`the ___________ __ the _____e __ the ___ _h_ ___ "the h__he_t _______e__e __ t_ _____ th_t _e ___e ____________e_ __ __________te__"`

`ABCDEFGHIJKLMNOPQRSTUVWXYZ`

`h   et`

显然，`MAIM`对应的`th_t`中间那个空格填入`a`很合适，它就是`that`

`MAL TIRRUEZF CR MAL RKZYIOL EX MAL OIY UAE RICF "MAL ACWALRM DYEUPLFWL CR ME DYEU MAIM UL IZL RKZZEKYFLF GH OHRMLZH"`

`the _a_______ __ the ____a_e __ the _a_ _h_ _a__ "the h__he_t _______e__e __ t_ _____ that _e a_e ____________e_ __ __________te__"`

`ABCDEFGHIJKLMNOPQRSTUVWXYZ`

`h     a  et`

`IZL`对应的`a_e`是什么呢？我们来想想`a`开头的三个字母的单词不会很多，对了，它很可能就是`are`。试试吧，有时破解密码就是连猜带蒙的艺术:)

当然也有可能是`age`，呵呵，不过按这个假设推理到了一定的地方就会推不下去，不得不再回过头来，回到`are`这个假设上来，不信你可以试试，呵呵

`MAL TIRRUEZF CR MAL RKZYIOL EX MAL OIY UAE RICF "MAL ACWALRM DYEUPLFWL CR ME DYEU MAIM UL IZL RKZZEKYFLF GH OHRMLZH"`

`the _a____r_ __ the _r_a_e __ the _a_ _h_ _a__ "the h__he_t _______e__e __ t_ _____ that _e are __rr_____e_ __ __________ter_"`

`ABCDEFGHIJKLMNOPQRSTUVWXYZ`

`h     a  et             r`

柿子要拣软的捏，下一个是谁呢？挑未知字母少的，嗯，就是`ME`了，它对应的是`t_`，应该填一个什么字母进去呢？`tw`？又不是台湾的缩写。对了是`to`错不了，就是它了。

`MAL TIRRUEZF CR MAL RKZYIOL EX MAL OIY UAE RICF "MAL ACWALRM DYEUPLFWL CR ME DYEU MAIM UL IZL RKZZEKYFLF GH OHRMLZH"`

`the _a_____r_ __ the _r_a_e o_ the _a_ _ho _a__ "the h__he_t __o_e__e __ to __o_ that _e are __rro____e_ __ __________ter_"`

`ABCDEFGHIJKLMNOPQRSTUVWXYZ`

`h   o   a  et             r`

下一个猜谁呢？`EX`对应的`o_`，怎么样我觉得不好因为可填入的字母太多了`of`/`ok`/`or`都可以，变化太大，还是先把他放一放吧

我看中了`UAE`，它对应的`_ho`很可能是`who`，当然也有可能是`tho`，不过，`who`可比它常用多了。另外`t`在已知字母表里已经有了

`MAL TIRRUEZF CR MAL RKZYIOL EX MAL OIY UAE RICF "MAL ACWALRM DYEUPLFWL CR ME DYEU MAIM UL IZL RKZZEKYFLF GH OHRMLZH"`

`the _a_wor_ __ the _r_a_e o_ the _a_ who _a__ "the h__he_t __ow_e__e __ to __ow that we are __rro____e_ __ __________ter_"`

`ABCDEFGHIJKLMNOPQRSTUVWXYZ`

`h   o   a  et       w    r`

这时你就要注意了，如果你刚才猜的是`tho`，那么`UL`这个位置上填充的就是`te`，这是什么玩意啊？

所以我们在这里也就间接的证明了原来的假设的正确性。

接着猜什么呢？

`EX`对应的`o_`我猜它是`of`吧。为什么呢？`or`中的`r`在已知的字母表中已经有了，

从语法上讲……`ok the`……好像不太通，相比之下……`of the`……就还可以接受了。

`MAL TIRRUEZF CR MAL RKZYIOL EX MAL OIY UAE RICF "MAL ACWALRM DYEUPLFWL CR ME DYEU MAIM UL IZL RKZZEKYFLF GH OHRMLZH"`

`the _a_wor_ __ the _r_a_e of the _a_ who _a__ "the h__he_t __ow_e__e __ to __ow that we are __rro____e_ __ __________ter_"`

`ABCDEFGHIJKLMNOPQRSTUVWXYZ`

`h   o   a  et       w  f r`

好像这一次猜测的结果对后面帮助不是很大

再怎么办呢？

我注意到`CR`出现的频率也很高，根据词频表，加上`aethowfr`这几个已知的字母不会出现在这个 2 个字母组成的单词上的这个限制，我猜`CR`对应的应该是`is`

`MAL TIRRUEZF CR MAL RKZYIOL EX MAL OIY UAE RICF "MAL ACWALRM DYEUPLFWL CR ME DYEU MAIM UL IZL RKZZEKYFLF GH OHRMLZH"`

`the _asswor_ is the s_r_a_e of the _a_ who sai_ "the hi_hes_t __ow_e__e is to __ow that we are s_rr____e_ __ ______ster_"`

`ABCDEFGHIJKLMNOPQRSTUVWXYZ`

`h i o   a  et    s  w  f r`

哈哈，柳暗花明啊！很显然，`RICF`对应的`sai_`就应该是`said`

`MAL TIRRUEZF CR MAL RKZYIOL EX MAL OIY UAE RICF "MAL ACWALRM DYEUPLFWL CR ME DYEU MAIM UL IZL RKZZEKYFLF GH OHRMLZH"`

`the _assword is the s_r_a_e of the _a_ who said "the hi_hes_t __ow_ede_ is to __ow that we are s_rr____ded __ ______ster_"`

`ABCDEFGHIJKLMNOPQRSTUVWXYZ`

`h i od  a  et    s  w  f r`

接着`TIRRUEZF`对应的`_assword`肯定就是`password`罗

`MAL TIRRUEZF CR MAL RKZYIOL EX MAL OIY UAE RICF "MAL ACWALRM DYEUPLFWL CR ME DYEU MAIM UL IZL RKZZEKYFLF GH OHRMLZH"`

`the password is the s_r_a_e of the _a_ who said "the hi_hes_t __ow_ede_ is to __ow that we are s_rr____ded __ ______ster_"`

`ABCDEFGHIJKLMNOPQRSTUVWXYZ`

`h i od  a  et    s pw  f r`

我想`ACWALRM`对应的`hi_hes_t`中毫无悬念的应该填入字母`g` `highest`

`MAL TIRRUEZF CR MAL RKZYIOL EX MAL OIY UAE RICF "MAL ACWALRM DYEUPLFWL CR ME DYEU MAIM UL IZL RKZZEKYFLF GH OHRMLZH"`

`the password is the s_r_a_e of the _a_ who said "the highest __ow_edge is to __ow that we are s_rr____ded __ ______ster_"`

`ABCDEFGHIJKLMNOPQRSTUVWXYZ`

`h i od  a  et    s pw gf r`

加油啊！只剩一点点了！

显然`DYEUPLFWL`中的`DYEU`也是一个单词而且我们又已知了那么多的字母不能填进去，激发一下想象力，多数它是`knowledge`！猜了再说，大不了再退回来就是了

`MAL TIRRUEZF CR MAL RKZYIOL EX MAL OIY UAE RICF "MAL ACWALRM DYEUPLFWL CR ME DYEU MAIM UL IZL RKZZEKYFLF GH OHRMLZH"`

`the password is the s_rname of the _an who said "the highest knowledge is to know that we are s_rro_nded __ mystery_"`

`ABCDEFGHIJKLMNOPQRSTUVWXYZ`

`h ikod  a  et  l s pw gfnr`

破纪录了，一下子猜出三个字母来了:)

`OIY`对应的`an`一定应该是`man`了，后面从句单词`who`给了我很大的启发

`MAL TIRRUEZF CR MAL RKZYIOL EX MAL OIY UAE RICF "MAL ACWALRM DYEUPLFWL CR ME DYEU MAIM UL IZL RKZZEKYFLF GH OHRMLZH"`

`the password is the s_rname of the man who said "the highest knowledge is to know that we are s_rro_nded __ m__ster_"`

`ABCDEFGHIJKLMNOPQRSTUVWXYZ`

`h ikod  a  et ml s pw gfnr`

接下来呢？

`RKZYIOL`对应的`s_rname`中只有一个字母没有填了，可选的范围很小了，我猜它是`surname`！

`MAL TIRRUEZF CR MAL RKZYIOL EX MAL OIY UAE RICF "MAL ACWALRM DYEUPLFWL CR ME DYEU MAIM UL IZL RKZZEKYFLF GH OHRMLZH"`

`the password is the surname of the man who said "the highest knowledge is to know that we are surrounded __ mystery_"`

`ABCDEFGHIJKLMNOPQRSTUVWXYZ`

`h ikod  a uet ml s pw gfnr`

你看只剩下 2 个单词了。

可选的字母也只剩下`bcjqvxyz`这八个字母了

`surrounded XX`

`XX`位置上还只能是`bcjqvxyz`中的 2 个字母，不用说，那就是`by`了。

或者`m__ster_`能满足在这 2 个空格中之填入一个字母，让其生效的只有一个`y`

知道了`GH`对应的是`_y`的话，不用说，它就是`by`了。

所以：

`MAL TIRRUEZF CR MAL RKZYIOL EX MAL OIY UAE RICF "MAL ACWALRM DYEUPLFWL CR ME DYEU MAIM UL IZL RKZZEKYFLF GH OHRMLZH"`

`the password is the surname of the man who said "the highest knowledge is to know that we are surrounded by mystery"`

`ABCDEFGHIJKLMNOPQRSTUVWXYZ`

`h ikodbya uet ml s pw gfnr`

所以答案正是`the password is the surname of the man who said "the highest knowledge is to know that we are surrounded by mystery"`

`google`一下，你就知道他是谁了。
