
# 为 `delphi` 程序添加功能按钮

> 原文：[`www.pediy.com/kssd/pediy06/pediy6539.htm`](https://www.pediy.com/kssd/pediy06/pediy6539.htm)

为 `delphi` 程序添加功能按钮

实例下载（或鼠标右键另存为）

我们知道，`delphi` 程序是将控件的属性放入 `RCDATA` 区，但将执行控件事件的事件指针及具体代码置入 `.code` 段，两者分离。

存放控件属性的二进制代码位于程序的后段，`.code` 段位于程序中段；所以添加控件属性不会有影响，但如果要加入控件对应事件指针，因为其位于程序中段，可能导致程序的运行错误。目前上无文章阐述此问题的解决。

`target.exe` 是 `delphi5` 写的一个程序，上面有 2 个按钮，并具备执行相应事件。我将尝试再加入一个按钮 `Button3`，并能执行 `Button2Click` 的同样功能。

## 1. 添加按钮属性

`exescope` 打开 `target.exe`，在 `Resource` 的 `RCDATA` 下 `TForm1` 见到控件属性

......

```
object Button2: TButton
    Left = 208
    Top = 32
    Width = 105
    Height = 25
    Caption = 'Button2'
    TabOrder = 1
    OnClick = Button2Click
  end
```

我们在后面复制 `Button2` 控件属性，并做适当修改后保存为 `Button3`。

```
object S: TButton
    Left = 40
    Top = 72
    Width = 105
    Height = 25
    Caption = 'Button3'
    TabOrder = 2
    OnClick = T
  end
```

## 2. 添加 `button3` 事件指针

用 `hex workshop` 打开修改后的 `target.exe` 程序，搜索 `Button2Click`，见到如下代码

![](img/98a9ef5ffa8cb5daa8091f9f9e1c1aca.jpg)

以上是 `delphi` 控件事件指针表，可以由以下结构表述：

```
Type TControlProcedurePtr = packed record
     PtrCount  : Word;
     Ptr1      : TPtr;
     Ptr2      : TPtr;
     ......
     PtrN      : TPtr;    // n = PtrCount
     FrmName   : PChar;
end;

type TPtr = packed record
     TPtrLen    : Word;
     Ptr        : Dword;
     PtrName    : PChar;
end;
```

添加事件指针，最重要的就是计算清楚：如何通过缩小原有 `TPtr` 结构的长度，使得加入新指针 `TPtr` 结构后整个 `TControlProcedurePtr` 长度仍旧保持不变。

通过上述结构，我们知道 `TPtr` 结构最短长度为 8 字节。这里 `Button3.Name:='S'`，所以需要在前面 `Button1`、`Button2` 事件指针列表中挖出 8 个字节。我选择将

`OnClick = Button1Click` 改为 `OnClick = Button1C`
`OnClick = Button2Click` 改为 `OnClick = Button2C`

添加 `OnClick = T`

![](img/c97eb007c2545bf48c199585eb3f237b.jpg)

`4dd466d0e1e0cc123afbe504da2462ca`
