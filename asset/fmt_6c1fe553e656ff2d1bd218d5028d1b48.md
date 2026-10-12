
# 有声有色 3.36

> 原文：[`www.pediy.com/kssd/pediy03/forum503.htm`](https://www.pediy.com/kssd/pediy03/forum503.htm)

方法如下：

1. 这个软件的运行天数放在 `\windows\help\netweb.hlp` 里。

所以你只需删除这个文件便可以继续用了。

2. 但是如果你的已经过期了话，那么还的修改注册表：

```
-----------------------------------------

**Original contents Maybe deleted or modified**

H.L.K\Software\Microsoft\MediaPlayer\Player\CrlMoveTo: 0x00000000

H.L.K\Software\Microsoft\IE Setup\Setup\InstallOption: 0x00000000

H.L.K\Software\CLASSES\CLSID\{00005104-0000-0000-8008-00AA006O2ER4}\51851856985adsf: 0x00000000

H.L.K\Software\Nico Mak Computing\WinZip\WinIni\Setting: 0x00000000

H.L.K\Software\朱江软件有限公司\有声有色 3.12\TextSetting: 0x00000000

H.U\.DEFAULT\Software\Microsoft\Windows\CurrentVersion\Explorer\HotKey: 0x00000000

**Keys&Values Modified | Added in the 2ndShot**

H.L.K\Software\Microsoft\MediaPlayer\Player\CrlMoveTo: 0x00000001

H.L.K\Software\Microsoft\IE Setup\Setup\InstallOption: 0x00000001

H.L.K\Software\CLASSES\CLSID\{00005104-0000-0000-8008-00AA006O2ER4}\51851856985adsf: 0x00000001

H.L.K\Software\Nico Mak Computing\WinZip\WinIni\Setting: 0x00000001

H.L.K\Software\朱江软件有限公司\有声有色 3.12\TextSetting: 0x00000001

H.U\.DEFAULT\Software\Microsoft\Windows\CurrentVersion\Explorer\HotKey: 0x00000001

-----------------------------------------
```

他们中如果你只改一个，程式会给你改回来。要同时改。

3. 所以有这个方法：

在有声有色目录里建个 bat 文件，`ok.bat`

```
del %winbootdir%\help\newweb.hlp
```

然后在有声有色的属性里加上：在程式运行时先运行的批处理 -> `ok.bat`

4. 然后你就可以不用脱壳了，当然你也可以编个程式做以上功能。

   前面加上你的大名，告知这是你的作品，不得随意转载。

5. 付：`iceworld` 好象在白菜发表过这个破解。

6. 如果你是专为脱壳，算我没说。 `4dd466d0e1e0cc123afbe504da2462ca`
