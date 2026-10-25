
# 英语精灵 1.0.5 版

> 原文：`www.pediy.com/kssd/pediy05/pediy50043.htm`

各位大虾好，我是`lipan`，这是本人的第一篇破文，是模仿大家的文章写的，若有不正确之处，希望能得到前辈们的指教，谢谢。关于破解以`p-code`方式编译的`VB`程序一例。本文仅在研究破解技术，请大家支持软件作者制作出更优秀的软件。

目标软件：`EnglishElf`英语精灵 1.0.5 版 软件主页：`http://www.kytop.net/englishelf/`

工具软件：`VBDebug13e` 可以在看雪学院下载

## 过程

此软件计算机器码，然后对比输入的注册码。

好了开始！

运行`VBDebug`，载入`Englishelf`，执行，快速按下两个提示信息后，中断在程序入口。

选择`FormManager`，`FrmRegister`，`cmdRegister`，下断`0041AE6Ch`（点击`bPX`）。（为什么？呵呵，看看名字就能猜到呗）

`F5` 继续运行，出现未注册提示，不管按确定通过，进入运行界面，点击右上角弹出菜单，选择软件注册，出现注册窗口，随便填写`"9898989898989898"`，按下注册钮。

被拦住了！Bingo!

拦在如下代码：

```
0041AE6C: 04 FLdRfVar 0065F378h  《-拦在这里

0041AE6F: 21 FLdPrThis 0047FDC0h

0041AE70: 0F VCallAd frmRegister.txtSoftCode

0041AE73: 19 FStAdFunc

0041AE76: 08 FLdPr

0041AE79: 0D VCallHresult get\_\_ipropTEXTEDIT

0041AE7E: 3E FLdZeroAd

0041AE81: FD Lead2/PopTmpLdAdStr

0041AE85: 0A ImpAdCallFPR4 ModPublic!0041B98Ch

0041AE8A: 2F FFree1Str

0041AE8D: 1A FFree1Ad

0041AE90: 0B ImpAdCallI2 ModPublic!0041B328h  《-如此就从这里跟进去看看，f8 进入

0041AE95: 1C BranchF 0041AEEA ?  《-跳转判断句

0041AE98: 1B LitStr: '本软件正确注册，请访问主页以获取本软件的最新情况'  《-上面的语句看不太懂，

但是这一句显然是好的，如果不是上面的判断跳走了的话，呜呜。

0041AE9B: 0A ImpAdCallFPR4 ModPublic!00419EF4h

0041AEA0: F4 LitI2\_Byte: -> FFh 255

0041AEA2: 05 ImpAdLdRf

0041AEA5: 24 NewIfNullPr frmAbout 004052AC

0041AEA8: 0F VCallAd frmAbout.mnuNextScene
```

进入后，又是一大堆难看的代码，我偷懒一直按`f12,f8`组合，直到下面：

```
041B3A0: 31 FStStr 0047FD44h to 0065F3D4h

0041B3A3: 6C ILdRf 0047FFC4h  《-请看看 0047FFC4h

0041B3A6: 6C ILdRf 0043E3CCh  《-请看看 0043E3CCh

0041B3A9: 30 EqStr  《-经典的比较语句

0041B3AB: 1C BranchF 0041B3B6 ?  《-过了这一句就跳回上面的 0041AE95 的判断，game over

0041B3AE: F4 LitI2\_Byte: -> FFh 255

0041B3B0: 70 FStI2 0065F2E2

0041B3B3: 1E Branch 0041B3BB ?

0041B3B6: F4 LitI2\_Byte: -> 0h 0

0041B3B8: 70 FStI2 0065F2E2

0041B3BB: 15 ExitProcI2
```

看来`0041B3A9`的比较是关键，我就改！可惜，`p-code`太难懂了，不过难不到，比较一下`0041B3A3`和`0041B3A6`的十六进制码，发现一个是`6c70ff`，一个是`6c6cff`，那我就把前面中间的`70`改成了`6c`，比较就总是拿相同的字段比较，当然相同。`f5` go!

自然是本软件正确注册啦，呵呵。

其实，现在你可以在`VBDebug`的窗口中看到自己输入的注册码，真正的注册码，算出的假代码，真正的代码四个字符串，比较是在后两个代码之间进行的。

好了，看看偏移量，用`16`进制编辑软件改了，或者使用真正的注册码注册吧。祝你愉快！

希望能当作我加入组织的第一篇破文。谢谢！ `4dd466d0e1e0cc123afbe504da2462ca`
