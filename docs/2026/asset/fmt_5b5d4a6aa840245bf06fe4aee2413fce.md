
# [原创]华为电脑管家PcManager多屏协同功能破解

让友商电脑兼容了一下华为系专属的多屏协同功能

笔者注：文中出现的大写`C`，请轻声念出北京大爷的口头禅。

更新：目前已经支持美版Matebook等。

华为刚发布多屏协同功能的时候，我就被种草了。

后来一天在微博看到<a href="elink@ad9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0i4K6u0W2N6$3g2A6j5X3!0Q4x3X3g2U0L8W2)9J5c8Y4m8J5L8$3k6A6L8r3g2Q4x3V1j5K6y4e0p5K6x3e0M7I4y4e0t1J5">@Navis-MDT</a>发布的一个体验视频，果然碉堡了。

<a href="elink@cfcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4k6h3W2T1L8#2)9J5k6h3y4G2L8g2)9J5c8Y4c8$3i4K6u0r3N6W2)9J5c8V1W2V1d9g2V1%4k6%4N6G2c8W2)9K6c8X3k6A6k6q4)9K6c8o6p5H3x3K6c8Q4x3@1p5@1y4o6x3J5y4K6p5H3x3o6t1J5y4U0R3^5y4K6b7&6">华为多频协同体验视频</a>

瞬间有点想从“米boy”转为“花粉”用户，这...当然是不可能的了。

翻看微博回复，看到华为手机副总裁<a href="elink@543K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0i4K6u0W2N6$3g2A6j5X3!0Q4x3X3g2U0L8W2)9J5c8Y4g2Q4x3V1j5I4z5e0b7#2z5e0l9$3z5o6b7I4">@李小龙Bruce\_Lee</a>也转发了该视频，并回复网友“为啥不面向非华为电脑”说：“不是故意不做其他品牌的电脑，是电脑上WIFI/蓝牙硬件和驱动太乱了，我们没精力搞定兼容性问题。大神们想破解我们也不会拦着。”

![0.jpg](upload/attach/202001/394321\_EM2GY5MDDGX9KVM.jpg)

对啊，怎么忘了自己也是“大神”啊（羞涩~~）。

既然主人家都不反对，那我帮他们兼容一下，脑子开始发热，迅速准备搞一搞。

是的，脑子非常热，导致前期资讯准备不足，后面踩了很多坑。

下面且听我一一道来。

## 1

视频中提到华为电脑中需要使用“电脑管家”（视频中没看清界面），然后手机扫描连接即可。

听到“电脑管家”第一反应就是华为多屏协同怎么还跟腾讯合作？懵逼，腾讯NB。

后来一想不对，应该是华为自己的软件“电脑管家”，类似于小米笔记本上的“小米游戏盒子”之类的软件。

没有华为电脑，先去搜搜看是否能下载到软件。

![1.png](upload/attach/202001/394321\_JVEVS86M5CJYGDM.jpg)

果然有，华为官网很友好的给出了操作提示，果然是为客户着想的好公司。

![2.png](upload/attach/202001/394321\_2US9YNPXAXPMAXS.jpg)

版本挺多，下一个最新的。然后我选择了`9.1.6.33`，因为我看到日期是`2019-11-01`。（请仔细看图，并记住这里）

![3.png](upload/attach/202001/394321\_HD5HTP8BKNNKVM8.jpg)

## 2

OK，软件下回来了，接着就开始干正事了。

习惯性地把软件拉入了虚拟机运行，结果双击后没反应，以为是鼠标出问题或者虚拟机太卡，连续试了好几次依然没反应。

一瞬间，脑子里翻江倒海，难道华为加了反虚拟机、反调试、反...`C`

直接上windbg吧，一运行，直接退出，才发现是乌龙。

因为这两天在测试DLL劫持，留了个DLL在桌面，华为的“电脑管家”很不幸中枪了，被劫持，然后直接退出。（划重点）

删掉DLL，终于有了反应，但是...提示只支持64位系统，而我用的是`Win7 x86`...`C`

![4.png](upload/attach/202001/394321\_DYF2DSK9JJBPS9V.jpg)

换呗，还能咋办，切到`Win7 x64`虚拟机，问题又来了。

![5.png](upload/attach/202001/394321\_XVXEDGPNKMHSRV7.jpg)

