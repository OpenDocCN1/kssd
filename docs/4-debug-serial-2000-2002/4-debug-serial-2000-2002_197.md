
# 共创校园办公系统

> 原文：[`www.pediy.com/kssd/pediy03/forum561.htm`](https://www.pediy.com/kssd/pediy03/forum561.htm)

首先,大家过年好，本人写了第一篇破解文章   
文采全无，大家当笑话看得了，当是新年礼物。   
常来这里看人家破戒文章，自己也想侃侃而谈写一篇   
本人正在一家公司作开发校园办公系统的工作   
无奈想看看别人程序的功能，我就借来一套别人软件   
`<<共创校园办公系统>>` `http://www.jsccsoft.com`

这东西用 `VB` 写的，是根据硬件信息加的密,密码需要到公司注册，本人用简单的方法使其“不再挑婆家，是男人就嫁”。

## 破戒方法

已装此软件机为: `Y`
要装的机器为: `R`

我看了 `Y` 的注册表找到了点东西

```
REGEDIT4

[HKEY_CURRENT_USER\Software\VB and VBA Program Settings\CCSoft License]
[HKEY_CURRENT_USER\Software\VB and VBA Program Settings\CCSoft License\01001]
"username"="fengyang"
"SerialNumber"="01001-000001-2628"
"LicenseKey"="354288029"
[HKEY_CURRENT_USER\Software\VB and VBA Program Settings\CCSoft License\ProductID]
"0"="01001"
```

把它拷到 `R` 上，Double click the reg file
用安装盘安装 `R`, 你会发现注册号，已填好。
不过装完不能用,(能用就不用写了:)

用 `TRW` 追 `Y` 上的应用程序，发现它们都对一个叫 `NJSSCommLib32.dll` 的小有暧昧的关系
用 `PDLL32 njsscommlib32` 截获

我发现它有一个 export called `GetHDKey`

```
bpx getHDKey
```

when `Trw` repops up and you press `F12` 我用 `D *(esp+4)` 发现 string `"GTQ15890\0"` in data window. 我记下了。

我在 `R` 上试追不能正常运行的应用,当然这次又碰到这个叫 `NJSSCommLib32.dll` 的小子，不过这次它返回不是 string `"GTQ15890\0"`（怪不得我和应用之间夫妻关系不好，第三者插足!）
(我说我这张旧船票还能否登上你的破船)
我趁机将其改回`"GTQ151890\0"`, 嘿!

结果呢？
~~~~涛声依旧了~~~~ :>

（夫妻关系不会恢复正常,重新启动应用还须上述过程
:注册表不用改了,不过 `NJSSCommlib32.dll` 可就......
,所以为 `NJSSCommLib32.dll` 需要做个 `Patch`）
`4dd466d0e1e0cc123afbe504da2462ca`
