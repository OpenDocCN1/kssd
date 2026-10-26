
# [原创]一个外国人编写的名为ecGraph3D的三维控件的破解

## 文本信息

【破文标题】一个外国人编写的名为ecGraph3D的三维控件的破解

【破文作者】zhuliang

【作者邮箱】huangzhuliang@sina.com

【破解工具】`PEiD`,`OD`,`IDA`,`C32ASM`

【破解平台】Windows XP

【软件大小】2.40 MB

【软件授权】国外软件/三维控件

【软件语言】英文

【原版下载】[76bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2F1j5$3!0J5k6h3y4G2L8Y4y4#2L8s2c8A6L8X3N6Q4x3X3g2U0L8$3#2Q4x3X3g2S2N6g2)9J5c8X3c8G2N6$3&6D9L8$3q4V1M7#2)9J5c8X3g2U0k6%4u0S2M7r3R3K6k6q4)9J5k6o6p5I4x3W2)9J5k6s2y4W2N6s2g2H3i4K6u0W2k6i4S2W2](https://bbs.kanxue.com/elink@2aeK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2F1j5$3!0J5k6h3y4G2L8Y4y4#2L8s2c8A6L8X3N6Q4x3X3g2U0L8$3#2Q4x3X3g2S2N6g2)9J5c8X3c8G2N6$3&6D9L8$3q4V1M7#2)9J5c8X3g2U0k6%4u0S2M7r3R3K6k6q4)9J5k6o6p5I4x3W2)9J5k6s2y4W2N6s2g2H3i4K6u0W2k6i4S2W2)

【保护方式】注册验证

【软件简介】`ecGraph3D`控件是一个能把给定的数据用三维的形式显示出来的控件，用它来显示一些统计数据可以说是相当的形象（如图1所示），能把数据直观地表现出来。

【破解声明】一点心得，愿与大家分享。 版权所有，转载注明来自看雪论坛！

## 破解内容

用`ecGraph3D`控件来显示一些统计数据可以说是相当的形象（如图1所示），`ecGraph3D`控件能把数据直观地表现出来，对于我们气象行业来说是一个相当有用的控件，因为，气象行业有大量的统计数据要显示和输出。

这个控件的使用也比较简单，它带有一个针对VB的帮助文档，告诉用户如何在VB中使用该控件。按理说，因为这个Active X控件不仅仅能在VB使用，也能在VC中使用，所以不仅要提供对VB的帮助文档也要提供对VC的使用文档的。但是它没提供针对VC的帮助文档，我想这可能是因为这个控件是用VB写的吧。

这个控件使用注册验证保护，注册版与未注册版的区别在于控件的右下角有没有个带链接的框（如图1和图2）。

![](https://bbs.kanxue.com/upload/attach/200812/28365_r1awttfgldflw29.jpg)

图1未注册版

![](https://bbs.kanxue.com/upload/attach/200812/28365_7zwuswd3tp273gb.jpg)

图2注册版

按照帮助文档上的说明，以从官方得到的注册码为参数调用`RegisterecGraph3D`方法，就可以把未注册版转变为注册版。帮助文档上对`RegisterecGraph3D`方法的说明如下。

`RegisterecGraph3D`     Procedure (Sub)

  Public Sub `RegisterecGraph3D`(S$)

  Attempts to register this implementation of ecGraph using a registration key that can be obtained from [69fN6%4N6%4i4K6u0W2k6h3&6U0L8%4u0W2j5$3!0F1M7%4g2D9N6r3W2F1k6#2)9J5k6h3y4G2L8g2)9J5k6h3q4#2i4K6t1$3L8X3u0K6M7q4)9K6b7R3`.`.](https://bbs.kanxue.com/elink@875K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2F1j5$3!0J5k6h3y4G2L8Y4y4#2L8s2c8A6L8X3N6Q4x3X3g2U0L8$3#2Q4x3X3g2S2N6g2)9J5y4X3&6T1M7%4m8Q4x3@1t1`.)

 If successful, this converts ecGraph from a demo version to a registered version, removing the demo message printed on each graph.

 Parameter  Type  Description

 S$                  String  S

 See Also..

 Registered

 Example..

 Call `ecGraph3D1.RegisterecGraph3D`(s)

按照它给出的例子，可以用下面的代码进行注册。

```
Const ecGraph3DRegistrationKey$="98765-,3210-ABCDE-01234-56789"

Private Sub Form_Load()

     Dim k As Integer, KeyWords$

     '\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

     '\*\*\* Register all ecGraph3D controls on your form with your registration key obtained

     '\*\*\* from [06aN6%4N6%4i4K6u0W2k6h3&6U0L8%4u0W2j5$3!0F1M7%4g2D9N6r3W2F1k6#2)9J5k6h3y4G2L8g2)9J5k6h3q4#2i4K6u0o6i4K6t1$3L8X3u0K6M7q4)9K6b7X3!0@1K9r3g2J5N6$3W2K6k6g2)9J5y4X3&6T1M7%4m8Q4x3@1u0A6N6q4)9J5y4X3&6T1M7%4m8Q4x3@1u0%4K9h3I4D9i4K6t1$3L8X3u0K6M7q4)9K6b7X3k6#2L8X3y4@1K9h3!0F1i4K6t1$3L8X3u0K6M7q4)9K6b7X3W2F1i4K6t1$3L8X3u0K6M7q4)9K6b7V1c8W2L8h3!0Q4x3U0k6F1j5Y4y4H3i4K6y4n7L8h3!0V1k6b7`.`.](https://bbs.kanxue.com/elink@33eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2F1j5$3!0J5k6h3y4G2L8Y4y4#2L8s2c8A6L8X3N6Q4x3X3g2U0L8$3#2Q4x3X3g2S2N6g2)9J5b7#2)9J5y4X3&6T1M7%4m8Q4x3@1u0G2N6r3S2W2M7Y4N6A6M7$3g2Q4x3U0k6F1j5Y4y4H3i4K6y4n7K9i4c8Q4x3U0k6F1j5Y4y4H3i4K6y4n7N6$3W2D9L8q4)9J5y4X3&6T1M7%4m8Q4x3@1u0X3N6h3&6U0N6r3W2G2L8W2)9J5y4X3&6T1M7%4m8Q4x3@1u0A6L8W2)9J5y4X3&6T1M7%4m8Q4x3@1u0p5k6h3#2G2i4K6t1$3L8X3u0K6M7q4)9K6b7X3#2G2k6r3f1`.).

     '\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

     'in this demo program there are two ecGraph3D controls on the form: ecGraph3D1

     Call ecGraph3D1.RegisterecGraph3D(ecGraph3DRegistrationKey$)

     Me.Caption = "ecGraph3D - Simple demo"

     Call Command1_Click(1)

End Sub
```

跟注册相关的随了`RegisterecGraph3D`方法外，还跟`Registered`属性有关。在VB中使用下面的代码可以知道是否已经注册。

`Registered`     Property

  Property `Registered`() As Boolean 'Read only

  Returns True if this implementation of ecGraph has been successfully registered.

  See Also..

  `RegisterecGraph3D`   `Version`

  Example..

  x = `ecGraph3D1.Registered`

看到这里，控件有一个属性`Registered` ，猜想控件有一个变量用于保存是否注册了的信息，要想验证是不是这样，可以通过跟踪来证实，如果真是这样的话，那么修改这个变量让这个变量恒为true就可以实现爆破了。而跟踪`RegisterecGraph3D`方法就可以得到它的注册算法了，思路就是这简单，下面进行操作。

由于本人不会用VB，就用VC写了一个小程序来访问控件的`Registered`属性和调用它的`RegisterecGraph3D`方法。先用VC创建一个基于对话框的MFC工程，然后依次点“Project”菜单 、“Add To Project”子菜单、“Components And Controls”命令，在弹出的对话框里选择`ecGraph3D_ocx.ecGraph3D.lnk`，如图3所示，再点“Insert”,这样便把该控件增加到工程中了，之后工程中多了一个`C_ecGraph3D`类，而在类中的函数声明中有下面两语句：

```
BOOL GetRegistered();

void RegisterecGraph3D(BSTR* S);
```

![](https://bbs.kanxue.com/upload/attach/200812/28365_d37bvykujeu7oru.jpg)

图3

这两个成员函数就分别对应着控件的`Registered`属性和它的`RegisterecGraph3D`方法，原来VB的Active X控件中的属性和方法是通过类的成员函数来实现的，题外话。呵呵。

对于COM里面某个函数的定位我参考了看雪论坛上的一篇文章《跟踪调试COM组件的接口》，思路是：由于在调用COM里面某个函数时，会由`DispCallFunc`去分发，然后再调用COM里面的函数，那么，我们就在它分发的时候先把它拦截下来，之后再单步跟入某个函数。关键函数：`DispCallFunc`，位于`OLEAUT32.dll`。

对于我写的这个程序也是一样，用`OD`载入生成的.exe文件，在`DispCallFunc`下断，然后，运行程序，断下来后，往下面看，执行到`call ecx`，如下图：

![](https://bbs.kanxue.com/upload/attach/200812/28365_qwb85f30wdyj3rz.jpg)

在这里F7跟进去后，经过一个跳转，便到达了`GetRegistered()`函数，这函数在`IDA`里如下图所示。

![](https://bbs.kanxue.com/upload/attach/200812/28365_uu5o3mnu1wy3ih6.jpg)

注意到这几条汇编语句：

```
.text:1103D7C7         mov     cx, [esi+ecGraph3D.Registered]
.text:1103D7D1         mov     [ebp+var_18], ecx
.text:1103D7DD         mov     eax, [ebp+pblRegistered]
.text:1103D7E0         mov     cx, word ptr [ebp+var_18]
.text:1103D7E4         mov     [eax], cx
```

看到这几句可以知道，控件真的如我们猜想的那样用一个变量（`Registered`）用来标识控件是否已注册，而这个变量的值是`RegisterecGraph3D`函数中被设置的，那么修改相应的汇编代码让这个变量恒为true就可以实现爆破了，而跟踪`RegisterecGraph3D`方法就可以得到它的注册算法。由于我不熟悉VB，而注册算法里用到了很多从`MSVBM60.dll`中导出的如`__vbaI2I4`、`rtcMidCharBstr`、`rtcR8ValFromBstr`、`__vbaFpI2`的函数，这些函数的功能是什么在MSDN里也没有介绍，要知道它的功能只能通过分析函数的代码，这样的工作不是我等菜鸟能做的，希望大侠分析出来后告诉我。在这里我只给出它的爆破方法，而不给出它的注册码，况且论坛也不希望我们在这里散布注册码。

用同样的方法可以定位`RegisterecGraph3D`函数，下面是其代码：

```
.text:11041760 RegisterecGraph3D proc near             ; CODE XREF: .text:loc_110071F1 j
.text:11041760
.text:11041760 var_CC  = dword ptr -0CCh
.text:11041760 var_C8  = dword ptr -0C8h
.text:11041760 var_C4  = dword ptr -0C4h
.text:11041760 var_C0  = dword ptr -0C0h
.text:11041760 var_BC  = dword ptr -0BCh
.text:11041760 var_A8  = word ptr -0A8h
.text:11041760 var_90  = qword ptr -90h
.text:11041760 var_88  = dword ptr -88h
.text:11041760 var_84  = dword ptr -84h
.text:11041760 var_70  = dword ptr -70h
.text:11041760 var_68  = dword ptr -68h
.text:11041760 var_60  = dword ptr -60h
.text:11041760 var_50  = dword ptr -50h
.text:11041760 var_48  = dword ptr -48h
.text:11041760 var_40  = dword ptr -40h
.text:11041760 var_3C  = dword ptr -3Ch
.text:11041760 var_38  = dword ptr -38h
.text:11041760 var_34  = dword ptr -34h
.text:11041760 var_30  = dword ptr -30h
.text:11041760 var_2C  = dword ptr -2Ch
.text:11041760 v_short = dword ptr -28h
.text:11041760 v_str2  = dword ptr -24h
.text:11041760 v_str   = dword ptr -20h
.text:11041760 var_1C  = dword ptr -1Ch
.text:11041760 var_14  = dword ptr -14h
.text:11041760 var_10  = dword ptr -10h
.text:11041760 var_C   = dword ptr -0Ch
.text:11041760 var_8   = dword ptr -8
.text:11041760 this    = dword ptr  8
.text:11041760 pStrcode= dword ptr  0Ch ；这让我不明白了，
.text:11041760 arg_8   = dword ptr  10h ；返回明明是.text:110418CE  retn  8
.text:11041760 arg_C   = dword ptr  14h ；按理说不外就是两个参数吗
.text:11041760 arg_10  = dword ptr  18h ；为什么IDA分析出来有这么多参数的
.text:11041760 arg_14  = dword ptr  1Ch
.text:11041760 arg_18  = dword ptr  20h
.text:11041760 arg_1C  = dword ptr  24h
.text:11041760 arg_20  = dword ptr  28h
.text:11041760
.text:11041760         push    ebp
.text:11041761         mov     ebp, esp
.text:11041763         sub     esp, 14h
```