好吧，我错了，我直接本机`win10`安装吧，一分钟不到，终于装好了。

![6.png](upload/attach/202001/394321\_XFR4D3VVU8VWKBH.jpg)

但是从下载到装好时间已经过去了30分钟，都是自己作的。

## 3

点击立即体验，看看界面长啥样。

结果又退出了，退出了...出了...了

咋回事？难道是直接验证到不是华为电脑，直接退出？

继续windbg启动软件，看到如下信息：

```
Util!HttpUtil::OnDebug+0x13f8:
00007fff`0aa7a2c8 cc int 3
0:000> kv
# Child-SP RetAddr : Args to Child : Call Site
00 00000042`558fed10 00007fff`0aa78d2f : 00000219`596f44a0 00000042`558ff339 00000042`558ff2f4 00000042`558ff2f4 : Util!HttpUtil::OnDebug+0x13f8
01 00000042`558fee30 00007fff`0ab05616 : 00000042`558fefe0 00000219`596f44a0 00007fff`0aa60000 00000219`59660000 : Util!mba::util::UrlsManager::ParseXml+0x63f
02 00000042`558fee70 00007fff`0ab04879 : 00000000`00000006 00000000`00000006 00000042`558ff2f4 00000042`558ff339 : Util!CWMI::GetOutPutUIntByString+0x3bb6
03 00000042`558ff1b0 00007fff`0ab04af2 : 00000000`00000000 00000000`00000000 00000042`558ff3c0 00000000`00000000 : Util!CWMI::GetOutPutUIntByString+0x2e19
04 00000042`558ff2d0 00007fff`0aaf53df : 00000000`00000000 7fffffff`ffffffff 00000000`00000000 00000000`00000000 : Util!CWMI::GetOutPutUIntByString+0x3092
05 00000042`558ff3a0 00007ff6`fa838ff3 : 7fffffff`ffffffff 00000000`00000000 7fffffff`ffffffff 00000000`00000000 : Util!SMBIOSHelper::IsSupportDevice+0x1f
06 00000042`558ff400 00007ff6`fa852223 : 00000000`00000000 00000000`0000000a 00000000`00000000 00000000`00000000 : PCManager!SMBIOSHelper::operator=+0xa93
07 00000042`558ffc20 00007fff`4f6f7bd4 : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : PCManager!Compression::Compression+0x3b93
08 00000042`558ffc60 00007fff`5150ced1 : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : KERNEL32!BaseThreadInitThunk+0x14
09 00000042`558ffc90 00000000`00000000 : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : ntdll!RtlUserThreadStart+0x21
0:000> g
ntdll!NtTerminateProcess+0x14:
00007fff`5153c644 c3 ret
```

咋还出现`int3`了呢，难道这里就是检查是不是华为电脑的位置。

使用IDA开始粗略分析，进入`int3`的位置。一处异常，看不出什么东西。

![7.png](upload/attach/202001/394321\_GM66NJSADZ9Y3CH.jpg)

往上回溯两层，看到很重要的信息`SystemEnvironment::GetSupportMachineList`，确实很像是检查的地方。

![8.png](upload/attach/202001/394321\_4PSKEVCFH5YX788.jpg)

对`sub_180018D00`下断，查看参数。发现第三个参数指向一个xml配置文件路径，看名字很明显是表示支持的机器类型列表。

```
0:000> db 000001f0`587329b0
000001f0`587329c0 5c bb aa ce aa 5c 50 43-4d 61 6e 61 67 65 72 5f \....\PCManager\_
000001f0`587329d0 53 65 74 75 70 5f 39 2e-31 2e 36 2e 33 33 5c 48 Setup\_9.1.6.33\H
000001f0`587329e0 75 61 77 65 69 5c 50 43-4d 61 6e 61 67 65 72 5c uawei\PCManager\
000001f0`587329f0 5c 63 6f 6e 66 69 67 5c-4d 61 63 68 69 6e 65 54 \config\MachineT
000001f0`58732a00 79 70 65 4c 69 73 74 2e-78 6d 6c 00 ee fe ee ab ypeList.xml.....
```

打开一看，加密了。

![9.png](upload/attach/202001/394321\_HDQXJUEB2JR5JF3.jpg)

那看看附近代码，应该是先解密，可以把数据dump出来。

