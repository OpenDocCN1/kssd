
# 为Delphi的Form加一个Button

为Delphi的Form加一个Button

kongfoo/2004.6.1

玩Delphi也有几年了，可是真要无中生有，在Form上搞个Button出来却是不知道如何做。。。

今天试一下。先做一个只有2个Button的程序出来分析一下（`button2`的`visible`设为`false`）首先当然是`Dede`出场。`Dede`是Delphi分析方面的专家

看看窗体页，`TForm1`偏移`5aca0`。右边的信息告诉我们有关窗体的属性及各组件的资料全都顺序存放在一起。再在过程页的控件页看看2个Button的ID分别是`2f8`和`2fc`。

## 窗体页资料

``` 
object Form1: TForm1
  Left = 267
  Top = 107
  Width = 235
  Height = 143
  Caption = 'Form1'
  Color = clBtnFace
  Font.Charset = DEFAULT_CHARSET
  Font.Color = clWindowText
  Font.Height = -11
  Font.Name = 'MS Sans Serif'
  Font.Style = []
  OldCreateOrder = False
  PixelsPerInch = 96
  TextHeight = 13
  object Button1: TButton
    Left = 8
    Top = 16
    Width = 75
    Height = 25
    Caption = 'Button1'
    TabOrder = 0
  end
  object Button2: TButton
    Left = 96
    Top = 16
    Width = 75
    Height = 25
    Caption = 'Button2'
    TabOrder = 1
    Visible = False
  end
end
```

`Dede`专家给我们的信息已经很多，好，现在是`UE`出场啦。直接去`5aca0`看看，前面是属性以明文存放，后面是各属性的值（附图）。对着图片和`Dede`的数据分析一下：

![](https://www.kanxue.com/pediy6528/2.gif)

`06 TForm1`

`05 Form1`

`04 Left 03 10b(267)`

`03 Top 02 6b(107)`

`05 Width 03 00eb`

`06 Height 03 008f(143)`

`07 Caption 06 05 Form1`

`05 Color 07 09 clBtnFace`

`0c Font.Charset 07 0f DEFAULT_CHARSET`

`0a Font.Color 07 0c clWindowText`

`0b Font.Name 06 0d MS Sans Serif`

`0a Font.Style 000b`

`0e OldCreateOrder 08`

`0d PixelsPerInch 02 60(96)`

`0a TextHeight 02 0d(13)`

`00`

`07 TButton`

`07 Button1`

`04 Left 02 08`

`03 Top 02 10`

`05 Width 02 4b(75)`

`06 Height 02 19(25)`

`07 Caption 06 07 Button1`

`08 TabOrder 02 00`

`00`

`00`

`07 TButton`

`07 Button2`

`04 Left 02 60(96)`

`03 Top 02 10`

`05 Width 02 4b(75)`

`06 Height 02 19(25)`

`07 Caption 06 07 Button2`

`08 TabOrder 02 01`

`07 Visible 08 0000000000000000`

可见每个属性前面有1字节是标明属性长度的。按钮要以一个字节的0来标明开头和结尾。好，我们现在尝试在`Button2`前面加一个按钮。各属性如下：

```
00
07 TButton
07 Button3
04 Left 02 08
03 Top 02 20
05 Width 02 4b(75)
06 Height 02 19(25)
07 Caption 06 07 Button3
08 TabOrder 02 02
00
```

插入位置为`5add6`。`UE`的复制粘贴可以很方便进行操作。要注意的是不能加多或减少原程序字节数，只能修改。这样修改后虽然程序图标没有了，但可以正常运行，而且按钮也出来了。接下来给按钮增加`OnClick`事件处理。

在Delphi中给`Button1`加个`OnClick`事件，来到`UE`中找`Button1Click`字串（因为按钮属性里面的项目只说明处理函数名字）。找到后分析一下（分析`4ccf2`）：

`000002f8(ID) 0000 07 Button1`

`000002fc(ID) 0000 07 Button2`

`01 0013(hints) 0044d950(RVA) 0c Button1Click`

控件说明在前面，事件说明在后面。要在物理文件中加入我们的代码就要进行虚拟地址到物理地址的转换，拿出`PEeditor`工具，先看看`44d950`在哪个节，第1个节，就用第一个节的`Virtual Offset - Raw Offset = c00`。然后`44d950-ImageBase-c00 =4cd50`。去`4cd50`看看，`e8 b7 b2 ff ff c3`，是正确的函数啦。

好，我们先来加一个按钮。按钮属性：

```
00
07 TButton
07 Button3
04 Left 02 08
03 Top 02 25
05 Width 02 4b(75)
06 Height 02 19(25)
07 Caption 06 07 Button3
08 TabOrder 02 02
07 OnClick 07 0c Button3Click
00
```

在两种`button`属性中间粘贴一个`button`的属性就可以快速完成

再去`4ccf2`加事件处理程序说明。但插入了这么多字节之后，后面的内容都移位了，也就是说，所有程序中用到的后面内容的地址都要改变，这可不是好方案。哪位高人有好方案拿出来让偶菜鸟学习学习

注：文中所用到的位置请按照你的实际情况进行调整。

-------------------------------------------------

工具：`UltraEdit32 5.21` `DeDe 3.50.02` `Delphi7` `winXP`

-------------------------------------------------

上非
