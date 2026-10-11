
# Delphi 程序添加功能按钮

> 原文：[`www.pediy.com/kssd/pediy06/pediy6603.htm`](https://www.pediy.com/kssd/pediy06/pediy6603.htm)

这篇文章的目的是展示如何给 Delphi 程序添加按钮并增加事件指针。由于事件指针位于段中间，原空间不足以加入新指针，本文将解决这一问题。

使用 `reshack` 给 `target` 添加 2 个按钮：

```
object Form1: TForm1
  Left = 192
  Top = 107
  Width = 377
  Height = 188
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
    Left = 40
    Top = 32
    Width = 105
    Height = 25
    Caption = 'Button1'
    TabOrder = 0
    OnClick = Button1Click
  end
  object Button2: TButton
    Left = 208
    Top = 32
    Width = 105
    Height = 25
    Caption = 'Button2'
    TabOrder = 1
    OnClick = Button2Click
  end
  object Button3: TButton
    Left = 208
    Top = 72
    Width = 105
    Height = 25
    Caption = 'Button3'
    TabOrder = 1
    OnClick = Button3Click
  end
  object Button4: TButton
    Left = 40
    Top = 72
    Width = 105
    Height = 25
    Caption = 'Button4'
    TabOrder = 1
    OnClick = Button4Click
  end
end
```

添加上述代码后，程序会出现问题导致无法运行，但这不影响后续步骤，继续操作即可。

![](img/9fd810c6a231b9c32281ab5e5acf6f5b.jpg)

上图展示的是事件指针区域，空间非常紧凑。要为新加入的 `Button3` 和 `Button4` 寻找位置加入事件指针几乎不可能。

![](img/29f7d126e604031b5b394e3b23303ecd.jpg)

注意 `00442F70` 指向 `offset 00042370`，`00442F92` 是关键指向 `00042392`。观察上图中的 `00042392`，`0002` 代表这里有 2 个按钮，后面就是 `onclick` 事件指针表。理解这一点后，我们可以将 `0004225c` 处的 `00442F92` 修改为指向一个足够大的空间，然后重新建立 `onclick` 事件指针表，从而实现自定义需求。

![](img/fa178768ec7c733dd28793abcc266312.jpg)

我将 `000425c0` 处修改为指向 `00443272`（`offset 00042672`）的空白处。将以前的事件指针粘贴过来后，如下图所示：

![](img/81edecdf34a6df5d82f85a4daf2c533f.jpg)

将按钮个数修改为 4，并以一个事件指针 0x13 个字节的长度加上 2 个新事件。之后需要为这些事件添加对应的代码。`OnClick = Button3Click` 对应的代码地址为 `00443350`；`OnClick = Button4Click` 对应的代码地址为 `0044335b`。

```
00443350   .  B8 70334400   MOV EAX, modify.00443370
00443355   .  E8 FAF8FFFF   CALL modify.00442C54
0044335A   .  C3            RETN
0044335B   .  B8 80334400   MOV EAX, modify.00443380
00443360   .  E8 EFF8FFFF   CALL modify.00442C54
00443365   .  C3            RETN
00443366     90            NOP
00443367     90            NOP
```

全文完，谢谢。

转载时请尊重看雪论坛和作者，注明转自 `4dd466d0e1e0cc123afbe504da2462ca`
