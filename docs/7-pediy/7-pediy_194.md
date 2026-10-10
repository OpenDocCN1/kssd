

# delphi 程序添加功能按钮

> 原文：[`www.pediy.com/kssd/pediy06/pediy6603.htm`](https://www.pediy.com/kssd/pediy06/pediy6603.htm)

这篇文章的目的在如何给 delphi 的程序加上按钮后如何给他增加事件指针。由于事件指针位于段中间，原空间上不够我们加入指针。这篇文章就是要解决这个问题。  
用 reshack 给 target 加上 2 个按钮：  
object Form1: TForm1  
  Left = 192  
  Top = 107  
  Width = 377  
  Height = 188  
  Caption = 'Form1'  
  Color = clBtnFace  
  Font.Charset = DEFAULT\_CHARSET  
  Font.Color = clWindowText  
  Font.Height = -11  
  Font.Name = 'MS Sans Serif'  
  Font.Style = \[\]  
  OldCreateOrder = False  
  PixelsPerInch = 96  
  TextHeight = 13  
  object Button1: TButton  
    Left = 40  
    Top = 32  
    Width = 105  
    Height = 25  
    Caption = 'Button1'  
    TabOrder = 0  
    OnClick = Button1Click  
  end  
  object Button2: TButton  
    Left = 208  
    Top = 32  
    Width = 105  
    Height = 25  
    Caption = 'Button2'  
    TabOrder = 1  
    OnClick = Button2Click  
  end  
  object Button3: TButton  
    Left = 208  
    Top = 72  
    Width = 105  
    Height = 25  
    Caption = 'Button3'  
    TabOrder = 1  
    OnClick = Button3Click  
  end  
  object Button4: TButton  
    Left = 40  
    Top = 72  
    Width = 105  
    Height = 25  
    Caption = 'Button4'  
    TabOrder = 1  
    OnClick = Button4Click  
  end  
end  
如上添加好后程序会出问题导致不能运行，不要管他我们继续。

  
![](img/9fd810c6a231b9c32281ab5e5acf6f5b.jpg)

  
上图是事件指针，这里的空间很紧凑了。要想给我们加入的按钮 Button3、Button4 找地方加入事件指针是不大可能了，呵呵……

  
![](img/29f7d126e604031b5b394e3b23303ecd.jpg)

  
注意 00442F70 他指向 offset 00042370，00492F92 是关键指向 00042392，看上图的 00042392 是什么？0002 代表这里有 2 个按钮，后面就是 onclick 事件指针了表了。到这里明白了吗？我们把 0004225c 处的 00442F92 改到指向一个足够大的地方后我们重新建个 onclick 事件指针表就可以为所欲为了，^\_^

  
![](img/fa178768ec7c733dd28793abcc266312.jpg) 

  
我把 000425c0 处改为指向 00443272（offset00042672）的空白处。把以前的事件指针粘贴过来后，如下图：

  
![](img/81edecdf34a6df5d82f85a4daf2c533f.jpg) 

  
我们把按钮个数改为 4，然后以一个事件指针为 0x13 个字节的长度加上 2 个事件吧！：）ok！以后为该事件加对应代码吧。OnClick = Button3Click 的对应代码地址 00443350；OnClick = Button4Click 的是 0044335b  
00443350   .  B8 70334400   MOV EAX,modify.00443370  
00443355   .  E8 FAF8FFFF   CALL modify.00442C54  
0044335A   .  C3            RETN  
0044335B   .  B8 80334400   MOV EAX,modify.00443380  
00443360   .  E8 EFF8FFFF   CALL modify.00442C54  
00443365   .  C3            RETN  
00443366      90            NOP  
00443367      90            NOP

  
OK,全文完。谢谢  

织或团体个人转载时，请你尊重一下看雪论坛和作者，注明转自  4dd466d0e1e0cc123afbe504da2462ca