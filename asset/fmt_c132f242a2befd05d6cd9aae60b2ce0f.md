
# ThumbNailer 6.6.1.1

> 原文：[`www.pediy.com/kssd/pediy03/forum589.htm`](https://www.pediy.com/kssd/pediy03/forum589.htm)

软件名称：`ThumbNailer`   
整理日期：2001.1.29   
最新版本：`6.6.1.1`   
文件大小：`1324KB`   
软件授权：共享软件   
使用平台：`Win9x/Me/NT/2000`   
发布公司：`Home Page`

软件简介：

能够为一个满是图形的目录建立图片的缩略图，并建立一个`HTML`连接这些图片，支持`JPG`、`BMP`、`PNG`、`PCX`、`TGA`、`AVI` frame-extraction、`GIF`(non-LZW output)、`TIFF`(non LZW)、`WMF`(read-only)等格式。

下载地址：`http://www.newhua.com/THUMBNAILER.HTM`

破解工具：`HIEW`，`W32DASM`

这个软件没有输入注册码的地方,在左下解看到`30 DAYS LEFT IN TRIAL`用是我将时间跳到`2002`年后,再次运行它,看到`YOUR 30-DAY TRAL PERIOD HAS RUN OUT.`的提示,这就是入口点

用`W32DASM`反汇编后找到这句后

* `Possible StringData Ref from Data Obj ->"Your %d-day trial period has run "`-----------------破解它的入口点,向上找

->`"out.`

`Visit http://www.smalleranimals.com "`

->`"to register this application."`

`|`

`push 00590A30`

`lea edx, dword ptr [ebp-14]`

`push edx`

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`:00475B65(C)----------------------------------------------------------找到这个就行了`

`|`

`push 00000001`

`mov ecx, dword ptr [ebp-24]`

`call 00475897`

`mov ecx, 0059F530`

`call 00495729`

`mov ecx, dword ptr [0059F534]`

`00075B65`

`sub esp, 00000018`

`mov dword ptr [ebp-24], ecx`

`xor eax, eax`

`mov al, byte ptr [0059F54C]`

`test eax, eax`

`je 00475B76-------------------这个 JE 就是鬼啦!将 JE 改为 JNE 怎么样!也就是说将 74 改为 75 就成功了`

`mov ecx, 0059F530 4dd466d0e1e0cc123afbe504da2462ca`
