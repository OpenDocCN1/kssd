
# FastBrowser 3.4

> 原文：[`www.pediy.com/kssd/pediy03/forum864.htm`](https://www.pediy.com/kssd/pediy03/forum864.htm)

去除 `Fast Browser 3.4` (`Normal version`) 广告条

因为我使用 `169` 帐户，此软件不显示广告条，我只是根据广告条位置来大致完成修改工作。

总是习惯用 `file info` 查看程序，结果是 `FastBrowser.exe` 由 `DEPHI` 编写，thank god! `DEPHI` 的应用可能使我的工作更简单。

用 `EXESCOPE` 打开 `FastBrowser.exe`

`resource\\RCData\\TFastBrowserForm` 下看到

```
object AdPanel: TPanel  <--广告条？

  Left = 326           <-

  Top = 25             |

  Width = 464          | 位置、形状

  Height = 60          <-

  Align = alRight

  BevelOuter = bvNone

  ParentColor = True

  TabOrder = 1

  object NewBevel2: TNewBevel

    Left = 4

    Top = -1

    Width = 472

    Height = 61

    Style = deSunken

    Visible = False

  end

  object FlexActive1: TFlexActive

    Left = 0

    Top = 0

    Width = 468

    Height = 60

    TabOrder = 0

    ControlData = {

      000300005F30000034060000000000050800000066006200660076000C000000

      31003100310031003100310000000000000200000001000000000101}

  end

end
```

将上述内容全部删除后保存，广告条没有了，但我不知道实际效果如何？

`4dd466d0e1e0cc123afbe504da2462ca`
