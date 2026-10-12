
# 飓风邮箱清理器

> 原文：[`www.pediy.com/kssd/pediy03/forum310.htm`](https://www.pediy.com/kssd/pediy03/forum310.htm)

去除程序中广告条的一种方法。

`飓风邮箱清理器` 版本 `1.0`

`http://www.renliang.com/software/emailcleaner.exe` 大小 `697k`

安装 `EmailCleaner` 后，运行 `MailCleaner.exe`，发现此程序有广告条，该怎样把它去除呢。先把 `MailCleaner.exe` 和 `MailCleaner.enu` 复制一份，接着反安装 `EmailCleaner`。这时再运行 `MailCleaner.exe`，会提示“Couldn't load `advert.dll`”，这个程序没有加壳，用 `exescope` 之类的程序打开 `MailCleaner.exe`，展开资源下的 `RC DATA` 部分，这里有一个 `TMAINFORM`，选中它在右边即显示出主窗体各个部件的信息，其中有一段：

```
object AdImage1: TAdImage
    Left = 0
    Top = 27
    Width = 468
    Height = 60
    Cursor = crHandPoint
    AutoLoad = True
    Options = []
    ProgramID = 539
    BandwidthThrottle = 2147483647
    MinimumAdDisplayTime = 0
    NetworkIsUp = False
    Size = ADS468x60
end
```

这就是窗口上所显示的广告条，我们把这一段删除，然后保存，现在运行 `MailCleaner.exe`，怎么样，窗体上的广告条没了吧^\_^

这个方法适用于 `Delphi` 和 `C++Bulid` 设计的程序。 `4dd466d0e1e0cc123afbe504da2462ca`