通过IDA翻看了一下后面的函数，看到另一个特征`rapidxml::parse_error::vftable`，这是用了`rapidxml`开源库来解析xml文档。

把`rapidxml`的源码下回来，通过对比（一些log字符特征）来确认函数的功能。

![10.png](upload/attach/202001/394321\_HTC5W622T3MKVQ6.jpg)

基本弄清楚xml解密到解析的过程，对`sub_180014EA0`下断，查看参数。果然拿到解密后的xml文件。

```
0:000> dq 000000315e6feb48
00000031`5e6feb48 000001ec`051970b0 00000000`00000000
0:000> db 000001ec`051970b0
000001ec`051970b0 3c 3f 78 6d 6c 20 76 65-72 73 69 6f 6e 3d 22 31 <?xml version="1
000001ec`051970c0 2e 30 22 20 65 6e 63 6f-64 69 6e 67 3d 22 55 54 .0" encoding="UT
000001ec`051970d0 46 2d 38 22 3f 3e 20 0d-0a 3c 6d 61 63 68 69 6e F-8"?> ..<machin
000001ec`051970e0 65 54 79 70 65 6c 69 73-74 3e 20 0d 0a 09 3c 6d eTypelist> ...<m
```

xml数据长这样。

![11.png](upload/attach/202001/394321\_98KTNKZAEYYFKDX.jpg)

看到这里，其实已经有一个思路了，就是把自己电脑的品牌加入xml文件，重新加密回去，就能够就能够使用华为电脑管家了。

不过分析解密、拿品牌信息、加密等挺麻烦的，再看看还有方法吗？

比如直接patch比较品牌的函数，绕过检查。

继续分析，一堆xml的node的解析比较，因为华为在编译`rapidxml`时基本都是内联函数，导致跟源码结构并不是很一致，增加了分析难度。

经过一段数据的比较，还没有找到绕过的点。

我停了下来，看了看调用栈。

```
0:000> kv
# Child-SP RetAddr : Args to Child : Call Site
00 00000031`5e6fea20 00007fff`11184879 : 00000000`00000006 00000000`00000006 00000031`5e6feea4 00000031`5e6feee9 : Util!CWMI::GetOutPutUIntByString+0x4075
01 00000031`5e6fed60 00007fff`11184af2 : 00000000`00000000 00000000`00000000 00000031`5e6fef70 00000000`00000000 : Util!CWMI::GetOutPutUIntByString+0x2e19
02 00000031`5e6fee80 00007fff`111753df : 00000000`00000000 7fffffff`ffffffff 00000000`00000000 00000000`00000000 : Util!CWMI::GetOutPutUIntByString+0x3092
03 00000031`5e6fef50 00007ff7`70a18ff3 : 7fffffff`ffffffff 00000000`00000000 7fffffff`ffffffff 00000000`00000000 : Util!SMBIOSHelper::IsSupportDevice+0x1f
04 00000031`5e6fefb0 00007ff7`70a32223 : 00000000`00000000 00000000`0000000a 00000000`00000000 00000000`00000000 : PCManager!SMBIOSHelper::operator=+0xa93
05 00000031`5e6ff7d0 00007fff`4f6f7bd4 : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : PCManager!Compression::Compression+0x3b93
06 00000031`5e6ff810 00007fff`5150ced1 : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : KERNEL32!BaseThreadInitThunk+0x14
07 00000031`5e6ff840 00000000`00000000 : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : ntdll!RtlUserThreadStart+0x21
```

才看到被我忽略的一个很重要的信息`Util!SMBIOSHelper::IsSupportDevice`，这个函数名太明显了。

只需要修改这个返回值，基本应该就绕过检查了。

```
BOOL8 __fastcall SMBIOSHelper::IsSupportDevice(SMBIOSHelper *this)
{
return 1; //patch
}
```

是的，我好像又掉坑里，浪费了挺多时间。

## 4

但是前面为什么会出现异常呢，导致我掉进了一个确实是对的但又有点坑的位置呢。

想到xml的路径，此时我终于反应过来哪个异常可能是什么了。

因为我把软件放在了中文目录中，而软件对中文处理貌似有问题。（我没有深究了）

```
D:\华为\PCManager\_Setup\_9.1.6.33
```

果然，去掉中文目录后，再没有上面看到的`int3`。

所以，这又是个坑。`C`

本来正常的分析逻辑是这样的。
