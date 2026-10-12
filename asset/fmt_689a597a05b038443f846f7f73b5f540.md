
# 对付隐藏目录的加密光盘

> 原文：[`www.pediy.com/kssd/pediy02/forum-crack08.htm`](https://www.pediy.com/kssd/pediy02/forum-crack08.htm)
>
> 作者：Sun Bird \[CCG\]（我，Sun Bird 属于破解团体 China Cracking Group ^\_^）
>
> 日期：2000 年 3 月 22 日

现在有很多光盘都加了密，直接查看只有几个数百 K 的莫名其妙的文件，运行后要求输入密码，当密码正确时会调出光盘上的真正文件；或是象“DDR 跳舞碟”运行后自行调用音乐、足谱……明知道光盘内有上百兆的内容，可就是找不到，真是气死人！>:-(

这些使用隐藏目录方式加密的光盘，记得最早似乎是从一些漫画书类光盘开始的。这些光盘必须使用“冷雨浏览器”，而且必须输入正确的密码才能观看。

如果以前玩过 DOS 的朋友可能会记得，更改硬盘目录的最后几个字节就可以达到隐藏目的，而且使用 `PC Tools` 也看不到。采用这种方式加密并不影响使用，只要知道目录名，可以直接使用`CD XXX`命令进入加密子目录。本人认为，采用这种方式加密光盘，其原理也类似（个人愚见，如有错误，恳请高手指点）。同样，对这类光盘也可以直接`CD XXX`进入（但需要在 `DOS 7.X` 环境下）！

那么，如何才能知道加密的目录名称呢？

其实并不难，只要利用一个小软件“`File Monitor`”即可！该软件不大，是纯“绿色”免费软件，可在各大软件下载站点找到。下载的是一个压缩包，将包内文件全部解压至一个目录，自行建立快捷方式即可。目前最新版本为“`File Monitor for Windows NT/9x v4.27`”。

下面以某新版“DDR 跳舞碟”（众所周知，旧版的“DDR 跳舞碟”其加密目录为`DDR99\`，而新版的……）为例，Follow me.....

1. 运行软件`FileMon`，在`Options`内将`Capture Events`打上勾；

2. 运行“DDR 跳舞碟”，当选择的舞曲已调入内存后即可以退出“DDR”；

3. 回到`FileMon`，看到什么了？对！所有的文件调用均被记录下来啦！现在再将`Capture Events`前面的勾去掉（免得仍旧不断的增加记录），然后来看看记录的都是什么（当然，你也可以将记录保存，使用文本编辑器慢慢查看）：

```
………… ……… …………………
………… ……… …………………
Explorer FindOpen E:\DDR99.EXE SUCCESS
Explorer FindClose E:\DDR99.EXE SUCCESS
………… ……… …………………
………… ……… …………………
Ddr99 Open E:\IMAGS\BACK.BMP SUCCESS
Ddr99 Attributes E:\IMAGS\BACK.BMP SUCCESS
………… ……… …………………
………… ……… …………………
Ddr99 FindOpen E:\WAVE\FOOT.WAV NOTFOUND
Ddr99 FindOpen E:\WAVE\TITLE.WAV NOTFOUND
………… ……… …………………
………… ……… …………………
Ddr99 FindOpen E:\MSD\*.MSD SUCCESS
Ddr99 FindNext E:\MSD\*.MSD SUCCESS
………… ……… …………………
………… ……… …………………
Ddr99 Open E:\MUSICTITLE\TRACK_01.BMP SUCCESS
Ddr99 Attributes E:\MUSICTITLE\TRACK_01.BMP SUCCESS
………… ……… …………………
………… ……… …………………
Ddr99 FindOpen E:\BGM\TRACK_01S.WAV NOMORE
Ddr99 FindOpen E:\BGM\TRACK_01S.WAV NOMORE
………… ……… …………………
………… ……… …………………
Ddr99 FindOpen E:\BGM\S.WAV NOMORE
Ddr99 FindOpen E:\BGM\S.WAV NOMORE
………… ……… …………………
………… ……… …………………
Ddr99 Open E:\BGM\TRACK_01.WAV SUCCESS
Ddr99 Seek E:\BGM\TRACK_01.WAV SUCCESS
………… ……… …………………
………… ……… …………………
```

一切显而易见了，原来新版的“DDR 跳舞碟”其加密子目录为`BGM\`（因为记录的文件调用非常多，所以我将大部分省略了。）！

好啦，可以将喜欢的曲目拷贝下来后去退碟了?!##?@#$#@%&

对了，“`FileMon`”当然不仅可以做这点小 Case，利用它你还可以破解软件！

有些软件在运行时会读取某些“暗贴”，利用“`FileMon`”就可以了解该软件到底调用了哪些文件，这样可以集中精力对付可疑的文件调用。

好了、好了，收工，休息！

`4dd466d0e1e0cc123afbe504da2462ca`
