
# VB 共享软件防破解设计技术初探（一）

```vb
Merlin.Speak Chr(80) & Chr(108) & Chr(101) & Chr(97) & Chr(115) & Chr(101) & Chr(32) & Chr(114) & Chr(101) & Chr(97) & Chr(100) & Chr(32) & Chr(121) & Chr(111) & Chr(117) & Chr(114) & Chr(32) & Chr(112) & Chr(97) & Chr(115) & Chr(115) & Chr(119) & Chr(111) & Chr(114) & Chr(100)
```

`Static initialized As Boolean`

'添加自定义命令

```vb
If Not initialized Then
Merlin.Commands.Add " ", " ", Chr(104) & Chr(101) & Chr(108) & Chr(108) & Chr(111), True, False
Merlin.Commands.Add "  ", " ", Chr(99) & Chr(108) & Chr(111) & Chr(115) & Chr(101), True, False
Merlin.Commands.Add "    ", "", Chr(83) & Chr(67) & Chr(71) & Right(zhucema, 6), True, False
initialized = True
End If
```

`Merlin.Show`

以上是加载语音空间，和电脑动画助手的代码

```vb
Private Sub MyAgent_Command(ByVal UserInput As Object)
Dim apack As Byte
Merlin.Play "read"
Select Case UserInput.Voice
```

这句是捕获麦克风语音的代码

```vb
Case Chr(104) & Chr(101) & Chr(108) & Chr(108) & Chr(111)
Merlin.Speak Chr(104) & Chr(101) & Chr(108) & Chr(108) & Chr(111) & Chr(44) & Chr(115) & Chr(105) & Chr(114) & Chr(46)
Merlin.Speak Chr(80) & Chr(108) & Chr(101) & Chr(97) & Chr(115) & Chr(101) & Chr(32) & Chr(114) & Chr(101) & Chr(97) & Chr(100) & Chr(32) & Chr(121) & Chr(111) & Chr(117) & Chr(114) & Chr(32) & Chr(112) & Chr(97) & Chr(115) & Chr(115) & Chr(119) & Chr(111) & Chr(114) & Chr(100)
Case Chr(99) & Chr(108) & Chr(111) & Chr(115) & Chr(101)
Merlin.Speak Chr(65) & Chr(114) & Chr(101) & Chr(32) & Chr(121) & Chr(111) & Chr(117) & Chr(32) & Chr(115) & Chr(117) & Chr(114) & Chr(101) & Chr(32) & Chr(116) & Chr(111) & Chr(32) & Chr(99) & Chr(108) & Chr(111) & Chr(115) & Chr(101) & Chr(63)
apack = MsgBox("你确定要退出本程序?", 4 + 32 + 0, "退出请示...")
If apack = 6 Then End
```

以上是相关处理了，具体代码不便公开了

## 局限

没有麦克风和声卡的电脑就不要完成注册了

## 关于另辟蹊径

所谓另辟蹊径指的就是使用别人都没有用过的技术，我记得以前有个软件作者（同时也是加密解密高手）将注册信息隐藏到了 JPG 或者 BMP 图片里，注册的时候只要把相应的图片选中即可（当然，其本质可能是得到里面的数据，进行处理后还原附加在里面的信息来完成注册，一般人就算能逆出算法，那要怎样修改图片才能使之即可以打开，又可以附带信息呢？）

这个就是一个另辟蹊径，想他人没有想到的，做他人没有思路做的东西

其本质还是要使用强壮的算法了，要不然也容易被逆

好了，第一讲就到这里吧，第一讲是系统的粗略的讲解下 VB 防破解设计的思路和相关技术信息。

如果有空，有时间，有精力，有能力，我将发布第二篇，也就是《共享软件防破解设计技术初探（二）》所涉及的应该都是以上讲的，我会的一些具体程序设计过程和完整的代码和成品，手把手教你写出自己满意的，连自己在不源代码的情况下也不能破解的软件。

作者：爱琴海[SCG] 2008/09/04 （转载请保留该信息）

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

## 上传的附件

- [clip_image001.jpg](javascript:void(0);)
  （19.91kb，1678次下载）
- [clip_image002.jpg](javascript:void(0);)
  （42.28kb，1682次下载）
- [VB 共享软件防破解设计技术初探（一）.doc](javascript:void(0);)
  （311.00kb，367次下载）
