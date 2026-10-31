
# [原创]CTF第三题

[TOC]

## CTF3

### 基本调试 以及注意事项

修改随机基址，改为固定基址。

先过反调试。至少能输入字符串。为了不改变程序执行流程，只要把`jnz`改成`jz`就好了。修改反调调试验证函数的返回值。

反调试一定要跟到函数里面看一看，谁知道在函数外边判断前作者在里面做了什么？

`IDA` + `OD`发现输入被重新编码至少两次（后来发现有4次）。

![图片描述](https://bbs.kanxue.com/upload/attach/201710/766608_rhu4ibuimzknxtg.png)

进入`encode`函数内部，观察编码逻辑。

![图片描述](https://bbs.kanxue.com/upload/attach/201710/766608_oa0p7qs3yw8dsjn.png)

随便使用一段base64字符：

```text
QmFzZTY057yW56CB6KaB5rGC5oqKM+S4qjjkvY3lrZfoioLvvIgzKjg9MjTvvInovazljJbkuLo05LiqNuS9jeeahOWtl+iKgu+8iDQqNj0yNO+8ie+8jOS5i+WQjuWcqDbkvY3nmoTliY3pnaLooaXkuKTkuKow77yM5b2i5oiQ
```

长一点，否则两次编码以后长度会急剧缩短，第二次编码以后竟然给编码没了。

之后的反调试我也看了看，没发现做什么手脚，所以就直接`nop`了，而不是修改跳转。

### 第4轮编码以后

前面几轮编码都是一路过反调试，没意思就不多谈了。

![图片描述](https://bbs.kanxue.com/upload/attach/201710/766608_1glp8e90vbomjk2.png)

![图片描述](https://bbs.kanxue.com/upload/attach/201710/766608_6svdnpnd4y6d9o2.png)

![图片描述](https://bbs.kanxue.com/upload/attach/201710/766608_x3dzi9rlk6pvmmh.png)

```text
eax=0018E058 "de5909376cf44e85fbb3886506a393022cbc8b3dcbdbc610911b7f12712a9810"
```

竟然还计算长度，算毛啊，64个字节。

![图片描述](https://bbs.kanxue.com/upload/attach/201710/766608_d2xshamlal3ntjw.png)

![图片描述](https://bbs.kanxue.com/upload/attach/201710/766608_naskb8h0feqyou8.png)

试一试将`de5909376cf44e85fbb3886506a393022cbc8b3dcbdbc610911b7f12712a9810`作为输入，此时验证值变为：

```text
eax=0018E058 "e1869e48114c9f10c8c997263b8e387dbf9c555c67323d4fa1ba5506882e722a"
```

我突然有一个想法，如果两个串存放的位置相距不远，那么我输入`xxxxx`会得到`bbbbb`。我可以输入`xxxxxhhhhhhbbbbb`，将验证的值覆盖成`bbbbbb`，由于发生了截断，他只会发现`xxxxxx`，对`xxxxx`做运算得`bbbbb`。

结果两个值相隔好远啊。这个办法不通。

### 数学分析

如果输入`A1`，那么自然有`B1`与之对应；再输入`B1`，有`C1`与之对应。

而集合是有限的。如果存在有且仅有唯一解，这个解假设为输入`D1`运算完毕以后依然是`D1=F(D1)`。那么集合剩下的数字为了不产生`x=f(x)`，只能形成一些或大或小的循环，或者收敛于`D1`。如果存在唯一`D1`，接下来就看`f(x)`如何做到这一点。有点感觉是两个图像在区间上的交点问题，`A(x) = x`与`B(x)`是一个定义域值域都为`{Hex64}`的有限集合。要找的是两个图像的交点。知道`B(x)`满足`B(B(B(….))) – x = 0`有且仅有一个解。所以求导，分三种情况讨论....要么逆向算法，要么暴力呵呵。

那么很有可能就是之前计算两次长度哪里有问题。

![图片描述](https://bbs.kanxue.com/upload/attach/201710/766608_bkaofzrd7b8blnk.png)

果然当我们输入`AAAAAAAxxxxxxxxxxxxxxxxxxx`的时候，根据其`AAAAAAA`会计算出一个值`ccccccccccccccccccccc`。那么下次我们只要输入`AAAAAccccccccccccc`的值就好了。

按照前面的分析，想要攻破要么逆向算法，要么暴力，这显然是行不通的。我怀疑这里是作者故意留的。

如果此时就能得到正确答案，题目必然会多解。**作者为了避免多解不得不在后面做一些验证检查然**。此时距离`MessageBox`函数只有两个函数了。

也就是说为了保证输入唯一，作者必定在接下来的两个函数中对输入做一次验证。

接下来输入`aaaaaaa`，得到`432a34a4946708602bdf52a8b556a3c18ff9ab60c346da83579f149c4f366513`。

只要输入`aaaaaaa432a34a4946708602bdf52a8b556a3c18ff9ab60c346da83579f149c4f366513`就可以通过`memcpy`的验证。

![图片描述](https://bbs.kanxue.com/upload/attach/201710/766608_mo5cp7cirqit991.png)

果然，这里又来藏了点东西，弄了一个除零异常，注册了一个函数。好气啊。除零，`X32Debug`放不走除零异常，等等好像`X32Debug`能放走除零异常，我跟过来了。但是好像什么都没做，只是修改了堆栈。

![图片描述](https://bbs.kanxue.com/upload/attach/201710/766608_lv5if4gjcpr35bv.png)

这里果然有问题。好烦啊，又是反调试。

![图片描述](https://bbs.kanxue.com/upload/attach/201710/766608_vhyyhcjk2x2outn.png)

![图片描述](https://bbs.kanxue.com/upload/attach/201710/766608_yxceu1s1j4shvst.png)

作者代码可能写得有问题。最后连两个函数在我的调试过程中完全没有起到作用。

一路`F7`竟然过了。这道题肯定多解了。

后来我试了几次，程序在调试的时候单步跟是能过的。运行的时候有时候能有时候不能。单步能过，不但不就过不了。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [crackMe.rar](javascript:void(0);)

  （3.41MB，6次下载）
